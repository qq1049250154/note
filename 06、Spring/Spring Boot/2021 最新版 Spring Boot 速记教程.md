大家好，我是必读哥。由于微信变更了推送规则，有些小伙伴反映有时候没收到推文，但其实后台已经推送了。为了确保大家能在第一时间收到推送通知，请务必将本公众号设置为“星标”，谢谢大家了。

------

本文来源：http://r6d.cn/X6FP

结束了前面的《Spring 源码深度学习》，八月给自己放松了一下，看了几本小说和电视剧，还有写一个工作中用到的小工具，周报数据渲染的前端界面（前端是真的难）。

当然技术上的学习也要注意，所以看了**松哥写的《Spring Boot + Vue 全栈开发》**，来系统学习 `SpringBoot`，下面是简单的速记，根据使用场景可以快速定位到知识点：

`Demo` 脚手架项目地址：

https://github.com/Vip-Augus/springboot-note

**Table of Contents** *generated with DocToc*

- SpringBoot 速记

- - 一、引入依赖
  - 二、配置 Swagger 参数
  - 一、引入依赖
  - 二、配置邮箱的参数
  - 三、写模板和发送内容
  - 一、引用 Redis 依赖
  - 二、参数配置
  - 三、代码使用
  - 一、添加 mybatis 和 druid 依赖
  - 二、配置数据库和连接池参数
  - 三、其他 mybatis 配置
  - @ExceptionHandler 错误处理
  - @ModelAttribute 视图属性
  - 常规配置
  - HTTPS 配置
  - 构建项目
  - SpringBoot 基础配置
  - Spring Boot Starters
  - @SpringBootApplication
  - Web 容器配置
  - @ConfigurationProperties
  - Profile
  - @ControllerAdvice 用来处理全局数据
  - CORS 支持，跨域资源共享
  - 注册 MVC 拦截器
  - 开启 AOP 切面控制
  - 整合 Mybatis 和 Druid
  - 整合 Redis
  - 发送 HTML 样式的邮件
  - 整合 Swagger (API 文档)
  - 总结
  - 参考资料

## 构建项目

相比于使用 `IDEA` 的模板创建项目，我更推荐的是在 `Spring` 官网上选择参数一步生成项目。

https://start.spring.io/

关于 IDEA 发布过很多文字，可以关注微信公众号 Java后端，关注后输入 666 命令下载 Spring Boot 和 IDEA 相关文字的 PDF。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

我们只需要做的事情，就是修改组织名和项目名，点击 `Generate the project`，下载到本地，然后使用 `IDEA` 打开

这个时候，不需要任何配置，点击 `Application` 类的 `run` 方法就能直接启动项目。

------

## SpringBoot 基础配置

## Spring Boot Starters

**引用自参考资料 1 描述：**

> “
>
> **starter的理念**：starter 会把所有用到的依赖都给包含进来，避免了开发者自己去引入依赖所带来的麻烦。需要注意的是不同的 starter 是为了解决不同的依赖，所以它们内部的实现可能会有很大的差异，例如 jpa 的 starter 和 Redis 的 starter 可能实现就不一样，这是因为 starter 的本质在于 synthesize，这是一层在逻辑层面的抽象，也许这种理念有点类似于 Docker，因为它们都是在做一个“包装”的操作，如果你知道 Docker 是为了解决什么问题的，也许你可以用 Docker 和 starter 做一个类比。
>
> ”

我们知道在 `SpringBoot` 中很重要的一个概念就是，**「约定优于配置」**，通过特定方式的配置，可以减少很多步骤来实现想要的功能。

例如如果我们想要使用缓存 `Redis`

**在之前的可能需要通过以下几个步骤：**

1. 在 `pom` 文件引入特定版本的 `redis`
2. 在 `.properties` 文件中配置参数
3. 根据参数，新建一个又一个 `jedis` 连接
4. 定义一个工具类，手动创建连接池来管理

经历了上面的步骤，我们才能正式使用 `Redis`

