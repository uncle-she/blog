# Java 基础

## 基本类型

![primitive bytes](assets/Java基础_20190513-110011.png)

1. float 和 double

    - float 是单精度浮点数，内存分配 4 个字节，占 32 位，有效小数位 6 - 7 位
    - double 是双精度浮点数，内存分配 8 个字节，占 64 位，有效小数位 15 位
    - **默认声明的小数是 double 类型的**，如 `double d=4.0`
      如果声明： `float x = 4.0`则会报错，需要如下写法：`float x = 4.0f`或者`float x = (float)4.0`
      其中 4.0f 后面的 f 只是为了区别 double，并不代表任何数字上的意义

## Java 中的数组

``` java
String[] strs = new String[3];
String[] strs = {"aa", "bb", "cc"};

// 二维数组
String[][] str2d = new String[2][2];

String[][] str2d = new String[2][];
str2d[0] = new String[] {"aa", "bb"};
str2d[1] = new String[4];

String[][] str2d = {
    {"aa", "bb"},
    {"aaa", "bbb"}
}

// 数组也是一个对象
// 有 .length 成员
// 有 .clone() 方法
```

## hashCode / compareTo / equals

### hashCode : for operation in HashMap / HashTable / HashSet

``` java
@Override public int hashCode() {
    int result = aString != null ? aString.hashCode() : 0;
    result = 31 * result + (aLong != null ? aLong.hashCode() : 0);
    result = 31 * result + (aClassObject != null ? aClassObject.hashCode() : 0);
    result = 31 * result + primitiveInt;
    result = 31 * result + (primitiveFloat != +0.0f ? Float.floatToIntBits(primitiveFloat) : 0);
    return result;
}
```

### compareTo : 为了顺序，实现 `java.lang.Comparable<T>` 接口

``` java
@Override public int compareTo(Account aThat) {
    final int BEFORE = -1;
    final int EQUAL = 0;
    final int AFTER = 1;

    //this optimization is usually worthwhile, and can
    //always be added
    if (this == aThat) return EQUAL;

    //primitive numbers follow this form
    if (this.fAccountNumber < aThat.fAccountNumber) return BEFORE;
    if (this.fAccountNumber > aThat.fAccountNumber) return AFTER;

    //booleans follow this form
    if (!this.fIsNewAccount && aThat.fIsNewAccount) return BEFORE;
    if (this.fIsNewAccount && !aThat.fIsNewAccount) return AFTER;

    //objects, including type-safe enums, follow this form
    //note that null objects will throw an exception here
    int comparison = this.fAccountType.compareTo(aThat.fAccountType);
    if (comparison != EQUAL) return comparison;

    comparison = this.fLastName.compareTo(aThat.fLastName);
    if (comparison != EQUAL) return comparison;

    comparison = this.fFirstName.compareTo(aThat.fFirstName);
    if (comparison != EQUAL) return comparison;

    if (this.fBalance < aThat.fBalance) return BEFORE;
    if (this.fBalance > aThat.fBalance) return AFTER;

    //all comparisons have yielded equality
    //verify that compareTo is consistent with equals (optional)
    assert this.equals(aThat) : "compareTo inconsistent with equals.";

    return EQUAL;
}
```

### equals 主要用于 Collections API

``` java
@Override public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;

    TestClass testClass = (TestClass) o;

    if (primitiveInt != testClass.primitiveInt) return false;
    if (primitiveFloat != testClass.primitiveFloat) return false;

    if (!Objects.equals(aString, testClass.aString)) return false;
    if (!Objects.equals(aLong, testClass.aLong)) return false;
    return Objects.equals(aClassObject, testClass.aClassObject);
}
```

## Exception

1. 继承关系

    ![Exception](assets/Java基础_20190528-200107.png)

2. Checked Exception / Unchecked Exception

    ``` text
                ---> Throwable  <---
                |    (checked)     |
                |                  |
                |                  |
        ---> Exception           Error
        |    (checked)        (unchecked)
        |
    RuntimeException
    (unchecked)
    ```

