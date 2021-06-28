该篇是集合了百度众多的日志框架详解，java日志框架分析的总结篇。 
具体网址：

```
https://blog.csdn.net/foreverling/article/details/51385128
https://blog.csdn.net/chszs/article/details/8653460
http://baijiahao.baidu.com/s?id=1585361583532845302&wfr=spider&for=pc
http://www.importnew.com/28541.html
https://blog.csdn.net/victor_cindy1/article/details/78716917
https://zhuanlan.zhihu.com/p/24275518
https://my.oschina.net/aiguozhe/blog/87981
http://www.importnew.com/28541.html
https://blog.csdn.net/xktxoo/article/details/76359299
```


主要的就是这些了。 
java里常见的日志库有java.util.logging(JDKlog)、Apache log4j、log4j2、logback、slf4j等等。 
这么多的日志框架里如何选择。 
首先需要梳理的是日志接口，以及日志接口对应的实现。然后再考虑如何选择使用哪个日志框架。 




由图可知我们经常使用的日志框架实质上是一些接口的实现。而目前使用较广泛的统一日志规范接口就是slf4j和commons-logging。

slf4j
   它是基于API的java日志框架，slf4j提供了简单统一的日志记录的接口，开发者在配置部署
时只需要是吸纳这个接口就能是实现日志功能。它自身并没有提供具体的日志解决方案，它是负责
服务于各种各样的日志系统，允许用户在部署应用上使用自己常用的日志框架。
    也就是说，SLF4j是一个抽象层，它提供了众多的适配器能是配合其他所有开源日志框架。
为了考虑其他项目会使用大量的第三方库，而第三方库使用的日志框架又各不相同，不同的日志框
架又需要不同的配置，不同配置就会导致日志输出到不同的位置。所以我们就需要一个可以将日志
level、日志输出等统一管理，而slf4j的适配器又对各种日志都实现了接管，接管后就可以统一
配置这些第三方库中使用的日志。
1
2
3
4
5
6
7
8
commons-logging
commons-logging(jcl)也为众多日志实现库提供了统一的接口，作用和slf4j类似。它允许运行时绑定
任意的日志库。但commons-loggins对log4j和java.util.logging的配置问题兼容性不太好，还会
遇到类加载问题。所以当时log4j的作者又创作了slf4j.....
1
2
3
有了日志接口，就需要选择实现框架。在众多的日志框架如何选择

log4j
Log4j是apache下一个功能非常丰富的java日志库实现，Log4j应该是出现比较早而且最受欢迎的java
日志组件，它是基于java的开源的日志组件。Log4j的功能非常强大，通过Log4j可以把日志输出到控制
台、文件、用户界面。也可以输出到操作系统的事件记录器和一些系统常驻进程。值得一提的是：Log4j可
以允许你非常便捷地自定义日志格式和日志等级，可以帮助开发人员全方位的掌控自己的日志信息是一个日
志开源框架。
1
2
3
4
5
logback
logback是log4j的改进版本，而且原生支持slf4j(因为是同一作者开发)。所以logback+slf4j的组
合是日志框架的最佳选择。logback解决了log4j不能使用占位符的问题。logback可以通过jmx修改日
志配置，可以从jmx控制台直接操作，无需重启应用程序。它相比于log4j有更多的优点:

 - 原生实现了slf4j API(log4j需要中间层转换)
 - 支持xml、Groovy方式配置
 - 支持配置文件中加入条件判断
 - 更强大的过滤器
 - 更充分的测试
 - 更丰富的免费文档
 - 自动重载有变更的配置文件
 - 自动压缩历史日志
 等等
  1
  2
  3
  4
  5
  6
  7
  8
  9
  10
  11
  12
  13
  14
  Log4j2
  log4j2是log4j的升级版本2.x但并不兼容log4j因为它基本上把log4j版本的核心全部重构掉了，log4j2设计上很大程度模仿了slf4j-logback，性能也
  获得了很大的提升。具体的差异自行百度。
  1
  2
  至于网上不少说的性能比较，应该都是单个框架比较。但与slf4j绑定使用的话，怎么说都是logback性能最佳吧（猜测，欢迎指正）

日志级别
日志level定义应该遵循以下几个：

 - debug: 完整详细的记录流程的关键路径，用于开发人员比较感兴趣的跟踪和调试信息，生产环境中正常不会打开debug状态
 - info: 应简洁明确让管理员确定状态，记录相当重要有意义的信息。关键的系统参数的回显、后台服务的初始化状态、需要系统管理员确认的关键信息都需要使用info级别
 - warn: 能清楚告知发生了什么情况，指示潜在问题能引起别人重视，但不一定需要处理
 - error: 系统出现异常或不希望出现的问题，能及时得到关注处理。但也不是所有的异常都记录成error
