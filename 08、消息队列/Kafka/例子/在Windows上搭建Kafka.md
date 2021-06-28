## 一、下载、安装Kafka

访问Kafka的主页：

[Apache Kafkakafka.apache.org<img src="https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210427105151.png" alt="图标" style="zoom:33%;" />](https://link.zhihu.com/?target=http%3A//kafka.apache.org/)

进入其下载页面，截图如下：

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210427105154.jpeg)

选择相应的版本，这里选择 kafka_2.11-2.4.0.tgz，进入下面的页面：

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210427105157.jpeg)

选择清华的镜像站点进行下载。

下载到本地后，将文件解压到 D:\kafka_2.11-2.4.0，该文件夹包括了所有相关的运行文件及配置文件，其子文件夹bin\windows 下放的是在Windows系统启动zookeeper和kafka的可执行文件，子文件夹config下放的是zookeeper和kafka的配置文件。



## 二、启动kafka服务

我们需要先后启动zookeeper和kafka服务。

它们都需要进入 D:\kafka_2.11-2.4.0 目录，然后再启动相应的命令。

```text
cd D:\kafka_2.11-2.4.0
```

启动zookeeper服务，运行命令：

```text
bin\windows\zookeeper-server-start.bat config\zookeeper.properties
```

启动kafka服务，运行命令：

```text
bin\windows\kafka-server-start.bat config\server.properties
```

## 三、创建Topic，显示数据

Kafka中创建一个Topic，名称为iris

```text
bin\windows\kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic iris
```

创建成功后，可以使用如下命令，显示所有Topic的列表：

```text
bin\windows\kafka-topics.bat --list --zookeeper localhost:2181 
```

显示结果为

```text
iris
```

然后，我们通过Alink的 Kafka SinkStreamOp可以将iris数据集写入该Topic。这里不详细展开，有兴趣的读者可以参阅如下的文章。

[Alink品数：Alink连接Kafka数据源（Python版本）zhuanlan.zhihu.com![图标](https://pic4.zhimg.com/zhihu-card-default_ipico.jpg)](https://zhuanlan.zhihu.com/p/101143978)[Alink品数：Alink连接Kafka数据源（Java版本）zhuanlan.zhihu.com![图标](https://pic4.zhimg.com/zhihu-card-default_ipico.jpg)](https://zhuanlan.zhihu.com/p/101106492)



使用如下命令，读取（消费）topic iris中的数据：

```bash
 bin\windows\kafka-console-consumer.bat --bootstrap-server 127.0.0.1:9092 --topic iris --from-beginning
```

显示结果如下，略去了中间的大部分数据：

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210427105159.jpeg)

```text
{"sepal_width":3.4,"petal_width":0.2,"sepal_length":4.8,"category":"Iris-setosa","petal_length":1.6}
{"sepal_width":4.1,"petal_width":0.1,"sepal_length":5.2,"category":"Iris-setosa","petal_length":1.5}
{"sepal_width":2.8,"petal_width":1.5,"sepal_length":6.5,"category":"Iris-versicolor","petal_length":4.6}
{"sepal_width":3.0,"petal_width":1.8,"sepal_length":6.1,"category":"Iris-virginica","petal_length":4.9}
{"sepal_width":2.9,"petal_width":1.8,"sepal_length":7.3,"category":"Iris-virginica","petal_length":6.3}
...........
{"sepal_width":2.2,"petal_width":1.0,"sepal_length":6.0,"category":"Iris-versicolor","petal_length":4.0}
{"sepal_width":2.4,"petal_width":1.0,"sepal_length":5.5,"category":"Iris-versicolor","petal_length":3.7}
{"sepal_width":3.1,"petal_width":0.2,"sepal_length":4.6,"category":"Iris-setosa","petal_length":1.5}
{"sepal_width":3.4,"petal_width":0.2,"sepal_length":4.8,"category":"Iris-setosa","petal_length":1.9}
{"sepal_width":2.9,"petal_width":1.4,"sepal_length":6.1,"category":"Iris-versicolor","petal_length":4.7}
```