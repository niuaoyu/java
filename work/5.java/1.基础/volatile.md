
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

**什么是重排序？** 编译器和 CPU 为了提高执行效率，可能会打乱代码的执行顺序。只要**单线程**下运行结果不变，这种打乱就是允许的。但这在**多线程**下会导致严重 Bug。

```java
class ReorderExample {
    int a = 0;
    boolean flag = false;

    // 线程 1 执行
    public void writer() {
        a = 1;          // 步骤 1
        flag = true;    // 步骤 2
    }

    // 线程 2 执行
    public void reader() {
        if (flag) {     // 步骤 3
            int b = a;  // 步骤 4
        }
    }
}
```
因为“步骤1”和“步骤2”互相没有依赖关系，CPU 可能会**先执行步骤 2，再执行步骤 1**。

要知道类的成员变量是在堆空间所共有的，而不是在线程栈里私有的，共有就会出现问题。flag就是所有线程共享。

- **线程 1** 执行 `flag = true`。（线程执行writer方法会把flag调入到自己栈里，修改为true再把flag数值写回堆空间里的flag，变成true）
- 此时 **线程 2** 飞速插队，执行 `if (flag)`，发现是 `true`。
- **线程 2** 执行 `int b = a`。但此时 `a = 1` 还没执行完！所以 **`b` 拿到了错误的值 `0`**。

**怎么解决？** 如果把 `flag` 声明为 `volatile boolean flag = false;`，Java 会在底层插入“内存屏障”，**强行禁止步骤 1 和步骤 2 交换顺序**。


### 3. 它不能解决什么？

`volatile` **不能保证复合操作的原子性**：

```java
volatile int count;

count++; // 不是原子操作
```

**什么是原子性？** 一个操作要么彻底执行完，要么完全不执行，不能在中途被别的线程插队。**为什么 `volatile count++` 不行？** 因为 `count++` 在底层并不是一步完成的，它其实是**三个独立步骤（复合操作）**：

1. **读取** `count` 的当前值
2. 将值 **加 1**
3. 将新值 **写回** `count`

`volatile` 只能保证“读”的时候是最新的，“写”的时候立刻让别人看到，**但管不住别人在你“加1”的半途中插队**。

**线程交错举例（导致丢数据）：** 假设 `volatile int count = 0;`

1. **线程A** 读取 `count`，拿到 `0`。
2. **线程B** 也来读取 `count`，因为A还没改，拿到的也是 `0`。
3. **线程A** 在自己内存（线程栈）里计算 `0 + 1 = 1`，并写回，此时 `count = 1`。
4. **线程B** 也在自己内存（线程栈）里计算 `0 + 1 = 1`，并写回，此时 `count` 还是 `1`。

结果：两个线程各加了 1 次，但结果是 1，不是 2

简单理解：`volatile` 写入之前的操作，不能被重排到 `volatile` 写入之后；读取到这个 `volatile` 值的线程，也能看到之前的写入。

多个线程同时执行时仍然可能丢失更新。需要原子性时，应使用：

```java
AtomicInteger count = new AtomicInteger();

count.incrementAndGet();
```

或者使用：synchronized

`AtomicInteger` 不依赖普通的读写，它依赖的是 CPU 级别的特殊硬件指令（称为 **CAS：Compare-And-Swap，比较并交换**）。**`incrementAndGet()` 底层是怎么工作的？（伪代码解释）**
```java
// 循环重试，直到成功为止
while (true) {
    int current = count.get();      // 1. 获取当前值 (假设是0)
    int next = current + 1;         // 2. 计算目标值 (1)
    
    // 3. 最关键的一步：CAS (Compare And Swap)
    // 这一步由 CPU 硬件保证是一步到底的，绝对不会被打断！
    if (compareAndSet(current, next)) {
        // 如果当前内存里还是 0，说明没被别人改过，那就安全地把它改成 1，退出循环
        return next;
    }
    // 如果走到这里，说明刚才准备更新时，发现内存里的值不是 0 了（被别人抢先改成了 1）
    // 那就不更新，重新进入 while 循环，拿最新的值再试一次！
}
```

**通俗点说：** `count++` 是：“不管三七二十一，我拿过来加1就写回去。”（容易覆盖别人的结果） `AtomicInteger` 是：“我拿着旧值去更新，如果更新的瞬间发现旧值被别人改过了，我就**认怂作废，重新读取再算一遍**。” 并且，它的“比对旧值+更新新值”这个动作，是**CPU硬件级别直接支持的一条机器指令，天然防插队**。



volatile 解决的是 **变量修改后的可见性和部分有序性问题**，但不解决**复合操作的原子性问题**。适合用在：**状态标志、配置刷新、线程停止信号**等场景。