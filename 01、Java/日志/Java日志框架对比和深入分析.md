## 前言

作为一名资深的开发人员，对于日志记录框架一定不会很陌生。而且几乎在所有应用里面，一定会用到各种各样的

日志框架用来记录程序的运行信息。而对于一个成熟的java应用，这个是必不可少的。在开发和调试阶段，日志可以帮助我们更快的定位问题；而在应用的运维过程中，日志系统又可以帮助我们记录大部分的异常信息，通常很多企业会通过收集日志信息来对系统的运行状态进行实时监控预警。那么，你对日志框架到底有多了解呢？

### 常用的日志框架

**Log4j**

Log4j是apache下一个功能非常丰富的java日志库实现，Log4j应该是出现比较早而且最受欢迎的java日志组

件，它是基于java的开源的日志组件。Log4j的功能非常强大，通过Log4j可以把日志输出到控制台、文件、用户界面。也可以输出到操作系统的事件记录器和一些系统常驻进程。值得一提的是：Log4j可以允许你非常便捷地自定义日志格式和日志等级，可以帮助开发人员全方位的掌控自己的日志信息

**Log4j2**

Log4j2是Log4j1的升级版本。Log4j2基本上把Log4j版本的核心全部重构掉了，而且基于Log4j做了很多优化和改变

**Logback**

Logback是由Log4j创始人设计的另一个开源日志组件，也是作为Log4j的替代者出现的。而且官方是建议和

Slf4j一起使用，你们一定不知道Logback、slf4j、Log4j都是出自同一个人吧。 Logback是在Log4j的基础上做的改进版本，而Slf4j又是同一个人设计的，所以默认就对Slf4j无缝结合。

**JDK-Logging**

Jdk1.4版本以后开始提供的一个自带的日志库实现

**统一日志模块**

目前市面上有两个用得比较广泛的统一日志规范接口，分别是

**SLF4j**

SLF4j（Simple Logging Facade For Java）是基于API的java日志框架，SLF4j提供了一个简单统一的日

志记录接口，开发者在配置和部署时，只需要实现这个接口就可以实现日志功能。可以说，它并不是一个具体的日志解决方案，它只是服务于各种各样的日志系统，允许最终用户在部署应用上使用自己常用的日志系统

**Commons-Logging**

Common-logging 为众多具体的日志实现库提供了一个统一的接口，和SLF4j的作用类似，它允许在运行时绑定任意的日志库；

这里其实有个小故事，当年apache说服Log4j以及其他的日志框架按照Commons-Logging的标准来编写，但是由于Commons-Logging的类加载有点问题，实现起来不友好。因此Log4j的作者就创作了Slf4j，也因此与Commons-Logging两份天下

图说几个日志框架的关系



![img](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517172037.jpeg)



各个日志的功能演示

各个日志模块的功能演示和配置说明，我就不做多说了，网上搜索下一抓一大把，都讲得很详细。

slf4j和各个日志框架集成的原理

这里要重点说明一个东西，就是slf4j通过一个非常有趣而且很牛的设计，把各个日志框架去集成进来。

如果你们去看slf4j的源码，在LoggerFactory.java里面有一个这样的静态全局变量

```text
private static String STATIC_LOGGER_BINDER_PATH = "org/slf4j/impl/StaticLoggerBinder.class";
```

还有一段核心代码

```text
static Set<URL> findPossibleStaticLoggerBinderPathSet() {
    LinkedHashSet staticLoggerBinderPathSet = new LinkedHashSet();
    try {
        ClassLoader ioe = LoggerFactory.class.getClassLoader();
        Enumeration paths;
        if(ioe == null) {
            paths = ClassLoader.getSystemResources (STATIC_LOGGER_BINDER_PATH);
        } else {
            paths = ioe.getResources (STATIC_LOGGER_BINDER_PATH);
        }
        while(paths.hasMoreElements()) {
            URL path = (URL)paths.nextElement();
            staticLoggerBinderPathSet.add(path);
        }
    }
    catch (IOException var4) {
        Util.report ("Error getting resources from path", var4);
    }
    return staticLoggerBinderPathSet;
}
```

大家对ClassLoader机制了解的同学，这段代码看起来就非常容易懂了，通过ClassLoader去加载classpath下所有存在StaticLoggerBinder.class的文件。找到这个文件以后加到一个集合里面。通过加载到对应jar中的StaticLoggerBinder。来获取实例。

```text
public static ILoggerFactory getILoggerFactory() {
    if (INITIALIZATION_STATE == UNINITIALIZED) {
        synchronized (LoggerFactory.class) {
            if (INITIALIZATION_STATE == UNINITIALIZED) {
                INITIALIZATION_STATE = ONGOING_INITIALIZATION;
                performInitialization();
            }
        }
    }
    switch (INITIALIZATION_STATE) {
        case SUCCESSFUL_INITIALIZATION:
         return StaticLoggerBinder.getSingleton().getLoggerFactory();
        case NOP_FALLBACK_INITIALIZATION:
         return NOP_FALLBACK_FACTORY;
        case FAILED_INITIALIZATION:
         throw new IllegalStateException (UNSUCCESSFUL_INIT_MSG);
        case ONGOING_INITIALIZATION:
         // support re-entrant behavior.
        // See also http://jira.qos.ch/browse/SLF4J-97
        return SUBST_FACTORY;
    }
    throw new IllegalStateException ("Unreachable code");
}
```

在这段代码里面，可以看到有一个StaticLoggerBinder.getSingleton().getLoggerFactory()

这个就是在第三方的集成包中返回的实例。这个就是slf4j里面比较核心的一块

总结

以上简单的把java中常用的日志框架做了梳理

https://zhuanlan.zhihu.com/p/65737854