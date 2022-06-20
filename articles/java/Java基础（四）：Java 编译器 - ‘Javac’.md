## Java基础（四）：Java 编译器 - ‘Javac’

本篇我们来回顾一下 java 文件是如何通过 `javac` 命令编译的。

### 1、从 `javac` 命令开始

#### 简单例子

假设“HelloWorld.java” 包含如下代码：

```java
public class HellWorld {
  public static void main(String[] args) {
    System.out.println("Hello world!");
  }
}
```

（有关上述代码的解释，请参阅 Java 语言入门 。）

我们可以通过如下命令来编译上述文件：

```shell
$ javac HelloWorld.java
```

接着便会生成一个 “HelloWorld.class” 的文件，我们可以通过下面的命令进行运行：

```shell
$ java HelloWorld
Hello world!
```

这个例子的重点是：

1. 源文件名称 “HelloWorld.java” 必须与源文件中的类名称相匹配，如 `HelloWorld`。如果不匹配的话，那么你将会得到一个编译错误。
2. 字节码文件 “HelloWorld.class” 对应于类名。如果你对 “HelloWorld.class” 重命名，那么在尝试运行的时候将会返回一个错误。
3. 当使用 `java` 命令运行一个 java 应用，你提供的是类名称而非字节码文件名。

#### 含有包（package）的例子

实际项目中的Java代码会使用包（package）来组织类的命名空间，已减少类名冲突的风险。

如果你想在 `com.example` 包中声明一个 `HelloWorld` 的类，“HelloWorld.java” 文件就要包含下面的 Java 源码：

```java
package com.example;

public class HelloWorld {
  public static void main(String[] args) {
    System.out.println("Hello world!"); 
  } 
}

```

这个源代码文件需要存储在一个目录树中，其结构与包命名相对应。

```
. # the current directory (for this example) 
|
----com
    |
    ----example
        |
        ----HelloWorld.java
```

我们可以使用下面的命令进行编译：

```shell
$ javac com/example/HelloWorld.java
```

然后会生成一个 "com/example/HelloWorld.class" 文件，编译之后，文件结构应该就会是如下的样子：

```
. # the current directory (for this example) 
|
----com
    |
    ----example
        |
        ----HelloWorld.java
        ----HelloWorld.class
```

我们可以通过下面的命令运行程序：

```shell
$ java com.example.HelloWorld
Hello World!
```

此示例中需要注意的其他要点：

1. 目录结构必须与包名称结构相匹配。
2. 运行类时，必须提供完整的类名； 即“com.example.HelloWorld”不是“HelloWorld”。
3. 不必从当前目录编译和运行 Java 代码， 我们在这里只是为了说明。

#### 使用 `javac` 一次编译多个文件

如果你的程序包含多个源码文件（大多数都是这种情况），你可以一次性编译它们。或者，可以通过列出路径名同时编译多个文件：

```shell
$ javac Foo.java Bar.java
```

或使用 `shell` 命令的文件名通配符功能：

```shell
$ javac *.java
$ javac com/example/*.java
$ javac */**/*.java # 只有在 Zsh 或者安装了 globstar 命令行工具才能运行这条 shell
```

这将分别编译当前目录中，“com/example”目录和子目录中的所有 Java 源文件。 第三种选择是将源文件名（和编译器选项）列表作为文件提供。 例如：

```shell
$ javac @sourcefiles
```

其中 `sourcefiles` 包含:

```
Foo.java
Bar.java
com/example/HelloWorld.java
```

注意：像这样编译代码仅适用于小型单人项目和一次性程序。 除此之外，建议选择和使用 Java 构建工具。 或者，大多数程序员使用 Java IDE（例如 NetBeans、eclipse、IntelliJ IDEA），它们都提供了嵌入式编译器和“项目”的增量构建。

#### 使用 `javac` 的常用选项

如下有可能对你有用的几个 `javac` 命令的选项：

- `-d` 选项，可设置用于写入“.class”文件的目标目录。
- `-sourcepath` 选项，用来设置源码的搜索路径。
- `-cp` 或者 `-classpath` 选项，设置查找外部和以前编译的类的搜索路径。 
- `-version` 选项，用来打印编译器的版本信息。

### 2、编译不同版本的 Java 代码

Java 编程语言（及其运行时）自首次公开发布以来经历了许多变化。 这些变化包括：

- Java 编程语言语法和语义的变化。
- Java 标准类库提供的 API 的变化。
- Java（字节码）指令集和类文件格式的变化。

除了极少数例外（例如 enum 关键字、对某些“内部”类的更改等），这些更改都是向后兼容的。

- 使用旧版 Java 工具链编译的 Java 程序也可以在新版 Java 平台上运行，而无需重新编译。
- 使用旧版 Java 编写的 Java 程序也可以使用新的 Java 编译器成功编译。

#### 使用新版的编译器编译旧版的 Java

如果你需要（重）编译一些旧的Java代码，使其可以在更新的Java平台运行，你通常无需给定任何特殊的编译标志。在少数情况下（例如，如果您使用 enum 作为标识符），可以使用 -source 选项禁用新语法。 例如，给定以下类：

```java
public class OldSyntax {
  private static int enum; // 在 Java 5 以及后续版本里是非法的
}
```

使用 Java 5 编译器（或更高版本）编译这个类需要以下内容：

```shell
$ javac -source 1.4 OldSyntax.java
```

#### 为旧的执行平台编译代码

如果您需要编译 Java 以在较旧的 Java 平台上运行，最简单的方法便是安装你所需的最老版本的 JDK，然后使用 JDK 的编译器来进行构建。

您也可以使用较新的 Java 编译器进行编译，但是比较复杂。首先，有一些重要的先决条件必须满足：

- 你正在编译的代码不得使用在目标 Java 版本中不可用的 Java 语言结构。
- 代码不得依赖旧平台不可用的标准的 Java 类库，字段，方法等。
- 代码所依赖的第三方库也必须为旧平台构建并在编译时和运行时可用。

在满足前提条件的情况下，你便可以使用 `-target` 选项为旧平台重新编译代码。 例如:

```shell
$ javac -target 1.4 SomeClass.java
```

则会编译上述类以生成与 Java 1.4 或更高版本 JVM 兼容的字节码。（事实上，`-source` 选项意味着相当于一个兼容的 `-target`，因此 `javac -source 1.4 ...` 将具有相同的效果。 Oracle 文档中描述了 `-source` 和 `-target` 之间的关系。）

话虽如此，如果只是使用 -target 或 -source，仍是根据编译器的 JDK 提供的标准类库进行编译。 如果你不小心，最终可能会得到具有正确字节码版本的类，但依赖于不可用的 API。 解决方案是使用 `-bootclasspath` 选项。 例如：

```shell
$ javac -target 1.4 --bootclasspath path/to/java1.4/rt.jar SomeClass.java
```

将针对一组替代的运行时库进行编译。 如果正在编译的类（意外）依赖于较新的库，便会抛出编译错误。

### 总结

本文回顾了一下 `javac` 命令，但更多的时候我们使用的 IDE 已经代劳了命令行的输入，所以了解一下即可。

### 链接

- `javac` 命令：https://docs.oracle.com/javase/8/docs/technotes/tools/windows/javac.html













