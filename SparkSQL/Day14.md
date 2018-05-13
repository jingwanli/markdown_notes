##回顾

Row: 泛化的无类型的JVM object

取值方式:

row1(0)

row1.getInt(0)

row1.getAs[Int\](0)

schema , row ——>DF

case class ——> DS

DS创建: 从集合中来的方式,主要用于测试

##### DF创建

```scala
//从集合创建:
val seq1 = Seq(("Jack",28,184),("Tom",10,144),("Andy",16,165))
//方式1:
val df1 = spark.createDataFrame(seq1).withColomnRenamed("_1","name1"..withColomnRenamed("_2","age1").withColomnRenamed("_3","height1")
df1.orderBy(desc("age1")).show(10)
//方式2:简单!2.0.0的新语法!
val df2 = spark.createDataFrame(seq1).toDF("name","age","height")

//从rdd创建:
import org.apache.spark.sql.Row
import org.apahce.spark.sql.types._
val arr = Array(("Jack",28,184),("Tom",10,144),("Andy",16,165))
val rdd1 = sc.makeRDD(arr).map(f=>Row(f._1,f._2,f._3))
val schema = StructType(StructField("name",StringType,false)::StructField("age",IntegerType,false)::StructField("height",IntegerType,false)::Nil)
val rddToDF = spark.createDataFrame(rdd1,schema)
rddToDF.orderBy(desc("name")).show(false)                                                        
```

##### RDD / DF / DS转换

```scala
val rdd2 = spark.sparkContext.makeRDD(arr).map(f=>Person(f._1 , f._2 , f._3))
val ds2 = rdd2.toDS()
val df2 = rdd3.toDF()
ds2.orderBy(desc("name")).show(10)
df2.orderBy(desc("name")).show(10)

val ds3 = spark.createDataset(rdd2)
ds3.show(10)
```

![](/Users/lijingwan/Documents/img/屏幕快照 2018-05-02 下午5.00.23.png)

![](/Users/lijingwan/Documents/img/屏幕快照 2018-05-02 下午5.03.49.png)

机器学习两个包:

MLlib—>底层 RDD(维护期)

ml ——>底层 DF DS

Phython用不了DS. 只能用到DF

flatMap 里面必须是个集合!

![](/Users/lijingwan/Documents/img/屏幕快照 2018-05-02 下午6.33.23.png)

#### 三者之间的转换

1. DataFrame/Dataset转RDD:

   val rdd1 = testDF.rdd

   val rdd2 = testDS.rdd

2. RDD转DataFrame

   //一般用元组把一行的数据写在一起, 然后在toDF中指定字段名

   import spark.implicits._

   val testDF = rdd.map{line=>

   ​	(line._1, line.\_2)	

   }.toDF("col1","col2 ")

3. RDD转Dataset

   //核心就是要定义case class

   import spark.implicits._

   case class Coltest( col1:String, col2:Int )

   val testDS = rdd.map{

   ​	line=>Coltest(line._1,line.\_2)

   }.toDS

4. Dataset 转 DataFrame

   //把case class封装成Row

   import spark.implicits._

   val testDF = testDS.toDF

5. DataFrame 转 Dataset

   //每一列的类型后, 使用as方法( as方法后面还是跟的case class, 这个是核心 ),转成Dataset

   import spark.implicits._

   case class Coltest ….

   val testDS = testDF.as[Coltest]

> 特别注意

在使用一些特殊操作时, 一定要加上 import spark.implicits._ 不然toDF, toDS无法使用

```scala
val arr = Array(("Jack",28,184),("Tom",10,144),("Andy",16,165))
val rdd1 = sc.makeRDD(arr)
val df = rdd1.toDF()
val df = rdd1.toDF("name","age","height")
case class Person(name:String,age:Int,height:Int)
val ds = df.as[Person]
```

---

## Day14

读文件读出来的都是DF

phython用不了DS, 只能用DF

### Action

1. count —> df1.count(按行计算)

2. catalog

   spark.catalog.listFunctions.show(1000)//不加参数缺省显示20行

   spark.catalog.listFunctions.show(1000,false) //false表示完整显示

3. df1.collect :返回的是Array

   df1.collectAsList : 返回的是List,List里面是row

4. df1.take(2) //take取出来的是Row的Array

5. df1.first

6. df1.head(2)

7. reduce

   case class Person(name:String, age:Int, height:Int)

   val seq1 = Seq(Person("Jack",28,184),Person("Tom",10,144),Person("Andy", 16,165))

   val ds1 = spark.createDataset(seq1)

   ds1.reduce((x,y)=>Person("sum",x.age+y.age,x.height+y.height))

   //结果: Person = Person(sum,54,493)

### 结构属性

df1.columns: 查看列名

df1.dtypes : 查看列名和类型

df1.explain() :参看执行计划(计划分逻辑执行计划和物理执行计划)

df1.col("name") : 获取某个列

df1.printSchema : 常用

### Transformation

1. val df2 = df1.map(x=>x.getInt(0)) //取出第一列 ,取出的结果是DS

   方式2: 

   val  df2 = df1.map(x=>x.getAs[Int]\(0)) //结果仍然是DS

2. flatMap

