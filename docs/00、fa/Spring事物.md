# 介绍

## **一、背景**

### **1.1 拜神**

spring事务领头人叫Juergen Hoeller，于尔根·糊了...先混个脸熟哈，他写了几乎全部的spring事务代码。读源码先拜神，掌握他的源码的风格，读起来会通畅很多。最后一节咱们总结下这个大神的代码风格。

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210527103421.png)

### **1.2 事务的定义**

事务（Transaction）是数据库区别于文件系统的重要特性之一。目前国际认可的数据库设计原则是ACID特性，用以保证数据库事务的正确执行。Mysql的innodb引擎中的事务就完全符合ACID特性。

spring对于事务的支持，分层概览图如下：

<img src="https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210527103440.png" alt="img" style="zoom: 67%;" />

## **二、事务的ACID特性**

（箭头后，翻译自官网介绍：[InnoDB and the ACID Model ](https://dev.mysql.com/doc/refman/5.6/en/mysql-acid.html)）

- 原子性（Atomicity）：一个事务必须被视为一个不可分割的最小工作单元，整个事务中的所有操作要么全部提交成功，要么全部失败回滚。--》主要涉及InnoDB事务。相关特性：事务的提交，回滚，信息表。
- 一致性（consistency）：数据库总是从一个一致性的状态转换到另一个一致性的状态。在事务开始前后，数据库的完整性约束没有被破坏。例如违反了唯一性，必须撤销事务，返回初始状态。--》主要涉及内部InnoDB处理，以保护数据不受崩溃，相关特性：双写缓冲、崩溃恢复。
- 隔离性（isolation）：每个读写事务的对象对其他事务的操作对象能相互分离，即：事务提交前对其他事务是不可见的，通常内部加锁实现。--》主要涉及事务，尤其是事务隔离级别，相关特性：隔离级别、innodb锁的底层实现细节。
- 持久性（durability）：一旦事务提交，则其所做的修改会永久保存到数据库。--》涉及到MySQL软件特性与特定硬件配置的相互影响，相关特性：4个配置项：双写缓冲开关、事务提交刷新log的级别、binlog同步频率、表文件；写缓存、操作系统对于fsync()的支持、备份策略等。

## **三、事务的属性**

要保证事务的ACID特性，spring给事务定义了6个属性，对应于声明式事务注解（org.springframework.transaction.annotation.Transactional）@Transactional(key1=*,key2=*...)

- 事务名称：用户可手动指定事务的名称，当多个事务的时候，可区分使用哪个事务。对应注解中的属性value、transactionManager
- 隔离级别: 为了解决数据库容易出现的问题，分级加锁处理策略。 对应注解中的属性isolation
- 超时时间: 定义一个事务执行过程多久算超时，以便超时后回滚。可以防止长期运行的事务占用资源.对应注解中的属性timeout
- 是否只读：表示这个事务只读取数据但不更新数据, 这样可以帮助数据库引擎优化事务.对应注解中的属性readOnly
- 传播机制: 对事务的传播特性进行定义，共有7种类型。对应注解中的属性propagation
- 回滚机制：定义遇到异常时回滚策略。对应注解中的属性rollbackFor、noRollbackFor、rollbackForClassName、noRollbackForClassName

 其中隔离级别和传播机制比较复杂，咱们细细地品一品。

#### **3.1 隔离级别**

这一块比较复杂，我们从3个角度来看：3种错误现象、mysql的底层技术支持、分级处理策略。这一小节一定要好好看，已经开始涉及核心原理了。

 **一、.现象（三种问题）**

脏读(Drity Read)：事务A更新记录但未提交，事务B查询出A未提交记录。

不可重复读(Non-repeatable read): 事务A读取一次，此时事务B对数据进行了更新或删除操作，事务A再次查询数据不一致。

幻读(Phantom Read): 事务A读取一次，此时事务B插入一条数据事务A再次查询，记录多了。

**二、 mysql的底层支持（IndoDB事务模型）（一致性非锁定读VS锁定读）**

[官网飞机票：InnoDB Transaction Model](https://dev.mysql.com/doc/refman/5.6/en/innodb-transaction-model.html)

#### 3.2 两种读

在MVCC中，读操作可以分成两类，快照读（Snapshot read）和当前读（current read）。

快照读：普通的select

当前读：

```sql
select * from table where ? lock in share mode; （加S锁） 

select * from table where ? for update; （加X锁） 

insert, update, delete 操作前会先进行一次当前读（加X锁）
```

其中前两种锁定读,需要用户自己显式使用,最后一种是自动添加的。

**1.一致性非锁定读（快照读）**

　　一致性非锁定读(consistent nonlocking read)是指InnoDB存储引擎通过多版本控制(multi versionning)的方式来读取当前执行时间数据库中行的数据，如果读取的行正在执行DELETE或UPDATE操作，这是读取操作不会因此等待行上锁的释放。相反的，InnoDB会去读取行的一个快照数据

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210527103523.jpeg)

上面展示了InnoDB存储引擎一致性的非锁定读。之所以称为非锁定读，因为不需要等待访问的行上X锁的释放。快照数据是指该行之前版本的数据，该实现是通过undo段来完成。而undo用来事务中的回滚数据，因此快照数据本身没有额外的开销，此外，读取快照数据不需要上锁，因为没有事务需要对历史数据进行修改操作。

**2.锁定读（当前读）**

innoDB对select语句支持两种锁定读：

1）SELECT...FOR UPDATE:对读取的行加排它锁（X锁），其他事务不能对已锁定的行再加任何锁。

2 ) SELECT...LOCK IN SHARE MODE ：对读取的行加共享锁（S锁），其他事务可以再加S锁，X锁会阻塞等待。

注：这两种锁都必须处于事务中，事务commit，锁释放。所以必须begin或者start transaction 开启一个事务或者索性set autocommit=0把自动提交关掉（mysql默认是1，即执行完sql立即提交）

## **四、.分级处理策略（四种隔离级别）**

官网描述：

InnoDB使用不同的锁定策略支持每个事务隔离级别。对于关键数据的操作(遵从ACID原则)，您可以使用强一致性（默认Repeatable Read）。对于不是那么重要的数据操作，可以使用Read Committed/Read Uncommitted。Serializable执行比可重读更严格的规则，用于特殊场景：XA事务，并发性和死锁问题的故障排除。

#### **四种隔离级别**

1.Read Uncommitted（读取未提交内容）：可能读取其它事务未提交的数据。-脏读问题（脏读+不可重复读+幻读）

2.Read Committed（读取提交内容）：一个事务只能看见已经提交事务所做的改变。（不可重复读+幻读）

select...from : 一致性非锁定读的数据快照(MVCC)是最新版本的,但其他事务可能会有新的commit，所以同一select可能返回不同结果。-不可重复读问题

select...from for update : record lock行级锁.

3.Repeatable Read（可重读）：

- select…from ：同一事务内多次一致性非锁定读，取第一次读取时建立的快照版本(MVCC)，保证了同一事务内部的可重复读.—狭义的幻读问题得到解决。（Db插入了数据，只不过读不到）
- select...from for update （FOR UPDATE or LOCK IN SHARE MODE), UPDATE, 和 DELETE : next-key lock下一键锁.

　　1）对于具有唯一搜索条件的唯一索引，innoDB只锁定找到的索引记录.   （next-key lock 降为record lock）

　　2）对于其他非索引或者非唯一索引，InnoDB会对扫描的索引范围进行锁定，使用next-key locks，阻塞其他session对间隙的insert操作，-彻底解决广义的幻读问题。（DB没插入数据）

4.Serializable（可串行化）：这是最高的隔离级别，它是在每个读的数据行上加上共享锁（LOCK IN SHARE MODE）。在这个级别，可能导致大量的超时现象和锁竞争，主要用于分布式事务。

如下表：

| 不同隔离级别/可能出现的问题        | 脏读 | 不可重复读 | 幻读 |
| ---------------------------------- | ---- | ---------- | ---- |
| Read Uncommitted（读取未提交内容） | ✅    | ✅          | ✅    |
| Read Committed（读取提交内容）     | ❎    | ✅          | ✅    |
| Repeatable Read（可重读）          | ❎    | ❎          | ✅    |
| Serializable（可串行化）           | ❎    | ❎          | ❎    |

#### **传播机制**

org.springframework.transaction包下有一个事务定义接口TransactionDefinition，定义了7种事务传播机制，很多人对传播机制的曲解从概念开始，所以特地翻译了一下源码注释如下：

**1.PROPAGATION_REQUIRED**

支持当前事务;如果不存在，创建一个新的。类似于同名的EJB事务属性。这通常是事务定义的默认设置，通常定义事务同步作用域。

**2.PROPAGATION_SUPPORTS**

支持当前事务;如果不存在事务，则以非事务方式执行。类似于同名的EJB事务属性。

注意:

对于具有事务同步的事务管理器，PROPAGATION_SUPPORTS与没有事务稍有不同，因为它可能在事务范围内定义了同步。因此，相同的资源(JDBC的Connection、Hibernate的Session等)将在整个指定范围内共享。注意，确切的行为取决于事务管理器的实际同步配置!

小心使用PROPAGATION_SUPPORTS!特别是，不要依赖PROPAGATION_REQUIRED或PROPAGATION_REQUIRES_NEW,在PROPAGATION_SUPPORTS范围内(这可能导致运行时的同步冲突)。如果这种嵌套不可避免，请确保适当地配置事务管理器(通常切换到“实际事务上的同步”)。

**3.PROPAGATION_MANDATORY**

支持当前事务;如果当前事务不存在，抛出异常。类似于同名的EJB事务属性。

注意：

PROPAGATION_MANDATORY范围内的事务同步总是由周围的事务驱动。

**4.PROPAGATION_REQUIRES_NEW**

创建一个新事务，如果存在当前事务，则挂起当前事务。类似于同名的EJB事务属性。

注意:实际事务挂起不会在所有事务管理器上开箱即用。这一点特别适用于JtaTransactionManager，它需要TransactionManager的支持。

PROPAGATION_REQUIRES_NEW范围总是定义自己的事务同步。现有同步将被挂起并适当地恢复。

**5.PROPAGATION_NOT_SUPPORTED**

不支持当前事务，存在事务挂起当前事务;始终以非事务方式执行。类似于同名的EJB事务属性。

注意:实际事务挂起不会在所有事务管理器上开箱即用。这一点特别适用于JtaTransactionManager，它需要TransactionManager的支持。

事务同步在PROPAGATION_NOT_SUPPORTED范围内是不可用的。现有同步将被挂起并适当地恢复。

**6.PROPAGATION_NEVER**

不支持当前事务;如果当前事务存在，抛出异常。类似于同名的EJB事务属性。

注意：事务同步在PROPAGATION_NEVER范围内不可用。

**7.PROPAGATION_NESTED**

如果当前事务存在，则在嵌套事务中执行，如果当前没有事务，类似PROPAGATION_REQUIRED（创建一个新的）。EJB中没有类似的功能。

注意：实际创建嵌套事务只对特定的事务管理器有效。开箱即用，这只适用于 DataSourceTransactionManager（JDBC 3.0驱动）。一些JTA提供者也可能支持嵌套事务。

