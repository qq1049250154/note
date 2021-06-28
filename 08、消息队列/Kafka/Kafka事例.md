# kafka安装

**一、下载、安装Kafka**

访问Kafka的主页：

[Apache Kafkakafka.apache.org<img src="https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210427105151.png" alt="图标" style="zoom:33%;" />](https://link.zhihu.com/?target=http%3A//kafka.apache.org/)

进入其下载页面，截图如下：

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210427105154.jpeg)

选择相应的版本，这里选择 kafka_2.11-2.4.0.tgz，进入下面的页面：

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210427105157.jpeg)

选择清华的镜像站点进行下载。

下载到本地后，将文件解压到 D:\kafka_2.11-2.4.0，该文件夹包括了所有相关的运行文件及配置文件，其子文件夹bin\windows 下放的是在Windows系统启动zookeeper和kafka的可执行文件，子文件夹config下放的是zookeeper和kafka的配置文件。



**二、启动kafka服务**

我们需要先后启动zookeeper和kafka服务。

它们都需要进入 D:\kafka_2.11-2.4.0 目录，然后再启动相应的命令。

```text
cd D:\kafka_2.11-2.4.0
```

启动zookeeper服务，运行命令：

```text
bin\windows\zookeeper-server-start.bat config\zookeeper.properties
```

启动kafka服务，运行命令：

```text
bin\windows\kafka-server-start.bat config\server.properties
```

**三、创建Topic，显示数据**

Kafka中创建一个Topic，名称为iris

```text
bin\windows\kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic iris
```

创建成功后，可以使用如下命令，显示所有Topic的列表：

```text
bin\windows\kafka-topics.bat --list --zookeeper localhost:2181 
```

显示结果为

```text
iris
```

然后，我们通过Alink的 Kafka SinkStreamOp可以将iris数据集写入该Topic。这里不详细展开，有兴趣的读者可以参阅如下的文章。

[Alink品数：Alink连接Kafka数据源（Python版本）zhuanlan.zhihu.com![图标](https://pic4.zhimg.com/zhihu-card-default_ipico.jpg)](https://zhuanlan.zhihu.com/p/101143978)[Alink品数：Alink连接Kafka数据源（Java版本）zhuanlan.zhihu.com![图标](https://pic4.zhimg.com/zhihu-card-default_ipico.jpg)](https://zhuanlan.zhihu.com/p/101106492)



使用如下命令，读取（消费）topic iris中的数据：

```bash
 bin\windows\kafka-console-consumer.bat --bootstrap-server 127.0.0.1:9092 --topic iris --from-beginning
```

显示结果如下，略去了中间的大部分数据：

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210427105159.jpeg)

```text
{"sepal_width":3.4,"petal_width":0.2,"sepal_length":4.8,"category":"Iris-setosa","petal_length":1.6}
{"sepal_width":4.1,"petal_width":0.1,"sepal_length":5.2,"category":"Iris-setosa","petal_length":1.5}
{"sepal_width":2.8,"petal_width":1.5,"sepal_length":6.5,"category":"Iris-versicolor","petal_length":4.6}
{"sepal_width":3.0,"petal_width":1.8,"sepal_length":6.1,"category":"Iris-virginica","petal_length":4.9}
{"sepal_width":2.9,"petal_width":1.8,"sepal_length":7.3,"category":"Iris-virginica","petal_length":6.3}
...........
{"sepal_width":2.2,"petal_width":1.0,"sepal_length":6.0,"category":"Iris-versicolor","petal_length":4.0}
{"sepal_width":2.4,"petal_width":1.0,"sepal_length":5.5,"category":"Iris-versicolor","petal_length":3.7}
{"sepal_width":3.1,"petal_width":0.2,"sepal_length":4.6,"category":"Iris-setosa","petal_length":1.5}
{"sepal_width":3.4,"petal_width":0.2,"sepal_length":4.8,"category":"Iris-setosa","petal_length":1.9}
{"sepal_width":2.9,"petal_width":1.4,"sepal_length":6.1,"category":"Iris-versicolor","petal_length":4.7}
```

# SpringBoot集成kafka全面实战.

本文是SpringBoot+Kafka的实战讲解，如果对kafka的架构原理还不了解的读者，建议先看一下[《大白话kafka架构原理》](http://mp.weixin.qq.com/s?__biz=MzU1NDA0MDQ3MA==&mid=2247483958&idx=1&sn=dffaad318b50f875eea615bc3bdcc80c&chksm=fbe8efcfcc9f66d9ff096fbae1c2a3671f60ca4dc3e7412ebb511252e7193a46dcd4eb11aadc&scene=21#wechat_redirect)、[《秒懂kafka HA（高可用）》](http://mp.weixin.qq.com/s?__biz=MzU1NDA0MDQ3MA==&mid=2247483965&idx=1&sn=20dd02c4bf3a11ff177906f0527a5053&chksm=fbe8efc4cc9f66d258c239fefe73125111a351d3a4e857fd8cd3c98a5de2c18ad33aacdad947&scene=21#wechat_redirect)两篇文章。

一、生产者实践

- 普通生产者
- 带回调的生产者
- 自定义分区器
- kafka事务提交

二、消费者实践

- 简单消费
- 指定topic、partition、offset消费
- 批量消费
- 监听异常处理器
- 消息过滤器
- 消息转发
- 定时启动/停止监听器

## 一、前戏

1、在项目中连接kafka，因为是外网，首先要开放kafka配置文件中的如下配置（其中IP为公网IP），

```
advertised.listeners=PLAINTEXT://112.126.74.249:9092
```

2、在开始前我们先创建两个topic：topic1、topic2，其分区和副本数都设置为2，用来测试，

```shell
[root@iZ2zegzlkedbo3e64vkbefZ ~]#  cd /usr/local/kafka-cluster/kafka1/bin/
[root@iZ2zegzlkedbo3e64vkbefZ bin]# ./kafka-topics.sh --create --zookeeper 172.17.80.219:2181 --replication-factor 2 --partitions 2 --topic topic1
Created topic topic1.
[root@iZ2zegzlkedbo3e64vkbefZ bin]# ./kafka-topics.sh --create --zookeeper 172.17.80.219:2181 --replication-factor 2 --partitions 2 --topic topic2
Created topic topic2.
```

当然我们也可以不手动创建topic，在执行代码kafkaTemplate.send("topic1", normalMessage)发送消息时，kafka会帮我们自动完成topic的创建工作，但这种情况下创建的topic默认只有一个分区，分区也没有副本。所以，我们可以在项目中新建一个配置类专门用来初始化topic，如下，

```java
@Configuration
public class KafkaInitialConfiguration {
    // 创建一个名为testtopic的Topic并设置分区数为8，分区副本数为2
    @Bean
    public NewTopic initialTopic() {
        return new NewTopic("testtopic",8, (short) 2 );
    }

     // 如果要修改分区数，只需修改配置值重启项目即可
    // 修改分区数并不会导致数据的丢失，但是分区数只能增大不能减小
    @Bean
    public NewTopic updateTopic() {
        return new NewTopic("testtopic",10, (short) 2 );
    }
}
```

3、新建SpringBoot项目

① 引入pom依赖

```xml
<dependency>
 	<groupId>org.springframework.kafka</groupId>
	<artifactId>spring-kafka</artifactId>
</dependency>
```

② application.propertise配置（本文用到的配置项这里全列了出来）

```
###########【Kafka集群】###########
spring.kafka.bootstrap-servers=112.126.74.249:9092,112.126.74.249:9093
###########【初始化生产者配置】###########
# 重试次数
spring.kafka.producer.retries=0
# 应答级别:多少个分区副本备份完成时向生产者发送ack确认(可选0、1、all/-1)
spring.kafka.producer.acks=1
# 批量大小
spring.kafka.producer.batch-size=16384
# 提交延时
spring.kafka.producer.properties.linger.ms=0
# 当生产端积累的消息达到batch-size或接收到消息linger.ms后,生产者就会将消息提交给kafka
# linger.ms为0表示每接收到一条消息就提交给kafka,这时候batch-size其实就没用了
# 生产端缓冲区大小
spring.kafka.producer.buffer-memory = 33554432
# Kafka提供的序列化和反序列化类
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer
# 自定义分区器
# spring.kafka.producer.properties.partitioner.class=com.felix.kafka.producer.CustomizePartitioner
###########【初始化消费者配置】###########
# 默认的消费组ID
spring.kafka.consumer.properties.group.id=defaultConsumerGroup
# 是否自动提交offset
spring.kafka.consumer.enable-auto-commit=true
# 提交offset延时(接收到消息后多久提交offset)
spring.kafka.consumer.auto.commit.interval.ms=1000
# 当kafka中没有初始offset或offset超出范围时将自动重置offset
# earliest:重置为分区中最小的offset;
# latest:重置为分区中最新的offset(消费分区中新产生的数据);
# none:只要有一个分区不存在已提交的offset,就抛出异常;
spring.kafka.consumer.auto-offset-reset=latest
# 消费会话超时时间(超过这个时间consumer没有发送心跳,就会触发rebalance操作)
spring.kafka.consumer.properties.session.timeout.ms=120000
# 消费请求超时时间
spring.kafka.consumer.properties.request.timeout.ms=180000
# Kafka提供的序列化和反序列化类
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer
# 消费端监听的topic不存在时，项目启动会报错(关掉)
spring.kafka.listener.missing-topics-fatal=false
# 设置批量消费
# spring.kafka.listener.type=batch
# 批量消费每次最多消费多少条消息
# spring.kafka.consumer.max-poll-records=50
```

## 二、Hello Kafka

1、简单生产者

```java
@RestController
public class KafkaProducer {
    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;
    // 发送消息
    @GetMapping("/kafka/normal/{message}")
    public void sendMessage1(@PathVariable("message") String normalMessage) {
        kafkaTemplate.send("topic1", normalMessage);
    }
}
```

 2、简单消费

```java
@Component
public class KafkaConsumer {

    // 消费监听



    @KafkaListener(topics = {"topic1"})



    public void onMessage1(ConsumerRecord<?, ?> record){



        // 消费的哪个topic、partition的消息,打印出消息内容



        System.out.println("简单消费："+record.topic()+"-"+record.partition()+"-"+record.value());



    }



}
```

上面示例创建了一个生产者，发送消息到topic1，消费者监听topic1消费消息。监听器用@KafkaListener注解，topics表示监听的topic，支持同时监听多个，用英文逗号分隔。启动项目，postman调接口触发生产者发送消息，

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL3N6X21tYml6X3BuZy80aWNWdnd0d1ZleUZDTkpkdGhuVFZuYnJwOUxJMXd1djNhdVV6azR4U0JPdTZneGljMk1oSzV0bWdSNHd3NWQzUG9EeERCS2tUUGNDQjdwTTJYbzBTdzZRLzY0MA?x-oss-process=image/format,png)

可以看到监听器消费成功，

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL3N6X21tYml6X3BuZy80aWNWdnd0d1ZleUZDTkpkdGhuVFZuYnJwOUxJMXd1djNwc09pYzY3aWNYV2lid0k0a0ptV2VpYURpY01OVTB6QUxyR3dGVWd2aWI2c09pYjRtOU5zRllTeEMxcmdBLzY0MA?x-oss-process=image/format,png)

## 三、生产者

1、带回调的生产者

kafkaTemplate提供了一个回调方法addCallback，我们可以在回调方法中监控消息是否发送成功 或 失败时做补偿处理，有两种写法，

```java
@GetMapping("/kafka/callbackOne/{message}")



public void sendMessage2(@PathVariable("message") String callbackMessage) {



    kafkaTemplate.send("topic1", callbackMessage).addCallback(success -> {



        // 消息发送到的topic



        String topic = success.getRecordMetadata().topic();



        // 消息发送到的分区



        int partition = success.getRecordMetadata().partition();



        // 消息在分区内的offset



        long offset = success.getRecordMetadata().offset();



        System.out.println("发送消息成功:" + topic + "-" + partition + "-" + offset);



    }, failure -> {



        System.out.println("发送消息失败:" + failure.getMessage());



    });



}
@GetMapping("/kafka/callbackTwo/{message}")



public void sendMessage3(@PathVariable("message") String callbackMessage) {



    kafkaTemplate.send("topic1", callbackMessage).addCallback(new ListenableFutureCallback<SendResult<String, Object>>() {



        @Override



        public void onFailure(Throwable ex) {



            System.out.println("发送消息失败："+ex.getMessage());



        }



 



        @Override



        public void onSuccess(SendResult<String, Object> result) {



            System.out.println("发送消息成功：" + result.getRecordMetadata().topic() + "-"



                    + result.getRecordMetadata().partition() + "-" + result.getRecordMetadata().offset());



        }



    });



}
```

2、自定义分区器

我们知道，kafka中每个topic被划分为多个分区，那么生产者将消息发送到topic时，具体追加到哪个分区呢？这就是所谓的分区策略，Kafka 为我们提供了默认的分区策略，同时它也支持自定义分区策略。其路由机制为：

① 若发送消息时指定了分区（即自定义分区策略），则直接将消息append到指定分区；

② 若发送消息时未指定 patition，但指定了 key（kafka允许为每条消息设置一个key），则对key值进行hash计算，根据计算结果路由到指定分区，这种情况下可以保证同一个 Key 的所有消息都进入到相同的分区；

③  patition 和 key 都未指定，则使用kafka默认的分区策略，轮询选出一个 patition；

※ 我们来自定义一个分区策略，将消息发送到我们指定的partition，首先新建一个分区器类实现Partitioner接口，重写方法，其中partition方法的返回值就表示将消息发送到几号分区，

```java
public class CustomizePartitioner implements Partitioner {    @Override    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {        // 自定义分区规则(这里假设全部发到0号分区)        // ......        return 0;    }    @Override    public void close() {    }    @Override    public void configure(Map<String, ?> configs) {    }}
```

在application.propertise中配置自定义分区器，配置的值就是分区器类的全路径名，

```
# 自定义分区器spring.kafka.producer.properties.partitioner.class=com.felix.kafka.producer.CustomizePartitioner
```

3、kafka事务提交

如果在发送消息时需要创建事务，可以使用 KafkaTemplate 的 executeInTransaction 方法来声明事务，

```java
@GetMapping("/kafka/transaction")public void sendMessage7(){    // 声明事务：后面报错消息不会发出去    kafkaTemplate.executeInTransaction(operations -> {        operations.send("topic1","test executeInTransaction");        throw new RuntimeException("fail");    });    // 不声明事务：后面报错但前面消息已经发送成功了   kafkaTemplate.send("topic1","test executeInTransaction");   throw new RuntimeException("fail");}
```

## 四、消费者

1、指定topic、partition、offset消费

前面我们在监听消费topic1的时候，监听的是topic1上所有的消息，如果我们想指定topic、指定partition、指定offset来消费呢？也很简单，@KafkaListener注解已全部为我们提供，

```java
/** * @Title 指定topic、partition、offset消费 * @Description 同时监听topic1和topic2，监听topic1的0号分区、topic2的 "0号和1号" 分区，指向1号分区的offset初始值为8 * @Author long.yuan * @Date 2020/3/22 13:38 * @Param [record] * @return void **/@KafkaListener(id = "consumer1",groupId = "felix-group",topicPartitions = {        @TopicPartition(topic = "topic1", partitions = { "0" }),        @TopicPartition(topic = "topic2", partitions = "0", partitionOffsets = @PartitionOffset(partition = "1", initialOffset = "8"))})public void onMessage2(ConsumerRecord<?, ?> record) {    System.out.println("topic:"+record.topic()+"|partition:"+record.partition()+"|offset:"+record.offset()+"|value:"+record.value());}
```

属性解释：

① id：消费者ID；

② groupId：消费组ID；

③ topics：监听的topic，可监听多个；

④ topicPartitions：可配置更加详细的监听信息，可指定topic、parition、offset监听。

上面onMessage2监听的含义：监听topic1的0号分区，同时监听topic2的0号分区和topic2的1号分区里面offset从8开始的消息。

注意：topics和topicPartitions不能同时使用；

2、批量消费

设置application.prpertise开启批量消费即可，

```
# 设置批量消费spring.kafka.listener.type=batch# 批量消费每次最多消费多少条消息spring.kafka.consumer.max-poll-records=50
```

接收消息时用List来接收，监听代码如下，

```java
@KafkaListener(id = "consumer2",groupId = "felix-group", topics = "topic1")public void onMessage3(List<ConsumerRecord<?, ?>> records) {    System.out.println(">>>批量消费一次，records.size()="+records.size());    for (ConsumerRecord<?, ?> record : records) {        System.out.println(record.value());    }}
```

3、ConsumerAwareListenerErrorHandler 异常处理器

通过异常处理器，我们可以处理consumer在消费时发生的异常。

