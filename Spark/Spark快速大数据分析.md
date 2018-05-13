[TOC]

## 第1章 Spark数据分析导论

### Spark是什么

Spark的一个主要特点就是能够在内存中进行计算.

Spark 的核心是一个对由很多计算任务组成的、运行在多个工作机器或者是一个计算集群上的应用进行调度、分发以及监控的计算引擎。

各组件间密切结合的设计原则的优点:

1. 软件栈中所有的程序库和高级组件都可以从下层的改进中获益
	. 运行整个软件栈的代价变小了,	不需要运行 5 到 10 套独立的软件系统了，一个机构只需要运行一套软件系统即可.这就把原先尝试一种新的数据分析系统所需要的下载、部署并学习一个新的软件项目的代价简化成了只需要升级 Spark。
3. 我们能够构建出无缝整合不同处理模型的应用

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-19 上午11.52.28.png)

### Spark Core

Spark Core 实现了 Spark 的基本功能，包含任务调度、内存管理、错误恢复、与存储系统交互等模块。Spark Core 中还包含了对弹性分布式数据集(resilient distributed dataset，简称 RDD)的 API 定义。RDD 表示分布在多个计算节点上可以并行操作的元素集合，是Spark 主要的编程抽象。Spark Core 提供了创建和操作这些集合的多个 API。		

### Spark SQL

Spark SQL 是 Spark 用来操作结构化数据的程序包。通过 Spark SQL，我们可以使用 SQL
或者 Apache Hive 版本的 SQL 方言(HQL)来查询数据。Spark SQL 支持多种数据源，比如 Hive 表、Parquet 以及 JSON 等。除了为 Spark 提供了一个 SQL 接口，Spark SQL 还支持开发者将 SQL 和传统的 RDD 编程的数据操作方式相结合，不论是使用 Python、Java 还是 Scala，开发者都可以在单个的应用中同时使用 SQL 和复杂的数据分析。通过与 Spark所提供的丰富的计算环境进行如此紧密的结合，Spark SQL 得以从其他开源数据仓库工具中脱颖而出。Spark SQL 是在 Spark 1.0 中被引入的。

### Spark Streaming

Spark Streaming 是 Spark 提供的对实时数据进行流式计算的组件。比如生产环境中的网页服务器日志，或是网络服务中用户提交的状态更新组成的消息队列，都是数据流。Spark Streaming 提供了用来操作数据流的 API，并且与 Spark Core 中的 RDD API 高度对应。这样一来，程序员编写应用时的学习门槛就得以降低，不论是操作内存或硬盘中的数据，还是操作实时数据流，程序员都更能应对自如。从底层设计来看，Spark Streaming 支持与Spark Core 同级别的容错性、吞吐量以及可伸缩性。

### MLlib

Spark 中还包含一个提供常见的机器学习(ML)功能的程序库，叫作 MLlib。MLlib 提供了很多种机器学习算法，包括分类、回归、聚类、协同过滤等，还提供了模型评估、数据导入等额外的支持功能。MLlib 还提供了一些更底层的机器学习原语，包括一个通用的梯度下降优化算法。所有这些方法都被设计为可以在集群上轻松伸缩的架构。

### GraphX

GraphX 是用来操作图(比如社交网络的朋友关系图)的程序库，可以进行并行的图计算。与 Spark Streaming 和 Spark SQL 类似，GraphX 也扩展了 Spark 的 RDD API，能用来创建一个顶点和边都包含任意属性的有向图。GraphX 还支持针对图的各种操作(比如进行图分割的 subgraph 和操作所有顶点的 mapVertices)，以及一些常用图算法(比如 PageRank和三角计数)。

### 集群管理器

就底层而言，Spark 设计为可以高效地在一个计算节点到数千个计算节点之间伸缩计算。为了实现这样的要求，同时获得最大灵活性，Spark 支持在各种集群管理器(cluster manager)上运行，包括 Hadoop YARN、Apache Mesos，以及 Spark 自带的一个简易调度器，叫作独立调度器。如果要在没有预装任何集群管理器的机器上安装 Spark，那么 Spark自带的独立调度器可以让你轻松入门;而如果已经有了一个装有 Hadoop YARN 或 Mesos的集群，通过 Spark 对这些集群管理器的支持，你的应用也同样能运行在这些集群上。

## 第2章 Spark下载与入门

```scala
scala> val lines = sc.textFile("README.md") // 创建一个名为lines的RDD
      lines: spark.RDD[String] = MappedRDD[...]
scala> lines.count() // 统计RDD中的元素个数 res0: Long = 127
scala> lines.first() // 这个RDD中的第一个元素，也就是README.md的第一行 res1: String = # Apache Spark
```

### Spark核心概念简介

​	从上层来看，每个 Spark 应用都由一个驱动器程序(driver program)来发起集群上的各种并行操作。驱动器程序包含应用的 main 函数，并且定义了集群上的分布式数据集，还对这些分布式数据集应用了相关操作。在前面的例子里，实际的驱动器程序就是 Spark shell 本身，你只需要输入想要运行的操作就可以了。

​	驱动器程序通过一个 SparkContext 对象来访问 Spark。这个对象代表对计算集群的一个连接。shell 启动时已经自动创建了一个 SparkContext 对象，是一个叫作 sc 的变量。		
	一旦有了 SparkContext，你就可以用它来创建 RDD。我们可以在这些行上进行各种操作，比如 count()。

​	要执行这些操作，驱动器程序一般要管理多个执行器(executor)节点。比如，如果我们在集群上运行 count() 操作，那么不同的节点会统计文件的不同部分的行数。

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-19 下午1.42.00.png)

​	Spark 会自动将函数(比如 line.contains("Python"))发到各个执行器节点上。这样，你就可以在单一的驱动器程序中编程，并且让代码自动运行在多个节点上.

### 独立应用

​	除了交互式运行之外，Spark 也可以在 Java、Scala 或 Python 的独立程序中被连接使用。这与在 shell 中使用的主要区别在于你需要自行初始化SparkContext。接下来，使用的 API 就一样了。


​	连接 Spark 的过程在各语言中并不一样。在 Java 和 Scala 中，只需要给你的应用添加一个对于 spark-core 工件的 Maven 依赖。Maven 是一个流行的包管理工具，可以用于任何基于 Java 的语言，让你可以连接公共仓库
中的程序库。

#####初始化SparkContext

​	一旦完成了应用与 Spark 的连接，接下来就需要在你的程序中导入 Spark 包并且创建SparkContext。你可以通过先创建一个 SparkConf 对象来配置你的应用，然后基于这个SparkConf 创建一个 SparkContext 对象。

