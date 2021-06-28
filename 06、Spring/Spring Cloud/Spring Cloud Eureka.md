# Eureka简介

　　Eureka是Netflix开发的服务发现框架，本身是一个基于REST的服务，主要用于定位运行在AWS域中的中间层服务，以达到负载均衡和中间层服务故障转移的目的。SpringCloud将它集成在其子项目spring-cloud-netflix中，以实现SpringCloud的服务发现功能。

### 　　1、Eureka组件

　　Eureka包含两个组件：Eureka Server和Eureka Client。

#### 　　1.1 Eureka Server

　　Eureka Server提供服务注册服务，各个节点启动后，会在Eureka Server中进行注册，这样Eureka Server中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观的看到。
　　Eureka Server本身也是一个服务，默认情况下会自动注册到Eureka注册中心。
　　**如果搭建单机版的Eureka Server注册中心，则需要配置取消Eureka Server的自动注册逻辑。**毕竟当前服务注册到当前服务代表的注册中心中是一个说不通的逻辑。
　　Eureka Server通过**Register、Get、Renew**等接口提供服务的**注册、发现和心跳检测**等服务。

#### 　　2.1 Eureka Client

　　Eureka Client是一个java客户端，用于简化与Eureka Server的交互，**客户端同时也具备一个内置的、使用轮询(round-robin)负载算法的负载均衡器。在应用启动后**，将会向Eureka Server发送心跳,**默认周期为30秒**，如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，Eureka Server将会从服务注册表中把这个**服务节点移除(默认90秒)**。
　　Eureka Client分为两个角色，分别是：Application Service(Service Provider)和Application Client(Service Consumer)

#### 　　2.1.1 Application Service

　　服务提供方，是注册到Eureka Server中的服务。

#### 　　2.1.2 Application Client

　　服务消费方，通过Eureka Server发现服务，并消费。

　　在这里，Application Service和Application Client不是绝对上的定义，因为Provider在提供服务的同时，也可以消费其他Provider提供的服务；Consumer在消费服务的同时，也可以提供对外服务。

### 　　2、Eureka Server架构原理简介


![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/1010726-20190924030604638-1576995909.png)

> Register(服务注册)：把自己的IP和端口注册给Eureka。
> Renew(服务续约)：发送心跳包，每30秒发送一次。告诉Eureka自己还活着。
> Cancel(服务下线)：当provider关闭时会向Eureka发送消息，把自己从服务列表中删除。防止consumer调用到不存在的服务。
> Get Registry(获取服务注册列表)：获取其他服务列表。
> Replicate(集群中数据同步)：eureka集群中的数据复制与同步。
> Make Remote Call(远程调用)：完成服务的远程调用。

　　Eureka Server
　　Eureka Server既是一个注册中心，同时也是一个服务。那么搭建Eureka Server的方式和以往搭建Dubbo注册中心ZooKeeper的方式必然不同，那么首先搭建一个单机版的Eureka Server注册中心。

# CAP定理

　　CAP原则又称CAP定理，指的是在一个分布式系统中，Consistency（数据一致性）、 Availability（服务可用性）、Partition tolerance（分区容错性），三者不可兼得。CAP由Eric Brewer在2000年PODC会议上提出。该猜想在提出两年后被证明成立，成为我们熟知的CAP定理。

| **分布式系统CAP定理**            |                                                              |
| -------------------------------- | ------------------------------------------------------------ |
| 数据一致性(Consistency)          | 数据一致性(Consistency)也叫做数据原子性系统在执行某项操作后仍然处于一致的状态。在分布式系统中，更新操作执行成功后所有的用户都应该读到最新的值，这样的系统被认为是具有强一致性的。等同于所有节点访问同一份最新的数据副本。优点： 数据一致，没有数据错误可能。缺点： 相对效率降低。 |
| 服务可用性(Availablity)          | 每一个操作总是能够在一定的时间内返回结果，这里需要注意的是"一定时间内"和"返回结果"。一定时间内指的是，在可以容忍的范围内返回结果，结果可以是成功或者是失败。 |
| 分区容错性(Partition-torlerance) | 在网络分区的情况下，被分隔的节点仍能正常对外提供服务(分布式集群，数据被分布存储在不同的服务器上，无论什么情况，服务器都能正常被访问) |

| **定律：任何分布式系统只可同时满足二点，没法三者兼顾。** |                                                              |
| -------------------------------------------------------- | ------------------------------------------------------------ |
| CA，放弃P                                                | 如果想避免分区容错性问题的发生，一种做法是将所有的数据(与事务相关的)/服务都放在一台机器上。虽然无法100%保证系统不会出错，但不会碰到由分区带来的负面效果。当然这个选择会严重的影响系统的扩展性。 |
| CP，放弃A                                                | 相对于放弃"分区容错性"来说，其反面就是放弃可用性。一旦遇到分区容错故障，那么受到影响的服务需要等待一定时间，因此在等待时间内系统无法对外提供服务。 |
| AP，放弃C                                                | 这里所说的放弃一致性，并不是完全放弃数据一致性，而是放弃数据的强一致性，而保留数据的最终一致性。以网络购物为例，对只剩下一件库存的商品，如果同时接受了两个订单，那么较晚的订单将被告知商品告罄。 |

　　**Eureka和ZooKeeper的特性**

## ![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/1010726-20190927041018390-1437576544.png)

# 基本使用

## 搭建Eureka服务注册中心

