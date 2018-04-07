http://blog.csdn.net/looklook5/article/details/41749523

**Storm** **与Kafka 整合**

这里的目标是kafka 负责生产数据，storm 消费数据并将结果输出

一、[wurstmeister](https://github.com/wurstmeister)/[storm-kafka-0.8-plus](https://github.com/wurstmeister/storm-kafka-0.8-plus)

这里用的是引进别人家写的整合代码，因为使用的人也比较多，下面是项目地址

<https://github.com/wurstmeister/storm-kafka-0.8-plus>

下载、解压以及将这个目录下的代码添加进项目

[storm-kafka-0.8-plus-master\storm-kafka-0.8-plus-master\src\jvm](http://blog.csdn.net/looklook5/article/details/storm-kafka-0.8-plus-master/storm-kafka-0.8-plus-master/src/jvm)

 

将kafka 和 storm 的JAR 添加进项目，作为依赖jar 包

然后添加com.netflix.curator 的相关包括client、framework和recipes

下载地址：<http://maven.outofmemory.cn/com.netflix.curator/>

最新的所有com.google.common类，下载地址

[http://central.maven.org/maven2/com/google/guava/guava/18.0/guava-18.0.jar](http://blog.csdn.net/looklook5/article/details/storm-kafka-0.8-plus-master/storm-kafka-0.8-plus-master/src/jvm)

http://central.maven.org/maven2/com/google/guava/guava/18.0/

 

这样storm-kafka-0.8-plus项目应该就不会报错了。

 

二、kafka 生产者的创建

在我的这篇文章里3.6、ProducerJAVA API，有生产者的例子，可以拿来直接用。

<http://blog.csdn.net/looklook5/article/details/41248561>

 

三、创建消费 kafka 数据的Topology

storm-kafka-0.8-plus 给我们写了个测试代码

地址是：

<https://github.com/wurstmeister/storm-kafka-0.8-plus-test/blob/master/src/main/java/storm/kafka/KafkaSpoutTestTopology.java>

代码如下：

**[java]** [view plain](http://blog.csdn.net/looklook5/article/details/41749523) [copy](http://blog.csdn.net/looklook5/article/details/41749523)

1.  **package** storm.kafka;  

2.    

3.  **import** backtype.storm.Config;  

4.  **import** backtype.storm.LocalCluster;  

5.  **import** backtype.storm.StormSubmitter;  

6.  **import** backtype.storm.generated.StormTopology;  

7.  **import** backtype.storm.spout.SchemeAsMultiScheme;  

8.  **import** backtype.storm.topology.BasicOutputCollector;  

9.  **import** backtype.storm.topology.OutputFieldsDeclarer;  

\10. **import** backtype.storm.topology.TopologyBuilder;  

\11. **import** backtype.storm.topology.base.BaseBasicBolt;  

\12. **import** backtype.storm.tuple.Tuple;  

\13. **import** org.slf4j.Logger;  

\14. **import** org.slf4j.LoggerFactory;  

\15.   

\16. **import** java.util.Arrays;  

\17.   

\18. **public** **class** KafkaSpoutTestTopology {  

\19.     **public** **static** **final** Logger LOG = LoggerFactory.getLogger(KafkaSpoutTestTopology.**class**);  

\20.   

\21.     **public** **static** **class** PrinterBolt **extends** BaseBasicBolt {  

\22.         @Override  

\23.         **public** **void** declareOutputFields(OutputFieldsDeclarer declarer) {  

\24.         }  

\25.   

\26.         @Override  

\27.         **public** **void** execute(Tuple tuple, BasicOutputCollector collector) {  

\28.             LOG.info(tuple.toString());  

\29.         }  

\30.   

\31.     }  

\32.   

\33.     **private** **final** BrokerHosts brokerHosts;  

\34.   

\35.     **public** KafkaSpoutTestTopology(String kafkaZookeeper) {  

\36.         brokerHosts = **new** ZkHosts(kafkaZookeeper);  

\37.     }  

\38.   

\39.     **public** StormTopology buildTopology() {  

\40.         SpoutConfig kafkaConfig = **new** SpoutConfig(brokerHosts, "storm-sentence", "", "storm");  

\41.         kafkaConfig.scheme = **new** SchemeAsMultiScheme(**new** StringScheme());  

\42.         TopologyBuilder builder = **new** TopologyBuilder();  

\43.         builder.setSpout("words", **new** KafkaSpout(kafkaConfig), 10);  

\44.         builder.setBolt("print", **new** PrinterBolt()).shuffleGrouping("words");  

\45.         **return** builder.createTopology();  

\46.     }  

\47.   

\48.     **public** **static** **void** main(String[] args) **throws** Exception {  

\49.   

\50.         String kafkaZk = args[0];  

\51.         KafkaSpoutTestTopology kafkaSpoutTestTopology = **new** KafkaSpoutTestTopology(kafkaZk);  

\52.         Config config = **new** Config();  

\53.         config.put(Config.TOPOLOGY_TRIDENT_BATCH_EMIT_INTERVAL_MILLIS, 2000);  

\54.   

\55.         StormTopology stormTopology = kafkaSpoutTestTopology.buildTopology();  

\56.         **if** (args != **null** && args.length > 1) {  

\57.             String name = args[1];  

\58.             String dockerIp = args[2];  

\59.             config.setNumWorkers(2);  

\60.             config.setMaxTaskParallelism(5);  

\61.             config.put(Config.NIMBUS_HOST, dockerIp);  

\62.             config.put(Config.NIMBUS_THRIFT_PORT, 6627);  

\63.             config.put(Config.STORM_ZOOKEEPER_PORT, 2181);  

\64.             config.put(Config.STORM_ZOOKEEPER_SERVERS, Arrays.asList(dockerIp));  

\65.             StormSubmitter.submitTopology(name, config, stormTopology);  

\66.         } **else** {  

\67.             config.setNumWorkers(2);  

\68.             config.setMaxTaskParallelism(2);  

\69.             LocalCluster cluster = **new** LocalCluster();  

\70.             cluster.submitTopology("kafka", config, stormTopology);  

\71.         }  

\72.     }  

\73. }  

这里清晰的写出了创建一个与kafka整合的storm Topology,观察main 函数，从上往下看：

下面是关于zookeeper的设定以及spout和bolt 的设定

**[java]** [view plain](http://blog.csdn.net/looklook5/article/details/41749523) [copy](http://blog.csdn.net/looklook5/article/details/41749523)

1.  String kafkaZk = args[0];  

2.  KafkaSpoutTestTopology kafkaSpoutTestTopology = **new** KafkaSpoutTestTopology(kafkaZk);  

3.  StormTopology stormTopology = kafkaSpoutTestTopology.buildTopology();  

下面的语句中，storm-sentence是话题，下面的语句是要求在zookeeper 服务器中在根目录创建文件夹storm，用于kafka存放zookeeper相关数据

**[java]** [view plain](http://blog.csdn.net/looklook5/article/details/41749523) [copy](http://blog.csdn.net/looklook5/article/details/41749523)

1.  SpoutConfig kafkaConfig = **new** SpoutConfig(brokerHosts, " storm-sentence ", "", "storm");  

2.  builder.setSpout("words", **new** KafkaSpout(kafkaConfig), 10); 这里是设定spout，负责从kafka消费数据，其中word 是spout 名称，KafkaSpout 由storm-kafka-0.8-plus 提供，10为并发数。  

3.  builder.setBolt("print", **new** PrinterBolt()).shuffleGrouping("words"); 这个是设定spout 接下去的bolt, PrinterBolt看名称应该负责打印bolt的数据的类。shuffleGrouping("words")表示数据是采用随机模式。后面接的数据来自与叫做words的spout  

下面是设置Topology的相关设定

**[java]** [view plain](http://blog.csdn.net/looklook5/article/details/41749523) [copy](http://blog.csdn.net/looklook5/article/details/41749523)

1.  Config config = **new** Config(); 初始化一个storm设置  

2.  config.setNumWorkers(2);  这个代表分配2个Worker。  

3.  StormSubmitter.submitTopology(args[0], config, builder.createTopology()); 这个表示想Storm 服务器提交Topology任务，其中第一个参数是Topology的name.  

4.  config.setMaxTaskParallelism(3); 一个work的最大并发数为3  

5.  LocalCluster cluster = **new** LocalCluster(); 开启Storm本地模式   

6.  cluster.submitTopology("special-topology", config, builder.createTopology());  在本地网模式下提交storm任务。                  

7.  cluster.shutdown(); 关闭Storm本地模式。  

下面是我修改后的脚本

**[java]** [view plain](http://blog.csdn.net/looklook5/article/details/41749523) [copy](http://blog.csdn.net/looklook5/article/details/41749523)

1.  **import** com.google.common.collect.ImmutableList;  

2.  **import** com.ks.bolt.CounterBolt;  

3.  **import** com.ks.bolt.DateCutBolt;  

4.  **import** com.ks.bolt.InsertMysqlBolt;  

5.    

6.  **import** storm.kafka.BrokerHosts;  

7.  **import** storm.kafka.KafkaSpout;  

8.  **import** storm.kafka.SpoutConfig;  

9.  **import** storm.kafka.StringScheme;  

\10. **import** storm.kafka.ZkHosts;  

\11. **import** backtype.storm.Config;  

\12. **import** backtype.storm.LocalCluster;  

\13. **import** backtype.storm.StormSubmitter;  

\14. **import** backtype.storm.spout.SchemeAsMultiScheme;  

\15. **import** backtype.storm.topology.TopologyBuilder;  

\16. **public** **class** CountTopology {  

\17.   

\18.     /** 

\19.      * @param args 

\20.      */  

\21.     **public** **static** **void** main(String[] args) {  

\22.         **try**{  

\23.             String kafkaZookeeper = "carl:2181,slave1:2181,slave2:2181";  

\24.             BrokerHosts brokerHosts = **new** ZkHosts(kafkaZookeeper);  

\25.             SpoutConfig kafkaConfig = **new** SpoutConfig(brokerHosts, "test", "/storm", "stormid");  

\26.             kafkaConfig.scheme = **new** SchemeAsMultiScheme(**new** StringScheme());  

\27.             kafkaConfig.zkServers =  ImmutableList.of("carl","slave1","slave2");  

\28.             kafkaConfig.zkPort = 2181;  

\29.               

\30.             //kafkaConfig.forceFromStart = true;  

\31.               

\32.               

\33.             TopologyBuilder builder = **new** TopologyBuilder();  

\34.             builder.setSpout("spout", **new** KafkaSpout(kafkaConfig), 2);  

\35.           //*************************下面是所有处理逻辑，只关注这个*****************************  

\36.             builder.setBolt("datecut", **new** CounterBolt(),1).shuffleGrouping("spout");  

\37.             //*************************下面是所有处理逻辑，只关注这个*****************************  

\38.   

\39.             Config config = **new** Config();  

\40.             config.setDebug(**true**);  

\41.               

\42.             **if**(args!=**null** && args.length > 0) {  

\43.                 config.setNumWorkers(2);  

\44.                   

\45.                 StormSubmitter.submitTopology(args[0], config, builder.createTopology());  

\46.             } **else** {          

\47.                 config.setMaxTaskParallelism(3);  

\48.           

\49.                 LocalCluster cluster = **new** LocalCluster();  

\50.                 cluster.submitTopology("special-topology", config, builder.createTopology());  

\51.                   

\52.                 Thread.sleep(500000);  

\53.       

\54.                 cluster.shutdown();  

\55.             }  

\56.         }**catch** (Exception e) {  

\57.             e.printStackTrace();  

\58.         }  

\59.     }  

\60.   

\61. }  

这里在本地模式下让他运行20秒钟自动结束，因为这个比较耗资源。注意以下这句，

**[java]** [view plain](http://blog.csdn.net/looklook5/article/details/41749523) [copy](http://blog.csdn.net/looklook5/article/details/41749523)

1.  SpoutConfig kafkaConfig = **new** SpoutConfig(brokerHosts, "test", "/storm", "stormid");  

请记得在zookeeper 根目录下面创建文件夹storm,然后在storm 文件夹下面继续创建文件夹stormid 用于存放kafka信息数据

上面的Topology 设定了bolt为CounterBolt，因此还要建一个CounterBolt的bolt 类。

这里设定了，运行jar包敲参数为提交到storm服务器，不敲参数则是运行storm本地模式。

四、创建数据输出的Bolt

这里实现一个十分简单的bolt 类

**[java]** [view plain](http://blog.csdn.net/looklook5/article/details/41749523) [copy](http://blog.csdn.net/looklook5/article/details/41749523)

1.  **import** backtype.storm.topology.BasicOutputCollector;  

2.  **import** backtype.storm.topology.OutputFieldsDeclarer;  

3.  **import** backtype.storm.topology.base.BaseBasicBolt;  

4.  **import** backtype.storm.tuple.Tuple;  

5.    

6.  **public** **class** CounterBolt **extends** BaseBasicBolt {  

7.    

8.      /** 

9.       *  

\10.      */  

\11.     **private** **static** **final** **long** serialVersionUID = -5508421065181891596L;  

\12.       

\13.     **private** **static** **long** counter = 0;  

\14.       

\15.     @Override  

\16.     **public** **void** execute(Tuple tuple, BasicOutputCollector collector) {  

\17.           

\18.         System.out.println("msg = "+tuple.getString(0)+" -------------counter = "+(counter++));  

\19.   

\20.     }  

\21.   

\22.     @Override  

\23.     **public** **void** declareOutputFields(OutputFieldsDeclarer declarer) {  

\24.         // TODO Auto-generated method stub  

\25.   

\26.     }  

\27.   

\28. }  

这里很简单就是将bolt 获取的数据进行简单的输出，并统计接收到的数据条目数。这里继续BaseBasicBolt 类，因为这样开发会比较简单。因为这个是唯一的bolt，没有输出，因此在declareOutputFields 方法中不需要声明output。

 

System.out.println("msg = "+tuple.getString(0)+"-------------counter = "+(counter++));

这里tuple就是这个bolt 从上一个spout获取的数据集合。

这里是控制台输出，因此请用本地模式进行调试。

 

打包上传到服务器，运行

Storm jar jarnameCountTopology     回车，会看到他在等待数据传入。

这个时候运行kafka消费者程序，将数据输出，则会看到storm 会迅速输出数据和统计数目。

 

 