新建一个 ConsumerAwareListenerErrorHandler 类型的异常处理方法，用@Bean注入，BeanName默认就是方法名，然后我们将这个异常处理器的BeanName放到@KafkaListener注解的errorHandler属性里面，当监听抛出异常的时候，则会自动调用异常处理器，

```java
// 新建一个异常处理器，用@Bean注入@Beanpublic ConsumerAwareListenerErrorHandler consumerAwareErrorHandler() {    return (message, exception, consumer) -> {        System.out.println("消费异常："+message.getPayload());        return null;    };}// 将这个异常处理器的BeanName放到@KafkaListener注解的errorHandler属性里面@KafkaListener(topics = {"topic1"},errorHandler = "consumerAwareErrorHandler")public void onMessage4(ConsumerRecord<?, ?> record) throws Exception {    throw new Exception("简单消费-模拟异常");}// 批量消费也一样，异常处理器的message.getPayload()也可以拿到各条消息的信息@KafkaListener(topics = "topic1",errorHandler="consumerAwareErrorHandler")public void onMessage5(List<ConsumerRecord<?, ?>> records) throws Exception {    System.out.println("批量消费一次...");    throw new Exception("批量消费-模拟异常");}
```

执行看一下效果，

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL3N6X21tYml6X3BuZy80aWNWdnd0d1ZleUZDTkpkdGhuVFZuYnJwOUxJMXd1djNzaWJWNFpVUzNZQjJEQkVSc01PM1pZaG9RNjlCQmlidU1scTNpY1QwYm40bm9wNGliWWlhODNXZE9wQS82NDA?x-oss-process=image/format,png)

4、消息过滤器

消息过滤器可以在消息抵达consumer之前被拦截，在实际应用中，我们可以根据自己的业务逻辑，筛选出需要的信息再交由KafkaListener处理，不需要的消息则过滤掉。

配置消息过滤只需要为 监听器工厂 配置一个RecordFilterStrategy（消息过滤策略），返回true的时候消息将会被抛弃，返回false时，消息能正常抵达监听容器。

```java
@Componentpublic class KafkaConsumer {    @Autowired    ConsumerFactory consumerFactory;    // 消息过滤器    @Bean    public ConcurrentKafkaListenerContainerFactory filterContainerFactory() {        ConcurrentKafkaListenerContainerFactory factory = new ConcurrentKafkaListenerContainerFactory();        factory.setConsumerFactory(consumerFactory);        // 被过滤的消息将被丢弃        factory.setAckDiscarded(true);        // 消息过滤策略        factory.setRecordFilterStrategy(consumerRecord -> {            if (Integer.parseInt(consumerRecord.value().toString()) % 2 == 0) {                return false;            }            //返回true消息则被过滤            return true;        });        return factory;    }    // 消息过滤监听    @KafkaListener(topics = {"topic1"},containerFactory = "filterContainerFactory")    public void onMessage6(ConsumerRecord<?, ?> record) {        System.out.println(record.value());    }}
```

上面实现了一个"过滤奇数、接收偶数"的过滤策略，我们向topic1发送0-99总共100条消息，看一下监听器的消费情况，可以看到监听器只消费了偶数，

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL3N6X21tYml6X3BuZy80aWNWdnd0d1ZleUZDTkpkdGhuVFZuYnJwOUxJMXd1djNBV1hkQ1ppYTkzR1VmZjBpYnBUcmJpY0o2cHZhbEFXMUJTdmtkR2FhR0lNWmljaWFpYTdPbXJhZFZxOXcvNjQw?x-oss-process=image/format,png)

5、消息转发

在实际开发中，我们可能有这样的需求，应用A从TopicA获取到消息，经过处理后转发到TopicB，再由应用B监听处理消息，即一个应用处理完成后将该消息转发至其他应用，完成消息的转发。

在SpringBoot集成Kafka实现消息的转发也很简单，只需要通过一个@SendTo注解，被注解方法的return值即转发的消息内容，如下，

```java
/** * @Title 消息转发 * @Description 从topic1接收到的消息经过处理后转发到topic2 * @Author long.yuan * @Date 2020/3/23 22:15 * @Param [record] * @return void **/@KafkaListener(topics = {"topic1"})@SendTo("topic2")public String onMessage7(ConsumerRecord<?, ?> record) {    return record.value()+"-forward message";}
```

6、定时启动、停止监听器

默认情况下，当消费者项目启动的时候，监听器就开始工作，监听消费发送到指定topic的消息，那如果我们不想让监听器立即工作，想让它在我们指定的时间点开始工作，或者在我们指定的时间点停止工作，该怎么处理呢——使用KafkaListenerEndpointRegistry，下面我们就来实现：

① 禁止监听器自启动；

② 创建两个定时任务，一个用来在指定时间点启动定时器，另一个在指定时间点停止定时器；

新建一个定时任务类，用注解@EnableScheduling声明，KafkaListenerEndpointRegistry 在SpringIO中已经被注册为Bean，直接注入，设置禁止KafkaListener自启动，

```java
@EnableScheduling@Componentpublic class CronTimer {    /**     * @KafkaListener注解所标注的方法并不会在IOC容器中被注册为Bean，     * 而是会被注册在KafkaListenerEndpointRegistry中，     * 而KafkaListenerEndpointRegistry在SpringIOC中已经被注册为Bean     **/    @Autowired    private KafkaListenerEndpointRegistry registry;    @Autowired    private ConsumerFactory consumerFactory;    // 监听器容器工厂(设置禁止KafkaListener自启动)    @Bean    public ConcurrentKafkaListenerContainerFactory delayContainerFactory() {        ConcurrentKafkaListenerContainerFactory container = new ConcurrentKafkaListenerContainerFactory();        container.setConsumerFactory(consumerFactory);        //禁止KafkaListener自启动        container.setAutoStartup(false);        return container;    }    // 监听器    @KafkaListener(id="timingConsumer",topics = "topic1",containerFactory = "delayContainerFactory")    public void onMessage1(ConsumerRecord<?, ?> record){        System.out.println("消费成功："+record.topic()+"-"+record.partition()+"-"+record.value());    }    // 定时启动监听器    @Scheduled(cron = "0 42 11 * * ? ")    public void startListener() {        System.out.println("启动监听器...");        // "timingConsumer"是@KafkaListener注解后面设置的监听器ID,标识这个监听器        if (!registry.getListenerContainer("timingConsumer").isRunning()) {            registry.getListenerContainer("timingConsumer").start();        }        //registry.getListenerContainer("timingConsumer").resume();    }    // 定时停止监听器    @Scheduled(cron = "0 45 11 * * ? ")    public void shutDownListener() {        System.out.println("关闭监听器...");        registry.getListenerContainer("timingConsumer").pause();    }}
```

启动项目，触发生产者向topic1发送消息，可以看到consumer没有消费，因为这时监听器还没有开始工作，

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL3N6X21tYml6X3BuZy80aWNWdnd0d1ZleUZDTkpkdGhuVFZuYnJwOUxJMXd1djMySE1QbzdtOFlRQzZsVTVwT21mWmNTREtobTR0cUtMVzQzVXNUOTQ2aWM1NXhUQ2VNdlVXbmpnLzY0MA?x-oss-process=image/format,png)

11:42分监听器启动开始工作，消费消息，

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL3N6X21tYml6X3BuZy80aWNWdnd0d1ZleUZDTkpkdGhuVFZuYnJwOUxJMXd1djN1Njg0cXJabkl2aWFWUmZIOU0waWJkUEdLaHJCdW5kV2ljaWJpYW5rbVd1U2lhUzIwWEg3aWJXMWdPSmdRLzY0MA?x-oss-process=image/format,png)

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL3N6X21tYml6X3BuZy80aWNWdnd0d1ZleUZDTkpkdGhuVFZuYnJwOUxJMXd1djN2dnhZc1lhN2lic2xnbGtaSmE1WWdaMFE1eU8yZ0c5TVhKQlJiSXB5ZUNQeUd2S0tyZXR5NFZBLzY0MA?x-oss-process=image/format,png)

11：45分监听器停止工作，

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL3N6X21tYml6X3BuZy80aWNWdnd0d1ZleUZDTkpkdGhuVFZuYnJwOUxJMXd1djN6QXY5UGRkaWJ6bmczQ3U1QTNDOTVva1dpY2RaZ21tUU0waWFYVGxxeTJaM0Zjd0dsUFk3UGFTWVEvNjQw?x-oss-process=image/format,png)

 

**感兴趣的可以关注一下博主的公众号，1W+技术人的选择，致力于原创技术干货，包含Redis、RabbitMQ、Kafka、SpringBoot、SpringCloud、ELK等热门技术的学习&资料。**

# SpringBoot整合Kafka

## 简单使用

2.1 引入依赖

主要是spring-kafka依赖

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!--优化编码-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
```

**application.properties 添加变量参数**

设置配置参数，主题，topic等

```properties
kafka.bootstrap-servers=localhost:9092

kafka.topic.basic=test_topic
kafka.topic.json=json_topic
kafka.topic.batch=batch_topic
kafka.topic.manual=manual_topic

kafka.topic.transactional=transactional_topic
kafka.topic.reply=reply_topic
kafka.topic.reply.to=reply_to_topic
kafka.topic.filter=filter_topic
kafka.topic.error=error_topic

server.port=9093
```



2.2 Kafka配置

此处我们可以在application.properties中配置，也可以使用Java Config。我使用Java Config，看得更直观。

2.2.1 生产者

**配置类 StringProducerConfig.java**

```java
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.serialization.StringSerializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import java.util.HashMap;
import java.util.Map;

@Configuration
public class StringProducerConfig {
    @Value("${kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Bean
    public KafkaTemplate<String, String> stringKafkaTemplate() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        configProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        configProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);

        // -----------------------------额外配置，可选--------------------------
        //重试，0为不启用重试机制
        configProps.put(ProducerConfig.RETRIES_CONFIG, 1);
        //控制批处理大小，单位为字节
        configProps.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);
        //批量发送，延迟为1毫秒，启用该功能能有效减少生产者发送消息次数，从而提高并发量
        configProps.put(ProducerConfig.LINGER_MS_CONFIG, 1);
        //生产者可以使用的总内存字节来缓冲等待发送到服务器的记录
        configProps.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 1024000);

        return new KafkaTemplate<>(new DefaultKafkaProducerFactory<>(configProps));

    }

    /**
     * ----可选参数----
     *
     * configProps.put(ProducerConfig.ACKS_CONFIG, "1");
     * 确认模式, 默认1
     *
     * acks=0那么生产者将根本不会等待来自服务器的任何确认。
     * 记录将立即被添加到套接字缓冲区，并被认为已发送。在这种情况下，不能保证服务器已经收到了记录，
     * 并且<code>重试</code>配置不会生效(因为客户端通常不会知道任何故障)。每个记录返回的偏移量总是设置为-1。
     *
     * acks=1这将意味着领导者将记录写入其本地日志，但不会等待所有追随者的全部确认。
     * 在这种情况下，如果领导者在确认记录后立即失败，但在追随者复制之前，记录将会丢失。
     *
     * acks=all这些意味着leader将等待所有同步的副本确认记录。这保证了只要至少有一个同步副本仍然存在，
     * 记录就不会丢失。这是最有力的保证。这相当于acks=-1的设置。
     *
     *
     *
     * configProps.put(ProducerConfig.RETRIES_CONFIG, "3");
     * 设置一个大于零的值将导致客户端重新发送任何发送失败的记录，并可能出现暂时错误。
     * 请注意，此重试与客户机在收到错误后重新发送记录没有什么不同。
     * 如果不将max.in.flight.requests.per.connection 设置为1，则允许重试可能会更改记录的顺序，
     * 因为如果将两个批发送到单个分区，而第一个批失败并重试，但第二个批成功，则第二批中的记录可能会首先出现。
     * 注意：另外，如果delivery.timeout.ms 配置的超时在成功确认之前先过期，则在重试次数用完之前，生成请求将失败。
     *
     *
     * 其他参数请参考：http://www.shixinke.com/java/kafka-configuration
     * https://blog.csdn.net/xiaozhu_you/article/details/91493258
     */

}
```

**生产者 StringProducer.java**

```java
import com.itcloud.itcloudkafka.springTemplate.basicString.listener.KafkaSendResultHandler;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Component;
import java.util.concurrent.ExecutionException;

@Component
public class StringProducer {
    @Autowired
    @Qualifier("stringKafkaTemplate")
    private KafkaTemplate<String, String> kafkaTemplate;

    @Autowired
    private KafkaSendResultHandler kafkaSendResultHandler;

    @Value("${kafka.topic.basic}")
    private String basicTopic;

    public void send(String message) {
        kafkaTemplate.send(basicTopic, message);
    }

    /**
     * 异步发送
     * @param message
     */
    public void sendAsync(String message) {
        kafkaTemplate.send(basicTopic, message);
    }

    /**
     * 发送回调
     * @param message
     */
    public void sendAndCallback(String message) {
        // 配置发送回调，可选
        kafkaTemplate.setProducerListener(kafkaSendResultHandler);
        kafkaTemplate.send(basicTopic, message);
    }

    /**
     *  同步发送，默认异步
     * @param message
     */
    public void sendSync(String message) {
        try {
            kafkaTemplate.send(basicTopic, message).get();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }

}
```



**生产者回调 KafkaSendResultHandler.java**

```java
import lombok.extern.slf4j.Slf4j;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;
import org.springframework.kafka.support.ProducerListener;
import org.springframework.stereotype.Component;

/**
 * @author 司马缸砸缸了
 * @date 2019/12/30 10:53
 * @description 消息回调监听器
 */
@Component
@Slf4j
public class KafkaSendResultHandler implements ProducerListener {
    @Override
    public void onSuccess(ProducerRecord producerRecord, RecordMetadata recordMetadata) {
        log.info("Message send success : " + producerRecord.toString());
    }

    @Override
    public void onError(ProducerRecord producerRecord, Exception exception) {
        log.info("Message send error : " + exception);
    }

    /**
     *
     * @return true 代表成功也会调用onSuccess，默认为false
     */
    @Override
    public boolean isInterestedInSuccess() {
        return false;
    }

}
```



2.2.2 消费者

**配置类 StringConsumerConfig.java**

```java
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.EnableKafka;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.config.KafkaListenerContainerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import java.util.HashMap;
import java.util.Map;

@Configuration
@EnableKafka
public class StringConsumerConfig {
    @Value("${kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Value("${kafka.topic.transactional}")
    private String topic;

    /**
     * 单线程-单条消费
     * @return
     */
    @Bean
    public KafkaListenerContainerFactory<?> stringKafkaListenerContainerFactory() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        configProps.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        configProps.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        configProps.put(ConsumerConfig.GROUP_ID_CONFIG, topic);

        ConcurrentKafkaListenerContainerFactory<String, String> factory =
                new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(new DefaultKafkaConsumerFactory<>(configProps));

        return factory;
    }

//    创建topic，3个分区，每个分区一个副本
//    @Bean
//    public NewTopic batchTopic() {
//        return new NewTopic(topic, 3, (short) 1);
//    }

}
```



**消费者 StringConsumer.java**

```java
import lombok.extern.slf4j.Slf4j;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.support.KafkaHeaders;
import org.springframework.messaging.handler.annotation.Header;
import org.springframework.messaging.handler.annotation.Payload;
import org.springframework.stereotype.Component;

/**
 *  @KafkaListener多种使用方式
 */
@Component
@Slf4j
public class StringConsumer {

    @KafkaListener(topics = "${kafka.topic.basic}", containerFactory = "stringKafkaListenerContainerFactory")
    public void receiveString(String message) {
        System.out.println(String.format("Message : %s", message));
    }

    /**
     * 注解方式获取消息头及消息体
     *
     * @Payload：获取的是消息的消息体，也就是发送内容
     * @Header(KafkaHeaders.RECEIVED_MESSAGE_KEY)：获取发送消息的key
     * @Header(KafkaHeaders.RECEIVED_PARTITION_ID)：获取当前消息是从哪个分区中监听到的
     * @Header(KafkaHeaders.RECEIVED_TOPIC)：获取监听的TopicName
     * @Header(KafkaHeaders.RECEIVED_TIMESTAMP)：获取时间戳
     *
     */
    //    @KafkaListener(topics = "${kafka.topic.basic}", containerFactory = "stringKafkaListenerContainerFactory")
    public void receive(@Payload String message,
                        @Header(KafkaHeaders.RECEIVED_MESSAGE_KEY) Integer key,
                        @Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition,
                        @Header(KafkaHeaders.RECEIVED_TOPIC) String topic,
                        @Header(KafkaHeaders.RECEIVED_TIMESTAMP) long ts) {
        System.out.println(String.format("From partition %d : %s", partition, message));
    }

