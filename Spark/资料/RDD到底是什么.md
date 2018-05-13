# 前言

　　用Spark有一段时间了，但是感觉还是停留在表面，对于Spark的RDD的理解还是停留在概念上，即只知道它是个弹性分布式数据集，其他的一概不知

有点略显惭愧。下面记录下我对RDD的新的理解。

 

# 官方介绍

　　 弹性分布式数据集。 RDD是只读的、分区记录的集合。RDD只能基于在稳定物理存储中的数据集和其他已有的RDD上执行确定性操作来创建。

# 问题

​      只要你敢问度娘RDD是什么，包你看到一大片一模一样的答案，都是说这样的概念性的东西，没有任何的价值。

​      我只想知道 RDD为什么是弹性 而不是 不弹性， RDD到底是怎么存数据，在执行任务的过程中是咋哪个阶段读取数据。

 

# 什么是弹性

​    我的理解如下（若有误或不足，烦请指出更正）： 

​            1. RDD可以在内存和磁盘之间手动或自动切换

​            2. RDD可以通过转换成其他的RDD，即血统

​            3. RDD可以存储任意类型的数据

 

# 存储的内容是什么

​     根据编写Spark任务的代码来看，很直观的感觉是RDD就是一个只读的数据，例如  rdd.foreach(println)

​     但是不是， RDD其实不存储真是的数据，只存储数据的获取的方法，以及分区的方法，还有就是数据的类型。

​     百闻不如一见， 下面看看RDD的源码：

　　通过RDD的这两个抽象方法，我们可以看出 ：

​                          RDD其实是不存储真是数据的，存储的的只是 真实数据的分区信息getPartitions*，还有就是针对单个分区的读取方法* compute

​    到这里可能就有点疑惑，要是RDD只存储这分区信息和读取方法，那么RDD的依赖信息是怎么保存的？ 

​     其实RDD是有保存的，只是我粘贴出的只是RDD顶层抽象类，还要一点需要注意 ，**RDD只能向上依赖**，而真正实现这两个方法的RDD都是整个任务的输入端，即处于RDD血统的顶层，初代RDD 

​     举个例子：val rdd = sc.textFile(...); val rdd1 = rdd.map(f)  .  这里的 rdd是初代RDD， 是没有任何依赖的RDD的，所以没就没有保存依赖信息， 而 rdd1是子代RDD，那么它就必须得记录下自己是来源于谁，也就是血统，

   下面展示的是HadoopRDD和  MapPartitionsRDD

 

 //负责记录数据的分区信息  和 读取方法 

class HadoopRDD[K, V](
　　@transient sc: SparkContext,
　　broadcastedConf: Broadcast[SerializableConfiguration],
　　initLocalJobConfFuncOpt: Option[JobConf => Unit],
　　inputFormatClass: Class[_ <: InputFormat[K, V]],
　　keyClass: Class[K],
　　valueClass: Class[V],
　　minPartitions: Int)
　　extends RDD[(K, V)](sc, Nil) with Logging {

  override def getPartitions: Array[Partition] = { ***篇幅所限  自己查看**}

   override def compute(theSplit: Partition, context: TaskContext): InterruptibleIterator[(K, V)] = {***篇幅所限  自己查看**}

}

 

//子代RDD的作用起始很简单  就是记录初代RDD到底在干了什么才得到了自己

*private[spark] class MapPartitionsRDD[U: ClassTag, T: ClassTag](*

 

　　到这里，我们就大概了解了RDD到底存储了什么东西，

​               初代RDD: 处于血统的顶层，存储的是任务所需的数据的分区信息，还有单个分区数据读取的方法，没有依赖的RDD， 因为它就是依赖的开始。

​              子代RDD: 处于血统的下层， 存储的东西就是 初代RDD到底干了什么才会产生自己，还有就是初代RDD的引用

现在我们基本了解了RDD里面到底存储了些什么东西，那么问题就来了，到底读取数据发生在什么时候。

# 数据读取发生在什么时候

   直接开门见山的说， 数据读取是发生在运行的Task中，也就是说，数据是在任务分发的executor上运行的时候读取的，上源码：

final def iterator(split: Partition, context: TaskContext): Iterator[T] = {
　　if (storageLevel != StorageLevel.NONE) {

​          //先判断是否有缓存 ，有则直接从缓存中取 ， 没有就从磁盘中取出来， 然后再执行缓存操作 
　　　　SparkEnv.get.cacheManager.getOrCompute(this, split, context, storageLevel) 
　　} else {

​          //直接从磁盘中读取 或 从 检查点中读取 
　　　　computeOrReadCheckpoint(split, context)
　　}
}

　　在spark中的任务 最终是会被分解成多个TaskSet到executor上运行，TaskSet的划分是根据是否需要shuffle来的。

​     在spark中就只有两种Task，一种是ResultTask ，一种是ShuffleTask， 两种Task都是以相同的方式读取RDD的数据。