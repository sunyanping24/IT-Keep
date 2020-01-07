
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



