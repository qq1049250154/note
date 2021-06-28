## 一. 简介

kafka概念相关的介绍请看官方文档和其他博文  
[官方中文文档](http://kafka.apachecn.org/documentation.html)  
[kafka入门介绍](https://www.orchome.com/5)

## 二. 批量消费

开启批量消费主要3步

1.  消费者设置 max.poll.records
2.  消费者 开启批量消费 factory.setBatchListener(true);
3.  消费者批量接收 public void consumerBatch(List<ConsumerRecord<?, ?>> records)

### 2.1 引入依赖

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



### 2.2 Kafka配置

此处我们可以在application.properties中配置，也可以使用Java Config。我使用Java Config，看得更直观。

#### 2.2.1 生产者

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



#### 2.2.2 消费者

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



#### 2.2.3 测试

**运行**

```prism
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