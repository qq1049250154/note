下面内容转载自：

　　http://blog.csdn.net/xieyuooo/article/details/8607220

　　其实就Timer来讲就是一个调度器,而TimerTask呢只是一个实现了run方法的一个类,而具体的TimerTask需要由你自己来实现,例如这样:

```
Timer timer = ``new` `Timer();``timer.schedule(``new` `TimerTask() {``    ``public` `void` `run() {``      ``System.out.println(``"abc"``);``    ``}``}, ``200000` `, ``1000``);
```

　　这里直接实现一个TimerTask(当然，你可以实现多个TimerTask，多个TimerTask可以被一个Timer会被分配到多个Timer中被调度，后面会说到Timer的实现机制就是说内部的调度机制)，然后编写run方法，20s后开始执行，每秒执行一次，当然你通过一个timer对象来操作多个timerTask，其实timerTask本身没什么意义，只是和timer集合操作的一个对象，实现它就必然有对应的run方法，以被调用，他甚至于根本不需要实现Runnable，因为这样往往混淆视听了，为什么呢？也是本文要说的重点。

　　在说到timer的原理时，我们先看看Timer里面的一些常见方法：

```
public` `void` `schedule(TimerTask task, ``long` `delay)
```

　　这个方法是调度一个task，经过delay(ms)后开始进行调度，仅仅调度一次。

```
public` `void` `schedule(TimerTask task, Date time)
```

　　在指定的时间点time上调度一次。

```
public` `void` `schedule(TimerTask task, ``long` `delay, ``long` `period)
```

 　这个方法是调度一个task，在delay（ms）后开始调度，每次调度完后，最少等待period（ms）后才开始调度。

```
public` `void` `schedule(TimerTask task, Date firstTime, ``long` `period)
```

　　和上一个方法类似，唯一的区别就是传入的第二个参数为第一次调度的时间。

```
public` `void` `scheduleAtFixedRate(TimerTask task, ``long` `delay, ``long` `period)
```

 　调度一个task，在delay(ms)后开始调度，然后每经过period(ms)再次调度，**貌似和方法：schedule是一样的，其实不然**，后面你会根据源码看到，schedule在计算下一次执行的时间的时候，是通过当前时间（在任务执行前得到） + 时间片，而**scheduleAtFixedRate**方法是通过当前需要执行的时间（也就是计算出现在应该执行的时间）+ 时间片，前者是运行的实际时间，而后者是理论时间点，例如：**schedule**时间片是5s，那么理论上会在**5、10、15、20**这些时间片被调度，但是如果由于某些CPU征用导致未被调度，假如等到第8s才被第一次调度，那么**schedule**方法计算出来的下一次时间应该是第13s而不是第10s，这样有可能下次就越到20s后而**被少调度一次或多次**，而**scheduleAtFixedRate**方法就是每次理论计算出下一次需要调度的时间用以排序，若第8s被调度，那么计算出应该是第10s，所以它距离当前时间是2s，那么再调度队列排序中，会被优先调度，那么就**尽量减少漏掉调度**的情况。

```
public` `void` `scheduleAtFixedRate(TimerTask task, Date firstTime,``long` `period)
```

　　方法同上，唯一的区别就是第一次调度时间设置为一个Date时间，而不是当前时间的一个时间片，我们在源码中会详细说明这些内容。

　　接下来看源码

　　首先看Timer的构造方法有几种：

　　构造方法1：无参构造方法，简单通过Tiemer为前缀构造一个线程名称：

```
public` `Timer() {``  ``this``(``"Timer-"` `+ serialNumber());``}
```

 　创建的线程不为主线程，则主线程结束后，timer自动结束，而无需使用cancel来完成对timer的结束。

　　构造方法2：传入了是否为后台线程，后台线程当且仅当进程结束时，自动注销掉。

