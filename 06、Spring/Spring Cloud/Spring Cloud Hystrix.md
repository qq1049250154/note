`Hystrix` 是一个延迟和容错库，目的在隔离远程系统、服务、第三方库，组织级联故障，在负载的分布式系统中实现恢复能力。
 在多系统和微服务的情况下，需要一种机制来处理延迟和故障，并保护整合系统处于可用的稳定状态。`Hystrix` 就是实现这个功能的一个组件。

- 通过客户端库对延迟和故障进行保护和控制。
- 在一个复杂的分布式系统中停止级联故障
- 快速失败、迅速恢复
- 在合理的情况下回退、优雅降级
- 开启近实时监控、告警、操作控制

------

# 实例

***源码：https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-hystrix/spring-cloud-hystrix-simple\***

## pom 依赖



```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
    </dependencies>
```

## 配置文件



```yml
spring:
  application:
    name: spring-cloud-hystrix-simple
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

## Service、ServiceImpl、Controller、启动类



```java
// Service
public interface HystrixService {

    String getUser(String username) throws Exception;

}


// Service Impl
@Service
public class HystrixServiceImpl implements HystrixService {
    @Override
    @HystrixCommand(fallbackMethod = "defaultUser")
    public String getUser(String username) throws Exception {
        if ("laiyy".equals(username)) {
            return "this is real user, name:" + username;
        }
        throw new Exception();
    }

    /**
     * 在 getUser 出错时调用
     */
    public String defaultUser(String username) {
        return "this is error user, name: " + username;
    }

}


// Controller
@RestController
public class HystrixController {

    private final HystrixService hystrixService;

    @Autowired
    public HystrixController(HystrixService hystrixService) {
        this.hystrixService = hystrixService;
    }

    @GetMapping(value = "get-user")
    public String getUser(@RequestParam String username) throws Exception{
        return hystrixService.getUser(username);
    }
}


// Controller
@SpringBootApplication
@EnableDiscoveryClient
@EnableHystrix
public class SpringCloudHystrixSimpleApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudHystrixSimpleApplication.class, args);
    }

}
```

## 验证

访问 http://localhost:8888/get-user

**传入正确 username**

![image-20210206120146526](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210206120146526.png)

正确 username



**传入错误 username**

![image-20210206120155909](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210206120155909.png)

错误 username



## 注解、解释

在上例中，可以发现，在启动类上多了 `@EnableHystrix` 注解，在 Service 中多了 `@HystrixCommand(fallbackMethod = "defaultUser")` 和一个名称为 defaultUser 的方法。

- `@EnableHystrix`：开启 Hystrix 断路器
- `@HystrixCommand`：fallbackMethod 指定当该注解标记的方法出现失败、错误时，调用哪一个方法进行优雅的降级返回，对用户屏蔽错误，做优雅提示。

@HystrixCommand 需要注意

- fallbackMethod 的值，为方法名。如果方法指定的方法名不存在，会出现如下错误：



```java
com.netflix.hystrix.contrib.javanica.exception.FallbackDefinitionException: fallback method wasn't found: defaultUser1([class java.lang.String])
    at com.netflix.hystrix.contrib.javanica.utils.MethodProvider$FallbackMethodFinder.doFind(MethodProvider.java:190) ~[hystrix-javanica-1.5.12.jar:1.5.12]
    at com.netflix.hystrix.contrib.javanica.utils.MethodProvider$FallbackMethodFinder.find(MethodProvider.java:159) ~[hystrix-javanica-1.5.12.jar:1.5.12]
    at com.netflix.hystrix.contrib.javanica.utils.MethodProvider.getFallbackMethod(MethodProvider.java:73) ~[hystrix-javanica-1.5.12.jar:1.5.12]
    at com.netflix.hystrix.contrib.javanica.utils.MethodProvider.getFallbackMethod(MethodProvider.java:59) ~[hystrix-javanica-1.5.12.jar:1.5.12]
    ...
```

- fallbackMethod 方法的参数与 @HystrixCommand 标注的参数类型、顺序一致，如果不一致，会出现如下错误：



```java
com.netflix.hystrix.contrib.javanica.exception.FallbackDefinitionException: fallback method wasn't found: defaultUser([class java.lang.String])
    at com.netflix.hystrix.contrib.javanica.utils.MethodProvider$FallbackMethodFinder.doFind(MethodProvider.java:190) ~[hystrix-javanica-1.5.12.jar:1.5.12]
    at com.netflix.hystrix.contrib.javanica.utils.MethodProvider$FallbackMethodFinder.find(MethodProvider.java:159) ~[hystrix-javanica-1.5.12.jar:1.5.12]
    ...
```

------

# 断路器

***源码：https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-hystrix/spring-cloud-hystrix-feign-broker\***

断路器的作用：在服务出现错误的时候，熔断该服务，保护调用者，防止出现雪崩。

在使用 feign 断路器时，feign 默认是不开启 Hystrix 断路器的，需要手动配置。

## pom



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    </dependency>
</dependencies>
```

## 配置文件

application.yml



```yml
feign:
  hystrix:
    enabled: true
```

boorstrap.yml



```yml
spring:
  application:
    name: spring-cloud-hystrix-feign-broker
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

## feign、Fallback、Controller、启动类



```java
// feign
@FeignClient(name = "spring-cloud-ribbon-provider",fallback = FeignHystrixClientFallback.class)
public interface FeignHystrixClient {

    @RequestMapping(value = "/check", method = RequestMethod.GET)
    String feignHystrix();

}


// fallback
@Component
public class FeignHystrixClientFallback implements FeignHystrixClient {
    @Override
    public String feignHystrix() {
        return "error! this is feign hystrix";
    }
}

// controller
@RestController
public class FeignHystrixController {

    private final FeignHystrixClient feignHystrixClient;

    @Autowired
    public FeignHystrixController(FeignHystrixClient feignHystrixClient) {
        this.feignHystrixClient = feignHystrixClient;
    }

    @GetMapping(value = "feign-hystrix")
    public String feignHystrix(){
        return feignHystrixClient.feignHystrix();
    }
}


