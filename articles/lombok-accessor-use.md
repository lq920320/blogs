## `Lombok` 中 `@Accessor` 的几个属性用法介绍

本文简单介绍一下`Lombok`中在 Model 类上使用的注解 `@Accessor` 的用法。

#### 源码

先看下 `@Accessor` 的源码，我们看到有三个属性：

```java
@Target({ElementType.TYPE, ElementType.FIELD})
@Retention(RetentionPolicy.SOURCE)
public @interface Accessors {

	boolean fluent() default false;
	
	boolean chain() default false;
	
	String[] prefix() default {};
}
```

#### `@Accessor(fluent = true)` 的用法

使用fluent属性，getter和setter方法的方法名都是属性名，且setter方法返回当前对象。

```java
@Data
@Accessors(fluent = true)
public class TestModelB {
    private Integer id;
    private String name;
}
```

````java
    @Test
    public void modelBTest() {
        TestModelB modelB = new TestModelB();
        modelB.id(124);
        modelB.name("B");

        System.out.println(modelB.id());
        System.out.println(modelB.name());
    }
````

#### `@Accessor(chain = true) `的用法

使用chain属性，setter方法返回当前对象，且可以像一条链一样进行属性赋值。

```java
@Data
@Accessors(chain = true)
public class TestModelC {
    private Integer id;
    private String name;
}
```

```java
    @Test
    public void modelCTest() {
        TestModelC modelC = new TestModelC();
        modelC.setId(125).setName("C");

        System.out.println(modelC.getId());
        System.out.println(modelC.getName());
    }
```

#### `@Accessor(prefix = "m") `的用法

忽略字段的前缀"m"。

```java
@Data
@Accessors(prefix = "m")
public class TestModelA {
    private Integer mId;
    private String mName;
}
```

```java
    @Test
    public void modelATest() {
        TestModelA modelA = new TestModelA();
        modelA.setId(123);
        modelA.setName("A");

        System.out.println(modelA.getId());
        System.out.println(modelA.getName());
    }
```