    /**
     * 指定消费分区和初始偏移量
     *
     * @TopicPartition：topic--需要监听的Topic的名称，partitions --需要监听Topic的分区id，partitionOffsets --可以设置从某个偏移量开始监听
     * @PartitionOffset：partition --分区Id，非数组，initialOffset --初始偏移量
     *
     */
//    @KafkaListener(
//            containerFactory = "stringKafkaListenerContainerFactory",
//            topicPartitions = @TopicPartition(
//                    topic = "${kafka.topic.basic}",
//                    partitionOffsets = @PartitionOffset(
//                            partition = "0" ,
//                            initialOffset = "0")))
    public void receiveFromBegin(@Payload String payload,
                                 @Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition) {
        System.out.println(String.format("Read all message from partition %d : %s", partition, payload));
    }

    /**
     * ConsumerRecord 接收
     *
     * @param record
     */
//    @KafkaListener(topics = "${kafka.topic.basic}", containerFactory = "stringKafkaListenerContainerFactory")
    public void receive(ConsumerRecord<?, ?> record) {
        System.out.println("Message is :" + record.toString());
    }

}
```

消费者中使用多种方式@KafkaListener进行消费，注释已经很详细了。  
参考博客：https://www.jianshu.com/p/a64defb44a23

2.2.3 测试

**运行**

```java
@Autowired
    private StringProducer producer;

	@Test
    public void stringProducer() {
        for (int i = 0; i < 5; i++) {
            producer.send("Message【" + i + "】:my name is simagangzagangl");
        }

        try {
            Thread.sleep(1000 * 2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
```



**结果**

![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210427093013.png)

## 发送Java Bean消息

2.1 引入依赖

同上

2.2 Java Bean

定义一个对象

```java
@Data
public class JsonBean {
    private String messageId;
    private String messageContent;
    public JsonBean(){}

    public JsonBean(String messageId , String messageContent){
        this.messageId = messageId;
        this.messageContent = messageContent;
    }

    public String toString() {
        return String.format("%s:%s", messageId, messageContent);
    }
}
```



2.3 Kafka配置

此处我们可以在application.properties中配置，也可以使用Java Config。我使用Java Config，看得更直观。

2.3.1 生产者

**配置类 JsonProducerConfig.java**

```java
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.serialization.StringSerializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.support.serializer.JsonSerializer;
import java.util.HashMap;
import java.util.Map;

/**
 * 司马缸砸缸了
 * 生产者配置
 */
@Configuration
public class JsonProducerConfig {
    @Value("${kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Bean
    public KafkaTemplate<String, JsonBean> jsonKafkaTemplate() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        configProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        // value使用Json序列化
        configProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);

        return new KafkaTemplate<>(new DefaultKafkaProducerFactory<>(configProps));
    }
}
```

**生产者 JsonProducer.java**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Component;

@Component
public class JsonProducer {
    @Autowired
    @Qualifier("jsonKafkaTemplate")
    private KafkaTemplate<String, JsonBean> kafkaTemplate;

    @Value("${kafka.topic.json}")
    private String topic;

    public void send(JsonBean json) {
        kafkaTemplate.send(topic, json);
    }

}
```

2.3.2 消费者

**配置类 JsonConsumerConfig.java**

```java
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.EnableKafka;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.core.ConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import org.springframework.kafka.support.serializer.JsonDeserializer;
import java.util.HashMap;
import java.util.Map;

@Configuration
@EnableKafka
public class JsonConsumerConfig {
    @Value("${kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Value("${kafka.topic.basic}")
    private String topic;

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, JsonBean> basicKafkaListenerContainerFactory() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        configProps.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        // json反序列化
        configProps.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class);
        configProps.put(ConsumerConfig.GROUP_ID_CONFIG, topic);

        ConcurrentKafkaListenerContainerFactory<String, JsonBean> factory = new ConcurrentKafkaListenerContainerFactory<>();
        ConsumerFactory<String, JsonBean> basicConsumerFactory =
                new DefaultKafkaConsumerFactory<>(configProps,
                        new StringDeserializer(),
                        new JsonDeserializer<>(JsonBean.class));
        factory.setConsumerFactory(basicConsumerFactory);

        return factory;
    }
}
```



**消费者 JsonConsumer.java**

```java
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;

@Component
public class JsonConsumer {

    @KafkaListener(topics = "${kafka.topic.json}", containerFactory = "basicKafkaListenerContainerFactory")
    public void receive(JsonBean basic) {
        System.out.println("receive:" + basic.toString());
    }
}
```

使用多种方式@KafkaListener进行消费，参考博客：https://www.jianshu.com/p/a64defb44a23

2.3.3 测试

**运行**

```java
@Autowired
    private JsonProducer jsonProducer;

	@Test
    public void jsonProducer() {
        JsonBean json = new JsonBean();
        json.setMessageId("1");
        json.setMessageContent("this is message");
        jsonProducer.send(json);

        try {
            Thread.sleep(1000 * 2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
```



**结果**

![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210427093157.png)

## 批量消费

开启批量消费主要3步

1.  消费者设置 max.poll.records
2.  消费者 开启批量消费 factory.setBatchListener(true);
3.  消费者批量接收 public void consumerBatch(List<ConsumerRecord<?, ?>> records)

2.1 引入依赖

同上

2.2 Kafka配置

此处我们可以在application.properties中配置，也可以使用Java Config。我使用Java Config，看得更直观。

2.2.1 生产者

**配置类 BatchProducerConfig.java**

```prism
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.serialization.StringSerializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import java.util.HashMap;
import java.util.Map;

@Configuration
public class BatchProducerConfig {
    @Value("${kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Bean
    public KafkaTemplate<String, String> batchKafkaTemplate() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        configProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        configProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);

        // -----------------------------额外配置，可选--------------------------
        // 重试，0为不启用重试机制
        configProps.put(ProducerConfig.RETRIES_CONFIG, 0);
        // 控制批处理大小，单位为字节。(批量发送)
        // 当批次被填满时，批次里的所有消息被发送出去，不过生产者不一定都会等到批次被填满才发送，半满的或者
        // 一个消息的批次也可能被发送。
        configProps.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);
        // 批量发送，延迟为1毫秒，启用该功能能有效减少生产者发送消息次数，从而提高并发量（批量发送）
        // 该参数指定了生产者在发送批次之前等待更多消息加入批次的时间。
        // 批次填满或者linger.ms达到上限时把批次发送出去。
        configProps.put(ProducerConfig.LINGER_MS_CONFIG, 1);
        // 生产者可以使用的总内存字节来缓冲等待发送到服务器的记录
        configProps.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 1024000);

        return new KafkaTemplate<>(new DefaultKafkaProducerFactory<>(configProps));

    }

    /**
     * ----可选参数----
     *
     * configProps.put(ProducerConfig.ACKS_CONFIG, "1");
     * 确认模式, 默认1
     *
     * acks=0那么生产者将根本不会等待来自服务器的任何确认。
     * 记录将立即被添加到套接字缓冲区，并被认为已发送。在这种情况下，不能保证服务器已经收到了记录，
     * 并且<code>重试</code>配置不会生效(因为客户端通常不会知道任何故障)。每个记录返回的偏移量总是设置为-1。
     *
     * acks=1这将意味着领导者将记录写入其本地日志，但不会等待所有追随者的全部确认。
     * 在这种情况下，如果领导者在确认记录后立即失败，但在追随者复制之前，记录将会丢失。
     *
     * acks=all这些意味着leader将等待所有同步的副本确认记录。这保证了只要至少有一个同步副本仍然存在，
     * 记录就不会丢失。这是最有力的保证。这相当于acks=-1的设置。
     *
     *
     *
     * configProps.put(ProducerConfig.RETRIES_CONFIG, "3");
     * 设置一个大于零的值将导致客户端重新发送任何发送失败的记录，并可能出现暂时错误。
     * 请注意，此重试与客户机在收到错误后重新发送记录没有什么不同。
     * 如果不将max.in.flight.requests.per.connection 设置为1，则允许重试可能会更改记录的顺序，
     * 因为如果将两个批发送到单个分区，而第一个批失败并重试，但第二个批成功，则第二批中的记录可能会首先出现。
     * 注意：另外，如果delivery.timeout.ms 配置的超时在成功确认之前先过期，则在重试次数用完之前，生成请求将失败。
     *
     *
     * 其他参数：参考：http://www.shixinke.com/java/kafka-configuration
     * https://blog.csdn.net/xiaozhu_you/article/details/91493258
     */
}
```



**生产者 BatchProducer.java**

```prism
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Component;
import java.util.concurrent.ExecutionException;

@Component
public class BatchProducer {
    @Autowired
    @Qualifier("batchKafkaTemplate")
    private KafkaTemplate<String, String> kafkaTemplate;

    @Value("${kafka.topic.batch}")
    private String batchTopic;

    /**
     * 异步发送
     * @param message
     */
    public void send(String message) {
        kafkaTemplate.send(batchTopic, message);
    }

    /**
     *  同步发送，默认异步
     * @param message
     */
    public void sendSync(String message) {
        try {
            kafkaTemplate.send(batchTopic, message).get();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }

}
```



2.2.2 消费者

**配置类 BatchConsumerConfig.java**

```prism
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.EnableKafka;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.config.KafkaListenerContainerFactory;
import org.springframework.kafka.core.ConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import java.util.HashMap;
import java.util.Map;

@Configuration
@EnableKafka
public class BatchConsumerConfig {
    @Value("${kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Value("${kafka.topic.batch}")
    private String topic;

    /**
     * 多线程-批量消费
     * @return
     */
    @Bean
    public KafkaListenerContainerFactory<?> batchFactory(){
        ConcurrentKafkaListenerContainerFactory<String, String> factory =
                new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        // 控制多线程消费
        // 并发数(如果topic有3各分区。设置成3，并发数就是3个线程，加快消费)
        // 不设置setConcurrency就会变成单线程配置, MAX_POLL_RECORDS_CONFIG也会失效，
        // 接收的消息列表也不会是ConsumerRecord
        factory.setConcurrency(10);
        // poll超时时间
        factory.getContainerProperties().setPollTimeout(1500);
        // 控制批量消费
        // 设置为批量消费，每个批次数量在Kafka配置参数中设置（max.poll.records）
        factory.setBatchListener(true);
        return factory;
    }

    public ConsumerFactory<String, String> consumerFactory() {
        return new DefaultKafkaConsumerFactory<>(consumerConfigs());
    }

    /**
     * 消费者配置
     * @return
     */
    public Map<String, Object> consumerConfigs() {
        Map<String, Object> configProps = new HashMap<>();
        // 不用指定全部的broker，它将自动发现集群中的其余的borker, 最好指定多个，万一有服务器故障
        configProps.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        // key序列化方式
        configProps.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        // value序列化方式
        configProps.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        // GroupID
        configProps.put(ConsumerConfig.GROUP_ID_CONFIG, "test-group");
        // 批量消费消息数量
        configProps.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, 2);

        // -----------------------------额外配置，可选--------------------------

        // 自动提交偏移量
        // 如果设置成true,偏移量由auto.commit.interval.ms控制自动提交的频率
        // 如果设置成false,不需要定时的提交offset，可以自己控制offset，当消息认为已消费过了，这个时候再去提交它们的偏移量。
        // 这个很有用的，当消费的消息结合了一些处理逻辑，这个消息就不应该认为是已经消费的，直到它完成了整个处理。
        configProps.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");
        // 自动提交的频率
        configProps.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "1000");
        // Session超时设置
        configProps.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, "15000");

        // 该属性指定了消费者在读取一个没有偏移量的分区或者偏移量无效的情况下该作何处理：
        // latest（默认值）在偏移量无效的情况下，消费者将从最新的记录开始读取数据（在消费者启动之后生成的记录）
        // earliest ：在偏移量无效的情况下，消费者将从起始位置读取分区的记录
        configProps.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "latest");
        return configProps;
    }
}
```



**消费者 BatchConsumer.java**

```prism
import lombok.extern.slf4j.Slf4j;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;
import java.util.List;
import java.util.Optional;

@Component
@Slf4j
public class BatchConsumer {

    /**
     * 批量消息
     * @param records
     */
    @KafkaListener(topics = "${kafka.topic.batch}", containerFactory="batchFactory")
    public void consumerBatch(List<ConsumerRecord<?, ?>> records){
        log.info("接收到消息数量：{}",records.size());
        for(ConsumerRecord record: records) {
            Optional<?> kafkaMessage = Optional.ofNullable(record.value());
            log.info("Received: " + record);
            if (kafkaMessage.isPresent()) {
                Object message = record.value();
                String topic = record.topic();
                System.out.println("接收到消息：" + message);
            }
        }
    }

}
```



2.2.3 测试

**运行**

```java
@Autowired
    private BatchProducer batchProducer;

	@Test
    public void batchProducer() {
        for (int i = 0; i < 5; i++) {
            batchProducer.send("Message【" + i + "】:哈哈哈，我是批处理消息");
        }

        try {
            Thread.sleep(1000 * 2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
```



**结果**

![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210427093243.png)

## 自动，手动提交偏移量

**一. 简介**

我们可以把偏移量交给kafka去提交，也可以自己控制提交时机。例如消息处理过程缓慢的情况下，就需要自己控制偏移量何时提交比较好。

**二. 自动提交偏移量**

自动提交偏移量就是消费者配置中添加以下两个配置就可以，上篇博客《批量消费》就是自动提交偏移量，在此不在重复。  
**消费者配置类**

```prism
// 自动提交偏移量
        // 如果设置成true,偏移量由auto.commit.interval.ms控制自动提交的频率
        // 如果设置成false,不需要定时的提交offset，可以自己控制offset，当消息认为已消费过了，这个时候再去提交它们的偏移量。
        // 这个很有用的，当消费的消息结合了一些处理逻辑，这个消息就不应该认为是已经消费的，直到它完成了整个处理。
        configProps.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");
        // 自动提交的频率
        configProps.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "1000");
```



**三. 手动提交偏移量**

主要步骤：

1.  消费者配置 configProps.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, “false”);
2.  消费者配置ack模式factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.MANUAL_IMMEDIATE);
3.  消费者手动提交 consumer.commitSync();

**3.1 引入依赖**

主要是spring-kafka依赖

同上

**application.properties 添加变量参数**

设置配置参数，主题，topic等

同上

3.2 Kafka配置

此处我们可以在application.properties中配置，也可以使用Java Config。我使用Java Config，看得更直观。

3.2.1 生产者

**配置类 ManualProducerConfig.java**

```prism
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.serialization.StringSerializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import java.util.HashMap;
import java.util.Map;

/**
 * 手动提交偏移量
 */
@Configuration
public class ManualProducerConfig {
    @Value("${kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Bean
    public KafkaTemplate<String, String> manualKafkaTemplate() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        configProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        configProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        return new KafkaTemplate<>(new DefaultKafkaProducerFactory<>(configProps));

    }

    /**
     * ----可选参数----
     *
     * configProps.put(ProducerConfig.ACKS_CONFIG, "1");
     * 确认模式, 默认1
     *
     * acks=0那么生产者将根本不会等待来自服务器的任何确认。
     * 记录将立即被添加到套接字缓冲区，并被认为已发送。在这种情况下，不能保证服务器已经收到了记录，
     * 并且<code>重试</code>配置不会生效(因为客户端通常不会知道任何故障)。每个记录返回的偏移量总是设置为-1。
     *
     * acks=1这将意味着领导者将记录写入其本地日志，但不会等待所有追随者的全部确认。
     * 在这种情况下，如果领导者在确认记录后立即失败，但在追随者复制之前，记录将会丢失。
     *
     * acks=all这些意味着leader将等待所有同步的副本确认记录。这保证了只要至少有一个同步副本仍然存在，
     * 记录就不会丢失。这是最有力的保证。这相当于acks=-1的设置。
     *
     *
     *
     * configProps.put(ProducerConfig.RETRIES_CONFIG, "3");
     * 设置一个大于零的值将导致客户端重新发送任何发送失败的记录，并可能出现暂时错误。
     * 请注意，此重试与客户机在收到错误后重新发送记录没有什么不同。
     * 如果不将max.in.flight.requests.per.connection 设置为1，则允许重试可能会更改记录的顺序，
     * 因为如果将两个批发送到单个分区，而第一个批失败并重试，但第二个批成功，则第二批中的记录可能会首先出现。
     * 注意：另外，如果delivery.timeout.ms 配置的超时在成功确认之前先过期，则在重试次数用完之前，生成请求将失败。
     *
     *
     * 其他参数：参考：http://www.shixinke.com/java/kafka-configuration
     * https://blog.csdn.net/xiaozhu_you/article/details/91493258
     */

}
```



**生产者 ManualProducer.java**

```prism
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Component;
import java.util.concurrent.ExecutionException;

@Component
public class ManualProducer {
    @Autowired
    @Qualifier("manualKafkaTemplate")
    private KafkaTemplate<String, String> kafkaTemplate;

    @Value("${kafka.topic.manual}")
    private String basicTopic;

    /**
     * 异步发送
     * @param message
     */
    public void send(String message) {
        kafkaTemplate.send(basicTopic, message);
    }

    /**
     *  同步发送，默认异步
     * @param message
     */
    public void sendSync(String message) {
        try {
            kafkaTemplate.send(basicTopic, message).get();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }

}
```



3.2.2 消费者

**配置类 ManualConsumerConfig.java**

```prism
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.EnableKafka;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.config.KafkaListenerContainerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import org.springframework.kafka.listener.ContainerProperties;
import java.util.HashMap;
import java.util.Map;

@Configuration
@EnableKafka
public class ManualConsumerConfig {
    @Value("${kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Value("${kafka.topic.manual}")
    private String topic;

    @Bean
    public KafkaListenerContainerFactory<?> manualKafkaListenerContainerFactory() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        configProps.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        configProps.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        configProps.put(ConsumerConfig.GROUP_ID_CONFIG, "manual-group");
        // 手动提交
        configProps.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false");

        ConcurrentKafkaListenerContainerFactory<String, String> factory =
                new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(new DefaultKafkaConsumerFactory<>(configProps));
        // ack模式，详细见下文注释
        factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.MANUAL_IMMEDIATE);

        return factory;
    }

    /**
     * AckMode针对ENABLE_AUTO_COMMIT_CONFIG=false时生效，有以下几种：
     *
     * RECORD
     * 每处理一条commit一次
     *
     * BATCH(默认)
     * 每次poll的时候批量提交一次，频率取决于每次poll的调用频率
     *
     * TIME
     * 每次间隔ackTime的时间去commit(跟auto commit interval有什么区别呢？)
     *
     * COUNT
     * 累积达到ackCount次的ack去commit
     *
     * COUNT_TIME
     * ackTime或ackCount哪个条件先满足，就commit
     *
     * MANUAL
     * listener负责ack，但是背后也是批量上去
     *
     * MANUAL_IMMEDIATE
     * listner负责ack，每调用一次，就立即commit
     *
     */

}
```



**消费者 ManualConsumer.java**

```prism
import lombok.extern.slf4j.Slf4j;
import org.apache.kafka.clients.consumer.Consumer;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.OffsetAndMetadata;
import org.apache.kafka.common.TopicPartition;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.support.Acknowledgment;
import org.springframework.kafka.support.KafkaHeaders;
import org.springframework.messaging.handler.annotation.Header;
import org.springframework.messaging.handler.annotation.Payload;
import org.springframework.stereotype.Component;
import java.util.*;

@Component
@Slf4j
public class ManualConsumer {

    @KafkaListener(topics = "${kafka.topic.manual}", containerFactory = "manualKafkaListenerContainerFactory")
    public void receive(@Payload String message,
                        @Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition,
                        @Header(KafkaHeaders.RECEIVED_TOPIC) String topic,
                        Consumer consumer,
                        Acknowledgment ack) {
        System.out.println(String.format("From partition %d : %s", partition, message));
        // 同步提交
        consumer.commitSync();

        // ack这种方式提交也可以
        // ack.acknowledge();
    }

    /**
     * commitSync和commitAsync组合使用
     * <p>
     * 手工提交异步 consumer.commitAsync();
     * 手工同步提交 consumer.commitSync()
     * <p>
     * commitSync()方法提交最后一个偏移量。在成功提交或碰到无怯恢复的错误之前，
     * commitSync()会一直重试，但是commitAsync()不会。
     * <p>
     * 一般情况下，针对偶尔出现的提交失败，不进行重试不会有太大问题，因为如果提交失败是因为临时问题导致的，
     * 那么后续的提交总会有成功的。但如果这是发生在关闭消费者或再均衡前的最后一次提交，就要确保能够提交成功。
     * 因此，在消费者关闭前一般会组合使用commitAsync()和commitSync()。
     */
//    @KafkaListener(topics = "${kafka.topic.manual}", containerFactory = "manualKafkaListenerContainerFactory")
    public void manual(@Payload String message,
                       @Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition,
                       @Header(KafkaHeaders.RECEIVED_TOPIC) String topic,
                       Consumer consumer,
                       Acknowledgment ack) {
        try {
            System.out.println(String.format("From partition %d : %s", partition, message));
            // 同步提交
            consumer.commitSync();
        } catch (Exception e) {
            System.out.println("commit failed");
        } finally {
            try {
                consumer.commitSync();
            } finally {
                consumer.close();
            }
        }

    }

    /**
     * 手动提交，指定偏移量
     *
     * @param record
     * @param consumer
     */
//    @KafkaListener(topics = "${kafka.topic.manual}", containerFactory = "manualKafkaListenerContainerFactory")
    public void offset(ConsumerRecord record, Consumer consumer) {
        System.out.println(String.format("From partition %d : %s", record.partition(), record.value()));

        Map<TopicPartition, OffsetAndMetadata> currentOffset = new HashMap<>();
        currentOffset.put(new TopicPartition(record.topic(), record.partition()),
                new OffsetAndMetadata(record.offset() + 1));
        consumer.commitSync(currentOffset);
    }

}
```



3.2.3 测试

**运行**

```prism
@Autowired
    private ManualProducer manualProducer;

	@Test
    public void manualProducer() {
        manualProducer.send("我就是我，是手动提交偏移量");

        try {
            Thread.sleep(1000 * 2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
```



**结果**

![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210427093247.png)

## 事物

一. 简介

事务不止用在数据库中，kafka中我们并不希望消息监听器接收到一些错误的消息，使用事务就会进行回滚，不会发送到topic中。

有两种实现事务方式：

1.  @Transactional注解
2.  使用KafkaTemplate.executeInTransaction开启事务

二. @Transactional事务

开启批量消费主要3步

1.  生产者开启事务 factory.transactionCapable();
2.  生产者初始化事务管理器KafkaTransactionManager
3.  生产者添加@Transactional

2.1 引入依赖

主要是spring-kafka依赖

同上

**application.properties 添加变量参数**

设置配置参数，主题，topic等

同上

2.2 Kafka配置

此处我们可以在application.properties中配置，也可以使用Java Config。我使用Java Config，看得更直观。

2.2.1 生产者

**配置类 TransactionalProducerConfig.java**

```java
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.serialization.StringSerializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.core.ProducerFactory;
import org.springframework.kafka.transaction.KafkaTransactionManager;
import java.util.HashMap;
import java.util.Map;

@Configuration
public class TransactionalProducerConfig {
    @Value("${kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Bean
    public KafkaTemplate<String, String> transactionalTemplate() {
        KafkaTemplate template = new KafkaTemplate<String, String>(transactionalProducerFactory());
        return template;
    }

    /**
     * 生产者配置
     * @return
     */
    private Map<String, Object> configs() {
        Map<String, Object> props = new HashMap<>();
        // 连接地址
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        // 键的序列化方式
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        // 值的序列化方式
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        // 重试，0为不启用重试机制
        props.put(ProducerConfig.RETRIES_CONFIG, 1);
        // 控制批处理大小，单位为字节
        props.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);
        // 批量发送，延迟为1毫秒，启用该功能能有效减少生产者发送消息次数，从而提高并发量
        props.put(ProducerConfig.LINGER_MS_CONFIG, 1);
        // 生产者可以使用的总内存字节来缓冲等待发送到服务器的记录
        props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 1024000);
        return props;
    }

    @Bean
    public ProducerFactory<String, String> transactionalProducerFactory() {
        DefaultKafkaProducerFactory factory = new DefaultKafkaProducerFactory<>(configs());
        // 开启事务
        factory.transactionCapable();
        // 用来生成Transactional.id的前缀
        factory.setTransactionIdPrefix("tran-");
        return factory;
    }

    /**
     * 事务管理器
     * @param transactionalProducerFactory
     * @return
     */
    @Bean
    public KafkaTransactionManager transactionManager(ProducerFactory transactionalProducerFactory) {
        KafkaTransactionManager manager = new KafkaTransactionManager(transactionalProducerFactory);
        return manager;
    }
}
```



**生产者 TransactionalProducer.java**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.kafka.core.KafkaOperations;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

@Component
public class TransactionalProducer {
    @Autowired
    @Qualifier("errorTemplate")
    private KafkaTemplate<String, String> kafkaTemplate;