```scala
      import org.apache.spark.SparkConf
      import org.apache.spark.SparkContext
      import org.apache.spark.SparkContext._
      val conf = new SparkConf().setMaster("local").setAppName("My App")
      val sc = new SparkContext(conf)
```

- 集群 URL:告诉 Spark 如何连接到集群上。在这几个例子中我们使用的是 local，这个特殊值可以让 Spark 运行在单机单线程上而无需连接到集群
- 应用名:在例子中我们使用的是 My App。当连接到一个集群时，这个值可以帮助你在 集群管理器的用户界面中找到你的应用

### 构建独立应用

​	在单机上实现单词数统计很容易，但在分布式框架下，由于要在许多工作节点上读入并组合数据，单词数统计就成了一个很常用的例子	

​	下面我们来学习用 sbt 以及 Maven 来构建并打包一个简单的单词数统计的例程。我们可以把所有的例程构建在一起，但是为了展示最简单的构建过程，我们只保留了最基本的依赖。

```scala
// 创建一个Scala版本的Spark Context
val conf = new SparkConf().setAppName("wordCount")
val sc = new SparkContext(conf)
// 读取我们的输入数据
val input = sc.textFile(inputFile)
// 把它切分成一个个单词
val words = input.flatMap(line => line.split(" "))
// 转换为键值对并计数
val counts = words.map(word => (word, 1)).reduceByKey{case (x, y) => x + y} // 将统计出来的单词总数存入一个文本文件，引发求值 counts.saveAsTextFile(outputFile)
```

​	Spark 编程的核心概念:通过一个驱动器程序创建一个 SparkContext 和一系列 RDD，然后进行并行操作.

## 第3章 RDD编程

RDD:弹性分布式数据集(Resilient Distributed Dataset)

RDD 其实就是分布式的元素集合。在 Spark 中，对数据的所有操作不外乎创建 RDD、转化已有 RDD 以及调用 RDD 操作进行求值。而在这一切背后，Spark 会自动将RDD 中的数据分发到集群上，并将操作并行化执行。

### RDD基础

RDD: 不可变的分布式对象集合.

每个RDD被分为多个分区, 这些分区运行在集群中的不同节点上.

RDD 可以包含 Python、Java、Scala 中任意类型的对象，甚至可以包含用户自定义的对象。

> 创建方式

- 读取一个外部数据集
- 在驱动器程序里分发驱动器程序中的对象集合(比如 list 和 set)

> RDD支持的操作 

- 转化操作(transformation) :会生成一个新RDD
- 行动操作(action) : 会对RDD计算出一个结果,并把结果返回到驱动器程序中, 火把结果存储到外部存储系统(如HDFS)中.

转化操作和行动操作的区别在于 Spark 计算 RDD 的方式不同。虽然你可以在任何时候定义新的 RDD，但 Spark 只会惰性计算这些 RDD。它们只有第一次在一个行动操作中用到时，才会真正计算。

事实上，在行动操作 first() 中，Spark 只需要扫描文件直到找到第一个匹配的行为止，而不需要读取整个文件。

默认情况下，Spark 的 RDD 会在你每次对它们进行行动操作时重新计算.

如果想在多个行动操作中重用同一个RDD, 可以使用RDD.persist() 让 Spark 把这个 RDD 缓存下来。


我们可以让 Spark 把数据持久化到许多不同的地方，可用的选项会在表 3-6 中列出。在第一次对持久化的 RDD 计算之后，Spark 会把 RDD 的内容保存到内存中(以分区方式存储到集群中的各机器上)，这样在之后的行动操作中，就可以重用这些数据了。我们也可以把 RDD 缓存到磁盘上而不是内存中。


在实际操作中，你会经常用 persist() 来把数据的一部分读取到内存中，并反复查询这部分数据。


总的来说，每个 Spark 程序或 shell 会话都按如下方式工作。

1. 从外部数据创建出输入 RDD。
2.  使用诸如 filter() 这样的转化操作对 RDD 进行转化，以定义新的 RDD。
3. 告诉 Spark 对需要被重用的中间结果 RDD 执行 persist() 操作。
	. 使用行动操作(例如 count() 和 first() 等)来触发一次并行计算，Spark 会对计算进行优化后再执行。			

### 创建RDD

> 方式1			

把程序中一个已有的集合传给SparkContext的parallelize()方法	

```spark
val lines = sc.parallelize(List("pandas","i like pandas"))
```

不过，需要注意的是，除了开发原型和测试时，这种方式用得并不多，毕竟这种方式需要把你的整个数据集先放在一台机器的内存中。

> 方式2

从外部存储中读取数据来创建RDD.

```spark
val lines = sc.textFile("/path/to/README.md")
```

### RDD操作

转化操作返回的是 RDD，而行动操作返回的是其他的数据类型。

#### 转化操作

> 案例 假定一个日志文件log.txt, 内含若干消息, 希望选出其中的错误信息				

```spark
val inputRDD = sc.textFile("log.txt")
val errorRDD = inputRDD.filter(line=>line.contains("error"))
```

filter()操作不会改变已有的inputRDD中的数据, 实际上, 该操作会返回一个全新的RDD. inputRDD 在后面的程序中还可以继续使用，比如我们还可以从中搜索别的单词。

最后要说的是，通过转化操作，你从已有的 RDD 中派生出新的 RDD，Spark 会使用谱系图(lineage graph)来记录这些不同 RDD 之间的依赖关系。Spark 需要用这些信息来按需计算每个 RDD，也可以依靠谱系图在持久化的 RDD 丢失部分数据时恢复所丢失的数据。

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-19 下午8.30.11.png)

#### 行动操作

行动操作是第二种类型的 RDD 操作，它们会把最终求得的结果返回到驱动器程序，或者写入外部存储系统中。由于行动操作需要生成实际的输出，它们会强制执行那些求值必须用到的 RDD 的转化操作。

```spark
println("Input had " + badLinesRDD.count() + "concerning lines")
println("Here are 10 examples:")
badLinesRDD.take(10).foreach(println)
```

> collect( )函数			

RDD的函数, 用来获取整个RDD中的数据 

RDD 还有一个 collect() 函数，可以用来获取整
个 RDD 中的数据。如果你的程序把 RDD 筛选到一个很小的规模，并且你想在本地处理这些数据时，就可以使用它。记住，只有当你的整个数据集能在单台机器的内存中放得下时，才能使用 collect()，因此，collect() 不能用在大规模数据集上。

