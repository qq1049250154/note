在使用 SpringCloud 时，远程服务都是以 HTTP 接口形式对外提供服务，因此服务消费者在调用服务时，需要使用 HTTP Client 方式访问。在通常进行远程 HTTP 调用时，可以使用 RestTemplate、HttpClient、URLConnection、OkHttp 等，也可以使用 SpringCloud Feign 进行远程调用

# RestTemplate

***源码：https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-feign/spring-cloud-feign-simple\***

## 脱离 Eureka 的使用

在脱离 Eureka 使用 RestTemplate 调用远程接口时，只需要引入 web 依赖即可。

在使用 RestTemplate 时，需要先将 RestTemplate 交给 Spring 管理



```java
@Configuration
public class RestTemplateConfiguration {

    @Bean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }

}
```

编写一个 Controller，注入 RestTemplate，调用远程接口



```java
@RestController
public class RestTemplateController {

    private final RestTemplate restTemplate;

    @Autowired
    public RestTemplateController(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    @GetMapping(value = "rest-get", produces = "text/html;charset=utf-8")
    public String restTemplateGet(){
        return restTemplate.getForObject("https://gitee.com", String.class);
    }

}
```

访问 http://localhost:8080/rest-get

![img](https:////upload-images.jianshu.io/upload_images/13856126-4058f1f7a0468404.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

RestTemplate Simple

## 关联 Eureka 使用

将服务注册到 Eureka Server，并使用 RestTemplate 调用远程 Eureka Client 服务

此时，只需要按照一个标准的 Eureka Client 编写步骤，将项目改造成一个 Eureka Client，并编写另外一个 Client。将要使用 RestTemplate 的 Client 当做服务消费者，另外一个当做服务提供者。在进行远程调用时，只需要将 `getForObject` 的 url，改为 http://service-id 即可，具体传入参数使用 `?`、`&`、`=` 拼接即可。

在注册到 Eureka Server 后，进行 RestTemplate 远程调用时，service-id 会被 Eureka Client 解析为 Server 中注册的 ip、端口，以此进行远程调用。

## Rest Template

RestTemplate 提供了 11 个独立的方法，这 11 个方法对应了各种远程调用请求

|      方法名       | http 动作 |                             说明                             |
| :---------------: | :-------: | :----------------------------------------------------------: |
|  getForEntity()   |    GET    | 发送 GET 请求，返回的 ResponseEntity 包含了响应体所映射成的对象 |
|  getForObject()   |    GET    |         发送 GET 请求，返回的请求体将映射为一个对象          |
|  postForEntity()  |   POST    | 发送 POST 请求，返回包含一个对象的 ResponseEntity，这个对象是从响应体中映射得到的 |
|  postForObject()  |   POST    |         发送 POST 请求，返回根据响应体匹配形成的对象         |
| postForLocation() |   POST    |             发送 POST 请求，返回新创建资源的 URL             |
|       put()       |    PUT    |                      PUT 资源到指定 URL                      |
|     delete()      |  DELETE   |                发送 DELETE 请求，执行删除操作                |
| headForHeaders()  |   HEAD    |       发送 HEAD 请求，返回包含指定资源 URL 的 HTTP 头        |
| optionsFOrAllow() |  OPTIONS  |       发送 OPTIONS 请求，返回指定 URL 的 Allow 头信息        |
|     execute()     |           |               执行非响应 ResponseEntity 的请求               |
|    exchange()     |           |                执行响应 ResponseEntity 的请求                |

------

# Feign

使用 RestTemplate 进行远程调用，非常方便，但是也有一个致命的问题：硬编码。 在 RestTemplate 调用中，我们每个调用远程接口的方法，都将远程接口对应的 ip、端口，或 service-id 硬编码到了 URL 中，如果远程接口的 ip、端口、service-id 有修改的话，需要将所有的调用都修改一遍，这样难免会出现漏改、错改等问题，且代码不便于维护。为了解决这个问题，Netflix 推出了 Feign 来统一管理远程调用。

## 什么是 Feign

Feign 是一个声明式的 Web Service 客户端，只需要创建一个接口，并加上对应的 Feign Client 注解，即可进行远程调用。Feign 也支持编码器、解码器，Spring Cloud Open Feign 也对 Feign 进行了增强，支持了 SpringMVC 注解，可以像 SpringMVC 一样进行远程调用。

Feign 是一种声明式、模版化的 HTTP 客户端，在 Spring Cloud 中使用 Feign，可以做到使用 HTTP 请求访问远程方法就像调用本地方法一样简单，开发者完全感知不到是在进行远程调用。

Feign 的特性：

- 可插拔的注解支持
- 可插拔的 HTTP 编码器、解码器
- 支持 Hystrix  断路器、Fallback
- 支持 Ribbon 负载均衡
- 支持 HTTP 请求、响应压缩

## 简单示例

***源码：https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-feign/spring-cloud-feign-simple\***

使用 Feign 进行 github 接口调用

### pom 依赖



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
</dependencies>
```

### 配置文件

只是进行一个简单的远程调用，不需要注册 Eureka、不需要配置文件。

### 启动类



```java
@SpringBootApplication
@EnableFeignClients
public class SpringCloudFeignSimpleApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudFeignSimpleApplication.class, args);
    }

}
```

### Feign Client 配置



```java
@Configuration
public class GiteeFeignConfiguration {
    
