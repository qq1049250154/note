1. ### 主从架构应用场景

   -  主从架构是为了分担单节点访问的压力以及单节点故障期间仍可提供**读**操作服务； 
   -  由主节点提供**读写操作**，从节点只提供**读**操作，这样避免了多节点写导致的写操作互相同步问题，只需要从主节点同步至从节点即可； 

2. ### 主从架构数据同步过程

   -  全量同步 当

     第一次进行数据同步或repl_backlog_buffer被覆盖掉

     就会触发全量同步。全量同步的过程如下： 

     1.  从库发送psync命令到主库，主库启动同步； 
     2.  主库使用FULLRESYNC命令将主库的runID（主库实例id）和offset（复制进度）发送给从库； 
     3.  主库开启bgsave子进程，生成RDB快照； 
     4.  将RDB快照发送给从库；（这里是子进程去执行的，但是仍然会影响[Redis](https://cloud.tencent.com/product/crs?from=10680)性能，因为占用网络IO） 
     5.  从库加载RDB文件； 
     6.  在发送及同步时，主库仍然在处理客户端请求，在这个期间内的操作会被保存在replication_buffer中，当从库加载完RDB后回调通知主库，主库再把replication_buffer中的操作发送给从库。 `全量同步过程` 

     ![img](file:///C:/Users/Administrator/Documents/My Knowledge/temp/a22d5fea-9f5f-485e-a9be-066e933d1484/128/index_files/vyduodkohg.jpg)

   -  增量同步 主从建立连接之后，会分配一个repli_backlog_buffer，这是一个环形的缓冲区，主库记录master_repl_offset，从库记录slave_repl_offset。当主库发生写操作时，会把操作命令写入repli_backlog_buffer，并增加master_repl_offset的值，从库从repli_backlog_buffer读取数据并增加slave_repl_offset，如果因从库读取slave_repl_offset过慢，导致repli_backlog_buffer被覆盖重写，则需要进行全量同步。 可以通过repl_backlog_size参数来设置合理的repli_backlog_buffer，避免触发全量同步。 `例如缓冲空间的计算公式是：缓冲空间大小 = 主库写入命令速度 * 操作大小 - 主从库间网络传输命令速度 * 操作大小。如果主库每秒写入 2000 个操作，每个操作的大小为 2KB，网络每秒能传输 1000 个操作，那么，有 1000 个操作需要缓冲起来，这就至少需要 2MB 的缓冲空间。否则，新写的命令就会覆盖掉旧操作了。为了应对可能的突发压力，我们最终把 repl_backlog_size 设为 4MB。` `repli_backlog_buffer使用过程` 

   ![img](file:///C:/Users/Administrator/Documents/My Knowledge/temp/a22d5fea-9f5f-485e-a9be-066e933d1484/128/index_files/cdwpkqgu9n.jpg)

    `增量复制过程` 

   ![img](file:///C:/Users/Administrator/Documents/My Knowledge/temp/a22d5fea-9f5f-485e-a9be-066e933d1484/128/index_files/38d4c78x9u.jpg)

3.  **主从架构部署结构** 主从架构模式 

1.  主从架构模式 

![bpanx3hfil](redis主从架构.assets/bpanx3hfil.png)

 以上架构，如果从库较多，那么会导致主库过多的在处理主从同步工作，可以选取性能比较好的从库，作为二级主库，实现 主-从-从 的模式，分摊主库的同步工作

![8efd7qeem5](redis主从架构.assets/8efd7qeem5.jpeg)