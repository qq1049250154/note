Zuul 是由 Netflix 孵化的一个致力于“网关”的解决方案的开源组件。Zuul 在动态路由、监控、弹性、服务治理、安全等方面有着举足轻重的作用。
 Zuul 底层为 Servlet，本质组件是一系列 Filter 构成的责任链。

**Zuul 具备的功能**

- 认证、鉴权
- 压力控制
- 金丝雀测试（灰度发布）
- 动态路由
- 负载削减
- 静态响应处理
- 主动流量控制

# Zuul 入门案例

## Zuul Server

***Server源码：https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-zuul/spring-cloud-zuul-simple\***
 ***Client源码：https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-zuul/spring-cloud-zuul-provider-service-simple\***

### pom、yml



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
</dependencies>
```



```yml
spring:
  application:
    name: spring-cloud-zuul-simple
server:
  port: 8989
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    instance-id: ${spring.application.name}:${server.port}
    prefer-ip-address: true
zuul:
  routes:  # zuul 路由配置，map 结构
    spring-cloud-provider-service-simple: # 针对哪个服务进行路由
      path: /provider/**  # 路由匹配什么规则。 当前配置为 provider 开头的请求路由到 provider-service 上
      # serviceId: spring-cloud-provider-service-simple # 路由到哪个 serviceId 上（即哪个服务），可不设置
```

### 启动类



```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableZuulProxy        // 开启 Zuul 代理
public class SpringCloudZuulSimpleApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudZuulSimpleApplication.class, args);
    }

}
```

## 验证

先请求 provider-service： http://localhost:8081/get-result

![image-20210206120429422](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210206120429422.png)

Zuul Provider Service

再请求 zuul server： http://localhost:8989/provider/get-result

![image-20210206120438397](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210206120438397.png)

Zuul Server Provider



可以看到，响应结果一致，但通过 `Zuul Server` 的请求路径多了 `/provider`，由此验证 zuul server 路由代理成功

------

# 典型配置

在上例中，路由规则的配置



```yml
zuul:
  routes:  
    spring-cloud-provider-service-simple: 
      path: /provider/**  
      serviceId: spring-cloud-provider-service-simple 
```

实际上，可以将这个配置进行简化

## 指定路由的简化



```yml
zuul:
  routes:  
    spring-cloud-provider-service-simple: /provider/**  
```

## 默认简化

默认简化可以不指定路由规则：



```yml
zuul:
  routes:  
    spring-cloud-provider-service-simple: 
```

此时简化配置，相当于：



```yml
zuul:
  routes:  
    spring-cloud-provider-service-simple: 
      path: /spring-cloud-provider-service-simple/**
      serviceId: spring-cloud-provider-service-simple
```

## 多实例路由

一般情况下，一个服务会有多个实例，此时需要对这个服务进行负载均衡。默认情况下，Zuul 会使用 Eureka 中集成的基本负载均衡功能（轮询）。

如果需要使用 Ribbon 的负载均衡功能，有两种方式：

### Ribbon 脱离 Eureka 使用

需要在 `routes` 配置中指定 `serviceId`，这个操作需要禁止 Ribbon 使用 Eureka。
 ***此方式必须指定 serviceId\***



```yml
zuul:
  routes:
    spring-cloud-provider-service-simple: # 服务名称，需要和下方配置一致
      path: /ribbon-route/**
      serviceId: spring-cloud-provider-service-simple   # serviceId，需要和下方配置一致
ribbon:
  eureka:
    enabled: false    # 禁用掉 Eureka

spring-cloud-provider-service-simple:     # 服务名称，需要和上方配置一致
  ribbon:
    NIWSServerListClassName: com.netflix.loadbalancer.ConfigurationBasedServerList  # 设置 ServerList 的配置
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule    # 设置负载均衡策略
    listOfServers: http://localhost:8080,http://localhost:8081    # 负载的 server 列表
```

### Ribbon 不脱离 Eureka 使用

直接使用 ribbon 路由配置即可
 ***此方式可以不指定 serviceId\***



```yml
zuul:
  routes:
    spring-cloud-provider-service-simple:
      path: /ribbon-route/**

spring-cloud-provider-service-simple:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

## Zuul 本地跳转

如果在 zuul 中做一些逻辑处理，在访问某个接口时，跳转到 zuul 中的这个方法上来处理，就需要用到 zuul 本地跳转



```yml
zuul:
  routes:
    spring-cloud-provider-service-simple:
      path: /provider/**  # 只有访问 /provider 的时候才会 forward，但凡后面多一个路径就不行了。。。 为啥。。。
      url: forward:/client
```

此时，访问：http://localhost:8989/client ，可以访问到，访问 http://localhost:8989/provider ，也能访问到，如果访问 http://localhost:8989/provider/get-result ，理论上应该也能跳转到 /client，但是实际上会报 404 错误



```json
{
    "timestamp": "2019-02-15T06:50:12.565+0000",
    "status": 404,
    "error": "Not Found",
    "message": "No message available",
    "path": "/provider/get-result"
}
```

如果去掉 `url: forward:/client`，再访问 http://localhost:8989/provider/get-result ，结果正常：



```json

```

## Zuul 相同路径加载规则



```yml
zuul:
  routes:
    spring-cloud-provider-service-simple-a:
      path: /provider/**
      serviceId: spring-cloud-provider-service-simple-a
    spring-cloud-provider-service-simple-b:
      path: /provider/**
      serviceId: spring-cloud-provider-service-simple-b
```

可以发现，/provider/** 匹配了2个 serviceId，这个匹配结果只会路由到最后一个服务上。即：/provider/** 只会被路由到 simple-b 服务上。
 yml 解释器在工作时，如果同一个映射路径对应了多个服务，按照加载顺序，后面的规则会把前面的规则覆盖掉。

## 路由通配符

| 规则 |            解释            |                示例                |
| :--: | :------------------------: | :--------------------------------: |
| /**  | 匹配任意数据量的路径与字符 |    /client/aa，/client/aa/bb/cc    |
|  /*  |     匹配任意数量的字符     | /client/aa，/client/aaaaaaaaaaaaaa |
|  /?  |        匹配单个字符        |  /client/a，/client/b，/client/c   |

------

# 功能配置

## 路由配置

在配置路由规则时，可以配置一个统一的前缀



```yml
zuul:
  routes:
    spring-cloud-provider-service-simple:
      path: /provider/**
      serviceId: spring-cloud-provider-service-simple
  prefix: /api
```

访问 http://localhost:8989/provider/get-result



```json
{
    "timestamp": "2019-02-15T07:16:46.625+0000",
    "status": 404,
    "error": "Not Found",
    "message": "No message available",
    "path": "/provider/get-result"
}
```

访问 http://localhost:8989/api/provider/get-result



```kotlin
this is provider service! this port is: 8081
```

这样的设置，会将每个访问访问前都加上 prefix 前缀，但是实际上访问的是 `path` 配置的路径。
 如果某个服务不需要前缀，访问路径就是 prefix + path，则只需要在对应的服务配置设置 `stripPrefix: false` 即可



```yml
zuul:
  routes:
    spring-cloud-provider-service-simple:
      path: /provider/**
      serviceId: spring-cloud-provider-service-simple
      stripPrefix: false
  prefix: /api
```

此时访问： http://localhost:8989/pre/provider/get-result ，返回值为：



```json
{
    "timestamp": "2019-02-15T07:24:33.271+0000",
    "status": 404,
    "error": "Not Found",
    "message": "No message available",
    "path": "/pre/provider/get-result"
}
```

对比两个 404 错误，可以看到，同样是访问 /pre/provider/get-result，没有设置 `stripPrefix: false` 时，path 为 `/provider/get-result`，设置 `stripPrefix: false` 时，path 为 `/pre/provider/get-result`。即：设置 `stripPrefix: false` 时，请求路径和实际路径是一致的。

## 服务屏蔽、路径屏蔽

为了避免某些服务、路径被侵入，可以将其屏蔽掉



```yml
zuul:
  routes:
    spring-cloud-provider-service-simple:
      path: /provider/**
      serviceId: spring-cloud-provider-service-simple
  ignored-services: spring-cloud-provider-service-simple    # 此配置会在 zuul 路由时，忽略掉该服务
  ignored-patterns: /**/get-result/**   # 此配置会在 zuul 路由时，忽略掉可以匹配的路径
  prefix: /pre
```

## 敏感头信息

正常访问时，provider-service 接收到的 headers 为：



```undefined
cache-control: no-cache
user-agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36
postman-token: 43dbfb6e-f529-543e-99d9-5d0a06bf79e6
accept: */*
accept-encoding: gzip, deflate, br
accept-language: zh-CN,zh;q=0.9
x-forwarded-host: localhost:8989
x-forwarded-proto: http
x-forwarded-prefix: /provider
x-forwarded-port: 8989
x-forwarded-for: 0:0:0:0:0:0:0:1
content-length: 0
host: 10.10.10.141:8081
connection: Keep-Alive
```

设置敏感头：



```yml
zuul:
  routes:
    spring-cloud-provider-service-simple:
      path: /provider/**
      serviceId: spring-cloud-provider-service-simple
      sensitiveHeaders: postman-token,x-forwarded-for,Cookie
```

此时再次访问，获取 headers



```undefined
cache-control: no-cache
user-agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36
accept: */*
accept-encoding: gzip, deflate, br
accept-language: zh-CN,zh;q=0.9
x-forwarded-host: localhost:8989
x-forwarded-proto: http
x-forwarded-prefix: /provider
x-forwarded-port: 8989
content-length: 0
host: 10.10.10.141:8081
connection: Keep-Alive
```

对比发现，`sensitiveHeaders` 配置的 headers 在 provider-service 中已经接收不到了。
 默认情况下，`sensitiveHeaders` 会忽略三个 header：`Cookie`、`Set-Cookie`、`Authorization`

# Zuul Filter

## Zuul Filter 的特点

- Filter 类型：Filter 类型决定了当前的 Filter 在整个 Filter 链中的执行顺序。
- Filter 执行顺序：同一种类型的 Filter 通过 filterOrder() 来设置执行顺序
- Filter 执行条件：Filter 执行所需的标准、条件
- Filter 执行效果：符合某个条件，产生的执行结果

Zuul 内部提供了一个动态读取、编译、运行这些 Filter 的机制。Filter 之间不直接通信，在请求线程中会通过 RequestContext 共享状态，内部使用 ThreadLocal 实现，也可以在 Filter 之间使用 ThreadLocal 收集自己需要的状态、数据

Zuul Filter 的执行逻辑源码在 `com.netflix.zuul.http.ZuulServlet` 中



```java
public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
    try {
        this.init((HttpServletRequest)servletRequest, (HttpServletResponse)servletResponse);
        RequestContext context = RequestContext.getCurrentContext();        // 通过 RequestContext 获取共享状态
        context.setZuulEngineRan();

        try {
            this.preRoute();        // 执行请求之前的操作
        } catch (ZuulException var13) {
            this.error(var13);      // 出现错误的操作
            this.postRoute();
            return;
        }

        try {
            this.route();       // 路由操作
        } catch (ZuulException var12) {
            this.error(var12);
            this.postRoute();
            return;
        }

        try {
            this.postRoute();       // 请求操作
        } catch (ZuulException var11) {
            this.error(var11);
        }
    } catch (Throwable var14) {
        this.error(new ZuulException(var14, 500, "UNHANDLED_EXCEPTION_" + var14.getClass().getName()));
    } finally {
        RequestContext.getCurrentContext().unset();
    }
}
```

## Zuul 生命周期

Zuul 官方文档中，生命周期图。



![image-20210206120455318](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210206120455318.png)

Zuul Life Cycle

但官方文档的生命周期图不太准确。

- 在 postRoute 执行之前，即 postFilter 执行之前，如果没有出现过错误，会调用 error 方法，并调用 this.error(new ZuulException) 打印堆栈信息
- 在 postRoute 执行之前就已经报错，会调用 error 方法，再调用 postRoute，但是之后会直接 return，不会调用 this.error(new ZuulException) 打印堆栈信息

由此可以看出，整个 Filter 调用链的重点可能是 postFilter 也可能是 errorFilter

![image-20210206120530324](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210206120530324.png)

Zuul Life Cycle

pre、route 出现错误后，进入 error，再进入 post，再返回
 pre、route 没有出现错误，进入 post，如果出现错误，再进入 error，再返回

- pre：在 Zuul 按照规则路由到下级服务之前执行。如果需要对请求进行预处理，如：鉴权、限流等，都需要在此 Filter 实现
- route：Zuul 路由动作的执行者，是 Http Client、Ribbon 构建和发送原始 HTTP 请求的地方
- post：源服务返回结果或异常信息发生后执行，如果需要对返回值信息做处理，需要实现此类 Filter
- error：整个生命周期发生异常，都会进入 error Filter，可做全局异常处理。

Filter 之间，通过 `com.netflix.zuul.context.RequestContext` 类进行通信，内部采用 ThreadLocal 保存每个请求的一些信息，包括：请求路由、错误信息、HttpServletRequest、HTTPServletResponse，扩展了 ConcurrentHashMap，目的是为了在处理过程中保存任何形式的信息

------

# Zuul 原生 Filter

整合 `spring-boot-starter-actuator` 后，查看 idea 控制台 endpoints 栏的 mappings，可以看到多了几个 Actuator 端点

## routes 端点

访问 [http://localhost:8989/actuator/routes](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A8989%2Factuator%2Froutes) 可以查看当前 zuul server 映射了几个路径、服务



```json
{
    "/provider/**": "spring-cloud-provider-service-simple",
    "/spring-cloud-provider-service-simple/**": "spring-cloud-provider-service-simple"
}
```

访问 [http://localhost:8989/actuator/routes/details](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A8989%2Factuator%2Froutes%2Fdetails) 可以查看具体的映射信息



```json
{
    "/provider/**": {
        "id": "spring-cloud-provider-service-simple",       // serviceId
        "fullPath": "/provider/**",     // 映射 path
        "location": "spring-cloud-provider-service-simple",     // 服务名称，实际上也是 serviceId
        "path": "/**",      // 实际访问路径 
        "prefix": "/provider",      // 访问前缀
        "retryable": false,     // 是否开启重试
        "customSensitiveHeaders": false,        // 是否自定义了敏感 header
        "prefixStripped": true      // 是否去掉前缀（如果为 false，则实际访问时需要加 前缀，且实际请求的访问路径也会加上前缀）
    },
    "/spring-cloud-provider-service-simple/**": {
        "id": "spring-cloud-provider-service-simple",
        "fullPath": "/spring-cloud-provider-service-simple/**",
        "location": "spring-cloud-provider-service-simple",
        "path": "/**",
        "prefix": "/spring-cloud-provider-service-simple",
        "retryable": false,
        "customSensitiveHeaders": false,
        "prefixStripped": true
    }
}
```

## filters 端点

访问 [http://localhost:8989/actuator/filters](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A8989%2Factuator%2Ffilters) ，返回当前 zuul 的所有 filters

![image-20210206120546381](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210206120546381.png)

Zuul Filters

## 内置 Filters

|          名称           | 类型  | 顺序 |                             描述                             |
| :---------------------: | :---: | :--: | :----------------------------------------------------------: |
| ServletDetectionFilter  |  pre  |  -3  |           通过 Spring Dispatcher 检查请求是否通过            |
| Servlet30WrapperFilter  |  pre  |  -2  |   适配 HttpServletRequest 为 Servlet30RequestWrapper 对象    |
|  FormBodyWrapperFilter  |  pre  |  -1  |            解析表单数据，并为下游请求进行重新编码            |
|       DebugFilter       |  pre  |  1   |                        Debug 路由标识                        |
|   PreDecorationFilter   |  pre  |  5   |         处理请求上下文供后续使用，设置下游相关头信息         |
|   RibbonRoutingFilter   | route |  10  |       使用 Ribbon、Hystrix、嵌入式 HTTP 客户端发送请求       |
| SimpleHostRoutingFilter | route | 100  |               使用 Apache Httpclient 发送请求                |
|    SendForwardFilter    | route | 500  |                    使用 Servlet 转发请求                     |
|   SendResponseFilter    | post  | 1000 |                 将代理请求的响应写入当前响应                 |
|     SendErrorFilter     | error |  0   | 如果 RequestContext.getThrowable() 不为空，则转发到 error.path 哦诶之的路径 |

如果使用 `@EnableZuulServer` 注解，将减少 `PreDecorationFilter`、`RibbonRoutingFilter`、`SimpleHostRoutingFilter`

如果要替换到某个原生的 Filter，可以自实现一个和原生 Filter 名称、类型一样的 Filter，并替换。或者禁用掉某个filter，并自实现一个新的。
 禁用语法： `zuul.{SimpleClassName}.{filterType}.disable=true`，如 `zuul.SendErrorFilter.error.disable=true`

------

# 多级业务处理

在 Zuul Filter 链体系中，可以把一组业务逻辑细分，然后封装到一个个紧密结合的 Filter，设置处理顺序，组成一组 Filter 链。

## 自定义实现 Filter

在 Zuul 中实现自定义 Filter，继承 `ZuulFilter` 类即可，ZuulFilter 是一个抽象类，需要实现以下几个方法

- String filterType：使用返回值设定 Filter 类型，可以设置为 `pre`、`route`、`post`、`error`
- int filterOrder：使用返回值设置 Filter 执行次序
- boolean shouldFilter：使用返回值设定该 Filter 是否执行，可以作为开关来使用
- Object run：Filter 的核心执行逻辑



```java
// 自定义 ZuulFilter
public class FirstPreFilter extends ZuulFilter {
    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        System.out.println("自定义 Filter，类型为 pre！");
        return null;
    }
}


