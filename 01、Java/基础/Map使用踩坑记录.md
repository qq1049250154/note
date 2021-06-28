我们提到几个比较容易踩坑的点。作为 List 集合好兄弟 Map，我们也是天天都在使用，一不小心也会踩坑。

今天我就来总结这些常见的坑，再捞自己一手，防止后续同学再继续踩坑。

本文设计知识点如下：

![image-20210320131515515](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210320131515515.png)

image

## 不是所有的 Map 都能包含  null

这个踩坑经历还是发生在实习的时候，那时候有这样一段业务代码，功能很简单，从 XML 中读取相关配置，存入 Map 中。

代码示例如下：

<img src="https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210320131523060.png" alt="image-20210320131523060" style="zoom:67%;" />

image

那时候正好有个小需求，需要改动一下这段业务代码。改动的过程中，突然想到 `HashMap` 并发过程可能导致死锁的问题。

于是改动了一下这段代码，将 `HashMap` 修改成了 `ConcurrentHashMap`。

美滋滋提交了代码，然后当天上线的时候，就发现炸了。。。

应用启动过程发生 **NPE** 问题，导致应用启动失败。

![image-20210320131533784](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210320131533784.png)

image

根据异常日志，很快就定位到了问题原因。由于 XML 某一项配置问题，导致读取元素为 null，然后元素置入到 `ConcurrentHashMap` 中，抛出了空指针异常。

这不科学啊！之前 `HashMap` 都没问题，都可以存在 **null**，为什么它老弟 `ConcurrentHashMap` 就不可以？

![image-20210320131540099](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210320131540099.png)

image

翻阅了一下 `ConcurrentHashMap#put` 方法的源码，开头就看到了对 KV 的判空校验。

![image-20210320131547124](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210320131547124.png)

image

看到这里，不知道你有没有疑惑，为什么 `ConcurrentHashMap` 与 `HashMap` 设计的判断逻辑不一样？

求助了下万能的 Google，找到 **Doug Lea** 老爷子的回答：

![image-20210320131604250](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210320131604250.png)

image

