
_Gang of Four（四人帮）_，经典书籍《Design Patterns: Elements of Reusable Object-Oriented Software》总结的 23 种经典模式

# 创建型五种

## 单例模式Singleton

保证一个类只有一个实例，并提供全局访问点。比如，缓存，日志

``` java
class Singleton {

    private static Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
加载类的时候，静态方法直接加载到方法区里，那这个静态方法的Singleton实体应该是放在堆空间，然后有一个引用！这里对方法区，堆空间有点混乱

1. **Singleton 对象实体**（`new Singleton()` 创建出的数据）： 永远存放在**堆空间 (Heap)**。
2. **`instance` 静态引用变量**（指向上述对象的指针）：JDK 7 及以后**：类的静态变量和 `java.lang.Class` 对象一起存放在**堆空间 (Heap)**。
3. **静态方法代码与类元数据**（`getInstance()` 的执行逻辑、类结构等）： 存放在**方法区 (Method Area)** 中。

**关于“方法区”的补充说明：** “方法区”只是 JVM 规范中的一个**逻辑概念**，JDK 8 及以后，永久代被移除，方法区由**元空间 (Metaspace)** 实现，且使用的是操作系统的本地内存。

**⚠️ 额外的致命问题（线程安全）：** 这段“懒汉式”单例代码**不是线程安全的**。如果有两个线程同时执行到 `if (instance == null)`，它们可能都会判断为 true，从而在堆中 `new` 出两个不同的对象。 _建议修改为**双重检查锁 (Double-Checked Locking，需加 `volatile`)** 或使用**静态内部类**来实现安全的懒加载。_





## 工厂方法模式


## 抽象工厂模式


## 建造者模式


## 原型模式





# 结构性七种



# 行为型十一种