**但在 `Spring Boot` 中，一切因为 `Starter` 变得简单**

1. 在 `pom` 文件中引入 `spring-boot-starter-data-redis`
2. 在 `.properties` 文件中配置参数

通过上面两个步骤，配置自动生效，具体生效的 `bean` 是 `RedisAutoConfiguration`，自动配置类的名字都有一个特点，叫做 `xxxAutoConfiguration`。

可以来简单看下这个类：

```
@Configuration
@ConditionalOnClass(RedisOperations.class)
@EnableConfigurationProperties(RedisProperties.class)
@Import({ LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class })
public class RedisAutoConfiguration {

 @Bean
 @ConditionalOnMissingBean(name = "redisTemplate")
 public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory)
   throws UnknownHostException {
  RedisTemplate<Object, Object> template = new RedisTemplate<>();
  template.setConnectionFactory(redisConnectionFactory);
  return template;
 }

 @Bean
 @ConditionalOnMissingBean
 public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory)
   throws UnknownHostException {
  StringRedisTemplate template = new StringRedisTemplate();
  template.setConnectionFactory(redisConnectionFactory);
  return template;
 }

}

@ConfigurationProperties(prefix = "spring.redis")
public class RedisProperties {...}
```

可以看到，`Redis` 自动配置类，读取了以 `spring.redis` 为前缀的配置，然后加载 `redisTemplate` 到容器中，然后我们在应用中就能使用 `RedisTemplate` 来对缓存进行操作~（还有很多细节没有细说，例如 `@ConditionalOnMissingBean` 先留个坑(●´∀｀●)ﾉ）

```
@Autowired
private RedisTemplate redisTemplate;

ValueOperations ops2 = redisTemplate.opsForValue();
Book book = (Book) ops2.get("b1");
```

------

## @SpringBootApplication

该注解是加载项目的启动类上的，而且它是一个组合注解：

```
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
  @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {...}
```

下面是这三个核心注解的解释：

| 注解名                   | 解释                                                  |
| :----------------------- | :---------------------------------------------------- |
| @SpringBootConfiguration | **表明这是一个配置类，开发者可以在这个类中配置 Bean** |
| @EnableAutoConfiguration | **表示开启自动化配置**                                |
| @ComponentScan           | **完成包扫描，默认扫描的类位于当前类所在包的下面**    |

通过该注解，我们执行 `mian` 方法：

```
SpringApplication.run(SpringBootLearnApplication.class, args);
```

就可以启动一个 `SpringApplicaiton` 应用了。

------

## Web 容器配置

### 常规配置

| 配置名                             | 解释                                                         |
| :--------------------------------- | :----------------------------------------------------------- |
| server.port=8081                   | **配置了容器的端口号，默认是 8080**                          |
| server.error.path=/error           | **配置了项目出错时跳转的页面**                               |
| server.servlet.session.timeout=30m | **session 失效时间，m 表示分钟，如果不写单位，默认是秒 s**   |
| server.servlet.context-path=/      | **项目名称，不配置时默认为/。配置后，访问时需加上前缀**      |
| server.tomcat.uri-encoding=utf-8   | **Tomcat 请求编码格式**                                      |
| server.tomcat.max-threads=500      | **Tomcat 最大线程数**                                        |
| server.tomcat.basedir=/home/tmp    | **Tomcat 运行日志和临时文件的目录，如不配置，默认使用系统的临时目录** |

### HTTPS 配置

| 配置名                               | 解释           |
| :----------------------------------- | :------------- |
| server.ssl.key-store=xxx             | **秘钥文件名** |
| server.ssl.key-alias=xxx             | **秘钥别名**   |
| server.ssl.key-store-password=123456 | **秘钥密码**   |

想要详细了解如何配置 `HTTPS`，可以参考这篇文章 Spring Boot 使用SSL-HTTPS

------

## @ConfigurationProperties

这个注解可以放在类上或者 `@Bean` 注解所在方法上，这样 `SpringBoot` 就能够从配置文件中，读取特定前缀的配置，将属性值注入到对应的属性。

