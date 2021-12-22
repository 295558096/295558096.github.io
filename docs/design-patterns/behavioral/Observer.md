# 观察者模式

>Observer。

## 定义

- 事件处理模型。
- 观察者模式和责任链模式的区别在于，观察者模式一般不会产生中断，而且将通知通知到各个观察者。

## 类图

![观察者模式的 UML 图](image/observer_pattern_uml_diagram.jpg)

## 代码

### Observer接口及实现类

```java
public interface Observer {

    void action();

}

public class Dad implements Observer{

    @Override
    public void action() {
        System.out.println("Dad feed baby.");
    }
}

public class Mum implements Observer{

    @Override
    public void action() {
        System.out.println("mum hung baby.");
    }
}

public class Dog implements Observer{

    @Override
    public void action() {
        System.out.println("dog wang wang wang.");
    }
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
        System.out.println("baby wake up");
        for (Observer observer : observers) {
            observer.action();
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

