####4）HBase Shell工具命令

#####负载均衡器：一定是在hbase不忙的时候打开！

balance_switch true : 返回的是当前的状态（负载均衡默认打开的）

balance_switch false

#####小合并：compact合并表或者region:	compact 't1'  --小合并 region范围的（针对表进行合并）

注：合并是合并region里面的文件，而不是合并region！

​     compact '表名'

​             compact '表名'，‘列簇名’ //优先对列簇名指定的列簇合并，但其他列簇也会被合并

#####大合并：major_compact 大合并！针对所有region（针对所有表的region，hstore的storefile合并成一个文件）

当处于大合并时，hbase对外不提供写的服务！

major_compact '表名'  // 优先合并表名指定的表，实际是合并所有表

major_compact '表名','列簇名'

切分表和region： split  '表名'

split  '表名' 之后 status查看服务器上的平均region数量

注意：合并针对的hstore中的storefile

   切分针对的是region

###HBase的java客户端

####新建工程

导入jar包：build path-->hbase...-->lib

   -->hadoop的包

   -->mapreduce的包

不用导hive的包，没有用到hive

####1.建表
```java
package hbasedemos;

import java.io.IOException;

import org.apache.hadoop.conf.Configuration;

import org.apache.hadoop.hbase.HBaseConfiguration;

import org.apache.hadoop.hbase.HColumnDescriptor;

import org.apache.hadoop.hbase.HTableDescriptor;

import org.apache.hadoop.hbase.MasterNotRunningException;

import org.apache.hadoop.hbase.ZooKeeperConnectionException;

import org.apache.hadoop.hbase.client.HBaseAdmin;

public class Demo1 {

	private static Configuration conf = null;

	static{

		conf = new HBaseConfiguration().create();

//conf  = new Configuration();  //这两种方法是通用的。

		//conf.set("hbase.zookeeper.quorum", "master,slave1,slave2"); //环境变量里已经配置了

	}

	public static void createTable(String tableName,String[] columnFamily) throws MasterNotRunningException, 				         ZooKeeperConnectionException, IOException{

		HBaseAdmin admin = new HBaseAdmin(conf);

		//HTableDescriptor table = new HTableDescriptor(tableName); //已经过时！

    HTableDescriptor table = new HTableDescriptor(TableName.valueOf(tableName)); //新定义

		for(int i=0;i<columnFamily.length;i++){

			table.addFamily(new HColumnDescriptor(columnFamily[i])); //新建表的时候只会建列簇，是不会细分到列的！

		}

		if(admin.tableExists(tableName)){

			System.out.println("table exists!");

		}else{

			admin.createTable(table);

		}			

	}

	public static void main(String[] args)throws Exception {

		createTable("mem2",new String[]{"name","info","address"});
//String[] arr = new String[3];
// for(..arr.length...){arr[i]="hello";}

	}
}
```
//注意：如果把admin设置成成员变量。public static HBaseAdmin admin；

​      不可以写做 public static HBaseAdmin admin = new HBaseAdmin(conf);

需要在crateTable里：admin = new HBaseAdmin(conf);



因为HBaseAdmin()这个类在类初始化的时候（构造函数）可能会抛出很多io异常，

成员变量无法 try catch处理。

####2 删除表

public static void deleteTable(String tableName)throws Exception{		 

​		HBaseAdmin admin = new HBaseAdmin(conf);

​		admin.disableTable(tableName); //先失效

​		admin.deleteTable(tableName);	

//注意：删除表之前必须先disable！

​	}

####3 添加列簇：

 public static void addColumnFamily(String tableName,HColumnDescriptor columnFamily)throws Exception{

​		HBaseAdmin admin = new HBaseAdmin(conf);

​		admin.addColumn(tableName, columnFamily);

​		}	 

 main函数中：

addColumnFamily("mem2",new HColumnDescriptor("phone"));

####4 删除列簇：

 public static void deleteColumnFamily(String tableName,String columnFamily)throws Exception{ 

​		HBaseAdmin admin = new HBaseAdmin(conf);

​		admin.deleteColumn(tableName, columnFamily);

​	 }

//注意：添加列簇和删除列簇的列的参数是不同的！

*java中 admin的功能完成(admin修改的是表结构)

\-----------------------------------------------------------------------------------

####5 添加源码

​     mkdir /home/xdl/hbase-src

mv hbase-0.98.9-src.tar.gz ~/hbase-src

