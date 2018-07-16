## 一、对`java`平台的理解
- `java`是门面向面向对象语言(封装、继承、多态)，最显著的特性有两个方面:
- - **平台无关性**。“一次编写，导出执行”--强大的跨平台能力，基于强大的`java`虚拟机。`java`源代码经过编译成为能在`java`虚拟机运行的`.class`字节码，然后java虚拟机在运行时，把字节码解释成机器码运行。
- - **垃圾回收机制**。`java`虚拟机提供垃圾回收机制，通过jvm提供的垃圾收集器回收和分配内存。
- `java`语言特性：泛型、反射等
- `java`丰富的类库：核心类库、集合、`IO/NIO`、网络、并发、安全等
- `JRE（Java Runtime Environment）`，是指`Java`运行环境,包含了`JVM`和`Java`类库
- `jdk`，`JRE`的一个超集，提供各种强大的工具，包括`JRE`，`javac`(`java`语言编程编译器)，诊断工具等

## 二、对“java是解释执行”的理解 
- 不太准确
- 一般情况，开发的源代码，首先通过`javac`编译成字节码，然后，在运行时，通过java虚拟机（JVM）内嵌的解析器将字节码转换成最终的机械码。
- 但是常见的 `JVM`，比如我们大多数情况使用的 `OracleJDK` 提供的 **`Hotspot JVM`**，都提供了 **`JIT（Just-In-Time）`编译器**，也就是通常所说的**动态编译器**，JIT 能够在运行时将热点代码编译成机器码，这种情况下部分热点代码就属于**编译执行**，而不是解释执行了。

## PS：知识拓展
1. `java`泛型、`java`反射
2. `java`集合包、`java`并发包
3. `java`三个`ClassLoader`及`java`类加载机制
4. `java`垃圾回收机制
5. `java`常见垃圾收集器（`SerialGC`、`Parallel GC`、 `CMS`、 `G1`）
及其原理、各自适用于什么工作负载
6. `jdk`常见工具类（`jdk` 9 `AOT` 特性）

## 三、对比 `Exception` 和 `Error`
![image](https://github.com/NieBinQiang/java-i7w/blob/master/pic/Throwable.png)
- `Exception`类和`Error`都继承于`Throwable`，只有继承`Throwable`才可以被抛出或捕获
，他们是异常处理机制的基本组成类型
- `Exception`是在程序运行中，可以预料的，并且应该被捕获进行处理的异常；`Error`则是不大可能出现的错误，会导致程序处于非正常、不可恢复状态。

## 四、运行时异常和一般异常有什么区别
- `Exception`分为两种：可检查异常（`checkedException`）和不可检测异常（也叫做运行时异常 `RuntimeException`）
- 可检查异常：在源代码必须进行捕获，是编译检查的一部分
- 不可检查异常：即运行时异常，如`NullPointerException`、`ArrayOutOfBoundsException`等，这些通常是可以编码避免的逻辑错误，具体根据需要来判断是否捕获，并不会在编译时强制要求。

## PS：知识拓展
1. 异常处理的最佳实践
2. `NoClassDefFound`和`ClassNotFoundExcepTion`区别
3. 异常机制的性能开销