collect会把所有数据汇总到Driver的JVM内存中，对Driver压力很大，因此尽量不要执行collect操作；如果只是需要查看内容，可以使用saveAsTextFile，然后到Worker上查看对应文件（如果保存到HDFS，可从HDFS直接下载文件）；如果需要进行统计，建议编写统计代码，统计后将统计结果返回，以减小内存开销。

### 惰性求值

RDD的转化操作都是惰性求值的. 这意味着在被调用行动操作之前Spark不会开始计算.

我们不应该把 RDD 看作存放着特定数据的数据集，而最好把每个 RDD 当作我们通过转化操作构建出来的、记录如何计算数据的指令列表。

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-19 下午11.02.19.png)

而在 Spark 中，写出一个非常复杂的映射并不见得能比使用很多简单的连续操作获得好很多的性能。因此，用户可以用更小的操作来组织他们的程序，这样也使这些操作更容易管理。

### 向Spark传递函数

Spark 的大部分转化操作和一部分行动操作，都需要依赖用户传递的函数来计算。在我们支持的三种主要语言中，向 Spark 传递函数的方式略有区别。

##### Scala

在 Scala 中，我们可以把定义的内联函数、方法的引用或静态方法传递给 Spark，就像Scala 的其他函数式 API 一样。所传递的函数及其引用的数据需要时可以序列化的.

```scala
class SearchFunc(val query:String){
    def isMatch(s:String):Boolean={
        s.contains(query)
    }
  // 问题:"isMatch"表示"this.isMatch"，因此我们要传递整个"this"
   def getMatchesFuntionReference(rdd:RDD[String]):RDD[String])=	{
    	rdd.map(isMatch)
	}
    def getMatchesFieldReference(rdd:RDD[String]):RDD[String]={
   // 问题:"query"表示"this.query"，因此我们要传递整个"this"
        rdd.map(x=>x.split(query))
    }
    def getMatchedNoReference(rdd:RDD[String]):RDD[String] = {
        val query_=this.query
        rdd.map(x=>x.split(query_))
    }

```

### 常见的转化操作和行动操作

### 基本RDD

##### 1. 针对各个各个元素的转化操作

- 单输入, 单输出

  map( ) , filter( )

  map( )的返回值不需要和输入类型一样.

```scala
val input = sc.parallelize(List(1,2,3,4))
val result = input.map(x=>x*x)
println(result.collect().mkString(","))
```

- 单输入多输出

  flatMap( ) : 我们提供给 flatMap() 的函数被分别应用到了输入 RDD 的每个元素上。不过返回的不是一个元素，而是一个返回值序列的迭代器。输出的 RDD 倒不是由迭代器组成的。我们得到的是一个包含各个迭代器可访问的所有元素的 RDD

```scala
val lines = sc.parallelize(List("hello word","hi"))
val words = lines.flatMap(line=>line.split(" "))
words.first()
```

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-20 上午12.23.08.png)

##### 2. 伪集合操作

这些操作都要求操作的 RDD 是相同数据类型的。

我们的 RDD 中最常缺失的集合属性是元素的唯一性，因为常常有重复的元素。如果只要唯一的元素，我们可以使用 RDD.distinct() 转化操作来生成一个只包含不同元素的新RDD。不过需要注意，distinct() 操作的开销很大，因为它需要将所有数据通过网络进行混洗(shuffle)，以确保每个元素都只有一份。

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-20 上午8.58.55.png)

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-20 上午9.02.24.png)

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-20 上午9.52.10.png)

##### 3. 行动操作

你很有可能会用到基本 RDD 上最常见的行动操作 reduce()。它接收一个函数作为参数，这个函数要操作两个 RDD 的元素类型的数据并返回一个同样类型的新元素。


val sum = rdd.reduce((x, y) => x + y)


fold() 和 reduce() 类似，接收一个与 reduce() 接收的函数签名相同的函数，再加上一个“初始值”来作为每个分区第一次调用时的结果。

> fold函数			

```scala
scala> val list = List(1,2,3,4,5)
list: List[Int] = List(1, 2, 3, 4, 5)
 
scala> list.fold(10)(_*_)
res0: Int = 1200
```

fold函数实现了对list中所有元素的累乘操作。fold函数需要两个参数，一个参数是初始种子值，这里是10，另一个参数是用于计算结果的累计函数，这里是累乘。执行list.fold(10)(***)时，首先把初始值拿去和list中的第一个值1做乘法操作，得到累乘值10，然后再拿这个累乘值10去和list中的第2个值2做乘法操作，得到累乘值20，依此类推，一直得到最终的累乘结果1200。

fold有两个变体：foldLeft()和foldRight()，其中，foldLeft()，第一个参数为累计值，集合遍历的方向是从左到右。foldRight()，第二个参数为累计值，集合遍历的方向是从右到左。对于fold()自身而言，遍历的顺序是未定义的，不过，一般都是从左到右遍历。

==注意==:你所提供的初始值应当是你提供的操作的单位元素;也就是说，使用你的函数对这个初始值进行多次计算不会改变结果(例如 +对应的 0，* 对应的 1，或拼接操作对应的空列表)。	

tips:

fold() 和 reduce() 都要求函数的返回值类型需要和我们所操作的 RDD 中的元素类型相同。这很符合像 sum 这种操作的情况。但有时我们确实需要返回一个不同类型的值。例如，在计算平均值时，需要记录遍历过程中的计数以及元素的数量，这就需要我们返回一个二元组。可以先对数据使用 map() 操作，来把元素转为该元素和 1 的二元组，也就是我们所希望的返回类型。这样 reduce() 就可以以二元组的形式进行归约了。

> aggregate( )函数

aggregate() 函数则把我们从返回值类型必须与所操作的 RDD 类型相同的限制中解放出来。与 fold() 类似，使用 aggregate() 时，需要提供我们期待返回的类型的初始值。然后通过一个函数把 RDD 中的元素合并起来放入累加器。考虑到每个节点是在本地进行累加的，最终，还需要提供第二个函数来将累加器两两合并。

```scala
//需求:计算RDD的平均值
val rdd = List(1,2,3,4)
val input = sc.parallelize(rdd)
val result = input.aggregate((0,0))(
(acc,value) => (acc._1 + value, acc._2 + 1),
(acc1,acc2) => (acc1._1 + acc2._1, acc1._2 + acc2._2)
)
result: (Int, Int) = (10, 4)
val avg = result._1 / result._2
avg: Int = 2.5
```

程序的详细过程大概如下：

