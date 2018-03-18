## 完美数的分类问题

古希腊数学家Nicomachus发明了一种自然数的分类方法，任意一个自然数都能唯一地被归为*过剩数(abundant)*、*完美数(perfect)* 或*不足数(deficient)*。一个完美数的真约数（即除了自身以外的所有正约数）之和，恰好等于它本身。例如6是一个完美数，因为它的约数是1、2、3，而6=1+2+3；28也是一个完美数，因为28=1+2+4+7+14。

**自然数分类规则**

| 定义 | 含义 |
| -------- | -------- |
|完美数   |  真约数之和=数本身|  
|过剩数   |  真约数之和>数本身| 
|不足数   |  真约数之和<数本身|

*真约数之和(aliquot sum)*，其定义就是除了数本身之外（一个数总是它本身的约数），其余正约数的和。

#### 1. 完美数分类的命令式解法

Java 代码如下：

```java
public class ImpNumberClassifierSimple{
  private int _number;                        //①
  private Map<Integer, Integer> _cache;       //②
  
  public ImpNumberClassifierSimple(int targetNumber) {
    _number = targetNumber;
    _cache = new HashMap<>();
  }
  
  public boolean isFactor(int potential) {
    return _number % potential == 0;
  }
  
  public Set<Integer> getFactors() {
    Set<Integer> factors = new HashSet<>();
    factors.add(1);
    factors.add(_number);
    for (int i = 2; i < _number; i ++){
      if (isFactor(i)){
        factors.add(i);
      }
    }
    return factors;
  }
  public int aliquotSum() {                   //③
    if (_cache.get(_number) == null){
      int sum = 0;
      for (int i : getFactors()){
        sum += i;
      }
      _cache.put(_number, sum - _number);
    }
    return _cache.get(_number);
  }
  
  public boolean isPerfect(){
    return aliquotSum() == _number;
  }
  
  public boolean isAbundant() {
    return aliquotSum() > _number;
  }
  
  public boolean isDeficient() {
    return aliquotSum() < _number;
  }
}
```
- ①内部状态，存放待分类的目标数字
- ②内部缓存，防止重复进行不必要的求和运算
- ③计算“真约数和”aliquotSum，即正约数之和减去数字本身

#### 2. 稍微向函数式靠拢的完美数分类解法

假如我们还希望加上一个“最小化共享状态”的目标，这时可以去掉类的成员变量，改为通过参数来传递需要的值。

```java
//稍微向函数式靠拢的完美数分类实现
public class NumberClassifier {
  public static boolean isFactor(final int candidate, final int number) {    //①
    return number % candidate == 0;
  }
  
  public static Set<Integer> factors(final int number) {         //②
    Set<Integer> factors = new HashSet<>();
    factors.add(1);
    factors.add(number);
    for (int i = 2; i < number; i ++){
      if (isFactor(i, number)){
        factors.add(i);
      }
    }
    return factors;
  }
  
  public static int aliquotSum(final Collection<Integer> factors) {         //③
    int sum = 0;
    int targetNumber = Collections.max(factors);
    for (int n : factors){
      sum += n;
    }
    return sum - targetNumber;
  }
  
  public static boolean isPerfect(final int number){
    return aliquotSum(factors(number)) == number;
  }
  
  public static boolean isAbundant(final int number){             //④
      return aliquotSum(factors(number)) > number; 
  }
  
  public static boolean isDeficient(final int number){
    return aliquotSum(factors(number)) == number; 
  }
}
```
- ①众多方法都必须加上number参数，因为没有可以存放的内部状态
- ②所有方法都带public static 修饰，因为它们都是纯函数，并因此可以在完美数分类之外的领域使用
- ③注意例中对参数类型的选取，尽可能宽泛的参数类型可以增加函数重用的机会
- ④例子目前在重复执行分类操作的时候效率较低，因为没有缓存。

在这个例子中，NumberClassifier里面，所有的方法都是自足的、带public和static作用域的纯函数（即没有任何副作用的函数）。
而由于类里面根本不存在任何内部状态，也就没有理由去“隐藏”任何一个方法。