// 启动类
@SpringBootApplication
@EnableHystrix
@EnableDiscoveryClient
@EnableFeignClients
public class SpringCloudHystrixFeignBrokerApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudHystrixFeignBrokerApplication.class, args);
    }

}
```

## 验证

使用 `spring-cloud-ribbon-provider` 作为服务提供者，当前项目作为服务调用者

** spring-cloud-ribbon-provider 正常访问 **



![image-20210206120209369](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210206120209369.png)

feign hystrix success

** 停止 spring-cloud-ribbon-provider 服务，再次访问 **

![image-20210206120219631](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210206120219631.png)

feign hystrix error

可以看到，当 `spring-cloud-ribbon-provider` 服务停止时，会进入 Fallback 声明的 Class 中。

Fallback class 必须是 @FeignClient 标注的 interface 的实现类，且每个方法都需要自定义实现。

# Turbine

***源码：https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-hystrix/spring-cloud-hystrix-dashboard/spring-cloud-hystrix-dashboard-turbine\***

微服务继续沿用上例中的 hello-service、provider-service，新建 Turbine 项目

## pom



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-turbine</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
</dependencies>
```

## 配置文件



```yml
server:
  port: 9999

spring:
  application:
    name: spring-cloud-hystrix-dashboard-turbne

eureka:
  instance:
    prefer-ip-address: true
    instance-id: ${spring.application.name}:${server.port}
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/

management:
  endpoints:
    web:
      exposure:
        exclude: hystrix.stream # Turbine 要监控的端点
turbine:
  app-config: spring-cloud-hystrix-dashboard-hello-service,spring-cloud-hystrix-dashboard-provider-service # Turbine 要监控的服务
  cluster-name-expression: "'default'" # 集群名称，默认 default
```

## 启动类



```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableHystrixDashboard
@EnableTurbine // 开启 Turbine
public class SpringCloudHystrixDashboardTurbineApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudHystrixDashboardTurbineApplication.class, args);
    }

}
```

## 验证

浏览器中访问 Turbine： http://localhost:9999/hystrix ，输入 Turbine 监控端点：http://localhost:9999/turbine.stream

![image-20210206120241183](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210206120241183.png)

turbine 没有监控到 服务