     /**
     * 配置 Feign 日志级别
     * <p>
     * NONE：没有日志
     * BASIC：基本日志
     * HEADERS：header
     * FULL：全部
     * <p>
     * 配置为打印全部日志，可以更方便的查看 Feign 的调用信息
     *
     * @return Feign 日志级别
     */
    @Bean
    public Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
    
}
```

### FeignClient



```java
@FeignClient(name = "gitee-client", url = "https://www.gitee.com/", configuration = GiteeFeignConfiguration.class)
public interface GiteeFeignClient {

    @RequestMapping(value = "/search", method = RequestMethod.GET)
    String searchRepo(@RequestParam("q") String query);

}
```

`@FeignClient`：声明为一个 Feign 远程调用
 `name`：给远程调用起个名字
 `url`：指定要调用哪个 url
 `configuration`：指定配置信息

`@RequestMapping`：如同 SpringMVC 一样调用。

### Feign Controller



```java
@RestController
public class FeignController {

    private final GiteeFeignClient giteeFeignClient;

    @Autowired
    public FeignController(GiteeFeignClient giteeFeignClient) {
        this.giteeFeignClient = giteeFeignClient;
    }

    @GetMapping(value = "feign-gitee")
    public String feign(String query){
        return giteeFeignClient.searchRepo(query);
    }

}
```

### 验证调用结果

在浏览器中访问： http://localhost:8080/feign-gitee?query=spring-cloud-openfeign

![img](https:////upload-images.jianshu.io/upload_images/13856126-3c19c533d09dc0cb.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

Feign To Gitee

------

# @FeignClient、@RequestMapping

## 在 Feign 中使用 MVC 注解的注意事项

在 FeignClient 中使用 `@RequestMapping` 注解调用远程接口，需要注意：

- 注解必须为 `@RequestMapping`，不能为组合注解 `@GetMapping` 等，否则解析不到
- 必须指定 method，否则会出问题
- value 必须指定被调用方的 url，不能包含域名、ip 等

## 使用 @FeignClient 的注意事项

- 在启动类上必须加上 `@FeignClients` 注解，开启扫描
- 在 FeignClient 接口上必须指定 `@FeignClient` 注解，声明是一个 Feign 远程调用

## @FeignClient

源码



```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FeignClient {
    @AliasFor("name")
    String value() default "";

    /** @deprecated */
    @Deprecated
    String serviceId() default "";

    @AliasFor("value")
    String name() default "";

    String qualifier() default "";

    String url() default "";

    boolean decode404() default false;

    Class<?>[] configuration() default {};

    Class<?> fallback() default void.class;

    Class<?> fallbackFactory() default void.class;

    String path() default "";

    boolean primary() default true;
}
```

|     字段名      |                             含义                             |
| :-------------: | :----------------------------------------------------------: |
|      name       | 指定 FeignClient 的名称，如果使用到了 Eureka，且使用了 Ribbon 负载均衡，则 name 为被调用者的微服务名称，用于服务发现 |
|       url       |         一般用于调试，可以手动指定 feign 调用的地址          |
|    decode404    | 当 404 时，如果该字段为 true，会调用 decoder 进行解码，否则会抛出 FeignException |
|  configuration  | Feign 配置类，可以自定义 Feign 的 Encoder、Decoder、LogLevel、Contract 等 |
|    fallback     | 容错处理类，当远程调用失败、超时时，会调用对应接口的容错逻辑。Fallback 指定的类，必须实现 @FeignClient 标记的接口 |
| fallbackFactory | 工厂类，用于生成 fallback 类的示例，可以实现每个接口通用的容错逻辑，减少重复代码 |
|      path       |               定义当前 FeignClient 的统一前缀                |

------

# Feign 的运行原理

- 在启动类上加上 `@EnableFeignClients` 注解，开启对 Feign Client 扫描加载
- 在启用时，会进行包扫描，扫描所有的 `@FeignClient` 的注解的类，并将这些信息注入 Spring IOC 容器，当定义的 Feign 接口中的方法被调用时，通过 JDK 的代理方式，来生成具体的 `RestTemplate`。当生成代理时，Feign 会为每个接口方法创建一个 `RestTemplate` 对象，该对象封装了 HTTP 请求需要的全部信息，如：参数名、请求方法、header等
- 然后由 `RestTemplate` 生成 Request，然后把 Request 交给 Client 处理，这里指的 Client 可以是 JDK 原生的 `URLConnection`、Apache 的 `HTTP Client`、`OkHttp`。最后 Client 被封装到 LoadBalanceClient 类，结合 Ribbon 负载均衡发起服务间的调用。



# Feign GZIP 压缩

Feign 是通过 http 调用的，那么就牵扯到一个数据大小的问题。如果不经过压缩就发送请求、获取响应，那么会因为流量过大导致浪费流量，这时就需要使用数据压缩，将大流量压缩成小流量。

***源码：https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-feign/spring-cloud-feign-gzip\***

Spring Cloud Feign 支持对请求和响应进行 GZIP 压缩，以调高通信效率。

## 开启 gzip 压缩

application.yml



```yml
feign:
  compression:
    request:
      enabled: true
      mime-type: text/html,application/xml,application/json
      min-request-size: 2048
    response:
      enabled: true

