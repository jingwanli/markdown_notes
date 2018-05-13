## 复习spark core

8080:ui

4040: 某节点 worker

18080: history server

7077: spark

spark1.4版本最大的变化: 引入Spark R

Spark Job默认的调度模式: FIFO(一般的调度模式都是FIFO)

广播变量: 错误的是: 存储在磁盘或 HDFS

累加器: 可以自定义.

stage的Task数量: 由Partition决定

spark的master和worker通过什么方式通信?: 

rpc间的通信模型: akka(scala自带的一种通信方式)

​				1.6以后: 用的是netty

spark.deploy.recoveryMode支持哪种?

1.zookeeper 2.文件系统 3. 自定义 4. none

hive的元数据存储在derby 和 mysql有什么区别?: 多会话

Master的electedleader事件后做了哪些操作?

直接Alive

---

## 高级输入源--Kafka

Kafka:高吞吐量, 分布式, 快速, 可扩展, 分区(topic可以分区)和复制,基于发布/订阅模式的消息系统.采用scala编写的.

扩展: 水平扩展

  topic是可以分区的.

目前最高版本: kafka-2.11-1.0  (2.11指的是scala编译版本. 1.0 kafka版本)

卡夫卡:支持在线实时处理 和 批量离线处理.

消息可以被持久化到磁盘.所以可以批量离线处理.且不影响kafka的性能

### kafka的特性

o(1) : 是个常数. 虽然数据量规模不断增加,但是效率和服务基本没有变化的.数组的查找和长度有关,不是o(1),但是在vector(scala中的)中查找,是个常数(1层32,4层100w)

kafka设计目的: 提供一个发布订阅解决方案, 面向系统交换数据的! (高速,大容量—kafka是面向系统来进行交互数据的)

#### 基本组件/术语

1. broker : kafka一个服务器. 一个集群由多个broker组成,一个broker可以有多个topic. 介于生产者和消费者之间.
2. topic: 一个topic可以认为是一类消息, 一个消息由若干partition.
3. producer: push
4. consumer: pull

一个topic会分若干个副本. 

元数据: 都放在zookeeper中.但是zookeeper不适合频繁的访问.(效率比较低)

5. replication: 备份

   topic分成了若干个partition, partition做备份

   每个broker最多只有一个replications

6. Consumer Group 消费者群组

   单播 和 多播(发给所有consumer)

7. Partition(分区)

   备份: 最终持久化到磁盘. 

   默认168小时,保证消息在.

   数据是要刷到磁盘的.

   可以自定义保存策略, 默认是168小时,7天.

   168小时候,不保证消息仍然存在.

8. offset 偏移量

   消息是顺序保存的,kafka使用一个唯一的标识记录他们在该分区下的位置, 称为offset. offset只能递增,不能减! 

   从头读, 从尾写,顺序写.

   每份文件: 存一个数据本身, 一份索引.(索引其实就是偏移量,定位文件用)

   消息的确定: topic, partition, offset

   如果不分区, 可以保证消息的顺序

   如果分区,只能保证分区内的顺序.

#### Kafka设计原理

1. 持久化 

   有缓存!不是一条消息就持久化到磁盘

2. 生产者

   同步

   异步: 高效,但容易丢数据

3. 消费者

   why采取pull的方式?

   解耦

   offset是会移动的,当消息处理后

   采用pull模式:

   - broker不需要感知有多少个consumer
   - 如果采用push模式,一旦消息量级超过consumer的承受范围,会压垮consumer

   consumer消费完一条消息, kafka会自动的提交offset

4. 常量

5. 消息分发机制

   at most once: 消息可能会丢,但绝不会重复传输

   at least once : 消息绝不会丢, 但可能会重复传输

   exactly once:每条消息肯定会被传输一次且仅传输一次, 多数情况下这才是我们所追求的.

   ![](/Users/lijingwan/Documents/img/屏幕快照 2018-05-09 下午7.31.31.png)

   

6. 复制Replication

   Replication分为主从.主的当掉后,副本要参与竞争,选出新的leader.  

zookeeper: 1. 做协调 2. 写一部分元数据 

cd $KAFKA_HOME

cd data

ls: ..log:数据文件    .index索引文件

zkCli.sh -server master:2181

ls /

get /brokers/topics/…….

==zookeeper: 版本: 3.4.11==

