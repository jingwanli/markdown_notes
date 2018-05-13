![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-24 上午8.44.26.png)

2. 5个特点:
   - 有一个分区列表
   - 有一个计算函数compute
   - 有宽依赖和窄依赖
   - 有一个可选的
   - pairRDD存在一个可选的分区器
3. makeRDD() / parallelize( )
4. perisist(MEMORY_ONLY)



7.  选择一个最大值(除128 和 1 之间取最大值)

lines.repartition(1) //下面可以改

---

## Day09

### 宽依赖 窄依赖

宽依赖: 有Shuffle

作用: 1.  解决了容错的问题(记下了所有的过程,可以重新计算)

 2. 多少个action就有多少个job.

    job里有多少个shuffle+1就有多少个stage,

    遇到宽依赖,切一刀.

    最后一个stage: result stage

    前面的stage: shuffle stage

	3. 每个stage又可以分为多个task

    task的数目 看最后一个RDD

==总结:==

Application 

Job: Action

Stage: shuffle ,Stage又叫TaskSets

task: 最后一个RDD的分区数 , 会发到一个executor里去执行. 

最后一个task叫result task, 其他叫shuffleMaptask

DAG图:

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-24 上午9.17.40.png)

Spark从后往前找, 执行的时候,从前往后执行.

遇到stage切一刀, 最后找到task,发到executor里

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-24 上午9.19.45.png)

对于窄依赖, 容错比较简单.

对于宽依赖, 容错的成本高.(有冗余的计算,重算的效率低)

整个依赖: 有时被称为 Lineage图



> 思考题

RDD A — RDD B ---RDD C —RDD D, 如果RDD D中的分区数据丢失, 是只需要在C的分区上重算, 还是需要从RDDA 开始重头计算.

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-24 上午9.52.20.png)

cache(写内存) 缓存 提高性能. 但是做RDD容错不保险, 算完后, 内存系统自动清理.

### RDD容错

容错: 链条很长, 计算复杂, 为了容错,不做cache(不保险),而做checkpoint! 不要写本地磁盘,要写hdfs(高容错), 而且要斩断依赖!!后面的计算从checkpoint开始算!

手动的清理.

checkpoint 也是transformation!

直到遇到第一个action时,才会真正的写到磁盘里!

用checkpoint之前要设置一个checkpoint目录: sc.setCheckpointDir("hdfs路径")

rdd1.isCheckpointed (检查是否被checkpoint)

rdd2.checkpoint

rdd2.getCheckpointFile

rdd2.count

rdd2.getCheckpointFile // 变成了true

rdd2.dependencies(0).rdd //变成了checkpoint

rdd2.dependencies(0).rdd.collect// 变成了rdd2的数据!!!(存磁盘的数据!)

### 共享变量

两种类型:

1. 广播变量
2. 累加器

主要用于优化.

#####广播变量

允许在每个机器上缓存一个只读的变量, 而不是为机器上的每一个任务都生成一个副本.

语法:

val broadcastVar = sc.broadcast(Array(1,2,3))

broadcastVar.value

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-24 上午10.17.55.png)

##### 累加器

count只能计一个数

做优化: 扫描一次 ,用累加变量加出来

​	    count可能要扫描2次.

executor里,累加器只能用,不能读.

只有收到driver里才可以用value方法来读.

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-24 上午10.22.02.png)

### 案例

1. wordcount作业

```scala
//去标点
"abcdefg".replace("a","x")//把a换成x
"abcdefg".replace("a","") //把a去掉
"abcdefga".replaceAll("[ab]","")//把a和b都换掉
"ab,,fga".replaceAll("[a-c]","")//把a和b都换掉
//去停用词
val lst = List("in","on","to")
lst.contains("in") //对每一个单词都要做
key value 做表连接 做左连接? 右连接?
停用词做一个表


---wordcount-----
需求: 大写转小写, 去标点, 去停用词, 统计
步骤:
1. 读文件
2. 去空行
3. 大写转小写
4. 去标点
5. 分词
6. 去停用词
7. 统计
//方式一
val lst = List("in","what","the","a","on")
val lines = sc.textFile("data/spark.log")
val words = lines.filter(_.trim.size!=0).flatMap(_.trim.split("\\s+")).map(_.toLowerCase)
val pairRDD= words.map(_.replaceAll("[:;.,]"," ")).filter(!lst.contains()).map((_,1))
//replaceAll后面必须用空格替换!因为有可能把2个词合并为1个
//使用filter (会全部遍历,如果停用词很多,效率低!)
val result = pairRDD.reduceByKey(_+_).sortBy(_._2,false)

优化:
1. 分词的位置 : 在后面分词比较好!遍历的次数减少!因为前4步都是对整体做的操作!
2. 去停用词的时机(join) 先统计,后用停用词
//join一定要是key value对!
//方式二
val lst = List("in","what","the","a","on")
val lines = sc.textFile("data/spark.log")
val words = lines.filter(_.trim.size!=0).map(_.toLowerCase.replaceAll("[:;.,]"," ")).flatMap(_.trim.split("\\s+")).map((_.1))
val allWords = words.reduceByKey(_+_)
val stopWords = sc.makeRDD(List("in","what","the","a","on")).map((_,true))
allWords.leftOuterJoin(stopWords).filter(_._2._2==none).map(x=>(x._1,x._2._1)).sortBy(_.2,false).collect
```

