## 一、背景



### 1.1 拜神

spring事务领头人叫Juergen Hoeller，于尔根·糊了...先混个脸熟哈，他写了几乎全部的spring事务代码。读源码先拜神，掌握他的源码的风格，读起来会通畅很多。最后一节咱们总结下这个大神的代码风格。

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210201142615.png)



### 1.2 事务的定义

事务（Transaction）是数据库区别于文件系统的重要特性之一。目前国际认可的数据库设计原则是ACID特性，用以保证数据库事务的正确执行。Mysql的innodb引擎中的事务就完全符合ACID特性。

spring对于事务的支持，分层概览图如下：

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210201142618.png)

## 二、事务的ACID特性

（箭头后，翻译自官网介绍：[InnoDB and the ACID Model ](https://dev.mysql.com/doc/refman/5.6/en/mysql-acid.html)）

- **原子性（Atomicity）：\**一个事务必须被视为一个不可分割的最小工作单元，整个事务中的所有操作要么全部提交成功，要么全部失败回滚。\*\*--》\*\*\*\*主要涉及InnoDB事务。\*\*相关特性：事务的\*\*提交，回滚，信息表。\*\**\***
- **一致性（consistency）：\**\*\*数据库总是从一个一致性的状态转换到另一个一致性的状态\*\*。在事务开始前后，数据库的完整性约束没有被破坏。例如违反了唯一性，必须撤销事务，返回初始状态。\*\*\*\*--》\*\*\*\*\**\**主要\**涉及内部InnoDB处理，以保护数据不受崩溃，\**相关特性：\**双写缓冲、崩溃恢复。**
- **隔离性（isolation）：\**\*\*每个读写事务的对象对其他事务的操作对象能相互分离，即：事务\*\*提交前对其他事务是不可见的\*\*，通常内部加锁实现。\*\*\*\*--》\*\*\*\*\*\**\*主要涉及事务，尤其是事务隔离级别，相关特性：隔离级别、innodb锁的底层实现细节。**
- **持久性（durability）：\**\*\*\*\*一旦事务提交，则其所做的修改会永久保存到数据库\*\*。\*\**\**\*--》\*\*涉及到MySQL软件特性与特定硬件配置的相互影响，相关特性：4个配置项：双写缓冲开关、事务提交刷新log的级别、binlog同步频率、表文件；写缓存、操作系统对于`fsync()的支持、`备份策略等。\*\**\***

## 三、事务的属性

要保证事务的ACID特性，spring给事务定义了6个属性，对应于声明式事务注解（org.springframework.transaction.annotation.Transactional）@Transactional(key1=*,key2=*...)

- **事务名称**：用户可手动指定事务的名称，当多个事务的时候，可区分使用哪个事务。对应注解中的属性value、transactionManager
- **隔离级别**: 为了解决数据库容易出现的问题，分级加锁处理策略。 对应注解中的属性isolation
- **超时时间**: 定义一个事务执行过程多久算超时，以便超时后回滚。可以防止长期运行的事务占用资源.对应注解中的属性timeout
- **是否只读**：表示这个事务只读取数据但不更新数据, 这样可以帮助数据库引擎优化事务.对应注解中的属性readOnly
- **传播机制**: 对事务的传播特性进行定义，共有7种类型。对应注解中的属性propagation
- **回滚机制**：定义遇到异常时回滚策略。对应注解中的属性rollbackFor、noRollbackFor、rollbackForClassName、noRollbackForClassName

 其中隔离级别和传播机制比较复杂，咱们细细地品一品。



### 3.1 隔离级别

这一块比较复杂，我们从3个角度来看：3种错误现象、mysql的底层技术支持、分级处理策略。这一小节一定要好好看，已经开始涉及核心原理了。

####  **一、.现象（三种问题）**

**脏读****(Drity Read)**：事务A更新记录但未提交，事务B查询出A未提交记录。

**不可重复读****(Non-repeatable read):** 事务A读取一次，此时事务B对数据进行了更新或删除操作，事务A再次查询数据不一致。

**幻读****(Phantom Read)**: 事务A读取一次，此时事务B插入一条数据事务A再次查询，记录多了。

#### 二、 mysql的底层支持（IndoDB事务模型）（一致性非锁定读VS锁定读）

[官网飞机票：InnoDB Transaction Model](https://dev.mysql.com/doc/refman/5.6/en/innodb-transaction-model.html)

#### 两种读

在MVCC中，读操作可以分成两类，快照读（Snapshot read）和当前读（current read）。

**快照读**：普通的select

**当前读**：

1. select * from table where ? lock in share mode; （加S锁）
2. select * from table where ? for update; （加X锁）
3. insert, update, delete 操作前会先进行一次当前读（加X锁）

其中前两种锁定读,需要用户自己显式使用,最后一种是自动添加的。

#### 　　1.一致性非锁定读（快照读）

　　一致性非锁定读(consistent nonlocking read)是指InnoDB存储引擎通过多版本控制(multi versionning)的方式来读取当前执行时间数据库中行的数据，如果读取的行正在执行DELETE或UPDATE操作，这是读取操作不会因此等待行上锁的释放。相反的，InnoDB会去读取行的一个快照数据

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210201142608.jpg)

 

上面展示了InnoDB存储引擎一致性的非锁定读。之所以称为非锁定读，因为不需要等待访问的行上X锁的释放。快照数据是指该行之前版本的数据，该实现是通过undo段来完成。而undo用来事务中的回滚数据，因此快照数据本身没有额外的开销，此外，读取快照数据不需要上锁，因为没有事务需要对历史数据进行修改操作。

#### 　　2.锁定读（当前读）

innoDB对select语句支持两种锁定读：

1）SELECT...FOR UPDATE:对读取的行加排它锁（X锁），其他事务不能对已锁定的行再加任何锁。

2 ) SELECT...LOCK IN SHARE MODE ：对读取的行加共享锁（S锁），其他事务可以再加S锁，X锁会阻塞等待。

