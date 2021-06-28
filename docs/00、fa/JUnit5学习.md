# 基本操作

### 关于JUnit5

1. JUnit是常用的java单元测试框架，5是当前最新版本，其整体架构如下(图片来自网络)：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154624.png)
2. 从上图可见，整个JUnit5可以划分成三层：顶层框架(Framework)、中间的引擎（Engine），底层的平台（Platform）；
3. 官方定义JUnit5由三部分组成：Platform、Jupiter、Vintage，功能如下；
4. Platform：位于架构的最底层，是JVM上执行单元测试的基础平台，还对接了各种IDE（例如IDEA、eclipse），并且还与引擎层对接，定义了引擎层对接的API；
5. Jupiter：位于引擎层，支持5版本的编程模型、扩展模型；
6. Vintage：位于引擎层，用于执行低版本的测试用例；

- 可见整个Junit Platform是开放的，通过引擎API各种测试框架都可以接入；

### SpringBoot对JUnit5的依赖

1. 这里使用SpringBoot版本为2.3.4.RELEASE，在项目的pom.xml中依赖JUnit5的方法如下：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

1. 如下图红框，可见JUnit5的jar都被spring-boot-starter-test间接依赖进来了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154628.jpeg)

### 曾经的RunWith注解

1. 在使用JUnit4的时候，咱们经常这么写单元测试类：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class XXXTest {
}
```

1. 对于上面的RunWith注解，JUnit5官方文档的说法如下图红框所示，已经被ExtendWith取代：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154635.jpeg)
2. 咱们再来看看SpringBootTest注解，如下图，可见已经包含了ExtendWith：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154631.jpeg)
3. 综上所述，SpringBoot+JUnit5时，RunWith注解已经不需要了，正常情况下仅SpringBootTest注解即可，如果对扩展性有更多需求，可以添加ExtendWith注解，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154638.jpeg)

### 常用的JUnit5注解(SpringBoot环境)

注意，接下来提到的测试方法，是指当前class中所有被@Test、@RepeatedTest、@ParameterizedTest、@TestFactory修饰的方法；

1. ExtendWith：这是用来取代旧版本中的RunWith注解，不过在SpringBoot环境如果没有特别要求无需额外配置，因为SpringBootTest中已经有了；
2. Test：被该注解修饰的就是测试方法；
3. BeforeAll：被该注解修饰的必须是静态方法，会在所有测试方法之前执行，会被子类继承，取代低版本的BeforeClass；
4. AfterAll：被该注解修饰的必须是静态方法，会在所有测试方法执行之后才被执行，会被子类继承，取代低版本的AfterClass；
5. BeforeEach：被该注解修饰的方法会在每个测试方法执行前被执行一次，会被子类继承，取代低版本的Before；
6. AfterEach：被该注解修饰的方法会在每个测试方法执行后被执行一次，会被子类继承，取代低版本的Before；
7. DisplayName：测试方法的展现名称，在测试框架中展示，支持emoji；
8. Timeout：超时时长，被修饰的方法如果超时则会导致测试不通过；
9. Disabled：不执行的测试方法；

### 5版本已废弃的注解

以下的注解都是在5之前的版本使用的，现在已经被废弃：

| 被废弃的注解 | 新的继任者        |
| ------------ | ----------------- |
| Before       | BeforeEach        |
| After        | AfterEach         |
| BeforeClass  | BeforeAll         |
| AfterClass   | AfterAll          |
| Category     | Tag               |
| RunWith      | ExtendWith        |
| Rule         | ExtendWith        |
| ClassRule    | RegisterExtension |

### 版本和环境信息

整个系列的编码和执行在以下环境进行，供您参考：

1. 硬件配置：处理器i5-8400，内存32G，硬盘128G SSD + 500G HDD
2. 操作系统：Windows10家庭中文版
3. IDEA：2020.2.2 (Ultimate Edition)
4. JDK：1.8.0_181
5. SpringBoot：2.3.4.RELEASE
6. JUnit Jupiter：5.6.2
   接下来开始实战，咱们先建好SpringBoot项目；

### 关于lombok

为了简化代码，项目中使用了lombok，请您在IDEA中安装lombok插件；

### 源码下载

1. 如果您不想编码，可以在GitHub下载所有源码，地址和链接信息如下表所示(https://github.com/zq2599/blog_demos)：

| 名称               | 链接                                     | 备注                            |
| ------------------ | ---------------------------------------- | ------------------------------- |
| 项目主页           | https://github.com/zq2599/blog_demos     | 该项目在GitHub上的主页          |
| git仓库地址(https) | https://github.com/zq2599/blog_demos.git | 该项目源码的仓库地址，https协议 |
| git仓库地址(ssh)   | git@github.com:zq2599/blog_demos.git     | 该项目源码的仓库地址，ssh协议   |

1. 这个git项目中有多个文件夹，本章的应用在junitpractice文件夹下，如下图红框所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154643.jpeg)
2. junitpractice是父子结构的工程，本篇的代码在junit5experience子工程中，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154645.jpeg)

### 创建Maven父工程

1. 为了便于管理整个系列的源码，在此建立名为junitpractice的maven工程，后续所有实战的源码都作为junitpractice的子工程；
2. junitpractice的pom.xml如下，可见是以SpringBoot的2.3.4.RELEASE版本作为其父工程：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <modules>
        <module>simplebean</module>
        <!--
        <module>testenvironment</module>
        -->
    </modules>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>junitpractice</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>1.16.16</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

### 本篇的源码工程

接下来咱们准备一个简单的SpringBoot工程用于做单元测试，该工程有service和controller层，包含一些简单的接口和类；

1. 创建名为junit5experience的子工程，pom.xml如下，注意单元测试要依赖spring-boot-starter-test：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.bolingcavalry</groupId>
        <artifactId>junitpractice</artifactId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>junit5experience</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>junit5experience</name>
    <description>Demo project for simplebean in Spring Boot junit5</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

1. 写一些最简单的业务代码，首先是service层的接口HelloService.java：

```java
package com.bolingcavalry.junit5experience.service;

public interface HelloService {
    String hello(String name);
    int increase(int value);
    /**
     * 该方法会等待1秒后返回true，这是在模拟一个耗时的远程调用
     * @return
     */
    boolean remoteRequest();
}
```

1. 上述接口对应的实现类如下，hello和increase方法分别返回String型和int型，remoteRequest故意sleep了1秒钟，用来测试Timeout注解的效果：

```java
package com.bolingcavalry.junit5experience.service.impl;

import com.bolingcavalry.junit5experience.service.HelloService;
import org.springframework.stereotype.Service;

@Service()
public class HelloServiceImpl implements HelloService {
    @Override
    public String hello(String name) {
        return "Hello " + name;
    }

    @Override
    public int increase(int value) {
        return value + 1;
    }

    @Override
    public boolean remoteRequest() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException interruptedException) {
            interruptedException.printStackTrace();
        }

        return true;
    }
}
```

1. 添加一个简单的controller：

```java
package com.bolingcavalry.junit5experience.controller;

import com.bolingcavalry.junit5experience.service.HelloService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @Autowired
    private HelloService helloService;

    @RequestMapping(value = "/{name}", method = RequestMethod.GET)
    public String hello(@PathVariable String name){
        return helloService.hello(name);
    }
}
```

1. 启动类：

```java
package com.bolingcavalry.junit5experience;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Junit5ExperienceApplication {

    public static void main(String[] args) {
        SpringApplication.run(Junit5ExperienceApplication.class, args);
    }
}
```

- 以上就是一个典型的web工程，接下来一起为该工程编写单元测试用例；

### 编写测试代码

1. 在下图红框位置新增单元测试类：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154653.jpeg)
2. 测试类的内容如下，涵盖了刚才提到的常用注解，请注意每个方法的注释说明：

```java
package com.bolingcavalry.junit5experience.service.impl;

import com.bolingcavalry.junit5experience.service.HelloService;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import java.util.concurrent.TimeUnit;
import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest
@Slf4j
class HelloServiceImplTest {

    private static final String NAME = "Tom";

    @Autowired
    HelloService helloService;

    /**
     * 在所有测试方法执行前被执行
     */
    @BeforeAll
    static void beforeAll() {
        log.info("execute beforeAll");
    }

    /**
     * 在所有测试方法执行后被执行
     */
    @AfterAll
    static void afterAll() {
        log.info("execute afterAll");
    }

    /**
     * 每个测试方法执行前都会执行一次
     */
    @BeforeEach
    void beforeEach() {
        log.info("execute beforeEach");
    }

    /**
     * 每个测试方法执行后都会执行一次
     */
    @AfterEach
    void afterEach() {
        log.info("execute afterEach");
    }

    @Test
    @DisplayName("测试service层的hello方法")
    void hello() {
        log.info("execute hello");
        assertThat(helloService.hello(NAME)).isEqualTo("Hello " + NAME);
    }

    /**
     * DisplayName中带有emoji，在测试框架中能够展示
     */
    @Test
    @DisplayName("测试service层的increase方法\uD83D\uDE31")
    void increase() {
        log.info("execute increase");
        assertThat(helloService.increase(1)).isEqualByComparingTo(2);
    }

    /**
     * 不会被执行的测试方法
     */
    @Test
    @Disabled
    void neverExecute() {
        log.info("execute neverExecute");
    }