1. 首先定义一个初始值 (0, 0)，即我们期待的返回类型的初始值。
2. **(acc,value) => (acc._1 + value, acc._2 + 1)**， **value**是函数定义里面的**T**，这里是List里面的元素。所以**acc._1 + value, acc._2 + 1**的过程如下：
   1. 0+1, 0+1
   2. 1+2, 1+1
   3. 3+3, 2+1
   4. 6+4, 3+1
3. 结果为 (10,4)。在实际Spark执行中是分布式计算，可能会把List分成多个分区，假如3个，p1(1,2), p2(3), p3(4)，经过计算各分区的的结果 (3,2), (3,1), (4,1)，这样，执行 **(acc1,acc2) => (acc1._1 + acc2._1, acc1._2 + acc2._2)** 就是 **(3+3+4,2+1+1)** 即 **(10,4)**，然后再计算平均值。

> collect

RDD 的一些行动操作会以普通集合或者值的形式将 RDD 的部分或全部数据返回驱动器程序中。数据返回驱动器: 最简单,最常用--colllect

不过常用于单元测试中,因为需要将数据复制到驱动器进程中,collect( )要求所有数据都必须能一同放入单台机器的内存中.

> take( n )

返回RDD中的n个元素, 并且尝试只访问尽量少的分区, 因此该操作会得到一个不均衡的集合 (顺序与预期可能不同)

> top( )

如果为数据定义了顺序, 使用top( )获取前几个元素

top( )使用数据的默认顺序.但我们也可以提供自己的比较函数,来提取前几个元素

> takeSample(withReplacement, num, seed)

在驱动器中对数据进行采样

> 有时对RDD中所有元素应用一个行动操作, 而不把任何结果返回到驱动器程序中. 如 用JSON格式把数据发送到一个网络服务器上,或把数据存到数据库中
>
> 不论哪种情况, 都可以使用foreach()行动操作来对RDD中的每个元素进行操作,
>
> 而不需要把RDD发回本地.

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-20 下午2.25.33.png)

### 在不同的RDD类型间转换

有些函数只能用于特定类型的RDD,比如mean( )和variance( ) 只能用在数值RDD中

join( )只能用在键值对RDD上.

#### scala中的类型转换

在Scala中,将RDD转为由特定函数的RDD(比如在RDD[Double]上进行数值操作)是由隐式转换来自动处理的.

需导包: import org.apache.spark.SparkContext._

隐式转换可以隐式的将一个RDD转为各种封装类, 比如DoubleRDDFunctions(数值数据的RDD)和PairRDDFunctions(键值对RDD), 这样我们就有了诸如mean( ) 和 variance( )之类的额外函数.


隐式转换虽然强大，但是会让阅读代码的人感到困惑. 当我们在 Scala 文档中查找函数时，不要忘了那些封装的专用类中的函数。

### 持久化(缓存)

有时我们希望能多次使用同一个 RDD。如果简单地对 RDD 调用行动操作，Spark 每次都会重算 RDD 以及它的所有依赖.—消耗格外大

为了避免多次计算同一个RDD, 可以让Spark对数据进行持久化.

当我们让 Spark 持久化存储一个 RDD 时，计算出 RDD 的节点会分别保存它们所求出的分区数据。如果一个有持久化数据的节点发生故障，Spark 会在需要用到缓存的数据时重算丢失的数据分区。如果希望节点故障的情况不会拖累我们的执行速度，也可以把数据备份到多个节点上。


在 Scala和 Java 中，默认情况下 persist() 会把数据以序列化的形式缓存在 JVM 的堆空间中。

```scala
val result = input.map(x=>x*x)
result.persist(StorageLevel.DISK_ONLY)
println(result.count())
println(result.collect().mkString())
//注意，我们在第一次对这个 RDD 调用行动操作前就调用了 persist() 方法。persist() 调 用本身不会触发强制求值。
```

如果要缓存的数据太多，内存中放不下，Spark 会自动利用最近最少使用(LRU)的缓存策略把最老的分区从内存中移除。对于仅把数据存放在内存中的缓存级别，下一次要用到已经被移除的分区时，这些分区就需要重新计算。但是对于使用内存与磁盘的缓存级别的分区来说，被移除的分区都会写入磁盘。不论哪一种情况，都不必担心你的作业因为缓存了太多数据而被打断。不过，缓存不必要的数据会导致有用的数据被移出内存，带来更多重算的时间开销。

最后，RDD 还有一个方法叫作 unpersist()，调用该方法可以手动把持久化的 RDD 从缓存中移除。

## 第4章 键值对操作

键值对 RDD 通常用来进行聚合计算。我们一般要先通过一些初始 ETL(抽取、转化、装载)操作来将数据转化为键值对形式。键值对 RDD 提供了一些新的操作接口(比如统计每个产品的评论，将数据中键相同的分为一组，将两个不同的 RDD 进行分组合并等)。

有时，使用可控的分区方式把常被一起访问的数据放到同一个节点上，可以大大减少应用的通信开销。这会带来明显的性能提升.

为分布式数据集选择正确的分区方式和为本地数据集选择合适的数据结构很相似.

### 动机

Spark 为包含键值对类型的 RDD 提供了一些专有的操作。这些 RDD 被称为 pair RDD。Pair RDD 是很多程序的构成要素，因为它们提供了并行操作各个键或跨节点重新进行数据分组的操作接口。

### 创建Pair RDD

-  存储键值对的数据格式会在读取时直接返回由其键值对数据组成的pair RDD
- 当需要把普通的RDD转化为pair RDD时, 可以调用map( )函数来实现

```scala
val pairs = lines.map(x=>(x.split(" ")(0),x))
```

Java没有自带的二元组类型, 因此Spark的Java API让用户使用scala.Tuple2类来创建二元组.

```scala
val pairs= new Tuple2(elem1,elem2)
可以通过._1, ._2方法访问其中的元素
```

当用 Scala 和 Python 从一个内存中的数据集创建 pair RDD 时，只需要对这个由二元组组成的集合调用SparkContext.parallelize() 方法。

### Pair RDD的转化操作

Pair RDD 可以使用所有标准 RDD 上的可用的转化操作。由于 pair RDD 中包含二元组，所以需要传递的函数应当操作二元组而不是独立的元素

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-20 下午3.24.16.png)
			
![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-20 下午3.36.00.png)

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-20 下午3.37.34.png)

Pair RDD也还是RDD(元素为Tuple2中的元组), 因此同样支持RDD所支持的函数

```scala
pairs.filter{case (key,value)=>value.length<20} 
```

