## 第3章  权责让渡

函数式思维的好处之一，是能够将低层次细节（如垃圾收集）的控制权移交给运行时，从而消弭了一大批注定会发生的程序错误。开发者们可以一边熟视无睹地享受着最基本的抽象，比如内存，一边却会对更高层次的抽象感觉突兀。。然而不管层次高低，抽象的目的总是一样的：让开发者从繁琐的运作细节里解脱出来，去解答问题中非重复性的那些方面。


####  3.1 迭代让位于高阶函数

理解掌握的抽象层次永远要比日常使用的抽象层次更深一层。

程序员的工作效率依赖于抽象层，好比没有人会直接翻弄硬盘上或0或1磁盘记录来给计算机编程。抽象隐藏了繁杂的细节，只是有时候会连同重要的考虑因素一起隐藏掉。


#### 3.2 闭包

闭包(closure)是所有函数式语言都具备的一项平常特性，可是相关的论述却常常充斥着晦涩乃至神秘的字眼。
所谓闭包，实际上是一种特殊的函数，它在暗地里绑定了函数内部引用的所有变量。换句话说，这种函数（或方法）把它引用的所有东西都放在一个上下文里“包”了起来。

*Groovy语言中闭包绑定的简单示例*

```groovy
class Employee {
    def name, salary
}

def paidMore(amount) {
    return {Employee e -> e.salary > amount}
}

isHighPaid = paidMore(100000)
```
首先定义了一个简单的Employee类，类中带了两个字段。接着定义带有amount参数的paidMore函数，其返回值是一个以Employee实例为参数的代码块，或者叫闭包。
类型声明Employee可写可不写，这里写出来顺便起到文档的作用。接下来，我们给代码块传入参数值100000，并赋予isHighPaid的名称，于是数值100000就随着这一步赋值操作，永久地和代码块绑定在一起了。以后有员工数据被带入这个代码块求解的时候，它就可以拿绑定的数值作为标准去评判员工的工资高低。

*执行闭包*

```groovy
def Smithers = new Emplyee(name: "Fred", salary: 120000)
def Homer = new Employee(name: "Homer", salary: 80000)
println isHighPaid(Smithers)
println isHighPaid(Homer)
// true, false
```
本例创建了两笔员工数据，然后判断其工资是否达到标准线。闭包在生成的时候，会把引用的变量全部圈到代码块的作用域里，封闭、包围起来（故名闭包）。
闭包的每个实例都保有自己的一份变量取值，包括私有变量也是如此。也就是说，我们可以创建paidMore闭包的另一个实例，给它绑定另外的数值（当然实例的名字也要另取）。

*绑定另一个闭包*

```groovy
isHigherPaid = paidMore(200000)
println isHigherPaid(Smithers)
println isHigherPaid(Homer)
def Burns = new Employee(name: "Monty", salary: 1000000)
println isHigherPaid(Burns)
// false, false, true
```
闭包经常被函数式语言和框架当成是一种异地执行的机制，用来传递待执行的变换代码，如map()之类的高阶函数。在缺乏闭包特性的旧版Java平台上，Functional Java利用匿名内部类来模仿“真正的”闭包的某些行为，但语言的先天不足导致这种模仿是不彻底的。

*闭包的原理（Groovy示例）*
```groovy
def Cloure makeCounter() {
  def local_variable = 0
  return {return local_variable += 1}  // ①
}

c1 = makeCounter()  //②
c1()               // ③
c1()
c1()

c2 = makeCounter()     // ④

println "C1 = ${c1()}, C2 = ${c2()}"
// output:C1 = 4, C2 = 1 ⑤
```
- ①函数的返回值是一个代码块，而不是一个值
- ②c1现在只想代码块的一个实例
- ③调用c1将递增其内部变量，如果这个时候输出，其结果也会是1
- ④c2现在指向makeCounter()的一个全新实例，与其他实例没有关联
- ⑤每个实例的内部状态都是独立的，各自拥有一份local_variable

makeCounter()函数首先定义一个局部变量，明白无误地命名为local_variable，接着返回一个使用了该局部变量的代码块。注意makeCounter()函数的返回类型是Closure，而不是一个单穿的值。
代码块的工作仅仅是递增并返回其局部变量的值。方法中两次明确写出了return关键字，其实这两个地方Groovy都允许省略，不过那样的话，代码看起来就有些晦涩了。

为了演示makeCounter()函数的用法，我们给代码块分配了一个变量名c1，然后调用了三次。调用代码块的时候用到了Groovy提供的语法糖衣，也就是在代码块变量名后直接跟一对圆括号的写法（否则应该写成c1.call()）。接下来，我们第二次调用了makeCounter()，将返回的又一个代码块实例赋给变量c2.最后我们把c1和c2都调用了一次。
从运行结果来看，两个代码块实例都分别持有自己的一份local_variable变量。“闭包”这个名字来源于它创建*封闭上下文*的行为。虽然局部变量不是在代码块里面定义的，但只要代码块引用了该变量，两者就被绑定在一起，这种联系在代码块实例的全部生命期内都一直保持着。

从实现的角度来说，代码块实例从它被创建的一刻起，就持有其作用域内的一切事物的封闭副本。当代码块实例被垃圾收集的时候，它持有的引用也同时被回收。

闭包在这里表现出的函数式思维就是“让运行时去管理状态”。

**让语言去管理状态。**

闭包还是*推迟执行*原则的绝佳样板。我们把代码绑定到闭包之后，可以推迟到适当的时机再执行闭包。这个特点在很多场合都能发挥作用。例如必要的变量和函数可能并不在定义时的作用域里，要到执行的时候才准备好。那么我们把执行上下文放在闭包里保留起来，就可以等到正确的时机再完成执行。

