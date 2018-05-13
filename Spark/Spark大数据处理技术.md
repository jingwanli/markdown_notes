## 第2章 Spark RDD及编程接口

RDD: 弹性分布式数据集:一个RDD代表一个被分区的只读数据集.

生成途径:

- 来自内存集合和外部存储系统
- 通过转换操作来自于其他RDD(map, filter, join等)

RDD没有必要被随时实例化. 由于RDD的接口只支持粗粒度的操作(即一个操作会被应用在RDD的所有数据上),所以只要通过记录这些作用在RDD上的转换操作,来构建RDD的继承关系,就可以有效地进行容错处理,而不需要将实际的RDD数据进行拷贝.

开发者还可以对RDD进行2方面的控制操作:

- 持久化
- 分区

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 上午10.45.11.png)

#### RDD分区

对于一个RDD而言, 分区的多少设计对这个RDD进行并行计算的粒度, 每一个RDD分区的计算操作都在一个单独的任务中被执行.

如果没有指定, 那么将会使用默认值.

可以利用RDD的成员变量partitions所返回的partition数组的大小来查询一个RDD被划分的分区数.

####RDD优先位置(preferredLocations)

RDD的优先位置属性与Spark中的调度相关, 返回的是此RDD的每个partition所存储的位置.

#### RDD依赖关系(dependencies)

分类:

- 窄依赖(Narrow Dependencies): 每一个父RDD的分区最多只被子RDD的一个分区所使用
- 宽依赖(Wide Dependencies): 多个子RDD的分区会依赖同一个父RDD的分区

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 上午11.18.32.png)

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 上午11.19.13.png)



tips:

1. 窄依赖可以在集群的一个节点上如流水线一般的执行, 可以计算所有父RDD的分区

   宽依赖需要取得父RDD的所有分区上的数据进行计算将会执行类似于MapReduce一样的Shuffle操作.

2. 对于窄依赖来说, 节点计算失败后的恢复会更加有效, 只需要重新计算对应的父RDD的分区,而且可以在其他的节点上并行的计算.对于宽依赖来说,一个节点的失败将会导致父RDD的多个分区重新计算, 代价非常高

####RDD分区计算(compute)

对于Spark中每个RDD的计算都是以partition(分区)为单位的, 而且RDD中的compute函数都是在对迭代器进行复合, 不需要保存每次计算的结果.

#### RDD分区函数(partitioner)

目前,Spark中实现了两种类型的分区函数:

- HashPartitioner ( 哈希分区 )
- RangePartitoner(区域分区)

且patitioner这个属性只存在于( K,V )类型的RDD中,对于非(K,V)类型的partitioner的值就是None

partitioner函数既决定了RDD本身的分区数量, 也可以作为其父RDD shuffle输出(MapOutput)中每个分区进行数据切割的依据.

### 创建操作

#### 集合创建操作

RDD的形成可以由内部集合类型来生成. Spark中提供了parallelize和makeRDD两类函数来实现从集合生成RDD, 两个函数接口功能类似, 不同的是makeRDD还提供了一个可以指定每一个分区preferredLocations参数的实现版本.

#### 存储创建操作

Spark的整个生态系统与Hadoop是完全兼容的, 所以对于Hadoop支持的文件类型或者数据库类型, Spark也同样支持.

由于Hadoop的Api有新旧两个版本, 所以Spark有两套接口.

> hadoopRDD 和 newHadoopRDD

参数: 1. 输入格式(InputFormat) :指定数据的输入类型

 	2. 键类型
	3. 值类型
	4. 分区值: 指定由外部存储生成的RDD的partition数量的最小值. 若没有指定, 系统使用默认值defautMinSplits.

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 下午12.46.16.png)

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 下午12.46.38.png)

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 下午12.47.28.png)

### 转换操作

#### RDD基本转换操作

1. ​

- map[U:ClassTag\](f:T=>U): RDD[U]

- distinct( ):RDD[T]

- flatMap[U:ClassTag\](f:T=>TraversableOnce[U]):RDD[U]

- repartition(numPartitions:Int): RDD[T]

- coalesce(numPartitions:Int, shuffle:Boolea=false): RDD[T]

  repartition和coalesce是对RDD的分区进行重新划分, reparation只是coalesce接口中shuffle为true的简易实现.

> coalesce合并函数如何设置shuffle参数?

假设RDD有N个分区, 需要重新划分成M个分区:

- N<M: 利用HashPartitioner函数将数据重新分区为M个, 这时需要将shuffle参数设置为true
- N>M且N和M相差不多:shuffle设置为false,(父子之间是窄依赖关系).将N个分区中的若干个分区合并成一个新的分区.
- N>M 且 N和M差距悬殊: shuffle设置为true. 如果是窄依赖,他们会同处在一个stage中,会造成Spark程序运行的并行度不够, 从而影响性能.