// 注入 Spring 容器
@Bean
public FirstPreFilter firstPreFilter(){
    return new FirstPreFilter();
}
```

此时访问 [http://localhost:8989/provider/get-result](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A8989%2Fprovider%2Fget-result) ，查看控制台：



```php
Initializing Servlet 'dispatcherServlet'
Completed initialization in 0 ms
自定义 Filter，类型为 pre！
Flipping property: spring-cloud-provider-service-simple.ribbon.ActiveConnectionsLimit to use NEXT property: niws.loadbalancer.availabilityFilteringRule.activeConnectionsLimit = 2147483647
Shutdown hook installed for: NFLoadBalancer-PingTimer-spring-cloud-provider-service-simple
```

## 业务处理

使用 SecondFilter 验证是否传入参数 a，ThirdPreFilter 验证是否传入参数 b，在 PostFilter 统一处理返回内容。

***SecondPreFilter\***



```java
public class SecondPreFilter extends ZuulFilter {
    private static final Logger LOGGER = LoggerFactory.getLogger(SecondPreFilter.class);
    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return 2;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        LOGGER.info(">>>>>>>>>>>>> SecondPreFilter ！ <<<<<<<<<<<<<<<<");
        // 获取上下文
        RequestContext requestContext = RequestContext.getCurrentContext();
        // 从上下文获取 request
        HttpServletRequest request = requestContext.getRequest();
        // 从 request 获取参数 a
        String a = request.getParameter("a");
        // 如果参数 a 为空
        if (StringUtils.isBlank(a)) {
            LOGGER.info(">>>>>>>>>>>>>>>> 参数 a 为空！ <<<<<<<<<<<<<<<<");
            // 禁止路由，禁止访问下游服务
            requestContext.setSendZuulResponse(false);
            // 设置 responseBody，供 postFilter 使用
            requestContext.setResponseBody("{\"status\": 500, \"message\": \"参数 a 为空！\"}");
            // 用于下游 Filter 判断是否执行
            requestContext.set("logic-is-success", false);
            // Filter 结束
            return null;
        }
        requestContext.set("logic-is-success", true);
        return null;
    }
}
```

***ThirdPreFilter\***



```java
public class ThirdPreFilter extends ZuulFilter {

