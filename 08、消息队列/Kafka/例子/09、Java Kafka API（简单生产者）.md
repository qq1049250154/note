## 一. 简介

kafka概念相关的介绍请看官方文档和其他博文  
[官方中文文档](http://kafka.apachecn.org/documentation.html)  
[kafka入门介绍](https://www.orchome.com/5)

Apache Kafka引入一个新的java客户端（在org.apache.kafka.clients 包中），替代老的Scala客户端。

Kafka有4个核心API：

* Producer API 允许应用程序发送数据流到kafka集群中的topic。
* Consumer API 允许应用程序从kafka集群的topic中读取数据流。
* Streams API 允许从输入topic转换数据流到输出topic。
* Connect API 通过实现连接器（connector），不断地从一些源系统或应用程序中拉取数据到kafka，或从kafka提交数据到宿系统（sink system）或应用程序。

## 二. 实现

### 2.1 引入依赖

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

### 2.2 简单生产者

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

### 2.3 生产者带回调

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

## 源码地址

[IT-CLOUD-KAFKA-CLIENT](https://gitee.com/simagang/it-cloud-kafka-client) :spring整合kafka教程源码。博文在本CSDN kafka系列中。

---

## 项目推荐

> [IT-CLOUD](https://gitee.com/simagang/it-cloud) :IT服务管理平台，集成基础服务，中间件服务，监控告警服务等。  
> [IT-CLOUD-ACTIVITI6](https://gitee.com/simagang/it-cloud-activiti6) :Activiti教程源码。博文在本CSDN Activiti系列中。  
> [IT-CLOUD-ELASTICSEARCH](https://gitee.com/simagang/it-cloud-elasticsearch) :elasticsearch教程源码。博文在本CSDN elasticsearch系列中。  
> [IT-CLOUD-KAFKA](https://gitee.com/simagang/it-cloud-kafka) :spring整合kafka教程源码。博文在本CSDN kafka系列中。  
> [IT-CLOUD-KAFKA-CLIENT](https://gitee.com/simagang/it-cloud-kafka-client) :kafka client教程源码。博文在本CSDN kafka系列中。
>
> _开源项目，持续更新中，喜欢请 Star~_