使用例子：

```
@Configuration
@ConfigurationProperties(prefix = "spring.datasource")
public class DruidConfigBean {

    private Integer initialSize;

    private Integer minIdle;

    private Integer maxActive;
    
    private List<String> customs;
    
    ...
}
application.properties
spring.datasource.initialSize=5
spring.datasource.minIdle=5
spring.datasource.maxActive=20
spring.datasource.customs=test1,test2,test3
```

其中，如果对象是列表结构，可以在配置文件中使用 , 逗号进行分割，然后注入到相应的属性中。

------

## Profile

使用该属性，可以快速切换配置文件，在 `SpringBoot` 默认约定中，不同环境下配置文件名称规则为 `application-{profile}.propertie`，`profile` 占位符表示当前环境的名称。

1、配置 `application.properties`

```
spring.profiles.active=dev
```

2、在代码中配置 在启动类的 `main` 方法上添加 `setAdditionalProfiles("{profile}")`;

```
SpringApplicationBuilder builder = new SpringApplicationBuilder(SpringBootLearnApplication.class);
builder.application().setAdditionalProfiles("prod");
builder.run(args);
```

3、启动参数配置

```
java -jar demo.jar --spring.active.profile=dev
```

------

## @ControllerAdvice 用来处理全局数据

`@ControllerAdvice` 是 `@Controller` 的增强版。主要用来处理全局数据，一般搭配 `@ExceptionHandler` 、`@ModelAttribute` 以及 `@InitBinder` 使用。

### @ExceptionHandler 错误处理

```
/**
 * 加强版控制器，拦截自定义的异常处理
 *
 */
@ControllerAdvice
public class CustomExceptionHandler {
    
    // 指定全局拦截的异常类型，统一处理
    @ExceptionHandler(MaxUploadSizeExceededException.class)
    public void uploadException(MaxUploadSizeExceededException e, HttpServletResponse response) throws IOException {
        response.setContentType("text/html;charset=utf-8");
        PrintWriter out = response.getWriter();
        out.write("上传文件大小超出限制");
        out.flush();
        out.close();
    }
}
```

### @ModelAttribute 视图属性

```
@ControllerAdvice
public class CustomModelAttribute {
    
    // 
    @ModelAttribute(value = "info")
    public Map<String, String> userInfo() throws IOException {
        Map<String, String> map = new HashMap<>();
        map.put("test", "testInfo");
        return map;
    }
}


@GetMapping("/hello")
public String hello(Model model) {
    Map<String, Object> map = model.asMap();
    Map<String, String> infoMap = (Map<String, String>) map.get("info");
    return infoMap.get("test");
}
```

- `key : @ModelAttribute` 注解中的 `value` 属性
- 使用场景：任何请求 `controller` 类，通过方法参数中的 `Model` 都可以获取 `value`对应的属性
- 关注公众号Java后端编程，回复 Java 获取最新学习资料 。

------

## CORS 支持，跨域资源共享

`CORS（Cross-Origin Resource Sharing）`，跨域资源共享技术，目的是为了解决前端的跨域请求。

> “
>
> **引用：当一个资源从与该资源本身所在服务器不同的域或端口请求一个资源时，资源会发起一个跨域HTTP请求**
>
> ”

详细可以参考这篇文章-springboot系列文章之实现跨域请求(CORS)，这里只是记录一下如何使用：

例如在我的前后端分离 `demo` 中，如果没有通过 `Nginx` 转发，那么将会提示如下信息：

> “
>
> Access to fetch at ‘http://localhost:8888/login‘ from origin ‘http://localhost:3000‘ has been blocked by CORS policy: Response to preflight request doesn’t pass access control check: The value of the ‘Access-Control-Allow-Credentials’ header in the response is ‘’ which must be ‘true’ when the request’s credentials mode is ‘include’
>
> ”

为了解决这个问题，在前端不修改的情况下，需要后端加上如下两行代码：