    /**
     * 调用一个耗时1秒的方法，用Timeout设置超时时间是500毫秒，
     * 因此该用例会测试失败
     */
    @Test
    @Timeout(unit = TimeUnit.MILLISECONDS, value = 500)
    @Disabled
    void remoteRequest() {
        assertThat(helloService.remoteRequest()).isEqualTo(true);
    }
}
```

1. 接下来执行测试用例试试，点击下图红框中的按钮：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154658.jpeg)
2. 如下图，在弹出的菜单中，点击红框位置：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154700.jpeg)
3. 执行结果如下，可见Displayname注解的值作为测试结果的方法名展示，超时的方法会被判定为测试不通过，Disable注解修饰的方法则被标记为跳过不执行：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154703.jpeg)
4. 在父工程junitpractice的pom.xml文件所在目录，执行mvn test命令，可以看到maven执行单元测试的效果：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154706.jpeg)

- 至此，咱们对SpringBoot环境下的JUnit5有了最基本的了解，接下来的章节会展开更多知识点和细节，对单元测试做更深入的学习。

# Assumptions类

### Assertions和Assumptions简介

Assumptions和Assertions容易混淆，因此这里通过对比它们来学习：

1. Assertions即断言类，里面提供了很多静态方法，例如assertTrue，如果assertTrue的入参为false，就会抛出AssertionFailedError异常，Junit对抛出此异常的方法判定为失败；
2. Assumptions即假设类，里面提供了很多静态方法，例如assumeTrue，如果assumeTrue的入参为false，就会抛出TestAbortedException异常，Junit对抛出此异常的方法判定为跳过；
3. 简单的说，Assertions的方法抛出异常意味着测试不通过，Assumptions的方法抛出异常意味着测试被跳过(为什么称为"跳过"？因为mvn test的执行结果被标记为Skipped)；

### 写一段代码对比效果

1. 用代码来验证的效果最好，如下所示，一共四个方法，assertSuccess不抛出AssertionFailedError异常，assertFail抛出AssertionFailedError异常，assumpSuccess不抛出TestAbortedException异常，assumpFail抛出TestAbortedException异常

```java
package com.bolingcavalry.assertassume.service.impl;

import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assumptions.assumeTrue;

@SpringBootTest
@Slf4j
public class AssertAssumpTest {

    /**
     * 最简单的成功用例
     */
    @Test
    void assertSuccess() {
        assertEquals(2, Math.addExact(1,1));
    }

    /**
     * 最简单的失败用例
     */
    @Test
    void assertFail() {
        assertEquals(3, Math.addExact(1,1));
    }

    /**
     * assumeTrue不抛出异常的用例
     */
    @Test
    void assumpSuccess() {
        // assumeTrue方法的入参如果为true，就不会抛出异常，后面的代码才会继续执行
        assumeTrue(true);
        // 如果打印出此日志，证明assumeTrue方法没有抛出异常
        log.info("assumpSuccess的assumeTrue执行完成");
        // 接下来是常规的单元测试逻辑
        assertEquals(2, Math.addExact(1,1));
    }

    /**
     * assumeTrue抛出异常的用例
     */
    @Test
    void assumpFail() {
        // assumeTrue方法的入参如果为false，就会抛出TestAbortedException异常，后面就不会执行了
        assumeTrue(false, "未通过assumeTrue");
        // 如果打印出此日志，证明assumpFail方法没有抛出异常
        log.info("assumpFail的assumeTrue执行完成");
        // 接下来是常规的单元测试逻辑，但因为前面抛出了异常，就不再执行了
        assertEquals(2, Math.addExact(1,1));
    }
}
```

1. 点击下图红框位置执行单元测试：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154720.jpeg)
2. 执行结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154723.jpeg)
3. 另外，在target目录，可以看到surefire插件生成的单元测试报告TEST-com.bolingcavalry.assertassume.service.impl.AssertAssumpTest.xml，如下图所示，testcase节点中出现了skipped节点：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154726.jpeg)

- 上述对比验证再次说明Assertions和Assumptions的区别：都用来对比预期值和实际值，当预期值和实际值不一致时，Assertions的测试结果是执行失败，Assumptions的测试结果是跳过(或者忽略)；

### Assumptions实战

弄清楚的Assertions和Assumptions的区别，接下来趁热打铁，学习Assumptions类中几个重要的静态方法：assumeTrue、assumingThat

1. 最简单的用法如下，可见只有assumeTrue不抛出异常，后面的log.info才会执行：

```java
    @Test
    @DisplayName("最普通的assume用法")
    void tryAssumeTrue() {
        assumeTrue("CI".equals(envType));

        log.info("CI环境才会打印的assumeTrue");
    }
```

1. assumeTrue可以接受Supplier类型作为第二个入参，如果assumeTrue失败就会将第二个参数的内容作为失败提示：

```java
@Test
@DisplayName("assume失败时带自定义错误信息")
void tryAssumeTrueWithMessage() {
    // 第二个入参是Supplier实现，返回的内容用作跳过用例时的提示信息
    assumeTrue("CI".equals(envType),
               () -> "环境不匹配而跳过，当前环境：" + envType);

    log.info("CI环境才会打印的tryAssumeTrueWithMessage");
}
```

效果如下图：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154729.jpeg)
\3. 还有个assumingThat方法，可以接受Executable类型作为第二个入参，如果第一个入参为true就会执行Executable的execute方法，注意assumingThat方法的特点：不抛出异常，因此其所在的方法不会被跳过，这是和assumeTrue相比最大的区别(assumeTrue一旦入参为false就会抛出异常，其所在方法就被标记为跳过)：

```shell
    @Test
    @DisplayName("assume成功时执行指定逻辑")
    void tryAssumingThat() {
        // 第二个入参是Executable实现，
        // 当第一个参数为true时，执行第二个参数的execute方法
        assumingThat("CI".equals(envType),
                () -> {
                    log.info("这一行内容只有在CI环境才会打印");
                });

        log.info("无论什么环境都会打印的tryAssumingThat");
    }
```

- 接下来咱们执行上述代码，看看效果；

### 执行Assumptions代码

1. 先做准备工作，本次实战的springboot工程名为assertassume，咱们在工程的resources目录下添加两个配置文件：application.properties和application-test.properties，位置如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154732.jpeg)
2. application-test.properties内容如下：

```properties
envType:CI
```

1. application.properties内容如下：

```properties
envType:PRODUCTION
```

1. 完整的单元测试类如下，通过注解ActiveProfiles，指定了使用application-test.properties的配置，因此envType的值为CI：

```java
package com.bolingcavalry.assertassume.service.impl;

import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.ActiveProfiles;
import static org.junit.jupiter.api.Assumptions.assumeTrue;
import static org.junit.jupiter.api.Assumptions.assumingThat;

@SpringBootTest
@Slf4j
@ActiveProfiles("test")
public class AssumptionsTest {

    @Value("${envType}")
    private String envType;

    @Test
    @DisplayName("最普通的assume用法")
    void tryAssumeTrue() {
        assumeTrue("CI".equals(envType));

        log.info("CI环境才会打印的assumeTrue");
    }

    @Test
    @DisplayName("assume失败时带自定义错误信息")
    void tryAssumeTrueWithMessage() {
        // 第二个入参是Supplier实现，返回的内容用作跳过用例时的提示信息
        assumeTrue("CI".equals(envType),
                () -> "环境不匹配而跳过，当前环境：" + envType);

        log.info("CI环境才会打印的tryAssumeTrueWithMessage");
    }

    @Test
    @DisplayName("assume成功时执行指定逻辑")
    void tryAssumingThat() {
        // 第二个入参是Executable实现，
        // 当第一个参数为true时，执行第二个参数的execute方法
        assumingThat("CI".equals(envType),
                () -> {
                    log.info("这一行内容只有在CI环境才会打印");
                });

        log.info("无论什么环境都会打印的tryAssumingThat");
    }
}
```

1. 执行结果如下图，可见assume通过，所有信息都被打印出来了：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154736.jpeg)
2. 接下来把代码中的ActiveProfiles注解那一行注释掉，如下图红框：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154738.jpeg)
3. 执行结果如下，可见tryAssumingThat方法被标记为成功，不过从日志可见assumingThat的第二个入参executable没有被执行：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154741.jpeg)

- 至此，Assumptions类的常用方法体验完成，接下来的章节会继续学习其他常用类；

# Assertions类

### Assertions源码分析

1. 下图是一段最简单最常见的单元测试代码，也就是Assertions.assertEquals方法，及其执行效果：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154752.jpeg)
2. 将Assertions.assertEquals方法逐层展开，如下图所示，可见入参expected和actual的值如果不相等，就会在AssertionUtils.fail方法中抛出AssertionFailedError异常：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154755.png)
3. 用类图工具查看Assertions类的方法，如下图，大部分是与assertEquals方法类似的判断，例如对象是否为空，数组是否相等，判断失败都会抛出AssertionFailedError异常：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154759.jpeg)
4. 判断两个数组是否相等的逻辑与判断两个对象略有不同，可以重点看看，方法源码如下：

```java
	public static void assertArrayEquals(Object[] expected, Object[] actual) {
		AssertArrayEquals.assertArrayEquals(expected, actual);
	}
```

1. 将上述代码逐层展开，在AssertArrayEquals.java中见到了完整的数组比较逻辑，如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154802.jpeg)

- 接下来，咱们编写一些单元测试代码，把Assertions类常用的方法都熟悉一遍；

### 编码实战

1. 打开junitpractice工程的子工程assertassume，新建测试类AssertionsTest.java：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154805.jpeg)
2. 最简单的判断，两个入参相等就不抛异常（AssertionFailedError）：

```java
    @Test
    @DisplayName("最普通的判断")
    void standardTest() {
       assertEquals(2, Math.addExact(1, 1));
    }
```

1. 还有另一个assertEquals方法，能接受Supplier类型的入参，当判断不通过时才会调用Supplier.get方法获取字符串作为失败提示消息（如果测试通过则Supplier.get方法不会被执行）：

```java
    @Test
    @DisplayName("带失败提示的判断(拼接消息字符串的代码只有判断失败时才执行)")
    void assertWithLazilyRetrievedMessage() {
        int expected = 2;
        int actual = 1;

        assertEquals(expected,
                actual,
                // 这个lambda表达式，只有在expected和actual不相等时才执行
                ()->String.format("期望值[%d]，实际值[%d]", expected, actual));
    }