有时，我们只想访问 pair RDD 的值部分，这时操作二元组很麻烦。由于这是一种常见的使用模式，因此 Spark 提供了 mapValues(func) 函数，功能类似于 map{case (x, y): (x,func(y))}。

#### 聚合操作

> reduceByKey()				

没有被实现为向用户程序返回一个值的行动操作.

它会返回一个由各键和对应键归约出来的结果值组成新的RDD

```scala
//需求:在Scala中使用reduceByKey()和mapValues()计算每个键对应的平均值
rdd.mapValues(x=>(x,1)).reduceByKey((x,y)=>(x._1+y._1, x._2+y._2))
```

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-20 下午4.06.05.png)			
![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-20 下午4.11.14.png)

```scala
//需求:实现单词计数
val input = sc.textFile("....")
//方法1
val words = input.flatMap(x=>x.split(" "))
val result = words.map(x=>(x,1)).reduceByKey((x,y)=>x+y)
//方法2
input.flatMap(x=>x.split(" ")).countByValue()

```

> combineByKey( )		

combineByKey( )是最为常用的基于键进行聚合的函数.	


和 aggregate() 一样，combineByKey() 可以让用户返回与输入数据的类型不同的返回值。

-  处理方式:

  由于combineByKey() 会遍历分区中的所有元素，因此每个元素的键要么还没有遇到过，要么就和之前的某个元素的键相同。

  -  如果是新元素: combineByKey( )会使用一个叫做createCombiner()	的函数来创建那个键对应的累加器的初始值.需要注意的是，这一过程会在每个分区中第一次出现各个键时发生，而不是在整个 RDD 中第一次出现一个键时发生。
  - 如果这是一个在处理当前分区之前已经遇到的键，它会使用 mergeValue() 方法将该键的累加器对应的当前值与这个新的值进行合并。


  由于每个分区都是独立处理的，因此对于同一个键可以有多个累加器。如果有两个或者更多的分区都有对应同一个键的累加器，就需要使用用户提供的 mergeCombiners() 方法将各å个分区的结果进行合并。

  ```scala
  //使用combineByKey()求每个键对应的平均值
  val result = input.combineByKey(
         (v) => (v, 1),
         (acc: (Int, Int), v) => (acc._1 + v, acc._2 + 1),
         (acc1: (Int, Int), acc2: (Int, Int)) => (acc1._1 + acc2._1, acc1._2 + acc2._2)
         ).map{ case (key, value) => (key, value._1 / value._2.toFloat) }
         result.collectAsMap().map(println(_))

  ```

  ![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-20 下午4.35.42.png)			

> 案例 求平均成绩

```scala
val initialScores = Array(("Fred", 88.0), ("Fred", 95.0), ("Fred", 91.0), ("Wilma", 93.0), ("Wilma", 95.0), ("Wilma", 98.0))  
val d1 = sc.parallelize(initialScores)  
type MVType = (Int, Double) //定义一个元组类型(科目计数器,分数)  
d1.combineByKey(  
  score => (1, score),  
  (c1: MVType, newScore) => (c1._1 + 1, c1._2 + newScore),  
  (c1: MVType, c2: MVType) => (c1._1 + c2._1, c1._2 + c2._2)  
).map { case (name, (num, socre)) => (name, socre / num) }.collect  
```

参数含义的解释 :

a 、score => (1, score)，我们把分数作为参数,并返回了附加的元组类型。 以"Fred"为列，当前其分数为88.0 =>(1,88.0)  1表示当前科目的计数器，此时只有一个科目

b、(c1: MVType, newScore) => (c1._1 + 1, c1._2 + newScore)，注意这里的c1就是createCombiner初始化得到的(1,88.0)。在一个分区内，我们又碰到了"Fred"的一个新的分数91.0。当然我们要把之前的科目分数和当前的分数加起来即c1._2 + newScore,然后把科目计算器加1即c1._1 + 1

c、 (c1: MVType, c2: MVType) => (c1._1 + c2._1, c1._2 + c2._2)，注意"Fred"可能是个学霸,他选修的科目可能过多而分散在不同的分区中。所有的分区都进行mergeValue后,接下来就是对分区间进行合并了,分区间科目数和科目数相加分数和分数相加就得到了总分和总科目数

执行结果 :

res1: Array[(String, Double)] = Array((Wilma,95.33333333333333), (Fred,91.33333333333333))  

> 并行度调优

每个 RDD 都有固定数目的分区，分区数决定了在 RDD 上执行操作时的并行度。

在执行聚合或分组操作时，可以要求 Spark 使用给定的分区数。Spark 始终尝试根据集群的大小推断出一个有意义的默认值，但是有时候你可能要对并行度进行调优来获取更好的性能表现。

```scala
//需求:在Scala中自定义reduceByKey()的并行度
val data = Seq(("a", 3), ("b", 4), ("a", 1)) sc.parallelize(data).reduceByKey((x, y) => x + y) // 默认并行度 
sc.parallelize(data).reduceByKey((x, y) => x + y,10) // 自定义并行度
```

有时，我们希望在除分组操作和聚合操作之外的操作中也能改变 RDD 的分区。对于这样的情况，Spark 提供了 repartition() 函数。它会把数据通过网络进行混洗，并创建出新的分区集合。切记，对数据进行重新分区是代价相对比较大的操作。Spark 中也有一个优化版的 repartition()，叫作 coalesce()。你可以使用 Java 或 Scala 中的 rdd.partitions.size() 以及 Python 中的 rdd.getNumPartitions 查看 RDD 的分区数，并确保调用 coalesce() 时将 RDD 合并到比现在的分区数更少的分区中。

####数据分组

> groupByKey( )

-  如果数据已经以预期的方式提取了键:

groupByKey( )会使用RDD中的键来对数据进行分组

结果: [K, Iterable[V]]

- 如果数据未成对

可以根据除键相同以外的条件进行分组。它可以接收一个函数，对源 RDD 中的每个元素使用该函数，将返回结果作为键再进行分组。

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-21 下午1.38.06.png)			

> cogroup()	

对多个共享同一个键对的RDD进行分组.

对两个键的类型均为K

值的类型分别为V,W的RDD

cogroup()时:

得到的结果RDD为: [(K, (Iterable[V], Iterable[W]))]。如果其中的一个 RDD 对于另一个 RDD 中存在的某个键没有对应的记录，那么对应的迭代器则为空. cogroup() 提供了为多个 RDD 进行数据分组的方法。


cogroup() 不仅可以用于实现连接操作，还可以用来求键的交集。除此之外，cogroup() 还能同时应用于三个及以上的 RDD。