    private static final Logger LOGGER = LoggerFactory.getLogger(ThirdPreFilter.class);

    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return 3;
    }

    @Override
    public boolean shouldFilter() {
        RequestContext context = RequestContext.getCurrentContext();
        // 获取上下文中的 logic-is-success 中的值，用于判断当前 filter 是否执行
        return (boolean) context.get("logic-is-success");
    }

    @Override
    public Object run() throws ZuulException {
        LOGGER.info(">>>>>>>>>>>>> ThirdPreFilter ！ <<<<<<<<<<<<<<<<");
        // 获取上下文
        RequestContext requestContext = RequestContext.getCurrentContext();
        // 从上下文获取 request
        HttpServletRequest request = requestContext.getRequest();
        // 从 request 获取参数 a
        String a = request.getParameter("b");
        // 如果参数 a 为空
        if (StringUtils.isBlank(a)) {
            LOGGER.info(">>>>>>>>>>>>>>>> 参数 b 为空！ <<<<<<<<<<<<<<<<");
            // 禁止路由，禁止访问下游服务
            requestContext.setSendZuulResponse(false);
            // 设置 responseBody，供 postFilter 使用
            requestContext.setResponseBody("{\"status\": 500, \"message\": \"参数 b 为空！\"}");
            // 用于下游 Filter 判断是否执行
            requestContext.set("logic-is-success", false);
            // Filter 结束
            return null;
        }
        requestContext.set("logic-is-success", true);
        return null;
    }
}
```

***PostFilter\***



```java
public class PostFilter extends ZuulFilter {

    private static final Logger LOGGER = LoggerFactory.getLogger(PostFilter.class);

    @Override
    public String filterType() {
        return FilterConstants.POST_TYPE;
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        LOGGER.info(">>>>>>>>>>>>>>>>>>> Post Filter! <<<<<<<<<<<<<<<<");
        RequestContext context = RequestContext.getCurrentContext();
        // 处理返回中文乱码
        context.getResponse().setCharacterEncoding("UTF-8");
        // 获取上下文保存的 responseBody
        String responseBody = context.getResponseBody();
        // 如果 responseBody 不为空，则证明流程中有异常发生
        if (StringUtils.isNotBlank(responseBody)) {
            // 设置返回状态码
            context.setResponseStatusCode(500);
            // 替换响应报文
            context.setResponseBody(responseBody);
        }
        return null;
    }
}
```

访问 [http://localhost:8989/provider/add](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A8989%2Fprovider%2Fadd) 、[http://localhost:8989/provider/add?a=1](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A8989%2Fprovider%2Fadd%3Fa%3D1) 、[http://localhost:8989/provider/add?a=1&b=1](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A8989%2Fprovider%2Fadd%3Fa%3D1%26b%3D1) ，查看控制台

控制台：



```css
2019-02-18 14:09:44.890  INFO 5800 --- [nio-8989-exec-7] c.l.g.z.s.filter.FirstPreFilter          : >>>>>>>>>>>>>>>>> 自定义 Filter，类型为 pre！ <<<<<<<<<<<<<<<<<<
2019-02-18 14:09:44.890  INFO 5800 --- [nio-8989-exec-7] c.l.g.z.s.filter.SecondPreFilter         : >>>>>>>>>>>>> SecondPreFilter ！ <<<<<<<<<<<<<<<<
2019-02-18 14:09:44.890  INFO 5800 --- [nio-8989-exec-7] c.l.g.z.s.filter.SecondPreFilter         : >>>>>>>>>>>>>>>> 参数 a 为空！ <<<<<<<<<<<<<<<<
2019-02-18 14:09:44.890  INFO 5800 --- [nio-8989-exec-7] c.l.g.z.s.filter.PostFilter              : >>>>>>>>>>>>>>>>>>> Post Filter! <<<<<<<<<<<<<<<<
```



```css
2019-02-18 14:10:13.004  INFO 5800 --- [nio-8989-exec-5] c.l.g.z.s.filter.FirstPreFilter          : >>>>>>>>>>>>>>>>> 自定义 Filter，类型为 pre！ <<<<<<<<<<<<<<<<<<
2019-02-18 14:10:13.004  INFO 5800 --- [nio-8989-exec-5] c.l.g.z.s.filter.SecondPreFilter         : >>>>>>>>>>>>> SecondPreFilter ！ <<<<<<<<<<<<<<<<
2019-02-18 14:10:13.004  INFO 5800 --- [nio-8989-exec-5] c.l.g.z.s.filter.ThirdPreFilter          : >>>>>>>>>>>>> ThirdPreFilter ！ <<<<<<<<<<<<<<<<
2019-02-18 14:10:13.004  INFO 5800 --- [nio-8989-exec-5] c.l.g.z.s.filter.ThirdPreFilter          : >>>>>>>>>>>>>>>> 参数 b 为空！ <<<<<<<<<<<<<<<<
2019-02-18 14:10:13.005  INFO 5800 --- [nio-8989-exec-5] c.l.g.z.s.filter.PostFilter              : >>>>>>>>>>>>>>>>>>> Post Filter! <<<<<<<<<<<<<<<<
```



```css
2019-02-18 14:10:28.488  INFO 5800 --- [nio-8989-exec-9] c.l.g.z.s.filter.FirstPreFilter          : >>>>>>>>>>>>>>>>> 自定义 Filter，类型为 pre！ <<<<<<<<<<<<<<<<<<
2019-02-18 14:10:28.488  INFO 5800 --- [nio-8989-exec-9] c.l.g.z.s.filter.SecondPreFilter         : >>>>>>>>>>>>> SecondPreFilter ！ <<<<<<<<<<<<<<<<
2019-02-18 14:10:28.488  INFO 5800 --- [nio-8989-exec-9] c.l.g.z.s.filter.ThirdPreFilter          : >>>>>>>>>>>>> ThirdPreFilter ！ <<<<<<<<<<<<<<<<
2019-02-18 14:10:28.500  INFO 5800 --- [nio-8989-exec-9] c.l.g.z.s.filter.PostFilter              : >>>>>>>>>>>>>>>>>>> Post Filter! <<<<<<<<<<<<<<<<
```

返回值：



```json
{"status": 500, "message": "参数 a 为空！"}
```



```json
{"status": 500, "message": "参数 b 为空！"}
```



```csharp
result is : a + b = 2
```

由此验证自定义 Zuul Filter 成功。

# 限流算法

限流算法一般分为 `漏桶`、`令牌桶` 两种。

## 漏桶

漏桶的圆形是一个底部有漏孔的桶，桶的上方有一个入水口，水不断流进桶内，桶下方的漏孔会以一个相对恒定的速度漏水，在`入大于出`的情况下，桶在一段时间内就会被装满，这时候多余的水就会溢出；而在`入小于出`的情况下，漏桶起不到任何作用。

当请求或者具有一定体量的数据进入系统时，在漏桶作用下，流量被整形，不能满足要求的部分被削减掉，漏桶算法能强制限定流量速度。溢出的流量可以被再次利用起来，并非完全丢弃，可以把溢出的流量收集到一个队列中，做流量排队，尽量合理利用所有资。

![image-20210206120603449](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210206120603449.png)

leaky-bucket

## 令牌桶

令牌桶与漏桶的区别是，桶里放的是令牌而不是流量，令牌以一个恒定的速度被加入桶内，可以积压，可以溢出。当流量涌入时，量化请求用于获取令牌，如果取到令牌则方形，同时桶内丢掉这个令牌；如果取不到令牌，则请求被丢弃。
 由于桶内可以存一定量的令牌，那么就可能会解决一定程度的流量突发。这个也是漏桶与令牌桶的适用场景不同之处。

![image-20210206120611194](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210206120611194.png)

token-bucket

------

# 限流实例

在 Zuul 中实现限流，最简单的方式是使用 Filter 加上相关的限流算法，其中可能会考虑到 Zuul 多节点部署。因为算法的原因，这是需要一个 K/V 存储工具（Redis等）。

[`spring-cloud-zuul-ratelimit`](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fmarcosbarbero%2Fspring-cloud-zuul-ratelimit) 是一个针对 Zuul 的限流库
 限流粒度的策略：

- user：认证用户名或匿名，针对某用户粒度进行限流
- origin：客户机 ip，针对请求客户机 ip 粒度进行限流
- url：特定 url，针对某个请求 url 粒度进行限流
- serviceId：特定服务，针对某个服务 id 粒度进行限流

限流粒度临时变量存储方式：

- IN_MEMORY：基于本地内存，底层是 ConcurrentHashMap
- REDIS：基于 Redis K/V 存储
- CONSUL：基于 Consul K/V 存储
- JPA：基于 SpringData JPA，数据库存储
- BUKET4J：使用 Java 编写的基于令牌桶算法的限流库，四种模：`JCache`、`Hazelcast`、`Apache Ignite`、`Inifinispan`，后面三种支持异步

***源码：[https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-zuul/spring-cloud-zuul-ratelimit](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Flaiyy0728%2Fspring-cloud%2Ftree%2Fmaster%2Fspring-cloud-zuul%2Fspring-cloud-zuul-ratelimit)\***

## Zuul Server



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
    </dependency>
    <dependency>
        <groupId>com.marcosbarbero.cloud</groupId>
        <artifactId>spring-cloud-zuul-ratelimit</artifactId>
        <version>2.0.6.RELEASE</version>
    </dependency>
</dependencies>
```



