
_Gang of Four（四人帮）_，经典书籍《Design Patterns: Elements of Reusable Object-Oriented Software》总结的 23 种经典模式

# 创建型五种

核心解决：**对象怎么创建。** 一句话：**大家都用同一个对象。**

## 单例模式 Singleton

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
2. **`instance` 静态引用变量**（指向上述对象的指针）：JDK 7 及以后 ,类的静态变量和 `java.lang.Class` 对象一起存放在**堆空间 (Heap)**。
3. **静态方法代码与类元数据**（`getInstance()` 的执行逻辑、类结构等）： 存放在**方法区 (Method Area)** 中
4. 方法调用的临时变量，操作数，都保存在当前线程的临时栈里。

**关于“方法区”的补充说明：** “方法区”只是 JVM 规范中的一个**逻辑概念**，JDK 8 及以后，永久代被移除，方法区由**元空间 (Metaspace)** 实现，且使用的是操作系统的本地内存。

**⚠️ 额外的致命问题（线程安全）：** 这段“懒汉式”单例代码**不是线程安全的**。如果有两个线程同时执行到 `if (instance == null)`，它们可能都会判断为 true，从而在堆中 `new` 出两个不同的对象。 _建议修改为**双重检查锁 (Double-Checked Locking，需加 `volatile`)** 或使用**静态内部类**来实现安全的懒加载。_
``` java
class Singleton {
    private Singleton() {}

    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}

```




## 工厂方法模式 Factory Method

 **把“创建什么对象”交给子类/具体工厂决定， 把对象创建延迟到具体工厂。

**定义一个创建对象的接口，但由子类决定要实例化的类是哪一个。** 它将对象的创建推迟到了子类。



## 抽象工厂模式


## 建造者模式


## 原型模式





# 结构性七种



# 行为型十一种



****

例如：

```text
AnimalFactory
    │
    ├── DogFactory → Dog
    │
    └── CatFactory → Cat
```

你不需要直接：

```java
new Dog();
```

而是：

```java
factory.createAnimal();
```

一句话：

> 

---

### 3. 抽象工厂 Abstract Factory

> **一次创建一整套相关对象。**

例如电脑厂商：

```text
AppleFactory
 ├── createCPU()
 ├── createMemory()
 └── createScreen()
```

另一家：

```text
HuaweiFactory
 ├── createCPU()
 ├── createMemory()
 └── createScreen()
```

重点是：

> **创建一族相关产品。**

---

### 4. 建造者 Builder

> **把一个复杂对象的创建过程拆开，一步一步构建。**

比如：

```java
User user = new User.Builder()
        .name("Tom")
        .age(20)
        .address("Tokyo")
        .phone("123")
        .build();
```

特别适合：

```text
参数很多
对象构造复杂
有很多可选参数
```

一句话：

> **一步一步把复杂对象造出来。**

---

### 5. 原型 Prototype

> **通过复制已有对象创建新对象，而不是重新 new。**

例如：

```java
User user2 = user1.clone();
```

适合：

```text
创建对象成本比较高
而且新对象和旧对象结构非常相似
```

一句话：

> **照着已有对象复制一个。**

---

# 二、结构型模式：7种

核心解决：

> **类和对象之间怎么组织、组合。**

---

### 6. 适配器 Adapter

> **把一个接口转换成另一个接口。**

现实例子：

```text
中国插头
   ↓
转换器
   ↓
日本插座
```

Java：

```text
旧接口
   ↓
Adapter
   ↓
新接口
```

一句话：

> **接口不兼容？我帮你转换一下。**

---

### 7. 装饰器 Decorator

> **不修改原来的类，动态增加功能。**

例如：

```text
InputStream
    ↓
BufferedInputStream
    ↓
DataInputStream
```

一层一层增加功能。

类似：

```text
普通咖啡
 ↓
+ 牛奶
 ↓
+ 糖
 ↓
+ 奶油
```

一句话：

> **给对象套一层，再增加功能。**

---

### 8. 代理 Proxy

这个你刚刚已经学得很深入了。

> **为真实对象提供一个代理对象，在访问真实对象前后增加额外功能。**

```text
调用者
  ↓
Proxy
  ↓
日志 / 权限 / 事务
  ↓
真实对象
```

Java里面：

```text
静态代理
JDK动态代理
CGLIB
```

都是代理思想的具体实现方式。