```

1. assertAll方法可以将多个判断逻辑放在一起处理，只要有一个报错就会导致整体测试不通过，并且执行结果中会给出具体的失败详情：

```java
    @Test
    @DisplayName("批量判断(必须全部通过，否则就算失败)")
    void groupedAssertions() {
        // 将多个判断放在一起执行，只有全部通过才算通过，如果有未通过的，会有对应的提示
        assertAll("单个测试方法中多个判断",
                () -> assertEquals(1, 1),
                () -> assertEquals(2, 1),
                () -> assertEquals(3, 1)
        );
    }
```

上述代码执行结果如下：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154809.jpeg)

### 异常断言

1. Assertions.assertThrows方法，用来测试Executable实例执行execute方法时是否抛出指定类型的异常；
2. 如果execute方法执行时不抛出异常，或者抛出的异常与期望类型不一致，都会导致测试失败；
3. 写段代码验证一下，如下，1除以0会抛出ArithmeticException异常，符合assertThrows指定的异常类型，因此测试可以通过：

```java
    @Test
    @DisplayName("判断抛出的异常是否是指定类型")
    void exceptionTesting() {

        // assertThrows的第二个参数是Executable，
        // 其execute方法执行时，如果抛出了异常，并且异常的类型是assertThrows的第一个参数(这里是ArithmeticException.class)，
        // 那么测试就通过了，返回值是异常的实例
        Exception exception = assertThrows(ArithmeticException.class, () -> Math.floorDiv(1,0));

        log.info("assertThrows通过后，返回的异常实例：{}", exception.getMessage());
    }