```yml
server:
  port: 5555
spring:
  application:
    name: spring-cloud-ratelimit-zuul-server
eureka:
  instance:
    instance-id: ${spring.application.name}:${server.port}
    prefer-ip-address: true
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
zuul:
  routes:
    spring-cloud-ratelimit-provider-service:
      path: /provider/**
      serviceId: spring-cloud-ratelimit-provider-service
  ratelimit:
    key-prefix: springcloud # 按粒度拆分的临时变量 key 的前缀
    enabled: true # 启用开关
    repository: in_memory # key 的存储类型，默认是 in_memory
    behind-proxy: true # 表示代理之后
    default-policy:
      limit: 2 # 在一个单位时间内的请求数量
      quota: 1 # 在一个单位时间内的请求时间限制
      refresh-interval: 3 # 单位时间窗口
      type: 
        - user # 可指定用户粒度
        - origin # 可指定客户端地址粒度
        - url # 可指定 url 粒度
```

## Provider



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
</dependencies>
```



```yml
spring:
  application:
    name: spring-cloud-ratelimit-provider-service
server:
  port: 7070
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
    instance-id: ${spring.application.name}:${server.port}
```

## 验证

快速访问几次 [http://localhost:5555/provider/get-result](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A5555%2Fprovider%2Fget-result) ，返回值如下：



```json
{
    "timestamp": "2019-02-20T06:52:51.220+0000",
    "status": 429,
    "error": "Too Many Requests",
    "message": "429"
}
```

控制台打印异常如下：



```php
com.netflix.zuul.exception.ZuulException: 429
    at com.marcosbarbero.cloud.autoconfigure.zuul.ratelimit.support.RateLimitExceededException.<init>(RateLimitExceededException.java:13) ~[spring-cloud-zuul-ratelimit-core-2.0.6.RELEASE.jar:2.0.6.RELEASE]
    at com.marcosbarbero.cloud.autoconfigure.zuul.ratelimit.filters.RateLimitPreFilter.lambda$run$0(RateLimitPreFilter.java:106) ~[spring-cloud-zuul-ratelimit-core-2.0.6.RELEASE.jar:2.0.6.RELEASE]
    at java.util.ArrayList.forEach(ArrayList.java:1257) ~[na:1.8.0_171]
    at com.marcosbarbero.cloud.autoconfigure.zuul.ratelimit.filters.RateLimitPreFilter.run(RateLimitPreFilter.java:79) ~[spring-cloud-zuul-ratelimit-core-2.0.6.RELEASE.jar:2.0.6.RELEASE]
    at com.netflix.zuul.ZuulFilter.runFilter(ZuulFilter.java:117) ~[zuul-core-1.3.1.jar:1.3.1]
    at com.netflix.zuul.FilterProcessor.processZuulFilter(FilterProcessor.java:193) ~[zuul-core-1.3.1.jar:1.3.1]
    at com.netflix.zuul.FilterProcessor.runFilters(FilterProcessor.java:157) ~[zuul-core-1.3.1.jar:1.3.1]
    ...
```

正常访问结果如下：



```bash
zuul rate limit result !
```

------

# 动态路由

之前配置路由映射规则的方式，为“静态路由”。如果在迭代过程中，可能需要动态将路由映射规则写入内存。在“静态路由”配置中，需要重启 Zuul 应用。
 不需要重启 Zuul，又能修改映射规则的方式，称为“动态路由”。

- SpringCloud Config + Bus，动态刷新配置文件。好处是不用 Zuul 维护映射规则，可以随时修改，随时生效。缺点是需要单独集成一些使用并不频繁的组件。SpringCloud Config 没有可视化界面，维护也麻烦
- 重写 Zuul 配置读取方式，采用事件刷新机制，从数据库读取路由映射规则。此方式基于数据库，可轻松实现管理页面，灵活度高。

# 动态路由实现原理

![image-20210206120621457](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210206120621457.png)

动态路由原理核心类依赖图

## DiscoveryClientRouteLocator



```java
public class DiscoveryClientRouteLocator extends SimpleRouteLocator implements RefreshableRouteLocator {
    
    // 省略其他方法

    // 路由
    protected LinkedHashMap<String, ZuulRoute> locateRoutes() {
        // 省略方法实现
    }

    // 刷新
    public void refresh() {
        this.doRefresh();
    }
}
```

`locateRoutes` 方法继承自 `SimpleRouteLocator` 类，并重写规则，该方法主要的功能就是将配置文件中的映射规则信息包装成 `LinkedHashMap<String, ZuulRoute>`，键为路径 path，值 ZuulRoute 是配置文件的封装类。之前的映射配置信息就是使用 ZuulRoute 封装的。
 `refresh` 实现自 RefreshableRouteLocator 接口，添加刷新功能必须实现此方法，`doRefresh` 方法来自 `SimpleRouteLocator` 类

## SimpleRouteLocator

`SimpleRouteLocator` 是 `DiscoveryClientRouteLocator` 的父类，此类基本实现了 RouteLocator 接口，对读取配置文件信息做一些处理，提供方法 `doRefresh`、`locateRoutes` 供子类实现刷新策略与映射规则加载策略



```java
/**
 * Calculate all the routes and set up a cache for the values. Subclasses can call
 * this method if they need to implement {@link RefreshableRouteLocator}.
 */
protected void doRefresh() {
    this.routes.set(locateRoutes());
}

/**
 * Compute a map of path pattern to route. The default is just a static map from the
 * {@link ZuulProperties}, but subclasses can add dynamic calculations.
 */
protected Map<String, ZuulRoute> locateRoutes() {
    LinkedHashMap<String, ZuulRoute> routesMap = new LinkedHashMap<>();
    for (ZuulRoute route : this.properties.getRoutes().values()) {
        routesMap.put(route.getPath(), route);
    }
    return routesMap;
}
```

这两个方法都是 protectted 修饰，是为了让子类不用维护此类一些成员变量就能实现刷新或读取路由的功能。从注释上可以看到，调用 `doRedresh` 方法需要实现 `RefreshableRouteLocator`；`locateRoutes` 默认是一个静态的映射读取方法，如果需要动态记载映射，需要子类重写此方法。

## ZuulServerAutoConfiguration

ZuulServerAutoConfiguration 是 Spring Cloud Zuul 的配置类，主要目的是注册各种过滤器、监听器以及其他功能。Zuul 在注册中心新增服务后刷新监听器也是在这个类中注册的，底层是 Spring 的 ApplicationListener



```java
@Configuration
@EnableConfigurationProperties({ ZuulProperties.class })
@ConditionalOnClass(ZuulServlet.class)
@ConditionalOnBean(ZuulServerMarkerConfiguration.Marker.class)
public class ZuulServerAutoConfiguration {

    // 省略其他功能注册

    // Zuul 刷新监听器
    private static class ZuulRefreshListener
            implements ApplicationListener<ApplicationEvent> {

        @Autowired
        private ZuulHandlerMapping zuulHandlerMapping;

        private HeartbeatMonitor heartbeatMonitor = new HeartbeatMonitor();

        @Override
        public void onApplicationEvent(ApplicationEvent event) {
            if (event instanceof ContextRefreshedEvent
                    || event instanceof RefreshScopeRefreshedEvent
                    || event instanceof RoutesRefreshedEvent
                    || event instanceof InstanceRegisteredEvent) {
                reset();
            }
            else if (event instanceof ParentHeartbeatEvent) {
                ParentHeartbeatEvent e = (ParentHeartbeatEvent) event;
                resetIfNeeded(e.getValue());
            }
            else if (event instanceof HeartbeatEvent) {
                HeartbeatEvent e = (HeartbeatEvent) event;
                resetIfNeeded(e.getValue());
            }
        }

        private void resetIfNeeded(Object value) {
            if (this.heartbeatMonitor.update(value)) {
                reset();
            }
        }

        private void reset() {
            this.zuulHandlerMapping.setDirty(true);
        }
    }
}
```

其中，由方法 `onApplicationEvent`可知，Zuul 会接收 4 种事件通知 `ContextRefreshedEvent`、`RefreshScopeRefreshedEvent`、`RoutesRefreshedEvent`、`InstanceRegisteredEvent`，这四种通知都会去刷新路由映射配置信息，此外，心跳续约监视器 `HeartbeatEvent` 也会触发这个动作

## ZuulHandlerMapping

在 `ZuulServerAutoConfiguration#ZuulRefreshListener` 中，注入了 `ZuulHandlerMapping`，此类是将本地配置的映射关系，映射到远程的过程控制器



```java
/**
 * MVC HandlerMapping that maps incoming request paths to remote services.
 */
public class ZuulHandlerMapping extends AbstractUrlHandlerMapping {

    // 省略其他配置

    private volatile boolean dirty = true;

    public void setDirty(boolean dirty) {
        this.dirty = dirty;
        if (this.routeLocator instanceof RefreshableRouteLocator) {
            ((RefreshableRouteLocator) this.routeLocator).refresh();
        }
    }
}
```

`dirty` 属性很重要，它是用来控制当前是否需要重新加载映射配置信息的标记，在 Zuul 每次进行路由操作的时候都会检查这个值。如果为 true，则会触发配置信息的重新加载，同时再将其审核制为 false。由 `setDirty` 方法体可知，启动刷新动作必须实现 `RefreshableRouteLocator`，否则会出现类转换异常。

# 动态路由实战

***源码：[https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-zuul/spring-cloud-dynamic-route-zuul-server](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Flaiyy0728%2Fspring-cloud%2Ftree%2Fmaster%2Fspring-cloud-zuul%2Fspring-cloud-dynamic-route-zuul-server)\***

## Zuul Server



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
</dependencies>
```



```yml
spring:
  application:
    name: spring-cloud-dynamic-route-zuul-server
  datasource:
    url: jdbc:mysql://localhost:3306/springcloud?useUnicode=true&characterEncoding=utf-8&serverTimezone=Hongkong
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: 123456
  jpa:
    hibernate:
      ddl-auto: update
server:
  port: 5555
eureka:
  instance:
    instance-id: ${spring.application.name}:${server.port}
    prefer-ip-address: true
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```



```java
// zuul 路由实体
@Entity
@Table(name = "zuul_route")
@Data
public class ZuulRouteEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private int id;

    private String path;

    @Column(name = "service_id")
    private String serviceId;
    private String url;

    @Column(name = "strip_prefix")
    private boolean stripPrefix = true;
    private boolean retryable;
    private boolean enabled;
    private String description;
}