**四、总结**

本节讲解了事务的4大特性和6大属性的概念。并简单拓展了一下概念。可能大家会比较懵逼哈，不用担心只需要心里有个概念就可以了，下一章咱们从底层源码来看事务的实现机制。下面是隔离级别的表格，

注意：JtaTransactionManager的类注释上说：Transaction suspension (REQUIRES_NEW, NOT_SUPPORTED) is just available with a JTA TransactionManager being registered." 这是片面的，只是说JTA TransactionManager支持挂起，并没有说DataSourceTransactionManager不支持。经过第四节实测，发现完全是支持的。网上很多说REQUIRES_NEW、NOT_SUPPORTED必须要JTA TransactionManager才行的完全是错误的说法。

| 不同传播机制  | 事务名称 | 描述                         | 事务管理器要求               | 是否支持事务 | 是否开启新事务                      | 回滚规则                                                     |
| ------------- | -------- | ---------------------------- | ---------------------------- | ------------ | ----------------------------------- | ------------------------------------------------------------ |
| REQUIRED      | 要求     | 存在加入，不存在创建新       | 无                           | ✅            | 不一定                              | 存在一个事务：1.外部有事务加入，异常回滚；2.外部没事务创建新事务，异常回滚 |
| SUPPORTS      | 支持     | 存在加入，不存在非事务       | 无                           | ✅            | ❎                                   | 最多只存在一个事务： 1.外部有事务加入，异常回滚；2.外部没事务，内部非事务，异常不回滚 |
| MANDATORY     | 强制     | 存在加入，不存在抛异常       | 无                           | ✅            | ❎                                   | 最多只存在一个事务： 1.外部存在事务加入，异常回滚；2.外部不存在事务，异常无法回滚 |
| REQUIRES_NEW  | 要求新   | 存在挂起创建新，不存在创建新 | 无                           | ✅            | ✅                                   | 可能存在1-2个事务：1.外部存在事务挂起，创建新，异常回滚自己的事务 2.外部不存在事务，创建新， 异常只回滚新事务 |
| NOT_SUPPORTED | 不支持   | 存在挂起，不存在非事务       | 无                           | ❎            | ❎                                   | 最多只存在一个事务：1. 外部有事务挂起，外部异常回滚；内部非事务，异常不回滚2.外部无事务，内部非事务，异常不回滚 |
| NEVER         | 坚决不   | 存在抛异常                   | 无                           | ❎            | ❎                                   | 最多只存在一个事务：1.外部有事务，外部异常回滚；内部非事务不回滚 2.外部非事务，内部非事务，异常不回滚 |
| NESTED        | 嵌套     | 存在嵌套，不存在创建新       | DataSourceTransactionManager | ✅            | ❎（同一个物理事务，保存点实现嵌套） | 存在一个事务：1. 外部有事务，嵌套事务创建保存点，外部异常回滚全部事务；内部嵌套事务异常回滚到保存点；2.外部不存在事务，内部创建新事务，内部异常回滚 |

# 事例

## 一、引子

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

# 源码详解

## 一、引子

在Spring中，事务有两种实现方式：

1. **编程式事务管理：** 编程式事务管理使用TransactionTemplate可实现更细粒度的事务控制。
2. **申明式事务管理：** 基于Spring AOP实现。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。

申明式事务管理不需要入侵代码，通过@Transactional就可以进行事务操作，更快捷而且简单（尤其是配合spring boot自动配置，可以说是精简至极！），且大部分业务都可以满足，推荐使用。

其实不管是编程式事务还是申明式事务，最终调用的底层核心代码是一致的。本章分别从编程式、申明式入手，再进入核心源码贯穿式讲解。

## 二、事务源码



### 2.1 编程式事务TransactionTemplate

编程式事务，Spring已经给我们提供好了模板类TransactionTemplate，可以很方便的使用，如下图：

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210527112210.png)

TransactionTemplate全路径名是：org.springframework.transaction.support.TransactionTemplate。看包名也知道了这是spring对事务的模板类。（spring动不动就是各种Template...），看下类图先：

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210527112219.png)

一看，哟西，实现了TransactionOperations、InitializingBean这2个接口（熟悉spring源码的知道这个InitializingBean又是老套路），我们来看下接口源码如下：



```java
 1 public interface TransactionOperations {
 2 
 3     /**
 4      * Execute the action specified by the given callback object within a transaction.
 5      * <p>Allows for returning a result object created within the transaction, that is,
 6      * a domain object or a collection of domain objects. A RuntimeException thrown
 7      * by the callback is treated as a fatal exception that enforces a rollback.
 8      * Such an exception gets propagated to the caller of the template.
 9      * @param action the callback object that specifies the transactional action
10      * @return a result object returned by the callback, or {@code null} if none
11      * @throws TransactionException in case of initialization, rollback, or system errors
12      * @throws RuntimeException if thrown by the TransactionCallback
13      */
14     <T> T execute(TransactionCallback<T> action) throws TransactionException;
15 
16 }
17 
18 public interface InitializingBean {
19 
20     /**
21      * Invoked by a BeanFactory after it has set all bean properties supplied
22      * (and satisfied BeanFactoryAware and ApplicationContextAware).
23      * <p>This method allows the bean instance to perform initialization only
24      * possible when all bean properties have been set and to throw an
25      * exception in the event of misconfiguration.
26      * @throws Exception in the event of misconfiguration (such
27      * as failure to set an essential property) or if initialization fails.
28      */
29     void afterPropertiesSet() throws Exception;
30 
31 }
```



如上图，TransactionOperations这个接口用来执行事务的回调方法，InitializingBean这个是典型的spring bean初始化流程中（飞机票：[Spring IOC（四）总结升华篇](https://www.cnblogs.com/dennyzhangdd/p/7730050.html)）的预留接口，专用用来在bean属性加载完毕时执行的方法。

回到正题，TransactionTemplate的2个接口的impl方法做了什么？

```java
 1     @Override
 2     public void afterPropertiesSet() {
 3         if (this.transactionManager == null) {
 4             throw new IllegalArgumentException("Property 'transactionManager' is required");
 5         }
 6     }
 7 
 8 
 9     @Override
10     public <T> T execute(TransactionCallback<T> action) throws TransactionException {　　　　　　　// 内部封装好的事务管理器
11         if (this.transactionManager instanceof CallbackPreferringPlatformTransactionManager) {
12             return ((CallbackPreferringPlatformTransactionManager) this.transactionManager).execute(this, action);
13         }// 需要手动获取事务，执行方法，提交事务的管理器
14         else {// 1.获取事务状态
15             TransactionStatus status = this.transactionManager.getTransaction(this);
16             T result;
17             try {// 2.执行业务逻辑
18                 result = action.doInTransaction(status);
19             }
20             catch (RuntimeException ex) {
21                 // 应用运行时异常 -> 回滚
22                 rollbackOnException(status, ex);
23                 throw ex;
24             }
25             catch (Error err) {
26                 // Error异常 -> 回滚
27                 rollbackOnException(status, err);
28                 throw err;
29             }
30             catch (Throwable ex) {
31                 // 未知异常 -> 回滚
32                 rollbackOnException(status, ex);
33                 throw new UndeclaredThrowableException(ex, "TransactionCallback threw undeclared checked exception");
34             }// 3.事务提交
35             this.transactionManager.commit(status);
36             return result;
37         }
38     }
```



如上图所示，实际上afterPropertiesSet只是校验了事务管理器不为空，execute()才是核心方法，execute主要步骤：

1.getTransaction()获取事务，源码见3.3.1

2.doInTransaction()执行业务逻辑，这里就是用户自定义的业务代码。如果是没有返回值的，就是doInTransactionWithoutResult()。

3.commit()事务提交：调用AbstractPlatformTransactionManager的commit，rollbackOnException()异常回滚：调用AbstractPlatformTransactionManager的rollback()，事务提交回滚，源码见3.3.3

### 2.2 申明式事务@Transactional

#### 1.AOP相关概念

申明式事务使用的是spring AOP，即面向切面编程。（什么❓你不知道什么是AOP...一句话概括就是：把业务代码中重复代码做成一个切面，提取出来，并定义哪些方法需要执行这个切面。其它的自行百度吧...）AOP核心概念如下：

- 通知（Advice）:定义了切面(各处业务代码中都需要的逻辑提炼成的一个切面)做什么what+when何时使用。例如：前置通知Before、后置通知After、返回通知After-returning、异常通知After-throwing、环绕通知Around.
- 连接点（Joint point）：程序执行过程中能够插入切面的点，一般有多个。比如调用方式时、抛出异常时。
- 切点（Pointcut）:切点定义了连接点，切点包含多个连接点,即where哪里使用通知.通常指定类+方法 或者 正则表达式来匹配 类和方法名称。
- 切面（Aspect）:切面=通知+切点，即when+where+what何时何地做什么。
- 引入（Introduction）:允许我们向现有的类添加新方法或属性。
- 织入（Weaving）:织入是把切面应用到目标对象并创建新的代理对象的过程。

#### 2.申明式事务

申明式事务整体调用过程，可以抽出2条线：

1.使用代理模式，生成代理增强类。

2.根据代理事务管理配置类，配置事务的织入，在业务方法前后进行环绕增强，增加一些事务的相关操作。例如获取事务属性、提交事务、回滚事务。

过程如下图：

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210527112248.png)

