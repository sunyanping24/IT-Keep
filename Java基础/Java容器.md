<!-- TOC -->

- [说说`List`、`Set`、`Map`之间的区别](#说说listsetmap之间的区别)
- [`ArrayList`、`LinkedList`](#arraylistlinkedlist)
  - [ArrayList](#arraylist)
  - [LinkedList](#linkedlist)
- [Vector](#vector)
- [Map几个主要实现类的区别](#map几个主要实现类的区别)
  - [HashMap](#hashmap)
  - [Hashtable](#hashtable)
  - [LinkedHashMap](#linkedhashmap)
  - [TreeMap](#treemap)
- [HashMap 和 Hashtable 的区别](#hashmap-和-hashtable-的区别)
- [HashMap 和 HashSet区别](#hashmap-和-hashset区别)
  - [HashSet如何检查重复](#hashset如何检查重复)
- [Hashtable和ConcurrentHashMap的区别](#hashtable和concurrenthashmap的区别)
  - [**补充：**  CAS机制](#补充--cas机制)
- [Stack（后进先出）（目前不建议使用）](#stack后进先出目前不建议使用)
- [Queue(先进先出)（也不建议使用）](#queue先进先出也不建议使用)
- [HashMap详解](#hashmap详解)
  - [HashMap的存储结构](#hashmap的存储结构)
  - [HashMap的扩容机制](#hashmap的扩容机制)

<!-- /TOC -->


![Java集合框架图](http://sunyanping.gitee.io/it-keep/ASSET/Java集合框架图.png)

# 说说`List`、`Set`、`Map`之间的区别
这绝对是一道经典且常见的面试题。但是我却不是特别的感冒，把`List`和`Set`放在一起比较挺正常，毕竟都是对象容器，可是按照常理来说`Map`作为一种`key-value`存储结构的容器和他们两个来进行比较表面上看别没什么可相互比较的。还不如比较比较`Set`和`Map`的结构问题。

- `List`: List里存放的对象是有序的，同时也是可以重复的，List关注的是索引，拥有一系列和索引相关的方法，查询速度快。因为往list集合里插入或删除数据时，会伴随着后面数据的移动，所有插入删除数据速度慢。
- `Set`: Set里存放的对象是无序，不能重复的，集合中的对象不按特定的方式排序，只是简单地把对象加入集合中。
- `Map`: Map集合中存储的是键值对，键不能重复，值可以重复。根据键得到值，对map集合遍历时先得到键的set集合，对set集合进行遍历，得到相应的值。

*从上面的这个Java集合框架图中其实大概能够看出来，对于我们平时使用最多的那应该就是一些类，对于定义为接口的和抽象类的应该在平时开发时可能使用的比较少。这里就主要总结`ArrayList`、`LinkedList`、`Vector`、`Stack`、`HashSet`、`TreeSet`、`LinkedHashSet`、`HashMap`、`TreeMap`、`LinkedHashMap`、`Hashtable`、`WeakHashMap`等*

# `ArrayList`、`LinkedList`

## ArrayList  
**ArrayList 类提供了快速的基于索引的成员访问方式，对尾部成员的增加和删除支持较好。** 使用 ArrayList 创建的集合，允许对集合中的元素进行快速的随机访问，不过，向 ArrayList 中插入与删除元素的速度相对较慢。

- 底层实现：使用Object数组，所以在插入和删除上需要元素的移动，这是影响效率的主要因素。但是读取是基于数据索引的，效率非常高。**但是在末尾插入元素时效率就并不会低，所以大多情况下开发中使用ArrayList还是比较更广泛的**

## LinkedList
**LinkedList 类采用链表结构保存对象，这种结构的优点是便于向集合中插入或者删除元素。需要频繁向集合中插入和删除元素时，使用 LinkedList 类比 ArrayList 类效果高，但是 LinkedList 类随机访问元素的速度则相对较慢。** 这里的随机访问是指检索集合中特定索引位置的元素。
- 底层实现：使用双向链表数据结构，所以在插入和删除上不需要元素的移动，所以效率更高。但是链表存储需要的资源也更多，并且读取的效率不如ArrayList.

# Vector
Vector也是和ArrayList一样，底层使用的Object数组结构来实现的，但是Vector中操作容器内元素的方法都是加了`synchronized`关键字，来使得Vector成为线程安全的集合容器。  
一般情况下在项目中用到这个的地方很少，至少我似乎是没有使用过这个。如果各位在多线程操作容器的情况中使用到的话，因该考虑一下它的效率是不是一个存在的问题。

# Map几个主要实现类的区别
## HashMap
它根据键的hashCode值存储数据，大多数情况下可以直接定位到它的值，因而具有很快的访问速度，但遍历顺序却是不确定的。 HashMap最多只允许一条记录的键为null，允许多条记录的值为null。HashMap非线程安全，即任一时刻可以有多个线程同时写HashMap，可能会导致数据的不一致。如果需要满足线程安全，可以用 Collections的synchronizedMap方法使HashMap具有线程安全的能力，或者使用ConcurrentHashMap。

## Hashtable
Hashtable是遗留类，很多映射的常用功能与HashMap类似，不同的是它承自Dictionary类，并且是线程安全的，任一时间只有一个线程能写Hashtable，并发性不如ConcurrentHashMap，因为ConcurrentHashMap引入了分段锁。Hashtable不建议在新代码中使用，不需要线程安全的场合可以用HashMap替换，需要线程安全的场合可以用ConcurrentHashMap替换。

## LinkedHashMap
LinkedHashMap是HashMap的一个子类，保存了记录的插入顺序，在用Iterator遍历LinkedHashMap时，先得到的记录肯定是先插入的，也可以在构造时带参数，按照访问次序排序。

## TreeMap
TreeMap实现SortedMap接口，能够把它保存的记录根据键排序，默认是按键值的升序排序，也可以指定排序的比较器，当用Iterator遍历TreeMap时，得到的记录是排过序的。如果使用排序的映射，建议使用TreeMap。在使用TreeMap时，key必须实现Comparable接口或者在构造TreeMap传入自定义的Comparator，否则会在运行时抛出java.lang.ClassCastException类型的异常。

# HashMap 和 Hashtable 的区别
**线程是否安全：** HashMap 是非线程安全的，HashTable 是线程安全的；HashTable 内部的方法基本都经过synchronized 修饰。（如果你要保证线程安全的话就使用 ConcurrentHashMap 吧！）；   
**效率：** 因为线程安全的问题，HashMap 要比 HashTable 效率高一点。另外，HashTable 基本被淘汰，不要在代码中使用它；    
**对Null key 和Null value的支持：** HashMap 中，null 可以作为键，这样的键只有一个，可以有一个或多个键所对应的值为 null。。但是在 HashTable 中 put 进的键值只要有一个 null，直接抛出 NullPointerException。    
初始容量大小和每次扩充容量大小的不同 ： ①创建时如果不指定容量初始值，Hashtable 默认的初始大小为11，之后每次扩充，容量变为原来的2n+1。HashMap 默认的初始化大小为16。之后每次扩充，容量变为原来的2倍。②创建时如果给定了容量初始值，那么 Hashtable 会直接使用你给定的大小，而 HashMap 会将其扩充为2的幂次方大小。也就是说 HashMap 总是使用2的幂作为哈希表的大小,后面会介绍到为什么是2的幂次方。    
**底层数据结构：** JDK1.8 以后的 HashMap 在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间。Hashtable 没有这样的机制。

# HashMap 和 HashSet区别
比较这两个，虽然一个是存储的key-value，一个是整体对象。但是由于这两者的底层实现是有相互关联的。HashSet的底层实现就是使用HashMap来实现的。（HashSet 的源码非常非常少，因为除了 clone()、writeObject()、readObject()是 HashSet 自己不得不实现之外，其他方法都是直接调用 HashMap 中的方法。下面贴一下HashSet的源码：
```
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
    static final long serialVersionUID = -5024744406713321676L;

    private transient HashMap<E,Object> map;

    private static final Object PRESENT = new Object();

    public HashSet() {
        map = new HashMap<>();
    }

    public HashSet(Collection<? extends E> c) {
        map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
        addAll(c);
    }

    public HashSet(int initialCapacity, float loadFactor) {
        map = new HashMap<>(initialCapacity, loadFactor);
    }

    public HashSet(int initialCapacity) {
        map = new HashMap<>(initialCapacity);
    }

    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }

    public Iterator<E> iterator() {
        return map.keySet().iterator();
    }

    public int size() {
        return map.size();
    }

    public boolean isEmpty() {
        return map.isEmpty();
    }

    public boolean contains(Object o) {
        return map.containsKey(o);
    }

    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }

    public boolean remove(Object o) {
        return map.remove(o)==PRESENT;
    }

    public void clear() {
        map.clear();
    }
}
```
从这个源码中能够看到HashSet就是调用HashMap中的方法，实际上HashSet的元素就是存储为HashMap的Key。所以HashSet元素不能重复，并且器实现方法和HashMap的实现方法是一样的。
## HashSet如何检查重复
当你把对象加入HashSet时，HashSet会先计算对象的hashcode值来判断对象加入的位置，同时也会与其他加入的对象的hashcode值作比较，如果没有相符的hashcode，HashSet会假设对象没有重复出现。但是如果发现有相同hashcode值的对象，这时会调用equals（）方法来检查hashcode相等的对象是否真的相同。如果两者相同，HashSet就不会让加入操作成功。  

**hashCode（）与equals（）的相关规定：**

1. 如果两个对象相等，则hashcode一定也是相同的
2. 两个对象相等,对两个equals方法返回true
3. 两个对象有相同的hashcode值，它们也不一定是相等的
4. 综上，equals方法被覆盖过，则hashCode方法也必须被覆盖
5. hashCode()的默认行为是对堆上的对象产生独特值。如果没有重写hashCode()，则该class的两个对象无论如何都不会相等（即使这两个对象指向相同的数据）。


参考文档（不仅限于此篇博客）： [剖析面试最常见问题之java集合框架](https://snailclimb.gitee.io/javaguide/#/docs/java/collection/Java%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6%E5%B8%B8%E8%A7%81%E9%9D%A2%E8%AF%95%E9%A2%98?id=%e5%89%96%e6%9e%90%e9%9d%a2%e8%af%95%e6%9c%80%e5%b8%b8%e8%a7%81%e9%97%ae%e9%a2%98%e4%b9%8bjava%e9%9b%86%e5%90%88%e6%a1%86%e6%9e%b6)

# Hashtable和ConcurrentHashMap的区别
请参考这篇文章：[ConcurrentHashMap 和 Hashtable 的区别](https://snailclimb.gitee.io/javaguide/#/docs/java/collection/Java%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6%E5%B8%B8%E8%A7%81%E9%9D%A2%E8%AF%95%E9%A2%98?id=concurrenthashmap-%e5%92%8c-hashtable-%e7%9a%84%e5%8c%ba%e5%88%ab)

## **补充：**  CAS机制  
解决并发安全问题总结来说就是两种情况：有锁和无锁  
- **有锁：** 使用`synchronized`，这种常称为悲观锁，就是认为每次临界操作都会出现冲突，用锁来阻塞线程的执行。  
- **无锁：** 不使用锁，这种常称为乐观锁，就是认为每次临界操作都不会出现冲突，因此所有线程都可以不用阻塞的持续执行；而一旦出现了冲突，则重试操作一直到没有冲突为止。

CAS的机制就是用来在无锁的情况下检测冲突的。具体的实现原理如下：
>CAS 操作包含三个操作数 —— 内存位置（V）、预期原值（A）和新值(B)。 如果内存位置的值与预期原值相匹配，那么处理器会自动将该位置值更新为新值 。否则，处理器不做任何操作。无论哪种情况，它都会在 CAS 指令之前返回该 位置的值。（在 CAS 的一些特殊情况下将仅返回 CAS 是否成功，而不提取当前 值。）CAS 有效地说明了“我认为位置 V 应该包含值 A；如果包含该值，则将 B 放到这个位置；否则，不要更改该位置，只告诉我这个位置现在的值即可。”  
>
>通常将 CAS 用于同步的方式是从地址 V 读取值 A，执行多步计算来获得新 值 B，然后使用 CAS 将 V 的值从 A 改为 B。如果 V 处的值尚未同时更改，则 CAS 操作成功。  
>
>类似于 CAS 的指令允许算法执行读-修改-写操作，而无需害怕其他线程同时修改变量，因为如果其他线程修改变量，那么 CAS会检测它（并失败），算法可以对该操作重新计算

**CAS的问题**  
1. **ABA问题：** 当你获得对象当前数据后，在准备修改为新值前，对象的值被其他线程连续修改了两次，而经过两次修改后，对象的值又恢复为旧值，这样当前线程无法正确判断这个对象是否修改过  
**解决办法：** JDK1.5可以利用类AtomicStampedReference来解决这个问题，AtomicStampedReference内部不仅维护了对象值，还维护了一个时间戳。当AtomicStampedReference对应的数值被修改时，除了更新数据本身外，还必须要更新时间戳，对象值和时间戳都必须满足期望值，写入才会成功

2. **循环时间长开销大。** 自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。  
**解决办法：** JVM支持处理器提供的pause指令，使得效率会有一定的提升，pause指令有两个作用，第一它可以延迟流水线执行指令,使CPU不会消耗过多的执行资源，第二它可以避免在退出循环的时候因内存顺序冲突（memory order violation）而引起CPU流水线被清空（CPU pipeline flush），从而提高CPU的执行效率。

3. **只能保证一个共享变量的原子操作。** 当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性  
**解决办法：** 从Java1.5开始JDK提供了AtomicReference类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行CAS操作。


# Stack（后进先出）（目前不建议使用）
Stack继承了Vector，所以基本就知道了Stack的数据结构是什么样子的。当然它也是一个线程安全的栈。在Stack类中它实现了自己独有的几个方法，入栈和出栈的方法。
```
public class Stack<E> extends Vector<E> {
 
    public Stack() {}
    // 入栈
    public E push(E item) {...}
    // 出栈
    public synchronized E pop() {...}
    // 返回栈顶的元素，但是栈顶元素不会出栈
    public synchronized E peek() {...}

    public boolean empty() {...}

    public synchronized int search(Object o) {...}
}
```

# Queue(先进先出)（也不建议使用）
Queue是个接口继承自Collection，通常常见的是ArrayQueue。Queue也有自己的方法。
```
public interface Queue<E> extends Collection<E> {
    // 向队列末尾添加元素，若队列已满则发生异常
    boolean add(E e);
    // 向队列末尾添加元素若队列已满，返回false,若添加成功返回true
    boolean offer(E e);
    // 移除并返回队列头部元素，若队列已空抛出异常。
    E remove();
    // 移除并返回队列头部元素，若队列已空返回Null。另外，可以附加时间，时间单位参数设置超时。
    E poll();
    // 返回队列头部元素，若队列已空抛出异常。
    E element();
    // 返回队列头部元素，若队列已空返回Null。
    E peek();
}
```
需要注意的是，由于队列中poll和peek操作以Null为标志，所以队列中添加Null元素是不合法的。

# HashMap详解
在Java7中和Java8中HashMap的底层发生了一定的变化，因为现在Java8是绝对的主流，所以在这里整理一下Java8中的HashMap的实现机制。

## HashMap的存储结构
从结构实现来讲，HashMap是**数组+链表+红黑树**（红黑树是从Java8开始设计的）。       
![HashMap底层存储结构](http://sunyanping.gitee.io/it-keep/ASSET/HashMap底层存储结构.png)

1. Node是什么

HashMap类中有一个非常重要的字段，就是`Node[] table`，即哈希桶数组，明显它是一个Node的数组
```
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;    //用来定位数组索引位置
    final K key;
    V value;
    Node<K,V> next;   //链表的下一个node

    Node(int hash, K key, V value, Node<K,V> next) { ... }
    public final K getKey(){ ... }
    public final V getValue() { ... }
    public final String toString() { ... }
    public final int hashCode() { ... }
    public final V setValue(V newValue) { ... }
    public final boolean equals(Object o) { ... }
}
```
Node是HashMap的一个内部类，实现了Map.Entry接口，本质就是一个映射(键值对)。上图中的每个黑色圆点就是一个Node对象。

2. Hash桶（哈希表）     

HashMap就是使用哈希表来存储的。为了解决hash冲突，Java的HashMap使用了**链地址法**，链地址法就是使用**数组+链表**的方式来存储数据。也就是在数据的每一个元素上都对应一个链表数据结构，当数据被hash后，得到数组下标，需要把数据存放在对应的下标位置，但是该下标已经存放了对应的数据，然后就需要通过`equals()`方法来比较key是否相同，若key不相同，则需要将该数据存放到这个链表结构中。

为什么不同数据hash算法得到的下标会出现相同结构，这是因为hash算法本身的原因。这种情况称为**hash碰撞**。

如果哈希桶数组很大，即使较差的Hash算法也会比较分散，如果哈希桶数组数组很小，即使好的Hash算法也会出现较多碰撞，所以就需要在空间成本和时间成本之间权衡，其实就是在根据实际情况确定哈希桶数组（Node[] table）的大小，并在此基础上设计好的hash算法减少Hash碰撞。**所以好的Hash算法和扩容机制至关重要**。

3. HashMap的初始化数据和扩容

这里展示几个HashMap中的字段：
```
transient int size：表示当前HashMap包含的键值对数量
transient int modCount：表示当前HashMap修改次数
int threshold：表示当前HashMap能够承受的最多的键值对数量，一旦超过这个数量HashMap就会进行扩容
final float loadFactor：负载因子，用于扩容
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4：默认的table初始容量
static final float DEFAULT_LOAD_FACTOR = 0.75f：默认的负载因子
static final int TREEIFY_THRESHOLD = 8: 链表长度大于等于该参数转红黑树
static final int UNTREEIFY_THRESHOLD = 6: 当树的节点数小于等于该参数转成链表
```
首先，Node[] table的初始化长度length(默认值是16)，Load factor为负载因子(默认值是0.75)，threshold是HashMap所能容纳的最大数据量的Node(键值对)个数。threshold = length * Load factor。也就是说，在数组定义好长度之后，负载因子越大，所能容纳的键值对个数越多。

结合负载因子的定义公式可知，threshold就是在此Load factor和length(数组长度)对应下允许的最大元素数目，超过这个数目就重新resize(扩容)，扩容后的HashMap容量是之前容量的两倍。默认的**负载因子0.75是对空间和时间效率的一个平衡选择**，建议大家不要修改，除非在时间和空间比较特殊的情况下，比如内存空间很多而又对时间效率要求很高，可以降低负载因子Load factor的值；相反，如果内存空间紧张而对时间效率要求不高，可以增加负载因子loadFactor的值，这个值可以大于1。

size这个字段其实很好理解，就是HashMap中实际存在的键值对数量。注意和table的长度length、容纳最大键值对数量threshold的区别。而modCount字段主要用来记录HashMap内部结构发生变化的次数。强调一点，内部结构发生变化指的是结构发生变化，例如put新键值对，但是某个key对应的value值被覆盖不属于结构变化。

在HashMap中，哈希桶数组table的长度length大小必须为2的n次方(一定是合数)，这是一种非常规的设计。HashMap采用这种非常规设计，主要是为了在取模和扩容时做优化，同时减少冲突，HashMap定位哈希桶索引位置时，也加入了高位参与运算的过程。这里存在一个问题，即使负载因子和Hash算法设计的再合理，也免不了会出现拉链过长的情况，一旦出现拉链过长，则会严重影响HashMap的性能。于是，在JDK1.8版本中，对数据结构做了进一步的优化，引入了红黑树。而当链表长度太长（默认超过8）时，链表就转换为红黑树，利用红黑树快速增删改查的特点提高HashMap的性能。


## HashMap的扩容机制
HashMap对象内部的数组无法装载更多的元素时，对象就需要扩大数组的长度，以便能装入更多的元素。当然Java里的数组是无法自动扩容的，方法是使用一个新的数组代替已有的容量小的数组。

HashMap扩容可以分为3种情况：         
1. 使用默认构造方法初始化HashMap，在一开始初始化的时候会返回一个空的table，并且`thershold`为0，因此第一次扩容的容量为默认值`DEFAULT_INITIAL_CAPACITY`也就是16。
2. 使用指定初始容量的构造方法初始化HashMap，那么第一次扩容的容量就是指定的初始容量。
3. HashMap不是第一次扩容，如果HashMap已经扩容过的话，那么每次扩容table的容量都是原有的两倍。

**这边也可以引申到一个问题HashMap是先插入还是先扩容：HashMap初始化后首次插入数据时，先发生resize扩容再插入数据，之后每当插入的数据个数达到threshold时就会发生resize，此时是先插入数据再resize。**

**扩容机制核心方法是`Node<K,V>[] resize()`**，这里展示一下resize()的具体实现。          
```
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;//首次初始化后table为Null
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;//默认构造器的情况下为0
    int newCap, newThr = 0;
    if (oldCap > 0) {//table扩容过
            //当前table容量大于最大值得时候返回当前table
         if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                    oldCap >= DEFAULT_INITIAL_CAPACITY)
        //table的容量乘以2，threshold的值也乘以2           
        newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
    //使用带有初始容量的构造器时，table容量为初始化得到的threshold
     newCap = oldThr;
    else {  //默认构造器下进行扩容  
            // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
    //使用带有初始容量的构造器在此处进行扩容
         float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                    (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            HashMap.Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                // help gc
                oldTab[j] = null;
                if (e.next == null)
                    // 当前index没有发生hash冲突，直接对2取模，即移位运算hash &（2^n -1）
                    // 扩容都是按照2的幂次方扩容，因此newCap = 2^n
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof HashMap.TreeNode)
                    // 当前index对应的节点为红黑树，这里篇幅比较长且需要了解其数据结构跟算法，因此不进行详解，当树的高度小于等于UNTREEIFY_THRESHOLD则转成链表
                    ((HashMap.TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    // 把当前index对应的链表分成两个链表，减少扩容的迁移量
                    HashMap.Node<K,V> loHead = null, loTail = null;
                    HashMap.Node<K,V> hiHead = null, hiTail = null;
                    HashMap.Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            // 扩容后不需要移动的链表
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            // 扩容后需要移动的链表
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        // help gc
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        // help gc
                        hiTail.next = null;
                        // 扩容长度为当前index位置+旧的容量
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```







HashMap对象内部的数组无法装载更多的元素时，对象就需要扩大数组的长度，以便能装入更多的元素。当然Java里的数组是无法自动扩容的，方法是使用一个新的数组代替已有的容量小的数组。