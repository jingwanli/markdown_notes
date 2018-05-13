# Spark中的combineByKey

在数据分析中，处理Key，Value的Pair数据是极为常见的场景。譬如说，对Pair数据按照key分组、聚合，又或者更抽象的，则是根据key对value进行fold运算。

如果我们对编码的态度有些敷衍，大约会将其分别定义为三个函数：gruopByKey、aggregateByKey、foldByKey。站在调用者的角度，如此设计无可厚非，相反我还得击节赞叹。因为从函数名来看，确实体贴地照顾了用户的知识结构。换个角度，站在实现这一边，你可能会发现这三个函数乃孪生兄弟，具有相同的血统。

**所谓“抽象”，就是要寻找实现上的共同特征**。OO也好，FP也罢，都格外重视抽象的能力，因为抽象能在很大程度上化繁为简，且具备应对变化的能力。只是各自抽象的层次不同罢了。与OO中抽象接口的设计思路是等同的，个人认为，FP只是将抽象做到了极致，落实到了类型之上（暂且放开业务的羁绊），函数不过就是类型的转换罢了。

在Spark的语境中，RDD是核心数据结构，从Scala的语法出发，可以认为RDD是一个类型类，或者Monad容器（个人如此认为，若有不妥，还请方家指正）。这个容器到底装了什么数据呢？得看你如何为其装载数据，前面提到了Pair类型，在本文场景下，RDD实则是一个RDD[(K, V)]类型的数据。

“前戏”到此结束，现在来看看groupByKey、aggregateByKey和foldByKey到底要做什么（what to do）？

groupByKey是将一堆结构形如(K, V)的数据**根据K分组**，我们暂且将这个分组过程看做是一个黑盒子，想一想，**输出会是什么**？

aggregateByKey是将一堆结构形如（K, V)的数据**根据K**对数据进行**聚合**运算，**它的输出又会是什么呢**？

fold是针对一个集合中的数据进行**折叠**运算，运算符则取决传入的函数，例如sum或者produce等。因而foldByKey就是将一堆结构形如（K, V)的数据**根据K**对数据进行折叠运算，那么，**它的输出又会是什么**？

现在需要一点推演的能力。首先，不管过程如何，这三个运算接收的参数总是相同的，皆为(K, V)。输出结果定然不同，但它们却又具有相同的特征，即它们都是**根据K**分别对数据进行运算，换言之，计算可能不同，计算的结果可能不同，但结果一定是**根据K**来组织的。这里的K不就是Pair的key吗？对应的Value呢？我们还不知道，然而不管是阿猫阿狗，它总可以用一个抽象的类型参数来代表，记为C，结果就变成了(K, C)。这就是我们要找的一个抽象：

```
RDD[(K, V)] -> RDD[(K, C)]
```

然而，这个抽象还不够，我们需要找到将V（可能是多个V）变成C的方法。

现在假设我们要寻找的抽象是一台超级酷的果汁机。它能同时接受各种各样的水果，然后聪明地按照水果的种类分别榨出不同的果汁。苹果归苹果汁，橙子归橙汁，西瓜归西瓜汁。我们为水果定义类型为Fruit，果汁定义为Juice，根据前面的分析，这个过程就是将RDD[(String, Fruit)]转换为RDD[(String, Juice)]。

注意，在榨果汁前，水果可能有很多，即使是相同类型的水果，也会作为不同的RDD元素：

```
("apple", apple1), ("orange", orange1), ("apple", apple2)
```

转换的结果是每种水果只有一杯果汁（只是容量不同罢了）:

```
("apple", appleJuice), ("orange", orangeJuice)
```

那么，这个果汁机该由什么元件构成呢？现在，我们化身为机械师，想想这个果汁机的组成元件：

- 首先，它需要一个元件提供将各种水果榨为各种果汁的功能；
- 其次，它需要提供将果汁进行混合的功能；
- 最后，为了避免混合错误，还得提供能够根据水果类型进行混合的功能。

注意第二个函数和第三个函数的区别，前者只提供混合功能，即能够将不同容器的果汁装到一个容器中，而后者的输入已有一个前提，那就是已经按照水果类型放到不同的区域，果汁机在混合果汁时，并不会混淆不同区域的果汁。否则你得到的果汁就不是苹果汁或者橙汁，而是混合味儿的果汁。

再回到函数这边来。从函数的抽象层面看，这些操作具有共同的特征，都是将类型为RDD[(K,V)]的数据处理为RDD[(K,C)]。这里的V和C可以是相同类型，也可以是不同类型。这种操作并非单纯地对Pair的value进行map，而是针对不同的key值对原有的value进行联合（Combine）。因而，不仅类型可能不同，元素个数也可能不同。

于是，我们百折千回地寻找到了这个高度抽象的操作combineByKey。该方法在Spark的定义如下所示：

