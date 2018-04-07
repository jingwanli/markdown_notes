---
typora-root-url: ../img
---

# Storm

计算类型:

离线计算:

  - MapReduce
  - Hive
  - SparkSQL

实时计算:

- HBase
- Storm



storm  真正意义上的实时流式计算框架( 实时路况, 实时转账 . 没有结束,源源不断的有数据)

MapReduce   批量离线处理的计算框架

###Storm安装

安装条件:

1. 需要在Hadoop已成功安装的基础上,并要求Hadoop集群已经正常启动

   需要zookeeper正常启动的情况下

   bin/zkCli.sh -server master:2181

2. 解压并安装: unzip apache-storm-0.9.6.zip

3. 配置storm环境变量

   - cd apache-strom-0.9.3
   - vim~/.bash_profile
   - export STORM_HOME=/home/xdl/apache-storm-0.9.3
   - export PATH=\$STORM_HOME/bin:$PATH
   - source ~/.bash_profile

4. 修改storm.yaml配置文件

   - storm.zookeeper.servers:

     \- "master" 

     \- "slave" 

   - nimbus.host:“hadoop0"  //==注意:前面有空格!!!!==

     nimbus.host:Storm集群Nimbus机器地址，负责资源分配和任务调度

     storm.zookeeper.servers:Storm集群使用的Zookeeper集群地址

5.  将Storm安装文件复制到slave1 和 slave2节点

   - scp-r apache-storm-0.9.3 slave1:~/ 
   - scp-r ~/.bash_profile slave2:~/

6. 启动storm

   主节点:

   ​	storm nimbus > /dev/null 2>&1 &

   ​	2>&1 将错误输出重定向到标准输出

   ​	1:标准输出

   ​	&:在后台运行,控制台不打印storm运行日志

   从节点:

7. 验证:

   - master
     - core
     - nimbus
   - slave
     - supervisor

8. 运行一个Demo

   - storm *.jar com.aa.ss……...

9. Storm中的作业叫做Topology

###Storm基础

> 实时流计算

**实时流的背景**

​	数据open

​	离线计算的建模不能够满足实时性计算处理的需求

**应用实例**

​	金融服务

​	网络监控

​	电信数据管理

​	…...

**处理单位**

​	实时计算时,处理数据的单位是元组tuple,包含了监测,呼叫记录

**实时计算数据特性**

​	实时计算的数据以量大,  持续,  快速,  时变的数据流持续到达

——> 基础性的,新的研究问题------------实时计算

> 什么是Storm?

Storm是Twitter开源的,  分布式的,  高容错的实时计算系统.

Storm通过简单的API可以可靠的处理无界持续的数据流

​	Spout

​	Bolt

Storm保证每个消息都会得到处理.Storm对实时计算的处理相当于Hadoop的批处理(MR).

每秒可以处理数以百万计的消息. 经测试, 每个节点每秒可以处理100万个数据元组

> Storm和Hadoop之间的角色对比

![](/屏幕快照 2018-03-27 上午11.48.40.png)

|              Hadoop               |                  Storm                   |
| :-------------------------------: | :--------------------------------------: |
|    ResouceManager  MRAppMaster    |  Nimbus<br />负责资源的分配和任务的调度  |
|            NodeManager            | Supervisor<br />监视监控本节点任务的执行 |
| YarnChild<br />MapTask/ReduceTask |     Worker(工作进程)<br />Spout/Bolt     |

| 应用名称 |          |
| :------: | :------: |
|   job    | Topology |

|  组件接口  |            |
| :--------: | :--------: |
| Map/Reduce | Spout/Bolt |

### Storm的核心组件

- nimbus (master)：nimbus把分配好的任务计划信息保存到zookeeper中

- supervisor(slave...)： 负责监听分配给他的工作，根据需要启动或关闭worker进程

  ​					supervisor监听的是zookeeper

  ​				 	每个Worker进程执行的是Topology的一个子集	

- zookeeper :负责nimbus和supervisor之间的协调 (没有用到选举)

				存储nimbus分配给supervisor的工作计划信息

- Worker :工作进程

  ​		运行具体处理组件逻辑的进程(可以开启多个,并发执行)

				Topology ： 一个Topology是由运行在很多机器上的工作进程Worker组成

- Task (线程): 工作进程中的具体任务

  - spoutTask
  - boltTask
  - 每一个Spout/Bolt线程 被视为一个Task

- 注意:

  slot 任务槽. web界面可以看出. 使用了几个任务槽,就有几个worker.

  execulors 物理线程:相当于slot里开启了多少线程

  Task线程 启动线程 

  在Storm0.8以后,Task不在与物理线程对应, 同一个Spout/Bolt的task可能会共享一个物理线程,该线程称为一个Executor

  命令: storm kill exclamation-topology(topology别名) 

  storm jar 会根据你提交的源代码,内部的一个打包,拷贝到了storm工作中的本地目录.但是不可以直接运行这些jar包

  ### Storm Topology结构

  ![](/屏幕快照 2018-03-27 下午3.24.01.png)

  ### Storm特性

  1. 高可靠性: Storm可以保证Spout发出的每条信息都能被"完全处理"

     ​		处理以后要执行回调函数. 如果fail,会重发.

  2. 高容错型: 如果在消息处理过程中出现异常,重新安排.Storm保证一个处理单元永远运行

  3. 支持本地模式: 在进程中模拟一个storm的所有功能.本地模式主要用于开发和测试. 不是分布式的.

  4. 支持多种编程语言

     除了java实现spout和bolt, 还可以使用phython.scala等

  5. 高效

     用zeroMQ作为底层消息队列,保证消息能快速被处理

     秒级处理数百万数据

  ### Storm应用场景

  实时分析, 在线机器学习(需要不断的迭代),  持续计算,  分布式RPC

> 三大主要应用

1. 信息流的处理
2. 持续计算
3. 分布式远程程序调用(distributed RPC)

### WordCount案例

继承自BasicRichSpout类 — >抽象类,实现了IRichSpout —>继承自ISpout—>实现了序列化

​    ISpout ——>open方法:初始化时会调用一次 SpoutCollector…..发射器(非常重要)

​		—— > close

​		——>activate( )

​		——> nextTuple( ) 元组/数组 storm处理消息的单位.  源源不断的调用(最重要)

​			  通过jdbc可以调数据库,可以发起url的请求抓取数据. //产生tuple方法

​			  相当于数据源

​		——> ack 确认tuple被成功处理 //调用多次

​		——> fail() 当tuple处理失败时回调 //调用多次

​	USER

UID ,name, sex   age

111,   zhang  man  23将本行数据封装到一个元组中.元组中有key value

tuple  UID

tuple  UID  UNAME

> 注

集群模式: eclipse中,设置并发量. (任务数)