tar -zxvf ...

导入源码

####6 添加数据 //put一行(shell命令是put一个单元格)

​    public static void addData(String tableName, String rowKey, String[] column1, String[] value1, String[] column2,

​			String[] value2) throws Exception {

​		// HBase中存储的数据都是字节型

​		// 设置rowKey

​		Put put = new Put(Bytes.toBytes(rowKey));

​		// 或者 Put put = new Put(rowKey.getBytes());

​		// HTable用于和HBase进行通信

​		HTable table = new HTable(conf, Bytes.toBytes(tableName));

​		// 获取表的信息.添加数据的表一定是已经存在的表！

​		HTableDescriptor htd = table.getTableDescriptor();

​		// 获取表的列簇

​		HColumnDescriptor[] columnFamilies = htd.getColumnFamilies();

​		

 for (int i = 0; i < columnFamilies.length; i++) {

​			String familyName = columnFamilies[i].getNameAsString();

​			// 此处没有设置时间戳，时间戳当写入数据时自动生成！

​			if (familyName.equals("name")) {

​				for (int j = 0; j < column1.length; j++) {

​					put.add(Bytes.toBytes(familyName), Bytes.toBytes(column1[j]), Bytes.toBytes(value1[j]));

​				}

​			}

​			if (familyName.equals("hobby")) {

​				for (int j = 0; j < column2.length; j++) {

​					put.add(Bytes.toBytes(familyName), Bytes.toBytes(column2[j]), Bytes.toBytes(value2[j]));

​				}

​			}

​		}

//put封装了行键信息和添加列的信息

​		table.put(put);

​		System.out.println("add data successful!")；

​			}

main函数中：

String[] column1 = { "xing", "ming" };

​		String[] value1 = { "zhang", "san" };

​		String[] column2 = { "hobby" };

​		String[] value2 = { "basketball" };

​		addData("student","std1",column1,value1,column2,value2);

####7 查询数据 （get一行的数据）

​    public static Result getResult(String tableName, String rowKey) throws Exception {

​		Get get = new Get(Bytes.toBytes(rowKey));

​		HTable table = new HTable(conf, Bytes.toBytes(tableName));

​		// 获取表的中的一行信息

​		Result result = table.get(get);

List<Cell> list = result.listCells(); //用新方法要加入此行！

​		// 一行中有多个列，每个列有自己的列簇、列名、值、版本（时间戳）

​	    /* for (KeyValue kv : result.list()) {  //已经过时！

​			// 输出列簇信息

​			System.out.println("family:" + Bytes.toString(kv.getFamily()));//已经过时！

​			// 输出列名信息

​			System.out.println("qualifier:" + Bytes.toString(kv.getQualifier()));

​			// 输出单元格的值

​			System.out.println("value:" + Bytes.toString(kv.getValue()));

​			// 输出单元格值的版本号

​			System.out.println("timestamp:" + kv.getTimestamp());

​			System.out.println("----------------------------"); 

​		}  */ //全部注掉！

for(Cell cell : list){

System.out.println("family:" + Bytes.toString(cell.getFamilyArray(), cell.getFamilyOffset(),cell.getFamilyLength()));

System.out.println("qualifier:" + Bytes.toString(cell.getQualifierArray(),cell.getQualifierOffset(),cell.getQualifierLength()));

System.out.println("value:" + Bytes.toString(cell.getValueArray(),cell.getValueOffset(),cell.getValueLength()));

System.out.println("timestamp:" + cell.getTimestamp()); //获取时间戳未变！

 }

​		return result;

​	       }

####8 全表扫描查询 (查询所有行数据)

//遍历查询HBase表

