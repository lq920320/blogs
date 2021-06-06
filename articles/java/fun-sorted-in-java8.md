## 有趣的 `sorted()`

对于列表的排序，可以说是我们比较常见的场景了。Java 8 中引入了 `lambda` 以及 流式（Stream）计算，其中有一个排序的方法 `sorted()` ，`List`
对象本身也是实现了这个方法，可谓是排序好助手，今天我们就来写写关于这个排序方法的一些代码，不知道你是不是都用过。

### 1. 简单的列表排序

首先我们先创建一个测试的整型的列表：

```java
    private List<Integer> getTestList(){
        List<Integer> aList=new ArrayList<>();
        aList.add(2);
        aList.add(3);
        aList.add(9);
        aList.add(8);
        aList.add(5);
        aList.add(1);
        aList.add(4);
        aList.add(7);
        aList.add(6);
        return aList;
    }
```

可以使用 `Stream` 的方式进行直接排序，这里是创建了一个新的排序后的列表：

```java
    @Test
    public void listSort(){
        List<Integer> aList=getTestList();

        System.out.println("排序前：");
        aList.forEach(a->System.out.printf("%4d",a));
        System.out.println("\n排序后：");
        // 创建一个新列表
        aList=aList.stream().sorted().collect(Collectors.toList());

        aList.forEach(a->System.out.printf("%4d",a));
        System.out.println();
    }
```

结果为：

```
排序前：
   2   3   9   8   5   1   4   7   6
排序后：
   1   2   3   4   5   6   7   8   9
```

同样的，基于 `lambda` 我们可以用 `List` 实现的 `sort()` 方法来实现排序，我们可以看一下 `List.sort()` 的源码：

```java
    @SuppressWarnings({"unchecked", "rawtypes"})
    default void sort(Comparator<? super E>c){
        Object[]a = this.toArray();
        Arrays.sort(a, (Comparator)c);
        ListIterator<E> i = this.listIterator();
        for(Object e:a) {
            i.next();
            i.set((E)e);
        }
    }
```

可以看到，其参数是一个函数式接口（由 `@FunctionalInterface` 修饰的接口类） `Comparator`
，具体的实现方式大家可以自行研究，那么我们在使用的时候，就只需要考虑实现这个比较的接口就可以了，即按照我们的策略来实现 `Comparator.comparing(Function<? super T, ? extends U> keyExtractor)`
即可，这里因为是整型，所以我们可以直接按照默认顺序排序即可：

```java
    @Test
    public void listSort(){
        List<Integer> aList=getTestList();

        System.out.println("排序前：");
        aList.forEach(a->System.out.printf("%4d",a));
        System.out.println("\n排序后：");
        // 使用 List 实现的 sort 方法，comparing() 接收的是一个函数
        aList.sort(Comparator.comparing(a->a));

        aList.forEach(a->System.out.printf("%4d",a));
        System.out.println();
    }
```

这样我们也可以得到同样的结果：

```
排序前：
   2   3   9   8   5   1   4   7   6
排序后：
   1   2   3   4   5   6   7   8   9
```

### 2. 列表数据里有 `null`

作为一名 Java 程序员，`NPE` ，AKA `NullPointException`，可谓人生大敌，当列表中出现 `null` 时，应该如何排序呢？创建一个列表先：

```java
    private List<Integer> getTestListWithNull(){
        List<Integer> aList=new ArrayList<>();
        aList.add(2);
        aList.add(3);
        aList.add(9);
        aList.add(null);
        aList.add(8);
        aList.add(5);
        aList.add(null);
        aList.add(1);
        aList.add(4);
        aList.add(null);
        aList.add(7);
        aList.add(6);
        aList.add(null);
        return aList;
    }
```

可以看到，`null` 值穿插其中，这时候我们就要考虑相关的需求了，你是想要 `null` 值排在前面还是后面呢？`Comparator` 中 `comparing` 其中的一种实现源码如下：

```java
public static <T, U> Comparator<T> comparing(
            Function<? super T, ? extends U> keyExtractor,
            Comparator<? super U> keyComparator)
        {
            Objects.requireNonNull(keyExtractor);
            Objects.requireNonNull(keyComparator);
            return (Comparator<T> & Serializable)
            (c1, c2) -> keyComparator.compare(keyExtractor.apply(c1),
                                              keyExtractor.apply(c2));
        }
```

可以看到，除了接收的第一个 函数参数 外，还会接收另外一个子比较器 `Comparator` 接口，然后就找到了 `nullsLast` 和 `nullsFirst` 这两个方法，代码如下：

