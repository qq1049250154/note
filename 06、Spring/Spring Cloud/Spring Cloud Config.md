Spring Cloud Config 是 Spring Cloud 微服务体系中的配置中心，是微服务中不可或缺的一部分，其能够很好的将程序中配置日益增多的各种功能的开关、参数的配置、服务器的地址等配置修改后实时生效、灰度发布，分环境、分集群管理配置等进行全面的集中化管理，有利于系统的配置管理、维护。

# Spring Cloud Config 配置中心

## 配置中心对比

|      对比方面      | 重要性 |   SpringCloud Config   | Netflix archaius |          携程 Apollo           |                          disconf                          |
| :----------------: | :----: | :--------------------: | :--------------: | :----------------------------: | :-------------------------------------------------------: |
|    静态配置管理    |   高   |       基于 file        |        无        |              支持              |                           支持                            |
|    动态配置管理    |   高   |          支持          |       支持       |              支持              |                           支持                            |
|      统一管理      |   高   | 无，需要 git、数据库等 |        无        |              支持              |                           支持                            |
|     多维度管理     |   中   | 无，需要 git、数据库等 |        无        |              支持              |                           支持                            |
|      变更管理      |   高   | 无，需要 git、数据库等 |        无        |               无               |                            无                             |
|    本地配置缓存    |   高   |           无           |        无        |              支持              |                           支持                            |
|    配置更新策略    |   中   |           无           |        无        |               无               |                            无                             |
|       配置锁       |   中   |          支持          |      不支持      |             不支持             |                          不支持                           |
|      配置校验      |   中   |           无           |        无        |               无               |                            无                             |
|    配置生效时间    |   高   |   重启生效、手动刷新   |   手动刷新失效   |              实时              |                           实时                            |
|    配置更新推送    |   高   |      需要手动触发      |   需要手动触发   |              支持              |                           支持                            |
|    配置定时拉取    |   高   |           无           |        无        |              支持              | 配置更新目前依赖事件驱动，client 重启或者 server 推送操作 |
|    用户权限管理    |   中   | 无，需要 git、数据库等 |        无        |              支持              |                           支持                            |
|  授权、审核、审计  |   中   | 无，需要 git、数据库等 |        无        | 界面直接提供发布历史、回滚按钮 |           操作记录存在数据库中，但是无查询接口            |
|    配置版本管理    |   高   |          git           |        无        |              支持              |           操作记录存在数据库中，但是无查询接口            |
|    配置合规检测    |   高   |         不支持         |      不支持      |         支持（不完整）         |                                                           |
|    实例配置监控    |   高   | 需要结合 spring admin  |      不支持      |              支持              |          支持，可以查看每个配置再哪台机器上加载           |
|      灰度发布      |   中   |         不支持         |      不支持      |              支持              |                      不支持部分更新                       |
|      告警通知      |   中   |         不支持         |      不支持      |        支持邮件方式告警        |                     支持邮件方式告警                      |
|      统计报表      |   中   |         不支持         |      不支持      |             不支持             |                          不支持                           |
|      依赖关系      |   高   |         不支持         |      不支持      |             不支持             |                          不支持                           |
|  支持 SpringBoot   |   高   |        原生支持        |        低        |              支持              |                    与 SpringBoot 无关                     |
| 支持 Spring Config |   高   |        原生支持        |        低        |              支持              |                    与 SpringBoot 无关                     |
|     客户端支持     |   低   |          java          |       java       |           java、.net           |                           java                            |
|    业务系统入侵    |   高   |        入侵性弱        |     入侵性弱     |            入侵性弱            |                 入侵性弱、支持注解和 xml                  |
|      单点故障      |   高   |      支持 HA 部署      |   支持 HA 部署   |          支持 HA 部署          |              支持 HA 部署、高可用由 zk 提供               |
|   多数据中心部署   |   高   |          支持          |       支持       |              支持              |                           支持                            |
|      配置界面      |   中   | 无，需要 git、数据库等 |        无        |            统一界面            |                         统一界面                          |

## 配置中心具备的功能

- Open API
- 业务无关性
- 配置生效监控
- 一致性 K-V 存储
- 统一配置实时推送
- 配合灰度与更新
- 配置全局恢复、备份、历史
- 高可用集群

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/13856126-e085498756a6b3a0.png)

spring cloud 配置中心功能图

## 配置中心流转

配置中心各流程流转如图：

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/13856126-1fd2d37e83aa069d.png)

Spring cloud 配置中心流转图

## 配置中心支撑体系

配置中心的支撑体系大致有两类

- 开发管理体系

- 运维管理体系

  ![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/13856126-eb5e2cc16bbecc4a.png)

  Spring 开发、运维体系

------

# Spring Cloud Config

## Spring Cloud Config 概述

Spring Cloud Config 是一个集中化、外部配置的分布式系统，由服务端、客户端组成，它不依赖于注册中心，是一个独立的配置中心。Spring Cloud Config 支持多种存储配置信息的形式，目前主要有 jdbc、vault、Navicat、svn、git 等形式，默认为 git。

## git 版工作原理

配置客户端启动时，会向服务端发起请求，服务端接收到客户端的请求后，根据配置的仓库地址，将 git 上的文件克隆到本地的一个临时目录中，这个目录是一个 git 的本地仓库，然后服务端再读取本地文件，返回给客户端。这样做的好处是：当 git 服务故障或网络请求异常时，保证服务端依然能正常工作。

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/13856126-1342cc980108e730.png)

Spring Cloud Config git 版工作原理

------

# 入门案例

## config repo

使用 git 做配置中心的配置文件存储，需要一个 git 仓库，用于保存配置文件。 本例仓库地址： [https://gitee.com/laiyy0728/config-repo](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Flaiyy0728%2Fconfig-repo)

在仓库中，新建一个文件夹：`config-simple`，在文件夹内新建 3 个文件：`config-simple-dev.yml`、`config-simple-test.yml`、`config-simple-prod.yml`

![img]()

Spring Cloud Simple Config



***源码：[https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-config/spring-cloud-config-simple](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Flaiyy0728%2Fspring-cloud%2Ftree%2Fmaster%2Fspring-cloud-config%2Fspring-cloud-config-simple)\***

## config server



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```



```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/laiyy0728/config-repo # git 仓库地址
          search-paths: config-simple # 从哪个文件夹下拉取配置
  application:
    name: spring-cloud-config-simple-server
server:
  port: 9090
```



```java
@SpringBootApplication
@EnableConfigServer
public class SpringCloudConfigSimpleServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringCloudConfigSimpleServerApplication.class, args);
    }
}
```

## 验证 config server

启动 config server，查看 endpoints mappings

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/13856126-b2cea4372a5f4051.png)

Spring Cloud Config Endpoints

- label：代表请求的是哪个分支，默认是 master 分支
- name：代表请求哪个名称的远程文件
- profile：代表哪个版本的文件，如：dev、test、prod 等

从 mappings 中，可以看出，访问获取一个配置的信息，有多种方式，尝试获取 `/config-simple/config-simple.dev.yml` 配置信息：

### 由接口获取配置详细信息

[http://localhost:9090/config-simple/dev/master](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A9090%2Fconfig-simple%2Fdev%2Fmaster) 、[http://localhost:9090/config-simple/dev](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A9090%2Fconfig-simple%2Fdev)



```json
{
    "name": "config-simple",
    "profiles": [
        "dev"
    ],
    "label": "master",
    "version": "520b379e9c7f2e39bb56e599f914b6c08fe13c06",
    "state": null,
    "propertySources": [{
        "name": "https://gitee.com/laiyy0728/config-repo/config-simple/config-simple-dev.yml",
        "source": {
            "com.laiyy.gitee.config": "dev 环境，git 版 spring cloud config"
        }
    }]
}
```

### 由绝对文件路径获取配置文件内容

[http://localhost:9090/master/config-simple-dev.yml](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A9090%2Fmaster%2Fconfig-simple-dev.yml) 、[http://localhost:9090/config-simple-dev.yml](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A9090%2Fconfig-simple-dev.yml)



```yml
com:
  laiyy:
    gitee:
      config: dev 环境，git 版 spring cloud config