public static void getResultScanner(String tableName)throws Exception{

​	//扫描器

​	Scan scan = new Scan();

​	//扫描结果集，将包含全表中的所有行

​	ResultScanner rs = null;

​	//获得与HBase表进行通信的HTable对象

​	HTable table = new HTable(conf,Bytes.toBytes(tableName));

​	try{

​		rs = table.getScanner(scan);

​		//Result代表每一行

​		for(Result r: rs){

​			for(KeyValue kv : r.list()){

​				//获取Rowkey

​				System.out.println("ROW:"+ Bytes.toString(kv.getRow()));

​				System.out.println("family："+ Bytes.toString(kv.getFamily());

​				System.out.println("qualifier:" + Bytes.toString(kv.getQualifier()));

​				System.out.println("value:"+Bytes.toString(kv.getValue()));

​				System.out.println("timestamp:"+kv.getTimestamp());

​				System.out.println("-----------------------------");

​			}

​		}finally{

​			if(rs!=null)

​				rs.close();

​		}

​		for(Result r:rs){

​			for(Cell kv:r.listCells()){

​				.....

​			}

​		}

​	}

}

####9 通过列簇和列名查询（多行）  

getResultScannerByColumnFamilyAndColumnName(String tableName,String columnFamily,String columnName){

//在扫描全表的基础上添加：

scan.addFamily(columnFamily.getBytes());

scan.addColumn(columnFamily.getBytes(),columnName.getBytes());

}

//小结：单行的操作使用 put 、get(有行键参数)

多行的操作，用扫描器

10）获取单元格数据

​       通过行键，列族，列名获取数据

​        getResultByColumn(String tableName, String rowKey, String familyName, String columnName)

在get一行的数据的代码基础上增加：

get.addColumn(Bytes.toBytes(familyName),  Bytes.toBytes(columnName));

####11 更新数据

public static void updateTable(String tableName, String	rowKey, String familyName, String columnName,String value){

HTable table =  new HTable(conf,Bytes.toBytes(tableName));

Put put = new Put(Bytes.toBytes(rowKey)); //put就是update！

put.add(Bytes.toBytes(familyName), Bytes.toBytes(columnName), Bytes.toBytes(value));

table.put(put);

System.out.println("update table successful!");

   }

#### 12 查询某列数据的多个版本

public static void getResultByVersion(String tableName, String rowKey,String familyName,String columnName) throws Exception{

HTable table = new HTable(conf,Bytes.toBytes(tableName));

Get get = new Get(Bytes.toBytes(rowKey));

get.setTimeStamp(...l);// 不设置的话返回最新版本。设置版本号可以查看老版本(版本号+l )，因为版本号是long类型

get.addColumn(Bytes.toBytes(familyName), Bytes.toBytes(columnName));

get.setMaxVersion(5);

Result result = table.get(get);

for(KeyValue kv: result.list()){

System.out.println("family:" + Bytes.toBytes(kv.getFamily()));

System.out.println("qualifier:" + Bytes.toBytes(kv.getQualifier()));

System.out.println("value:" + Bytes.toBytes(kv.getValue()));

System.out.println("Timestamp:" + kv.getTimestamp());a

System.out.println("-------------------------------------");

}

} 

####13 根据行号删除表中数据（表结构没改）

public static void deleteDataByRowKey(String tableName,String rowKey,String familyName,String columnName) throws Exception{

HTable table = new HTable(conf, tableName);

Delete delete = new Delete(Bytes.toBytes(rowKey));

delete.deleteColumn(Bytes.toBytes(familyName),Bytes.toBytes(columnName));//删除单元格，本行注掉的话是删除一行

htable.delete(delete);

System.out.println("delete a column successful!");

}

####-----------------------总结------------------------

1.HBaseConfiguration

Configuration conf = new HBaseConfiguration().create();

 2.HBaseAdmin 相当于HBase的数据库管理员

建表

删表

修改表

负载均衡

compact

offline

online

...

管理类的操作都可以向HBaseAdmin发起

3.表

​      1)HTable: 对应HBase数据库中的表.用于和hbase进行通信

HTable table = new HTable(new Configuration(),Bytes.getBytes(tableName));

2)HTableDescriptor"  对表的描述信息

HTableDescriptor table = new HTableDescriptor(TableName.valueOf(tableName));

3)HColumnDescriptor 列簇

HColumnDescriptor[] columnFamilies = table.getColumnFamilies();

HTable主要是对表中数据的操作

新增、查询（get,scanner）、删除、修改 

 ---------上课内容---------- 

get：获取一行

getScanner：获取多行

scan不添加任何东西代表扫描全部

hbase是列式存储，只能一个单元格，一个单元格的写，java一次可以写一行。

注：hbase的java编程：支持实时查询，底层依赖的是hbase自己的一套命令。

适合小型的数据查询和修改。

mapreduce：适合大量数据的导入导出。比如初始化。mapreduce不提供实时查询

   （java可以按单元格导入或者按行导入，当数据量很大时不合适）

   一般的hbase开发，基本上都用java

####-------------------------------------上课内容-------------------------------------

###mapreduce批量操作hbase