首先创建一个 [Maven](http://c.biancheng.net/maven/) 项目，取名为 eureka-server，在 pom.xml 中配置 Eureka 的依赖信息，代码如下所示。

```xml
<!-- Spring Boot -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.6.RELEASE</version>
    <relativePath />
</parent>
<dependencies>
    <!-- eureka -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
</dependencies>
<!-- Spring Cloud -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

创建一个启动类 EurekaServerApplication，代码如下所示。

```java
@EnableEurekaServer
@SpringBootApplication
    public static void main(String[] args) {
        SpringApplication.run(EurekaServer Application.class, args);
    }
}
```

这里所说的启动类，跟我们之前讲的 Spring Boot 几乎完全一样，只是多了一个 @EnableEurekaServer 注解，表示开启 Eureka Server。

接下来在 src/main/resources 下面创建一个 application.properties 属性文件，增加下面的配置：

```properties
spring.application.name=eureka-server
server.port=8761
\# 由于该应用为注册中心, 所以设置为false, 代表不向注册中心注册自己
eureka.client.register-with-eureka=false
\# 由于注册中心的职责就是维护服务实例, 它并不需要去检索服务, 所以也设置为 false
eureka.client.fetch-registry=false
```

eureka.client.register-with-eureka 一定要配置为 false，不然启动时会把自己当作客户端向自己注册，会报错。

接下来直接运行 EurekaServerApplication 就可以启动我们的注册中心服务了。我们在 application.properties 配置的端口是 8761，则可以直接通过 http://localhost：8761/ (http://localhost%EF%BC%9A8761/) 去浏览器中访问，然后便会看到 Eureka 提供的 Web 控制台。

## 使用Eureka编写服务提供者

#### 1）创建项目注册到 Eureka

注册中心已经创建并且启动好了，接下来我们实现将一个服务提供者 eureka-client-user-service 注册到 Eureka 中，并提供一个接口给其他服务调用。

首先还是创建一个 [Maven](http://c.biancheng.net/maven/) 项目，然后在 pom.xml 中增加相关依赖，代码如下所示。

```xml
<!-- Spring Boot -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.6.RELEASE</version>
    <relativePath />
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- eureka -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
</dependencies>
<!-- Spring Cloud -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

创建一个启动类 App，代码如下所示。

```java
@SpringBootApplication
@EnableDiscoveryClient
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
```

启动类的方法与之前没有多大区别，只是注解换成 @EnableDiscoveryClient，表示当前服务是一个 Eureka 的客户端。

接下来在 src/main/resources 下面创建一个 application.properties 属性文件，增加下面的配置：

```properties
spring.application.name= eureka-client-user-service
server.port=8081
eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka/
\# 采用IP注册
eureka.instance.preferIpAddress=true
\# 定义实例ID格式
eureka.instance.instance-id=${spring.application.name}:${spring.cloud.client.ip-address}:${server.port}
```

eureka.client.serviceUrl.defaultZone 的地址就是我们之前启动的 Eureka 服务的地址，在启动的时候需要将自身的信息注册到 Eureka 中去。

执行 App 启动服务，我们可以看到控制台中有输出注册信息的日志：

DiscoveryClient_EUREKA-CLIENT-USER-SERVICE/eureka-client-user-service:192.168.31.245:8081 - registration status: 204

我们可以进一步检查服务是否注册成功。回到之前打开的 Eureka 的 Web 控制台，刷新页面，就可以看到新注册的服务信息了。

#### 2）编写提供接口

创建一个 Controller，提供一个接口给其他服务查询，代码如下所示。

```java
纯文本复制
@RestControllerpublic class UserController {    
    @GetMapping("/user/hello")    
    public String hello() {
        return “hello”;    
    }
}
```

重启服务，访问 http://localhost:8081/user/hello (http://localhost%EF%BC%9A8081/user/hello)，如果能看到我们返回的 Hello 字符串，就证明接口提供成功了。

## 使用Eureka编写服务消费者

#### 1）直接调用接口

