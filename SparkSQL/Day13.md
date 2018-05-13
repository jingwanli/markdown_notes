## Spark复习

rdd缺点:

1. 不停的new对象, 增加了gc的负担

2. 不支持细粒度的更新.

   (hadoop的产品也基本都不支持, 除了hbase)

checkepoint:

写的是HDFS(高容错)

斩断依赖!

用的时候一定要设置一个目录!

cache,persist,checkpoint都是transformation.

有action才可以触发!

作业调度:(都在spark context)

DAGScheduler

taskScheduler

schedulerBackend:

----

2.0以前

sparkcontext

sqlcontext

hivecontext

streamingcontext

2.0以后

sparksession(sparkcontext/sqlcontext/hivecontext)

streamingcontext

---

@transient : 注解: 后面的变量在传递的时候不用被序列化

spark-shell里可以写的,有时候在idea里会报错:

```scala
//创建DataSet
import org.apache.spark.sql.SparkSession

object SparkSQLDemo{
    def main(args:Array[String]):Unit={
        val spark = SparkSession
        .builder
        .appName("test")
        .master("local")
        .enableHiveSupport()
        .getOrCreate()
 	case class 				Person(name:String,age:Int,height:Double)
	val seq1 = 	Seq(Person("andy",9,135),Person("tom",12	,140),Person("jack",10,130))
	import spark.implicits._
	val ds1 = spark.createDataset(seq1)
	ds1.show()       
    }
}


```

```scala
//创建DataFrame
val seq1 = Seq(("jack",28,184),("tom",10,144),("andy",16,165))
//方式1:
val df1 =  spark.createDataFrame(seq1).withColumnRenamed("_1","name1").withColumnRenamed("_2","age1").withColumnRenamed("_3","height1")
//方式2:
val df1 = spark.createDataFrame(seq1).toDF("name","age","height")
//显示
df1.show
```

```scala
//rdd转化为DF
import org.apache.spark.sql.Row
import org.apache.spark.sql.types._

//方式1:有可能转不过去!(2.0以后才有!)
val arr = Array(("jack",28,184.0),("tom",10,144),("andy",16,165.0))
val rdd = sc.makeRDD(arr)
rdd.toDF
//方式2:
val rdd1 = rdd.map(x=>Row(x._1,x._2,x._3))
//false代表字段不可以为空
val schema = StructType(List(StructField("name",StringType,false),StructField("age",IntegerType,false),StructField("height",DoubleType,false))
val df = spark.createDataFrame(rdd1,schema)
df.show //注意: 前面的height要跟.0, 无法自动类型推断
//方式3                       
val arr = Array(("jack",28,184.0),("tom",10,144),("andy",16,165.0))    
val rddToDF= sc.makeRDD(arr).toDF("name","age","height")                       
                        
```

```scala
//rdd转DataSet
case class Person(name:String,age:Int,height:Double)
val arr = Array(("jack",28,184.0),("tom",10,144.0),("andy",16,165.0))
val rdd2 = spark.sparkContext.makeRDD(arr).map(f=>Person(f._1,f._2,f._3))
val df = rdd2.toDF
val ds = rdd2.toDS
//排序
ds.orderBy(desc("name")).show
```

```scala
//从ds取出rdd
ds.rdd
```

CSV:逗号分隔的文件

用其他分隔的话,每一行当做一个字段.

 csv中,可以把表头加上.

 ```scala
//方式1:缺省的
val df = spark.read.csv("data/sql1.txt") 
//方式2:
import org.apache.spark.sql.types._
val schema2 = StructType(StructField("name"),StringType,false)::StructType(StructField("age"),IntegerType,false)::StructType(StructField("height"),IntegerType,false)::Nil)
val df7 = spark.read.options(Map(("delimiter",","),("header","false"))).schema(schema2).csv("file:///home/spark/t01.csv") //不指定schema的话,读出来的就都是string
df7.show   //读出来的是DF
df7.printSchema
//方式3:
spark.read.options("header","true").option("inferschema","true").csv("file:///home/spark/t01.csv") //可以不指定schema,因为指定了自动类型推断 
//option可以跟多个
 ```

![](/Users/lijingwan/Documents/img/屏幕快照 2018-05-02 下午3.56.13.png)