注：这两种锁都必须处于事务中，事务commit，锁释放。所以必须begin或者start transaction 开启一个事务或者索性set autocommit=0把自动提交关掉（mysql默认是1，即执行完sql立即提交）

#### **三、.分级处理策略（四种隔离级别）**

**官网描述：**

InnoDB使用不同的锁定策略支持每个事务隔离级别。对于关键数据的操作(遵从ACID原则)，您可以使用强一致性（默认Repeatable Read）。对于不是那么重要的数据操作，可以使用Read Committed/Read Uncommitted。Serializable执行比可重读更严格的规则，用于特殊场景：XA事务，并发性和死锁问题的故障排除。

 

**四种隔离级别：**

**1.Read Uncommitted****（读取未提交内容）**：可能读取其它事务未提交的数据。-脏读问题（脏读+不可重复读+幻读）

**2.Read Committed****（读取提交内容）**：一个事务只能看见已经提交事务所做的改变。（不可重复读+幻读）

select...from : 一致性非锁定读的数据快照(MVCC)是最新版本的,但其他事务可能会有新的commit，所以同一select可能返回不同结果。-不可重复读问题
select...from for update : record lock行级锁.

**3.Repeatable Read****（可重读）**：

- select…from ：同一事务内多次一致性非锁定读，取第一次读取时建立的快照版本(MVCC)，保证了同一事务内部的可重复读.—狭义的幻读问题得到解决。（Db插入了数据，只不过读不到）
- select...from for update （FOR UPDATE or LOCK IN SHARE MODE), UPDATE, 和 DELETE : next-key lock下一键锁.

　　1）对于具有唯一搜索条件的唯一索引，innoDB只锁定找到的索引记录.   （next-key lock 降为record lock）

　　2）对于其他非索引或者非唯一索引，InnoDB会对扫描的索引范围进行锁定，使用next-key locks，阻塞其他session对间隙的insert操作，-彻底解决广义的幻读问题。（DB没插入数据）

 

**4.Serializable****（可串行化）**：这是最高的隔离级别，它是在每个读的数据行上加上共享锁（LOCK IN SHARE MODE）。在这个级别，可能导致大量的超时现象和锁竞争，主要用于分布式事务。

如下表：

| 不同隔离级别/可能出现的问题                | 脏读 | 不可重复读 | 幻读 |
| ------------------------------------------ | ---- | ---------- | ---- |
| **Read Uncommitted****（读取未提交内容）** | ✅    | ✅          | ✅    |
| **Read Committed****（读取提交内容）**     | ❎    | ✅          | ✅    |
| **Repeatable Read****（可重读）**          | ❎    | ❎          | ✅    |
| **Serializable****（可串行化）**           | ❎    | ❎          | ❎    |

 

### 3.2 传播机制

org.springframework.transaction包下有一个事务定义接口TransactionDefinition，定义了7种事务传播机制，很多人对传播机制的曲解从概念开始，所以特地翻译了一下源码注释如下：

#### 1.PROPAGATION_REQUIRED

支持当前事务;如果不存在，创建一个新的。类似于同名的EJB事务属性。这通常是事务定义的默认设置，通常定义事务同步作用域。

#### 2.PROPAGATION_SUPPORTS