命令式语言围绕状态来建立编程模型，参数传递是其典型特征。闭包作为一种对行为的建模手段，让我们把代码和上下文同时封装在单一结构，也就是闭包本身里面，像传统数据结构一样可以传递到其他位置，然后在恰当的时间和地点完成执行。

**抓住上下文，而非状态。**

#### 3.3 柯里化和函数的部分施用

柯里化（currying）和函数的部分施用（partial application）都是从数学里借用过来的编程语言技法。柯里化和函数部分施用都有能力操纵函数或方法的参数数目，一般是通过向一部分参数代入一个或多个默认值的办法来实现的（这部分参数被称为“固定参数”）。

##### 3.3.1  定义与辨析
乍看起来，柯里化和部分施用的使用效果是一样的。两者都可以创建有一部分预设参数值的函数。

柯里化指的是从一个多函数变成一连串单参数函数的变换。它描述的是变换的过程，不涉及变换之后对函数的调用。调用者可以决定对多少个参数实施变换，余下的部分将衍生为一个参数数目较少的新函数。

部分施用指通过提前代入一部分参数值，使一个多参数得意省略部分参数，从而转化为一个参数数目较少的函数。这种技法叫作“部分施用”，顾名思义，就是让函数先作用于其中一些参数，经过部分的求解，结果返回一个由余下参数构成签名的函数。

柯里化和部分施用都是在我们提供部分参数值之后，产出可以凭余下参数实施调用的一个函数。不同的地方在于，函数柯里化的结果是返回链条中下一个函数，而部分施用是把参数的取值绑定到用户在操作中提供的具体值上，因而产生一个“元数”（参数的数目）较少的函数。用元数大于二的函数来套一下这里的解释，它们之间的区别就会比较清楚了。

举个例子，函数process(x, y, z)完全柯里化之后将变成process(x)(y)(z)的形式，其中process(x)和process(x)(y)都是单参数的函数。
如果只对第一个参数柯里化，那么process(x)的返回值将是一个单参数的函数，而这个唯一的参数又接受另一个参数的输入。
而部分施用的结果直接是一个减少了元数的函数。如果在process(x, y, z)上部分施用一个参数，那么我们将得到还剩下两个参数的函数：process(y, z).

##### 3.3.2 Groovy的情况
Groovy通过curry()函数实现柯里化，这个函数来自Closure类。

*Groovy语言中的柯里化*

```groovy
def product = {x, y -> x * y}

def quadrate = product.curry(4)      //①
def octate = product.curry(8)        //②

println "4×4：${quadrate.call(4)}"   //③
println "8×5：${octate(5)}"          //④
```
- ①调用curry()来固定一个参数，返回结果是一个单参数的函数。
- ②octate()函数总是对传入的参数乘以8.
- ③quadrate()是一个单参数的函数，可以通过Closure类的call()方法来调用它。
- ④Groovy提供了一层语法 糖衣，可以让调用语句的写法更自然一些。

本例首先定义接受两个参数的代码块product。利用Groovy内建的curry()方法，在product的基础上构造出两个新的代码块，quadrate和octate。
Groovy为调用代码块提供了特别的便利，我们既可以执行call()方法，也可以使用Groovy在语言层面提供的语法糖衣，也就是在代码块的名称后面紧跟一对圆括号，参数则写在括号里。

curry()虽然叫这个名字，它在背后对代码块所做的事情其实属于函数的部分施用。尽管名不符实，但用它来模拟出柯里化的效果还是可行的。做法是通过连续的部分施用使函数变形为一连串单参数的函数，如下。

*Groovy语言中部分施用与柯里化的对比*

```groovy
def volume = {h, w, l -> h * w * l}
def area = volume.curry(1)
def lengthPA = volume.curry(1, 1)         //①
def lengthC = volume.curry(1).curry(1)    //②

println "参数取值为2×3×4的长方体，体积为${volume(2, 3, 4)}"
println "参数取值为3×4的长方形，面积为${area(3, 4)}"
println "参数取值为6的线段，长度为${lengthPA(6)}"
println "参数取值为6的线段，经柯里化函数求得的长度为${lengthC(4)}"
```
- ①部分施用
- ②柯里化

两种写法只有微妙的区别，最终的计算结果也完全相同，但如果你在一名函数式程序员面前不加区分地使用这两个名词，他一定会纠正你。很不幸，Groovy把这两个密切相关的概念混为一谈了。

函数式编程赋予我们另一套新的构造单元，代替以往命令式语言所使用的机制来完成相同的目标。这些构造单元之间的关系经过了细致的安排。复合（composition），是函数式语言拼组这些构造单元的一般方式。

##### 3.3.5 一般用途
1. 函数工厂

我们在传统面向对象编程中会用到工厂方法的场合，正适合柯里化（以及部分施用）表现它的才干。如下，Groovy实现的加法函数和递增函数。

```groovy
def adder = {x, y -> x + y}
def incrementer = adder.curry(1)

println "incrementer 7 : ${incrementer(7)}"   //8
```
例中从adder()函数派生出了incrementer函数。

2. Template Method模式

GoF模式集里面有一项Template Method（模板方法）模式。其用意是在固定的算法框架内部安排一些抽象方法，为后续的具体实现保留一部分的灵活性。部分施用和柯里化也可以起到相同的作用。部分施用技法注入当前已经确定的行为，留下未确定的参数给具体实现去发挥，其思路与模板方法这种面向对象的设计模式如出一辙。

第6章将会用一个例子来说明，若干模式（包括模板方法模式在内）怎样因为部分施用和其他函数式技法而失去了存在意义。

3. 隐含参数

