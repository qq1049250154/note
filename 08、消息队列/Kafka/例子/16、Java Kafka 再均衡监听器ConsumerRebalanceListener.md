## 一. 简介

kafka概念相关的介绍请看官方文档和其他博文  
[官方中文文档](http://kafka.apachecn.org/documentation.html)  
[kafka入门介绍](https://www.orchome.com/5)

ConsumerRebalanceListener再均衡监听器提供两个方法：

* onPartitionsRevoked（Collection） 在均衡开始之前和消费者停止读取消息之后调用，一般用来提交偏移量
* onPartitionsAssigned(Collection) 在重新分配分区之后和消费者开始读取消息之前调用，一般用来指定消费偏移量

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


### 2.2 使用事例

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