```
public` `Timer(``boolean` `isDaemon) {``  ``this``(``"Timer-"` `+ serialNumber(), isDaemon);``}
```

 　另外两个构造方法负责传入名称和将timer启动：

```
public` `Timer(String name, ``boolean` `isDaemon) {``   ``thread.setName(name);``   ``thread.setDaemon(isDaemon);``   ``thread.start();`` ``}
```

 　这里有一个thread，这个thread很明显是一个线程，被包装在了Timer类中，我们看下这个thread的定义是：

```
private` `TimerThread thread = ``new` `TimerThread(queue);
```

　　而定义TimerThread部分的是：

```
class` `TimerThread ``extends` `Thread {
```

 　看到这里知道了，Timer内部包装了一个线程，用来做独立于外部线程的调度，而TimerThread是一个default类型的，默认情况下是引用不到的，是被Timer自己所使用的。

　　**接下来看下有那些属性**

　　除了上面提到的thread，还有一个很重要的属性是：

```
private` `TaskQueue queue = ``new` `TaskQueue();
```

 　看名字就知道是一个队列，队列里面可以先猜猜看是什么，那么大概应该是我要调度的任务吧，先记录下了，接下来继续向下看：

里面还有一个属性是：threadReaper，它是Object类型，只是重写了finalize方法而已，是为了垃圾回收的时候，将相应的信息回收掉，做GC的回补，也就是当timer线程由于某种原因死掉了，而未被cancel，里面的队列中的信息需要清空掉，不过我们通常是不会考虑这个方法的，所以知道java写这个方法是干什么的就行了。

　　**接下来看调度方法的实现：**

　　对于上面6个调度方法，我们不做一一列举，为什么等下你就知道了：

　　来看下方法：

```
public` `void` `schedule(TimerTask task, ``long` `delay)
```

　　的源码如下：

```
public` `void` `schedule(TimerTask task, ``long` `delay) {``    ``if` `(delay < ``0``)``      ``throw` `new` `IllegalArgumentException(``"Negative delay."``);``    ``sched(task, System.currentTimeMillis()+delay, ``0``);``  ``}
```

　　这里调用了另一个方法，将task传入，第一个参数传入System.currentTimeMillis()+delay可见为第一次需要执行的时间的时间点了（如果传入Date，就是对象.getTime()即可，所以传入Date的几个方法就不用多说了），而第三个参数传入了0，这里可以猜下要么是时间片，要么是次数啥的，不过等会就知道是什么了；另外关于方法：sched的内容我们不着急去看他，先看下重载的方法中是如何做的

　　再看看方法：

```
public` `void` `schedule(TimerTask task, ``long` `delay,``long` `period)
```

　　源码为：

```
public` `void` `schedule(TimerTask task, ``long` `delay, ``long` `period) {``    ``if` `(delay < ``0``)``      ``throw` `new` `IllegalArgumentException(``"Negative delay."``);``    ``if` `(period <= ``0``)``      ``throw` `new` `IllegalArgumentException(``"Non-positive period."``);``    ``sched(task, System.currentTimeMillis()+delay, -period);``  ``}
```

　　看来也调用了方法sched来完成调度，和上面的方法唯一的调度时候的区别是增加了传入的period，而第一个传入的是0，所以确定这个参数为时间片，而不是次数，注意这个里的period加了一个负数，也就是取反，也就是我们开始传入1000，在调用sched的时候会变成-1000，其实最终阅读完源码后你会发现这个算是老外对于一种数字的理解，而并非有什么特殊的意义，所以阅读源码的时候也有这些困难所在。

　　最后再看个方法是：

```
public` `void` `scheduleAtFixedRate(TimerTasktask,``long` `delay,``long` `period)
```

　　源码为：

