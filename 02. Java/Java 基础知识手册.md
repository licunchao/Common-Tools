# Java 基础知识手册

## 1. Java 概述

- **特点**：跨平台（JVM）、面向对象、健壮性、安全性、多线程。
- 核心机制：
  - **JDK** (Java Development Kit): 开发工具包。
  - **JRE** (Java Runtime Environment): 运行环境。
  - **JVM** (Java Virtual Machine): 虚拟机，实现“一次编写，到处运行”。
- **入口**：`public static void main(String[] args)`

## 2. 基本数据类型 (8种)

| 类型     | 关键字          | 字节 | 范围/描述                 |
| -------- | --------------- | ---- | ------------------------- |
| **整数** | `byte`          | 1    | -128 ~ 127                |
|          | `short`         | 2    | -32,768 ~ 32,767          |
|          | `int` (默认)    | 4    | -2^31 ~ 2^31-1            |
|          | `long`          | 8    | -2^63 ~ 2^63-1 (后缀 `L`) |
| **浮点** | `float`         | 4    | 单精度 (后缀 `F`)         |
|          | `double` (默认) | 8    | 双精度                    |
| **字符** | `char`          | 2    | Unicode 字符 ('A')        |
| **布尔** | `boolean`       | 1*   | `true` / `false`          |

> *注：boolean 具体大小由 JVM 实现决定。

## 3. 面向对象编程 (OOP)

### 3.1 三大特性

1. **封装 (Encapsulation)**: 隐藏内部细节，通过 `private` 属性和 `public` getter/setter 访问。
2. **继承 (Inheritance)**: `extends` 关键字，子类复用父类代码，单继承。
3. 多态 (Polymorphism):
   - **编译时多态**: 方法重载 (Overload)。
   - **运行时多态**: 方法重写 (Override)，父类引用指向子类对象。

### 3.2 类与对象

```java
// 定义类
public class Person {
    // 成员变量
    private String name;
    
    // 构造方法
    public Person(String name) {
        this.name = name;
    }
    
    // 成员方法
    public void sayHello() {
        System.out.println("Hello, " + name);
    }
}

// 使用
Person p = new Person("Alice");
p.sayHello();
```

### 3.3 关键字对比

| 关键字      | 作用                                           |
| ----------- | ---------------------------------------------- |
| `this`      | 指向当前对象实例                               |
| `super`     | 指向父类实例                                   |
| `static`    | 静态修饰，属于类而非对象，可直接通过类名调用   |
| `final`     | 不可变：类不可继承，方法不可重写，变量不可修改 |
| `abstract`  | 抽象类/方法，不能有实例，子类必须实现抽象方法  |
| `interface` | 接口，完全抽象，支持多实现 (`implements`)      |

## 4. 核心 API

### 4.1 字符串处理

- **String**: 不可变字符序列，存放在常量池。
- **StringBuilder**: 可变，线程不安全，效率高（单线程推荐）。
- **StringBuffer**: 可变，线程安全，效率略低。

```java
String s = "Hello";
s.length();
s.substring(0, 3);
s.equals("Hello"); // 内容比较，不要用 ==
```

### 4.2 集合框架 (Collections)

位于 `java.util` 包。

- List (有序，可重复):
  - `ArrayList`: 数组实现，查询快，增删慢。
  - `LinkedList`: 链表实现，增删快，查询慢。
- Set (无序，不可重复):
  - `HashSet`: 基于 HashMap，允许一个 null。
  - `TreeSet`: 基于红黑树，自然排序或定制排序。
- Map (键值对):
  - `HashMap`: 最常用，允许 null 键/值，非线程安全。
  - `Hashtable`: 线程安全，不允许 null (已过时，推荐 `ConcurrentHashMap`)。
  - `TreeMap`: 按键排序。

### 4.3 异常处理 (Exception)

- **体系**: `Throwable` -> `Error` (系统错误) / `Exception` (程序错误)。

- 分类:

  - **Checked Exception**: 编译期强制处理 (如 `IOException`, `SQLException`)。
  - **Unchecked Exception**: 运行时异常 (如 `NullPointerException`, `ArrayIndexOutOfBoundsException`)。

- 关键字:

  ```java
  try {
      // 可能出错的代码
  } catch (SpecificException e) {
      // 处理逻辑
  } finally {
      // 必然执行 (常用于关闭资源)
  }
  // 或者抛出
  throw new RuntimeException("Error message");
  ```

### 4.4 泛型 (Generics)

- 提供编译期类型安全检查，避免强制类型转换。
- 常用形式：`List<String>`, `Map<K, V>`, `class Box<T> {}`.

## 5. 高级特性

### 5.1 多线程 (Concurrency)

- 创建方式:
  1. 继承 `Thread` 类。
  2. 实现 `Runnable` 接口 (推荐)。
  3. 实现 `Callable` 接口 (有返回值)。
- 关键字:
  - `synchronized`: 同步锁，保证线程安全。
  - `volatile`: 保证可见性和禁止指令重排，不保证原子性。
- **线程池**: 推荐使用 `Executors` 工厂类或 `ThreadPoolExecutor`。

### 5.2 IO 流

- 分类:
  - **字节流** (`InputStream`/`OutputStream`): 处理图片、视频等二进制数据。
  - **字符流** (`Reader`/`Writer`): 处理文本数据。
- **NIO**: `Channel`, `Buffer`, `Selector` (非阻塞 IO)。

### 5.3 反射 (Reflection)

- 运行时动态获取类的信息（属性、方法、构造器）并操作对象。
- 核心类：`Class`, `Method`, `Field`, `Constructor`。
- 应用：框架底层（Spring, MyBatis）广泛使用。

### 5.4 注解 (Annotation)

- 元数据，用于编译检查或运行时处理。
- 内置：`@Override`, `@Deprecated`, `@SuppressWarnings`。
- 元注解：`@Retention`, `@Target`, `@Documented`。

## 6. Java 8+ 新特性 (现代 Java 必备)

1. Lambda 表达式

   : 简化匿名内部类。

   ```java
   (a, b) -> a + b;
   ```

2. Stream API

   : 函数式数据操作（过滤、映射、归约）。

   ```java
   list.stream().filter(s -> s.startsWith("A")).collect(Collectors.toList());
   ```

3. **Optional**: 优雅地处理空指针问题。

4. **新日期时间 API**: `java.time` 包 (`LocalDate`, `LocalDateTime`), 线程安全，替代旧的 `Date`/`Calendar`。

5. **接口默认方法**: `default` 关键字允许接口包含方法实现。

## 7. 内存管理 (JVM 简述)

- **堆 (Heap)**: 存放对象实例，GC 主要区域。
- **栈 (Stack)**: 存放局部变量、方法调用链。
- **方法区 (Method Area)**: 存放类信息、常量、静态变量。
- **垃圾回收 (GC)**: 自动回收不再使用的对象，常见算法：标记 - 清除、复制、标记 - 整理。

------

*本手册涵盖 Java SE 核心知识点，适用于快速复习与查阅。*