# 开启日志
logging:
  level:
    com.laiyy.gitee.feign.springcloudfeigngzip.feign.GiteeFeignClient: debug
```

由于使用 gzip 压缩，压缩后的数据是二进制，那么在获取 Response 的时候，就不能和之前一样直接使用 String 来接收了，需要使用 ResponseEntity<byte[]> 接收



```java
@FeignClient(name = "gitee-client", url = "https://www.gitee.com/", configuration = GiteeFeignConfiguration.class)
public interface GiteeFeignClient {

    @RequestMapping(value = "/search", method = RequestMethod.GET)
    ResponseEntity<byte[]> searchRepo(@RequestParam("q") String query);

}
```

对应的 Controller 也需要改为 ResponseEntity<byte[]>



```java
@GetMapping(value = "feign-gitee")
public ResponseEntity<byte[]> feign(String query){
    return giteeFeignClient.searchRepo(query);
}
```

## 验证 gzip 压缩

开启 FeignClient 日志

### 没有使用 GZIP 压缩

在 `spring-cloud-feign-simple` 项目中，开启日志：



```yml
logging:
  level:
    com.laiyy.gitee.feign.springcloudfeignsimple.feign.GiteeFeignClient: debug
```

访问：http://localhost:8080/feign-gitee?query=spring-cloud-openfeign ，可以看到，在控制台中打印了日志信息：

![img](https:////upload-images.jianshu.io/upload_images/13856126-239dafd9a98c41d9.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

no-gzip

### 使用了 GZIP 压缩

在 `spring-cloud-feign-gzip` 中开启日志：



```yml
logging:
  level:
    com.laiyy.gitee.feign.springcloudfeigngzip.feign.GiteeFeignClient: debug
```

访问：http://localhost:8080/feign-gitee?query=spring-cloud-openfeign ，可以看到，在控制台中打印了日志信息：

![img](https:////upload-images.jianshu.io/upload_images/13856126-f6d8326ff23c9c39.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

gzip

## 对比

### Request 对比

经过对比，可以看到在没有开启 gzip 之前，request 是：



```rust
---> GET https://www.gitee.com/search?q=spring-cloud-openfeign HTTP/1.1
---> END HTTP (0-byte body)
```

开启 gzip 之后，request 是：



```rust
---> GET https://www.gitee.com/search?q=spring-cloud-openfeign HTTP/1.1
Accept-Encoding: gzip
Accept-Encoding: deflate
---> END HTTP (0-byte body)
```

可以看到，request 中增加了 `Accept-Encoding: gzip`，证明 request 开启了 gzip 压缩。

### Response 对比

在没有开启 gzip 之前，response 是：



```xml
cache-control: no-cache
connection: keep-alive
content-type: text/html; charset=utf-8
date: Wed, 23 Jan 2019 07:22:45 GMT
expires: Sun, 1 Jan 2000 01:00:00 GMT
pragma: must-revalidate, no-cache, private
server: nginx
set-cookie: gitee-session-n=BAh7CEkiD3Nlc3Npb25faWQGOgZFVEkiJTIyM2VlNjhkMWVmZGJlMWY5YmIxN2M5MGVlODEzY2Q5BjsAVEkiF21vYnlsZXR0ZV9vdmVycmlkZQY7AEY6CG5pbEkiEF9jc3JmX3Rva2VuBjsARkkiMTlsSDZmQk1CWXpWWVFTSTFtbkwzb0VJTjZjbVdVKzhYZjE0ako0djIvRUk9BjsARg%3D%3D--97ef4dc9c69d79b8f6ca42b9d0b6eaeb121d8048; domain=.gitee.com; path=/; HttpOnly
set-cookie: oschina_new_user=false; path=/; expires=Sun, 23-Jan-2039 07:22:44 GMT
set-cookie: user_locale=; path=/; expires=Sun, 23-Jan-2039 07:22:44 GMT
set-cookie: aliyungf_tc=AQAAAAq0GH/W3wsAygAc2kkmdVRPGWZs; Path=/; HttpOnly
status: 200 OK
transfer-encoding: chunked
x-rack-cache: miss
x-request-id: 437df6eccbd8a2b93912a7b84644b33d
x-runtime: 0.646640
x-ua-compatible: IE=Edge,chrome=1
x-xss-protection: 1; mode=block

