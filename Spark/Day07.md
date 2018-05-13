## Day07

------回顾-------

Spark支持三种集群部署模式: Standalone, Yarn, Mesos

Driver:管理整个应用

1. 包含main方法
2. 初始化SparkContext

 Cluster manager: Standalone模式下:是master

​				Yarn模式下: 是resource manager

Yarn的两种模式:

Deploy mode: 部署模式: 看Driver运行哪个地方:

运行在Cluster: 集群模式

运行在客户端: Client模式:适用于交互(打印到控制台), 调试

---------------上课内容----------------

配置文件的路径: $SPARK_HOME/conf

重要的配置文件: 

- spark-default.conf
- spark-env.sh
- slaves
- log4j.properties

测试(2个程序[WordCount, pi])3种方式:

-  spark-shell

- run-example SparkPi 10

- spark-submit - - class org.apache.spark.examples.SparkPi  ./examples/jars/spark-examples_2.11-2.2.0.jar 10

  格式: spark-submit - - class 主类路径 jar包路径 程序输入参数(可能是多个)这是最简单的方式

==注意:==spark-shell, run-example最终都是调用了spark-submit

​	我们提交的spark任务都是使用spark-submit

>  spark-defaults.conf:

spark.driver.memory: 512m 默认是1024m

spark.cores.max : 6  为应用程序分配的最大CPU核心数(启动一个应用程序, 会拿6个core)

>  log4j.properties:

定义写日志的级别.

Log4j只建议使用四个级别: 优先级从低到高分别是:

DEBUG, INFO, WARN, ERROR (级别越低,打印的信息越多)

更改日志级别的方式:

 1. 在程序中 sc.setLogLevel("WARN")//设置日志级别

2.  log4j.properties中:

==变量设置的优先级:==

程序设置>spark-submit - -选项 > spark-defaults.conf 配置 >

spark-env.sh 配置 > 默认值

### 服务启动和停止

1. start-all.sh  stop-all.sh

2. 逐一启动主节点, 从节点(在不同的节点上发指令)

   start-master.sh    stop-master.sh

   start-slave.sh   spark://nodel:7077 (需要告诉程序去哪里找master)

3. 一次启停全部的从节点:

   start-slaves.sh    stop-slaves.sh (必须在主节点跑!)

4. 很少使用:

   spark-daemon.sh start org.apache.spark.deploy.master.Master

   spark-daemon.sh stop  org.apache.spark.deploy.master.Master

   spark-daemon.sh status org.apache.spark.deploy.master.Master

### Spark安装检查

Web UI: http://node1:8080/

不启动history server则跑完成的程序就不会被记录下来.

spark核心数设置为6, 只能启一个应用程序

想启动多个需要加参数(设置为6的情况下): spark-shell - -total  -executor-cores 2 

​					//将spark-shell看做一个应用, 最多只能占2个core

​					然后执行作业



### Spark部署运行模式:

Apache Spark支持多种部署模式。

最简单的就是单机本地模式（Spark所有进程都运行在一台机器的JVM中)(jps看不见,因为jps看的是下线程, 这些都是以线程的方式存在一台JVM中)

伪分布式模式（在一台机器中模拟集群运行，相关的进程在同一台机器上）

分布式模式包括：Standalone、Yarn、Mesos。

(standalone也有cluster和client之分)

#### 本地模式

启动方式: spark-shell - -master local

部署在单机, 主要用于测试或实验.

不用启动master, worker进程

#### 伪分布式

local-cluster[N,cores, memory]，N模拟集群的Slave节点个数；

cores模拟集群中各个Slave节点上的内核数；memory模拟集群的各个Slave节点上的内存大小；

启动方式: spark-shell - -master local-cluster[3,2,1024]

#### 集群模式*

spark://：Spark的Standalone模式

Yarn ：Yarn模式

分布式部署有client, cluster两种运行模式:

如果driver在client上,则为client模式

如果driver在cluster上,则为cluster模式

cluster没有返回结果,client有返回结果.

core: 集群中核心数core每台机器可以不同. (改配置,配置中内存不能写小数)

生产环境中,用的是集群管理器. 将集群分成几类.



### RDD( Resilient Distributed Datasets)弹性分布式数据集

 五大特征:

….

创建RDD的方法:

1. 在内存中创建: parallelizing一个已存在的集合(makeRDD)
2. 外部存储系统中引用一个数据集.(广义的), 如HDFS, HBASE, 或者 Hadoop InputFormat的任何数据源.

SparkContext只允许有一个!

### 路径问题

hdfs://node1:8020/user/spark/data/a.log

在spark-shell中可以写为: data/a.log

==前面的:hdfs://node1:8020/user/spark相当于家目录==

但是在idea中必须写全路径!!

用hdfs更方便!

如果用文件系统的话, 各节点都要有数据, 否则driver发到没有数据的节点,会报错!

action触发job!

 一个application可以有多个job, 有几个action就有几个job!!

collect( ) :把所有的数据收回到Driver里面. 生产环境中尽量不要用.!

### WordCount详解