```
// 第一行，支持的域
@CrossOrigin(origins = "http://localhost:3000")
@RequestMapping(value = "/login", method = RequestMethod.GET)
@ResponseBody
public String login(HttpServletResponse response) {
    // 第二行，响应体添加头信息（这一行是解决上面的提示）
    response.setHeader("Access-Control-Allow-Credentials", "true");
    return HttpRequestUtils.login();
}
```

------

## 注册 MVC 拦截器

在 `MVC` 模块中，也提供了类似 `AOP` 切面管理的扩展，能够拥有更加精细的拦截处理能力。

核心在于该接口：`HandlerInterceptor`，使用方式如下：

```
/**
 * 自定义 MVC 拦截器
 */
public class MyInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // 在 controller 方法之前调用
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        // 在 controller 方法之后调用
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        // 在 postHandle 方法之后调用
    }
}
```

注册代码：

```
/**
 * 全局控制的 mvc 配置
 */
@Configuration
public class MyWebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new MyInterceptor())
                // 表示拦截的 URL
                .addPathPatterns("/**")
                // 表示需要排除的路径
                .excludePathPatterns("/hello");
    }
}
```

拦截器执行顺序：`preHandle` -> `controller` -> `postHandle` -> `afterCompletion`，同时需要注意的是，只有 `preHandle` 方法返回 `true`，后面的方法才会继续执行。

------

## 开启 AOP 切面控制

切面注入是老生常谈的技术，之前学习 `Spring` 时也有了解，可以参考我之前写过的文章参考一下：

Spring自定义注解实现AOP

Spring 源码学习(八) AOP 使用和实现原理

在 `SpringBoot` 中，使用起来更加简便，只需要加入该依赖，使用方法与上面一样。

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

------

## 整合 Mybatis 和 Druid

`SpringBoot` 整合数据库操作，目前主流使用的是 `Druid` 连接池和 `Mybatis` 持久层，同样的，`starter` 提供了简洁的整合方案

项目结构如下：

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

### 一、添加 mybatis 和 druid 依赖

```
 <dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.2</version>
</dependency>

<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.18</version>
</dependency>
```

### 二、配置数据库和连接池参数

```
# 数据库配置
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=utf8&allowMultiQueries=true
spring.datasource.username=root
spring.datasource.password=12345678

# druid 配置
spring.datasource.druid.initial-size=5
spring.datasource.druid.max-active=20
spring.datasource.druid.min-idle=5
spring.datasource.druid.max-wait=60000
spring.datasource.druid.pool-prepared-statements=true
spring.datasource.druid.max-pool-prepared-statement-per-connection-size=20
spring.datasource.druid.max-open-prepared-statements=20
spring.datasource.druid.validation-query=SELECT 1
spring.datasource.druid.validation-query-timeout=30000
spring.datasource.druid.test-on-borrow=true
spring.datasource.druid.test-on-return=false
spring.datasource.druid.test-while-idle=false
#spring.datasource.druid.time-between-eviction-runs-millis=
#spring.datasource.druid.min-evictable-idle-time-millis=
#spring.datasource.druid.max-evictable-idle-time-millis=10000

# Druid stat filter config
spring.datasource.druid.filters=stat,wall
spring.datasource.druid.web-stat-filter.enabled=true
spring.datasource.druid.web-stat-filter.url-pattern=/*
# session 监控
spring.datasource.druid.web-stat-filter.session-stat-enable=true
spring.datasource.druid.web-stat-filter.session-stat-max-count=10
spring.datasource.druid.web-stat-filter.principal-session-name=admin
spring.datasource.druid.web-stat-filter.principal-cookie-name=admin
spring.datasource.druid.web-stat-filter.profile-enable=true
# stat 监控
spring.datasource.druid.web-stat-filter.exclusions=*.js,*.gif,*.jpg,*.bmp,*.png,*.css,*.ico,/druid/*
spring.datasource.druid.filter.stat.db-type=mysql
spring.datasource.druid.filter.stat.log-slow-sql=true
spring.datasource.druid.filter.stat.slow-sql-millis=1000
spring.datasource.druid.filter.stat.merge-sql=true
spring.datasource.druid.filter.wall.enabled=true
spring.datasource.druid.filter.wall.db-type=mysql
spring.datasource.druid.filter.wall.config.delete-allow=true
spring.datasource.druid.filter.wall.config.drop-table-allow=false

# Druid manage page config
spring.datasource.druid.stat-view-servlet.enabled=true
spring.datasource.druid.stat-view-servlet.url-pattern=/druid/*
spring.datasource.druid.stat-view-servlet.reset-enable=true
spring.datasource.druid.stat-view-servlet.login-username=admin
spring.datasource.druid.stat-view-servlet.login-password=admin
#spring.datasource.druid.stat-view-servlet.allow=
#spring.datasource.druid.stat-view-servlet.deny=
spring.datasource.druid.aop-patterns=cn.sevenyuan.demo.*
```