申明式事务使用@Transactional这种注解的方式，那么我们就从springboot 容器启动时的自动配置载入（[spring boot容器启动详解](https://www.cnblogs.com/dennyzhangdd/p/8028950.html)）开始看。在/META-INF/spring.factories中配置文件中查找，如下图：

 ![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210527112252.png)

载入2个关于事务的自动配置类： 

org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,
org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,

jta咱们就不看了，看一下TransactionAutoConfiguration这个自动配置类：



```java
 1 @Configuration
 2 @ConditionalOnClass(PlatformTransactionManager.class)
 3 @AutoConfigureAfter({ JtaAutoConfiguration.class, HibernateJpaAutoConfiguration.class,
 4         DataSourceTransactionManagerAutoConfiguration.class,
 5         Neo4jDataAutoConfiguration.class })
 6 @EnableConfigurationProperties(TransactionProperties.class)
 7 public class TransactionAutoConfiguration {
 8 
 9     @Bean
10     @ConditionalOnMissingBean
11     public TransactionManagerCustomizers platformTransactionManagerCustomizers(
12             ObjectProvider<List<PlatformTransactionManagerCustomizer<?>>> customizers) {
13         return new TransactionManagerCustomizers(customizers.getIfAvailable());
14     }
15 
16     @Configuration
17     @ConditionalOnSingleCandidate(PlatformTransactionManager.class)
18     public static class TransactionTemplateConfiguration {
19 
20         private final PlatformTransactionManager transactionManager;
21 
22         public TransactionTemplateConfiguration(
23                 PlatformTransactionManager transactionManager) {
24             this.transactionManager = transactionManager;
25         }
26 
27         @Bean
28         @ConditionalOnMissingBean
29         public TransactionTemplate transactionTemplate() {
30             return new TransactionTemplate(this.transactionManager);
31         }
32     }
33 
34     @Configuration
35     @ConditionalOnBean(PlatformTransactionManager.class)
36     @ConditionalOnMissingBean(AbstractTransactionManagementConfiguration.class)
37     public static class EnableTransactionManagementConfiguration {
38 
39         @Configuration
40         @EnableTransactionManagement(proxyTargetClass = false)
41         @ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "false", matchIfMissing = false)
42         public static class JdkDynamicAutoProxyConfiguration {
43 
44         }
45 
46         @Configuration
47         @EnableTransactionManagement(proxyTargetClass = true)
48         @ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "true", matchIfMissing = true)
49         public static class CglibAutoProxyConfiguration {
50 
51         }
52 
53     }
54 
55 }
```



TransactionAutoConfiguration这个类主要看：

1.2个类注解

@ConditionalOnClass(PlatformTransactionManager.class)即类路径下包含PlatformTransactionManager这个类时这个自动配置生效，这个类是spring事务的核心包，肯定引入了。

@AutoConfigureAfter({ JtaAutoConfiguration.class, HibernateJpaAutoConfiguration.class, DataSourceTransactionManagerAutoConfiguration.class, Neo4jDataAutoConfiguration.class })，这个配置在括号中的4个配置类后才生效。

\2. 2个内部类

TransactionTemplateConfiguration事务模板配置类：

@ConditionalOnSingleCandidate(PlatformTransactionManager.class)当能够唯一确定一个PlatformTransactionManager bean时才生效。

@ConditionalOnMissingBean如果没有定义TransactionTemplate bean生成一个。

EnableTransactionManagementConfiguration开启事务管理器配置类：

@ConditionalOnBean(PlatformTransactionManager.class)当存在PlatformTransactionManager bean时生效。

@ConditionalOnMissingBean(AbstractTransactionManagementConfiguration.class)当没有自定义抽象事务管理器配置类时才生效。（即用户自定义抽象事务管理器配置类会优先，如果没有，就用这个默认事务管理器配置类）

EnableTransactionManagementConfiguration支持2种代理方式：

- 1.JdkDynamicAutoProxyConfiguration：

@EnableTransactionManagement(proxyTargetClass = false)，即proxyTargetClass = false表示是JDK动态代理支持的是：面向接口代理。

@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "false", matchIfMissing = false)，即spring.aop.proxy-target-class=false时生效，且没有这个配置不生效。



- 2.CglibAutoProxyConfiguration：

@EnableTransactionManagement(proxyTargetClass = true)，即proxyTargetClass = true标识Cglib代理支持的是子类继承代理。
@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "true", matchIfMissing = true)，即spring.aop.proxy-target-class=true时生效，且没有这个配置默认生效。

**注意了，****默认没有配置，走的Cglib代理。说明**@Transactional注解支持直接加在类上。

好吧，看了这么多配置类，终于到了@EnableTransactionManagement这个注解了。



```java
 1 @Target(ElementType.TYPE)
 2 @Retention(RetentionPolicy.RUNTIME)
 3 @Documented
 4 @Import(TransactionManagementConfigurationSelector.class)
 5 public @interface EnableTransactionManagement {
 6 
 7     //proxyTargetClass = false表示是JDK动态代理支持接口代理。true表示是Cglib代理支持子类继承代理。
 8     boolean proxyTargetClass() default false;
 9 
10     //事务通知模式(切面织入方式)，默认代理模式（同一个类中方法互相调用拦截器不会生效），可以选择增强型AspectJ
11     AdviceMode mode() default AdviceMode.PROXY;
12 
13     //连接点上有多个通知时，排序，默认最低。值越大优先级越低。
14     int order() default Ordered.LOWEST_PRECEDENCE;
15 
16 }
```



重点看类注解@Import(TransactionManagementConfigurationSelector.class)

TransactionManagementConfigurationSelector类图如下：

# ![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210527112316.png)

如上图所示，TransactionManagementConfigurationSelector继承自AdviceModeImportSelector实现了ImportSelector接口。



```java
 1 public class TransactionManagementConfigurationSelector extends AdviceModeImportSelector<EnableTransactionManagement> {
 2 
 3     /**
 4      * {@inheritDoc}
 5      * @return {@link ProxyTransactionManagementConfiguration} or
 6      * {@code AspectJTransactionManagementConfiguration} for {@code PROXY} and
 7      * {@code ASPECTJ} values of {@link EnableTransactionManagement#mode()}, respectively
 8      */
 9     @Override
10     protected String[] selectImports(AdviceMode adviceMode) {
11         switch (adviceMode) {
12             case PROXY:
13                 return new String[] {AutoProxyRegistrar.class.getName(), ProxyTransactionManagementConfiguration.class.getName()};
14             case ASPECTJ:
15                 return new String[] {TransactionManagementConfigUtils.TRANSACTION_ASPECT_CONFIGURATION_CLASS_NAME};
16             default:
17                 return null;
18         }
19     }
20 
21 }
```



如上图，最终会执行selectImports方法导入需要加载的类，我们只看proxy模式下，载入了AutoProxyRegistrar、ProxyTransactionManagementConfiguration2个类。

- AutoProxyRegistrar：`给容器中注册一个 InfrastructureAdvisorAutoProxyCreator 组件；利用后置处理器机制在对象创建以后，包装对象，返回一个代理对象（增强器），代理对象执行方法利用拦截器链进行调用；`
- ProxyTransactionManagementConfiguration：就是一个配置类，定义了事务增强器。

#### AutoProxyRegistrar

先看AutoProxyRegistrar实现了ImportBeanDefinitionRegistrar接口，复写registerBeanDefinitions方法，源码如下：



```java
 1 public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
 2         boolean candidateFound = false;
 3         Set<String> annoTypes = importingClassMetadata.getAnnotationTypes();
 4         for (String annoType : annoTypes) {
 5             AnnotationAttributes candidate = AnnotationConfigUtils.attributesFor(importingClassMetadata, annoType);
 6             if (candidate == null) {
 7                 continue;
 8             }
 9             Object mode = candidate.get("mode");
10             Object proxyTargetClass = candidate.get("proxyTargetClass");
11             if (mode != null && proxyTargetClass != null && AdviceMode.class == mode.getClass() &&
12                     Boolean.class == proxyTargetClass.getClass()) {
13                 candidateFound = true;
14                 if (mode == AdviceMode.PROXY) {//代理模式
15                     AopConfigUtils.registerAutoProxyCreatorIfNecessary(registry);
16                     if ((Boolean) proxyTargetClass) {//如果是CGLOB子类代理模式
17                         AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
18                         return;
19                     }
20                 }
21             }
22         }
23         if (!candidateFound) {
24             String name = getClass().getSimpleName();
25             logger.warn(String.format("%s was imported but no annotations were found " +
26                     "having both 'mode' and 'proxyTargetClass' attributes of type " +
27                     "AdviceMode and boolean respectively. This means that auto proxy " +
28                     "creator registration and configuration may not have occurred as " +
29                     "intended, and components may not be proxied as expected. Check to " +
30                     "ensure that %s has been @Import'ed on the same class where these " +
31                     "annotations are declared; otherwise remove the import of %s " +
32                     "altogether.", name, name, name));
33         }
34     }
```



代理模式：AopConfigUtils.registerAutoProxyCreatorIfNecessary(registry);

最终调用的是：registerOrEscalateApcAsRequired(InfrastructureAdvisorAutoProxyCreator.class, registry, source);基础构建增强自动代理构造器



```java
 1 private static BeanDefinition registerOrEscalateApcAsRequired(Class<?> cls, BeanDefinitionRegistry registry, Object source) {
 2         Assert.notNull(registry, "BeanDefinitionRegistry must not be null");　　　　　　 //如果当前注册器包含internalAutoProxyCreator
 3         if (registry.containsBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME)) {//org.springframework.aop.config.internalAutoProxyCreator内部自动代理构造器
 4             BeanDefinition apcDefinition = registry.getBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME);
 5             if (!cls.getName().equals(apcDefinition.getBeanClassName())) {//如果当前类不是internalAutoProxyCreator
 6                 int currentPriority = findPriorityForClass(apcDefinition.getBeanClassName());
 7                 int requiredPriority = findPriorityForClass(cls);
 8                 if (currentPriority < requiredPriority) {//如果下标大于已存在的内部自动代理构造器，index越小，优先级越高,InfrastructureAdvisorAutoProxyCreator index=0,requiredPriority最小，不进入
 9                     apcDefinition.setBeanClassName(cls.getName());
10                 }
11             }
12             return null;//直接返回
13         }//如果当前注册器不包含internalAutoProxyCreator，则把当前类作为根定义
14         RootBeanDefinition beanDefinition = new RootBeanDefinition(cls);
15         beanDefinition.setSource(source);
16         beanDefinition.getPropertyValues().add("order", Ordered.HIGHEST_PRECEDENCE);//优先级最高
17         beanDefinition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
18         registry.registerBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME, beanDefinition);
19         return beanDefinition;
20     }
```



如上图，APC_PRIORITY_LIST列表如下图：



```java
 1 /**
 2      * Stores the auto proxy creator classes in escalation order.
 3      */
 4     private static final List<Class<?>> APC_PRIORITY_LIST = new ArrayList<Class<?>>();
 5 
 6     /**
 7      * 优先级上升list
 8      */
 9     static {
10         APC_PRIORITY_LIST.add(InfrastructureAdvisorAutoProxyCreator.class);
11         APC_PRIORITY_LIST.add(AspectJAwareAdvisorAutoProxyCreator.class);
12         APC_PRIORITY_LIST.add(AnnotationAwareAspectJAutoProxyCreator.class);
13     }
```



如上图，由于InfrastructureAdvisorAutoProxyCreator这个类在list中第一个index=0,requiredPriority最小，不进入，所以没有重置beanClassName，啥都没做，返回null.

#### 那么增强代理类何时生成呢？

InfrastructureAdvisorAutoProxyCreator类图如下：

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210527112337.png)