当我们需要频繁调用一个函数，而每次的参数值都差不多的时候，可以运用柯里化来设置隐含参数。举个例子，我们在操作持久化框架的时候，每次都要在第一个参数里写上数据源的位置。而经过部分施用以后，就不需要反复地写出这个参数值了。

*运用部分施用技法设置隐含参数值*

```
(defn db-connect[data-source query params]
    ...)
    
(def dbc (partial db-connect "db/some-data-source"))

(dbc "select * from %1" "cust") 
```
dbc函数在操作数据的时候不需要在提供数据源，数据源已经自动设置好了。面向对象编程中“封装”概念的本质，也就是魔术般出现在每个函数里的隐含上下文this，我们可以在函数式编程中加以模拟，用柯里化的方式吧this传递给所有的函数，让this在使用这的面前隐藏起来。

#### 3.4 递归
递归，按照维基百科的定义，是“一种自相似的方式来重复事物的过程”。它也是我们想运行时托付操作细节的一个例子，而且和函数式编程有着极为密切的联系。
以具体的时间来说，递归是以一种带点计算机科学味道的方式来对一组事物进行迭代，让事物的集合反复对自身调用同样的方法，使集合随着每次迭代不断缩小，同时要始终小心地保证退出条件的有效性。
很多时候，我们的问题核心就是对一个不断变短的列表反复做同一件事。



## 第4章   用巧不用蛮
我们转换范式的收获，表现在费更少的力气完成更多的事情。很多函数式编程构造的目的只有一个：从频繁出现的场景中消灭掉烦人的实现细节。

### 4.1 记忆

“memoization”这个词是英国的人工智能研究者Donald Michie生造出来的，指的是在函数级别上对需求多次使用的值进行缓存的机制。
目前来说，函数式编程语言皮鞭都支持记忆特性，有些事直接内建在语言里，也有一些需要开发者自行实现，但实现起来相对容易。

记忆可以用在这样的场合。假设我们有一个反复调用的函数，需要挖掘它的性能潜力。增加一个内部缓存是很容易想到的方案。每次我们根据一组特定的参数求得结果之后，就用参数值做查找用的键，把结果缓存起来。以后当函数又遇到相同参数的时候，就不需要重新计算一遍了，可以直接返回缓存的结果。
这种缓存函数计算结果的做法，是计算机科学里一种典型的折中方案：用更多的内存（我们一般不缺内存）去换取长期来说更高的效率。

只有纯（pure）函数才可以适用于缓存技术。纯函数是没有副作用的函数：它不引用其他值可变的类字段，除返回值之外不设置其他的变量，其结果完全由输入参数决定。
很显然，只有在函数对同样一组参数总是返回相同结果的前提下，我们才可以放心地使用缓存起来的结果。

#### 4.1.1 缓存
缓存是很常见的需求（同时也是制造晦涩错误的源头）。我们首先分两种情况去剖析函数缓存的用法，一种是类内部缓存，另一种是外部调用。
然后详细说明缓存的两种实现方式，一种是手工进行状态管理，另一种是记忆机制。

**1. 方法级别的缓存**

上一章判定数字归属的工作由Classifier类负责，我们可以想见其中一种典型的用例，是让同一个数字把几个分类方法都跑一遍。如下代码：

```
if(Classifier.isPerfect(n)) print "!"
else if(Classifier.isAbundant(n)) print "+"
else if(Classifier.isDeficient(n)) print "-"
```
按照先前的实现，被调用到的每一个分类方法，都只能够重新计算真约数和。这种情况恰好是*类内部缓存*的应用范本：在常规的使用中，每检查一个数字，都要调用sumOfFactors()方法若干次。
原来的实现方案对于一个频繁出现的用例来说过于低效。

**2. 缓存求和结果**

再利用已有的工作成果是提高代码效率的办法之一。因为真约数和的计算成本高昂，所以我们希望每个数字只计算一次。

```groovy
/**
* 缓存求和结果
*/
class ClassifierCachedSum {
  private sumCache = [:]
  def sumOfFactors(number){
    if (! sumCache.containsKey(number)){
      sumCache[number] = factorsOf(number).sum()
    }
  }
  return sumCache[number]
  
}
//其余代码不变
```

本例增加一个和类一起初始化的散列sumCache。在sumOfFactors()方法中，我们首先检查传入的参数是否已经在缓存里有对应的计算结果，有的话直接返回，否则才会执行昂贵的计算，并在返回之前把求和结果置入缓存。

这种情况是*外部缓存*的范本：调用方受益于缓存起来的结果，才有了第二次运行的高速。

缓存求和结果成绩斐然，但也付出了一些代价。ClassifierCachedSum不可以在纯粹有静态方法组成。类中的缓存就代表类有了状态，所有与缓存打交道的方法都不可以是静态的，于是产生了更多的连锁效应。
缓存可以提高性能，但缓存有代价：它提高了代价的非本质复杂性和维护负担。

**3. 缓存一切结果**

既然缓存求和的结果大大提高了代码的性能，何不试试把所有可能出现的中间结果都缓存起来呢？