####三种访问模式：

  1.HBase作为输入源：可以用sqoop。（sqoop底层也是mr）

  2.HBase作为输出源：常用于数据迁移（比如hbase初始化，可以用mr，也可以用sqoop）

  3.HBase作为共享源（HBase的结果存到HBase的另一张表中）  

（hbase比较稳定，如果时钟同步正常的话）

####MapReduce API（从mr的角度来看输入输出）

##### 1.HBase作为输入源：

从HBase中读取数据，使用mr计算，结果存入hdfs

查询年龄是25岁的用户信息

（也可以用scanner做，可以查找出数据。

   但如果需要重新建模，再存入hdfs中，使用mr做）

scan.setBatch 设置为0 表时不限制

缓存：10000条 hbase里是一条一条的数据

hbase 秒级别：8000-10000

设置hadoop的推测执行为false：

生产过程中将推测行为关闭，默认是打开的，保证数据的一致性

map中：

行键：row

result：一行数据

context：

tostringbinary：如果包含特殊字符

导包的时候：export java source....依赖包.（hbase要依赖的包也打入进去）

package com.mr.inputsource;

-------------------------------------源 码-------------------------------------------

import org.apache.commons.logging.Log;

import org.apache.commons.logging.LogFactory;

import org.apache.hadoop.conf.Configuration;

import org.apache.hadoop.fs.FileSystem;

import org.apache.hadoop.fs.Path;

import org.apache.hadoop.hbase.HBaseConfiguration;

import org.apache.hadoop.hbase.client.Scan;

import org.apache.hadoop.hbase.mapreduce.TableMapReduceUtil;

import org.apache.hadoop.hbase.util.Bytes;

import org.apache.hadoop.io.Text;

import org.apache.hadoop.mapreduce.Job;

import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;

  //HBase 作为输入源，从HBase表中读取数据，使用MapReduce计算完成之后，将数据存储到HDFS中

public class Main {	

​	static final Log LOG = LogFactory.getLog(Main.class);

​	//JobName

​	public static final String NAME = "Member Test1";

​	//HDFS system file 

​	public static final String TEMP_INDEX_PATH = "/tmp/member_user";	

​	//HBase作为输入源的HBase中的表 member_user

​	public static String inputTable = "member_user";

​	public static void main(String[] args)throws Exception {		

​		//1.获得HBase的配置信息

​		Configuration conf = HBaseConfiguration.create();

​		//conf = new Configuration();

​		//2.创建全表扫描器scan对象

​		Scan scan = new Scan();

​		scan.setBatch(0);

​		scan.setCaching(10000);

​		scan.setMaxVersions();

​		scan.setTimeRange(System.currentTimeMillis() - 3*24*3600*1000L, System.currentTimeMillis());

​		//3.配置scan,添加扫描的条件，列族和列族名

​		scan.addColumn(Bytes.toBytes("info"), Bytes.toBytes("age"));

​		//4.设置hadoop的推测执行为fasle

​		conf.setBoolean("mapred.map.tasks.speculative.execution", false);

​		conf.setBoolean("mapred.reduce.tasks.speculative.execution", false);

​		//5.设置HDFS的存储HBase表中数据的路径

​		Path tmpIndexPath = new Path(TEMP_INDEX_PATH);

​		FileSystem fs = FileSystem.get(conf);

​		//判断该路径是否存在，如果存在则首先进行删除

​		if(fs.exists(tmpIndexPath)){

​			fs.delete(tmpIndexPath,true);

​		}	

​		//创建job对象

​		Job job = new Job(conf,NAME);

​	        //设置执行JOB的类

​		job.setJarByClass(Main.class);

​		//job.setMapperClass(MemberMapper.class);

​		//设置TableMapper类的相关信息，即对MemberMapper类的初始化设置

​		//(hbase输入源对应的表， 扫描器， 负责整个计算的逻辑，输出key的类型，输出value的类型，job )

​		//TableMapReduceUtil.initTableMapperJob(table, scan, mapper, outputKeyClass, outputValueClass, job);

​		TableMapReduceUtil.initTableMapperJob(inputTable, scan, MemberMapper.class, Text.class, Text.class, job);

​		//设置作业的Reduce任务个数为0

​		job.setNumReduceTasks(0);

​		//设置从HBase表中经过MapReduece计算后的结果以文本的格式输出

​		job.setOutputFormatClass(TextOutputFormat.class);

​		

​		//设置作业输出结果保存到HDFS的文件路径

​		FileOutputFormat.setOutputPath(job, tmpIndexPath);

​	

​		//开始运行作业

​		boolean success = job.waitForCompletion(true);

​		System.exit(success?0:1);

​	}

}

