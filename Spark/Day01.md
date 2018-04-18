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

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-18 上午9.03.58.png)

Spark架构

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-18 上午9.05.43.png)

集群的计算资源:CPU 和 内存(不包括磁盘)

Mesos可以选择调度模式: 粗粒度,细粒度

其他两种: 粗粒度

==主节点==: start-dfs.sh(启动与hdfs相关的东西)

​	    stop-dfs.sh

​	    start-yarn.sh

hdfs dfs -ls data/ 等价于 hadoop fs -ls data/

### Spark开发环境搭建

就是jar

导入spark相关的依赖有2个办法

1. 引入相关的jars(操作简单. 但是有可能其他软件的包会和它有冲突)-手工管理
2. 使用maven/sbt管理jars(操作复杂)

Maven要装在装Idea的节点



