## 作业

Flume:

x.event.getBody.array().toBuffer

方式1:百度  方式2:看源码(看不懂去找其父类的源码)

---

> 黑名单过滤

DStream的transformation比RDD少

可以通过transfrom—>转化成对rdd的操作. 可以写spark core ,spark sql

ConstantInputDStream: 常量, 用于测试!

sparkSQL: 首先要有SparkSession!!

```scala
////程序有误!!!
ds1.transform(rdd=>{
    val spark = SparkSession.builder().config(ssc.sparkContext.getConf).getOrCreate()
    
    import spark.implicits._
    val df1 = rdd.toDF("num")
  	//DSL和SQl都可以
   // df1.groupBy("num")
  	df1.createOrReplaceTempView("t1")
    spark.sql("select num, count(*) from t1 group by num").rdd
})
```

## Dstream有状态转换操作

### 滑动窗口

时间窗口: 

1. 多少时间取一次,并转换成ds//最小的时间

		2. 不仅仅是计算当前批次的,定义一个窗口的长度! 
		3. 窗口过多长时间计算一次(滑动的时间)

//2,3必须是1的整数倍!

//之前的数据: checkpoint!(建议不放在本地, 放在hdfs上, 生产环境: hdfs). 我们只需要设置一个checkpoint的目录, 框架自动完成

```scala
object 
```

#### 滑动窗口转换操作

reduceByWindow

reduceByKeyAndWindow提供了2中算法的实现:

1. window1 = time1 + time2 + time3

2. 利用了之前的数据! (高效) 利用了加法的逆操作

   (重复的越多, 算法的效率越高!)

updateStateByKey :旧版本 没有windows,(比如一直求和),一直统计这个状态

mapWithState(1.6版本后) :效率高

```scala
//[Int]--value的泛型
def updateFunc(currValues:Seq[Int],preValues:Option[Int]):Option[Int]={
    val currentCount=currValues.sum
    val prevCount = preValues.getOrElse(0)
    Some(currValues + pre)
}
val result = words.updateStateByKey[Int](updateFunc)
```

方法和函数的区别:

绝大多数没有区别

定义格式不同: 方法 def,

​			函数是1等公民,都可以用(val , def)

一般隐式转换都可以转过去

def f1(a:Int, b:Int) = a+b //方法

val f2 = f1 //f2是函数

<<scala编程>>

mapWithState

stateSnapShots(): 加此方法,只打印最终结果!

DStream 处理结果: 一般是存在db里的!

如果需要再次处理,可以放在hdfs上

通过RDD怎么写到数据库中?( 以前是通过sql )

```scala
val stateDstream = ......print(100)

//方法1
//遍历rdd 可以用map 或者 mapPartitions(这个比较好)
//这里可以重新分区一下(避免小文件)
//foreachPartition 按分区来,参数是迭代器
stateDstream.foreachRDD(rdd=>{
    rdd.foreachPartion(iter=>{
        saveAsMysql(iter) //用函数封装
    })
})

object DBUtils{
    def wordcountSaveAsMySQL(iter:Iterator[String,Int]):Unit={
        val url = "jdbc:mysql://master:3306/spark"
        val user = "hive"
        val password = "hive"
        var conn:Connection = null
        var stmt:PreparedStatement = null
        try{
            conn = DriverManager.getConnection(url,user,password)
            iter.foreach(record=>{
                val sql = "insert into wordcount values(?,?)"
                stmt = conn.prepareStatement(sql)
                stmt.setString(1,record._1)
                stmt.setInt(2,record._2)
                stmt.executeUpdate()
            })
        }catch{
            case e:Exception => e.printStatckTrace()
        }finally{
            if (stmt!=null) stmt.close()
            if (conn!=null) conn.close()
        }
    }
    
    main...{ 
     //测试用 
        ----------------
       val arr = Array(("spark",10),("hive",15),("hbase",10))
        val rdd = ssc.sparkContext.para
		val ds = new Constant
        ---------------
        val arr = Array(("spark,10"),("hello",15),("hbase",15))
        val rdd = ssc.sparkContext.parallelize(arr)
        val ds = new ConstantInputDStream(ssc, rdd)
        /////////////
        ds.foreachRDD((rdd,time)=>{
            rdd.foreachPartition(iter=>
                                
        })
        
    }
}
//方法2


```











