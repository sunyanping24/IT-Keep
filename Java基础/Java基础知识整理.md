<!-- TOC -->

- [try...catch...finally](#trycatchfinally)
- [String的不可变性](#string的不可变性)
- [除了String有常量池，其它8种基本类型的对象池](#除了string有常量池其它8种基本类型的对象池)
- [synchronized关键字](#synchronized关键字)
- [全局变量、成员变量、局部变量](#全局变量成员变量局部变量)
- [创建对象的几种方式](#创建对象的几种方式)

<!-- /TOC -->

# try...catch...finally
java中处理异常必然会用到的代码块，通常的几种出现方式：`try...catch`、`try...finally`、`try...catch...finally`。也就是说try是必不可少的块。
```
try {
  // 可能出现异常的代码块
} catch() {
  // 异常处理
} finally {
  // 无论是否出现异常都需要执行的代码块。
  // 通常用来：释放物理链接（数据库连接、网络连接、数据磁盘链接）
}
```
**如果在finally块中只是为了释放这些物理连接，那可以使用JAVA7中提供的一个语法糖（自动资源管理技术）来省略finally块的简写方式。如下：这样当代码块使用完创建的流时会自动关闭资源的占用。**
```
try (FileInputStream stream = new FileInputStream();) {

} catch {

}
```
**区分一个概念**： Java的GC清理的是内存区域的数据，并不会管物理连接，这是需要手动进行关闭的。  
```
try {
  return 0;
} catch  () {
  return 1;
} finally  {
  return 2;
}
```
//输出： 2  
**finally块中的输出是会将try、catch块中的输出覆盖掉的。所以不要在finally块中使用`return`、`throw`语句。**  
**通常在finally块都是会执行的，除非出现类似于`System.exit(-1)`这样的直接退出程序的代码。**

**总结：**
- 程序先执行try的代码，若遇到异常，则执行catch代码块，再执行finally块；若无异常，则直接执行finally代码块。
- finally中的return、throw会覆盖掉前面的return和throw。
- 除非遇到退出程序的代码，否则finally都是会执行的。

# String的不可变性

先看String类中的源码：  
```
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];

    /** Cache the hash code for the string */
    private int hash; // Default to 0
    ....
}
```
1. String类是由`final`修饰的class，它不可被扩展
2. String的值实际是由`final`修饰的`char[]`，并且String类并没有提供修改的方法  

综上以上两除源码的实现，String类不可变。

```
String str1 = "hello";
String str2 = "hello";
String str3 = new String("hello");

String str4 = "str";
String str5 = "str";
String str6 = str4 + str5;  // 存放在堆内存
String str7 = "strstr";
String str8 = "str" + "str";    //存放在常量池


System.out.println(str1 == str2);   // true
System.out.println(str1.equals(str2)); // true
System.out.println(str1 == str3);   // false
System.out.println(str1.equals(str3));  // true
// 变量字符串拼接 先开辟内存空间
// 常量字符串拼接 先拼接，再开辟内存空间
System.out.println(str6 == str7);   // false
System.out.println(str6 == str8);   // false
System.out.println(str7 == str8);   // true
```
**变量字符串的拼接，实际是使用`new StringBuilder`来进行拼接的**

String不可变的好处：
1. 不存在线程安全的问题
2. 可以设计String常量池，减少内存占用，性能高
3. String的hash值只需要被计算一次，比如使用String常量做Map的Key时hash值不需要被重复计算

# 除了String有常量池，其它8种基本类型的对象池
以下几种创建Integer对象的方式有什么区别：
```
Integer i = 1;    // 自动装箱
Integer i = Integer.valueOf(1);   // 装箱
Integer i = new Integer(1);  
```
`Integer i = Integer.valueOf(1)`和`Integer i = 1`这两者是一样的，都是装箱操作，后面的写法只是简写的方式，实际上和前面的是一样的。
主要说`Integer i = Integer.valueOf(1)`和`Integer i = new Integer(1)`的区别：  

`Integer.valueOf()`的源码实现：
```
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}

  private static class IntegerCache {
      static final int low = -128;
      static final int high;
      static final Integer cache[];
      ......
  }
```
**总结**：
- 其中Integer有个内部类`IntegerCache`用来缓存Integer对象，当Integer.valueOf(1)创建的Integer对象再缓存中已经存在，则是从缓存中直接拿的,不需要创建新的对象。
- `new Integer()`始终是创建一个新的对象

*其他几种基本类型也是使用相同的实现方法*

# synchronized关键字  
`synchronized`关键字可以修饰**实例方法**、**静态方法**、**代码块**。
- 修饰实例方法： 执行时需要获得实例方法对象锁
- 修饰静态方法：执行时需要获得类对象锁。（修饰静态方法相当于是修饰类，不论new了多少实例，该方法只有一份）
- 修饰代码块：执行时需要获得实例代码块的对象锁。当修饰代码块时，使用的`synchronized(Clazz)`也相当于修饰类。
```
public class Demo1 {
    /**
     * 修饰类
     */
    public static synchronized void test1() {
        System.out.println(Thread.currentThread().getId() + " Demo1 test1()");
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    /**
     * 修饰方法
     */
    public synchronized void test2() {
        System.out.println(Thread.currentThread().getId() + " Demo1 test2()");
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    

    public void test3() {
        // 修饰方法
        synchronized (this) {
            System.out.println(Thread.currentThread().getId() + " Demo1 test2()");
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }

    public void test4() {
        // 修饰类
        synchronized (Demo1.class) {
            System.out.println(Thread.currentThread().getId() + " Demo1 test2()");
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```
```
Demo1 demo1 = new Demo1();
new Thread(() -> {
    demo1.test1();
}).start();

new Thread(() -> {
    demo1.test1();
}).start();
```
间隔5s调用

```
Demo1 demo1 = new Demo1();
Demo1 demo11 = new Demo1();
new Thread(() -> {
    demo1.test1();
}).start();

new Thread(() -> {
    demo11.test1();
}).start();
```
间隔5s调用

```
Demo1 demo1 = new Demo1();
Demo1 demo11 = new Demo1();
new Thread(() -> {
    demo1.test2();
}).start();

new Thread(() -> {
    demo11.test2();
}).start();
```
不间隔，两次都可以直接调用

```
Demo1 demo1 = new Demo1();
new Thread(() -> {
    demo1.test2();
}).start();

new Thread(() -> {
    demo1.test2();
}).start();
```
间隔5s调用

以上test3()同test2()，test4()同test1()，区别的地方在于一个同步执行的内容是整个方法内容，一个是方法中的代码块。

# 全局变量、成员变量、局部变量
||存储区域|生命周期|作用域|
|---|---|---|---|
|全局变量|全局变量也就是全局静态变量，是存放在方法区。|当类加载的时候就被创建，在类中只有一份；会跟着类的消失而消失。 |作用于所有类，直接被类调用。 |
|成员变量|成员变量如果没有实例化那么是存放在栈中；实例化了的对象是放在堆中，栈中存放的是指向堆中对象的引用地址。|在对象被创建时而存在，当对象被垃圾回收器回收时，也会消失。|作用于当前类，被对象调用。（除静态方法不能使用，静态方法没有隐式的this）|
|局部变量|局部变量放在栈中，`new`关键字创建的对象存放在堆中，基本类型的引用存放在栈中，基本类型值存放在栈帧中。|当方法被调用时而存在，当方法调用结束而消失。|作用于一个局部区域，比方说在一个方法中，方法调用。|

# 创建对象的几种方式
1. 使用`new`关键字
2. 使用`Class.forName("xxx.class")`获取到Class对象，`Class.newInstance()`创建对象
```
Class c = Class.forName("cn.qidd.other.Order");
Object o = c.newInstance();
System.out.println(o);
```
3. 使用`Class.forName("xxx.class")`获取到Class对象，`Class.getConstructor()`获取到构造器对象Constructor，`Constructor.newInstance()`创建对象
```
Class<?> aClass = Class.forName("cn.qidd.other.Order");
Constructor<?> constructor = aClass.getConstructor(String.class, String.class);
Object instance = constructor.newInstance("1", "交易订单");
System.out.println(instance);
```
4. 使用`clone()`方法。不太常用吧，反正我好像没用过。
```
Order order2 = new Order();
Object clone = order2.clone();
System.out.println(clone);
```
5. 对象的序列化和反序列化
```
Order order3 = new Order();
ObjectOutputStream os = new ObjectOutputStream(new FileOutputStream("order.obj"));
os.writeObject(order3);
// 再反序列化
ObjectInputStream is = new ObjectInputStream(new FileInputStream("order.obj"));
Object o1 = is.readObject();
System.out.println(o1);
```
上面使用的反射有两种方式，其中使用Class对象创建的相当于使用的是对象的无参构造，Constructor创建对象可以使用无参也可以使用有参构造器。