```scala
case class Peoples(age:Int, names:String)
val seq1 = Seq(Peoples(30,"Tom,Andy,Jack"),Peoples(20,"Pand,Gate,Sundy"))
val ds = spark.createDataset(seq1)
val ds2 = ds.flatMap(x=>{
    val age = x.age
    val people = x.names.split(",").map(y=>Peoples(age,y))
    people
})
//结果形式: 30 Tom
//		   30 Andy
//		   30 Jack
//		   20 Pand
//		   20 Gate
//		   20 Sundy
//注: flatMap: 一定是将类似集合的东西压平了..
```

3. filter , where

   df1.filter("sal > 2000").show

4. randomSplit (RDD也可以用,在这里将DF, DS按给定的参数分成多份)

```scala
val df2 = df1.randomSplit(Array(0.5,0.6,0.7))
//结果为:Array里面套着DS.相加>1 数据会有重复
//数据量特别小的时候,比例并不是特别的准
df2(0).count
df2(1).count
df2(2).count
```

5. limit

   val df2 = df1.limit( 5 ) //有点像rdd中的takeSample

6. distinct 按行去重

   df1.union(df1).distinct.count //返回值为DS

7.  dropDuplicates 按列去重

   df1.dropDuplicates("deptno").count //3行,前面随机的

   df1.dropDuplicates( "mgr", "job" ).count

8. describe

   df1.describe().show //describe以后的结果是DF

   求均值,最大值,最小值,方差…..后的结果显示,相当于做了一个统计信息

   也可以指定表列: df1.describe("sal").show

##### 存储相关

9. 与RDD一致

   使用MEMORY_ONLY时,要导包

   使用Checkpoint,要设置路径

```scala
import org.apache.spark.storage.StorageLevel
spark.sparkContext.setCheckpointDir("hdfs://node1:8020/checkpoint")
df1.show()
df1.checkpoint()
df1.cache()
df1.persist(StorageLevel.MEMORY_ONLY)
df1.count()
df1.unpersist(true)
```

#####select相关

10. 列的多种表示方法: 5种:

    - " "
    - $" "
    - '
    - col( )
    - ds(" ") //ds表示表名

    ![](/Users/lijingwan/Documents/img/屏幕快照 2018-05-02 下午10.42.13.png)

正确写法: df1.select( $"ename" , \$"sal" + 100).show

​		df1.select( 'name, 'sal+100 ).show

//注意: 两个$都要加! 不准混用! 

11. 删除列

    val df2 = df1.drop("comm", "sal")

12. 修改列名 withColumn

    val df2 = df1.withColumn("sal", $"sal"+1000 ) //修改后的列名在前面

    val df2 = df1.withColumnRenamed("sal","newsal").show

13. where子句

    df1.where("sal > 1000").show

    df1.where($"sal" >1000).show

14. cast类型转换

    方式1:

    df1.selectExpr("cast(empno as string)").printSchema

    //结果: empno:string( nullable = true )

    方式2:

    import org.apache.spark.sql.types.StringType

    df1.select('empno.cast(StringType))

15. where操作

    df1.filter("sal>1000 and job == 'MANAFER' ").show

16. groupby

    - spark.sql("select count(*) from t1 group by deptno").show
    - df1.groupBy("deptno").count( ).show
    - df1.groupBy("deptno").agg("sal"->"sum", "sal"->"count" ).show
    - df1.groupBy("deptno").agg(max("sal").as("max1"),count("sal"), min("sal")).show

---

回顾Day14

val ds1 =spark.read.textFile("data/spark.log")

> 作业:

练习：采用Datasets操作，实现WordCount实例，并且按照count值降序显示前50行数据，其中word转换成小写，去除标点符号，去除停用词。考查点：

1、spark读取文件

2、dataset转换操作、聚合操作

重点在：数据清洗，去除标点、去停用词等。需要自己自定义停用词集合 和 标点符号集合

3、dataset排序及显示

要求：

使用这种方式读数据：val ds1 =spark.read.textFile("data/spark.log")

分别在spark-shell、IDEA中实现

```scala
package sparkSQL

import org.apache.spark.sql.SparkSession

object wordCount {
  def main(args: Array[String]): Unit = {
    val spark = SparkSession
      .builder()
        .appName("wordCount")
          .master("local")
          //.master("spark://master:7077")
        .enableHiveSupport()
      .getOrCreate()
   val sc =  spark.sparkContext
     sc.setLogLevel("WARN")
   // sc.addJar("/home/xdl/IdeaProjects/HelloWOrld/out/artifacts/HelloWOrld_jar/HelloWOrld.jar")
    val words = spark.read.textFile("hdfs://master:9000/a.txt")
    import spark.implicits._
		//对数据进行去开头末尾的空格,空行,将数据整合到一起,转成小写,去标点,切分
    val word = words.filter(x=>x.trim.size>0).flatMap(lines=>{
        lines.trim.toLowerCase().replaceAll("""[,." "!]"""," ").split("\\s+")
    })
	//创建的停用词数组
val arr = Array("he","hello","ha","flume")
		//去停用词 对value字段进行wordcount		
    val info = word.filter(!arr.contains(_)).groupBy("value").count().orderBy("count")
    info.show()

  }
}
```









