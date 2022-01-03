# 构造者模式

>Builder。

## 定义

- **建造者模式（Builder）**，将一个复杂的对象的**构建**与它的**表示**分离，使得同样的构建过程可以创建不同的表示。
- **与工厂模式的区别，建造者模式更关注于零件装配的顺序**。

## 使用场景

- 一个基本部件不会变，而其组合经常变化的时候。
- 构造复杂对象。

## 优点

- 具有良好的封装性，客户端不必知道产品内部组成的细节。
- 建造者独立，易扩展。
- 便于控制细节风险。可以对建造过程逐步细化，而不对其他模块产生任何影响。

## 缺点

- 产品必须有共同点，范围有限制。
- 如果内部变化复杂，会有很多建造类。

## 类图

![1018770-20190618194851222-1919518130](image/1018770-20190618194851222-1919518130-20220103200924655.png)

## 代码

### Director类

- 导演类起到封装的作用，避免高层模块深入到建造者内部的实现类。在建造者模式比较庞大时，导演类可以有多个。

```java
public class Director {

    public void Construct(Builder builder) {
        builder.BuildPartA();
        builder.BuildPartB();
    }

}
```

### Builder类

-  抽象建造者类，确定产品由两个部件PartA和PartB组成，并声明一个得到产品建造后结果的方法getResult()。

```java
public abstract class Builder {
    
    public abstract void BuildPartA();        //产品的A部件
    public abstract void BuildPartB();        //产品的B部件
    public abstract Product getResult();    //获取产品建造后结果

}
```

### ConcreteBuilder类

-  具体建造者类，有几个产品类就有几个具体的建造者，而且这多个产品类具有相同的接口或抽象类。这里给出一个产品类的样例，多个产品类同理。

```java
public class ConcreteBuilder1 extends Builder {

    private Product product = new Product();

    //设置产品零件
    @Override
    public void BuildPartA() {
        product.add("部件A");
    }

    @Override
    public void BuildPartB() {
        product.add("部件B");
    }

    //组建一个产品
    @Override
    public Product getResult() {
        return product;
    }

}
```

### Client客户端

```java
public class Client {

    public static void main(String[] args) {

        Director director = new Director();
        Builder builder1 = new ConcreteBuilder1();
        Builder builder2 = new ConcreteBuilder2();

        //指挥者用ConcreteBuilder1的方法来建造产品
        director.Construct(builder1);
        Product product1 = builder1.getResult();
        product1.show();

        //指挥者用ConcreteBuilder2的方法来建造产品
        director.Construct(builder2);
        Product product2 = builder2.getResult();
        product2.show();

    }

}
```

