
# JDK JRE JVM 一句话解释清楚，画图，类比，深入浅出？

写的.java文件通过JDK提供的环境 编译为.class文件，送到JRE当中的JVM 做解释执行字节码的操作

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


# 1.javac编译命令怎么用？2.java执行怎么用?







