
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
加载类的时候，静态方法直接加载到方法区里，那这个静态方法的Singleton实体应该是放在堆空间，然后有一个引用！


## 工厂方法模式


## 抽象工厂模式


## 建造者模式


## 原型模式





# 结构性七种



# 行为型十一种