```
public` `void` `scheduleAtFixedRate(TimerTask task, ``long` `delay, ``long` `period) {``    ``if` `(delay < ``0``)``      ``throw` `new` `IllegalArgumentException(``"Negative delay."``);``    ``if` `(period <= ``0``)``      ``throw` `new` `IllegalArgumentException(``"Non-positive period."``);``    ``sched(task, System.currentTimeMillis()+delay, period);``  ``}
```

 　唯一的区别就是在period没有取反，其实你最终阅读完源码，上面的取反没有什么特殊的意义，老外不想增加一个参数来表示scheduleAtFixedRate，而scheduleAtFixedRate和schedule的大部分逻辑代码一致，因此用了参数的范围来作为区分方法，也就是当你传入的参数不是正数的时候，你调用schedule方法正好是得到scheduleAtFixedRate的功能，而调用scheduleAtFixedRate方法的时候得到的正好是schedule方法的功能，呵呵，这些讨论没什么意义，讨论实质和重点：

 　来看sched方法的实现体：

```
private` `void` `sched(TimerTask task, ``long` `time, ``long` `period) {``    ``if` `(time < ``0``)``      ``throw` `new` `IllegalArgumentException(``"Illegal execution time."``);` `    ``synchronized``(queue) {``      ``if` `(!thread.newTasksMayBeScheduled)``        ``throw` `new` `IllegalStateException(``"Timer already cancelled."``);` `      ``synchronized``(task.lock) {``        ``if` `(task.state != TimerTask.VIRGIN)``          ``throw` `new` `IllegalStateException(``            ``"Task already scheduled or cancelled"``);``        ``task.nextExecutionTime = time;``        ``task.period = period;``        ``task.state = TimerTask.SCHEDULED;``      ``}` `      ``queue.add(task);``      ``if` `(queue.getMin() == task)``        ``queue.notify();``    ``}``  ``}
```

　　queue为一个队列，我们先不看他数据结构，看到他在做这个操作的时候，发生了同步，所以在timer级别，这个是线程安全的，最后将task相关的参数赋值，主要包含**nextExecutionTime**（下一次执行时间），period（时间片），state（状态），然后将它放入queue队列中，做一次notify操作，为什么要做notify操作呢？看了后面的代码你就知道了。

 

　　简言之，这里就是讲task放入队列queue的过程，此时，你可能对queue的结构有些兴趣，那么我们先来看看queue属性的结构TaskQueue：

```
class` `TaskQueue {` `  ``private` `TimerTask[] queue = ``new` `TimerTask[``128``];` `  ``private` `int` `size = ``0``;
```

 　可见，TaskQueue的结构很简单，为一个数组，加一个size，有点像ArrayList，是不是长度就128呢，当然不是，ArrayList可以扩容，它可以，只是会造成内存拷贝而已，所以一个Timer来讲，只要内部的task个数不超过128是不会造成扩容的；内部提供了add(TimerTask)、size()、getMin()、get(int)、removeMin()、quickRemove(int)、rescheduleMin(long newTime)、isEmpty()、clear()、fixUp()、fixDown()、heapify()；

　　这里面的方法大概意思是：

　　**add(TimerTaskt)**为增加一个任务

　　**size()**任务队列的长度

　　**getMin()**获取当前排序后最近需要执行的一个任务，下标为1，队列头部0是不做任何操作的。

　　**get(inti)**获取指定下标的数据，当然包括下标0.

　　**removeMin()**为删除当前最近执行的任务，也就是第一个元素，通常只调度一次的任务，在执行完后，调用此方法，就可以将TimerTask从队列中移除。

　　**quickRmove(inti)**删除指定的元素，一般来说是不会调用这个方法的，这个方法只有在Timer发生purge的时候，并且当对应的**TimerTask**调用了**cancel**方法的时候，才会被调用这个方法，也就是取消某个**TimerTask**，然后就会从队列中移除（注意如果任务在执行中是，还是仍然在执行中的，虽然在队列中被移除了），还有就是这个cancel方法并不是Timer的cancel方法而是TimerTask，一个是调度器的，一个是单个任务的，最后注意，这个quickRmove完成后，是将队列最后一个元素补充到这个位置，所以此时会造成顺序不一致的问题，后面会有方法进行回补。

