MQ 全称为 `Message Queue`，是一种应用程序程序对应用程序的通信方式，应用程序通过读写出入队列的消息来通信，而无需专用连接来连接它们。消息传递指的是程序之间通过在消息中发送数据来进行通信，而不是通过直接调用来通信，直接调用通常用于诸如远程过程调用的技术。

------

# 主流 MQ 对比

主流 MQ 有 Kafka、RocketMQ、RabbitMQ 等

## Kafka

Kafka 是 Apache 的一个子项目，使用 Scala 实现的一个高性能分布式 publish/subscribe 消息队列系统，主要特点：

- 快速持久化：通过磁盘顺序读写与零拷贝机制，可以在 O(1) 的系统开销下进行消息持久化
- 高吞吐：在一台普通的服务器上可以达到 10W/S 的吞吐量
- 高堆积：支持 Topic 下消费者长时间离线，消息堆积量大
- 完全的分布式系统，Broker、Producer、Consumer 都原生支持分布式，依赖 ZK 实现负载均衡
- 支持 Hadoop 数据并行加载；对于像 Hadoop 一样的日志数据和离线分析系统，但又要求实时处理的限制，是一个可行的解决方案

## RocketMQ

前身是 Metaq，3.0 版本更名为 RocketMQ，alibaba 出品，现交 Apache 孵化。RocketMQ 是一款分布式、队列模型的消息中间件，特点：

- 能够保证严格的消息顺序
- 提供丰富的消息拉取模式
- 高效的订阅者水平扩展能力
- 实时的消息订阅机制
- 支持事务消息
- 亿级消息堆积能力

## RabbitMQ

使用 Erlang 编写的一个开源的消息队列，本身支持：AMQP、XMPP、SMTP、STOMP 等协议，是一个重量级消息队列，更适合企业级开发。同时也实现的 broker 架构，生产者不会讲消息直接发送给队列，消息在发送给客户端时，现在中心队列排队。
 对路由、负载均衡、数据持久化有很好的支持。

------

# RocketMQ 单机环境搭建

## 版本

- JDK 版本：1.8+
- RocketMQ：4.4.0
- Maven：3.x
- os：CentOS 6.5 x64

## 单机版环境搭建

1. 解压 RocketMQ 4.4.0 到指定文件夹，并修改解压后的文件夹名称



```bash
unzip rocketmq-all-4.4.0-bin-release.zip -d /usr/local/include/mq/
cd /usr/local/include/mq/
mv rocketmq-all-4.4.0-bin-release/ rocketmq
```

1. 创建日志、数据文件夹



```bash
mkdir logs store && cd store
mkdir commitlog consumequeue index
```

1. 修改 `conf/2m-2s-async/broker-a.properties`



```properties
# 所属集群名称
brokerClusterName=rocketmq-cluster
# brijer 名称，不同的配置文件，名称不一样
brokerName=broker-a
# 0 表示 master，大于0表示 salve
brokerId=0
# nameServer 地址，分号分割
namesrvAddr=192.168.52.200:9876
# 在发送消息时，自动创建服务器不存在的 Topic，默认创建的队列数
defaultTopicQueueNums=4
# 是否允许 Broker 自动创建 Topic，生产环境需关闭
autoCreateTopicEnable=true
# 是否允许 Broker 自动创建订阅组，生产环境需关闭
autoCreateSubscriptionGroup=true
# Broker 对外服务的监听端口
listenPort=10911
# 删除文件的时间点，默认是凌晨4点
deleteWhen=04
# 文件保留时间，默认 48 小时
fileReservedTime=48
# commitLog 每个文件的大小，默认 1G
mapedFileSizeCommitLog=1073741824
# consumeQueue 每个文件默认存 30W 条，根据需求调整
mapedFileSizeConsumeQueue=300000
# 检测屋里文件磁盘空间
diskMaxUsedSpaceRatio=88
# 存储路径
storePathRootDir=/usr/local/include/mq/rocketmq/store
# commitLog 存储路径
storePathCommitLog=/usr/local/include/mq/rocketmq/store/commitlog
# 消息队列存储路径
storePathConsumeQueue=/usr/local/include/mq/rocketmq/store/consumequeue
# 消息索引存储路径
storePathIndex=/usr/local/include/mq/rocketmq/store/index
# checkpoint 文件存储路径
storeCheckPoint=/usr/local/include/mq/rocketmq/store/checkpoint
# abort 文件存储路径
abortFile=/usr/local/include/mq/rocketmq/store/abort
# 限制消息大小
maxMessageSize=65535
# broker 角色
# 1、ASYNC_MASTER：异步复制的 Master
# 2、SYNC_MASTER：同步双鞋 Master
# 3、SLAVE：从
brokerRole=ASYNC_MASTER
# 刷盘方式
# 1、ASYNC_FLUSH：异步刷盘
# 2、SYNC_FLUSH：同步刷盘
flushDiskType=ASYNC_FLUSH


#checkTransactionMessageEnable=false

# 发送消息的线程数量
# sendMessageThreadPoolNums=128
# 拉取消息线程池数量
# pullMessageThreadPoolNums=128
```

