
## Java中的字符串可以有多长？

在java中，一提到字符串，首先我们想到的便是String类，然后再处理字符串的过程中，还会用到StringBuffer以及StringBuilder。
显然，这三者是有区别的，而且在特定的场景下使用对应的类来进行字符串相关的处理效果更佳，接下来在这篇文章中，我们来细细品味这三者的联系与不同。


### String、StringBuffer和StringBuilder的区别

#### 1. String

String: 字符串常量，字符串长度不可变。Java中的String是immutable（不可变）的。

打开String类的源文件可见：
```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];

    /** Cache the hash code for the string */
    private int hash; // Default to 0

    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = -6849794470754667710L;
    
    ......

}
```

String类由final修饰，因此只能赋值一次，且不可再修改。另外，列举出了String类中所有的成员属性，从上面可以看出String类其实是通过char数组来保存字符串的。

再看一下String的相关操作：
```java
    /**
     * @param      beginIndex   the beginning index, inclusive.
     * @return     the specified substring.
     * @exception  IndexOutOfBoundsException  if
     *             {@code beginIndex} is negative or larger than the
     *             length of this {@code String} object.
     */
    public String substring(int beginIndex) {
        if (beginIndex < 0) {
            throw new StringIndexOutOfBoundsException(beginIndex);
        }
        int subLen = value.length - beginIndex;
        if (subLen < 0) {
            throw new StringIndexOutOfBoundsException(subLen);
        }
        return (beginIndex == 0) ? this : new String(value, beginIndex, subLen);
    }

    /**
     * @param   str   the {@code String} that is concatenated to the end
     *                of this {@code String}.
     * @return  a string that represents the concatenation of this object's
     *          characters followed by the string argument's characters.
     */
    public String concat(String str) {
        int otherLen = str.length();
        if (otherLen == 0) {
            return this;
        }
        int len = value.length;
        char buf[] = Arrays.copyOf(value, len + otherLen);
        str.getChars(buf, len);
        return new String(buf, true);
    }
    
    /**
     * @param   oldChar   the old character.
     * @param   newChar   the new character.
     * @return  a string derived from this string by replacing every
     *          occurrence of {@code oldChar} with {@code newChar}.
     */
    public String replace(char oldChar, char newChar) {
        if (oldChar != newChar) {
            int len = value.length;
            int i = -1;
            char[] val = value; /* avoid getfield opcode */

            while (++i < len) {
                if (val[i] == oldChar) {
                    break;
                }
            }
            if (i < len) {
                char buf[] = new char[len];
                for (int j = 0; j < i; j++) {
                    buf[j] = val[j];
                }
                while (i < len) {
                    char c = val[i];
                    buf[i] = (c == oldChar) ? newChar : c;
                    i++;
                }
                return new String(buf, true);
            }
        }
        return this;
    }
```
从上面的三个方法可以看出，无论是sub操、concat还是replace操作都不是在原有的字符串上进行的，而是重新生成了一个新的字符串对象。也就是说进行这些操作后，最原始的字符串并没有被改变。

在这里要永远记住一点：**“对String对象的任何改变都不影响到原对象，相关的任何change操作都会生成新的对象”。**

#### 2. StringBuffer（JDK1.0）

StringBuffer: 字符串变量（Synchronized，即线程安全）。如果要频繁对字符串内容进行修改，出于效率考虑最好使用StringBuffer，如果想转成String类型，可以调用StringBuffer的toString()方法。

StringBuffer是一个线程安全的，可变的字符序列。一个string buffer就像一个String一样，但可以修改。
在任何时间点它都包含一些特定的字符序列，但序列的长度和内容可以通过某些方法调用来改变其长度和内容。

字符串缓冲区（String buffers）对于多线程使用时安全的。必要时这些方法是用synchronized 关键字修饰的（加锁的），以便任何特定的实例上的所有操作都表现得好像它们以某个顺序发生，这与所涉及的每个单独线程所进行的方法调用的顺序一致。

StringBuffer 上的主要操作是 append 和 insert 方法，可重载这些方法，以接受任意类型的数据。每个方法都能有效地将给定的数据转换成字符串，然后将该字符串的字符追加或插入到字符串缓冲区中。
append 方法始终将这些字符添加到缓冲区的末端；而 insert 方法则在指定的点添加字符。

#### 3. StringBuilder（JDK5.0）

StringBuilder: 字符串变量（非线程安全）。在内部，StringBuilder对象被当作是一个包含字符序列的变长数组。

StringBuilder类提供与StringBuffer兼容的API，但不保证同步。用于在单个线程（通常情况下）使用String buffer的地方用作StringBuffer的嵌入式替换。在可能的情况下，建议将此类优先用于StringBuffer，因为在大多数实现中它会更快。原因是StringBuilder不需要考虑线程安全。

其构造方法如下：

|构造方法|	描述|
|-|-|
|StringBuilder()|	创建一个容量为16的StringBuilder对象（16个空元素）|
|StringBuilder(CharSequence cs)|	创建一个包含cs的StringBuilder对象，末尾附加16个空元素|
|StringBuilder(int initCapacity)|	创建一个容量为initCapacity的StringBuilder对象|
|StringBuilder(String s)|	创建一个包含s的StringBuilder对象，末尾附加16个空元素|

#### 4. 三者区别

String 类型和StringBuffer的主要性能区别：String是不可变的对象, 因此在每次对String类型进行改变的时候，都会生成一个新的 String 对象，然后将指针指向新的 String 对象，所以经常改变内容的字符串最好不要用 String ，因为每次生成对象都会对系统性能产生影响，特别当内存中无引用对象多了以后， JVM 的 GC 就会开始工作，性能就会降低。

