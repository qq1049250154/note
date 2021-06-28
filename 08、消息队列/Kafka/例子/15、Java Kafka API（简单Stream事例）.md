## 一. 简介

kafka概念相关的介绍请看官方文档和其他博文  
[官方中文文档](http://kafka.apachecn.org/documentation.html)  
[kafka入门介绍](https://www.orchome.com/5)

Kafka Streams从一个或多个输入topic进行连续的计算并输出到0或多个外部topic中。

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


### 2.2 简单Stream事例

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


### 2.3 IP访问检测

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