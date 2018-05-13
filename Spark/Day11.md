## 回顾

分区器: 不是pairRDD没有分区器. 都是按key分区

hash分区: key对分区号取余

range: 重要的要确定分区的边界.使用了抽样来确定.水塘抽样

​	   sortby用range分区. 

shuffle: writer  reader   sortShuffle 是job划分成stage的依据.

那些算子会产生Shuffle??

##Day11

一个应用程序: 一个driver和若干个job. —>action划分

​			一个job, 若干个stage —> Shuffle

​			一个stage, 若干个task —>最后一个分区数

(java,单一职责: 一个类功能尽量单一一些,不要什么都做)

> SparkContext三大组件:

倒着推:

driver拿到DAG. 

1. DAGScheduler: 将DAG化成stage/taskSet :按照Shuffle(1个)
2. TaskScheduler: stage—> 输出 task(1个)
3. SchedulerBackend: 负责处理executor相关的东西(多个)

(集群服务和task之间解耦,屏蔽了物理层)

> Spark作业调度基本流程(Standalone)

1. SparkContext向master申请资源
2. master通知worker, 启动executor
3. executor向Driver注册 ,Driver把task发出去(Driver时刻监控executor!executor向Driver汇报)
4. 注销

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-26 上午8.56.45.png)



非常重要!!

Client Endpoint 代表app

Driver Endpoint 代表driver

向系统申请资源: spark submit时候,跟参数,根据参数申请资源. 没有指定用缺省参数.

Master Endpoint

Worder Endpoint 

划分消息发消息是由上述两个组件来完成的.

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-26 上午9.09.37.png)

###综合案例

> 综合练习二 日志分析

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-26 上午10.01.13.png)

PV: Page View

UV: user View

独立IP:

不一定准: 1. 网络爬虫   2. 上网有代理服务器(ip都算为一个)

```scala
//第一次上 先把过滤掉的数据取出来,看看匹配了多少
import java.util.regex.Pattern
object LogAnalyze{
    def main(args:Array[String]):Unit={
        var str1 = """....."""
        var str2 = """....."""
        var str3 = """....."""
        val pattern = Pattern.compile("""(\S+) (.+) \[(.+)\]""")
  		val match1 = pattern.matcher(str1)
        println("match1 ="+match1.matches())
  ------------      
         val pattern2 = Pattern.compile("""(\S+) (.+) \[(.+)\] \"(.+)\" (\d+) (\d+) \"(.+)\" \"(.+)\"   """)
        val match2 = pattern2.matcher(str2)
        println("match2 = " + match2.matches())
        for(i<-1 to 8) println(match2.group(i))
       
        -----------------------
        val pattern = Pattern.compile("""(\S+) (.+) \[(.+)\] \"(.+)\" (\d+) (\d+) \"(.+)\" \"(.+)\"   """)//一定定义在map外面!
      
        val...读文件
        val valueRDD = lines.map(line=>{
      		val match1 = pattern.matcher(line)
            if(match1.matches()){
            val ip = match1.group(1)
            val date = match1.group(3)
            val stat = match1.group(5)
            val page = match1.group(7)
            (ip,date,stat,page)
            }
            else{//匹配不上,把数据带出来!
                val ip = line
                val date = ""
                val stat = ""
                val page = ""
                (ip,date,stat,page)
            }
            println(valueRDD.count) //5318

        })
        valueRDD.cache()
      //1.统计每日PV和独立IP
      //  按天做一个wordcount
        val pv = valueRDD.map(value=>value._2.split(":")(0),1).reduceByKey(_+_).repartition(1).saveAsTextFile("data/pv")
        val ip = valueRDD.map(value=>((value._),1))
        
    
        
    }
    
}

日志信息:1. 流式处理
		2.定期抓取的数据

用case class 就不用写下划线_.2直接.成员变量

case class ...
logInfo

```

> 找共同的朋友

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-26 上午11.28.16.png)

