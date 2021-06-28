# 纯 Java 搭建 SSM 项目

在 Spring Boot 项目中，正常来说是不存在 XML 配置，这是因为 Spring Boot 不推荐使用 XML ，注意，并非不支持，Spring Boot 推荐开发者使用 Java 配置来搭建框架，Spring Boot 中，大量的自动化配置都是通过 Java 配置来实现的，这一套实现方案，我们也可以自己做，即自己也可以使用纯 Java 来搭建一个 SSM 环境，即在项目中，不存在任何 XML 配置，包括 web.xml 。



环境要求：

- 使用纯 Java 来搭建 SSM 环境，要求 Tomcat 的版本必须在 7 以上。



## 1 创建工程

创建一个普通的 Maven 工程（注意，这里可以不必创建 Web 工程），并添加 SpringMVC 的依赖，同时，这里环境的搭建需要用到 Servlet ，所以我们还需要引入 Servlet 的依赖（一定不能使用低版本的 Servlet），最终的 pom.xml 文件如下：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.1.6.RELEASE</version>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
    <scope>provided</scope>
</dependency>
```

## 2 添加 Spring 配置

工程创建成功之后，首先添加 Spring 的配置文件，如下：

```java
@Configuration
@ComponentScan(basePackages = "org.javaboy", useDefaultFilters = true, excludeFilters = {@ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Controller.class)})
public class SpringConfig {
}
```

关于这个配置，我说如下几点：

- @Configuration 注解表示这是一个配置类，在我们这里，这个配置的作用类似于 applicationContext.xml
- @ComponentScan 注解表示配置包扫描，里边的属性和 xml 配置中的属性都是一一对应的，useDefaultFilters 表示使用默认的过滤器，然后又除去 Controller 注解，即在 Spring 容器中扫描除了 Controller 之外的其他所有 Bean 。

## 3 添加 SpringMVC 配置

接下来再来创建 springmvc 的配置文件：

```java
@Configuration
@ComponentScan(basePackages = "org.javaboy",useDefaultFilters = false,includeFilters = {@ComponentScan.Filter(type = FilterType.ANNOTATION,classes = Controller.class)})
public class SpringMVCConfig {
}
```

**注意，如果不需要在 SpringMVC 中添加其他的额外配置，这样就可以了。即 视图解析器、JSON 解析、文件上传……等等，如果都不需要配置的话，这样就可以了。**

## 4 配置 web.xml

此时，我们并没有 web.xml 文件，这时，我们可以使用 Java 代码去代替 web.xml 文件，这里会用到 WebApplicationInitializer ，具体定义如下：

```java
public class WebInit implements WebApplicationInitializer {
    public void onStartup(ServletContext servletContext) throws ServletException {
        //首先来加载 SpringMVC 的配置文件
        AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
        ctx.register(SpringMVCConfig.class);
        // 添加 DispatcherServlet
        ServletRegistration.Dynamic springmvc = servletContext.addServlet("springmvc", new DispatcherServlet(ctx));
        // 给 DispatcherServlet 添加路径映射
        springmvc.addMapping("/");
        // 给 DispatcherServlet 添加启动时机
        springmvc.setLoadOnStartup(1);
    }
}
```

WebInit 的作用类似于 web.xml，这个类需要实现 WebApplicationInitializer 接口，并实现接口中的方法，当项目启动时，onStartup 方法会被自动执行，我们可以在这个方法中做一些项目初始化操作，例如加载 SpringMVC 容器，添加过滤器，添加 Listener、添加 Servlet 等。

**注意：**

由于我们在 WebInit 中只是添加了 SpringMVC 的配置，这样项目在启动时只会去加载 SpringMVC 容器，而不会去加载 Spring 容器，如果一定要加载 Spring 容器，需要我们修改 SpringMVC 的配置，在 SpringMVC 配置的包扫描中也去扫描 @Configuration 注解，进而加载 Spring 容器，还有一种方案可以解决这个问题，就是直接在项目中舍弃 Spring 配置，直接将所有配置放到 SpringMVC 的配置中来完成，这个在 SSM 整合时是没有问题的，在实际开发中，较多采用第二种方案，第二种方案，SpringMVC 的配置如下：

```java
@Configuration
@ComponentScan(basePackages = "org.javaboy")
public class SpringMVCConfig {
}
```

这种方案中，所有的注解都在 SpringMVC 中扫描，采用这种方案的话，则 Spring 的配置文件就可以删除了。

## 5 测试

最后，添加一个 HelloController ，然后启动项目进行测试：

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
}
```

