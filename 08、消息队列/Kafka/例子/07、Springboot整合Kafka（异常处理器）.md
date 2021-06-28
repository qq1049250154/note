## 一. 简介

kafka概念相关的介绍请看官方文档和其他博文  
[官方中文文档](http://kafka.apachecn.org/documentation.html)  
[kafka入门介绍](https://www.orchome.com/5)

Kafka异常处理器为了方便异常统一管理，使业务逻辑处理与异常分离开。  
KafkaListener中所抛出的异常都会经过ConsumerAwareErrorHandler异常处理器进行处理。

## 二. 异常处理器

开启消费转发主要1步：

1.  消费者配置ConsumerAwareErrorHandler

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

#### 2.2.2 消费者

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

#### 2.2.3 测试

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