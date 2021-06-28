## 一. 简介

kafka概念相关的介绍请看官方文档和其他博文  
[官方中文文档](http://kafka.apachecn.org/documentation.html)  
[kafka入门介绍](https://www.orchome.com/5)

Kafka0.11.0.0版本之后，使用AdminClient作为集群管理工具，这个是在kafka-client包下的，这是一个抽象类，具体的实现是org.apache.kafka.clients.admin.KafkaAdminClient。

KafkaAdminClient包含了功能：

1.  创建Topic：createTopics(Collection newTopics)
2.  删除Topic：deleteTopics(Collection topics)
3.  罗列所有Topic：listTopics()
4.  查询Topic：describeTopics(Collection topicNames)
5.  查询集群信息：describeCluster()
6.  查询ACL信息：describeAcls(AclBindingFilter filter)
7.  创建ACL信息：createAcls(Collection acls)
8.  删除ACL信息：deleteAcls(Collection filters)
9.  查询配置信息：describeConfigs(Collection resources)
10.  修改配置信息：alterConfigs(Map<ConfigResource, Config> configs)
11.  修改副本的日志目录：alterReplicaLogDirs(Map<TopicPartitionReplica, String> replicaAssignment)
12.  查询节点的日志目录信息：describeLogDirs(Collection brokers)
13.  查询副本的日志目录信息：describeReplicaLogDirs(Collection replicas)
14.  增加分区：createPartitions(Map<String, NewPartitions> newPartitions)

## 二. 集群管理工具AdminClient

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

```prism
kafka.bootstrap-servers=localhost:9092
server.port=9093
```

### 2.2 AdminClient工具类

**KafkaAdminUtil.java**

```java
import org.apache.kafka.clients.admin.*;
import org.apache.kafka.common.KafkaFuture;
import org.apache.kafka.common.Node;
import org.apache.kafka.common.config.ConfigResource;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import java.util.*;
import java.util.concurrent.ExecutionException;

/**
 * @author 司马缸砸缸了
 * @date 2019/12/31 15:08
 * @description
 */

/**
 * 主要功能包括：
 *
 * 创建Topic：createTopics(Collection<NewTopic> newTopics)
 * 删除Topic：deleteTopics(Collection<String> topics)
 * 显示所有Topic：listTopics()
 * 查询Topic：describeTopics(Collection<String> topicNames)
 * 查询集群信息：describeCluster()
 * 查询ACL信息：describeAcls(AclBindingFilter filter)
 * 创建ACL信息：createAcls(Collection<AclBinding> acls)
 * 删除ACL信息：deleteAcls(Collection<AclBindingFilter> filters)
 * 查询配置信息：describeConfigs(Collection<ConfigResource> resources)
 * 修改配置信息：alterConfigs(Map<ConfigResource, Config> configs)
 * 修改副本的日志目录：alterReplicaLogDirs(Map<TopicPartitionReplica, String> replicaAssignment)
 * 查询节点的日志目录信息：describeLogDirs(Collection<Integer> brokers)
 * 查询副本的日志目录信息：describeReplicaLogDirs(Collection<TopicPartitionReplica> replicas)
 * 增加分区：createPartitions(Map<String, NewPartitions> newPartitions)
 */
@Component
public class KafkaAdminUtil {

    @Autowired
    private AdminClient adminClient;

    /**
     * 创建主题
     */
    public void testCreateTopic() throws InterruptedException {
        NewTopic topic = new NewTopic("topic.quick.simagangl", 3, (short) 1);
        adminClient.createTopics(Arrays.asList(topic));
        Thread.sleep(1000);
    }

    /**
     * 查看主题
     */
    public void testSelectTopicInfo() throws ExecutionException, InterruptedException {
        DescribeTopicsResult result = adminClient.describeTopics(Arrays.asList("topic.quick.simagangl"));
        result.all().get().forEach((k, v) -> System.out.println("k: " + k + " ,v: " + v.toString() + "\n"));
    }

    /**
     * 删除主题
     */
    public void deleteTopic() throws InterruptedException {
        adminClient.deleteTopics(Arrays.asList("topic.quick.simagangl"));
        Thread.sleep(1000);
    }

    /**
     * topic配置描述
     */
    public void describeConfig() throws ExecutionException, InterruptedException {
        DescribeConfigsResult ret = adminClient.describeConfigs(Collections.singleton(new ConfigResource(ConfigResource.Type.TOPIC, "batch_topic")));
        Map<ConfigResource, Config> configs = ret.all().get();
        for (Map.Entry<ConfigResource, Config> entry : configs.entrySet()) {
            ConfigResource key = entry.getKey();
            Config value = entry.getValue();
            System.out.println(String.format("Resource type: %s, resource name: %s", key.type(), key.name()));
            Collection<ConfigEntry> configEntries = value.entries();
            for (ConfigEntry each : configEntries) {
                System.out.println(each.name() + " = " + each.value());
            }
        }
    }

    /**
     * 集群描述
     *
     * @throws ExecutionException
     * @throws InterruptedException
     */
    public void describeCluster() throws ExecutionException, InterruptedException {
        DescribeClusterResult ret = adminClient.describeCluster();
        System.out.println(String.format("Cluster id: %s, controller: %s", ret.clusterId().get(), ret.controller().get()));
        System.out.println("Current cluster nodes info: ");
        for (Node node : ret.nodes().get()) {
            System.out.println(node);
        }

        Thread.sleep(1000);
    }

    /**
     * 更新topic配置
     */
    public void alterConfigs() throws ExecutionException, InterruptedException {
        Config topicConfig = new Config(Arrays.asList(new ConfigEntry("cleanup.policy", "compact")));
        adminClient.alterConfigs(Collections.singletonMap(
                new ConfigResource(ConfigResource.Type.TOPIC, "batch_topic"), topicConfig)).all().get();
    }

    /**
     * 删除指定主题
     */
    public void deleteTopics() throws ExecutionException, InterruptedException {
        KafkaFuture<Void> futures = adminClient.deleteTopics(Arrays.asList("batch_topic")).all();
        futures.get();
    }

    /**
     * 描述给定的主题
     */
    public void describeTopics() throws ExecutionException, InterruptedException {
        DescribeTopicsResult ret = adminClient.describeTopics(Arrays.asList("batch_topic", "__consumer_offsets"));
        Map<String, TopicDescription> topics = ret.all().get();
        for (Map.Entry<String, TopicDescription> entry : topics.entrySet()) {
            System.out.println(entry.getKey() + " ===> " + entry.getValue());
        }
    }

    /**
     * 打印集群中的所有主题
     */
    public void listAllTopics() throws ExecutionException, InterruptedException {
        ListTopicsOptions options = new ListTopicsOptions();
        // 包括内部主题，如_consumer_offsets
        options.listInternal(true);
        ListTopicsResult topics = adminClient.listTopics(options);
        Set<String> topicNames = topics.names().get();
        System.out.println("Current topics in this cluster: " + topicNames);
    }
}
```

其他用法请参看API

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