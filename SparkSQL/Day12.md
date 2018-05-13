## SparkSQL

如果SparkSQL解决不了, 仍然要回到SparkCore中去解决.

前身: Shark 把SQL翻译成了spark的作业,但是只是从执行计划开始翻译,前面的部分用的hive

sparkSQL只用了hive元信息:表的结构, ...

hive主导的:hive on spark (apache委员会)

spark主导的: sparksql (Cloudery公司)

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-27 上午8.59.51.png)

Spark SQL目前支持:Scala, Java, Python三种语言.

Spark SQL: 操纵的是 DateFrame 或者是 DateSet(底层是RDD)

Spark: 操纵的是RDD

---

> DateFrame核心:

Schema:==包含了以ROW为单位的每行数据的列的信息.==

Schema主要作用: 告诉spark每行数据是什么类型.

​		使用了堆外内存.

主要缺点: 编译的时候不检查, 是类型不安全的. 不是纯粹的面向对象的风格.

> DS核心

Encoder:

编译时有类型检查. 2.0中,df只是ds的特例.

type DataFrame = Dataset[Row]

Dateset是由case class定义的! 

DATASET: = RDD + CASE CLASS

DATEFRAME = RDD + SCHEMA

### SparkSQL API

2.0以前:

sparkcontext

sqlcontext

hivecontext

streamingcontext

2.0以后:

SparkSession(sparkcontext/sqlcontext/hivecontext都包含了): SparkSQL的入口.

streamingcontext

标注: @transient 说明后面的成员变量传递的时候不用被序列化

#### Row

范化的无类型的JVM对象

```scala
import org.apache.spark.sql.Row
val row1 = Row(1,1.1,"abc") //row1(0) --> Any类型
row1.getInt(0)
row1.getDouble(1)
row1.getString(2)
//方式2
row1.getAs[Int](0)
row1.getAs[String](2)


```

#### schema

DataFrame(带有Schema信息的rdd)

Spark通过Schema就能读懂数据

```scala
import org.apache.spark.sql.types._
val schema = (new StructType).add.....
```

sequence的缺省实现就是List!



## DataSet & DataFrame的操作

```scala
object 
```



------

### Yarn启动Spark:

###### start-yarn.sh

###### spark-shell - -master yarn

------

spark-shell - -help//

spark-shell - -deploy-mode cluster// 报错!~



