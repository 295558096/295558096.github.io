# Java 面向对象

## 三大特性

### 封装

#### 简介

字面上来理解就是包装的意思，专业点就是**信息隐藏**，是在业务分析和数据建模的阶段，根据业务抽象出各种特定的类型。类型内部包含这该类数据相关的属性和方法，根据数据访问权限的不通，对这些方法、属性进行不同的权限限定。

由于权限的存在，数据被保护在抽象数据类型的内部，可以尽可能地隐藏内部的细节，对外提供一下**安全**、**可靠**的接口。

用户或者客户端**无需知道对象内部的细节**，可以通过对象提供的对外接口对对象进行访问。

#### 优点

- 良好的封装能够减少耦合。
- 类内部的结构可以自由修改。
- 可以对成员进行更精确的控制。
- 隐藏内部信息、隐藏内部实现细节。

### 继承

#### 简述

继承是使用已存在的类的定义作为基础建立新类的技术，新类的定义可以增加新的数据或新的功能，也可以用父类的功能，但不能选择性地继承父类。

继承实现了 **IS-A** 关系。

继承应该遵循**里氏替换原则**，子类对象必须能够替换掉所有父类对象。

继承者是被继承者的特殊化，它除了拥有被继承者的特性外，还拥有自己独有得特性。

#### 特点

- 子类拥有父类非private的属性和方法。
- 子类可以拥有自己属性和方法，即子类可以对父类进行扩展。
- 子类可以用自己的方式实现父类的方法。

#### 缺陷

- 父类变，子类就必须变。
- 继承破坏了封装，对于父类而言，它的实现细节对与子类来说都是透明的。
- 继承是一种强耦合关系。

#### 继承关键点——构造器

构造器在继承关系中，只能够被调用，不能被继承。 调用父类的构造方法我们使用`super()`即可。

构建过程是从父类“向外”扩散的，也就是**从父类开始向子类一级一级地完成构建**。

即使没有显示的引用父类的构造器，**编译器会默认给子类调用父类的构造器。**

父类需要有默认构造器。如果父类没有默认构造器，就必须显示的使用`super()`来调用父类构造器，并且需要在子类构造器方法的第一行进行调用，否则编译器会报错。

#### 继承关键点——protected关键字

继承的过程中，往往需要通过`protected`来控制属性的权限，达到进一步细粒度的控制访问权限的目的。

- 对于用户和客户端

  被`protected`修饰的方法和属性是和`private`一样的，无法调用或者访问。

- 对于同一个包下的其他类

  被`protected`修改的方法和属性是可以正常访问到的。

#### 继承关键点——向上转型

将子类转换成父类，在继承关系上面是**向上**移动的，所以一般称之为**向上转型**。由于向上转型是**从一个叫专用类型向较通用类型转换**，所以它总是**安全**的，唯一发生变化的可能就是属性和方法的丢失。

### 多态

>父类引用使用子类对象。

#### 简述

多态本身氛围编译时多态和运行多态。

编译时多态主要指方法的重载。

运行时多态指程序中定义的对象引用所指向的具体类型在运行期间才确定。我们常说的多态性，指的是运行时多态。

#### 关键因素

- 继承——在多态中必须存在有继承关系的子类和父类。
- 重写——子类对父类中某些方法进行重新定义，在调用这些方法时就会调用子类的方法。
- 向上转型——在多态中需要将子类的引用赋给父类对象，只有这样该引用才能够具备技能调用父类的方法和子类的方法。

#### 实现形式

##### 继承

Java的类是单继承的，所以类的继承关系要谨慎。

基于继承的实现机制主要表现在父类和继承该父类的一个或多个子类**对某些方法的重写**，**多个子类对同一方法的重写可以表现出不同的行为**。

如果父类是**抽象类**，那么**子类必须要实现父类中所有的抽象方法**，这样该父类所有的子类一定存在统一的对外接口，但其内部的具体实现可以各异。

##### 接口

Java的接口可以是多继承的。