启动项目，访问接口，结果如下：
![1-1](https://typoralim.oss-cn-beijing.aliyuncs.com/img/1-1.png)

## 6 其他配置

### 6.1 静态资源过滤

静态资源过滤在 SpringMVC 的 XML 中的配置如下：

```xml
<mvc:resources mapping="/**" location="/"/>
```

在 Java 配置的 SSM 环境中，如果要配置静态资源过滤，需要让 SpringMVC 的配置继承 WebMvcConfigurationSupport ，进而重写 WebMvcConfigurationSupport 中的方法，如下：

```java
@Configuration
@ComponentScan(basePackages = "org.javaboy")
public class SpringMVCConfig extends WebMvcConfigurationSupport {
    @Override
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/js/**").addResourceLocations("classpath:/");
    }
}
```

重写 addResourceHandlers 方法，在这个方法中配置静态资源过滤，这里我将静态资源放在 resources 目录下，所以资源位置是 `classpath:/` ，当然，资源也可以放在 webapp 目录下，此时只需要修改配置中的资源位置即可。如果采用 Java 来配置 SSM 环境，一般来说，可以不必使用 webapp 目录，除非要使用 JSP 做页面模板，否则可以忽略 webapp 目录。

### 6.2 视图解析器

在 XML 文件中，通过如下方式配置视图解析器：

```java
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

如果通过 Java 类，一样也可以实现类似功能。

首先为我们的项目添加 webapp 目录，webapp 目录中添加一个 jsp 目录，jsp 目录中添加 jsp 文件：
![1-2](https://typoralim.oss-cn-beijing.aliyuncs.com/img/1-2.png)

然后引入 JSP 的依赖：

```java
<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>javax.servlet.jsp-api</artifactId>
    <version>2.3.1</version>
</dependency>
```

然后，在配置类中，继续重写方法：

```java
@Configuration
@ComponentScan(basePackages = "org.javaboy")
public class SpringMVCConfig extends WebMvcConfigurationSupport {
    @Override
    protected void configureViewResolvers(ViewResolverRegistry registry) {
        registry.jsp("/jsp/", ".jsp");
    }
}
```

接下来，在 Controller 中添加控制器即可访问 JSP 页面：

```java
@Controller
public class HelloController2 {
    @GetMapping("/hello2")
    public String hello() {
        return "hello";
    }
}
```

### 6.3 路径映射

有的时候，我们的控制器的作用仅仅只是一个跳转，就像上面小节中的控制器，里边没有任何业务逻辑，像这种情况，可以不用定义方法，可以直接通过路径映射来实现页面访问。如果在 `XML` 中配置路径映射，如下：

```xml
<mvc:view-controller path="/hello" view-name="hello" status-code="200"/>
```

这行配置，表示如果用户访问 `/hello` 这个路径，则直接将名为 `hello` 的视图返回给用户，并且响应码为 `200`，这个配置就可以替代 `Controller` 中的方法。

相同的需求，如果在 `Java` 代码中，写法如下：

```java
@Configuration
@ComponentScan(basePackages = "org.javaboy")
public class SpringMVCConfig extends WebMvcConfigurationSupport {
    @Override
    protected void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/hello3").setViewName("hello");
    }
}
```

此时，用户访问 `/hello3` 接口，就能看到名为 `hello` 的视图文件。

### 6.4 JSON 配置

SpringMVC 可以接收JSON 参数，也可以返回 JSON 参数，这一切依赖于 HttpMessageConverter。

HttpMessageConverter 可以将一个 JSON 字符串转为 对象，也可以将一个对象转为 JSON 字符串，实际上它的底层还是依赖于具体的 JSON 库。

所有的 JSON 库要在 SpringMVC 中自动返回或者接收 JSON，都必须提供和自己相关的 HttpMessageConverter 。

SpringMVC 中，默认提供了 Jackson 和 gson 的 HttpMessageConverter ，分别是：MappingJackson2HttpMessageConverter 和 GsonHttpMessageConverter 。

正因为如此，我们在 SpringMVC 中，如果要使用 JSON ，对于 jackson 和 gson 我们只需要添加依赖，加完依赖就可以直接使用了。具体的配置是在 AllEncompassingFormHttpMessageConverter 类中完成的。

如果开发者使用了 fastjson，那么默认情况下，SpringMVC 并没有提供 fastjson 的 HttpMessageConverter ，这个需要我们自己提供，如果是在 XML 配置中，fastjson 除了加依赖，还要显式配置 HttpMessageConverter，如下：

```xml
<mvc:annotation-driven>
    <mvc:message-converters>
        <bean class="com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter">
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>
```

在 Java 配置的 SSM 中，我们一样也可以添加这样的配置：

```java
@Configuration
@ComponentScan(basePackages = "org.javaboy")
public class SpringMVCConfig extends WebMvcConfigurationSupport {
    @Override
    protected void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        FastJsonHttpMessageConverter converter = new FastJsonHttpMessageConverter();
        converter.setDefaultCharset(Charset.forName("UTF-8"));
        FastJsonConfig fastJsonConfig = new FastJsonConfig();
        fastJsonConfig.setCharset(Charset.forName("UTF-8"));
        converter.setFastJsonConfig(fastJsonConfig);
        converters.add(converter);
    }
}
```

然后，就可以在接口中直接返回 JSON 了，此时的 JSON 数据将通过 fastjson 生成。

# 创建一个 Spring Boot 项目的三种方法

## Spring Boot 介绍

我们刚开始学习 JavaWeb 的时候，使用 Servlet/JSP 做开发，一个接口搞一个 Servlet ，很头大，后来我们通过隐藏域或者反射等方式，可以减少 Servlet 的创建，但是依然不方便，再后来，我们引入 Struts2/SpringMVC 这一类的框架，来简化我们的开发 ，和 Servlet/JSP 相比，引入框架之后，生产力确实提高了不少，但是用久了，又发现了新的问题，即配置繁琐易出错，要做一个新项目，先搭建环境，环境搭建来搭建去，就是那几行配置，不同的项目，可能就是包不同，其他大部分的配置都是一样的，Java 总是被人诟病配置繁琐代码量巨大，这就是其中一个表现。那么怎么办？Spring Boot 应运而生，Spring Boot 主要提供了如下功能：

1. 为所有基于 Spring 的 Java 开发提供方便快捷的入门体验。
2. 开箱即用，有自己自定义的配置就是用自己的，没有就使用官方提供的默认的。
3. 提供了一系列通用的非功能性的功能，例如嵌入式服务器、安全管理、健康检测等。
4. 绝对没有代码生成，也不需要XML配置。

Spring Boot 的出现让 Java 开发又回归简单，因为确确实实解决了开发中的痛点，因此这个技术得到了非常广泛的使用，松哥很多朋友出去面试 Java 工程师，从2017年年初开始，Spring Boot基本就是必问，现在流行的 Spring Cloud 微服务也是基于 Spring Boot，因此，所有的 Java 工程师都有必要掌握好 Spring Boot。

## 系统要求

截至本文写作（2019.09），Spring Boot 目前最新版本是 2.1.8，要求至少 JDK8，集成的 Spring 版本是 5.1.9 ，构建工具版本要求如下：

| Build Tool | Version |
| :--------- | :------ |
| Maven      | 3.3+    |
| Gradle     | 4.4+    |

内置的容器版本分别如下：

| Name         | Version |
| :----------- | :------ |
| Tomcat 9.0   | 4.0     |
| Jetty 9.4    | 3.1     |
| Undertow 2.0 | 4.0     |

## 三种创建方式

初学者看到 Spring Boot 工程创建成功后有那么多文件就会有点懵圈，其实 Spring Boot 工程本质上就是一个 Maven 工程，从这个角度出发，松哥在这里向大家介绍三种项目创建方式。

### 在线创建

这是官方提供的一个创建方式，实际上，如果我们使用开发工具去创建 Spring Boot 项目的话（即第二种方案），也是从这个网站上创建的，只不过这个过程开发工具帮助我们完成了，我们只需要在开发工具中进行简单的配置即可。

首先打开 `https://start.spring.io` 这个网站，如下：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/2-2.png)](http://www.javaboy.org/images/boot2/2-2.png)

