

`volatile` 用来修饰变量，主要解决**多线程之间变量可见性和指令重排序**问题。

```java
private volatile boolean running = true;
```
### 1. 保证可见性

一个线程修改了 `volatile` 变量，其他线程能**立即看到最新值**，不会一直使用自己线程缓存中的旧值。

```java
class Task implements Runnable {
    private volatile boolean running = true;

    public void stop() {
        running = false;
    }

    @Override
    public void run() {
        while (running) {
            // 执行任务
        }
    }
}
```

如果没有 `volatile`，执行线程可能一直读到 `running == true`，导致无法退出循环。

### 2. 防止指令重排序

`volatile` 会建立一定的 happens-before 关系，保证对变量的写入顺序对其他线程可见，常用于状态发布、单例初始化等场景。


### 它不能解决什么？

`volatile` **不能保证复合操作的原子性**：

```java
volatile int count;

count++; // 不是原子操作
```

`count++` 实际上包含：

1. 读取 `count`
2. 加 1
3. 写回 `count`

多个线程同时执行时仍然可能丢失更新。

需要原子性时，应使用：

```java
AtomicInteger count = new AtomicInteger();

count.incrementAndGet();
```

或者使用：synchronized

volatile 解决的是 **变量修改后的可见性和部分有序性问题**，但不解决**复合操作的原子性问题**。适合用在：**状态标志、配置刷新、线程停止信号**等场景。