继承是通过重写父类的同一方法的几个不同子类来体现的，那么就可就是**通过实现接口并覆盖接口中同一方法**的几不同的类体现的。

 在接口的多态中，指向接口的引用必须是指定这实现了该接口的一个类的实例程序，在运行时，根据对象引用的实际类型来执行对应的方法。

## 面向对象设计的SOLID原则

>S.O.L.I.D是面向对象设计和编程(OOD&OOP)中几个重要编码原则(Programming Priciple)的首字母缩写。

| 设计原则     | 全称                            | 缩写 |
| ------------ | ------------------------------- | ---- |
| 单一职责原则 | Single Responsibility Principle | SRP  |
| 开放封闭原则 | Open Closed Principle           | OCP  |
| 里氏替换原则 | Liskov Substitution Principle   | LSP  |
| 接口隔离原则 | Interface Segregation Principle | ISP  |
| 依赖倒置原则 | Dependency Inversion Principle  | DIP  |

### 单一职责原则

>THERE SHOULD NEVER BE MORE THAN ONE REASON FOR A CLASS TO CHANGE.

#### 简述

确保某个类被修改的原因有且只有一个。换句话说就是让一个类只做一种类型责任，当这个类需要承当其他类型的责任的时候，就需要分解这个类。

#### 实质

解耦和增强内聚性。

#### 目标

不同的职责封装到不同的类或模块中。

#### 建议

做好设计再进行编码实现，避免编程过程中不经意导致的职责扩散。

### 开放封闭原则

#### 简述

软件实体应该是可扩展，而不可修改的。即对扩展是开放的，而对修改是封闭的。

- 对扩展开放，意味着有新的需求或变化时，可以对现有代码进行扩展，以适应新的情况。
- 对修改封闭，意味着类一旦设计完成，就可以独立完成其工作，而不要对类进行任何修改。

#### 实质

所有面向对象原则的核心。

#### 目标

封装变化，降低耦合。

#### 建议

- 开放封闭原则，是最为重要的设计原则，**Liskov替换原则**和**合成/聚合复用原则**为开放封闭原则的实现提供保证。
- 可以通过**Template Method模式**和**Strategy模式**进行重构，实现**对修改封闭、对扩展开放**的设计思路。
- **封装变化**，是实现开放封闭原则的重要手段，对于经常发生变化的状态一般将其封装为一个抽象。
- **拒绝滥用抽象**，只将经常变化的部分进行抽象，这种经验可以从设计模式的学习与应用中获得。

### 里氏替换原则

>对子类型的特别定义。

#### 简介

由芭芭拉·利斯科夫（Barbara Liskov）在1987年在一次会议上名为“数据的抽象与层次”的演说中首先提出。一个子类的实例在任何情况下都应该能够替换任何其超类的实例。

#### 实质

子类可以扩展父类的功能，但不能改变父类原有的功能。

#### 目标

派生类（子类）对象可以在程式中代替其基类（超类）对象。

#### 建议

- 子类可以实现父类的抽象方法，但不能覆盖父类的非抽象方法。
- 子类中可以增加自己特有的方法。
- 当子类的方法**重载父类的方法**时，方法的**前置条件**`形参`要比父类方法的输入参数更**宽松**。
- 当子类的方法**实现父类的抽象方法**时，方法的**后置条件**`返回值`要比父类更**严格**。

### 接口隔离原则

#### 简介

客户端不应该依赖它不需要的接口。一个类对另一个类的依赖应该建立在最小的接口上。如果强迫用户使用它们不使用的方法，那么这些客户就会面临由于这些不使用的方法的改变所带来的改变。

#### 目标

避免接口污染。

#### 建议

- 提供调用者需要的方法，屏蔽其不需要的方法。

### 依赖倒置原则

#### 简介

程序要依赖于抽象接口，不要依赖于具体实现。

#### 实质

面向抽象进行编程。

#### 目标

面向接口编程。

#### 建议

- 高层模块不应该依赖于低层模块，二者都应该依赖于抽象。
- 抽象不应该依赖于细节，细节应该依赖于抽象。