```groovy
//缓存所有的计算结果
class ClassifierCached {
  private sumCache = [:], factorCache = [:]
  
  def sumOfFactors(number) {
    if (! sumCache.containsKey(number))
      sumCache[number] = factorsOf(number).sum()
    sumCache[number]
  }
  
  def isFactor(number, potential){
    number % potential == 0
  }
  
  def factorsOf(number) {
    if (!factorCache.containsKey(number))
      factorCache[number] = (1..number).findAll {isFactor(number,it)}
    factorCache[number]
  }
  
  def isPerfect(number) {
    sumOfFactors(number) == 2 * number
  }
  
  def isAbundant(number) {
    sumOfFactors(number) > 2 * number
  }
  
  def isDeficient(number) {
    sumOfFactors(number) < 2 * number
  }
}
```
ClassifierCached类除了缓存真约数的计算结果，还缓存了每个数的约数。
但是当我们把测试的取值范围增加8000个数字，马上就变成下面的结果：
```
java.lang.OutOfMemoryError: Java head space
```
这几次测试告诉我们，负责编写缓存代码的开发者不仅要顾及代码的正确性，连它的执行环境也要考虑在内。所谓“不确定因素”说的就是这样的东西：代码中的状态，开发者不仅要费心照应它，还要条分缕析它的一切明暗牵连。好在很多语言已经有了突破困境的办法，例如记忆机制。

#### 4.1.2 引入“记忆”

函数式编程费了很大力气来遏制不确定因素，并为此在运行时内建了许多重用机制。“记忆”是其中的一种特性，它作为编程语言的固有设施，自动地缓存重复出现的函数返回值。

Groovy语言记忆一个函数的办法是，先将要记忆的函数定义成闭包，然后对该闭包执行memoize()方法来获得一个新函数，以后我们调用这个新函数的时候，其结果就会被缓存起来。

“记忆一个函数”这件事情，运用了所谓的“元函数”技法：我们操纵的对象是函数本身，而非函数的结果。Groovy把记忆特性内建在它的Closure类里面，其他语言各有各的实现方式。

为了让sumOfFactors()得到像上一节“缓存求和结果”那样的缓存能力，我们记忆了sumOfFactors()方法，如下例：

```groovy
//记忆求和结果
package com.nelford.fl.memoization

class ClassifiermemoizedSum {
  def static isFactor(number, potential){
    return number % potential == 0
  }
  
  def static factorsOf(number) {
    (1..number).findAll {i -> isFactor(number, i)}
  }
  
  def static sumFactors = { number ->
    factorsOf(number).inject(0, {i, j -> i + j})
  }
  def static sumOfFactors = sumFactors.memoize()
  
  def static isPerfect(number) {
    sumOfFactors(number) == 2 * number
  }
  
  def static isAbundant(number) {
    sumOfFactors(number) > 2 * number
  }
  
  def static isDeficient(number) {
    sumOfFactors(number) < 2 * number
  }
}
```
按照代码块的格式（注意看=和参数的写法）来实现sumFactors()方法。方法本身平平无奇，说不定可以直接在哪个库里找到现成的。未来记忆sumFactors()，我们对它调用了memoize()方法，并将返回的新函数命名为sumOfFactors。

```groovy
//记忆所有计算结果
package com.nealford.ft.memization

class Classifiermemoized {
  def static dividesBy = {number, potential -> 
    number % potential == 0
  }
  def static isFactor = dividesBy.memoize()
  
  def static factorsOf(number) {
    (1..number).findAll{ i -> isFactor.call(number, i)}
  }
  
  def static sumFactors = { number ->
    factorsOf(number).inject(0, {i, j -> i + j})
  }
  def static sumOfFactors = sumFactors.memoize()
  
  def static isPerfect(number) {
    sumOfFactors(number) == 2 * number
  }
    
  def static isAbundant(number) {
    sumOfFactors(number) > 2 * number
  }
    
  def static isDeficient(number) {
    sumOfFactors(number) < 2 * number
  }
}
```
扩大记忆范围拖慢了第一次运行的速度，但后续运行的速度是所有版本中最快的——但这只是数据量很小的情况。
随着数据量变大，它的性能也像命令式缓存版本那样急剧下滑。事实上当数据量达到8000的时候，就出现了内存不足。命令式的版本要想防范这种陷阱，需要小心地查看警戒条件，注意执行环境是否超出安全范围——命令式编程的不确定因素又一次出现在我们面前。
相比之下，通过记忆方式实现的上例修正起来十分简单，只需要在函数的层次上做改动。修改后的记忆版本可以轻松应付10000条数据量。我们只需要用memoizeAtMost(1000)方法代替原来的memoize()。

表4-6：Groovy提供的几个记忆方法

| 方法 | 说明 |
| -------- | -------- |
|memoize()   |  将闭包转化为带缓存的实例|  
|memoizeAtMost()   |  将闭包转化为带缓存的实例，且规定了缓存的数量上限| 
|memoizeAtLeast()  |  将闭包转化为带缓存的实例，缓存大小可自动调整，且规定了缓存的数量下限|
|memoizeBetween()  |  将闭包转化为带缓存的实例，缓存大小可自动调整，且规定了缓存的数量上限和下限|

在命令式的思路下，开发者就是代码的主人（以及一切责任的承担者）。而函数式语言的思路是，为了操纵一些标准的构造，我们来制作一些通用的机件，有时候还在机件上设置若干调节旋钮（
也就是函数的不同变体和参数的不同组合）。函数是语言的基本元素，因此函数层面上的优化会附带产生功能的提升。实际上，我们写出来的缓存绝不可能比语言设计者产生的更高效，因为语言设计者可以无视他们给语言订的规矩：开发者无法触碰的底层设施，不过是语言设计者手中的玩物，他们拥有的优化手段和空间是“凡人”无法企及的。
但我们将缓存等问题交托给语言，不仅仅是因为它的效率更高，更因为我们从此可以在更高的层次上去思考问题。

*语言设计者实现出现的机制总是比开发者自己做的效率更高，因为他们可以不受语言本身的限制。*

手工建立缓存的工作不算复杂，但它给代码增加了状态的影响和额外的复杂性。而借助函数式语言的特性，例如记忆，我们可以在函数级别上完成缓存工作，只需要微不足道的改动，就能取得比命令式做法更好的效果。
在函数式编程消除了不确定因素之后，我们得以专注解决真正的问题。