<!DOCTYPE html>
<html lang='zh-CN'>
<head>
<title>spring-cloud-openfeign · Search - Gitee</title>
<link href="https://assets.gitee.com/assets/favicon-e87ded4710611ed62adc859698277663.ico" rel="shortcut icon" type="image/vnd.microsoft.icon" />
<meta charset='utf-8'>
<meta content='always' name='referrer'>
<meta content='Gitee' property='og:site_name'>
...
</html>

<--- END HTTP (48623-byte body)
```

在开启 gzip 之后，response 是：



```csharp
<--- HTTP/1.1 200 OK (987ms)
cache-control: no-cache
connection: keep-alive
content-encoding: gzip    ----------------------------- 第一处不同
content-type: text/html; charset=utf-8
date: Wed, 23 Jan 2019 07:20:59 GMT
expires: Sun, 1 Jan 2000 01:00:00 GMT
pragma: must-revalidate, no-cache, private
server: nginx
set-cookie: gitee-session-n=BAh7CEkiD3Nlc3Npb25faWQGOgZFVEkiJTVmYmMwNTQyNWU4OGMzMmYyN2M3MDQ1ZmZiNjY5ZDIzBjsAVEkiF21vYnlsZXR0ZV9vdmVycmlkZQY7AEY6CG5pbEkiEF9jc3JmX3Rva2VuBjsARkkiMVdaQ2tqYTVuTjd6WU1UKzU5R1hNbnRlbUNQaXhoSzRLRmJreXduTU51cUU9BjsARg%3D%3D--8843239d46616524d58af2611f2db9614b8518b1; domain=.gitee.com; path=/; HttpOnly
set-cookie: oschina_new_user=false; path=/; expires=Sun, 23-Jan-2039 07:20:58 GMT
set-cookie: user_locale=; path=/; expires=Sun, 23-Jan-2039 07:20:58 GMT
set-cookie: aliyungf_tc=AQAAAHojVAEggQsAygAc2ugaNNgiXCKR; Path=/; HttpOnly
status: 200 OK
transfer-encoding: chunked
x-rack-cache: miss
x-request-id: 53b45c93d5062be2c5643d9402d0a6de
x-runtime: 0.412080
x-ua-compatible: IE=Edge,chrome=1
x-xss-protection: 1; mode=block

Binary data              -------------------------------- 第二处不同
<--- END HTTP (11913-byte body)          ---------------------------- 第三处不同
```

对比可以发现：

- 在 response 的 `content-type` 上面多了一个 `content-encoding: gzip`
- 在没有开启 gzip 之前控制台打印了 html 信息，开启后没有打印，换成了 `Binary data` 二进制
- END HTTP 在没开启 gzip 之前为 48623 byte，开启后为 11913 byte

由此可以证明，response 开启 gzip 成功

------

# Feign 配置

***源码：https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-feign/spring-cloud-feign-config\***

## 对单个指定特定名称的 Feign 进行配置

在之前的例子中，在对 FeignClient 的配置中，使用的是 `@FeignClient` 的 `configuration` 属性指定的配置类，也可以使用配置文件对 `@FeignClient` 注解的接口进行配置

### FeignClient



```java
@FeignClient(name = "gitee-client", url = "https://www.gitee.com")
public interface GiteeFeignClient {

    @RequestMapping(value = "/search", method = RequestMethod.GET)
    ResponseEntity<byte[]> searchRepo(@RequestParam("q") String query);

}
```

### application.yml



```yml
feign:
  client:
    config:
      gitee-client:  # 这里指定的是 @FeignClient 的 name/value 属性的值
        connectTimeout: 5000  # 链接超时时间
        readTimeout: 5000 # 读超时
        loggerLevel: none # 日志级别
        # errorDecoder: # 错误解码器（类路径）
        # retryer: # 重试机制（类路径）
        # requestInterceptors: 拦截器配置方式 一：多个拦截器， 需要注意如果有多个拦截器，"-" 不能少
          # - Intecerptor1 类路径，
          # - Interceptpt2 类路径
        # requestInterceptors: 拦截器配置方式 二：多个拦截器，用 [Interceptor, Interceptor] 配置，需要配置类路径
        # decode404: false 是否 404 解码
        # encoder： 编码器（类路径）
        # decoder： 解码器（类路径）
        # contract： 契约（类路径）
logging:
  level:
    com.laiyy.gitee.feign.springcloudfeignconfig.feign.GiteeFeignClient: debug