### 三、其他 mybatis 配置

本地工程，将 `xml` 文件放入 `resources` 资源文件夹下，所以需要加入以下配置，让应用进行识别：

```
 <build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*</include>
            </includes>
        </resource>
    </resources>
    ...
</build>
```

通过上面的配置，我本地开启了三个页面的监控：`SQL` 、 `URL` 和 `Sprint` 监控，如下图：

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**通过上面的配置，SpringBoot 很方便的就整合了 Druid 和 mybatis，同时根据在 `properties` 文件中配置的参数，开启了 `Druid` 的监控。**

但我根据上面的配置，始终开启不了 `session` 监控，所以如果需要配置 `session` 监控或者调整参数具体配置，可以查看官方网站



------

## 整合 Redis

我用过 `Redis` 和 `NoSQL`，但最熟悉和常用的，还是 `Redis` ，所以这里记录一下如何整合

### 一、引用 Redis 依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <exclusions>
        <exclusion>
            <artifactId>lettuce-core</artifactId>
            <groupId>io.lettuce</groupId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
```

### 二、参数配置

```
# redis 配置
spring.redis.database=0
spring.redis.host=localhost
spring.redis.port=6379
spring.redis.password=
spring.redis.jedis.pool.max-active=8
spring.redis.jedis.pool.max-idle=8
spring.redis.jedis.pool.max-wait=-1ms
spring.redis.jedis.pool.min-idle=0
```

### 三、代码使用

```
@Autowired
private RedisTemplate redisTemplate;

@Autowired
private StringRedisTemplate stringRedisTemplate;

@GetMapping("/testRedis")
public Book getForRedis() {
    ValueOperations<String, String> ops1 = stringRedisTemplate.opsForValue();
    ops1.set("name", "Go 语言实战");
    String name = ops1.get("name");
    System.out.println(name);
    ValueOperations ops2 = redisTemplate.opsForValue();
    Book book = (Book) ops2.get("b1");
    if (book == null) {
        book = new Book("Go 语言实战", 2, "none name", BigDecimal.ONE);
        ops2.set("b1", book);
    }
    return book;
}
```

这里只是简单记录引用和使用方式，更多功能可以看这里：

------

## 发送 HTML 样式的邮件

之前也使用过，所以可以参考这篇文章：

### 一、引入依赖

```
 <!-- mail -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

### 二、配置邮箱的参数

```
# mail
spring.mail.host=smtp.qq.com
spring.mail.port=465
spring.mail.username=xxxxxxxx
spring.mail.password=xxxxxxxx
spring.mail.default-encoding=UTF-8
spring.mail.properties.mail.smtp.socketFactory.class=javax.net.ssl.SSLSocketFactory
spring.mail.properties.mail.debug=true
```

如果使用的是 `QQ` 邮箱，需要在邮箱的设置中获取授权码，填入上面的 `password` 中

### 三、写模板和发送内容