#####--------------------------------------源 码-------------------------------------------

package com.mr.inputsource;

import java.io.IOException;

import org.apache.hadoop.hbase.KeyValue;

import org.apache.hadoop.hbase.client.Result;

import org.apache.hadoop.hbase.io.ImmutableBytesWritable;

import org.apache.hadoop.hbase.mapreduce.TableMapper;

import org.apache.hadoop.hbase.util.Bytes;

import org.apache.hadoop.io.Text;

import org.apache.hadoop.io.Writable;

//HBase 中的表 作为 输入源

 // 扩展自Mapper类，所有以HBase作为输入源的Mapper类需要继承该类

 public class MemberMapper extends TableMapper<Writable, Writable>{	

​	private Text k = new Text();

​	private Text v = new Text();

​	public static final String FIELD_COMMON_SEPARATOR="\u0001";

​	@Override 

​	protected void setup(Context context) throws IOException ,InterruptedException {		

​	}

​	@Override

​	public void map(ImmutableBytesWritable row, Result columns, 

​			Context context) throws IOException ,InterruptedException {

​		String value = null;

​		//获得行键值

​		String rowkey = new String(row.get());

​		//一行中的所有 列族

​		byte[] columnFamily = null;

​		//一行中的所有列名

​		byte[] columnQualifier = null;		

​		long ts = 0L;		

​		try{

​			//遍历一行中的所有列

​			for(KeyValue kv : columns.list()){				

​				//单元格的值

​				value = Bytes.toStringBinary(kv.getValue()); // 25				

​				//获得一行中的所有列族

​				columnFamily = kv.getFamily(); //info				

​				//获得一行中的所有列名

​				columnQualifier = kv.getQualifier();				

​				//获取单元格的时间戳

​				ts = kv.getTimestamp();

​				if("25".equals(value)){

​					k.set(rowkey);

​					v.set(Bytes.toString(columnFamily)+FIELD_COMMON_SEPARATOR+Bytes.toString(columnQualifier)

​							+FIELD_COMMON_SEPARATOR+value+FIELD_COMMON_SEPARATOR+ts);

​					context.write(k, v);

​					break;

​				}

​			}

​		}catch(Exception e){

​			e.printStackTrace();

​			System.err.println("Error:"+e.getMessage()+",Row:"+Bytes.toString(row.get())+",Value"+value);

​		}		

​	}

}

--------------------------------------源 码-------------------------------------------

#####2.HBase作为输出源：

   将HDFS的数据 经过mr后输出到hbase数据库中。

第一个参数可以认为是job的名字

mr默认使用制表位进行切割。

-1 ：表示切割完以后返回所有元素。

main中：原始目录解析

打包的时候 手动选一个，并且选择lib 一起打包！

表必须先存在！！！！

####sqoop与HBase(从hbase角度来看)

使用sqoop将mysql中的数据导入HBase：(查询速度比mysql快很多)

bin/sqoop import --connect jdbc:mysql://master:3306/pingjia --username hadoop --password hadoop --table  spider --hbase-table PINGJIA.SPIDER --column-family f1 --hbase-row-key id --hbase-create-table -m 1

###HBash核心知识点

####一.HBase核心结构

传统的关系型数据库：毕加数算法

底层维护一棵树，不断写数据，树越来越大。

关系型数据库必须写入磁盘才是写入成功

​      3种遍历数据库的方式前序 中序 后序

​       HBase：LSM树

​       LSM- Log Structure Merge Tree 日志结构合并树

内存：顺序存储、

memstore里维护一棵树，从内存-》flush到磁盘

小合并：磁盘上merge

大合并 merge上合并成一颗大树

写磁盘： 顺序写、随机写（需要找空间）

 顺序写效率高

HBase采用顺序写（可以和写内存的速度相媲美）

why 写效率高？

写：两种1）写到内存：写入内存对用户来说就是写入成功

2）写到磁盘

get：第一次读的时候稍慢，是放入内存。后面读取速度会很快

 查询时候也是先从缓存中查

读写独立：

保证一致的写效率：

####二.底层持久化

#####1.存储基本机构

hbase实在hdfs基础上构建了自己的文件类型（128k为文件单位）