一句话：

> **我不让你直接找真实对象，我先替它接待你。**

---

### 9. 外观 Facade

> **给复杂系统提供一个简单入口。**

例如你有：

```text
订单系统
库存系统
支付系统
物流系统
```

用户本来要：

```text
创建订单
 ↓
扣库存
 ↓
支付
 ↓
通知物流
```

现在提供：

```java
orderFacade.createOrder();
```

内部帮你完成全部操作。

一句话：

> **复杂系统，我给你一个简单门面。**

---

### 10. 桥接 Bridge

> **把抽象和实现分离，让它们可以独立变化。**

例如：

```text
形状
├── 圆
└── 方

颜色
├── 红
└── 蓝
```

如果直接组合：

```text
红圆
蓝圆
红方
蓝方
```

组合爆炸。

桥接：

```text
Shape
  ↓
Color
```

两个维度独立变化。

一句话：

> **把两个容易变化的维度拆开。**

---

### 11. 组合 Composite

> **把单个对象和对象集合统一对待。**

典型例子：

```text
文件夹
├── 文件
├── 文件
└── 文件夹
     ├── 文件
     └── 文件
```

文件和文件夹都可以：

```java
display();
```

所以：

```text
叶子节点
+
容器节点
```

统一成一个接口。

一句话：

> **让“一个”和“一群”用同一种方式操作。**

---

### 12. 享元 Flyweight

> **大量对象有相同数据时，把公共数据共享起来，减少对象数量。**

比如：

```text
100万个棋子
```

没必要每个棋子都保存：

```text
颜色
字体
图片
...
```

可以共享：

```text
共享对象
  ↓
字体
颜色
图片

每个对象只保存自己的：
  ↓
位置
状态
```

一句话：

> **能共享的就别重复创建。**

---

# 三、行为型模式：11种

核心解决：

> **对象之间怎么协作、怎么分配职责。**

---

### 13. 模板方法 Template Method

> **父类规定算法的大致流程，子类实现具体步骤。**

例如：

```java
public void process() {

    step1();

    step2();

    step3();
}
```

父类：

```text
规定流程：
1 → 2 → 3
```

子类：

```text
实现具体的1、2、3
```

一句话：

> **流程我规定，具体步骤你实现。**

---

### 14. 策略 Strategy

> **把一组可以互相替换的算法封装起来。**

例如支付：

```text
PaymentStrategy
    ├── AliPay
    ├── WeChatPay
    └── CreditCard
```

调用：

```java
payment.setStrategy(new AliPay());
payment.pay();
```

一句话：

> **算法可以随时换。**

---

### 15. 观察者 Observer

> **一个对象状态发生变化，自动通知多个对象。**

例如：

```text
微信公众号
      ↓
发布文章
      ↓
通知
 ┌────┼────┐
 ↓    ↓    ↓
用户1 用户2 用户3
```

Java / Spring 中非常常见。

一句话：

> **我发生变化了，通知所有关注我的人。**

---

### 16. 责任链 Chain of Responsibility

> **把多个处理者串成一条链，请求沿着链逐个处理。**

例如 HTTP 请求：

```text
请求
 ↓
登录过滤器
 ↓
权限过滤器
 ↓
参数过滤器
 ↓
业务
```

每个节点决定：

```text
我处理
或者
交给下一个
```

一句话：

> **一个不处理，就交给下一个。**

---

### 17. 命令 Command

> **把一个操作封装成一个对象。**

例如：

```java
Command command = new SaveCommand();
command.execute();
```

于是：

```text
操作
 ↓
变成对象
```

这样可以：

```text
排队
撤销
记录
重做
```

一句话：

> **把“做什么”封装成对象。**

---

### 18. 迭代器 Iterator

> **不暴露集合内部结构，统一遍历集合。**

你平时写：

```java
Iterator iterator = list.iterator();

while (iterator.hasNext()) {
    Object obj = iterator.next();
}
```

就是典型迭代器。

一句话：

> **统一遍历不同集合。**

---

### 19. 中介者 Mediator

> **对象之间不要直接互相通信，统一通过中介者。**

原本：

```text
A ↔ B
A ↔ C
A ↔ D
B ↔ C
B ↔ D
C ↔ D
```

关系爆炸。

变成：

```text
       A
       ↓
B → Mediator ← C
       ↑
       D
```

一句话：