    @Value("${kafka.topic.transactional}")
    private String topic;

    /**
     * 事务发送
     * @param message
     */
    @Transactional
    public void send(String message) {
        kafkaTemplate.send(topic, message);
        throw new RuntimeException("fail");
    }

    /**
     * 使用KafkaTemplate.executeInTransaction开启事务
     * 
     * 本地事务，不需要事务管理器
     * @param message
     * @throws InterruptedException
     */
    public void testExecuteInTransaction(String message) throws InterruptedException {
        kafkaTemplate.executeInTransaction(new KafkaOperations.OperationsCallback() {
            @Override
            public Object doInOperations(KafkaOperations kafkaOperations) {
                kafkaOperations.send(topic, message);
                throw new RuntimeException("fail");
                //return true;
            }
        });
    }
}
```



2.2.2 消费者

没什么特别之处  
**配置类 TransactionalConsumerConfig.java**

```java
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.EnableKafka;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.config.KafkaListenerContainerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import java.util.HashMap;
import java.util.Map;

@Configuration
@EnableKafka
public class TransactionalConsumerConfig {
    @Value("${kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Value("${kafka.topic.transactional}")
    private String topic;

    /**
     * 单线程-单条消费
     * @return
     */
    @Bean
    public KafkaListenerContainerFactory<?> transactionalKafkaListenerContainerFactory() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        configProps.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        configProps.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        configProps.put(ConsumerConfig.GROUP_ID_CONFIG, topic);

        ConcurrentKafkaListenerContainerFactory<String, String> factory =
                new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(new DefaultKafkaConsumerFactory<>(configProps));

        return factory;
    }

}
```



**消费者 TransactionalConsumer.java**

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.support.KafkaHeaders;
import org.springframework.messaging.handler.annotation.Header;
import org.springframework.messaging.handler.annotation.Payload;
import org.springframework.stereotype.Component;

@Component
@Slf4j
public class TransactionalConsumer {

    @KafkaListener(topics = "${kafka.topic.transactional}", containerFactory = "transactionalKafkaListenerContainerFactory")
    public void receive(@Payload String message,
                        @Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition) {
        System.out.println(String.format("From partition %d : %s", partition, message) );
    }

}
```



2.2.3 测试

**运行**

```java
@Autowired
    private TransactionalProducer transactionalProducer;

    @Test
    public void transactionalProducer() {
        transactionalProducer.send("测试事务");

        try {
            Thread.sleep(1000 * 2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
```



**结果**  
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210427093722.png)

三. 使用KafkaTemplate.executeInTransaction开启事务

事例包含在上文中的TransactionalProducer.java中

```java
/**
     * 使用KafkaTemplate.executeInTransaction开启事务
     * 
     * 本地事务，不需要事务管理器
     * @param message
     * @throws InterruptedException
     */
    public void testExecuteInTransaction(String message) throws InterruptedException {
        kafkaTemplate.executeInTransaction(new KafkaOperations.OperationsCallback() {
            @Override
            public Object doInOperations(KafkaOperations kafkaOperations) {
                kafkaOperations.send(topic, message);
                throw new RuntimeException("fail");
                //return true;
            }
        });
    }
```

## 消息转发

一. 简介

kafka概念相关的介绍请看官方文档和其他博文  
[官方中文文档](http://kafka.apachecn.org/documentation.html)  
[kafka入门介绍](https://www.orchome.com/5)

系统A从Topic-1中获取到消息，进行处理后转发到Topic-2中，系统B监听Topic-2获取消息再次进行处理。

二. 消费转发

开启消费转发主要3步：

1.  消费者配置factory.setReplyTemplate(replyKafkaTemplate);
2.  监听方法加上@SendTo注解
3.  监听方法return 消息

2.1 引入依赖

主要是spring-kafka依赖

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!--优化编码-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
```

**application.properties 添加变量参数**

设置配置参数，主题，topic等

```properties
kafka.bootstrap-servers=localhost:9092

kafka.topic.basic=test_topic
kafka.topic.json=json_topic
kafka.topic.batch=batch_topic
kafka.topic.manual=manual_topic

kafka.topic.transactional=transactional_topic
kafka.topic.reply=reply_topic
kafka.topic.reply.to=reply_to_topic
kafka.topic.filter=filter_topic
kafka.topic.error=error_topic

server.port=9093
```



2.2 Kafka配置

此处我们可以在application.properties中配置，也可以使用Java Config。我使用Java Config，看得更直观。

2.2.1 生产者

没什么不同  
**配置类 ReplyProducerConfig.java**

```java
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.serialization.StringSerializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import java.util.HashMap;
import java.util.Map;

@Configuration
public class ReplyProducerConfig {
    @Value("${kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Bean
    public KafkaTemplate<String, String> replyKafkaTemplate() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        configProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        configProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);

        // -----------------------------额外配置，可选--------------------------
        //重试，0为不启用重试机制
        configProps.put(ProducerConfig.RETRIES_CONFIG, 1);
        //控制批处理大小，单位为字节
        configProps.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);
        //批量发送，延迟为1毫秒，启用该功能能有效减少生产者发送消息次数，从而提高并发量
        configProps.put(ProducerConfig.LINGER_MS_CONFIG, 1);
        //生产者可以使用的总内存字节来缓冲等待发送到服务器的记录
        configProps.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 1024000);

        return new KafkaTemplate<>(new DefaultKafkaProducerFactory<>(configProps));

    }

    /**
     * ----可选参数----
     *
     * configProps.put(ProducerConfig.ACKS_CONFIG, "1");
     * 确认模式, 默认1
     *
     * acks=0那么生产者将根本不会等待来自服务器的任何确认。
     * 记录将立即被添加到套接字缓冲区，并被认为已发送。在这种情况下，不能保证服务器已经收到了记录，
     * 并且<code>重试</code>配置不会生效(因为客户端通常不会知道任何故障)。每个记录返回的偏移量总是设置为-1。
     *
     * acks=1这将意味着领导者将记录写入其本地日志，但不会等待所有追随者的全部确认。
     * 在这种情况下，如果领导者在确认记录后立即失败，但在追随者复制之前，记录将会丢失。
     *
     * acks=all这些意味着leader将等待所有同步的副本确认记录。这保证了只要至少有一个同步副本仍然存在，
     * 记录就不会丢失。这是最有力的保证。这相当于acks=-1的设置。
     *
     *
     *
     * configProps.put(ProducerConfig.RETRIES_CONFIG, "3");
     * 设置一个大于零的值将导致客户端重新发送任何发送失败的记录，并可能出现暂时错误。
     * 请注意，此重试与客户机在收到错误后重新发送记录没有什么不同。
     * 如果不将max.in.flight.requests.per.connection 设置为1，则允许重试可能会更改记录的顺序，
     * 因为如果将两个批发送到单个分区，而第一个批失败并重试，但第二个批成功，则第二批中的记录可能会首先出现。
     * 注意：另外，如果delivery.timeout.ms 配置的超时在成功确认之前先过期，则在重试次数用完之前，生成请求将失败。
     *
     *
     * 其他参数：参考：http://www.shixinke.com/java/kafka-configuration
     * https://blog.csdn.net/xiaozhu_you/article/details/91493258
     */

}
```



**生产者 ReplyProducer.java**

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Component;

@Component
public class ReplyProducer {
    @Autowired
    @Qualifier("replyKafkaTemplate")
    private KafkaTemplate<String, String> kafkaTemplate;

    @Value("${kafka.topic.reply}")
    private String topic;

    public void send(String message) {
        kafkaTemplate.send(topic, message);
    }
}
```



2.2.2 消费者

**配置类 ReplyConsumerConfig.java**

```java
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.EnableKafka;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.config.KafkaListenerContainerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import java.util.HashMap;
import java.util.Map;

@Configuration
@EnableKafka
public class ReplyConsumerConfig {
    @Value("${kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Value("${kafka.topic.transactional}")
    private String topic;

    @Autowired
    @Qualifier("replyKafkaTemplate")
    private KafkaTemplate replyKafkaTemplate;

    /**
     * 单线程-单条消费
     * @return
     */
    @Bean
    public KafkaListenerContainerFactory<?> replyKafkaListenerContainerFactory() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        configProps.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        configProps.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        configProps.put(ConsumerConfig.GROUP_ID_CONFIG, topic);

        ConcurrentKafkaListenerContainerFactory<String, String> factory =
                new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(new DefaultKafkaConsumerFactory<>(configProps));
        factory.setReplyTemplate(replyKafkaTemplate);
        return factory;
    }

}
```



