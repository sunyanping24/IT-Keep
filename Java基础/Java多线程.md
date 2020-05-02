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
