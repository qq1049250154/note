[MySQL](https://cloud.tencent.com/product/cdb?from=10680) 中存在慢查询，[Redis](https://cloud.tencent.com/product/crs?from=10680) 中也存在慢查询，Redis 的慢查询是**命令执行超过设定阈值的查询就是慢查询**。我们来整理一下。

**慢查询**

**Redis 会记录命令执行时间超过设定阈值时间的命令，这里的慢查询说的是命令执行慢，并非是 I/O 慢。**


![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210201143518.png)

一般情况下，我们都是通过客户端连接 Redis 服务器，然后发送命令给 Redis 服务器，Redis 服务器会把每个客户端发来的命令缓存入一个队列，然后逐个进行执行，最后再把结果返回给客户端。而我们这里的**慢查询指的就是“执行命令”的那部分**。而非网络 I/O 或者 命令排队的问题。

**关于慢查询的配置**

慢查询的配置有两条，分别如下：


![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210201143522.png)

slowlog-log-slower-than: **慢查询阈值，命令执行时超过该配置参数设定的值，则被认为是慢查询；**

slowlog-max-len: **慢查询日志最大记录数**，也就是 Redis 最多记录多少条慢查询的记录。慢查询的记录是一个 list 类型，如果最大允许记录 100 条，那么在记录第 101 条时，第 1 条记录就会被移除，始终保持慢查询的记录数为 100 条。

**慢查询相关的命令**

这里来看几条命令，这些命令都是与慢查询相关的命令，命令与配置的截图如下：


![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210201143524.png)

config 命令是用于配置的命令，不单单仅限于使用到慢查询中，由于我们会使用这两条命令，因此也在这里提一下。config 命令可以动态的获取和设置 Redis 服务器的部分参数，通常我们查看和设置参数是进入 redis.conf 这个文件中，但是在运行时可以通过 config 命令来快速的获取和设置参数。

来使用 config get 获取 slowlog-log-slower-than 和 slowlog-max-len 两个参数，命令如下。

```
127.0.0.1:6379> config get slowlog-log-slower-than
1) "slowlog-log-slower-than"
2) "10000"
127.0.0.1:6379> config get slowlog-max-len
1) "slowlog-max-len"
2) "128"
```

从运行的返回结果可以看到，slowlog-log-slower-than 的值为 10000，该值的单位为微秒，那么 10000 微秒就是 10 毫秒。slowlog-max-len 的值为 128，即 Redis 只保存 128 条慢查询的记录。

slowlog-log-slower-than 如果设置为 0，则表示记录所有的命令到慢查询队列中，如果 slowlog-log-slower-than 设置为 小于 0 的值，则表示不记录任何命令到慢查询队列中。

在 redis-cli 下，使用 config 命令竟然没有语法提示！

上面两个配置是关于慢查询的配置，关于慢查询的命令 Redis 提供了 slowlog 的命令，该命令可以提供一些参数，介绍如下。

slowlog get [n]: **获取[指定条数]的慢查询列表**；

slowlog len: **获取慢查询记录条数**；

slowlog reset: **清空慢查询列表**。

以上就是关于慢查询的配置和命令了。

**命令演示**

整个关于慢查询的配置和命令并不是很多，这里的命令演示，主要用来看看 slowlog get 命令的返回结果。

**查看慢查询的配置参数**

来看 slowlog-log-slower-than 和 slowlog-max-len 两个配置参数的值。

```
127.0.0.1:6379> config get slowlog-log-slower-than
1) "slowlog-log-slower-than"
2) "10000"
127.0.0.1:6379> config get slowlog-max-len
1) "slowlog-max-len"
2) "128"
```

slowlog-log-slower-than 的默认值是 10000 微秒，也就是 10 毫秒，这个在前面已经描述过。slowlog-max-len 的默认值是 128，也就是说慢查询命令队列可以保存 128 条慢查询记录。

**设置慢查询配置参数**

将两个参数的值设置为适合我们进行学习的值。

```
127.0.0.1:6379> config set slowlog-log-slower-than 0
OK
127.0.0.1:6379> config set slowlog-max-len 5
OK
```

slowlog-log-slower-than 的值设置为 0，这样可以把所有的命令都记录下来，然后将 slowlog-max-len 的值设置为 5，也就是只保存 5 条慢查询记录。

**随便测试几条命令**

随便测试几条命令，方便我们后面查看慢查询中的内容。

```
127.0.0.1:6379> keys *
1) "k1"
2) "k2"
127.0.0.1:6379> get k1
"v1"
127.0.0.1:6379> hgetall k2
1) "k21"
2) "v1"
3) "k22"
4) "v2"
```

**查看 Redis 记录的慢查询数量**

通过 slowlog len 命令来查看记录了多少条慢查询命令。

```
127.0.0.1:6379> slowlog len
(integer) 5
```

**查看慢查询**

通过 slowlog get 命令来查看慢查询列表。

```
127.0.0.1:6379> slowlog get
1) 1) (integer) 6
   2) (integer) 1610156168
   3) (integer) 2
   4) 1) "slowlog"
      2) "len"
   5) "127.0.0.1:58406"
   6) ""
2) 1) (integer) 5
   2) (integer) 1610156086
   3) (integer) 10
   4) 1) "hgetall"
      2) "k2"
   5) "127.0.0.1:58406"
   6) ""
3) 1) (integer) 4
   2) (integer) 1610156073
   3) (integer) 9
   4) 1) "get"
      2) "k1"
   5) "127.0.0.1:58406"
   6) ""
4) 1) (integer) 3
   2) (integer) 1610156064
   3) (integer) 6
   4) 1) "keys"
      2) "*"
   5) "127.0.0.1:58406"
   6) ""
5) 1) (integer) 2
   2) (integer) 1610155960
   3) (integer) 5
   4) 1) "config"
      2) "set"
      3) "slowlog-max-len"
      4) "5"
   5) "127.0.0.1:58406"
   6) ""
```

可以看到上面的列表就是所有慢查询的命令，因为我们只保留了 5 条。为了查看方便我们只查看一条，命令如下。

```
127.0.0.1:6379> slowlog get 1
1) 1) (integer) 7
   2) (integer) 1610156232
   3) (integer) 24
   4) 1) "slowlog"
      2) "get"
   5) "127.0.0.1:58406"
   6) ""
```

在上面的输出中，一共有六行，分别解释：

\1. 记录的慢查询标号，倒序显示

\2. 记录该命令的时间戳

\3. 执行命令的耗时，微秒为单位

\4. 执行的具体命令

\5. 执行该命令客户端的 IP 地址和端口号

有了上面的信息就可以方便我们定位是哪个客户端执行了哪个命令导致的慢查询了。

总结

Redis 变慢的情况可能是对某些数据结构做了比较慢的操作，也可能是用了不合适的数据结构等。当然了，导致 Redis 变慢的情况比较多，不单单是因为执行命令部分导致，但是慢查询只能帮我们记录执行慢的命令，至于导致 Redis 的慢的原因，要多方面的查找。