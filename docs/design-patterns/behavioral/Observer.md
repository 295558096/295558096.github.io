# 观察者模式

>Observer。

## 定义

- 事件处理模型。
- 观察者模式和责任链模式的区别在于，观察者模式一般不会产生中断，而且将通知通知到各个观察者。
- 事件类要包含唤醒的源对象本身。
- hook钩子函数、call back、observer、listener

## 类图

![观察者模式的 UML 图](image/observer_pattern_uml_diagram.jpg)

## 代码

### Observer接口及实现类

```java
public interface Observer {

    void action(WakeEvent event);

}

public class Dad implements Observer{

    @Override
    public void action(WakeEvent event) {
        System.out.println("Dad feed baby.");
    }
}

public class Mum implements Observer{

    @Override
    public void action(WakeEvent event) {
        System.out.println("mum hung baby.");
    }
}

public class Dog implements Observer{

    @Override
    public void action(WakeEvent event) {
        System.out.println("dog wang wang wang.");
    }
}
```

### 唤醒事件类

```java
public class WakeEvent {

    public String msg;

    public Date time;

    public Baby source;

}
```



### 被观察的类

```java
public class Baby {

    List<Observer> observers = new ArrayList<>();

    {
        observers.add(new Dad());
        observers.add(new Mum());
        observers.add(new Dog());
    }

    void wakeUp() {
        WakeEvent event = new WakeEvent();
        event.msg = "baby wake";
        event.time = new Date();
	      event.source = this;
        System.out.println("baby wake up");
        for (Observer observer : observers) {
            observer.action(event);
        }
    }
}
```

### 客户端调用

```java
public class ObserverTest {

    public static void main(String[] args) {
        Baby baby = new Baby();
        baby.wakeUp();
    }

}
```