​    hbase和hdfs协作来存储数据

hive是没有自己的文件类型的，存储结构就是hdfs

HBase处理的两种基本文件类型

1.

2.

先写日志，保证数据的完整性，如果flush失败，去日志里重新写一遍

写日志，写的是真实数据

hbase没有数据库的概念，但是有命名空间的概念，hbase的核心思想是：bigtable

但是有命名空间，用来组织表

hbase在启动的时候就会把meta表的region随机分配在任意一个hregionserver

用户表的regions一般都存储在硬盘中（除非发起查询，才会维护到缓存中，缓存的周期取决于具体算法）

两张特殊的表：

1.root表：只有1个region（管理的是meta，数据量已经很小）。记录meta表的region信息。（已经取消root表，数据迁移到zk）

2.META表：存储了用户Region信息：几个region，地址在哪。

   		    META表可以有多个region，meta表的regions全部保存在内存中。（有副本，只是在内存中维护了一份）

​     若某个hregionserver当机，zk会及时发现启动failover机制，根据副本恢复（hdfs中，3个副本）。

​     

zk中记录了-root表的location。所以客户端先请求的是zk！

客户端会将查询过的位置缓存起来，且缓存不会主动失效

#####2.hbase工作流程

 	1）客户端链接zk

​	2）zk中检索到 拥有root-region的的regionserver，得到持有对应行键的meta表的region服务器名，操作结果被缓存下来（1，2所有操作）

​	3）查询meta表服务器，检索包含给定行键的region所在的服务器

 	前提：启动hbase时，hmaster负责把region分配给每个hregionserver

​      	   包括root 和 meta

​         zk中->     ls /hbase  可以看到znode上的meta的很多信息 

​         cd hbase-0.98....

​	vim hbase-site.xml  （唯一的关系配置）

​	配置了：告诉了zk，hbase的集群列表，会在zk上注册元数据

​	也配置了落地的磁盘信息：hdfs的根目录（meta也在里面）

###### hadoop fs -ls /hbase 两类文件：	

​	1）日志文件

​	  WALs 日志（先写日志）

​	2）表的目录下的文件（region）

​     	 	每个列组有单独的目录（hbase的data下)

###### 表

​	 系统表里也有meta，如果新建一个meta表时可以的（因为命名空间不同）

#####3.region切分

​	1当region大于阀值，就需要切分（阀值大概10g），

按照行键切分。

​	2 regionserver通过父region内创建切分目录来完成，之后会关闭region，此region不再接收任何请求（下线）

​	3 meta表更新 已经切分的会标记，

​	4.原始region最终会被清除，从meta表中删除

​	5.切分大小默认10g，可以根据实际情况调整 

######region阀值配置

hbase-site.xml

region 合并（指的是region内部文件的合并）

合并

1.memstore的flush操作会触发compaction

\2. 数目足够多时，合并会将他们合并成规模更少但是更大的文件

​    如果这些文件中最大的  ----》触发切分

#####小合并：

​	1.合并文件数：hbase.hstore.compaction.min 默认为3&& 〉=2

​	2.合并文件数最大为：10

​	3.设置hbase.hstore.compaction.max.size 默认设置为memstore的flush容量64M

   hbase	...                            max       ：Long.MAX_VALUE

减少需要进行校合并的文件列表 

在达到单次compaction之前（10次），小于该阀值的文件将都会被合并

小合并发生在region范围内。

​        大合并：（所有region，所有hstore，所有storefile全部成一个）

\1.  将所有文件合并成一个

 \2. 当memstore被flush到磁盘，执行了compact 或者 major_compact

​     或产生了相关api调用，或运行后台线程会触发该检查

​	大合并的周期，一般默认24小时，自动检查（生产过程中把自动大合并关闭！）

 	3.如果调用 major_compact或者majorCompact()API调用都会强制大合并

小：storefile合并成大文件 列簇内部合并。

设计表的时候尽量用单个列簇（多个列族都会合并，产生多个小文件）

文件64M只是个阀值，不是限制。大于64M会合并 10g的时候切分 。超过最大阀值的大小的文件，就不产生后面的小合并

#####大：跨region的，3个region分别合并

region的切分也是在单个

小合并：局部合并

大合并：针对所有region 合并storefile（减少大量的小文件）

hbase的备份：奢侈 replication

==HBase的数据不能导入关系型数据库（因为HBase中的数据是非结构化的！）==