当然，我们不需要先写好类，再往上添加及一层。memoize()和它的兄弟们都是定义在Closure类里面的库函数。

被记忆的内容应该是值不可变的，这一点非常重要。如果被记忆的函数在求解器返回结果的时候，需要依赖参数以外的任何因素，那么它的输出是不可预料的。如果被记忆的函数有副作用，那么当它直接返回缓存结果的时候，就跳过了产生副作用的那部分代码，没有执行。

*请保证所有被记忆的函数：*
- 没有副作用
- 不依赖任何外部信息

### 4.2 缓求值

缓求值（lazy evaluation）是函数式编程语言常见的一种特性，指尽可能地推迟求解表达式。缓求值的集合不会预先算好所有的元素，而是在用到的时候才落实下来，这样做有几个好处。
第一，昂贵的运算只有到了绝对必要的时候才执行。
第二，我们可以建立无限大的集合，只要一直接到请求，就一直送出元素。
第三，按缓求值的方式来使用映射、筛选等函数式的概念，可以产生更高效的代码。

```
//演示“非严格求值”的伪代码
print length([2+1, 3*2, 1/0, 5-4])
```

这段代码会得到怎样的执行结果，取决于所有编程语言的一项性质：它是*严格求值*（strict）的，还是*非严格求值*（non-strict）的（也叫缓求值，lazy）。
在严格求值的编程语言里，执行（甚至编译）这段代码，会因为列表中的第三个元素而发生“零被除”异常。而在非严格求值的语言里，它会得出4的结果，准确地报告列表元素的数目。
毕竟我们调用的方法叫作length()，而不叫lengthAndThrowExceptionWhenDivByZero()！常用的费严格求值语言有Haskell.Java虽然不属于这个阵营，但缓求值的思路仍然可以给我们带来好处。

#### 4.2.1 Java语言下的缓求值迭代子

我们的探索可以从*素数*（只能被1和它本身整除的自然数）类开始。

```java
//例4-11 寻找素数，Java实现
public class Prime{
  public static boolean isFactor(final int potential, final int number) {
    return number % potential == 0;
  }
  
  public static Set<Integer> getFactors(final int number) {
    Set<Integer> factors = new HashSet<>();
    factors.add(1);
    factors.add(number);
    for (int i = 2; i < sqrt(number) + 1; i ++){
      if (isFactor(i, number)){
        factors.add(i);
        factors.add(number / i);
      }
    }
    return factors;
  }
  
  public static int sumFactors(final int number) {
    int sum = 0;
    for (int i : getFactors(number)){
      sum += i;
    }
    return sum;
  }
  
  public static boolean isPrime(final int number){
    return sumFactors(number) == number + 1;
  }
  
  public static Integer nextPrimeFrom(final int lastPrime) {
    int candidate = lastPrime + 1;
    while (!isPrime(candidate)) candidate ++;
    return candidate;
  }
}
```
Java本身不提供缓求值的集合，但这并不妨碍我们实现一个特殊的Iterator来模拟缓求值集合。

```java
//例4-12 素数迭代子，Java实现
public class PrimeIterator implements Iterator<Integer> {
  private int lastPrime = 1;
  
  @Override
  public boolean hasNext(){
    return true;
  }
  
  @Override
  public Integer next(){ 
    return lastPrime = Prime.nextPrimeFrom(lastPrime);
  }
  
  @Override
  public void remove() {
    throw new RuntimeException("颠覆宇宙真理的异常");
  }
}
```
一般来说，开发者会把迭代子想象成在背后有一个存储数据的集合，实际上任何对象只要支持了Iterator接口，就可以算是一个迭代子。
hasNext()总是返回true，因为素数有无穷多个。
remove()方法在这里没有意义，因此我们让它在意为调用的时候抛出异常。
承担只要工作的next()方法在它仅有一行的方法体里做了两件事情。首先调用nextPrimeFrom()方法，根据上一个素数来找到下一个素数。其次，它利用Java用一条语句来同时完成赋值和返回操作的能力，更新了内部的lastPrime字段。

#### 4.2.2 使用Totally Lazy框架的完美数分类实现

```java
//例4-13  使用Totally Lazy Java框架实现的完美数分类
import com.googlecode.totallylazy.Predicate;
import com.googlecode.totallylazy.Sequence;

import com.googlecode.totallylazy.Predicate.is;
import com.googlecode.totallylazy.numbers.Numbers.*;
import com.googlecode.totallylazy.predicates.WherePredicate.where;

public class NumberClassifier {
  public static Predicate<Number> isFactor(Number n) {
    return where(remainder(n), is(zero));                    //①
  }
  
  public static Sequence<Number> getFactors(final Number n) {
    return range(1, n).filter(isFactor(n));
  }
  
  public static Sequence<Number> factors(final Number n) {
    return getFactors(n).memorise();
  }
  
  public static Number aliquotSum(Number n) {
    return subtract(factors(n).reduce(sum), n);
  }
  
  public static boolean isPerfect(Number n) {
    return equalTo(n, aliquotSum(n));
  }
  
  public static boolean isAbundant(Number n) {
    return greaterThan(aliquotSum(n), n);
  }
  
  public static boolean isDeficient(Number n) {
    return lessThen(aliquotSum(n), n);
  }
}
```
- ①remainder等函数和where等谓词都是框架提供的。

开头的连串静态导入让我们得以省略频繁出现的类前缀，减少了行文中的干扰。经过这样的处理，代码已经不像典型的Java语句，但可读性很高。
Totally Lazy框架对Java的不宜不可能越出Java语法发界限，因此它没有运用运算符重载，而通过增加适当的方法来改善表现力。于是`num % i == 0`就写成了`where(remainder(n), is(zero))`的形式。