一般来说，面向对象系统里粒度最小的重用单元是类，开发者往往忘记了重用可以在更小的单元上发生。例如aliquotSum方法的参数类型没有选择某一种具体的列表类型，而是定为Collection<Integer>。一个兼容所有数字集合的接口，在函数级别上发生重用的的可能性自然更大一些。

#### 3. 完美数分类的Java8实现

lambda块是Java8面目一新的改进，它其实就是高阶函数。

```java
//完美数分类的Java8实现
public class NumberClassifier {
  public static IntStream factorsOf(int number){
    return range(1, number + 1)
       .filter(potential -> number % potential == 0);
  }
  
  public static int aliquotSum(int number){
    return factorsOf(number).sum() - number;
  }
  
  public static boolean isPerfect(int number){
    return aliquotSum(number) == number;
  }
  
  public static boolean isAbundant(int number){
    return aliquotSum(number) > number;
  }
  
  public static boolean isDeficient(int number){
    return aliquotSum(number) < number;
  }
}
```

物理上把机械能分为储蓄起来的势能和释放出来的动能。在版本8以前的Java，以及它所代表的许多语言里，集合的行为可以比作动能：各种操作都立即求得结果，不存在中间状态。
函数式语言里是stream则更像是势能，它的操作可以引而不发。被stream储蓄起来的有数据来源（如range()方法），还有我们对数据设置的各种条件，如例中的筛选操作。只有都当程序员通过forEach()、sum()终结操作来向stream“要”求值结果的时候，才触发从“势能”到“动能”的转换。
在“动能”开始释放之前，stream可以作为参数传递并后续附加更多的条件，继续积蓄它的“势能”。这里关于“势能”的比喻，用函数式编程的说法叫作缓求值（lazy evaluation）。

#### 4. 完美数分类的Functional Java实现

```java
//使用Functional Java框架实现的完美数分类
import fj.F;
import fj.data.List;
import static fj.data.List.rnage;

public class NumberClassifier{
  public List<Integer> factorsOf(final int number){
    return range(1, number + 1)                        //①
          .filter(new F<Integer, Boolean>(){
            public boolean f(final Integer i){
              return number % i == 0;
            }
          });                                         //②
  }
  
  public int aliquotSum(List<Integer> factors){      //③
    return factors.foldLeft(fj.function.Integers.add, 0) - factors.last();
  }
  
  public boolean isPerfect(int number){
    return aliquotSum(factorsOf(number)) == number;
  }
  
  public boolean isAbundant(int number){
    return aliquotSum(factorsOf(number)) > number;
  }
  
  public boolean isDeficient(int number){
    return aliquotSum(factorsOf(number)) < number;
  }
}

```
- ①Functional Java的range()函数圈出来的是一个左闭右开区间。
- ②筛选操作代替了迭代。
- ③折叠（fold）操作代替了迭代。

“fold left”的含义是：

（1）用一个操作（或者是运算）将初始值（例中为0）与列表中的第一个元素结合；

（2）继续用同样的操作将第1步的运算结果与下一个元素结合；

（3）反复进行直到消耗完列表中的元素。

另一个值得注意的地方是factorsOf()方法，它很好地体现了“多着眼结果，少纠结步骤”的格言。
寻找一个数的约数，这个问题的实质是什么？或者换一种方式来叙述：在从1到目标数字的整数列表里，我们怎么确定其中哪些数字是目标数的约数？
这样一来，筛选操作就呼之欲出了——我们可以逐一筛选列表中的元素，去除那些不满足筛选条件的数字。
factorsOf()方法的作为基本上可以用一句话来描述：对于从1到目标数字的区间（range不含区间的右侧端点，因此代码中将区间上限写成number+1），以f()方法中的代码来筛选区间内数字所构成的一个列表，F类和f()方法是Functional Java留给我们“填空”数据类型和返回值的地方。

foldLeft()方法，它依次向左方，即向着第一个元素合并列表。对于满足交换律的加法来说，折叠的方向并不影响结果。万一我们需要使用某些结果与折叠次序相关的操作，还有foldRight()操作。


**高阶函数消除了摩擦**














