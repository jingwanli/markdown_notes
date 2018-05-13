![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-25 上午8.40.42.png)

2. 宽依赖: 有shuffle , 一对多

   窄依赖: 一对一 或者 N对一 .  map ,filter

3. checkpoint: 斩断了依赖,...

   cache: = presist(MEMORY_ONLY). 有可能被换出

   presist: 

4. 广播变量: 只读

   累加器 :

5. case class Person(name:String, age:Int)

   Object SortPerson{

   ​	def mainargs:Array[String]){

   ​		val p1 = Person("Lili",100)

   ​		val p2 = Person("Amy",100)

   ​		val p3 = Person("Olivia",100)

   ​		val p4 = Person("Jane",100)

   ​		val pairs=Array(p1,p2,p3,p4)

   ​		pairs.sortBy(Person=>(Person.name,Person.age))

   ​	}

   }

6. 数据库: commit以后, 数据写到磁盘, 除非物理介质损坏, 否则数据是不会丢失的.

---

## Day10

### 分区器

分区器直接决定了:

1. RDD中分区的个数
2. RDD中每条数据经过Shuffle过程属于哪个分区
3. reduce的个数

只有key-value类型的RDD才有分区器,非key-value类型的RDD分区器的值是none,但是可以分区

shuffle, 重组!—分区器起作用

常见的两种:

 Hash分区: key的hash值对分区数取余

​		    所以同一个key, 一定在同一个分区.

Range分区: 主要用于排序

### 实验

```scala
val rdd1 = sc.makeRDD(1 to 100)//6个分区
val rdd2 = rdd1.union(rdd1).map((_,1))
val rdd3 = rdd2.reduceByKey(_+_)
val rdd4 = rdd3.sortByKey()
//sortByKey是transformation! 但是会触发job!!!
rdd2.partitioner:没有分区!
rdd3.partitioner: hash分区器
rdd4.partitioner:range分区器(整体有序! 内部需要排过才有序!整体有序: 每个分区的值都大于或小于另一个分区的值)

rdd3.getNumPartitions: 12个分区
(1 to 100).map(_.hashCode % 12)

rdd3.mapPartitonsWithIndex((idx, iter)=>Iterator(idx.toString + ":" + iter.toArray.mkString(","))).collect

//定义成函数
import org.apache.spark.rdd.RDD
def getElement(rdd:RDD[_])={
    rdd.mapPartitonsWithIndex((idx, iter)=>Iterator(idx.toString + ":" + iter.toArray.mkString(","))).collect
}


```

> sortByKey

是transformation,但是会触发job, why?

采样,水塘采样算法, 采样的结果 collect 所以会触发job

why采样? 因为要确定分区的边界,需要去采样来看数据总体上来说是什么样子.

但是sortByKey最终还是创建了一个新的RDD,故是transformation.

数据倾斜: 比如12的倍数特别多.

打散! 分区数调成11.(分区数下调)

主动使用partition

```scala
val rdd1 = sc.makeRDD(1 to 100)
val rdd2 = rdd1.map((_,1))
rdd2.getNumPartitions

val rdd3 = rdd2.partitionBy(new org.apache.spark.HashPartitioner(10))
rdd3.getNumPartitions //10
rdd3.partitioner// key-value才有分区器
//主动使用了分区器,系统会重新定义分区器

val rdd4 = rdd3.map(_._1)
rdd4.partitioner //丢掉了分区器

val rdd5 = rdd3.map((_,1))
rdd5.partitioner //丢掉了分区器(动了key)

val rdd6 = rdd3.map(x=>(x._1,x_2*2))
rdd6.partitioner//丢了分区器
rdd6.reduceByKey(_+_)//一定有分区器

val rdd7 = rdd3.mapValues(x=>x*2)
rdd7.partitioner //不会丢掉分区器!

//只要动了key,一定会有新的partitioner!即子RDD可能会失去父RDD的分区方式


```

### 自定义分区器

