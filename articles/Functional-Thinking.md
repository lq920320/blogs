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

复合（composition），作为一种重用机制，在函数式语言中主要表现为通过参数来传递作为第一等语言成分的函数，各种函数式编程库都频繁地运用了这种手法。
与面向对象语言相比，函数式语言的重用发生于较粗的粒度级别上，着眼于提取一些共通的运作机制，并参数化地调整其行为。面向对象又一群互相发送通信消息（或者叫调用方法）的对象组成。

从根本上说，以模式为载体的重用是细粒度的：一种解答方案（如Flyweight模式）与另一种解答方案（如Memento模式）之间，是井水不犯河水的正交关系。
设计模式所解决的都是很专门的问题，模式的用处就在于我们能够为手头的问题找到对应的模式，但反过来，模式和问题之间这种狭窄的对应关系又限制了它的使用面。

函数式编程不追求复现结构之间经典的（耦合）关系，它以定义各类型“对象”之间“态射”（morphism）关系的数学分支——范畴论为基础，希望从代码中抽取另一种粗粒度的脉络加以重用。
大多数应用都离不开对列表元素的操作，函数式的重用机制就建立在列表的概念，以及可以连同执行上下文一起传递的代码块的概念之上。函数式依赖作为第一等语言成分的函数（第一等的函数允许出现在其他任何语言构造允许出现的位置上）去充当参数和返回值。