如上图所示，看2个核心方法：InstantiationAwareBeanPostProcessor接口的postProcessBeforeInstantiation实例化前+BeanPostProcessor接口的postProcessAfterInitialization初始化后。关于spring bean生命周期飞机票：[Spring IOC（四）总结升华篇](https://www.cnblogs.com/dennyzhangdd/p/7730050.html)



```
 1     @Override
 2     public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
 3         Object cacheKey = getCacheKey(beanClass, beanName);
 4 
 5         if (beanName == null || !this.targetSourcedBeans.contains(beanName)) {
 6             if (this.advisedBeans.containsKey(cacheKey)) {//如果已经存在直接返回
 7                 return null;
 8             }//是否基础构件（基础构建不需要代理）：Advice、Pointcut、Advisor、AopInfrastructureBean这四类都算基础构建
 9             if (isInfrastructureClass(beanClass) || shouldSkip(beanClass, beanName)) {
10                 this.advisedBeans.put(cacheKey, Boolean.FALSE);//添加进advisedBeans ConcurrentHashMap<k=Object,v=Boolean>标记是否需要增强实现，这里基础构建bean不需要代理，都置为false，供后面postProcessAfterInitialization实例化后使用。
11                 return null;
12             }
13         }
14 
15         // TargetSource是spring aop预留给我们用户自定义实例化的接口，如果存在TargetSource就不会默认实例化，而是按照用户自定义的方式实例化，咱们没有定义，不进入
18         if (beanName != null) {
19             TargetSource targetSource = getCustomTargetSource(beanClass, beanName);
20             if (targetSource != null) {
21                 this.targetSourcedBeans.add(beanName);
22                 Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(beanClass, beanName, targetSource);
23                 Object proxy = createProxy(beanClass, beanName, specificInterceptors, targetSource);
24                 this.proxyTypes.put(cacheKey, proxy.getClass());
25                 return proxy;
26             }
27         }
28 
29         return null;
30     }
```



通过追踪，由于InfrastructureAdvisorAutoProxyCreator是基础构建类，

advisedBeans.put(cacheKey, Boolean.FALSE)

添加进advisedBeans ConcurrentHashMap<k=Object,v=Boolean>标记是否需要增强实现，这里基础构建bean不需要代理，都置为false，供后面postProcessAfterInitialization实例化后使用。

我们再看postProcessAfterInitialization源码如下：



```
 1     @Override
 2     public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
 3         if (bean != null) {
 4             Object cacheKey = getCacheKey(bean.getClass(), beanName);
 5             if (!this.earlyProxyReferences.contains(cacheKey)) {
 6                 return wrapIfNecessary(bean, beanName, cacheKey);
 7             }
 8         }
 9         return bean;
10     }
11 
12     protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {　　　　　　 // 如果是用户自定义获取实例，不需要增强处理，直接返回
13         if (beanName != null && this.targetSourcedBeans.contains(beanName)) {
14             return bean;
15         }// 查询map缓存，标记过false,不需要增强直接返回
16         if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
17             return bean;
18         }// 判断一遍springAOP基础构建类，标记过false,不需要增强直接返回
19         if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
20             this.advisedBeans.put(cacheKey, Boolean.FALSE);
21             return bean;
22         }
23 
24         // 获取增强List<Advisor> advisors
25         Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);　　　　　　 // 如果存在增强
26         if (specificInterceptors != DO_NOT_PROXY) {
27             this.advisedBeans.put(cacheKey, Boolean.TRUE);// 标记增强为TRUE,表示需要增强实现　　　　　　　　  // 生成增强代理类
28             Object proxy = createProxy(
29                     bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
30             this.proxyTypes.put(cacheKey, proxy.getClass());
31             return proxy;
32         }
33 　　　　 // 如果不存在增强，标记false,作为缓存，再次进入提高效率，第16行利用缓存先校验
34         this.advisedBeans.put(cacheKey, Boolean.FALSE);
35         return bean;
36     }
```



下面看核心方法createProxy如下：



```
 1     protected Object createProxy(
 2             Class<?> beanClass, String beanName, Object[] specificInterceptors, TargetSource targetSource) {
 3 　　　　 // 如果是ConfigurableListableBeanFactory接口（咱们DefaultListableBeanFactory就是该接口的实现类）则，暴露目标类
 4         if (this.beanFactory instanceof ConfigurableListableBeanFactory) {　　　　　　　　  //给beanFactory->beanDefinition定义一个属性：k=AutoProxyUtils.originalTargetClass,v=需要被代理的bean class
 5             AutoProxyUtils.exposeTargetClass((ConfigurableListableBeanFactory) this.beanFactory, beanName, beanClass);
 6         }
 7 
 8         ProxyFactory proxyFactory = new ProxyFactory();
 9         proxyFactory.copyFrom(this);
10 　　　　 //如果不是代理目标类
11         if (!proxyFactory.isProxyTargetClass()) {//如果beanFactory定义了代理目标类（CGLIB）
12             if (shouldProxyTargetClass(beanClass, beanName)) {
13                 proxyFactory.setProxyTargetClass(true);//代理工厂设置代理目标类
14             }
15             else {//否则设置代理接口（JDK）
16                 evaluateProxyInterfaces(beanClass, proxyFactory);
17             }
18         }
19 　　　　 //把拦截器包装成增强（通知）
20         Advisor[] advisors = buildAdvisors(beanName, specificInterceptors);
21         proxyFactory.addAdvisors(advisors);//设置进代理工厂
22         proxyFactory.setTargetSource(targetSource);
23         customizeProxyFactory(proxyFactory);//空方法，留给子类拓展用，典型的spring的风格，喜欢处处留后路
24 　　　　 //用于控制代理工厂是否还允许再次添加通知，默认为false（表示不允许）
25         proxyFactory.setFrozen(this.freezeProxy);
26         if (advisorsPreFiltered()) {//默认false，上面已经前置过滤了匹配的增强Advisor
27             proxyFactory.setPreFiltered(true);
28         }
29         //代理工厂获取代理对象的核心方法
30         return proxyFactory.getProxy(getProxyClassLoader());
31     }
```



最终我们生成的是CGLIB代理类.到此为止我们分析完了代理类的构造过程。

#### ProxyTransactionManagementConfiguration

下面来看ProxyTransactionManagementConfiguration：



```
 1 @Configuration
 2 public class ProxyTransactionManagementConfiguration extends AbstractTransactionManagementConfiguration {
 3 
 4     @Bean(name = TransactionManagementConfigUtils.TRANSACTION_ADVISOR_BEAN_NAME)
 5     @Role(BeanDefinition.ROLE_INFRASTRUCTURE)//定义事务增强器
 6     public BeanFactoryTransactionAttributeSourceAdvisor transactionAdvisor() {
 7         BeanFactoryTransactionAttributeSourceAdvisor j = new BeanFactoryTransactionAttributeSourceAdvisor();
 8         advisor.setTransactionAttributeSource(transactionAttributeSource());
 9         advisor.setAdvice(transactionInterceptor());
10         advisor.setOrder(this.enableTx.<Integer>getNumber("order"));
11         return advisor;
12     }
13 
14     @Bean
15     @Role(BeanDefinition.ROLE_INFRASTRUCTURE)//定义基于注解的事务属性资源
16     public TransactionAttributeSource transactionAttributeSource() {
17         return new AnnotationTransactionAttributeSource();
18     }
19 
20     @Bean
21     @Role(BeanDefinition.ROLE_INFRASTRUCTURE)//定义事务拦截器
22     public TransactionInterceptor transactionInterceptor() {
23         TransactionInterceptor interceptor = new TransactionInterceptor();
24         interceptor.setTransactionAttributeSource(transactionAttributeSource());
25         if (this.txManager != null) {
26             interceptor.setTransactionManager(this.txManager);
27         }
28         return interceptor;
29     }
30 
31 }
```



核心方法：transactionAdvisor()事务织入

定义了一个advisor，设置事务属性、设置事务拦截器TransactionInterceptor、设置顺序。核心就是事务拦截器TransactionInterceptor。

TransactionInterceptor使用通用的spring事务基础架构实现“声明式事务”，继承自TransactionAspectSupport类（该类包含与Spring的底层事务API的集成），实现了MethodInterceptor接口。spring类图如下：

![img](https://img2018.cnblogs.com/blog/584866/201809/584866-20180914144614118-1426254578.png)

事务拦截器的拦截功能就是依靠实现了MethodInterceptor接口，熟悉spring的同学肯定很熟悉MethodInterceptor了，这个是spring的方法拦截器，主要看invoke方法：



```
 1 @Override
 2     public Object invoke(final MethodInvocation invocation) throws Throwable {
 3         // Work out the target class: may be {@code null}.
 4         // The TransactionAttributeSource should be passed the target class
 5         // as well as the method, which may be from an interface.
 6         Class<?> targetClass = (invocation.getThis() != null ? AopUtils.getTargetClass(invocation.getThis()) : null);
 7 
 8         // 调用TransactionAspectSupport的 invokeWithinTransaction方法
 9         return invokeWithinTransaction(invocation.getMethod(), targetClass, new InvocationCallback() {
10             @Override
11             public Object proceedWithInvocation() throws Throwable {
12                 return invocation.proceed();
13             }
14         });
15     }
```



如上图TransactionInterceptor复写MethodInterceptor接口的invoke方法，并在invoke方法中调用了父类TransactionAspectSupport的invokeWithinTransaction()方法，源码如下：



```
 1 protected Object invokeWithinTransaction(Method method, Class<?> targetClass, final InvocationCallback invocation)
 2             throws Throwable {　　
 3 
 4         // 如果transaction attribute为空,该方法就是非事务（非编程式事务）
 5         final TransactionAttribute txAttr = getTransactionAttributeSource().getTransactionAttribute(method, targetClass);
 6         final PlatformTransactionManager tm = determineTransactionManager(txAttr);
 7         final String joinpointIdentification = methodIdentification(method, targetClass, txAttr);
 8 　　　　 // 标准声明式事务：如果事务属性为空 或者 非回调偏向的事务管理器
 9         if (txAttr == null || !(tm instanceof CallbackPreferringPlatformTransactionManager)) {
10             // Standard transaction demarcation with getTransaction and commit/rollback calls.
11             TransactionInfo txInfo = createTransactionIfNecessary(tm, txAttr, joinpointIdentification);
12             Object retVal = null;
13             try {
14                 // 这里就是一个环绕增强，在这个proceed前后可以自己定义增强实现
15                 // 方法执行
16                 retVal = invocation.proceedWithInvocation();
17             }
18             catch (Throwable ex) {
19                 // 根据事务定义的，该异常需要回滚就回滚，否则提交事务
20                 completeTransactionAfterThrowing(txInfo, ex);
21                 throw ex;
22             }
23             finally {//清空当前事务信息，重置为老的
24                 cleanupTransactionInfo(txInfo);
25             }//返回结果之前提交事务
26             commitTransactionAfterReturning(txInfo);
27             return retVal;
28         }
29 　　　　 // 编程式事务：（回调偏向）
30         else {
31             final ThrowableHolder throwableHolder = new ThrowableHolder();
32 
33             // It's a CallbackPreferringPlatformTransactionManager: pass a TransactionCallback in.
34             try {
35                 Object result = ((CallbackPreferringPlatformTransactionManager) tm).execute(txAttr,
36                         new TransactionCallback<Object>() {
37                             @Override
38                             public Object doInTransaction(TransactionStatus status) {
39                                 TransactionInfo txInfo = prepareTransactionInfo(tm, txAttr, joinpointIdentification, status);
40                                 try {
41                                     return invocation.proceedWithInvocation();
42                                 }
43                                 catch (Throwable ex) {// 如果该异常需要回滚
44                                     if (txAttr.rollbackOn(ex)) {
45                                         // 如果是运行时异常返回
46                                         if (ex instanceof RuntimeException) {
47                                             throw (RuntimeException) ex;
48                                         }// 如果是其它异常都抛ThrowableHolderException
49                                         else {
50                                             throw new ThrowableHolderException(ex);
51                                         }
52                                     }// 如果不需要回滚
53                                     else {
54                                         // 定义异常，最终就直接提交事务了
55                                         throwableHolder.throwable = ex;
56                                         return null;
57                                     }
58                                 }
59                                 finally {//清空当前事务信息，重置为老的
60                                     cleanupTransactionInfo(txInfo);
61                                 }
62                             }
63                         });
64 
65                 // 上抛异常
66                 if (throwableHolder.throwable != null) {
67                     throw throwableHolder.throwable;
68                 }
69                 return result;
70             }
71             catch (ThrowableHolderException ex) {
72                 throw ex.getCause();
73             }
74             catch (TransactionSystemException ex2) {
75                 if (throwableHolder.throwable != null) {
76                     logger.error("Application exception overridden by commit exception", throwableHolder.throwable);
77                     ex2.initApplicationException(throwableHolder.throwable);
78                 }
79                 throw ex2;
80             }
81             catch (Throwable ex2) {
82                 if (throwableHolder.throwable != null) {
83                     logger.error("Application exception overridden by commit exception", throwableHolder.throwable);
84                 }
85                 throw ex2;
86             }
87         }
88     }
```



 如上图，我们主要看第一个分支，申明式事务，核心流程如下：

1.createTransactionIfNecessary():如果有必要，创建事务

2.InvocationCallback的proceedWithInvocation()：InvocationCallback是父类的内部回调接口，子类中实现该接口供父类调用，子类TransactionInterceptor中invocation.proceed()。回调方法执行

3.异常回滚completeTransactionAfterThrowing()

#### 1.createTransactionIfNecessary():



```
 1 protected TransactionInfo createTransactionIfNecessary(
 2             PlatformTransactionManager tm, TransactionAttribute txAttr, final String joinpointIdentification) {
 3 
 4         // 如果还没有定义名字，把连接点的ID定义成事务的名称
 5         if (txAttr != null && txAttr.getName() == null) {
 6             txAttr = new DelegatingTransactionAttribute(txAttr) {
 7                 @Override
 8                 public String getName() {
 9                     return joinpointIdentification;
10                 }
11             };
12         }
13 
14         TransactionStatus status = null;
15         if (txAttr != null) {
16             if (tm != null) {
17                 status = tm.getTransaction(txAttr);
18             }
19             else {
20                 if (logger.isDebugEnabled()) {
21                     logger.debug("Skipping transactional joinpoint [" + joinpointIdentification +
22                             "] because no transaction manager has been configured");
23                 }
24             }
25         }
26         return prepareTransactionInfo(tm, txAttr, joinpointIdentification, status);
27     }
```



核心就是：

1）getTransaction()，根据事务属性获取事务TransactionStatus，大道归一，都是调用PlatformTransactionManager.getTransaction()，源码见3.3.1。
2）prepareTransactionInfo(),构造一个TransactionInfo事务信息对象，绑定当前线程：ThreadLocal<TransactionInfo>。

#### 2.invocation.proceed()回调业务方法:

最终实现类是ReflectiveMethodInvocation，类图如下：

![img](https://img2018.cnblogs.com/blog/584866/201809/584866-20180917161011294-1859782822.png)

如上图，ReflectiveMethodInvocation类实现了ProxyMethodInvocation接口，但是ProxyMethodInvocation继承了3层接口...ProxyMethodInvocation->MethodInvocation->Invocation->Joinpoint

Joinpoint：连接点接口，定义了执行接口：Object proceed() throws Throwable; 执行当前连接点，并跳到拦截器链上的下一个拦截器。

Invocation：调用接口，继承自Joinpoint，定义了获取参数接口： Object[] getArguments();是一个带参数的、可被拦截器拦截的连接点。

MethodInvocation：方法调用接口，继承自Invocation，定义了获取方法接口：Method getMethod(); 是一个带参数的可被拦截的连接点方法。

ProxyMethodInvocation：代理方法调用接口，继承自MethodInvocation，定义了获取代理对象接口：Object getProxy();是一个由代理类执行的方法调用连接点方法。

ReflectiveMethodInvocation：实现了ProxyMethodInvocation接口，自然就实现了父类接口的的所有接口。获取代理类，获取方法，获取参数，用代理类执行这个方法并且自动跳到下一个连接点。

下面看一下proceed方法源码：



```
 1 @Override
 2     public Object proceed() throws Throwable {
 3         //    启动时索引为-1，唤醒连接点，后续递增
 4         if (this.currentInterceptorIndex == this.interceptorsAndDynamicMethodMatchers.size() - 1) {
 5             return invokeJoinpoint();
 6         }
 7 
 8         Object interceptorOrInterceptionAdvice =
 9                 this.interceptorsAndDynamicMethodMatchers.get(++this.currentInterceptorIndex);
10         if (interceptorOrInterceptionAdvice instanceof InterceptorAndDynamicMethodMatcher) {
11             // 这里进行动态方法匹配校验，静态的方法匹配早已经校验过了（MethodMatcher接口有两种典型：动态/静态校验）
13             InterceptorAndDynamicMethodMatcher dm =
14                     (InterceptorAndDynamicMethodMatcher) interceptorOrInterceptionAdvice;
15             if (dm.methodMatcher.matches(this.method, this.targetClass, this.arguments)) {
16                 return dm.interceptor.invoke(this);
17             }
18             else {
19                 // 动态匹配失败，跳过当前拦截，进入下一个（拦截器链）
21                 return proceed();
22             }
23         }
24         else {
25             // 它是一个拦截器，所以我们只调用它:在构造这个对象之前，切入点将被静态地计算。
27             return ((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this);
28         }
29     }
```



咱们这里最终调用的是((MethodInterceptor) interceptorOrInterceptionAdvice).invoke(this);就是TransactionInterceptor事务拦截器回调 目标业务方法（addUserBalanceAndUser）。

#### 3.completeTransactionAfterThrowing()

#### 最终调用AbstractPlatformTransactionManager的rollback()，提交事务commitTransactionAfterReturning()最终调用AbstractPlatformTransactionManager的commit(),源码见3.3.3

#### 总结：

可见不管是编程式事务，还是声明式事务，最终源码都是调用事务管理器的PlatformTransactionManager接口的3个方法：

1. getTransaction
2. commit
3. rollback

下一节我们就来看看这个事务管理如何实现这3个方法。

 

[回到顶部](https://www.cnblogs.com/dennyzhangdd/p/9602673.html#_labelTop)

## 三、事务核心源码

咱们看一下核心类图：

![img](https://images2018.cnblogs.com/blog/584866/201809/584866-20180906161402774-1489010672.png)

如上提所示，PlatformTransactionManager顶级接口定义了最核心的事务管理方法，下面一层是AbstractPlatformTransactionManager抽象类，实现了PlatformTransactionManager接口的方法并定义了一些抽象方法，供子类拓展。最后下面一层是2个经典事务管理器：

1.DataSourceTransactionmanager,即JDBC单数据库事务管理器，基于Connection实现，

2.JtaTransactionManager,即多数据库事务管理器（又叫做分布式事务管理器），其实现了JTA规范，使用XA协议进行两阶段提交。

我们这里只看基于JDBC connection的DataSourceTransactionmanager源码。

PlatformTransactionManager接口：



```
1 public interface PlatformTransactionManager {
2     // 获取事务状态
3     TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;
4 　　// 事务提交
5     void commit(TransactionStatus status) throws TransactionException;
6 　　// 事务回滚
7     void rollback(TransactionStatus status) throws TransactionException;
8 }
```





### 1. getTransaction获取事务

![img](https://img2018.cnblogs.com/blog/584866/201811/584866-20181129183338568-2047123961.png)

AbstractPlatformTransactionManager实现了getTransaction()方法如下：



```
 1     @Override
 2     public final TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException {
 3         Object transaction = doGetTransaction();
 4 
 5         // Cache debug flag to avoid repeated checks.
 6         boolean debugEnabled = logger.isDebugEnabled();
 7 
 8         if (definition == null) {
 9             // Use defaults if no transaction definition given.
10             definition = new DefaultTransactionDefinition();
11         }
12 　　　　  // 如果当前已经存在事务
13         if (isExistingTransaction(transaction)) {
14             // 根据不同传播机制不同处理
15             return handleExistingTransaction(definition, transaction, debugEnabled);
16         }
17 
18         // 超时不能小于默认值
19         if (definition.getTimeout() < TransactionDefinition.TIMEOUT_DEFAULT) {
20             throw new InvalidTimeoutException("Invalid transaction timeout", definition.getTimeout());
21         }
22 
23         // 当前不存在事务，传播机制=MANDATORY（支持当前事务，没事务报错），报错
24         if (definition.getPropagationBehavior() == TransactionDefinition.PROPAGATION_MANDATORY) {
25             throw new IllegalTransactionStateException(
26                     "No existing transaction found for transaction marked with propagation 'mandatory'");
27         }// 当前不存在事务，传播机制=REQUIRED/REQUIRED_NEW/NESTED,这三种情况，需要新开启事务，且加上事务同步
28         else if (definition.getPropagationBehavior() == TransactionDefinition.PROPAGATION_REQUIRED ||
29                 definition.getPropagationBehavior() == TransactionDefinition.PROPAGATION_REQUIRES_NEW ||
30                 definition.getPropagationBehavior() == TransactionDefinition.PROPAGATION_NESTED) {
31             SuspendedResourcesHolder suspendedResources = suspend(null);
32             if (debugEnabled) {
33                 logger.debug("Creating new transaction with name [" + definition.getName() + "]: " + definition);
34             }
35             try {// 是否需要新开启同步// 开启// 开启
36                 boolean newSynchronization = (getTransactionSynchronization() != SYNCHRONIZATION_NEVER);
37                 DefaultTransactionStatus status = newTransactionStatus(
38                         definition, transaction, true, newSynchronization, debugEnabled, suspendedResources);
39                 doBegin(transaction, definition);// 开启新事务
40                 prepareSynchronization(status, definition);//预备同步
41                 return status;
42             }
43             catch (RuntimeException ex) {
44                 resume(null, suspendedResources);
45                 throw ex;
46             }
47             catch (Error err) {
48                 resume(null, suspendedResources);
49                 throw err;
50             }
51         }
52         else {
53             // 当前不存在事务当前不存在事务，且传播机制=PROPAGATION_SUPPORTS/PROPAGATION_NOT_SUPPORTED/PROPAGATION_NEVER，这三种情况，创建“空”事务:没有实际事务，但可能是同步。警告：定义了隔离级别，但并没有真实的事务初始化，隔离级别被忽略有隔离级别但是并没有定义实际的事务初始化，有隔离级别但是并没有定义实际的事务初始化，
54             if (definition.getIsolationLevel() != TransactionDefinition.ISOLATION_DEFAULT && logger.isWarnEnabled()) {
55                 logger.warn("Custom isolation level specified but no actual transaction initiated; " +
56                         "isolation level will effectively be ignored: " + definition);
57             }
58             boolean newSynchronization = (getTransactionSynchronization() == SYNCHRONIZATION_ALWAYS);
59             return prepareTransactionStatus(definition, null, true, newSynchronization, debugEnabled, null);
60         }
61     }
```



 如上图，源码分成了2条处理线，

1.当前已存在事务：isExistingTransaction()判断是否存在事务，存在事务handleExistingTransaction()根据不同传播机制不同处理

2.当前不存在事务: 不同传播机制不同处理

handleExistingTransaction()源码如下：



```
 1 private TransactionStatus handleExistingTransaction(
 2             TransactionDefinition definition, Object transaction, boolean debugEnabled)
 3             throws TransactionException {
 4 　　　　　// 1.NERVER（不支持当前事务;如果当前事务存在，抛出异常）报错
 5         if (definition.getPropagationBehavior() == TransactionDefinition.PROPAGATION_NEVER) {
 6             throw new IllegalTransactionStateException(
 7                     "Existing transaction found for transaction marked with propagation 'never'");
 8         }
 9 　　　　  // 2.NOT_SUPPORTED（不支持当前事务，现有同步将被挂起）挂起当前事务
10         if (definition.getPropagationBehavior() == TransactionDefinition.PROPAGATION_NOT_SUPPORTED) {
11             if (debugEnabled) {
12                 logger.debug("Suspending current transaction");
13             }
14             Object suspendedResources = suspend(transaction);
15             boolean newSynchronization = (getTransactionSynchronization() == SYNCHRONIZATION_ALWAYS);
16             return prepareTransactionStatus(
17                     definition, null, false, newSynchronization, debugEnabled, suspendedResources);
18         }
19 　　　　  // 3.REQUIRES_NEW挂起当前事务，创建新事务
20         if (definition.getPropagationBehavior() == TransactionDefinition.PROPAGATION_REQUIRES_NEW) {
21             if (debugEnabled) {
22                 logger.debug("Suspending current transaction, creating new transaction with name [" +
23                         definition.getName() + "]");
24             }// 挂起当前事务
25             SuspendedResourcesHolder suspendedResources = suspend(transaction);
26             try {// 创建新事务
27                 boolean newSynchronization = (getTransactionSynchronization() != SYNCHRONIZATION_NEVER);
28                 DefaultTransactionStatus status = newTransactionStatus(
29                         definition, transaction, true, newSynchronization, debugEnabled, suspendedResources);
30                 doBegin(transaction, definition);
31                 prepareSynchronization(status, definition);
32                 return status;
33             }
34             catch (RuntimeException beginEx) {
35                 resumeAfterBeginException(transaction, suspendedResources, beginEx);
36                 throw beginEx;
37             }
38             catch (Error beginErr) {
39                 resumeAfterBeginException(transaction, suspendedResources, beginErr);
40                 throw beginErr;
41             }
42         }
43 　　　　 // 4.NESTED嵌套事务
44         if (definition.getPropagationBehavior() == TransactionDefinition.PROPAGATION_NESTED) {
45             if (!isNestedTransactionAllowed()) {
46                 throw new NestedTransactionNotSupportedException(
47                         "Transaction manager does not allow nested transactions by default - " +
48                         "specify 'nestedTransactionAllowed' property with value 'true'");
49             }
50             if (debugEnabled) {
51                 logger.debug("Creating nested transaction with name [" + definition.getName() + "]");
52             }// 是否支持保存点：非JTA事务走这个分支。AbstractPlatformTransactionManager默认是true，JtaTransactionManager复写了该方法false，DataSourceTransactionmanager没有复写，还是true,
53             if (useSavepointForNestedTransaction()) { 
54                 // Usually uses JDBC 3.0 savepoints. Never activates Spring synchronization.
55                 DefaultTransactionStatus status =
56                         prepareTransactionStatus(definition, transaction, false, false, debugEnabled, null);
57                 status.createAndHoldSavepoint();// 创建保存点
58                 return status;
59             }
60             else {
61                 // JTA事务走这个分支，创建新事务
62                 boolean newSynchronization = (getTransactionSynchronization() != SYNCHRONIZATION_NEVER);
63                 DefaultTransactionStatus status = newTransactionStatus(
64                         definition, transaction, true, newSynchronization, debugEnabled, null);
65                 doBegin(transaction, definition);
66                 prepareSynchronization(status, definition);
67                 return status;
68             }
69         }
70 
71         
72         if (debugEnabled) {
73             logger.debug("Participating in existing transaction");
74         }
75         if (isValidateExistingTransaction()) {
76             if (definition.getIsolationLevel() != TransactionDefinition.ISOLATION_DEFAULT) {
77                 Integer currentIsolationLevel = TransactionSynchronizationManager.getCurrentTransactionIsolationLevel();
78                 if (currentIsolationLevel == null || currentIsolationLevel != definition.getIsolationLevel()) {
79                     Constants isoConstants = DefaultTransactionDefinition.constants;
80                     throw new IllegalTransactionStateException("Participating transaction with definition [" +
81                             definition + "] specifies isolation level which is incompatible with existing transaction: " +
82                             (currentIsolationLevel != null ?
83                                     isoConstants.toCode(currentIsolationLevel, DefaultTransactionDefinition.PREFIX_ISOLATION) :
84                                     "(unknown)"));
85                 }
86             }
87             if (!definition.isReadOnly()) {
88                 if (TransactionSynchronizationManager.isCurrentTransactionReadOnly()) {
89                     throw new IllegalTransactionStateException("Participating transaction with definition [" +
90                             definition + "] is not marked as read-only but existing transaction is");
91                 }
92             }
93         }// 到这里PROPAGATION_SUPPORTS 或 PROPAGATION_REQUIRED或PROPAGATION_MANDATORY，存在事务加入事务即可，prepareTransactionStatus第三个参数就是是否需要新事务。false代表不需要新事物
94         boolean newSynchronization = (getTransactionSynchronization() != SYNCHRONIZATION_NEVER);
95         return prepareTransactionStatus(definition, transaction, false, newSynchronization, debugEnabled, null);
96     }
```



如上图，当前线程已存在事务情况下，新的不同隔离级别处理情况：

1.NERVER：不支持当前事务;如果当前事务存在，抛出异常:"Existing transaction found for transaction marked with propagation 'never'"
2.NOT_SUPPORTED：不支持当前事务，现有同步将被挂起:suspend()
3.REQUIRES_NEW挂起当前事务，创建新事务:

　　1)suspend()

　　2)doBegin()
4.NESTED嵌套事务

　　1）非JTA事务：createAndHoldSavepoint()创建JDBC3.0保存点，不需要同步

　　2) JTA事务：开启新事务，doBegin()+prepareSynchronization()需要同步

 这里有几个核心方法：挂起当前事务suspend()、开启新事务doBegin()。

suspend()源码如下：



```
 1 protected final SuspendedResourcesHolder suspend(Object transaction) throws TransactionException {
 2         if (TransactionSynchronizationManager.isSynchronizationActive()) {// 1.当前存在同步，
 3             List<TransactionSynchronization> suspendedSynchronizations = doSuspendSynchronization();
 4             try {
 5                 Object suspendedResources = null;
 6                 if (transaction != null) {// 事务不为空，挂起事务
 7                     suspendedResources = doSuspend(transaction);
 8                 }// 解除绑定当前事务各种属性：名称、只读、隔离级别、是否是真实的事务.
 9                 String name = TransactionSynchronizationManager.getCurrentTransactionName();
10                 TransactionSynchronizationManager.setCurrentTransactionName(null);
11                 boolean readOnly = TransactionSynchronizationManager.isCurrentTransactionReadOnly();
12                 TransactionSynchronizationManager.setCurrentTransactionReadOnly(false);
13                 Integer isolationLevel = TransactionSynchronizationManager.getCurrentTransactionIsolationLevel();
14                 TransactionSynchronizationManager.setCurrentTransactionIsolationLevel(null);
15                 boolean wasActive = TransactionSynchronizationManager.isActualTransactionActive();
16                 TransactionSynchronizationManager.setActualTransactionActive(false);
17                 return new SuspendedResourcesHolder(
18                         suspendedResources, suspendedSynchronizations, name, readOnly, isolationLevel, wasActive);
19             }
20             catch (RuntimeException ex) {
21                 // doSuspend failed - original transaction is still active...
22                 doResumeSynchronization(suspendedSynchronizations);
23                 throw ex;
24             }
25             catch (Error err) {
26                 // doSuspend failed - original transaction is still active...
27                 doResumeSynchronization(suspendedSynchronizations);
28                 throw err;
29             }
30         }// 2.没有同步但，事务不为空，挂起事务
31         else if (transaction != null) {
32             // Transaction active but no synchronization active.
33             Object suspendedResources = doSuspend(transaction);
34             return new SuspendedResourcesHolder(suspendedResources);
35         }// 2.没有同步但，事务为空，什么都不用做
36         else {
37             // Neither transaction nor synchronization active.
38             return null;
39         }
40     }
```



doSuspend(),挂起事务，AbstractPlatformTransactionManager抽象类doSuspend()会报错：不支持挂起，如果具体事务执行器支持就复写doSuspend()，DataSourceTransactionManager实现如下：

```
1 @Override
2     protected Object doSuspend(Object transaction) {
3         DataSourceTransactionObject txObject = (DataSourceTransactionObject) transaction;
4         txObject.setConnectionHolder(null);
5         return TransactionSynchronizationManager.unbindResource(this.dataSource);
6     }
```

挂起DataSourceTransactionManager事务的核心操作就是：

1.把当前事务的connectionHolder数据库连接持有者清空。

2.当前线程解绑datasource.其实就是ThreadLocal移除对应变量（TransactionSynchronizationManager类中定义的private static final ThreadLocal<Map<Object, Object>> *resources = new NamedThreadLocal<Map<Object, Object>>("Transactional resources");*）

TransactionSynchronizationManager事务同步管理器，该类维护了多个线程本地变量ThreadLocal，如下图：



```
 1 public abstract class TransactionSynchronizationManager {
 2 
 3     private static final Log logger = LogFactory.getLog(TransactionSynchronizationManager.class);
 4     // 事务资源：map<k,v> 两种数据对。1.会话工厂和会话k=SqlsessionFactory v=SqlSessionHolder 2.数据源和连接k=DataSource v=ConnectionHolder
 5     private static final ThreadLocal<Map<Object, Object>> resources =
 6             new NamedThreadLocal<Map<Object, Object>>("Transactional resources");
 7     // 事务同步
 8     private static final ThreadLocal<Set<TransactionSynchronization>> synchronizations =
 9             new NamedThreadLocal<Set<TransactionSynchronization>>("Transaction synchronizations");
10 　　// 当前事务名称
11     private static final ThreadLocal<String> currentTransactionName =
12             new NamedThreadLocal<String>("Current transaction name");
13 　　// 当前事务的只读属性
14     private static final ThreadLocal<Boolean> currentTransactionReadOnly =
15             new NamedThreadLocal<Boolean>("Current transaction read-only status");
16 　　// 当前事务的隔离级别
17     private static final ThreadLocal<Integer> currentTransactionIsolationLevel =
18             new NamedThreadLocal<Integer>("Current transaction isolation level");
19 　　// 是否存在事务
20     private static final ThreadLocal<Boolean> actualTransactionActive =
21             new NamedThreadLocal<Boolean>("Actual transaction active");
22 。。。
23 }
```



doBegin()源码如下：



```
 1 @Override
 2     protected void doBegin(Object transaction, TransactionDefinition definition) {
 3         DataSourceTransactionObject txObject = (DataSourceTransactionObject) transaction;
 4         Connection con = null;
 5 
 6         try {// 如果事务还没有connection或者connection在事务同步状态，重置新的connectionHolder
 7             if (!txObject.hasConnectionHolder() ||
 8                     txObject.getConnectionHolder().isSynchronizedWithTransaction()) {
 9                 Connection newCon = this.dataSource.getConnection();
10                 if (logger.isDebugEnabled()) {
11                     logger.debug("Acquired Connection [" + newCon + "] for JDBC transaction");
12                 }// 重置新的connectionHolder
13                 txObject.setConnectionHolder(new ConnectionHolder(newCon), true);
14             }
15 　　　　　　　//设置新的连接为事务同步中
16             txObject.getConnectionHolder().setSynchronizedWithTransaction(true);
17             con = txObject.getConnectionHolder().getConnection();
18 　　　　     //conn设置事务隔离级别,只读
19             Integer previousIsolationLevel = DataSourceUtils.prepareConnectionForTransaction(con, definition);
20             txObject.setPreviousIsolationLevel(previousIsolationLevel);//DataSourceTransactionObject设置事务隔离级别
21 
22             // 如果是自动提交切换到手动提交
23             // so we don't want to do it unnecessarily (for example if we've explicitly
24             // configured the connection pool to set it already).
25             if (con.getAutoCommit()) {
26                 txObject.setMustRestoreAutoCommit(true);
27                 if (logger.isDebugEnabled()) {
28                     logger.debug("Switching JDBC Connection [" + con + "] to manual commit");
29                 }
30                 con.setAutoCommit(false);
31             }
32 　　　　　　　// 如果只读，执行sql设置事务只读
33             prepareTransactionalConnection(con, definition);
34             txObject.getConnectionHolder().setTransactionActive(true);// 设置connection持有者的事务开启状态
35 
36             int timeout = determineTimeout(definition);
37             if (timeout != TransactionDefinition.TIMEOUT_DEFAULT) {
38                 txObject.getConnectionHolder().setTimeoutInSeconds(timeout);// 设置超时秒数
39             }
40 
41             // 绑定connection持有者到当前线程
42             if (txObject.isNewConnectionHolder()) {
43                 TransactionSynchronizationManager.bindResource(getDataSource(), txObject.getConnectionHolder());
44             }
45         }
46 
47         catch (Throwable ex) {
48             if (txObject.isNewConnectionHolder()) {
49                 DataSourceUtils.releaseConnection(con, this.dataSource);
50                 txObject.setConnectionHolder(null, false);
51             }
52             throw new CannotCreateTransactionException("Could not open JDBC Connection for transaction", ex);
53         }
54     }
```



如上图，开启新事务的准备工作doBegin()的核心操作就是：

1.DataSourceTransactionObject“数据源事务对象”，设置ConnectionHolder，再给ConnectionHolder设置各种属性：自动提交、超时、事务开启、隔离级别。

2.给当前线程绑定一个线程本地变量，key=DataSource数据源 v=ConnectionHolder数据库连接。

### 2. commit提交事务

一、讲解源码之前先看一下资源管理类：

SqlSessionSynchronization是SqlSessionUtils的一个内部类，继承自TransactionSynchronizationAdapter抽象类，实现了事务同步接口TransactionSynchronization。

类图如下：

![img](https://img2018.cnblogs.com/blog/584866/201809/584866-20180919151553950-212649457.png)

TransactionSynchronization接口定义了事务操作时的对应资源的（JDBC事务那么就是SqlSessionSynchronization）管理方法：



```
 1     // 挂起事务 　　 2　　　void suspend();
 3     // 唤醒事务 　　 4　　　void resume();
 5     
 6     void flush();
 7 
 8     // 提交事务前
 9     void beforeCommit(boolean readOnly);
10 
11     // 提交事务完成前
12     void beforeCompletion();
13 
14     // 提交事务后
15     void afterCommit();
16 
17     // 提交事务完成后
18     void afterCompletion(int status);
```



后续很多都是使用这些接口管理事务。

二、 commit提交事务

 

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210527112420.png)

AbstractPlatformTransactionManager的commit源码如下：



```java
 1 @Override
 2     public final void commit(TransactionStatus status) throws TransactionException {
 3         if (status.isCompleted()) {// 如果事务已完结，报错无法再次提交
 4             throw new IllegalTransactionStateException(
 5                     "Transaction is already completed - do not call commit or rollback more than once per transaction");
 6         }
 7 
 8         DefaultTransactionStatus defStatus = (DefaultTransactionStatus) status;
 9         if (defStatus.isLocalRollbackOnly()) {// 如果事务明确标记为回滚，
10             if (defStatus.isDebug()) {
11                 logger.debug("Transactional code has requested rollback");
12             }
13             processRollback(defStatus);//执行回滚
14             return;
15         }//如果不需要全局回滚时提交 且 全局回滚
16         if (!shouldCommitOnGlobalRollbackOnly() && defStatus.isGlobalRollbackOnly()) {
17             if (defStatus.isDebug()) {
18                 logger.debug("Global transaction is marked as rollback-only but transactional code requested commit");
19             }//执行回滚
20             processRollback(defStatus);
21             // 仅在最外层事务边界（新事务）或显式地请求时抛出“未期望的回滚异常”
23             if (status.isNewTransaction() || isFailEarlyOnGlobalRollbackOnly()) {
24                 throw new UnexpectedRollbackException(
25                         "Transaction rolled back because it has been marked as rollback-only");
26             }
27             return;
28         }
29 　　　　 // 执行提交事务
30         processCommit(defStatus);
31     }
```

如上图，各种判断：

- 1.如果事务明确标记为本地回滚，-》执行回滚
- 2.如果不需要全局回滚时提交 且 全局回滚-》执行回滚
- 3.提交事务，核心方法processCommit()

processCommit如下：

```java
 1 private void processCommit(DefaultTransactionStatus status) throws TransactionException {
 2         try {
 3             boolean beforeCompletionInvoked = false;
 4             try {//3个前置操作
 5                 prepareForCommit(status);
 6                 triggerBeforeCommit(status);
 7                 triggerBeforeCompletion(status);
 8                 beforeCompletionInvoked = true;//3个前置操作已调用
 9                 boolean globalRollbackOnly = false;//新事务 或 全局回滚失败
10                 if (status.isNewTransaction() || isFailEarlyOnGlobalRollbackOnly()) {
11                     globalRollbackOnly = status.isGlobalRollbackOnly();
12                 }//1.有保存点，即嵌套事务
13                 if (status.hasSavepoint()) {
14                     if (status.isDebug()) {
15                         logger.debug("Releasing transaction savepoint");
16                     }//释放保存点
17                     status.releaseHeldSavepoint();
18                 }//2.新事务
19                 else if (status.isNewTransaction()) {
20                     if (status.isDebug()) {
21                         logger.debug("Initiating transaction commit");
22                     }//调用事务处理器提交事务
23                     doCommit(status);
24                 }
25                 // 3.非新事务，且全局回滚失败，但是提交时没有得到异常，抛出异常
27                 if (globalRollbackOnly) {
28                     throw new UnexpectedRollbackException(
29                             "Transaction silently rolled back because it has been marked as rollback-only");
30                 }
31             }
32             catch (UnexpectedRollbackException ex) {
33                 // 触发完成后事务同步，状态为回滚
34                 triggerAfterCompletion(status, TransactionSynchronization.STATUS_ROLLED_BACK);
35                 throw ex;
36             }// 事务异常
37             catch (TransactionException ex) {
38                 // 提交失败回滚
39                 if (isRollbackOnCommitFailure()) {
40                     doRollbackOnCommitException(status, ex);
41                 }// 触发完成后回调，事务同步状态为未知
42                 else {
43                     triggerAfterCompletion(status, TransactionSynchronization.STATUS_UNKNOWN);
44                 }
45                 throw ex;
46             }// 运行时异常
47             catch (RuntimeException ex) {　　　　　　　　　　　　// 如果3个前置步骤未完成，调用前置的最后一步操作
48                 if (!beforeCompletionInvoked) {
49                     triggerBeforeCompletion(status);
50                 }// 提交异常回滚
51                 doRollbackOnCommitException(status, ex);
52                 throw ex;
53             }// 其它异常
54             catch (Error err) {　　　　　　　　　　　　　　// 如果3个前置步骤未完成，调用前置的最后一步操作
55                 if (!beforeCompletionInvoked) {
56                     triggerBeforeCompletion(status);
57                 }// 提交异常回滚
58                 doRollbackOnCommitException(status, err);
59                 throw err;
60             }
61 
62             // Trigger afterCommit callbacks, with an exception thrown there
63             // propagated to callers but the transaction still considered as committed.
64             try {
65                 triggerAfterCommit(status);
66             }
67             finally {
68                 triggerAfterCompletion(status, TransactionSynchronization.STATUS_COMMITTED);
69             }
70 
71         }
72         finally {
73             cleanupAfterCompletion(status);
74         }
75     }
```



如上图，commit事务时，有6个核心操作，分别是3个前置操作，3个后置操作，如下：

1.prepareForCommit(status);源码是空的，没有拓展目前。

2.triggerBeforeCommit(status); 提交前触发操作

```java
1 protected final void triggerBeforeCommit(DefaultTransactionStatus status) {
2         if (status.isNewSynchronization()) {
3             if (status.isDebug()) {
4                 logger.trace("Triggering beforeCommit synchronization");
5             }
6             TransactionSynchronizationUtils.triggerBeforeCommit(status.isReadOnly());
7         }
8     }
```

triggerBeforeCommit源码如下：

```
1 public static void triggerBeforeCommit(boolean readOnly) {
2         for (TransactionSynchronization synchronization : TransactionSynchronizationManager.getSynchronizations()) {
3             synchronization.beforeCommit(readOnly);
4         }
5     }
```

 如上图，TransactionSynchronizationManager类定义了多个ThreadLocal（线程本地变量），其中一个用以保存当前线程的事务同步：

```
private static final ThreadLocal<Set<TransactionSynchronization>> synchronizations = new NamedThreadLocal<Set<TransactionSynchronization>>("Transaction synchronizations");
```

遍历事务同步器，把每个事务同步器都执行“提交前”操作，比如咱们用的jdbc事务，那么最终就是SqlSessionUtils.beforeCommit()->this.holder.getSqlSession().commit();提交会话。(源码由于是spring管理实务，最终不会执行事务提交，例如是DefaultSqlSession：执行清除缓存、重置状态操作)

3.triggerBeforeCompletion(status);完成前触发操作，如果是jdbc事务，那么最终就是，

SqlSessionUtils.beforeCompletion->

TransactionSynchronizationManager.unbindResource(sessionFactory); 解绑当前线程的会话工厂

this.holder.getSqlSession().close();关闭会话。(源码由于是spring管理实务，最终不会执行事务close操作，例如是DefaultSqlSession，也会执行各种清除收尾操作)

4.triggerAfterCommit(status);提交事务后触发操作。TransactionSynchronizationUtils.*triggerAfterCommit();*->TransactionSynchronizationUtils.invokeAfterCommit，如下：



```
1 public static void invokeAfterCommit(List<TransactionSynchronization> synchronizations) {
2         if (synchronizations != null) {
3             for (TransactionSynchronization synchronization : synchronizations) {
4                 synchronization.afterCommit();
5             }
6         }
7     }
```



好吧，一顿找，最后在TransactionSynchronizationAdapter中复写过，并且是空的....SqlSessionSynchronization继承了TransactionSynchronizationAdapter但是没有复写这个方法。

\5. triggerAfterCompletion(status, TransactionSynchronization.STATUS_COMMITTED);

TransactionSynchronizationUtils.TransactionSynchronizationUtils.invokeAfterCompletion,如下：



```
 1 public static void invokeAfterCompletion(List<TransactionSynchronization> synchronizations, int completionStatus) {
 2         if (synchronizations != null) {
 3             for (TransactionSynchronization synchronization : synchronizations) {
 4                 try {
 5                     synchronization.afterCompletion(completionStatus);
 6                 }
 7                 catch (Throwable tsex) {
 8                     logger.error("TransactionSynchronization.afterCompletion threw exception", tsex);
 9                 }
10             }
11         }
12     }
```



afterCompletion：对于JDBC事务来说，最终：

1）如果会话任然活着，关闭会话，

2）重置各种属性：SQL会话同步器（SqlSessionSynchronization）的SQL会话持有者（SqlSessionHolder）的referenceCount引用计数、synchronizedWithTransaction同步事务、rollbackOnly只回滚、deadline超时时间点。

6.cleanupAfterCompletion(status);

1）设置事务状态为已完成。

2) 如果是新的事务同步，解绑当前线程绑定的数据库资源，重置数据库连接

3）如果存在挂起的事务（嵌套事务），唤醒挂起的老事务的各种资源：数据库资源、同步器。



```
 1     private void cleanupAfterCompletion(DefaultTransactionStatus status) {
 2         status.setCompleted();//设置事务状态完成　　　　　　 //如果是新的同步，清空当前线程绑定的除了资源外的全部线程本地变量：包括事务同步器、事务名称、只读属性、隔离级别、真实的事务激活状态
 3         if (status.isNewSynchronization()) {
 4             TransactionSynchronizationManager.clear();
 5         }//如果是新的事务同步
 6         if (status.isNewTransaction()) {
 7             doCleanupAfterCompletion(status.getTransaction());
 8         }//如果存在挂起的资源
 9         if (status.getSuspendedResources() != null) {
10             if (status.isDebug()) {
11                 logger.debug("Resuming suspended transaction after completion of inner transaction");
12             }//唤醒挂起的事务和资源（重新绑定之前挂起的数据库资源，唤醒同步器，注册同步器到TransactionSynchronizationManager）
13             resume(status.getTransaction(), (SuspendedResourcesHolder) status.getSuspendedResources());
14         }
15     }
```



对于DataSourceTransactionManager，doCleanupAfterCompletion源码如下：



```
 1     protected void doCleanupAfterCompletion(Object transaction) {
 2         DataSourceTransactionObject txObject = (DataSourceTransactionObject) transaction;
 3 
 4         // 如果是最新的连接持有者，解绑当前线程绑定的<数据库资源，ConnectionHolder>
 5         if (txObject.isNewConnectionHolder()) {
 6             TransactionSynchronizationManager.unbindResource(this.dataSource);
 7         }
 8 
 9         // 重置数据库连接（隔离级别、只读）
10         Connection con = txObject.getConnectionHolder().getConnection();
11         try {
12             if (txObject.isMustRestoreAutoCommit()) {
13                 con.setAutoCommit(true);
14             }
15             DataSourceUtils.resetConnectionAfterTransaction(con, txObject.getPreviousIsolationLevel());
16         }
17         catch (Throwable ex) {
18             logger.debug("Could not reset JDBC Connection after transaction", ex);
19         }
20 
21         if (txObject.isNewConnectionHolder()) {
22             if (logger.isDebugEnabled()) {
23                 logger.debug("Releasing JDBC Connection [" + con + "] after transaction");
24             }// 资源引用计数-1，关闭数据库连接
25             DataSourceUtils.releaseConnection(con, this.dataSource);
26         }
27         // 重置连接持有者的全部属性
28         txObject.getConnectionHolder().clear();
29     }
```



 



### 3. rollback回滚事务

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210527112451.png)

 AbstractPlatformTransactionManager中rollback源码如下：



```
1     public final void rollback(TransactionStatus status) throws TransactionException {
2         if (status.isCompleted()) {
3             throw new IllegalTransactionStateException(
4                     "Transaction is already completed - do not call commit or rollback more than once per transaction");
5         }
6 
7         DefaultTransactionStatus defStatus = (DefaultTransactionStatus) status;
8         processRollback(defStatus);
9     }
```



 processRollback源码如下：



```
 1     private void processRollback(DefaultTransactionStatus status) {
 2         try {
 3             try {// 解绑当前线程绑定的会话工厂，并关闭会话
 4                 triggerBeforeCompletion(status);
 5                 if (status.hasSavepoint()) {// 1.如果有保存点，即嵌套式事务
 6                     if (status.isDebug()) {
 7                         logger.debug("Rolling back transaction to savepoint");
 8                     }//回滚到保存点
 9                     status.rollbackToHeldSavepoint();
10                 }//2.如果就是一个简单事务
11                 else if (status.isNewTransaction()) {
12                     if (status.isDebug()) {
13                         logger.debug("Initiating transaction rollback");
14                     }//回滚核心方法
15                     doRollback(status);
16                 }//3.当前存在事务且没有保存点，即加入当前事务的
17                 else if (status.hasTransaction()) {//如果已经标记为回滚 或 当加入事务失败时全局回滚（默认true）
18                     if (status.isLocalRollbackOnly() || isGlobalRollbackOnParticipationFailure()) {
19                         if (status.isDebug()) {//debug时会打印：加入事务失败-标记已存在事务为回滚
20                             logger.debug("Participating transaction failed - marking existing transaction as rollback-only");
21                         }//设置当前connectionHolder：当加入一个已存在事务时回滚
22                         doSetRollbackOnly(status);
23                     }
24                     else {
25                         if (status.isDebug()) {
26                             logger.debug("Participating transaction failed - letting transaction originator decide on rollback");
27                         }
28                     }
29                 }
30                 else {
31                     logger.debug("Should roll back transaction but cannot - no transaction available");
32                 }
33             }
34             catch (RuntimeException ex) {//关闭会话，重置SqlSessionHolder属性
35                 triggerAfterCompletion(status, TransactionSynchronization.STATUS_UNKNOWN);
36                 throw ex;
37             }
38             catch (Error err) {
39                 triggerAfterCompletion(status, TransactionSynchronization.STATUS_UNKNOWN);
40                 throw err;
41             }
42             triggerAfterCompletion(status, TransactionSynchronization.STATUS_ROLLED_BACK);
43         }
44         finally {、、解绑当前线程
45             cleanupAfterCompletion(status);
46         }
47     }
```



如上图，有几个公共方法和提交事务时一致，就不再重复。

这里主要看doRollback，DataSourceTransactionManager的doRollback()源码如下：



```
 1 protected void doRollback(DefaultTransactionStatus status) {
 2         DataSourceTransactionObject txObject = (DataSourceTransactionObject) status.getTransaction();
 3         Connection con = txObject.getConnectionHolder().getConnection();
 4         if (status.isDebug()) {
 5             logger.debug("Rolling back JDBC transaction on Connection [" + con + "]");
 6         }
 7         try {
 8             con.rollback();
 9         }
10         catch (SQLException ex) {
11             throw new TransactionSystemException("Could not roll back JDBC transaction", ex);
12         }
13     }
```



好吧，一点不复杂，就是Connection的rollback.

##  四、时序图

特地整理了时序图（简单的新事务，没有画出保存点等情况）如下：

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210527112456.jpeg)

 