```

## config client

在 config server 中获取配置文件以及成功，接下来需要在 config client 中，通过 config server 获取对应的配置文件



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-client</artifactId>
    </dependency>
</dependencies>
```

***application.yml\***



```yml
spring:
  cloud:
    config:
      label: master
      uri: http://localhost:9090
      name: config-simple
      profile: dev
  application:
    name: spring-cloud-config-simple-client
server:
  port: 9091
```



```java
// 用于从远程 config server 获取配置文件内容
@Component
@ConfigurationProperties(prefix = "com.laiyy.gitee")
public class ConfigInfoProperties {
    private String config;

    public String getConfig() {
        return config;
    }

    public void setConfig(String config) {
        this.config = config;
    }
}


// 用于打印获取到的配置文件内容
@RestController
public class ConfigController {

    private final ConfigInfoProperties configInfoProperties;

    @Autowired
    public ConfigController(ConfigInfoProperties configInfoProperties) {
        this.configInfoProperties = configInfoProperties;
    }

    @GetMapping(value = "/get-config-info")
    public String getConfigInfo(){
        return configInfoProperties.getConfig();
    }

}


// 启动类
@SpringBootApplication
public class SpringCloudConfigSimpleClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudConfigSimpleClientApplication.class, args);
    }

}
```

## 验证 config client

