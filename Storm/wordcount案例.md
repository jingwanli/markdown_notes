## 案例

1. WordSpout.java

   ```java
   package storm_test;

   import java.util.Map;
   import java.util.Random;

   import backtype.storm.spout.SpoutOutputCollector;
   import backtype.storm.task.TopologyContext;
   import backtype.storm.topology.OutputFieldsDeclarer;
   import backtype.storm.topology.base.BaseRichSpout;
   import backtype.storm.tuple.Fields;
   import backtype.storm.tuple.Values;

   public class WordSpout extends BaseRichSpout {

   	private SpoutOutputCollector collector;
   	
   	private static String ss[] = {"hello hello hello","hadoop hello hbase hbase","hbase hbase hbase","storm storm hbase"};
   	
   	@Override
   	public void nextTuple() {
   		Random random = new Random();
   		String word = ss[random.nextInt(ss.length)];
   		Values values = new Values(word);
   		collector.emit(values);
   		
   	}

   	@Override
   	public void open(Map arg0, TopologyContext arg1, SpoutOutputCollector collector) {
   		this.collector = collector; 
   	}

   	@Override
   	public void declareOutputFields(OutputFieldsDeclarer declarer) {
   		declarer.declare(new Fields("word"));
   	}

   }

   ```

2. WordCountBolt.java

   ```java
   package storm_test;

   import backtype.storm.topology.BasicOutputCollector;
   import backtype.storm.topology.OutputFieldsDeclarer;
   import backtype.storm.topology.base.BaseBasicBolt;
   import backtype.storm.tuple.Fields;
   import backtype.storm.tuple.Tuple;
   import backtype.storm.tuple.Values;

   public class WordCountBolt extends BaseBasicBolt {

   	@Override
   	public void execute(Tuple input, BasicOutputCollector collector) {
   		String word = input.getString(0);
   		String[] words = word.split(" ");
   		for(String w:words){
   			collector.emit(new Values(w));
   		}
   		
   	}

   	@Override
   	public void declareOutputFields(OutputFieldsDeclarer declarer) {
   		 declarer.declare(new Fields("word1"));
   	}

   }

   ```

3. WordCountBolt2.java

   ```java
   package storm_test;

   import java.util.HashMap;
   import java.util.Map;

   import backtype.storm.topology.BasicOutputCollector;
   import backtype.storm.topology.OutputFieldsDeclarer;
   import backtype.storm.topology.base.BaseBasicBolt;
   import backtype.storm.tuple.Fields;
   import backtype.storm.tuple.Tuple;
   import backtype.storm.tuple.Values;

   public class WordCountBolt2 extends BaseBasicBolt {

   	private Map<String,Integer> map = new HashMap<>();
   	@Override
   	public void execute(Tuple input, BasicOutputCollector collector) {
   		String word = input.getString(0);
   		int count;
   		if(map.containsKey(word)){
   			count = map.get(word);
   		}else{
   			count = 0;
   		}
   		count++;
   		map.put(word, count);
   		collector.emit(new Values(word,count));
   	} 

   	@Override
   	public void declareOutputFields(OutputFieldsDeclarer declarer) {
   		declarer.declare(new Fields("word","count"));
   	}

   }

   ```

4. WordCountTopology.java

```java
package storm_test;

import backtype.storm.Config;
import backtype.storm.LocalCluster;
import backtype.storm.StormSubmitter;
import backtype.storm.topology.TopologyBuilder;
import backtype.storm.tuple.Fields;

public class WordCountTopology {

	public static void main(String[] args)throws Exception {
        TopologyBuilder builder = new TopologyBuilder();
        builder.setSpout("wordspout", new WordSpout(),5); //并发量 
        builder.setBolt("splitBolt",new WordCountBolt(),3).shuffleGrouping("wordspout");
        builder.setBolt("wordcountBolt", new WordCountBolt2(),5).fieldsGrouping("splitBolt", new Fields("word1"));
        
        Config conf = new Config();
        conf.setDebug(true); 
        
        if(args != null && args.length>0){
        	//cluster mode
        	conf.setNumWorkers(3); 
        	StormSubmitter.submitTopologyWithProgressBar(args[0],conf,builder.createTopology());
        }else{
        	conf.setMaxTaskParallelism(3);
        	LocalCluster cluster = new LocalCluster();
        	cluster.submitTopology("word-count", conf, builder.createTopology());
       // 	Thread.sleep(10000);
      //   	cluster.shutdown();
        }
        
	}

}

```

storm jar wordcount2.jar storm_test.WordCountTopology(参数) wordcount(别名)