支持当前事务;如果不存在事务，则以非事务方式执行。类似于同名的EJB事务属性。
注意:
对于具有事务同步的事务管理器，PROPAGATION_SUPPORTS与没有事务稍有不同，因为它可能在事务范围内定义了同步。因此，相同的资源(JDBC的Connection、Hibernate的Session等)将在整个指定范围内共享。注意，确切的行为取决于事务管理器的实际同步配置!
小心使用PROPAGATION_SUPPORTS!特别是，不要依赖PROPAGATION_REQUIRED或PROPAGATION_REQUIRES_NEW,在PROPAGATION_SUPPORTS范围内(这可能导致运行时的同步冲突)。如果这种嵌套不可避免，请确保适当地配置事务管理器(通常切换到“实际事务上的同步”)。

#### 3.PROPAGATION_MANDATORY

支持当前事务;如果当前事务不存在，抛出异常。类似于同名的EJB事务属性。
注意：

PROPAGATION_MANDATORY范围内的事务同步总是由周围的事务驱动。

#### 4.PROPAGATION_REQUIRES_NEW

创建一个新事务，如果存在当前事务，则挂起当前事务。类似于同名的EJB事务属性。
注意:实际事务挂起不会在所有事务管理器上开箱即用。这一点特别适用于JtaTransactionManager，它需要TransactionManager的支持。
PROPAGATION_REQUIRES_NEW范围总是定义自己的事务同步。现有同步将被挂起并适当地恢复。

#### **5.**PROPAGATION_NOT_SUPPORTED

不支持当前事务，存在事务挂起当前事务;始终以非事务方式执行。类似于同名的EJB事务属性。
注意:实际事务挂起不会在所有事务管理器上开箱即用。这一点特别适用于JtaTransactionManager，它需要TransactionManager的支持。
事务同步在PROPAGATION_NOT_SUPPORTED范围内是不可用的。现有同步将被挂起并适当地恢复。

#### **6.**PROPAGATION_NEVER

不支持当前事务;如果当前事务存在，抛出异常。类似于同名的EJB事务属性。

注意：事务同步在PROPAGATION_NEVER范围内不可用。

#### **7.**PROPAGATION_NESTED

如果当前事务存在，则在嵌套事务中执行，如果当前没有事务，类似PROPAGATION_REQUIRED（创建一个新的）。EJB中没有类似的功能。
注意：实际创建嵌套事务只对特定的事务管理器有效。开箱即用，这只适用于 DataSourceTransactionManager（JDBC 3.0驱动）。一些JTA提供者也可能支持嵌套事务。



## 四、总结

本节讲解了事务的4大特性和6大属性的概念。并简单拓展了一下概念。可能大家会比较懵逼哈，不用担心只需要心里有个概念就可以了，下一章咱们从底层源码来看事务的实现机制。下面是隔离级别的表格，

**注意：JtaTransactionManager的类注释上说：Transaction suspension (REQUIRES_NEW, NOT_SUPPORTED) is just available with a JTA TransactionManager being registered." 这是片面的，只是说JTA TransactionManager支持挂起，并没有说DataSourceTransactionManager不支持。经过第四节实测，发现完全是支持的。网上很多说REQUIRES_NEW、NOT_SUPPORTED必须要JTA TransactionManager才行的完全是错误的说法。**

| 不同传播机制  | 事务名称 | 描述                         | 事务管理器要求               | 是否支持事务 | 是否开启新事务                      | 回滚规则                                                     |
| ------------- | -------- | ---------------------------- | ---------------------------- | ------------ | ----------------------------------- | ------------------------------------------------------------ |
| REQUIRED      | 要求     | 存在加入，不存在创建新       | 无                           | ✅            | 不一定                              | 存在一个事务：1.外部有事务加入，异常回滚；2.外部没事务创建新事务，异常回滚 |
| SUPPORTS      | 支持     | 存在加入，不存在非事务       | 无                           | ✅            | ❎                                   | 最多只存在一个事务： 1.外部有事务加入，异常回滚；2.外部没事务，内部非事务，异常不回滚 |
| MANDATORY     | 强制     | 存在加入，不存在抛异常       | 无                           | ✅            | ❎                                   | 最多只存在一个事务： 1.外部存在事务加入，异常回滚；2.外部不存在事务，异常无法回滚 |
| REQUIRES_NEW  | 要求新   | 存在挂起创建新，不存在创建新 | 无                           | ✅            | ✅                                   | 可能存在1-2个事务：1.外部存在事务挂起，创建新，异常回滚自己的事务 2.外部不存在事务，创建新， 异常只回滚新事务 |
| NOT_SUPPORTED | 不支持   | 存在挂起，不存在非事务       | 无                           | ❎            | ❎                                   | 最多只存在一个事务：1. 外部有事务挂起，外部异常回滚；内部非事务，异常不回滚2.外部无事务，内部非事务，异常不回滚 |
| NEVER         | 坚决不   | 存在抛异常                   | 无                           | ❎            | ❎                                   | 最多只存在一个事务：1.外部有事务，外部异常回滚；内部非事务不回滚 2.外部非事务，内部非事务，异常不回滚 |
| NESTED        | 嵌套     | 存在嵌套，不存在创建新       | DataSourceTransactionManager | ✅            | ❎（同一个物理事务，保存点实现嵌套） | 存在一个事务：1. 外部有事务，嵌套事务创建保存点，外部异常回滚全部事务；内部嵌套事务异常回滚到保存点；2.外部不存在事务，内部创建新事务，内部异常回滚 |

 

 一、引子