```

- 以上是Assertions的常规用法，接下来要重点关注的就是和超时相关的测试方法；

### 超时相关的测试

1. 超时测试的主要目标是验证指定代码能否在规定时间内执行完，最常用的assertTimeout方法内部实现如下图，可见被测试的代码通过ThrowingSupplier实例传入，被执行后再检查耗时是否超过规定时间，超过就调用fail方法抛AssertionFailedError异常：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154823.jpeg)
2. assertTimeout的用法如下，期望时间是1秒，实际上Executable实例的execute用了两秒才完成，因此测试失败：

```java
    @Test
    @DisplayName("在指定时间内完成测试")
    void timeoutExceeded() {
        // 指定时间是1秒，实际执行用了2秒
        assertTimeout(ofSeconds(1), () -> {
            try{
              Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
    }
```

执行结果如下图：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154827.jpeg)
\3. 上面的演示中，assertTimeout的第二个入参类型是Executable，此外还有另一个assertTimeout方法，其第二个入参是ThrowingSupplier类型，该类型入参的get方法必须要有返回值，假设是XXX，而assertTimeout就拿这个XXX作为它自己的返回值，使用方法如下：

```java
    @Test
    @DisplayName("在指定时间内完成测试")
    void timeoutNotExceededWithResult() {

        // 准备ThrowingSupplier类型的实例，
        // 里面的get方法sleep了1秒钟，然后返回一个字符串
        ThrowingSupplier<String> supplier = () -> {

            try{
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            return "我是ThrowingSupplier的get方法的返回值";
        };

        // 指定时间是2秒，实际上ThrowingSupplier的get方法只用了1秒
        String actualResult = assertTimeout(ofSeconds(2), supplier);

        log.info("assertTimeout的返回值：{}", actualResult);
    }
```

上述代码执行结果如下，测试通过并且ThrowingSupplier实例的get方法的返回值也被打印出来：
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154831.jpeg)
\4. 刚才咱们看过了assertTimeout的内部实现代码，是将入参Executable的execute方法执行完成后，再检查execute方法的耗时是否超过预期，这种方法的弊端是必须等待execute方法执行完成才知道是否超时，assertTimeoutPreemptively方法也是用来检测代码执行是否超时的，但是避免了assertTimeout的必须等待execute执行完成的弊端，避免的方法是用一个新的线程来执行execute方法，下面是assertTimeoutPreemptively的源码：

```java
public static void assertTimeoutPreemptively(Duration timeout, Executable executable) {
	AssertTimeout.assertTimeoutPreemptively(timeout, executable);
}
```

1. assertTimeoutPreemptively方法的Executable入参，其execute方法会在一个新的线程执行，假设是XXX线程，当等待时间超过入参timeout的值时，XXX线程就会被中断，并且测试结果是失败，下面是assertTimeoutPreemptively的用法演示，设置的超时时间是2秒，而Executable实例的execute却sleep了10秒：

```java
    @Test
    void timeoutExceededWithPreemptiveTermination() {
        log.info("开始timeoutExceededWithPreemptiveTermination");
        assertTimeoutPreemptively(ofSeconds(2), () -> {
            log.info("开始sleep");
            try{
                Thread.sleep(10000);
                log.info("sleep了10秒");
            } catch (InterruptedException e) {
                log.error("线程sleep被中断了", e);
            }
        });
    }
```

1. 来看看执行结果，如下图，通过日志可见，Executable的execute方法是在新的线程执行的，并且被中断了，提前完成单元测试，测试结果是不通过：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154834.jpeg)

### 第三方断言库

1. 除了junit的Assertions类，还可以选择第三方库提供的断言能力，比较典型的有AssertJ, Hamcrest, Truth这三种，它们都有各自的特色和适用场景，例如Hamcrest的特点是匹配器(matchers )，而Truth来自谷歌的Guava团队，编写的代码是链式调用风格，简单易读，断言类型相对更少却不失功能；
2. springboot默认依赖了hamcrest库，依赖关系如下图：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154837.jpeg)
3. 一个简单的基于hamcrest的匹配器的单元测试代码如下，由于预期和实际的值不相等，因此会匹配失败：

```java
package com.bolingcavalry.assertassume.service.impl;

import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import static org.hamcrest.CoreMatchers.equalTo;
import static org.hamcrest.CoreMatchers.is;
import static org.hamcrest.MatcherAssert.assertThat;

@SpringBootTest
@Slf4j
public class HamcrestTest {

    @Test
    @DisplayName("体验hamcrest")
    void assertWithHamcrestMatcher() {
        assertThat(Math.addExact(1, 2), is(equalTo(5)));
    }
}
```

1. 执行结果如下：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154841.jpeg)

- 以上就是JUnit5常用的断言功能，希望本篇能助您夯实基础，为后续写出更合适的用例做好准备；

# 按条件执行

### 自定义测试方法的执行顺序

今天要写的测试方法很多，为了管理好这些方法，在学习按条件执行之前先来看看如何控制测试方法的执行顺序：

1. 给测试类添加注解TestMethodOrder，注解的value是OrderAnnotation.class
2. 给每个测试方法添加Order注解，value值是数字，越小的value越优先执行
3. 使用方法如下图所示：
   ![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210204154857.jpeg)

- 接下来的实战中，咱们就用上述方法控制测试方法的执行顺序；

### 按操作系统设置条件

1. 注解EnabledOnOs指定多个操作系统，只有当前操作系统是其中的一个，测试方法才会执行；
2. 注解DisabledOnOs指定多个操作系统，只要当前操作系统是其中的一个，测试方法就不会执行；
3. 测试代码如下：

```java
    @Test
    @Order(1)
    @EnabledOnOs(OS.WINDOWS)
    @DisplayName("操作系统：只有windows才会执行")
    void onlyWindowsTest() {
        assertEquals(2, Math.addExact(1, 1));
    }

    @Test
    @Order(2)
    @EnabledOnOs({OS.WINDOWS, OS.LINUX})
    @DisplayName("操作系统：windows和linux都会执行")
    void windowsORLinuxTest() {
        assertEquals(2, Math.addExact(1, 1));
    }

    @Test
    @Order(3)
    @DisabledOnOs({OS.MAC})
    @DisplayName("操作系统：只有MAC才不会执行")
    void withoutMacTest() {
        assertEquals(2, Math.addExact(1, 1));
    }
1234567891011121314151617181920212223
```

1. 我这里是windows操作系统，上述三个方法执行结果如下：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201003134012558.jpg#pic_center)

### 按JAVA环境设置条件

1. 注解EnabledOnJre指定多个JRE版本，只有当前JRE是其中的一个，测试方法才会执行；
2. 注解DisabledOnJre指定多个JRE版本，只要当前JRE是其中的一个，测试方法就不会执行；
3. 注解EnabledForJreRange指定JRE版本的范围，只有当前JRE在此范围内，测试方法才会执行；
4. 注解DisabledForJreRange指定JRE版本的范围，只要当前JRE在此范围内，测试方法就不会执行；
5. 测试代码如下：

```java
    @Test
    @Order(4)
    @EnabledOnJre({JRE.JAVA_9, JRE.JAVA_11})
    @DisplayName("Java环境：只有JAVA9和11版本才会执行")
    void onlyJava9And11Test() {
        assertEquals(2, Math.addExact(1, 1));
    }

    @Test
    @Order(5)
    @DisabledOnJre({JRE.JAVA_9})
    @DisplayName("Java环境：JAVA9不执行")
    void withoutJava9Test() {
        assertEquals(2, Math.addExact(1, 1));
    }

    @Test
    @Order(6)
    @EnabledForJreRange(min=JRE.JAVA_8, max=JRE.JAVA_11)
    @DisplayName("Java环境：从JAVA8到1之间的版本都会执行")
    void fromJava8To11Test() {
        assertEquals(2, Math.addExact(1, 1));
    }
1234567891011121314151617181920212223
```

1. 我这里是JDK8，执行结果如下：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201003140238340.jpg#pic_center)

### 按系统属性设置条件

1. 注解EnabledIfSystemProperty指定系统属性的key和期望值(模糊匹配)，只有当前系统有此属性并且值也匹配，测试方法才会执行；
2. 注解DisabledIfSystemProperty指定系统属性的key和期望值(模糊匹配)，只要当前系统有此属性并且值也匹配，测试方法就不会执行；
3. 测试代码如下：

```java
    @Test
    @Order(7)
    @EnabledIfSystemProperty(named = "os.arch", matches = ".*64.*")
    @DisplayName("系统属性：64位操作系统才会执行")
    void only64BitArch() {
        assertEquals(2, Math.addExact(1, 1));
    }

    @Test
    @Order(8)
    @DisabledIfSystemProperty(named = "java.vm.name", matches = ".*HotSpot.*")
    @DisplayName("系统属性：HotSpot不会执行")
    void withOutHotSpotTest() {
        assertEquals(2, Math.addExact(1, 1));
    }

12345678910111213141516
```

1. 上述测试方法执行结果如下：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201003140750152.jpg#pic_center)

### 按环境变量设置条件

1. 注解EnabledIfEnvironmentVariable指定环境变量的key和期望值(模糊匹配)，只有当前系统有此环境变量并且值也匹配，测试方法才会执行；
2. 注解DisabledIfEnvironmentVariable指定环境变量的key和期望值(模糊匹配)，只要当前系统有此环境变量并且值也匹配，测试方法就不会执行；
3. 测试代码如下：

```java
    @Test
    @Order(9)
    @EnabledIfEnvironmentVariable(named = "JAVA_HOME", matches = ".*")
    @DisplayName("环境变量：JAVA_HOME才会执行")
    void onlyJavaHomeExistsInEnvTest() {
        assertEquals(2, Math.addExact(1, 1));
    }

    @Test
    @Order(10)
    @DisabledIfEnvironmentVariable(named = "GOPATH", matches = ".*")
    @DisplayName("环境变量：有GOPATH就不执行")
    void withoutGoPathTest() {
        assertEquals(2, Math.addExact(1, 1));
    }

12345678910111213141516
```

1. 上述测试方法执行结果如下：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201003141152866.jpg#pic_center)

### 自定义条件(从junit5.7版本开始)

1. 前面的条件注解很丰富，但终究是固定、有限的，无法满足所有场景，它们不够用时，咱们还可以自定义前提条件，即EnabledIf和DisabledIf注解；
2. 有两个关键点要格外注意，首先是EnabledIf和DisabledIf的package，注意是org.junit.jupiter.api.condition，不要用这个：org.springframework.test.context.junit.jupiter.EnabledIf，一旦用错，执行测试时会抛出异常；
3. 第二个要注意的是EnabledIf和DisabledIf对应的junit版本，它们是从5.7版本版本才开始的，而本文用的SpringBoot版本是2.3.4.RELEASE，间接依赖的junit版本是5.6.2，因此，必须在pom.xml中做下图红框中的修改，将间接依赖去掉，并主动依赖5.7.0，才能将junit从5.6.2升级到5.7，这样才能用上EnabledIf和DisabledIf：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201003142641727.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)
4. EnabledIf的用法很简单，value是个存在的方法的名字，该方法必须返回boolean类型，demo如下，customCondition是个很简单的方法，被用来做是否执行单元测试的判断条件：

```java
    boolean customCondition() {
        return true;
    }

    @Test
    @Order(11)
    @EnabledIf("customCondition")
    @DisplayName("自定义：customCondition返回true就执行")
    void onlyCustomConditionTest() {
        assertEquals(2, Math.addExact(1, 1));
    }

    @Test
    @Order(12)
    @DisabledIf("customCondition")
    @DisplayName("自定义：customCondition返回true就不执行")
    void withoutCustomConditionTest() {
        assertEquals(2, Math.addExact(1, 1));
    }
12345678910111213141516171819
```

1. 上述测试方法执行结果如下：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201003143451127.jpg#pic_center)
2. 前面的代码中，EnabledIf和DisabledIf注解被用来修饰方法，其实它们还可以修饰类，用于控制整个类是否执行单元测试，不过修饰类的时候，对应的自定义方法必须是static类型；
3. 前面的代码中，customCondition方法和使用它的EnabledIf注解在同一个类中，其实它们也可以在不同的类中，不过此时EnabledIf注解的value要给出：包名、类名、方法名，如下所示，注意类名和方法名之间的连接符是#：

```java
    @Test
    @Order(12)
    @DisabledIf("com.example.Conditions#customCondition")
    @DisplayName("自定义：customCondition返回true就不执行")
    void withoutCustomConditionTest() {
        assertEquals(2, Math.addExact(1, 1));
    }
1234567
```

- 以上就是常用的按条件执行单元测试的各种实例了，希望本文能给您提供参考，助您在各种场景更加精确的控制用例的执行逻辑；

# 标签(Tag)和自定义注解

### 设置标签

1. 在父工程junitpractice里新建名为tag的子工程，今天的单元测试代码都写在这个tag工程中；
2. 一共写两个测试类，第一个FirstTest.java如下，可见类上有Tag注解，值为first，另外每个方法上都有Tag注解，其中first1Test方法有两个Tag注解：

```java
package com.bolingcavalry.tag.service.impl;

import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Tag;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import static org.junit.jupiter.api.Assertions.assertEquals;

@SpringBootTest
@Slf4j
@Tag("first")
public class FirstTest {

    @Test
    @Tag("easy")
    @Tag("important")
    @DisplayName("first-1")
    void first1Test() {
        log.info("first1Test");
        assertEquals(2, Math.addExact(1, 1));
    }

    @Test
    @Tag("easy")
    @DisplayName("first-2")
    void first2Test() {
        log.info("first2Test");
        assertEquals(2, Math.addExact(1, 1));
    }

    @Test
    @Tag("hard")
    @DisplayName("first-3")
    void first3Test() {
        log.info("first3Test");
        assertEquals(2, Math.addExact(1, 1));
    }
}
123456789101112131415161718192021222324252627282930313233343536373839
```

1. 第二个测试类SecondTest.java，也是类和方法都有Tag注解：

```java
package com.bolingcavalry.tag.service.impl;

import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Tag;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import static org.junit.jupiter.api.Assertions.assertEquals;

@SpringBootTest
@Slf4j
@Tag("second")
public class SecondTest {

    @Test
    @Tag("easy")
    @DisplayName("second-1")
    void second1Test() {
        log.info("second1Test");
        assertEquals(2, Math.addExact(1, 1));
    }

    @Test
    @Tag("easy")
    @DisplayName("second-2")
    void second2Test() {
        log.info("second2Test");
        assertEquals(2, Math.addExact(1, 1));
    }

    @Test
    @Tag("hard")
    @Tag("important")
    @DisplayName("second-3")
    void second3Test() {
        log.info("second3Test");
        assertEquals(2, Math.addExact(1, 1));
    }
}
123456789101112131415161718192021222324252627282930313233343536373839
```

- 以上就是打好了标签的测试类和测试方法了，接下来看看如何通过这些标签对测试方法进行过滤，执行单元测试有三种常用方式，咱们挨个尝试每种方式如何用标签过滤；

### 在IDEA中做标签过滤

1. 如下图所示，点击红框中的Edit Configurations…：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201003233816956.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)
2. 如下图红框，在弹出的窗口上新增一个JUnit配置：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201003234321851.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)
3. 接下来的操作如下图所示，Test kind选择Tags，就会按照标签过滤测试方法，Tag expression里面填写过滤规则，后面会详细讲解这个规则，这里先填个已存在的标签important：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004000010578.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)
4. 创建好JUnit配置后，执行下图红框中的操作即可执行单元测试：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004000737685.jpg#picpic_leftcenter)
5. 执行结果如下，所有打了important标签的测试方法被执行：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/2020100400101425.jpg#pic_left)

### 用maven命令时做标签过滤

1. 前面试过IDEA上按标签过滤测试方法，其实用maven命令执行单元测试的时候也能按标签来过滤，接下来试试；
2. 在父工程junitpractice的pom.xml所在目录下，执行以下命令，即可开始单元测试，并且只执行带有标签的方法：

```shell
mvn clean test -Dgroups="important"
1
```

1. 执行完毕后结果如下：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004003549330.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)
2. 翻看日志，可见只有打了important标签的测试方法被执行了，如下图红框所示：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004003730363.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)
3. 再看看其他子工程的执行情况，用前一篇文章里的conditional为例，可见没有任何测试方法被执行，如下图红框所示：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004003944854.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)
4. 再去看看surefire插件给出的测试报告，报告文件在junitpractice\tag\target\surefire-reports目录下，下图红框中的文件就是测试报告：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004004147332.jpg#pic_center)
5. 打开上图红框中的一个文件，如下图红框，可见只有打了important标签的测试方法被执行了：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/2020100400440483.jpg#pic_left)

- 以上就是maven命令执行单元测试时使用标签过滤的方法，接下来试试在使用maven-surefire-plugin插件时如何通过做标签过滤

### 用surefire插件时做标签过滤

1. surefire是个测试引擎(TestEngine)，以maven插件的方式来使用，打开tag子工程的pom.xml文件，将build节点配置成以下形式，可见groups就是标签过滤节点，另外excludedGroups节点制定的hard标签的测试方法不会执行：

```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.22.2</version>
                <configuration>
                    <!--要执行的标签-->
                    <groups>important</groups>
                    <!--不要执行的标签-->
                    <excludedGroups>hard</excludedGroups>
                </configuration>
            </plugin>
        </plugins>
    </build>
123456789101112131415161718
```

1. 在tag子工程的pom.xml所在目录，执行命令mvn clean test即可开始单元测试，结果如下，可见打了important标签的first1Test被执行，而second3Test方法尽管有important标签，但是由于其hard标签已经被设置为不执行，因此second3Test没有被执行：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004093024632.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)

### 标签表达式

1. 前面咱们用三种方法执行了单元测试，每次都是用important标签过滤，其实除了指定标签，JUnit还支持更复杂的标签过滤，即标签表达式
2. 所谓标签表达式，就是用"非"、“与”、"或"这三种操作符将更多的标签连接起来，实现更复杂的过滤逻辑；
3. 上述三种操作符的定义和用法如下表：

| 操作符 | 作用 | 举例              | 举例说明                                                     |
| ------ | ---- | ----------------- | ------------------------------------------------------------ |
| &      | 与   | important & easy  | 既有important，又有easy标签， 在本文是first1Test             |
| !      | 非   | important & !easy | 有important，同时又没有easy标签， 在本文是second3Test        |
| \|     | 或   | important \| hard | 有important标签的，再加上有hard标签的， 在本文是first1Test、first3Test、second3Test |

1. 试试标签表达式的效果，如下图红框，修改前面创建好的IDEA配置，从之前的important改为important | hard：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004102148202.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)
2. 再次执行这个配置，结果如下图红框所示，只有这三个方法被执行：first1Test、first3Test、second3Test，可见标签表达式生效了：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004102439736.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)
3. 在maven命令和surefire插件中使用标签表达式的操作就不在文中执行了，请您自行验证；

### 自定义注解

1. JUnit支持自定义注解，先回顾之前的代码，看咱们是如何给方法打标签的，以first3Test方法为例：

```java
    @Test
    @Tag("hard")
    @DisplayName("first-3")
    void first3Test() {
        log.info("first3Test");
        assertEquals(2, Math.addExact(1, 1));
    }
1234567
```

1. 接下来咱们创建一个注解，将@Tag(“hard”)替换掉，新注解的源码如下，可见仅是一个普通的注解定义：

```java
package com.bolingcavalry.tag.service.impl;

import org.junit.jupiter.api.Tag;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Tag("hard")
public @interface Hard {
}
12345678910111213
```

1. 修改first3Test方法的注解，去掉@Tag(“hard”)，改为@Hard：

```java
    @Test
    @Hard
    @DisplayName("first-3")
    void first3Test() {
        log.info("first3Test");
        assertEquals(2, Math.addExact(1, 1));
    }
1234567
```

1. 执行前面创建的tag-important配置，可见hard标签的过滤依旧有效：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004105042615.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)

### 更加简化的自定义注解

1. 上述Hard注解取代了@Tag(“hard”)，其实还可以更进一步对已有注解做简化，下面是个新的注解：HardTest.java，和Hard.java相比，多了个@Test，作用是集成了Test注解的能力

```java
package com.bolingcavalry.tag.service.impl;

import org.junit.jupiter.api.Tag;
import org.junit.jupiter.api.Test;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Tag("hard")
@Test
public @interface HardTest {
}
123456789101112131415
```

1. 于是，first3Test方法的注解可以改成下面的效果，可见Test和Tag注解都去掉了：

```java
    @HardTest
    @DisplayName("first-3")
    void first3Test() {
        log.info("first3Test");
        assertEquals(2, Math.addExact(1, 1));
    }
123456
```

1. 执行前面创建的tag-important配置，可见hard标签的过滤依旧有效：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004110724644.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)

### 标签命名规范

最后一起来看看给标签取名时有哪些要注意的地方：

1. 标签名左右两侧的空格是无效的，执行测试的时候会做trim处理，例如下面这个标签会被当作hard来过滤：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201004112510320.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)
2. 标签名不能有这六个符号， ( ) & | !

- 至此，JUnit5的标签过滤和自定义注解功能都学习完成了，有了这些能力，咱们可以更加灵活和随心所欲的应付不同的场景和需求；

# 参数化测试(Parameterized Tests)基础

### 极速体验

1. 现在，咱们以最少的步骤体验最简单的参数化测试；
2. 在父工程junitpractice里新建名为parameterized的子工程，pom.xml内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.bolingcavalry</groupId>
        <artifactId>junitpractice</artifactId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <groupId>com.bolingcavalry</groupId>
    <artifactId>parameterized</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>parameterized</name>
    <description>Demo project for parameterized expirence in Spring Boot junit</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.junit</groupId>
                <artifactId>junit-bom</artifactId>
                <version>5.7.0</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>

    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.jupiter</groupId>
                    <artifactId>junit-jupiter</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768697071
```

1. 新建测试类HelloTest.java，在这个位置：junitpractice\parameterized\src\test\java\com\bolingcavalry\parameterized\service\impl，内容如下：

```java
package com.bolingcavalry.parameterized.service.impl;

import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.MethodOrderer;
import org.junit.jupiter.api.Order;
import org.junit.jupiter.api.TestMethodOrder;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ValueSource;
import org.springframework.boot.test.context.SpringBootTest;
import static org.junit.jupiter.api.Assertions.assertTrue;

@SpringBootTest
@Slf4j
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
public class HelloTest {

    @Order(1)
    @DisplayName("多个字符串型入参")
    @ParameterizedTest
    @ValueSource(strings = { "a", "b", "c" })
    void stringsTest(String candidate) {
        log.info("stringsTest [{}]", candidate);
        assertTrue(null!=candidate);
    }
}    
1234567891011121314151617181920212223242526
```

1. 执行该测试类，结果如下图：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201005190634735.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)
2. 从上图可见执行参数化测试需要两步：首先用@ParameterizedTest取代@Test，表名此方法要执行参数化测试，然后用@ValueSource指定每次测试时的参数来自字符串类型的数组：{ “a”, “b”, “c” }，每个元素执行一次；
3. 至此，咱们已体验过最简单的参数化测试，可见就是想办法使一个测试方法多次执行，每次都用不同的参数，接下来有关参数化测试的更多配置和规则将配合实战编码逐个展开，一起来体验吧；

### 版本要求

- 先看看SpringBoot-2.3.4.RELEASE间接依赖的junit-jupiter-5.6.2版本中，ParameterizedTest的源码，如下图红框所示，此时的ParameterizedTest还只是体验版：
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201006084545848.jpg#pic_center)
- 再看看junit-jupiter-5.7.0版本的ParameterizedTest源码，此时已经是稳定版了：
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201006085132156.jpg#pic_center)
- 综上所述，如果要使用参数化测试，最好是将junit-jupiter升级到5.7.0或更高版本，如果您的应用使用了SpringBoot框架，junit-jupiter是被spring-boot-starter-test间接依赖进来的，需要排除这个间接依赖，再手动依赖进来才能确保使用指定版本，在pom.xml中执行如下三步操作：

1. dependencyManagement节点添加junit-bom，并指定版本号：

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.junit</groupId>
      <artifactId>junit-bom</artifactId>
      <version>5.7.0</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
1234567891011
```

1. 排除spring-boot-starter-test和junit-jupiter的间接依赖关系：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
  <scope>test</scope>
  <exclusions>
    <exclusion>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter</artifactId>
    </exclusion>
  </exclusions>
</dependency>
1234567891011
```

1. 添加junit-jupiter依赖，此时会使用dependencyManagement中指定的版本号：

```xml
<dependency>
  <groupId>org.junit.jupiter</groupId>
  <artifactId>junit-jupiter</artifactId>
  <scope>test</scope>
</dependency>
12345
```

1. 如下图，刷新可见已经用上了5.7.0版本：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201006091514944.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)

- 版本问题解决了，接下来正式开始学习Parameterized Tests，先要了解的是有哪些数据源；

### ValueSource数据源

1. ValueSource是最简单常用的数据源，支持以下类型的数组：

```java
    short

    byte

    int

    long

    float

    double

    char

    boolean

    java.lang.String
    
    java.lang.Class
12345678910111213141516171819
```

1. 下面是整形数组的演示：

```java
    @Order(2)
    @DisplayName("多个int型入参")
    @ParameterizedTest
    @ValueSource(ints = { 1,2,3 })
    void intsTest(int candidate) {
        log.info("ints [{}]", candidate);
        assertTrue(candidate<3);
    }
12345678
```

1. 从上述代码可见，入参等于3的时候assertTrue无法通过，测试方法会失败，来看看实际执行效果，如下图：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201006092907432.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)

### null、空字符串数据源

1. 在用字符串作为入参时，通常要考虑入参为null的情况，此时ValueSource一般会这样写：

```java
@ValueSource(strings = { null, "a", "b", "c" })
1
```

1. 此时可以使用@NullSource注解来取代上面的null元素，下面这种写法和上面的效果一模一样：

```java
    @NullSource
    @ValueSource(strings = { "a", "b", "c" })
12
```

1. 执行结果如下图红框，可见null作为入参被执行了一次：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201006094916970.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)
2. 与@NullSource代表null入参类似，@EmptySource代表空字符串入参，用法和执行结果如下图所示：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201006100531583.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)
3. 如果想同时用null和空字符串做测试方法的入参，可以使用@NullAndEmptySource，用法和执行结果如下图所示：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201006101318682.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)

