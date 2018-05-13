###集群搭建

####一、Linux服务器的配置

​	HadoopMaster 和 HadoopSlave节点 都要完成

​	root 用户

​	zkpk 普通用户

​	1. 时钟同步  

​		crontab -e

​		0 1 * * * /usr/sbin/ntpdate cn.pool.ntp.org  自动时钟同步

​		分钟   小时    天    月  周

​		/user/sbin/ntpdate cn.pool.ntp.org  手动时钟同步

​		集群中节点之间互相通信的约定

​	2.主机名 localhost

​		vim /etc/sysconfig/network

​		master

​		slave

​		slave1

​		slave2

​		.....

​	3.主机列表

​		在集群中任何一个节点都有可能需要跟其他节点进行通信

​		就需要知道其他节点的ip地址和主机名

​		vim /etc/hosts

​		master

​			192.168.157.202 master

​			192.168.157.203 slave

​		slave

​			192.168.157.202 master

​			192.168.157.203 slave

​			

​	4.关闭防火墙

​		集群内部节点之间互相通信，不能有防火墙

​		setup

​		master 和 slave 都完成

​	5.配置网络

​		桥接

​		NET

​		vim /etc/sysconfig/network-script/ifcfj-eth0

​		

​	6.免秘钥登录

​		生成秘钥

​		master

​			/home/zkpk   ls mkdir  /bin

​			ssh-keygen -t rsa    生成秘钥

​			.ssh 

​				公钥

​				私钥

​			.bash_profile 查看公钥: 家目录下, .ssh

​		slave 

####二、安装Hadoop集群

​	1.解压安装Hadoop  hadoop-2.5.2

​		tar -zxvf hadoop-2.5.2.tar.gz

​	2.配置hadoop的环境变量 hadoop-env.sh

​		export JAVA_HOME=/usr/java/jdk1.7.0_71

​	3.配置Yarn的环境变量

​		export JAVA_HOME=........

​	4.hadoop的核心配置 core-site.xml

​		1)HDFS文件系统的入口或者地址

​			<propert>

​				<name>fs.defaultFS</name>

​				<value>hdfs://master:9000</value>

​			</property>

​		2)HDFS文件系统在各个节点上存储空间的落地磁盘位置

​			<propert>

​				<name>hadoop.tmp.dir</name>

​				<value>/home/zkpk/hadoopdata</value>

​			</property>

​	5.HDFS文件系统属性 

​		配置文件系统存储文件的冗余备份数

​		<propert>

​				<name>dfs.replication</name>

​				<value>3</value>

​		</property>

​		

​	6.YARN的配置

​	7.MapReduce框架在运行时资源的分配和调度交给yarn来做

​		mapred-site.xml

​		<propert>

​				<name>mapreduce.framwork.name</name>

​				<value>yarn</value>

​		</property>	

​	8.slaves 配置从节点的ip地址

​		vim slaves

​		slave	

​	9.创建数据目录

​		mkdir /home/zkpk/hadoopdata   master

​		mkdir /home/zkpk/hadoopdata   slave		

​	10.配置启动hadoop的环境变量	

​		/home/zkpk/hadoop-2.5.2/sbin/

​			start-all.sh

​				start-dfs.sh

​				start-yarn.sh

​			stop-all.sh

​				stop-dfs.sh

​				stop-yarn.sh		

​		vim /home/zkpk/.bash_profile

​			export HADOOP_HOME=/home/zkpk/hadoop-2.5.2

​			export PATH=\$HADOOP_HOME/bin:\$HADOOP_HOME/sbin:$PATH

