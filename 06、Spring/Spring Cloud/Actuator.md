在 Hystrix Dashboard 中，使用了 /actuator/hystrix.stream，查看正在运行的项目的运行状态。 其中 `/actuator` 代表 SpringBoot 中的 Actuator 模块。该模版提供了很多生产级别的特性，如：监控、度量应用。Actuator 的特性可以通过众多的 REST 端点，远程 shell、JMX 获得。

***源码：https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-boot-actuator\***

# 常见 Actuator 端点

![](https://typoralim.oss-cn-beijing.aliyuncs.com/img/13856126-c598e06c51c60fa6.png)

Actuator endpoints 端点

*路径省略前缀：/actuator*

|      路径       | HTTP 动作 |                             描述                             |
| :-------------: | :-------: | :----------------------------------------------------------: |
|   /conditions   |    GET    | 提供了一份自动配置报告，记录哪些自动配置条件通过了，哪些没通过 |
|  /configprops   |    GET    |           描述配置属性（包含默认值）如何注入 Bean            |
|     /caches     |    GET    |                   获取所有的 Cachemanager                    |
| /caches/{cache} |  DELETE   |                    移除某个 CacheManager                     |
|     /beans      |    GET    |         描述应用程序上下文全部 bean，以及他们的关系          |
|   /threaddump   |    GET    |                       获取线程活动快照                       |
|      /env       |    GET    |                       获取全部环境属性                       |
| /env/{toMatch}  |    GET    |                获取指定名称的特定环境的属性值                |
|     /health     |    GET    |    报告应用长须的健康指标，由 HealthIndicator 的实现提供     |
|   /httptrace    |    GET    |       提供基本的 HTTP 请求跟踪信息(时间戳、HTTP 头等)        |
|      /info      |    GET    |         获取应用程序定制信息，由 Info 开头的属性提供         |
|    /loggers     |    GET    |                     获取 bean 的日志级别                     |
| /loggers/{name} |    GET    |              获取某个 包、类 路径端点的日志级别              |
| /loggers/{name} |   POST    |              新增某个 包、类 路径端点的日志级别              |
|    /mappings    |    GET    | 描述全部的 URL 路径，以及它们和控制器(包含Actuator端点)的映射关系 |
|    /metrics     |    GET    |    报告各种应用程序度量信息，如：内存用量、HTTP 请求计数     |
| /metrics/{name} |    GET    |                     根据名称获取度量信息                     |
|    /shutdown    |   POST    |      关闭应用，要求 endpoints.shutdown.enabled 为 true       |
| /scheduledtasks |    GET    |                     获取所有定时任务信息                     |

SpringCloud 中，默认开启了 `/actuator/health`、`/actuator/info` 端点，其他端点都屏蔽掉了。如果需要开启，自定义开启 endpoints 即可



```yml
management:
  endpoints:
    web:
      exposure:
        exclude: ['health','info','beans','env']
```

如果要开启全部端点，设置 `exclude: *` 即可

------

# 配置明细端点

引入pom文件，设置 yml



```xml
 <dependencies>
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-actuator</artifactId>
      </dependency>
  </dependencies>
```



```yml
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

## beans 端点

使用 /actuator/beans 端点，获取 JSON 格式的 bean 装配信息。访问： http://localhost:8080/actuator/beans



```json
{
    "contexts": {
        "application": {
            "beans": {
                "webEndpointDiscoverer": {    // bean 名称
                    "aliases": [],    // bean 别名
                    "scope": "singleton",   // bean 作用域
                    "type": "org.springframework.boot.actuate.endpoint.web.annotation.WebEndpointDiscoverer",   // bean 类型
                    "resource": "class path resource [org/springframework/boot/actuate/conditionsure/endpoint/web/WebEndpointconditionsuration.class]",   // class 文件物理地址
                    "dependencies": [   // 当前 bean 注入的 bean 的列表
                        "endpointOperationParameterMapper",
                        "endpointMediaTypes"
                    ]
                },
                "parentId": null
            }
        }
    }
}
```

## conditions 端点

/beans 端点可以看到当前 Spring 上下文有哪些 bean， /conditions 端点可以看到为什么有这个 bean
 SpringBoot 自动配置构建与 Spring 的条件化配置之上，提供了众多的 `@Conditional` 注解的配置类，根据 `@Conditional` 条件决定是否自动装配这些 bean。
 /conditions 端点提供了一个报告，列出了所有条件，根据条件是否通过进行分组

访问 http://localhost:8080/actuator/conditions



```json
{
    "contexts": {
        "application": {
            "positiveMatches": {    // 成功的自动装配
                "AuditEventsEndpointAutoConfiguration#auditEventsEndpoint": [{
                        "condition": "OnBeanCondition",   // 装配条件
                        "message": "@ConditionalOnBean (types: org.springframework.boot.actuate.audit.AuditEventRepository; SearchStrategy: all) found bean 'auditEventRepository'; @ConditionalOnMissingBean (types: org.springframework.boot.actuate.audit.AuditEventsEndpoint; SearchStrategy: all) did not find any beans"    // 装配信息
                    },
                    {
                        "condition": "OnEnabledEndpointCondition",
                        "message": "@ConditionalOnEnabledEndpoint no property management.endpoint.auditevents.enabled found so using endpoint default"
                    }
                ],
      },
            "negativeMatches": {      // 失败的自动装配
        "RabbitHealthIndicatorAutoConfiguration": {
          "notMatched": [   // 没有匹配到的条件
            {
              "condition": "OnClassCondition",
              "message": "@ConditionalOnClass did not find required class 'org.springframework.amqp.rabbit.core.RabbitTemplate'"
            }],
            "matched": []   // 匹配到的条件
          },
      },
            "unconditionalClasses": [     // 无条件装配
        "org.springframework.boot.actuate.autoconfigure.management.HeapDumpWebEndpointAutoConfiguration"
      ]
        }
    }
}
```

可以看到，在 `negativeMatches` 的 `RabbitHealthIndicatorAutoConfiguration` 自动状态，没有匹配到 `org.springframework.amqp.rabbit.core.RabbitTemplate` 类，所有没有自动装配。

## env 端点



```json
{
    "activeProfiles": [],   // 启用的 profile 
    "propertySources": [{
            "name": "server.ports",   // 端口设置
            "properties": {
                "local.server.port": {
                    "value": 8080   // 本地端口
                }
            }
        },
        {
            "name": "servletContextInitParams",   // servlet 上下文初始化参数信息
            "properties": {}
        },
        {
            "name": "systemProperties",     // 系统配置
            "properties": {
        "java.runtime.name": {    // 配置名称
          "value": "Java(TM) SE Runtime Environment"    // 配置值
        }
      }
        },
        {
            "name": "systemEnvironment",    // 系统环境
            "properties": {
        "USERDOMAIN_ROAMINGPROFILE": {    
          "value": "DESKTOP-QMHTL6V",   // 环境名称
          "origin": "System Environment Property \"USERDOMAIN_ROAMINGPROFILE\""   // 环境值
        }
      }
        },
        {
            "name": "applicationConfig: [classpath:/application.yml]",    // 配置文件信息
            "properties": {
                "management.endpoints.web.exposure.include": {      // 配置文件 key
                    "value": "*",       // 配置文件 value
                    "origin": "class path resource [application.yml]:5:18"    // 位置
                }
            }
        }
    ]
}
```

## mappings 端点

mappings 端点可以生成一份 `控制器到端点` 的映射，即 访问路径与控制器 的映射



```json
{
    "contexts": {
        "application": {
            "mappings": {
                "dispatcherServlets": {   // 所有的 DispatcherServlet
                    "dispatcherServlet": [{ // DispatcherServlet
                            "handler": "ResourceHttpRequestHandler [class path resource [META-INF/resources/], class path resource [resources/], class path resource [static/], class path resource [public/], ServletContext resource [/], class path resource []]",   // 处理器
                            "predicate": "/**/favicon.ico",   // 映射路径
                            "details": null
                        },
                        {
                            "handler": "public java.lang.Object org.springframework.boot.actuate.endpoint.web.servlet.AbstractWebMvcEndpointHandlerMapping$OperationHandler.handle(javax.servlet.http.HttpServletRequest,java.util.Map<java.lang.String, java.lang.String>)",
                            "predicate": "{[/actuator/auditevents],methods=[GET],produces=[application/vnd.spring-boot.actuator.v2+json || application/json]}",   // 断言（包含端点、请求方式、类型等）
                            "details": {    // 详细信息
                                "handlerMethod": {  // 处理器
                                    "className": "org.springframework.boot.actuate.endpoint.web.servlet.AbstractWebMvcEndpointHandlerMapping.OperationHandler", // handle 实现类
                                    "name": "handle", // 处理器名称
                                    "descriptor": "(Ljavax/servlet/http/HttpServletRequest;Ljava/util/Map;)Ljava/lang/Object;"    // 描述符
                                },
                                "requestMappingConditions": {   // 请求映射条件
                                    "consumes": [],   // 入参类型
                                    "headers": [],    // 入参头
                                    "methods": [      // 请求方式
                                        "GET"
                                    ],
                                    "params": [],     // 参数
                                    "patterns": [
                                        "/actuator/auditevents"   // 请求 URL
                                    ],
                                    "produces": [{    // 返回值类型
                                            "mediaType": "application/vnd.spring-boot.actuator.v2+json",
                                            "negated": false
                                        },
                                        {
                                            "mediaType": "application/json",
                                            "negated": false
                                        }
                                    ]
                                }
                            }
                        }
                    ]
                },
                "servletFilters": [{    // 所有 Filters
                        "servletNameMappings": [],    // servlet 名称映射
                        "urlPatternMappings": [
                            "/*"      // filter 过滤路径
                        ],
                        "name": "webMvcMetricsFilter",    // filter 名称
                        "className": "org.springframework.boot.actuate.metrics.web.servlet.WebMvcMetricsFilter"   // Filter 实现类
                    }
                ],
                "servlets": [{    // 所有 Servlet
                        "mappings": [],   // 映射
                        "name": "default",    // servlet 名称
                        "className": "org.apache.catalina.servlets.DefaultServlet"    // servlet 实现类
                    },
                    {
                        "mappings": [
                            "/"
                        ],
                        "name": "dispatcherServlet",
                        "className": "org.springframework.web.servlet.DispatcherServlet"
                    }
                ]
            },
            "parentId": null
        }
    }
}
```

# 运行时度量端点

## metrics 端点

### 访问结果

metrics 端点主要用于在项目运行中，查看计数器、度量器等，如：当前可用内存、空闲内存等

访问：http://localhost:8080/actuator/metrics ，可用查看所有的计数器、度量器名称， 访问 http://localhost:8080/actuator/metrics/{name} 可以查看具体信息

http://localhost:8080/actuator/metrics



```json
{
    "names": [
        "jvm.memory.max",
        "jvm.threads.states",
        "jvm.gc.pause",
        "http.server.requests",
        "jvm.gc.memory.promoted"
    ]
}
```

http://localhost:8080/actuator/metrics/jvm.memory.max



```json
{
    "name": "jvm.memory.max",   // 名称
    "description": "The maximum amount of memory in bytes that can be used for memory management",    // 介绍
    "baseUnit": "bytes",    // 单位
    "measurements": [{
        "statistic": "VALUE",
        "value": 5579472895     // 大小，单位 bytes
    }],
    "availableTags": [{
            "tag": "area",
            "values": [
                "heap",
                "nonheap"
            ]
        },
        {
            "tag": "id",
            "values": [
                "Compressed Class Space",
                "PS Survivor Space",
                "PS Old Gen",
                "Metaspace",
                "PS Eden Space",
                "Code Cache"
            ]
        }
    ]
}
```

http://localhost:8080/actuator/metrics/http.server.requests



```json
{
    "name": "http.server.requests",
    "description": null,
    "baseUnit": "seconds",   
    "measurements": [{
            "statistic": "COUNT",
            "value": 28   // 发起的请求总数
        },
        {
            "statistic": "TOTAL_TIME",
            "value": 1.9647074519999999   // 总时长
        },
        {
            "statistic": "MAX",
            "value": 0.002769728
        }
    ],
    "availableTags": [{
            "tag": "exception",
            "values": [
                "None"
            ]
        },
        {
            "tag": "method",
            "values": [
                "GET"
            ]
        },
        {
            "tag": "uri",
            "values": [     // 访问过的 uri
                "/actuator/caches",
                "/**/favicon.ico",
                "/actuator/threaddump",
                "/actuator/env/{toMatch}",
                "/actuator/loggers",
                "/actuator/mappings",
                "/actuator/auditevents",
                "/**",
                "/actuator/env",
                "/actuator/metrics/{requiredMetricName}",
                "/actuator",
                "/actuator/beans",
                "/actuator/httptrace",
                "/actuator/loggers/{name}",
                "/actuator/scheduledtasks",
                "/actuator/conditions",
                "/actuator/heapdump",
                "/actuator/metrics"
            ]
        },
        {
            "tag": "outcome",
            "values": [
                "CLIENT_ERROR",
                "SUCCESS"
            ]
        },
        {
            "tag": "status",
            "values": [
                "404",
                "200"
            ]
        }
    ]
}
```

### metrics 端点介绍

|         前缀          |    分类    |                             介绍                             |
| :-------------------: | :--------: | :----------------------------------------------------------: |
|       jvm.gc.*        | 垃圾收集器 | 已经发生过的垃圾收集次数、消耗时间，使用与标记-请求垃圾收集器和并行垃圾收集器`java.lang.management.GarbageCollectorMXBean` |
|     jvm.memory.*      |  内存相关  | 分配给应用程序的内存数量和空闲的内容数量等 `java.lang.Runtime` |
|     jvm.classes.*     |  类加载器  | JVM 类加载器加载与卸载的类的数量 `java.lang.management.ClassLoadingMXBean` |
| process.*、system.cpu |    系统    | 系统信息，如：处理器数量`java.lang.Runtime`、运行时间`java.lang.management.RuntimeMXBean`、平均负载`java.lang.management.OperatingSystemMXBean` |
|     jvm.threads.*     | JVM 线程池 | JVM 线程、守护线程数量、峰值等 `java.lang.management.ThreadMXBean` |
|       tomcat.*        |   tomcat   |                       tomcat 相关内容                        |
|     datasource.*      |   数据源   | 数据源链接数据等，仅当 Spring 上下文存在 DataSource 才会有这个信息 |

## httptrace 端点

httptrace 端点主要用于报告所有的 web 请求的详细信息，包括：请求方法、路径、时间戳、头信息等。http://localhost:8080/actuator/httptrace/



```json
{
    "traces": [{
        "timestamp": "2019-02-13T06:29:35.039Z",            // 时间戳
        "principal": null,
        "session": null,    
        "request": {        // 请求
            "method": "GET",        // 请求方式
            "uri": "http://localhost:8080/actuator/httptrace/",     // 请求路径
            "headers": {        // 请求 Headers
                "cookie": [     
                    "yfx_c_g_u_id_10000001=_ck19021309350014873926766957773; yfx_f_l_v_t_10000001=f_t_1550021700485__r_t_1550021700485__v_t_1550021700485__r_c_0"
                ]
            },
            "remoteAddress": null       // 远程地址
        },
        "response": {   // 响应
            "status": 200,      // 响应码
            "headers": {        // 响应 Header
                "Content-Type": [
                    "application/vnd.spring-boot.actuator.v2+json;charset=UTF-8"
                ]
            }
        },
        "timeTaken": 36     // 用时
    }]
}
```

需要注意，/httptrace 端点只能显示最近的 100 个请求信息，其中也包含对 /httptrace 端点自己的请求。

## threaddump 端点

threaddump 端点可以查看的应用程序的每个线程，其中包含线程的阻塞状态、所状态等，http://localhost:8080/actuator/threaddump/



```json
{
    "threads": [{
            "threadName": "DestroyJavaVM",      // 线程名称
            "threadId": 44,     // 线程 id
            "blockedTime": -1,  
            "blockedCount": 0,
            "waitedTime": -1,
            "waitedCount": 0,
            "lockName": null,
            "lockOwnerId": -1,
            "lockOwnerName": null,
            "inNative": false,
            "suspended": false,
            "threadState": "RUNNABLE",  // 线程状态
            "stackTrace": [],       // 跟踪栈
            "lockedMonitors": [],
            "lockedSynchronizers": [],
            "lockInfo": null
        }
    ]
}
```

## health 端点

health 端点：查看当前应用程序的运行状态。 http://localhost:8080/actuator/health/



```json
{
    "status": "UP"
}
```

health 端点在特定情况下，会有额外的信息，如：登录状态、数据库状态等

### Spring Boot 自带的监控指示器

|    键     |         健康指示器         |                             说明                             |
| :-------: | :------------------------: | :----------------------------------------------------------: |
|   none    | ApplicationHealthIndicator |                          永远为 UP                           |
|    db     | DataSourceHealthIndicator  |   如果数据库能连上，则内容为 UP 和数据库类型；否则为 DOWN    |
| diskSpace |  DiskSpaceHealthIndicator  | 如果可用空间大于阈值，则内容为 UP 和可用磁盘内容；否则为 DOWN |
|    jms    |     JmsHealthIndicator     |  如果能连上消息代理，则为 UP 和 JMS 提供方名称；否则为 DOWN  |
|   mail    |    MailHealthIndicator     | 如果能连上邮件服务器，则内容为 UP 个邮件服务器主机、端口；否则为 DOWN |
|   mongo   |    MongoHealthIndicator    | 如果能连上 MongoDb 服务器，则内容为 UP 和 MongoDB 服务器版本；否则为 DOWN |
|  rabbit   |   RabbitHealthIndicator    | 如果能连上 Rabbit 服务器，则内容为 UP 和 Rabbit 版本号；否则为 DOWN |
|   redis   |    RedisHealthIndicator    | 如果能连上 Redis 服务器，则内容为 UP 和 Redis 服务器版本；否则为 DOWN |
|   solr    |    SolrHealthIndicator     |      如果能连上 Solr 服务器，则内容为 UP ；否则为 DOWN       |

------

# 关闭应用程序

关闭应用程序可以使用 /actuator/shutdown 端点，使用 POST 请求 http://localhost:8080/actuator/shutdown



```json
{
    "timestamp": "2019-02-13T07:46:54.614+0000",
    "status": 404,
    "error": "Not Found",
    "message": "No message available",
    "path": "/actuator/shutdown"
}
```

这是因为为了保护应用程序，shutdown 端点没有打开的原因，需要打开 shutdown 端点



```yml
management:
  endpoint:
    shutdown:
      enabled: true
```

再次请求：



```json
{
    "message": "Shutting down, bye..."
}
```

------

# 获取应用信息

获取应用信息使用 /actuator/info 端点，默认的响应是：



```json
{}
```

可以通过带 info 前缀的属性，向 info 端点的响应增加内容，如：



```yml
info:
  contect:
    email: laiyy0728@gmail.com
    phone: 18888888888
```

再次请求：



```json
{
    "contect": {
        "email": "laiyy0728@gmail.com",
        "phone": 18888888888
    }
}
```



# 参考

https://www.jianshu.com/p/31832dc1d30e

https://www.jianshu.com/p/5d0962a21129