1. 修改 `conf` 下所有的 xml 文件，将 xml 中的 `${user.home}` 修改为 rocketmq 目录



```dart
sed -i 's#${user.home}#/usr/local/include/mq/rocketmq#g' *.xml
```

1. 修改 `bin/runbroker.sh`、`bin/runserver.sh` 中的 JVM 参数

   ![image-20210517164813850](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517164814.png)

   runbroker.sh

   ![image-20210517164821386](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517164821.png)

   runserver.sh

   

2. 启动 broker



```undefined
nohup sh ./bin/mqnamesrv &
```

1. 使用 `conf/2m-2s-async/broker-a.properties` 配置文件，启动 broker



```jsx
nohup sh ./bin/mqbroker -c /usr/local/include/mq/rocketmq/conf/2m-2s-async/broker-a.properties > /dev/null 2>&1 &
```

1. 使用 jps 查看启动结果

   ![img]()

   jps

## RocketMQ 控制台搭建

下载地址：[https://github.com/apache/rocketmq-externals.git](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fapache%2Frocketmq-externals.git) master 分支，拉取到本地，使用 IDE 打开，修改：`rocketmq-externals-master\rocketmq-console\src\main\resources\application.properties` 配置文件，指定 RocketMQ nameserver 地址(默认端口为 9876)：



```undefined
rocketmq.config.namesrvAddr=192.168.52.200:9876
```

访问 [http://localhost:8080](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A8080)

![image-20210517164835156](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517164835.png)

rocketmq-console



------

# 消息的生产、消费

## 一个简单的消息生产者

使用 SpringBoot 搭建一个简单的消息生产者：



```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>org.apache.rocketmq</groupId>
        <artifactId>rocketmq-client</artifactId>
        <version>${rocketmq.version}</version>
    </dependency>

</dependencies>
```



```java
public class Producer {

    public static void main(String[] args) throws MQClientException, UnsupportedEncodingException, RemotingException, InterruptedException, MQBrokerException {
        // 1、创建 DefaultMQProducer
        DefaultMQProducer producer = new DefaultMQProducer("demo-producer");

        // 2、设置 name server
        producer.setNamesrvAddr("192.168.52.200:9876");

        // 3、开启 producer
        producer.start();

        // 4、创建消息
        Message message = new Message("TOPIC_DEMO", "TAG_A", "KEYS_!", "HELLO！".getBytes(RemotingHelper.DEFAULT_CHARSET));

        // 5、发送消息
        SendResult result = producer.send(message);
        System.out.println(result);
        // 6、关闭 producer
        producer.shutdown();
    }
}
```

运行验证控制台打印信息：



```go
SendResult [sendStatus=SEND_OK, msgId=C0A80067617C18B4AAC26A932C790000, offsetMsgId=C0A834C800002A9F0000000000000000, messageQueue=MessageQueue [topic=TOPIC_DEMO, brokerName=broker-a, queueId=0], queueOffset=0]
16:40:30.148 [NettyClientSelector_1] INFO RocketmqRemoting - closeChannel: close the connection to remote address[192.168.52.200:10909] result: true
16:40:30.152 [NettyClientSelector_1] INFO RocketmqRemoting - closeChannel: close the connection to remote address[192.168.52.200:9876] result: true
16:40:30.152 [NettyClientSelector_1] INFO RocketmqRemoting - closeChannel: close the connection to remote address[192.168.52.200:10911] result: true
```

查看 rocketmq-console 中 `消息` 选项卡，并选择主题为 `TOPIC_DEMO`：

![image-20210517164843367](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517164843.png)

TOPIC_DEMO



点击 `MESSAGE DETAIL` 查看消息具体内容：

![image-20210517164859716](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517164859.png)

message detail



## 一个简单的消息消费者

依赖与生产者一致



```java
public class Consumer {

    public static void main(String[] args) throws MQClientException {
        // 1、创建 DefaultMQPushConsumer
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("demo-consumer");

        // 2、设置 name server
        consumer.setNamesrvAddr("192.168.52.200:9876");

        // 设置消息拉取最大数
        consumer.setConsumeMessageBatchMaxSize(2);

        // 3、设置 subscribe
        consumer.subscribe("TOPIC_DEMO", // 要消费的主题
                "*" // 过滤规则
        );

        // 4、创建消息监听
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
                // 5、获取消息信息
                for (MessageExt msg : list) {
                    // 获取主题
                    String topic = msg.getTopic();
                    // 获取标签
                    String tags = msg.getTags();
                    // 获取信息
                    try {
                        String result = new String(msg.getBody(), RemotingHelper.DEFAULT_CHARSET);
                        System.out.println("Consumer 消费信息：topic：" + topic+ "，tags：" + tags + "，消息体：" + result);
                    } catch (UnsupportedEncodingException e) {
                        e.printStackTrace();
                        return ConsumeConcurrentlyStatus.RECONSUME_LATER;
                    }
                }
                // 6、返回消息读取状态
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });

        // 7、启动消费者（启动后会阻塞）
        consumer.start();
    }
}
```

运行消费者，查看控制台打印信息：



```undefined
Consumer 消费信息：topic：TOPIC_DEMO，tags：TAG_A，消息体：HELLO！
```



# 顺序消息

RocketMQ 顺序消息：消息有序是指可以按照消息发送顺序来消费。RocketMQ 可以严格的保证消息有序，但是这个顺序逼格不是全局顺序，只是分区(queue)顺序。要保证群居顺序，只能有一个分区。



在 MQ 模型中，顺序要由三个阶段保证：

- 消息被发送时，保持顺序
- 消息被存储时的顺序和发送的顺序一致
- 消息被消费时的顺序和存储的顺序一致

发送时保持顺序，意味着对于有顺序要求的消息，用户应该在同一个线程中采用同步的方式发送。存储保持和发送的顺序一致，则要求在同一线程中被发送出来的消息 A/B，存储时 A 要在 B 之前。而消费保持和存储一致，则要求消息 A/B 到达 Consumer 之后必须按照先后顺序被处理。

![image-20210517164909660](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517164909.png)

order

## 生产者



```java
package com.laiyy.study.rocketmqprovider.order;

import org.apache.rocketmq.client.exception.MQBrokerException;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.MessageQueueSelector;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.common.message.MessageQueue;
import org.apache.rocketmq.remoting.common.RemotingHelper;
import org.apache.rocketmq.remoting.exception.RemotingException;

import java.io.UnsupportedEncodingException;
import java.util.List;

/**
 * @author laiyy
 * @date 2019/4/21 16:18
 * @description
 */
public class OrderProducer {

    public static void main(String[] args) throws MQClientException, UnsupportedEncodingException, RemotingException, InterruptedException, MQBrokerException {
        // 1、创建 DefaultMQProducer
        DefaultMQProducer producer = new DefaultMQProducer("demo-producer");

        // 2、设置 name server
        producer.setNamesrvAddr("192.168.52.200:9876");

        // 3、开启 producer
        producer.start();

        // 连续发送 5 条信息
        for (int index = 1; index <= 5; index++) {
            // 创建消息
            Message message = new Message("TOPIC_DEMO", "TAG_A", "KEYS_!", ("HELLO！" + index).getBytes(RemotingHelper.DEFAULT_CHARSET));

            // 指定 MessageQueue，顺序发送消息
            // 第一个参数：消息体
            // 第二个参数：选中指定的消息队列对象（会将所有的消息队列传进来，需要自己选择）
            // 第三个参数：选择对应的队列下标
            SendResult result = producer.send(message, new MessageQueueSelector() {
                // 第一个参数：所有的消息队列对象
                // 第二个参数：消息体
                // 第三个参数：传入的消息队列下标
                @Override
                public MessageQueue select(List<MessageQueue> list, Message message, Object o) {
                    // 获取队列下标
                    int index = (int) o;
                    return list.get(index);
                }
            }, 0);
            System.out.println("发送第：" + index + " 条信息成功：" + result);
        }
        // 关闭 producer
        producer.shutdown();
    }
}
```

控制台输出结果：



```bash
发送第：1 条信息成功：SendResult [sendStatus=SEND_OK, msgId=C0A800677E4C18B4AAC26ACE66560000, offsetMsgId=C0A834C800002A9F00000000000000B8, messageQueue=MessageQueue [topic=TOPIC_DEMO, brokerName=broker-a, queueId=0], queueOffset=1]
发送第：2 条信息成功：SendResult [sendStatus=SEND_OK, msgId=C0A800677E4C18B4AAC26ACE66630001, offsetMsgId=C0A834C800002A9F0000000000000171, messageQueue=MessageQueue [topic=TOPIC_DEMO, brokerName=broker-a, queueId=0], queueOffset=2]
发送第：3 条信息成功：SendResult [sendStatus=SEND_OK, msgId=C0A800677E4C18B4AAC26ACE66660002, offsetMsgId=C0A834C800002A9F000000000000022A, messageQueue=MessageQueue [topic=TOPIC_DEMO, brokerName=broker-a, queueId=0], queueOffset=3]
发送第：4 条信息成功：SendResult [sendStatus=SEND_OK, msgId=C0A800677E4C18B4AAC26ACE66690003, offsetMsgId=C0A834C800002A9F00000000000002E3, messageQueue=MessageQueue [topic=TOPIC_DEMO, brokerName=broker-a, queueId=0], queueOffset=4]
发送第：5 条信息成功：SendResult [sendStatus=SEND_OK, msgId=C0A800677E4C18B4AAC26ACE666C0004, offsetMsgId=C0A834C800002A9F000000000000039C, messageQueue=MessageQueue [topic=TOPIC_DEMO, brokerName=broker-a, queueId=0], queueOffset=5]
17:45:11.545 [NettyClientSelector_1] INFO RocketmqRemoting - closeChannel: close the connection to remote address[192.168.52.200:10909] result: true
17:45:11.548 [NettyClientSelector_1] INFO RocketmqRemoting - closeChannel: close the connection to remote address[192.168.52.200:9876] result: true
17:45:11.549 [NettyClientSelector_1] INFO RocketmqRemoting - closeChannel: close the connection to remote address[192.168.52.200:10911] result: true

Process finished with exit code 0
```

可以看到，所有消息的  `queueId` 都为 0，顺序消息生产成功。

## 消费者



```java
public class OrderConsumer {

    public static void main(String[] args) throws MQClientException {
        // 1、创建 DefaultMQPushConsumer
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("demo-consumer");

        // 2、设置 name server
        consumer.setNamesrvAddr("192.168.52.200:9876");

        // 设置消息拉取最大数
        consumer.setConsumeMessageBatchMaxSize(2);

        // 3、设置 subscribe
        consumer.subscribe("TOPIC_DEMO", // 要消费的主题
                "*" // 过滤规则
        );

        // 4、创建消息监听
        consumer.registerMessageListener(new MessageListenerOrderly() {
            @Override
            public ConsumeOrderlyStatus consumeMessage(List<MessageExt> list, ConsumeOrderlyContext consumeOrderlyContext) {
                // 5、获取消息信息
                for (MessageExt msg : list) {
                    // 获取主题
                    String topic = msg.getTopic();
                    // 获取标签
                    String tags = msg.getTags();
                    // 获取信息
                    try {
                        String result = new String(msg.getBody(), RemotingHelper.DEFAULT_CHARSET);
                        System.out.println("Consumer 消费信息：topic：" + topic+ "，tags：" + tags + "，消息体：" + result);
                    } catch (UnsupportedEncodingException e) {
                        e.printStackTrace();
                        return ConsumeOrderlyStatus.SUSPEND_CURRENT_QUEUE_A_MOMENT;
                    }
                }
                // 6、返回消息读取状态
                return ConsumeOrderlyStatus.SUCCESS;
            }
        });
        // 启动消费者
        consumer.start();
    }
}
```

顺序消费者与之前的 demo 最大的不同，在于 `message listener` 从 `MessageListenerConcurrently` 变为 `MessageListenerOrderly`，消费标识从 `ConsumeConcurrentlyStatus` 变为 `ConsumeOrderlyStatus`。

查看控制台输出：



```undefined
Consumer 消费信息：topic：TOPIC_DEMO，tags：TAG_A，消息体：HELLO！1
Consumer 消费信息：topic：TOPIC_DEMO，tags：TAG_A，消息体：HELLO！2
Consumer 消费信息：topic：TOPIC_DEMO，tags：TAG_A，消息体：HELLO！3
Consumer 消费信息：topic：TOPIC_DEMO，tags：TAG_A，消息体：HELLO！4
Consumer 消费信息：topic：TOPIC_DEMO，tags：TAG_A，消息体：HELLO！5
```

------

# 事务消息

在 RocketMQ 4.3 版本后，开放了事务消息。

## RocketMQ 事务消息流程

RocketMQ 的事务消息，只要是通过消息的异步处理，可以保证本地事务和消息发送同事成功执行或失败，从而保证数据的最终一致性。

![img]()

Transaction message

MQ 事务消息解决分布式事务问题，但是第三方 MQ 支持事务消息的中间件不多，如 RockctMQ，它们支持事务的方式也是类似于采用二阶段提交，但是市面上一些主流的 MQ 都是不支持事务消息的，如：Kafka、RabbitMQ

以 RocketMQ 为例，事务消息实现思路大致为：

- 第一阶段的 Prepared 消息，会拿到消息的地址
- 第二阶段执行本地事务
- 第三阶段通过第一阶段拿到的地址去访问消息，并修改状态

也就是说，在业务方法内想要消息队列提交两次消息，一次发送消息和一次确认消息。如果确认消息发送失败，RocketMQ 会定期扫描消息集群中的事务消息。这时候发现了 prepared 消息，它会向消息发送者确认，所以生产方需要实现一个 check 接口。RocketMQ 会根据发送端设置的策略来决定是回滚还是继续发送确认消息。这样就保证了消息发送与本地事务同时成功或同时失败。



![img]()

Transaction message

事务消息的成功投递需要三个 Topic，分别是

- Half Topic：用于记录所有的 prepare 消息
- Op Half Topic：记录以及提交了状态的 prepare 消息
- Real Topic：事务消息真正的 topic，在 commit 后才会将消息写入该 topic，从而进行消息投递。

## 事务消息实现



```java
public class TransactionProducer {

    public static void main(String[] args) throws MQClientException, UnsupportedEncodingException, RemotingException, InterruptedException, MQBrokerException {
        // 1、创建 TransactionMQProducer
        TransactionMQProducer producer = new TransactionMQProducer("transaction-producer");

        // 2、设置 name server
        producer.setNamesrvAddr("192.168.52.200:9876");

        // 3、指定消息监听对象，用于执行本地事务和消息回查
        TransactionListenerImpl transactionListener = new TransactionListenerImpl();
        producer.setTransactionListener(transactionListener);

        // 4、线程池
        ThreadPoolExecutor executor = new ThreadPoolExecutor(2, 5, 100, TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(2000), new ThreadFactory() {
            @Override
            public Thread newThread(Runnable r) {
                Thread thread = new Thread(r);
                thread.setName("client-transaction-msg-thread");
                return thread;
            }
        });

        producer.setExecutorService(executor);

        // 5、开启 producer
        producer.start();

        // 6、创建消息
        Message message = new Message("TRANSACTION_TOPIC", "TAG_A", "KEYS_!", "HELLO！TRANSACTION!".getBytes(RemotingHelper.DEFAULT_CHARSET));

        // 7、发送消息
        TransactionSendResult result = producer.sendMessageInTransaction(message, "hello-transaction");

        System.out.println(result);

        // 关闭 producer
        producer.shutdown();
    }

}
```

事务消息监听器：



```java
public class TransactionListenerImpl implements TransactionListener {

    /**
     * 存储对应书屋的状态信息， key：事务id，value：事务执行的状态
     */
    private ConcurrentMap<String, Integer> maps = new ConcurrentHashMap<>();

    /**
     * 执行本地事务
     *
     * @param message
     * @param o
     * @return
     */
    @Override
    public LocalTransactionState executeLocalTransaction(Message message, Object o) {
        // 事务id
        String transactionId = message.getTransactionId();

        // 0：执行中，状态未知
        // 1：本地事务执行成功
        // 2：本地事务执行失败

        maps.put(transactionId, 0);

        try {
            System.out.println("正在执行本地事务。。。。");
            // 模拟本地事务
            TimeUnit.SECONDS.sleep(65);
            System.out.println("本地事务执行成功。。。。");
            maps.put(transactionId, 1);
        } catch (InterruptedException e) {
            e.printStackTrace();
            maps.put(transactionId, 2);
            return LocalTransactionState.ROLLBACK_MESSAGE;
        }

        return LocalTransactionState.COMMIT_MESSAGE;
    }

    /**
     * 消息回查
     *
     * @param messageExt
     * @return
     */
    @Override
    public LocalTransactionState checkLocalTransaction(MessageExt messageExt) {
        String transactionId = messageExt.getTransactionId();

        System.out.println("正在执行消息回查，事务id：" + transactionId);

        // 获取事务id的执行状态
        if (maps.containsKey(transactionId)) {
            int status = maps.get(transactionId);
            System.out.println("消息回查状态：" + status);
            switch (status) {
                case 0:
                    return LocalTransactionState.UNKNOW;
                case 1:
                    return LocalTransactionState.COMMIT_MESSAGE;
                default:
                    return LocalTransactionState.ROLLBACK_MESSAGE;
            }
        }
        return LocalTransactionState.UNKNOW;
    }
}
```

运行生产者，查看控制台输出：



```objectivec
正在执行本地事务。。。。
正在执行消息回查，事务id：C0A800678F0818B4AAC26AEDDEB10000
消息回查状态：0
本地事务执行成功。。。。
```

需要注意：消息回查会隔一段时间执行一次，如果执行本地事务的时间太短，则控制台不会输出事务回查日志。

------

# 广播消息

## 生产者



```java
public class Producer {

    public static void main(String[] args) throws MQClientException, UnsupportedEncodingException, RemotingException, InterruptedException, MQBrokerException {
        // 1、创建 DefaultMQProducer
        DefaultMQProducer producer = new DefaultMQProducer("boardcast-producer");

        // 2、设置 name server
        producer.setNamesrvAddr("192.168.52.200:9876");

        // 3、开启 producer
        producer.start();

        for (int index = 1; index <= 10; index++) {
            Message message = new Message("BOARD_CAST_TOPIC", "TAG_A", "KEYS_" + index, ("HELLO！" + index).getBytes(RemotingHelper.DEFAULT_CHARSET));
            SendResult result = producer.send(message);
            System.out.println(result);
        }

        // 关闭 producer
        producer.shutdown();
    }

}
```

## 消费者

消费者需要将消费模式修改为 广播消费：  consumer.setMessageModel(MessageModel.BROADCASTING);



```java
public class Consumer {

    public static void main(String[] args) throws MQClientException {
        // 1、创建 DefaultMQPushConsumer
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("boardcast-consumer");

        // 2、设置 name server
        consumer.setNamesrvAddr("192.168.52.200:9876");

        // 设置消息拉取最大数
        consumer.setConsumeMessageBatchMaxSize(2);


        // 修改消费模式，默认是集群消费模式，修改为广播消费模式
        consumer.setMessageModel(MessageModel.BROADCASTING);

        // 3、设置 subscribe
        consumer.subscribe("BOARD_CAST_TOPIC", // 要消费的主题
                "*" // 过滤规则
        );

        // 4、创建消息监听
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
                // 5、获取消息信息
                for (MessageExt msg : list) {
                    // 获取主题
                    String topic = msg.getTopic();
                    // 获取标签
                    String tags = msg.getTags();
                    // 获取信息
                    try {
                        String result = new String(msg.getBody(), RemotingHelper.DEFAULT_CHARSET);
                        System.out.println("A  Consumer 消费信息：topic：" + topic+ "，tags：" + tags + "，消息体：" + result);
                    } catch (UnsupportedEncodingException e) {
                        e.printStackTrace();
                        return ConsumeConcurrentlyStatus.RECONSUME_LATER;
                    }
                }
                // 6、返回消息读取状态
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        consumer.start();
    }
}
```

## 验证

### 生产者控制台输出



```bash
SendResult [sendStatus=SEND_OK, msgId=C0A80067971418B4AAC26B2965570000, offsetMsgId=C0A834C800002A9F00000000000026D0, messageQueue=MessageQueue [topic=BOARD_CAST_TOPIC, brokerName=broker-a, queueId=1], queueOffset=0]
SendResult [sendStatus=SEND_OK, msgId=C0A80067971418B4AAC26B2965660001, offsetMsgId=C0A834C800002A9F000000000000278F, messageQueue=MessageQueue [topic=BOARD_CAST_TOPIC, brokerName=broker-a, queueId=2], queueOffset=10]
SendResult [sendStatus=SEND_OK, msgId=C0A80067971418B4AAC26B29656C0002, offsetMsgId=C0A834C800002A9F000000000000284E, messageQueue=MessageQueue [topic=BOARD_CAST_TOPIC, brokerName=broker-a, queueId=3], queueOffset=0]
SendResult [sendStatus=SEND_OK, msgId=C0A80067971418B4AAC26B2965700003, offsetMsgId=C0A834C800002A9F000000000000290D, messageQueue=MessageQueue [topic=BOARD_CAST_TOPIC, brokerName=broker-a, queueId=0], queueOffset=0]
SendResult [sendStatus=SEND_OK, msgId=C0A80067971418B4AAC26B29657B0004, offsetMsgId=C0A834C800002A9F00000000000029CC, messageQueue=MessageQueue [topic=BOARD_CAST_TOPIC, brokerName=broker-a, queueId=1], queueOffset=1]
SendResult [sendStatus=SEND_OK, msgId=C0A80067971418B4AAC26B2965880005, offsetMsgId=C0A834C800002A9F0000000000002A8B, messageQueue=MessageQueue [topic=BOARD_CAST_TOPIC, brokerName=broker-a, queueId=2], queueOffset=11]
SendResult [sendStatus=SEND_OK, msgId=C0A80067971418B4AAC26B29658E0006, offsetMsgId=C0A834C800002A9F0000000000002B4A, messageQueue=MessageQueue [topic=BOARD_CAST_TOPIC, brokerName=broker-a, queueId=3], queueOffset=1]
SendResult [sendStatus=SEND_OK, msgId=C0A80067971418B4AAC26B2965960007, offsetMsgId=C0A834C800002A9F0000000000002C09, messageQueue=MessageQueue [topic=BOARD_CAST_TOPIC, brokerName=broker-a, queueId=0], queueOffset=1]
SendResult [sendStatus=SEND_OK, msgId=C0A80067971418B4AAC26B29659D0008, offsetMsgId=C0A834C800002A9F0000000000002CC8, messageQueue=MessageQueue [topic=BOARD_CAST_TOPIC, brokerName=broker-a, queueId=1], queueOffset=2]
SendResult [sendStatus=SEND_OK, msgId=C0A80067971418B4AAC26B2965AB0009, offsetMsgId=C0A834C800002A9F0000000000002D87, messageQueue=MessageQueue [topic=BOARD_CAST_TOPIC, brokerName=broker-a, queueId=2], queueOffset=12]
19:24:35.135 [NettyClientSelector_1] INFO RocketmqRemoting - closeChannel: close the connection to remote address[192.168.52.200:10911] result: true
19:24:35.140 [NettyClientSelector_1] INFO RocketmqRemoting - closeChannel: close the connection to remote address[192.168.52.200:9876] result: true
19:24:35.140 [NettyClientSelector_1] INFO RocketmqRemoting - closeChannel: close the connection to remote address[192.168.52.200:10909] result: true
```

### 消费者控制台输出



```undefined
A  Consumer 消费信息：topic：BOARD_CAST_TOPIC，tags：TAG_A，消息体：HELLO！1
A  Consumer 消费信息：topic：BOARD_CAST_TOPIC，tags：TAG_A，消息体：HELLO！2
A  Consumer 消费信息：topic：BOARD_CAST_TOPIC，tags：TAG_A，消息体：HELLO！5
A  Consumer 消费信息：topic：BOARD_CAST_TOPIC，tags：TAG_A，消息体：HELLO！4
A  Consumer 消费信息：topic：BOARD_CAST_TOPIC，tags：TAG_A，消息体：HELLO！3
A  Consumer 消费信息：topic：BOARD_CAST_TOPIC，tags：TAG_A，消息体：HELLO！7
A  Consumer 消费信息：topic：BOARD_CAST_TOPIC，tags：TAG_A，消息体：HELLO！6
A  Consumer 消费信息：topic：BOARD_CAST_TOPIC，tags：TAG_A，消息体：HELLO！8
A  Consumer 消费信息：topic：BOARD_CAST_TOPIC，tags：TAG_A，消息体：HELLO！9
A  Consumer 消费信息：topic：BOARD_CAST_TOPIC，tags：TAG_A，消息体：HELLO！10
```



```undefined
B  Consumer 消费信息：topic：BOARD_CAST_TOPIC，tags：TAG_A，消息体：HELLO！1
B  Consumer 消费信息：topic：BOARD_CAST_TOPIC，tags：TAG_A，消息体：HELLO！2
B  Consumer 消费信息：topic：BOARD_CAST_TOPIC，tags：TAG_A，消息体：HELLO！3
B  Consumer 消费信息：topic：BOARD_CAST_TOPIC，tags：TAG_A，消息体：HELLO！5
B  Consumer 消费信息：topic：BOARD_CAST_TOPIC，tags：TAG_A，消息体：HELLO！4
B  Consumer 消费信息：topic：BOARD_CAST_TOPIC，tags：TAG_A，消息体：HELLO！6
B  Consumer 消费信息：topic：BOARD_CAST_TOPIC，tags：TAG_A，消息体：HELLO！7
B  Consumer 消费信息：topic：BOARD_CAST_TOPIC，tags：TAG_A，消息体：HELLO！8
B  Consumer 消费信息：topic：BOARD_CAST_TOPIC，tags：TAG_A，消息体：HELLO！9
B  Consumer 消费信息：topic：BOARD_CAST_TOPIC，tags：TAG_A，消息体：HELLO！10
```

RocketMQ 集群模式分为四种：`单 master`、`多 master`、`多 master 多 slave 异步复制`、`多 master 多 slave 同步双写`

# 四种集群模式

## 单 master

风险较大，一旦 broker 宕机或者重启，将导致整个服务部可用。不建议线上环境使用

## 多 master

一个集群全部都是 master，没有 slave

- 优点
   配置简单，单个 master 宕机，或者重启未付，对应用没有影响，在磁盘配置为 RAID10 时，即是机器宕机不可恢复的情况，消息也不会丢失（异步刷盘会丢失少量消息，同步刷盘不会丢失消息），性能最高
- 缺点
   单个 broker 宕机期间，这台机器上未被消费的消息在机器恢复之前不可订阅，消息的实时性会受到影响。

## 多 master 多 slave 异步复制

每个 master 配置一个 slave，有多对 master slave，HA 采用的是异步复制方式，主备有短暂的消息延迟（毫秒级），master 收到消息后立即向应用返回成功标志，同时向 slave 写入消息。

- 优点
   即是磁盘损坏，消息丢失的非常少，且消息的实时性不会受到影响。因为 master 宕机后，消费者仍然可以从 slave 消费，此过程对应用透明，不需要人工干预，性能同多个 master 模式一样
- 缺点
   master 宕机，磁盘损坏下，会丢失少量消息

## 多 master 多 slave 同步双写

每个 master 配置一个 slave，有多对 master slave，HA 采用同步双写模式，主备都成功才会返回成功

- 优点
   数据与服务都无单点，master 宕机情况下，消息无延迟，服务可用性与数据可用性最高
- 缺点
   性能比异步复制低 10% 左右，发送单个 master 的 RT 会略高，主机宕机后，slave 不能自动切换为主机（后续版本会支持）

------

# 一主一从

## 修改 master 配置

进入 `conf/2m-2s-async`，修改文件：`broker-a-s.properties`:



```css
rm -rf broker-a-s.properties 
cp broker-a.properties  broker-a-s.properties
```

然后打开 `broker-a-s.properties`，修改：



```undefined
brokerId=1
brokerRole=SLAVE
```

修改两个配置文件的 nameserver 为两个服务器对应的 nameserver 地址，多个地址用英文分号分割

## 修改 slave 配置

将 master 的 `broker-a.properties`、`broker-a-s.properties` 同步过来，在 master 上执行



```ruby
scp broker-a.properties 192.168.52.201:/usr/local/include/mq/rocketmq/conf/2m-2s-async/
scp broker-a-s.properties 192.168.52.201:/usr/local/include/mq/rocketmq/conf/2m-2s-async/
```

## 启动集群

依次启动 master、slave 的 nameserver



```undefined
nohup ./bin/mqnamesrv &
```

在 master 上使用 `broker-a.properties` 启动 broker



```jsx
nohup sh ./bin/mqbroker -c /usr/local/include/mq/rocketmq/conf/2m-2s-async/broker-a.properties > /dev/null 2>&1 &
```

在 slave 上使用 `broker-a-s.properties` 启动 broker



```jsx
nohup sh ./bin/mqbroker -c /usr/local/include/mq/rocketmq/conf/2m-2s-async/broker-a-s.properties > /dev/null 2>&1 &
```

## 验证集群

在 rocketmq-console 中，修改 nameserver 配置：



```undefined
rocketmq.config.namesrvAddr=192.168.52.200:9876;192.168.52.201:9876
```

启动 console，并查看集群属性



![img](https:////upload-images.jianshu.io/upload_images/13856126-7a9c6948a714c69b.png?imageMogr2/auto-orient/strip|imageView2/2/w/964/format/webp)

一主一从

## 缺陷

当主节点挂掉后，消息将无法写入

------

# 双主双从

双主双从，异步刷盘，同步复制（生产环境建议采用此方式）

## 集群搭建

准备4份 RocketMQ 环境，修改配置文件 `conf/2m-2s-sync/broker-a.properties`，将 `brokerRole` 改为：`SYNC_MASTER`,`flushDiskType` 改为 `ASYNC_FLUSH`，nameserver 为四台服务器的 nameserver 地址其他与之前 async 的配置一样

修改 `conf/2m-2s-sync/broker-a-s.0properties` 的 `brokerId` 为大于 0 的值，`brokerRole` 为 `SLAVE`，nameserver 为四台服务器的 nameserver 地址。

修改 `conf/2m-2s-sync/broker-b.0properties`、`conf/2m-2s-sync/broker-b-2.0properties`，与 a 的区别在与 `brokerName` 都为 broker-b

## 启动集群

每台机器都启动 nameserveer
 `nohup ./bin/mqnamesrv &`

在第一台机器上启动 broker-a



```dart
nohup sh ./bin/mqbroker -c /usr/local/include/mq/rocketmq/conf/2m-2s-sync/broker-a.properties > /dev/null 2>&1 &
```

在第二台机器上启动 broker-b



```dart
nohup sh ./bin/mqbroker -c /usr/local/include/mq/rocketmq/conf/2m-2s-sync/broker-b.properties > /dev/null 2>&1 &
```

在第三台机器上启动 broker-a-s



```bash
nohup sh ./bin/mqbroker -c /usr/local/include/mq/rocketmq/conf/2m-2s-sync/broker-a-s.properties > /dev/null 2>&1 &
```

在第四台机器上启动 broker-b-s



```dart
nohup sh ./bin/mqbroker -c /usr/local/include/mq/rocketmq/conf/2m-2s-sync/broker-b-s.properties > /dev/null 2>&1 &
```

## 验证集群

修改 rocket-console 的配置：`rocketmq.config.namesrvAddr=192.168.52.200:9876;192.168.52.201:9876;192.168.52.202:9876;192.168.52.203:9876`，启动 console，打开 `集群选项卡`：

![img](https:////upload-images.jianshu.io/upload_images/13856126-8fd4762ba4210d67.png?imageMogr2/auto-orient/strip|imageView2/2/w/1104/format/webp)

双主双从-同步双写-异步刷盘

# 参考

https://www.jianshu.com/p/d9794470d274

https://www.jianshu.com/p/98ead09e8dac

ttps://www.jianshu.com/p/970decf53bc1

