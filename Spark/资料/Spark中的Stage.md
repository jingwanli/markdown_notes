#### **窄依赖和宽依赖**

##### **窄依赖**：

指父RDD的每一个分区最多被一个子RDD的分区所用，表现为一个父RDD的分区对应于一个子RDD的分区，和两个父RDD的分区对应于一个子RDD 的分区。图中，map/filter和union属于第一类，对输入进行协同划分（co-partitioned）的join属于第二类。

##### **宽依赖：**

指子RDD的分区依赖于父RDD的所有分区，这是因为shuffle类操作，如图中的groupByKey和未经协同划分的join。 

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-26 下午10.59.26.png)

##### **Stage**:

一个Job会被拆分为多组Task，每组任务被称为一个Stage就像Map Stage， Reduce Stage。Stage的划分在RDD的论文中有详细的介绍，简单的说是以shuffle和result这两种类型来划分。在Spark中有两类task，一类是shuffleMapTask，一类是resultTask，第一类task的输出是shuffle所需数据，第二类task的输出是result，stage的划分也以此为依据，shuffle之前的所有变换是一个stage，shuffle之后的操作是另一个stage。比如 rdd.parallize(1 to 10).foreach(println) 这个操作没有shuffle，直接就输出了，那么只有它的task是resultTask，stage也只有一个；如果是rdd.map(x => (x, 1)).reduceByKey(_ + _).foreach(println), 这个job因为有reduce，所以有一个shuffle过程，那么reduceByKey之前的是一个stage，执行shuffleMapTask，输出shuffle所需的数据，reduceByKey到最后是一个stage，直接就输出结果了。如果job中有多次shuffle，那么每个shuffle之前都是一个stage. 
会根据RDD之间的依赖关系将DAG图划分为不同的阶段，对于窄依赖，由于partition依赖关系的确定性，partition的转换处理就可以在同一个线程里完成，窄依赖就被spark划分到同一个stage中，而对于宽依赖，只能等父RDD shuffle处理完成后，下一个stage才能开始接下来的计算。之所以称之为ShuffleMapTask是因为它需要将自己的计算结果通过shuffle到下一个stage中 

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-26 下午10.59.33.png)

### **Stage划分思路**

因此spark划分stage的整体思路是：从后往前推，遇到宽依赖就断开，划分为一个stage；遇到窄依赖就将这个RDD加入该stage中。因此在图2中RDD C,RDD D,RDD E,RDDF被构建在一个stage中,RDD A被构建在一个单独的Stage中,而RDD B和RDD G又被构建在同一个stage中。 
　　在spark中，Task的类型分为2种：ShuffleMapTask和ResultTask；简单来说，DAG的最后一个阶段会为每个结果的partition生成一个ResultTask，即每个Stage里面的Task的数量是由该Stage中最后一个RDD的Partition的数量所决定的！而其余所有阶段都会生成ShuffleMapTask；之所以称之为ShuffleMapTask是因为它需要将自己的计算结果通过shuffle到下一个stage中；也就是说图2中的stage1和stage2相当于mapreduce中的Mapper,而ResultTask所代表的stage3就相当于mapreduce中的reducer。

### **总结**

map,filtre为窄依赖， 
groupbykey为款依赖 
遇到一个宽依赖就分一个stage