　　**rescheduleMin**(long newTime)是重新设置当前执行的任务的下一次执行时间，并在队列中将其从新排序到合适的位置，而调用的是后面说的**fixDown**方法。

　　对于**fixUp**和**fixDown**方法来讲，前者是当新增一个task的时候，首先将元素放在队列的尾部，然后向前找是否有比自己还要晚执行的任务，如果有，就将两个任务的顺序进行交换一下。而fixDown正好相反，执行完第一个任务后，需要加上一个时间片得到下一次执行时间，从而需要将其顺序与后面的任务进行对比下。

　　其次可以看下**fixDown**的细节为：

```
private` `void` `fixDown(``int` `k) {``    ``int` `j;``    ``while` `((j = k << ``1``) <= size && j > ``0``) {``      ``if` `(j < size &&``        ``queue[j].nextExecutionTime > queue[j+``1``].nextExecutionTime)``        ``j++; ``// j indexes smallest kid``      ``if` `(queue[k].nextExecutionTime <= queue[j].nextExecutionTime)``        ``break``;``      ``TimerTask tmp = queue[j]; queue[j] = queue[k]; queue[k] = tmp;``      ``k = j;``    ``}``  ``}
```

 　这种方式并非排序，而是找到一个合适的位置来交换，因为并不是通过队列逐个找的，而是每次移动一个二进制为，例如传入1的时候，接下来就是2、4、8、16这些位置，找到合适的位置放下即可，顺序未必是完全有序的，它只需要看到距离调度部分的越近的是有序性越强的时候就可以了，这样即可以保证一定的顺序性，达到较好的性能。

　　最后一个方法是**heapify**，其实就是将队列的后半截，全部做一次**fixeDown**的操作，这个操作主要是为了回补**quickRemove**方法，当大量的quickRmove后，顺序被打乱后，此时将一半的区域做一次非常简单的排序即可。

　　这些方法我们不在说源码了，只需要知道它提供了类似于ArrayList的东西来管理，内部有很多排序之类的处理，我们继续回到Timer，里面还有两个方法是：cancel()和方法purge()方法，其实就cancel方法来讲，一个取消操作，在测试中你会发现，如果一旦执行了这个方法timer就会结束掉，看下源码是什么呢：

```
public` `void` `cancel() {``    ``synchronized``(queue) {``      ``thread.newTasksMayBeScheduled = ``false``;``      ``queue.clear();``      ``queue.notify(); ``// In case queue was already empty.``    ``}``  ``}
```

 　貌似仅仅将队列清空掉，然后设置了newTasksMayBeScheduled状态为false，最后让队列也调用了下notify操作，但是没有任何地方让线程结束掉，那么就要回到我们开始说的Timer中包含的thread为：TimerThread类了，在看这个类之前，再看下Timer中最后一个purge()类，当你对很多Task做了cancel操作后，此时通过调用purge方法实现对这些cancel掉的类空间的回收，上面已经提到，此时会造成顺序混乱，所以需要调用队里的heapify方法来完成顺序的重排，源码如下：

```
public` `int` `purge() {``     ``int` `result = ``0``;` `     ``synchronized``(queue) {``       ``for` `(``int` `i = queue.size(); i > ``0``; i--) {``         ``if` `(queue.get(i).state == TimerTask.CANCELLED) {``           ``queue.quickRemove(i);``           ``result++;``         ``}``       ``}` `       ``if` `(result != ``0``)``         ``queue.heapify();``     ``}``     ``return` `result;``   ``}
```

　　那么调度呢，是如何调度的呢，那些notify，和清空队列是如何做到的呢？我们就要看看TimerThread类了，内部有一个属性是：newTasksMayBeScheduled，也就是我们开始所提及的那个参数在cancel的时候会被设置为false。