```

### 验证

此时配置的 loggerLevel 为 none，不打印日志，访问： http://localhost:8080/feign-gitee?query=spring-cloud-openfeign ，可以看到控制台没有任何消息

将 loggerLevel 改为 full，再次访问可以看到打印日志消息。

将 loggerLevel 改为 feign.Logger.Level 中没有的级别，再次测试：loggerLevel: haha，可以看到控制启动报错：



```ruby
***************************
APPLICATION FAILED TO START
***************************

Description:

Failed to bind properties under 'feign.client.config.gitee-client.logger-level' to feign.Logger$Level:

    Property: feign.client.config.gitee-client.loggerlevel
    Value: haha
    Origin: class path resource [application.yml]:7:22
    Reason: failed to convert java.lang.String to feign.Logger$Level

Action:

Update your application's configuration. The following values are valid:

    BASIC
    FULL
    HEADERS
    NONE
```

可以验证此配置是正确的。

## 对全部 FeignClient 配置

对全部 FeignClient 启用配置的方法也有两种：1、`@EnableFeignClients` 注解有一个 `defaultConfiguration` 属性，可以指定全局 FeignClient 的配置。2、使用配置文件对全局 FeignClient 进行配置

application.yml



```yml
feign:
  client:
    config:
      defautl:  # 全局的配置需要把 client-name 指定为 default
        connectTimeout: 5000  # 链接超时时间
        readTimeout: 5000 # 读超时
        loggerLevel: full # 日志级别
```

如果有多个 FeignClient，每个 FeignClient 都需要单独配置，如果有一样的配置，可以提取到全局配置中，需要注意：全局配置需要放在最后一位。

在了解了 FeignClient 的配置、请求响应的压缩后，基本的调用已经没有问题。


# Http Client 替换

***源码：https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-feign/spring-cloud-feign-httpclient\***

Feign 默认情况下使用的是 JDK 原生的 URLConnection 发送 HTTP 请求，没有连接池，但是对每个地址都会保持一个长连接。可以利用 Apache HTTP Client 替换原始的 URLConnection，通过设置连接池、超时时间等，对服务调用进行调优。

在类 `feign/Client$Default.java` 中，可以看到，默认执行 http 请求的是 URLConnection



```java
public static class Default implements Client {

    @Override
    public Response execute(Request request, Options options) throws IOException {
      HttpURLConnection connection = convertAndSend(request, options);
      return convertResponse(connection).toBuilder().request(request).build();
    }
}
```

在类 `org/springframework/cloud/openfeign/ribbon/FeignRibbonClientAutoConfiguration.java` 中，可以看到引入了三个类：`HttpClientFeignLoadBalancedConfiguration`、`OkHttpFeignLoadBalancedConfiguration`、`DefaultFeignLoadBalancedConfiguration`

可以看到在 `DefaultFeignLoadBalancedConfiguration` 中，使用的是 `Client.Default`，即使用 URLConnection

## 使用 Apache Http Client 替换 URLConnection

### pom 依赖



```xml
<!-- 引入 httpclient -->
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
</dependency>

<!-- 引入 feign 对 httpclient 的支持 -->
<dependency>
    <groupId>com.netflix.feign</groupId>
    <artifactId>feign-httpclient</artifactId>
    <version>8.18.0</version>
</dependency>
```

### 配置文件



```yml
feign:
  httpclient:
    enabled: true
```

### 查看验证配置

在类 `HttpClientFeignLoadBalancedConfiguration` 上，有注解：`@ConditionalOnClass(ApacheHttpClient.class)`、`@ConditionalOnProperty(value = "feign.httpclient.enabled", matchIfMissing = true)`：在 `ApacheHttpClient` 类存在且 `feign.httpclient.enabled` 为 true 时启用配置。

在 `HttpClientFeignLoadBalancedConfiguration` 123 行打上断点，重新启动项目，可以看到确实进行了 ApacheHttpClient 的声明。在将 `feign.httpclient.enabled` 设置为 false 后，断点就进不来了。由此可以验证 ApacheHttpClient 替换成功。

## 使用 OkHttp 替换 URLConnection

### pom 依赖



```xml
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-okhttp</artifactId>
    <version>10.1.0</version>
</dependency>
```

### 配置文件



```yml
feign:
  httpclient:
    enabled: false
  okhttp:
    enabled: true
```

### 验证配置

在 `OkHttpFeignLoadBalancedConfiguration` 第 84 行打断点，重新启动项目，可以看到成功进入断点；当把 `feign.okhttp.enabled` 设置为 false 后，重新启动项目，没进入断点。证明 OkHttp 替换成功。

------

# GET 方式传递 POJO等

***源码：https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-feign/spring-cloud-feign-multi-params\***

SpringMVC 是支持 GET 方法直接绑定 POJI 的，但是 Feign 的实现并未覆盖所有 SpringMVC 的功能，常用的解决方式：

- 把 POJO 拆散成一个一个单独的属性放在方法参数里
- 把方法参数变成 Map 传递
- 使用 GET 传递 @RequestBody，这种方式有违 RESTFul。

实现 Feign 的 RequestInterceptor 中的 apply 方法，统一拦截转换处理 Feign 中 GET 方法传递 POJO 问题。而 Feign 进行 POST 多参数传递要比 Get 简单。

## provider

provider 用于模拟用户查询、修改操作，作为服务生产者

### pom 依赖：



```xml
 <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