注: 当shuffle参数为false时,如果传入的参数大于现有分区数,name分区数保持不变, 也就是说在不进行shuffle洗牌的情况下,是无法将RDD的分区数目变多的.

2. ​

- randomSplit(weights:Array[Double],seed:Long=System.namoTime) : Array[RDD[T]]
- glom( ):RDD[Array[T]]

randomSplit函数是根据weights权重将一个RDD切分成多个RDD

glom函数是将RDD中每一个分区中类型为T的元素转换成数组Array[T],这样每一个分区就只有一个数组元素.

3. 集合操作

- union(other:RDD[T]) : RDD[T]
- intersection(other:RDD[T]) :RDD[T]
- intersection(other:RDD[T], partitioner:Partitioner)
- subtract(other:RDD[T]) : RDD[T]
- subtract(other:RDD[T],p:Partitioner):RDD[T]

这些是针对RDD的集合操作, union将两个RDD集合中的数据进行合并,返回两个RDD的并集(不会去重). 

intersection操作返回两个RDD集合的交集,且交集中不会包含相同的元素.

subtract: val result = A.substract(B), 则result中将会包含在A中出现且不在B中出现的元素.

intersection和substract一般情况下都会有shuffle的过程.

4. ​

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 下午1.30.10.png)

5. ​

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 下午1.37.48.png)

   ![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 下午1.39.30.png)

6.

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 下午1.41.44.png)

#### 键值RDD转换操作

1.

- partitionBy(partitioner:Partitioner):RDD[(K, V)]
- mapValues[U\](f:V=>U):RDD[(K,U)]
- flatMapValues[U\](f:V=>TraversableOnce[U]):RDD[(K,U)]

partionBy, mapValues和flatMapValues 与 基本转换操作中的repartition, map 和 flatMap功能类似. 

partitionBy接口根据partition函数生产新的ShuffleRDD,将原RDD重新分区(在repartition中也是现将RDD[T]转化为RDD[K, V], 这里的V是null, 然后使用RDD[K, V]作为参数生产ShuffleRDD)

mapValues和flatMapValues针对[K,V]中的V值进行map操作和flatMap操作.

2. ​

combineByKey[C\](createCombiner: V=>C, mergeValue):(C,V)=>C, mergeCombiners:(C, C) => C, partitioner:Partitioner, mapSideCombine:Boolean=true, serializer:Serializer = null):RDD[(K,C)]

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 下午2.56.43.png)

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 下午2.57.22.png)

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 下午3.01.21.png)

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 下午3.05.46.png)

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 下午3.06.38.png)

combineByKey接口的内部实现是分三步来完成的: 

首先根据是否需要在Map端进行combine操作决定是否对RDD先进行一次mapPartitions操作(利用createCombiner, mergeValue, mergeCombiners三个函数)来达到减小Shuffle数据量的作用.

第二步根据partitioner函数对MapPartitionsRDD进行shuffle操作, 

最后对于shuffle结果在进行一次combine操作.

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 下午3.14.55.png)

#### 再论RDD依赖关系

转换操作构建了RDD之间的大部分依赖关系

但是如果深入调试Spark中RDD依赖的数据结构就会发现, Spark内部生成的RDD对象一般多于用户书写的Spark应用程序中包含的RDD.

根本原因是: Spark的一些操作与RDD不是一一对应的.

### 控制操作

1. ​

- cache( ):RDD[T]
- persist( ):RDD[T]
- persist(level:StorageLevel) :RDD[T]

2. ​

- checkpoint

checkpoint接口是将RDD十九华在HDFS中, 与persist(如果也持久化在磁盘上)的一个区别是: checkpoint将会切断此RDD之前的依赖关系,而persist接口依然保留着RDD的依赖关系.

checkpoint的主要作用:

a. 如果一个Spark程序会长时间驻留运行, 过长的依赖将会占用很多系统资源,定期将RDD进行checkpoint操作,能够有效的节省系统资源 

b. 维护过长的依赖关系还会出现一个问题: 如果Spark在运行过程中出现节点失败的情况, 那么RDD进行容错重算的成本会非常高.\

### 行动操作

每一次行动操作,都会触发一次Spark的调度并返回相应的结果.

分类:

1. 行动操作将标量或者集合返回给Spark的客户端程序. 比如返回RDD中数据集的数量或者返回RDD中的一部分符合条件的数据
2. 行动操作将RDD直接保存到外部文件系统或者数据库中, 比如将RDD保存到HDFS文件系统中.

#### 集合标量行动操作

- first

- count

- reduce(f:(T,T)=>T) 对RDD中的元素进行二元计算, 返回计算结果