Totally Lazy的简便语法有一部分是受到JUnit测试框架（http://junit.org）的扩展库Hamcrest(https://code.google.com/p/hamcrest)的启发，甚至直接使用了Hamcrest的一些类。
在Totally Lazy的remainder()方法和Hamcrest的is()方法携手之下我们用一次where()调用来完成了isFactor()方法的工作。
factors()方法也变成对range()对象进行的一次filter()调用。约数的求和工作则由我们现在已经很熟悉的reduce()方法来完成。
由于Java不支持运算符重载，求解aliquotSum所需要的减法运算也只好变成了对subtract()方法的调用。最后，isPerfect()方法使用Hamcrest提供的equalTo()方法来判断目标数本身是否等于它的真约数和。

Totally Lazy利用Java语言中不甚起眼的静态导入特性，出色地营造了富于可读性的代码。很多开发者武断地相信Java是一种糟糕的内部DSL(领域专用语言)宿主，Totally Lazy戳破了这种论调。缓求值也是Totally Lazy积极运用的原则之一，一切操作都被尽可能地推迟。
我们如果希望建立更传统一些的缓求值数据结构，高阶函数会是很重要的一件工具。

#### 4.2.5 缓求值的好处

缓求值列表有几个好处。第一，我们可以用它创建无限长度的序列。由于不需要求解还没用到的元素值，我们可以用缓求值集合来建模无限列表。

第二，减少占用的存储空间。假如能够用推导的方法得到后续的值，那就不必预先存储完整的列表了——这是牺牲速度来换取存储空间的做法。是否采用缓求值集合，取决于我们如何权衡元素值的存储和计算的花费。

第三，缓求值集合有利于运行时产生更高效率的代码。


## 第5章  演化的语言

函数式编程语言和面向对象语言对待待代码重用的方式不一样，面向对象语言喜欢大量地建立有很多操作的各种数据结构，函数式语言也有很多操作，但对应的数据结构却很少。
面向对象语言鼓励我们建立专门针对某个类的方法，我们从类的关系中发现重复出现的模式并加以重用。
函数式语言的重用表现在函数的通用性上，它们鼓励在数据结构上使用各种共通的变换，并通过高阶函数来调整操作以满足具体事项的要求。

### 5.1 少量的数据结构搭配大量的操作

*100个函数操作一种数据结构的组合，要好过10个函数操作10种数据结构的组合。 ——Alan Perlis*

在面向对象的命令式编程语言里面，重用的单元是类和用作类间通信的消息，通常可以表述成一幅类图。

比起在定制的类结构上做文章，把封装的单元缩小到函数级别，有利于在更基础的层面上更细粒度地实施重用。

### 5.2 让语言其迎合问题

很多开发者都有一种误解，认为自己的工作就是把复杂的业务问题翻译成某种编程语言，如Java。他们会有这样的想法，原因在Java身上。Java不是一种特别灵活的语言，我们只能死板地用一些现成的结构来拼凑自己的设计。
而另外一些开发者使用的语言可塑性更强，他们不会拿问题去硬套语言，而是想办法揉捏手中的语言来迎合问题。

重塑语言的能力算不上函数式语言独有的特性，现代语言普遍可以轻巧地揉捏语言来贴合问题域，不过在这种能力的影响下，更容易催生带有浓厚函数式、描述式风格的代码。

*让程序去贴合问题，不要反过来。*

### 5.3 对分发机制的再思考

我们用“分发机制”这个词来泛称各种语言中用作“动态地选择行为”的特性。与Java的做法相比，几种函数式JVM语言的分发机制更加简洁、灵活，比如Groovy，Clojure等。

### 5.4 运算符重载

运算符重载是函数式语言常见的特性，它允许我们重新定义运算符（诸如+、-、*），使之适用于新的类型，并承载新的行为。Java语言没有运算符重载特性，这是它的设计者在语言形成阶段就刻意作出的决定，不过时至今日，几乎所有现代语言都包含了运算符重载特性，连同Java平台上的一众衍生语言在内。

#### 5.4.1 Groovy

Groovy希望改造Java的语法，同时又自然地保留了Java的语义。在此思路下，Groovy将运算符自动映射成方法，从而令运算符重载变成了方法的实现问题。例如我们想针对Integer类重载`+`运算符的话，只要覆盖该类的plus()方法即可。完整的映射列表可以查阅Groovy的文档，下表列出了几种常用运算符的映射关系。

表5-1：Groovy语言部分运算符和方法之间的映射关系

| 运算符 | 方法 |
| -------- | -------- |
|x + y   |  x.plus(y)  |  
|x * y   |  x.multiply(y)| 
|x / y   |  x.div(y)   |
|x ** y  |  x.power(y) |

#### 5.4.2 Scala

Scala也支持运算符重载，它的做法是完全不区分运算符和方法：运算符不过是一些名字比较特别的方法罢了。因此，如果我们想覆盖乘法运算符的默认行为，只要实现`*`方法就可以了。

Java语言的设计者从使用C++语言的经验中得出结论，认为运算符重载会给语言增加过多的复杂性，因此刻意从Java语言中排除了这种特性。现代语言大多数已经相当程度地消除了定义上的复杂性，但以往关于滥用运算符重载的告诫都还是成立的。

*想要契合问题的表达习惯，可以利用运算符重载来改变语言的外貌，不必创造全新的语言。*

### 5.5 函数式的数据结构

很多函数式语言根本就没有Java那样的“异常”概念，它们肯定有别的方式可以表达错误状况下的行为。

