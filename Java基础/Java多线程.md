<!-- TOC -->

- [计算机核心数和线程数的计算关系](#计算机核心数和线程数的计算关系)
- [Java中创建异步任务的几种方法（创建线程的方法）](#java中创建异步任务的几种方法创建线程的方法)
  - [继承Thread，创建线程](#继承thread创建线程)
  - [创建线程池，线程池自己创建线程](#创建线程池线程池自己创建线程)
- [Thread中的 run() 和 start() 的区别](#thread中的-run-和-start-的区别)
- [线程的阻塞](#线程的阻塞)
- [线程池](#线程池)
  - [Java中几种创建线程池的方法：](#java中几种创建线程池的方法)
  - [`ThreadPoolExecutor`](#threadpoolexecutor)
  - [线程池都提供了`submit()`和`execute()`方法](#线程池都提供了submit和execute方法)
  - [ThreadPoolExecutor使用的阻塞队列](#threadpoolexecutor使用的阻塞队列)
- [几种JDK提供的并发容器](#几种jdk提供的并发容器)
  - [ConcurrentHashMap: 线程安全的HashMap](#concurrenthashmap-线程安全的hashmap)
  - [CopyOnWriteArrayList](#copyonwritearraylist)
  - [ConcurrentLinkedQueue](#concurrentlinkedqueue)
  - [BlockingQueue](#blockingqueue)
- [ThreadLocal](#threadlocal)
  - [线程上下文和ThreadLocal](#线程上下文和threadlocal)
  - [ThreadLocal的内存泄漏问题](#threadlocal的内存泄漏问题)
- [补充：Java的4种引用类型（强、软、弱、虚）](#补充java的4种引用类型强软弱虚)
  - [强引用](#强引用)
  - [软引用](#软引用)
  - [弱引用](#弱引用)
  - [虚引用](#虚引用)
- [java.util.concurrent包](#javautilconcurrent包)
- [Locks-ReentrantLock、ReadWriteLock](#locks-reentrantlockreadwritelock)
  - [ReentrantLock](#reentrantlock)
    - [ReentrantLock的可重入性](#reentrantlock的可重入性)
    - [ReentrantLock-公平锁、非公平锁](#reentrantlock-公平锁非公平锁)
    - [与Condition结合进行线程的调度](#与condition结合进行线程的调度)
  - [ReadWriteLock](#readwritelock)
  - [其它常用的方法](#其它常用的方法)
- [Future、FutureTask、Callable、ComplatableFuture](#futurefuturetaskcallablecomplatablefuture)
  - [Future](#future)
  - [FutureTask](#futuretask)
  - [Callable](#callable)
  - [ComplatableFuture](#complatablefuture)

<!-- /TOC -->

为什么并发是最常见的一种提高任务处理能力的手段，归根结底---为了充分利用计算机的计算能力。那么也就带来了最常见的几种问题：  
1. 线程数和计算机的核心数之间的关系，如何均衡，如何才能发挥计算最大的计算能力
2. 并发带来的内存数据安全问题 

# 计算机核心数和线程数的计算关系
![计算机核心数和线程数的计算公式](http://sunyanping.gitee.io/it-keep/ASSET/计算机核心数和线程数的计算公式.png)    
`w/c`中的W(等待时间)---计算时间+（IO时间+...），c---计算时间，目前我在实际应用中碰到的使用多线程处理的地方都是IO比较密集的地方，比如数据库插入数据。这中就是比较耗时的操作，w/c的值就相对来说比较大。比如保存1K条数据，计算耗时5ms，IO耗时100ms，那W/C=(5+100)/5=21。如果是1核1线程的计算机就可以设置成22，如果是4核8线程可以设置成168.
当然这几公式里面的CPU都是按照1C1T来说的。

# Java中创建异步任务的几种方法（创建线程的方法）
> 在网上能看到经常能看到类似的问题：      
>问：Java中创建线程有几种方法。      
>答：继承Thread类，或者实现Runnable。

其实上面的这种回答我认为是不准确的。因为在Thread类中包含`run()`方法，继承了Thread类重写`run()`方法，`run()`就是具体的任务，再使用类似`new MyThread().start()`的方法就可以创建新线程并执行；但是实现Runnable时，重写`run()`方法，使用时需要使用`new Thread(new MyRunnable()).start()`类似的方法，这很明显就能看出来Runnable只是提供了一个任务的实现方法，并不能创建线程，再使用时还是需要使用`new Thread()`来创建线程，然后将MyRunnable()作为任务传递给这个创建的线程。所以我认为上面的说法并不准确。

我认为在Java中创建线程的方式有两种：
- 继承Thread，使用`new Thread()`的方法
- 使用线程池，线程池内部自己去创建线程


## 继承Thread，创建线程
Thread是一个类，也是实现了`Runnable`，其中包含了很多方法。所以采用这中方式也是相对于增加了开销.
- 使用线程池（一般在实际的开发中需要使用这种方式，方便对线程的统一管理）

```
public static void main(String[] args) {
    new Thread2().start();
}

static class Thread2 extends Thread {
    @Override
    public void run() {
        System.out.println("线程：" + Thread.currentThread().getName() + ", Thread2");
    }
}
```

## 创建线程池，线程池自己创建线程
```
ExecutorService executors = Executors.newFixedThreadPool(2);
```

**这里也总结一下几种创建任务的方法**：      
- 继承Thread，如上展示的方式；
- 实现Runnable，代码如下；
```
static class Thread1 implements Runnable {
    @Override
    public void run() {
        System.out.println("线程：" + Thread.currentThread().getName() + ", Thread1");
    }
}

public static void main(String[] args) {
    new Thread(new Thread1()).start();
}
```

- 实现Callable      
因为使用Thread和Runnable创建的任务，都是使用了`run()`方法，`run()`方法是一个没有返回值的方法，所以创建的任务不能方便的获取到结果。所以为了补足这个缺点，就增加了`Callable`类，可以使用`call()`方法创建一个有返回值的任务。
```
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

示例：      
```
public class MyCallable implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        TimeUnit.MILLISECONDS.sleep(2000);
        return 1;
    }
}

public static void main(String[] args) throws Exception {
    ExecutorService executors = Executors.newSingleThreadExecutor();
    Future<Integer> submit = executors.submit(new MyCallable());
    System.out.println(submit.get());
    executors.shutdown();
}
```

# Thread中的 run() 和 start() 的区别
- `run()`: 只是在原线程中调用  
- `start()`: 创建了一个新线程，并且`start()`中实现了`run()`,所以也会启动新创建的这个线程。

# 线程的阻塞
- `sleep`  
使得线程在指定时间内进入阻塞状态，在这段时间内线程不能获得CPU执行时间片段，在超过这段时间后，线程重新进入可执行状态。

- `yield`  
使当前线程放弃执行当前获取到的CPU执行时间片段，但是线程仍旧处于可执行状态。

- `wait` 和 `notify`  
`wait`使得线程处于阻塞状态，有两种方式使线程处于阻塞状态：指定时间、不指定时间。
（1）当指定时间时：线程阻塞时间超过该时间，则解除阻塞处于可执行状态；或者使用`notify`唤醒线程处于可执行状态。  
（2）当不指定时间时：只能使用`notify`来唤醒

**`sleep`和`wait`的区别**  
（1）sleep使正在执行的线程，放弃CPU时间片，让其他的线程获取到该时间片得到执行的权利。并且sleep并不会释放同步资源锁。  
（2）wait指的是当前线程让自己暂时退让出同步资源锁，以便其他正在等待该资源的线程得到该资源进而运行。所以wait方法是只能在同步方法或者同步代码块中使用才有意义。只有同步方法或者同步代码块才有锁。  
（3）由于wait上述2的原因，wait本身设计中也是Object类中的方法，而sleep是Thread的方法。  

# 线程池

在使用多线程编程时，一般是需要使用到线程池的。使用线程池的好处有：（1）减少线程的频繁创建与销毁；（2）可以控制并发数量；

## Java中几种创建线程池的方法：
- `Executors.newFixedThreadPool`: 创建固定线程数的线程池
- `Executors.newSingleThreadExecutor`: 创建单任务线程
- `Executors.newCachedThreadPool`: 创建可变线程池，可根据需要创建新线程的线程池，但是在以前构造的线程可用时将重用它们。
- `Executors.newScheduledThreadPool`: 创建延迟连接池
- `ThreadPoolExecutor`    

**弊端**    
(1)`FixedThreadPool` 和 `SingleThreadExecutor` ： 允许请求的队列长度为 `Integer.MAX_VALUE` ，可能堆积大量的请求，从而导致OOM。    
(2)`CachedThreadPool` 和 `ScheduledThreadPool` ： 允许创建的线程数量为 `Integer.MAX_VALUE` ，可能会创建大量线程，从而导致OOM。

## `ThreadPoolExecutor`
由于以上的弊端在使用时我们建议使用`ThreadPoolExecutor`来创建线程池。它提供了几种构造方法，用来创建。

以上几种方法根据实际的使用场景来进行使用。比如线程数量是可控的，这种没什么问题。但是线程数量不可控，可能出现任务特别多的时候，就会有资源不可用出现异常的问题。在平时使用的时候，通常我们会考虑更好的方案。
java中提供的`ThreadPoolExecutor`类可以很好的解决这些一般常见的问题。`ThreadPoolExecutor`继承了`AbstractExecutorService`类，并提供了四个构造器，可以通过构造器来创建线程池。构造器中的几个参数如下：
- `corePoolSize`: 核心池的大小，默认线程池中的线程数为0，当创建了任务时才会创建新的线程。可以通过`prestartAllCoreThreads()`或者`prestartCoreThread()`方法初始化线程数为corePoolSize或者1个。
- `maximumPoolSize`: 线程池的最大线程数
- `keepAliveTime`: 空闲线程的存活时间
- `unit`: 存活时间单位
- `workQueue`: 阻塞队列
- `threadFactory`: 线程工厂
- `handler`: 拒绝任务的策略  
1）ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。   
2）ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。   
3）ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）  
4）ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务
```
public static final ThreadPoolExecutor executor = new ThreadPoolExecutor(
    5,
    15,
    18000,
    TimeUnit.MILLISECONDS,
    new LinkedBlockingDeque<>(5000),
    new DefaultThreadFactory("view"),
    new ThreadPoolExecutor.CallerRunsPolicy());
```

## 线程池都提供了`submit()`和`execute()`方法
- `submit()`用来提交需要有返回值的任务，线程池会返回一个`Future`类型的对象，通过`Future`可以判断这个任务是否执行成功，并且可以使用`get()`函数来获取返回值，只是`get()`方法会阻塞当前线程直到任务完成。  
- `execute()`用来提交不需要返回值的任务

## ThreadPoolExecutor使用的阻塞队列

可以参考这篇文章: [https://blog.csdn.net/xiaojin21cen/article/details/87363143](https://blog.csdn.net/xiaojin21cen/article/details/87363143)

**在使用无限队列的时候一定要注意,一般情况下一定要给无限队列设置一个队列可以容纳的值,否则当任务数大于核心线程数时，线程数最多也只能达到corePoolSize的值**

# 几种JDK提供的并发容器
`java.util.concurrent`包下的几个类的介绍：
## ConcurrentHashMap: 线程安全的HashMap
## CopyOnWriteArrayList
线程安全的List，**一般使用于读多写少**的并发环境中，若写频繁的情况下慎用，因为这个东西的底层实现原理的原因，性能可能并不如想象中的好。

**使用案例中的坑**   
在项目中曾经使用过该容器，当时需要将几万条数据读出写入到一个容器中，然后再统一处理，为了提高效率，使用的是多线程读取，然后写入，就使用了CopyOnWriteArrayList，结果由于测试不足，导致在生产上数据量突然增大的情况下，向象容器写入数据时的效率急剧下降。后来因为找不到很好的解决办法，最终取消掉了这块的多线程处理的方式。反而效率更好了。--都是因为年轻不懂事。

**底层实现原理**  
CopyOnWriteArrayList容器允许并发读，读操作是无锁的，性能较高。至于写操作，比如向容器中添加一个元素，则首先将当前容器复制一份，然后在新副本上执行写操作，结束之后再将原容器的引用指向新容器。  
看一下几段源码
```添加操作
public boolean add(E e) {
    //ReentrantLock加锁，保证线程安全
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        //拷贝原容器，长度为原容器长度加一
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        //在新副本上执行添加操作
        newElements[len] = e;
        //将原容器引用指向新副本
        setArray(newElements);
        return true;
    } finally {
        //解锁
        lock.unlock();
    }
} 
```

```删除操作
public E remove(int index) {
    //加锁
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        E oldValue = get(elements, index);
        int numMoved = len - index - 1;
        if (numMoved == 0)
            //如果要删除的是列表末端数据，拷贝前len-1个数据到新副本上，再切换引用
            setArray(Arrays.copyOf(elements, len - 1));
        else {
            //否则，将除要删除元素之外的其他元素拷贝到新副本中，并切换引用
            Object[] newElements = new Object[len - 1];
            System.arraycopy(elements, 0, newElements, 0, index);
            System.arraycopy(elements, index + 1, newElements, index,
                                numMoved);
            setArray(newElements);
        }
        return oldValue;
    } finally {
        //解锁
        lock.unlock();
    }
}
```
添加操作和删除操作基本是一个道理，都是将新的元素Copy到新的容器中，然后引用再指向新的对象。
```
public E get(int index) {
    return get(getArray(), index);
}
```
读数据和锁没有关系，直接读，这样的话读的效率是非常高的，就相当于和ArrayList一模一样的效率。

**这种实现方式是存在一定的缺点的：**  
- **内存占用问题：** 毕竟每次执行写操作都要将原容器拷贝一份，数据量大时，对内存压力较大，可能会引起频繁GC；
- **无法保证实时性：** Vector对于读写操作均加锁同步，可以保证读和写的强一致性。而CopyOnWriteArrayList由于其实现策略的原因，写和读分别作用在新老不同容器上，在写操作执行过程中，读不会阻塞但读取到的却是老容器的数据。

## ConcurrentLinkedQueue
直接参考这篇博客，我觉得总结得非常好：[https://blog.csdn.net/u013991521/article/details/53068549](https://blog.csdn.net/u013991521/article/details/53068549)

## BlockingQueue
阻塞队列的一个接口，通过链表、数组等方式实现了这个接口。表示阻塞队列，非常适合用于作为数据共享的通道。通常使用的比如`ArrayBlockingQueue`、`LinkedBlockingQueue`。

- ArrayBlockingQueue：规定大小的BlockingQueue，其构造必须指定大小。其所含的对象是FIFO顺序排序的。
- LinkedBlockingQueue：大小不固定的BlockingQueue，若其构造时指定大小，生成的BlockingQueue有大小限制，不指定大小，其大小有Integer.MAX_VALUE来决定。其所含的对象是FIFO顺序排序的。
- PriorityBlockingQueue：类似于LinkedBlockingQueue，但是其所含对象的排序不是FIFO，而是依据对象的自然顺序或者构造函数的Comparator决定。

**有界队列/有限队列**           
当使用有限的 maximumPoolSizes时，有界队列（如 ArrayBlockingQueue）有助于防止资源耗尽，但是可能较难调整和控制。队列大小和最大池大小可能需要相互折衷：使用大型队列和小型池可以最大限度地降低 CPU 使用率、操作系统资源和上下文切换开销，但是可能导致人工降低吞吐量。如果任务频繁阻塞（例如，如果它们是 I/O边界），则系统可能为超过您许可的更多线程安排时间。使用小型队列通常要求较大的池大小，CPU使用率较高，但是可能遇到不可接受的调度开销，这样也会降低吞吐量。

**无界队列/无限队列**           
使用无界队列（例如，不具有预定义容量的 LinkedBlockingQueue）将导致在所有 corePoolSize 线程都忙时新任务在队列中等待。这样，创建的线程就不会超过 corePoolSize。（因此，maximumPoolSize的值也就无效了。）当每个任务完全独立于其他任务，即任务执行互不影响时，适合于使用无界队列；例如，在 Web页服务器中。这种排队可用于处理瞬态突发请求，当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。


# ThreadLocal
*此处是参考这篇文章整理：[http://www.threadlocal.cn/](http://www.threadlocal.cn/)*

## 线程上下文和ThreadLocal
正常来讲，每个线程都会有一个 “上下文(context)” ，在这个上下文中可以存放数据，在该线程的管辖范围内都能获取到这些数据。

但是线程本来就是属于重型对象，非常的耗费资源，如果在创建线程的时候再创建个上下文，岂不是相当于线程更加加重了。所以在JDK的设计中，设计者就做了一个优化，
当创建线程时，只是为上下文标记了一个位置，而在真正使用线程上下文的时候再创建这个上下文。这就相当于延迟创建了线程上下文。这样的话就提升了创建线程的效率。
而用来**负责创建线程上下文的对象就是`ThreadLocal`**。

通常我们可以看见类似这样的代码：
```
private static ThreadLocal<String> threadLocal = new ThreadLocal<>();
...
threadLocal.set("hello");
...
threadLocal.get();
```
这就是通常`ThreadLocal`的使用方法。
线程的上下文就是用来存放东西的，ThreadLocal负责创建线程context，同样，也是通过ThreadLocal来操作context中的数据的，通常使用set和get来进行存取操作。

ThreadLocal在set数据时，将自己也作为参数存进去了`map.set(this, value)`，如下：
```
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```
为什么ThreadLocal要将自己放进去呢？因为一个线程的上下文只有一个，如果是一个ThreadLocal对象操作context那就没有问题；但是如果有多个ThreadLocal操作线程的上下文，那就需要区分是哪个ThreadLocal存进去的数据，这样取数据时才能不混乱。       

我们可以将ThreadLocal看作一个代理对象，如下图所示：     
![将ThreadLocal看作一个代理对象](http://sunyanping.gitee.io/it-keep/ASSET/Thread和ThreadContext和ThreadLocal之间的关系.png)

## ThreadLocal的内存泄漏问题

可以参考这篇文章理解：[https://blog.csdn.net/qq_31821675/article/details/105316177](https://blog.csdn.net/qq_31821675/article/details/105316177)

通过TheadLocal将数据存放在ThreadContext（ThreadLocal是一片内存区域），那如果说这块区域的数据不能被及时销毁，必然就存在内存泄露的风险。

**为什么内存会发生泄露？**    
内存发生泄漏的前提有两个：  
- ThreadLocalRef用完后，没有手动清除Entry
- ThreadLocalRef用完后，当前线程没有销毁仍然在继续运行，比如使用线程池的情况下

> 个人钻牛角尖时的一个想法：由于key指向ThreadLocal实例是弱引用，会不会在当前线程运行期间，将ThreadLocal实例给回收了呢？     
答案自然是：不会。      
因为当前线程只要仍然使用着ThreadLocal，首先ThreadLocal变量指向ThreadLocal实例的强引用就必然存在，那么ThreadLocal实例自然不会被回收，那么key也就会一直指向ThreadLocal实例。在Java8中这个想法也是牛逼的很。当然假如ThreadLocal实例只被key引用，那只要gc一执行，ThreadLocal实例肯定被回收。

**ThreadLocal中的弱引用如何理解？** 

在Thread内部维护了一个ThreadLocal.ThreadLocalMap，数据都是在这个Map结构中存放。ThreadLocal在`set()`时将自己作为参数传入，并且将自己作为了Map的key，数据作为Map的value。ThreadLocal使用`get()`获取数据时，就是通过ThreadLocal和key之间的联系来取的，所以key必须指向对应的ThreadLocal对象，key指向ThreadLocal的这个引用就是弱引用。

**为什么不设计成强引用呢？**   

**1. 如果设计成强引用**

那么在线程用完ThreadLocalRef后，没有没有手动清除Rntry，当前线程也没有销毁仍然在运行，那由于key一直引用ThreadLocal实例，ThreadLocal实例不会被回收，同时Entry数据不会销毁。这样（1）ThreadLocal实例占用内存（2）Entry数据占用内存。更易发生内存泄漏。
![ThreadLocal设计成强引用](http://sunyanping.gitee.io/it-keep/ASSET/ThreadLocal设计成强引用.png)

**2. 如果设计成弱引用**     
那么就算线程用完ThreadLocalRef后，，没有没有手动清除Rntry，当前线程也没有销毁仍然在运行，那由于key引用ThreadLocal实例是弱引用，ThreadLocal实例在下次gc时也会被回收，key的引用也会变成null，当下次调用set/get/remove方法时，由于key==null，那value也会被清除（这个在ThreadLocal的源码中能看到），从而避免内存泄漏。
![ThreadLocal设计成弱引用](http://sunyanping.gitee.io/it-keep/ASSET/ThreadLocal设计成弱引用.png)

**所以在使用ThreadLocal的地方一定要记得使用`remove()`来清理内存中的数据。**

# 补充：Java的4种引用类型（强、软、弱、虚）
Java中设计了4种引用类型（强、软、弱、虚引用），在Java种使用最多最广泛的就是Strong Reference，比如：`String a = "123"`，就属于Strong Reference。
Strong Reference为JVM内部实现，其他三种引用类型全部继承自Reference类，如下所示：    
![](http://sunyanping.gitee.io/it-keep/ASSET/Java中的reference类结构.jpg)

## 强引用
Strong Rerence这个类并不存在，默认的对象都是强引用类型，因为有后来的新引用所衬托，所以才起了个名字叫"强引用"。      

如果JVM垃圾回收器 GC 可达性分析结果为可达，表示引用类型仍然被引用着，这类对象始终不会被垃圾回收器回收，即使JVM发生OOM也不会回收。而如果 GC 的可达性分析结果为不可达，那么在GC时会被回收。

## 软引用
软引用是一种比强引用生命周期稍弱的一种引用类型。在JVM内存充足的情况下，软引用并不会被垃圾回收器回收，只有在JVM内存不足的情况下，才会被垃圾回收器回收。所以软引用一般用来实现一些内存敏感的缓存，只要内存空间足够，对象就会保持不被回收掉。
```
SoftReference<String> softReference = new SoftReference<String>(new String("123"));
String str = softReference.get();
```

## 弱引用
弱引用是一种比软引用生命周期更短的引用。它的生命周期很短，不论当前内存是否充足，都只能存活到下一次垃圾收集之前。
```
WeakReference<String> weakReference = new WeakReference<String>(new String("123"));

System.gc();

if(weakReference.get() == null)
{
    System.out.println("weakReference已经被GC回收");
}

```
输出结果：

weakReference已经被GC回收

## 虚引用

虚引用与前面的几种都不一样，这种引用类型不会影响对象的生命周期，所持有的引用就跟没持有一样，随时都能被GC回收。

需要注意的是，在使用虚引用时，必须和引用队列关联使用。在对象的垃圾回收过程中，如果GC发现一个对象还存在虚引用，则会把这个虚引用加入到与之关联的引用队列中。

程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。

如果程序发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象内存被回收之前采取必要的行动防止被回收。虚引用主要用来跟踪对象被垃圾回收器回收的活动。
```
PhantomReference<String> phantomReference = new PhantomReference<String>(new String("123"), new ReferenceQueue<String>());

System.out.println(phantomReference.get());
```

# java.util.concurrent包
1. locks部分：显式锁（互斥锁和速写锁）相关
2. atomic部分：原子变量类相关，是构建非阻塞算法的基础
3. executor部分：线程池相关
4. collection部分：并发容器相关
5. tools部分：同步工具相关，如信号量、闭锁、栅栏等功能

# Locks-ReentrantLock、ReadWriteLock
在Java中最常见同步锁是使用`synchronized`来实现的，`synchronized`是一个关键字，只要按照该关键字的使用规则对代码块/静态方法/实例方法进行修饰，这些代码块/类/实例方法就具有线程互斥性，以达到同步的效果。
而其中的实现原理像其他关键字一样，底层是交给了JVM通过C++来实现的。在使用时无法在Java层面进行扩展和优化，灵活性不高，比如在使用时无法中断一个正在等待获取锁的线程，或者无法在请求一个锁时无限的等待下去。
在Java5开始，Java中构建了一个和synchronized一样效果的Java类，同时还扩展了一些其他特性，比如定时锁的等待、可中断的锁和公平锁等。这个就是java.util.concurrent.locks下的锁。其中常用的主要包含了`ReentrantLocl`
和`ReadWriteLock`。

## ReentrantLock
ReentrantLock的通用使用如下：
```
final Lock lock = new ReentrantLock();

public void method1() {
    try {
        lock.lock();
        System.out.println("method1 enter");
        TimeUnit.MILLISECONDS.sleep(2000);
        System.out.println("method1 exit");
    }catch (InterruptedException e) {
        e.printStackTrace();
    } finally {
        lock.unlock();
    }
}
```
以上的代码可以实现和synchronized同样的效果，但是需要的是显式的获取锁和释放锁，所以ReentrantLock也称为显式锁。

### ReentrantLock的可重入性
ReentrantLock和synchronized都具有可重入性，在当前线程已经获取锁的情况下，需要再次获取锁时，该线程会直接获取锁，不需要进行阻塞。
```
final Lock lock = new ReentrantLock();

public void method1() {
    try {
        lock.lock();
        System.out.println("method1 enter");
        TimeUnit.MILLISECONDS.sleep(2000);
        method2();
        System.out.println("method1 exit");
    }catch (InterruptedException e) {
        e.printStackTrace();
    } finally {
        lock.unlock();
    }
}

public void method2()  {
    try {
        lock.lock();
        System.out.println(Thread.currentThread().getName());
        System.out.println("method2 enter");
        TimeUnit.MILLISECONDS.sleep(6000);
        System.out.println("method2 exit");
    } catch (InterruptedException e) {
        e.printStackTrace();
    } finally {
        lock.unlock();
    }
}

public static void main(String[] args) {
    DemoReentrantLock demo = new DemoReentrantLock();
    new Thread(() -> {
        System.out.println(Thread.currentThread().getName());
        demo.method1();
    }).start();
    new Thread(() -> {
        System.out.println(Thread.currentThread().getName());
        demo.method2();
    }).start();
}
```
上面这段代码中，1线程调用 method1，在 method1 中调用了method2 ；2线程调用method2 。 如果method1中调用method2时如果无可重入性，那么该线程就需要与其他线程竞争执行 method2，就需要竞争锁，而出现阻塞；而锁具有可重入性时，该线程会直接获得锁(相当于最优先)，那就可以直接执行method2而不用阻塞。

### ReentrantLock-公平锁、非公平锁
synchronized是一个非公平锁，因为他无法确认按照线程申请锁的顺序使得线程获得锁。      
ReentrantLock既可以做非公平锁，也可以做公平锁。它有两个构造函数`ReentrantLock()`和`ReentrantLock(boolean fair)`，后者可以创建公平锁，以使线程按照申请锁的顺序获得锁。

### 与Condition结合进行线程的调度
通常线程间通信依靠的是Thread的wait()、notify()、notifyAll()等来处理，wait()可以挂起当前线程,从而将当前线程竞争到的时间片让出给其他的线程，使用 notify()、notifyAll()来唤起某个或者全部挂起的线程，以让挂起的线程重新竞争时间片，然后继续执行。

Condition也实现了同样的功能(await()、signal()、signalAll())，并扩展出了一些新的功能。和Thread提供的方法相比，Thread挂起线程使用的是一个内置队列，所有挂起的线程都在该队列中等待；而Condition相当于替换了这个内置队列，可以有多个队列，所以在使用上更加的灵活。如下代码的使用：
```
public class DemoCondition1 {

    private Lock lock = new ReentrantLock();
    private Condition condition1 = lock.newCondition();
    private Condition condition2 = lock.newCondition();

    public void method1() {
        lock.lock();
        try {
            System.out.println("method1 entry");
            TimeUnit.MILLISECONDS.sleep(2000);

            System.out.println("method1 当前线程挂起");
            condition1.await();

            System.out.println("method1 继续执行");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }

    }

    public void method2() {
        lock.lock();
        try {
            System.out.println("method2 entry");
            TimeUnit.MILLISECONDS.sleep(2000);

            System.out.println("method2 当前挂起");
            condition1.await();

            System.out.println("method2 继续执行");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }

    }

    public void method3() {
        lock.lock();
        try {
            System.out.println("method3 entry");
            TimeUnit.MILLISECONDS.sleep(2000);

            System.out.println("method3 当前挂起");
            condition2.await();

            System.out.println("method3 继续执行");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }

    }

    public void method4() {
        lock.lock();
        try {
            System.out.println("method4 entry");
            TimeUnit.MILLISECONDS.sleep(2000);

            System.out.println("method4 当前唤醒c1所有");
            condition1.signalAll();

            System.out.println("method4 继续执行");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }

    }

    public void method5() {
        lock.lock();
        try {
            System.out.println("method5 entry");
            TimeUnit.MILLISECONDS.sleep(2000);

            System.out.println("method5 当前唤醒c2");
            condition2.signal();

            System.out.println("method5 继续执行");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }

    }

    public static void main(String[] args) {
        final DemoCondition1 demo = new DemoCondition1();
        new Thread(demo::method1).start();
        new Thread(demo::method2).start();
        new Thread(demo::method3).start();
        new Thread(demo::method4).start();
        new Thread(demo::method5).start();
    }
}
```

## ReadWriteLock
`ReadWriteLock`的实现是`ReentrantReadWriteLock`，通常也是使用这个类。能够看出来这是个读写锁。

这个锁是既是一个独享锁，也是一个共享锁。在`ReadWriteLock`中有两个方法：
```
Lock readLock();
Lock writeLock();
```
因为读和写不同，读是不会改变数据的，所以在读的时候并不会产生线程安全的问题。所以读是可以支持多个线程同时操作。这个相当于对锁做了一个更细粒度的优化。
```
public class DemoReadWriteLock {

    private ReadWriteLock lock = new ReentrantReadWriteLock();
    private Lock readLock = lock.readLock();
    private Lock writeLock = lock.writeLock();

    public void method1() {
        readLock.lock();
        try {
            System.out.println("method1 enter");
            TimeUnit.MILLISECONDS.sleep(2000);
            System.out.println("method1 exit");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            readLock.unlock();
        }
    }

    public void method2() {
        writeLock.lock();
        try {
            System.out.println("method2 enter");
            TimeUnit.MILLISECONDS.sleep(2000);
            System.out.println("method2 exit");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            writeLock.unlock();
        }
    }

    public static void main(String[] args) {
        final DemoReadWriteLock demo = new DemoReadWriteLock();
        new Thread(demo::method1).start();
        new Thread(demo::method1).start();

        new Thread(demo::method2).start();
        new Thread(demo::method2).start();
    }
```
以上的使用中method1使用了ReadLock，method2使用了WriteLock，所以method1可以看到多个线程同时执行，也就是说ReadLock是可以让多个线程同时获得。

## 其它常用的方法
```
// 尝试获取锁,立即返回获取结果 轮询锁
public boolean tryLock();

//尝试获取锁,最多等待 timeout 时长 超时锁
public boolean tryLock(long timeout, TimeUnit unit);

//可中断锁,调用线程 interrupt 方法,则锁方法抛出 InterruptedException  中断锁
public void lockInterruptibly();

```

# Future、FutureTask、Callable、ComplatableFuture

## Future
Future 声明了对具体的 Runnable 或者 Callable 任务执行进行取消、查询、结果获取等方法。必要时可以通过 get 方法获取执行结果，该方法会阻塞直到任务返回结果。

```
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

Future 提供了三种功能：     
- **取消任务**：参数表示是否允许中途取消（中断）    
- **判断状态**：是否已取消、是否已完成      
- **获取结果**：两种方式，指不指定时间      
因为 Future 只是一个接口，所以无法直接用来创建对象，因此有了 FutureTask。

## FutureTask
![FutureTask的继承关系](http://sunyanping.gitee.io/it-keep/ASSET/FutureTask的继承关系.png)

可以看出 RunnableFuture 继承了 Runnable 接口和 Future 接口，而 FutureTask 实现了 RunnableFuture 接口。**所以它既可以作为 Runnable 被线程执行，又可以作为 Future 得到 Callable 的返回值**。

 FutureTask 内部有这几种状态：
 ```
 private volatile int state;
private static final int NEW          = 0;
private static final int COMPLETING   = 1;
private static final int NORMAL       = 2;
private static final int EXCEPTIONAL  = 3;
private static final int CANCELLED    = 4;
private static final int INTERRUPTING = 5;
private static final int INTERRUPTED  = 6;
 ```

当创建一个 FutureTask 对象时，初始状态是 NEW，在运行过程中，运行状态仅在方法set，setException 和 cancel 中转换为终端状态。有四种状态转换过程：

1. NEW -> COMPLETING -> NORMAL：正常执行并返回结果（run 执行成功再设置状态为 COMPLETING）
2. NEW -> COMPLETING -> EXCEPTIONAL：执行过程中出现异常（setException 先设置状态为 COMPLETING）
3. NEW -> CANCELLED：执行前被取消
4. NEW -> INTERRUPTING -> INTERRUPTED：执行时被中断（cancel 参数为 true 才可能出现这个状态）

事实上，FutureTask 是 Future 接口的一个唯一实现类。

## Callable
Callable提供了一个具有返回值的创建任务的方法，是对Runnable缺点的一个补充。
```
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```
这是一个泛型接口，返回的类型就是传进来的 V 类型。一般是结合 ExecutorService 来使用。ExecutorService 声明了几种 submit 方法，其中有一个就是传入 Callable。

## ComplatableFuture
ComplatableFutures 是Java8中新增的一个用于异步编程的API。也是对Future 的一个扩展。

**Future的局限性**          
1. 不能手动完成
2. Future 的结果在非阻塞的情况下，不能执行更进一步的操作。Future 不会通知你它已经完成了，它提供了一个阻塞的 get() 方法通知你结果。你无法给 Future 植入一个回调函数，当 Future 结果可用的时候，用该回调函数自动的调用 Future 的结果。
3. 多个 Future 不能串联在一起组成链式调用 有时候你需要执行一个长时间运行的计算任务，并且当计算任务完成的时候，你需要把它的计算结果发送给另外一个长时间运行的计算任务等等。你会发现你无法使用 Future 创建这样的一个工作流。
4. 不能组合多个 Future 的结果 假设你有10个不同的Future，你想并行的运行，然后在它们运行未完成后运行一些函数。你会发现你也无法使用 Future 这样做。
5. 没有异常处理 Future API 没有任务的异常处理结构


CompletableFuture 实现了 Future 和 CompletionStage 接口，并且提供了许多关于创建，链式调用和组合多个 Future 的便利方法集，而且有广泛的异常处理支持。

以下展示ComplatableFuture简单的使用示例：

示例一
```
 public static void main(String[] args) {
    // 异步执行完不需要有返回值，可以使用runAsync()
    // 异步执行完需要有返回值，可以使用supplyAsync()
    try {
//            CompletableFuture<String> completableFuture = new CompletableFuture<>();
//            completableFuture.complete("Future's Result");
//            String s = completableFuture.get();
//            System.out.println(s);

        CompletableFuture<Void> runAsync = CompletableFuture.runAsync(() -> {
            try {
                TimeUnit.SECONDS.sleep(10);

            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("this is not the main thread");
        });
    } catch (Exception e) {
        e.printStackTrace();
    }




    try {
        CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() -> {
            try {
                TimeUnit.SECONDS.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "Hello";
        });
        String s = completableFuture.get();
        System.out.println(s);

    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

示例二
```
public static void main(String[] args) throws ExecutionException, InterruptedException {
    // 异步任务执行完之后，需要执行下一步任务时可以使用thenApply() thenApplyAsync(),相当于回调函数，可以写成链式
    // 一般在回调完成之后需要返回值的话，就是用上述函数，
    // 回调完不需要返回值的话，可以使用thenAccept() thenRun()

    Executor executor = Executors.newFixedThreadPool(2);
    CompletableFuture<String> hello_runAsync = CompletableFuture.runAsync(() -> {
        try {
            sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + ": Hello runAsync");
    }, executor).thenApplyAsync((e) -> {
        System.out.println(Thread.currentThread().getName() + ": Hello thenApplyAsync");
        return "";
    }, executor).thenApply((e) -> {
        try {
            Thread.sleep(3000);
        } catch (InterruptedException ex) {
            ex.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + ": Hello thenApply");
        return "";
    });

    hello_runAsync.get();

    System.out.println(Thread.currentThread().getName() + ": Hello Hello");
}
```

示例三：
```
public static void main(String[] args) throws ExecutionException, InterruptedException {
    // 对于这种链的调用，如果出现异常则会直接抛出。API中提供了解决出现异常时的处理方案
    // 1、exceptionally() 当抛出异常时会进行处理
    // 2、handle() 是否出现异常都会处理

    CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
        // Code which might throw an exception
        String a = null;
//            if (a.equals(1)) {}
        return "Some result";
    }).thenApply(result -> {
        return "processed result";
    }).thenApply(result -> {
        return "result after further processing";
    }).thenApply(result -> {
        // do something with the final result
        return "something";
    }).handle((res, ex) -> {
        if (ex != null) {
            return "E";
        }
        return res;
    });

    System.out.println(future.get());
}
```

示例四：Compose
```
public static void main(String[] args) throws ExecutionException, InterruptedException {
    // 使用thenCompose() 组合两个独立的future
    // 使用thenCompose() 组合得到的就是异步最后直接的结果，要是使用thenApply() 得到的并不是 需要两步获取 才能得到
    // 这是使用于一个future依赖于另一个future完成之后来组合

    CompletableFuture<String> completableFuture2 = createHello().thenCompose(CompletableFutureComposeDemo::createWorld);

    String s = completableFuture2.get();
    System.out.println(s);


    CompletableFuture<CompletableFuture<String>> future = createHello().thenApply(CompletableFutureComposeDemo::createWorld);
    CompletableFuture<String> completableFuture = future.get();
    String s1 = completableFuture.get();
    System.out.println(s1);
}


static CompletableFuture<String> createHello() {
    try {
        Thread.sleep(2000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("1");
    return CompletableFuture.supplyAsync(() -> "Hello");
}

static CompletableFuture<String> createWorld(String s) {
    System.out.println(2);
    return CompletableFuture.supplyAsync(() -> s + " World");
}
```

示例五：Combine
```
public static void main(String[] args) throws ExecutionException, InterruptedException {

    CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(1);
        return "Hello";
    });

    CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
        System.out.println(2);
        return "World";
    });

    CompletableFuture<String> completableFuture = future1.thenCombine(future2, (a, b) -> {
        System.out.println(a + 3 + b);
        return a + b;
    });

    System.out.println(completableFuture.get());

}
```