```java
    @Test
    public void listNullTest() {
        List<Integer> aList = getTestListWithNull();

        System.out.println("排序前：");
        aList.forEach(a -> {
            if (a != null) {
                System.out.printf("%4d", a);
            } else {
                System.out.printf("%8s", a);
            }
        });
        System.out.println("\n排序后：");
        // 正常的排序会报错
        // aList.sort(Comparator.comparing(a -> a)); // throw NPE
        aList = aList.stream().sorted(
            // nullsLast() / nullsFirst()
            // 对应的整个列表的顺序为： Comparator.naturalOrder() 正序；Comparator.reverseOrder() 反序
            Comparator.comparing(a -> a, Comparator.nullsLast(Comparator.naturalOrder()))
        ).collect(Collectors.toList());
        
        aList.forEach(a -> {
            if (a != null) {
                System.out.printf("%4d", a);
            } else {
                System.out.printf("%8s", a);
            }
        });
        System.out.println();
    }
```

这样，我们就可以实现按照空值在前或者空值在后实现排序：

```
排序前：
   2   3   9    null   8   5    null   1   4    null   7   6    null
排序后：
   1   2   3   4   5   6   7   8   9    null    null    null    null
```

### 3. 我有我自己的想法

有时候，规则需要我们自己定，而且计算机并不能很好的理解和计算我们制定的计算规则，比如我把一堆数据按照四季分好了四组，但是顺序是随机的，怎么才能按照“春”、“夏”、“秋”、“冬”的顺序排序呢？上面我们提到过，`comparing()` 的
函数参数 是可以按照我们想要的方式自己实现的，有点像策略模式，策略可以自己定。那么这时候，就需要一个辅助列表（需要的顺序规则），然后通过 `indexOf` 函数，按照下标的大小进行排序：

```java
    @Test
    public void listInSomeOrderTest() {
        List<String> seasons = new ArrayList<>();
        seasons.add("夏");
        seasons.add("冬");
        seasons.add("春");
        seasons.add("秋");

        System.out.println("排序前：");
        seasons.forEach(s -> System.out.printf("%4s", s));
        System.out.println("\n一般排序后：");
        seasons = seasons.stream().sorted().collect(Collectors.toList());
        seasons.forEach(s -> System.out.printf("%4s", s));
        System.out.println();

        // 固定顺序
        List<String> theOrders = Arrays.asList("春", "夏", "秋", "冬");
        // 按照 theOrders 排序
        seasons = seasons.stream().sorted(Comparator.comparing(theOrders::indexOf)).collect(Collectors.toList());
        System.out.println("按照固定顺序排序后：");
        seasons.forEach(s -> System.out.printf("%4s", s));
        System.out.println();
    }
```

这样就可以按照我们想法进行排序：

```
排序前：
   夏   冬   春   秋
一般排序后：
   冬   夏   春   秋
按照固定顺序排序后：
   春   夏   秋   冬
```

### 4. 多属性排序

后端程序员应该对 SQL 都不陌生，有时候我们在做统计的时候，应该遇到过这样的需求：一批工人里，拉一个单子，所有的工人年龄从小到大排，薪水从高到低排。翻译成 SQL
即为：查一批数据，同时按照年龄升序排列以及薪水倒序排列。想必在脑子里你已经写完这段 SQL 了，不过今天我们不写 SQL，同样地，先建一批数据：

```java
    private List<Worker> getTestDatas() {
        List<Worker> workers = new ArrayList<>();
        workers.add(new Worker() {{
            setId(1);
            setAge(20);
            setSalary(1000);
        }});
        workers.add(new Worker() {{
            setId(2);
            setAge(22);
            setSalary(1200);
        }});
        workers.add(new Worker() {{
            setId(3);
            setAge(20);
            setSalary(800);
        }});
        workers.add(new Worker() {{
            setId(4);
            setAge(20);
            setSalary(700);
        }});
        workers.add(new Worker() {{
            setId(5);
            setAge(22);
            setSalary(1800);
        }});
        workers.add(new Worker() {{
            setId(6);
            setAge(21);
            setSalary(1100);
        }});
        workers.add(new Worker() {{
            setId(7);
            setAge(22);
            setSalary(1600);
        }});
        workers.add(new Worker() {{
            setId(8);
            setAge(21);
            setSalary(1200);
        }});
        workers.add(new Worker() {{
            setId(9);
            setAge(20);
            setSalary(600);
        }});
        workers.add(new Worker() {{
            setId(10);
            setAge(20);
            setSalary(1200);
        }});
        workers.add(new Worker() {{
            setId(11);
            setAge(21);
            setSalary(1500);
        }});
        workers.add(new Worker() {{
            setId(12);
            setAge(20);
            setSalary(400);
        }});
        workers.add(new Worker() {{
            setId(13);
            setAge(20);
            setSalary(1100);
        }});
        workers.add(new Worker() {{
            setId(14);
            setAge(21);
            setSalary(1500);
        }});
        workers.add(new Worker() {{
            setId(15);
            setAge(21);
            setSalary(1600);
        }});
        return workers;
    }

@Data 
private static class Worker {
    /**
     * ID
     */
    private Integer id;
    /**
     * 年纪
     */
    private Integer age;
    /**
     * 薪水
     */
    private Integer salary;
}
```