**消费者 ReplyConsumer.java**

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.stereotype.Component;

/**
 * 转发消息
 */
@Component
@Slf4j
public class ReplyConsumer {

    @KafkaListener(topics = "${kafka.topic.reply}", containerFactory = "replyKafkaListenerContainerFactory")
    @SendTo("reply_to_topic")
    public String receiveString(String message) {
        System.out.println(String.format("Message : %s", message));
        return "reply : " + message;
    }

    @KafkaListener(topics = "${kafka.topic.reply.to}", containerFactory = "replyKafkaListenerContainerFactory")
    public void receiveStringTo(String message) {
        System.out.println(String.format("Message to : %s", message));
    }

}
```



2.2.3 测试

**运行**

```java
@Autowired
    private ReplyProducer replyProducer;

	@Test
    public void replyProducer () {
        replyProducer.send("测试转发消息");

        try {
            Thread.sleep(1000 * 2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
```



**结果**  
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210427093953.png)

## 消息过滤

一. 简介

kafka概念相关的介绍请看官方文档和其他博文  
[官方中文文档](http://kafka.apachecn.org/documentation.html)  
[kafka入门介绍](https://www.orchome.com/5)

消息过滤器可以在消息抵达监听容器前被拦截，筛选出数据再交由KafkaListener处理。  
只需要为监听容器工厂配置一个RecordFilterStrategy就行。  
返回true的时候消息将会被抛弃，返回false时，消息能正常抵达监听容器。

二. 消息过滤

开启消费转发主要1步：

1.  消费者为监听容器工厂配置一个RecordFilterStrategy

2.1 引入依赖

主要是spring-kafka依赖

```prism
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!--优化编码-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
```

**application.properties 添加变量参数**

设置配置参数，主题，topic等

```prism
kafka.bootstrap-servers=localhost:9092

kafka.topic.basic=test_topic
kafka.topic.json=json_topic
kafka.topic.batch=batch_topic
kafka.topic.manual=manual_topic

kafka.topic.transactional=transactional_topic
kafka.topic.reply=reply_topic
kafka.topic.reply.to=reply_to_topic
kafka.topic.filter=filter_topic
kafka.topic.error=error_topic

server.port=9093
```



2.2 Kafka配置

此处我们可以在application.properties中配置，也可以使用Java Config。我使用Java Config，看得更直观。

2.2.1 生产者

没什么不同  
**配置类 FilterProducerConfig.java**

```prism
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.serialization.StringSerializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import java.util.HashMap;
import java.util.Map;

@Configuration
public class FilterProducerConfig {
    @Value("${kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Bean
    public KafkaTemplate<String, String> filterKafkaTemplate() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        configProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        configProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);

        // -----------------------------额外配置，可选--------------------------
        //重试，0为不启用重试机制
        configProps.put(ProducerConfig.RETRIES_CONFIG, 1);
        //控制批处理大小，单位为字节
        configProps.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);
        //批量发送，延迟为1毫秒，启用该功能能有效减少生产者发送消息次数，从而提高并发量
        configProps.put(ProducerConfig.LINGER_MS_CONFIG, 1);
        //生产者可以使用的总内存字节来缓冲等待发送到服务器的记录
        configProps.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 1024000);

        return new KafkaTemplate<>(new DefaultKafkaProducerFactory<>(configProps));

    }

    /**
     * ----可选参数----
     *
     * configProps.put(ProducerConfig.ACKS_CONFIG, "1");
     * 确认模式, 默认1
     *
     * acks=0那么生产者将根本不会等待来自服务器的任何确认。
     * 记录将立即被添加到套接字缓冲区，并被认为已发送。在这种情况下，不能保证服务器已经收到了记录，
     * 并且<code>重试</code>配置不会生效(因为客户端通常不会知道任何故障)。每个记录返回的偏移量总是设置为-1。
     *
     * acks=1这将意味着领导者将记录写入其本地日志，但不会等待所有追随者的全部确认。
     * 在这种情况下，如果领导者在确认记录后立即失败，但在追随者复制之前，记录将会丢失。
     *
     * acks=all这些意味着leader将等待所有同步的副本确认记录。这保证了只要至少有一个同步副本仍然存在，
     * 记录就不会丢失。这是最有力的保证。这相当于acks=-1的设置。
     *
     *
     *
     * configProps.put(ProducerConfig.RETRIES_CONFIG, "3");
     * 设置一个大于零的值将导致客户端重新发送任何发送失败的记录，并可能出现暂时错误。
     * 请注意，此重试与客户机在收到错误后重新发送记录没有什么不同。
     * 如果不将max.in.flight.requests.per.connection 设置为1，则允许重试可能会更改记录的顺序，
     * 因为如果将两个批发送到单个分区，而第一个批失败并重试，但第二个批成功，则第二批中的记录可能会首先出现。
     * 注意：另外，如果delivery.timeout.ms 配置的超时在成功确认之前先过期，则在重试次数用完之前，生成请求将失败。
     *
     *
     * 其他参数：参考：http://www.shixinke.com/java/kafka-configuration
     * https://blog.csdn.net/xiaozhu_you/article/details/91493258
     */

}
```

**生产者 ReplyProducer.java**

```prism
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Component;

@Component
public class FilterProducer {
    @Autowired
    @Qualifier("filterKafkaTemplate")
    private KafkaTemplate<String, String> kafkaTemplate;

    @Value("${kafka.topic.filter}")
    private String topic;

    public void send(String message) {
        kafkaTemplate.send(topic, message);
    }

}
```

2.2.2 消费者

**配置类 FilterConsumerConfig.java**

```prism
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.EnableKafka;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.config.KafkaListenerContainerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import org.springframework.kafka.listener.ContainerProperties;
import org.springframework.kafka.listener.adapter.RecordFilterStrategy;
import java.util.HashMap;
import java.util.Map;

@Configuration
@EnableKafka
public class FilterConsumerConfig {
    @Value("${kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Value("${kafka.topic.filter}")
    private String topic;

    /**
     * 单线程-单条消费
     *
     * @return
     */
    @Bean
    public KafkaListenerContainerFactory<?> filterKafkaListenerContainerFactory() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        configProps.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        configProps.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        configProps.put(ConsumerConfig.GROUP_ID_CONFIG, topic);
        // 手动提交
        configProps.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false");

        // 监听容器工厂
        ConcurrentKafkaListenerContainerFactory<String, String> factory =
                new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(new DefaultKafkaConsumerFactory<>(configProps));
        // ack模式
        factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.MANUAL);
        // 将过滤抛弃的消息自动确认
        factory.setAckDiscarded(true);
        factory.setRecordFilterStrategy(new RecordFilterStrategy() {
            @Override
            public boolean filter(ConsumerRecord consumerRecord) {
                long data = Long.parseLong((String) consumerRecord.value());
                System.out.println("filterContainerFactory filter : " + data);
                if (data % 2 == 0) {
                    return false;
                }
                // 过滤奇数
                // 返回true将会被丢弃
                return true;
            }
        });
        return factory;
    }

}
```

**消费者 FilterConsumer.java**

```prism
import lombok.extern.slf4j.Slf4j;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.support.Acknowledgment;
import org.springframework.stereotype.Component;

/**
 * 消息过滤
 */
@Component
@Slf4j
public class FilterConsumer {