```scala
方式1:----> 是错误的! (200,600) (100) 没有!!!!!不能遍历所有的两两用户!!!!
输入：邻接表

100, 200 300 400 500 600  
200, 100 300 400  
300, 100 200 400 500  
400, 100 200 300  
500, 100 300  
600, 100  

第一列表示用户，后面的表示用户的好友。
需求：查找两两用户的共同好友。

思路：1、key为两两用户，value为其中一个用户的所有好友

     2、求两个用户所有好友的交集

步骤：1、map：取每一行，组合user和其任一好友为key（key中的两个字段按字典序排列），user的所有好友为value

            2、reduce：求两个用户之间好友的交集

--------------------------
package dabook  
  
import org.apache.spark.{SparkConf, SparkContext}  
  
object CommFriend {  
  
  
  def intersection(set1: Set[String], set2: Set[String]):Set[String]={  
    if(set1 == null || set2 == null){  
      return null;  
    }  
    var result = Set[String]()  
    var small = Set[String]()  
    var big = Set[String]()  
    if(set1.size < set2.size){  
      small = set1  
      big = set2  
    }  
    else {  
      small = set2  
      big = set1  
    }  
  
//    println(small.mkString(","))  
  
    for(s <- small){  
      if(big.contains(s)){  
        result += s  
      }  
    }  
  
    return result  
  }  
  
  def mymap(x:String): Map[Tuple2[String, String], String] ={  
    val arr = x.split(",")  
    if(arr.length != 2){  
      return null  
    }  
    val host = arr(0).trim  
    val friends = arr(1).trim  
    val arr_friends = friends.split(" ")  
    var result = Map[Tuple2[String, String], String]()  
    for(friend <- arr_friends){  
      if(host.compareTo(friend) < 0){  
        result += (Tuple2(host, friend)->friends)  
      }  
      else {  
        result += (Tuple2(friend, host)->friends)  
      }  
    }  
    return result  
  }  
  
  def commfri(x:Iterable[String]): Set[String] ={  
    if(x.size != 2){  
      return null  
    }  
    val arr = x.toArray  
    var set1 = Set[String]()  
    var set2 = Set[String]()  
    val sz1 = arr(0).split(" ")  
    val sz2 = arr(1).split(" ")  
    for(s <- sz1){  
      set1 += s  
    }  
    for(s <- sz2){  
      set2 += s  
    }  
    intersection(set1, set2)  
  }  
  
  def main(args: Array[String]): Unit = {  
    //val sparkContext = SparkSession.builder().master("local[2]").appName("common friend").getOrCreate()  
    //val input = sparkContext.read.textFile("hdfs://127.0.0.1:9000/user/root/dabook/commfriend/cofriend").rdd  
  
    val sparkConf = new SparkConf().setAppName("commfri").setMaster("local[2]")  
    val sc = new SparkContext(sparkConf)  
    val input1 = sc.textFile("/home/xdk/file/dabook/cofriend")  
  
    val comm = input1.flatMap(x=>mymap(x)).groupByKey().map(x=>(x._1, commfri(x._2)))  
    comm.foreach(println)  
  }  
  
  def test(): Unit ={  
    var set1 = Set("1", "2", "s", "s", "d", "3")  
    var set2 = Set("3", "2", "1");  
  
    var res = intersection(set2, set1)  
  
    for(s <- res){  
      println(s)  
    }  
  }  
  
}  

输出：
[plain] view plain copy
((200,300),Set(100, 400))  
((100,600),Set())  
((300,400),Set(100, 200))  
((100,500),Set(300))  
((200,400),Set(100, 300))  
((100,200),Set(300, 400))  
((100,300),Set(200, 400, 500))  
((100,400),Set(200, 300))  
((300,500),Set(100))  
```

算法:

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-26 上午11.47.04.png)

```scala
import org.apache.spark.{SparkConf, SparkContext}

case class Persons(id1:Int, id2:Int) extends Ordered[Persons] {
  override def compare(that: Persons): Int = {
    if (id1 == that.id1) id2 compare that.id2
    else id1 compare that.id1
  }
}

object FindFriends {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setMaster("local").setAppName("FindFriends")
    val sc = new SparkContext(conf)
    sc.setLogLevel("WARN")

    val lines = sc.textFile("C:\\_D\\friend.txt")

    // 将数据转为key、value对的形式(这里的key有点特殊，还有顺序)，再拉平。这里是关键
    val pairs = lines.flatMap(line => {
      val tokens = line.split(", ")
      val person = tokens(0).toInt
      val friends = tokens(1).split(" ").map(_.toInt).sorted

      val result = for(i <- 0 until friends.size; j <- i+1 until friends.size) yield
        (Persons(friends(i), friends(j)), person)
      result
    })

    pairs.groupByKey.sortByKey().foreach(x=>println(x._1 + " : " + x._2.mkString(",")))
  }
}
-----------------
验证方法:
    val friends = Array(0, 1, 2, 3)
    for (i <- 0 until friends.size; j <- i + 1 until friends.size){
       println(friends(i), friends(j))
       println("token" +i,j)
    }
```

