# 策略模式

> Strategy。

## 接口 Comparable

- **可比较**的接口。
- Comparable接口包含了一个`compareTo()`方法用来进行类的比较。
- 如果一个类有**多种**比较方式，仅仅实现Comparable接口是没办法做的的。

## 接口 Comparator

- 比较器接口。
- 体现了设计原则里的开闭原则。

## 类图

![策略模式类图](image/%E7%AD%96%E7%95%A5%E6%A8%A1%E5%BC%8F%E7%B1%BB%E5%9B%BE.png)