    @KafkaListener(topics = "${kafka.topic.filter}", containerFactory = "filterKafkaListenerContainerFactory")
    public void receiveString(String message, Acknowledgment ack) {
        System.out.println(String.format("Message : %s", message));
        ack.acknowledge();
    }

}
```



2.2.3 测试

**运行**

```prism
@Autowired
    private FilterProducer filterProducer;

    @Test
    public void filterProducer() {
        for (int i = 0; i < 5; i++) {
            filterProducer.send(String.valueOf(i));
        }

        try {
            Thread.sleep(1000 * 2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
```



**结果**  
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210427094000.png)

## 异常处理器

一. 简介

kafka概念相关的介绍请看官方文档和其他博文  
[官方中文文档](http://kafka.apachecn.org/documentation.html)  
[kafka入门介绍](https://www.orchome.com/5)

Kafka异常处理器为了方便异常统一管理，使业务逻辑处理与异常分离开。  
KafkaListener中所抛出的异常都会经过ConsumerAwareErrorHandler异常处理器进行处理。

二. 异常处理器

开启消费转发主要1步：

1.  消费者配置ConsumerAwareErrorHandler

2.1 引入依赖

主要是spring-kafka依赖

```prism
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!--优化编码-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
```

**application.properties 添加变量参数**

设置配置参数，主题，topic等

```prism
kafka.bootstrap-servers=localhost:9092

kafka.topic.basic=test_topic
kafka.topic.json=json_topic
kafka.topic.batch=batch_topic
kafka.topic.manual=manual_topic

kafka.topic.transactional=transactional_topic
kafka.topic.reply=reply_topic
kafka.topic.reply.to=reply_to_topic
kafka.topic.filter=filter_topic
kafka.topic.error=error_topic

server.port=9093
```

2.2 Kafka配置

此处我们可以在application.properties中配置，也可以使用Java Config。我使用Java Config，看得更直观。

2.2.1 生产者

没什么不同  
**配置类 ErrorProducerConfig.java**

```prism
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.serialization.StringSerializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.core.ProducerFactory;
import java.util.HashMap;
import java.util.Map;

@Configuration
public class ErrorProducerConfig {
    @Value("${kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Bean
    public KafkaTemplate<String, String> errorTemplate() {
        KafkaTemplate template = new KafkaTemplate<String, String>(errorProducerFactory());
        return template;
    }

    /**
     * 生产者配置
     * @return
     */
    private Map<String, Object> configs() {
        Map<String, Object> props = new HashMap<>();
        // 连接地址
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        // 键的序列化方式
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        // 值的序列化方式
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        // 重试，0为不启用重试机制
        props.put(ProducerConfig.RETRIES_CONFIG, 1);
        // 控制批处理大小，单位为字节
        props.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);
        // 批量发送，延迟为1毫秒，启用该功能能有效减少生产者发送消息次数，从而提高并发量
        props.put(ProducerConfig.LINGER_MS_CONFIG, 1);
        // 生产者可以使用的总内存字节来缓冲等待发送到服务器的记录
        props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 1024000);
        return props;
    }

    @Bean
    public ProducerFactory<String, String> errorProducerFactory() {
        DefaultKafkaProducerFactory factory = new DefaultKafkaProducerFactory<>(configs());
        return factory;
    }

}
```

**生产者 ErrorProducer.java**

```prism
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Component;

@Component
public class ErrorProducer {
    @Autowired
    @Qualifier("errorTemplate")
    private KafkaTemplate<String, String> kafkaTemplate;

    @Value("${kafka.topic.error}")
    private String topic;

    public void send(String message) {
        kafkaTemplate.send(topic, message);
    }

}
```

2.2.2 消费者

**配置类 ErrorConsumerConfig.java**

```prism
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.EnableKafka;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.config.KafkaListenerContainerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import java.util.HashMap;
import java.util.Map;

@Configuration
@EnableKafka
public class ErrorConsumerConfig {
    @Value("${kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Value("${kafka.topic.error}")
    private String topic;

    /**
     * 单线程-单条消费
     * @return
     */
    @Bean
    public KafkaListenerContainerFactory<?> errorKafkaListenerContainerFactory() {
        Map<String, Object> configProps = new HashMap<>();
        configProps.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        configProps.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        configProps.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        configProps.put(ConsumerConfig.GROUP_ID_CONFIG, topic);

        ConcurrentKafkaListenerContainerFactory<String, String> factory =
                new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(new DefaultKafkaConsumerFactory<>(configProps));

        return factory;
    }

}
```

**消费者 ErrorConsumer.java**

```prism
import lombok.extern.slf4j.Slf4j;
import org.apache.kafka.clients.consumer.Consumer;
import org.springframework.context.annotation.Bean;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.listener.ConsumerAwareListenerErrorHandler;
import org.springframework.kafka.listener.ListenerExecutionFailedException;
import org.springframework.kafka.support.KafkaHeaders;
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageHeaders;
import org.springframework.messaging.handler.annotation.Header;
import org.springframework.messaging.handler.annotation.Payload;
import org.springframework.stereotype.Component;
import java.util.List;

@Component
@Slf4j
public class ErrorConsumer {

    @KafkaListener(topics = "${kafka.topic.error}",
            containerFactory = "errorKafkaListenerContainerFactory",
            errorHandler = "consumerAwareErrorHandler")
    public void receive(@Payload String message,
                        @Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition) {
        System.out.println(String.format("From partition %d : %s", partition, message) );
        throw new RuntimeException("fail");
    }

    /**
     * 单条消息
     * @return
     */
    @Bean
    public ConsumerAwareListenerErrorHandler consumerAwareErrorHandler() {
        return new ConsumerAwareListenerErrorHandler() {

            @Override
            public Object handleError(Message<?> message, ListenerExecutionFailedException e, Consumer<?, ?> consumer) {
                log.info("ConsumerAwareListenerErrorHandler receive : "+message.getPayload().toString());
                return null;
            }
        };
    }

    /**
     * 批量消息
     * @return
     */
    @Bean
    public ConsumerAwareListenerErrorHandler consumerAwareErrorHandlerBatch() {
        return new ConsumerAwareListenerErrorHandler() {

            @Override
            public Object handleError(Message<?> message, ListenerExecutionFailedException e, Consumer<?, ?> consumer) {
                log.info("consumerAwareErrorHandler receive : "+message.getPayload().toString());
                MessageHeaders headers = message.getHeaders();
                List<String> topics = headers.get(KafkaHeaders.RECEIVED_TOPIC, List.class);
                List<Integer> partitions = headers.get(KafkaHeaders.RECEIVED_PARTITION_ID, List.class);
                List<Long> offsets = headers.get(KafkaHeaders.OFFSET, List.class);

                return null;
            }
        };
    }
}
```

2.2.3 测试

**运行**

```prism
@Autowired
    private ErrorProducer errorProducer;

	@Test
    public void errorProducer () {
        errorProducer.send("测试错误消息");

        try {
            Thread.sleep(1000 * 2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
```

**结果**  
![在这里插入图片描述](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210427095341.png)

## 集群管理工具AdminClient

一. 简介

kafka概念相关的介绍请看官方文档和其他博文  
[官方中文文档](http://kafka.apachecn.org/documentation.html)  
[kafka入门介绍](https://www.orchome.com/5)

Kafka0.11.0.0版本之后，使用AdminClient作为集群管理工具，这个是在kafka-client包下的，这是一个抽象类，具体的实现是org.apache.kafka.clients.admin.KafkaAdminClient。

KafkaAdminClient包含了功能：

1.  创建Topic：createTopics(Collection newTopics)
2.  删除Topic：deleteTopics(Collection topics)
3.  罗列所有Topic：listTopics()
4.  查询Topic：describeTopics(Collection topicNames)
5.  查询集群信息：describeCluster()
6.  查询ACL信息：describeAcls(AclBindingFilter filter)
7.  创建ACL信息：createAcls(Collection acls)
8.  删除ACL信息：deleteAcls(Collection filters)
9.  查询配置信息：describeConfigs(Collection resources)
10.  修改配置信息：alterConfigs(Map<ConfigResource, Config> configs)
11.  修改副本的日志目录：alterReplicaLogDirs(Map<TopicPartitionReplica, String> replicaAssignment)
12.  查询节点的日志目录信息：describeLogDirs(Collection brokers)
13.  查询副本的日志目录信息：describeReplicaLogDirs(Collection replicas)
14.  增加分区：createPartitions(Map<String, NewPartitions> newPartitions)

二. 集群管理工具AdminClient

2.1 引入依赖

主要是spring-kafka依赖

```prism
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!--优化编码-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
```

**application.properties 添加变量参数**

```prism
kafka.bootstrap-servers=localhost:9092
server.port=9093
```

2.2 AdminClient工具类

**KafkaAdminUtil.java**

```java
import org.apache.kafka.clients.admin.*;
import org.apache.kafka.common.KafkaFuture;
import org.apache.kafka.common.Node;
import org.apache.kafka.common.config.ConfigResource;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import java.util.*;
import java.util.concurrent.ExecutionException;

/**
 * @author 司马缸砸缸了
 * @date 2019/12/31 15:08
 * @description
 */

/**
 * 主要功能包括：
 *
 * 创建Topic：createTopics(Collection<NewTopic> newTopics)
 * 删除Topic：deleteTopics(Collection<String> topics)
 * 显示所有Topic：listTopics()
 * 查询Topic：describeTopics(Collection<String> topicNames)
 * 查询集群信息：describeCluster()
 * 查询ACL信息：describeAcls(AclBindingFilter filter)
 * 创建ACL信息：createAcls(Collection<AclBinding> acls)
 * 删除ACL信息：deleteAcls(Collection<AclBindingFilter> filters)
 * 查询配置信息：describeConfigs(Collection<ConfigResource> resources)
 * 修改配置信息：alterConfigs(Map<ConfigResource, Config> configs)
 * 修改副本的日志目录：alterReplicaLogDirs(Map<TopicPartitionReplica, String> replicaAssignment)
 * 查询节点的日志目录信息：describeLogDirs(Collection<Integer> brokers)
 * 查询副本的日志目录信息：describeReplicaLogDirs(Collection<TopicPartitionReplica> replicas)
 * 增加分区：createPartitions(Map<String, NewPartitions> newPartitions)
 */
@Component
public class KafkaAdminUtil {

    @Autowired
    private AdminClient adminClient;

    /**
     * 创建主题
     */
    public void testCreateTopic() throws InterruptedException {
        NewTopic topic = new NewTopic("topic.quick.simagangl", 3, (short) 1);
        adminClient.createTopics(Arrays.asList(topic));
        Thread.sleep(1000);
    }

    /**
     * 查看主题
     */
    public void testSelectTopicInfo() throws ExecutionException, InterruptedException {
        DescribeTopicsResult result = adminClient.describeTopics(Arrays.asList("topic.quick.simagangl"));
        result.all().get().forEach((k, v) -> System.out.println("k: " + k + " ,v: " + v.toString() + "\n"));
    }

    /**
     * 删除主题
     */
    public void deleteTopic() throws InterruptedException {
        adminClient.deleteTopics(Arrays.asList("topic.quick.simagangl"));
        Thread.sleep(1000);
    }

    /**
     * topic配置描述
     */
    public void describeConfig() throws ExecutionException, InterruptedException {
        DescribeConfigsResult ret = adminClient.describeConfigs(Collections.singleton(new ConfigResource(ConfigResource.Type.TOPIC, "batch_topic")));
        Map<ConfigResource, Config> configs = ret.all().get();
        for (Map.Entry<ConfigResource, Config> entry : configs.entrySet()) {
            ConfigResource key = entry.getKey();
            Config value = entry.getValue();
            System.out.println(String.format("Resource type: %s, resource name: %s", key.type(), key.name()));
            Collection<ConfigEntry> configEntries = value.entries();
            for (ConfigEntry each : configEntries) {
                System.out.println(each.name() + " = " + each.value());
            }
        }
    }

    /**
     * 集群描述
     *
     * @throws ExecutionException
     * @throws InterruptedException
     */
    public void describeCluster() throws ExecutionException, InterruptedException {
        DescribeClusterResult ret = adminClient.describeCluster();
        System.out.println(String.format("Cluster id: %s, controller: %s", ret.clusterId().get(), ret.controller().get()));
        System.out.println("Current cluster nodes info: ");
        for (Node node : ret.nodes().get()) {
            System.out.println(node);
        }

        Thread.sleep(1000);
    }

    /**
     * 更新topic配置
     */
    public void alterConfigs() throws ExecutionException, InterruptedException {
        Config topicConfig = new Config(Arrays.asList(new ConfigEntry("cleanup.policy", "compact")));
        adminClient.alterConfigs(Collections.singletonMap(
                new ConfigResource(ConfigResource.Type.TOPIC, "batch_topic"), topicConfig)).all().get();
    }

    /**
     * 删除指定主题
     */
    public void deleteTopics() throws ExecutionException, InterruptedException {
        KafkaFuture<Void> futures = adminClient.deleteTopics(Arrays.asList("batch_topic")).all();
        futures.get();
    }

    /**
     * 描述给定的主题
     */
    public void describeTopics() throws ExecutionException, InterruptedException {
        DescribeTopicsResult ret = adminClient.describeTopics(Arrays.asList("batch_topic", "__consumer_offsets"));
        Map<String, TopicDescription> topics = ret.all().get();
        for (Map.Entry<String, TopicDescription> entry : topics.entrySet()) {
            System.out.println(entry.getKey() + " ===> " + entry.getValue());
        }
    }

    /**
     * 打印集群中的所有主题
     */
    public void listAllTopics() throws ExecutionException, InterruptedException {
        ListTopicsOptions options = new ListTopicsOptions();
        // 包括内部主题，如_consumer_offsets
        options.listInternal(true);
        ListTopicsResult topics = adminClient.listTopics(options);
        Set<String> topicNames = topics.names().get();
        System.out.println("Current topics in this cluster: " + topicNames);
    }
}
```

其他用法请参看API

# Java Kafka Api

## 简单生产者

一. 简介

kafka概念相关的介绍请看官方文档和其他博文  
[官方中文文档](http://kafka.apachecn.org/documentation.html)  
[kafka入门介绍](https://www.orchome.com/5)

Apache Kafka引入一个新的java客户端（在org.apache.kafka.clients 包中），替代老的Scala客户端。

Kafka有4个核心API：

* Producer API 允许应用程序发送数据流到kafka集群中的topic。
* Consumer API 允许应用程序从kafka集群的topic中读取数据流。
* Streams API 允许从输入topic转换数据流到输出topic。
* Connect API 通过实现连接器（connector），不断地从一些源系统或应用程序中拉取数据到kafka，或从kafka提交数据到宿系统（sink system）或应用程序。

二. 实现

2.1 引入依赖

主要是spring-kafka依赖

```prism
<dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit-dep</artifactId>
            <version>4.9</version>
            <scope>test</scope>
        </dependency>

        <!-- kafka start -->
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka_2.11</artifactId>
            <version>0.10.1.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>0.10.1.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-streams</artifactId>
            <version>0.10.1.1</version>
        </dependency>
        <!-- kafka end -->

        <dependency>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-assembly-plugin</artifactId>
            <version>2.2-beta-5</version>
        </dependency>
        <dependency>
            <groupId>commons-lang</groupId>
            <artifactId>commons-lang</artifactId>
            <version>2.3</version>
        </dependency>
        <dependency>
            <groupId>commons-collections</groupId>
            <artifactId>commons-collections</artifactId>
            <version>3.2.1</version>
        </dependency>
    </dependencies>
```

2.2 简单生产者

send调用是异步的，它返回一个Future。如果future调用get()，则将阻塞，直到相关请求完成并返回该消息的metadata，或抛出发送异常。

**BasicProducer.java**

```prism
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;
import java.util.Properties;

/**
 * 基本生产者
 */
public class BasicProducer {

    public static void main(String[] args) {
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        // key序列化
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        // value序列化
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);

        // ------ 可选参数 -------
        // 确认模式
        props.put(ProducerConfig.ACKS_CONFIG, "1");
        // 重试次数，0为不启用重试机制
        props.put(ProducerConfig.RETRIES_CONFIG, 0);
        // 控制批处理大小，单位为字节
        props.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);
        // 批量发送，延迟为1毫秒，启用该功能能有效减少生产者发送消息次数，从而提高并发量
        props.put(ProducerConfig.LINGER_MS_CONFIG, 1);
        // 生产者可以使用的总内存字节来缓冲等待发送到服务器的记录
        props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 33554432);

        Producer<String, String> producer = new KafkaProducer<>(props);
        for (int i = 0; i < 10; i++) {
            producer.send(new ProducerRecord<String, String>("my-topic", Integer.toString(i), Integer.toString(i)));
        }
        producer.close();
    }
}

/**
 * ----可选参数----
 *
 * configProps.put(ProducerConfig.ACKS_CONFIG, "1");
 * 确认模式, 默认1
 *
 * acks=0那么生产者将根本不会等待来自服务器的任何确认。
 * 记录将立即被添加到套接字缓冲区，并被认为已发送。在这种情况下，不能保证服务器已经收到了记录，
 * 并且<code>重试</code>配置不会生效(因为客户端通常不会知道任何故障)。每个记录返回的偏移量总是设置为-1。
 *
 * acks=1这将意味着领导者将记录写入其本地日志，但不会等待所有追随者的全部确认。
 * 在这种情况下，如果领导者在确认记录后立即失败，但在追随者复制之前，记录将会丢失。
 *
 * acks=all这些意味着leader将等待所有同步的副本确认记录。这保证了只要至少有一个同步副本仍然存在，
 * 记录就不会丢失。这是最有力的保证。这相当于acks=-1的设置。
 *
 *
 *
 * configProps.put(ProducerConfig.RETRIES_CONFIG, "3");
 * 设置一个大于零的值将导致客户端重新发送任何发送失败的记录，并可能出现暂时错误。
 * 请注意，此重试与客户机在收到错误后重新发送记录没有什么不同。
 * 如果不将max.in.flight.requests.per.connection 设置为1，则允许重试可能会更改记录的顺序，
 * 因为如果将两个批发送到单个分区，而第一个批失败并重试，但第二个批成功，则第二批中的记录可能会首先出现。
 * 注意：另外，如果delivery.timeout.ms 配置的超时在成功确认之前先过期，则在重试次数用完之前，生成请求将失败。
 *
 *
 * 其他参数：参考：http://www.shixinke.com/java/kafka-configuration
 * https://blog.csdn.net/xiaozhu_you/article/details/91493258
 */
```

详细参数请参考：  
http://www.shixinke.com/java/kafka-configuration  
https://blog.csdn.net/xiaozhu_you/article/details/91493258

2.3 生产者带回调

send添加Callback参数

**BasicCallbackProducer.java**

```prism
import org.apache.kafka.clients.producer.*;
import org.apache.kafka.common.serialization.StringSerializer;
import java.util.Properties;

/**
 * 生产者--带回调
 */
public class BasicCallbackProducer {

    public static void main(String[] args) {
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        // key序列化
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        // value序列化
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);

        // ------ 可选配置 -------
        // 确认模式
        props.put(ProducerConfig.ACKS_CONFIG, "1");
        // 重试次数，0为不启用重试机制
        props.put(ProducerConfig.RETRIES_CONFIG, 0);
        // 控制批处理大小，单位为字节
        props.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);
        // 批量发送，延迟为1毫秒，启用该功能能有效减少生产者发送消息次数，从而提高并发量
        props.put(ProducerConfig.LINGER_MS_CONFIG, 1);
        // 生产者可以使用的总内存字节来缓冲等待发送到服务器的记录
        props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 33554432);

        Producer<String, String> producer = new KafkaProducer<>(props);

        // ProducerRecord消息封装类
        ProducerRecord<String, String> record;
        for (int i = 0; i < 10; i++) {
            record = new ProducerRecord<String, String>("my-topic", null,
                    System.currentTimeMillis(), Integer.toString(i), Integer.toString(i));

            // 发送消息时指定一个callback, 并覆盖onCompletion()方法，在成功发送后获取消息的偏移量及分区
            producer.send(record, new Callback() {
                @Override
                public void onCompletion(RecordMetadata metaData, Exception exception) {
                    // 记录异常信息
                    if (null != exception) {
                        System.out.println("Send message exception." + exception);
                    }
                    if (null != metaData) {
                        System.out.println(String.format("offset:%s,partition:%s",
                                metaData.offset(), metaData.partition()));
                    }
                }
            });
        }

        producer.close();
    }
}
```

## 多线程生产者

一. 简介

kafka概念相关的介绍请看官方文档和其他博文  
[官方中文文档](http://kafka.apachecn.org/documentation.html)  
[kafka入门介绍](https://www.orchome.com/5)

为了提高Kafka吞吐量，在数据量比较大切对消息顺序没有要求的情况下，可以使用多线程生产者。  
实现方式：实例化一个KafkaProducer对象在多线程中共享。KafkaProducer是线程安全的。

二. 实现

2.1 引入依赖

主要是spring-kafka依赖

```prism
<dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit-dep</artifactId>
            <version>4.9</version>
            <scope>test</scope>
        </dependency>

        <!-- kafka start -->
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka_2.11</artifactId>
            <version>0.10.1.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>0.10.1.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-streams</artifactId>
            <version>0.10.1.1</version>
        </dependency>
        <!-- kafka end -->

        <dependency>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-assembly-plugin</artifactId>
            <version>2.2-beta-5</version>
        </dependency>
        <dependency>
            <groupId>commons-lang</groupId>
            <artifactId>commons-lang</artifactId>
            <version>2.3</version>
        </dependency>
        <dependency>
            <groupId>commons-collections</groupId>
            <artifactId>commons-collections</artifactId>
            <version>3.2.1</version>
        </dependency>
    </dependencies>
```

2.2 多线程生产者

**生产者线程 KafkaProducerThread.java**

```prism
import org.apache.kafka.clients.producer.Callback;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;

/**
 * 生产者线程
 */
public class KafkaProducerThread implements Runnable {

    private KafkaProducer<String, String> producer = null;

    private ProducerRecord<String, String> record = null;

    public KafkaProducerThread(KafkaProducer<String, String> producer, ProducerRecord<String, String> record) {
        this.producer = producer;
        this.record = record;
    }

    @Override
    public void run() {
        producer.send(record, new Callback() {

            @Override
            public void onCompletion(RecordMetadata metaData,
                                     Exception exception) {
                if (null != exception) {// 发送异常记录异常信息
                    System.out.println("Send message exception:" + exception);
                }
                if (null != metaData) {
                    System.out.println(String.format("offset:%s,partition:%s",
                            metaData.offset(), metaData.partition()));
                }
            }

        });
    }

}
```


**生产者 MultiThreadProducer.java**

```prism
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;
import java.util.Properties;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * 生产者--多线程
 */
public class MultiThreadProducer {

    public static void main(String[] args) {
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        // 生产者
        KafkaProducer<String, String> producer = new KafkaProducer<String, String>(props);

        // 线程池
        ExecutorService executor = Executors.newFixedThreadPool(5);
        ProducerRecord<String, String> record;

        try {
            for (int i = 0; i < 10; i++) {
                record = new ProducerRecord<String, String>("my-topic", Integer.toString(i), Integer.toString(i));
                executor.submit(new KafkaProducerThread(producer, record));
            }

            Thread.sleep(3000);
        } catch (Exception e) {
            System.out.println("Send message exception" + e);
        } finally {
            producer.close();
            executor.shutdown();
        }
    }
}
```

详细参数请参考：  
http://www.shixinke.com/java/kafka-configuration  
https://blog.csdn.net/xiaozhu_you/article/details/91493258

## 简单消费者

一. 简介

kafka概念相关的介绍请看官方文档和其他博文  
[官方中文文档](http://kafka.apachecn.org/documentation.html)  
[kafka入门介绍](https://www.orchome.com/5)

Apache Kafka引入一个新的java客户端（在org.apache.kafka.clients 包中），替代老的Scala客户端。

Kafka有4个核心API：

* Producer API 允许应用程序发送数据流到kafka集群中的topic。
* Consumer API 允许应用程序从kafka集群的topic中读取数据流。
* Streams API 允许从输入topic转换数据流到输出topic。
* Connect API 通过实现连接器（connector），不断地从一些源系统或应用程序中拉取数据到kafka，或从kafka提交数据到宿系统（sink system）或应用程序。

二. 实现

2.1 引入依赖

主要是spring-kafka依赖

```prism
<dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit-dep</artifactId>
            <version>4.9</version>
            <scope>test</scope>
        </dependency>

        <!-- kafka start -->
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka_2.11</artifactId>
            <version>0.10.1.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>0.10.1.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-streams</artifactId>
            <version>0.10.1.1</version>
        </dependency>
        <!-- kafka end -->

        <dependency>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-assembly-plugin</artifactId>
            <version>2.2-beta-5</version>
        </dependency>
        <dependency>
            <groupId>commons-lang</groupId>
            <artifactId>commons-lang</artifactId>
            <version>2.3</version>
        </dependency>
        <dependency>
            <groupId>commons-collections</groupId>
            <artifactId>commons-collections</artifactId>
            <version>3.2.1</version>
        </dependency>
    </dependencies>
```

2.2 简单消费者

事例包含简单订阅，指定分区订阅，自动，手动提交偏移量。

**ConsumerTest.java**

```prism
import org.apache.kafka.clients.consumer.*;
import org.apache.kafka.common.TopicPartition;
import org.apache.kafka.common.serialization.StringDeserializer;
import java.util.*;

/**
 * Kafka消费者相关Api
 */
public class ConsumerTest {