// dao
public interface ZuulPropertiesDao extends JpaRepository<ZuulRouteEntity, Integer> {

    @Query("FROM ZuulRouteEntity WHERE enabled = TRUE")
    List<ZuulRouteEntity> findAllByParams();

}


// 动态路由实现
public class DynamicZuulRouteLocator extends SimpleRouteLocator implements RefreshableRouteLocator {

    @Autowired
    private ZuulProperties zuulProperties;

    @Autowired
    private ZuulPropertiesDao zuulPropertiesDao;

    public DynamicZuulRouteLocator(String servletPath, ZuulProperties properties) {
        super(servletPath, properties);
        this.zuulProperties = properties;
    }

    @Override
    public void refresh() {
        doRefresh();
    }

    @Override
    protected Map<String, ZuulProperties.ZuulRoute> locateRoutes() {
        Map<String, ZuulProperties.ZuulRoute> routeMap = new LinkedHashMap<>();
        routeMap.putAll(super.locateRoutes());
        routeMap.putAll(getProperties());
        Map<String, ZuulProperties.ZuulRoute> values = new LinkedHashMap<>();
        routeMap.forEach((path, zuulRoute) -> {
            path = path.startsWith("/") ? path : "/" + path;
            if (StringUtils.hasText(this.zuulProperties.getPrefix())) {
                path = this.zuulProperties.getPrefix() + path;
                path = path.startsWith("/") ? path : "/" + path;
            }
            values.put(path, zuulRoute);
        });
        return values;
    }

    private Map<String, ZuulProperties.ZuulRoute> getProperties() {
        Map<String, ZuulProperties.ZuulRoute> routeMap = new LinkedHashMap<>();
        List<ZuulRouteEntity> list = zuulPropertiesDao.findAllByParams();
        list.forEach(entity -> {
            if (org.apache.commons.lang.StringUtils.isBlank(entity.getPath())) {
                return;
            }
            ZuulProperties.ZuulRoute route = new ZuulProperties.ZuulRoute();
            BeanUtils.copyProperties(entity, route);
            route.setId(String.valueOf(entity.getId()));
            routeMap.put(route.getPath(), route);
        });
        return routeMap;
    }
}


// 注册到 Spring
@Configuration
public class DynamicZuulConfig {

    private final ZuulProperties zuulProperties;

    private final ServerProperties serverProperties;

    @Autowired
    public DynamicZuulConfig(ZuulProperties zuulProperties, ServerProperties serverProperties) {
        this.zuulProperties = zuulProperties;
        this.serverProperties = serverProperties;
    }

