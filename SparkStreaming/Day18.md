## Spark Streaming

rdd是静态的.对输入进行切片,如周期是1s,将1s进行切片,然后再转化为RDD.

storm是面向消息的.

但是spark Streaming不是, 是针对rdd的.(若干条消息组成一个rdd)

周期性的触发一个作业(而不是action了)

receiver在后台是独占资源的.(receiver用于接受数据),所以直接写local不好使(直接写local意味着启动了一个核,只能接受数据,不能处理数据) local_2

spark的调度的成本太高,所以收到一批才处理.

storm的延迟:ms级.延迟更低

spark streaming: 亚秒级.吞吐量更大.

kafka是scala开发的.

storm:clojure语言开发的: jvm上的Lisp, 很多扩展是用java完成的.

​	  Lisp: 第一个函数式编程语言

jstorm: java开发的

---

DStream: 离散化的流(有时也成为DS)

两条路:1. 把数据接收过来 ,切分

​	    2. 处理数据

互不影响.

取数据:10s

是过一段时间就取一次,到了10s,将所有的数据汇总起来.!!不是每10s取一次(数据量太大了)

窗口操作: 10s取一次数,要统计1min的数据情况:Window

> 数据源1:

文件系统

TCP

rdd: 自己写好的rdd,给DS

注:机器学习在向流式的处理过渡

---

idea 写sparkcore 打包的时候: master 和 addjar要去掉!!(master不要写死了!)

写在程序里的级别,是no1

---

建议不要写local[*],有可能只分配了一个core,那么只能接受数据,无法处理.(至少写local[2])

rdd: key-value: key不能是数组.

因为数组算的是地址,要算哈希值.

### 输入流

```scala
object StreamingFileWordCount{
    main...
    //ssc可以使用sc,或者sparkconf两种方式
    
    val conf = new SparkConf().setAppName("").setMaster("local[2]")
    val ssc = new StreamingContext(conf,Seconds(10))
    //监控的是一个目录!处理新到的!
    val lines = ssc.textFileStream("file:///home/xdl/data/")
    val words = lines.flatMap(_.split("\\s+"))
    val wordCounts = words.map((_,1)).reduceByKey(_+_)
    //输出:
    wordCounts.print()
    
    ssc.start()
    ssc.awaitTermination()
  //mv 不好使.(时间戳没变!)
  // cp可以.自己新建可以.(根据时间戳)
}

//使用hdfs的时候,local[2] 可以设置为local(来源是文件系统,不会监控.和网络的不同)特例!
ssc.sparkContext.setLogLevel("WARN") //sc自动创建!
之前有的文件,不会处理.
必须重新拷贝!只处理从启动任务到10s中的文件.
```

```scala
object StreamingSocketWordCount{
    main...
    //修改的部分:必须用local[2]
    val lines =  ssc.socketTextStream("node2",4444)
    
}
//命令行:
yum install nc (root权限)
nc -lk 4444
输入: hello
hello word
nc没有跟参数,只能用于本地

//数据源, 可以写一个程序
object SocketLikeNC{
    main...
    val words = "hello spark hello hadoop hello java hello scala hbase kafka flume hive storm".split("\\s+")//转换为数组
    val n = words.size
    val host = "node2"
    val port = 4444
    val random = new Random()
    val ss = new ServerSocket(port)
    val socket = ss.accept()
    println("starting...." + socket.getInetAddress)//拿到监听的地址
    while(true){
        val out = new PrintWriter(socket.getOutputStream)
        out.println(words(random.nextInt(n)))
//得到2个单词        out.println(words(random.nextInt(n))+ " " words(random.nextInt(n)))
 out.flush()
 Thread.sleep(500)
    }   
}


```