在Spring中，事务有两种实现方式：

1. **编程式事务管理：** 编程式事务管理使用底层源码可实现更细粒度的事务控制。spring推荐使用TransactionTemplate,典型的模板模式。
2. **申明式事务管理：** 添加@Transactional注解，并定义传播机制+回滚策略。基于Spring AOP实现，本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。



## 二、简单样例



### 需求：

创建用户时，新建一个用户余额表。如果用户余额创建失败抛出异常，那么用户表也回滚，即要保证“新增用户+新增用户余额”一起成功 或 回滚。



### 2.1 申明式事务管理

如下图，只需要在service.impl层，业务方法上添加@Transactional注解，定义事务的传播机制为REQUIRED（不写这个参数，默认就是REQUIRED），遇到Exception异常就一起回滚。

REQUIRED传播机制下：存在加入事务，不存在创建新事务。保证了当前方法中的所有数据库操作都在一个物理事务中，当遇到异常时会整个业务方法一起回滚。



```java
 1 /**
 2      * 创建用户并创建账户余额
 3      *
 4      * @param name
 5      * @param balance
 6      * @return
 7      */
 8     @Transactional(propagation= Propagation.REQUIRED, rollbackFor = Exception.class)
 9     @Override
10     public void addUserBalanceAndUser(String name, BigDecimal balance) {
11         log.info("[addUserBalanceAndUser] begin!!!");
12         //1.新增用户
13         userService.addUser(name);
14         //2.新增用户余额
15         UserBalance userBalance = new UserBalance();
16         userBalance.setName(name);
17         userBalance.setBalance(new BigDecimal(1000));
18         this.addUserBalance(userBalance);
19         log.info("[addUserBalanceAndUser] end!!!");
20     }
```



### 2.2 编程式事务管理

编程式事务管理，我们使用Spring推荐的transactionTemplate。我这里因为使用的是spring cloud的注解配置，实现用了自动配置类配置好了TransactionTemplate这个类型的bean.使用的时候直接注入bean使用即可（当然老式的xml配置也是一样的）。如下：

```java
 1 /**
 2      * 创建用户并创建账户余额(手动事务，不带结果)
 3      *
 4      * @param name
 5      * @param balance
 6      * @return
 7      */
 8     @Override
 9     public void addUserBalanceAndUserWithinTT(String name, BigDecimal balance) {
10         //实现一个没有返回值的事务回调
11         transactionTemplate.execute(new TransactionCallbackWithoutResult() {
12             @Override
13             protected void doInTransactionWithoutResult(TransactionStatus status) {
14                 try {
15                     log.info("[addUserBalanceAndUser] begin!!!");
16 
17                     //1.新增用户
18                     userService.addUser(name);
19                     //2.新增用户余额
20                     UserBalance userBalance = new UserBalance();
21                     userBalance.setName(name);
22                     userBalance.setBalance(new BigDecimal(1000));
23                     userBalanceRepository.insert(userBalance);
24                     log.info("[addUserBalanceAndUser] end!!!");
25                     //注意：这里catch住异常后，设置setRollbackOnly，否则事务不会滚。当然如果不需要自行处理异常，就不要catch了
26                 } catch (Exception e) {
27                     // 异常回滚
28                     status.setRollbackOnly();
29                     log.error("异常回滚!,e={}",e);
30                 }
31 
32             }
33         });
34     }
```

注意：

1.可以不用try catch，transactionTemplate.execute自己会捕捉异常并回滚。--》推荐

2.如果有业务异常需要特殊处理，记得：status.setRollbackOnly(); 标识为回滚。--》特殊情况才使用

## 三、总结

spring支持的这两种方式都可以，个人认为大部分情况下@Transactional可以满足需要。

