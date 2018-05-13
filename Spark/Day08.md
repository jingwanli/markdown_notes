

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-23 上午8.47.03.png)		

1. standalone, yarn, mesos

2. 粗粒度:一开始就可以申请到资源, 在任务的执行过程中持有此资源 (yarn, standalone支持粗粒度)

   细粒度:

3. Application: 应用程序

   driver: 管理应用程序的组件.& 将work发送到executor….

   ​	   包含了main方法,而且初始化了context

   executor: 负责task的执行

   job: actio触发的job.shuffle将job分成两个stage

4. $SPARK_HOME/config

   spark.cores.max: 每个application的最大核心数

   spark_worker_cores:

   spark_worker_memory:

5. local: 所有的东西都在一个jvm中, 不需要启动服务(与hdfs无关)

   伪分布式: 所有的进程都在一台机器上 local cluster[3个参数:第一个多少个worker, worker里多少个core, slave里有多少个memory]

   分布式集群: 进程分布在整个的cluster上

6. start-slave.sh 

7.  spark submit -master spark://node0:7078 -class WordCount ./MyApp.jar

8. standalone( master  worker)

   yarn(yarn start-all.sh /// ??)

9. client: driver 不在我的集群上: client,主要用于测试

   cluster: driver在cluster上,没有返回信息

10. Driver和executor和master有关系!

    ![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-23 上午9.11.13.png)



-----

idea中: ctrl+N 输入RDD	
宽依赖:有shuffle

窄依赖	

创建RDD:

 从已存在的集合中创建: 用于测试(创建一个数组, list, range变成RDD)

从外部存储系统中创建: 有接口

主要的算子分为两大类:

1. Transformation: 得到一个新的RDD
2. Action: 触发Job

driver负责将task发送到executor上执行

collect: 把worker上的数据都收集起来

尽量不要用 : 用的时候,先sample

可以使用 takeSample

---

------上课内容-----

action触发job,几个action就有几个job

sample: 采样(百分比)

takesample: 直接确定了采样的个数(不是百分比了)

调试简单的方法: 1.打印(用local模式!)	2. 写日志 	

kill -15 4288: -15意思是发送了一个终止的信号,可能杀不掉(如果有工作没做完,会做完退出)

kill-9 4288:强杀

打印的时候,可能本地的机器上没有数据, 可能在worker中执行.——> 很难查看!

print语句查看可以用 : 

spark-shell - -master local

#### 算子示例

1. 找到包含特殊字符的行(error) & 删除空行

val  rdd1 = sc.textFile("file:///home/spark/a.log")

val rdd2 = rdd1.filter(line=>line.trim.size!=0)

val rdd3 = rdd2.filter(line=>line.trim.contains("ERROR")) //头尾的空格去掉

方式二:

hdfs dfs -put a.log data/

val rdd1 = sc.textFile("data/a.log")

在idea里要写全路径: "hdfs://node:8020/user/spark/data/a.log"

2. 找到哪一行包含的词最多

…..前2步不变

val rdd3 = rdd2.map(line=>line.split("\\\s+").size)

rdd3.collect

rdd3.max

方式2:(错误!!!!)

val rdd4 = rdd3.reduce((x,y)=> if(x>y) x else y) 

reduce是action 直接返回结果 

val rdd5 = rdd3.reduce((x,y)=>{ //map只有单个参数!

val maxInt = if(x>y) x else y

val minInt= if(x<y) x  else y

(maxInt.toString() + minInt)

})

方式3: rdd3.stats (全转化成浮点型)

限制: 只能用于单列!

3. 集合的交并差

val rdd1 = sc.parallelize(1 to 20)

val rdd2 = sc.parallelize(10 to 30)

rdd1.union(rdd2).collect //不加collect不能打印! 

不去重!

rdd1.intersection(rdd2).collect //数据顺序不保证!(分区)

val rdd3 = rdd1.substract(rdd2).collect

###### intersection的实现:(源码)

先将2个集合中的值映射为: (V,null)

然后rdd1.cogroup(rdd2)

然后进行过滤(过滤条件是模式匹配!!)

4. 将多个RDD中同一个Key对应的value组合到一起

val names= sc.parallelize()

val nameAndType = names.cogroup(types)

nameAndType.collect.foreach(println)

### Pair RDD

> 什么是Pair RDD

创建方式:

val  lines = sc.textFile("data/a.log")

val pairRDD = lines.flatMap(_.split("\\\s+")).map(\_,1)

pairRDD.reduceByKey(\_+_) —>返回的是RDD!

==注意!!!== reduce是action, reduceByKey是转换!

==join也必须是key value的!!!!!!!必须有key==

源码:

隐式转换要在object里面找

(reduceByKey 在ctrl+n找不到. 是reduce隐式转换成的!)	

> reduceByKey	

> groupByKey

两者的区别:

reduceByKey 效率高: 存在map端的combiner

groupByKey: 不存在map端的combiner, shuffer的时候传的数据量大!

能用reduceByKey的时候不要使用groupByKey

> keys

> values

> sortByKey 

返回一个按key排序的RDD(默认升序)

看视频!!

RDD是分区的, 只读的(不可变), 有依赖,可持久化

### RDD控制算子:

cache, persist, checkpoint (分到transformation类里)

持久化的单位: partition

RDD控制算子都是懒执行的

```scala
val list = List("")
```

cache将计算结果写入不同的介质:

- 内存:指的是JVM的堆内存
- 堆外内存: 不受JVM控制的内存(因为要保持不变性,需要不停的new对象)
- 磁盘: 写硬盘

rddn.cache(): 在第一个action执行的时候才执行!

重复性的计算, 建议cache! 如hdfs读数据.(否则每次都要从hdfs中读数据) 看具体情况. 如果hdfs读数据后,直接filter, 要从filter后的地方进行缓存. (cache的时候,只是做标记,第一个action后才缓存)

被缓存了不代表永远在内存中, 有可能被换出内存(内存不够用,被丢弃),只是在内存允许的情况下才被放进去. 数据是分片的,所以有可能被换出一部分(也有可能部分收益)

persist : 可以指定持久化级别参数. 是放入jvm还是堆外, 是写1份还是写2份(最多2份), 是序列化的还是反序列化的.(可以进行组合)

cache( ) == persist ( MEMORY_ONLY )

persist(MEMORY_AND_DISK): 优先写内存,如果内存够不写磁盘.(先被换出,因为默认可以写磁盘)

> Storage的级别

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-23 下午2.21.49.png)