启动 config client，观察控制台，发现 config client 拉取配置的路径是：[http://localhost:8888](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A8888) ，而不是在 yml 中配置的 localhost:9090。这是因为 boot 启动时加载配置文件的顺序导致的。boot 默认先加载 bootstrap.yml 配置，再加载 application.yml 配置。所以需要将 config server 配置移到 bootstrap.yml 中

![img](https:////upload-images.jianshu.io/upload_images/13856126-e016c2907e4b97be.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

Config client default fetch server



***bootstrap.yml\***



```yml
spring:
  cloud:
    config:
      label: master  # 代表请求 git 哪个分支，默认 master
      uri: http://localhost:9090 # config server 地址
      name: config-simple # 获取哪个名称的远程文件，可以有多个，英文逗号隔开
      profile: dev # 代表哪个分支
```

***application.yml\***



```yml
spring:
  application:
    name: spring-cloud-config-simple-client
server:
  port: 9091
```

![img](https:////upload-images.jianshu.io/upload_images/13856126-a20970fe3fdcc7dc.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

config client remote server

访问 [http://localhost:9091/get-config-info](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A9091%2Fget-config-info)

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/13856126-a52d8b92f98cd21a.png)

get config info

# 手动刷新

***源码：[https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-config/spring-cloud-config-refresh](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Flaiyy0728%2Fspring-cloud%2Ftree%2Fmaster%2Fspring-cloud-config%2Fspring-cloud-config-refresh)\***

手动刷新的 config server 依然选用示例中的 `spring-cloud-config-simple-server`

## config client



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-client</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
</dependencies>
```

***bootstrap.yml\***



```yml
spring:
  cloud:
    config:
      label: master
      uri: http://localhost:9090
      name: config-simple
      profile: dev
```

***application.yml\***



```yml
spring:
  application:
    name: spring-cloud-config-refresh-client
server:
  port: 9091
management:
  endpoints:
    web:
      exposure:
        include: '*' # 暴露端点，用于手动刷新
  endpoint:
    health:
      show-details: always
```

***改造 config properties、config controller\***



```java
@Component
@RefreshScope // 标注为配置刷新域
public class ConfigInfoProperties {

    @Value("${com.laiyy.gitee.config}")
    private String config;

    public String getConfig() {
        return config;
    }

    public void setConfig(String config) {
        this.config = config;
    }
}

@RestController
@RefreshScope // 标注为配置刷新域
public class ConfigController {

    private final ConfigInfoProperties configInfoProperties;

    @Autowired
    public ConfigController(ConfigInfoProperties configInfoProperties) {
        this.configInfoProperties = configInfoProperties;
    }

    @GetMapping(value = "/get-config-info")
    public String getConfigInfo(){
        return configInfoProperties.getConfig();
    }

}
```

## 验证

访问 [http://localhost:9091/get-config-info](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A9091%2Fget-config-info) ，观察返回值为：



```undefined
dev 环境，git 版 spring cloud config
```

修改 [https://gitee.com/laiyy0728/config-repo/blob/master/config-simple/config-simple-dev.yml](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Flaiyy0728%2Fconfig-repo%2Fblob%2Fmaster%2Fconfig-simple%2Fconfig-simple-dev.yml) 的内容为：



```yml
com:
  laiyy:
    gitee:
      config: dev 环境，git 版 spring cloud config，使用手动刷新。。。
```

再次访问 [http://localhost:9091/get-config-info](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A9091%2Fget-config-info) ，观察返回值仍为：



```undefined
dev 环境，git 版 spring cloud config
```

这是因为没有进行手动刷新，POST 访问：[http://localhost:9091/actuator/refresh](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A9091%2Factuator%2Frefresh) ，返回信息如下：



```json
[
    "config.client.version",
    "com.laiyy.gitee.config"
]
```

控制台输出如下：



```csharp
Fetching config from server at : http://localhost:9090
Located environment: name=config-simple, profiles=[dev], label=master, version=a04663a171b0d8f552c3d549ad38401bd6873b95, state=null
Located property source: CompositePropertySource {name='configService', propertySources=[MapPropertySource {name='configClient'}, MapPropertySource {name='https://gitee.com/laiyy0728/config-repo/config-simple/config-simple-dev.yml'}]}
```

此时，再次访问 [http://localhost:9091/get-config-info](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A9091%2Fget-config-info) ，观察返回值变为：



```undefined
dev 环境，git 版 spring cloud config，使用手动刷新。。。
```

由此证明，手动刷新成功

------

# 半自动刷新

***源码：[https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-config/spring-cloud-config-bus](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Flaiyy0728%2Fspring-cloud%2Ftree%2Fmaster%2Fspring-cloud-config%2Fspring-cloud-config-bus)\***

半自动刷新依赖于 Spring Cloud Bus 总线，而 Bus 总线依赖于 RabbitMQ。 Spring Cloud Bus 刷新配置的流程图：

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/13856126-c1a63e325073b2e3.png)

Spring Cloud Bus 流程图

***Rabbit MQ 请自行安装启动，在此不做描述\***

## config server bus



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-monitor</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-bus-amqp</artifactId>
    </dependency>
</dependencies>
```



```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/laiyy0728/config-repo.git
          search-paths: config-simple
    bus:
      trace:
        enabled: true # 是否启用bus追踪
  application:
    name: spring-cloud-config-bus-server
  rabbitmq: # rabbit mq 配置
    host: 192.168.67.133
    port: 5672
    username: guest
    password: guest
server:
  port: 9090
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
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable();
    }
}


@SpringBootApplication
@EnableConfigServer
public class SpringCloudConfigBusServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudConfigBusServerApplication.class, args);
    }

}
```

## config client bus



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-bus-amqp</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-client</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
</dependencies>
```

***bootstrap.yml\***



```yml
spring:
  cloud:
    config:
      label: master
      uri: http://localhost:9090
      name: config-simple
      profile: test
```

***application.yml\***



```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/laiyy0728/config-repo.git
          search-paths: config-simple
    bus:
      trace:
        enabled: true # \u662F\u5426\u542F\u7528bus\u8FFD\u8E2A
  application:
    name: spring-cloud-config-bus-server
  rabbitmq: # rabbit mq \u914D\u7F6E
    host: 192.168.67.133
    port: 5672
    username: guest
    password: guest
server:
  port: 9090
management:
  endpoints:
    web:
      exposure:
        include: '*'
  endpoint:
    health:
      show-details: always
```

其余 Java 类与手动刷新一致。

## 验证

访问 [http://localhost:9095/get-config-info](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A9095%2Fget-config-info)

![img](https:////upload-images.jianshu.io/upload_images/13856126-f642f610df78c9d2.png?imageMogr2/auto-orient/strip|imageView2/2/w/419/format/webp)

Config Bus



将 `config-simple/config-simple-test.yml` 内容修改为



```yml
com:
  laiyy:
    gitee:
      config: test 环境，git 版 spring cloud config，bus 半自动刷新配置
```

再次访问 [http://localhost:9095/get-config-info](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A9095%2Fget-config-info) ，返回值仍为：



```bash
test 环境，git 版 spring cloud config
```

使用 bus 刷新配置，POST 请求 [http://localhost:9095/actuator/bus-refresh](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A9095%2Factuator%2Fbus-refresh) ，查看控制台输出：



```csharp
 Bean 'org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration' of type [org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration$$EnhancerBySpringCGLIB$$7c355e31] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
 Fetching config from server at : http://localhost:9090
 Located environment: name=config-simple, profiles=[test], label=master, version=45c17b3b2a7918ed7093251f2085641df446e961, state=null
 Located property source: CompositePropertySource {name='configService', propertySources=[MapPropertySource {name='configClient'}, MapPropertySource {name='https://gitee.com/laiyy0728/config-repo.git/config-simple/config-simple-test.yml'}]}
 No active profile set, falling back to default profiles: default
 Started application in 1.108 seconds (JVM running for 264.482)
 Received remote refresh request. Keys refreshed []
```

再次访问 [http://localhost:9095/get-config-info](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A9095%2Fget-config-info) ，返回值变为：



```bash
test 环境，git 版 spring cloud config，bus 半自动刷新配置
```

## refresh、bus-refresh 比较

- refresh：只能刷新单节点，即：只能刷新指定 ip 的配置信息
- bus-refresh：批量刷新，可以刷新订阅了 rabbit queue 的所有节点配置

------

# 自动刷新

自动刷新实际上很简单，只需要暴露一个 bus-refresh 节点，并在 config-server 的 git 中，配置 `webhook` 指向暴露出来的 bus-refresh 节点即可，多个 bus-refresh 节点用英文逗号分隔

![img](https:////upload-images.jianshu.io/upload_images/13856126-50884feea8fa8ea8.png?imageMogr2/auto-orient/strip|imageView2/2/w/866/format/webp)

Config webhook

# 服务端 git 配置详解

git 的版 config 有多种配置：

- uri 占位符
- 模式匹配
- 多残酷
- 路径搜索占位符

***源码：[https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-config/spring-cloud-placeholder](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Flaiyy0728%2Fspring-cloud%2Ftree%2Fmaster%2Fspring-cloud-config%2Fspring-cloud-placeholder)\***

公共依赖



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

## git 中 uri 占位符

Spring Cloud Config Server 支持占位符的使用，支持 {application}、{profile}、{label}，这样的话就可以在配置 uri 的时候，通过占位符使用应用名称来区分应用对应的仓库进行使用。

*** config server ***



```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/laiyy0728/{application}        # {application} 是匹配符，匹配项目名称
          search-paths: config-simple
  application:
    name: spring-cloud-placeholder-server
server:
  port: 9090
```

*** config client ***

bootstrap.yml



```yml
spring:
  cloud:
    config:
      label: master
      uri: http://localhost:9090
      name: config-repo
      profile: dev
```

使用 `{application}` 时，需要注意，在 config client 中配置的 `name`，既是 config 管理中心的 git 名称，又是需要匹配的配置文件名称。即：远程的 config git 管理中心地址为：[https://gitee.com/laiyy0728/config-repo](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Flaiyy0728%2Fconfig-repo) ，在仓库中 `config-simple` 文件夹下，必须有一个 `config-simple.yml` 配置文件。否则 config client 会找不到配置文件。

## 模式匹配、多存储库

***config server\***



```yml
spring:
  profiles:
    active: native # 本地配置仓库，在测试本地配置仓库之前，需要注释掉这一行
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/laiyy0728/config-repo
          search-paths: config-simple
          repos:
            simple: https://gitee.com/laiyy0728/simple
            special:
              pattern: special*/dev*,*special*/dev*
              uri: https://gitee.com/laiyy0728/special
        native:
          search-locations:  C:/Users/laiyy/AppData/Local/Temp/config-simple # 本地配置仓库路径
  application:
    name: spring-cloud-placeholder-server
server:
  port: 9090
```

## 路径搜索占位符



```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/laiyy0728/config-repo
          search-paths: config-* # 匹配以 config 开头的文件夹
  application:
    name: spring-cloud-placeholder-server
server:
  port: 9090
```

------

# 关系型数据库实现配置中心

架构图：



![img](https:////upload-images.jianshu.io/upload_images/13856126-c84aaaf3db9b7645.png?imageMogr2/auto-orient/strip|imageView2/2/w/776/format/webp)

config server mysql

公共依赖



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

## mysql config server



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
</dependencies>
```



```yml
server:
  port: 9090
spring:
  application:
    name: spring-cloud-customer-repo-mysql
  cloud:
    config:
      server:
        jdbc:
          sql: SELECT `KEY`, `VALUE` FROM PROPERTIES WHERE application = ? NAD profile = ? AND label = ?
      label: master
  profiles:
    active: jdbc
  datasource:
    url: jdbc:mysql:///springcloud?useUnicode=true&charsetEncoding=UTF-8
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
logging:
  level:
    org.springframework.jdbc.core: debug
    org.springframework.jdbc.core.StatementCreatorUtils: Trace
```

其余配置、config client 与之前一致即可。

## 验证

启动 config server、config client，可以看到，config server 打印日志如下：



```kotlin
...

Executing prepared SQL query
Executing prepared SQL statement [SELECT `KEY`, `VALUE` FROM PROPERTIES WHERE application = ? AND profile = ? AND lable = ?]
Setting SQL statement parameter value: column index 1, parameter value [config-simple], value class [java.lang.String], SQL type unknown
Setting SQL statement parameter value: column index 2, parameter value [dev], value class [java.lang.String], SQL type unknown
Setting SQL statement parameter value: column index 3, parameter value [master], value class [java.lang.String], SQL type unknown
Executing prepared SQL query
Executing prepared SQL statement [SELECT `KEY`, `VALUE` FROM PROPERTIES WHERE application = ? AND profile = ? AND lable = ?]
Setting SQL statement parameter value: column index 1, parameter value [config-simple], value class [java.lang.String], SQL type unknown
Setting SQL statement parameter value: column index 2, parameter value [default], value class [java.lang.String], SQL type unknown
Setting SQL statement parameter value: column index 3, parameter value [master], value class [java.lang.String], SQL type unknown
Executing prepared SQL query
Executing prepared SQL statement [SELECT `KEY`, `VALUE` FROM PROPERTIES WHERE application = ? AND profile = ? AND lable = ?]
Setting SQL statement parameter value: column index 1, parameter value [application], value class [java.lang.String], SQL type unknown
Setting SQL statement parameter value: column index 2, parameter value [dev], value class [java.lang.String], SQL type unknown
Setting SQL statement parameter value: column index 3, parameter value [master], value class [java.lang.String], SQL type unknown
Executing prepared SQL query
Executing prepared SQL statement [SELECT `KEY`, `VALUE` FROM PROPERTIES WHERE application = ? AND profile = ? AND lable = ?]
Setting SQL statement parameter value: column index 1, parameter value [application], value class [java.lang.String], SQL type unknown
Setting SQL statement parameter value: column index 2, parameter value [default], value class [java.lang.String], SQL type unknown
Setting SQL statement parameter value: column index 3, parameter value [master], value class [java.lang.String], SQL type unknown

...
```

访问 [http://localhost:9091/get-config-info](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A9091%2Fget-config-info) ，返回数据如下：

![img](https:////upload-images.jianshu.io/upload_images/13856126-3635704d692494d9.png?imageMogr2/auto-orient/strip|imageView2/2/w/513/format/webp)

spring cloud config mysql



------

# 非关系数据库实现配置中心

以 mongodb 为例，需要 spring cloud config server mongodb 依赖，github 地址：[https://github.com/spring-cloud-incubator/spring-cloud-config-server-mongodb](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fspring-cloud-incubator%2Fspring-cloud-config-server-mongodb)

## config server monngodb



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>

    <!-- mongogdb 在 spring cloud config server 的依赖，这个依赖是快照依赖，需要指定 spring 的仓库 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server-mongodb</artifactId>
        <version>0.0.3.BUILD-SNAPSHOT</version>
    </dependency>
</dependencies>
```



```yml
server:
  port: 9090
spring:
  application:
    name: spring-cloud-customer-repo-mongodb
  data:
    mongodb:
      uri: mongodb://192.168.67.133/springcloud # mongo 数据库地址
```



```java
@SpringBootApplication
@EnableMongoConfigServer  // 一定注意，不能写为 EnableConfigServer，一定要是 MongooConfigServer
public class SpringCloudCustomerRepoMongodbApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudCustomerRepoMongodbApplication.class, args);
    }

}
```

## mongo 测试数据

collection 名称：springcloud

数据：



```json
{
    "label": "master",
    "profile": "prod",
    "source": {
        "com": {
            "laiyy": {
                "gitee": {
                    "config": "I am the mongdb configuration file from dev environment. I will edit."
                }
            }
        }
    }
}
```

## config client



```yml
spring:
  cloud:
    config:
      label: master
      uri: http://localhost:9090
      name: springcloud # 这里指定的是 collection name
      profile: prod
```

## 验证

***config client 与之前的一致\***

访问 [http://localhost:9091/get-config-info](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A9091%2Fget-config-info) ，返回值为：

![img](https:////upload-images.jianshu.io/upload_images/13856126-dd96662f828ba816.png?imageMogr2/auto-orient/strip|imageView2/2/w/633/format/webp)

mongo result

# 本地参数覆盖远程参数



```yml
spring:
  cloud:
    config:
      allow-override: true
      override-none: true
      override-system-properties: false
```

- allow-override：标识 `override-system-properties` 是否启用，默认为 true，设置为 false 时，意味着禁用用户的设置
- override-none：当此项为 true，`override-override` 为 true，外部的配置优先级更低，而且不能覆盖任何存在的属性源。默认为 false
- override-system-properties：用来标识外部配置是否能够覆盖系统配置，默认为 true



```java
@ConfigurationProperties("spring.cloud.config")
public class PropertySourceBootstrapProperties {
    private boolean overrideSystemProperties = true;
    private boolean allowOverride = true;
    private boolean overrideNone = false;

    public PropertySourceBootstrapProperties() {
    }

    // 省略 getter、setter
}
```

------

# 客户端功能扩展

## 客户端自动刷新

***源码：[https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-config/spring-cloud-autoconfig](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Flaiyy0728%2Fspring-cloud%2Ftree%2Fmaster%2Fspring-cloud-config%2Fspring-cloud-autoconfig)\***

在有些应用上，不需要再服务端批量推送的时候，客户端本身需要获取变化参数的情况下，使用客户端的自动刷新能完成此功能。

***config server\*** 依然采用 `spring-cloud-config-simple-server`，基础配置不变，配置文件 repo 依然是 [https://gitee.com/laiyy0728/config-repo](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Flaiyy0728%2Fconfig-repo)

### 配置拉取、刷新二方库

新建一个二方库（spring-cloud-autoconfig-refresh），用于其他项目引入，以自动刷新配置（用于多个子项目使用同一个配置中心，自动刷新）



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-client</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-autoconfigure</artifactId>
    </dependency>
</dependencies>
```

整个二方库只有这一个类，作用是获取定时刷新时间，并刷新配置



```java
@Configuration
@ConditionalOnClass(RefreshEndpoint.class)
@ConditionalOnProperty("spring.cloud.config.refreshInterval")
@AutoConfigureAfter(RefreshAutoConfiguration.class)
@EnableScheduling
public class SpringCloudAutoconfigRefreshApplication implements SchedulingConfigurer {

    private static final Logger LOGGER = LoggerFactory.getLogger(SpringCloudAutoconfigRefreshApplication.class);

    @Autowired
    public SpringCloudAutoconfigRefreshApplication(RefreshEndpoint refreshEndpoint) {
        this.refreshEndpoint = refreshEndpoint;
    }

    @Value("${spring.cloud.config.refreshInterval}")
    private long refreshInterval;

    private final RefreshEndpoint refreshEndpoint;


    @Override
    public void configureTasks(ScheduledTaskRegistrar scheduledTaskRegistrar) {
        final long interval = getRefreshIntervalilliseconds();
        LOGGER.info(">>>>>>>>>>>>>>>>>>>>>> 定时刷新延迟 {} 秒启动，每 {} 毫秒刷新一次配置 <<<<<<<<<<<<<<<<", refreshInterval, interval);
        scheduledTaskRegistrar.addFixedDelayTask(new IntervalTask(refreshEndpoint::refresh, interval, interval));
    }

    /**
     * 返回毫秒级时间间隔
     */
    private long getRefreshIntervalilliseconds() {
        return refreshInterval * 1000;
    }

}
```

***/resources/META-INF/spring.factories\***



```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.laiyy.gitee.config.springcloudautoconfigrefresh.SpringCloudAutoconfigRefreshApplication
```

### 客户端引入二方库

创建客户端项目(spring-cloud-autoconfig-client)



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-client</artifactId>
    </dependency>

    <!-- 将配置好的自刷刷新作为二方库引入 -->
    <dependency>
        <groupId>com.laiyy.gitee.config</groupId>
        <artifactId>spring-cloud-autoconfig-refresh</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </dependency>
</dependencies>
```

***bootstrap.yml\***



```yml
spring:
  cloud:
    config:
      uri: http://localhost:9090
      label: master
      name: config-simple
      profile: dev
```

***application.yml\***



```yml
server:
  port: 9091
spring:
  application:
    name: spring-cloud-autoconfig-client
  cloud:
    config:
      refreshInterval: 10 # 延迟时间、定时刷新时间
```

其余配置与 `spring-cloud-config-simple-client` 一致

### 验证

启动项目，访问 [http://localhost:9090/get-config-info](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A9090%2Fget-config-info) ，正常返回信息。修改 config repo 配置文件，等待 10 秒后，再次访问，可见返回信息已经变为修改后信息。
 查看 client 控制台，可见定时刷新日志



```csharp
Exposing 2 endpoint(s) beneath base path '/actuator'
>>>>>>>>>>>>>>>>>>>>>> 定时刷新延迟 10 秒启动
No TaskScheduler/ScheduledExecutorService bean found for scheduled processing
Tomcat started on port(s): 9091 (http) with context path ''
Started SpringCloudAutoconfigClientApplication in 4.361 seconds (JVM running for 5.089)
Initializing Spring DispatcherServlet 'dispatcherServlet'
Initializing Servlet 'dispatcherServlet'
Fetching config from server at : http://localhost:9090      ------------------ 第一次请求
Completed initialization in 7 ms
Located environment: name=config-simple, profiles=[dev], label=master, version=00324826262afd5178a648a469247f4fffea945e, state=null
Bean 'org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration' of type [org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration$$EnhancerBySpringCGLIB$$f67277ed] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
Fetching config from server at : http://localhost:9090       ------------------ 定时刷新配置
Located environment: name=config-simple, profiles=[dev], label=master, version=00324826262afd5178a648a469247f4fffea945e, state=null
Located property source: CompositePropertySource {name='configService', propertySources=[MapPropertySource {name='configClient'}, MapPropertySource {name='https://gitee.com/laiyy0728/config-repo/config-simple/config-simple-dev.yml'}]}

... 省略其他多次刷新
```

## 客户端回退

客户端回退机制，可以在出现网络中断时、或者配置服务因维护而关闭时，使得客户端可以正常使用。当启动回退时，客户端适配器将配置“缓存”到计算机中。要启用回退功能，只需要指定缓存存储的位置即可。

***源码：[https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-config/spring-cloud-config-fallback](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Flaiyy0728%2Fspring-cloud%2Ftree%2Fmaster%2Fspring-cloud-config%2Fspring-cloud-config-fallback)\***

***config server\*** 依然采用 `spring-cloud-config-simple-server`，基础配置不变，配置文件 repo 依然是 [https://gitee.com/laiyy0728/config-repo](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Flaiyy0728%2Fconfig-repo)

### 二方库



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-rsa</artifactId>
        <version>1.0.7.RELEASE</version>
    </dependency>
</dependencies>
```

***config-client.properties\***：用于设置是否开启配置



```bash
spring.cloud.config.enabled=false
```

***/resources/META-INF/spring.factories\***



```java
org.springframework.cloud.bootstrap.BootstrapConfiguration=\
  com.laiyy.gitee.config.springcloudconfigfallbackautorefresh.ConfigServerBootStrap
```



```java
// 用于拉取远程配置文件，并保存到本地
@Order(0)
public class FallbackableConfigServerPropertySourceLocator extends ConfigServicePropertySourceLocator {

    private static final Logger LOGGER = LoggerFactory.getLogger(FallbackableConfigServerPropertySourceLocator.class);

    private boolean fallbackEnabled;

    private String fallbackLocation;

    @Autowired(required = false)
    private TextEncryptor textEncryptor;

    public FallbackableConfigServerPropertySourceLocator(ConfigClientProperties defaultProperties, String fallbackLocation) {
        super(defaultProperties);
        this.fallbackLocation = fallbackLocation;
        this.fallbackEnabled = !StringUtils.isEmpty(fallbackLocation);
    }

    @Override
    public PropertySource<?> locate(Environment environment){
        PropertySource<?> propertySource = super.locate(environment);
        if (fallbackEnabled && propertySource != null){
            storeLocally(propertySource);
        }
        return propertySource;
    }

    /**
     * 转换配置文件
     */
    private void storeLocally(PropertySource propertySource){
        StringBuilder builder = new StringBuilder();
        CompositePropertySource source = (CompositePropertySource) propertySource;
        for (String propertyName : source.getPropertyNames()) {
            Object property = source.getProperty(propertyName);
            if (textEncryptor != null){
                property = "{cipher}" + textEncryptor.encrypt(String.valueOf(property));
            }
            builder.append(propertyName).append("=").append(property).append("\n");
        }
        LOGGER.info(">>>>>>>>>>>>>>>>> file content: {} <<<<<<<<<<<<<<<<<<<", builder);
        saveFile(builder.toString());
    }

    /**
     * 保存配置到本地
     * @param content 配置内容
     */
    private void saveFile(String content){
        File file = new File(fallbackLocation + File.separator + ConfigServerBootStrap.FALLBACK_NAME);
        try {
            FileCopyUtils.copy(content.getBytes(), file);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}


// 用于判断从远程拉取配置文件，还是从本地拉取（spring boot 2.0，spring cloud F版）
@Configuration
@EnableConfigurationProperties
@PropertySource(value = {"config-client.properties","file:{spring.cloud.config.fallback-location:}/fallback.properties"}, ignoreResourceNotFound = true)
public class ConfigServerBootStrap {

    public static final String FALLBACK_NAME = "fallback.properties";

    private final ConfigurableEnvironment configurableEnvironment;

    @Autowired
    public ConfigServerBootStrap(ConfigurableEnvironment configurableEnvironment) {
        this.configurableEnvironment = configurableEnvironment;
    }

    @Value("${spring.cloud.config.fallback-location:}")
    private String fallbackLocation;

    @Bean
    public ConfigClientProperties configClientProperties(){
        ConfigClientProperties configClientProperties = new ConfigClientProperties(this.configurableEnvironment);
        configClientProperties.setEnabled(false);
        return configClientProperties;
    }

    @Bean
    public FallbackableConfigServerPropertySourceLocator fallbackableConfigServerPropertySourceLocator(){
        ConfigClientProperties client = configClientProperties();
        return new FallbackableConfigServerPropertySourceLocator(client, fallbackLocation);
    }

}
```

在 SpringBoot 1.0、Spring Cloud G 版中，会启动报错：



```tsx
***************************
APPLICATION FAILED TO START
***************************

Description:

The bean 'configClientProperties', defined in class path resource [com/laiyy/gitee/config/springcloudconfigfallbackautorefresh/ConfigServerBootStrap.class], could not be registered. A bean with that name has already been defined in class path resource [org/springframework/cloud/config/client/ConfigServiceBootstrapConfiguration.class] and overriding is disabled.

Action:

Consider renaming one of the beans or enabling overriding by setting spring.main.allow-bean-definition-overriding=true

2019-03-07 10:10:11.230 ERROR 13828 --- [           main] o.s.boot.SpringApplication               : Application run failed

org.springframework.beans.factory.support.BeanDefinitionOverrideException: Invalid bean definition with name 'configClientProperties' defined in class path resource [com/laiyy/gitee/config/springcloudconfigfallbackautorefresh/ConfigServerBootStrap.class]: Cannot register bean definition [Root bean: class [null]; scope=; abstract=false; lazyInit=false; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=com.laiyy.gitee.config.springcloudconfigfallbackautorefresh.ConfigServerBootStrap; factoryMethodName=configClientProperties; initMethodName=null; destroyMethodName=(inferred); defined in class path resource [com/laiyy/gitee/config/springcloudconfigfallbackautorefresh/ConfigServerBootStrap.class]] for bean 'configClientProperties': There is already [Root bean: class [null]; scope=; abstract=false; lazyInit=false; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=org.springframework.cloud.config.client.ConfigServiceBootstrapConfiguration; factoryMethodName=configClientProperties; initMethodName=null; destroyMethodName=(inferred); defined in class path resource [org/springframework/cloud/config/client/ConfigServiceBootstrapConfiguration.class]] bound.
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.registerBeanDefinition(DefaultListableBeanFactory.java:894) ~[spring-beans-5.1.2.RELEASE.jar:5.1.2.RELEASE]
    at org.springframework.context.annotation.ConfigurationClassBeanDefinitionReader.loadBeanDefinitionsForBeanMethod(ConfigurationClassBeanDefinitionReader.java:274) ~[spring-context-5.1.2.RELEASE.jar:5.1.2.RELEASE]
    ....
```

如果按照报错提示，增加了 `spring.main.allow-bean-definition-overriding=true` 的配置，没有任何作用；如果修改了 bean 名称



```java
@Bean(name="clientProperties")
public ConfigClientProperties configClientProperties(){
    ConfigClientProperties configClientProperties = new ConfigClientProperties(this.configurableEnvironment);
    configClientProperties.setEnabled(false);
    return configClientProperties;
}
```

会有如下报错：



```java
***************************
APPLICATION FAILED TO START
***************************

Description:

Method configClientProperties in org.springframework.cloud.config.client.ConfigClientAutoConfiguration required a single bean, but 2 were found:
    - configClientProperties: defined by method 'configClientProperties' in class path resource [org/springframework/cloud/config/client/ConfigClientAutoConfiguration.class]
    - clientProperties: defined by method 'configClientProperties' in class path resource [com/laiyy/gitee/config/springcloudconfigfallbackautorefresh/ConfigServerBootStrap.class]


Action:

Consider marking one of the beans as @Primary, updating the consumer to accept multiple beans, or using @Qualifier to identify the bean that should be consumed
```

原因：`ConfigClientProperties` 在初始化时已经默认单例加载。即：这个 bean 不能被重新注册到 spring 容器中。
 解决办法：将 spring 容器已经加载的单例的 `ConfigClientProperties` 注入进来，并在构造中设置为 false 即可



```java
@Configuration
@EnableConfigurationProperties
@PropertySource(value = {"configClient.properties", "file:${spring.cloud.config.fallbackLocation:}/fallback.properties"}, ignoreResourceNotFound = true)
public class ConfigServerBootStrap {

    public static final String FALLBACK_NAME = "fallback.properties";

    private final ConfigurableEnvironment configurableEnvironment;

    private final ConfigClientProperties configClientProperties;

    @Autowired
    public ConfigServerBootStrap(ConfigurableEnvironment configurableEnvironment, ConfigClientProperties configClientProperties) {
        this.configurableEnvironment = configurableEnvironment;
        this.configClientProperties = configClientProperties;

        this.configClientProperties.setEnabled(false);
    }

    @Value("${spring.cloud.config.fallbackLocation:}")
    private String fallbackLocation;

    @Bean
    public FallbackableConfigServerPropertySourceLocator fallbackableConfigServerPropertySourceLocator() {
        return new FallbackableConfigServerPropertySourceLocator(configClientProperties, fallbackLocation);
    }
}
```

### config client

***bootstrap.yml\***



```yml
spring:
  cloud:
    config:
      uri: http://localhost:9090
      label: master
      name: config-simple
      profile: dev
      fallbackLocation: E:\\springcloud
```

***application.yml\***



```yml
server:
  port: 9091
spring:
  application:
    name: spring-cloud-autoconfig-client
  main:
    allow-bean-definition-overriding: true
management:
  endpoints:
    web:
      exposure:
        include: '*'
  endpoint:
    health:
      show-details: always
```

其余配置、JAVA 类不变

### 验证

启动 config client，查看控制台，可见打印了 2 次远程拉取同步本地文件的信息：



```ruby
>>>>>>>>>>>>>>>>> file content: config.client.version=ee39bf20c492b27c2d1b1d0ff378ad721e79a758
com.laiyy.gitee.config=dev 环境，git 版 spring cloud config-----!
 <<<<<<<<<<<<<<<<<<<
```

查看本地 E:\springcloud 文件夹，可见多了一个 `fallback.properties` 文件

![img](https:////upload-images.jianshu.io/upload_images/13856126-5437ad4e488bcf37.png?imageMogr2/auto-orient/strip|imageView2/2/w/650/format/webp)

fallback local properties



文件内容：



![img](https:////upload-images.jianshu.io/upload_images/13856126-4f2b0dbb3a6341d9.png?imageMogr2/auto-orient/strip|imageView2/2/w/601/format/webp)

fallback local properties content

更新 config repo 的对应配置文件后，POST 访问 config client 刷新端口：[http://localhost:9091/actuator/refresh](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A9091%2Factuator%2Frefresh) 可见控制台再次打印同步本地文件信息。此时停止 config server 访问，再次访问 [http://localhost:9091/get-config-info](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A9091%2Fget-config-info) ，返回的信息是同步后的更新结果，由此验证客户端回退成功。

# 客户端高可用

***源码：[https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-config/spring-cloud-config-ha/spring-cloud-config-ha-client](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Flaiyy0728%2Fspring-cloud%2Ftree%2Fmaster%2Fspring-cloud-config%2Fspring-cloud-config-ha%2Fspring-cloud-config-ha-client)\***

`客户端高可用` 主要解决当前服务端不可用哪个的情况下，客户端依然可用正常启动。从客户端触发，不是增加配置中心的高可用性，而是降低客户端对配置中心的依赖程度，从而提高整个分布式架构的健壮性。

## 实现

### 配置的自动装配

**pom.xml**



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-client</artifactId>
    </dependency>
</dependencies>
```

**配置文件解析**



```java
@Component
@ConfigurationProperties(prefix = ConfigSupportProperties.CONFIG_PREFIX)
public class ConfigSupportProperties {

    /**
     * 加载的配置文件前缀
     */
    public static final String CONFIG_PREFIX = "spring.cloud.config.backup";

    /**
     * 默认文件名
     */
    private final String DEFAULT_FILE_NAME = "fallback.properties";

    /**
     * 是否启用
     */
    private boolean enabled = false;

    /**
     * 本地文件地址
     */
    private String fallbackLocation;

    public String getFallbackLocation() {
        return fallbackLocation;
    }

    public void setFallbackLocation(String fallbackLocation) {
        if (!fallbackLocation.contains(".")) {
            // 如果只指定了文件路径，自动拼接文件名
            fallbackLocation = fallbackLocation.endsWith(File.separator) ? fallbackLocation : fallbackLocation + File.separator;
            fallbackLocation += DEFAULT_FILE_NAME;
        }
        this.fallbackLocation = fallbackLocation;
    }

    public boolean isEnabled() {
        return enabled;
    }

    public void setEnabled(boolean enabled) {
        this.enabled = enabled;
    }
}
```

**自动装配实现类**



```java
@Configuration
@EnableConfigurationProperties(ConfigSupportProperties.class)
public class ConfigSupportConfiguration implements ApplicationContextInitializer<ConfigurableApplicationContext>, Ordered {


    private final Logger logger = LoggerFactory.getLogger(getClass());

    /**
     * 重中之重！！！！
     * 一定要注意加载顺序！！！
     *
     * bootstrap.yml 加载类：org.springframework.cloud.bootstrap.config.PropertySourceBootstrapConfiguration 的加载顺序是 
     * HIGHEST_PRECEDENCE+10，
     * 如果当前配置类再其之前加载，无法找到 bootstrap 配置文件中的信息，继而无法加载到本地
     * 所以当前配置类的装配顺序一定要在 PropertySourceBootstrapConfiguration 之后！
     */
    private final Integer orderNumber = Ordered.HIGHEST_PRECEDENCE + 11;

    @Autowired(required = false)
    private List<PropertySourceLocator> propertySourceLocators = Collections.EMPTY_LIST;

    @Autowired
    private ConfigSupportProperties configSupportProperties;

    /**
     * 初始化操作
     */
    @Override
    public void initialize(ConfigurableApplicationContext configurableApplicationContext) {
        // 判断是否开启 config server 管理配置
        if (!isHasCloudConfigLocator(this.propertySourceLocators)) {
            logger.info("Config server 管理配置未启用");
            return;
        }
        logger.info(">>>>>>>>>>>>>>> 检查 config Server 配置资源 <<<<<<<<<<<<<<<");
        ConfigurableEnvironment environment = configurableApplicationContext.getEnvironment();
        MutablePropertySources propertySources = environment.getPropertySources();
        logger.info(">>>>>>>>>>>>> 加载 PropertySources 源：" + propertySources.size() + " 个");

        // 判断配置备份功能是否启用
        if (!configSupportProperties.isEnabled()) {
            logger.info(">>>>>>>>>>>>> 配置备份未启用，使用：{}.enabled 打开 <<<<<<<<<<<<<<", ConfigSupportProperties.CONFIG_PREFIX);
            return;
        }

        if (isCloudConfigLoaded(propertySources)) {
            // 可以从 spring cloud 中获取配置信息
            PropertySource cloudConfigSource = getLoadedCloudPropertySource(propertySources);
            logger.info(">>>>>>>>>>>> 获取 config service 配置资源 <<<<<<<<<<<<<<<");
            Map<String, Object> backupPropertyMap = makeBackupPropertySource(cloudConfigSource);
            doBackup(backupPropertyMap, configSupportProperties.getFallbackLocation());
        } else {
            logger.info(">>>>>>>>>>>>>> 获取 config Server 资源配置失败 <<<<<<<<<<<<<");
            // 不能获取配置信息，从本地读取
            Properties backupProperty = loadBackupProperty(configSupportProperties.getFallbackLocation());
            if (backupProperty != null) {
                Map backupSourceMap = new HashMap<>(backupProperty);
                PropertySource backupSource = new MapPropertySource("backupSource", backupSourceMap);

                propertySources.addFirst(backupSource);
            }
        }

    }


    @Override
    public int getOrder() {
        return orderNumber;
    }


    /**
     * 从本地加载配置
     */
    private Properties loadBackupProperty(String fallbackLocation) {
        logger.info(">>>>>>>>>>>> 正在从本地加载！<<<<<<<<<<<<<<<<<");
        PropertiesFactoryBean propertiesFactoryBean = new PropertiesFactoryBean();
        Properties properties = new Properties();
        try {
            FileSystemResource fileSystemResource = new FileSystemResource(fallbackLocation);
            propertiesFactoryBean.setLocation(fileSystemResource);
            propertiesFactoryBean.afterPropertiesSet();
            properties = propertiesFactoryBean.getObject();
            if (properties != null){
                logger.info(">>>>>>>>>>>>>>> 读取成功！<<<<<<<<<<<<<<<<<<<<<<<<");
            }
        }catch (Exception e){
            e.printStackTrace();
            return null;
        }
        return properties;
    }


    /**
     * 备份配置信息
     */
    private void doBackup(Map<String, Object> backupPropertyMap, String fallbackLocation) {
        FileSystemResource fileSystemResource = new FileSystemResource(fallbackLocation);
        File file = fileSystemResource.getFile();
        try {
            if (!file.exists()){
                file.createNewFile();
            }
            if (!file.canWrite()){
                logger.info(">>>>>>>>>>>> 文件无法写入：{} <<<<<<<<<<<<<<<", fileSystemResource.getPath());
                return;
            }
            Properties properties = new Properties();
            Iterator<String> iterator = backupPropertyMap.keySet().iterator();
            while (iterator.hasNext()) {
                String key = iterator.next();
                properties.setProperty(key, String.valueOf(backupPropertyMap.get(key)));
            }
            FileOutputStream fileOutputStream = new FileOutputStream(fileSystemResource.getFile());
            properties.store(fileOutputStream, "backup cloud config");
        }catch (Exception e){
            logger.info(">>>>>>>>>> 文件操作失败！ <<<<<<<<<<<");
            e.printStackTrace();
        }
    }

    /**
     * 将配置信息转换为 map
     */
    private Map<String, Object> makeBackupPropertySource(PropertySource cloudConfigSource) {
        Map<String, Object> backupSourceMap = new HashMap<>();
        if (cloudConfigSource instanceof CompositePropertySource) {
            CompositePropertySource propertySource = (CompositePropertySource) cloudConfigSource;
            for (PropertySource<?> source : propertySource.getPropertySources()) {
                if (source instanceof MapPropertySource){
                    MapPropertySource mapPropertySource = (MapPropertySource) source;
                    String[] propertyNames = mapPropertySource.getPropertyNames();
                    for (String propertyName : propertyNames) {
                        if (!backupSourceMap.containsKey(propertyName)) {
                            backupSourceMap.put(propertyName, mapPropertySource.getProperty(propertyName));
                        }
                    }
                }
            }
        }

        return backupSourceMap;
    }


    /**
     * config server 管理配置是否开启
     */
    private boolean isHasCloudConfigLocator(List<PropertySourceLocator> propertySourceLocators) {
        for (PropertySourceLocator propertySourceLocator : propertySourceLocators) {
            if (propertySourceLocator instanceof ConfigServicePropertySourceLocator) {
                return true;
            }
        }
        return false;
    }

    /**
     * 获取 config service 配置资源
     */
    private PropertySource getLoadedCloudPropertySource(MutablePropertySources propertySources) {
        if (!propertySources.contains(PropertySourceBootstrapConfiguration.BOOTSTRAP_PROPERTY_SOURCE_NAME)){
            return null;
        }
        PropertySource<?> propertySource = propertySources.get(PropertySourceBootstrapConfiguration.BOOTSTRAP_PROPERTY_SOURCE_NAME);
        if (propertySource instanceof CompositePropertySource) {
            for (PropertySource<?> source : ((CompositePropertySource) propertySource).getPropertySources()) {
                // 如果配置源是 config service，使用此配置源获取配置信息
                // configService 是 bootstrapProperties 加载 spring cloud 的实现：ConfigServiceBootstrapConfiguration
                if ("configService".equals(source.getName())){
                    return source;
                }
            }
        }
        return null;
    }

    /**
     * 判断是否可以从 spring cloud 中获取配置信息
     */
    private boolean isCloudConfigLoaded(MutablePropertySources propertySources) {
        return getLoadedCloudPropertySource(propertySources) != null;
    }
}
```

**META-INF/spring.factories**



```undefined
org.springframework.cloud.bootstrap.BootstrapConfiguration=\
  com.laiyy.gitee.confog.springcloudconfighaclientautoconfig.ConfigSupportConfiguration
```

### 客户端实现



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-client</artifactId>
    </dependency>
    <dependency>
        <groupId>com.laiyy.gitee.confog</groupId>
        <artifactId>spring-cloud-config-ha-client-autoconfig</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </dependency>
</dependencies>
```

**bootstrap.yml**



```yml
spring:
  cloud:
    config:
      label: master
      uri: http://localhost:9090
      name: config-simple
      profile: dev
      backup:
        enabled: true # 自定义配置 -- 是否启用客户端高可用配置
        fallbackLocation: D:/cloud  # 自动备份的配置文档存放位置
```

**application.yml**



```yml
server:
  port: 9015

spring:
  application:
    name: spring-cloud-config-ha-client-config
```

**启动类**



```java
@SpringBootApplication
@RestController
public class SpringCloudConfigHaClientConfigApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudConfigHaClientConfigApplication.class, args);
    }

    @Value("${com.laiyy.gitee.config}")
    private String config;

    @GetMapping(value = "/config")
    public String getConfig(){
        return config;
    }

}
```

### config server



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>
</dependencies>
```



```yml
server:
  port: 9090
spring:
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/laiyy0728/config-repo.git
          search-paths: config-simple
```



```java
@EnableConfigServer
@SpringBootApplication
public class SpringCloudConfigHaClientConfigServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudConfigHaClientConfigServerApplication.class, args);
    }

}
```

## 验证

先后启动 `config-server`、`config-client`，查看`config-client`控制台输出：



```csharp
Fetching config from server at : http://localhost:9090
Located environment: name=config-simple, profiles=[dev], label=master, version=ee39bf20c492b27c2d1b1d0ff378ad721e79a758, state=null
Located property source: CompositePropertySource {name='configService', propertySources=[MapPropertySource {name='configClient'}, MapPropertySource {name='https://gitee.com/laiyy0728/config-repo.git/config-simple/config-simple-dev.yml'}]}
>>>>>>>>>>>>>>> 检查 config Server 配置资源 <<<<<<<<<<<<<<<
>>>>>>>>>>>>> 加载 PropertySources 源：11 个
>>>>>>>>>>>> 获取 config service 配置资源 <<<<<<<<<<<<<<<
No active profile set, falling back to default profiles: default
```

查看 `d:/cloud`，可见存在 `fallback.properties` 文件，打开文件，可见配置信息如下：



```properties
#backup cloud config
#Wed Apr 10 14:49:36 CST 2019
config.client.version=ee39bf20c492b27c2d1b1d0ff378ad721e79a758
com.laiyy.gitee.config=dev \u73AF\u5883\uFF0Cgit \u7248 spring cloud config-----\!
```

访问 [http://localhost:9015/config](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A9015%2Fconfig) ，可见打印信息如下：

![img](https:////upload-images.jianshu.io/upload_images/13856126-4650f38d9a39149e.png?imageMogr2/auto-orient/strip|imageView2/2/w/415/format/webp)

config-client-ha-result



------

停止 `server`、`client`，删除 `d:/cloud/fallback.properties`，将 `ConfigSupportConfiguration` 的 orderNumber 改为 `Ordered.HIGHEST_PRECEDENCE + 9`，再次先后启动 `config-server`、`config-client`，查看控制 `client` 控制台输出如下：



```csharp
>>>>>>>>>>>>>>> 检查 config Server 配置资源 <<<<<<<<<<<<<<<
>>>>>>>>>>>>> 加载 PropertySources 源：10 个
>>>>>>>>>>>>>> 获取 config Server 资源配置失败 <<<<<<<<<<<<<
Fetching config from server at : http://localhost:9090
```

可见，`PropertySources` 源从原来的 11 个，变为 10 个。原因是 `bootstrap.yml` 的加载顺序问题。
 在源码：`org.springframework.cloud.bootstrap.config.PropertySourceBootstrapConfiguration` 中，其加载顺序为：`Ordered.HIGHEST_PRECEDENCE + 10`，而 `ConfigSupportConfiguration` 的加载顺序为 `Ordered.HIGHEST_PRECEDENCE + 9`，先于 bootstrap.yml 配置文件加载执行，所以无法获取到远程配置信息，继而无法备份配置信息。

------

重新进行第一步验证，然后将 `config-server`、`config-client` 停掉后，只启动 `config-client`，可见其控制台打印信息如下：



```ruby
>>>>>>>>>>>>>>> 检查 config Server 配置资源 <<<<<<<<<<<<<<<
>>>>>>>>>>>>> 加载 PropertySources 源：10 个
>>>>>>>>>>>>>> 获取 config Server 资源配置失败 <<<<<<<<<<<<<
>>>>>>>>>>>> 正在从本地加载！<<<<<<<<<<<<<<<<<
>>>>>>>>>>>>>>> 读取成功！<<<<<<<<<<<<<<<<<<<<<<<<
```

访问 [http://localhost:9015/config](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A9015%2Fconfig) 正常返回信息。

由此验证`客户端高可用`成功

------

# 服务端高可用

服务端高可用，一般情况下是通过与注册中心结合实现。通过 Ribbon 的负载均衡选择 Config Server 进行连接，来获取配置信息。

***源码：[https://gitee.com/laiyy0728/spring-cloud/tree/master/spring-cloud-config/spring-cloud-config-ha/spring-cloud-config-ha-server](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Flaiyy0728%2Fspring-cloud%2Ftree%2Fmaster%2Fspring-cloud-config%2Fspring-cloud-config-ha%2Fspring-cloud-config-ha-server)\***

eureka 选择使用 `spring-cloud-eureka-server-simple`

## config server



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>
</dependencies>
```



```yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/laiyy0728/config-repo.git
          search-paths: config-simple
  application:
    name: spring-cloud-config-ha-server-app
server:
  port: 9090
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```



```java
@SpringBootApplication
@EnableConfigServer
@EnableDiscoveryClient
public class SpringCloudConfigHaServerConfigApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudConfigHaServerConfigApplication.class, args);
    }

}
```

## config client



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-client</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
</dependencies>
```

**application.yml**



```yml
server:
  port: 9016

spring:
  application:
    name: spring-cloud-config-ha-server-client
```

**bootstrap.yml**



```yml
spring:
  cloud:
    config:
      label: master
      name: config-simple
      profile: dev
      discovery:
        enabled: true # 是否从注册中心获取 config server
        service-id: spring-cloud-config-ha-server-app # 注册中心 config server 的 serviceId
eureka:
  client:
    service-url:
      defauleZone: http://localhost:8761/eureka/
```



```java
@SpringBootApplication
@RestController
public class SpringCloudConfigHaServerClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudConfigHaServerClientApplication.class, args);
    }

    @Value("${com.laiyy.gitee.config}")
    private String config;

    @GetMapping(value = "/config")
    public String getConfig(){
        return config;
    }

}
```

启用验证：访问 [http://localhost:9016/config](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A9016%2Fconfig) ,返回值如下：

![img](https:////upload-images.jianshu.io/upload_images/13856126-2e4592149edf9bf3.png?imageMogr2/auto-orient/strip|imageView2/2/w/415/format/webp)

config-client-ha-result

# 参考

https://www.jianshu.com/p/588084c959f5

https://www.jianshu.com/p/99550377db8f

https://www.jianshu.com/p/b69450259416

https://www.jianshu.com/p/ac6dda3115c6

https://www.jianshu.com/p/5de23aa2386e