2. 简单的TopN

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-25 下午2.13.10.png)

```scala
val lines = sc.textFile("data/1.dat")
val data = lines.filter(_.trim.size!=0).Map(_.split(",")(2).trim.toInt)//拿到第3列(注意取值时要trim,防止逗号前后有空格) //data为数组
data.sortBy(x=>x,false).take(10)//
data.sortBy(x=>x,false).top(10)
data.sortBy(x=>x,false).first //取第一个
```

3. 分组求TopN

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-25 下午2.23.10.png)

```scala
spark-shell中:
val lines = sc.textFile("data/2.dat")
val data = lines.map(x=>x.split("\\s+")).map(x=>(x(0),x(1).toInt))

//方式2
val data = lines.map({
    val value = x.split("\\s+")
    (value(0),value(1).toInt)
})
data.collect

val rdd1 = data.groupByKey //返回的是tuple(String, Iterable)

val rdd1 = data.groupByKey.map(x=>(x._1,x._2.toArray.sorted.reverse.take(3)))

方式3:
case class MyClass(classname:String, score:Int)//建议用case class,自定义的话还要做序列化,反序列化
val lines = sc.textFile("data/2.dat")
val data = lines.map(x=>x.split("\\s+")).map(x=>MyClass(x(0),x(1).trim.toInt))
data.map(x=>(x.classname,x.score.sorted.reverse.take(3)))
  //可读性强一些
```

3. mapPartitions

   问题: rdd分了几个区

   ​	每个区有多少个元素

   ​	这些元素是什么?

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-25 下午7.13.04.png)

```scala
//mapPartitions返回的迭代器,一个迭代器就是在一个分区里面.

val rdd = sc.makeRDD(1 to 100,3)
//求数量
rdd.getNumPartitions
rdd.partitions.size

rdd.mapPartitions(x=>{
    Iterator(x.size) //x也是一个迭代器,返回一个迭代器
}).collect //结果: Array[int] = Array(33,33,34)
//在rdd的转化过程中,不可以写collect(只是为了查看结果,因为使用了collect数据类型变成了数组,操作的已经不是rdd,而是数组)

//要想打印出分区索引号,使用mapPartitionWithIndex算子
rdd.mapPartitionsWithIndex((index,iter)=>{
    Iterator(index.toString + ":"+ iter.size.toString)
}).collect  Array(0:33,1:33,2:34)

//打印每一个元素
rdd.mapPartitionsWithIndex((index,iter)=>{
    Iterator(index.toString + ":"+ iter.toArray.mkString(","))
}).collect
//结果: Array[String] = Array(0:1,2,3,4,5....., 1:34,35,36......,2:67,68,...100)


map:针对每一个元素
mapPartitions: 针对每一个分区做动作
//需求:向数据库中写数据库:
1. 连接(最占资源,1s的任务,50%以上都在做连接)->连接池
2. 组织SQL
3. 执行SQL
4. 释放资源
对于map: 每一条语句都要执行上面4个步骤

```

4. 求均值1

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-25 下午7.12.29.png)

```scala
//groupBy尽量少用,数据量特别大的时候容易内存溢出
//因为groupBy可能合并到单节点的数据特别多.(key相同的在同一个分区)
val arr = Array(("spark",2),("hadoop",6),("hadoop",4),("spark",6));
val rdd1 = sc.makeRDD(arr)
rdd1.groupByKey().map(x=>  (x._1,x._2.sum.toDouble/x._2.size))
//groupByKey 输出:spark,Array(2,6)-->是个元组!

//方式2
val arr = Array(("spark",2),("hadoop",6),("hadoop",4),("spark",6));
val rdd1 = sc.makeRDD(arr)
rdd1.mapValues(x=>(x,1)).collect 
//结果为Array((spark,(2,1),(hadoop,(6,1)))
rdd1.mapValues(x=>(x,1)).reduceByKey((x,y)=>(x._1+y._1,x._2+y._2)) //结果:Array((hadoop,(10,2)),(spark,(8,2)))

rdd1.mapValues(x=>(x,1)).reduceByKey((x,y)=>(x._1+y._1,x._2+y._2)).mapValues(x=>x._1.toDouble/x._2).collect
//返回结果: Array((spark,4.0), (hadoop,5.0))
```

