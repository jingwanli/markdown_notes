## SparkSQL和Hive

数据仓库: 面向主题, OLAP(在线分析系统),(不产生数据,主要做分析,语句最多的是select),hive可以做到水平扩展.

传统的数据库:面向OLTP(在线交易处理),做不到水平线性扩展(机器扩大一倍,能力扩大一倍,垂直扩展: 4g==> 8g)

service mysql status

service mysql stop

scp MySQL-* master:/opt/software

装mysql

1. 换用户==>root

2. rpm -aq | grep mysql*

3. yum -y remove mysql-libs*

   mysql版本: 

4. yum -y remove mysql-lib*

5. ​

rpm -e - -nodeps 软件名称:删除安装的软件

gedit &  :后台打开一个记事本

vim一个文件后: esc + :set number

---

传统数据库还有:

层次数据库

网络数据库:有交叉

现在已经都被关系数据库替代.

---

去IOE: IBM Oracle EMC(共享存储)

瓶颈: IO

数据仓库特点:

1. 面向主题的
2. 集成的
3. 反映历史变化的
4. 相对稳定的

> 数据仓库体系架构

![](/Users/lijingwan/Documents/img/屏幕快照 2018-05-04 下午7.18.38.png)

1. ETL在整个数据仓库建设的80%(工作量)

2. 整个数据存储部分: 是一个数据库, 只不过把数据分开存放了.(相当于目录和文件的关系)

   数据仓库存储引擎: 存放的是明细数据(满足第三范式),拿过来的数据是什么样的就是什么样的数据,不会做任何汇总.(当然要经过转换和清洗). 明细数据满足第三范式, 按理说是没有冗余的.但是它做查询的效率很低, 因为有大量的表连接.

   引擎到后面各集市的转换通常也认为是ETL

3. 数据仓库分层

   层和层之间不要交叉

   同一层之间的数据不要交叉

   ![](/Users/lijingwan/Documents/img/屏幕快照 2018-05-04 下午7.31.48.png)

   后面的分层方式不可取!

![](/Users/lijingwan/Documents/img/屏幕快照 2018-05-04 下午8.35.39.png)

![](/Users/lijingwan/Documents/img/屏幕快照 2018-05-04 下午8.40.28.png)

![](/Users/lijingwan/Documents/img/屏幕快照 2018-05-04 下午8.43.38.png)

维度没有单位

度量有单位

![](/Users/lijingwan/Documents/img/屏幕快照 2018-05-04 下午8.48.37.png)

![](/Users/lijingwan/Documents/img/屏幕快照 2018-05-04 下午8.51.18.png)



---

作业讲解

1. wordcount

```scala
...spark-shell
val ds1 = spark.read.textFile("data/spark.log")
ds1.printSchema
val ds2 = ds1.filter(_.trim.size!=0).flatMap(_.split("\\s+"))
ds2.show
val ds3 = ds2.map(_.trim.toLowerCase)
//去标点 去停用词
val lst = List("and","a","at","of","in","the")
val ds4 = ds3.filter(!lst.contains(_))
val ds5 = ds4.map(_.replaceAll("[,;?!\"\']"," "))
ds5.printSchema
ds5.groupBy("value").count.sort(-'count).show

idea中:
注意:
import spark.implicits._
在windows下,加上.enableHiveSupport()可能会报错
读文件:hdfs要写全
//方式2
groupby上面的写法只能算一个值
ds5.groupBy("value").agg(Map("value"->"count")).show
//方式3
import org.apache.spark.sql.functions.count
ds5.groupBy("value").agg(count("value")).show

```

2. SparkSQL  统计

```scala
//idea中:

object SparkSQL2{
  ..main...
   ..spark...=....
  
 //a. df1.createOrReplaceTempView("t1")
import spark.sql
sql(
"""
    select date,sale,
    sum(sale) over (order by date) as salesum
    from t1
""".stripMargin).show 
//stripMargin是将乱掉的格式去掉
    
// b.
 df2.......
 sql("""
 	select name,class,s
 	row_number() over (partition by class order by s desc) as rank1
 	from t2
 	having rank1 ==1
 """)
    
//c
 val df3....
 sql("""
 select A,B,sum(C) 
 from t3
 group by cube(A,B)
 """
 )
    
 //d
 sql("""
 select name,dept,sal, sum(sal) over(partition by dept) as salsum,
sal/(sum(sal) over (partition by dept)) as ratio 
 from t4
 """)
    
 //e 求所有的
 sql("""
 select id,sum(salary), 
 from t5
 group by cube(id)
 """)
    
 sql("""
 	select id,job,sum(salary)
 """)
}

----DSL
import org.apache.spark.sql.functions._
import...Window
df1.select($"date",$"sale",sum("sale").over(Window.orderBy("date")))
//$改成col也可以
//引号里的东西编译时是不做检查的,只有运行时异常.
df2.select($"name",$"class",$"s",row_number().over(Window.partitionBy("Class").orderBy(-'s).alias("rankS")).where("rankS=1").show
//这里where不是where条件,而是filter! 做的是过滤!用filter也可以!

df3.cube($"A",$"B").sum("C").show

df4.select($"name",$"dept",$"sal",sum("sal").over(Window.partitionBy("dept"))

df5.groupBy("id").sum()        




```