“异常”违背了大多数函数式语言所遵循的一些前提条件。首先，函数式语言偏好没有副作用的纯函数。抛出异常的行为本身就是一种副作用，会导致程序路径偏离正轨（进入异常的流程）。
函数式语言以操作值为其根本，因此喜欢在返回值里表明错误并作出响应，这样就不需要打断程序的一般流程了。

*引用透明性*（referential transparency）是函数式语言重视的另一项性质：发出调用的例程不必关心它的访问对象真的是一个值，还是一个返回值的函数。可是如果函数有可能抛出异常的话，用它来代替就不再是安全的了。

#### 5.5.1 函数式的错误处理

我们在Java语言下抛开异常来处理错误，返回值的限制是首先会遇到的绊脚石，语言规定了方法只能返回一个值，不过，我们可以把多个值装进单个Object（或其子类）对象里面一起返回，比如利用Map来返回多个值。

用Map来返回多个值的设计存在一些明显的缺点。首先，Map中放置内存没有类型安全的保障，编译器无法捕捉到类型方面的错误。假如改用枚举类型来充当键，可以稍微弥补这个缺点，但效果有限。
第二，方法的调用者无法直接得知执行是否成功，需要逐一比对所有可能键，增加负担。
第三，没有办法强制结果只含有一对键值，届时将出现歧义。

我们真正需要的机制，不但要返回两个（或更多）值，还必须能够保证类型安全。

#### 5.5.2 Either类

函数式语言也经常会与遇到返回两种截然不同的值的需求，它们用来建模这种行为的常用数据结构是Either类。Either的设计规定了它要么持有“左值”，要么持有“右值”，但绝不会同时持有两者。
这种数据结构也被称为不相交联合体（disjoint union）。C语言和一些衍生语言中会有一种联合体（union）数据结构，能够在同一个位置上容纳不同类型的单个实例。
不相交联合体为两种类型的实例都准备了位置，但只会持有其中一种类型的单个实例。

#### 5.5.3 Option类

Either类代表了一种使用方便、用途广泛的概念，除了用来实现双重返回值，还被用来构建一些通用的数据结构。在表示函数返回值这个用途上，还有另外一个选项——Option。Option类表述了异常处理中将为简化的一种场景，它的取值要么是none，表示不存在有效值，要么是some，表示成功返回。

```java
//Option 的用法
public static Option<Double> divide(double x, double y){
  if(y == 0)
    return Option.none();
  return Option.some(x / y);
}
```

从上例可以看到，Option的取值为none()或some()其中之一，类似于Either的左值和右值，只不过专为表达“方法不一定返回有效结果”的意思而缩窄了定义。
Option类可以近似地看作Either类的一个子集；Option一般只用来表示成功或者失败两种情况，而Either可以容纳任意的内容。

#### 5.5.4 Either树和模式匹配

Either利用Java的泛型支持，形成一种双重类型的数据结构，可以在其左值或右值上不同时地容纳两种类型的取值。比如我们可以让Either容纳Exception（左值）或者Integer（右值）类型的取值，如下图：

___________________________

|  Exception |  Integer   |

___________________________

接下来，我们准备在Either抽象的基础上构造一棵树形结构，这样做的好处要从*模式匹配*说起。

##### 1.Scala的模式匹配

作为分发机制的模式匹配是Scala语言吸引人的特性之一。

```scala
//例5-32  使用Scala模式匹配来实现成绩分等
val VALID_GRADES = Set("A", "B", "C", "D", "F")

def letterGrade(value : Any) : String = value match {
  case x: Int if (90 to 100).contains(x) => "A"
  case x: Int if (80 to 90).contains(x) => "B"
  case x: Int if (70 to 80).contains(x) => "C"
  case x: Int if (60 to 70).contains(x) => "D"
  case x: Int if (0 to 60).contains(x) => "F"
  case x: String if VALID_GRADES(x.toUpperCase)) => x.toUpperCase
}

//例5-33  测试成绩分等函数
printf("Amy的成绩为%d，获得%s等\n", 91, letterGrade(91))
printf("Bob的成绩为%d，获得%s等\n", 72, letterGrade(72))
printf("Sam的成绩为%d，获得%s等\n", 44, letterGrade(44))
printf("Roy转学前已获%s等，记为%s等\n", "B", letterGrade("B"))
```

letterGrade函数的整个函数体都是针对其参数不同取值的match块。我们设定了一系列的模式来防备每一种取值情况，除了参数的类型，这些模式还可以根据各种
细致的条件来划分和筛选参数的取值。模式匹配的语法将每一种取值情况划分明了，比起笨拙的连串if语句高明多了。

模式匹配可以和Scala的case类联用，这种特殊性质的类可以帮助我们隐去case语句中冗长的条件判断，将之转移到case类的定义里。

```scala
//例5-34  Scala中针对case类的匹配模式
class Color(val red:Int, val green:Int, val blue:Int)

case class Red(r:Int) extends Color(r, 0, 0)
case class Green(g:Int) extends Color(0, g, 0)
case class Blue(b:Int) extends Color(0, 0, b)

def printColor(c:Color) c match {
  case Red(v) => println("Red: " + v)
  case Green(v) => println("Green: " + v)
  case Blue(v) => println("Blue: " + v)
  case col:Color => {
    print("R: " + col.red + ",")
    print("G: " + col.green + ",")
    println("B: " + col.blue)
  }
  
  case null => println("invalid color")
}
```

首先创建了基类Color，然后以case类的形式建立特化的版本来表示单色。为了在函数中判断传入的参数是哪一种颜色，我们使用了match来对所有可能的取值选项作模式匹配，最后还匹配处理了空值的情况。

**Scala的case类**