```scala
//命令行中:
cd $KAFKA_HOME
cd data
ls
cd mykafka1-0
ls //可以看到1个索引文件...index, 1个数据文件...log
还可以看到 consumer_offsets-3 //消费的信息做成了一个topic,不是放在zookeeper里,而是放在了kafka里.
--------------
zkCli.sh -server master:2181
ls /
//可以看到brokers
ls /brokers
ls /brokers/topics
ls /brokers/topics/mykafka1
ls /brokers/topics/mykafka1/parititions/0
//注: 最后一层看的语法不一样,要用get!
get /brokers/topics/mykafka1/partitions/0/state
```

> 版本号

zookeeper: 3.4.11

kafka_2.11-0.11.0.2—kafka最高版本是1.0

2.11是编译kafka使用的scala的版本号

 生产环境中: 不要用kill -9   建议用kill -15 —>为了不丢数据!

在config/server.properties配置文件中加入: 

 delete.topic.enable=true //无用的topic做真正的删除     //不设置的话, 只是打了个标记!

​	config/consumer.properties中修改:

​	zookeeper.connect=master:2181,slave1:2181,slave2:2181

> 启动kafka

kafka-server-start.sh -daemon ../config/server.properties

jps: 看到kafka进程

然后远程拷贝到其他节点!

然后启动kafka: kafka-server-start.sh  -daemon ../config/server.properties

---

想要让zk中显示为:

zookeeper, kafka0.11 //即将其他都放入kafka0.11文件夹

how? ---修改配置文件:

\#consumer.properties (3台都要改)

==zookeeper.connect=master:2181,slave1:2181,slave2:2181/kafka0.11==

### kafka常用命令

1. 创建主题:

   kafka-topics.sh - -zookeeper node1:2181/kafka0.11 - -create - -topic mykafka1 - -partition  5   - -replication-factor 2

   注:broker的个数小于等于节点数, 分区数大于等于broker的个数. replication-factor必须小于等于broker的个数

   //zookeeper后面加任意一个节点即可!

2. 列出所有主题

   kafka-topics.sh - -zookeeper master:2181/kafka0.11 - -list

   //kafka0.11必须加!

3. 看详细主题信息(把参数跟全!)

   kafka-topics.sh - -zookeeper master:2181/kafka0.11  - -describe - -topic mykafka1

4. 在控制台上去创建一个生产者 

   kafka-console-producer.sh - -broker-list master:9092,slave1:9092, slave2:9092 - -topic mykafka1

   //控制台输入内容(生产者)

   //在另一台节点启动一个消费者

   kafka-console-consumer.sh - -bootstrap-server master:9092,slave1:9092,slave2:9092 - -topic mykafka1

   //数据的顺序不保证!(如果只有一个partition, 是保证顺序的)

5. 创建消费者

   kafka-console-consumer.sh --bootstrap-server node3:9092,node3:9093 --topic mykafka1

   \# 也可以使用，会有警告这是一个将被淘汰的命令

   kafka-console-consumer.sh –zookeeper node2:12181/kafka0.11 --topic mykafka1

6. 查询 topic 的所有记录数

   kafka-run-class.sh kafka.tools.GetOffsetShell --topic mykafka1 --broker-list master:9092, slave1:9092,slave2:9092

   //结果: mykafka1:2:4:1 分别是:topic:分区数: 消息数

   \# 查询 topic 在分区上的记录数

   kafka-run-class.sh kafka.tools.GetOffsetShell --topic mykafka1 --broker-list master:9092, slave1:9092,slave2:9092 --partitions 0

7. 查询 topic 的消费者情况

   kafka-consumer-groups.sh --bootstrap-server master:9092, slave1:9092,slave2:9092 --describe --group 1

   //查看group

   more config/consumer-properties 查看group id

   zkCli.sh -server master:2181

   get /consumers ———查看是否有消费组的情况,(没有找到)

### Spark配置

编写kafka生产者程序, 需要调用kafka的API, 对应的jar包在$KAFKA_HOME/lib中

编写kafka消费者程序,需要调用Spark Streaming的API,对应的jar需要下载

如果不适用maven管理:

 cd $KAFKA_HOME

ls

cd libs —>

cd $SPARK_HOME/jars—> 新建目录mkdir kafka0.11

将libs所有的包放入kafka0.11 然后重新添加

#### 生产者程序 KafkaWordProducer.scala

