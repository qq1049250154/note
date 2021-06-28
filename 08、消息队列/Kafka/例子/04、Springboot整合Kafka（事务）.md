## 一. 简介

kafka概念相关的介绍请看官方文档和其他博文  
[官方中文文档](http://kafka.apachecn.org/documentation.html)  
[kafka入门介绍](https://www.orchome.com/5)

事务不止用在数据库中，kafka中我们并不希望消息监听器接收到一些错误的消息，使用事务就会进行回滚，不会发送到topic中。

有两种实现事务方式：

1.  @Transactional注解
2.  使用KafkaTemplate.executeInTransaction开启事务

## 二. @Transactional事务

开启批量消费主要3步

1.  生产者开启事务 factory.transactionCapable();
2.  生产者初始化事务管理器KafkaTransactionManager
3.  生产者添加@Transactional

### 2.1 引入依赖

主要是spring-kafka依赖

```java
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



### 2.2 Kafka配置

此处我们可以在application.properties中配置，也可以使用Java Config。我使用Java Config，看得更直观。

#### 2.2.1 生产者

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



#### 2.2.2 消费者

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



#### 2.2.3 测试

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

## 三. 使用KafkaTemplate.executeInTransaction开启事务

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