> **大家别互相联系，都找中介。**

---

### 20. 备忘录 Memento

> **保存对象某个时刻的状态，以便以后恢复。**

最典型：

```text
编辑器
 ↓
输入文字
 ↓
保存状态
 ↓
继续修改
 ↓
撤销
 ↓
恢复之前状态
```

一句话：

> **给对象拍快照，之后可以恢复。**

---

### 21. 状态 State

> **对象的行为随着内部状态变化而变化。**

例如订单：

```text
订单
 │
 ├── 待支付
 │     ↓
 │   支付
 │
 ├── 已支付
 │     ↓
 │   发货
 │
 └── 已完成
```

同一个：

```java
order.pay();
```

不同状态下行为不同。

一句话：

> **状态不同，行为不同。**

---

### 22. 访问者 Visitor

> **把对对象结构的操作从对象本身中分离出来。**

例如：

```text
文件系统
├── File
└── Directory
```

现在你想对它们进行：

```text
杀毒
统计大小
压缩
备份
```

不想一直修改 File / Directory。

于是：

```text
File
Directory
    ↓
Visitor
 ├── AntivirusVisitor
 ├── SizeVisitor
 └── BackupVisitor
```

一句话：

> **数据结构不变，不断增加新的操作。**

这个模式相对难，先混个脸熟就行。

---

### 23. 解释器 Interpreter

> **定义一套语言/表达式的规则，然后解释执行。**

例如：

```text
a + b
```

或者：

```text
age > 18 AND gender = "male"
```

系统解析：

```text
表达式
 ↓
语法树
 ↓
解释执行
```

典型场景：

```text
SQL解析
规则引擎
表达式计算
脚本语言
```

一句话：

> **自己定义一套小语言，然后解释它。**

---

# 四、给你一张“背诵版”

如果只是为了**先建立整体认识**，建议先背这张：

### 创建型 5

```text
单例       → 只能有一个
工厂方法   → 创建一个产品
抽象工厂   → 创建一族产品
建造者     → 一步步构建复杂对象
原型       → 复制已有对象
```

### 结构型 7

```text
适配器     → 接口转换
装饰器     → 动态增加功能
代理       → 控制对象访问
外观       → 给复杂系统提供简单入口
桥接       → 两个变化维度独立
组合       → 一个和一群统一处理
享元       → 共享对象，减少重复
```

### 行为型 11

```text
模板方法   → 流程固定，步骤变化
策略       → 算法可以替换
观察者     → 状态变化通知别人
责任链     → 一个个处理者传递
命令       → 把操作封装成对象
迭代器     → 统一遍历
中介者     → 通过中介通信
备忘录     → 保存/恢复状态
状态       → 状态决定行为
访问者     → 数据结构与操作分离
解释器     → 解释一套语言/表达式
```

---

# 五、结合你现在学的 Spring，哪些最值得重点学？

**不用23个全部同等深度。**

你现在走 Java → Spring 这条路线，我建议优先级：

```text
★★★★★
代理 Proxy
工厂 Factory
单例 Singleton
模板方法 Template Method
策略 Strategy
观察者 Observer
责任链 Chain of Responsibility
装饰器 Decorator

★★★★☆
适配器 Adapter
建造者 Builder
组合 Composite
外观 Facade

★★★☆☆
状态 State
命令 Command
迭代器 Iterator

★★☆☆☆
桥接 Bridge
享元 Flyweight
备忘录 Memento
中介者 Mediator
访问者 Visitor
解释器 Interpreter
```

尤其你刚刚学完：

```text
静态代理
    ↓
JDK动态代理
    ↓
CGLIB
    ↓
Spring AOP
```

这里的 **Proxy（代理模式）** 是非常值得彻底理解的。

然后你再去看 Spring：

```text
Spring IoC
   ↓
工厂模式 + 单例模式 + 反射

Spring AOP
   ↓
代理模式

Spring MVC
   ↓
前端控制器 / 责任链 / 适配器等

Spring事务
   ↓
代理 + AOP

Spring各种Builder
   ↓
建造者模式
```

你会慢慢发现：

> **设计模式不是23个互不相关的“背诵题”，而是一些反复出现的解决问题的套路。**

等你把这些模式和 Spring 源码里的实际应用对应起来，才算真正学会，而不是只会背“单例——一个对象、工厂——创建对象”这种口诀。