```
MailServiceImpl.java
@Autowired
private JavaMailSender javaMailSender;

@Override
public void sendHtmlMail(String from, String to, String subject, String content) {
    try {
        MimeMessage message = javaMailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(message, true);
        helper.setFrom(from);
        helper.setTo(to);
        helper.setSubject(subject);
        helper.setText(content, true);
        javaMailSender.send(message);
    } catch (MessagingException e) {
        System.out.println("发送邮件失败");
        log.error("发送邮件失败", e);
    }
}
mailtemplate.html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>邮件</title>
</head>
<body>
<div th:text="${subject}"></div>
<div>书籍清单
    <table border="1">
        <tr>
            <td>图书编号</td>
            <td>图书名称</td>
            <td>图书作者</td>
        </tr>
        <tr th:each="book:${books}">
            <td th:text="${book.id}"></td>
            <td th:text="${book.name}"></td>
            <td th:text="${book.author}"></td>
        </tr>
    </table>
</div>
</body>
</html>
test.java
@Autowired
private MailService mailService;

@Autowired
private TemplateEngine templateEngine;
    
@Test
public void sendThymeleafMail() {
    Context context = new Context();
    context.setVariable("subject", "图书清册");
    List<Book> books = Lists.newArrayList();
    books.add(new Book("Go 语言基础", 1, "nonename", BigDecimal.TEN));
    books.add(new Book("Go 语言实战", 2, "nonename", BigDecimal.TEN));
    books.add(new Book("Go 语言进阶", 3, "nonename", BigDecimal.TEN));
    context.setVariable("books", books);
    String mail = templateEngine.process("mailtemplate.html", context);
    mailService.sendHtmlMail("xxxx@qq.com", "xxxxxxxx@qq.com", "图书清册", mail);
}
```

通过上面简单步骤，就能够在代码中发送邮件，例如我们每周要写周报，统计系统运行状态，可以设定定时任务，统计数据，然后自动化发送邮件。

------

## 整合 Swagger (API 文档)

### 一、引入依赖

```
<!-- swagger -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>

<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```

### 二、配置 Swagger 参数

```
SwaggerConfig.java
@Configuration
@EnableSwagger2
@EnableWebMvc
public class SwaggerConfig {

    @Bean
    Docket docket() {
        return new Docket(DocumentationType.SWAGGER_2)
                .select()
                .apis(RequestHandlerSelectors.basePackage("cn.sevenyuan.demo.controller"))
                .paths(PathSelectors.any())
                .build().apiInfo(
                        new ApiInfoBuilder()
                                .description("Spring Boot learn project")
                                .contact(new Contact("JingQ", "https://github.com/vip-augus", "=-=@qq.com"))
                                .version("v1.0")
                                .title("API 测试文档")
                                .license("Apache2.0")
                                .licenseUrl("http://www.apache.org/licenese/LICENSE-2.0")
                                .build());

    }
}
```

设置页面 `UI`

```
@Configuration
@EnableRedisHttpSession(maxInactiveIntervalInSeconds = 60)
public class MyWebMvcConfig implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("swagger-ui.html")
                .addResourceLocations("classpath:/META-INF/resources/");

        registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/");
    }
}
```

通过这样就能够识别 `@ApiOperation` 等接口标志，在网页查看 `API` 文档，参考文档：Spring Boot实战：集成Swagger2

------

## 总结

这边总结的整合经验，只是很基础的配置，在学习的初期，秉着先跑起来，然后不断完善和精进学习。

而且单一整合很容易，但多个依赖会出现想不到的错误，所以在解决环境问题时遇到很多坑，想要使用基础的脚手架，可以尝试跑我上传的项目。

数据库脚本在 `resources` 目录的 `test.sql` 文件中

------

## 参考资料

1、Spring Boot Starters

2、Spring Boot 使用SSL-HTTPS

3、Spring Boot（07）——ConfigurationProperties介绍

4、springboot系列文章之实现跨域请求(CORS)

5、Spring Data Redis（一）–解析RedisTemplate

6、Spring Boot实战：集成Swagger2

```
PS：欢迎在留言区留下你的观点，一起讨论提高。如果今天的文章让你有新的启发，欢迎转发分享给更多人。
```