11. 格式化HDFS文件系统

			只需要格式化一次就OK了

			master
		
				/home/zkpk/hadoopdata/dfs/id=1|2		
		
			slave
		
				/home/zkpk/hadoopdata
		
				/home/zkpk/hadoopdata/dfs/id=1
		
			当你格式了多次，导致集群启动失败
		
				slave: rm -rf /home/zkpk/hadoopdata/*
		
				先关闭之前的残留进程
		
				stop-all.sh
		
				kill -9 进程号
		
		12. 启动集群
		
			==start-all.sh==	
		
			master：
		
				ResourceManager  RM
		
				NameNode NN
		
				SecondaryNameNode SNN
		
			slave：
		
				NodeManager  NM
		
				DataNode DN
		
			slave1：
		
				NodeManager  NM
		
				DataNode DN
		
		13.集群调试	
		
			RM NM
		
			/home/zkpk/hadoop-2.5.2/logs/....   resourcemanager.log
		
		14.HDFS文件操作
		
			hadoop fs -put ./a.txt hdfs://master:9000/
		
		15.操作MR
		
			cd /home/zkpk/hadoop-2.5.2/share/hadoop/mapreduce	
		
			hadoop jar ./hadoop-mapreduce-examples-2.5.2.jar pi 5 5
		
		16. http://master:50070   关于分布式的文件系统而开发的一个web系统，用以查看HDFS的各种信息
		
		      http://master:18088  yarn 提供的一个web系统，用来展示yarn上运行的应用程序的及执行状态和进度信息

###一、大数据的概论

​	基本概念

​	大数据的背景

​	四个特征

​		速度快

​		数据量大

​		数据种类多

​		价值密度低，商业价值大

​		

​	大数据中心的建设，大数据项目的开发流程

​		数据源：数据产生的地方，对应的是具体的业务系统 电商、电信等等

​		数据的采集 

​			sqoop：关系型数据库与大数据平台之间的ETL工具 Extract Transfer Load

​			Flume：海量日志采集系统

​			scribe

​			chuwa

​			Kafka

​		数据的存储

​			HBase  实时的数据库

​			HDFS 分布式的文件系统

​			Tochya 基于内存的分布式的文件系统

​		数据的计算

​			实时计算

​				即时查询 ad-hoc

​					HBase Impala 

​				Streaming

​					Storm

​					Spark streming

​				离线计算

​				Batch

​					MapRedcue 

​					Hive

###三、访问HDFS

​	1.HDFS分布式文件系统

​		HDFS的概念 Hadoop Distributed FileSystem 分布式的文件系统 

​			将一个超大文件，该文件已经超过了单台节点磁盘的总容量，我们可以将该文件进行切分

​			一个一个的块（128M），将这些块分配到各个节点上进行存储。

​		基于流式数据访问模式，超大文件的需求而开发的

​		适合应用在大规模的数据集上

​		1）HDFS的优点

​			处理超大文件

​			处理结构化、非结构化、半结构化

​			一次写入多次读取 的流式数据访问

​			可以运行在廉价的商用机器上

​		2）缺点

​			不适合处理低延迟的数据访问

​			无法高效的存储大量的小文件

​			不支持多用户写入以及对文件进行任意的修改

​			

​		3)hdfs的特性

​			高容错，可扩展和可配置

​			跨平台  linux windows

​			shell命令接口  hadoop fs -put   hadoop fs -get

​			机架感知功能

​				机架：机房内组织服务器机柜的一种方式

​			负载均衡

​			Web界面

​		4）HDFS的目标

​			监测和恢复硬件故障

2.HDFS的核心设计

​		1）数据块(Data Block)，任何一个文件系统都有数据块的概念，数据块是文件系统中存储文件的最小单位

​			就 好比计算机中存储数据的最小单位是字节一样

​			windows 1k 2k 3k   500G			----1T

​			linux 8T 16t 32t

​			hdfs  64M---128M   10*32 = 320T

​			吞吐量大，但是数据的反应速度慢

​			hadoop fsck /sogou.500w.utf -files -locations -blocks

​			抽象成数据块所带来的好处

​				方便数据的备份，提供数据的容错 能力和可用性

​				一个超大文件有可能大于某一个节点的磁盘总容量	

​		2）数据副本的存放策略

​			副本：数据块block  副本的因子3   第一个块  第二个块（从第一个块复制而来）第三个块

​			副本因子是可以配置 hdfs-site.xml 块的大小也可以配置，hdfs-site.xml 			

​			第一个块存储 rank1 机架中的某一台服务器上

​			第二个块存储在不同于rank1 的另外一个rank2 上面的某一台服务器上

​			第三个块一般情况下会出在第一个块副本所在的机架rank1上面的某一台服务器上

3.HDFS的体系结构

​	HDFS的接口

​        一、命令行接口

​	hadoop fs -put 源文件  目标存储路径    将本地文件上传到HDFS文件系统中

​	hadoop fs -copyFromLocal src des

​	hadoop fs -ls hdfs://master:9000/ | /  查看HDFS文件系统中某一个路径下的文件或者文件夹信息	

​	hadoop fs -get /a.txt  从HDFS中下载一个文件到本地文件系统

​	hadoop fs -copyToLocal /README.txt	

​	hadoop fs -cat /README.txt	

​	hadoop fs -mkdir /aa    在hdfs文件系统的跟目录下创建一个名字叫aa的目录	

​	hadoop fs -rm 文件

​	hadoop fs -rmr 删除文件夹

​	二 java接口

​	1.从Hadoop URL中读取数据

```java
package FSDemos;

import java.io.IOException;
import java.io.InputStream;
import java.net.URL;

import org.apache.hadoop.fs.FsUrlStreamHandlerFactory;
import org.apache.hadoop.io.IOUtils;

public class Demo1 {
static {
	URL.setURLStreamHandlerFactory(new FsUrlStreamHandlerFactory());
}
	public static void main(String[] args) {
			InputStream in = null;
			try {
				in = new URL(args[0]).openStream();
				IOUtils.copyBytes(in, System.out,4096, false);
			} catch (IOException e) {
				e.printStackTrace();
			}finally {
				IOUtils.closeStream(in);
			}
			
	}

}

```

2. java访问接口---FileSystem

   - 读取文件1

   ```java
   package FSDemos;

   import java.io.IOException;
   import java.io.InputStream;
   import java.net.URI;

   import org.apache.hadoop.conf.Configuration;
   import org.apache.hadoop.fs.FileSystem;
   import org.apache.hadoop.fs.Path;
   import org.apache.hadoop.io.IOUtils;

   public class Demo2 {

   	public static void main(String[] args) throws IOException {
   		String uri = args[0];
   		Configuration conf = new Configuration();
   		FileSystem fs = FileSystem.get(URI.create(uri),conf);
   		InputStream in = null;
   		in = fs.open(new Path(uri));
   		IOUtils.copyBytes(in,System.out,4096,false);
   		IOUtils.closeStream(in);
   	}

   }
   ```

   - 随机读(FSDataInputStream)

   ```java
   package FSDemos;

   import java.io.IOException;
   import java.net.URI;

   import org.apache.hadoop.conf.Configuration;
   import org.apache.hadoop.fs.FSDataInputStream;
   import org.apache.hadoop.fs.FileSystem;
   import org.apache.hadoop.fs.Path;
   import org.apache.hadoop.io.IOUtils;

   public class Demo3 {
   	public static void main(String[] args) throws IOException {
   		String uri = args[0];
   		Configuration conf = new Configuration();
   		FileSystem fs = FileSystem.get(URI.create(uri),conf);
   		FSDataInputStream in = null;
   		in = fs.open(new Path(uri));
   		byte[] buffer = new byte[256];
   		int byteRead = 0;
   		while((byteRead=in.read(buffer))>0) {
   			System.out.write(buffer,0,byteRead);
   		}
   		IOUtils.closeStream(in);
   	}

   }
   ```

3. 写入数据到FileSystem ---FileCopy

   ```java
   package FSDemos;

   import java.io.BufferedInputStream;
   import java.io.FileInputStream;
   import java.io.InputStream;
   import java.net.URI;

   import org.apache.hadoop.conf.Configuration;
   import org.apache.hadoop.fs.FSDataOutputStream;
   import org.apache.hadoop.fs.FileSystem;
   import org.apache.hadoop.fs.Path;
   import org.apache.hadoop.io.IOUtils;

   public class FileCopy {

   	public static void main(String[] args) throws Exception {
   		String localSrc = args[0];
   		String dst = args[1];
   		InputStream in = new BufferedInputStream(new FileInputStream(localSrc));
   		Configuration conf = new Configuration();
   		URI uri = URI.create(dst);
   		FileSystem fs = FileSystem.get(uri,conf);
   		FSDataOutputStream out = fs.create(new Path(uri));
   		IOUtils.copyBytes(in, out, 4096,true);
   	}

   }

   ```

4. 文件系统复制到hdfs — append (文件必须先存在!!!)

**与FSDataInputStream类不同的是，FSDataOutputStream类不允许在文件中定位。这是因为HDFS只允许对一个已打开的文件顺序写入，或在现有文件的末尾追加数据**

```java
package hadooptest;
import java.io.BufferedInputStream;
import java.io.FileInputStream;
import java.net.URI;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IOUtils;

public class FileAppend {

	public static void main(String[] args) throws Exception {
		String localSrc = args[0];
		String dest = args[1];
		BufferedInputStream in = null;
		in = new BufferedInputStream(new FileInputStream(localSrc));
		Configuration conf = new Configuration();
		FileSystem fs = FileSystem.get(URI.create(dest), conf);
		FSDataOutputStream out = fs.append(new Path(dest));
		IOUtils.copyBytes(in, out, 4096,false);
	}

}
```



5. 在hdfs中创建目录

   public  boolean mkdirs(Path f)throws IOException

   **这个方法可以一次性新建所有必要但还没有的父目录，就像java．io．File类的mkdirs()方法。如果目录（以及所有父目录）都已经创建成功，则返回true。通常，你不需要显式创建一个目录，因为调用create()方法写入文件时会自动创建父目录**

```java
package hadooptest;

import java.net.URI;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;

public class MkdirDemo {

	public static void main(String[] args)throws Exception {
		Configuration conf = new Configuration();
		FileSystem fs = FileSystem.get(URI.create(args[0]),conf);
		fs.mkdirs(new Path(args[0]));
	}

}
```

6. 查询(检索到)一个目录 (多个目录情况下)
   - FileStatus
   - 检查文件或目录是否存在: public boolean exists(Path f) throws IOException

```java
package hadooptest;

import java.net.URI;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileStatus;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.FileUtil;
import org.apache.hadoop.fs.Path;

public class FileStatusTest {

	public static void main(String[] args)throws Exception {
		Configuration conf = new Configuration();
		FileSystem fs  = FileSystem.get(URI.create(args[0]),conf);
		boolean flag = fs.exists(new Path(args[0]));
		if(flag){
			FileStatus status = fs.getFileStatus(new Path(args[0]));
			FileStatus[] status2 = {status};
			Path[] paths = FileUtil.stat2Paths(status2);
			for(int i=0; i<paths.length;i++){
				System.out.println(paths[i]);
				System.out.println(paths[i].getName());
			}
		}
	}

}
```

7. 查找一个文件或目录的信息很实用，但通常你还需要能够列出目录的内容

   这就是FileSystem的listStatus()方法的功能：

   - ==publicFileStatus[] listStatus(Path f) throws IOException==
   - publicFileStatus[] listStatus(Path f， PathFilterfilter) throws  IOException
   - publicFileStatus[] listStatus(Path[] files) throws IOException
   - publicFileStatus[] listStatus(Path[] files, PathFilter filter) throws  IOException

```java
package hadooptest;

import java.net.URI;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileStatus;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.FileUtil;
import org.apache.hadoop.fs.Path;

public class ListTest {

	public static void main(String[] args)throws Exception {
		String uri = args[0];
		Configuration conf = new Configuration();
		FileSystem fs = FileSystem.get(URI.create(uri),conf);
		Path[] paths = new Path[args.length];
		for(int i = 0; i< args.length;i++){
			paths[i] = new Path(args[i]);
		}
		FileStatus[] status = fs.listStatus(paths);
		Path[] listedPaths = FileUtil.stat2Paths(status);
		for(Path p : listedPaths){
			System.out.println(p);
		}
	}

}

--------------------------------
    方法二:
FileSystem fs = FileSystem.get(URI.create(uri),conf);
FileStatus[] status = fs.listStatus(new Path(uri));
Path[] paths = FileUtil.stat2Paths(status);
    
```

8. 删除数据
   - 使用FileSystem的delete()方法可以永久性删除文件或目录
   - public boolean delete(Pathf, booleanrecursive) throws IOException
   - 如果f是一个文件或空目录，那么recursive(递归)的值就会被忽略。只有在recruslve值为true时，一个非空目录及其内容才会被删除（否则会抛出IOException异常）

```java
public class DeleteTest {
	public static void main(String[] args) throws IOException { 
		Configuration conf = new Configuration();
		FileSystem fs = FileSystem.get(URI.create(args[0]), conf);
		fs.delete(new Path(args[0]));
	}
 }
```



### HDFS的运行机制

####一、HDFS中数据流的读写

​	1.RPC的概念

​		RPC(Remote Procedure Call) 远程过程调用控制协议，

​		这个协议Hadoop工程师已经开发好了，主要是向远程计算机上发起请求和获取响应，而

​		不需要了解底层网络技术。

​		有RPC协议使得在分布式的网络程序应用中更加的便捷和容易

​		协议：

​			网络层：IP协议，IP地址，唯一表示网络中的一台主机，路由问题

​			传输层：TCP协议，UDP协议

​					TCP协议：传输数据的，三次握手，安全性非常高，不会丢失数据包

​						支付宝，网银，第三支付等等。。。。转账支付等业务都是用的TCP协议

​					UDP协议：无连接的协议，工业领域，在互联网中WWW UDP极为不安全

​			应用层：HTTP协议，

​					Tomcat http://localhost:8080/jingdong/product.jsp			

​			RPC协议，是为大数据远程调用控制而开发的一套独立的协议		

​			hdfs://localhost:9000/aa/bb.jsp   client----RPC协议----> cluster

​	2.RPC的模式

​		采用的客户机/服务器模式

​		Client（任意一个节点）------>服务器（Hadoop的HDFS集群）

​		请求程序就是一个客户机，而服务提供程序就是一个服务器。

​		

​	3.Hadoop的整个体系是构建在RPC协议上的，RPC的底层是基于IPC(进程间的通信)，因为HADOOP

​		集群是master/slave架构，所以其内部通信和与客户端的交互就是必不可少的了

​		

​	4.RPC的实现流程

​	  1）RPC框架主要包含的模块

​		  通信模块：两个相互协作的通信模块实现请求-----应答

​					hadoop fs -put ./a.txt hdfs://master:9000/

​		  代理程序：客户端和服务器端均包含代理程序

​			

​		  调度程序：调度程序接受来自通信模块的请求消息，并根据其中的标志

​					选择一个代理程序处理。

​	   2）一个具体的RPC请求从发送到获取结果所经历的过程

​			hadoop fs -put ./a.txt hdfs://master:9000/

​			hadooop fs -get /a.txt					

​			1、客户程序以本地方式调用系统产生的Stub程序；

​			2、该Stub程序将函数调用信息按照网络通信模块的要求封装成消息包，并交给通信模块发送到远程服务器端；

​			3、远程服务器端接收到此消息后，将此消息发送给相应的Stub程序；

​			4、Stub程序拆封消息，形成被调过程要求的形式，并调用对应的函数；

​			5、被调用函数按照所获参数执行，并将结果返回给Stub程序；

​			6、Stub将此结果封装成消息，通过网络通信模块逐级地传送给客户程序；

​	5.Hadoop RPC基本框架的实现		

​	   1）定义RPC协议，由于RPC协议的工作模式客户机/服务器模式

​			//构造一个客户端代理对象（该对象实现了某种协议，其实就是RPC协议），就可以发起RPC的请求了。

​			public static VersionedProtocol getProxy/waitForProxy(): 

​			//为某一个协议实例构造一个服务器对象，用于处理客户端发起的RPC请求		

​			public static Server getServer(): 			

​		  interface ClientProtocal extends org.apache.hadoop.ipc.VersiondProtocal{

​			//版本号。默认情况下，不同版本号的RPC Client和Server之间不能相互通信

​			public static final long versionID = 1L;

​			String echo(String value) throws IOException;

​			int add(int v1,int v2) throws IOException;

​			int f(int v1, int v2)throws IOException;			

​		  }

​		  

​	   2)实现RPC协议。RPC协议通常是一个接口，需要开发人员去具体的实现

​	   

​		  class ClientProtocalImpl implements ClientProtocal{

​		  

​			 public long getProtocolVersion(String protocol , long clientVersion){

​				return ClientProtocol.versionID;

​			}

​			String echo(String value) throws IOException{

​				System.out.println(value);

​			}

​			int add(int v1,int v2) throws IOException{

​				int a = v1+v2;

​				return a;

​			}

​			int f(int v1, int v2)throws IOException{

​				int a = v1*v2;

​				return a;

​			}

​			

​		  }

​	   3)构造并启动RPCServer  构造的方法：new RPC.builder(conf) 启动RPCServer .start()

​	   

​		public class MyServer {

​			public static final String ADDRESS="localhost";

​			public static final int PORT = 2454; 

​			

​			public static void main(String[] args)throws Exception {

​				final Server server = new RPC.Builder(new Configuration()).setProtocol(ClientProtocol.class)

​								.setInstance(new ClientProtocolImpl()).setBindAddress(ADDRESS).setPort(MyServer.PORT)

​								.setNumHandlers(5).build();

​				server.start();

​			}

​		}

​		

​	   4)构造客户端PRCClient。并发送RPC的请求

​	   

​		public class MyClient {

​			public static void main(String[] args) throws Exception {

​				ClientProtocol proxy = (ClientProtocol) RPC.getProxy(

​						ClientProtocol.class, 1L, new InetSocketAddress(

​								MyServer.ADDRESS, MyServer.PORT), new Configuration());

​				final int result = proxy.add(3, 5);

​				String r = proxy.echo(result+"");

​				System.out.println(r);

​			}

​		}

​		

​	   5）测试

​			先启动Server

​			然后启动Client向Server发起远程RPC请求

​	   

​	   

####二、HDFS的HA机制

​	Hadoop集群的架构主从Master/Slave ，是由一个NameNode和多个DataNode组成的

​	NN:主要存储的元数据信息

​	DN:数据本身，文件数据的内容

​	读文件：首先从NN上去请求该文件的元信息，包含了该文件所在的block块的列表信息，每个block都在哪些datanode节点上

​	然后，根据元数据地址信息，去访问文件所在的具体的datanode节点上的block块信息。

​	总结一下：NN上面存储了所有文件的元数据地址信息，如果NN突然宕机

​	NN的单点故障问题

​	SecondaryNameNode是NameNode的辅助

​	NameNode存放元数据的时候，元数据落地到磁盘的格式和目录

​	hadoopdata/dfs/name

​	namesecondary			   

​	HA High Available

​	NameNode的热备份来完成的

​	Active状态

​	共享文件存储系统来完成

​	集群协调器完成 zookeeper	

​	三台slave节点

​	zookeeper的安装

####三、HDFS的联邦机制

​	1.Federation的定义

​	Federation联邦，用于超大规模的的集群，一般都在上万甚至几十万台集群的规模

​	BAT

​	允许在集群中有多个NameNode，并且这些NameNode分别管理属于自己的一部分数据

​	互相之间独立运行，但他们共享所有的DataNode数据节点。	

​	2.Federation解决单NameNode的以下几个问题

​		HA主要解决了NameNode的单点故障问题

​		Federation不不能解决NameNode的单点问题，我们需要HA为联邦集群中的每一个NameNode

​		准备一个backup的NameNode，以应对NameNode突然宕机的问题	

​		1）HDFS的集群的扩展性。因为联邦扩大了NameNode的内存

​		2）性能更高效。多个NameNode分别管理属于自己的数据，并且同时对外提供服务器。

​		3）良好的隔离性。因为不同的NameNode处理不同的业务，这些业务之间因为NameNode请求的

​		不同，他们之间没有互相影响。

​	3.Federation的架构	

​		1）多个NameNode之间是联合的，同时他们之间互相独立且不需要互相协调，各自分工，管理自己的区域

​		   群众的所有DataNode被用作通用的存储设备，也就是说某一个DataNode不会专属于某一个特定的NameNode	   

​		2）一个block Pool一定是属于它自己的NameNode，每个DataNode都会存储属于集群中的所有blockpool的数据块

​		3）block pool 内部自治，互相不影响，也不会互相交流，当一个NameNode宕机了，不会影响其他的NameNode	

​		4）block Pool是NameNode命名空间的值，是管理的基本单位，当某一个NameNode被删除时，其对应的DataNode上的bolckpool也会被删除

###Hadoop IO流

####一、HDFS的数据完整性

​	1.数据存储 

​		1）Hadoop在使用过程中，理想的状态下不会出现数据的算坏或者丢失

​		2）Hadoop的IO流对磁盘的操作，今天已经非常的成熟，几乎不会因为IO流而把错误的数据引入而导致出现数据算坏

​		3）数据体量大到Hadoop能够处理的极限，数据算坏的概率就会增大

​		4）当某一台DataNode节点宕机，其上数据块就会被算坏丢失。

​	2.数据的校验

​		1）当数据块Block第一次引入系统的时候，会计算出校验和checkasum，

​		以元文数据的方式保存到.meta结尾的文件中。数据块的复制必然要通过一个

​		不可靠的通信路径传递到目标的DataNode节点上，在这个过程中还需要计算校验和

​		如果发现计算出来的校验和与原来的不一致，则认为该数据块已经损坏，可以选择ECC

​		内存条来自动修复，或者NameNode进行标记重新发起数据块的复制操作。

​		2）数据块容易损坏，但校验和也有可能损坏，只不过校验和信息非常小，损坏概率极低。

​	3.校验码信息

​		是利用CRC-32 循环冗余校验码算法来生成checksum校验和，结果就是一个32位的整数

​	4.校验和的生成

​		1）HDFS会对所有写入的文件进行计算校验和，并且在读取数据的时候验证校验和

​			io.bytes.per.checksum 指定了校验和默认存储所需要的空间，但实际上校验和

​			是CRC-32 即需要4个字节，所要校验和的存储额外开销比例不足1%

​		2）DataNode负责验证收到的数据存储数据和校验和

​			a.它在收到客户端的数据或复制期间其他datanode的数据时执行这个操作

​			b.正在写数据的客户端将数据及其校验和发送到由一系列datanode组成的管线，管线中最后一个datanode负责验证校验和。

​			c.果datanode监测到错误，客户端便会收到一个CheckSumException异常。

​	5.读数据时的数据完整性

​		客户端Client 读取文件数据的时候，会对所读数据块进行校验和的验证

​		验证的过程会产生日志记录。

​	6.DataNode会定期去对其上所存储的数据块进行校验，其实就是对校验的校验

​		定期是以文件被第一次写入到HDFS中开始算起，经过三周，三周时间一到，会自动触发

​		数据块的校验和验证。

​	7.数据完整性-如何修复算坏的数据块

​		HDFS存储着数据块的副本，可以通过副本来复制出完好无损的新的数据块

​		1）客户端在读取数据块时，如果监测到错误，就向namenode报告已损坏的数据块

​		及其正在尝试操作的这个datanode，最后才抛出CheckSumException异常。

​		2）Namenode将这个已损坏的数据块的副本标记为已损坏，

​		之后它安排这个数据块的一个副本复制到另一个datanode，

​		这样数据块的副本因子又回到期望水平	

​		3）已算坏了的数据块就会被删除

​	8.读取数据时，是否进行校验和的验证

​		FileSystem fs = FileSystem.get(URI.create(args[0]), conf);

​		fs.setVerfiyCheckSum(false); //在读取数据时不进行数据块校验码的验证

​		InputStream in = fs.open();

​		。。。。

​		hadoop fs -get -ignoreCrc hdfs://master:9000/a.txt //不验证

​		hadoop fs -copyToLocal /a.txt 

​	9.LocalFileSysem && CheckSumFileSystem

​	

####二、压缩

​	.rar .zip .tar.gz .... windows linux unix hdfs等都有压缩文件数据的需求

​	1.压缩的目的

​		1）可以减少文件在磁盘上的存储空间

​		2）通过压缩可以减少数据在网络上的传输时间

​	2.压缩的工具、格式、算法 各有千秋

​		四种压缩deflat bzip gzip lzo snappy.......

​	3.压缩的案例

​		压缩

​		解压缩		

​	4.压缩和输入分片

​		输入分片 ----->逻辑分片，并不是真正意义上的物理分片

​		逻辑分片记录了数据的开始位置和结束位置，位置一般是和block对应的，但也存在跨快的现象

​		逻辑分片的意义在于开启多少个Map任务，对应就是开启多少Map线程。

​		分布式的计算框架MapReduce  移动计算而不移动数据	

​		逻辑分片和物理分片不一定是一一对应的，一般情况下都是对应的。

​		a.txt -----》1280M / 128 = 10 block

​		hadoop jar wordcount.jar /a.txt /output

​		压缩形成压缩文件，压缩文件是否能被MapReduce进行处理，就要考虑分片的问题。

​		sogou.500w.utf8.gz

​	5.MapReduce框架在处理数据的时候可以采用压缩

​	

​		1）对MapReduce计算结果输出的压缩

​			conf.setBoolean("mapred.output.compress", true);

​			conf.setClass ("mapred.output.compression.codec",  GzipCodec.class, CompressionCodec.class);

​					job.waitForCompletion(true);

​			#压缩的类型是行压缩RECORD还是块压缩BLOCK

​			SequenceFileOutputFormat.setOutputCompressionType(job, SequenceFile.CompressionType.RECORD);

​			SequenceFileOutputFormat.setOutputCompressionType(job, SequenceFile.CompressionType.BLOCK);

​		2）对MapReduce Map中间结果的输出进行压缩

​			减少数据的传输，提高MapReduce计算的整体性能			

####三、序列化

​	1. Java中序列化

​		系列化、流化、字节化、字节的序列就叫做流化

​		计算机存储数据的基本单位是字节

​		大部分的API都支持序列化

​		Intger Long Float Double....

​		String.....

​		java.io.Serializable

​		entity

​			class User implements java.io.Serializable{

​				private String name;

​				private int age;

​			}

​		InputStream

​		OutputStream

​		BufferedReader(new InputStreamReader(new  InputStream(System.in)))

​	2.序列化的作用

​		流式数据访问

​		HDFS就是基于流式数据访问模式的

​	3.序列化的概念：所谓序列化（Serialization），是指将结构化对象转化为字节流，以便在网络上传输或写到磁盘进行永久存储

​	4.反序列化的概念：反序列化（deserialization）是指将字节流转回结构化对象的逆过程

​	5.序列化的应用场景

​		1）进程间的通信 DataNode1----block---->DatNode2

​			RPC协议将消息序列化为二进制的字节序列，然后发送到远程节点，远程节点再将

​			二进制的序列化反序列化为原始的消息进行处理。

​			紧凑

​			快速

​			可扩展

​			互操作

​		2)数据永久性的存储   a.txt------>hdfs://master:9000/a.txt	

​	6.Hadoop提供了自己的序列化框架Writable

​		紧凑、速度快、很难用Java意外的语言进行扩展和使用 Python C++ Ruby R 

​	7.Writable序列化框架是Hadoop的核心

​		MapReduce key-value  key type  value type		

​	8.Avro

​	9. Writable接口

​		Writable接口定义了两个方法：

​		一个将其状态写到DataOutput二进制流，

​		另一个从Datalnput二进制流读取其状态：

​		 public interface Writable {

​			void write(DataOutput out) throws IOException;

​			void readFields(Datalnput in) throws IOException;

​		}

#### 四、基于文件数据结构的SequenceFile 序列化文件

lSequenceFile

考虑日志文件，其中每一条日志记录是一行文本。如果想记录二进制类型，纯文本是不合适的。这种情况下，Hadoop的SequenceFile类非常合适，因为上述类提供了二进制键／值对的永久存储的数据结构。当作为日志文件的存储格式时，你可以自己选择键，比如由LongWritable类型表示的时间戳，以及值可以是Writable类型，用于表示日志记录的数量。

SequenceFiles同样也可以作为小文件的容器。而HDFS和MapReduce是针对大文件进行优化的，所以通过SequenceFile类型将小文件包装起来，可以获得更高效率的存储和处理

SequenceFile的写操作

​     public classSequenceFileWriteDemo {

  private static final String[]DATA={

  "One,two,buckle myshoe",

  "Three, four,shut thedoor",

  "Five,six,pick upsticks",

  "Seven,eight,lay themstright",

  "Nine,ten, a big fathen"

  };

public static void main(String[] args) throws IOException {

  String uri=args[0];

  Configuration conf=newConfiguration();

  FileSystemfs=FileSystem.get(URI.create(uri),conf);

  Path path=new Path(uri);

  IntWritable key=newIntWritable();

  Text value=new Text();

  SequenceFile.Writer writer=null;

​                try{

  writer=SequenceFile.createWriter(fs,conf, path, key.getClass(), value.getClass());

​                                        for(int i=0;i<100;i++){

  key.set(100-i);

  value.set(DATA[i%DATA.length]);

  System.out.printf("[%s]\t%s \t%s \n",                         writer.getLength(),key,value);

  writer.append(key, value);

  }

  }finally{

  IOUtils.closeStream(writer);

  }

  }

}

顺序文件中存储的键是从100到1降序排列的整数。表示为IntWritable对象。值为Text对象。在将每条记录追加到SequenceFile．Writer实例末尾之前，需要使用getLength()方法来获取文件访问的当前位置。

读取SequenceFile

public class SequenceFileReadDemo{

  publicstatic void main(String[] args) throws IOException {

  Stringuri=args[0];

  Configurationconf=new Configuration();

  FileSystemfs=FileSystem.get(URI.create(uri),conf);

  Pathpath=new Path(uri);

  SequenceFile.Readerreader=null;

​                        try{

  reader=newSequenceFile.Reader(fs,path,conf);

Writablekey=(Writable) ReflectionUtils.newInstance(reader.getKeyClass(), conf);

Writablevalue=(Writable)ReflectionUtils.newInstance(reader.getValueClass(),conf);

  long position = reader.getPosition();

  while(reader.next(key,value)){

  String synSeen= reader.syncSeen()?"*" : "";

  System.out.printf("[%s%s] \t%s \t%s\n", position,synSeen,key,value);

  position=reader.getPosition();

  }

  }finally{

  IOUtils.closeStream(reader);

  }

  }

}