```
  def combineByKey[C](
      createCombiner: V => C,
      mergeValue: (C, V) => C,
      mergeCombiners: (C, C) => C,
      partitioner: Partitioner,
      mapSideCombine: Boolean = true,
      serializer: Serializer = null): RDD[(K, C)] = {
    //实现略
  }
```

声明式风格与命令式风格不同之处在于它说明了代码做了什么（what to do），而不是怎么做(how to do)。combineByKey函数主要接受了三个函数作为参数，分别为createCombiner、mergeValue、mergeCombiners。这三个函数足以说明它究竟做了什么。理解了这三个函数，就可以很好地理解combineByKey。

要将RDD[(K,V)]combine为RDD[(K,C)]，就需要提供一个函数，能够完成从V到C的combine，称之为combiner。如果V和C类型一致，则函数为V => V。倘若C是一个集合，例如Iterable[V]，则createCombiner为V => Iterable[V]。

mergeValue则将原RDD中Pair的Value合并为操作后的C类型数据。合并操作的实现决定了结果的运算方式。所以，mergeValue更像是声明了一种合并方式，它是由整个combine运算的结果来导向的。函数的输入为原RDD中Pair的V，输出为结果RDD中Pair的C。

最后的mergeCombiners则会根据每个Key对应的多个C，进行归并。

再回到果汁机的案例。果汁机的功能类似于groupByKey+foldByKey操作。但我们没有必要自己去实现这个榨取果汁的功能，可以直接调用combineByKey函数：

```
case class Juice(volumn: Int) {
    def add(j: Juice):Juice = Juice(volumn + j.volumn)
}
case class Fruit(kind: String, weight: Int) {
    def makeJuice:Juice = Juice(weight * 100)
}
val apple1 = Fruit("apple", 5)
val apple2 = Fruit("apple", 8)
val orange1 = Fruit("orange", 10)
     
val fruit = sc.parallelize(List(("apple", apple1) , ("orange", orange1) , ("apple", apple2))) 
val juice = fruit.combineByKey(
    f => f.makeJuice,
    (j:Juice, f) => j.add(f.makeJuice),
    (j1:Juice, j2:Juice) => j1.add(j2) 
)
```

执行juice.collect，结果为：

```
Array[(String, Juice)] = Array((orange, Juice(1000)), (apple, Juice(1300)))
```

RDD中有许多针对Pair RDD的操作在内部实现都调用了combineByKey函数（Spark的2.0版本则实现为combineByKeyWithClassTag）。例如groupByKey：

```
class PairRDDFunctions[K, V](self: RDD[(K, V)])
    (implicit kt: ClassTag[K], vt: ClassTag[V], ord: Ordering[K] = null)
  extends Logging
  with SparkHadoopMapReduceUtil
  with Serializable {
    def groupByKey(partitioner: Partitioner): RDD[(K, Iterable[V])] = {
        val createCombiner = (v: V) => CompactBuffer(v)
        val mergeValue = (buf: CompactBuffer[V], v: V) => buf += v
        val mergeCombiners = (c1: CompactBuffer[V], c2: CompactBuffer[V]) => c1 ++= c2
        val bufs = combineByKey[CompactBuffer[V]](
          createCombiner, mergeValue, mergeCombiners, partitioner, mapSideCombine=false)
        bufs.asInstanceOf[RDD[(K, Iterable[V])]]
      }
}
```

groupByKey函数针对PairRddFunctions的RDD[(K, V)]按照key对value进行分组。它在内部调用了combineByKey函数，传入的三个函数分别承担了如下职责：

- createCombiner是将原RDD中的K类型转换为Iterable[V]类型，实现为CompactBuffer。
- mergeValue实则就是将原RDD的元素追加到CompactBuffer中，即将追加操作(+=)视为合并操作。
- mergeCombiners则负责针对每个key值所对应的Iterable[V]，提供合并功能。

再例如，我们要针对科目对成绩求平均值：

```
val scores = sc.parallelize(List(("chinese", 88.0) , ("chinese", 90.5) , ("math", 60.0), ("math", 87.0)))
```

平均值并不能一次获得，而是需要求得各个科目的总分以及科目的数量。因此，我们需要针对scores进行combine，从(String, Float)combine为(String, (Float, Int))。在调用combineByKey函数后，再通过map来获得平均值。代码如下：

```
val avg = scores.combineByKey(
    (v) => (v, 1),
    (acc: (Float, Int), v) => (acc._1 + v, acc._2 + 1),
    (acc1:(Float, Int), acc2:(Float, Int)) => (acc1._1 + acc2._1, acc1._2 + acc2._2)
).map{ case (key, value) => (key, value._1 / value._2.toFloat) }
```

除了可以进行group、average之外，根据传入的函数实现不同，我们还可以利用combineByKey完成诸如aggregate、fold等操作。这是一个高度的抽象，但从声明的角度来看，却又不需要了解过多的实现细节。这正是函数式编程的魅力。