3. Checked Exceptions (like: IOException) must check
4. Unchecked Exceptions
5. Errors : 如 `StackOverflowError` and `OutOfMemoryError`.
6. 子类 can throw 比父类更少 checked exceptions，不能更多
7. try-with-resources

    Java 7 后，我们可以使用简单的语法来使用 extend AutoCloseable 的资源，资源将会自动的 close.

    ``` java
    try (Scanner contents = new Scanner(new File(playerFile))) {
      return Integer.parseInt(contents.nextLine());
    } catch (FileNotFoundException e ) {
      logger.warn("File not found, resetting score.");
      return 0;
    }
    ```

## Overide vs Overload + @Overide 注解

## 类的加载顺序

<https://blog.csdn.net/weixin_37766296/article/details/80545283>

## JAVA 线程

### 线程的生命周期

![Thread State](assets/Java基础_20190509-175029.png)

![Runable State](assets/Java基础_20190509-175046.png)

### Daemon thread vs User thread

A daemon thread is a thread that does not prevent the JVM from exiting when the program finishes but the thread is still running. An example for a daemon thread is the garbage collection.

主线程结束后，如果只有 Daemon threads，Daemon threads 自动退出，程序结束。

优先级: User thread > Daemon Thread

You can use the `setDaemon(boolean)` method to change the `Thread` daemon properties before the thread starts.

### Thread.sleep(ms) vs Threa.yield()

sleep(): causes the thread to definitely stop executing for a given amount of time; if no other thread or process needs to be run, the CPU will be idle (and probably enter a power saving mode).

yield(): basically means that the thread is not doing anything particularly important and if any other threads or processes need to be run, they should. Otherwise, the current thread will continue to run.

### java.util.concurrent.Executors

提供一个直接处理任务的层，帮你管理 Thread 对象

Comand 设计模式

### Java 8 Lambda

Lambda 对应的接口类型

1. Java 7 已经存在的

    - java.lang.Runnable : () -> {}
    - java.util.concurrent.Callable: () -> { return V } , can throw exception
    - java.security.PrivilegedAction
    - java.util.Comparator: (T t1, T t2) -> { return int }
    - java.io.FileFilter
    - java.beans.PropertyChangeListener

2. Java 8 新增的 `java.util.function`

    - `Predicate<T>`——接收 T 对象并返回 boolean
    - `Consumer<T>`——接收 T 对象，不返回值
    - `Function<T, R>`——接收 T 对象，返回R对象
    - `Supplier<T>`——提供 T 对象（例如工厂），不接收值
    - `UnaryOperator<T>`——接收 T 对象，返回 T 对象
    - `BinaryOperator<T>`——接收两个 T 对象，返回 T 对象

## 常用 JVM 参数

### GC相关

1. `-XX:+PrintGCDetails`
2. ``

### 堆分配相关

1. -Xmx –Xms：指定java堆最大值（默认值是物理内存的1/4(<1GB)）和初始java堆最小值（默认值是物理内存的1/64(<1GB))

    默认(MinHeapFreeRatio参数可以调整)空余堆内存小于**40%**时，JVM就会增大堆直到-Xmx的最大限制.，默认(MaxHeapFreeRatio参数可以调整)空余堆内存大于**70%**时，JVM会减少堆直到 -Xms的最小限制。开发过程中，**通常会将 -Xms 与 -Xmx两个参数的配置相同的值**，其目的是为了能够在java垃圾回收机制清理完堆区后不需要重新分隔计算堆区的大小而浪费资源。

    **注意：此处设置的是Java堆大小，也就是新生代大小 + 年老代大小**

### 栈的分配相关

1. `-Xss`

## 基本数据结构

Stack
Deque, Vector
Queue, List, Set
Collection

Map: HashMap, HashTable
List: ArrayList, LinkedList