5. 求均值2

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-25 下午7.11.33.png)

```scala
import scala.util.Random
val random = new Random(100)
val arr = (1 to 100).map(x=>(random.nextInt(1000),random.nextInt(1000))).toArray
val rdd = sc.makeRDD(arr) 
//结果:Array((915,241),(874,988),...)
//方式1:
arr.map(x=>x._1).sum
arr.map(x=>x._1).size
rdd.map(x=>x._1).sum
rdd.map(x=>x._1).count//这里只能用count!

//方式2: 用向量(array是不能直接加的!)vector
//可以把vector想成一个可以累加的数组
import breeze.linalg.Vector
val rdd1 = rdd.map(x=>(1,Vector(x._1.toDouble,x._2.toDouble)))
//加一个key,可以改变数据结构,让我们只处理value
val rdd2 = rdd1.mapValues(x=>(x,1)).reduceByKey((x,y)=>(x._1+y._1,x._2+y._2))
//x._1是vector, x._2是1
rdd2.mapValues(x=>x._1/x._2.toDouble)
//x._1是向量形式的,x._2是100
//因为已经进入到别的包中,无法做自动类型转换了.
//所以x._2必须toDouble
//生产过程中这个还是不太好,要做二次聚合.因为map后进入一个分区了,容易内存溢出!

                        
```

6. 二次排序

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-25 下午8.35.34.png)

```scala
//需求:对数组arr, 按照字段1,2排序
case class MyObject(x:Int,y:Int) extends Ordered[MyObject]{ 
    def compare(other:MyObject):Int={
        if(x-other.x!=0) x-other.x
        else other.y - y
    }
}
val lines = sc.textFile("data/data2.txt")
val rdd1 = lines.map(line=>(new MyObject(line.split(" ")(0).toInt,line.split(" ")(1).toInt),line))
val sorted = rdd1.sortByKey(false)
val result = sorted.map(sortedLine=>sortedLine._2)
result.collect().foreach(println)
注意: 二次排序后的结果写文件!文件内及文件间的数据是有顺序的, 但是文件间的顺序是不保证的!
--------------------------------------------
import scala.util.Random
val random = new Random(100)
val arr = (1 to 10000).map(x=>(random.nextInt(100),random.nextInt(100),random.nextInt(100)))
val rdd = sc.makeRDD(arr)
rdd.sortBy(_._1).saveAsTextFile("data/t6")//t6是个目录!是系统要新建的(创建前不能存在!)
rdd.sortBy(x=>(x._1,x._2).saveAsTextFile("data/t7") //x=>后要加括号,把其整体视为key




```

7. join操作

```scala

```

---

groupBy(function) 
function返回key，传入的RDD的各个元素根据这个key进行分组

```
val a = sc.parallelize(1 to 9, 3)
a.groupBy(x => { if (x % 2 == 0) "even" else "odd" }).collect//分成两组
/*结果 
Array(
(even,ArrayBuffer(2, 4, 6, 8)),
(odd,ArrayBuffer(1, 3, 5, 7, 9))
)
*/
```

```
val a = sc.parallelize(1 to 9, 3)
def myfunc(a: Int) : Int =
{
  a % 2//分成两组
}
a.groupBy(myfunc).collect123456
```

/* 
结果  

Array( 
(0,ArrayBuffer(2, 4, 6, 8)),  

(1,ArrayBuffer(1, 3, 5, 7, 9)) 
) 
*/

------

## groupBy和groupByKey

groupBy(function) 
function返回key，传入的RDD的各个元素根据这个key进行分组

```
val a = sc.parallelize(1 to 9, 3)
a.groupBy(x => { if (x % 2 == 0) "even" else "odd" }).collect//分成两组
/*结果 
Array(
(even,ArrayBuffer(2, 4, 6, 8)),
(odd,ArrayBuffer(1, 3, 5, 7, 9))
)

val a = sc.parallelize(1 to 9, 3)
def myfunc(a: Int) : Int =
{
  a % 2//分成两组
}
a.groupBy(myfunc).collect
```

/* 
结果  
Array( 
(0,ArrayBuffer(2, 4, 6, 8)),  
(1,ArrayBuffer(1, 3, 5, 7, 9)) 
) 
*/

groupByKey( )

```
val a = sc.parallelize(List("dog", "tiger", "lion", "cat", "spider", "eagle"), 2)
val b = a.keyBy(_.length)//给value加上key，key为对应string的长度 //结果: RDD[(Int, String)]
b.groupByKey.collect 
//结果 Array((4,ArrayBuffer(lion)), (6,ArrayBuffer(spider)), (3,ArrayBuffer(dog, cat)), (5,ArrayBuffer(tiger, eagle)))
```