#### 连接

分类: 右外连接, 左外连接, 交叉连接, 以及 内连接

> 内连接 join

只有在两个pair RDD中都存在的键才叫输出.

当一个输入对应的某个键有多个值时，生成的 pair RDD 会包括来自两个输入 RDD 的每一组相对应的记录。

```scala
storeAddress = {
       (Store("Ritual"), "1026 Valencia St"), (Store("Philz"), "748 Van Ness Ave"),
       (Store("Philz"), "3101 24th St"), (Store("Starbucks"), "Seattle")}
     storeRating = {
       (Store("Ritual"), 4.9), (Store("Philz"), 4.8))}
     storeAddress.join(storeRating) == {
       (Store("Ritual"), ("1026 Valencia St", 4.9)),
       (Store("Philz"), ("748 Van Ness Ave", 4.8)),
       (Store("Philz"), ("3101 24th St", 4.8))}
```

> 左外连接和右外连接

leftOuterJoin(other) 和 rightOuterJoin(other)都会根据键连接两个RDD, 但是允许结果中存在其中一个pair RDD所缺失的键

```scala
storeAddress.leftOuterJoin(storeRating) ==
     {(Store("Ritual"),("1026 Valencia St",Some(4.9))),
       (Store("Starbucks"),("Seattle",None)),
       (Store("Philz"),("748 Van Ness Ave",Some(4.8))),
       (Store("Philz"),("3101 24th St",Some(4.8)))}
     storeAddress.rightOuterJoin(storeRating) ==
     {(Store("Ritual"),(Some("1026 Valencia St"),4.9)),
       (Store("Philz"),(Some("748 Van Ness Ave"),4.8)),
       (Store("Philz"), (Some("3101 24th St"),4.8))}
```

####数据排序

很多时候，让数据排好序是很有用的，尤其是在生成下游输出时。如果键有已定义的顺序，就可以对这种键值对 RDD 进行排序。当把数据排好序后，后续对数据进行 collect()或 save() 等操作都会得到有序的数据。

> sortByKey( )

接收一个ascending的参数(默认 true), 也可以自定义比较函数

```scala
//需求:以字符串顺序对整数进行自定义排序
val input: RDD[(Int, Venue)] = ...
     implicit val sortIntegersByString = new Ordering[Int] {
       override def compare(a: Int, b: Int) = a.toString.compare(b.toString)
     }
     rdd.sortByKey()
```

## Day02

配置文件: spark-env.sh

给worker分配的core 和 memory,如果集群中每台机器的设置不同,how? 分别设置, 然后先stop-slave.sh

之后改设置,然后 start-slave.sh spark://node1:7077

在生产环境下, 用管理工具来管理.(将集群分类)

memory必须是正数! 如果是1.5g 要写为1500m

### Yarn模式

启动yarn: start-yarn.sh

## Spark Core编程基础

### RDD(Resilient Distributed Datasets) 弹性分布式数据集

表现形式: 可以对它分区 等等

Spark的核心, 数据就存在RDD里, 对Spark操作,就相当于操作RDD.

它是一个容错, 可以并行执行的分布式数据集.

##### 5大特征:

1. 一个分区的列表

   分了区的list 

2. 一个计算函数compute, 对每个分区进行计算(通常是开发人员开发的)

3. 对其他RDDs的依赖(宽依赖(有shuffle),窄依赖)列表

4. 对key-value RDDs来说, 存在一个分区器(partitioner)(可选的)—>系统提供的算hash值,可以自己定义

5. 对每个分区有一个优先位置的列表(可选的)

##### 5大特征包含4个函数和一个属性

- compute 
- getPartitioner
- getDependence
- ​
- 一个属性: partitioner

Driver的核心就是sparkContext

sparkcontext: 在一个应用程序中,只允许有一个.

物理模型:

Driver里初始化SparkContext

Driver会把任务发送到整个集群(移动计算不移动数据)

运算后的结果, 收集到Driver

我们的语句编写,实际操作的是Driver

但是语句的执行在Worker中运行的, 所以有时候写print打印不出来.

每个app都会到master节点上注册.

一个project包含多个app. 实例化一个SparkContext就是建立一个app

val rdd1 = sc.textaFile("data/a.log")

//加了主机名和端口号,就不可以简写了! 

hdfs://node1:8020/user/spark/data/a.log

本地系统:

cp  a.log ~/

val rdd1 = sc.textFile("file:///home/spark/a.log")					

### RDD操作

##### action触发Job!  几个action就有几个Job

##### transformation生成新的RDD!

> 注意

map是transformation    foreach是action

ctrl+n 搜索源码关键字

map: 遍历所有元素

mapPartitions: 对一个分区遍历一遍

数据集太大, 想看一下数据,需要抽样

抽样: 1. 有放回的抽样 : withReplacement

1. 无放回的抽样

sample(withReplacement, faction, seed):设定了seed后,每次抽样的结果都相同(抽样结果的数量,不一定与设定的值对等)

collect不要用! ???看视频

### Pair RDD的行动操作

和转化操作一样，所有基础 RDD 支持的传统行动操作也都在 pair RDD 上可用。Pair RDD提供了一些额外的行动操作，可以让我们充分利用数据的键值对特性

![](/Users/lijingwan/Documents/img/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-04-21%20%E4%B8%8B%E5%8D%883.25.57.png)

### 数据分区??

Spark 程序可以通过控制RDD 分区方式来减少通信开销。分区并不是对所有应用都有好处的——比如，如果给定RDD 只需要被扫描一次，我们完全没有必要对其预先进行分区处理。只有当数据集多次在诸如连接这种基于键的操作中使用时，分区才会有帮助。

Spark 中所有的键值对 RDD 都可以进行分区。	但 Spark 可以确保同一组的键出现在同一个节点上。你也可以使用范围分区法，将键在同一个范围区间内的记录都放在同一个节点上。

```scala
// 初始化代码;从HDFS商的一个Hadoop SequenceFile中读取用户信息
// userData中的元素会根据它们被读取时的来源，即HDFS块所在的节点来分布
// Spark此时无法获知某个特定的UserID对应的记录位于哪个节点上
val sc = new SparkContext(...)
val userData = sc.sequenceFile[UserID, UserInfo]("hdfs://...").persist()
// 周期性调用函数来处理过去五分钟产生的事件日志
// 假设这是一个包含(UserID, LinkInfo)对的SequenceFile def processNewLogs(logFileName: String) {
       val events = sc.sequenceFile[UserID, LinkInfo](logFileName)
       val joined = userData.join(events)// RDD of (UserID, (UserInfo, LinkInfo)) pairs
       val offTopicVisits = joined.filter {
         case (userId, (userInfo, linkInfo)) => // Expand the tuple into its components
           !userInfo.topics.contains(linkInfo.topic)
}.count()
       println("Number of visits to non-subscribed topics: " + offTopicVisits)
     }

```