### 配置文件：



```yml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    instance-id: ${spring.application.name}:${server.port}
spring:
  application:
    name: spring-cloud-feign-multi-params-provider
server:
  port: 8888
```

### 实体、启动类、Controller



```java
// 实体
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {

    private int id;

    private String name;

}

// 启动类
@SpringBootApplication
@EnableDiscoveryClient
public class SpringCloudFeignMultiParamsProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudFeignMultiParamsProviderApplication.class, args);
    }

}


// Controller
@RestController
@RequestMapping(value = "/user")
public class UserController {

    @GetMapping(value = "/add")
    public String addUser(User user){
        return "hello!" + user.getName();
    }

    @PostMapping(value = "/update")
    public String updateUser(@RequestBody User user){
        return "hello! modifying " + user.getName();
    }

}
```

## consumer

consumer 用于模拟服务调用，属于服务消费者，调用 provider 的具体实现

### pom 依赖：



```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>

<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

### 配置文件：



```yml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    instance-id: ${spring.application.name}:${server.port}
spring:
  application:
    name: spring-cloud-feign-multi-params-consumer
server:
  port: 8889

feign:
  client:
    config:
      spring-cloud-feign-multi-params-provider:
        loggerLevel: full
logging:
  level:
    com.laiyy.gitee.feign.multi.params.springcloudfeignmultiparamscomsumer.MultiParamsProviderFeignClient: debug
```

### 实体、启动类、Controller、FeignClient



```java
// 实体与 provider 一致，不再赘述

// 启动类
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class SpringCloudFeignMultiParamsComsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudFeignMultiParamsComsumerApplication.class, args);
    }

}


// Controller
@RestController
public class UserController {

    private final MultiParamsProviderFeignClient feignClient;

    @Autowired
    public UserController(MultiParamsProviderFeignClient feignClient) {
        this.feignClient = feignClient;
    }

    @GetMapping(value = "add-user")
    public String addUser(User user){
        return feignClient.addUser(user);
    }

    @PostMapping(value = "update-user")
    public String updateUser(@RequestBody User user){
        return feignClient.updateUser(user);
    }

}

// FeignClient
@FeignClient(name = "spring-cloud-feign-multi-params-provider")
public interface MultiParamsProviderFeignClient {

    /**
     * GET 方式
     * @param user user
     * @return 添加结果
     */
    @RequestMapping(value = "/user/add", method = RequestMethod.GET)
    String addUser(User user);

    /**
     * POST 方式
     * @param user user
     * @return 修改结果
     */
    @RequestMapping(value = "/user/update", method = RequestMethod.POST)
    String updateUser(@RequestBody User user);
}
```

## 验证调用

使用 POST MAN 测试工具，调用 consumer 接口，利用 Feign 进行远程调用

调用 `update-user`，验证调用成功

![img](https:////upload-images.jianshu.io/upload_images/13856126-c50030555ed24648.png?imageMogr2/auto-orient/strip|imageView2/2/w/582/format/webp)

POST 方式调用 update

调用 `add-user`，验证调用失败

![img](https:////upload-images.jianshu.io/upload_images/13856126-2739adc367083a29.png?imageMogr2/auto-orient/strip|imageView2/2/w/782/format/webp)

GET 方式调用 add

控制台报错：



```bash
{"timestamp":"2019-01-24T08:24:42.887+0000","status":405,"error":"Method Not Allowed","message":"Request method 'POST' not supported","path":"/user/add"}] with root cause

feign.FeignException: status 405 reading MultiParamsProviderFeignClient#addUser(User); content:
{"timestamp":"2019-01-24T08:24:42.887+0000","status":405,"error":"Method Not Allowed","message":"Request method 'POST' not supported","path":"/user/add"}
    at feign.FeignException.errorStatus(FeignException.java:62) ~[feign-core-9.5.1.jar:na]
    at feign.codec.ErrorDecoder$Default.decode(ErrorDecoder.java:91) ~[feign-core-9.5.1.jar:na]
    at feign.SynchronousMethodHandler.executeAndDecode(SynchronousMethodHandler.java:138) ~[feign-core-9.5.1.jar:na]
    ...
```

命名是 GET 调用，为什么到底层就变成了 POST 调用？

## GET 传递 POJO 解决方案

Feign 的远程调用中，GET 是不能传递 POJO 的，否则就是 POST，为了解决这个错误，可以实现 RequestInterceptor，解析 POJO，传递 Map 即可解决

在 consumer 中，增加一个实体类，用于解析 POJO



```java
/**
 * @author laiyy
 * @date 2019/1/24 10:33
 * @description 实现 Feign Request 拦截器，实现 GET 传递 POJO
 */