*注：创建对象的方法为匿名类方式创建，仅测试代码中使用，不建议生产代码中使用。*

我们知道，流（`Stream`）范式的操作方法分为三类：Intermediate（中间操作），比如 `filter`、`map`、`peek`
等等，这类操作都是惰性（Lazy）化的，仅仅调用到这类方法，并没有真正执行流的遍历；Terminal（终止操作），比如 `toList`、`min`、`forEach`
等等，一个流只能有一个终止操作，当这个操作执行后，流就被使用「光」了，无法再被操作；Short-circuiting（短路操作/骤死操作），比如 `limit`、`anyMatch`、`findFirst` 等等，对于一个 terminal
操作，如果它接受的是一个无限大的流，但能在有限的时间计算出结果。当操作一个无限大的流，而又希望在有限时间内完成操作，则在管道内拥有一个 short-circuiting 操作是必要非充分条件。

好，通过上面的介绍我们可以知道，一个流只有一个终止操作，只有终止操作执行的时候才会进行流的遍历，而且只会遍历一次，时间复杂度为 `O(N)`，那么，我们可以放置两个中间操作 `sorted()` 来实现我们的多属性排序，代码如下：

```java

    @Test
    public void listWithFieldsTest() throws JsonProcessingException {
        // 根据年纪升序，根据薪水降序，得到一个有序的列表
        // 排序条件逆序设置：先排序的条件放在后面，后排序的条件放前面
        ObjectMapper objectMapper = new ObjectMapper();
        List<Worker> testWorkers = getTestDatas();
        // 原顺序
        System.out.println("排序前：");
        for (Worker testWorker : testWorkers) {
            System.out.println(objectMapper.writeValueAsString(testWorker));
        }
        // 排序后
        List<Worker> sortedWorkers = testWorkers.stream()
            // 薪水倒序
            .sorted(Comparator.comparing(Worker::getSalary, Comparator.reverseOrder()))
            .sorted(Comparator.comparing(Worker::getAge))
            .collect(Collectors.toList());
        System.out.println("排序后：");
        for (Worker sortedWorker : sortedWorkers) {
            System.out.println(objectMapper.writeValueAsString(sortedWorker));
        }
    }
```

然后，结果如预期：

```
排序前：
{"id":1,"age":20,"salary":1000}
{"id":2,"age":22,"salary":1200}
{"id":3,"age":20,"salary":800}
{"id":4,"age":20,"salary":700}
{"id":5,"age":22,"salary":1800}
{"id":6,"age":21,"salary":1100}
{"id":7,"age":22,"salary":1600}
{"id":8,"age":21,"salary":1200}
{"id":9,"age":20,"salary":600}
{"id":10,"age":20,"salary":1200}
{"id":11,"age":21,"salary":1500}
{"id":12,"age":20,"salary":400}
{"id":13,"age":20,"salary":1100}
{"id":14,"age":21,"salary":1500}
{"id":15,"age":21,"salary":1600}
排序后：
{"id":10,"age":20,"salary":1200}
{"id":13,"age":20,"salary":1100}
{"id":1,"age":20,"salary":1000}
{"id":3,"age":20,"salary":800}
{"id":4,"age":20,"salary":700}
{"id":9,"age":20,"salary":600}
{"id":12,"age":20,"salary":400}
{"id":15,"age":21,"salary":1600}
{"id":11,"age":21,"salary":1500}
{"id":14,"age":21,"salary":1500}
{"id":8,"age":21,"salary":1200}
{"id":6,"age":21,"salary":1100}
{"id":5,"age":22,"salary":1800}
{"id":7,"age":22,"salary":1600}
{"id":2,"age":22,"salary":1200}
```

> 这里我们留一个疑问：为什么先排序的条件放在后面，后排序的条件放前面呢？希望评论区给出你的见解。

### 结论

看完这些，你还有哪些独特的使用方式呢？也欢迎给出。通过本文我们可以知道，只要实现了自己的策略方法（`indexOf` 此类），那么就可以实现你想要的排序规则或者其他的什么需求，函数式编程的乐趣大概就在于自己可以创造属于自己的规则吧。

### 链接

- 代码 gitee 地址：https://gitee.com/lq920320/java-tests/blob/master/src/test/java/other/ListSortTest.java