<figcaption style="margin: 5px 0px 0px; padding: 0px; max-width: 100%; box-sizing: border-box !important; word-wrap: break-word !important; text-align: center; color: rgb(136, 136, 136); font-size: 12px; font-family: PingFangSC-Light; overflow-wrap: break-word !important;">来源:[http://cs.oswego.edu/pipermail/concurrency-interest/2006-May/002485.html](https://links.jianshu.com/go?to=http%3A%2F%2Fcs.oswego.edu%2Fpipermail%2Fconcurrency-interest%2F2006-May%2F002485.html)</figcaption>

总结一下：

- null 会引起歧义，如果 value 为 null，我们无法得知是值为 null，还是 key 未映射具体值？
- **Doug Lea** 并不喜欢 null，认为 null 就是个隐藏的炸弹。

上面提到 **Josh Bloch** 正是 `HashMap` 作者，他与 **Doug Lea** 在 null 问题意见并不一致。

也许正是因为这些原因，从而导致 `ConcurrentHashMap` 与 `HashMap` 对于 null 处理并不一样。

最后贴一下常用 Map 子类集合对于 null 存储情况：

![image-20210320131616552](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210320131616552.png)

image

上面的实现类约束，都太不一样，有点不好记忆。其实只要我们在加入元素之前，主动去做空指针判断，不要在 Map 中存入 null，就可以从容避免上面问题。

## 自定义对象为 key

先来看个简单的例子，我们自定义一个 `Goods` 商品类，将其作为 Key 存在 Map 中。

示例代码如下：

![image-20210320131623154](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210320131623154.png)

image

上面代码中，第二次我们加入一个相同的商品，原本我们期望新加入的值将会替换原来旧值。但是实际上这里并没有替换成功，反而又加入一对键值。

翻看一下 `HashMap#put` 的源码：

> 以下代码基于 JDK1.7

![image-20210320131629616](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210320131629616.png)

image

这里首先判断 `hashCode` 计算产生的 hash，如果相等，再判断 `equals` 的结果。但是由于 `Goods`对象未重写的`hashCode` 与 `equals` 方法，默认情况下 `hashCode` 将会使用父类对象 Object 方法逻辑。

而 `Object#hashCode` 是一个 **native** 方法，默认将会为每一个对象生成不同 **hashcode**（**与内存地址有关**），这就导致上面的情况。

所以如果需要使用自定义对象做为 Map 集合的 key，那么一定记得**重写**`hashCode` 与 `equals` 方法。

然后当你为自定义对象重写上面两个方法，接下去又可能踩坑另外一个坑。

![image-20210320131635157](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210320131635157.png)

image

> 使用 lombok 的 `EqualsAndHashCode` 自动重写 `hashCode` 与 `equals` 方法。

上面的代码中，当 Map 中置入自定义对象后，接着修改了商品金额。然后当我们想根据同一个对象取出 Map 中存的值时，却发现取不出来了。

上面的问题主要是因为 `get` 方法是根据对象 的 hashcode 计算产生的 hash 值取定位内部存储位置。

![image-20210320131644760](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210320131644760.png)

image

当我们修改了金额字段后，导致 `Goods` 对象 hashcode 产生的了变化，从而导致 get 方法无法获取到值。

通过上面两种情况，可以看到使用自定义对象作为 Map 集合 key，还是挺容易踩坑的。

所以尽量避免使用自定义对象作为 Map 集合 key，如果一定要使用，记得重写 `hashCode` 与 `equals` 方法。另外还要保证这是一个不可变对象，即对象创建之后，无法再修改里面字段值。

## 错用 ConcurrentHashMap 导致线程不安全

我们都知道 `HashMap` 其实是一个**线程不安全**的容器，多线程环境为了线程安全，我们需要使用 `ConcurrentHashMap`代替。

但是不要认为使用了 `ConcurrentHashMap` 一定就能保证线程安全，在某些错误的使用场景下，依然会造成线程不安全。

![image-20210320131650812](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210320131650812.png)

image

上面示例代码，我们原本期望输出 **1001**，但是运行几次，得到结果都是小于 **1001**。

深入分析这个问题原因，实际上是因为第一步与第二步是一个组合逻辑，不是一个原子操作。

`ConcurrentHashMap` 只能保证这两步单的操作是个原子操作，线程安全。但是并不能保证两个组合逻辑线程安全，很有可能 A 线程刚通过 get 方法取到值，还未来得及加 1，线程发生了切换，B 线程也进来取到同样的值。

这个问题同样也发生在其他线程安全的容器，比如 `Vector`等。

上面的问题解决办法也很简单，加锁就可以解决，不过这样就会使性能大打折扣，所以不太推荐。

我们可以使用 `AtomicInteger` 解决以上的问题。

![image-20210320131702472](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210320131702472.png)

image

## List 集合这些坑，Map 中也有

[List踩坑那篇文章](https://links.jianshu.com/go?to=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzU4ODI1MjA3NQ%3D%3D%26mid%3D2247485709%26idx%3D2%26sn%3D58bdfb8fb73d95f75041215de10450ba%26chksm%3Dfddedfc9caa956dfd727d57e01454465c1d3c091fdff193a98b31770f2151a74869ede05e263%26scene%3D21%23wechat_redirect)中我们提过，`Arrays#asList` 与 `List#subList` 返回 List 将会与原集合互相影响，且可能并不支持 `add` 等方法。同样的，这些坑爹的特性在 Map 中也存在，一不小心，将会再次掉坑。

![image-20210320131709459](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210320131709459.png)

image

Map 接口除了支持增删改查功能以外，还有三个特有的方法，能返回所有 key，返回所有的 value，返回所有 kv 键值对。



```bash
// 返回 key 的 set 视图Set<K> keySet()；// 返回所有 value   Collection 视图Collection<V> values();// 返回 key-value 的 set 视图Set<Map.Entry<K, V>> entrySet();
```

这三个方法创建返回新集合，底层其实都依赖的原有 Map 中数据，所以一旦 Map 中元素变动，就会同步影响返回的集合。

另外这三个方法返回新集合，是不支持的新增以及修改操作的，但是却支持 `clear、remove` 等操作。

示例代码如下：

![image-20210320131716128](https://typoralim.oss-cn-beijing.aliyuncs.com/img/image-20210320131716128.png)

image

所以如果需要对外返回 Map 这三个方法产生的集合，建议再来个套娃。



```cpp
new ArrayList<>(map.values());
```

最后再简单提一下，使用 `foreach` 方式遍历新增/删除 Map 中元素，也将会和 List 集合一样，抛出 `ConcurrentModificationException`。

## 总结

**首先**，从上面文章可以看到不管是 List 提供的方法返回集合，还是 Map 中方法返回集合，底层实际还是使用原有集合的元素，这就导致两者将会被互相影响。所以如果需要对外返回，请使用套娃大法，这样让别人用的也安心。

**第二**， Map 各个实现类对于 null 的约束都不太一样，这里建议在 Map 中加入元素之前，主动进行空指针判断，提前发现问题。

**第三**，慎用自定义对象作为 Map 中的 key，如果需要使用，一定要重写 `hashCode` 与 `equals` 方法，并且还要保证这是个不可变对象。

**第四**，`ConcurrentHashMap` 是线程安全的容器，但是不要思维定势，不要片面认为使用 `ConcurrentHashMap` 就会线程安全。

## 最后

你在使用 Map 的过程还踩过什么坑，欢迎留言讨论。



作者：JAVA进阶之道
链接：https://www.jianshu.com/p/73c2a16f3ba7
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。