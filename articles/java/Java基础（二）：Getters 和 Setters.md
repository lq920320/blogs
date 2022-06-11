## Java基础（二）：Getters 和 Setters



本篇文章我们来讨论一下访问Java类中数据的标准方式，`getters` 以及 `setters`。

### 1、使用一个 `setter` 或 `getter` 来实现一个约束

对于一个包含私有变量的对象来说，`setters` 和 `getters` 可以在一些限制下访问和改变这些变量。比如：

```java
    public class Person {
        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            if (name != null && name.length() > 2) {
                this.name = name;
            }
        }
    }
```

在这个 `Person` 类中，有一个变量：`name` 。这个变量可以使用 `getName()` 方法访问获取，可以使用 `setName(String)` 来改变赋值，而且，赋值 `name` 需要一个新值并且长度要大于2个字母以及不能为 `null`。使用一个 `setter` 方法来允许其他类对 `name` 进行赋值，相对于直接将 `name` 变量设置为 `puiblic` （所有人都可以访问），有了一定的约束，这也更加安全。同样的逻辑也可以被用到 `getter` 方法上：

```java
public String getName() {
    if (name.length() > 16) {
        return "Name is too large!";  
    } else {
        return name;  
    }
}
```

在上面修改后的 `getName()` 方法中，`name` 只会返回长度小于等于16的值，否则，便会返回 “Name is too large!”。可以允许程序员按照他们的意愿来创建可访问和可修改的变量，而防止客户端类不必要的编辑变量。

### 2、为什么要使用 `Getters` 和 `Setters`?

来看一个Java中基本的类，包含一个变量的 `getters` 和 `setters` ：

```java
public void CountHolder {
  private int count = 0;
  
  public int getCount() {
    return count;
  }
  
  public void setCount(int c) {
    this.count = c;
  }
}
```

我们不能直接访问变量 `count` ，因为它是私有的（`private`）。但是我们可以通过 `getCount()` 和 `setCount(int)` 方法来访问，因为他们是公开的（`public`）。对一些人来说，便产生了疑问：为什么要引入一个中间人呢？为什么不直接将 `count` 设置为公开的呢？

```java
public void CountHolder {
  public int count = 0;
}
```

对于使用场景和意图上说，这两种方式在功能上是完全一致的。二者之间的区别在于可拓展性。来看一下每个类描述了什么：

- **First**：我有一个方法可以返回给你一个 `int` 数量值，还有一个方法可以让你把这个值改成别的 `int` 值。
- **Second**：我有一个 `int` 值，你可以随意获取和赋值。

听起来好像是很相似，但是 **first** 实际在本质上谨慎得多，它只允许你按照它的要求与它的内部属性进行交互。类本身便有了主要的控制权，它可以选择交互的发生方式。**second** 暴露了它的内部实现，现在不仅容易受到外部用户的影响和控制，而且在API情况下，需要致力于维护该实现（或以其他方式发布非向后兼容的API）。

让我们再考虑下，如果我们想要同步访问 `count` 来修改和获取其值时。首先，这是很简单的：

```java
public void CountHolder {
  private int count = 0;
  
  public synchronized int getCount() {
    return count;
  }
  
  public synchronized void setCount(int c) {
    this.count = c;
  }
}
```

但是在第2个例子里，如果不检查并修改引用 `count` 变量的每个位置，这现在几乎是不可能的。 更糟糕的是，如果这是在类库中提供给他人使用的类，那么你便无法执行该修改，并被迫做出上述艰难选择（发布非向后兼容的版本）。

所以它就引出了这个问题：公共变量是好事（或者至少，没那么坏）吗？

我不确定。一方面，可以看到经受住时间考验的公共变量的示例（比如：`System.out` 中引用的变量）。 另一方面，提供一个公共变量除了极端情况没有任何好处之外，却可以实现最小的开销和潜在的减少冗长。 我的指导方针是，如果你打算公开一个变量，应该根据以下标准来判断它，并带有极端的偏见：

1. 该变量应该没有任何可以臆想的理由来改变其实现。这是非常容易搞砸的事情（而且，即使你做对了，需求也会改变），这就是为什么 `getter`/`setter` 是常用方法的原因。 如果要拥有一个公共变量，则确实需要考虑清楚，尤其是在库/框架/API 中发布时。
2. 该变量需要足够频繁地引用，以保证减少冗长所带来的最小收益。 我甚至不认为使用方法与直接引用的开销应该是在这里考虑。 对于我保守估计的 99.9% 的应用程序来说，这太微不足道了。

我可能还有很多没有考虑到的。 如果有任何疑问，请始终使用 `getter`/`setter`。

### 3、添加 `Getters` 和 `Setters`

封装是OOP（面向对象编程）中的一个基本概念。 它是将数据和代码包装为一个单元。 在这种情况下，最好将变量声明为私有，然后通过 `Getter` 和 `Setter` 访问它们以查看或者修改。

```java
    public class Sample {
        private String name;
        private int age;

        public int getAge() {
            return age;
        }

        public void setAge(int age) {
            this.age = age;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }
```

这些私有变量不能由外部类直接访问，因此，它们得以免受一些无权访问的风险。但是，如果你想查看或者修改它们，可以使用 `Getter` 和 `Setter` 方法。

`getXxx()` 方法会返回变量 `xxx` 的现有值，而你也可以使用 `setXxx()` 方法来对变量 `xxx` 设置一个新值。

方法命名的约束是（例子中的变量为 `variableName`）：

- 所有的非 `boolean` 变量

  ```java
  getVariableName() // Getter，变量名称为驼峰
  setVariableName(..) // Setter，变量名称为驼峰
  ```

- `boolean` 变量

  ```java
  isVariableName() // Getter，变量名称驼峰
  setVariableName(..) // Setter，变量名称为驼峰
  ```

公共的 `Getter` 和 `Setter` 也是 Java Bean 中属性定义的一部分。

### 总结

我们在定义数据类供客户端访问的时候，还是要尽量定义成私有变量，然后对外提供 `getter` 和 `setter` 方法。