　　另一个属性定义了

```
private` `TaskQueue queue;
```

 　也就是我们所调用的queue了，这下联通了吧，不过这里是queue是通过构造方法传入的，传入后赋值用以操作，很明显是Timer传递给这个线程的，我们知道它是一个线程，所以执行的中心自然是run方法了，所以看下run方法的body部分是：

```
public` `void` `run() {``    ``try` `{``      ``mainLoop();``    ``} ``finally` `{``      ``synchronized``(queue) {``        ``newTasksMayBeScheduled = ``false``;``        ``queue.clear(); ``// Eliminate obsolete references``      ``}``    ``}``  ``}
```

　　try很简单，就一个mainLoop，看名字知道是主循环程序，finally中也就是必然执行的程序为将参数为为false，并将队列清空掉。

那么最核心的就是mainLoop了，是的，看懂了mainLoop一切都懂了：

```
private` `void` `mainLoop() {``    ``while` `(``true``) {``      ``try` `{``        ``TimerTask task;``        ``boolean` `taskFired;``        ``synchronized``(queue) {``          ``// Wait for queue to become non-empty``          ``while` `(queue.isEmpty() && newTasksMayBeScheduled)``            ``queue.wait();``          ``if` `(queue.isEmpty())``            ``break``; ``// Queue is empty and will forever remain; die` `          ``// Queue nonempty; look at first evt and do the right thing``          ``long` `currentTime, executionTime;``          ``task = queue.getMin();``          ``synchronized``(task.lock) {``            ``if` `(task.state == TimerTask.CANCELLED) {``              ``queue.removeMin();``              ``continue``; ``// No action required, poll queue again``            ``}``            ``currentTime = System.currentTimeMillis();``            ``executionTime = task.nextExecutionTime;``            ``if` `(taskFired = (executionTime<=currentTime)) {``              ``if` `(task.period == ``0``) { ``// Non-repeating, remove``                ``queue.removeMin();``                ``task.state = TimerTask.EXECUTED;``              ``} ``else` `{ ``// Repeating task, reschedule``                ``queue.rescheduleMin(``                 ``task.period<``0` `? currentTime  - task.period``                        ``: executionTime + task.period);``              ``}``            ``}``          ``}``          ``if` `(!taskFired) ``// Task hasn't yet fired; wait``            ``queue.wait(executionTime - currentTime);``        ``}``        ``if` `(taskFired) ``// Task fired; run it, holding no locks``          ``task.run();``      ``} ``catch``(InterruptedException e) {``      ``}``    ``}``  ``}
```

 　可以发现这个timer是一个死循环程序，除非遇到不能捕获的异常或break才会跳出，首先注意这段代码：

```
while` `(queue.isEmpty() &&newTasksMayBeScheduled)``            ``queue.wait();
```

　　循环体为循环过程中，条件为queue为空且newTasksMayBeScheduled状态为true，可以看到这个状态其关键作用，也就是跳出循环的条件就是要么队列不为空，要么是newTasksMayBeScheduled状态设置为false才会跳出，而wait就是在等待其他地方对queue发生notify操作，从上面的代码中可以发现，当发生add、cancel以及在threadReaper调用finalize方法的时候会被调用，第三个我们基本可以不考虑其实发生add的时候也就是当队列还是空的时候，发生add使得队列不为空就跳出循环，而cancel是设置了状态，否则不会进入这个循环，那么看下面的代码：

```
if` `(queue.isEmpty())``  ``break``;
```

 　当跳出上面的循环后，如果是设置了newTasksMayBeScheduled状态为false跳出，也就是调用了cancel，那么queue就是空的，此时就直接跳出外部的死循环，所以cancel就是这样实现的，如果下面的任务还在跑还没运行到这里来，cancel是不起作用的。