==注意==: 此种默认情况下，连接操作会将两个数据集中的所有键的哈希值都求出来，将该哈希值相同的记录通过网络传到同一台机器上，然后在那台机器上对所有键相同的记录进行连接操作.

==解决方法== 在程序开始时, 对userData表使用partitionBy( )转化操作, 将这张表转为哈希分区.

```scala
 val sc = new SparkContext(...)
     val userData = sc.sequenceFile[UserID, UserInfo]("hdfs://...")
.partitionBy(new HashPartitioner(100)) // 构造100个分区 .persist()
```

具体来说，当调用 userData.join(events) 时，Spark 只会对 events 进行数据混洗操作，将 events 中特定 UserID 的记录发送到 userData 的对应分区所在的那台机器上(见图 4-5)。这样，需要通过网络传输的数据就大大减少了，程序运行速度也可以显著提升了。

==注意==partitionBy() 是一个转化操作，因此它的返回值总是一个新的RDD，但它不会改变原来的 RDD。RDD 一旦创建就无法修改。因此应该对 partitionBy() 的结果进行持久化，并保存为 userData，而不是原来的 sequenceFile() 的输出。

100表示分区数目, 总的来说,这个值至少应该和集群中的总核心数一样.			
![](/Users/lijingwan/Documents/img/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-04-21%20%E4%B8%8B%E5%8D%885.41.32.png)

![](/Users/lijingwan/Documents/img/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-04-21%20%E4%B8%8B%E5%8D%885.42.10.png)			

#### 获取RDD的分区方式

可以使用RDD的partitioner属性来获取RDD的分区方式.

返回值为一个 scala.Option对象

调用isDefined( )来检查其中是否有值,且调用get( )来获取其中的值.

如果值存在, 则值会是一个spark.Partitioner对象.

```scala
//需求:获取RDD的分区方式
 scala> val pairs = sc.parallelize(List((1, 1), (2, 2), (3, 3)))
     pairs: spark.RDD[(Int, Int)] = ParallelCollectionRDD[0] at parallelize at
     <console>:12
     scala> pairs.partitioner
     res0: Option[spark.Partitioner] = None
     scala> val partitioned = pairs.partitionBy(new spark.HashPartitioner(2))
     partitioned: spark.RDD[(Int, Int)] = ShuffledRDD[1] at partitionBy at <console>:14
     scala> partitioned.partitioner
     res1: Option[spark.Partitioner] = Some(spark.HashPartitioner@5147788d)
```

如果确实要在后续操作中使用 partitioned，那就应当在定义partitioned 时，在第三行输入的最后加上 persist()。

如果不调用 persist() 的话，后续的 RDD 操作会对partitioned 的整个谱系重新求值，这会导致对 pairs 一遍又一遍地进行哈希分区操作。

#### 从分区中获益的操作

Spark 的许多操作都引入了将数据根据键跨节点进行混洗的过程。所有这些操作都会从数据分区中获益。

就 Spark 1.0 而言，能够从数据分区中获益的操作有 cogroup(), groupWith()、join()、leftOuterJoin()、rightOuterJoin(), groupByKey()、reduceByKey()、combineByKey() 以及 lookup()。

对于诸如 cogroup() 和join() 这样的二元操作，预先进行数据分区会导致其中至少一个 RDD(使用已知分区器的那个 RDD)不发生数据混洗。

#### 影响分区方式的操作

Spark 内部知道各操作会如何影响分区方式，并将会对数据进行分区的操作的结果 RDD 自动设置为对应的分区器。

不过，转化操作的结果并不一定会按已知的分区方式分区，这时输出的 RDD 可能就会没有设置分区器。例如，当你对一个哈希分区的键值对 RDD 调用 map() 时，由于传给 map()的函数理论上可以改变元素的键，因此结果就不会有固定的分区方式。Spark 不会分析你的函数来判断键是否会被保留下来。不过，Spark 提供了另外两个操作 mapValues() 和flatMapValues() 作为替代方法，它们可以保证每个二元组的键保持不变。

> 所有会为生成的结果 RDD 设好分区方式的操作:

这里列出了所有会为生成的结果 RDD 设好分区方式的操作:cogroup()、groupWith()、join()、leftOuterJoin()、rightOuterJoin(), groupByKey()、reduceByKey()、combineByKey()、partitionBy()、sort()、mapValues()(如果父 RDD 有分区方式的话)、flatMapValues()(如果父 RDD 有分区方式的话)，以及 filter()(如果父 RDD 有分区方式的话)。

其他所有的操作生成的结果都不会存在特定的分区方式。

>  二元操作的分区			

- 输出数据的分区方式取决于父RDD的分区方式. 
- 默认: 采用哈希分区, 分区的操作和操作的并行度一样		
-  如果其中一个父RDD已经设置过分区方式: 结果就会采用那种分区方式
- 如果两个父RDD都设置过, 结果RDD会采用第一个父RDD的分区方式

#### 示例: PageRank??

PageRank 是一种从 RDD 分区中获益的更复杂的算法

算法会维护两个数据集:一个由 (pageID, linkList) 的元素组成，包含每个页面的相邻页面的列表;另一个由 (pageID, rank) 元素组成，包含每个页面的当前排序值。它按如下步骤进行计算。

(1)将每个页面的排序值初始化为 1.0。

(2)在每次迭代中，对页面 p，向其每个相邻页面(有直接链接的页面)发送一个值为rank(p)/numNeighbors(p) 的贡献值。

(3)将每个页面的排序值设为 0.15 + 0.85 * contributionsReceived。

```scala
// 假设相邻页面列表以Spark objectFile的形式存储
val links = sc.objectFile[(String, Seq[String])]("links")
                   .partitionBy(new HashPartitioner(100))
                   .persist()
// 将每个页面的排序值初始化为1.0;由于使用mapValues，生成的RDD // 的分区方式会和"links"的一样
var ranks = links.mapValues(v => 1.0)
// 运行10轮PageRank迭代 for(i <- 0 until 10) {
       val contributions = links.join(ranks).flatMap {
         case (pageId, (links, rank)) =>
           links.map(dest => (dest, rank / links.size))
       }
       ranks = contributions.reduceByKey((x, y) => x + y).mapValues(v => 0.15 + 0.85*v)
     }
// 写出最终排名 ranks.saveAsTextFile("ranks")
```