    @Bean
    public DynamicZuulRouteLocator dynamicZuulRouteLocator(){
        return new DynamicZuulRouteLocator(serverProperties.getServlet().getContextPath(), zuulProperties);
    }
}
```

## 验证

在数据库中增加三条数据



```sql
INSERT INTO `springcloud`.`zuul_route` (`id`, `description`, `enabled`, `path`, `retryable`, `service_id`, `strip_prefix`, `url`) VALUES ('1', '重定向到百度', '\1', '/baidu/**', '\0', NULL, '\1', 'http://www.baidu.com');
INSERT INTO `springcloud`.`zuul_route` (`id`, `description`, `enabled`, `path`, `retryable`, `service_id`, `strip_prefix`, `url`) VALUES ('2', 'url', '\1', '/client/**', '\0', NULL, '\1', 'http://localhost:8081');
INSERT INTO `springcloud`.`zuul_route` (`id`, `description`, `enabled`, `path`, `retryable`, `service_id`, `strip_prefix`, `url`) VALUES ('3', 'serviceId', '\1', '/client-1/**', '\0', 'client-a', '\1', NULL);
```

访问 [http://localhost:5555/baidu/get-result](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A5555%2Fbaidu%2Fget-result) 、[http://localhost:5555/client/get-result](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A5555%2Fclient%2Fget-result) 、[http://localhost:5555/client-a/get-result](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A5555%2Fclient-a%2Fget-result)

![image-20210206120700961](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210206120700961.png)

重定向百度





```kotlin
this is provider service! this port is: 8081 headers: [user-agent]: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36; [cache-control]: no-cache; [postman-token]: a35929aa-9bcf-f5e4-b79a-0e684b1611ed; [token]: E477CA7B8E7CDCDCE3331742544DE9F1; [content-type]: application/x-www-form-urlencoded;charset=UTF-8; [accept]: */*; [accept-encoding]: gzip, deflate, br; [accept-language]: zh-CN,zh;q=0.9; [x-forwarded-host]: localhost:5555; [x-forwarded-proto]: http; [x-forwarded-prefix]: /client; [x-forwarded-port]: 5555; [x-forwarded-for]: 0:0:0:0:0:0:0:1; [host]: localhost:8081; [connection]: Keep-Alive; 
```



```json
{
    "timestamp": "2019-02-21T03:15:18.997+0000",
    "status": 404,
    "error": "Not Found",
    "message": "No message available",
    "path": "/client-a/get-result"
}
```

由此可以证明，动态路由配置成功。

------

# 灰度发布

灰度发布是指在系统迭代新功能时的一种平滑过渡的上线发布方式。灰度发布是在原有的系统基础上，额外增加一个新版本，在这个新版本中，有需要验证的功能修改或添加，使用负载均衡器，引入一小部分流量到新版本应用中，如果这个新版本没有出现差错，再平滑地把线上系统或服务一步步替换成新版本，直至全部替换上线结束。

## 灰度发布实现方式

灰度发布可以使用元数据来实现，元数据有两种

- 标准元数据：标准元数据是服务的各种注册信息，如：ip、端口、健康信息、续约信息等，存储于专门为服务开辟的注册表中，用于其他组件取用以实现整个微服务生态
- 自定义元数据：自定义元数据是使用 `eureka.instance.metadata-map.{key}={value}` 配置，其内部实际上是维护了一个 map 来保存子弹元数据信息，可配置再远端服务，随服务一并注册保存在 Eureka 注册表，对微服务生态没有影响。

## 灰度发布实战

***源码：[https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-zuul/spring-cloud-zuul-metadata](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Flaiyy0728%2Fspring-cloud%2Ftree%2Fmaster%2Fspring-cloud-zuul%2Fspring-cloud-zuul-metadata)\***

### provider



```yml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
    instance-id: ${spring.application.name}:${server.port}

spring:
  application:
    name: spring-cloud-metadata-provider-service

---
server:
  port: 7070
spring:
  profiles: node1   # 设定 profile，可以使用 mvn spring-boor:run -Dspring.profiles.active=node1 启动，或者在启动类使用   SpringApplication.run(SpringCloudMetadataProviderServiceApplication.class, "--spring.profiles.active=node1"); 启动
eureka:
  instance:
    metadata-map:
      host-mark: running # 设定当前节点的 metadata，zuul server 使用这个标注来进行路由转发
---
spring:
  profiles: node2
server:
  port: 7071
eureka:
  instance:
    metadata-map:
      host-mark: running
---
spring:
  profiles: node3
server:
  port: 7072
eureka:
  instance:
    metadata-map:
      host-mark: gray  # 当前节点是灰度节点
```



```java
@SpringBootApplication
@EnableDiscoveryClient
@RestController
public class SpringCloudMetadataProviderServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudMetadataProviderServiceApplication.class, args);
        // SpringApplication.run(SpringCloudMetadataProviderServiceApplication.class, "--Dspring.profiles.active=node1"); // 非 maven 启动
    }

    @Value("${server.port}")
    private int port;

    @GetMapping(value = "/get-result")
    public String getResult(){
        return "metadata provider service result, port: " + port;
    }
}
```

### Zuul Server



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
    </dependency>

    <!-- 实现通过 metadata 进行灰度路由 -->
    <dependency>
        <groupId>io.jmnarloch</groupId>
        <artifactId>ribbon-discovery-filter-spring-cloud-starter</artifactId>
        <version>2.1.0</version>
    </dependency>
</dependencies>
```



```yml
spring:
  application:
    name: spring-cloud-metadata-zuul-server
server:
  port: 5555
eureka:
  instance:
    instance-id: ${spring.application.name}:${server.port}
    prefer-ip-address: true
  client:
    service-url:
      defautlZone: http://localhost:8761/eureka/
zuul:
  routes:
    spring-cloud-metadata-provider-service:
      path: /provider/**
      serviceId: spring-cloud-metadata-provider-service
management:
  endpoints:
    web:
      exposure:
        include: '*'
  endpoint:
    health:
      show-details: always
```



```java
// 实现灰度的 Filter
public class GrayFilter extends ZuulFilter {

    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return FilterConstants.PRE_DECORATION_FILTER_ORDER - 1;
    }

    @Override
    public boolean shouldFilter() {
        RequestContext context = RequestContext.getCurrentContext();
        return !context.containsKey(FilterConstants.FORWARD_TO_KEY) && !context.containsKey(FilterConstants.SERVICE_ID_KEY);
    }

    @Override
    public Object run() throws ZuulException {
        HttpServletRequest request = RequestContext.getCurrentContext().getRequest();
        String grayMark = request.getHeader("gray_mark");
        if (StringUtils.isNotBlank(grayMark) && StringUtils.equals("enable", grayMark)) {
            RibbonFilterContextHolder.getCurrentContext().add("host-mark", "gray");
        } else {
            RibbonFilterContextHolder.getCurrentContext().add("host-mark", "running");
        }
        return null;
    }
}


@SpringBootApplication
@EnableDiscoveryClient
@EnableZuulProxy
public class SpringCloudMetadataZuulServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudMetadataZuulServerApplication.class, args);
    }

    @Bean
    public GrayFilter grayFilter(){
        return new GrayFilter();
    }

}
```

### 验证

正常访问： [http://localhost:5555/provider/get-result](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A5555%2Fprovider%2Fget-result) ，查看返回值， port 在 7070 和 7071 之间轮询。

![image-20210206120711207](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210206120711207.png)

gray running



启用 gray_mark header，再次访问，发现 port 始终都是 7072，由此验证灰度成功

![image-20210206120833020](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210206120833020.png)

gray

# Zuul 应用优化

Zuul 是建立在 Servlet 上的同步阻塞架构，所有在处理逻辑上面是和线程密不可分，每一次请求都需要在线程池获取一个线程来维护 I/O 操作，路由转发的时候又需要从 http 客户端获取线程来维持连接，这样会导致一个组件占用两个线程资源的情况。所以在 Zuul 的使用中，对这部分的优化很有必要。

Zuul 的优化分为以下几个类型：

- 容器优化：内置容器 tomcat 与 undertow 的比较与参数设置
- 组件优化：内部集成的组件优化，如 Hystrix 线程隔离、Ribbon、HttpClient、OkHttp 选择等
- JVM 参数优化：适用于网关应用的 JVM 参数建议
- 内部优化：内部原生参数，内部源码，重写等

## 容器优化

把 tomcat 替换为 undertow。`undertow` 翻译为“暗流”，是一个轻量级、高性能容器。`undertow` 提供阻塞或基于 XNIO 的非阻塞机制，包大小不足 1M，内嵌模式运行时的堆内存占用只有 4M。



```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```



```yml
server:
  undertow:
    io-threads: 10
    worker-threads: 10
    direct-buffers: true
    buffer-size: 1024 # 字节数
```

|               配置项               |                            默认值                            |                             说明                             |
| :--------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|     server.undertow.io-threads     |  `Math.max(Runtime.getRuntime().availableProcessors(), 2)`   | 设置 IO 线程数，它主要执行非阻塞的任务，它们负责多个连接，默认设置每个 CPU 核心有一个线程。不要设置太大，否则启动项目会报错：打开文件数过多 |
|   server.undertow.worker-threads   |                        io-threads * 8                        | 阻塞任务线程数，当执行类型 Servlet 请求阻塞 IO 操作，undertow 会从这个线程池中取得线程。值设置取决于系统线程执行任务的阻塞系统，默认是 IO 线程数 * 8 |
|   server.undertow.direct-buffers   | 取决于 JVM 最大可用内存大小`Runtime.getRuntime().maxMemory()`，小于 64MB 默认为 false，其余默认为 true |           是否分配直接内存（NIO 直接分配的堆外内存           |
|    server.undertow.buffer-size     | 最大可用内存 <64MB：512 字节；64MB< 最大可用内存 <128MB：1024 字节；128MB < 最大可用内存：1024*16 - 20 字节 | 每块 buffer 的空间大小，空间越小利用越充分，设置太大会影响其他应用 |
| server.undertow.buffers-per-region |      最大可用内存 <128MB：10；128MB < 最大可用内存：20       | 每个区域分配的 buffer 数量，pool 大小是 buffer-size * buffer-per-region |

## 组件优化

### Hystrix

在 Zuul 中默认集成了 Hystrix 熔断器，使得网关应用具有弹性、容错的能力。但是如果使用默认配置，可能会遇到问题。如：第一次请求失败。这是因为第一次请求的时候，zuul 内部需要初始化很多信息，十分耗时。而 hystrix 默认超时时间是一秒，可能会不够。

解决方式：

- 加大超时时间



```yml
hystrix:
 command:
   default: 
     execution:
       isolation:
         thread:
           timeoutInMilliseconds: 5000 
```

- 禁用 hystrix 超时



```yml
hystrix:
 command:
   default: 
     execution:
       timeout:
         enabled: false    
```

Zuul 中关于 Hystrix 的配置还有一个很重要的点：`Hystrix 线程隔离策略`。

|            |  线程池模式(THREAD)  | 信号量模式(SEMAPHORE) |
| :--------: | :------------------: | :-------------------: |
|  官方推荐  |          是          |          否           |
|    线程    |    与请求线程分离    |    与请求线程公用     |
|    开销    | 上下文切换频繁，较大 |         较小          |
|    异步    |         支持         |        不支持         |
| 应对并发量 |          大          |          小           |
|  适用场景  |       外网交互       |       内网交互        |

如果应用需要与外网交互，由于网络开销比较大、请求比较耗时，选用线程隔离，可以保证有剩余容器（tomcat 等）线程可用，不会由于外部原因使线程一直在阻塞或等待状态，可以快速返回失败
 如果应用不需要与外网交互，并且体量较大，使用信号量隔离，这类应用响应通常非常快，不会占用容器线程太长时间，使用信号量线程上下文就会成为一个瓶颈，可以减少线程切换的开销，提高应用运转的效率，也可以气到对请求进行全局限流的作用。

### Ribbon



```yml
ribbon:
  ConnectTimeout: 3000
  ReadTimeout: 60000
  MaxAutoRetries: 1 # 对第一次请求的发我的重试次数
  MaxAutoRetriesNextServer: 1 # 要重试的下一个服务的最大数量（不包括第一个服务）
  OkToRetryOnAllOperations: true
```

`ConnectTimeout`、`ReadTimeout` 是当前 HTTP 客户端使用 HttpClient 的时候生效的，这个超时时间最终会被设置到 HttpClient 中。在设置的时候要结合 Hystrix 超时时间综合考虑。设置太小会导致请求失败，设置太大会导致 Hystrix 熔断控制变差。

## JVM 参数优化

根据实际情况，调整 JVM 参数

## 内有优化

在官方文档中，zuul 部分将 `zuul.max.host.coonnections` 属性拆分成了 `zuul.host.maxTotalConnections`、`zuul.host.maxPerRouteConnections`，默认值分别为 200、20。
 需要注意：这个配置只在使用 HttpClient 时有效，使用 OkHttp 无效。

zuul 中还有一个超时时间，使用 serviceId 映射与 url 映射的设置是不一样的，如果使用 serviceId 映射，`ribbon.ReadTimeout` 与 `ribbon.SocketTimeout` 生效；如果使用 url 映射，`zuul.host.connect-timeout-millis` 与 `zuul.host.socket-timeout-millis` 生效

------

# Zuul 原理、核心

zuul 官方提供了一张架构图，很好的描述了 Zuul 工作原理

![image-20210206120846003](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210206120846003.png)

Zuul 架构图

Zuul Servlet 通过 RequestContext 通关着由许多 Filter 组成的核心组件，所有操作都与 Filter 息息相关。请求、ZuulServlet、Filter 共同构建器 Zuul 的运行时声明周期

![image-20210206120905811](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210206120905811.png)

zuul life cycle

Zuul 的请求来自于 DispatcherServlet，然后交给 ZuulHandlerMapping 处理初始化得来的路由定位器`RouteLocator`，为后续的请求分发做好准备，同时整合了基于事件从服务中心拉取服务列表的机制；
 进入 ZuulController，主要职责是初始化 ZuulServlet 以及集成 ServletWrappingController，通过重写 handleRequest 方法来将 ZuulServlet 引入声明周期，之后所有的请求都会经过 ZuulServlet；
 当请求进入 ZuulServlet 之后，第一次调用会初始化 ZuulRunner，非第一次调用就按照 Filter 链的 order 顺序执行；
 ZuulRunner 中将请求和响应初始化为 RequestContext，包装成 FilterProcessor 转换为为调用 preRoute、route、postRoute、error 方法；
 最后再 Filter 链中经过种种变换，得到预期结果。

## EnableZuulProxy、EnableZuulServer

对比 `EnableZuulProxy`、`EnableZuulServer`



```java
@EnableCircuitBreaker
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import(ZuulProxyMarkerConfiguration.class)
public @interface EnableZuulProxy {
}
```



```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({ZuulServerMarkerConfiguration.class})
public @interface EnableZuulServer {
}
```

这两个注解的区别在于 `@Import` 中的配置类不一样。查看两个配置类的源码：



```java
/**
 * Responsible for adding in a marker bean to trigger activation of 
 * {@link ZuulProxyAutoConfiguration}
 *
 * @author Biju Kunjummen
 */

@Configuration
public class ZuulProxyMarkerConfiguration {
    @Bean
    public Marker zuulProxyMarkerBean() {
        return new Marker();
    }

    class Marker {
    }
}
```



```java
/**
 * Responsible for adding in a marker bean to trigger activation of 
 * {@link ZuulServerAutoConfiguration}
 *
 * @author Biju Kunjummen
 */

@Configuration
public class ZuulServerMarkerConfiguration {
    @Bean
    public Marker zuulServerMarkerBean() {
        return new Marker();
    }

    class Marker {
    }
}
```

可以看到，这两个配置类的源码一致，区别在于类的注释上 `@link` 指向的自动装配类不一样，`ZuulProxyMarkerConfiguration` 对应的是 `ZuulProxyAutoConfiguration`；`ZuulServerMarkerConfiguration` 对应的是 `ZuulServerAutoConfiguration`

查看 `ZuulProxyAutoConfiguration` 和 `ZuulServerAutoConfiguration` 的类注解



```java
@Configuration
@Import({ RibbonCommandFactoryConfiguration.RestClientRibbonConfiguration.class,
        RibbonCommandFactoryConfiguration.OkHttpRibbonConfiguration.class,
        RibbonCommandFactoryConfiguration.HttpClientRibbonConfiguration.class,
        HttpClientConfiguration.class })