===========参考========

《Spring实战4》第四章 面向切面的Spring　　

[SpringBoot事务注解@Transactional](https://www.cnblogs.com/jpfss/p/9151313.html)

# 总结

## 一、概念

事务的概念很多，只有对整体有一个把控，才能见微知著。比如一上来直接问REQUIRED，你一定很懵，但了解了大致关系后，就很清晰：Spring事务定义了六大属性-》其中一个属性是传播机制-》REQUIRED是其中一个，默认的传播机制。梳理出来三张图，如下：



### 1.1 框架概览

对于数据库事务，国际性认可的标准是ACID，即**原子性、一致性、隔离性、持久性**。Spring作为一个支持数据库持久化的框架，定义了六大属性来实现这四大特性。属性分别是：**事务名称、隔离级别（4种）、超时时间、是否只读、传播机制（7种）、回滚机制**。用户使用时定义其中的一种或几种，再结合Spring对底层数据库的驱动，即可实现ACID良好的数据库操作了。持久化层Spring提供了对Spring -Mybatis的很好的支持，可以轻松对接JDBC事务如下图：

 ![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210527112508.png)



### 1.2 重点属性

#### 1. 四大隔离级别对应的可能出现的问题如下图：

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210527112512.png)

***\*Read Uncommitted\** < \**Read Committed\** < \**Repeatable Read\** < \**Serializable. 从左往右：一致性越来越强，性能越来越低。\****

