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


### 2.2 简单消费者

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