> KMeans算法实现

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-26 下午2.18.34.png)

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-26 下午2.19.33.png)

```scala
import breeze.linalg.{Vector, sum}
import breeze.numerics.{pow, sqrt}
import org.apache.spark.{SparkConf, SparkContext}

object KMeans {
  def getDist(x: Vector[Double], y: Vector[Double]):Double ={
    sqrt(sum(pow(x-y, 2)))
  }

  def getIndex(p: Vector[Double], centers: Array[Vector[Double]]): Int = {
    val dist = centers.map(point => getDist(point, p))
    dist.indexOf(dist.min)
  }

  def main(args: Array[String]) {
    // K : 分类的个数；minDist : 中心点移动的最小距离，迭代终止条件；两个参数最好从参数传进来
    val K = 3
    val minDist = 0.01

    val conf = new SparkConf().setMaster("spark://node1:7077").setAppName("KMeans")
    val sc = new SparkContext(conf)
    sc.addJar("C:\\work\\examples\\SparkExamplesArchive\\out\\artifacts\\SparkExamplesArchive_jar\\SparkExamplesArchive.jar")
    sc.setLogLevel("WARN")

    // 1、读文件
    val lines = sc.textFile("hdfs://node1:8020/user/spark/data/iris1.data")
    val data = lines.filter(_.trim.size!=0).map(line=> Vector(line.split(",").map(_.trim.toDouble))).cache()

    // 2、获得K个随机的中心点
    val centerPoints = data.takeSample(withReplacement = false, K, 4)
    var tempDist = 1.0

    while(tempDist > minDist) {
      // 得到每个点的分类
      val closest = data.map(p => (getIndex(p, centerPoints), (p, 1.0)))

      // 计算新的中心点
      val pointStats = closest.reduceByKey((x,y) => (x._1+y._1, x._2+y._2))
      // collectAsMap很重要，保证新旧中心的对应关系
      val newcenterPoints = pointStats.map(pair =>(pair._1, pair._2._1 / pair._2._2)).collectAsMap()

      // 计算中心点移动的距离
      val dist = for (i <- 0 until K) yield {
        getDist(centerPoints(i), newcenterPoints(i))
      }
      tempDist = dist.sum

      // 重新定义中心点
      for ((key, value) <- newcenterPoints) {
        centerPoints(key) = value
      }
      println("distSum = " + tempDist + "")
    }

    // 打印结果
    println("Final centers:")
    centerPoints.foreach(println)
    sc.stop()
  }
}
```

1. 生产环境中: data.collect 不要用, 容易内存溢出!(数据量太大) 先采样再collect
2. 打印数组内容:  toBuffer / makeString
3. 重复利用的数据: data.cache()
4. 遍历数据集: map / foreach
5. 函数定义: map里. 

			不要定义一个函数, 把rdd传入,再把rdd带出来

6. reduceBykey(\_+_): 其中元素的类型不能变

7. rdd不允许嵌套: 

   val a = sc.makeRDD(1 to 100).map((_,1))

   val b = sc.makeRDD(1 to 100).map((_,1))

   a.map(x=>(b,x)) //必须嵌套的话,先把rdd转换为array

> WordCount优化

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-26 下午2.20.55.png)

```scala
val rdd1 = spark.read.textFile("data/spark.log")
val rdd2 = rdd1.filter(_.trim.length!=0).map(_.trim.toLowerCase)
val rdd3 = rdd2.map(_.replaceAll("[,.?;:!'\\(\\)]", ""))
val rdd4 = rdd3.flatMap(_.split("\\s+"))
val rdd5 = rdd4.map((_,1)).reduceByKey(_+_)
val stopword = List("the", "a", "in", "and", "to", "of", "for", "is", "are")
val rdd6 = sc.makeRDD(stopword).map((_, true))
val rdd7 = rdd5.outerLeftjoin(rdd6)
rdd7.filter(x=>x._2._2.getOrElse("")!=true).map(x=>(x._1, x._2._1)).sortBy(_._2, false).collect

```

> KNN算法

