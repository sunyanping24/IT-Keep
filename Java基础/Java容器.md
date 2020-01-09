<!-- TOC -->

- [说说`List`、`Set`、`Map`之间的区别](#说说listsetmap之间的区别)
- [`ArrayList`、`LinkedList](#arraylistlinkedlist)
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

<!-- /TOC -->


![Java集合框架图](/ASSET/Java集合框架图.png)

# 说说`List`、`Set`、`Map`之间的区别
这绝对是一道经典且常见的面试题。但是我却不是特别的感冒，把`List`和`Set`放在一起比较挺正常，毕竟都是对象容器，可是按照常理来说`Map`作为一种`key-value`存储结构的容器和他们两个来进行比较表面上看别没什么可相互比较的。还不如比较比较`Set`和`Map`的结构问题。

- `List`: List里存放的对象是有序的，同时也是可以重复的，List关注的是索引，拥有一系列和索引相关的方法，查询速度快。因为往list集合里插入或删除数据时，会伴随着后面数据的移动，所有插入删除数据速度慢。
- `Set`: Set里存放的对象是无序，不能重复的，集合中的对象不按特定的方式排序，只是简单地把对象加入集合中。
- `Map`: Map集合中存储的是键值对，键不能重复，值可以重复。根据键得到值，对map集合遍历时先得到键的set集合，对set集合进行遍历，得到相应的值。

*从上面的这个Java集合框架图中其实大概能够看出来，对于我们平时使用最多的那应该就是一些类，对于定义为接口的和抽象类的应该在平时开发时可能使用的比较少。这里就主要总结`ArrayList`、`LinkedList`、`Vector`、`Stack`、`HashSet`、`TreeSet`、`LinkedHashSet`、`HashMap`、`TreeMap`、`LinkedHashMap`、`Hashtable`、`WeakHashMap`等*

# `ArrayList`、`LinkedList

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

3. **只能保证一个共享变量的原子操作。**当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性  
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