### 枚举数据源(EnumSource)

1. EnumSource可以让一个枚举类中的全部或者部分值作为测试方法的入参；
2. 创建枚举类Types.java，用于接下来的实战，如下，很简单只有三个值：

```java
public enum Types {
    SMALL,
    BIG,
    UNKNOWN
}
12345
```

1. 先尝试用Types的每个值作为入参执行测试，可见只要添加@EnumSource即可，JUnit根据测试方法的入参类型知道要使用哪个枚举：

```java
    @Order(6)
    @DisplayName("多个枚举型入参")
    @ParameterizedTest
    @EnumSource
    void enumSourceTest(Types type) {
        log.info("enumSourceTest [{}]", type);
    }
1234567
```

1. 执行结果如下图所示：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/2020100610314689.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)
2. 如果不想执行枚举的所有值，而只要其中一部分，可以在name属性中指定：

```java
@EnumSource(names={"SMALL", "UNKNOWN"})
1
```

1. 执行结果如下图所示：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201006103606409.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)
2. 也可以指定哪些值不被执行，此时要添加mode属性并设置为EXCLUDE（mode属性如果不写，默认值是INCLUDE，前面的例子中就是默认值）：

```java
@EnumSource(mode= EnumSource.Mode.EXCLUDE, names={"SMALL", "UNKNOWN"})
1
```

