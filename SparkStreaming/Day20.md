## spark整合flume

ps -ef | grep yum

yum install telnet

---

安装nc 和 telnet

root用户

```scala
//安装nc
mount /dev/sr0 /mnt -> 虚拟机的网络配置光盘(因为所有的安装包都在里面)->cd /mnt->cd Package ->ll | grep nc
-> rpm -ivh nc...
//使用nc
nc -lk 33333//其中33333是端口号 ,nc只能往本机发数据
```

```scala
//安装telnet telnet分为客户端 和 服务器端,都要装
rpm -ivh xinetd... //服务器端的安装依赖xinetd,故要先安装xinetd
rpm -ivh telnet... //服务器端
rmp -ivh telnet... //客户端
```

flume的数据: event

接口里面最重要的: kafka

转换: 无状态的: 一个时间段的数据全部算完,结果该输出的输出,就不管了.

有状态的: 前面一段时间也要包括. 有windows的操作(有时间窗口的操作)

### DStream无状态转换操作

ds的转换操作比rdd的转换操作要少.

receiver一直在接受数据(后台运行),并把数据做成rdd

ds的满足不了需求,我们再去操作rdd.

```scala
ds.transform(x=>)...//x是变成了rdd,相当于转化成了rdd的操作
```

> 黑名单过滤

1. wordcount:("a"…."z") //流的形式得到的数据

   RDD

2. blacklist = List["a","b","c","d","e"] (array也可以)//固定的数据结构

3. 过滤

```scala
val arr1 = Array(("spark",true),("scala",false))
val rdd1 = sc.makeRDD(arr1)
val arr2 = Array("1 hadoop","2 spark","3 scala","4 java","5 hive")
val rdd2 = sc.makeRDD(arr2)
...
spark-shell
//方法1: join,先变成key-value对
val rdd3 = rdd2.map(line=>(line.split("\\s+")(1),line))
rdd3.leftOuterJoin(rdd1).collect
//rdd3.leftOuterJoin(rdd1).filter(x=>x._2._2!=true)//none和true变成了some!!!!都不等于true
rdd3.leftOuterJoin(rdd1).filter(x=>if (x._2._2.getOrElse(false)) false else true).collect
-------------------
//只有key-value对才能做join!!
val rdd3 = rdd2.map(line=>(line.split("\\s+")(1),line))
rdd3.join(rdd1).collect(默认的和key进行连接不需要写条件!!) //结果:Array((scala,(3 scala,false)),.......)
rdd3.leftOuterJoin(rdd1).filter(x=>(!x._._2.getOrElse(false))).map(_._2._1).collect


//去停用词:
mapPartitions; 将List设为广播变量

//getOrElse(true):
//getOrElse(false):


//方法2: 是否包含
```

```scala
idea中:

```



这里的count 不是action !!

---

## 新整理 Spark Streaming 与 Flume整合

需要启动: 1. idea  2. flume的agent 3.telnet

> 模式

推模式:(Flume push SparkStreaming)//数据量过大容易被推死.没有缓存,数据处理不动,消费不掉

拉模式(SparkStreaming pull Flume)//数据要存在某个地方,我们才可以去pull. 叫做sink: 是spark提供的 -涉及到导包,导jar的问题 

//对于我们来说,区别仅仅在于取数据的端口不同.

导包: spark-streaming-flume-sink_2.11-2.2.0.jar

​	scala-library-2.11.8.jar 

​	拷贝到:$FLUME_HOME/lib中

去哪里找jar?: 去idea的机器上找,idea的所有jar都放在m2中:

cd .m2

ls //结果: repository , settings.xml.bak

find . -name spark-streaming-flume-sink_2.11-2.2.0.jar

备注: scala-library-2.11.8.jar需要删除

## DStream

receiver 一直在接收数据,一直在后台运行,并将接收到的数据弄成RDD

ds.transform(x=>) //x是RDD

> 黑名单过滤

```scala
分析:
1.wordcount("a",...."z")
//上面的数据是以流的方式得到的
2.blacklist = List["a","d","c","e"] //固定的数据结构
题目:
假设arr1为黑名单数据,true表示数据生效,需要被过滤.false表示数据未生效
val arr1 = Array(("spark",true),("scala",false))
val rdd1 = sc.makeRDD(arr1)

假设:arr2为原始的数据,格式为(time,name),需要根据arr1中的数据将arr2的数据过滤
如"2 spark"要被过滤掉
val arr2 = Array("1 hadoop","2 spark","3 scala","4 java","5 hive")
val rdd2 = sc.makeRDD(arr2)

结果:Array(3 scala,5 hive, 4 java,1 hadoo p)



```



