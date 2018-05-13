

```scala
import org.apache.spark.{SparkConf, SparkContext}

object WordCount{

	def main(args:Array[String]) : Unit = {

		val conf = new SparkConf( ).setAppName("wordcount").setMaster("local")
        val sc = new SparkContext(conf)
        
        sc.setLogLevel("WARN")//设置日志级别
        
        val lines = sc.textFile("hdfs://node1:8020/user/spark/data/a.scala")
        lines.flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_).foreach(println _)
        sc.stop()
	}
}
```