　　接下来是获取一个当前系统时间和上次预计的执行时间，如果预计执行的时间小于当前系统时间，那么就需要执行，此时判定时间片是否为0，如果为0，则调用removeMin方法将其移除，否则将task通过rescheduleMin设置最新时间并排序：

```
currentTime = System.currentTimeMillis();``executionTime = task.nextExecutionTime;``if` `(taskFired = (executionTime<=currentTime)) {``   ``if` `(task.period == ``0``) { ``// Non-repeating, remove``      ``queue.removeMin();``      ``task.state = TimerTask.EXECUTED;``   ``} ``else` `{ ``// Repeating task, reschedule``      ``queue.rescheduleMin(``      ``task.period<``0` `? currentTime  - task.period``               ``: executionTime + task.period);``   ``}``}
```

 　这里可以看到，period为负数的时候，就会被认为是按照按照当前系统时间+一个时间片来计算下一次时间，就是前面说的schedule和scheduleAtFixedRate的区别了，其实内部是通过正负数来判定的，也许java是不想增加参数，而又想增加程序的可读性，才这样做，其实通过正负判定是有些诡异的，也就是你如果在schedule方法传入负数达到的功能和scheduleAtFixedRate的功能是一样的，相反在scheduleAtFixedRate方法中传入负数功能和schedule方法是一样的。

　　同时你可以看到period为0，就是只执行一次，所以时间片正负0都用上了，呵呵，然后再看看mainLoop接下来的部分：

```
if` `(!taskFired)``// Taskhasn't yet fired; wait``  ``queue.wait(executionTime- currentTime);
```

 　这里是如果任务执行时间还未到，就等待一段时间，当然这个等待很可能会被其他的线程操作add和cancel的时候被唤醒，因为内部有notify方法，所以这个时间并不是完全准确，在这里大多数情况下是考虑Timer内部的task信息是稳定的，cancel方法唤醒的话是另一回事。

　　最后：

```
if` `(taskFired) ``// Task fired; run it, holding no locks``  ``task.run();
```

 　如果线程需要执行，那么调用它的run方法，而并非启动一个新的线程或从线程池中获取一个线程来执行，所以TimerTask的run方法并不是多线程的run方法，虽然实现了Runnable，但是仅仅是为了表示它是可执行的，并不代表它必须通过线程的方式来执行的。

 

　　**回过头来再看看：**

　　**Timer**和**TimerTask**的简单组合是多线程的嘛？不是，一个Timer内部包装了“**一个Thread”**和“**一个Task”**队列，这个队列按照一定的方式将任务排队处理，包含的线程在**Timer**的构造方法调用时被启动，这个Thread的run方法无限循环这个Task队列，若队列为空且没发生**cancel**操作，此时会一直等待，如果等待完成后，队列还是为空，则认为发生了cancel从而跳出死循环，结束任务；循环中如果发现任务需要执行的时间小于系统时间，则需要执行，那么根据任务的时间片从新计算下次执行时间，若时间片为0代表只执行一次，则直接移除队列即可。

　　但是是否能实现多线程呢？可以，任何东西是否是多线程完全看个人意愿，多个Timer自然就是多线程的，每个Timer都有自己的线程处理逻辑，当然Timer从这里来看并不是很适合很多任务在短时间内的快速调度，至少不是很适合同一个timer上挂很多任务，在多线程的领域中我们更多是使用多线程中的：

```
Executors.newScheduledThreadPool
```

 来完成对调度队列中的线程池的处理，内部通过***new\* ScheduledThreadPoolExecutor**来创建线程池的**Executor**的创建，当然也可以调用：

```
Executors.unconfigurableScheduledExecutorService
```

 　方法来创建一个**DelegatedScheduledExecutorService**其实这个类就是包装了下下**scheduleExecutor**，也就是这只是一个壳，英文理解就是被委派的意思，被托管的意思。

　　

　　具体的使用例子可以参考这篇博文：

　　http://www.bdqn.cn/news/201305/9303.shtml