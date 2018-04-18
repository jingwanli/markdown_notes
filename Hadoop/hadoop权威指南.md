# 第1章 初识Hadoop

### 数据存储于分析

多年来磁盘存储容量快速增加的同时, 其访问速度--磁盘数据读取速度—却未能与时俱进.

要实现多个磁盘数据的并行读写,需要解决的问题:

1. 硬件故障
2. 大多数分析任务需要以某种方式结合大部分数据共同完成分析任务

MapReduce提出了一个编程模型, 该模型将上述磁盘读写问题进行抽象, 并转换为对一个数据集(由键/值对组成)的计算.

> why需要MapReduce? why不使用关系型数据库?

因为寻址时间的提高远远慢于传输速率的提高.寻址是导致磁盘操作延迟的主要原因, 而传输速率取决于磁盘的带宽.

如果数据库系统只更新一小部分记录, 那么传统的B树更有优势, 但数据库更新大部分数据时, B树的效率比MR低得多,因为需要使用"排序/合并"来重建数据库.

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-08 下午10.32.03.png)

### 数据集的结构化

- 结构化数据: 具有既定格式的实体化数据,如 XML文档或者是满足特定预定义格式的数据库表
- 半结构化数据:比较松散, 虽然可能有格式, 但经常被忽略
- 没有特别的内部结构,如纯文本或图像数据

MR对于非结构化或半结构化数据非常有效.

### 网格计算与MR

数据本地化(data locality)特性是MapReduce的核心特征, 并因此获得良好的性能.

# 第2章 关于MapReduce

MapReduce程序本质上是并行的,因此可以将大规模的数据分析人物交给任何一个拥有足够多机器的运营商. MapReduce的优势在于处理大规模数据集.

### WordCount

map:

```java
package org.zkpk.wordcount;
import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class WordMap extends Mapper<Object, Text, Text, IntWritable> {
	protected void map(Object key, Text value, Context context)
			throws IOException, InterruptedException {
		String[] lines = value.toString().split(" ");
		for (String word : lines) {
			// 每个单词出现１次，作为中间结果输出
			context.write(new Text(word), new IntWritable(1));
		}
	}

}
```

reduce:

```java
package org.zkpk.wordcount;

import java.io.IOException;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class WordReduce extends Reducer<Text, IntWritable, Text, IntWritable> {
	protected void reduce(Text key, Iterable<IntWritable> values,
			Context context) throws IOException, InterruptedException {
		int sum = 0;
		for (IntWritable count : values) {
			sum = sum + count.get();
			context.write(key, new IntWritable(sum));// 输出最终结果
		}
	}

}
```

main:

```java
package org.zkpk.wordcount;

import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;

public class WordMain {
	public static void main(String[] args) throws IOException,
			ClassNotFoundException, InterruptedException {
		if (args.length != 2 || args == null) {
			System.out.println("please input Path!");
			System.exit(0);
		}
		Configuration configuration = new Configuration();
		Job job = new Job(configuration, WordMain.class.getSimpleName());
		// 打jar包
		job.setJarByClass(WordMain.class);
		// 通过job设置输入/输出格式
		job.setInputFormatClass(TextInputFormat.class);
		job.setOutputFormatClass(TextOutputFormat.class);
		// 设置输入/输出路径
		FileInputFormat.setInputPaths(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
		// 设置处理Map/Reduce阶段的类
		job.setMapperClass(WordMap.class);
		job.setReducerClass(WordReduce.class);
		// 设置最终输出key/value的类型m
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);
		// 提交作业
		job.waitForCompletion(true);
		
	}

}
```