1
2
3
4
5
6
7
根据上述，这里将以slf4j为日志接口，而其中logback与slf4j配合桥接性能最优，所以在日志实现上选择logback。 
此处以maven进行管理，首先引入依赖，这两个依赖就够了，版本可根据需求修改

    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>1.7.22</version>
    </dependency>
     <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.1.6</version>
    </dependency>
1
2
3
4
5
6
7
8
9
10
11
classpath中编写logback的配置文件logback.xml, logback.xml文件放在/src/main/resources/路径中

logback.xml

<?xml version="1.0" encoding="UTF-8"?>
<!-- 
scan:当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。 
scanPeriod:设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。
-->
<configuration scan="true" scanPeriod="60 seconds" debug="false">

    <!-- 上下文变量设置,用来定义变量值,其中name的值是变量的名称，value的值时变量定义的值。
        通过<property>定义的值会被插入到logger上下文中。定义变量后，可以使“${}”来使用变量。 -->
    <property name="CONTEXT_NAME" value="logback-test" />
    
    <!-- 上下文名称：<contextName>, 每个logger都关联到logger上下文，
        默认上下文名称为“default”。但可以使用<contextName>设置成其他名字，用于区分不同应用程序的记录。
        一旦设置，不能修改。 -->
    <contextName>${CONTEXT_NAME}</contextName>
    
    <!-- <appender>是<configuration>的子节点，是负责写日志的组件。
        有两个必要属性name和class。
        name指定appender名称，
        class指定appender的实现类。 -->
    <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <!-- 对日志进行格式化。 -->
        <encoder>
            <pattern>
                %d{yyyy-MM-dd HH:mm:ss.SSS}|%level|%class|%thread|%method|%line|%msg%n
            </pattern>
        </encoder>
    </appender>
    
    <appender name="file" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。 -->
        <file>${logs.dir}/logback-test.log</file>
    
        <!-- 当发生滚动时的行为  -->
        <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
            <!-- 必须包含“%i”例如，假设最小值和最大值分别为1和2，命名模式为 mylog%i.log,会产生归档文件mylog1.log和mylog2.log。还可以指定文件压缩选项，例如，mylog%i.log.gz 或者 没有log%i.log.zip -->
            <FileNamePattern>${logs.dir}/logback-test.%i.log</FileNamePattern>
            <!-- 窗口索引最小值 -->
            <minIndex>1</minIndex>
            <!-- 窗口索引最大值 -->
            <maxIndex>1</maxIndex>
        </rollingPolicy>
    
        <!-- 激活滚动的条件。 -->
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <!-- 活动文件的大小，默认值是10MB -->
            <maxFileSize>30MB</maxFileSize>
        </triggeringPolicy>
    
        <!-- 对记录事件进行格式化。 -->
        <encoder>
            <charset>UTF-8</charset>
            <Pattern>
                %d{yyyy-MM-dd HH:mm:ss.SSS}|%level|%class|%thread|%method|%line|%msg%n
            </Pattern>
        </encoder>
    </appender>
    
    <!-- 特殊的<logger>元素，是根logger。只有一个level属性，应为已经被命名为"root".
        level:设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，不能设置为INHERITED或者同义词NULL。默认是DEBUG。
        <root>可以包含零个或多个<appender-ref>元素，标识这个appender将会添加到这个loger。 -->
    <root>
        <level value="WARN" />
        <appender-ref ref="stdout" />
        <appender-ref ref="file" />
    </root>
    
    <!-- 用来设置某一个 包 或者具体的某一个 类 的日志打印级别、以及指定<appender>, 
        name:用来指定受此logger约束的某一个包或者具体的某一个类。
        level:用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，还有一个特俗值INHERITED或者同义词NULL，代表强制执行上级的级别。如果未设置此属性，那么当前loger将会继承上级的级别。 
        additivity:是否向上级logger传递打印信息。默认是true。(这个logger的上级就是上面的root)
        <logger>可以包含零个或多个<appender-ref>元素，标识这个appender将会添加到这个logger。-->
    <logger name="xuyihao.logback.test" level="DEBUG" additivity="true"></logger>

</configuration>
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75

点赞
1

评论

分享

收藏
13

举报
关注
一键三连

java日志 配置文件
12-25
java日志 配置文件 包含配置文件各个参数的定义及参数含义
log4j.properties日志配置文件
06-21
这是一个log4j配置文件，可以在控制台打印输出debug信息，方便项目调试，无需修改，拿来即用，放在项目的classpath目录下即可。

————————————————
版权声明：本文为CSDN博主「食火的埃尔德里奇」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_26917447/article/details/80447653

https://blog.csdn.net/qq_26917447/article/details/80447653