    /**
     * 简单消费者--订阅主题
     */
    public static void subscribe() {
        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        // 消费者组
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "test");
        // key序列化方式
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        // value序列化方式
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);

        // ------ 可选参数 -------
        props.put(ConsumerConfig.CLIENT_ID_CONFIG, "client");

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        // 订阅主题
        consumer.subscribe(Arrays.asList("my-topic"));
        try {
            while (true) {
                // 拉取消息
                ConsumerRecords<String, String> records = consumer.poll(100);
                for (ConsumerRecord<String, String> record : records) {
                    System.out.printf(
                            "partition = %d, offset = %d,key= %s value = %s%n",
                            record.partition(), record.offset(), record.key(), record.value());
                }
            }
        } catch (Exception e) {
            // TODO 异常处理
            e.printStackTrace();
        } finally {
            consumer.close();
        }

    }

    /**
     * 简单消费者--订阅主题带再均衡处理器
     */
    public static void subscribeWithRebalence() {
        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        // 消费者组
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "test");
        // key序列化方式
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        // value序列化方式
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);

        // ------ 可选参数 -------
        props.put(ConsumerConfig.CLIENT_ID_CONFIG, "client");

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        // 1.主题 2.消费者发生平衡操作时回调进行相应的业务处理
        consumer.subscribe(Arrays.asList("my-topic"),
                new ConsumerRebalanceListener() {
                    // 在均衡开始之前和消费者停止读取消息之后调用
                    @Override
                    public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
                        // 提交位移
                        consumer.commitSync();
                    }

                    // 在重新分配分区之后和消费者开始读取消息之前调用
                    @Override
                    public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
                        long committedOffset = -1;
                        for (TopicPartition topicPartition : partitions) {
                            // 获取该分区已消费的位移
                            committedOffset = consumer.committed(topicPartition).offset();
                            // 重置位移到上一次提交的位移处开始消费
                            consumer.seek(topicPartition, committedOffset + 1);
                        }
                    }
                });

        // 订阅指定的分区
        // consumer.assign(Arrays.asList(new TopicPartition("my-topic", 0), new TopicPartition("my-topic", 2)));
        try {
            while (true) {
                // 拉取消息
                ConsumerRecords<String, String> records = consumer.poll(100);
                for (ConsumerRecord<String, String> record : records) {
                    System.out.printf(
                            "partition = %d, offset = %d,key= %s value = %s%n",
                            record.partition(), record.offset(), record.key(), record.value());
                }
            }
        } catch (Exception e) {
            // TODO 异常处理
            e.printStackTrace();
        } finally {
            consumer.close();
        }

    }

    /**
     * 自动提交位移
     */
    public static void autoCommit() {

        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        // 消费者组
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "test");
        // key序列化方式
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        // value序列化方式
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        // 自动提交偏移量
        // 如果设置成true,偏移量由auto.commit.interval.ms控制自动提交的频率
        // 如果设置成false,不需要定时的提交offset，可以自己控制offset，当消息认为已消费过了，这个时候再去提交它们的偏移量。
        // 这个很有用的，当消费的消息结合了一些处理逻辑，这个消息就不应该认为是已经消费的，直到它完成了整个处理。
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");
        // 自动提交的频率
        props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "1000");

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);

        // 订阅主题
        consumer.subscribe(Arrays.asList("my-topic"));

        try {
            while (true) {
                // 拉取消息
                ConsumerRecords<String, String> records = consumer.poll(1000);
                for (ConsumerRecord<String, String> record : records)
                    System.out.printf(
                            "partition = %d, offset = %d,key= %s value = %s%n",
                            record.partition(), record.offset(), record.key(),
                            record.value());
            }
        } catch (Exception e) {
            // TODO 异常处理
            e.printStackTrace();
        } finally {
            consumer.close();
        }

    }

    /**
     * 手动提交位移
     */
    public static void manualCommit() {

        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        // 消费者组
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "test");
        // key序列化方式
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        // value序列化方式
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        // 自动提交偏移量
        // 如果设置成true,偏移量由auto.commit.interval.ms控制自动提交的频率
        // 如果设置成false,不需要定时的提交offset，可以自己控制offset，当消息认为已消费过了，这个时候再去提交它们的偏移量。
        // 这个很有用的，当消费的消息结合了一些处理逻辑，这个消息就不应该认为是已经消费的，直到它完成了整个处理。
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false");
        // 设置一次fetch请求取得的records最大大小为1K,默认是5M
        props.put(ConsumerConfig.FETCH_MAX_BYTES_CONFIG, 1024);

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);

        // 订阅主题
        consumer.subscribe(Arrays.asList("my-topic"));

        try {
            // 最少处理10条消息后才进行提交
            int minCommitSize = 10;
            // 消息计算器
            int icount = 0;
            while (true) {
                // 等待拉取消息
                ConsumerRecords<String, String> records = consumer.poll(1000);
                for (ConsumerRecord<String, String> record : records) {
                    // 简单打印出消息内容,模拟业务处理
                    System.out.printf(
                            "partition = %d, offset = %d,key= %s value = %s%n",
                            record.partition(), record.offset(), record.key(),
                            record.value());
                    icount++;
                }
                // 在业务逻辑没有处理成功后提交位移
                if (icount >= minCommitSize) {
                    // 同步提交
                    // consumer.commitSync();

                    // 异步提交，可以设置回调OffsetCommitCallback
                    consumer.commitAsync(new OffsetCommitCallback() {

                        @Override
                        public void onComplete(
                                Map<TopicPartition, OffsetAndMetadata> offsets,
                                Exception exception) {
                            if (null == exception) {
                                // TODO 表示位移成功提交
                                System.out.println("提交成功");
                            } else {
                                // TODO 表示提交位移发生了异常，根据业务进行相关处理
                                System.out.println("发生异常");
                            }
                        }
                    });
                    // 重设置计数器
                    icount = 0;
                }
            }
        } catch (Exception e) {
            // TODO 异常处理
            e.printStackTrace();
        } finally {
            consumer.close();
        }
    }

    /**
     * 订阅指定分区
     */
    public static void position() {
        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        // 消费者组
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "test");
        // key序列化方式
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        // value序列化方式
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        // 自动提交偏移量
        // 如果设置成true,偏移量由auto.commit.interval.ms控制自动提交的频率
        // 如果设置成false,不需要定时的提交offset，可以自己控制offset，当消息认为已消费过了，这个时候再去提交它们的偏移量。
        // 这个很有用的，当消费的消息结合了一些处理逻辑，这个消息就不应该认为是已经消费的，直到它完成了整个处理。
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");
        // 自动提交的频率
        props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "1000");
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);

        // 订阅主题
        consumer.assign(Arrays.asList(new TopicPartition("my-topic", 2)));
        // 构造待查询的分区
        TopicPartition partition = new TopicPartition("my-topic", 2);
        try {
            // committed返回OffsetAndMetadata队象，通过它可以获取指定分区的偏移量
            long commitOffset = consumer.committed(partition).offset();
            // position获取下一次拉取得位置
            long position = consumer.position(partition);
            System.out.println(commitOffset);
            System.out.println(position);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            consumer.close();
        }
    }

    public static void main(String[] args) {
        subscribe();
        // manualCommit();
        //autoCommit();
        //position();
    }
}
```


详细参数请参考：  
http://www.shixinke.com/java/kafka-configuration  
https://blog.csdn.net/xiaozhu_you/article/details/91493258

## 指定开始时间消费

一. 简介

kafka概念相关的介绍请看官方文档和其他博文  
[官方中文文档](http://kafka.apachecn.org/documentation.html)  
[kafka入门介绍](https://www.orchome.com/5)

Kafka消费者API提供了offsetsForTimes(Map<TopicPartition, Long> timestampsToSearch)方法，入参为Map，Key是待查询分区，Value是待查询的时间戳，返回值是 大于等于时间戳的第一条消息的偏移量和时间戳。如果分区不存在，该方法会一直阻塞。  
如果我们希望从某个时间开始消费，就可以使用它找到偏移量，之后调用seek(TopicPartition partition, long offset)将消费偏移量重置到指定的查询得到的偏移量。

二. 实现

2.1 引入依赖

主要是spring-kafka依赖

```prism
<dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit-dep</artifactId>
            <version>4.9</version>
            <scope>test</scope>
        </dependency>

        <!-- kafka start -->
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka_2.11</artifactId>
            <version>0.10.1.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>0.10.1.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-streams</artifactId>
            <version>0.10.1.1</version>
        </dependency>
        <!-- kafka end -->

        <dependency>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-assembly-plugin</artifactId>
            <version>2.2-beta-5</version>
        </dependency>
        <dependency>
            <groupId>commons-lang</groupId>
            <artifactId>commons-lang</artifactId>
            <version>2.3</version>
        </dependency>
        <dependency>
            <groupId>commons-collections</groupId>
            <artifactId>commons-collections</artifactId>
            <version>3.2.1</version>
        </dependency>
    </dependencies>
```

2.2 消费者–指定时间

**ConsumerForTime.java**

```prism
import org.apache.kafka.clients.consumer.*;
import org.apache.kafka.common.TopicPartition;
import org.apache.kafka.common.serialization.StringDeserializer;
import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;
import java.util.Properties;

/**
 * @author 司马缸砸缸了
 * @date 2020/1/9 17:09
 * @description 根据时间消费
 */

public class ConsumerForTime {

    public static void main(String[] args) {

        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        // 消费者组
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "test");
        // key序列化方式
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        // value序列化方式
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        // 自动提交偏移量
        // 如果设置成true,偏移量由auto.commit.interval.ms控制自动提交的频率
        // 如果设置成false,不需要定时的提交offset，可以自己控制offset，当消息认为已消费过了，这个时候再去提交它们的偏移量。
        // 这个很有用的，当消费的消息结合了一些处理逻辑，这个消息就不应该认为是已经消费的，直到它完成了整个处理。
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");
        // 自动提交的频率
        props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "1000");

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);

        // 订阅主题
        consumer.assign(Arrays.asList(new TopicPartition("my-topic", 0)));
        try {
            Map<TopicPartition, Long> timestampsToSearch = new HashMap<TopicPartition, Long>();
            // 构造查询的分区
            TopicPartition partition = new TopicPartition("my-topic", 0);
            // 设置查询12个小时之前消息的偏移量
            timestampsToSearch.put(partition, (System.currentTimeMillis() - 12 * 3600 * 1000));
            // 会返回时间大于等于查找时间的第一个偏移量
            Map<TopicPartition, OffsetAndTimestamp> offsetMap = consumer.offsetsForTimes(timestampsToSearch);
            OffsetAndTimestamp offsetTimestamp = null;
            // 遍历查询的分区，我们只查询了一个分区
            for (Map.Entry<TopicPartition, OffsetAndTimestamp> entry : offsetMap.entrySet()) {
                // 若查询时间大于时间戳索引文件中最大记录的索引时间,此时value为空,即待查询时间点之后没有新消息生成
                offsetTimestamp = entry.getValue();
                if (null != offsetTimestamp) {
                    System.out.printf("partition = %d, offset = %d,timestamp= %d%n",
                            entry.getKey().partition(), entry.getValue()
                                    .offset(), entry.getValue().timestamp());
                    // 重置消费起始偏移量
                    consumer.seek(partition, entry.getValue().offset());
                }
            }
            while (true) {
                // 拉取消息
                ConsumerRecords<String, String> records = consumer.poll(1000);
                for (ConsumerRecord<String, String> record : records)
                    System.out.printf(
                            "partition = %d, offset = %d,key= %s value = %s%n",
                            record.partition(), record.offset(), record.key(),
                            record.value());
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            consumer.close();
        }
    }

}
```


详细参数请参考：  
http://www.shixinke.com/java/kafka-configuration  
https://blog.csdn.net/xiaozhu_you/article/details/91493258

## 多线程消费者

一. 简介

kafka概念相关的介绍请看官方文档和其他博文  
[官方中文文档](http://kafka.apachecn.org/documentation.html)  
[kafka入门介绍](https://www.orchome.com/5)

KafkaConsumer是非线程安全的，多线程要处理好线程同步，使用方式：为每个线程实例化一个KafkaConsumer对象。  
注意：属于一个消费者组的线程受分区数影响，如果消费者大于分区数，那么就会有一部分消费者线程空闲。

二. 实现

2.1 引入依赖

主要是spring-kafka依赖

```prism
<dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit-dep</artifactId>
            <version>4.9</version>
            <scope>test</scope>
        </dependency>

        <!-- kafka start -->
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka_2.11</artifactId>
            <version>0.10.1.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>0.10.1.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-streams</artifactId>
            <version>0.10.1.1</version>
        </dependency>
        <!-- kafka end -->

        <dependency>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-assembly-plugin</artifactId>
            <version>2.2-beta-5</version>
        </dependency>
        <dependency>
            <groupId>commons-lang</groupId>
            <artifactId>commons-lang</artifactId>
            <version>2.3</version>
        </dependency>
        <dependency>
            <groupId>commons-collections</groupId>
            <artifactId>commons-collections</artifactId>
            <version>3.2.1</version>
        </dependency>
    </dependencies>
```

2.2 多线程消费者

**消费者线程 ConsumerThread.java**

```prism
import java.util.Arrays;
import java.util.Map;
import java.util.Properties;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;

/**
 * 消费者线程
 */
public class ConsumerThread extends Thread {

    private KafkaConsumer<String, String> consumer;

    public ConsumerThread(Map<String, Object> consumerConfig, String topic) {
        Properties props = new Properties();
        props.putAll(consumerConfig);
        this.consumer = new KafkaConsumer<String, String>(props);
        consumer.subscribe(Arrays.asList(topic));
    }

    @Override
    public void run() {
        try {
            while (true) {
                ConsumerRecords<String, String> records = consumer.poll(1000);
                for (ConsumerRecord<String, String> record : records) {
                    System.out
                            .printf("threadId=%s,partition = %d, offset = %d,key= %s value = %s%n",
                                    Thread.currentThread().getId(),
                                    record.partition(), record.offset(),
                                    record.key(), record.value());
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            consumer.close();
        }
    }
}
```


**消费者 ConsumerExecutor.java**

```prism
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.StringDeserializer;
import java.util.HashMap;
import java.util.Map;

/**
 * 消费者--多线程
 */
public class ConsumerExecutor {

    public static void main(String[] args) {
        Map<String, Object> props = new HashMap<String, Object>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        // 消费者组
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "test");
        // key序列化方式
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        // value序列化方式
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        // 自动提交偏移量
        // 如果设置成true,偏移量由auto.commit.interval.ms控制自动提交的频率
        // 如果设置成false,不需要定时的提交offset，可以自己控制offset，当消息认为已消费过了，这个时候再去提交它们的偏移量。
        // 这个很有用的，当消费的消息结合了一些处理逻辑，这个消息就不应该认为是已经消费的，直到它完成了整个处理。
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");
        // 自动提交的频率
        props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "1000");

        for (int i = 0; i < 5; i++) {
            new ConsumerThread(props, "my-topic-thread").start();
        }
    }
}
```

## 简单Stream事例

一. 简介

kafka概念相关的介绍请看官方文档和其他博文  
[官方中文文档](http://kafka.apachecn.org/documentation.html)  
[kafka入门介绍](https://www.orchome.com/5)

Kafka Streams从一个或多个输入topic进行连续的计算并输出到0或多个外部topic中。

二. 实现

2.1 引入依赖

主要是spring-kafka依赖

```prism
<dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit-dep</artifactId>
            <version>4.9</version>
            <scope>test</scope>
        </dependency>

        <!-- kafka start -->
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka_2.11</artifactId>
            <version>0.10.1.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>0.10.1.1</version>
        </dependency>
        <!-- streams -->
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-streams</artifactId>
            <version>0.10.1.1</version>
        </dependency>
        <!-- kafka end -->

        <dependency>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-assembly-plugin</artifactId>
            <version>2.2-beta-5</version>
        </dependency>
        <dependency>
            <groupId>commons-lang</groupId>
            <artifactId>commons-lang</artifactId>
            <version>2.3</version>
        </dependency>
        <dependency>
            <groupId>commons-collections</groupId>
            <artifactId>commons-collections</artifactId>
            <version>3.2.1</version>
        </dependency>
    </dependencies>
```

2.2 简单Stream事例

**BasicStream.java**

```prism
import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.kstream.KStreamBuilder;
import java.util.HashMap;
import java.util.Map;

/**
 * Stream简单事例
 */
public class BasicStream {

    public static void main(String[] args) throws InterruptedException {
        Map<String, Object> props = new HashMap<>();
        // 流处理应用程序的标识符。在Kafka集群中必须是唯一的
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, "my-stream-application");
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        // key序列化反序列化
        props.put(StreamsConfig.KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());
        // value序列化反序列化
        props.put(StreamsConfig.VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());
        StreamsConfig config = new StreamsConfig(props);

        KStreamBuilder builder = new KStreamBuilder();
        // 从topic my-input-topic中输入，经过mapValues转换后，输出到my-output-topic
        builder.stream("my-input-topic").mapValues(value -> value.toString()).to("my-output-topic");

