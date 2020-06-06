<!-- TOC -->

- [计算机核心数和线程数的计算关系](#计算机核心数和线程数的计算关系)
- [Java中创建异步任务的几种方法（创建线程的方法）](#java中创建异步任务的几种方法创建线程的方法)
- [Thread中的`run()`和`start()`的区别](#thread中的run和start的区别)
- [线程的阻塞](#线程的阻塞)
- [线程池](#线程池)
  - [Java中几种创建线程池的方法：](#java中几种创建线程池的方法)
  - [`ThreadPoolExecutor`](#threadpoolexecutor)
  - [线程池都提供了`submit()`和`execute()`方法](#线程池都提供了submit和execute方法)
  - [ThreadPoolExecutor使用的阻塞队列](#threadpoolexecutor使用的阻塞队列)
- [几种JDK提供的并发容器](#几种jdk提供的并发容器)
  - [`ConcurrentHashMap`: 线程安全的HashMap](#concurrenthashmap-线程安全的hashmap)
  - [`CopyOnWriteArrayList`](#copyonwritearraylist)
  - [`ConcurrentLinkedQueue`](#concurrentlinkedqueue)
  - [`BlockingQueue`（待实战中总结）](#blockingqueue待实战中总结)
  - [`ConcurrentSkipListMap`（待实战中总结）](#concurrentskiplistmap待实战中总结)
- [ThreadLocal](#threadlocal)
  - [线程上下文和ThreadLocal](#线程上下文和threadlocal)
  - [ThreadLocal的内存泄漏问题](#threadlocal的内存泄漏问题)
- [Java的4种引用类型（强、软、弱、虚）](#java的4种引用类型强软弱虚)
  - [强引用](#强引用)
  - [软引用](#软引用)
  - [弱引用](#弱引用)
  - [虚引用](#虚引用)

<!-- /TOC -->

为什么并发是最常见的一种提高任务处理能力的手段，归根结底---为了充分利用计算机的计算能力。那么也就带来了最常见的几种问题：  
1. 线程数和计算机的核心数之间的关系，如何均衡，如何才能发挥计算最大的计算能力
2. 并发带来的内存数据安全问题 

# 计算机核心数和线程数的计算关系
![计算机核心数和线程数的计算公式](http://sunyanping.gitee.io/it-keep/ASSET/计算机核心数和线程数的计算公式.png)    
`w/c`中的W(等待时间)---计算时间+（IO时间+...），c---计算时间，目前我在实际应用中碰到的使用多线程处理的地方都是IO比较密集的地方，比如数据库插入数据。这中就是比较耗时的操作，w/c的值就相对来说比较大。比如保存1K条数据，计算耗时5ms，IO耗时100ms，那W/C=(5+100)/5=21。如果是1核1线程的计算机就可以设置成22，如果是4核8线程可以设置成168.
当然这几公式里面的CPU都是按照1C1T来说的。

# Java中创建异步任务的几种方法（创建线程的方法）
- 实现`Runnable`
Runnable是一个函数接口，其中只有一个`run()`函数
- 继承`Thread`
Thread是一个类，也是实现了`Runnable`，其中包含了很多方法。所以采用这中方式也是相对于增加了开销.
- 使用线程池（一般在实际的开发中需要使用这种方式，方便对线程的统一管理）

```
public static void main(String[] args) {
    new Thread(new Thread1()).start();
    new Thread2().start();
}

static class Thread1 implements Runnable {
    @Override
    public void run() {
        System.out.println("线程：" + Thread.currentThread().getName() + ", Thread1");
    }
}

static class Thread2 extends Thread {
    @Override
    public void run() {
        System.out.println("线程：" + Thread.currentThread().getName() + ", Thread2");
    }
}
```

# Thread中的`run()`和`start()`的区别
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

**在使用无限队列的时候一定要注意,一般情况下一定要给无限队列设置一个队列可以容纳的值,否则线程数最多只能达到corePoolSize的值**

# 几种JDK提供的并发容器
`java.util.concurrent`包下的几个类的介绍：
## `ConcurrentHashMap`: 线程安全的HashMap
## `CopyOnWriteArrayList`
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

## `ConcurrentLinkedQueue`
直接参考这篇博客，我觉得总结得非常好：[https://blog.csdn.net/u013991521/article/details/53068549](https://blog.csdn.net/u013991521/article/details/53068549)

## `BlockingQueue`（待实战中总结）
阻塞队列的一个接口，通过链表、数组等方式实现了这个接口。表示阻塞队列，非常适合用于作为数据共享的通道。通常使用的比如`ArrayBlockingQueue`、`LinkedBlockingQueue`。

## `ConcurrentSkipListMap`（待实战中总结）
跳表的实现。这是一个 Map，使用跳表的数据结构进行快速查找。

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
![](http://sunyanping.gitee.io/IT-Keep/ASSET/Thread和ThreadContext和ThreadLocal之间的关系.png)

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
![ThreadLocal设计成强引用](/ASSET/ThreadLocal设计成强引用.png)

**2. 如果设计成弱引用**     
那么就算线程用完ThreadLocalRef后，，没有没有手动清除Rntry，当前线程也没有销毁仍然在运行，那由于key引用ThreadLocal实例是弱引用，ThreadLocal实例在下次gc时也会被回收，key的引用也会变成null，当下次调用set/get/remove方法时，由于key==null，那value也会被清除（这个在ThreadLocal的源码中能看到），从而避免内存泄漏。
![ThreadLocal设计成弱引用](/ASSET/ThreadLocal设计成弱引用.png)

**所以在使用ThreadLocal的地方一定要记得使用`remove()`来清理内存中的数据。**

# Java的4种引用类型（强、软、弱、虚）
Java中设计了4种引用类型（强、软、弱、虚引用），在Java种使用最多最广泛的就是Strong Reference，比如：`String a = "123"`，就属于Strong Reference。
Strong Reference为JVM内部实现，其他三种引用类型全部继承自Reference类，如下所示：    
![](http://sunyanping.gitee.io/IT-Keep/ASSET/Java中的reference类结构.jpg)

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