这里要配置的按顺序分别如下：

- 项目构建工具是 Maven 还是 Gradle ？松哥见到有人用 Gradle 做 Java 后端项目，但是整体感觉 Gradle 在 Java 后端中使用的还是比较少，Gradle 在 Android 中使用较多，Java 后端，目前来看还是 Maven 为主，因此这里选择第一项。
- 开发语言，这个当然是选择 Java 了。
- Spring Boot 版本，可以看到，目前最新的稳定版是 2.1.8 ，这里我们就是用最新稳定版。
- 既然是 Maven 工程，当然要有项目坐标，项目描述等信息了，另外这里还让输入了包名，因为创建成功后会自动创建启动类。
- Packing 表示项目要打包成 jar 包还是 war 包，Spring Boot 的一大优势就是内嵌了 Servlet 容器，打成 jar 包后可以直接运行，所以这里建议打包成 jar 包，当然，开发者根据实际情况也可以选择 war 包。
- 然后选选择构建的 JDK 版本。
- 最后是选择所需要的依赖，输入关键字如 web ，会有相关的提示，这里我就先加入 web 依赖。

所有的事情全部完成后，点击最下面的 `Generate Project` 按钮，或者点击 `Alt+Enter` 按键，此时会自动下载项目，将下载下来的项目解压，然后用 IntelliJ IDEA 或者 Eclipse 打开即可进行开发。

### 使用开发工具创建

有人觉得上面的步骤太过于繁琐，那么也可以使用 IDE 来创建，松哥这里以 IntelliJ IDEA 和 STS 为例，需要注意的是，IntelliJ IDEA 只有 ultimate 版才有直接创建 Spring Boot 项目的功能，社区版是没有此项功能的。

#### IntelliJ IDEA

