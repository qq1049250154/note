## 一. 简介

kafka概念相关的介绍请看官方文档和其他博文  
[官方中文文档](http://kafka.apachecn.org/documentation.html)  
[kafka入门介绍](https://www.orchome.com/5)

消息过滤器可以在消息抵达监听容器前被拦截，筛选出数据再交由KafkaListener处理。  
只需要为监听容器工厂配置一个RecordFilterStrategy就行。  
返回true的时候消息将会被抛弃，返回false时，消息能正常抵达监听容器。

## 二. 消息过滤

开启消费转发主要1步：

1.  消费者为监听容器工厂配置一个RecordFilterStrategy

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

#### 2.2.2 消费者

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



#### 2.2.3 测试

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