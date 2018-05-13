## Day15

> 练习&作业 from dan
>
> ![](/Users/lijingwan/Documents/img/屏幕快照 2018-05-03 下午8.06.53.png)

```scala
val df = spark.read.options(Map(("header","true"),("inferschema","true"))).csv("file:///home/spark/data/sparkpv.csv")
或者
val df = spark.read.
option("header", "true").
option("inferschema","true").
csv("file:///home/spark/data/sparkpv.csv")

val df2 = spark.read.
option("header", "true").
option("inferschema","true").
csv("file:///home/spark/data/anay.dat")
df2.printSchema
df2.createOrReplaceTempView("t2")
spark.sql("select * from t2").show

spark.sql("""
  select name, id, sum(num) 
    from t2 
  group by cube(name, id) 
  order by 1,2""").show
  
spark.sql("""
  select name, id, sum(num) 
    from t2 
  group by rollup(name, id) 
  order by 1,2""").show


df.printSchema
df.createOrReplaceTempView("t1")
spark.sql("select * from t1").show

spark.sql("""
select cookieid,createtime,pv,
      sum(pv) over(partition by cookieid order by createtime) as pv1 
from t1
""").show

spark.sql("""
select cookieid,createtime,pv,
      sum(pv) over(partition by cookieid order by createtime) as pv1,
	  sum(pv) over(partition by cookieid order by createtime
	          rows between unbounded preceding and current row) as pv2
from t1
""").show

--- partition by & order by
spark.sql("""
select cookieid,createtime,pv,
       sum(pv) over(partition by cookieid order by createtime) as pv1
from t1
""").show

spark.sql("""
select cookieid,createtime,pv,
       sum(pv) over(partition by cookieid) as pv2
from t1
""").show

spark.sql("""
SELECT cookieid, createtime, pv,
       SUM(pv) OVER(PARTITION BY cookieid) AS pv2
FROM t1
""").repartition(2).show
//分区

//3
spark.sql("""
select cookieid,createtime,pv,
       sum(pv) over(order by createtime) as pv3
from t1
""").show
//4
spark.sql("""
select cookieid,createtime,pv,
       sum(pv) over() as pv4
from t1
""").show

spark.sql("""
select cookieid,createtime,pv,
       sum(pv) over(partition by cookieid order by createtime) as pv1
from t1
""").repartition(2).show

spark.sql("""
select cookieid,createtime,pv,
       sum(pv) over(partition by cookieid order by createtime) as pv1,
	   sum(pv) over(partition by cookieid order by createtime
	           rows between unbounded preceding and current row) as pv2
from t1
""").show 


spark.sql("""
select cookieid,createtime,pv,
       sum(pv) over(partition by cookieid order by createtime) as pv1,
	   sum(pv) over(partition by cookieid order by createtime rows between unbounded preceding and current row) as pv2,
	   sum(pv) over(partition by cookieid) as pv3,
	   sum(pv) over(partition by cookieid order by createtime rows between 1 preceding and 1 following) as pv4,
	   sum(pv) over(partition by cookieid order by createtime rows between 1 preceding and current row) as pv5,
	   sum(pv) over(partition by cookieid order by createtime rows between current row and 1 following) as pv6,
	   sum(pv) over(partition by cookieid order by createtime rows between current row and unbounded following) as pv7
from t1
""").show 

//计算比例
spark.sql("""
select cookieid,createtime,pv,
       sum(pv) over(partition by cookieid) as pv1,
	   pv/sum(pv) over(partition by cookieid) as ratio
from t1   
""").show

spark.sql("""
  select 
         lag(pv)    over (PARTITION BY cookieid ORDER BY pv) as col1,
    from t1
  order by cookieid""").show


  spark.sql("""
  select cookieid, createtime,pv,      
         la  as col3
    from t1  
  order by cookieid""").show
  
  
 
1、按日统计sale的累计值

val df1 = spark.read.
option("header", "true").
option("inferschema","true").
csv("file:///home/spark/data/exer1.csv")

df1.createOrReplaceTempView("s1")
spark.sql("""
select date,
     sum(sale) over(order by date) as total
from s1
	 """).show
	 
//DSL
import org.apache.spark.sql.expressions.Window
val ww1 = Window.orderBy("date")
df1.select($"date",sum("sale").over(ww1).alias("total")).show

2、各班成绩第一的信息
val df2 = spark.read.
option("header", "true").
option("inferschema","true").
csv("file:///home/spark/data/exer2.csv")

df2.createOrReplaceTempView("s2")
spark.sql("""
select name,class, s, rank1 
    from(select name,class,s,
		 rank() over(partition by class order by s desc) as rank1
	     from t2) 
	where rank1=1
""").show

spark.sql("""
 select name,class,s,
		 rank() over(partition by class order by s desc) as rank1
	     from s2
	having rank1=1
""").show
	 
val ww2 = Window.partitionBy("class").orderBy(desc("s"))
df2.select($"name",$"class",$"s",rank().over(ww2).alias("rank1")).filter("rank1=1").show

3、分类统计 (A、B列各种情况的统计信息，如m的合计值，a的合计值，x、b的合计值，m、a的合计值等)

val df3 = spark.read.
option("header", "true").
option("inferschema","true").
csv("file:///home/spark/data/exer3.csv")

df3.createOrReplaceTempView("s3")   

//cube，根据GROUP BY维度的所有组合进行聚合  m的合计值，a的合计值

spark.sql("""
  select A, B, sum(C) 
    from s3
  group by cube(A, B) 
  order by 1,2""").show
  
 spark.sql("""
  select A, B, sum(C) 
    from s3
  group by rollup(A, B) 
  order by 1,2""").show


4、个人工资占部门工资的比例
val df4 = spark.read.
option("header", "true").
option("inferschema","true").
csv("file:///home/spark/data/exer4.csv")

df4.createOrReplaceTempView("s4")

spark.sql("""
select name,dept,sal,
       sum(sal) over(partition by dept) as sal1,
	   sal/sum(sal) over(partition by dept) as ratio
from s4   
""").show
//order by 要加到s4后面.
//如果在组内排dept,没意义,组就是按照dept分的,在组内将看不到效果!

5、
求各部门（ID）的工资总数
求各部门（ID）的工资总数 及 全部的工资数
求按ID、JOB分类的工资总数
val df5 = spark.read.
option("header", "true").
option("inferschema","true").
csv("file:///home/xdl/exer5.csv")

df5.createOrReplaceTempView("s5")

spark.sql("""
select id,job,name,salary,
       sum(salary) over(partition by id) as idtotal
from s5  
""").show


spark.sql("""
select id,job,name,salary,
       sum(salary) over(partition by id) as idtotal,
       sum(salary) over(order by id rows between unbounded preceding and unbounded following) as total
from s5  
""").show

spark.sql("""
select id,job,name,salary,
       sum(salary) over(partition by job) as jobtotal,
       sum(salary) over(partition by id) as idtotal
from s5  
""").show

要求：
1、最终在IDEA中实现
2、包括SQL和DSL

val df = spark.read.
option("header", "true").
option("inferschema","true").
csv("file:///home/spark/data/sparkpv.csv")
```



