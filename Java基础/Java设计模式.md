<!-- TOC -->

- [观察者模式](#观察者模式)
- [代理模式](#代理模式)
  - [静态代理](#静态代理)
  - [动态代理](#动态代理)

<!-- /TOC -->

# 观察者模式
观察者模式比较常见的适用的场景：广播、通知等。例如Eureka实现的服务发现。
![观察者模式](http://sunyanping.gitee.io/it-keep/ASSET/观察者模式.jpg)
Java内置了相关的接口和抽象类：`Observer`、`Observable`，在使用时直接继承/实现即可.  
*被观察者*
```
public class BallObservable extends Observable {
    private Map<String, String> map = new HashMap<>();
    public void setMap(Map<String, String> map) {
        this.map = map;
        setChanged();
        notifyObservers();
    }
    public Map<String, String> getMap() {
        return map;
    }
}
```
*观察者*
```
public class PeopleObserver implements Observer {
    @Override
    public void update(Observable o, Object arg) {
        BallObservable ballObservable = (BallObservable) o;
        Map<String, String> map = ballObservable.getMap();
        System.out.println(JSON.toJSON(map));
    }
}
```
*测试代码*
```
    public static void main(String[] args) throws InterruptedException {
        BallObservable ballObservable = new BallObservable();
        ballObservable.addObserver(new PeopleObserver());

        Map<String, String> map = new HashMap<>();
        map.put("a", "1");
        ballObservable.setMap(map);   // 当调用setMap()方法改变ballObservable对象的属性值时，自动执行PeopleObserver中的update方法

        Thread.sleep(2000);
        map.put("b", "2");
        ballObservable.setMap(map);

        Thread.sleep(2000);
        map.put("c", "3");
        ballObservable.setMap(map);
    }
```

# 代理模式
**Proxy代理模式是一种结构型设计模式，主要解决的问题是：在直接访问对象时带来的问题。** 

代理是一种常用的设计模式，其目的就是为其他对象提供一个代理以控制对某个对象的访问。代理类负责为委托类预处理消息，过滤消息并转发消息，以及进行消息被委托类执行后的后续处理。
![](http://sunyanping.gitee.io/it-keep/ASSET/代理模式结构.png)

为了保持行为的一致性，代理类和委托类通常会实现相同的接口，所以在访问者看来两者没有丝毫的区别。通过代理类这中间一层，能有效控制对委托类对象的直接访问，也可以很好地隐藏和保护委托类对象，同时也为实施不同控制策略预留了空间，从而在设计上获得了更大的灵活性。

更通俗的说，代理解决的问题当两个类需要通信时，引入第三方代理类，将两个类的关系解耦，让我们只了解代理类即可，而且代理的出现还可以让我们完成与另一个类之间的关系的统一管理，但是切记，代理类和委托类要实现相同的接口，因为代理真正调用的还是委托类的方法。

按照代理的创建时期，代理类可以分为两种：        
- **静态**：由程序员创建代理类或特定工具自动生成源代码再对其编译。在程序运行前代理类的.class文件就已经存在了。      
- **动态**：在程序运行时运用反射机制动态创建而成。

## 静态代理
创建接口控制的实现对象和代理对象的一致行为
```
public interface IUserDao {
    void save();
}
```

实现类
```
public class UserDaoImpl implements IUserDao {
    @Override
    public void save() {
        System.out.println("保存数据");
    }
}
```

代理类
```
public class UserDaoProxy implements IUserDao{
    // 代理的目标对象
    private IUserDao target;
    // 将需要代理的目标对象传入
    public UserDaoProxy(IUserDao userDao) {
        this.target = userDao;
    }

    @Override
    public void save() {
        System.out.println("开启事务");
        target.save();
        System.out.println("提交事务");
    }
}
```

客户端调用
```
public static void main(String[] args) {
    IUserDao proxy = new UserDaoProxy(new UserDaoImpl());
    proxy.save();
}
```
      
**优点：**           
代理使客户端不需要知道实现类是什么，怎么做的，而客户端只需知道代理即可（解耦合）。

**缺点：**      
（1）代理类和实现类需要实现相同的接口，这样就会导致很多的重复代码，如果接口增加一个方法，每个实现类都得实现这个方法，并且代理类也得实现。增加了代码的维护难度。
（2）代理对象只服务一种类型的对象，要服务多种不同类型的对象，就需要创建多种不同类型的代理。如果要代理的对象较多那就非常麻烦。


## 动态代理
动态代理类只能代理一种类型的对象，若要代理不同类型的对象，就需要创建多个代理，在开发中是非常麻烦的事情。所以就需要找出一种只需要一个代理就可以代理不同类型对象的办法，而这种办法就是使用Java中的反射，在运行时根据不同类型对象做出不同的动作。这种方法称之为反射。

Java中要想实现动态代理机制，需要`java.lang.reflect.InvocationHandler`接口和 `java.lang.reflect.Proxy` 类的支持。

这种设计模式代码架构示例展示如下：

代理类
```
public class ProxyFactory implements InvocationHandler {
    // 被代理的对象
    private Object target;
    // 给被代理的对象赋值
    public ProxyFactory(Object target) {
        this.target = target;
    }
    /**
     * @param proxy 代理
     * @param method 真实实现需要执行的方法
     * @param args 方法参数
     * @return
     * @throws Throwable
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("开启事务");
        method.invoke(target, args);
        System.out.println("提交事务");
        return null;
    }

    /**
     * 生成代理对象
     * @param target
     * @return
     */
    public static Object getProxyInstance(Object target) {
        return Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), new ProxyFactory(target));
    }
}
```

2个不同类型的对象：
```
public class OrderDaoImpl implements IOrderDao {
    @Override
    public void save() {
        System.out.println("保存数据..");
    }
}
```

```
public class UserDaoImpl implements IUserDao {
    @Override
    public void save() {
        System.out.println("保存数据");
    }
}
```

客户端调用
```
public static void main(String[] args) {
    IUserDao proxyInstance = (IUserDao) ProxyFactory.getProxyInstance(new UserDaoImpl());
    proxyInstance.save();

    IOrderDao instance = (IOrderDao)ProxyFactory.getProxyInstance(new OrderDaoImpl());
    instance.save();
}
```

动态代理解决了静态代理的缺点问题，只用一个代理就能够代理多个不同类型的对象。