#### 自定义分区方式 ??

要实现自定义的分区器，你需要继承 org.apache.spark.Partitioner 类并实现下面三个方法。

- numPartitions: Int:返回创建出来的分区数。
- getPartition(key: Any): Int:返回给定键的分区编号(0 到 numPartitions-1)。
- equals():Java 判断相等性的标准方法。这个方法的实现非常重要，Spark 需要用这个方法来检查你的分区器对象是否和其他分区器实例相同，这样 Spark 才可以判断两个RDD 的分区方式是否相同。	

有一个问题需要注意，当你的算法依赖于 Java 的 hashCode() 方法时，这个方法有可能会键值对操作返回负数。你需要十分谨慎，确保getPartition() 永远返回一个非负数。			

```scala
//需求: 自定义基于域名的分区器, 这个分区器只对URL中的域名部分求哈希
class DomainNamePartitioner(numParts: Int) extends Partitioner {
       override def numPartitions: Int = numParts
       override def getPartition(key: Any): Int = {
         val domain = new Java.net.URL(key.toString).getHost()
         val code = (domain.hashCode % numPartitions)
         if(code < 0) {
code + numPartitions // 使其非负 }else{
code }
}
// 用来让Spark区分分区函数对象的Java equals方法
override def equals(other: Any): Boolean = other match {
         case dnp: DomainNamePartitioner =>
           dnp.numPartitions == numPartitions
         case _ =>
           false
} }

```

## 第5章 数据读取与保存

### 动机

Spark支持很多种输入输出源。一部分原因是 Spark 本身是基于 Hadoop 生态圈而构建，特别是 Spark 可以通过 Hadoop MapReduce 所使用的 InputFormat 和 OutputFormat 接口访问数据，而大部分常见的文件格式与存储系统(例如 S3、HDFS、Cassandra、HBase 等)都支持这种接口。

3类常见的数据源:

1.  文件格式与文件系统

   对于存储在本地文件系统或分布式文件系统(比如 NFS、HDFS、Amazon S3 等)中的数据，Spark 可以访问很多种不同的文件格式，包括文本文件、JSON、SequenceFile，以及 protocol buffer。

2. Spark SQL中的结构化数据源

3. 数据库与键值存储

### 文件格式

![](/Users/lijingwan/Documents/img/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-04-21%20%E4%B8%8B%E5%8D%888.44.05.png)

除了 Spark 中直接支持的输出机制，还可以对键数据(或成对数据)使用 Hadoop 的新旧文件 API。由于 Hadoop 接口要求使用键值对数据，所以也只能这样用，即使有些格式事实上忽略了键。对于那些会忽视键的格式，通常使用假的键(比如 null)。

#### 文本文件

读写方式:

-  读取单个文本文件: 每一行都成为RDD的一个元素
- 读取多个完整的文本文件一次性读取为一个pair RDD,其中键是文件名, 值是文件内容

1. 读取文本文件:

   ```scala
   val input = sc.nextFile("file:///home/holden/repos/spark/README.md")
   ```

   如果要控制分区数的话，可以指定 minPartitions。

   ​		
   如果多个输入文件以一个包含数据所有部分的目录的形式出现，可以用两种方式来处理。

   - 可以仍使用 textFile 函数，传递目录作为参数，这样它会把各部分都读取到 RDD中。
   - 有时候有必要知道数据的各部分分别来自哪个文件(比如将键放在文件名中的时间数据)，有时候则希望同时处理整个文件。如果文件足够小，那么可以使用 SparkContext.wholeTextFiles() 方法，该方法会返回一个 pair RDD，其中键是输文件的文件名。

> wholeTextFiles( )			

在每个文件表示一个特定时间段内的数据时非常有用.

```scala
//在scala中求每个文件的平均值
val input = sc.wholeTextFiles("file://home/holden/salesFiles")
val result = input.mapValues{y =>
       val nums = y.split(" ").map(x => x.toDouble)
       nums.sum / nums.size.toDouble
     }
```

Spark 支持读取给定目录中的所有文件，以及在输入路径中使用通配字符(如 part-*.txt)。

2. 保存文本文件

saveAsTextFile() 方法接收一个路径，并将RDD 中的内容都输入到路径对应的文件中。	Spark 将传入的路径作为目录对待，会在那个
目录下输出多个文件。这样，Spark 就可以从多个节点上并行输出了.		
在这个方法中，我们不能控制数据的哪一部分输出到哪个文件中，不过有些输出格式支持控制。

#### JSON

Json-半结构化数据

读取方式:

- 读取JSON最简单的方式: 将数据作为文本文件读取, 然后使用JSON解析器来对RDD中的值进行映射操作.
- 使用JSON序列化库来讲数据转为字符串, 然后将其写出去.
- 使用自定义的Hadoop格式来操作JSON数据

1.  读取JSON

   使用JSON解析器

   这种方法假设文件中的每一行都是一条JSON记录.如果你有跨行的JSON数据, 你就只能读入整个文件,然后对每个文件进行解析.

   如果构建JSON解析器的开销太大, 可以使用mapPartitions()来重用解析器. Scala中使用的JSON库为:Jackson

```scala
import com.fasterxml.jackson.module.scala.DefaultScalaModule
import com.fasterxml.jackson.module.scala.experimental.ScalaObjectMapper import com.fasterxml.jackson.databind.ObjectMapper
import com.fasterxml.jackson.databind.DeserializationFeature
...
case class Person(name: String, lovesPandas: Boolean) // 必须是顶级类
...
// 将其解析为特定的case class。使用flatMap，通过在遇到问题时返回空列表(None) // 来处理错误，而在没有问题时返回包含一个元素的列表(Some(_))
val result = input.flatMap(record => {
       try {
         Some(mapper.readValue(record, classOf[Person]))
       } catch {
         case e: Exception => None
}})
```

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-21 下午10.22.00.png)

2. 保存JSON

可以使用之前将字符串 RDD 转为解析好的 JSON 数据的库，将由结构化数据组成的 RDD 转为字符串 RDD，然后使用 Spark 的文本文件 API 写出去。

```scala
//选出喜爱熊猫的人, 并保存为JSON
result.filter(p => P.lovesPandas).map(mapper.writeValueAsString(_))
       .saveAsTextFile(outputFile)

```

​			
​		
​	



​	


​			
​		
​	


​			
​		
​	


​			
​		
​	

​	















​		
​		