1. 执行结果如下，可见SMALL和UNKNOWN都没有执行：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201006104556989.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)

### 方法数据源(MethodSource)

1. @MethodSource可以指定一个方法名称，该方法返回的元素集合作为测试方法的入参；
2. 先来定义一个方法，该方法一般是static类型(否则要用@TestInstance修饰)，并且返回值是Stream类型：

```java
    static Stream<String> stringProvider() {
        return Stream.of("apple1", "banana1");
    }
123
```

1. 然后，测试方法用@MethodSource，并指定方法名stringProvider：

```java
    @Order(9)
    @DisplayName("静态方法返回集合，用此集合中每个元素作为入参")
    @ParameterizedTest
    @MethodSource("stringProvider")
    void methodSourceTest(String candidate) {
        log.info("methodSourceTest [{}]", candidate);
    }
1234567
```

1. 上面的stringProvider方法和测试方法methodSourceTest在同一个类中，如果它们不在同一个类中，就要指定静态方法的整个package路径、类名、方法名，如下所示，类名和方法名之间用#连接：

```java
@Order(10)
    @DisplayName("静态方法返回集合，该静态方法在另一个类中")
    @ParameterizedTest
    @MethodSource("com.bolingcavalry.parameterized.service.impl.Utils#getStringStream")
    void methodSourceFromOtherClassTest(String candidate) {
        log.info("methodSourceFromOtherClassTest [{}]", candidate);
    }
1234567
```

1. 如果不在@MethodSource中指定方法名，JUnit会寻找和测试方法同名的静态方法，举例如下，静态方法methodSourceWithoutMethodNameTest会被作为测试方法的数据来源：

```java
    static Stream<String> methodSourceWithoutMethodNameTest() {
        return Stream.of("apple3", "banana3");
    }

    @Order(11)
    @DisplayName("静态方法返回集合，不指定静态方法名，自动匹配")
    @ParameterizedTest
    @MethodSource
    void methodSourceWithoutMethodNameTest(String candidate) {
        log.info("methodSourceWithoutMethodNameTest [{}]", candidate);
    }
1234567891011
```

1. 执行结果如下：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201006111806881.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)

### Csv格式数据源(CsvSource)

1. 前面的测试方法入参都只有一个，在面对多个入参的测试方法时，@CsvSource就派上用场了，演示代码如下所示，可见数据是普通的CSV格式，每条记录有两个字段，对应测试方法的两个入参：

```java
    @Order(12)
    @DisplayName("CSV格式多条记录入参")
    @ParameterizedTest
    @CsvSource({
            "apple1, 11",
            "banana1, 12",
            "'lemon1, lime1', 0x0A"
    })
    void csvSourceTest(String fruit, int rank) {
        log.info("csvSourceTest, fruit [{}], rank [{}]", fruit, rank);
    }
1234567891011
```

1. 执行结果如下，通过日志可以确定，每条记录的两个字段能匹配到测试方法的两个入参中：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201006112801631.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)
2. 另外@CsvSource还提供了一个属性nullValues，作用是将指定的字符串识别为null，下面这个设置就是把CSV数据中所有的NIL识别为null，再传给测试方法：

```java
    @Order(13)
    @DisplayName("CSV格式多条记录入参(识别null)")
    @ParameterizedTest
    @CsvSource(value = {
            "apple2, 21",
            "banana2, 22",
            "'lemon2, lime2', 0x0A",
            "NIL, 3" },
            nullValues = "NIL"
    )
    void csvSourceWillNullTokenTest(String fruit, int rank) {
        log.info("csvSourceWillNullTokenTest, fruit [{}], rank [{}]", fruit, rank);
    }
12345678910111213
```

1. 执行结果如下，可见字符串NIL到测试方法后已变成null：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201006114048236.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)

### Csv文件数据源

1. @CsvSource解决了测试方法入参有多个字段的问题，但是把作为入参的测试数据写在源文件中似乎不合适，尤其是数据量很大的情况下，这种场景适合用@CsvFileSource，该注解用于指定csv文件作为数据源，注意numLinesToSkip属性指定跳过的行数，可以用来跳过表头：

```java
    @Order(14)
    @DisplayName("CSV文件多条记录入参")
    @ParameterizedTest
    @CsvFileSource(files = "src/test/resources/two-column.csv", numLinesToSkip = 1)
    void csvFileTest(String country, int reference) {
        log.info("csvSourceTest, country [{}], reference [{}]", country, reference);
    }
1234567
```

1. 在src/test/resources/创建文件two-column.csv，内容如下：

```csv
Country, reference
Sweden, 1
Poland, 2
"United States of America", 3
1234
```

1. 上述代码执行结果如下，代码中没有测试数据，显得更加简洁一些：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201006115905306.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_center)

### 期待《进阶》篇

- 至此，咱们队JUnit5的参数化测试(Parameterized)有了初步的了解，可以通过各种数据源注解给测试方法制造更多的参数，但仅掌握这些还是不够的，依然有一些问题待解决，例如更自由的数据源定制、跟完善的多字段处理方案等等，下一篇[《进阶》](https://blog.csdn.net/boling_cavalry/article/details/108942301)咱们一起来体验更多参数化测试的高级功能；

# 参数化测试(Parameterized Tests)进阶

### 自定义数据源

1. 前文使用了很多种数据源，如果您对它们的各种限制不满意，想要做更彻底的个性化定制，可以开发ArgumentsProvider接口的实现类，并使用@ArgumentsSource指定；
2. 举个例子，先开发ArgumentsProvider的实现类MyArgumentsProvider.java：

```java
package com.bolingcavalry.parameterized.service.impl;

import org.junit.jupiter.api.extension.ExtensionContext;
import org.junit.jupiter.params.provider.Arguments;
import org.junit.jupiter.params.provider.ArgumentsProvider;
import java.util.stream.Stream;

public class MyArgumentsProvider implements ArgumentsProvider {

    @Override
    public Stream<? extends Arguments> provideArguments(ExtensionContext context) throws Exception {
        return Stream.of("apple4", "banana4").map(Arguments::of);
    }
}
1234567891011121314
```

1. 再给测试方法添加@ArgumentsSource，并指定MyArgumentsProvider：

```java
    @Order(15)
    @DisplayName("ArgumentsProvider接口的实现类提供的数据作为入参")
    @ParameterizedTest
    @ArgumentsSource(MyArgumentsProvider.class)
    void argumentsSourceTest(String candidate) {
        log.info("argumentsSourceTest [{}]", candidate);
    }
1234567
```

1. 执行结果如下：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201006121420500.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)

### 参数转换

1. 参数化测试的数据源和测试方法入参的数据类型必须要保持一致吗？其实JUnit5并没有严格要求，而事实上JUnit5是可以做一些自动或手动的类型转换的；
2. 如下代码，数据源是int型数组，但测试方法的入参却是double：

```java
    @Order(16)
    @DisplayName("int型自动转为double型入参")
    @ParameterizedTest
    @ValueSource(ints = { 1,2,3 })
    void argumentConversionTest(double candidate) {
        log.info("argumentConversionTest [{}]", candidate);
    }
1234567
```

1. 执行结果如下，可见int型被转为double型传给测试方法（Widening Conversion）：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201006122222100.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)
2. 还可以指定转换器，以转换器的逻辑进行转换，下面这个例子就是将字符串转为LocalDate类型，关键是@JavaTimeConversionPattern：

```java
    @Order(17)
    @DisplayName("string型，指定转换器，转为LocalDate型入参")
    @ParameterizedTest
    @ValueSource(strings = { "01.01.2017", "31.12.2017" })
    void argumentConversionWithConverterTest(
            @JavaTimeConversionPattern("dd.MM.yyyy") LocalDate candidate) {
        log.info("argumentConversionWithConverterTest [{}]", candidate);
    }
12345678
```

1. 执行结果如下：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201006122946246.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)

### 字段聚合(Argument Aggregation)

1. 来思考一个问题：如果数据源的每条记录有多个字段，测试方法如何才能使用这些字段呢？
2. 回顾刚才的@CsvSource示例，如下图，可见测试方法用两个入参对应CSV每条记录的两个字段，如下所示：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201006123555541.jpg#pic_left)
3. 上述方式应对少量字段还可以，但如果CSV每条记录有很多字段，那测试方法岂不是要定义大量入参？这显然不合适，此时可以考虑JUnit5提供的字段聚合功能(Argument Aggregation)，也就是将CSV每条记录的所有字段都放入一个ArgumentsAccessor类型的对象中，测试方法只要声明ArgumentsAccessor类型作为入参，就能在方法内部取得CSV记录的所有字段，效果如下图，可见CSV字段实际上是保存在ArgumentsAccessor实例内部的一个Object数组中：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/2020100616403441.jpg#pic_center)
4. 如下图，为了方便从ArgumentsAccessor实例获取数据，ArgumentsAccessor提供了获取各种类型的方法，您可以按实际情况选用：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201006164320682.jpg#pic_left)
5. 下面的示例代码中，CSV数据源的每条记录有三个字段，而测试方法只有一个入参，类型是ArgumentsAccessor，在测试方法内部，可以用ArgumentsAccessor的getString、get等方法获取CSV记录的不同字段，例如arguments.getString(0)就是获取第一个字段，得到的结果是字符串类型，而arguments.get(2, Types.class)的意思是获取第二个字段，并且转成了Type.class类型：

