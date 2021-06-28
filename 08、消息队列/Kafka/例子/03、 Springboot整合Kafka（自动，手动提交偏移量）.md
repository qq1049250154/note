## 一. 简介

kafka概念相关的介绍请看官方文档和其他博文  
[官方中文文档](http://kafka.apachecn.org/documentation.html)  
[kafka入门介绍](https://www.orchome.com/5)

我们可以把偏移量交给kafka去提交，也可以自己控制提交时机。例如消息处理过程缓慢的情况下，就需要自己控制偏移量何时提交比较好。

## 二. 自动提交偏移量

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



## 三. 手动提交偏移量

主要步骤：

1.  消费者配置 configProps.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, “false”);
2.  消费者配置ack模式factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.MANUAL_IMMEDIATE);
3.  消费者手动提交 consumer.commitSync();

### 3.1 引入依赖

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



### 3.2 Kafka配置

此处我们可以在application.properties中配置，也可以使用Java Config。我使用Java Config，看得更直观。

#### 3.2.1 生产者

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



#### 3.2.2 消费者

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



#### 3.2.3 测试

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

## 源码地址

[IT-CLOUD-KAFKA](https://gitee.com/simagang/it-cloud-kafka) :spring整合kafka教程源码。博文在本CSDN kafka系列中。

---

## 项目推荐

> [IT-CLOUD](https://gitee.com/simagang/it-cloud) :IT服务管理平台，集成基础服务，中间件服务，监控告警服务等。  
> [IT-CLOUD-ACTIVITI6](https://gitee.com/simagang/it-cloud-activiti6) :Activiti教程源码。博文在本CSDN Activiti系列中。  
> [IT-CLOUD-ELASTICSEARCH](https://gitee.com/simagang/it-cloud-elasticsearch) :elasticsearch教程源码。博文在本CSDN elasticsearch系列中。  
> [IT-CLOUD-KAFKA](https://gitee.com/simagang/it-cloud-kafka) :spring整合kafka教程源码。博文在本CSDN kafka系列中。  
> [IT-CLOUD-KAFKA-CLIENT](https://gitee.com/simagang/it-cloud-kafka-client) :kafka client教程源码。博文在本CSDN kafka系列中。
>
> _开源项目，持续更新中，喜欢请 Star~_