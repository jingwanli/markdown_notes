工程中新建folder

复制mysql(驱动)—>粘贴到lib中—>build path

1. WordSpout 继承自 BaseRichSpout

2. WordSplitBolt  继承自BaseBasicBolt

   发送的数据是 Values类型

   declarer.declare(new Fields("word")); // declare的参数是Fields类型.

3. WordCountBolt 

   hashmap中 count不要在声明的里面初始化

   而在containsKey(word) 中初始化

4. WordSaveBolt 继承自 BaseRichBolt

   ```java
   private static String driver= "com.mysql.jdbc.Driver";
   private static String url = "jdbc:mysql://master:3306/test";
   private static String username = "hadoop";
   private static String  password = "hadoop"
   private Connection conn;
   try{
       Class.forName(driver);
       connection = DriverManager.getConnection(url,username,password);
   	
       
   }catch(Exception e){
       e.printStackTrace();
   }
    public void execute(Tuple input){ //继承的类不同,这里的参数有变化
         String word =  input.getStringByField("word");
      Integer count= input.getInteger(1);
      
      String sql = "insert into wordcount values(?, ?)";
      try{
         PrepaerdStatement pstmt = connection.prepareStatement(sql);
          pstmt.setString(1,word);
          pstmt.setInt(2,count);
      }catch(Exception e){
          e.printStackTrace();
          
      }
    }
   ```



   ```

5. WordCountTopology

   TopologyBuilder builder = new TopologyBuilder();

   builder.setSpout("wordspout",new WordSpout(),5);

   builder.setBolt("split",new WordSplitBolt(),3).shuffleGrouping("wordspout");

   builder.setBolt("count", new WordCountBolt(), 2).fieldGrouping("split",new Fields("word"));

   builder.setBolt("save",new WordSaveBolt(),3).fieldsGrouping("count",new Field("word", "count")); //最后一个参数是指定分组依据

   Config conf = new Config();

   conf.setDebug(true); 

   conf.setNumWorkers(3);

   //运行方式有2种 : Cluster Model/Local Cluster

   if(args != null && args.length>0){	    StormSubmitter.submitTopology(args[0],conf,builder.createTopology());

   }else{

   ​	conf.setNumWorkers(2);

   ​	LocalCluster local = new LocalCluster();

   ​	local.submitTopology("word-count-test",conf, builder.createTopology());

   }

6. 运行

   打jar包 或 在本地运行(程序建本地集群的时间比较长)



7. 查错

   ssh slave1

   ​	cd apache-storm-0.9.6

   ​	cd logs/ — 查看最新的日志

   ​	空指针异常

   ​	查看图形界面=> 查看问题所在~

   ​	线下可以运行—eclipse有驱动包

   解决方案: 1. 将驱动包一起打包. //此方法不行

   ​		 2. cd apache-storm-….

   ​		     cd lib

   ​		    cp ~/mysql-connector-java-5.1.28.jar

   ​			并cp到2个slave节点中 (依赖包)

   ​			kill 掉storm中的运行进程

   ​			重启storm :(3个节点)

   ​				storm nimbus>/dev/null 2>&1 &

   ​				….

   ​			运行

storm kill storm-word-count

storm jar wordcountextend.jar storm.stormtest word-count-topology



bin/storm jar ./examples/storm-starter/storm-starter-topologies-0.9.3.jar storm.starter.ExclamationTopologyexclamation-topology 

其中，jar命令是专门负责提交任务使用的，storm-starter-topologies-0.9.3.jar是包含Topology实现代码的JAR包，storm.starter.ExclamationTopology的main方法是Topology的入口,exclamation-topology是strom应用程序的别名

stormkill  exclamation-topology  结束Topology
   ```