首先在创建项目时选择 Spring Initializr，如下图：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/2-3.png)](http://www.javaboy.org/images/boot2/2-3.png)

然后点击 Next ，填入 Maven 项目的基本信息，如下：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/2-4.png)](http://www.javaboy.org/images/boot2/2-4.png)

再接下来选择需要添加的依赖，如下图：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/2-5.png)](http://www.javaboy.org/images/boot2/2-5.png)

勾选完成后，点击 Next 完成项目的创建。

#### STS

这里我再介绍下 Eclipse 派系的 STS 给大家参考， STS 创建 Spring Boot 项目，实际上也是从上一小节的那个网站上来的，步骤如下：

首先右键单击，选择 New -> Spring Starter Project ，如下图：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/2-6.png)](http://www.javaboy.org/images/boot2/2-6.png)

然后在打开的页面中填入项目的相关信息，如下图：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/2-7.png)](http://www.javaboy.org/images/boot2/2-7.png)

这里的信息和前面提到的都一样，不再赘述。最后一路点击 Next ，完成项目的创建。

### Maven 创建

上面提到的几种方式，实际上都借助了 `https://start.spring.io/` 这个网站，松哥记得在 2017 年的时候，这个网站还不是很稳定，经常发生项目创建失败的情况，从2018年开始，项目创建失败就很少遇到了，不过有一些读者偶尔还是会遇到这个问题，他们会在微信上问松哥这个问题腰怎么处理？我一般给的建议就是直接使用 Maven 来创建项目。步骤如下：

首先创建一个普通的 Maven 项目，以 IntelliJ IDEA 为例，创建步骤如下：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/2-8.png)](http://www.javaboy.org/images/boot2/2-8.png)

注意这里不用选择项目骨架（如果大伙是做练习的话，也可以去尝试选择一下，这里大概有十来个 Spring Boot 相关的项目骨架），直接点击 Next ，下一步中填入一个 Maven 项目的基本信息，如下图：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/2-9.png)](http://www.javaboy.org/images/boot2/2-9.png)

然后点击 Next 完成项目的创建。

创建完成后，在 pom.xml 文件中，添加如下依赖：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.8.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

添加成功后，再在 java 目录下创建包，包中创建一个名为 App 的启动类，如下：

```java
@EnableAutoConfiguration
@RestController
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }
}
```

@EnableAutoConfiguration 注解表示开启自动化配置。

然后执行这里的 main 方法就可以启动一个 Spring Boot 工程了。

## 项目结构

使用工具创建出来的项目结构大致如下图：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/2-10.png)](http://www.javaboy.org/images/boot2/2-10.png)

对于我们来说，src 是最熟悉的， Java 代码和配置文件写在这里，test 目录用来做测试，pom.xml 是 Maven 的坐标文件，就这几个。

# 理解 Spring Boot 项目中的 parent

前面和大伙聊了 Spring Boot 项目的三种创建方式，这三种创建方式，无论是哪一种，创建成功后，pom.xml 坐标文件中都有如下一段引用：



```xml
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.1.8.RELEASE</version>
	<relativePath/> <!-- lookup parent from repository -->
</parent>
```

对于这个 parent 的作用，你是否完全理解？有小伙伴说，不就是依赖的版本号定义在 parent 里边吗？是的，没错，但是 parent 的作用可不仅仅这么简单哦！本文松哥就来和大伙聊一聊这个 parent 到底有什么作用。

## 基本功能

当我们创建一个 Spring Boot 工程时，可以继承自一个 `spring-boot-starter-parent` ，也可以不继承自它，我们先来看第一种情况。先来看 parent 的基本功能有哪些？

1. 定义了 Java 编译版本为 1.8 。
2. 使用 UTF-8 格式编码。
3. 继承自 `spring-boot-dependencies`，这个里边定义了依赖的版本，也正是因为继承了这个依赖，所以我们在写依赖时才不需要写版本号。
4. 执行打包操作的配置。
5. 自动化的资源过滤。
6. 自动化的插件配置。
7. 针对 application.properties 和 application.yml 的资源过滤，包括通过 profile 定义的不同环境的配置文件，例如 application-dev.properties 和 application-dev.yml。

**请注意，由于application.properties和application.yml文件接受Spring样式占位符 `$ {...}` ，因此 Maven 过滤更改为使用 `@ .. @` 占位符，当然开发者可以通过设置名为 resource.delimiter 的Maven 属性来覆盖 `@ .. @` 占位符。**

### 源码分析

当我们创建一个 Spring Boot 项目后，我们可以在本地 Maven 仓库中看到看到这个具体的 parent 文件，以 2.1.8 这个版本为例，松哥 这里的路径是 `C:\Users\sang\.m2\repository\org\springframework\boot\spring-boot-starter-parent\2.1.8.RELEASE\spring-boot-starter-parent-2.1.8.RELEASE.pom` ,打开这个文件，快速阅读文件源码，基本上就可以证实我们前面说的功能，如下图：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/3-1.png)](http://www.javaboy.org/images/boot2/3-1.png)

我们可以看到，它继承自 `spring-boot-dependencies` ，这里保存了基本的依赖信息，另外我们也可以看到项目的编码格式，JDK 的版本等信息，当然也有我们前面提到的数据过滤信息。最后，我们再根据它的 parent 中指定的 `spring-boot-dependencies` 位置，来看看`spring-boot-dependencies` 中的定义：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/3-2.png)](http://www.javaboy.org/images/boot2/3-2.png)

在这里，我们看到了版本的定义以及 dependencyManagement 节点，明白了为啥 Spring Boot 项目中部分依赖不需要写版本号了。

## 不用 parent

但是并非所有的公司都需要这个 parent ，有的时候，公司里边会有自己定义的 parent ，我们的 Spring Boot 项目要继承自公司内部的 parent ，这个时候该怎么办呢？

一个简单的办法就是我们自行定义 dependencyManagement 节点，然后在里边定义好版本号，再接下来在引用依赖时也就不用写版本号了，像下面这样：

```xml
<dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-dependencies</artifactId>
			<version>2.1.8.RELEASE</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>
```

这样写之后，依赖的版本号问题虽然解决了，但是关于打包的插件、编译的 JDK 版本、文件的编码格式等等这些配置，在没有 parent 的时候，这些统统要自己去配置。

# 理解Spring Boot 配置文件 application.properties

在 Spring Boot 中，配置文件有两种不同的格式，一个是 properties ，另一个是 yaml 。



虽然 properties 文件比较常见，但是相对于 properties 而言，yaml 更加简洁明了，而且使用的场景也更多，很多开源项目都是使用 yaml 进行配置（例如 Hexo）。除了简洁，yaml 还有另外一个特点，就是 yaml 中的数据是有序的，properties 中的数据是无序的，在一些需要路径匹配的配置中，顺序就显得尤为重要（例如我们在 Spring Cloud Zuul 中的配置），此时我们一般采用 yaml。

本文主要来看看 properties 的问题。

## 位置问题

首先，当我们创建一个 Spring Boot 工程时，默认 resources 目录下就有一个 application.properties 文件，可以在 application.properties 文件中进行项目配置，但是这个文件并非唯一的配置文件，在 Spring Boot 中，一共有 4 个地方可以存放 application.properties 文件。

1. 当前项目根目录下的 config 目录下
2. 当前项目的根目录下
3. resources 目录下的 config 目录下
4. resources 目录下

按如上顺序，四个配置文件的优先级依次降低。如下：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/4-1.png)](http://www.javaboy.org/images/boot2/4-1.png)

这四个位置是默认位置，即 Spring Boot 启动，默认会从这四个位置按顺序去查找相关属性并加载。但是，这也不是绝对的，我们也可以在项目启动时自定义配置文件位置。

例如，现在在 resources 目录下创建一个 javaboy 目录，目录中存放一个 application.properties 文件，那么正常情况下，当我们启动 Spring Boot 项目时，这个配置文件是不会被自动加载的。我们可以通过 spring.config.location 属性来手动的指定配置文件位置，指定完成后，系统就会自动去指定目录下查找 application.properties 文件。

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/4-2.png)](http://www.javaboy.org/images/boot2/4-2.png)

此时启动项目，就会发现，项目以 `classpath:/javaboy/application.propertie` 配置文件启动。

这是在开发工具中配置了启动位置，如果项目已经打包成 jar ，在启动命令中加入位置参数即可：

```properties
java -jar properties-0.0.1-SNAPSHOT.jar --spring.config.location=classpath:/javaboy/
```

## 文件名问题

对于 application.properties 而言，它不一定非要叫 application ，但是项目默认是去加载名为 application 的配置文件，如果我们的配置文件不叫 application ，也是可以的，但是，需要明确指定配置文件的文件名。

方式和指定路径一致，只不过此时的 key 是 spring.config.name 。

首先我们在 resources 目录下创建一个 app.properties 文件，然后在 IDEA 中指定配置文件的文件名：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/4-3.png)](http://www.javaboy.org/images/boot2/4-3.png)

指定完配置文件名之后，再次启动项目，此时系统会自动去默认的四个位置下面分别查找名为 app.properties 的配置文件。当然，允许自定义文件名的配置文件不放在四个默认位置，而是放在自定义目录下，此时就需要明确指定 spring.config.location 。

配置文件位置和文件名称可以同时自定义。

## 普通的属性注入

由于 Spring Boot 源自 Spring ，所以 Spring 中存在的属性注入，在 Spring Boot 中一样也存在。由于 Spring Boot 中，默认会自动加载 application.properties 文件，所以简单的属性注入可以直接在这个配置文件中写。

例如，现在定义一个 Book 类：

```java
public class Book {
    private Long id;
    private String name;
    private String author;
    //省略 getter/setter
}
```

然后，在 application.properties 文件中定义属性：

```properties
book.name=三国演义
book.author=罗贯中
book.id=1
```

按照传统的方式（Spring中的方式），可以直接通过 @Value 注解将这些属性注入到 Book 对象中：

```java
@Component
public class Book {
    @Value("${book.id}")
    private Long id;
    @Value("${book.name}")
    private String name;
    @Value("${book.author}")
    private String author;
    //省略getter/setter
}
```

**注意**

Book 对象本身也要交给 Spring 容器去管理，如果 Book 没有交给 Spring 容器，那么 Book 中的属性也无法从 Spring 容器中获取到值。

配置完成后，在 Controller 或者单元测试中注入 Book 对象，启动项目，就可以看到属性已经注入到对象中了。

一般来说，我们在 application.properties 文件中主要存放系统配置，这种自定义配置不建议放在该文件中，可以自定义 properties 文件来存在自定义配置。

例如在 resources 目录下，自定义 book.properties 文件，内容如下：

```properties
book.name=三国演义
book.author=罗贯中
book.id=1
```

此时，项目启动并不会自动的加载该配置文件，如果是在 XML 配置中，可以通过如下方式引用该 properties 文件：

```xml
<context:property-placeholder location="classpath:book.properties"/>
```

如果是在 Java 配置中，可以通过 @PropertySource 来引入配置：

```java
@Component
@PropertySource("classpath:book.properties")
public class Book {
    @Value("${book.id}")
    private Long id;
    @Value("${book.name}")
    private String name;
    @Value("${book.author}")
    private String author;
    //getter/setter
}
```

这样，当项目启动时，就会自动加载 book.properties 文件。

这只是 Spring 中属性注入的一个简单用法，和 Spring Boot 没有任何关系。

## 类型安全的属性注入

Spring Boot 引入了类型安全的属性注入，如果采用 Spring 中的配置方式，当配置的属性非常多的时候，工作量就很大了，而且容易出错。

使用类型安全的属性注入，可以有效的解决这个问题。

```java
@Component
@PropertySource("classpath:book.properties")
@ConfigurationProperties(prefix = "book")
public class Book {
    private Long id;
    private String name;
    private String author;
    //省略getter/setter
}
```

这里，主要是引入 @ConfigurationProperties(prefix = “book”) 注解，并且配置了属性的前缀，此时会自动将 Spring 容器中对应的数据注入到对象对应的属性中，就不用通过 @Value 注解挨个注入了，减少工作量并且避免出错。

# Spring Boot中的 yaml 配置



## 狡兔三窟

首先 application.yaml 在 Spring Boot 中可以写在四个不同的位置，分别是如下位置：

1. 项目根目录下的 config 目录中
2. 项目根目录下
3. classpath 下的 config 目录中
4. classpath 目录下

四个位置中的 application.yaml 文件的优先级按照上面列出的顺序依次降低。即如果有同一个属性在四个文件中都出现了，以优先级高的为准。

那么 application.yaml 是不是必须叫 application.yaml 这个名字呢？当然不是必须的。开发者可以自己定义 yaml 名字，自己定义的话，需要在项目启动时指定配置文件的名字，像下面这样：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/5-1.png)](http://www.javaboy.org/images/boot2/5-1.png)

当然这是在 IntelliJ IDEA 中直接配置的，如果项目已经打成 jar 包了，则在项目启动时加入如下参数：

```properties
java -jar myproject.jar --spring.config.name=app
```

这样配置之后，在项目启动时，就会按照上面所说的四个位置按顺序去查找一个名为 app.yaml 的文件。当然这四个位置也不是一成不变的，也可以自己定义，有两种方式，一个是使用 `spring.config.location` 属性，另一个则是使用 `spring.config.additional-location` 这个属性，在第一个属性中，表示自己重新定义配置文件的位置，项目启动时就按照定义的位置去查找配置文件，这种定义方式会覆盖掉默认的四个位置，也可以使用第二种方式，第二种方式则表示在四个位置的基础上，再添加几个位置，新添加的位置的优先级大于原本的位置。

配置方式如下：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/5-2.png)](http://www.javaboy.org/images/boot2/5-2.png)

这里要注意，配置文件位置时，值一定要以 `/` 结尾。

## 数组注入

yaml 也支持数组注入，例如

```properties
my:
  servers:
	- dev.example.com
	- another.example.com
```

这段数据可以绑定到一个带 Bean 的数组中：

```java
@ConfigurationProperties(prefix="my")
@Component
public class Config {

	private List<String> servers = new ArrayList<String>();

	public List<String> getServers() {
		return this.servers;
	}
}
```

项目启动后，配置中的数组会自动存储到 servers 集合中。当然，yaml 不仅可以存储这种简单数据，也可以在集合中存储对象。例如下面这种：

```yaml
redis:
  redisConfigs:
    - host: 192.168.66.128
      port: 6379
    - host: 192.168.66.129
      port: 6380
```

这个可以被注入到如下类中：

```java
@Component
@ConfigurationProperties(prefix = "redis")
public class RedisCluster {
    private List<SingleRedisConfig> redisConfigs;
	//省略getter/setter
}
```

## 优缺点

不同于 properties 文件的无序，yaml 配置是有序的，这一点在有些配置中是非常有用的，例如在 Spring Cloud Zuul 的配置中，当我们配置代理规则时，顺序就显得尤为重要了。当然 yaml 配置也不是万能的，例如，yaml 配置目前不支持 @PropertySource 注解。

# 自定义 Spring Boot 中的 starter



## 1.核心知识

其实 Starter 的核心就是条件注解 `@Conditional` ，当 classpath 下存在某一个 Class 时，某个配置才会生效，前面松哥已经带大家学习过不少 Spring Boot 中的知识点，有的也涉及到源码解读，大伙可能也发现了源码解读时总是会出现条件注解，其实这就是 Starter 配置的核心之一，大伙有兴趣可以翻翻历史记录，看看松哥之前写的关于 Spring Boot 的文章，这里我就不再重复介绍了。

## 2.定义自己的 Starter

### 2.1定义

所谓的 Starter ，其实就是一个普通的 Maven 项目，因此我们自定义 Starter ，需要首先创建一个普通的 Maven 项目，创建完成后，添加 Starter 的自动化配置类即可，如下：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-autoconfigure</artifactId>
    <version>2.1.8.RELEASE</version>
</dependency>
```

配置完成后，我们首先创建一个 HelloProperties 类，用来接受 application.properties 中注入的值，如下：

```java
@ConfigurationProperties(prefix = "javaboy")
public class HelloProperties {
    private static final String DEFAULT_NAME = "江南一点雨";
    private static final String DEFAULT_MSG = "牧码小子";
    private String name = DEFAULT_NAME;
    private String msg = DEFAULT_MSG;
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String getMsg() {
        return msg;
    }
    public void setMsg(String msg) {
        this.msg = msg;
    }
}
```

这个配置类很好理解，将 application.properties 中配置的属性值直接注入到这个实例中， `@ConfigurationProperties` 类型安全的属性注入，即将 application.properties 文件中前缀为 javaboy 的属性注入到这个类对应的属性上， 最后使用时候，application.properties 中的配置文件，大概如下：

```properties
javaboy.name=zhangsan
javaboy.msg=java
```

关注类型安全的属性注入，读者可以参考松哥之前的这篇文章：[Spring Boot中的yaml配置简介](https://mp.weixin.qq.com/s/dbSBzFICIDPLkj5Tuv2-yA)，这篇文章虽然是讲 yaml 配置，但是关于类型安全的属性注入和 properties 是一样的。

配置完成 HelloProperties 后，接下来我们来定义一个 HelloService ，然后定义一个简单的 say 方法， HelloService 的定义如下：

```java
public class HelloService {
    private String msg;
    private String name;
    public String sayHello() {
        return name + " say " + msg + " !";
    }
    public String getMsg() {
        return msg;
    }
    public void setMsg(String msg) {
        this.msg = msg;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

这个很简单，没啥好说的。

接下来就是我们的重轴戏，自动配置类的定义，用了很多别人定义的自定义类之后，我们也来自己定义一个自定义类。先来看代码吧，一会松哥再慢慢解释：

```java
@Configuration
@EnableConfigurationProperties(HelloProperties.class)
@ConditionalOnClass(HelloService.class)
public class HelloServiceAutoConfiguration {
    @Autowired
    HelloProperties helloProperties;

    @Bean
    HelloService helloService() {
        HelloService helloService = new HelloService();
        helloService.setName(helloProperties.getName());
        helloService.setMsg(helloProperties.getMsg());
        return helloService;
    }
}
```

关于这一段自动配置，解释如下：

- 首先 @Configuration 注解表明这是一个配置类。
- @EnableConfigurationProperties 注解是使我们之前配置的 @ConfigurationProperties 生效，让配置的属性成功的进入 Bean 中。
- @ConditionalOnClass 表示当项目当前 classpath 下存在 HelloService 时，后面的配置才生效。
- 自动配置类中首先注入 HelloProperties ，这个实例中含有我们在 application.properties 中配置的相关数据。
- 提供一个 HelloService 的实例，将 HelloProperties 中的值注入进去。

做完这一步之后，我们的自动化配置类就算是完成了，接下来还需要一个 spring.factories 文件，那么这个文件是干嘛的呢？大家知道我们的 Spring Boot 项目的启动类都有一个 @SpringBootApplication 注解，这个注解的定义如下：

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM,
				classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
}
```

大家看到这是一个组合注解，其中的一个组合项就是 @EnableAutoConfiguration ，这个注解是干嘛的呢？

@EnableAutoConfiguration 表示启用 Spring 应用程序上下文的自动配置，该注解会自动导入一个名为 AutoConfigurationImportSelector 的类,而这个类会去读取一个名为 spring.factories 的文件, spring.factories 中则定义需要加载的自动化配置类，我们打开任意一个框架的 Starter ，都能看到它有一个 spring.factories 文件，例如 MyBatis 的 Starter 如下：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/6-1.png)](http://www.javaboy.org/images/boot2/6-1.png)

那么我们自定义 Starter 当然也需要这样一个文件，我们首先在 Maven 项目的 resources 目录下创建一个名为 META-INF 的文件夹，然后在文件夹中创建一个名为 spring.factories 的文件，文件内容如下：

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=org.javaboy.mystarter.HelloServiceAutoConfiguration
```

在这里指定我们的自动化配置类的路径即可。

如此之后我们的自动化配置类就算完成了。

### 2.2本地安装

如果在公司里，大伙可能需要将刚刚写好的自动化配置类打包，然后上传到 Maven 私服上，供其他同事下载使用，我这里就简单一些，我就不上传私服了，我将这个自动化配置类安装到本地仓库，然后在其他项目中使用即可。安装方式很简单，在 IntelliJ IDEA 中，点击右边的 Maven Project ，然后选择 Lifecycle 中的 install ，双击即可，如下：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/6-2.png)](http://www.javaboy.org/images/boot2/6-2.png)

双击完成后，这个 Starter 就安装到我们本地仓库了，当然小伙伴也可以使用 Maven 命令去安装。

## 3.使用 Starter

接下来，我们来新建一个普通的 Spring Boot 工程，这个 Spring Boot 创建成功之后，加入我们自定义 Starter 的依赖，如下：

```xml
<dependency>
    <groupId>org.javaboy</groupId>
    <artifactId>mystarter</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

此时我们引入了上面自定义的 Starter ，也即我们项目中现在有一个默认的 HelloService 实例可以使用，而且关于这个实例的数据，我们还可以在 application.properties 中进行配置，如下：

```properties
javaboy.name=牧码小子
javaboy.msg=java
```

配置完成后，方便起见，我这里直接在单元测试方法中注入 HelloSerivce 实例来使用，代码如下：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class UsemystarterApplicationTests {

    @Autowired
    HelloService helloService;
    @Test
    public void contextLoads() {
        System.out.println(helloService.sayHello());
    }
}
```

执行单元测试方法，打印日志如下：

[![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/6-3.png)](http://www.javaboy.org/images/boot2/6-3.png)

好了，一个简单的自动化配置类我们就算完成了，是不是很简单！

# 理解自动化配置的原理

Spring Boot 中的自动化配置确实够吸引人，甚至有人说 Spring Boot 让 Java 又一次焕发了生机，这话虽然听着有点夸张，但是不可否认的是，曾经臃肿繁琐的 Spring 配置确实让人感到头大，而 Spring Boot 带来的全新自动化配置，又确实缓解了这个问题。



你要是问这个自动化配置是怎么实现的，很多人会说不就是 starter 嘛！那么 starter 的原理又是什么呢？松哥以前写过一篇文章，介绍了自定义 starter：

- [徒手撸一个 Spring Boot 中的 Starter ，解密自动化配置黑魔法！](https://mp.weixin.qq.com/s/tKr_shLQnvcQADr4mvcU3A)

这里边有一个非常关键的点，那就是**条件注解**，甚至可以说条件注解是整个 Spring Boot 的基石。

条件注解并非一个新事物，这是一个存在于 Spring 中的东西，我们在 Spring 中常用的 profile 实际上就是条件注解的一个特殊化。

想要把 Spring Boot 的原理搞清，条件注解必须要会用，因此今天松哥就来和大家聊一聊条件注解。

## 定义

Spring4 中提供了更加通用的条件注解，让我们可以在满足不同条件时创建不同的 Bean，这种配置方式在 Spring Boot 中得到了广泛的使用，大量的自动化配置都是通过条件注解来实现的，查看松哥之前的 Spring Boot 文章，凡是涉及到源码解读的文章，基本上都离不开条件注解：

- [40 篇原创干货，带你进入 Spring Boot 殿堂！](https://mp.weixin.qq.com/s/tm1IqiEvRZwDAb-F5yJ5Aw)

有的小伙伴可能没用过条件注解，但是开发环境、生产环境切换的 Profile 多多少少都有用过吧？实际上这就是条件注解的一个特例。

## 实践

抛开 Spring Boot，我们来单纯的看看在 Spring 中条件注解的用法。

首先我们来创建一个普通的 Maven 项目，然后引入 spring-context，如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.1.5.RELEASE</version>
    </dependency>
</dependencies>
```

然后定义一个 Food 接口：

```java
public interface Food {
    String showName();
}
```

Food 接口有一个 showName 方法和两个实现类：

```java
public class Rice implements Food {
    public String showName() {
        return "米饭";
    }
}
public class Noodles implements Food {
    public String showName() {
        return "面条";
    }
}
```

分别是 Rice 和 Noodles 两个类，两个类实现了 showName 方法，然后分别返回不同值。

接下来再分别创建 Rice 和 Noodles 的条件类，如下：

```java
public class NoodlesCondition implements Condition {
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        return context.getEnvironment().getProperty("people").equals("北方人");
    }
}
public class RiceCondition implements Condition {
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        return context.getEnvironment().getProperty("people").equals("南方人");
    }
}
```

在 matches 方法中做条件属性判断，当系统属性中的 people 属性值为 ‘北方人’ 的时候，NoodlesCondition 的条件得到满足，当系统中 people 属性值为 ‘南方人’ 的时候，RiceCondition 的条件得到满足，换句话说，哪个条件得到满足，一会就会创建哪个 Bean 。

接下来我们来配置 Rice 和 Noodles ：

```java
@Configuration
public class JavaConfig {
    @Bean("food")
    @Conditional(RiceCondition.class)
    Food rice() {
        return new Rice();
    }
    @Bean("food")
    @Conditional(NoodlesCondition.class)
    Food noodles() {
        return new Noodles();
    }
}
```

这个配置类，大家重点注意两个地方：

- 两个 Bean 的名字都为 food，这不是巧合，而是有意取的。两个 Bean 的返回值都为其父类对象 Food。
- 每个 Bean 上都多了 @Conditional 注解，当 @Conditional 注解中配置的条件类的 matches 方法返回值为 true 时，对应的 Bean 就会生效。

配置完成后，我们就可以在 main 方法中进行测试了：

```java
public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
        ctx.getEnvironment().getSystemProperties().put("people", "南方人");
        ctx.register(JavaConfig.class);
        ctx.refresh();
        Food food = (Food) ctx.getBean("food");
        System.out.println(food.showName());
    }
}
```

首先我们创建一个 AnnotationConfigApplicationContext 实例用来加载 Java 配置类，然后我们添加一个 property 到 environment 中，添加完成后，再去注册我们的配置类，然后刷新容器。容器刷新完成后，我们就可以从容器中去获取 food 的实例了，这个实例会根据 people 属性的不同，而创建出来不同的 Food 实例。

这个就是 Spring 中的条件注解。

## 进化

条件注解还有一个进化版，那就是 Profile。我们一般利用 Profile 来实现在开发环境和生产环境之间进行快速切换。其实 Profile 就是利用条件注解来实现的。

还是刚才的例子，我们用 Profile 来稍微改造一下：

首先 Food、Rice 以及 Noodles 的定义不用变，条件注解这次我们不需要了，我们直接在 Bean 定义时添加 @Profile 注解，如下：

```java
@Configuration
public class JavaConfig {
    @Bean("food")
    @Profile("南方人")
    Food rice() {
        return new Rice();
    }
    @Bean("food")
    @Profile("北方人")
    Food noodles() {
        return new Noodles();
    }
}
```

这次不需要条件注解了，取而代之的是 @Profile 。然后在 Main 方法中，按照如下方式加载 Bean：

```java
public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
        ctx.getEnvironment().setActiveProfiles("南方人");
        ctx.register(JavaConfig.class);
        ctx.refresh();
        Food food = (Food) ctx.getBean("food");
        System.out.println(food.showName());
    }
}
```

效果和上面的案例一样。

这样看起来 @Profile 注解貌似比 @Conditional 注解还要方便，那么 @Profile 注解到底是什么实现的呢？

我们来看一下 @Profile 的定义：

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(ProfileCondition.class)
public @interface Profile {
	String[] value();
}
```

可以看到，它也是通过条件注解来实现的。条件类是 ProfileCondition ，我们来看看：

```java
class ProfileCondition implements Condition {
	@Override
	public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
		MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
		if (attrs != null) {
			for (Object value : attrs.get("value")) {
				if (context.getEnvironment().acceptsProfiles(Profiles.of((String[]) value))) {
					return true;
				}
			}
			return false;
		}
		return true;
	}
}
```

看到这里就明白了，其实还是我们在条件注解中写的那一套东西，只不过 @Profile 注解自动帮我们实现了而已。

@Profile 虽然方便，但是不够灵活，因为具体的判断逻辑不是我们自己实现的。而 @Conditional 则比较灵活。

## 结语

两个例子向大家展示了条件注解在 Spring 中的使用，它的一个核心思想就是当满足某种条件的时候，某个 Bean 才会生效，而正是这一特性，支撑起了 Spring Boot 的自动化配置。