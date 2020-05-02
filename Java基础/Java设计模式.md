<!-- TOC -->

- [观察者模式](#观察者模式)

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