```java
    @Order(18)
    @DisplayName("CsvSource的多个字段聚合到ArgumentsAccessor实例")
    @ParameterizedTest
    @CsvSource({
            "Jane1, Doe1, BIG",
            "John1, Doe1, SMALL"
    })
    void argumentsAccessorTest(ArgumentsAccessor arguments) {
        Person person = new Person();
        person.setFirstName(arguments.getString(0));
        person.setLastName(arguments.getString(1));
        person.setType(arguments.get(2, Types.class));

        log.info("argumentsAccessorTest [{}]", person);
    }
123456789101112131415
```

1. 上述代码执行结果如下图，可见通过ArgumentsAccessor能够取得CSV数据的所有字段：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201006165125634.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)

### 更优雅的聚合

1. 前面的聚合解决了获取CSV数据多个字段的问题，但依然有瑕疵：从ArgumentsAccessor获取数据生成Person实例的代码写在了测试方法中，如下图红框所示，测试方法中应该只有单元测试的逻辑，而创建Person实例的代码放在这里显然并不合适：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201006172127491.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)
2. 针对上面的问题，JUnit5也给出了方案：通过注解的方式，指定一个从ArgumentsAccessor到Person的转换器，示例如下，可见测试方法的入参有个注解@AggregateWith，其值PersonAggregator.class就是从ArgumentsAccessor到Person的转换器，而入参已经从前面的ArgumentsAccessor变成了Person：

```java
    @Order(19)
    @DisplayName("CsvSource的多个字段，通过指定聚合类转为Person实例")
    @ParameterizedTest
    @CsvSource({
            "Jane2, Doe2, SMALL",
            "John2, Doe2, UNKNOWN"
    })
    void customAggregatorTest(@AggregateWith(PersonAggregator.class) Person person) {
        log.info("customAggregatorTest [{}]", person);
    }
12345678910
```

1. PersonAggregator是转换器类，需要实现ArgumentsAggregator接口，具体的实现代码很简单，也就是从ArgumentsAccessor示例获取字段创建Person对象的操作：

```java
package com.bolingcavalry.parameterized.service.impl;

import org.junit.jupiter.api.extension.ParameterContext;
import org.junit.jupiter.params.aggregator.ArgumentsAccessor;
import org.junit.jupiter.params.aggregator.ArgumentsAggregationException;
import org.junit.jupiter.params.aggregator.ArgumentsAggregator;

public class PersonAggregator implements ArgumentsAggregator {

    @Override
    public Object aggregateArguments(ArgumentsAccessor arguments, ParameterContext context) throws ArgumentsAggregationException {

        Person person = new Person();
        person.setFirstName(arguments.getString(0));
        person.setLastName(arguments.getString(1));
        person.setType(arguments.get(2, Types.class));

        return person;
    }
}
1234567891011121314151617181920
```

1. 上述测试方法的执行结果如下：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201006173839559.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)

### 进一步简化

1. 回顾一下刚才用注解指定转换器的代码，如下图红框所示，您是否回忆起JUnit5支持自定义注解这一茬，咱们来把红框部分的代码再简化一下：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201006175212615.jpg#pic_left)
2. 新建注解类CsvToPerson.java，代码如下，非常简单，就是把上图红框中的@AggregateWith(PersonAggregator.class)搬过来了：

```java
package com.bolingcavalry.parameterized.service.impl;

import org.junit.jupiter.params.aggregator.AggregateWith;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.PARAMETER)
@AggregateWith(PersonAggregator.class)
public @interface CsvToPerson {
}
12345678910111213
```

1. 再来看看上图红框中的代码可以简化成什么样子，直接用@CsvToPerson就可以将ArgumentsAccessor转为Person对象了：

```java
    @Order(20)
    @DisplayName("CsvSource的多个字段，通过指定聚合类转为Person实例(自定义注解)")
    @ParameterizedTest
    @CsvSource({
            "Jane3, Doe3, BIG",
            "John3, Doe3, UNKNOWN"
    })
    void customAggregatorAnnotationTest(@CsvToPerson Person person) {
        log.info("customAggregatorAnnotationTest [{}]", person);
    }
12345678910
```

1. 执行结果如下，可见和@AggregateWith(PersonAggregator.class)效果一致：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201006182725314.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)

### 测试执行名称自定义

1. 文章最后，咱们来看个轻松的知识点吧，如下图红框所示，每次执行测试方法，IDEA都会展示这次执行的序号和参数值：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/2020100618424952.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)
2. 其实上述红框中的内容格式也可以定制，格式模板就是@ParameterizedTest的name属性，修改后的测试方法完整代码如下，可见这里改成了中文描述信息：

```java
    @Order(21)
    @DisplayName("CSV格式多条记录入参(自定义展示名称)")
    @ParameterizedTest(name = "序号 [{index}]，fruit参数 [{0}]，rank参数 [{1}]")
    @CsvSource({
            "apple3, 31",
            "banana3, 32",
            "'lemon3, lime3', 0x3A"
    })
    void csvSourceWithCustomDisplayNameTest(String fruit, int rank) {
        log.info("csvSourceWithCustomDisplayNameTest, fruit [{}], rank [{}]", fruit, rank);
    }
1234567891011
```

1. 执行结果如下：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201006185438542.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)

- 至此，JUnit5的参数化测试(Parameterized)相关的知识点已经学习和实战完成了，掌握了这么强大的参数输入技术，咱们的单元测试的代码覆盖率和场景范围又可以进一步提升了；

# 综合进阶（终篇）

### 版本设置

- 《JUnit5学习》系列的代码都在用SpringBoot：2.3.4.RELEASE框架，间接依赖的JUnit版本是5.6.2；
- 本文有两个特性要求JUnit版本达到5.7或者更高，它们是测试方法展现名称生成器和动态生成测试方法；
- 对于使用SpringBoot：2.3.4.RELEASE框架的工程，如果要指定JUnit版本，需要做以下三步操作：

1. dependencyManagement节点添加junit-bom，并指定版本号：

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.junit</groupId>
      <artifactId>junit-bom</artifactId>
      <version>5.7.0</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
1234567891011
```

1. 排除spring-boot-starter-test和junit-jupiter的间接依赖关系：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
  <scope>test</scope>
  <exclusions>
    <exclusion>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter</artifactId>
    </exclusion>
  </exclusions>
</dependency>
1234567891011
```

1. 添加junit-jupiter依赖，此时会使用dependencyManagement中指定的版本号：

```xml
<dependency>
  <groupId>org.junit.jupiter</groupId>
  <artifactId>junit-jupiter</artifactId>
  <scope>test</scope>
</dependency>
12345
```

1. 如下图，刷新可见已经用上了5.7.0版本：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201007173846994.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)

- 版本问题解决了，接下来正式进入进阶实战；

### 测试方法展现名称生成器(Display Name Generators)

1. 把Display Name Generators翻译成测试方法展现名称生成器，可能刷新了读者们对本文作者英文水平的认知，请您多包含…
2. 先回顾一下如何指定测试方法的展现名称，如果测试方法使用了@DisplayName，在展示单元测试执行结果时，就会显示@DisplayName指定的字符串，如下图所示：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201007180121692.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)
3. 除了用@DisplayName指定展示名称，JUnit5还提供了一种自动生成展示名称的功能：@DisplayNameGeneration，来看看它是如何生成展示名称的；
4. 演示代码如下所示，当@DisplayNameGeneration的value设置为ReplaceUnderscores时，会把方法名的所有下划线替换为空格：

```java
package com.bolingcavalry.advanced.service.impl;

import org.junit.jupiter.api.DisplayNameGeneration;
import org.junit.jupiter.api.DisplayNameGenerator;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
public class ReplaceUnderscoresTest {

    @Test
    void if_it_is_zero() {
    }
}
123456789101112131415
```

1. 执行结果如下图，方法if_it_is_zero展示出的名字为if it is zero：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201007181804774.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)
2. 在上述替换方式的基础上，JUnit5还提供了另一种生成展示名称的方法：测试类名+连接符+测试方法名，并且类名和方法名的下划线都会被替换成空格，演示代码如下，使用了注解@IndicativeSentencesGeneration，其separator属性就是类名和方法名之间的连接符：

```java
package com.bolingcavalry.advanced.service.impl;

import org.junit.jupiter.api.DisplayNameGenerator;
import org.junit.jupiter.api.IndicativeSentencesGeneration;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
@IndicativeSentencesGeneration(separator = "，测试方法：", generator = DisplayNameGenerator.ReplaceUnderscores.class)
public class IndicativeSentences_Test {

    @Test
    void if_it_is_one_of_the_following_years() {
    }
}
123456789101112131415
```

1. 执行结果如下：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201007183416612.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)

### 重复测试（Repeated Tests）

1. 重复测试就是指定某个测试方法反复执行多次，演示代码如下，可见@Test已被@RepeatedTest(5)取代，数字5表示重复执行5次：

```java
    @Order(1)
    @DisplayName("重复测试")
    @RepeatedTest(5)
    void repeatTest(TestInfo testInfo) {
        log.info("测试方法 [{}]", testInfo.getTestMethod().get().getName());
    }
123456
```

1. 执行结果如下图：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201007183954175.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)
2. 在测试方法执行时，如果想了解当前是第几次执行，以及总共有多少次，只要给测试方法增加RepetitionInfo类型的入参即可，演示代码如下，可见RepetitionInfo提供的API可以得到总数和当前次数：

```java
    @Order(2)
    @DisplayName("重复测试，从入参获取执行情况")
    @RepeatedTest(5)
    void repeatWithParamTest(TestInfo testInfo, RepetitionInfo repetitionInfo) {
        log.info("测试方法 [{}]，当前第[{}]次，共[{}]次",
                testInfo.getTestMethod().get().getName(),
                repetitionInfo.getCurrentRepetition(),
                repetitionInfo.getTotalRepetitions());
    }
123456789
```