```scala
package cn.itxdl.scala.demo

import scala.io.Source
import scala.math.{sqrt, pow}

case class LabelPoint(val label : String, val feature : Array[Double])

object KNN {
  // 计算距离
  def dist(point1 : Array[Double], point2 : Array[Double]):Double = {
    val a = point1.zip(point2)
    sqrt(a.map(x=>pow(x._1 - x._2, 2)).sum)
  }

  // 根据最近的距离，得到类别
  def getCategory(testData:Array[Double], trainData:Array[LabelPoint], K:Int=10) : Array[(Int, String)] = {
    val values = for (data <- trainData) yield {
      (dist(testData, data.feature), data.label)
    }
    val category = values.sorted.take(K).map(x=>(x._2, 1)).groupBy(x=>x._1).map(x=>(x._2.size, x._1)).toArray.sortBy(x=>x._1).reverse
    category
  }

  def main(args: Array[String]): Unit = {
    val K = 20
    // 读样本文件，获取数据
    val trainFile = "C:\\Users\\wangsu\\iris.dat"
    val trainSource = Source.fromFile(trainFile)
    val lines = trainSource.getLines().toArray
    val trainData = for (line <- lines) yield {
      val value = line.split(",")
      LabelPoint(value.last, value.init.map(_.toDouble))
    }
    trainSource.close()
    trainData.foreach(x=>println("label = " + x.label + "; feature = " + x.feature.toBuffer))

    // 读预测数据文件
    val testFile = "C:\\Users\\wangsu\\iris1.dat"
    val testSource = Source.fromFile(testFile)
    val testLines = testSource.getLines().toArray
    val testData = for (line <- testLines) yield {
      line.split(",").map(_.toDouble)
    }
    testSource.close()
    testData.foreach(x=>println(x.toBuffer))

    // 计算类别
    val result = for (data <- testData) yield {
      getCategory(data, trainData, K)
    }

    // 打印结果
    result.foreach(x=>println(x.toBuffer))
  }
}
```

```scala
package Homework

import scala.io.Source
import scala.math._
case class gusData(data:Array[Double])
case class flowerData(key:String,value:Array[Double])
case class guessFlower(types:String,sampleData:Array[Double],space:Double) extends Ordered[guessFlower]{
  override def compare(that: guessFlower):Int = {
   this.space compare that.space
  }
}
object flower {
  var i = 0
  def main(args: Array[String]): Unit = {
    val lines = Source.fromFile("""/home/xdl/data/oldData.data""").getLines().toArray
    val data = for (line <- lines if (line.size > 0)) yield {
      val value = line.split(",")
      flowerData(value.last, value.init.map(_.toDouble))
    }
    val datas = data.toBuffer
    //for(element<-datas){println(element.key +"--->"+element.value.toBuffer)}
    //key --->flower.type value ----->flower.data   (parameter/参数)
    val guessDatas = Source.fromFile("""/home/xdl/data/Test.txt""").getLines().toArray
    val gussData = for (info <- guessDatas if (guessDatas.length > 0)) yield {
      val gussDataTmp = info.split(",")
      gusData(gussDataTmp.map(_.toDouble))
    }
    //for (x<-gussData)println(x.data.toBuffer)//println(Test.txt.data)(parameter/参数)
    val guessTypeProcess = for (x <- datas; y <- gussData) yield {
      guessFlower(x.key, y.data, sqrt(pow((x.value(0) - y.data(0)), 2) + pow((x.value(1) - y.data(1)), 2) + pow((x.value(2) - y.data(2)), 2) + pow((x.value(3) - y.data(3)), 2)))
    }
    // for (x<-guessTypeProcess) println("types:"+x.types+"data :"+x.sampleData.toBuffer+"space :"+x.space)
    val guessTypeGroup = guessTypeProcess.groupBy(_.sampleData).toArray
    val guessType = for (x <- guessTypeGroup) yield {
      x._2.sorted
    }
    guessType.map(x=>x.take(3).foreach(x=>println(x.types+"\t:"+x.sampleData.toBuffer)))

  }
}


```