最后一项: 本地放1份,另外的节点放一份.

### RDD分区

目的: 设置合理的并行度, 提高数据处理的性能.

设置的不合理网络io可能是瓶颈(hadoop中: 磁盘io,网络io两个瓶颈)

原则: 

1. 尽可能使得分区的个数, 等于集群==核心==数目
2. 尽可能使同一个RDD不同分区内的记录的数量一致

>  获取分区数的api:

1. rdd1.getNumPartitions
2. rdd1.partition.size

> 手动指定分区

val rdd1 = sc.makeRDD(arr, 5 ) //5是分区数设置

对于hdfs文件, 考虑的是块的个数

对于parallelize(makeRDD)方法,默认情况下, 分区的个数会受配置参数: spark.default.parallelism的影响, 该参数(后面都可以手动改)也用于控制Shuffle过程中默认使用的任务数.( 与Shuffle以后的数据形式也有关)

local 默认等于cpu分区的总个数

standalone: 取集群中的所有核心数的总和或者2,取较大值

Mesos:8个

对于textFIle:

1. 每个HDFS的分片的个数
2. ​

local: 如果rdd1手动分区变化后需要rdd2 接一下, rdd1是不变的!!

==同时求最大值最小值==

val arr = (1 to 1000).toArray

val rdd1 = sc.make

### Word Count

val lines = sc.textFile("data/a.log") //lines是rdd

val  words = lines.flatMap(_.split("\\\s+"))

val pairRDD = words.map(_,1)

val result = pairRDD.reduceByKey(\_+_)

result.collect

result.toDebugString

//看视频!!!!

###在idea中写word count:

```scala
object WordCount {
    def main(args:Array[String]):Unit={
        //创建sparkContext
        val conf = new SparkConf().setMaster("local").setAppName("WordCount")
        val sc = new SparkContext(conf)
        sc.addJar("")
        //读文件, 创建RDD,对RDD进行转换
        sc.setLogLevel("WARN")
        val lines = sc.textFile("hdfs:?/node1:8020/user/spark/data/a.log")
        val pairRDD= lines.flatMap(_.split("\\s+")).map((_,1)).reduceByKey(_+_)
//输出结果
        pairRDD.collect().foreach(println_)
    //关闭sparkContext
        sc.stop()
    //打包,在集群中运行
    }
    
}
```

