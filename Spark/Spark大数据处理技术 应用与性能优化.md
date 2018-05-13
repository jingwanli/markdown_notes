## 第1章 Spark简介

### Spark是什么?

Spark 是基于内存计算的大数据并行计算框架.

目前，Spark 生态系统已经发展成为一个包含多个子项目的集合，其中包含 SparkSQL、Spark Streaming、GraphX、MLlib 等子项目，AMPLab 开发以 Spark 为核心的 BDAS 时提出的目标是:one stack to rule them all，也就是说在一套软件栈内完成各种大数据分析任务.

同时我们也应该看到 Spark 并不是完美的，RDD 模型适合的是粗粒度的全局数据并行计算。不适合细粒度的、需要异步更新的计算。

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-19 上午10.31.35.png)

Spark 将分布式数据抽象为弹性分布式数据集(RDD)，实现了应用任务调度、RPC、序列化和压缩，并为运行在其上的上层组件提供 API.其底层采用 Scala这种函数式语言书写而成，并且所提供的 API 深度借鉴 Scala 函数式的编程思想，提供与Scala 类似的编程接口。

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-19 上午10.34.40.png)

Spark 将数据在分布式环境下分区，然后将作业转化为有向无环图(DAG)，并分阶段进行 DAG 的调度和任务的分布式并行处理。

##### spark架构

Spark 架构采用了分布式计算中的 Master-Slave 模型。Master 是对应集群中的含有Master 进程的节点，Slave 是集群中含有 Worker 进程的节点。Master 作为整个集群的控制器，负责整个集群的正常运行;Worker 相当于是计算节点，接收主节点命令与进行状态汇报;Executor 负责任务的执行;Client 作为用户的客户端负责提交应用，Driver 负责控制一个应用的执行，如图 1-4 所示。

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-19 上午10.52.33.png)


Spark 的整体流程为:Client 提交应用，Master 找到一个 Worker 启动 Driver，Driver 向Master 或者资源管理器申请资源，之后将应用转化为 RDD Graph，再由 DAGScheduler 将RDD Graph 转化为 Stage 的有向无环图提交给 TaskScheduler，由 TaskScheduler 提交任务给Executor 执行。在任务执行的过程中，其他组件协同工作，确保整个应用顺利执行。


​				
​			
​		
​	
​		
​	
​	
​		
​	



​		
​	


​			
​		
​					


​			
​		
​	