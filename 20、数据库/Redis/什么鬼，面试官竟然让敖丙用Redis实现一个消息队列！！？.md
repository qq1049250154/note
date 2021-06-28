[GitHub 9.4k Star 的Java工程师成神之路 ，不来了解一下吗?](https://github.com/hollischuang/toBeTopJavaer)

[GitHub 9.4k Star 的Java工程师成神之路 ，真的不来了解一下吗?](https://github.com/hollischuang/toBeTopJavaer)

[GitHub 9.4k Star 的Java工程师成神之路 ，真的确定不来了解一下吗?](https://github.com/hollischuang/toBeTopJavaer)

众所周知，redis是一个高性能的**key-value**数据库，在NoSQL数据库市场上，redis自己就占据了将近半壁江山，足以见到其强大之处。同时，由于redis的单线程特性，我们可以将其用作为一个**消息队列**。本篇文章就来讲讲如何将redis整合到spring boot中，并用作消息队列的……

# 一、什么是消息队列

> “消息队列”是在消息的传输过程中保存消息的容器。——《百度百科》

**消息**我们可以理解为在计算机中或在整个计算机网络中传递的数据。

**队列**是我们在学习数据结构的时候学习的基本数据结构之一，它具有先进先出的特性。

所以，**消息队列**就是一个保存消息的容器，它具有先进先出的特性。

## 为什么会出现消息队列？

1. 异步：常见的B/S架构下，客户端向服务器发送请求，但是服务器处理这个消息需要花费的时间很长的时间，如果客户端一直等待服务器处理完消息，会造成客户端的系统资源浪费；而使用消息队列后，服务器直接将消息推送到消息队列中，由专门的处理消息程序处理消息，这样客户端就不必花费大量时间等待服务器的响应了；
2. 解耦：传统的软件开发模式，模块之间的调用是直接调用，这样的系统很不利于系统的扩展，同时，模块之间的相互调用，数据之间的共享问题也很大，每个模块都要时时刻刻考虑其他模块会不会挂了；使用消息队列以后，模块之间不直接调用，而是通过数据，且当某个模块挂了以后，数据仍旧会保存在消息队列中。最典型的就是**生产者-消费者**模式，本案例使用的就是该模式；
3. 削峰填谷：某一时刻，系统的并发请求暴增，远远超过了系统的最大处理能力后，如果不做任何处理，系统会崩溃；使用消息队列以后，服务器把请求推送到消息队列中，由专门的处理消息程序以合理的速度消费消息，降低服务器的压力。

下面一张图我们来简单了解一下消息队列

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210220134238.jpeg)

由上图可以看到，消息队列充当了一个中间人的角色，我们可以通过操作这个消息队列来保证我们的系统稳定。

# 二、环境准备

> Java环境：jdk1.8
>
> spring boot版本：2.2.1.RELEASE
>
> redis-server版本：3.2.100

# 三、相关依赖

这里只展示与redis相关的依赖，

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.integration</groupId>
    <artifactId>spring-integration-redis</artifactId>
</dependency>
12345678
```

这里解释一下这两个依赖：

- 第一个依赖是对redis NoSQL的支持
- 第二个依赖是spring integration与redis的结合，这里添加这个代码主要是为了实现分布式锁

# 四、配置文件

这里只展示与redis相关的配置

```
redis所在的的地址
spring.redis.host=localhost
redis数据库索引，从0开始，可以从redis的可视化客户端查看
spring.redis.database=1
redis的端口，默认为6379
spring.redis.port=6379
redis的密码
spring.redis.password=
连接redis的超时时间(ms)，默认是2000
spring.redis.timeout=5000
连接池最大连接数
spring.redis.jedis.pool.max-active=16
连接池最小空闲连接
spring.redis.jedis.pool.min-idle=0
连接池最大空闲连接
spring.redis.jedis.pool.max-idle=16
连接池最大阻塞等待时间（负数表示没有限制）
spring.redis.jedis.pool.max-wait=-1
连接redis的客户端名
spring.redis.client-name=mall
123456789101112131415161718192021
```

# 五、代码配置

redis用作消息队列，其在spring boot中的主要表现为一个`RedisTemplate.convertAndSend()`方法和一个`MessageListener`接口。所以我们要在IOC容器中注入一个`RedisTemplate`和一个实现了`MessageListener`接口的类。话不多说，先看代码

## 配置RedisTemplate

配置RedisTemplate的主要目的是配置序列化方式以解决乱码问题，同时合理配置序列化方式还能降低一点性能开销。

```
/**
 * 配置RedisTemplate，解决乱码问题
 */
@Bean
public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
    LOGGER.debug("redis序列化配置开始");
    RedisTemplate<String, Object> template = new RedisTemplate<>();
    template.setConnectionFactory(factory);
    // string序列化方式
    RedisSerializer serializer = new GenericJackson2JsonRedisSerializer();
    // 设置默认序列化方式
    template.setDefaultSerializer(serializer);
    template.setKeySerializer(new StringRedisSerializer());
    template.setHashValueSerializer(serializer);
    LOGGER.debug("redis序列化配置结束");
    return template;
}
1234567891011121314151617
```

代码第12行，我们配置默认的序列化方式为`GenericJackson2JsonRedisSerializer`

代码第13行，我们配置键的序列化方式为`StringRedisSerializer`

代码第14行，我们配置哈希表的值的序列化方式为`GenericJackson2JsonRedisSerializer`

### RedisTemplate几种序列化方式的简要介绍

| 序列化方式                           | 介绍                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| `StringRedisSerializer`              | 将对象序列化为字符串，但是经测试，无法序列化对象，一般用在key上 |
| `OxmSerializer`                      | 将对象序列化为xml性质，本质上是字符串                        |
| `ByteArrayRedisSerializer`           | 默认序列化方式，将对象序列化为二进制字节，但是需要对象实现Serializable接口 |
| `GenericFastJsonRedisSerializer`     | json序列化，使用fastjson序列化方式序列化对象                 |
| `GenericJackson2JsonRedisSerializer` | json序列化，使用jackson序列化方式序列化对象                  |

# 六、redis队列监听器（消费者）

上面说了，与redis队列监听器相关的类为一个名为`MessageListener`的接口，下面是该接口的源码

```
public interface MessageListener {
    void onMessage(Message message, @Nullable byte[] pattern);
}
123
```

可以看到，该接口仅有一个`onMessage(Message message, @Nullable byte[] pattern)`方法，该方法便是监听到队列中消息后的回调方法。下面解释一下这两个参数：

- message：redis消息类，该类中仅有两个方法
  - `byte[] getBody()`以二进制形式获取消息体
  - `byte[] getChannel()`以二进制形式获取消息通道
- pattern：二进制形式的消息通道，和`message.getChannel()`返回值相同

介绍完接口，我们来实现一个简单的redis队列监听器

```
@Component
public class RedisListener implement MessageListener{
    private static final Logger LOGGER = LoggerFactory.getLogger(RedisListener.class);

    @Override
    public void onMessage(Message message,byte[] pattern){
        LOGGER.debug("从消息通道={}监听到消息",new String(pattern));
        LOGGER.debug("从消息通道={}监听到消息",new String(message.getChannel()));
        LOGGER.debug("元消息={}",new String(message.getBody()));
        // 新建一个用于反序列化的对象，注意这里的对象要和前面配置的一样
        // 因为我前面设置的默认序列化方式为GenericJackson2JsonRedisSerializer
        // 所以这里的实现方式为GenericJackson2JsonRedisSerializer
        RedisSerializer serializer=new GenericJackson2JsonRedisSerializer();
        LOGGER.debug("反序列化后的消息={}",serializer.deserialize(message.getBody()));
    }
}
12345678910111213141516
```

代码很简单，就是输出参数中包含的关键信息。需要注意的是，`RedisSerializer`的实现要与上面配置的序列化方式一致。

队列监听器实现完以后，我们还需要将这个监听器添加到redis队列监听器容器中，代码如下：

```
@Bean
public public RedisMessageListenerContainer container(RedisConnectionFactory factory) {
    RedisMessageListenerContainer container = new RedisMessageListenerContainer();
    container.setConnectionFactory(factory);
    container.addMessageListener(redisListener, new PatternTopic("demo-channel"));
    return container;
}
1234567
```

这几行代码大概意思就是新建一个Redis消息监听器容器，然后将监听器和管道名想绑定，最后返回这个容器。

这里要注意的是，这个管道名和下面将要说的推送消息时的管道名要一致，不然监听器监听不到消息。

# 七、redis队列推送服务（生产者）

上面我们配置了RedisTemplate将要在这里使用到。

代码如下：

```
@Service
public class Publisher{
    @Autowrite
    private RedisTemplate redis;

    public void publish(Object msg){
        redis.convertAndSend("demo-channel",msg);
    }
}
123456789
```

关键代码为第7行，`redis.convertAndSend()`这个方法的作用为，向某个通道（参数1）推送一条消息（第二个参数）。

这里还是要注意上面所说的，生产者和消费者的通道名要相同。

至此，消息队列的生产者和消费者已经全部编写完成。

# 八、遇到的问题及解决办法

## 1、spring boot使用log4j2日志框架问题

在我添加了`spring-boot-starter-log4j2`依赖并在`spring-boot-starter-web`中排除了`spring-boot-starter-logging`后，运行项目，还是会提示下面的错误：

```
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:.....m2/repository/ch/qos/logback/logback-classic/1.2.3/logback-classic-1.2.3.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:.....m2/repository/org/apache/logging/log4j/log4j-slf4j-impl/2.12.1/log4j-slf4j-impl-2.12.1.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [ch.qos.logback.classic.util.ContextSelectorStaticBinder]
12345
```

这个错误就是maven中有多个日志框架导致的。后来通过依赖分析，发现在`spring-boot-starter-data-redis`中，也依赖了`spring-boot-starter-logging`，解决办法也很简单，下面贴出详细代码

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.integration</groupId>
    <artifactId>spring-integration-redis</artifactId>
</dependency>
12345678910111213141516171819202122232425262728
```

## 2、redis队列监听器线程安全问题

redis队列监听器的监听机制是：使用一个线程监听队列，队列有未消费的消息则取出消息并生成一个新的线程来消费消息。如果你还记得，我开头说的是由于redis单线程特性，因此我们用它来做消息队列，但是如果监听器每次接受一个消息就生成新的线程来消费信息的话，这样就完全没有使用到redis的单线程特性，同时还会产生线程安全问题。

### 单一消费者（一个通道只有一个消费者）的解决办法

最简单的办法莫过于为`onMessage()`方法加锁，这样简单粗暴却很有用，不过这种方式无法控制队列监听的速率，且无限制的创造线程最终会导致系统资源被占光。

那如何解决这种情况呢？线程池。

在将监听器添加到容器的配置的时候，`RedisMessageListenerContainer`类中有一个方法`setTaskExecutor(Executor taskExecutor)`可以为监听容器配置线程池。配置线程池以后，所有的线程都会由该线程池产生，由此，我们可以通过调节线程池来控制队列监听的速率。

### 多个消费者（一个通道有多个消费者）的解决办法

单一消费者的问题相比于多个消费者来说还是较为简单，因为Java内置的锁都是只能控制自己程序的运行，不能干扰其他的程序的运行；然而现在很多时候我们都是在分布式环境下进行开发，这时处理多个消费者的情况就很有意义了。

那么这种问题如何解决呢？分布式锁。

下面来简要科普一下什么是分布式锁：

> 分布式锁是指在分布式环境下，同一时间只有一个客户端能够从某个共享环境中（例如redis）获取到锁，只有获取到锁的客户端才能执行程序。
>
> 然后分布式锁一般要满足：排他性（即同一时间只有一个客户端能够获取到锁）、避免死锁（即超时后自动释放）、高可用（即获取或释放锁的机制必须高可用且性能佳）

上面讲依赖的时候，我们导入了一个`spring-integration-redis`依赖，这个依赖里面包含了很多实用的工具类，而我们接下来要讲的分布式锁就是这个依赖下面的一个工具包`RedisLockRegistry`。

首先讲一下如何使用，导入了依赖以后，首先配置一个Bean

```
@Bean
public RedisLockRegistry redisLockRegistry(RedisConnectionFactory factory) {
    return new RedisLockRegistry(factory, "demo-lock",60);
}
1234
```

`RedisLockRegistry`的构造函数，第一个参数是redis连接池，第二个参数是锁的前缀，即取出的锁，键名为“demo-lock:KEY_NAME”，第三个参数为锁的过期时间（秒），默认为60秒，当持有锁超过该时间后自动过期。

使用锁的方法，下面是对监听器的修改

```
@Component
public class RedisListener implement MessageListener{
    @Autowrite
    private RedisLockRegistry redisLockRegistry;

    private static final Logger LOGGER = LoggerFactory.getLogger(RedisListener.class);

    @Override
    public void onMessage(Message message,byte[] pattern){
        Lock lock=redisLockRegistry.obtain("lock");
        try{
            lock.lock(); //上锁
            LOGGER.debug("从消息通道={}监听到消息",new String(pattern));
            LOGGER.debug("从消息通道={}监听到消息",new String(message.getChannel()));
            LOGGER.debug("元消息={}",new String(message.getBody()));
            // 新建一个用于反序列化的对象，注意这里的对象要和前面配置的一样
            // 因为我前面设置的默认序列化方式为GenericJackson2JsonRedisSerializer
            // 所以这里的实现方式为GenericJackson2JsonRedisSerializer
            RedisSerializer serializer=new GenericJackson2JsonRedisSerializer();
            LOGGER.debug("反序列化后的消息={}",serializer.deserialize(message.getBody()));
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock(); //解锁
        }
    }
}
123456789101112131415161718192021222324252627
```

上面代码的代码比起前面的监听器代码，只是多了一个注入的`RedisLockRegistry`，一个通过`redisLockRegistry.obtain()`方法获取锁，一个加锁一个解锁，然后这就完成了分布式锁的使用。

注意这个获取锁的方法`redisLockRegistry.obtain()`，其返回的是一个名为RedisLock的锁，这是一个私有内部类，它实现了Lock接口，因此我们不能从代码外部创建一个他的实例，只能通过obtian()方法来获取这个锁。