访问 hello-service(http://localhost:8080/get-provider-data)、provider-service(http://localhost:8081/get-hello-service)，再次查看 turbine

![image-20210206120250894](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210206120250894.png)

turbine 监控到服务

------

# 异常处理

***源码：https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-hystrix/spring-cloud-hystrix-dashboard/spring-cloud-hystrix-dashboard-exception\***

Hystrix 的异常处理中，有五种出错情况会被 Fallback 截获，触发 Fallbac

- FAILURE：执行失败，抛出异常
- TIMEOUT：执行超时
- SHORT_CIRCUITED：断路器打开
- THREAD_POOL_REJECTED：线程池拒绝
- SEMAPHORE_REJECTED：信号量拒绝

但是有一种类型的异常不会触发 Fallback 且不会被计数、不会熔断——`BAD_REQUEST`。BAD_ERQUEST 会抛出 HystrixBadRequestException，这种异常一般是因为对应的参数或系统参数异常引起的，对于这类异常，可以根据响应创建对应的异常进行异常封装或直接处理。

## BAD_REQUEST 处理

### pom



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
</dependencies>
```

### 配置文件



```yml
server:
  port: 9999

spring:
  application:
    name: spring-cloud-hystrix-dashboard-exception

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
    instance-id: ${spring.application.name}:${server.port}

management:
  endpoints:
    web:
      exposure:
        exclude: hystrix.stream

feign:
  hystrix:
    enabled: true
```

### 异常处理

在 Hystrix 中处理异常，需要继承 HystrixCommand<R> 并重写 run、getFallback 方法

#### bad request



```java
public class FallBackBadRequestException extends HystrixCommand<String> {

    private static final Logger LOGGER = LoggerFactory.getLogger(FallBackBadRequestException.class);

    public FallBackBadRequestException() {
        // HystrixCommand 分组 key
        super(HystrixCommandGroupKey.Factory.asKey("GroupBadRequestException"));
    }

    @Override
    protected String run() throws Exception {
        // 直接抛出异常，模拟 BAD_REQUEST  
        throw new HystrixBadRequestException("this is HystrixBadRequestException！");
    }

    // Fallback 回调
    @Override
    protected String getFallback() {
        System.out.println("Fallback 错误信息：" + getFailedExecutionException().getMessage());
        return "this is HystrixBadRequestException Fallback method!";
    }
}
```

#### 其他错误



```java
public class FallBackOtherException extends HystrixCommand<String> {

    public FallBackOtherException() {
        super(HystrixCommandGroupKey.Factory.asKey("otherException"));
    }

    @Override
    protected String run() throws Exception {
        throw new Exception("other exception");
    }

    @Override
    protected String getFallback() {
        return "fallback!";
    }
}
```

#### 模拟 feign 调用



```java
public class ProviderServiceCommand extends HystrixCommand<String> {

    private final String name;

    public ProviderServiceCommand(String name){
        super(HystrixCommandGroupKey.Factory.asKey("springCloud"));
        this.name = name;
    }

    @Override
    protected String run() throws Exception {
        // 模拟 feign 远程调用返回
        return "spring cloud!" + name;
    }

    @Override
    protected String getFallback() {
        return "spring cloud fail!";
    }
}
```

#### Controller



```java
@RestController
public class ExceptionController {

    private static final Logger LOGGER = LoggerFactory.getLogger(ExceptionController.class);

    // 模拟 feign 调用
    @GetMapping(value = "provider-service-command")
    public String providerServiceCommand(){
        return new ProviderServiceCommand("laiyy").execute();
    }

    // bad Request
    @GetMapping(value = "fallback-bad-request")
    public String fallbackBadRequest(){
        return new FallBackBadRequestException().execute();
    }

    // 其他错误
    @GetMapping(value = "fallback-other")
    public String fallbackOther(){
        return new FallBackOtherException().execute();
    }

    // @HystrixCommand 处理 Fallback
    @GetMapping(value = "fallback-method")
    @HystrixCommand(fallbackMethod = "fallback")
    public String fallbackMethod(String id){
        throw new RuntimeException("fallback method !");
    }

    public String fallback(String id, Throwable throwable){
        LOGGER.error(">>>>>>>>>>>>>>> 进入 @HystrixCommand fallback！");
        return "this is @HystrixCommand fallback!";
    }
}
```

### 验证

依次请求 Controller 中的接口，可以看到，除了 `fallback-bad-request` 外，其他的接口都进入了 Fallback 方法中。证明了 BAD_REQUEST 不会触发 Fallback。

## BAD_REQUEST Fallback

可以使用 ErrorDecoder 对 BAD_REQUEST 进行包装



```java
@Component
public class BadRequestErrorDecoder implements ErrorDecoder {
    @Override
    public Exception decode(String methodKey, Response response) {
        if (response.status() >= 400 && response.status() <= 499) {
            String error = null;
            try {
                error = Util.toString(response.body().asReader());
            } catch (IOException e) {
                System.out.println("BadRequestErrorDecoder 出错了！" + e.getLocalizedMessage());
            }
            return new HystrixBadRequestException(error);
        }
        return FeignException.errorStatus(methodKey, response);
    }
}
```

之后在 yml 配置文件中增加对微服务调用的 ErrorDecoder 配置



```yml
feign:
  hystrix:
    enabled: true
  client:
    config:
      spring-cloud-hystrix-dashboard-provider-service: # 针对哪个服务
        errorDecoder: com.laiyy.gitee.dashboard.springcloudhystrixdashboardexception.decoder.BadRequestErrorDecoder # 错误解码器
```

------

# Hystrix 配置

一个简单的 Hystrix 配置，基本有一下几个内容



```yml
hystrix:
  command:
    default: # default 为全局配置，如果需要针对某个 Fallback 配置，需要使用 HystrixCommandKey
      circuitBreaker:
        errorThresholdPercentage: 50 # 这是打开 Fallback 并启动 Fallback 的错误比例。默认 50%
        forceOpen: false # 是否强制打开断路器，拒绝所有请求。默认 false
      execution:
        isolation:
          strategy: THREAD # SEMAPHORE  请求隔离策略，默认 THREAD
          # 当 strategy 为 THREAD 时
          thread:
            timeoutInMilliseconds: 5000 # 执行超时时间  默认 1000
            interruptOnTimeout: true # 超时时是否中断执行，默认 true
          # 当 strategy 为 SEMAPHORE 时
          semaphore:
            maxConcurrentRequests: 10 # 最大允许请求数，默认 10
        # 是否开启超时
        timeout:
          enabled: true # 默认 true
  # 当隔离策略为 thread 时
  threadpool: 
    default: # default 为全局配置，如果需要再很对某个 线程池 配置，需要使用 HystrixThreadPoolKey
      coreSize: 10 # 默认线程池大小，默认 10
      maximumSize: 10 # 最大线程池，默认 10
      allowMaximumSizeToDivergeFromCoreSize: false # 是否允许 maximumSize 配置生效
```

## 隔离策略



```yml
hystrix:
  command:
    default: 
      execution:
        isolation:
          strategy: THREAD # SEMAPHORE  请求隔离策略，默认 THREAD
```

隔离策略有两种：线程隔离策略和信号量隔离策略。分别对应：`THREAD`、`SEMAPHORE`。

### 线程隔离

Hystrix 默认的隔离策略，通过线程池大小可以控制并发量，当线程饱和时，可以拒绝服务，防止出现问题。

优点：

- 完全隔离第三方应用，请求线程可以快速收回
- 请求线程可以继续接受新的请求，如果出现线程问题，线程池隔离是独立的，不会影响其他应用
- 当失败的应用再次变得可用时，线程池将清理并可以立即恢复
- 独立的线程池提高了并发性

缺点：

- 增加CPU开销，每个命令的执行都涉及到线程的排队、调度、上下文切换等。

### 信号量隔离

使用一个原子计数器(信号量)来记录当前有多少线程正在运行，当请求进来时，先判断计数器的数值(默认10)，如果超过设置则拒绝请求，否则正常执行，计数器+1。成功执行后，计数器-1。

与`线程隔离`的最大区别是，执行请求的线程依然是`请求线程`，而不是线程隔离中分配的线程池。

### 对单个 HystrixCommand 配置隔离策略等



```java
@HystrixCommand(fallbackMethod = "defaultUser", commandProperties = {
        // 配置线程隔离策略， value 可以是 THREAD 或 SEMAPHORE
        @HystrixProperty(name = HystrixPropertiesManager.EXECUTION_ISOLATION_STRATEGY, value = "THREAD")
})
```

### 应用场景

线程隔离：第三方应用、接口；并发量大
 信号量隔离：内部应用、中间件(redis)；并发量不大

## HystrixCommandKey、HystrixThreadPoolKey

HystrixCommandKey 是一个 @HystrixCommand 注解标注的方法的 key，默认为标注方法的`方法名`，也可以使用 @HystrixCommand 进行配置
 HystrixThreadPoolKey 是 Hystrix 开启线程隔离策略后，指定的线程池名称，可以使用 @HystrixCommand 配置



```java
@HystrixCommand(fallbackMethod = "defaultUser",
    // 隔离策略
    commandProperties = {
        @HystrixProperty(name = HystrixPropertiesManager.EXECUTION_ISOLATION_STRATEGY, value = "THREAD")
}, 
    // HystrixCommandKey
    commandKey = "commandKey", 
    // HystrixThreadPoolKey
    threadPoolKey = "threadPoolKey", 
    // 线程隔离策略配置超时时间
    threadPoolProperties = {
        @HystrixProperty(name = HystrixPropertiesManager.EXECUTION_ISOLATION_THREAD_INTERRUPT_ON_TIMEOUT, value = "5000")
})
```

# Hystrix 线程调整

线程的调整主要依赖于在生产环境中的实际情况与服务器配置进行相对应的调整，由于生产环境不可能完全一致，所以没有一个具体的值。

# 请求缓存

Hystrix 请求缓存是 Hystrix 在同一个上下文请求中缓存请求结果，与传统缓存有区别。Hystrix 的请求缓存是在同一个请求中进行，在第一次请求调用结束后对结果缓存，然后在接下来同参数的请求会使用第一次的结果。
 Hystrix 请求缓存的声明周期为一次请求。传统缓存的声明周期根据时间需要设定，最长可能长达几年。

Hystrix 请求有两种方式：继承 HystrixCommand 类、使用 @HystrixCommand 注解。Hystrix 缓存同时支持这两种方案。

## Cache Consumer

***源码：https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-hystrix/spring-cloud-hystrix-cache/spring-cloud-hystrix-cache-impl\***

### pom、yml



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    </dependency>
</dependencies>
```



```yml
server:
  port: 8989
spring:
  application:
    name: spring-cloud-hystrix-cache-impl

eureka:
  instance:
    instance-id: ${spring.application.name}:${server.port}
    prefer-ip-address: true
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

### Interceptor



```java
public class CacheContextInterceptor implements HandlerInterceptor {

    private HystrixRequestContext context;

    /**
     * 请求前
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        context = HystrixRequestContext.initializeContext();
        return true;
    }

    /**
     * 请求
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    /**
     * 请求后
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        context.shutdown();
    }
}


/**
 * 将 Interceptor 注册到Spring MVC 控制器
 */
@Configuration
public class CacheConfiguration {

    /**
     * 声明一个 cacheContextInterceptor 注入 Spring 容器
     */
    @Bean
    @ConditionalOnClass(Controller.class)
    public CacheContextInterceptor cacheContextInterceptor(){
        return new CacheContextInterceptor();
    }

    @Configuration
    @ConditionalOnClass(Controller.class)
    public class WebMvcConfig extends WebMvcConfigurationSupport{

        private final CacheContextInterceptor interceptor;

        @Autowired
        public WebMvcConfig(CacheContextInterceptor interceptor) {
            this.interceptor = interceptor;
        }

        /**
         * 将 cacheContextInterceptor 添加到拦截器中
         */
        @Override
        protected void addInterceptors(InterceptorRegistry registry) {
            registry.addInterceptor(interceptor);
        }
    }

}
```

### @HystrixCommand 方式



```java
// feign 调用接口
public interface IHelloService {

    String hello(int id);

    String getUserToCommandKey(@CacheKey int id);

    String updateUser(@CacheKey int id);

}


// 具体实现
@Component
public class HelloServiceImpl implements IHelloService {

    private final RestTemplate restTemplate;

    @Autowired
    public HelloServiceImpl(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    @Override
    @CacheResult
    @HystrixCommand
    public String hello(int id) {
        String result = restTemplate.getForObject("http://spring-cloud-hystrix-cache-provider-user/get-user/{1}", String.class, id);
        System.out.println("正在进行远程调用：hello " + result);
        return result;
    }

    @Override
    @CacheResult
    @HystrixCommand(commandKey = "getUser")
    public String getUserToCommandKey(int id) {
        String result = restTemplate.getForObject("http://spring-cloud-hystrix-cache-provider-user/get-user/{1}", String.class, id);
        System.out.println("正在进行远程调用：getUserToCommandKey " + result);
        return result;
    }

    @Override
    @CacheRemove(commandKey = "getUser")
    @HystrixCommand
    public String updateUser(int id) {
        System.out.println("正在进行远程调用：updateUser " + id);
        return "update success";
    }
}
```

### 继承 HystrixCommand 类形式



```java
public class HelloCommand extends HystrixCommand<String> {

    private RestTemplate restTemplate;

    private int id;

    public HelloCommand(RestTemplate restTemplate, int id){
        super(HystrixCommandGroupKey.Factory.asKey("springCloudCacheGroup"));
        this.id = id;
        this.restTemplate = restTemplate;
    }

    @Override
    protected String run() throws Exception {
        String result = restTemplate.getForObject("http://spring-cloud-hystrix-cache-provider-user/get-user/{1}", String.class, id);
        System.out.println("正在使用继承 HystrixCommand 方式进行远程调用：" + result);
        return result;
    }

    @Override
    protected String getFallback() {
        return "hello command fallback";
    }

    @Override
    protected String getCacheKey() {
        return String.valueOf(id);
    }

    public static void cleanCache(int id) {
        HystrixRequestCache.getInstance(
                HystrixCommandKey.Factory.asKey("springCloudCacheGroup"),
                HystrixConcurrencyStrategyDefault.getInstance())
                .clear(String.valueOf(id));
    }
}
```

### Controller



```java
@RestController
public class CacheController {

    private final RestTemplate restTemplate;

    private final IHelloService helloService;

    @Autowired
    public CacheController(RestTemplate restTemplate, IHelloService helloService) {
        this.restTemplate = restTemplate;
        this.helloService = helloService;
    }

    /**
     * 缓存测试
     */
    @GetMapping(value = "/get-user/{id}")
    public String getUser(@PathVariable int id) {
        helloService.hello(id);
        helloService.hello(id);
        helloService.hello(id);
        helloService.hello(id);
        return "getUser success!";
    }

    /**
     * 缓存更新
     */
    @GetMapping(value = "/get-user-id-update/{id}")
    public String getUserIdUpdate(@PathVariable int id){
        helloService.hello(id);
        helloService.hello(id);
        helloService.hello(5);
        helloService.hello(5);
        return "getUserIdUpdate success!";
    }

    /**
     * 继承 HystrixCommand 方式
     */
    @GetMapping(value = "/get-user-id-by-command/{id}")
    public String getUserIdByCommand(@PathVariable int id){
        HelloCommand helloCommand = new HelloCommand(restTemplate, id);
        helloCommand.execute();
        System.out.println("from Cache:"  + helloCommand.isResponseFromCache()) ;
        helloCommand = new HelloCommand(restTemplate, id);
        helloCommand.execute();
        System.out.println("from Cache:"  + helloCommand.isResponseFromCache()) ;
        return "getUserIdByCommand success!";
    }

    /**
     * 缓存、清除缓存
     */
    @GetMapping(value = "/get-and-update/{id}")
    public String getAndUpdateUser(@PathVariable int id){
        // 缓存数据
        helloService.getUserToCommandKey(id);
        helloService.getUserToCommandKey(id);

        // 缓存清除
        helloService.updateUser(id);

        // 再次缓存
        helloService.getUserToCommandKey(id);
        helloService.getUserToCommandKey(id);

        return "getAndUpdateUser success!";
    }
}
```

## Cache Service

***源码：https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-hystrix/spring-cloud-hystrix-cache/spring-cloud-hystrix-cache-provider-user\***

### pom、yml



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
</dependencies>
```



```yml
server:
  port: 9999
spring:
  application:
    name: spring-cloud-hystrix-cache-provider-user

eureka:
  instance:
    prefer-ip-address: true
    instance-id: ${spring.application.name}:${server.port}
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

### Controller



```java
@RestController
public class UserController {

    @GetMapping(value = "/get-user/{id}")
    public User getUser(@PathVariable int id) {
        switch (id) {
            case 1:
                return new User("zhangsan", "list", 22);
            case 2:
                return new User("laiyy", "123456", 24);
            default:
                return new User("hahaha", "error", 0);
        }
    }

}
```

## 验证

### 验证 @HystrixCommand 注解形式缓存

请求 http://localhost:8989/get-user/2 ，查看控制台输出，发现控制台输出一次：



```bash
正在进行远程调用：hello {"username":"laiyy","password":"123456","age":24}
```

在 `HelloServiceImpl` 中，去掉 hello 方法的 @CacheResult 注解，重新启动后请求，发现控制台输出了 4 次：



```bash
正在进行远程调用：hello {"username":"laiyy","password":"123456","age":24}
正在进行远程调用：hello {"username":"laiyy","password":"123456","age":24}
正在进行远程调用：hello {"username":"laiyy","password":"123456","age":24}
正在进行远程调用：hello {"username":"laiyy","password":"123456","age":24}
```

由此验证 @HystrixCommand 注解形式缓存成功

### 验证 @HystrixCommand 形式中途修改参数

请求 http://localhost:8989/get-user-id-update/2 ，查看控制台，发现控制台输出：



```bash
正在进行远程调用：hello {"username":"laiyy","password":"123456","age":24}
正在进行远程调用：hello {"username":"hahaha","password":"error","age":0}
```

由此验证在调用 hello 方法时，hello 的参数改变后，会再次进行远程调用

### 验证清理缓存

请求 http://localhost:8989/get-and-update/2 ，查看控制，发现控制台输出：



```bash
正在进行远程调用：getUserToCommandKey {"username":"laiyy","password":"123456","age":24}
正在进行远程调用：updateUser 2
正在进行远程调用：getUserToCommandKey {"username":"laiyy","password":"123456","age":24}
```

修改 update 方法的 commandKey，重新启动项目，再次请求，发现控制台输出：



```bash
正在进行远程调用：getUserToCommandKey {"username":"laiyy","password":"123456","age":24}
正在进行远程调用：updateUser 2
```

比较后发现，修改 commandKey 后，没有进行再次调用，证明 update 没有清理掉 getUserToCommandKey 的缓存。
 由此验证在调用 getUserToCommandKey 方法时，会根据 `commandKey` 进行缓存，在调用 updateUser 方法时，会根据 `commandKey` 进行缓存删除。缓存删除后再次调用，会再次调用远程接口。

## 继承 HystrixCommand 方式

访问 http://localhost:8989/get-user-id-by-command/2 ，查看控制台：



```csharp
正在使用继承 HystrixCommand 方式进行远程调用：{"username":"laiyy","password":"123456","age":24}
from Cache:false
from Cache:true
```

可以看到，第二次请求中，`isResponseFromCache` 为 true，证明缓存生效。

由上面几种方式请求可以验证，Husyrix 的缓存可以由 @HystrixCommand 实现，也可以由继承 HystrixCommand 实现。

## 总结

- @CacheResult：使用该注解后，调用结果会被缓存，要和 @HystrixCommand 同时使用，注解参数用 cacheKeyMethod
- @CacheRemove：清除缓存，需要指定 commandKey，参数为 commandKey、cacheKeyMethod
- @CacheKey：指定请求参数，默认使用方法的所有参数作为缓存 key，直接属性为 value。
   一般在读操作接口上使用 @CacheResult、在写操作接口上使用 @CacheRemove

注意事项：
 再一些请求量大或者重复调用接口的情况下，可以利用缓存有效减轻请求压力，但是在使用 Hystrix 缓存时，需要注意：

- 需要开启 @EnableHystrix
- 需要初始化 HystrixRequestContext
- 在指定了 HystrixCommand 的 commandKey 后，在 @CacheRemove 也要指定 commandKey

如果不初始化 HystrixRequestContext，即在 `CacheContextInterceptor` 中不使用 `HystrixRequestContext.initializeContext()` 初始化，进行调用时会出现如下错误：



```swift
java.lang.IllegalStateException: Request caching is not available. Maybe you need to initialize the HystrixRequestContext?
    at com.netflix.hystrix.HystrixRequestCache.get(HystrixRequestCache.java:104) ~[hystrix-core-1.5.12.jar:1.5.12]
    at com.netflix.hystrix.AbstractCommand$7.call(AbstractCommand.java:478) ~[hystrix-core-1.5.12.jar:1.5.12]
    ...
```

另外，使用 RestTemplate 进行远程调用时，在指定远程服务时，如果出现如下错误，需要在 RestTemplate 上使用 @LoadBalance



```css
java.net.UnknownHostException: spring-cloud-hystrix-cache-provider-user
    at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:184) ~[na:1.8.0_171]
    at java.net.PlainSocketImpl.connect(PlainSocketImpl.java:172) ~[na:1.8.0_171]
    at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392) ~[na:1.8.0_171]
    at java.net.Socket.connect(Socket.java:589) ~[na:1.8.0_171]
    at java.net.Socket.connect(Socket.java:538) ~[na:1.8.0_171]
    ...
```

# Hystrix Collapser

***源码：https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-hystrix/spring-cloud-hsytrix-collapser\***

## 实例

### pom、yml



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    </dependency>
</dependencies>
```



```yml
server:
  port: 8989

spring:
  application:
    name: spring-cloud-hystrix-collapser

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
  instance:
    instance-id: ${spring.application.name}:${server.port}
    prefer-ip-address: true
```

### service、config、controller



```java
public interface ICollapserService {

    Future<Animal> collapsing(Integer id);

    Animal collapsingSyn(Integer id);

    Future<Animal> collapsingGlobal(Integer id);

}
```



```java
@Component
public class CollpaserServiceImpl implements ICollapserService {

    @Override
    @HystrixCollapser(batchMethod = "collapsingList", collapserProperties = {
            @HystrixProperty(name = HystrixPropertiesManager.TIMER_DELAY_IN_MILLISECONDS, value = "1000")
    })
    public Future<Animal> collapsing(Integer id) {
        return null;
    }

    @Override
    @HystrixCollapser(batchMethod = "collapsingList", collapserProperties = {
            @HystrixProperty(name = HystrixPropertiesManager.TIMER_DELAY_IN_MILLISECONDS, value = "1000")
    })
    public Animal collapsingSyn(Integer id) {
        return null;
    }

    @Override
    @HystrixCollapser(batchMethod = "collapsingListGlobal",
            scope = com.netflix.hystrix.HystrixCollapser.Scope.GLOBAL,
            collapserProperties = {
                    @HystrixProperty(name = HystrixPropertiesManager.TIMER_DELAY_IN_MILLISECONDS, value = "10000")
            })
    public Future<Animal> collapsingGlobal(Integer id) {
        return null;
    }

    @HystrixCommand
    public List<Animal> collapsingList(List<Integer> animalParam) {
        System.out.println("collapsingList 当前线程：" + Thread.currentThread().getName());
        System.out.println("当前请求参数个数：" + animalParam.size());
        List<Animal> animalList = Lists.newArrayList();
        animalParam.forEach(num -> {
            Animal animal = new Animal();
            animal.setName(" Cat - " + num);
            animal.setAge(num);
            animal.setSex("male");
            animalList.add(animal);
        });
        return animalList;
    }

    @HystrixCommand
    public List<Animal> collapsingListGlobal(List<Integer> animalParam) {
        System.out.println("collapsingList 当前线程：" + Thread.currentThread().getName());
        System.out.println("当前请求参数个数：" + animalParam.size());
        List<Animal> animalList = Lists.newArrayList();
        animalParam.forEach(num -> {
            Animal animal = new Animal();
            animal.setName(" Dog - " + num);
            animal.setAge(num);
            animal.setSex("female");
            animalList.add(animal);
        });
        return animalList;
    }

}
```



```java
@Configuration
public class CollapserConfiguration {


    @Bean
    @ConditionalOnClass(Controller.class)
    public HystrixCollapserInterceptor hystrixCollapserInterceptor(){
        return new HystrixCollapserInterceptor();
    }

    @Configuration
    @ConditionalOnClass(Controller.class)
    public class WebMvcConfig extends WebMvcConfigurationSupport{
        private final HystrixCollapserInterceptor interceptor;

        @Autowired
        public WebMvcConfig(HystrixCollapserInterceptor interceptor) {
            this.interceptor = interceptor;
        }

        @Override
        protected void addInterceptors(InterceptorRegistry registry) {
            registry.addInterceptor(interceptor);
        }
    }

}
```



```java
@RestController
public class CollapserController {

    private final ICollapserService collapserService;

    @Autowired
    public CollapserController(ICollapserService collapserService) {
        this.collapserService = collapserService;
    }

    @GetMapping("/get-animal")
    public String getAnimal() throws Exception{
        Future<Animal> animal = collapserService.collapsing(1);
        Future<Animal> animal2 = collapserService.collapsing(2);
        System.out.println(animal.get().getName());
        System.out.println(animal2.get().getName());
        return "success";
    }

    @GetMapping(value = "/get-animal-syn")
    public String getAnimalSyn() throws Exception{
        Animal animal1 = collapserService.collapsingSyn(1);
        Animal animal2 = collapserService.collapsingSyn(2);
        System.out.println(animal1.getName());
        System.out.println(animal2.getName());
        return "success";
    }

    @GetMapping(value = "/get-animal-global")
    public String getAnimalGlobal() throws Exception {
        Future<Animal> animal1 = collapserService.collapsingGlobal(1);
        Future<Animal> animal2 = collapserService.collapsingGlobal(2);
        System.out.println(animal1.get().getName());
        System.out.println(animal2.get().getName());
        return "success";
    }

}
```

### 验证

访问： http://localhost:8989/get-animal 、http://localhost:8989/get-animal-syn 、 http://localhost:8989/get-animal-global ，查看控制台：

http://localhost:8989/get-animal：



```undefined
collapsingList 当前线程：hystrix-CollpaserServiceImpl-1
当前请求参数个数：2
 Cat - 1
 Cat - 2
```

http://localhost:8989/get-animal-syn：



```undefined
collapsingList 当前线程：hystrix-CollpaserServiceImpl-2
当前请求参数个数：1
collapsingList 当前线程：hystrix-CollpaserServiceImpl-3
当前请求参数个数：1
 Cat - 1
 Cat - 2
```

http://localhost:8989/get-animal-global：



```undefined
collapsingListGlobal 当前线程：hystrix-CollpaserServiceImpl-4
当前请求参数个数：2
 Dog - 1
 Dog - 2
```

可以看到，get-animal、gei-animal-global 合并了请求，get-animal-syn 没有合并请求

## 注意点

- 要合并的请求，必须返回 Future，通过在实现类上增加 `@HystrixCollapser` 注解，之后调用该方法来实现请求合并。（需要特别注意：方法返回值不是 Future 无法合并请求）
- `@HystrixCollapser` 表示合并请求，调用该注解标注的方法时，实际上调用的是 `batchMethod` 方法，且利用 `HystrixProperty` 指定 `timerDelayInMilliseconds`，表示合并多少 ms 之内的请求。默认是 10ms
- 如果不在 `@HystrixCollapser` 中添加 scope=GLOBAL ，则只会合并服务的多次请求。当 `scope=GLOBAL` 时，会合并规定时间内的所有服务的多次请求。

scope 有两个值， `REQUEST\GLOBAL`。 REQUEST 表示合并单个服务的调用，GLOBAL 表示合并所有服务的调用。默认为 REQUEST。

## 总结

Hystrix Collapser 主要用于请求合并的场景。当在某个时间内有大量或并发的相同请求时，适用于请求合并；如果在某个时间内只有很少的请求，且延迟也不高，使用请求反而会增加复杂度和延迟，Collapser 本身也需要时间进行处理。

------

# Hystrix 线程传递、并发

***源码：https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-hystrix/spring-cloud-hystrix-thread\***

Hystrix 的两种隔离策略：线程隔离、信号量隔离。

如果是信号量隔离，则 Hystrix 在请求时会尝试获取一个信号量，如果成功拿到，则继续请求，该请求在一个线程内完成。如果是线程隔离，Hystrix 会把请求放入线程池执行，这是可能产生线程变化，导致`线程1`的上下文在`线程2`中无法获取到。

## 不适应 Hystrix 调用

pom、yml 等不再赘述

### service、controller



```java
// 接口
public interface IThreadService {
    String getUser(int id);
}


// 具体调用实现
@Component
public class ThreadServiceImpl implements IThreadService {

    private static final Logger LOGGER = LoggerFactory.getLogger(ThreadServiceImpl.class);

    private final RestTemplate restTemplate;

    @Autowired
    public ThreadServiceImpl(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

    @Override
    public String getUser(int id) {
        LOGGER.info("================================== Service ==================================");
        LOGGER.info(" ThreadService, Current thread id : {}", Thread.currentThread().getId());
        LOGGER.info(" ThreadService, HystrixThreadLocal: {}", HystrixThreadLocal.threadLocal.get());
        LOGGER.info(" ThreadService, RequestContextHolder: {}", RequestContextHolder.currentRequestAttributes().getAttribute("userId", RequestAttributes.SCOPE_REQUEST));
        // 这里调用的是 spring-cloud-hystrix-cache 的 provider
        return restTemplate.getForObject("http://spring-cloud-hystrix-cache-provider-user/get-user/{1}", String.class, id);
    }
}


// thread local
public class HystrixThreadLocal {
    public static ThreadLocal<String> threadLocal = new InheritableThreadLocal<>();
}


// controller
@RestController
public class ThreadController {

    private static final Logger LOGGER = LoggerFactory.getLogger(ThreadController.class);

    private final IThreadService threadService;

    @Autowired
    public ThreadController(IThreadService threadService) {
        this.threadService = threadService;
    }

    @GetMapping(value = "/get-user/{id}")
    public String getUser(@PathVariable int id) {
        // 放入上下文
        HystrixThreadLocal.threadLocal.set("userId:" + id);
        // 利用 RequestContextHolder
        RequestContextHolder.currentRequestAttributes().setAttribute("userId", "userId:" + id, RequestAttributes.SCOPE_REQUEST);
        LOGGER.info("================================== Controller ==================================");
        LOGGER.info(" ThreadService, Current thread id : {}", Thread.currentThread().getId());
        LOGGER.info(" ThreadService, HystrixThreadLocal: {}", HystrixThreadLocal.threadLocal.get());
        LOGGER.info(" ThreadService, RequestContextHolder: {}", RequestContextHolder.currentRequestAttributes().getAttribute("userId", RequestAttributes.SCOPE_REQUEST));
        return threadService.getUser(id);
    }

}
```

### 验证

访问 http://localhost:8989/get-user/1 ，查看控制台输出



```objectivec
================================== Controller ==================================
ThreadService, Current thread id : 37
ThreadService, HystrixThreadLocal: userId:1
ThreadService, RequestContextHolder: userId:1
================================= Service ==================================
ThreadService, Current thread id : 37
ThreadService, HystrixThreadLocal: userId:1
ThreadService, RequestContextHolder: userId:1
```

可以看到， thread id 是一样的，userid 也是一样的，这说明了此时是使用一个线程进行调用。

## Hystrix 分线程调用

在 Service 的实现类上增加 @HystrixCommand 注解，重启再次访问，查看可控制台输出如下：



```objectivec
================================== Controller ==================================
ThreadService, Current thread id : 36
ThreadService, HystrixThreadLocal: userId:1
ThreadService, RequestContextHolder: userId:1
================================= Service ==================================
ThreadService, Current thread id : 60
ThreadService, HystrixThreadLocal: userId:1
```

并在 `ThreadService, RequestContextHolder` 的位置抛出如下异常：



```css
java.lang.IllegalStateException: No thread-bound request found: Are you referring to request attributes outside of an actual web request, or processing a request outside of the originally receiving thread? If you are actually operating within a web request and still receive this message, your code is probably running outside of DispatcherServlet: In this case, use RequestContextListener or RequestContextFilter to expose the current request.
    at org.springframework.web.context.request.RequestContextHolder.currentRequestAttributes(RequestContextHolder.java:131) ~[spring-web-5.1.2.RELEASE.jar:5.1.2.RELEASE]
    at com.laiyy.gitee.hystrix.thread.springcloudhystrixthread.service.ThreadServiceImpl.getUser(ThreadServiceImpl.java:36) ~[classes/:na]
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_171]
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_171]
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_171]
    at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_171]
    at com.netflix.hystrix.contrib.javanica.command.MethodExecutionAction.execute(MethodExecutionAction.java:116) ~[hystrix-javanica-1.5.12.jar:1.5.12]
    at com.netflix.hystrix.contrib.javanica.command.MethodExecutionAction.executeWithArgs(MethodExecutionAction.java:93) ~[hystrix-javanica-1.5.12.jar:1.5.12]
    ...
```

可以看到，在 controller 中，线程id是 36，在 service 中，线程id是 60，线程id不一致，可以证明线程隔离已经生效了。此时 controller 调用 @HystrixCommand 标注的方法时，是分线程处理的；RequestContextHolder 中报错：没有线程变量绑定。

## 解决办法

解决办法有两种：修改 Hystrix 隔离策略，使用信号量即可；使用 HystrixConcurrencyStrategy(官方推荐)

使用 HystrixConcurrencyStrategy 实现 wrapCallable 方法，对于依赖的 ThreadLocal 状态以实现应用程序功能的系统至关重要。

### 官方文档

wrapCallable 方法源码：



```java
/**
 * Provides an opportunity to wrap/decorate a {@code Callable<T>} before execution.
 * <p>
 * This can be used to inject additional behavior such as copying of thread state (such as {@link ThreadLocal}).
 * <p>
 * <b>Default Implementation</b>
 * <p>
 * Pass-thru that does no wrapping.
 * 
 * @param callable
 *            {@code Callable<T>} to be executed via a {@link ThreadPoolExecutor}
 * @return {@code Callable<T>} either as a pass-thru or wrapping the one given
 */
public <T> Callable<T> wrapCallable(Callable<T> callable) {
    return callable;
}
```

通过重写这个方法，实现想要封装的线程参数方法。
 可以看到，返回值是一个 Callable，所以需要首先实现一个 Callable 类，在该类中，在执行请求之前，包装 HystrixThreadCallable 对象，传递 `RequestContextHolder`、`HystrixThreadLocal` 类，将需要的对象信息设置进去，这样在下一个线程中就可以拿到了。

### 具体实现



```java
// Callable
public class HystrixThreadCallable<S> implements Callable<S> {

    private final RequestAttributes requestAttributes;
    private final Callable<S> callable;
    private String params;

    public HystrixThreadCallable(RequestAttributes requestAttributes, Callable<S> callable, String params) {
        this.requestAttributes = requestAttributes;
        this.callable = callable;
        this.params = params;
    }

    @Override
    public S call() throws Exception {
        try {
            // 在执行请求之前，包装请求参数
            RequestContextHolder.setRequestAttributes(requestAttributes);
            HystrixThreadLocal.threadLocal.set(params);
            // 执行具体请求
            return callable.call();
        } finally {
            RequestContextHolder.resetRequestAttributes();
            HystrixThreadLocal.threadLocal.remove();
        }
    }
}


// HystrixConcurrencyStrategy
public class SpringCloudHystrixConcurrencyStrategy extends HystrixConcurrencyStrategy {

    private HystrixConcurrencyStrategy hystrixConcurrencyStrategy;

    @Override
    public <T> Callable<T> wrapCallable(Callable<T> callable) {
        // 传入需要传递到分线程的参数
        return new HystrixThreadCallable<>(RequestContextHolder.currentRequestAttributes(), callable, HystrixThreadLocal.threadLocal.get());
    }

    public SpringCloudHystrixConcurrencyStrategy() {
        init();
    }

    private void init() {
        try {
            this.hystrixConcurrencyStrategy = HystrixPlugins.getInstance().getConcurrencyStrategy();

            if (this.hystrixConcurrencyStrategy instanceof SpringCloudHystrixConcurrencyStrategy){
                return;
            }

            HystrixCommandExecutionHook commandExecutionHook = HystrixPlugins.getInstance().getCommandExecutionHook();
            HystrixEventNotifier eventNotifier = HystrixPlugins.getInstance().getEventNotifier();
            HystrixMetricsPublisher metricsPublisher = HystrixPlugins.getInstance().getMetricsPublisher();
            HystrixPropertiesStrategy propertiesStrategy = HystrixPlugins.getInstance().getPropertiesStrategy();

            HystrixPlugins.reset();

            HystrixPlugins.getInstance().registerConcurrencyStrategy(this);
            HystrixPlugins.getInstance().registerCommandExecutionHook(commandExecutionHook);
            HystrixPlugins.getInstance().registerEventNotifier(eventNotifier);
            HystrixPlugins.getInstance().registerMetricsPublisher(metricsPublisher);
            HystrixPlugins.getInstance().registerPropertiesStrategy(propertiesStrategy);
        }catch (Exception e){
            throw e;
        }
    }

    @Override
    public ThreadPoolExecutor getThreadPool(HystrixThreadPoolKey threadPoolKey, HystrixProperty<Integer> corePoolSize, HystrixProperty<Integer> maximumPoolSize, HystrixProperty<Integer> keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) {
        return this.hystrixConcurrencyStrategy.getThreadPool(threadPoolKey, corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue);
    }

    @Override
    public ThreadPoolExecutor getThreadPool(HystrixThreadPoolKey threadPoolKey, HystrixThreadPoolProperties threadPoolProperties) {
        return this.hystrixConcurrencyStrategy.getThreadPool(threadPoolKey, threadPoolProperties);
    }

    @Override
    public BlockingQueue<Runnable> getBlockingQueue(int maxQueueSize) {
        return this.hystrixConcurrencyStrategy.getBlockingQueue(maxQueueSize);
    }

    @Override
    public <T> HystrixRequestVariable<T> getRequestVariable(HystrixRequestVariableLifecycle<T> rv) {
        return this.hystrixConcurrencyStrategy.getRequestVariable(rv);
    }
}
```

## 验证

重启后访问： http://localhost:8989/get-user/1 ，查看控制台输出



```objectivec
================================= Controller ==================================
ThreadService, Current thread id : 39
ThreadService, HystrixThreadLocal: userId:1
ThreadService, RequestContextHolder: userId:1
================================= Service ==================================
ThreadService, Current thread id : 73
ThreadService, HystrixThreadLocal: userId:1
ThreadService, RequestContextHolder: userId:1
```

可以看到，线程id不一样，但是 RequestContextHolder、HystrixThreadLocal 的值都可以在分线程中拿到了。

## 注意点

在 `SpringCloudHystrixConcurrencyStrategy#init()` 中，`HystrixPlugins.getInstance().registerConcurrencyStrategy()` 方法，只能被调用一次，否则会出现错误。
 源码解释：



```java
/**
 * Register a {@link HystrixConcurrencyStrategy} implementation as a global override of any injected or default implementations.
 * 
 * @param impl
 *            {@link HystrixConcurrencyStrategy} implementation
 * @throws IllegalStateException
 * // 如果多次调用或在初始化默认值之后（如果在尝试注册之前发生了使用） 会抛出异常
 *             if called more than once or after the default was initialized (if usage occurs before trying to register)
 */
public void registerConcurrencyStrategy(HystrixConcurrencyStrategy impl) {
    if (!concurrencyStrategy.compareAndSet(null, impl)) {
        throw new IllegalStateException("Another strategy was already registered.");
    }
}
```

------

# Hystrix 命令注解

## HystrixCommand

作用：封装执行的代码，具有故障延迟容错、断路器、统计等功能，是阻塞命令，可以与 Observable 公用

com.netflix.hystrix.HystrixCommand 类



```java
/**
 * Used to wrap code that will execute potentially risky functionality (typically meaning a service call over the network)
 * with fault and latency tolerance, statistics and performance metrics capture, circuit breaker and bulkhead functionality.
 * This command is essentially a blocking command but provides an Observable facade if used with observe()
 * 
 * @param <R>
 *            the return type
 * 
 * @ThreadSafe
 */
public abstract class HystrixCommand<R> extends AbstractCommand<R> implements HystrixExecutable<R>, HystrixInvokableInfo<R>, HystrixObservable<R> {
}
```

用于包装将执行潜在风险功能的代码（通常意味着通过网络进行服务调用）具有故障和延迟限，统计和性能指标捕获，断路器和隔板功能。
 该命令本质上是一个阻塞命令，但如果与observe（）一起使用，则提供一个Observable外观

## HystrixObservableCommand



```java
/**
 * Used to wrap code that will execute potentially risky functionality (typically meaning a service call over the network)
 * with fault and latency tolerance, statistics and performance metrics capture, circuit breaker and bulkhead functionality.
 * This command should be used for a purely non-blocking call pattern. The caller of this command will be subscribed to the Observable<R> returned by the run() method.
 * 
 * @param <R>
 *            the return type
 * 
 * @ThreadSafe
 */
public abstract class HystrixObservableCommand<R> extends AbstractCommand<R> implements HystrixObservable<R>, HystrixInvokableInfo<R> {
}
```

用于包装将执行潜在风险功能的代码（通常意味着通过网络进行服务调用）具有故障和延迟限，统计和性能指标捕获，断路器和隔板功能。
 此命令应该用于纯粹的非阻塞调用模式。 此命令的调用者将订阅run（）方法返回的Observable <R>。

## 区别

- HystrixCommand 默认是阻塞的可以提供同步、异步两种方式；HystrixObservableCommand 是非阻塞的，只能提供异步的方式
- HystrixCommand 的方法是 run；HystrixObservableCommand 的方法是 construct
- HystrixCommand 一个实例一次只能发一条数据，HystrixObservableCommand 可以发送多条数据

HystrixCommand 注解：

![image-20210206120325153](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210206120325153.png)

HystrixCommand 注解

- commandKey：全局唯一标识符，如果不设置，默认是方法名
- defaultFallback：默认 fallback 方法，不能有入参，返回值和方法保持一致，比 fallbackMethod 的优先级低
- fallbackMethod：指定处理回退逻辑的方法，必须和 HystrixCommand 标注的方法在通一个类里，参数、返回值要保持一致
- ignoreExceptions：定义不希望哪些异常被 fallback，而是直接抛出
- commandProperties：配置命名属性，如隔离策略
- threadPoolProperties：配置线程池相关属性
- groupKey：全局唯一分组名称，内部会根据这个值展示统计数、仪表盘等，默认使的线程划分是根据组名称进行的，一般会在创建 HystrixCommand 时指定组来实现默认的线程池划分
- threadPoolKey：对服务的线程池信息进行设置，用于 HystrixThreadPool 监控、metrics、缓存等。

# 参考

https://www.jianshu.com/p/cde6c96e6fdd

https://www.jianshu.com/p/834766fd3a08

https://www.jianshu.com/p/05857f30ec51

https://www.jianshu.com/p/5b02adc23763