@Component
public class FeignRequestInterceptor implements RequestInterceptor {
    private final ObjectMapper objectMapper;
    @Autowired
    public FeignRequestInterceptor(ObjectMapper objectMapper) {
        this.objectMapper = objectMapper;
    }
    @Override
    public void apply(RequestTemplate template) {
        if ("GET".equals(template.method()) && template.body() != null) {
            try {
                JsonNode jsonNode = objectMapper.readTree(template.body());
                template.body(null);

                Map<String, Collection<String>> queries = new HashMap<>();

                // 构建 Map
                buildQuery(jsonNode, "", queries);

                // queries 就是 POJO 解析为 Map 后的数据
                template.queries(queries);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    private void buildQuery(JsonNode jsonNode, String path, Map<String, Collection<String>> queries) {
        if (!jsonNode.isContainerNode()) {
            // 如果是叶子节点
            if (jsonNode.isNull()) {
                return;
            }
            Collection<String> values = queries.get(path);
            if (CollectionUtils.isEmpty(values)) {
                values = new ArrayList<>();
                queries.put(path, values);
            }
            values.add(jsonNode.asText());
            return;
        }
        if (jsonNode.isArray()){
            // 如果是数组节点
            Iterator<JsonNode> elements = jsonNode.elements();
            while (elements.hasNext()) {
                buildQuery(elements.next(), path, queries);
            }
        } else {
            Iterator<Map.Entry<String, JsonNode>> fields = jsonNode.fields();
            while (fields.hasNext()) {
                Map.Entry<String, JsonNode> entry = fields.next();
                if (StringUtils.hasText(path)) {
                    buildQuery(entry.getValue(), path + "." + entry.getKey(), queries);
                } else {
                    // 根节点
                    buildQuery(entry.getValue(), entry.getKey(), queries);
                }
            }
        }
    }
}
```

重新启动 consumer，再次调用 `add-user`，验证结果：

![img](https:////upload-images.jianshu.io/upload_images/13856126-d4997f10ff9ed05f.png?imageMogr2/auto-orient/strip|imageView2/2/w/784/format/webp)

GET 成功调用远程接口

# 文件上传

***源码：https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-feign/spring-cloud-feign-file\***

Feign 的子项目 feign-form(https://github.com/OpenFeign/feign-form) 支持文件上传，其中实现了上传所需要的 Encoder

模拟文件上传：`spring-cloud-feign-file-server`、`spring-cloud-feign-file-client`，其中 server 模拟文件服务器，作为服务提供者；client 模拟文件上传，通过 FeignClient 发送文件到文件服务器

## FileClient

### pom 依赖



```xml
<!-- eureka client -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

<!-- feign -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>

<!-- Feign文件上传依赖-->
<dependency>
    <groupId>io.github.openfeign.form</groupId>
    <artifactId>feign-form</artifactId>
    <version>3.0.3</version>
</dependency>
<dependency>
    <groupId>io.github.openfeign.form</groupId>
    <artifactId>feign-form-spring</artifactId>
    <version>3.0.3</version>
</dependency>
```

### 配置文件



```yml
spring:
  application:
    name: spring-cloud-feign-file-client

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    instance-id: ${spring.application.name}:${server.port}
    prefer-ip-address: true
server:
  port: 8888
```

### 启动类、FeignClient、配置、Controller



```java
// 启动类
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class SpringCloudFeignFileClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringCloudFeignFileClientApplication.class, args);
    }
}

// FeignClient
@FeignClient(value = "spring-cloud-feign-file-server", configuration = FeignMultipartConfiguration.class)
public interface FileUploadFeignClient {

    /**
     * feign 上传图片
     *
     * produces、consumes 必填
     * 不要将 @RequestPart 写成 @RequestParam
     *
     * @param file 上传的文件
     * @return 上传的文件名
     */
    @RequestMapping(value = "/upload-file", method = RequestMethod.POST, produces = MediaType.APPLICATION_JSON_UTF8_VALUE, consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    String fileUpload(@RequestPart(value = "file")MultipartFile file);

}

// configuration
@Configuration
public class FeignMultipartConfiguration {

    /**
     * Feign Spring 表单编码器
     * @return 表单编码器
     */
    @Bean
    @Primary
    @Scope("prototype")
    public Encoder multipartEncoder(){
        return new SpringFormEncoder();
    }

}

// Controller
@RestController
public class FileUploadController {

    private final FileUploadFeignClient feignClient;

    @Autowired
    public FileUploadController(FileUploadFeignClient feignClient) {
        this.feignClient = feignClient;
    }

