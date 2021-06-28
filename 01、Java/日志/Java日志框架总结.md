## 1.　前言

　　从写代码开始，就陆陆续续接触到了许多日志框架，较常用的属于LOG4J，LogBack等。每次自己写项目时，就copy前人的代码或网上的demo。配置log4j.properties或者logback.properties就能搞定。这种思想一直持续到最近，前几天写了一个小demo，放在liunx上跑的时候竟然报stackOverFlow异常,仔细看异常信息，log4j-over-slf4j与slf4j-log4j12共存导致stack overflow异常，这一刻终于来了，我不得不去正式日志框架了。

## 2.　解决方案

　　先贴一下我的解决方案。既然报冲突了，不管三七二十一，先删除一个jar包就OK！第一种解决方案直接在项目的WEB-INF\lib下删除slf4j-log4j12.jar包。

```
cd /WEB-INF/lib
rm -rf slf4j-log4j12-1.7.25.jar
```

　　这种方案只是临时方案，再次打包部署时还会出现此问题。第二种方案是在pom.xml找到引用此JAR包的地方，我是因为引入了zookeeper，自带了此JAR包。使用<exclusion>标签排除此JAR包。或者使用<packagingExcludes>标签在打包时排除此JAR包。个人推荐使用<packagingExcludes>，因为我的<exclusion>标签不管用，此处不做详解，直接贴代码！

```
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.6</version>
    <exclusions>
        <exclusion>
            <artifactId>slf4j-log4j12</artifactId>
            <groupId>org.slf4j</groupId>
        </exclusion>
     </exclusions>
</dependency>
```

```
<plugin>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-war-plugin</artifactId>
<configuration>
    <packagingExcludes>WEB-INF/lib/slf4j-log4j12-1.7.25.jar</packagingExcludes>
</configuration>
</plugin>
```



## 3.　基本日志框架之间关系

　　类比java的面向接口编程思想，JAVA日志框架分为接口层，实现层，还多了个桥接层，桥接层联系接口层和实现层。

　　![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517171920.png)

　　接口层：SELF4J,COMMONS-LOGGING

　　实现层：LOG4J,LOGBACK,JDK-LOOGING,LOG4J2

　　以上为通用的日志框架实现（即实现）和门面（即接口）。日志门面的出现很大程度缓解了日志系统的混乱，很多库的作者不在使用具体的日志框架实现了，而是去使用接口层，即面向接口编程。此处，贴一段话，方面更能理解。

　　应用程序直接使用这些具体日志框架的API来满足日志输出需求当然是可以的，但是由于各个日志框架之间的API通常是不兼容的，这样做就使得应用程序丧失了更换日志框架的灵活性。比直接使用具体日志框架API更合理的选择是使用日志门面接口。日志门面接口提供了一套独立于具体日志框架实现的API，应用程序通过使用这些独立的API就能够实现与具体日志框架的解耦，这跟JDBC是类似的。最早的日志门面接口是commons-logging，但目前最受欢迎的是slf4j。日志门面接口本身通常并没有实际的日志输出能力，它底层还是需要去调用具体的日志框架API的，也就是实际上它需要跟具体的日志框架结合使用。由于具体日志框架比较多，而且互相也大都不兼容，日志门面接口要想实现与任意日志框架结合可能需要对应的桥接器，就好像JDBC与各种不同的数据库之间的结合需要对应的JDBC驱动一样。

## 4.　StackOverFlow异常分析

　　![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517171923.png)

　　上图来自SLF4J官网。如图，上层都是使用SLF4JAPI对外暴露接口。SLF4JAPI使用的是slf4j-api.jar包。接着下层各个日志框架的实现就不一样了，最左边的是slf4j的一个空实现。第二列和第五列是logback和slf4j的一个简单实现，这2个框架没有使用所谓的桥接器，直接继承slf4j，实现slf4j的接口。log4j和jul的实现是要依靠桥接器，如上，slf4j-log412.jar和slf4j-jdk14.jar就是桥接器，分别连接slf4j，log4j和slf4j，jul。下面的log4j.jar和JVM runtime便是具体实现。

　　其他日志系统转掉回slf4j，如果只存在slf-4j转到日志系统实现类，便不会存在StackOverFlow的异常。如果我们使用log4j日志系统，但又想使用别的日志系统，此时就要使用从日志系统到slf4j的桥接类 log4j-over-slf4j，这个库定义了与log4j一致的接口（包名、类名、方法签名均一致），但是接口的实现却是对slf4j日志接口的包装，即间接调用了slf4j日志接口，实现了对日志的转发。

　　既然存在这么多桥接器，万一我的系统中存在slf4j -> log4j 和 log4j -> slf4j的桥接器。。。。就会出现互相委托，无限循环，堆栈溢出的状况。所有就会有出现异常。slf4j关于桥接器的详细介绍参考slf4j官方网站：https://www.slf4j.org/legacy.html

　　比如，我现在想使用slf4j的实现类logback日志框架

　　1.　引入slf4j & logback日志包和slf4j -> logback桥接器；

　　2.　排除common-logging、log4j、log4j2日志包；

　　3.　引入jdk-logging -> slf4j、common-logging -> slf4j、log4j -> slf4j、log4j2 -> slf4j桥接器；

  　　4.　排除slf4j -> jdk-logging、slf4j -> common-logging、slf4j -> log4j、slf4j -> log4j2桥接器。

## 参考

https://www.cnblogs.com/xiaobingblog/p/11502976.html