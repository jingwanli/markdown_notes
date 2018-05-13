## Spark概述

MR(基于进程)存在的缺点:

1. 表达能力有限
2. 磁盘IO开销大:主要是mapreduce和mapreduce之间的开销
3. 延迟高:慢

Spark是MR的优化,非重写

Spark:All in one

Storm的延迟: 毫秒

Spark的延迟:亚秒(0.1秒)

Spark生态系统

![](/Users/lijingwan/Documents/img/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-04-23%20%E4%B8%8A%E5%8D%8810.55.21.png)

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-18 上午9.03.58.png)

Spark架构

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-18 上午9.05.43.png)

1. Spark运行架构包括: Master(集群资源管理), Slaves(运行任务的工作节点), 应用程序的控制节点(Driver) 和 每个工作节点上负责任务的执行进程(Executor)
2. Master是集群资源的管理者(Cluster Manager) Spark支持: Standalone, Yarn, Mesos
3. Slave在spark中被称为Worker
4. Driver Program

集群的计算资源:CPU 和 内存(不包括磁盘)

Mesos可以选择调度模式: 粗粒度,细粒度

Standalone模式下的运作方式:

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-23 上午11.53.27.png)



Master只管理集群资源, 如果没有Driver管理跑起来的应用, 影响集群的扩展性. 类比hadoop, 没有yarn之前,就是没有appmaster, 集群的扩展受到限制.

Spark集群部署模式:

1. Standalone: 独立模式.自带完整的服务, 可以单独部署到一个集群中, 无需依赖任何其他资源管理系统. 从一定程度上说, 该模式是其他两种的基础.

2.  yarn:生产环境很多都是基于yarn的.仅支持粗粒度模式.

   YARN上的Container资源是不可以动态伸缩的, 一旦Container启动后, 可使用的资源不能再发生变化

3. spark on yarn的支持两种模式:

   - yarn-cluster:生产环境
   - yarn-client: 适用于交互和调试, 希望立即看到app的输出.

==注意: standalone和yarn 只能用粗粒度方式!!!!!==

Mesos:

用户可选择两种调度模式之一:

1. 粗粒度模式: 每个应用程序的运行环境由一个Driver和若干Executor组成, 其中, 每个Executor占用若干资源,内部可以运行多个Task. (对应多个"slot"). 应用程序的各个任务正式运行之前,需要将运行环境中的资源全部申请好, 且运行过程中要一直占用这些资源, 即使不用,最后程序运行结束后, 回收这些资源.
2. 细粒度模式: 按需分配

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-23 下午12.13.21.png)

==主节点==: start-dfs.sh(启动与hdfs相关的东西)

​	    stop-dfs.sh

​	    start-yarn.sh

hdfs dfs -ls data/ 等价于 hadoop fs -ls data/

### Spark开发环境搭建

规划:

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-23 下午1.12.22.png)

导入spark相关的依赖有2个办法

1. 引入相关的jars(操作简单. 但是有可能其他软件的包会和它有冲突)-手工管理
2. 使用maven/sbt管理jars(操作复杂)

Maven要装在装Idea的节点.

验证安装成功的方式:

spark-shell

run-example SparkPi 10

spark-submit - -class org.apache.spark.examples.SparkPi  ./examples/jars/spark-examples_2.11-2.2.0    jar  10

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-23 下午1.26.41.png)

> KNN程序

与过滤空行有关的:

1. "      ".trim

				2. "      ".trim.size == 0

val data1 = template.filter(_.trim.size!=0).filter(\_.trim.split(",").size==5).map…...

























