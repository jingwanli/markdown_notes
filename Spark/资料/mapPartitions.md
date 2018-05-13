rdd的mapPartitions是map的一个变种，它们都可进行分区的并行处理。

​    两者的主要区别是调用的粒度不一样：map的输入变换函数是应用于RDD中每个元素，而mapPartitions的输入函数是应用于每个分区。

​    假设一个rdd有10个元素，分成3个分区。如果使用map方法，map中的输入函数会被调用10次；而使用mapPartitions方法的话，其输入函数会只会被调用3次，每个分区调用1次。

   

​    //生成10个元素3个分区的rdd a，元素值为1~10的整数（1 2 3 4 5 6 7 8 9 10），sc为SparkContext对象

​    val a = sc.parallelize(1 to 10, 3)

​    //定义两个输入变换函数，它们的作用均是将rdd a中的元素值翻倍

​    //map的输入函数，其参数e为rdd元素值   

​    def myfuncPerElement(e:Int):Int = {

​           println("e="+e)

​           e*2

​      }

​     //mapPartitions的输入函数。iter是分区中元素的迭代子，返回类型也要是迭代子

​    def myfuncPerPartition ( iter : Iterator [Int] ) : Iterator [Int] = {

​         println("run in partition")

​         var res = for (e <- iter ) yield e*2

​          res

​    }

​    

​    val b = a.map(myfuncPerElement).collect

​    val c =  a.mapPartitions(myfuncPerPartition).collect

​    在[Spark](http://lib.csdn.net/base/spark) shell中运行上述代码，可看到打印了3次run in partition，打印了10次e=。

 

​      从输入函数（myfuncPerElement、myfuncPerPartition）层面来看，map是推模式，数据被推到myfuncPerElement中；mapPartitons是拉模式，myfuncPerPartition通过迭代子从分区中拉数据。

 

​    这两个方法的另一个区别是在[大数据](http://lib.csdn.net/base/hadoop)集情况下的资源初始化开销和批处理处理，如果在myfuncPerPartition和myfuncPerElement中都要初始化一个耗时的资源，然后使用，比如[数据库](http://lib.csdn.net/base/mysql)连接。在上面的例子中，myfuncPerPartition只需初始化3个资源（3个分区每个1次），而myfuncPerElement要初始化10次（10个元素每个1次），显然在大数据集情况下（数据集中元素个数远大于分区数），mapPartitons的开销要小很多，也便于进行批处理操作。

​    

####    mapPartitionsWithIndex和mapPartitons类似，只是其参数多了个分区索引号。

转载：http://wanshi.iteye.com/blog/2183906

\

eg:分区到底是怎么运行的？

scala> def myfunc[T](iter:Iterator[T]):Iterator[(T,T)]={
     | var res = List[(T,T)]()
     | var pre = iter.next
     | while (iter.hasNext){
     | val cur =iter.next
     | res.::=(pre,cur)
     | pre = cur
     | }
     | res.iterator
     | }
myfunc: [T](iter: Iterator[T])Iterator[(T, T)]
scala> val rdd1 = sc.parallelize(1 to 9,3)
rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[0] at parallelize at <console>:27
scala> rdd1 .collect
[Stage 0:>                                                          (0 + 0) / 3]17/02/21 20:47:10 WARN SizeEstimator: Failed to check whether UseCompressedOops is set; assuming yes
res0: Array[Int] = Array(1, 2, 3, 4, 5, 6, 7, 8, 9)                             
scala> val rdd5 = rdd1 .mapPartitions(myfunc)
rdd5: org.apache.spark.rdd.RDD[(Int, Int)] = MapPartitionsRDD[1] at mapPartitions at <console>:31
scala> rdd5.collect
res1: Array[(Int, Int)] = Array((2,3), (1,2), (5,6), (4,5), (8,9), (7,8))
scala> 
scala> def myfunc1(e:Int):Int ={
     | println("e="+e)
     | e*2
     | }
myfunc1: (e: Int)Int
scala> def func2(iter:Iterator[Int]):Iterator[Int]={
     | println("run in partition")
     | var res = for(e<-iter)yield e*2
     | res
     | }
func2: (iter: Iterator[Int])Iterator[Int]
scala> val b = rdd1.map(myfunc1).collect
e=4
e=5
e=6
e=7
e=1
e=2
e=3
e=8
e=9
b: Array[Int] = Array(2, 4, 6, 8, 10, 12, 14, 16, 18)
scala> val c = rdd1.mapPartition(func2).collect()
<console>:31: error: value mapPartition is not a member of org.apache.spark.rdd.RDD[Int]
         val c = rdd1.mapPartition(func2).collect()
                      ^
scala> val c = rdd1.mapPartitions(func2).collect()
run in partition
run in partition
run in partition
c: Array[Int] = Array(2, 4, 6, 8, 10, 12, 14, 16, 18)
scala> 