1. 上述代码执行结果如下：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201007184643444.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)
2. 在上图的左下角可见，重复执行的结果被展示为"repetition X of X"这样的内容，其实这部分信息是可以定制的，就是RepeatedTest注解的name属性，演示代码如下，可见currentRepetition和totalRepetitions是占位符，在真正展示的时候会被分别替换成当前值和总次数：

```java
    @Order(3)
    @DisplayName("重复测试，使用定制名称")
    @RepeatedTest(value = 5, name="完成度：{currentRepetition}/{totalRepetitions}")
    void repeatWithCustomDisplayNameTest(TestInfo testInfo, RepetitionInfo repetitionInfo) {
        log.info("测试方法 [{}]，当前第[{}]次，共[{}]次",
                testInfo.getTestMethod().get().getName(),
                repetitionInfo.getCurrentRepetition(),
                repetitionInfo.getTotalRepetitions());
    }
123456789
```

1. 上述代码执行结果如下：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201007185541192.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)

### 嵌套测试(Nested Tests)

1. 如果一个测试类中有很多测试方法（如增删改查，每种操作都有多个测试方法），那么不论是管理还是结果展现都会显得比较复杂，此时嵌套测试(Nested Tests)就派上用场了；
2. 嵌套测试(Nested Tests)功能就是在测试类中创建一些内部类，以增删改查为例，将所有测试查找的方法放入一个内部类，将所有测试删除的方法放入另一个内部类，再给每个内部类增加@Nested注解，这样就会以内部类为单位执行测试和展现结果，如下图所示：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201007192036597.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)
3. 嵌套测试的演示代码如下：

```java
package com.bolingcavalry.advanced.service.impl;

import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Nested;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
@Slf4j
@DisplayName("嵌套测试演示")
public class NestedTest {

    @Nested
    @DisplayName("查找服务相关的测试")
    class FindService {
        @Test
        void findByIdTest() {}
        @Test
        void findByNameTest() {}
    }

    @Nested
    @DisplayName("删除服务相关的测试")
    class DeleteService {
        @Test
        void deleteByIdTest() {}
        @Test
        void deleteByNameTest() {}
    }
}
12345678910111213141516171819202122232425262728293031
```

1. 上述代码执行结果如下，可见从代码管理再到执行和结果展示，都被分组管理了：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201007192325363.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)

### 动态测试(Dynamic Tests)

1. 之前咱们写的测试方法，主要是用@Test修饰，这些方法的特点就是在编译阶段就已经明确了，在运行阶段也已经固定；
2. JUnit5推出了另一种类型的测试方法：动态测试(Dynamic Tests)，首先，测试方法是可以在运行期间被生产出来的，生产它们的地方，就是被@TestFactory修饰的方法，等到测试方法被生产出来后再像传统的测试方法那样被执行和结果展示；
3. 下面是演示代码，testFactoryTest方法被@TestFactory修饰，返回值是Iterable类型，里面是多个DynamicTest实例，每个DynamicTest实例代表一个测试方法，因此，整个DynamicDemoTest类中有多少个测试方法，在编译阶段是不能确定的，只有在运行阶段执行了testFactoryTest方法后，才能根据返回值确定下来：

```java
package com.bolingcavalry.advanced.service.impl;

import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.DynamicTest;
import org.junit.jupiter.api.TestFactory;
import org.springframework.boot.test.context.SpringBootTest;
import java.util.Arrays;
import static org.junit.jupiter.api.DynamicTest.dynamicTest;

@SpringBootTest
@Slf4j
class DynamicDemoTest {

    @TestFactory
    Iterable<org.junit.jupiter.api.DynamicTest> testFactoryTest() {

        DynamicTest firstTest = dynamicTest(
            "一号动态测试用例",
            () -> {
                log.info("一号用例，这里编写单元测试逻辑代码");
            }
        );

        DynamicTest secondTest = dynamicTest(
                "二号动态测试用例",
                () -> {
                    log.info("二号用例，这里编写单元测试逻辑代码");
                }
        );

        return Arrays.asList(firstTest, secondTest);
    }
}
123456789101112131415161718192021222324252627282930313233
```

1. 上述代码的执行结果如下，可见每个DynamicTest实例就相当于以前的一个@Test修饰的方法，会被执行和统计：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201007194138946.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)

### 多线程并发执行(Parallel Execution)的介绍

- 《JUnit5学习》系列的最后，咱们来看一个既容易理解又实用的特性：多线程并发执行(Parallel Execution)
- JUnit5中的并发执行测试可以分为以下三种场景：

1. 多个测试类，它们各自的测试方法同时执行；
2. 一个测试类，里面的多个测试方法同时执行；
3. 一个测试类，里面的一个测试方法，在重复测试(Repeated Tests)或者参数化测试(Parameterized Tests)的时候，这个测试方法被多个线程同时执行；

### 多线程并发执行(Parallel Execution)实战

1. 前面介绍了多线程并发执行有三种场景，文章篇幅所限就不逐个编码实战了，就选择第三种场景来实践吧，即：一个测试类里面的一个测试方法，在重复测试时多线程并发执行，至于其他两种场景如何设置，接下来的文中也会讲清楚，您自行实践即可；
2. 首先是创建JUnit5的配置文件，如下图，在test文件夹上点击鼠标右键，在弹出的菜单选择"New"->“Directory”:
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201007201812119.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)
3. 弹出的窗口如下图，双击红框位置的"resources"，即可新建resources目录：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201007202008102.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)
4. 在新增的resources目录中新建文件junit-platform.properties，内容如下，每个配置项都有详细的说明：

```properties
# 并行开关true/false
junit.jupiter.execution.parallel.enabled=true
# 方法级多线程开关 same_thread/concurrent
junit.jupiter.execution.parallel.mode.default = same_thread
# 类级多线程开关 same_thread/concurrent
junit.jupiter.execution.parallel.mode.classes.default = same_thread

# 并发策略有以下三种可选：
# fixed：固定线程数，此时还要通过junit.jupiter.execution.parallel.config.fixed.parallelism指定线程数
# dynamic：表示根据处理器和核数计算线程数
# custom：自定义并发策略，通过这个配置来指定：junit.jupiter.execution.parallel.config.custom.class
junit.jupiter.execution.parallel.config.strategy = fixed

# 并发线程数，该配置项只有当并发策略为fixed的时候才有用
junit.jupiter.execution.parallel.config.fixed.parallelism = 5
123456789101112131415
```

1. 由于实践的是同一个类同一个方法多次执行的并发，因此上述配置中，类级多线程开关和方法级多线程开关都选择了"同一个线程"，也就是说不需要并发执行多个类或者多个方法，请您根据自己的需求自行调整；
2. 关于并发策略，这里选择的是动态调整，我这里是i5-8400处理器，拥有六核心六线程，稍后咱们看看执行效果与这个硬件配置是否有关系；
3. 接下来编写测试代码，先写一个单线程执行的，可见@Execution的值为SAME_THREAD，限制了重复测试时在同一个线程内顺序执行：

```java
package com.bolingcavalry.advanced.service.impl;

import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.*;
import org.junit.jupiter.api.parallel.Execution;
import org.junit.jupiter.api.parallel.ExecutionMode;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ValueSource;
import org.springframework.boot.test.context.SpringBootTest;
import static org.junit.jupiter.api.Assertions.assertTrue;

@SpringBootTest
@Slf4j
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class ParallelExecutionTest {

    @Order(1)
    @Execution(ExecutionMode.SAME_THREAD)
    @DisplayName("单线程执行10次")
    @RepeatedTest(value = 10, name="完成度：{currentRepetition}/{totalRepetitions}")
    void sameThreadTest(TestInfo testInfo, RepetitionInfo repetitionInfo) {
        log.info("测试方法 [{}]，当前第[{}]次，共[{}]次",
                testInfo.getTestMethod().get().getName(),
                repetitionInfo.getCurrentRepetition(),
                repetitionInfo.getTotalRepetitions());
    }
}
123456789101112131415161718192021222324252627
```

1. 执行结果如下，可见确实是单线程：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201007203555606.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)
2. 重复测试时并发执行的代码如下，@Execution的值为CONCURRENT：

```java
    @Order(2)
    @Execution(ExecutionMode.CONCURRENT)
    @DisplayName("多线程执行10次")
    @RepeatedTest(value = 10, name="完成度：{currentRepetition}/{totalRepetitions}")
    void concurrentTest(TestInfo testInfo, RepetitionInfo repetitionInfo) {
        log.info("测试方法 [{}]，当前第[{}]次，共[{}]次",
                testInfo.getTestMethod().get().getName(),
                repetitionInfo.getCurrentRepetition(),
                repetitionInfo.getTotalRepetitions());
    }
12345678910
```

1. 执行结果如下，从红框1可见顺序已经乱了，从红框2可见十次测试方法是在五个线程中执行的：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/2020100720442434.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)
2. 最后是参数化测试的演示，也可以设置为多线程并行执行：

```java
    @Order(3)
    @Execution(ExecutionMode.CONCURRENT)
    @DisplayName("多个int型入参")
    @ParameterizedTest
    @ValueSource(ints = { 1,2,3,4,5,6,7,8,9,0 })
    void intsTest(int candidate) {
        log.info("ints [{}]", candidate);
    }
12345678
```

1. 执行结果如下图，可见也是5个线程并行执行的：
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201007204823295.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JvbGluZ19jYXZhbHJ5,size_16,color_FFFFFF,t_70#pic_left)

### 结束语

至此，《JUnit5学习》系列已经全部完成，感谢您的耐心阅读，希望这个原创系列能够带给您一些有用的信息，为您的单元测试提供一些参考，如果发现文章有错误，期待您能指点一二；

# 参考

https://blog.csdn.net/boling_cavalry/article/details/108810587