```scala
package cn.xdl.spark

import org.apache.spark.{SparkConf, SparkContext}

import scala.math.{pow, sqrt}
case class LabelPoint(val label:String,val feature:Array[Double])

object KNN_TS {
  def main(args: Array[String]): Unit = {

    val K = 9

    val conf = new SparkConf().setAppName("KNN_TS").setMaster("spark://master:7077")
    val sc = new SparkContext(conf)

    val lines = sc.textFile("file:///home/xdl/iris1.dat")

    sc.addJar("C:\\Users\\Administrator\\IdeaProjects\\ScalaStudy\\src\\KNN_TS.jar")

   // val lines = sc.textFile("C:\\iris1.dat")


    val irisDataFilterBlank = lines.filter(line => line.trim.size != 0)


    val symbolDataRdd = irisDataFilterBlank.map(irisDataInfo => irisDataInfo.trim.split(",")).filter(irisData=>irisData.size == 5).collect().toBuffer

    val testDataRdd = irisDataFilterBlank.map(irisDataInfo => irisDataInfo.trim.split(",")).filter(irisData=>irisData.size == 4).collect().toBuffer


    val symbolData = symbolDataRdd.map(value=>{
      LabelPoint(value.last,value.init.map(_.toDouble))
    })

    //symbolData(0).feature.toBuffer

    val testData = testDataRdd.map(value=>{
        value.map(_.toDouble)
    })


    //计算两点之间的距离
    def  distance(point_one: Array[Double],point_two : Array[Double]) : Double ={
      val  pointZip = point_one.zip(point_two)
      sqrt(pointZip.map(x=>pow(x._1-x._2,2)).sum)
    }

    //根据最近距离，得到类别
    def getCategory(testData : Array[Double],symbolData : Array[LabelPoint],K : Int = 9) : Array[(Int,String)] = {
      val values = for(data <- symbolData) yield {
        (distance(testData,data.feature),data.label)
      }
      val category = values.sorted.take(K).map(x=>(x._2,1)).groupBy(x=>x._1).map(x=>(x._2.size,x._1)).toArray.sortBy(x=>x._1).reverse
      category
    }

    // 计算类别
    val result = for(data <- testData) yield{
      (getCategory(data,symbolData.toArray,K),data)
    }

    // 打印结果

     result.foreach(x=> {
       println((x._2.toBuffer,x._1.toBuffer))
     }
       )
    
   // testData.foreach(x =>println(x.toBuffer))  //测试

  }
}

```

```scala
import org.apache.spark.{SparkConf, SparkContext}
import breeze.linalg.Vector

object SparkKNN2 {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName("SparkKNN").setMaster("spark://master:7077")
    val sc = new SparkContext(conf)
    //jar包所在位置
    sc.addJar("/home/xdl/IdeaProjects/HelloWOrld/out/artifacts/HelloWOrld_jar/HelloWOrld.jar")
    sc.setLogLevel("WARN")
    // 1、读文件，获取数据,并对数据进行去 空格,空行.
    val lines = sc.textFile("hdfs://master:9000/iris1.dat").filter(x=>(x.trim.size!=0)).map(_.split(","))
    //2、将数据分成两类。一类是已知分类的数据；一类是待求分类的数据
   
    //获取到没有类型的点
    val testPoint = lines.filter(_.size==4).map(x=>{
      (Vector(x(0).toDouble,x(1).toDouble,x(2).toDouble,x(3).toDouble))
    }).cache().collect()
    //获取到有类型的点
    val oldPoint = lines.filter(_.size==5).map(x=>{
      (x(4),((Vector(x(0).toDouble,x(1).toDouble,x(2).toDouble,x(3).toDouble))))
    }).cache()
     /*    创建广播变量
     有疑问,collcet是收集回driver所在的节点其他节点就没有了吗?
     还是只是起到收集回来便于打印,其他节点数据也存在
    val broads=sc.broadcast(testPoint)
    遍历未知类型的数据的集合
    val result = broads.value.map(x=>{
    */
    //遍历未知类型的数据的集合
    val result = testPoint.map(x=>{
      //计算距离
         val info =  oldPoint.map(y=>{
                ((x-y._2).t*(x-y._2),y._1)
         })
      //选取九个最近的点做wordcount
         val info1 =info.sortByKey().take(9).map(map1=>(map1._2)).groupBy(gb=>gb).map(map2=>(map2._1,map2._2.size))
            (x,info1)
    })
          //打印结果
          result.foreach(x=>(println(x._1,x._2.mkString(","))))
    sc.stop()
  }
}
```

> 打jar包

集群模式下:

工程的out - artifacts-spa…. 是jar包的路径

spark submit 提交时:

setMaster可以去掉, addjar可以去掉

addjar仅仅是在集群模式里用

打jar包:FIle-project structure-jar-------

每次rebuild一下,重新刷新jar

---

conf.setJars 可以加多个jar路径

Linux下: find . -name *.jar

家目录(任意路径)下: spark-submit  - -class 包名+类名 jar包路径 // 集群模式