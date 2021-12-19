# 单例模式

> Singleton。

## 应用场景

主需要有一个类的实例即可。

- 各种各样的Factory。
- 数据库的DataSource对象。
- ...

## 核心思想

- 私有化构造器。

## 实现

### 饿汉式单例

- **一般场景下推荐**。

- JVM 保证线程安全，每个class文件只会load到内存中一次。
- 静态变量会在Class对象load到内存之后完成实例化。
- `缺点` 不管用到与否，类装载时就完成实例化。

```java
public class HungarySingleton {

    private static final HungarySingleton INSTANCE = new HungarySingleton();

    private HungarySingleton() {
    }

    public static HungarySingleton getInstance() {
        return INSTANCE;
    }

    public static void main(String[] args) {
        HungarySingleton instance1 = HungarySingleton.getInstance();
        HungarySingleton instance2 = HungarySingleton.getInstance();
        System.out.println(instance1.hashCode());
        System.out.println(instance2.hashCode());
        System.out.println(instance1.equals(instance2));
    }
}
```

### 懒汉式单例

- 懒汉式的延迟初始化解决了直接初始化的问题，但是也带来了线程不安全的新问题。

```java
public class LazySingleton {

    private static LazySingleton INSTANCE = null;

    private LazySingleton() {
    }

    public static LazySingleton getInstance() {
        if (INSTANCE == null) {
            INSTANCE = new LazySingleton();
        }
        return INSTANCE;
    }

    public static void main(String[] args) {
        LazySingleton instance1 = LazySingleton.getInstance();
        LazySingleton instance2 = LazySingleton.getInstance();
        System.out.println(instance1.hashCode());
        System.out.println(instance2.hashCode());
        System.out.println(instance1.equals(instance2));
    }
}

```

### 双重检查单例

- **推荐**，在通过反序列化可以强制序列化对象问题出现之前被认为完美的实现方式。

- 通过双重检查，避免检查和创建实例的**非原子性**导致的问题。
- 添加`volatile`关键字，避免编译器`指令重排`导致的检查失效。

```java
public class DouleCheckSingleton {

    private static volatile DouleCheckSingleton INSTANCE = null;

    private DouleCheckSingleton() {
    }

    public static DouleCheckSingleton getInstance() throws InterruptedException {
        if (INSTANCE == null) {
            TimeUnit.SECONDS.sleep(1);
            synchronized (DouleCheckSingleton.class) {
                if (INSTANCE == null) {
                    INSTANCE = new DouleCheckSingleton();
                }
            }
        }
        return INSTANCE;
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(() -> {
                try {
                    System.out.println(DouleCheckSingleton.getInstance().hashCode());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }).start();

        }
    }
}
```

### 静态内部类的单例

- 推荐，由JVM保证单例的同时做到了延迟初始化。

- 静态内部类不会再外部类加载的时候进行初始化，会在第一次被调用的时候再进行初始化，避免了过早初始化的问题。

```java
public class InnerClassSingleton {

    private InnerClassSingleton() {
    }

    public static InnerClassSingleton getInstance() {
        return SingletonHolder.INSTANCE;
    }

    private static class SingletonHolder {
        private final static InnerClassSingleton INSTANCE = new InnerClassSingleton();
    }

    public static void main(String[] args) {
        InnerClassSingleton instance1 = InnerClassSingleton.getInstance();
        InnerClassSingleton instance2 = InnerClassSingleton.getInstance();
        System.out.println(instance1.hashCode());
        System.out.println(instance2.hashCode());
        System.out.println(instance1.equals(instance2));
    }
}
```

### 枚举单例

- 推荐，目前认为最安全的单例方式。
- 枚举类是`没有构造方法`的，即使获取到了`.class文件`或者`Class对象`，也无法进行序列化去实例一个对象。

```java
public enum EnumSingleton {
    
    INSTANCE;

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(() -> System.out.println(EnumSingleton.INSTANCE.hashCode())).start();
        }
    }

}
```

