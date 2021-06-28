## 一. 简介

kafka概念相关的介绍请看官方文档和其他博文  
[官方中文文档](http://kafka.apachecn.org/documentation.html)  
[kafka入门介绍](https://www.orchome.com/5)

Kafka消费者API提供了offsetsForTimes(Map<TopicPartition, Long> timestampsToSearch)方法，入参为Map，Key是待查询分区，Value是待查询的时间戳，返回值是 大于等于时间戳的第一条消息的偏移量和时间戳。如果分区不存在，该方法会一直阻塞。  
如果我们希望从某个时间开始消费，就可以使用它找到偏移量，之后调用seek(TopicPartition partition, long offset)将消费偏移量重置到指定的查询得到的偏移量。

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


### 2.2 消费者–指定时间

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