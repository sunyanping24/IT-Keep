<!-- TOC -->

- [Java的三大特性封装、继承、多态](#java的三大特性封装继承多态)
- [Java中的访问权限](#java中的访问权限)
- [重载（overload）和重写(Override)](#重载overload和重写override)
- [super和this的区别](#super和this的区别)
- [try...catch...finally](#trycatchfinally)
- [Java的异常处理](#java的异常处理)
  - [异常的分类](#异常的分类)
  - [异常的处理机制](#异常的处理机制)
- [String的不可变性](#string的不可变性)
- [除了String有常量池，其它8种基本类型的对象池](#除了string有常量池其它8种基本类型的对象池)
- [synchronized关键字](#synchronized关键字)
  - [synchronized的实现原理简单描述](#synchronized的实现原理简单描述)
- [全局变量、成员变量、局部变量](#全局变量成员变量局部变量)
- [创建对象的几种方式](#创建对象的几种方式)
- [String的拼接，使用`+`还是StringBuilder或者StringBuffer](#string的拼接使用还是stringbuilder或者stringbuffer)
- [形参和实参](#形参和实参)
- [构造器](#构造器)
- [为什么子类构造方法默认第一行调用父类构造方法](#为什么子类构造方法默认第一行调用父类构造方法)
- [线程的阻塞](#线程的阻塞)
- [Java中线程状态的切换](#java中线程状态的切换)
- [`volatile`关键字](#volatile关键字)
  - [补充：Java的内存模型](#补充java的内存模型)
- [抽象类和接口的区别](#抽象类和接口的区别)
  - [抽象类有构造方法为什么不能实例化？为什么有构造方法却不能被实例化？](#抽象类有构造方法为什么不能实例化为什么有构造方法却不能被实例化)
- [堆外内存](#堆外内存)
- [自定义注解](#自定义注解)
- [Java的反射机制](#java的反射机制)

<!-- /TOC -->

# Java的三大特性封装、继承、多态

1. 封装    
封装就是将类的信息隐藏在类内部，不允许外部程序直接访问，而是通过该类的方法实现对隐藏信息的操作和访问。  
**如何实现**：          
（1）属性访问控制符为`private`（2）创建getter/setter方法（3）在getter/setter方法中添加属性访问的控制语句

2. 继承     
继承是从已有的类中派生出新的类，新的类能吸收已有类的数据属性和行为，并能扩展新的能力。

3. 多态     
多态指的是对象的多种形态。多态有两种：引用多态和方法多态。继承是多态的实现基础。
**引用多态**：父类的引用可以指向本类的对象；父类的引用可以指向子类的对象。  
**方法多态**：创建父类对象时，调用的方法为父类方法；创建子类对象时，调用的方法是子类重写的方法或继承自父类的方法；

# Java中的访问权限
||同类|同包类|不同包子类|不同包非子类|
|---|:---:|:---:|:---:|:---:|
|public|yes|yes|yes|yes|
|protected|yes|yes|yes|no|
|(default)不写|yes|yes|no|no|
|private|yse|no|no|no|

# 重载（overload）和重写(Override)
- **重载表现在同类中：** 方法名称相同，签名不同。
- **重写表现在父子类中：** 方法签名相同，通常会在子类重写的方法上加`@Override`注解，来表示这是一个重写的方法。当然不加这个注解也可以，他本质上还是方法的重写。

# super和this的区别
这里直接用知乎上的同学的回答，将的还是比较详细的：[原文参考这里](https://zhuanlan.zhihu.com/p/37609279)
1. super(参数)是调用基类中的某一个构造函数（构造函数的第一条语句）；this(参数)是调用本类中另一种形成的构造函数（构造函数中的第一条语句）
2. super引用当前对象的直接父类中的成员（用来访问直接父类中被隐藏的父类中成员数据或者函数，基类与派生类中有相同成员定义时如：super.变量名 super.成员函数名(实参)）；this代表当前对象名（在程序中产生二义性之处，应使用this来指明当前对象；如果函数的形参与类中的成员数据同名，这时需要用this来指明成员变量名）
3. 调用super()必须写在子类构造方法的第一行，否则编译不通过。每个子类构造方法的第一条语句，都是隐含地调用super()，如果父类没有这种形式的构造函数，那么在编译的时候就会报错。
4. super()和this()类似,区别是，super()从子类中调用父类的构造方法，this()在同一类内调用其它方法。
5. super()和this()均需放在构造方法内第一行。
6. 尽管可以用this调用一个构造器，但却不能调用两个。
7. this和super不能同时出现在一个构造函数里面，因为this必然会调用其它的构造函数，其它的构造函数必然也会有super语句的存在，所以在同一个构造函数里面有相同的语句，就失去了语句的意义，编译器也不会通过。
8. this()和super()都指的是对象，所以，均不可以在static环境中使用。包括：static变量,static方法，static语句块。
9. 从本质上讲，this是一个指向本对象的指针, 然而super是一个Java关键字。

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

# Java的异常处理
## 异常的分类
![Java的异常处理结构](http://sunyanping.gitee.io/it-keep/ASSET/Java的异常处理结构.png)
Throwable： 有两个重要的子类：Exception（异常）和 Error（错误），二者都是 Java 异常处理的重要子类，各自都包含大量子类。异常和错误的区别是：异常能被程序本身可以处理，错误是无法处理。  
**Error（错误）**:是程序无法处理的错误，表示运行应用程序中较严重问题。大多数错误与代码编写者执行的操作无关，而表示代码运行时 JVM（Java 虚拟机）出现的问题。例如，Java虚拟机运行错误（Virtual MachineError），当 JVM 不再有继续执行操作所需的内存资源时，将出现 OutOfMemoryError。这些异常发生时，Java虚拟机（JVM）一般会选择线程终止。这些错误表示故障发生于虚拟机自身、或者发生在虚拟机试图执行应用时，如Java虚拟机运行错误（Virtual MachineError）、类定义错误（NoClassDefFoundError）等。这些错误是不可查的，因为它们在应用程序的控制和处理能力之 外，而且绝大多数是程序运行时不允许出现的状况。对于设计合理的应用程序来说，即使确实发生了错误，本质上也不应该试图去处理它所引起的异常状况。在 Java中，错误通过Error的子类描述。   
**Exception（异常）**:是程序本身可以处理的异常。Exception 类有一个重要的子类 RuntimeException。RuntimeException 类及其子类表示“JVM 常用操作”引发的错误。例如，若试图使用空值对象引用、除数为零或数组越界，则分别引发运行时异常（NullPointerException、ArithmeticException）和 ArrayIndexOutOfBoundException。
Exception（异常）分两大类：运行时异常和非运行时异常(编译异常)。程序中应当尽可能去处理这些异常。
1. 运行时异常：都是RuntimeException类及其子类异常，如NullPointerException(空指针异常)、IndexOutOfBoundsException(下标越界异常)等，这些异常是不检查异常，程序中可以选择捕获处理，也可以不处理。这些异常一般是由程序逻辑错误引起的，程序应该从逻辑角度尽可能避免这类异常的发生。运行时异常的特点是Java编译器不会检查它，也就是说，当程序中可能出现这类异常，即使没有用try-catch语句捕获它，也没有用throws子句声明抛出它，也会编译通过。
2. 非运行时异常 （编译异常）：是RuntimeException以外的异常，类型上都属于Exception类及其子类。从程序语法角度讲是必须进行处理的异常，如果不处理，程序就不能编译通过。如IOException、SQLException等以及用户自定义的Exception异常，一般情况下不自定义检查异常。

通常，Java的异常（Throwable）分为可查的异常（checked exceptions）和不可查的异常（unchecked exceptions）。  
![](http://sunyanping.gitee.io/it-keep/ASSET/可捕获和不可捕获异常的分类.jpg)
1. 可查异常（编译器要求必须处置的异常）：正确的程序在运行中，很容易出现的、情理可容的异常状况。除了Exception中的RuntimeException及RuntimeException的子类以外，其他的Exception类及其子类(例如：IOException和ClassNotFoundException)都属于可查异常。这种异常的特点是Java编译器会检查它，也就是说，当程序中可能出现这类异常，要么用try-catch语句捕获它，要么用throws子句声明抛出它，否则编译不会通过。
2. 不可查异常(编译器不要求强制处置的异常):包括运行时异常（RuntimeException与其子类）和错误（Error）。RuntimeException表示编译器不会检查程序是否对RuntimeException作了处理，在程序中不必捕获RuntimException类型的异常，也不必在方法体声明抛出RuntimeException类。RuntimeException发生的时候，表示程序中出现了编程错误，所以应该找出错误修改程序，而不是去捕获RuntimeException。

## 异常的处理机制
1. 抛出异常：当一个方法出现错误引发异常时，方法创建异常对象并交付运行时系统，异常对象中包含了异常类型和异常出现时的程序状态等异常信息。运行时系统负责寻找处置异常的代码并执行。
2. 捕获异常：在方法抛出异常之后，运行时系统将转为寻找合适的异常处理器（exception handler）。潜在的异常处理器是异常发生时依次存留在调用栈中的方法的集合。当异常处理器所能处理的异常类型与方法抛出的异常类型相符时，即为合适 的异常处理器。运行时系统从发生异常的方法开始，依次回查调用栈中的方法，直至找到含有合适异常处理器的方法并执行。当运行时系统遍历调用栈而未找到合适 的异常处理器，则运行时系统终止。同时，意味着Java程序的终止。

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

## synchronized的实现原理简单描述
在synchronized修饰的代码块或者方法，编译后的字节码中方法的访问标记多了一个ACC_SYNCHRONIZED，该标记表示，进入该方法时，JVM需要进行monitorenter操作，退出该方法时，无论是否正常返回，JVM均需要进行monitorexit操作。当执行monitorenter时，如果目标对象的monitor计数器为0，表示此对象没有被任何其他对象所持有。此时，JVM会将该锁对象的持有线程设置为当前线程，并且将计数器+1；如果目标锁对象的计数器不为0，判断锁对象的持有线程是否是当前线程，如果是，再次将计数器+1（锁的可重入性）。如果锁对象的持有线程不是当前线程，当前线程需要等待，直至持有线程释放锁。当执行monitorexit时，JVM会将锁对象的计数器-1，当计数器值减为0时，代表该锁对象已经被释放。

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

# String的拼接，使用`+`还是StringBuilder或者StringBuffer
- String是线程安全的，StringBuilder是非线程安全的，StringBuffer是线程安全的。
- 当创建一个String串时首选使用`+`来进行拼接，这样实际上编译之后会创建一个`StringBuilder`对象来进行拼接，所以和StringBuilder拼接式没什么区别的，使用`+`代码相对来说简洁，并且不需要再做一次`toString()`转换。
- 如果是多线程的操作就使用StringBuffer。
- 但是注意当使用for循环类似的字符串拼接的话，使用`StringBuilder`来拼接，否则`str = str + "hello"`这种需要创建一个新的对象来接收结果，所以会创建很多个对象，性能较差，而使用StringBuilder只创建一个对象。

# 形参和实参

参数的传递包含两种：基本类型传参和引用类型传参。

我们创建方法时的参数称为形参，调用方法时传的参数称为实参，就是把实参传递给了形参。在方法中使用时实际使用的是形参。在传递基本类型的时候，传递的是值，比如`int =10`，传给形参的是实际的值。在传递引用类型的时候，传递的是引用，所以调用时的参数的指针指向和形参的指针指向时相同的地址。

所以，在被调用的函数中形参是基本类型时，形参的改变是和实参没有关系的；但是形参是引用类型时，形参的改变也会引起实参的改变。

有几个需要注意的点：  
- 基本类型是值传递没问题  
- String类型是引用类型，为什么结果表现出来去和值传递相似呢。（1）String类型不可变（2）String的底层实际上是将字符存放在`char[]`数组，数组也是不可变。  
- 数组也表现出来的是值传递的现象，理由如上。  
实际上java中是不存在引用传递的，只有值传递。只不过表现为引用传递的，是因为实参的引用指向了，形参的拷贝上。

# 构造器
构造器是类中的一个特殊方法，方法名和类名相同，没有返回值类型，也不能使用`void`来声明返回值类型。  

**构造器的调用**
- 构造器的存在是为了类的初始化，不用主动调用，当使用`new`关键字创建类时会自动调用。   
- 构造器在Java中也有抽象，对应的类是`Constructor`，使用该类的`newInstance()`方法进行创建的类，同样也会自动调用类的构造方法。  
- 当一个子类继承父类时，如果创建子类的对象时，调用了构造方法，首先会调用父类的构造方法，可以显式调用（手写`super()`方法，可以调用无参构造，也可以调用有参构造），也可以隐式调用（当不显式调用时，默认在子类的构造方法的第一行会有`super()`方法，这样只能调用父类的无参构造）。

# 为什么子类构造方法默认第一行调用父类构造方法
因为子类继承了父类的属性和方法，父类的属性的初始化工作需要父类的构造方法来做，所以子类的构造方法第一行默认调用父类的构造方法，是为了**帮助子类做初始化工作**。

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

# Java中线程状态的切换
1. 初始(NEW)：新创建了一个线程对象，但还没有调用start()方法。    
2. 运行(RUNNABLE)：Java线程中将就绪（ready）和运行中（running）两种状态笼统的称为“运行”。
线程对象创建后，其他线程(比如main线程）调用了该对象的start()方法。该状态的线程位于可运行线程池中，等待被线程调度选中，获取CPU的使用权，此时处于就绪状态（ready）。就绪状态的线程在获得CPU时间片后变为运行中状态（running）。  
3. 阻塞(BLOCKED)：表示线程阻塞于锁。  
4. 等待(WAITING)：进入该状态的线程需要等待其他线程做出一些特定动作（通知或中断）。  
5. 超时等待(TIMED_WAITING)：该状态不同于WAITING，它可以在指定的时间后自行返回。  
6. 终止(TERMINATED)：表示该线程已经执行完毕。  

![](http://sunyanping.gitee.io/it-keep/ASSET/Java的线程状态.jpg)

# `volatile`关键字
volatile关键字，用来确保将变量的更新操作通知到其他线程。当把变量声明为volatile类型后，编译器与运行时都会注意到这个变量是共享的，因此不会将该变量上的操作与其他内存操作一起重排序。volatile变量不会被缓存在寄存器或者对其他处理器不可见的地方，因此在读取volatile类型的变量时总会返回最新写入的值。  
在访问volatile变量时不会执行加锁操作，因此也就不会使执行线程阻塞，因此volatile变量是一种比sychronized关键字更轻量级的同步机制。  
![](http://sunyanping.gitee.io/it-keep/ASSET/CPU缓存.png)  
当对非 volatile 变量进行读写的时候，每个线程先从内存拷贝变量到CPU缓存中。如果计算机有多个CPU，每个线程可能在不同的CPU上被处理，这意味着每个线程可以拷贝到不同的 CPU cache 中。  
而声明变量是 volatile 的，JVM 保证了每次读变量都从内存中读，跳过 CPU cache 这一步。

**当一个变量定义为 volatile 之后，将具备两种特性：**  
1. 保证此变量对所有的线程的可见性，这里的“可见性”，如本文开头所述，当一个线程修改了这个变量的值，volatile 保证了新值能立即同步到主内存，以及每次使用前立即从主内存刷新。但普通变量做不到这点，普通变量的值在线程间传递均需要通过主内存（详见：Java内存模型）来完成。
2. 禁止指令重排序优化。有volatile修饰的变量，赋值后多执行了一个“load addl $0x0, (%esp)”操作，这个操作相当于一个内存屏障（指令重排序时不能把后面的指令重排序到内存屏障之前的位置），只有一个CPU访问内存时，并不需要内存屏障；（什么是指令重排序：是指CPU采用了允许将多条指令不按程序规定的顺序分开发送给各相应电路单元处理）。

**volatile 性能：**  
volatile 的读性能消耗与普通变量几乎相同，但是写操作稍慢，因为它需要在本地代码中插入许多内存屏障指令来保证处理器不发生乱序执行。

## 补充：Java的内存模型  
Java虚拟机规范试图定义一种Java内存模型，以屏蔽各种硬件和操作系统操作内存的差异，来保证Java程序在各个平台都能达到一致的访问效果。   

Java内存模型的主要目标就是定义程序中各个变量的访问规则，即在虚拟机中将变量存储到内存和从内存读取变量的底层细节。这里说的变量包含实例字段、静态字段和构成数组对象的元素，不包含局部变量和方法参数，因为这2个是线程私有的并不存在线程安全的问题，也就不会有资源共享的问题。   

Java内存模型设计将这些变量存到主内存，每个线程自己私有的变量存到自己的工作内存，然后线程的工作内存中保存了主内存的副本拷贝，线程对所有的变量的操作都在工作内存中完成。如果线程间的变量转递就都必须通过主内存才能完成。比如实例变量a=0，在线程A、B同时操作下，肯定是工作内存修改完成之后然后再修改主内存的值，这样如果并发执行那就产生资源竞争的问题，导致产生和预期不一致的结果。   
![Java内存模型](http://sunyanping.gitee.io/it-keep/ASSET/Java内存模型.png)

![Java内存模型1](http://sunyanping.gitee.io/it-keep/ASSET/JVM内存模型.png)


**上述讲的`volatile`关键字声明的变量就是解决这类问题的。`volatile`声明的变量是对所有线程具有可见性的，对该变量的读取总是最新写入的值。**

# 抽象类和接口的区别
1. 抽象类和接口**都不能实例化**
2. 抽象类中可以有变量、抽象/非抽象方法，接口中只能有常量（`public static final`）、抽象方法(`public abstract`)。
3. 抽象类中可以有构造方法，接口中不能有构造方法。

## 抽象类有构造方法为什么不能实例化？为什么有构造方法却不能被实例化？
**为什么不能实例化**        
1. 实践角度： 一般来说实践中会将一部分属性/方法封装到类中，但是这些属性/方法不能满足我们创建实例的要求，所以我们考虑将这个类设计为抽象类。然后在实现类中扩展其他的属性方法以满足需要。所以设计为抽象类时就意味着这个类不需要实例化。
2. 技术角度：抽象类中包含抽象方法，当实例化时无法为抽象方法分配内存空间，并且抽象方法不实现，当调用时也会产生异常，所以抽象类就设计为不允许实例化。  
       
**不能实例化但是又有构造方法，这有什么意义**        
因为抽象类中仍然可以包含属性/方法，实现类若要想在实例化时为这些属性赋值，那就需要使用父类的构造方法。我们使用显示的super()方法来调用父类的构造器/方法。         
总的来说：我认为只要包含属性的类中，都必须存在构造方法，这才是合理的设计。

# 堆外内存
**直接的文件拷贝操作，或者I/O操作是系统内核内存和设备间的通信，而不是通过程序直接和外设通信的。**       
所以在Java程序中，JVM的I/O操作时依靠**堆外内存**的，堆外内存不受JVM的控制。I/O操作需要JVM将变量拷贝到堆外内存，然后从堆外内存将变量写/读磁盘。          
![堆外内存](http://sunyanping.gitee.io/it-keep/ASSET/堆外内存.png)      
在java中提供了2种操作堆外内存的API: `unsafe`(不对外)、`ByteBuffer`(在NIO框架中多有用到)

比如在Netty框架中，使用了`ByteBuffer`，只是他将此封装了，提供了一个`ByteBuf`的API来进行使用。这也是Netty框架中0拷贝思想的基础实现。

# 自定义注解
Java注解也是一个`.java`文件，由`@interface`关键字来修饰的一个Java类。自定义注解时需要使用到Java中提供的元注解，也就是写在@interface上方的那些注解。
- `@Documented`: 用于标记在生成Java doc时是否将该注解包含进去。
- `@Target`: 用于指定该注解使用的地方，TYPE-类、接口、枚举/FIELD-属性/METHOD-方法/PARAMETER-参数/CONSTRUCTOR-构造方法/LOCAL_VARIABLE-局部变量/ANNOTATION_TYPE-注释类型声明/PACKAGE-包声明。
- `@Inherited`: 允许子类继承父类中的注解。
- `@Retention`: 定义注解的存活阶段，源码级别（SOURCE）、编译级别(字节码级别)(CLASS)、运行时级别(RUNTIME)。
- `@Constraint`: 用于校验属性值是否合法。

注解的内容的语法格式： `数据类型 属性名() default 默认值`。

自定义注解在实践中常用的场景：AOP场景、权限控制、统一处理某类业务...

# Java的反射机制
> 有这么一句话：Java中一切皆对象

Java的三大特性之一：封装，封装是一切的基础，把所有的东西都抽象化，封装成一个个的类对象。包含封装成的这些类对象也不例外，这些类对象在Java中也被抽象化成`Class`对象，在`Class`中包含了：`Field`(抽象的是类属性)、`Method`(抽象的是类方法)、`Constractor`(抽象的是类构造方法)。同时也提供了这些对象的操作方法，包含修改、执行等。类对象最终的执行是要加载在内存中的，这些本来是一些静态的东西，但是有了`Class`对象之后，这些本来是静态的东西也可以被在执行时进行动态的修改。这中手段称之为反射(`reflection`)。

和反射操作相关的API放在`java.lang.reflect`包下，通常使用的一些API如下：
- **获取类(Class)对象**
```
Class clazz = Class.forName("xxx.ClassName"); 
```

- **构造器(Constactor)相关**
```
Class c = Class.forName("cn.qidd.Person");
// 获取到它的所有构造方法
// 1-获取到所有公共构造方法，返回数组
Constructor[] constructors = c.getConstructors();
System.out.println(Arrays.toString(constructors));
// 2-获取到所有构造方法，返回数组
Constructor[] declaredConstructors = c.getDeclaredConstructors();
System.out.println(Arrays.toString(declaredConstructors));
// 3-获取到公有构造方法，返回构造器对象。根据参数类型指定构造器。
// Constructor getConstructor(Class<?> ...parameterType)
Constructor constructor = c.getConstructor(String.class, int.class);
System.out.println(constructor);
// 4-通过获取到的构造方法，创建类实例
Object obj = constructor.newInstance("张三", 18);
System.out.println(obj);
// 5-获取到私有构造方法返回构造器。并创建类实例。
Constructor con = c.getDeclaredConstructor(String.class);
con.setAccessible(true); // 当不设置时非法异常，此处意思是指示使用Java反射的对象在使用时取消Java语言的访问权限
Object o1 = con.newInstance("张三");
System.out.println(o1);
```

- **属性(Field)相关**
```
Class c = Class.forName("cn.qidd.Person");
Field[] fields = c.getFields(); // 获取到所有公共成员变量
Field[] fields1 = c.getDeclaredFields(); // 获取到所有成员变量
Field f = c.getField("address"); // 获取某公共成员变量
Field f2 = c.getDeclaredField("name"); // 获取任意一个成员变量
Constructor constructor = c.getConstructor();
Object o = constructor.newInstance();
f2.setAccessible(true);
f2.set(o, "张三");
System.out.println(o);
```

- **方法(Method)相关**

```
Class c = Class.forName("cn.qidd.Person");
// 获取所有公共的方法（将他的父类的公共的方法也拿到）
Method[] methods = c.getMethods();
// 获取所有的方法（只有自己的），包含私有的
Method[] methods1 = c.getDeclaredMethods();
// 获取单个方法并使用（public）
Constructor con = c.getConstructor();
Object o = con.newInstance();

Method method = c.getMethod("show");
method.setAccessible(true);
Object invoke = method.invoke(o); // 调用o的show方法
// 获取单个方法（private修饰）
Method m = c.getDeclaredMethod("function",String.class);
m.setAccessible(true);
Object o1 = m.invoke(o, "孙艳平");
System.out.println(o1);
```