    @PostMapping(value = "upload")
    public String upload(MultipartFile file){
        return feignClient.fileUpload(file);
    }

}
```

## FileServer

### pom



```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

### 配置文件



```yml
spring:
  application:
    name: spring-cloud-feign-file-server


eureka:
  instance:
    prefer-ip-address: true
    instance-id: ${spring.application.name}:${server.port}
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
server:
  port: 8889
```

### 启动类、Controller



```java
@SpringBootApplication
@EnableDiscoveryClient
public class SpringCloudFeignFileServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudFeignFileServerApplication.class, args);
    }

}

// Controller 模拟文件上传的处理
@RestController
public class FileUploadController {

    @PostMapping(value = "/upload-file")
    public String fileUpload(MultipartFile file) {
        return file.getOriginalFilename();
    }

}
```

## 验证文件上传

POST MAN 调用 client 上传接口

![img](https:////upload-images.jianshu.io/upload_images/13856126-3d97cb29953f25df.png?imageMogr2/auto-orient/strip|imageView2/2/w/731/format/webp)

file upload

------

# 图片流

***源码：https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-feign/spring-cloud-feign-file\***

通过 Feign 返回图片，一般是字节数组

在`文件上传`代码的基础上，再加上图片获取

## FeignClient



```java
/**
  * 获取图片
  * @return 图片
  */
@RequestMapping(value = "/get-img")
ResponseEntity<byte[]> getImage();


@GetMapping(value = "/get-img")
public ResponseEntity<byte[]> getImage(){
    return feignClient.getImage();
}
```

## FeignServer



```java
@GetMapping(value = "/get-img")
public ResponseEntity<byte[]> getImages() throws IOException {
    FileSystemResource resource = new FileSystemResource(getClass().getResource("/").getPath() + "Spring-Cloud.png");
    HttpHeaders headers = new HttpHeaders();
    headers.add("Content-Type", MediaType.APPLICATION_OCTET_STREAM_VALUE);
    headers.add("Content-Disposition", "attachment; filename=Spring-Cloud.png");
    return  ResponseEntity.status(HttpStatus.OK).headers(headers).body(FileCopyUtils.copyToByteArray(resource.getInputStream()));
}
```

## 验证

在浏览器访问：http://localhost:8888/get-img ，实现图片流下载

------

# Feign 传递 Headers

***源码：https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-feign/spring-cloud-feign-multi-params\***

在认证、鉴权中，无论是哪种权限控制框架，都需要传递 header，但在使用 Feign 的时候，会发现外部请求 ServiceA 时，可以获取到 header，但是在 ServiceA 调用 ServiceB 时，ServiceB 无法获取到 Header，导致 Header 丢失。

在 `spring-cloud-feign-multi-params` 基础上，实现传递 Header。

## 验证 Header 无法传递问题

![img](https:////upload-images.jianshu.io/upload_images/13856126-6ce5ec0553ecf758.png?imageMogr2/auto-orient/strip|imageView2/2/w/524/format/webp)

post man 传递 header1

consumer 打印 header



![img](https:////upload-images.jianshu.io/upload_images/13856126-9db9c4a5d1f56ea5.png?imageMogr2/auto-orient/strip|imageView2/2/w/407/format/webp)

Consumer 打印 header

provider 打印 header



![img](https:////upload-images.jianshu.io/upload_images/13856126-1d91d53189644113.png?imageMogr2/auto-orient/strip|imageView2/2/w/485/format/webp)

Provider 打印 header

## HeaderInterceptor

在 Consumer 增加 HeaderInterceptor，做 header 传递



```java
@Component
public class FeignHeaderInterceptor implements RequestInterceptor {

    @Override
    public void apply(RequestTemplate template) {
        if (null == getRequest()){
            return;
        }
        template.header("oauth-token", getHeaders(getRequest()).get("oauth-token"));
    }

    private HttpServletRequest getRequest(){
        try {
            return ((ServletRequestAttributes) RequestContextHolder.currentRequestAttributes()).getRequest();
        }catch (Exception e){
            return null;
        }
    }

    private Map<String, String> getHeaders(HttpServletRequest request) {
        Map<String, String> headers = new HashMap<>();
        Enumeration<String> headerNames = request.getHeaderNames();
        while (headerNames.hasMoreElements()) {
            String key = headerNames.nextElement();
            String value = request.getHeader(key);
            headers.put(key, value);
        }
        return headers;
    }

}
```

## 验证 header

用 postman 重新请求一遍，查看 provider 控制台打印：

![img](https:////upload-images.jianshu.io/upload_images/13856126-958d1262904b96f9.png?imageMogr2/auto-orient/strip|imageView2/2/w/646/format/webp)

provider header

# 参考

https://www.jianshu.com/p/a227a8070731

https://www.jianshu.com/p/e29d7f6be6e3

https://www.jianshu.com/p/11710629c226

https://www.jianshu.com/p/7a99fc4b212d