@ConditionalOnBean(ZuulProxyMarkerConfiguration.Marker.class)
public class ZuulProxyAutoConfiguration extends ZuulServerAutoConfiguratio
}
```



```java
@Configuration
@EnableConfigurationProperties({ ZuulProperties.class })
@ConditionalOnClass({ZuulServlet.class, ZuulServletFilter.class})
@ConditionalOnBean(ZuulServerMarkerConfiguration.Marker.class)
public class ZuulServerAutoConfiguration {
}
```

可以发现，这两个类是通过 `ZuulServerMarkerConfiguration`、`ZuulProxyMarkerConfiguration` 中 Marker 类是否存在，当做是否进行自动装配的开关。
 对比两个 `AutoCOnfiguration` 的具体源码实现，经过对比，可以分析出：
 `ZuulServerAutoConfiguration` 的功能是：

- 初始化配置加载器
- 初始化路由定位器
- 初始化路由映射器
- 初始化配置刷新监听器
- 初始化 ZuulServlet 加载器
- 初始化 ZuulController
- 初始化 Filter 执行解析器
- 初始化部分 Filter
- 初始化 Metrix 监控

`ZuulProxyAutoConfiguration` 的功能是：

- 初始化服务注册、发现监听器
- 初始化服务列表监听器
- 初始化 zuul 自定义的 endpoint
- 初始化一些 `ZuulServerAutoConfiguration` 中没有的 filter'
- 引入 http 客户端的两种方式：HttpClient、OkHttp

## Filter 链

### filter 装载

zuul 中的 Filter 必须经过初始化装载，才能在请求中发挥作用，其过程如下



![image-20210206120916967](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210206120916967.png)

zuul-filter-life-cycle

#### zuul filter 连初始化过程



```java
public class ZuulServerAutoConfiguration {

  // 省略其他代码

  @Configuration
  protected static class ZuulFilterConfiguration {
    @Autowired
    private Map<String, ZuulFilter> filters;
    @Bean
    public ZuulFilterInitializer zuulFilterInitializer(
      CounterFactory counterFactory, TracerFactory tracerFactory) {
      FilterLoader filterLoader = FilterLoader.getInstance();
      FilterRegistry filterRegistry = FilterRegistry.instance();
      return new ZuulFilterInitializer(this.filters, counterFactory, tracerFactory, filterLoader, filterRegistry);
    }
  }
}
```



```java
public class ZuulFilterInitializer {

  // 省略其他代码

  // @PostConstruct：表明在 Bean 初始化之前，就把 Filter 的信息保存到 FilterRegistry
    @PostConstruct
    public void contextInitialized() {
        log.info("Starting filter initializer");

        TracerFactory.initialize(tracerFactory);
        CounterFactory.initialize(counterFactory);

        for (Map.Entry<String, ZuulFilter> entry : this.filters.entrySet()) {
            filterRegistry.put(entry.getKey(), entry.getValue());
        }
    }

  // @PreDestroy：表明在 bean 销毁之前清空 filterRegistry 与 FilterLoader。filterloader 可以通过 filter 名、filter class、filter 类型来查询得到相应的 filter
    @PreDestroy
    public void contextDestroyed() {
        log.info("Stopping filter initializer");
        for (Map.Entry<String, ZuulFilter> entry : this.filters.entrySet()) {
            filterRegistry.remove(entry.getKey());
        }
        clearLoaderCache();

        TracerFactory.initialize(null);
        CounterFactory.initialize(null);
    }

  private void clearLoaderCache() {
        Field field = ReflectionUtils.findField(FilterLoader.class, "hashFiltersByType");
        ReflectionUtils.makeAccessible(field);
        @SuppressWarnings("rawtypes")
        Map cache = (Map) ReflectionUtils.getField(field, filterLoader);
        cache.clear();
    }
}
```

#### zuul filter 请求调用过



```java
public class ZuulServletFilter implements Filter {

    private ZuulRunner zuulRunner;

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

        String bufferReqsStr = filterConfig.getInitParameter("buffer-requests");
        boolean bufferReqs = bufferReqsStr != null && bufferReqsStr.equals("true") ? true : false;

        zuulRunner = new ZuulRunner(bufferReqs);
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        try {
            init((HttpServletRequest) servletRequest, (HttpServletResponse) servletResponse);
            try {
                preRouting();
            } catch (ZuulException e) {
                error(e);
                postRouting();
                return;
            }
            
            // Only forward onto to the chain if a zuul response is not being sent
            if (!RequestContext.getCurrentContext().sendZuulResponse()) {
                filterChain.doFilter(servletRequest, servletResponse);
                return;
            }
            
            try {
                routing();
            } catch (ZuulException e) {
                error(e);
                postRouting();
                return;
            }
            try {
                postRouting();
            } catch (ZuulException e) {
                error(e);
                return;
            }
        } catch (Throwable e) {
            error(new ZuulException(e, 500, "UNCAUGHT_EXCEPTION_FROM_FILTER_" + e.getClass().getName()));
        } finally {
            RequestContext.getCurrentContext().unset();
        }
    }

    void postRouting() throws ZuulException {
        zuulRunner.postRoute();
    }
    
    // 省略其他代码
}
```



```java
public class ZuulRunn{

  // 省略其他代码

  public void postRoute() throws ZuulException {
        FilterProcessor.getInstance().postRoute();
  }

  // 省略其他代码
}
```



```java
public class FilterProcessor {
    public void postRoute() throws ZuulException {
        try {
            runFilters("post");
        } catch (ZuulException e) {
            throw e;
        } catch (Throwable e) {
            throw new ZuulException(e, 500, "UNCAUGHT_EXCEPTION_IN_POST_FILTER_" + e.getClass().getName());
        }
    }

    public Object runFilters(String sType) throws Throwable {
        if (RequestContext.getCurrentContext().debugRouting()) {
            Debug.addRoutingDebug("Invoking {" + sType + "} type filters");
        }
        boolean bResult = false;
        List<ZuulFilter> list = FilterLoader.getInstance().getFiltersByType(sType);
        if (list != null) {
            for (int i = 0; i < list.size(); i++) {
                ZuulFilter zuulFilter = list.get(i);
                Object result = processZuulFilter(zuulFilter);
                if (result != null && result instanceof Boolean) {
                    bResult |= ((Boolean) result);
                }
            }
        }
        return bResult;
    }

    // 省略其他代码
}
```

## 核心路由的实现

Zuul 的路由有一个顶级接口 `RouteLocator`。所有关于路由的功能都是由此而来，其中定义了三个方法：获取忽略的 path 集合、获取路由列表、根据 path 获取路由信息。

![image-20210206120930255](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210206120930255.png)

Route 类图



`SimpleRouteLocator` 是一个基本实现，主要功能是对 ZuulServer 的配置文件中路由规则的维护，实现了 Ordered 接口，可以对定位器优先级进行设置。
 Spring 是一个大量使用策略模式的框架，在策略模式下，接口的实现类有一个优先级问题，Spring 通过 Ordered 接口实现优先级。

`ZuulProperties$ZuulRoute` 类就是维护路由规则的类，具体属性如下：



```java
public static class ZuulRoute {

        private String id;

        private String path;

        private String serviceId;

        private String url;

        private boolean stripPrefix = true;
    
        private Boolean retryable;

        private Set<String> sensitiveHeaders = new LinkedHashSet<>();

        private boolean customSensitiveHeaders = false;
}
```

`RefreshableRouteLocator` 扩展了 RouteLocator 接口，在 ZuulHandlerMapping 中才实质性生效：凡是实现了 RefreshableRouteLocator，都会被时间监听器所刷新：



```java
public void setDirty(boolean dirty) {
        this.dirty = dirty;
        if (this.routeLocator instanceof RefreshableRouteLocator) {
            ((RefreshableRouteLocator) this.routeLocator).refresh();
        }
    }
```

`DiscoveryClientRouteLocator` 实现了 RefreshableRouteLocator，扩展了 SimpleRouteLocator。其作用是整合配置文件与注册中心的路由信息。

`CompositeRouteLocator`，在 `ZuulServerAutoConfiguration` 中配置加载时，有一个很重要的注解：`@Primary`，表示所有的 `RouteLocator` 中，优先加载它，也就是说，所有的定位器都要在这里装配，可以看做其他路由定位器的处理器。
 zuul 通过它来将请求域路由规则进行关联，这个操作在 `ZuulHandlerMapping` 中：



```java
public class ZuulHandlerMapping extends AbstractUrlHandlerMapping {

  // 省略其他代码

    private final RouteLocator routeLocator;

    private final ZuulController zuul;

  public ZuulHandlerMapping(RouteLocator routeLocator, ZuulController zuul) {
        this.routeLocator = routeLocator;
        this.zuul = zuul;
        setOrder(-200);
    }

  @Override
    protected Object lookupHandler(String urlPath, HttpServletRequest request) throws Exception {
        if (this.errorController != null && urlPath.equals(this.errorController.getErrorPath())) {
            return null;
        }
        if (isIgnoredPath(urlPath, this.routeLocator.getIgnoredPaths())) return null;
        RequestContext ctx = RequestContext.getCurrentContext();
        if (ctx.containsKey("forward.to")) {
            return null;
        }
        if (this.dirty) {
            synchronized (this) {
                if (this.dirty) {
                    registerHandlers();
                    this.dirty = false;
                }
            }
        }
        return super.lookupHandler(urlPath, request);
    }

  private void registerHandlers() {
        Collection<Route> routes = this.routeLocator.getRoutes();
        if (routes.isEmpty()) {
            this.logger.warn("No routes found from RouteLocator");
        }
        else {
            for (Route route : routes) {
                registerHandler(route.getFullPath(), this.zuul);
            }
        }
    }

}
```

ZuulHandlerMapping 将映射规则交给 `ZuulController` 处理，而 ZuulController 又到 ZuulServlet 中处理，最后到达异域或源服务发送 http 请求的 route 类型的 filter 中，默认有三种发送 http 请求的 filter

- RibbonRoutingFilter：优先级 10，使用 Ribbon、Hystrix、嵌入式 HTTP 客户端发送请求
- SimpleHostRoutingFilter：优先级 100，室友 Apache HttpClient 发送请求
- SendForwardFilter：优先级 500，使用 Servlet 发送请求

# Zuul 文件上传

***源码：[https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-zuul/spring-cloud-zuul-file-upload](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Flaiyy0728%2Fspring-cloud%2Ftree%2Fmaster%2Fspring-cloud-zuul%2Fspring-cloud-zuul-file-upload)\***

## Zuul Server



```yml
server:
  port: 5555
