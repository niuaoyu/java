需掌握内容：
1. JDK JRE JVM 一句话解释清楚，画图，类比，深入浅出？
2. 类加载器?
3. 1.javac编译命令怎么用？2.java执行怎么用?
4. public class vs class?



# JDK JRE JVM 一句话解释清楚，画图，类比，深入浅出？

写的.java文件通过JDK提供的环境 编译为.class文件，送到JRE当中的JVM 做解释执行字节码的操作

---
# 类加载器 = Java 里专门负责 “找到 .class 文件并读进内存” 

- 写 `new Student()`  类加载器就去硬盘上找 `Student.class`

可以理解为：**类加载器 = 找文件的小弟**，classpath 是什么？路径列表，这个列表告诉类加载器 **“去哪些文件夹里找 .class 文件”**， “一旦配了 classpath 环境变量，就**只能在**你给的路径里找，**不再**去当前目录找了”
#### 情况1：不配 classpath（默认）
```
类加载器找 Student.class：
1️⃣ 当前目录 ✅
```
✅ 能找到
#### 情况2：配了 classpath=D:\lib
```
类加载器找 Student.class：
1️⃣ D:\lib  ✅
2️⃣ 当前目录 ❌（不再去了！）
```
❌ 如果 `D:\lib` 里没有，就报错 `ClassNotFoundException`  
👉 即使当前目录有，也不认

#### 最容易翻车的点（你很可能遇到）
```bash
set classpath=.;D:\lib
```

**这个 `.` 代表当前目录**  

- ✅ 配了 `.` → 当前目录 + D:\lib 都找  
- ❌ 不配 `.` → 只找 D:\lib，当前目录被忽略

---
# 1.javac编译命令怎么用？2.java执行怎么用?
## 一、javac 编译命令用法

### 基本格式
```bash
javac [选项] 源文件名.java
```

### 最常见用法

| 场景 | 命令 |
|------|------|
| 编译单个文件 | `javac Hello.java` |
| 编译多个文件 | `javac Hello.java World.java` |
| 编译所有Java文件 | `javac *.java` |
| 指定输出目录 | `javac -d 目录 Hello.java` |
| 指定依赖的jar包 | `javac -cp lib.jar Hello.java` |

### 实际例子

假设你有：
```
C:\mycode\Hello.java
```

```bash
# 切换到文件所在目录
cd C:\mycode

# 编译
javac Hello.java

# 结果：生成 Hello.class
```

## 二、java 执行命令用法

### 基本格式
```bash
java [选项] 类名 [参数]
```

> ⚠️ **注意**：是**类名**，不是文件名！不要加 `.class` 后缀

### 最常见用法

| 场景 | 命令 |
|------|------|
| 执行单个类 | `java Hello` |
| 带参数执行 | `java Hello 张三 18` |
| 指定classpath | `java -cp .;lib Hello` |
| 指定jar包主类 | `java -jar app.jar` |

### 实际例子

```bash
# 编译后
javac Hello.java

# 执行 ✅ 正确
java Hello

# 执行 ❌ 错误（不要加.class）
java Hello.class   # 报错
```

## 三、完整流程图

```
源代码               字节码              运行结果
Hello.java  ──javac→  Hello.class  ──java→  输出
```

### 实战命令顺序
```bash
# 1. 编译
javac Hello.java

# 2. 执行
java Hello
```

## 四、最容易犯的错误

| 错误 | 原因 | 正确 |
|------|------|------|
| `java Hello.class` | 多写了 `.class` | `java Hello` |
| `javac Hello` | 忘了 `.java` 后缀 | `javac Hello.java` |
| `java hello` | 大小写不匹配 | `java Hello`（类名大小写必须一致）|

## 五、配合 classpath 使用

### 当文件不在当前目录时

假设 `Hello.class` 在 `D:\classes` 目录：

```bash
# 让类加载器去 D:\classes 找
java -cp D:\classes Hello
```

### 同时保留当前目录
```bash
# . 代表当前目录
java -cp .;D:\classes Hello
```
## 一句话记忆

> **`javac 文件名.java` 编译出 `.class`**  
> **`java 类名` 执行（不加后缀，大小写一致）**

---
# public class vs class

自我描述：一个源文件只能有一个public class（类名跟文件名一样），可以没有，可以有多个class，每个class编译成一个.class文件，每个class都可以有一个main方法，没有main方法的class编译后（javac B.java）不能用java B，报错。