```scala
import.....
//一定继承了抽象类或接口!不会从0开始写
class MyPartitioner extends Partitioner{
    override def numPartitions:Int = 2
    override def getPartition(key:Any):Int={
        val value = key.toString.toInt
        if (value > 100) 1 else 0
    }
}

object UserDefinedPartitioner{
    def main(args:Array[String]):Unit={
 		val conf = new SparkConf().setMaster("local").setAppName("..")
        val sc = new SparkContext(conf)
        sc.setLogLevel("WARN")
        val rdd1 = sc.parallelize(List(99,88,200,10,900,1000)).map(x=>(x,1))
        val rdd3 = rdd1.repartition(2)
        val rdd2= rdd.partionBy(new MyPartitioner)
      import org.apache.spark.rdd.RDD
def getElement(rdd:RDD[_])={
    rdd.mapPartitonsWithIndex((idx, iter)=>Iterator(idx.toString + ":" + iter.toArray.mkString(","))).collect
}
        val a = getElement(rdd2) //返回的是迭代器!
        println(a.toBuffer)
        
        val b = getElement(rdd3)
        println(b.toBuffer)
        
        rdd4 = rdd1.reduceByKey(_+_)
        val c = getElement(rdd4)
        println(c.toBuffer) //按奇偶分(因为有2个分区)
        sc.stop()
    }
    
}
```

### Shuffle

与Hadoop的shuffle没有本质的区别.

> wordCount中的shuffle

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-25 上午10.36.09.png)

如果要算其中1步的运行时间:

加时间戳是看不出的.加打印语句不会生效.

因为不遇到action,其他的都只是标记

去UI界面中看!

所有的算子的源码: 都以 withScope开头, 作用是记录了各种信息,然后画到ui界面中.

Hadoop中的shuffle是sort shuffle(要排序)

Spark Shuffle版本的演进:

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-25 上午10.45.40.png)

1.6版: 将1.1 ,1.5结合起来:

可以使用堆外内存,也加入了sortShuffle

2.0: 去掉了hash Shuffle 只剩下sort shuffle

堆外内存: 不属于JVM的内存,Spark程序来管理.(memory manager来管理),提高性能(没有jc了)

### Hash Shuffle 

> Hash V1

两个阶段: Shuffle write, Shuffle read

stage里有很多Task. Task与Task之间,处理的逻辑一样, 处理的数据不同.

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-25 上午11.09.24.png)

1个core的话, 任务不能并行, 每次只能执行一个Task

(同时跑两个,其实是分时间片, 从cpu的角度来说, 是串行. 效率低(要进行进程切换))

HashShuffle: 3个reducer ,写3个文件

shuffer write: 会产生大量的小文件.

> Hash V2

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-25 上午11.16.32.png)

改进: 在executor里做到文件共享.

> Sort Shuffle

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-25 上午11.17.59.png)

注意:Kafka是顺序写文件.消息都需要些文件(顺序写文件效率很高)

因为sort排序占资源大,所以1.5引入了堆外内存.

> 面试题

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-25 上午11.22.49.png)

问题1: 写本地磁盘.

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-25 上午11.25.01.png)

写到哪里可以手动设置.

配磁盘方式:

10T: 推荐5个2T的.

可以同时读,同时写,效率高

不用磁盘阵列(成本高)

生产环境下, 集群不建议写在云上.(保证不了数据的本地性,都是虚拟的)

上面的diskb,….. 是不同的物理盘

### 常见算子中的Shuffle

> 分区

repartition: 有shuffle

coalesce:没有shuffle

```scala
val rdd1 = rdd0.coalesce(1)

val rdd2 = rdd0.coalesce(10,true)
//分区数>原来的分区, 必须指定true->指定shuffle
```

### KMeans过程

> KNN 

用来分类

> KMeans

聚类 (不知道分几类, 把其聚为几类)

1. 随便取2个点
2. 离谁近就属于哪个类: 求举例
3. 求各类的中心点: 求均值

```scala
1. 读文件 假设K3
2. 随机找3个点,作为初始的中心点
3. 遍历数据集计算,计算某点与三个中心点的距离,距离中心点最近就属于哪个中心点
4. 根据新的分类计算新的中心点(求均值)
5. 循环
退出条件: 1.指定循环次数
		 2.所有的中心点几乎不再移动:中心点移动的最大值小于某个常数. 


```

















