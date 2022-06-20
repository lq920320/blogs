## Java基础（三）：引用数据类型


### 1、解引用（dereferencing）

解引用通常在使用 `.` 操作时发生：

```java
Obeject obj = new Object();
String text = obj.toString();  // obj 被解引用了
```

解引用遵循存储在引用中的内存地址，指向实际对象所在的内存位置。 找到对象后，将调用请求的方法（在本例中为 `toString`）。

当一个应用是 `null` 值时，解引用的结果便会抛出一个空指针异常（`NullPointerException`）：

```java
Object obj = null;
obj.toString(); // 当执行到这行代码时会抛出一个 NullPointerException
```

`null` 表示没有值，内存地址也无处指向。所以没有对象可以调用所请求的方法。

### 2、实例化引用类型

```java
Object obj = new Object(); // 注意 ‘new’ 关键字
```

何处：

- `Object` 时一个引用类型。
- `obj` 是一个存储新引用的变量。
- `Object()` 是调用 `Object` 的构造方法。

发生了什么：

- 内存为这个对象分配了空间。
- 调用构造函数 `Object()` 来初始化那个内存空间。
- 内存地址存储在 obj 中，以便它引用新创建的对象。

这与基本类型不同：

```java
int i = 10;
```

真实值 10 存储在变量 `i` 中。