        Thread.sleep(3000);
        KafkaStreams streams = new KafkaStreams(builder, config);
        streams.start();
    }
}
```

2.3 IP访问检测

该事例原自《Kafka入门与实践》书中。过滤掉访问量大于等于2的IP

**自定义处理器 IpBlackListProcessor.java**

```prism
import org.apache.kafka.streams.kstream.Windowed;
import org.apache.kafka.streams.processor.Processor;
import org.apache.kafka.streams.processor.ProcessorContext;

/**
 * 自定义Processor完成黑名单处理
 */
public class IpBlackListProcessor implements Processor<Windowed<String>, Long> {

    @Override
    public void init(ProcessorContext context) {

    }

    @Override
    public void process(Windowed<String> key, Long value) {
    	// 此处制作简单记录，实际可以存到Redis中进行访问时间限制。
        System.out.println("ip:" + key.key() + "被加入到黑名单, 请求次数为:" + value);
    }

    @Override
    public void punctuate(long timestamp) {

    }

    @Override
    public void close() {

    }
}
```


**BlackListChecker.java**

```prism
import java.util.Date;
import java.util.Properties;
import org.apache.commons.lang.time.DateFormatUtils;
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.KeyValue;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.kstream.KStream;
import org.apache.kafka.streams.kstream.KStreamBuilder;
import org.apache.kafka.streams.kstream.KeyValueMapper;
import org.apache.kafka.streams.kstream.Predicate;
import org.apache.kafka.streams.kstream.TimeWindows;
import org.apache.kafka.streams.kstream.Windowed;
import org.apache.kafka.streams.processor.Processor;
import org.apache.kafka.streams.processor.ProcessorSupplier;

/**
 *stream事例：IP访问过滤
 * 过滤掉访问量大于等于2的IP
 */
public class BlackListChecker {

	/**
	 * java8实时计算黑名单
	 */
	public static void checkBlackListOfJava8(){

		Properties props = new Properties();
		props.put(StreamsConfig.APPLICATION_ID_CONFIG, "ip-blacklist-checker");
		props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
		// Key序列化与反序化类
		props.put(StreamsConfig.KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
		// Value序列化与反序化类
		props.put(StreamsConfig.VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());
		// 从最新消息开始消费
		props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
		// 指定保存当前位置的时间间隔，默认30s
		props.put(StreamsConfig.COMMIT_INTERVAL_MS_CONFIG, "1000");
		// 设置轮询kafka主题获取数据源的时间间隔，默认100
		props.put(StreamsConfig.POLL_MS_CONFIG, "10");

		KStreamBuilder builder = new KStreamBuilder();
		// 读取access-log主题
		KStream<String, String> accessLog = builder.stream("access-log");
		// 将每个消息构建成KeyValue，为了根据key分组
		accessLog.map((key,value) -> new KeyValue<>(value, value))
		// 根据key分组
		.groupByKey()
		// 指定时间窗口为1分钟, 即每次统计用户在1分钟内的请求
		// 将KGroupStream转换为KTable
		.count(TimeWindows.of(60 * 1000L).advanceBy(60*1000), "access-count")
		//转为KStream
		.toStream()
		.filter((Windowed<String> key, Long value) -> null!=value && value >=2)
		// 处理命中的记录
		.process(()-> new IpBlackListProcessor());

		// 启动
		KafkaStreams streams = new KafkaStreams(builder, props);
		streams.start();


	}

	/**
	 * java7实时计算黑名单
	 */
	public static void checkBlackListOfJava7() {
		Properties props = new Properties();
		props.put(StreamsConfig.APPLICATION_ID_CONFIG, "ip-blacklist-checker");
		props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
		// Key序列化与反序化类
		props.put(StreamsConfig.KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
		// Value序列化与反序化类
		props.put(StreamsConfig.VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());
		// 从最新消息开始消费
		props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
		// 指定保存当前位置的时间间隔，默认30s
		props.put(StreamsConfig.COMMIT_INTERVAL_MS_CONFIG, "1000");
		// 设置轮询kafka主题获取数据源的时间间隔，默认100
		props.put(StreamsConfig.POLL_MS_CONFIG, "10");

		KStreamBuilder builder = new KStreamBuilder();
		KStream<String, String> accessLog = builder.stream("access-log");
		// 将每个消息构建成KeyValue，为了根据key分组
		// 我们没有设置Key，设置key与Value相同
		accessLog.map(new KeyValueMapper<String, String, KeyValue<String, String>>() {
					@Override
					public KeyValue<String, String> apply(String key, String value) {
						return new KeyValue<String, String>(value, value);
					}
				})
				.groupByKey()
				// 指定时间窗口为1分钟, 即每次统计用户在1分钟内的请求
				// access-count为状态数据key
				.count(TimeWindows.of(60 * 1000L).advanceBy(60*1000), "access-count")
				// 转为KStream
				.toStream()
				// 过滤数据
				.filter(new Predicate<Windowed<String>, Long>() {

					@Override
					public boolean test(Windowed<String> key, Long value) {//指定规则
						System.out.println("请求时间："+DateFormatUtils.format(new Date(System.currentTimeMillis()), "HH:mm:ss")+",IP:"+key.key()+",请求次数:"+value);
						if(null!=value&&value.longValue()>=2){
							return true;
						}
						// 返回false的数据将被过滤掉
						return false;
					}
				})
				// 处理命中的记录
				.process(new ProcessorSupplier<Windowed<String>, Long>() {
					@Override
					public Processor<Windowed<String>, Long> get() {
						// 使用自定义的Process对命中的记录进行处理
						return new IpBlackListProcessor();
					}
				}, "access-count");

		// 启动
		KafkaStreams streams = new KafkaStreams(builder, props);
	    streams.start();

	}

	public static void main(String[] args) {
		checkBlackListOfJava7();
	}
}
```

## 再均衡监听器ConsumerRebalanceListener

一. 简介

kafka概念相关的介绍请看官方文档和其他博文  
[官方中文文档](http://kafka.apachecn.org/documentation.html)  
[kafka入门介绍](https://www.orchome.com/5)

ConsumerRebalanceListener再均衡监听器提供两个方法：

* onPartitionsRevoked（Collection） 在均衡开始之前和消费者停止读取消息之后调用，一般用来提交偏移量
* onPartitionsAssigned(Collection) 在重新分配分区之后和消费者开始读取消息之前调用，一般用来指定消费偏移量

二. 实现

2.1 引入依赖

主要是spring-kafka依赖

```prism
<dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit-dep</artifactId>
            <version>4.9</version>
            <scope>test</scope>
        </dependency>

        <!-- kafka start -->
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka_2.11</artifactId>
            <version>0.10.1.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>0.10.1.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-streams</artifactId>
            <version>0.10.1.1</version>
        </dependency>
        <!-- kafka end -->

        <dependency>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-assembly-plugin</artifactId>
            <version>2.2-beta-5</version>
        </dependency>
        <dependency>
            <groupId>commons-lang</groupId>
            <artifactId>commons-lang</artifactId>
            <version>2.3</version>
        </dependency>
        <dependency>
            <groupId>commons-collections</groupId>
            <artifactId>commons-collections</artifactId>
            <version>3.2.1</version>
        </dependency>
    </dependencies>
```

2.2 使用事例

**ConsumerRebalanceListenerTest.java**

```prism
import org.apache.kafka.clients.consumer.*;
import org.apache.kafka.common.TopicPartition;
import org.apache.kafka.common.serialization.StringDeserializer;
import java.util.*;

/**
 * Kafka消费者ConsumerRebalanceListener使用
 */
public class ConsumerRebalanceListenerTest {

    /**
     * 简单消费者--订阅主题带再均衡处理器
     */
    public static void subscribeWithRebalence() {
        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        // 消费者组
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "test");
        // key序列化方式
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        // value序列化方式
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);

        // ------ 可选参数 -------
        props.put(ConsumerConfig.CLIENT_ID_CONFIG, "client");

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        // 1.主题 2.消费者发生平衡操作时回调进行相应的业务处理
        consumer.subscribe(Arrays.asList("my-topic"),
                new ConsumerRebalanceListener() {
                    // 在均衡开始之前和消费者停止读取消息之后调用
                    @Override
                    public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
                        // 提交位移
                        consumer.commitSync();
                    }

                    // 在重新分配分区之后和消费者开始读取消息之前调用
                    @Override
                    public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
                        long committedOffset = -1;
                        for (TopicPartition topicPartition : partitions) {
                            // 获取该分区已消费的位移
                            committedOffset = consumer.committed(topicPartition).offset();
                            // 重置位移到上一次提交的位移处开始消费
                            consumer.seek(topicPartition, committedOffset + 1);
                        }
                    }
                });

        // 订阅指定的分区
        // consumer.assign(Arrays.asList(new TopicPartition("my-topic", 0), new TopicPartition("my-topic", 2)));
        try {
            while (true) {
                // 拉取消息
                ConsumerRecords<String, String> records = consumer.poll(100);
                for (ConsumerRecord<String, String> record : records) {
                    System.out.printf(
                            "partition = %d, offset = %d,key= %s value = %s%n",
                            record.partition(), record.offset(), record.key(), record.value());
                }
            }
        } catch (Exception e) {
            // TODO 异常处理
            e.printStackTrace();
        } finally {
            consumer.close();
        }

    }

    /**
     * 提交指定偏移量 和 再均衡处理器实现
     */
    public static void commitCurrentOffsets() {
        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        // 消费者组
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "test");
        // key序列化方式
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        // value序列化方式
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        // 自动提交偏移量
        // 如果设置成true,偏移量由auto.commit.interval.ms控制自动提交的频率
        // 如果设置成false,不需要定时的提交offset，可以自己控制offset，当消息认为已消费过了，这个时候再去提交它们的偏移量。
        // 这个很有用的，当消费的消息结合了一些处理逻辑，这个消息就不应该认为是已经消费的，直到它完成了整个处理。
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false");
        // ------ 可选参数 -------
        props.put(ConsumerConfig.CLIENT_ID_CONFIG, "client");

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);

        Map<TopicPartition, OffsetAndMetadata> currentOffsets = new HashMap<>();

        // 1.主题 2.消费者发生平衡操作时回调进行相应的业务处理
        consumer.subscribe(Arrays.asList("my-topic"),
                new ConsumerRebalanceListener() {
                    // 在均衡开始之前和消费者停止读取消息之后调用
                    @Override
                    public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
                        System.out.println("Lost partitions in rebalance. committing current offsets:" + currentOffsets);
                        // 提交位移
                        consumer.commitSync(currentOffsets);
                    }

                    // 在重新分配分区之后和消费者开始读取消息之前调用
                    @Override
                    public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
                        // Do nothing, 也可以如下指定消费偏移量

//                        long committedOffset = -1;
//                        for (TopicPartition topicPartition : partitions) {
//                            // 获取该分区已消费的位移
//                            committedOffset = consumer.committed(topicPartition).offset();
//                            // 重置位移到上一次提交的位移处开始消费
//                            consumer.seek(topicPartition, committedOffset + 1);
//                        }
                    }
                });

        // 订阅指定的分区
        try {
            while (true) {
                // 拉取消息, 参数100ms是poll timeout时间
                ConsumerRecords<String, String> records = consumer.poll(100);
                for (ConsumerRecord<String, String> record : records) {
                    System.out.printf(
                            "partition = %d, offset = %d,key= %s value = %s%n",
                            record.partition(), record.offset(), record.key(), record.value());

                    // 设置需要提交的偏移量
                    currentOffsets.put(new TopicPartition(record.topic(), record.partition()),
                            new OffsetAndMetadata(record.offset() + 1, "no metadata")
                    );
                }
                // 手动异步提交指定偏移量
                consumer.commitAsync(currentOffsets, null);
            }
        } catch (Exception e) {
            // TODO 异常处理
            e.printStackTrace();
        } finally {
            try {
                consumer.commitSync(currentOffsets);
            } finally {
                consumer.close();
                System.out.println("closed consumer...");
            }
        }
    }

    public static void main(String[] args) {
    }
}
```



# 可视化工具Kafka Tool 2

下载地址：http://www.kafkatool.com/download.html

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517162812.png)

**2、安装**

根据不同的系统下载对应的版本，我这里kafka版本是1.1.0，下载kafka tool 2.0.1。

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517162815.png)

双击下载完成的exe图标，傻瓜式完成安装。

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517162822.png)

**3、简单使用**

kafka环境搭建请参考：[CentOS7.5搭建Kafka2.11-1.1.0集群](https://www.cnblogs.com/frankdeng/p/9403883.html)

**1）连接kafka**

打开kafka tool安装目录，点击exe文件

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517162824.png)

提示设置kafka集群连接

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517162826.png)

 点击确定，设置

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517162829.png)

设置完了，点击Test测试是否能连接，连接通了，然后点击Add，添加完成设置。出现如下界面

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517162831.png)

**2）简单使用**

配置以字符串的形式显示kafka消息体

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517162833.png)

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517162835.png)

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517162837.png)

或者通过如下界面配置

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517162840.png)

注释：更改完Content Types，要点击Update和Refresh按钮

再次查看kafka的数据：

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517162842.png)

# Spring-Kafka —— 消费重试机制实现

## 消息处理问题

在从Kafka主题接收消息之后立即处理消息的消费者的实现非常简单。不幸的是，现实要复杂得多，并且由于各种原因，消息处理可能会失败。其中一些原因是永久性问题，例如数据库约束失败或消息格式无效。其他，如消息处理中涉及的依赖系统的临时不可用，可以在将来解决。在这些情况下，重试消息处理可能是一种有效的解决方案。

 

## 非阻塞重试逻辑

在像Kafka这样的流媒体系统中，我们不能跳过消息并在以后回复它们。一旦我们移动当前消息中指向Kafka的指针，我们就无法返回。为简单起见，我们假设消息偏移在成功的消息处理之后就被记住了。在这种情况下，除非我们成功处理当前消息，否则我们无法接收下一条消息。如果处理单个消息不断失败，则会阻止系统处理下一条消息。很明显，我们希望避免这种情况，因为通常一次消息处理失败并不意味着下一次消息处理失败。此外，在较长时间（例如一小时）之后，由于各种原因，失败消息的处理可能成功。对他们来说，我们所依赖的系统可以再次出现。

 

在消息处理失败时，我们可以将消息的副本发布到另一个主题并等待下一条消息。让我们将新主题称为'retry_topic'。'retry_topic'的消费者将从Kafka接收消息，然后在开始消息处理之前等待一些预定义的时间，例如一小时。通过这种方式，我们可以推迟下一次消息处理尝试，而不会对'main_topic'消费者产生任何影响。如果'retry_topic'消费者中的处理失败，我们只需放弃并将消息存储在'failed_topic'中，以便进一步手动处理此问题。

 

## 业务重试场景

现在让我们考虑以下场景。一条新消息被写入主题'main_topic'。如果此消息的处理失败，那么我们应该在5分钟内再次尝试。我们怎么做？我们应该向'retry_topic'写一条新消息，它包装失败的消息并添加2个字段：

- 'retry_number'，值为1
- 'retry_timestamp'，其值计算为现在+ 5分钟

这意味着'main_topic'使用者将失败的消息处理的责任委托给另一个组件。'main_topic'消费者未被阻止，可以接收下一条消息。'retry_topic'消费者将立即收到'main_topic'消费者发布的消息。它必须从消息中读取'retry_timestamp'值并等到那一刻，阻塞线程。线程唤醒后，它将再次尝试处理该消息。如果成功，那么我们可以获取下一个可用消息。否则我们必须再次尝试。我们要做的是克隆消息，递增'attempt_number'值（它将为2）并将'retry_timestamp'值设置为now + 5分钟。消息克隆将再次发布到'retry__topic。
如果我们到达重试最高次数。现在是时候说“停止”了。我们将消息写入'failed_topic'并将此消息视为未处理。有人必须手动处理它。
下面的图片可以帮助您理解消息流：


![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210422214226.png)

## 总结

正如您所注意到的，在发生某些故障时实施推迟消息处理并不是一件容易的事情。请记住：

- 可以仅按顺序从主题分区中使用消息
- 您不能跳过消费并稍后再处理此消息
- 如果要推迟处理某些消息，可以将它们重新发布到单独的主题，每个延迟值一个
- 处理失败的消息可以通过克隆消息并将其重新发布到重试主题之一来实现，其中包含有关尝试次数和下次重试时间戳的更新信息
- 除非是时候处理消息，否则重试主题的消费者应该阻止该线程
- 重试主题中的消息按时间顺序自然组织，必须按顺序处理

 

## 问题

# 参考

https://blog.csdn.net/yy756127197/category_7220143.html