*[态射（morphism）](https://baike.baidu.com/item/%E6%80%81%E5%B0%84/7289870?fr=aladdin)：数学上，态射（morphism）是两个数学结构之间保持结构的一种过程抽象。最常见的这种过程的例子是在某种意义上保持结构的函数或映射。
例如，在集合论中，态射就是函数；在群论中，它们是群同态；而在拓扑学中，它们是连续函数；在泛代数（universal algebra）的范围，态射通常就是同态。*

函数式语言不需要那么辅助性的支撑结构和公式化的死板套语，它可以 帮助我们卸去一部分累赘，同时又实现与传统方式相同的设计概念。例如当语言有了闭包特性，就不需要Command模式了。
从根本上说，设计模式的存在意义就是弥补语言功能上的弱点，例如不把“行为”用一个本身无甚意义的骨架类包装起来的话，就没办法像传递数值一样传递它。当然，Command模式还有其他用途，
例如实现操作回退（undo），不过它的主要功能是打开一条门路让代码块能够被传递到方法中执行。

#### 6.2.1  Template Method模式

函数被提升为第一等的语言成分，对Template Method模式的实现有简化的效果，因为可以消除一些为了迂回语言限制而存在的结构。
Template Method模式在一个方法里面定义好算法的骨架，但留下一部分未实现的步骤，强迫子类按照规定好的算法结构来补全缺失的步骤定义。

```groovy
//例6-1 Template Method模式的“标准实现”
abstract class Customer {
  def plan
  
  def Customer(){
     plan = []
  }
  def abstract checkCredit()
  def abstract checkInventory()
  def abstract ship()
  
  def process(){
     checkCredit()
     checkInventory()
     ship()
  }
}

```
例6-1中process()方法依赖于checkCredit()、checkInventory()和ship()三个方法，这三个方法被设定为抽象方法，子类必须为它们提供具体的实现。

//例6-2 第一等函数对Template Method模式的影响

```groovy
class CustomerBlocks{
  def plan, checkCredit, checkInventory, ship
  
  def CustomerBlocks() {
     plan = []
  }
  
  def process() {
     checkCredit()
     checkInventory()
     ship()
  }
}
```
原先需要特地按照规定格式来声明的算法步骤，在6-2中成了类里面最普通不过的属性，可以像普通美好的属性一样赋值。这正是一个实现细节被语言特性吸收掉的例子。

前后两个实现并不等价。按照6-1的Template Method模式“传统”实现，子类必须补全算法依赖的几个抽象方法的实现。虽然子类可以用空的方法体应付了事，但绝不可能全然无视这些空缺的方法。
抽象方法的定义相当于一种特殊形式的文档，提醒子类将指定的方法纳入考虑。然而反过来，事先规定好全部的方法声明又有僵化之虞，未必适合需要更多灵活性的情况。
例如我们也许希望任意指定Customer类处理时使用的方法序列。

语言对代码块等特性支持得越深入，对开发者的亲和力就越好。


#### 6.2.2  Strategy模式

Strategy模式也是因为第一等函数而得到简化的一种常见模式。Strategy模式定义一个算法族，并将每一种算法都在相同的接口下封装起来，令同一族的算法能够互换使用。
这样做的好处是算法的变化不影响使用方，也不受使用方的影响。第一等函数让建立和操纵各种策略的工作变得十分简单。

例6-4给出了Strategy模式的一个传统实现。

```groovy
//例6-4 用Strategy模式来处理两个数字的积的问题
interface Calc {
   def product(n, m)
}

class CalcMult implements Calc {
   def product(n, m) {n * m}
}

class CalcAdds implements Calc {
   def product(n, m) {
     def result = 0
     n.times {
       result += m
     }
     result
   }
}
```
例6-4为计算两个数字的积定义了统一的接口。我们在接口下实现了两个具体类（也就是两种策略），一个直接用乘法计算，另一个用累加的方法。

例6-4公式化的累赘成分太多了，加入我们正确发挥Groovy作为第一等函数的能力，应该可以做得更好。

```groovy
// 例6-6 简洁地表达和测试幂的不同计算策略
@Test
public void exp_verifier() {
  def listOfExp = [
     {i, j -> Math.pow(i, j)},
     {i, j -> 
       def result = i
       (j-1).times {result *= i}
       result
     }]
     
  listOfExp.each {e -> 
    assertEquals(32, e(2, 5))
    assertEquals(100, e(10, 2))
    assertEquals(1000, e(10, 3))
  }
}
```
例6-6的两种幂计算策略都是以Groovy代码块的形式就地定义的，我们为表述上的便利而牺牲了规范的形式。传统手法规定了每种策略的名称和结构，在某些情况下更可取。
但例6-6的好处是我们可以随时给代码增加更严格的防护，而想要突破传统方式设下的框框就没那么容易了。

#### 6.2.3 Flyweight模式和记忆

Flyweight模式是一种在大量的细粒度对象引用之间共享数据的优化技巧。我们维护一个对象池，然后引用池中的对象来构成需要的视图。

Flyweight模式使用了“标准品”对象的概念——可以代表所有其他同型对象的一个典型的对象实例。

同理，我们在程序中没必要为每个用户都创建各自的产品对象列表，相反可以只创建一份标准品的列表，让用户引用列表中的对象来表示自己持有的产品。

把不同实例共用的信息保存起来是个好主意，我么希望把它也带到函数式编程中去。我们在第4章讨论过函数“记忆”，被记忆的函数允许运行时缓存其结果。
比如在函数内定义了标准品，我们只需要在这个函数上调用memoize()方法，就可以的带相应的带记忆能力的函数实例。

#### 6.2.4 Factory模式和柯里化

在设计模式的语境下，柯里化相当于产出函数的工厂。第一等函数（或高阶函数）是函数式编程语言共同的特性，我们可以用函数来充当其他任何的语言成分。
因此我们可以很容易地设立一个根据条件来返回其他函数的函数，也就是函数工厂。我们可以可以看一个例子，假设有一个用于两数相加的普通函数，经过柯里化加工，我们可以制造出一个总是其参数加一的递增函数。
```groovy
//例6-13 作为函数工厂的柯里化
def adder = {x, y -> x + y}
def incrementer = adder.curry(1)

println "7 的递增： ${incrementer(7)}"

```
在adder上通过柯里化把第一个参数固定为1，这就是我们的函数工厂，它会为我们产出一个单参数的函数。

```
//例6-14 递归式的筛选函数，Scala实现
object CurryTest extends App {
  def filter(xs: List[Int], p: Int => Boolean): List[Int] = 
    if (xs.isEmpty) xs
    else if (p(xs.head)) xs.head :: filter(xs.tail, p)
    else filter(xs.tail, p)
    
  def dividesBy(n: Int)(x: Int) = ((x % n) == 0)   //①
  
  val nums = List(1, 2, 3, 4, 5, 6, 7, 8)
  println(filter(nums, dividesBy(2)))           //②
  println(filter(nums, dividesBy(3)))
}
```
- ①定义时已经指明函数将被柯里化使用
- ②filter要求传入一个集合（nums）和一个单参数的函数（柯里化之后dividesBy()函数就变成单参数了）。

在拘谨的模式眼光看来，例6-14“不经意”地柯里化了dividesBy()方法，有些不可思议。dividesBy()本来接受两个参数，并根据第二个参数能否被第一个参数整除来返回true或false。
然而在它参与到filter()操作的时候，我们只为它提供了一个参数，以柯里化的方式来制造一个单参数的函数去充当filter()方法的谓词。

模式在函数式模式编程下有三种归宿，这个例子同时反映了其中的两种。首先，柯里化已经内建在语言或运行时里面，函数式工厂的概念天然存在，不需要我们添加额外的结构。
其次，柯里化带给我们一种全新的实现途径。恪守Java的程序员怎么都想不到会有例6-14那样的柯里化解法，他们不曾拥有真正的可传递的代码，对于在通用函数上构造专用函数的思路更为陌生。
甚至很可能，大多数习惯于命令式编程的开发者根本没想过在这种地方使用设计模式，毕竟在通用版本的基础上构造一个特化的dividesBy()方法，看起来只是一个小问题，而设计模式是为了更大型的问题准备的，因为主要依赖结构来解决问题的设计模式有着很重的实现负担。
柯里化交给我们的答案如此轻巧，都不值得像模式那样专门为解答方案起一个名字，因为柯里化只是在做它的本职工作而已。

*柯里化可以把通用的函数改造成专用的函数。*

### 6.3 结构化重用和函数式重用的对比

> 面向对象编程通过封装不确定因素来使代码能被人理解；函数式编程通过尽量减少不确定因素来使代码能被人理解。     ——Michael Feathers

简化状态的封装和使用，是面向对象的目标之一。自然地，状态就成了面向对象的抽象用来解决问题的常规武器，维系状态所需要的众多类和交互因此被派生出来——这些正是“不确定因素”。

函数式编程不喜欢把结构耦合在一起，它依靠零件之间的复合来组织抽象，以达到减少不确定因素的目的。

### 以结构为载体的代码重用

命令式的、（尤其是）面向对象的编程风格，使用结构和消息作为建筑材料。如果一段面向对象的代码值得重用，那么我们会把它提取到另一个类中，然后通过继承来访问它。

```java
//例6-15  命令式的完美数分类实现
public class ClassifierAlpla {
  private int number;
  
  public ClassifierAlpla(int number) {
     this.number = number;
  }
  
  public boolean isFactor(int potential_factor) {
     return number % potential_factor == 0;
  }
  
  public Set<Integer> factors() {
    HashSet<Integer> factors = new HashSet<>();
    for(int i = 1; i <= sqrt(number); i ++){
      if(isFactor(i)){
         factors.add(i);
         factors.add(number / i);
      }
    }
    return factors;
  }
  
  static public int sum(Set<Integer> factors) {
    Iterator it = factors.iterator();
    int sum = 0;
    while(it.hasNext())
      sum += (Integer)it.next();
    return sum;
  }
  
  public boolean isPerfect() {
    return sum(factors()) - number == number;
  }
  
  public boolean isAbundant() {
    return sum(factors()) - number > number;
  }
  
  public boolean isDeficient() {
    return sum(factors()) - number < number;
  }
}
```

```java

//例6-16 命令式的素数判定实现
public class PrimeAlpha{
   private int number;
   
   public PrimeAlpha(int number) {
     this.number = number;
   }
   
   public boolean isPrime() {
     Set<Integer> primeSet = new HashSet<Integer>(){{
       add(1);
       add(number);
     }};
     return number > 1 && factors().equals(primeSet);
   }
   
   public boolean isFactor(int potential_factor) {
     return number % potential_factor == 0;
   }
   
   public Set<Integer> factors() {
     HashSet<Integer> factors = new HashSet<>();
     for(int i = 1; i <= sqrt(number); i++) {
       if(isFactor(i)){
         factors.add(i);
         factors.add(number / i);
       }
     }
     return factors;
   }
}
```

例6-16有几个值得注意的地方。第一个是isPrime()方法中有点奇特的初始化代码。我们使用的语法结构叫作*实例初始化块（instance initializer）*，是Java的一种构造技巧。
这个代码块放在类里面，但又不属于任何方法，它是Java在构造器外提供的另一种控制实例创建的手段。

#### 1.用重构来消灭重复

功能重复的问题可以用重构来解决，我们把重复的部分单独提取到新的Factors类。

```java
//例6-17 经过提取、重构的共通代码
public class FactorsBeta {
  protected int number;
  
  public FactorsBeta(int number) {
    this.number = number;
  }
  
  public boolean isFactor(int potential_factor) {
    return number % potential_factor == 0;
  }
  
  public Set<Integer> getFactors() {
    HashSet<Integer> factors = new HashSet<>();
    for(int i = 1; i <= sqrt(number); i++) {
      if (isFactor(i)) {
        factors.add(i);
        factors.add(number / i);
      }
    }
    return factors;
  }
}
```

由于被提取的两个方法都使用了成员变量number，因此number也一起搬到了超类。执行重构的时候，IDE会询问我们怎样处理number的访问（生成读写方法，设定protected前缀等）。

```java
//例6-18 重构之后简化了完美数分类的程序
public class ClassifierBeta extends FactorsBeta {
  
  public ClassifierBeta(int number) {
    super(number);
  }
  
  public int sum() {
    Iterator it = getFactors().iterator();
    int sum = 0;
    while(it.hasNext())
      sum += (Integer) it.next();
    return sum;
  }
  
  public boolean isPerfect() {
    return sum() - number == number;
  }
  
  public boolean isAbundant() {
    return sum() - number > number;
  }
  
  public boolean isDeficient() {
    return sum() - number < number;
  }
}
```


```java
//例6-19 重构之后简化了的素数判定程序
public class PrimeBeta extends FactorsBeta {
  public PrimeBeta(int number) {
    super(number);
  }
  
  public boolean isPrime() {
    Set<Integer> primeSet = new HashSet<Integer>(){{
       add(1);
       add(number);
    }};
    return getFactors().equals(primeSet);
  }
}
```

无论我们重构时怎样安排number字段的访问选项，在决策的时候都不能只考虑当前的类，而要把关系网里所有的类都考虑进来。
很多时候这种思考是有益的，因为可以帮助我们划定问题的边界。坏处是父类中的修改会影响下游的类。

这是一种通过耦合来实现的代码重用：两个代码单元（ClassifierBeta类和PrimeBeta类）因为共享了父类的number字段和getFactors()方法，而被绑在了一起。
语言本身的耦合规则促成了这种用法。面向对象规定了耦合式的交互风格（例如规定我们通过继承来获得对成员变量的访问权），对于各种事物如何耦合我们有一套预定的规则——这是好事，因为可以一致地推理各种情况下的行为。
继承时很有用的特性，只是面向对象语言滥用了继承，有时候别的一些抽象更符合需要。

#### 2.以复合的方式实现的重用

```java
//例6-20  稍微向函数式靠拢的完美数分类实现
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

```java
//例6-21 函数式版本的素数判定程序
public class FPrime {
  public static boolean isPrime(int number) {
    Set<Integer> factors = Factors.of(number);
    return number > 1 && 
         factors.size() == 2 &&
         factors.contains(1) &&
         factors.contains(number);
  }
}
```

我们像前面那样，把重复代码提取出来，放进单独的Factors类。再把factors()方法改名为of()，以提高代码的可读性。

```java
//例6-22  经过函数式重构发Factors类
public class Factors {
  static public boolean isFactor(int number, int potential_factor) {
    return number % potential_factor == 0;
  }
  
  static public Set<Integer> of(int number) {
    HashSet<Integer> factors = new HashSet<>();
    for(int i = 1; i <= sqrt(number); i++){
      if(isFactor(number, i)){
        factors.add(i);
        factors.add(number / i);
      }    
    }
    return factors;
  }
}
```

因为函数式版本的实现通过参数传递所有的状态，我们提取出来的重用代码也就不包含任何共享的状态。我们接下来就可以在Factors类的基础上重构完美数分类和素数查找程序了。

```java
//例6-23  经过重构的完美数分类程序
public class FClassifier {
  
  public static int sumOfFactors(int number) {
    Iterator<Integer> it = Factors.of(number).iterator();
    int sum = 0;
    while(it.hasNext()) {
      sum += it.next();
    }
    return sum;
  }
  
  public static boolean isPerfect(int number) {
    return sumOfFactors(number) - number == number;
  }
  
  public static boolean isAbundant(int number) {
    return sumOfFactors(number) - number > number;
  }
  
  public static boolean isDeficient(int number) {
    return sumOfFactors(number) - number < number;
  }  
}
```

我们并没有借助任何特殊的类或语言来提高最后这一版实现的函数式程度。我们通过复合（composition）而不是耦合（coupling）来达到代码重用的目的。

耦合与复合的差别是微妙的，但也是重要的。在需要重构大量代码的时候，我们就会发现处处都会遇到耦合，因为耦合是面向对象语言非常基本的重用机制。
盘根错节、难以理解的耦合关系已经妨害了面向对象语言下的代码重用，只在一些非常成熟的书领域，如对象关系映射、图形界面组件库等少数方面，才取得了令人满意的效果。
而在那些面向对象特征不那么明显的领域（例如各种业务应用），高水平的重用只是美好的愿望。

像函数式程序员一样思考，意味着要用新的视角去看待编程中的一切方面。代码重用是不同编程范式的共同追求，但函数式程序员在这个问题上有着不同于命令式抽象的解决途径。


## 第7章  现实应用

### 7.1 Java 8

设计Java 8的工程师们很聪明，他们没有生硬地在语言中安插高阶函数，而是巧妙地让旧的接口也享受到函数的巧妙。

Java 8的stream上可以串联组合一连串的函数，直到我们调用一个产生输出的函数（成为*终结函数*），如collect()、forEach()来终结这种串联。

Java 8除了增加函数式的特性，还增加了一些配合使用的语法糖衣。例如`filter(name -> name.length() > 1)`，完整的写法其实应该是`filter((name) -> name.length() > 1)`。
因为被传递的lambda块只有一个参数，所以允许省略意义不大的括号。stream操作有时还可以吧返回类型“隐藏”起来。

filter()方法要求传入的参数类型Predicate<T>，其实就是一个返回Boolean值的函数。

```java
//例7-2  手工创建一个谓词实例
Predicate<String> p = (name) -> name.startWith("Mr");
List<String> l = List.of("Mr Rogers", "Mr Robinson", "Mr Ed");
l.stream().filter(p).forEach(i -> System.out.println(i));
```

例7-2通过赋值来创建谓词的实例，被赋值给谓词变量的是作为筛选条件的lambda块。最后在第三行调用filter()方法的时候，把谓词实例当作参数传了进去。

collect()执行的是我们熟悉的化约操作：将集合中的元素结合成（一般）比较少的结果值，甚至单一值（如求和操作）。
Java 8有专门的reduce()方法，但这里的collect()更合适，因为它处理值可变的容器（如StringBuilder）的效率更高。

#### 7.1.1 函数式接口

含有单一方法的接口是Java的一种习惯用法，称为SAM(Single Abstract Method, 单抽象方法)接口，Runnable和Callable接口都是有代表性的例子。
很多时候，SAM接口主要被当成一种将代码传递到异地执行的机制来使用。现在Java 8有了lambda块这种更好的传递代码的媒介。
而我们用过一种叫作“函数式接口”的巧妙机制，可以让lambda和SAM携起手来。
函数式接口是对旧有SAM接口的增强，它允许我们用lambda块取代传统的匿名类来就地实例化一个接口。
例如Runnable接口就被加上了@FunctionalInterface标注，于是，编译器知道并验证Runnable确实是一个接口（而非class或enum），并且符合函数式接口的要求。

取代Runnable匿名内部类的Java 8新语法，下面的代码通过传递lambda块来创建了新的线程：

```java
new Thread(() -> System.out.println("Inside thread")).start();
```



Java 8还允许我们在接口上声明*默认方法*。“默认方法”是在一些接口类型中声明的，以default关键字标记的，非抽象、非静态类的public方法（且带有方法体定义）。
默认方法会被自动地添加到实现了接口的类中，这就为我们提供了一条在类上“装饰”默认功能的方便途径。善用这种特性的例子，如Comparator接口，其默认方法多达十余个。

```java
例7-3  Comparator接口中的方法
List<Integer> n = List.of(1, 4, 45, 12, 5, 6, 9, 101);
Comparator<Integer> c1 = (x, y) -> x - y;
Comparator<Integer> c2 = c1.reversed();
System.out.println("Smallest = " + n.stream().min(c1).get());
System.out.println("Largest = " + n.stream().min(c2).get());
```

例7-3给lambda块套上了一层外皮，创建出一个Comparator实例。然后，我们只需要调用默认方法reversed()，就得到了一个颠倒了方向的比较器实例。
在种在接口上附着默认方法的能力，可以认为是对“mixin”特性的一种模仿，也是对Java语言有益的补充。

有些早期的面向对象的语言规定，类的属性和方法都集中在同一处，有单个的代码块来构成完整的类定义。
而另外一种语言则允许开发者先定义属性，以后再择机完成方法的定义，然后“混入”类中。
随着面向对象语言的演化，mixin特性在现代语言中的实现原理和细节也发生了变化。

Ruby、Groovy等类似语言也允许通过mixin的形式，在既有的类层次上增补功能。这些语言中的mixin是介于接口和父类之间的一种结构。
它和接口一样都是类型，都可执行instanceof检查，也都遵循一样的扩展规则。同一个类上可以混入不限数目的mixin。
但是有一点和接口不一样，mixin除了规定方法的签名，还可以实现该签名对应的行为。
Java 8的默认方法向Java语言引入了mixin机制，从此JDK不需要在单纯为了给静态方法找个归属，而生硬地设置Arrays、Collections这样的类了。

#### 7.1.2 Optional类型

Optional防止方法的返回结果出现无法区分表示错误的null和作为有效结果的null的情况。
Java 8还提供了ifPresent()方法，可以用在终结操作的位置上，设定在仅当存在有效结果时执行的一个代码块。
例如下面的例子仅在有返回值的时候打印出结果：

```
n.stream()
    .min((x,y) -> x - y)
    .ifPresent(z -> System.out.println("smallest is " + z));
```

如果我们还想处理其他情况，可以求助于功能类似的orElse()方法。

#### 7.1.3 Java 8的stream

stream在很多方面的行为都与几何相似，但有一些关键点区别。

- stream不存储值，只担当从输入源引出的管道角色，一直连接到终结操作上产生输出。
- stream从设计上就偏向函数式风格，避免与状态发生关联。如filter()操作在返回筛选结果的stream时，并不会改动底下的集合。
- stream上的操作尽可能做到缓求值。
- stream可以没有边界（无限长）。例如我们可以构造一个返回所有数字的stream，然后用limit()、findFirst()等方法来取得其一部分子集。
- stream很像Iterator的实例一样，也是消耗品，用过之后必须重新生成新的stream才能再次操作。

stream的操作部分为*中间操作*和*终结操作*。中间操作一律返回新的stream，并且总是缓求值的。
例如在stream上调用filter()操作，并不会真的在stream上进行筛选。而是产生一个新的stream，让它只为终结操作的遍历过程提供满足筛选条件的值。
终结操作遍历stream，产生结果值和副作用（如果我们让函数产生副作用的话；虽然不鼓励，但是允许）。

### 7.2 函数式的基础设施

#### 7.2.1 架构

函数式的架构从根本上贯彻“值不可变”的思路，最大化地发挥其优点。学会值不可变的角度去思考，是我们掌握函数式程序员的思维方法的一条重要门径。

值不可变的类可以摆脱Java开发中一大批典型的负累。在函数式的思维下，我们会意识到，测试是为了确认代码中成功地制造了我们需要的变化。
换言之，测试的真正目的是对可变事物的检验——可变的事物越多，就需要越多的测试来保证其正确性。

**可变的状态与测试数量有直接的关联：可变的状态越多，要求的测试也越多。**

如果我们严格限制可变性，只允许在规定的地方制造变化，那么就大大缩小了可能出错的位置范围，需要的测试也随之减少。对于值不可变的类来说，由于变化只发生在构造期间，它的单元测试也就变得十分轻松了。
并且我们不再需要设置复制构造器（copy constructor），也不再需要折腾clone()方法棘手的实现细节。
值不可变的对象很适合充当map和set数据结构中的键；Java不允许字典型集合中的键在它被集合引用期间发生取值的变化，值不可变的对象完全符合这项要求。

值不可变的对象天生就是线程安全的，完全不会发生同步方面的问题。它们还绝对不会由于异常而处于不明确的、预料之外的状态。因为所有的初始化过程都发生在对象的构造期间，而Java的对象构造具有原子性，
也就是说，议程只会发生在我们得到对象实例之前。Joshua Bloch称这种性质为*具有原子性的失败*（failure atomicity）:只要对象构造完毕，就不会在发生由值可变性引发的失败。

实现一个值不可变的Java类，我们需要做到以下事情。

- 把所有的字段都标记为final。

  Java要求被标记为final的字段，要么在声明时初始化，要么在构造器中初始化。
- 把类标记为final，防止被子类覆盖。

  如果类可以被覆盖，类中的方法也就有可能被改变行为，因此以防万一，我们干脆禁止子类化。Java的String类就采取了这样的防范策略。
- 不要提供无参数的构造器。

  一个值不可变的对象，它的一切状态都必须通过构造器来设定。假如我们没有需要设定的状态，那建立这么一个对象又有何必要呢？在无状态的类里面安排几个静态方法就足够了。
  所以说，值不可变的类根本不应该出现无参数的构造器。JavaBeans的标准规定要有默认构造器，我们摒除无参数构造器违反了这条规定。不过反正JavaBeans里有各种setXXX方法，本身就不可能是值不可变的。
- 提供至少一个构造器。

  构造器是我们在对象里添置状态的最后机会！
- 除了构造器之外，不要提供任何制造变化的方法。

  我们不但要避免沿袭JavaBeans风格的setXXX的方法，还必须小心防范，不能返回任何值可变的对象引用。标记了final的对象引用并不等于它所指向的一切都不可变。因此，我们需要预防性地复制所有通过getXXX方法返回的对象引用。
  
Groovy用语法糖衣掩盖了实现值不可变的繁琐细节：
```groovy
//例7-4 值不可变的client类
@Immutable
class Client {
   String name, city, state, zip
   String[] streets
}
```

添加@Immutable标注使得这类自动获得以下特性：

- 它成为一个final类。
- 自动为属性生成相应的私有字段和get方法。
- 任何企图修改属性值的操作都会得到ReadOnlyPropertyException。
- Groovy会生成普通的和基于map的两种构造器。
- 集合类的实例会被加上适当的包装，数组（以及其他可被复制的对象）会被复制。
- 自动生成默认的equals()、hashcode()和toString()方法。

@Immutable标注生动地阐释了本书屡屡强调的要旨：把实现细节托付给运行时。

/++++++++++++++++++++++++++++++++++++++++++++++++++++/

最终一致性

分布式计算中的最终一致性（eventual consistency）模型，不对模型的变更操作施加硬性的时间限制，而只是保证，当更新发生后，模型最终会回复到一致的状态。

事务要求系统满足ACID（即原子性Atomic、一致性Consistent、隔离性Isolated、持久性Durable）性质，而最终一致性要求满足BASE(即基本可用Basically Available、软状态Soft state、最终一致性Eventual consistency)性质。

/++++++++++++++++++++++++++++++++++++++++++++++++++++/

#### 7.2.2 Web 框架

web领域与函数式编程简直是天作之合：我们可以把整个web看作是一系列请求到响应的变换。每一种函数语言下都有非常丰富的，或热门或冷门的web框架可供选择。这些问问框架大多具备以下共同特性。

- 路由框架

  多数现代的Web应用框架，包括函数式的Web框架在内，都从应用的主体功能中剥离路由的相关细节，将之交托给专门的路由功能库。路由信息通常放置在一个能够被路由库理解和遍历的标准数据结构里（例如存放在另一个Vector结构中的Vector结构）。
  
- 以函数作为路由的目标

  很多函数式的Web应用用函数来充当路由的目的地。Web请求很容易被理解成一个接受Request，返回Response的函数；有一部分函数式Web框架提供了语法糖衣来简化这方面的基本操作。
  
- 领域专用语言（DSL）

  Martin Fowler将DSL定义为表达能力有限，专门针对一个狭窄问题域的计算机编程语言。其中内部DSL使用较为广泛。内部DSL是在其宿主语言之上构造出来的新“语言”，且利用宿主语言的语法糖衣来形成自身的风格。
  大多数现代Web框架会在路由、HTML内嵌元素、数据库交互等几个地方使用DSL。DSL在各类语言中都是常见的编程手段，但函数式语言尤其偏好描述性的代码风格，这一点恰好也常常是DSL的目的。
  
- 与构建工具紧密集成

  函数式Web框架通常不和IDE栓在一起，反倒和命令行的构建工具紧密集成，用构建工具来执行从生成新项目骨架到运行测试的一切任务。IDE和编辑器可以围绕现有的工具链来实现自动化，但构建工具和构建产物之间的耦合关系太过紧密，很难拆解之后重新安置。
  
函数式语言下的Web开发会有一些变得更容易的部分，也会有变得更困难的一部分。总的来说，用不可变的值来编程，会降低测试的负担，因为需要测试验证的状态变化减少了。一旦我们在架构中树立起（值不可变性等）惯例，各部分的配个将更加顺畅。


#### 7.2.3 数据库

Datomic是一种值不可变的数据库，进入到库里的每一笔事实都会被打上时间戳。由于Datomic存储*值*而非*数据*，它的空间利用率并不低。
当一个值被输入到系统之后，所有需要使用该值的地方都可以指向原始的实例（而该原始实例永远不会变更，因为它是值不可变的），这样就提高了空间的利用率。
如果未来需要向应用程序呈现更新的数据，我们只要指向另一个值就可以了。Datomic在信息上增加了时间的概念，使得每一笔事实都总是维持在正确的上下文里。

Datomic的设计产生了一些有意思的结果。

- 永久地记录所有的schema变更和数据变更

  由于保留了一切事物的记录（包括schema的变动），数据库可以轻易地回退到旧的版本。而关系型数据库为了解决这个问题而发展出了一个完整的工具门类（数据库迁移工具）。
  
- 读取和写入分离

  Datomic的架构天然地分离了读取和写入操作，也就是说绝不会发生因为查询而推迟更新的情况。因此从架构上说，Datomic拥有一个CQRS系统的存在。
  
- 事件驱动型架构中的值不可变性和时间戳

  事件驱动的架构依靠一个事件流来反映应用程序的状态变化，而捕获所有信息并加上时间戳的数据库，正好可以完美地扮演事件流的角色，数据库本身的特性即可满足回退和重放事件的需求。


## 第8章  多语言与多范式