```scala
import java.util
import org.apache.kafka.clients.producer.{KafkaProducer,ProducerConfig}   
object KafkaWordProducer{
    def main(args:Array[String]):Unit{
        val brokers = "master:9092,slave1:9092,slave2:9092"
        val topic = "mykafka1"
        val props = new util.HashMap[String,Object]()
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,brokers)
     props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer")
props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,   "org.apache.kafka.common.serialization.StringSerializer")
//放进去的是key value对!key用来做分区
//value是消息
    val producer = new KafkaProducer[String, String](props)

    while(true) {
      for (i <- 1 to messagesPerSec) {
        val str = (1 to wordsPerMessage).map(x => Random.nextInt(10).toString).mkString(" ")
        println(str)
        val message = new ProducerRecord[String, String](topic, null, str)
        producer.send(message)
      }
      println("*********************************************************")
      Thread.sleep(1000)
    }  }}

```

```scala
要只发送一条消息的话:
val msg = new ProducerRecord[String,String](topic,null,"xxx yyy zzz")
producer.send(msg)
producer.close() //这里一定要close,否则发不出去!
---------------------
命令行中:
kafka-console-consumer.sh --bootstrap-server master:9092,slave1:9092,slave2:9092 --topic mykafka1
----------------------
多条消息:
val random = new Random()
while(true){
        val str = (1 to 5).map(x=>random.nextInt(10).toString).mkString(" ")
        println(str)
    	val msg = new ProducerRecord[String,String](topic,null,str)
		producer.send(msg) //close不需要,因为一直在发 
    	Thread.sleep(500)
}
```

//查看maven如何管理的包:

File—>Project Structure

gedit &: 打开记事本!

//配置参数从哪里来? 方式1 与brokers打交道, 方式2 与zookeeper打交道

```scala
object KafkaStreaming1{
    def main(args:Array[String]):Unit={
        val conf = new SparkConf().setAppName("Kafka Streaming1").setMaster("local[2]")
        //10s处理一次
        val ssc = new StreamingContext(conf,Seconds(10))
        ssc.sparkContext.setLogLevel("WARN")
        //这里不仅要知道topic,而且要知道topic的分区
        val topic = Map("mykafka1"->5)
        val zkQuorum="master:2181,slave1:2181,slave2:2181/kafka0.11"
        //test1为group名,而且lines为key value对,key对我们没有用,value为具体消息
        val lines = KafkaUtils.createStream(ssc,zkQuorum,"test1",topic).map(_._2)
        
        val words = lines.flatMap(_.split("\\s+"))
        val wordCounts = words.map((_,1)).reduceByKey(_+_)
        wordCounts.print()
        
        ssc.start()
        ssc.awaitTermination()   
    }
    
}
//消费者消费消息的版本: 1. broker-list 高版本
					2. zookeeper 低版本
消费者://另一个版本:(命令行中)
kafka-console-consumer.sh --zookeeper master:2181,slave1:2181,slave2:2181/kafka0.11 --topic mykafka1
```

查询group:  (排错的时候用的多)

kafka-consumer-groups.sh --bootstrap-server master:9092, slave1:9092,slave2:9092 --describe --group test1 //(没有查到)

zookeeper中: 命令行:

get /kafka0.11/consumers/test1/offsets/mykafka1/0

//列出所有的组: (non-zookeeper-based consumers)

//为经过zookeeper的组查询(使用bootstrap的形式)

kafka-consumer-groups.sh - -bootstrap-server master:9092,slave1:9092,slave2:9092  - -list

//经过zookeeper的组查询:

kafka-consumer-groups.sh  - -zookeeper master:2181,slave1:2181,slave2:2181/kafka0.11 - -describe  - -group test1

---

以前的版本: 0.8之前: 消费者的api:  scala, zookeeper(offset), 系统管理offset

0.8之后的版本 :新消费者API: java/broker-list管理, offset(消息队列) ,允许自己管理offset

//offset放到消息队列中.zookeeper不适合大量的频繁交互,性能有影响

---

kafka: 选举机制,zk选出一个leader 是分布式. 有容错.数据不同的topic 分区.

flume: 只有采集节点和汇聚节点! 不是分布式,没有容错.

> 区分

高可用: 一个设备工作, down掉后,转换成另一个机器.

冗余: 2个机器同时工作, 一个down掉,另一个还可以继续使用