spring:
  application:
    name: spring-cloud-zuul-file-upload
  servlet:
    multipart:
      enabled: true # 使用 http multipart 上传
      max-file-size: 100MB # 文件最大大小，默认 1M，不配置则为 -1
      max-request-size: 100MB # 请求最大大小，默认 10M，不配置为 -1
      file-size-threshold: 1MB # 当上传文件达到 1NB 时进行磁盘写入
      location: / # 上传的临时目录
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
    instance-id: ${spring.application.name}:${server.port}

hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 30000 # 超时时间 30 秒，防止大文件上传出现超时
ribbon:
  ConnectionTimeout: 3000 # Ribbon 链接超时时间
  ReadTimeout: 30000    # Ribbon 读超时时间
```



```java
@SpringBootApplication
@EnableZuulProxy
@EnableDiscoveryClient
@RestController
public class SpringCloudZuulFileUploadApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudZuulFileUploadApplication.class, args);
    }


    @PostMapping(value = "/upload")
    public String uploadFile(@RequestParam(value = "file") MultipartFile file) throws Exception {
        byte[] bytes = file.getBytes();
        File fileToSave = new File(file.getOriginalFilename());
        FileCopyUtils.copy(bytes, fileToSave);
        return fileToSave.getAbsolutePath();
    }

}
```

## 测试结果

![image-20210206120942163](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210206120942163.png)

Zuul file upload

## 注意事项

如果使用的 cloud 版本是 Finchley 之前的版本，在上传中文名称的文件时，会出现乱码的情况。解决办法：在调用接口上加上 `/zuul` 根节点。如： [http://localhost:5555/zuul/upload](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A5555%2Fzuul%2Fupload)

需要注意的是，在 @RequestMapping、@PostMapping 上不能加 `/zuul`，这个节点是 zuul 自带的。也就是说，即是在项目中没有 `/zuul` 开头的映射，使用 zuul 后都会加上 `/zuul` 根映射。

------

# Zuul 使用技巧

## Zuul 饥饿加载

Zuul 内部使用 Ribbon 远程调用，根据 Ribbon 的特性，第一次调用会去注册中心获取注册表，初始化 Ribbon 负载信息，这是一种懒加载策略，但是这个过程很耗时。为了避免这个问题，可以使用饥饿加载



```yml
zuul:
  ribbon:
    eager-load:
      enabled: true
```

## 修改请求体

***源码：[https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-zuul/spring-cloud-zuul-change-param](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Flaiyy0728%2Fspring-cloud%2Ftree%2Fmaster%2Fspring-cloud-zuul%2Fspring-cloud-zuul-change-param)\***

### zuul server



```java
@Configuration
public class ChangeParamZuulFilter extends ZuulFilter {
    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return FilterConstants.PRE_DECORATION_FILTER_ORDER + 1;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        RequestContext currentContext = RequestContext.getCurrentContext();
        Map<String, List<String>> requestQueryParams = currentContext.getRequestQueryParams();
        if (requestQueryParams == null) {
            requestQueryParams = Maps.newHashMap();
        }
        
        List<String> arrayList = Lists.newArrayList();
        // 增加一个参数
        arrayList.add("1111111");
        requestQueryParams.put("test", arrayList);
        currentContext.setRequestQueryParams(requestQueryParams);
        return null;
    }
}
```

### provider



```java
@RestController
public class TestController {
    
    @PostMapping("/change-params")
    public Map<String, Object> modifyRequestEntity (HttpServletRequest request) {
        Map<String, Object> bodyParams = new HashMap<>();
        Enumeration enu = request.getParameterNames();  
        while (enu.hasMoreElements()) {  
            String paraName = (String)enu.nextElement();  
            bodyParams.put(paraName, request.getParameter(paraName));
        }
        return bodyParams;
    }
}
```

### 验证

访问 [http://localhost:5555/provider/change-params](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A5555%2Fprovider%2Fchange-params)

![image-20210206120952385](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210206120952385.png)

change params

## zuul 中使用 OkHttp



```xml
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
</dependency>
```



```yml
ribbon:
  okhttp:
    enabled: true
  http:
    client:
      enabled: false
```

## zuul 重试



```xml
<!-- retry -->
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
</dependency>
```



```yml
spring:
  cloud:
    loadbalancer:
      retry:
        enabled: true

zuul:
  retryable: true

ribbon:
  ConnectTimeout: 3000
  ReadTimeout: 60000
  MaxAutoRetries: 1 # 对第一次请求的服务的重试次数
  MaxAutoRetriesNextServer: 1 # 要重试的下一个服务的最大数量（不包括第一个服务）
  OkToRetryOnAllOperations: true
```

## header 传递



```java
@Configuration
public class AddHeaderZuulFilter extends ZuulFilter {
    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return FilterConstants.PRE_DECORATION_FILTER_ORDER + 1;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        RequestContext currentContext = RequestContext.getCurrentContext();
        currentContext.addZuulRequestHeader("key", "value");
        return null;
    }
}
```

在下游 `spring-cloud-change-param-provider` 中查看 header 传递



```java
@RestController
public class TestController {

    @PostMapping("/change-params")
    public Map<String, Object> modifyRequestEntity (HttpServletRequest request) {
        Map<String, Object> bodyParams = new HashMap<>();
        Enumeration enu = request.getParameterNames();  
        while (enu.hasMoreElements()) {  
            String paraName = (String)enu.nextElement();  
            bodyParams.put(paraName, request.getParameter(paraName));
        }

        Enumeration<String> headerNames = request.getHeaderNames();
        while (headerNames.hasMoreElements()) {
            String header = headerNames.nextElement();
            String value = request.getHeader(header);
            System.out.println(header + " ---> " + value);
        }
        return bodyParams;
    }
}
```

访问 [http://localhost:5555/provider/change-params](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A5555%2Fprovider%2Fchange-params) ，查看控制台：



```rust
user-agent ---> Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36
cache-control ---> no-cache
origin ---> chrome-extension://fhbjgbiflinjbdggehcddcbncdddomop
postman-token ---> a99dcd1c-7c76-2d7e-8a4c-fa5c2bdb6222
accept ---> */*
accept-encoding ---> gzip, deflate, br
accept-language ---> zh-CN,zh;q=0.9
x-forwarded-host ---> localhost:5555
x-forwarded-proto ---> http
x-forwarded-prefix ---> /provider
x-forwarded-port ---> 5555
x-forwarded-for ---> 0:0:0:0:0:0:0:1
key ---> value          ---------------------- zuul server 中增加的 header
content-type ---> application/x-www-form-urlencoded;charset=UTF-8
content-length ---> 12
host ---> 10.10.10.141:7070
connection ---> Keep-Alive
```

## Zuul 整合 Swagger

### provider



```xml
<dependencies>
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
  </dependency>

  <dependency>
      <groupId>io.springfox</groupId>
      <artifactId>springfox-swagger2</artifactId>
      <version>2.7.0</version>
  </dependency>
</dependencies>
```



```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableSwagger2   // 这个注解必须加，否则解析不到
public class SpringCloudChangeParamProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudChangeParamProviderApplication.class, args);
    }

}
```

### Zuul Server



```xml
<!-- swagger -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
```



```java
/**
 * @author laiyy
 * @date 2019/2/25 15:24
 * @description
 */
@Configuration
@EnableSwagger2 // 这个注解必须加
public class Swagger2Configuration {

    private final ZuulProperties zuulProperties;

    @Autowired
    public Swagger2Configuration(ZuulProperties zuulProperties) {
        this.zuulProperties = zuulProperties;
    }

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo());
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder().title("spring cloud swagger 2")
                .description("spring cloud 整合 swagger2")
                .termsOfServiceUrl("")
                .contact(new Contact("laiyy", "laiyy0728@gmail.com", "laiyy0728@gmail.com")).version("1.0")
                .build();
    }



// 第一种配置方式
    @Primary
    @Bean
    public SwaggerResourcesProvider swaggerResourcesProvider() {
        return () -> {
            List<SwaggerResource> resources = new ArrayList<>();
            zuulProperties.getRoutes().values().stream()
                    .forEach(route -> resources.add(createResource(route.getServiceId(), route.getServiceId(), "2.0")));
            return resources;
        };
    }

    private SwaggerResource createResource(String name, String location, String version) {
        SwaggerResource swaggerResource = new SwaggerResource();
        swaggerResource.setName(name);
        swaggerResource.setLocation("/" + location + "/v2/api-docs");
        swaggerResource.setSwaggerVersion(version);
        return swaggerResource;
    }



// 第二种配置方式（推荐使用）
//    @Component
//    @Primary
//    public class ZuulSwaggerResourceProvider implements SwaggerResourcesProvider {
//
//        private final RouteLocator routeLocator;
//
//        @Autowired
//        public ZuulSwaggerResourceProvider(RouteLocator routeLocator) {
//            this.routeLocator = routeLocator;
//        }
//
//        @Override
//        public List<SwaggerResource> get() {
//            List<SwaggerResource> resources = Lists.newArrayList();
//            routeLocator.getRoutes().forEach(route -> {
//                resources.add(createResource(route.getId(), route.getFullPath().replace("**", "v2/api-docs")));
//            });
//            return resources;
//        }
//
//        private SwaggerResource createResource(String name, String location) {
//            SwaggerResource swaggerResource = new SwaggerResource();
//            swaggerResource.setName(name);
//            swaggerResource.setLocation(location);
////            swaggerResource.setLocation("/" + location + "/api-docs");
//            swaggerResource.setSwaggerVersion("2.0");
//            return swaggerResource;
//        }
//    }
}
```

### 验证

访问 [http://localhost:5555/swagger-ui.html](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A5555%2Fswagger-ui.html)

![image-20210206121007350](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210206121007350.png)

Zuul Swagger

# 参考

https://www.jianshu.com/p/a1059fd9d144

https://www.jianshu.com/p/e8126da2f4fd

https://www.jianshu.com/p/a895b23da09a

https://www.jianshu.com/p/5d2d5c752922

https://www.jianshu.com/p/10ac931202e3

https://www.jianshu.com/p/408d089c72af