- collect( ) /toArray( ) : 以集合形式返回RDD的元素

- take( num: Int) 将RDD作为集合, 返回集合中[0, num-1]下标的元素

- top( num:Int ) 按照默认的或者是指定的排序规则, 返回前num个元素

- takeOrdered( num: Int ) :以与top相反的排序规则, 返回前num个元素

- aggregate[U\](zeroValue: U)(seqOp:(U,T)=>U, combOp(U,U)=>U)

  主要提供两个函数:

  一个是seqOp函数, 其将RDD(类型为T) 中的每一个分区的数据聚合成为类型为U的值.

  另一个函数combOp将各个分区聚合起来的值合并在一起得到最终类型为U的返回值.

- fold(zeroValue:T)(op:(T,T)=>T)

fold是aggregate的便利接口,其中op操作既seqOp操作也是combOp操作, 且最终的返回类型也是T. 即与RDD中每一个元素的类型是一样的.

- lookup(key:K):Seq[V]

  lookup是针对(K,V)类型的RDD的行动操作, 对于给定的键值,返回与次键值相对应的所有值.

#### 存储行动操作

由于Hadoop的API有新旧两个版本,所以Spark为了能够兼容Hadoop的所有版本,也提供了两套API

- saveAsTextFile(path:String)
- saveAsTextFile(path:String, codec:Class[_<: CompressionCodec])
- saveAsObjectFile(path:String)
- saveAsHadoopFile[F<:OutputFormat[K,V]\](path:String)
- saveAsHadoopFile[F<:OutputFormat[K,V]\](path:String,codec:Class[_<:CompressionCodec])
- saveAsHadoopFile(path:String, keyClass:Class[\_] valueClass:Class[_])
- outputFormatClass: Class[_ <: OutputFormat[\_,\_]],codec:Class\[_<:CompressionCodec])
- saveAsHadoopDataset(conf:JobConf)

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 下午4.35.08.png)

与旧版本的Hadoop Api的接口使用方法类似, 前两个API支持将RDD保存到HDFS中,而saveAsNewAPIHadoopDataSet则支持所有的MR兼容的输入输出类型.



##第3章 Spark运行模式及原理

###Spark运行模式概述

#### Spark运行模式列表

​	在实际应用中,Spark应用程序的运行模式取决于传递给SparkContext的MASTER环境变量的值, 个别模式还需要依赖辅助的程序接口来配合使用, 目前所支持的MASTER环境变量由特定的字符串或URL所组成.

- Local [ N ] : 本地模式, 使用N个线程
- Local cluster[worker, core, Memory] :伪分布式模式. 可以配置所需要启动的虚拟工作节点的数量.以及每个工作节点所管理的CPU数量和内存尺寸.
- Spark://hostname:port , Standalone模式, 需要部署Spark到相关节点, URL为Spark Master主机地址和端口
- Mesos://hostname:port.  Mesos模式. 需要部署Spark和Mesos到相关节点.URL为Mesos主机地址和端口
- YARN standalone/Yarn cluster: YARN模式一, 主程序逻辑和任务都运行在YARN集群中
- YARN client: YARN模式二, 主程序逻辑运行在本地, 具体任务运行在YARN集群中.

#### Spark基本工作流程

各运行模式从根本上都是将Spark的应用分为任务调度和任务执行两个部分.

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 下午6.46.42.png)

所有的Spark应用程序都离不开SparkContext和Executor两部分:

- Executor 负责执行任务, 运行Executor的机器成为Worker节点,
- SparkContext由用户程序启动, 通过资源调度模块与Executor通信

注意: SparkContext和Executor这两部分的核心代码实现在各种运行模式中都是共用的, 在它们之上, 根据部署模式的不同, 包装了不同调度模块以及相关的适配代码

SparkContext: 程序运行的总入口, 在其初始化过程中, Spark会分别创建DAGScheduler作业调度和TaskScheduler任务调度两级调度模块.

> 作业调度模块

基于任务阶段的高层调度模块, 它为每个Spark作业计算具有依赖关系的多个调度阶段(通常根据shuffle来划分),然后为每个阶段构建出具体的任务(通常会考虑数据的本地性等),然后以TaskSets(任务组)的形式提交给任务调度模块来具体执行.

> 任务调度模块

负责具体启动任务, 监控和汇报任务运行情况

Tips:

作业调度模块和具体的部署运行模式无关, 在各种运行模式下逻辑相同.不同的运行模式的区别主要体现在任务调度模块.不同的部署和运行模式, 根据底层资源调度方式的不同,各自实现了自己特定的任务调度模块, 用来将任务实际调度给对应的计算资源.

#### 相关基本类

>  为了抽象出一个公共的接口供DAGScheduler作业调度模块使用, 所有的这些运行模式实现的任务调度模块都是基于这两个Trait的:

1. TaskScheduler

   主要用于与DAGScheduler交互, 负责任务的具体调度和运行,其核心接口是:submitTasks 和 cancelTasks

2. SchedulerBackend

   与底层资源调度系统交互(如Mesos/YARN),配合TaskScheduler实现具体任务执行所需的资源分配,核心接口是receiveOffers

两者之间的实际交互过程取决于具体调度模式, 理论上这两者的实现是成对匹配工作的, 之所以拆成两部分,是有利于相似的调度模式共享代码共欧能模块.

> TaskSchedulerImpl

TaskSchedulerImpl实现了TaskScheduler接口,提供了大多数本地和分布式运行调度模式的任务调度接口.

此外还实现了resourceOffers和statusUpdate这两个接口供BackEnd调用,用于提供调度资源和更新任务状态.

> Executor

实际任务的运行,最终都由Executor类来执行. Executor对每一个任务创建一个TaskRunner类, 交给线程池运行. 运行的结果通过ExecutorBackend接口返回

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 下午7.19.40.png)

### Local模式

#### 部署及程序运行

Spark默认设置为Local模式.

调用方式:

1. 根据用户传入的参数来选择运行模式

   ./bin/run-example org.apache.spark.examples.SparkPi local

2. 在代码中配置Master为Local来实现

   val conf = new SparkConf( )

   ​		.setMaster("local")

   ​		.setAppName("My application")

   ​		.set("spark.executor.memory","1g")

   val sc = new SparkContext( conf )

Tip:为了使应用程序能够更灵活在各种部署环境下使用,不建议把相关设置写死.

#### 内部实现原理

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 下午7.29.46.png)

### Standalone模式

#### 部署及程序运行

Spark集群由Mater节点和Worker节点构成, 用户程序通过与Master节点交互,申请所需的资源, Worker节点负责具体Executor的启动运行

> 部署方式

只需要将编译好的或下载的Spark发布版本复制到打算用来构建Spark集群的各个节点上即可,为了方便通过启动脚本使用,部署的路径最好一致.

启动方式:

1) ./sbin/start-master.sh —>打印出自身的主机和端口信息

也可以在Master节点的网页上找到主机和端口信息

默认网址: http://MasterURL:8080

接下来在打算作为Worker的节点启动一个或者多个Worker, 启动命令如下:

./bin/spark-class org.apache.spark.deploy.worker.Worker spark://MasterURL:PORT
2)通过脚本启动集群

在conf下创建slaves的文件

免密钥

然后可以通过脚本来启动:

- sbin/start-master.sh
- sbin/start-slaves.sh
- sbin/start-all.sh: 执行上述两步
- sbin/stop-master.sh
- sbin/stop-slaves.sh
- sbin/stop-all.sh

注意: 这些脚本都需要在你打算作为Master节点运行的机器上执行.

#### 集群参数配置:

1. conf/spark-env.sh

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 下午9.19.46.png)

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 下午9.20.47.png)

#### 内部实现原理

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 下午9.25.46.png)

### Local cluster模式

#### 部署及程序运行

Local cluster伪分布式模式. 

在SparkContext初始化过程中, 在本地启动一个所有服务都在单机上运行的伪分布式Spark集群.

启动命令:

./bin/run-example org.apache.spark.examples.SparkPi local-cluster[2,2,1024]

#### 内部实现原理

基于Standalone模式来实现的.启动Master和Worker均在本地.

后续流程和资源调度流程与Standalone完全相同.

### Mesos模式

#### 部署及程序运行

### YARN standalone / YARN cluster模式

#### 部署及程序运行

YARN cluster模式, 顾名思义就是通过Hadoop YARN框架来调度Spark应用所需的资源.

YARN Standalone 在0.9版本前使用. 1.0以后改名为YARN cluster

首先需要部署一个YARN集群供该模式使用.实际应用中,因果关系往往是反的

#####打包

- 通过sbt assembly命令将所有的依赖关系打包成大的JAR包供YARN调度框架使用.
- 应用程序本身也需要打包成JAR包供YARN调度框架使用.

##### 配置

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 下午9.48.51.png)

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 下午9.49.42.png)

#### 内部实现原理

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 下午9.51.50.png)

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-22 下午9.55.09.png)

### YARN client模式

#### 部署及程序运行

#### 内部实现原理

YARN client 模式与YARN cluster模式的区别在于: 在YARN cluster模式中, 应用程序(包括Spark-Context)都是作为YARN框架所需要的Application Master,在由YARN ResourceManager为其分配的一个随机节点运行.

YARN client模式中, SparkContext运行在本地,该模式适用于应用程序本身需要在本地进行交互的场合.如Spark Shell, Shark等等.