使用 StringBuffer 类时，每次都会对 StringBuffer 对象本身进行操作，而不是生成新的对象并改变对象引用。所以多数情况下推荐使用 StringBuffer ，特别是字符串对象经常改变的情况下。

在某些特别情况下， String 对象的字符串拼接其实是被 Java Compiler 编译成了 StringBuffer 对象的拼接，所以这些时候 String 对象的速度并不会比 StringBuffer 对象慢，例如：
```
String s1 = "This is only a" + " simple" + " test";
StringBuffer Sb = new StringBuilder("This is only a").append(" simple").append(" test");
```

生成 String s1对象的速度并不比 StringBuffer慢。其实在Java Compiler里，自动做了如下转换，Java Compiler直接把上述第一条语句编译为：
```
String s1 = “This is only a simple test”;
```

所以速度很快。但要注意的是，如果拼接的字符串来自另外的String对象的话，Java Compiler就不会自动转换了，速度也就没那么快了，例如：
```
String s2 = “This is only a”;
String s3 = “ simple”;
String s4 = “ test”;
String s1 = s2 + s3 + s4;
```

这时候，Java Compiler会规规矩矩的按照原来的方式去做，String的concatenation（即+）操作利用了StringBuilder（或StringBuffer）的append方法实现，
此时，对于上述情况，若s2，s3，s4采用String定义，拼接时需要额外创建一个StringBuffer（或StringBuilder），之后将StringBuffer转换为String；
若采用StringBuffer（或StringBuilder），则不需额外创建StringBuffer。

#### 5. 使用策略

（1）基本原则：如果要操作少量的数据，用String ；单线程操作大量数据，用StringBuilder ；多线程操作大量数据，用StringBuffer。

（2）不要使用String类的"+"来进行频繁的拼接，因为那样的性能极差的，应该使用StringBuffer或StringBuilder类，这在Java的优化上是一条比较重要的原则。例如：
```
String result = "";
for (String s : hugeArray) {
    result = result + s;
}
 
// 使用StringBuilder
StringBuilder sb = new StringBuilder();
for (String s : hugeArray) {
    sb.append(s);
}
String result = sb.toString();
```

当出现上面的情况时，显然我们要采用第二种方法，因为第一种方法，每次循环都会创建一个String result用于保存结果，除此之外二者基本相同（对于jdk1.5及之后版本）。

（3）为了获得更好的性能，在构造 StringBuffer 或 StringBuilder 时应尽可能指定它们的容量。当然，如果你操作的字符串长度（length）不超过 16 个字符就不用了，当不指定容量（capacity）时默认构造一个容量为16的对象。不指定容量会显著降低性能。

（4）StringBuilder一般使用在方法内部来完成类似"+"功能，因为是线程不安全的，所以用完以后可以丢弃。StringBuffer主要用在全局变量中。

（5）相同情况下使用 StringBuilder 相比使用 StringBuffer 仅能获得 10%~15% 左右的性能提升，但却要冒多线程不安全的风险。而在现实的模块化编程中，负责某一模块的程序员不一定能清晰地判断该模块是否会放入多线程的环境中运行，因此：除非确定系统的瓶颈是在 StringBuffer 上，并且确定你的模块不会运行在多线程模式下，才可以采用StringBuilder；否则还是用StringBuffer。

### 字符串可以有多长？

一般情况下，我们在Java代码中使用String，小至几KB，几MB，甚至上百MB，这些都是比较常见的，很少有GB这种数量级的。那么，String的容量究竟有多少呢？

```
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];

    /** Cache the hash code for the string */
    private int hash; // Default to 0

    /** use serialVersionUID from JDK 1.0.2 for interoperability */
    private static final long serialVersionUID = -6849794470754667710L;
 
  .......
}
```

我们查看其源码可以知道，String是用char[]来存储的，数组下标为int类型，因此理论上最大能存放的元素个数只有Integer.MAX_VALUE个，这个数也就是2G。所以说char[]数组能存储2G大小的字符。因此在理想情况下也就是内存无限大，堆可以无限大的情况下，一个String类型的极限大小就是2G,长度为2147483647个字符。然而事实上并非如此，执行这段代码的时候便会抛出OutOfMemoryError，这与JVM的限制也是有关系的。

```
char[] chars = new char[Integer.MAX_VALUE];
System.out.println(Integer.MAX_VALUE);
```
抛出异常：         

```
java.lang.OutOfMemoryError: Requested array size exceeds VM limit
```

继续尝试，当执行如下代码时：

```
char[] chars = new char[Integer.MAX_VALUE - 2];
System.out.println(Integer.MAX_VALUE);
```

抛出的异常如下，堆内存也是限制条件之一：

```
java.lang.OutOfMemoryError: Java heap space
```

由于一个字符占两个字节，所以字符串的最大容量限制应该是你最大堆内存的一半，即`Integer.MAX_VALUE/2=1073741823`个字符。

综上所述，你应该会得到一个长度为Integer.MAX_VALUE（Java规范中通常是2147483647 （2^31 - 1），String类用于内部存储的数组最大的规格）或者是你最大堆内存的一半（因为每个字符占两个字节），以较小者为准。

参考：

https://blog.csdn.net/kingzone_2008/article/details/9220691

https://stackoverflow.com/questions/1179983/how-many-characters-can-a-java-string-have