```scala
//RDD队列流
object RDDQueueWordCount{
    
    main...{
    //修改:
    val rddQueue = new Queue[RDD[Int]]
        //true表示只处理一个rdd
        //false可以处理多个
        //默认是true
    val quequeStream = ssc.queueStream(rddQueue,false)
        /
    val ds1 = queueStream.map(x=>(x%10,1))
    val ds2 = ds1.reduceByKey(_+_)
    
     ds2.print()
     ssc.start()
   //  ssc.awaitTermination()
     //未完! 下面还要定期的生成rdd数据!
        for(i<-1 to 6){
            //
            rddQueue.synchronized{
                rddQueue += ssc.sparkContext.makeRDD(1 to 100,4)
            }
        Thread.sleep(2000)
        }
        //流式作业是不需要关闭的,希望其要持续进行
        ssc.stop()
       
       
    }
}

```

### 条件子句

```scala
sum() over (partition by ... order by... rows/range between...and ...)
over后面的有些资料叫窗口子句
这里把rows/range....and 叫窗口子句
Windowing Specifications in HQL/Spark SQL
1. A Window Frame that has only the start boundary, then it is interpreted
as:
Between start boundary and current row. 译:窗体子句如果只有开始边界，那么它会被默认解释成从开始边界
到当前行。

2. A Window Specification with an Order Specification and no Window
Frame is interpreted as:
Range between unbounded preceding and current row. 译:如果只有排序子句而没有窗体子句，则窗体子句被解释翻译成一
个 Range，范围是从第一行到当前行。

3. A Window Specification with no Order and no Window Frame is
interpreted as:
Rows between unbounded preceding and unbounded following. 译:如果既没有排序子句和窗体子句，则窗体子句被默认解释成一个
Rows，范围是从第一行到最后一行。

4. If there is no Partition Specification all rows belong to the same Partition.
译:如果没有分区子句，那么所有行都属于一个分区。

```

### Day17作业

```scala
//hive中使用string! hive不区分大小写
1. hive中create table () stored as parquet //
//parquet压缩率非常的高 大概是80%,速度快!
create table hive.user1(uid String,itemid )
2. spark sql insert
3. 

object SparkSQL1{
    main...
    val spark = SparkSession.builder().
    master: spark://master:7077
    ......
    .....
 //列存储的,不要对它指定分隔符等.!!
    //hive缺省格式是text
    
    spark.sparkContext.setLogLevel("WARN")
    
   val userDF = spark.read.option()... 自动类型推断要加!.reparation(2)
    //不重新分区的话,会出现一堆小文件(因为压缩比很高!)
   // userDF.printSchema 在shell里试
    //看到时间戳被解析成了string
    userDF.createOrReplaceTempView("t1")
    
 sql("insert overwrite table hive.user select * from t1")   
    //过滤字段 用drop
    //建表的时候,把那个字段不要建!用select过滤
    方法2: 用drop过滤:
    userDF.drop("user_geohash")
    //在读DF时候直接.也可以!!!
     
    //默认路径:hadoop fs -ls  /user/hive/warehouse/hive.db/user1
//方式2:hdfs dfs -ls /user/hive/warehouse/hive.db/user1
    spark.stop()
}


内部表:删除表后,数据也删掉了!
drop table hive.user1;
delete table hive.user2;(DML)
truncate table hive.user;(DDL)速度很快
对于每个表: 都有一个 highwater
delete : oracle里, 发出此语句,如果没有提交,可以回滚
mysql隐式提交, oracle:隐式不提交. 
truncate 直接把高水位移到0.(数据删除了找不回来!)

------------------
import org.apache.spark.sql.functions._
import spark.sql
val df = sql("select uid from hive.user1")
//只查uid.!列存储,可以提高速度!
df.cache()
//找出不重复的id有多少
df.select(countDistinct("uid")).show
val a = df.dropDuplicates("uid").count //返回值为long,不能show!
println(s"a = $a")

//找出uid出现最多的20个用户
import spark.implicits._
df.groupBy("uid").count.orderBy($"count".desc).take(20)

//找出item_type出现最多的20个
sql("select item_type from hive.user1").groupBy("item_type").count.orderBy($"count".desc).take(20)

//各种type出现的次数
sql("select ")

spark.stop()
```



![](/Users/lijingwan/Documents/img/屏幕快照 2018-05-07 下午3.25.53.png)

receive 接收到数据后, 先存入内存

MEMORY_AND_DISK._2 :是序列化的数据,且存2份副本!















