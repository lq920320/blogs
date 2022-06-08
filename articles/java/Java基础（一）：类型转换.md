## Java基础（一）：类型转换



Java的基本类型共有八种，基本类型可以分为三类，字符类型 `char`，布尔类型 `boolean` 以及数值类型 `byte`、`short`、`int`、`long`、`float`、`double`。 数值类型又可以分为整数类型 `byte`、`short`、`int`、`long` 和浮点数类型 `float`、`double`。

对于非数值类型：

| 类型  | 占用空间大小（bits）                     | 取值范围   |
| ----- | ---------------------------------------- | ---------- |
| char  | 16                                       | -          |
| bloon | 1（理论上是1位，实际按照1字节即8位处理） | false/true |

对于数值类型来说，每种类型又有各自的范围（3.5E38即为 3.5 x 10^38）：

| 类型   | 占用空间大小（bits） | 取值范围                |
| ------ | -------------------- | ----------------------- |
| byte   | 8                    | 从 -128 到 127          |
| short  | 16                   | 从 -2^15 到 +2^15 - 1   |
| int    | 32                   | 从 -2^31 到 +2^31 - 1   |
| long   | 64                   | 从 -2^63 到 +2^63 - 1   |
| float  | 32                   | 从 -3.4E38 到 +3.4E38   |
| double | 64                   | 从 -1.7E308 到 +1.7E308 |

### 1、数值类型转换

数值类型的转换有两种方式：*隐式转换* 和 *显式转换*。**隐式转换**通常发生在 源类型 比 目标类型 取值范围小的场景。比如：

```java
// 隐式转换
byte byteVar = 42;
short shortVar = byteVar;
int intVar = shortVar;
long longVar = intVar;
float floatVar = longVar;
double doubleVar = floatVar;
```

**显示转换**则通常发生在在 源类型 比 目标类型 取值范围大的场景。比如：

```java
// 显式转换
double doubleVar = 42.0d;
float floatVar = (float) doubleVar;
long longVar = (long) floatVar;
int intVar = (int) longVar;
short shortVar = (short) intVar;
byte byteVar = (byte) shortVar;
```

当转换浮点类型（`float`，`double`）到其他数值类型时，其数值的取值按照向下取整。

### 2、基本的数值类型提升

在运算过程中会发生数值类型的提升，但也有一些情况是不能提升和转换的。

```java
    static void testNumericPromotion() {
        char char1 = 1, char2 = 2;
        short short1 = 1, short2 = 2;
        int int1 = 1, int2 = 2;
        float float1 = 1.0f, float2 = 2.0f;
        // char1 = char1 + char2; // Error: 不能从 int 转换为 char
        // short1 = short1 + short2; // Error: 不能从 int 转换为 short
        int1 = char1 + char2; // char 被提升为了 int
        int1 = short1 + short2; // short 被提升为了 int.
        int1 = char1 + short2; // char 和 short 都可以被提升为 int.
        float1 = short1 + float2; // short 被提升为了 float.
        int1 = int1 + int2; // int 未变更.
    }
```

### 3、非数值类型的转换

**boolean** 不能由其他任意一种基础类型转换而来，它也不能转换成其他基础类型。

**char** 通过 `Unicode` 指定的代码点映射，可以从任意一种数值基础类型转换得来，也可以转换成任意数值类型。一个 `char` 在内存中表示为无符号 16 位整数值（2 个字节），因此转换为 `byte` 类型时（1 个字节）将删除其中的 8 位（这对于 `ASCII` 字符是安全的）。虽然 `Character ` 类的实用方法使用 `int`（4字节）来传递/接受代码点值，但仅用 `short`（2 个字节）类型便足以存储 `Unicode` 代码点。

```java
int badInt = (int) true; // 编译错误：不兼容的类型
char char1 = (char) 65; // A
byte byte1 = (byte) 'A'; // 65
short short1 = (short) 'A'; // 65
int int1 = (int) 'A'; // 65
char char2 = (char) 8253; // ‽
byte byte2 = (byte) '‽'; // 61 (将代码点截取到 ASCII 范围内)
short short2 = (short) '‽'; // 8253
int int2 = (int) '‽'; // 8253
```

### 4、对象转换

和基础类型一样，对象的转换也可以分为 *隐式转换* 和 *显示转换*。

**隐式转换**通常发生在当 源类型 继承或实现 目标类型（转换为父类或者接口类）时。

**显式转换**通常在 源类型 被 目标类型 继承或者实现时（转换为子类）完成。当一个对象转换时，如果不是目标类型（或者目标类型的子类）时，便会抛出一个运行时异常（`ClassCastException`）。

```java
Float floatVar = new Float(42.0f);
Number n = floatVar; // 隐式（Float 实现了 Number）
Float floatVar2 = (Float) n; // 显式
Double doubleVar = (Double) n; // 抛出异常（对象类型不是Double，也不是Double的子类）
```

### 5、使用 `instanceof` 检查一个对象是否可以转换

Java 提供了一个 `instanceof` 操作符来校验一个对象是否为特定类型，或者是否为那个类型的子类。那么代码便可以正确地选择转换或者不转换那个对象。

```java
Object obj = Calendar.getInstance();
long time = 0;

if (obj instanceof Calendar) {
  time = ((Calendar) obj).getTime().getTime();
  System.out.println("time1:" + time);
}
if (obj instanceof Date) {
  time = ((Date) obj).getTime(); // 这行代码不会执行，因为obj不是 Date 类型
  System.out.println("time2:" + time);
}
```



### 总结

本文我们回顾了一些 Java 的基础类型，以及各个类型的转换。特别要注意一些隐式转换，可能出现精度丢失、范围丢失等等，如果需要转换，应该更优先考虑显式转换。

### 链接

- Numeric Primitive Data Types：https://chortle.ccsu.edu/java5/Notes/chap08/ch08_6.html