#### 官方建议：

Repeatable Read（一致性3星，性能2星）> Read Committed（一致性2星，性能3星） > Read Uncommitted（一致性1星，性能4星） > Serializable（一致性4星，性能1星）。

#### 我的建议：

**高并发场景使用Read Committed，低并发场景使用Repeatable Read。**其它两个没用过（Read Uncommitted一致性太低了；Serializable sql序列化，数据库会成为瓶颈）。

 

#### 2. 七大传播机制概念图如下：

![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210527112516.png)


## 二、源码风格

这里我们主要看Juergen Hoeller的源码风格，发现他很好的遵循了Spring的一贯风格：

1.预留方法给customer自己定义扩充，Spring一贯喜欢预留各种后门...

2.利用各种设计模式：

1）template模板模式：构造事务模板，编程式事务源码

2）代理模式：生成增强代理类，声明式事务源码

\3. Spring-AOP MethodInterceptor方法拦截器：申明式事务中使用TransactionInterceptor事务拦截器，该类实现了MethodInterceptor接口，继承自Interceptor接口，最终在业务方法invoke()前后进行增强。

\4. 面向接口编程，高度抽象：PlatformTransactionManager接口定义getTransaction、commit、rollback。AbstractPlatformTransactionManager抽象类实现通用的获取事务、提交事务、回滚事务。DataSourceTransactionManager继承抽象类，并实现特性接口。

当然看过spring transaction包，除了Spring的一贯风格外，最大的体会是注释丰富，日志丰富（尤其是debug，甚至不需要debug光看日志就可以清晰知道核心流程）。勉之。


## 三、应用

本系列针对Spring+Mybatis+Mysql,讲解了2种典型的事务的实现方案（数据库事务采用DataSourceTransactionManager来管理事务，支持JDBC驱动,通过Connection和数据库进行交互。）：

1.编程式事务

2.申明式事务

并测试了4种隔离级别和7种传播机制。相信通过实测，大家对spring 事务的使用有一定的掌握。


## 四、不足

本系列只讲解了单数据库的事务，后续会尝试分析分布式事务的实测。