面向对象系统，尤其是一些差异较大，而需要互相通信的系统之间，经常使用一些简单的类来作为数据的承载容器。case类自动地附带了以下语法便利。

- 类名可以直接用作一个工厂方法。我们不必动用new关键字就可以构造一个新实例，如val bob = Person("Bob", 42)。
- 经类参数列表传入的所有值都会自动被赋予val类型，也就是说它们都成了类中值不可变的内部字段。
- 编译器自动为case类生成合理的equals()、hashCode()和toString()默认实现。
- 编译器添加到类中的copy()方法可以通过返回新副本的形式，实现对原实例的字段修改。


Java不支持模式匹配，因此我们没有办法写出像Scala那样清晰可读的分发代码。但如果让泛型和我们熟悉的数据结构联起手来，未必不能学到一点神韵。

##### 2.Either树

表5-2：构造一棵树需要的三种抽象

 | 树的抽象 | 说明 |
 | -------- | -------- |
 |empty  |  没有值的单元  |  
 |leaf   |  放置了某种数据类型的值的单元 | 
 |node   |  指向其他的leaf或node   |
 
 我们直接使用Functional Java框架中的Either类。理论上，Either 抽象内可以放置数据的栏位可以扩展为任意的数量。Either<Empty, Either<Leaf, Node>>
 
 ```java
//例5-35  在Either上建立起来的树结构
import fj.data.Either;
import static fj.data.Either.left;
import static fj.data.Either.right;

public abstract class Tree {
  private Tree() {}
  
  public abstract Either<Empty, Either<Leaf, Node>> toEither();
  
  public static final class Empty extends Tree {
    public Either<Empty, Either<Leaf, Node>> toEither(){
      return left(this);
    }
    public Empty(){}
  }
  
  public static final class Leaf extends Tree {
    publi final int n;
    
    @Override
    public Either<Empty, Either<Leaf, Node>> toEither() {
      return right(Either.<Leaf, Node>left(this));
    }
    
    public Leaf() {}
  }
  
  public final class Node extends Tree {
    public final Tree left;
    public final Tree right;
    
    public Either<Empty, Either<Left, Node>> toEither() {
      return right(Either.<Left, Node>right(this));
    }
    
    public Node(Tree left, Tree right){
      this.left = left;
      this.right = right;
    }
  }
}
```

Tree的内部类在其内部定义了三个final具体类：Empty、Leaf、Node。Tree类规定了Empty要放在最左边的位置，Leaf放在中间，Node放最右边。
每个内部类都实现了toEither()方法，保证本类的实例放在了三值Either中正确的栏位上。我们用来搭建树结构的构造单元，三值Either，相当于传统计算机科学术语中“联合体”（union）的概念，虽然允许放入三种类型，但在任意时刻只会持有其中的一种。

##### 3.以模式匹配的方式实现树的遍历

Scala的模式匹配鼓励我们把不同的情况分开考虑。Functional Java 框架中Either的left()和right()都实现了Iterable接口，我们模拟模式匹配的基本条件已经满足。

```java
//例5-36 模仿模式匹配的语法来获取树的深度
static public int depth(Tree t){
  for(Empty e : t.toEither().left())
    return 0;
  for(Either<Leaf, Node> ln : t.toEither().right()){
    for(Leaf leaf : ln.left())
      return 1;
    for(Node node : ln.right())
      return 1 + max(depth(node.left), depth(node.right));
  }
  throw new RuntimeException("Inexhaustible pattern match on tree");
}
```

depth()方法是一个递归的深度查找函数。由于我们的树采用了特殊的单元结构（<Empty, <Left, Node>>），我们可以将每一个“栏位”都当成一个“case”来对待。
如果单元是empty，那么这根枝条的深度为零。如果单元是leaf，那么我们给树的深度累计一层。如果单元是node，那么我们应该累计一层，并继续帝国搜索单元的左子树和右子树。

```java
//例5-37  判定给定值在树中是否存在
static public boolean inTree(Tree t, int value) {
  for(Empty e : t.toEither().left())
    return false;
  for(Either<Leaf, Node> ln : t.toEither().right()){
    for(Leaf leaf : ln.left()){
      return value == leaf.n;
    }
    for(Node node : ln.right())
      return inTree(node.left, value) | inTree(node.right, value);
  }
  return false;
}
```

两个例子都是按照单元结构的“栏位”来返回相应结果。
如果遇到empty单元，我们就返回false，表示搜索失败了。如果遇到leaf， 我们检查单元中放置的值，如果匹配就返回true。当遇到node单元的时候，我们继续递归搜索下游分支，用|（非短路或运算符）合并左右子树的搜索结果。


## 第6章  模式与重用

### 6.1 函数式语言中的设计模式

函数式世界中有一种代表性的意见认为，设计模式的概念本身在函数式编程中就站不住脚，函数式编程不需要这样的东西。
如果我们认可设计模式是“赋予了名字的、编目记录下来的常见问题的解决方案”，那么这个概念一点都不过时。只不过在不同的范式下，模式有可能呈现为截然不同的外在形象。
因为函数式世界用来搭建程序的材料不一样了，所以解决问题的手法也不一样了，GoF模式集中有一部分传统模式失去了存在的意义，但还有一部分模式，它们要解决的问题依然存在，只是解决的手段发生了很大的变化。

传统设计模式在函数式编程的世界中大致有三种归宿。

- 模式已被吸收成为语言的一部分。
- 模式中描述的解决办法在函数式范式下依然成立，但实现细节有所变化。
- 由于在新的语言或范式下获得了原本没有的能力，产生了新的解决方案（例如很多问题都可以用元编程干净利落地解决，但Java没有元编程能力可用）。

### 6.2 函数级别的重用



















