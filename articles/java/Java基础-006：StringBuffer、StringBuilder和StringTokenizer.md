## Java 基础-006：`StringBuffer`、`StringBuilder`和`StringTokenizer`



本文我们来看一下字符串相关的其他类：`StringBuffer`、`StringBuilder` 和 `StringTokenizer`。



### 1、`StringBuffer`

**关键点：**

- 用于创建可变（可修改）字符串。

- **可变**：可被改变。

- 是线程安全的，即多个线程不能同时访问它。

**方法：**

- `public synchronized StringBuffffer append(String s)`

- `public synchronized StringBuffffer insert(int offffset, String s)`

- `public synchronized StringBuffffer replace(int startIndex, int endIndex, String str)`

- `public synchronized StringBuffffer delete(int startIndex, int endIndex)`

- `public synchronized StringBuffffer reverse()`

- `public int capacity()`

- `public void ensureCapacity(int minimumCapacity)`

- `public char charAt(int index)`

- `public int length()`

- `public String substring(int beginIndex)`

- `public String substring(int beginIndex, int endIndex)`

**展示 `String` 与 `StringBuffer` 实现点不同之处的例子：**

```java
class Test{
    public static void main(String[] args) {
        String str = "study";
        str.concat("tonight");
        System.out.println(str); // 输出: study
        StringBuffer strB = new StringBuffer("study");
        strB.append("tonight");
        System.out.println(strB); // 输出: studytonight
    }
}
```

### 2、`StringBuilder`

Java 中 `StringBuilder` 也是用来创建一个可变（可修改）的字符串。除了它是非同步的，`StringBuilder` 与 `StringBuffer` 是相同的。JDK 1.5 之后的版本可用。

#### 2.1 对比一下 `StringBuffer`、`StringBuilder`、`Formatter` 和 `StringJoiner`

`StringBuffer`、`StringBuilder`、`Formatter` 和 `StringJoiner` 类都是 Java SE 实用程序类，主要用于从其他信息中组装字符串：

- `StringBuffer` 类从 Java 1.0 开始就出现了，它提供了多种方法来构建和修改包含字符序列的“缓冲区”。

- Java 5 中添加了 `StringBuilder` 类，以解决原始 `StringBuffer` 类的性能问题。 这两个类的 API 基本相同。 `StringBuffer` 和 `StringBuilder` 的主要区别在于前者是线程安全和同步的，而后者不是。

下面的例子说明了 `StringBuilder` 的用法：

```java
int one = 1;
String color = "red";
StringBuilder sb = new StringBuilder();
sb.append("One=").append(one).append(", Color=").append(color).append('\n');
System.out.print(sb);
// 打印 "One=1, Colour=red" 然后是 ASCII 换行符
```

（ `StringBuffer` 的用法相同：只要把上面的 `StrinBuilder` 替换成 `StringBuffer` 即可）

`StringBuffer` 和 `StringBuilder` 类适用于组装和修改字符串； 即它们提供了替换和删除字符以及以各种方式添加它们的方法。 其余两个类特定于字符串组装的任务。

- `Formatter `类是在 Java 5 中添加的，它仿照了 C 标准库中的 `sprintf` 函数。 接受一个带有嵌入格式说明符的格式字符串和一系列其他参数，并通过将参数转换为文本并将它们替换为格式说明符来生成一个字符串。 格式说明符的详细信息说明了参数如何转换为文本。

- `StringJoiner` 类是在 Java 8 中添加的。它是一种特殊用途的格式化程序，可以简洁地格式化字符串序列，并在它们之间使用分隔符。 它采用流畅的 API 设计，可与 Java 8 的流一起使用。

这里有一些典型的例子来展示 `Formatter` 的用法：

```java
// 这与上面 StringBuilder 例子做了同样的事情
int one = 1;
String color = "red";
Formatter f = new Formatter();
System.out.print(f.format("One=%d, colour=%s%n", one, color));
// 打印 "One=1, Colour=red" 然后跟了一个换行符
// 使用 String.format 便捷方法也是一样
System.out.print(String.format("One=%d, color=%s%n", one, color));
```

StringJoiner 类对于上述任务并非理想选择，因此这里有一个格式化字符串数组的示例。

```java
StringJoiner sj = new StringJoiner(", ", "[", "]");
for (String s : new String[]{"A", "B", "C"}) {
    sj.add(s);
}
System.out.println(sj);
// Prints "[A, B, C]" C]"
```

4 个类的使用场景可总结如下：

- `StringBuilder` 适用于任何字符串组装或者修改任务。

- `StringBuffer` （只）用于你需要一个线程安全的 `StringBuilder` 时。

- Formatter 提供了更丰富的字符串格式化功能，但不如 StringBuilder 高效。 这是因为每次调用 Formatter.format(...) 都需要：
  
  - 解析格式字符串；
  
  - 创建并填充可变参数数组，以及
  
  - 自动装箱任何原始类型参数。

- `StringJoiner` 使用分隔符对字符串序列提供简洁高效的格式化，但不适用于其他格式化任务。



#### 2.2 把一个字符串重复 n 次

问题：创建一个包含 n 个字符串 `s` 副本的字符串对象。

简单的方法是重复连接字符串：

```java
final int n = ...
final String s = ...
String result = "";
for (int i = 0; i < n; i++) {
    result += s;
}
```

这会创建 n 个包含 s 的 1 到 n 次重复的新字符串实例，导致运行时间为 `O(s.length() * n²) = O(s.length() * (1+2+...+(n-1 )+n))`。

为了避免这种情况，应该使用 `StringBuilder` ，它允许在 `O(s.length() * n)` 时间内创建字符串：

```java
final int n = ...
final String s = ...

StringBuilder builder = new StringBuilder();

for (int i = 0; i < n; i++) {
    builder.append(s);
}

String result = builder.toString();
```

### 3、String Tokenizer

`java.util.StringTokenizer` 类可以实现将字符串分解为词语（token）。 这是拆分字符串的一种简单方法。

分隔符集（分隔词语的字符）可以在创建时指定，也可以基于每个词语指定。

**根据空格拆分字符串**

```java
StringTokenizer st = new StringTokenizer("apple ball cat dog", " ");
while (st.hasMoreTokens()) {
    System.out.println(st.nextToken());
}
```

结果为：

```
apple
ball
cat
dog
```

**`StringTokenizer` 按照逗号 ',' 进行分割**

```java
StringTokenizer st = new StringTokenizer("apple,ball cat,dog", ",");
while (st.hasMoreTokens()) {
    System.out.println(st.nextToken());
}
```

```
apple
ball cat
dog
```

### 4、将字符串拆分为固定长度的部分

**将一个字符串拆分成全部是已知长度的子串**

诀窍是使用正则表达式 `\G` 的 `look-behind`，这意味着“上一个匹配的末尾”：

```java
String[] parts = str.split("(?<=\\G.{8})");
```

正则表达式匹配最后一个匹配结束后的 8 个字符。 由于在这种情况下匹配是零宽度，我们可以更简单地说“最后一次匹配后面的 8 个字符”。

方便的是，`\G` 被初始化为输入的开始，因此它也适用于输入的第一部分。

**将一个字符串拆分成全部是变量长度的子串**

与已知长度示例相同，然而是将长度插入正则表达式：

```java
int length = 5;
String[] parts = str.split("(?<=\\G.{" + length + "})");
```



### 总结

本文把剩余字符串相关的类回顾了一下。