创建服务消费者，消费我们刚刚编写的 user/hello 接口，同样需要先创建一个 [Maven](http://c.biancheng.net/maven/) 项目 eureka-client-article-service，然后添加依赖，依赖和服务提供者的一样，这里就不贴代码了。

创建启动类 App，启动代码与前面所讲也是一样的。唯一不同的就是 application.properties 文件中的配置信息：

```properties
spring.application.name=eureka-client-article-service
server.port=8082
```

RestTemplate 是 [Spring](http://c.biancheng.net/spring/) 提供的用于访问 Rest 服务的客户端，RestTemplate 提供了多种便捷访问远程 Http 服务的方法，能够大大提高客户端的编写效率。我们通过配置 RestTemplate 来调用接口，代码如下所示。

```java
@Configuration
public class BeanConfiguration {
    @Bean
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

创建接口，在接口中调用 user/hello 接口，代码如下所示。

```java
@RestController
public class ArticleController {
    @Autowired
    private RestTemplate restTemplate;
    @GetMapping("/article /callHello")
    public String callHello() {
        return restTemplate.getForObject("http://localhost:8081/user/hello", String.class);
    }
}
```

执行 App 启动消费者服务，访问 /article/callHello 接口来看看有没有返回 Hello 字符串，如果返回了就证明调用成功。访问地址为 http://localhost:8082/article/callHello (http://localhost%EF%BC%9A8082/article/callHello)。

#### 2）通过 Eureka 来消费接口

上面提到的方法是直接通过服务接口的地址来调用的，和我们之前的做法一样，完全没有用到 Eureka 带给我们的便利。既然用了注册中心，那么客户端调用的时候肯定是不需要关心有多少个服务提供接口，下面我们来改造之前的调用代码。

首先改造 RestTemplate 的配置，添加一个 @LoadBalanced 注解，这个注解会自动构造 LoadBalancerClient 接口的实现类并注册到 Spring 容器中，代码如下所示。

```java
@Configuration
public class BeanConfiguration {
    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

接下来就是改造调用代码，我们不再直接写固定地址，而是写成服务的名称，这个名称就是我们注册到 Eureka 中的名称，是属性文件中的 spring.application.name，相关代码如下所示。

```java
@GetMapping("/article/callHello2")
public String callHello2() {
    return restTemplate.getForObject("http://eureka-client-user-service/user/hello", String.class);
}
```

## Eureka注册中心开启密码认证

Eureka 自带了一个 Web 的管理页面，方便我们查询注册到上面的实例信息，但是有一个问题：如果在实际使用中，注册中心地址有公网 IP 的话，必然能直接访问到，这样是不安全的。所以我们需要对 Eureka 进行改造，加上权限认证来保证安全性。


 改造我们的 eureka-server，通过集成 [Spring](http://c.biancheng.net/spring/)-Security 来进行安全认证。

 在 pom.xml 中添加 Spring-Security 的依赖包，代码如下所示。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

然后在 application.properties 中加上认证的配置信息：

```properties
spring.security.user.name=yinjihuan #用户名
spring.security.user.password=123456 #密码
```

增加 Security 配置类：

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // 关闭csrf
        http.csrf().disable();
        // 支持httpBasic
        http.authorizeRequests().anyRequest().authenticated().and().httpBasic();
    }
}
```

重新启动注册中心，访问 http://localhost:8761/，此时浏览器会提示你输入用户名和密码，输入正确后才能继续访问 Eureka 提供的管理页面。

 在 Eureka 开启认证后，客户端注册的配置也要加上认证的用户名和密码信息：

```properties
eureka.client.serviceUrl.defaultZone=http://zhangsan:123456@localhost:8761/eureka/
```

## 相关配置

### 服务保护

#### 　服务保护模式

　　服务保护模式（自我保护模式）：一般情况下，微服务在Eureka上注册后，会每30秒发送心跳包，Eureka通过心跳来判断服务时候健康，同时会定期删除超过90秒没有发送心跳服务。

　　导致Eureka Server接收不到心跳包的可能：一是微服务自身的原因，二是微服务与Eureka之间的网络故障。通常微服务的自身的故障只会导致个别服务出现故障，一般不会出现大面积故障，而网络故障通常会导致Eureka Server在短时间内无法收到大批心跳。虑到这个区别，Eureka设置了一个阀值，当判断挂掉的服务的数量超过阀值时，Eureka Server认为很大程度上出现了网络故障，将不再删除心跳过期的服务。

　　那么这个阀值是多少呢？Eureka Server在运行期间，会统计心跳失败的比例在15分钟内是否低于85%，如果低于85%，Eureka Server则任务是网络故障，不会删除心跳过期服务。

　　这种服务保护算法叫做Eureka Server的服务保护模式。

　　这种不删除的，90秒没有心跳的服务，称为无效服务，但是还是保存在服务列表中。如果Consumer到注册中心发现服务，则Eureka Server会将所有好的数据（有效服务数据）和坏的数据（无效服务数据）都返回给Consumer。

#### 　服务保护模式的存在必要性

　　因为同时保留"好数据"与"坏数据"总比丢掉任何数据要更好，当网络故障恢复后，Eureka Server会退出"自我保护模式"。

　　Eureka还有客户端缓存功能(也就是微服务的缓存功能)。即便Eureka Server集群中所有节点都宕机失效，微服务的Provider和Consumer都能正常通信。

　　微服务的负载均衡策略会自动剔除死亡的微服务节点（**Robbin**）。

　　只要Consumer不关闭，缓存始终有效，直到一个应用下的所有Provider访问都无效的时候，才会访问Eureka Server重新获取服务列表。

#### 　关闭服务保护模式

　　可以通过全局配置文件来关闭服务保护模式，商业项目中不推荐关闭服务保护，因为网络不可靠很容易造成网络波动、延迟、断线的可能。如果关闭了服务保护，可能导致大量的服务反复注册、删除、再注册。导致效率降低。在商业项目中，服务的数量一般都是几十个，大型的商业项目中服务的数量可能上百、数百，甚至上千：

 

```
# 关闭自我保护:true为开启自我保护，false为关闭自我保护
eureka.server.enableSelfPreservation=false
# 清理间隔(单位:毫秒，默认是60*1000)，当服务心跳失效后多久，删除服务。
eureka.server.eviction.interval-timer-in-ms=60000
```

#### 　优雅关闭服务（优雅停服）

　　在Spring Cloud中，可以通过HTTP请求的方式，通知Eureka Client优雅停服，这个请求一旦发送到Eureka Client，那么Eureka Client会发送一个shutdown请求到Eureka Server，Eureka Server接收到这个shutdown请求后，会在服务列表中标记这个服务的状态为down，同时Eureka Client应用自动关闭。这个过程就是优雅停服。

　　如果使用了优雅停服，则不需要再关闭Eureka Server的服务保护模式。

　　POM依赖：
　　优雅停服是通过Eureka Client发起的，所以需要在Eureka Client中增加新的依赖，这个依赖是autuator组件，添加下述依赖即可。

 

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-actuator</artifactId>
</dependency>
```

　　修改全局配置文件：

　　Eureka Client默认不开启优雅停服功能，需要在全局配置文件中新增如下内容：

 

```
# 启用shutdown，优雅停服功能
endpoints.shutdown.enabled=true
# 禁用密码验证
endpoints.shutdown.sensitive=false
```

　　发起shutdown请求：

　　必须通过POST请求向Eureka Client发起一个shutdown请求。请求路径为：http://ip:port/shutdown。可以通过任意技术实现，如：HTTPClient、form表单，AJAX等。

　　建议使用优雅停服方式来关闭Application Service/Application Client服务。

### 自定义 Eureka 的 InstanceID

客户端在注册时，服务的 Instance ID 的默认值的格式如下：

​	${spring.cloud.client.hostname}:${spring.application.name}:${spring.application. instance_id:${server.port}}

翻译过来就是“主机名：服务名称：服务端口”。当我们在 Eureka 的 Web 控制台查看服务注册信息的时候，就是这样的一个格式：

​	user-PC：eureka-client-user-service：8081

很多时候我们想把 IP 显示在上述格式中，此时，只要把主机名替换成 IP 就可以了，或者调整顺序也可以。可以改成下面的样子，用“服务名称：服务所在 IP：服务端口”的格式来定义：

```properties
eureka.instance.instance-id=${spring.application.name}:${spring.cloud.client.ip-address}:${server.port}
```

定义之后我们看到的就是 eureka-client-user-service：192.168.31.245：8081，一看就知道是哪个服务，在哪台机器上，端口是多少。

 我们还可以点击服务的 Instance ID 进行跳转，这个时候显示的名称虽然变成了 IP，但是跳转的链接却还是主机名。

 所以还需要加一个配置才能让跳转的链接变成我们想要的样子，使用 IP 进行注册，如图 2 所示：

```properties
eureka.instance.preferIpAddress=true
```


![Eureka实例信息IP链接](https://typoralim.oss-cn-beijing.aliyuncs.com/img/5-1ZR0162543438.png)

 图 2 Eureka实例信息IP链接

### 	自定义实例跳转链接

刚刚我们通过配置实现了用 IP 进行注册，当点击 Instance ID 进行跳转的时候，就可以用 IP 跳转了，跳转的地址默认是 IP+Port/info。我们可以自定义这个跳转的地址：

```properties
eureka.instance.status-page-url=c.biancheng.net
```

效果如图 3 所示。
![Eureka实例信息自定义链接](https://typoralim.oss-cn-beijing.aliyuncs.com/img/5-1ZR0164AWa.png)

 图 3 Eureka实例信息自定义链接

# Eureka集群搭建实现高可用服务注册中心

![这里写图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/2018091022462152.png)

为了提高高可用，Eureka是提供集群功能。不仅Eureka Client端可以集群，做为注册中心的Eureka Server也可以集群 。

多个Eureka Client进程向注册中心注册的Eureka Service Id相同，则被认为是一个集群。
多个Eureka Server相互注册，组成一个集群

每个Eureka Server注册的消息发生变化时，会各个服务之间定时同步，中间过程每个服务的数据可能不一致，但是最终会变得一致
Eureka Server集群之间的状态是采用异步方式同步的，所以不保证节点间的状态一定是一致的，不过基本能保证最终状态是一致的。

### Eureka Server 服务之间之间同步：Eureka Server配置中心集群

每个Eureka Server注册的消息发生变化时，会各个服务之间定时同步，中间过程每个服务的数据可能不一致，但是最终会变得一致
Eureka Server集群之间的状态是采用异步方式同步的，所以不保证节点间的状态一定是一致的，不过基本能保证最终状态是一致的。

### 集群重要的类com.netflix.eureka.registry.PeerAwareInstanceRegistryImpl

为了保证集群里所有Eureka Server节点的状态同步，所有以下操作都会同步到集群的所有服务上：服务注册（Registers）、服务注册（Registers）、服务更新（Renewals）、服务取消（Cancels）,服务超时（Expirations）和服务状态变更（Status Changes）。以下是一些部分重要

- syncUp：在Eureka Server重启或新的Eureka Server节点加进来的，会执行初始化，从而能够正常提供服务。当Eureka server启动时，他会从其它节点获取所有的注册信息。如果获取同步失败，它在一定时间（此值由决定）内拒绝服务。
- replicateToPeers： 复印所有的eureka操作到集群中其他节点，请求再次转发到其它的Eureka Server，调用同样的接口，传入同样的参数，除了会在header中标记isReplication=true，从而避免重复的replicate
- register: 注册登录的实例，并且复印此实例的信息到所有的eureka server的节点。如果其它Eureka server调用此节点，只在本节点更新实例信息，避免通知其他节点执行更新
- renew：心跳
- cancel

### 新的Eureka Server节点加入集群后的影响

当有新的节点加入到集群中，会对现在Eureka Server和Eureka Client有什么影响以及他们如何发现新增的Eureka Server节点：

- 新增的Eureka Server：在Eureka Server重启或新的Eureka Server节点加进来的，它会从集群里其它节点获取所有的实例注册信息。如果获取同步失败，它会在一定时间（此值由决定决定）内拒绝服务。
- 已有Eureka Server如何发现新的Eureka Server:
  - 已有的Eureka Server：在运行过程中，Eureka Server之间会定时同步实例的注册信息。这样即使新的Application Service只向集群中一台注册服务，则经过一段时间会集群中所有的Eureka Server都会有这个实例的信息。那么Eureka Server节点之间如何相互发现，各个节点之间定时（时间由eureka.server.peer-eureka-nodes-update-interval-ms决定）更新节点信息，进行相互发现。
  - Service Consumer：Service Consumer刚启动时，它会从配置文件读取Eureka Server的地址信息。当集群中新增一个Eureka Server中时，那么Service Provider如何发现这个Eureka Server？Service Consumer会定时（此值由eureka.client.eureka-service-url-poll-interval-seconds决定）调用Eureka Server集群接口，获取所有的Eureka Server信息的并更新本地配置。



### 	搭建步骤

Eureka 的集群搭建方法很简单：每一台 Eureka 只需要在配置中指定另外多个 Eureka 的地址就可以实现一个集群的搭建了。

下面我们以 2 个节点为例来说明搭建方式。假设我们有 master 和 slaveone 两台机器，需要做的就是：

- ​		将 master 注册到 slaveone 上面。
- ​		将 slaveone 注册到 master 上面。



 如果是 3 台机器，以此类推：

- ​		将 master 注册到 slaveone 和 slavetwo 上面。

- ​		将 slaveone 注册到 master 和 slavetwo 上面。

- ​		将 slavetwo 注册到 master 和 slaveone 上面。

  

创建一个新的项目 eureka-server-cluster，配置跟 eureka-server 一样。

 首先，我们需要增加 2 个属性文件，在不同的环境下启动不同的实例。增加 application-master.properties：

```properties
server.port=8761
# 指向你的从节点的Eureka
eureka.client.serviceUrl.defaultZone=http://用户名:密码@localhost:8762/eureka/
增加 application-slaveone.properties：
server.port=8762
# 指向你的主节点的Eureka
eureka.client.serviceUrl.defaultZone=http://用户名:密码 @localhost:8761/eureka/
在 application.properties 中添加下面的内容：
spring.application.name=eureka-server-cluster
# 由于该应用为注册中心, 所以设置为false, 代表不向注册中心注册自己
eureka.client.register-with-eureka=false
# 由于注册中心的职责就是维护服务实例, 并不需要检索服务, 所以也设置为 false
eureka.client.fetch-registry=false
spring.security.user.name=zhangsan
spring.security.user.password=123456
# 指定不同的环境
spring.profiles.active=master
```

在 A 机器上默认用 master 启动，然后在 B 机器上加上 --spring.profiles.active=slaveone 启动即可。

 这样就将 master 注册到了 slaveone 中，将 slaveone 注册到了 master 中，无论谁出现问题，应用都能继续使用存活的注册中心。

 之前在客户端中我们通过配置 eureka.client.serviceUrl.defaultZone 来指定对应的注册中心，当我们的注册中心有多个节点后，就需要修改 eureka.client.serviceUrl.defaultZone 的配置为多个节点的地址，多个地址用英文逗号隔开即可：

```properties
eureka.client.serviceUrl.defaultZone=http://zhangsan:123456@localhost:8761
                /eureka/,http://zhangsan:123456@localhost:8762/eureka/
```



# Eureka开发时快速移除失效服务

在实际开发过程中，我们可能会不停地重启服务，由于 Eureka 有自己的保护机制，故节点下线后，服务信息还会一直存在于 Eureka 中。我们可以通过增加一些配置让移除的速度更快一点，当然只在开发环境下使用，生产环境下不推荐使用。

 首先在我们的 eureka-server 中增加两个配置，分别是关闭自我保护和清理间隔：

```properties
eureka.server.enable-self-preservation=false
# 默认 60000 毫秒
eureka.server.eviction-interval-timer-in-ms=5000
```

然后在具体的客户端服务中配置下面的内容：

```properties
eureka.client.healthcheck.enabled=true
# 默认 30 秒
eureka.instance.lease-renewal-interval-in-seconds=5
# 默认 90 秒
eureka.instance.lease-expiration-duration-in-seconds=5
```

eureka.client.healthcheck.enabled 用于开启健康检查，需要在 pom.xml 中引入 actuator 的依赖，代码如下所示。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

其中：

- ​		eureka.instance.lease-renewal-interval-in-seconds 表示 Eureka Client 发送心跳给 server 端的频率。
- ​		eureka.instance.lease-expiration-duration-in-seconds 表示 Eureka Server 至上一次收到 client 的心跳之后，等待下一次心跳的超时时间，在这个时间内若没收到下一次心跳，则移除该 Instance。


 更多的 Instance 配置信息可参考源码中的配置类：org.springframework.cloud.netflix.eureka.EurekaInstanceConfigBean。

 更多的 Server 配置信息可参考源码中的配置类：org.springframework.cloud.netflix.eureka.server.EurekaServerConfigBean。

# Spring Cloud Eureka配置详解

**Eureka包含四个部分的配置**

instance：当前Eureka Instance实例信息配置； client：Eureka Client客户端特性配置； server：Eureka Server注册中心特性配置； dashboard：Eureka Server注册中心仪表盘配置。

### 1 Eureka Instance实例信息配置

Bean类：org.springframework.cloud.netflix.eureka.EurekaInstanceConfigBean进行加载。包装成com.netflix.appinfo.InstanceInfo对象发送的Eureka服务端。

开头：eureka.instance

#### 1.1 元数据

Eureka客户端在想服务注册中心发送注册请求时的自身服务信息描述对象。主要包含服务名称、实例名称、实例IP、实例端口等，以及一些负载均衡策略或者自定义其他特殊用途元素信息。

格式：eureka.instance.metadatamap.< key >=< value >

如：eureka.instance.metadatamap.zone = shanghai

#### 1.2 实例名称配置

用于区分同一服务中不同实例的标识。（即服务名称相同，主机或者端口不同），避免启动时选择端口的麻烦。

如：eureka.instance.instanceId=${spring.application.name}:${random.int}

#### 1.3 端点配置

在InstanceInfo中，可以看到一些URL的配置信息，比如homePageUrl、statusPageUrl、healthCheckUrl。它们分别代表了应用主页的URL、状态页的URL、健康检查的URL。其中，状态页和监控检查的URL在Spring Cloud Eureka中默认使用了spring-boot-actuator模块提供的/info端点和/health端点。为了服务的正常运作，必须确认Eureka客户端的/health端点在发送元数据的时候，是一个能够被注册中心访问的地址，否则服务注册中心不会根据应用的健康状态来更改状态（仅当开启了healthcheck功能时，以该端点信息作为健康检查标准）。而/info端点如果不正确的话，会导致在Eureka面板单击服务实例时，无法访问到服务实例提供的信息接口。

在一些特殊的情况下，**比如，为应用设置了context-path，**这时，所有spring-boot-actuator模块的监控端点都会增加一个前缀。所以，我们就需要做类似如下的配置，为/info和/health端点也加上类似的前缀：



```
management.context-path=/hello
eureka.instance.statusPageUrlPath=${management.context-path}/info
eureka.instance.healthCheckUrlPath=${management.context-path}/health
```

另外，有时候为了安全考虑，也有可能会修改/info和/health端点的原始路径。这个时候，我们也需要做一些特殊配置，例如：

```
ndpoints.info.pah=/appinfo
endpoints.health.path=/cheakHealth
eureka.instance.statusPageUrlPath=/${endpoints.info.pah}
eureka.instance.healthCheckUrlPath=/${endpoints.health.path}
```

上面实例使用的是相对路径。

由于Eureka的服务注册中心默认会以HTTP的方式来访问和暴露这些端点，因此当客户端应用以HTTPS的方式来暴露服务和监控端点时，相对路径的配置方式就无法满足要求了。所以，Spring Cloud Eureka还提供了绝对路径的配置参数，例如：

 

```
eureka.instance.homePageUrl=https://${eureka.instance.homename}
eureka.instance.statusPageUrlPath=https://${eureka.instance.homename}/info
eureka.instance.healthCheckUrlPath=https://${eureka.instance.homename}/health
```

#### 1.4 健康检测

**默认情况下，Eureka中各个服务实例的健康检查并不是通过spring-boot-actuator模块的/health端点来实现的，而是依靠客户端心跳的方式保持服务实例的存活**，在Eureka的服务续约与剔除机制下，客户端的监控状态从注册到注册中心开始都会处于UP状态，除非心跳终止一段时间之后，服务注册中心将其剔除。默认的心跳实现方式可以有效检查客户端进程是否正常运作，但却无法保证客户端应用能够正常提供服务。由于大多数的应用都会有一些其他的外部资源依赖，比如数据库。缓存、消息代理等，如果应用与这些外部资源无法联通的时候，实际上已经不能提供正常的对外服务了，但此时心跳依然正常，所以它还是会被服务消费者调用，而这样的调用实际上并不能获得预期的结果。

在Spring Cloud Eureka中，我们可以通过简单的配置，把Eureka客户端的监控检查交给spring-boot-actuator模块的/health端点，以实现更加全面的健康状态维护。

详细步骤如下：

```
1 在pom.xml中加入spring-boot-starter-actuator模块的依赖。
2 在application.properties中增加参数配置eureka.client.healthcheck.enabled=true
3 如果客户端/health端点做了特殊处理，需要参照1.3中的端点配置。
```

#### 1.5 其他配置

其他配置附于后面的表清单中。

### 2 Eureka Client客户端特性配置

Eureka Client客户端特性配置是对作为Eureka客户端的特性配置，包括Eureka注册中心，本身也是一个Eureka Client。

Eureka Client特性配置全部在org.springframework.cloud.netflix.eureka.EurekaClientConfigBean中，实际上它是com.netflix.discovery.EurekaClientConfig的实现类，替代了netxflix的默认实现。

开头：eureka.client

### 3 server：Eureka Server注册中心特性配置

Eureka Server注册中心端的配置是对注册中心的特性配置。Eureka Server的配置全部在org.springframework.cloud.netflix.eureka.server.EurekaServerConfigBean里，实际上它是com.netflix.eureka.EurekaServerConfig的实现类，替代了netflix的默认实现。

Eureka Server的配置全部以eureka.server.xxx的格式进行配置。

### 4 Eureka Server注册中心仪表盘配置

注册中心仪表盘的配置主要是控制注册中心的可视化展示。以eureka.dashboard.xxx的格式配置。

### 5 Spring Cloud Eureka常用配置清单

清单来源：https://www.cnblogs.com/li3807/p/7282492.html（致谢）

| 配置名称                                          | 默认值 默认值 | 说明 说明                                                    |
| ------------------------------------------------- | ------------- | ------------------------------------------------------------ |
| 服务注册中心配置                                  |               | Bean类：org.springframework.cloud.netflix.eureka.server.EurekaServerConfigBean |
| eureka.server.enableSelfPreservation              | false         | 关闭注册中心的保护机制，Eureka 会统计15分钟之内心跳失败的比例低于85%将会触发保护机制，不剔除服务提供者，如果关闭服务注册中心将不可用的实例正确剔除 |
| 服务实例类配置                                    |               | Bean类：org.springframework.cloud.netflix.eureka.EurekaInstanceConfigBean |
| eureka.instance.preferIpAddress                   | false         | 不使用主机名来定义注册中心的地址，而使用IP地址的形式，如果设置了eureka.instance.ip-address 属性，则使用该属性配置的IP，否则自动获取除环路IP外的第一个IP地址 |
| eureka.instance.ipAddress                         |               | IP地址                                                       |
| eureka.instance.hostname                          |               | 设置当前实例的主机名称                                       |
| eureka.instance.appname                           |               | 服务名，默认取 spring.application.name 配置值，如果没有则为 unknown |
| eureka.instance.leaseRenewalIntervalInSeconds     | 30            | 定义服务续约任务（心跳）的调用间隔，单位：秒                 |
| eureka.instance.leaseExpirationDurationInSeconds  | 90            | 定义服务失效的时间，单位：秒                                 |
| eureka.instance.statusPageUrlPath                 | /info         | 状态页面的URL，相对路径，默认使用 HTTP 访问，如果需要使用 HTTPS则需要使用绝对路径配置 |
| eureka.instance.statusPageUrl                     |               | 状态页面的URL，绝对路径                                      |
| eureka.instance.healthCheckUrlPath                | /health       | 健康检查页面的URL，相对路径，默认使用 HTTP 访问，如果需要使用 HTTPS则需要使用绝对路径配置 |
| eureka.instance.healthCheckUrl                    |               | 健康检查页面的URL，绝对路径                                  |
| 服务注册类配置                                    |               | Bean类：org.springframework.cloud.netflix.eureka.EurekaClientConfigBean |
| eureka.client.serviceUrl.                         |               | 指定服务注册中心地址，类型为 HashMap，并设置有一组默认值，默认的Key为 defaultZone；默认的Value为 http://localhost:8761/eureka ，如果服务注册中心为高可用集群时，多个注册中心地址以逗号分隔。如果服务注册中心加入了安全验证，这里配置的地址格式为：[http://:@localhost:8761/eureka](https://blog.csdn.net/qq_34553637/article/details/86081457) 其中 <username> 为安全校验的用户名；<password> 为该用户的密码 |
| eureka.client.fetchRegistery                      | true          | 检索服务                                                     |
| eureka.client.registeryFetchIntervalSeconds       | 30            | 从Eureka服务器端获取注册信息的间隔时间，单位：秒             |
| eureka.client.registerWithEureka                  | true          | 启动服务注册                                                 |
| eureka.client.eurekaServerConnectTimeoutSeconds   | 5             | 连接 Eureka Server 的超时时间，单位：秒                      |
| eureka.client.eurekaServerReadTimeoutSeconds      | 8             | 读取 Eureka Server 信息的超时时间，单位：秒                  |
| eureka.client.filterOnlyUpInstances               | true          | 获取实例时是否过滤，只保留UP状态的实例                       |
| eureka.client.eurekaConnectionIdleTimeoutSeconds  | 30            | Eureka 服务端连接空闲关闭时间，单位：秒                      |
| eureka.client.eurekaServerTotalConnections        | 200           | 从Eureka 客户端到所有Eureka服务端的连接总数                  |
| eureka.client.eurekaServerTotalConnectionsPerHost | 50            | 从Eureka客户端到每个Eureka服务主机的连接总数                 |
| 注册中心仪表盘配置                                |               | Bean类：org.springframework.cloud.netflix.eureka.server.EurekaDashboardProperties |
| eureka.dashboard.path                             |               | 仪表盘访问路径                                               |
| eureka.dashboard.enabled                          | true          | 是否启用仪表盘                                               |

 

# Eureka的REST API及API扩展

## 	Eureka REST API

Eureka 作为注册中心，其本质是存储了每个客户端的注册信息，Ribbon 在转发的时候会获取注册中心的服务列表，然后根据对应的路由规则来选择一个服务给 Feign 来进行调用。如果我们不是 [Spring Cloud](http://c.biancheng.net/spring_cloud/) 技术选型，也想用 Eureka，可以吗？完全可以。

 如果不是 [Spring](http://c.biancheng.net/spring/) Cloud 技术栈，笔者推荐用 Zookeeper，这样会方便些，当然用 Eureka 也是可以的，这样的话就会涉及如何注册信息、如何获取注册信息等操作。其实 Eureka 也考虑到了这点，提供了很多 REST 接口来给我们调用。

 我们举一个比较有用的案例来说明，比如对 Nginx 动态进行 upstream 的配置。

 在架构变成微服务之后，微服务是没有依赖的，可以独立部署，端口也可以随机分配，反正会注册到注册中心里面，调用方也无须关心提供方的 IP 和 Port，这些都可以从注册中心拿到。

 但是有一个问题：API 网关的部署能这样吗？API 网关大部分会用 Nginx 作为负载，那么 Nginx 就必须知道 API 网关有哪几个节点，这样网关服务就不能随便启动了，需要固定。

 当然网关是不会经常变动的，也不会经常发布，这样其实也没什么大问题，唯一不好的就是不能自动扩容了。

 其实利用 Eureka 提供的 API 我们可以获取某个服务的实例信息，也就是说我们可以根据 Eureka 中的数据来动态配置 Nginx 的 upstream。

 这样就可以做到网关的自动部署和扩容了。网上也有很多的方案，结合 Lua 脚本来做，或者自己写 Sheel 脚本都可以。

 下面举例说明如何获取 Eureka 中注册的信息。具体的接口信息请查看官方文档“https://github.com/Netflix/eureka/wiki/Eureka-REST-operations“。

 获取某个服务的注册信息，可以直接 GET 请求：http://localhost：8761/eureka/apps/eureka-client-user-service。其中，eureka-client-user-service 是应用名称，也就是 spring.application.name。

 在浏览器中，数据的显示格式默认是 XML 格式的，如图 1 所示。


![Eureka中的服务信息数据](https://typoralim.oss-cn-beijing.aliyuncs.com/img/5-1ZR01G04O92.png)

 图 1 Eureka 中的服务信息数据


 如果想返回 Json数据的格式，可以用一些接口测试工具来请求，比如 Postman，在请求头中添加下面两行代码即可。

​	Content-Type:application/json Accept:application/json

如果 Eureka 开启了认证，记得添加认证信息，用户名和密码必须是 Base64 编码过的 Authorization：Basic 用户名：密码，其余的接口就不做过多讲解了，大家可以自己去尝试。Postman 直接支持了 Basic 认证，将选项从 Headers 切换到 Authorization，选择认证方式为 Basic Auth 就可以填写用户信息了。

 填写完之后，直接发起请求就可以了。我们切换到 Headers 选项中，就可以看到请求头中已经多了一个 Authorization 头。

## 	元数据使用

Eureka 的元数据有两种类型，分别是框架定好了的标准元数据和用户自定义元数据。标准元数据指的是主机名、IP 地址、端口号、状态页和健康检查等信息，这些信息都会被发布在服务注册表中，用于服务之间的调用。自定义元数据可以使用 eureka.instance.metadataMap 进行配置。

 自定义元数据说得通俗点就是自定义配置，我们可以为每个 Eureka Client 定义一些属于自己的配置，这个配置不会影响 Eureka 的功能。

 自定义元数据可以用来做一些扩展信息，比如灰度发布之类的功能，可以用元数据来存储灰度发布的状态数据，Ribbon 转发的时候就可以根据服务的元数据来做一些处理。当不需要灰度发布的时候可以调用 Eureka 提供的 REST API 将元数据清除掉。

 下面我们来自定义一个简单的元数据，在属性文件中配置如下：

​	eureka.instance.metadataMap.biancheng=zhangsan

上述代码定义了一个 key 为 biancheng 的配置，value 是 zhangsan。重启服务，然后通过 Eureka 提供的 REST API 来查看刚刚配置的元数据是否已经存在于 Eureka 中，如图 2 所示。


![自定义元数据查看](https://typoralim.oss-cn-beijing.aliyuncs.com/img/5-1ZR01I051239.png)

 图 2 自定义元数据查看

## 	EurekaClient 使用

当我们的项目中集成了 Eureka 之后，可以通过 EurekaClient 来获取一些我们想要的数据，比如刚刚上面讲的元数据。我们就可以直接通过 EurekaClient 来获取（代码如下所示），不用再去调用 Eureka 提供的 REST API。

```java
@Autowired
private EurekaClient eurekaClient;
@GetMapping("/article/infos")
public Object serviceUrl() {
    return eurekaClient.getInstancesByVipAddress( "eureka-client-user-service", false);
}
```

通过 PostMan 来调用接口看看有没有返回我们想要的数据。这时我们会发现，通过 EurekaClient 获取的数据跟我们自己去掉 API 获取的数据是一样的，从使用角度来说前者比较方便。

 除了使用 EurekaClient，还可以使用 DiscoveryClient（代码如下所示），这个不是 Feign 自带的，是 Spring Cloud 重新封装的，类的路径为 org.springframework.cloud.client.discovery.DiscoveryClient。

```java
@Autowired
private DiscoveryClient discoveryClient;
@GetMapping("/article/infos")
public Object serviceUrl() {
    return discoveryClient.getInstances("eureka-client-user-service");
}
```

## 	健康检查

默认情况下，Eureka 客户端是使用心跳和服务端通信来判断客户端是否存活，在某些场景下，比如 [MongoDB](http://c.biancheng.net/mongodb/) 出现了异常，但你的应用进程还是存在的，这就意味着应用可以继续通过心跳上报，保持应用自己的信息在 Eureka 中不被剔除掉。

 Spring Boot Actuator 提供了 /actuator/health 端点，该端点可展示应用程序的健康信息，当 MongoDB 异常时，/actuator/health 端点的状态会变成 DOWN，由于应用本身确实处于存活状态，但是 MongoDB 的异常会影响某些功能，当请求到达应用之后会发生操作失败的情况。

 在这种情况下，我们希望可以将健康信息传递给 Eureka 服务端。这样 Eureka 中就能及时将应用的实例信息下线，隔离正常请求，防止出错。通过配置如下内容开启健康检查：

​	eureka.client.healthcheck.enabled=true

我们可以通过扩展健康检查的端点来模拟异常情况，定义一个扩展端点，将状态设置为 DOWN，代码如下所示。

```java
@Component
public class CustomHealthIndicator extends AbstractHealthIndicator {
    @Override
    protected void doHealthCheck(Builder builder) throws Exception {
        builder.down().withDetail("status", false);
    }
}
```

扩展好后我们访问 /actuator/health 可以看到当前的状态是 DOWN，如图 3 所示。
![查看应用健康状态](https://typoralim.oss-cn-beijing.aliyuncs.com/img/5-1ZR01K20G11.png)

 图 3 查看应用健康状态


 Eureka 中的状态是 UP，这种情况下请求还是能转发到这个服务中，下面我们开启监控检查，再次查看 Eureka 中的状态，发现状态变为 DOWN(1)。

## 	服务上下线监控

在某些特定的需求下，我们需要对服务的上下线进行监控，上线或下线都进行邮件通知，Eureka 中提供了事件监听的方式来扩展。

 目前支持的事件如下：

- ​		EurekaInstanceCanceledEvent 服务下线事件。
- ​		EurekaInstanceRegisteredEvent 服务注册事件。
- ​		EurekaInstanceRenewedEvent 服务续约事件。
- ​		EurekaRegistryAvailableEvent Eureka 注册中心启动事件。
- ​		EurekaServerStartedEvent Eureka Server 启动事件。


 基于 Eureka 提供的事件机制，可以监控服务的上下线过程，在过程发生中可以发送邮件来进行通知。下面代码只是演示了监控的过程，并未发送邮件。

```java
@Component
public class EurekaStateChangeListener {
    @EventListener
    public void listen(EurekaInstanceCanceledEvent event) {
        System.err.println(event.getServerId() + "\t" + event.getAppName() + " 服务下线 ");
    }
    @EventListener
    public void listen(EurekaInstanceRegisteredEvent event) {
        InstanceInfo instanceInfo = event.getInstanceInfo();
        System.err.println(instanceInfo.getAppName() + " 进行注册 ");
    }
    @EventListener
    public void listen(EurekaInstanceRenewedEvent event) {
        System.err.println(event.getServerId() + "\t" + event.getAppName() + " 服务进行续约 ");
    }
    @EventListener
    public void listen(EurekaRegistryAvailableEvent event) {
        System.err.println(" 注册中心启动 ");
    }
    @EventListener
    public void listen(EurekaServerStartedEvent event) {
        System.err.println("Eureka Server启动 ");
    }
}
```

注意：在 Eureka 集群环境下，每个节点都会触发事件，这个时候需要控制下发送通知的行为，不控制的话每个节点都会发送通知。

# Eureka控制台快速查看Swagger文档

在服务很多的情况下，我们想通过 Eureka 中注册的实例信息，能够直接跳转到 API 文档页面，这个时候可以定义 Eureka 的 Page 地址。在 application.properties 中增加如下配置即可：

```
eureka.instance.status-page-url=http://${spring.cloud.client.ip-address}: ${server.port}/swagger-ui.html
```

在 Eureka Web 控制台就可以直接点击注册的实例跳转到 Swagger 文档页面了，如图 1 所示。



![Eureka自定义Swagger主页地址](https://typoralim.oss-cn-beijing.aliyuncs.com/img/5-1ZS0160S03M.png)
图 1 Eureka 自定义 Swagger 主页地址

## 请求认证

当我们的服务中有认证的逻辑，程序中会把认证的 Token 设置到请求头中，在用 Swagger 测试接口的时候也需要带上 Token 才能完成接口的测试。

点击 Authorize 按钮（如图 2 所示），填写认证信息（如图 3 所示）。



![Authorize入口按钮](Spring Cloud Eureka.assets/5-1ZS016121a63.png)
图 2 Authorize 入口按钮


![Authorize信息填写](https://typoralim.oss-cn-beijing.aliyuncs.com/img/5-1ZS01613251M.png)
图 3 Authorize 信息填写


默认的请求头名称是 Token，这里改成了 Authorization，通过配置文件修改：

```
swagger.authorization.key-name=Authorization
```

# 参考

http://c.biancheng.net/view/5324.html

https://www.jianshu.com/p/16d90b0b0a10

https://www.cnblogs.com/zyon/p/11023750.html

https://blog.csdn.net/hry2015/article/details/82597311?utm_medium=distribute.pc_relevant.none-task-blog-baidujs-2

https://www.cnblogs.com/jing99/p/11576133.html



