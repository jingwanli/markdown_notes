---
typora-root-url: ../img
---



###复习：zookeeper

分布式系统的协调器

解决分布式系统中的一致性问题，Paoxs算法

数据的发布与订阅

命名服务

软性的负载均衡 kafka 消费者和生产者

存放元数据

故障的检测和修复

####zookeeper的安装与部署：

1）tar -zxvf zookeeper...

 2）配置环境变量

 3） 配置 zoo.cfg

tickTime = 2000 

initTime= //选举产生leader的时间

同步时间：leader选出以后，与follower同步的时间

dataDir:zk存放数据的目录

server.1 = master:2888:3888 //选举与同步的端口

cliPort =2181 //开放给客户端进行访问的端口号

4）安装文件拷贝到slave1，slave2

5）修改myid

6）zkServer.sh start //3个节点需要分别启动

​	zkServer.sh status

####zookeeper的工作原理：

角色：

leader:投票的发起和决议，系统的更新

follower:负责处理客户端的请求，并生成响应，同时参与leader选举的投票

observer：负责客户端与follower的连接

   负责leader和follower的通信

zk核心 原子广播模式

 基于zap协议

广播模式：选出leader后，进入

恢复模式：zk启动时，自动进入，

####存储结构：

目录层次结构（树形结构）

znode（存放数据的节点）

zkCli.sh -server master:2181 

ls / :查看所有的 znode

create /zk aaaa：创建znode，名字是zk，值是aaaa

get /zk    delete /zk

set /zk  :  对znode节点所关联的字符串进行设置

持久性节点

临时节点：当注册的源失效，立刻失效

--------------------------------------上课内容----------------------------------------

###配置时钟同步：

su root

crontab -e 

如果没有 ，安装

yum install -y vixie-cron（三个节点分别查看）

hadoop的缺点：1，2，3

Hbase：分布式的、面向列的开源数据库。底层依赖hdfs

### 文件系统：

存储文件

数据库：以表的形式存储（文件不可以），但也是以文件的形式落地到文件系统的磁盘里。

在分布式系统装的数据库叫分布式数据库。

面向列式：传统数据库用行式存储

数据量很大，主要要减少寻址时间，空间大不怕冗余。牺牲存储空间换取查询时间。

传统关系型数据库：有宽表，字段很多。也是牺牲存储空间换取查询时间。

####hdfs：可以存储结构化，非结构化，半结构化。

####bigtable:

高可靠性：依赖hdfs ，3个副本

高性能：hbase内部存储机制。经过写缓存...（插入，查询非常快），缓存数据库都是列式存储

面向列: 行式存储的话，关系型db：2次查询，一次查询出where 另一次select 字段；

列存储:主键称作行键

可伸缩：可扩展

zookeeper为hbase提供了稳定的服务。zk存储了hbase的元信息。

zk是hbase的协调器，负责解决hmaster的单点问题。

######单机模式：

######分布式模式：

######伪分布式：一台机器开启多个jvm（开启多个进程）。类似机房中一台强大的服务器，通过云平台将大的服务器分成n个服务器。

######完全分布式：整个服务分布在各个节点上。

####安装部署：

启动集群，启动zk

解压

hbase使用java编写，需要配jdk

修改配置文件：

hbase.zookeeper.quorum , master,slave1,slave2

//zk的位置 不写的话默认有一个zk的位置信息

启动zookeeper并查看状态

启动hbase：需要在主节点启动

主从架构

看到zookeeper 中多了一个hbase

Web ------>   http://master:60010

###HBase的核心功能模块和基本概念

####一、HBase 和 Hadoop的关系：

1. HDFS为hbase提供底层高可靠的存储机制 

   > Hive底层依赖HDFS 和 MP
   >
   > HBase 底层依赖HDFS 计算依赖MR
   >
   > zk可以给hdfs提供ha机制
   >
   > Ambari 集群的管理工具

2. mapreduce 为hbase提供高性能的计算能力

####二、HBase的核心功能模块

![](/hbsase-1.png)

####1.client

客户端可以向hbase发起rpc请求，包括crud 对hbase发起增删改查的命令

client通过rpc协议 与hmaster 和 hregionserver进行通信

client：client通过rpc向hmaster发起管理类的操作

​	管理类操作：建表，删表，修改表，类似于RDBMS中的DDL语言

client：通过rpc协议向HRegionServer发起数据读写类的操作

​	数据读写类操作：写入数据、删除数据、修改数据、查询数据

​	类似RDBMS中的DML语言

client种类：java接口、hbase shell 命令行接口、avro序列化的访问接口

####2.zookeeper

Quorum 协调通信

分布式的问题：发起查询，先到达hmaster，hmaster传递请求，存在一致性和协调的问题

zk存储了分布式系统中公共的元数据，存储在zk集群中。

hbase中的zookeeper：

1）存储hbase的元数据信息 以znode的方式进行存储（相当于一个小型的数据库）

2）实时监控RegionServer：hbase向zk注册节点，每个regionserver都会向zk注册临时节点，若当机，zk所在的节点立刻向leader回报，告诉hbase某节点死掉，让hmaster做数据迁移

3）存储所有region的寻址入口：

region：bigtable（类比北京）随着越来越大，切分成多个region

bigtable的存储单位是region

4）保证hbase中只有一个hmaster：zk中有选举机制，选中一个为hmaster？

hbase-site.xml中  并没有指定hmaster，但指定了regionserver（slave1，slave2）

可能有多个hmaster，zk确定一个hmaster。

hmaster并没有存储元信息的权限，元信息存储在zk中，hmaster有调用权限。

####3.hmaster

hbase集群中运行在主节点的守护进程，在hbase集群中只能有一个hmaster

hmaster的主要是对表（hbase中存的是表，表的单位：region）和region的管理：

1）用户client对table表自身的增删改查

2）管理hregionserver的负载均衡，调整region分布

region分布在hregionserver

3）在region split后，负责新region的分配

达到阀值，split切分

4）在hregionserver当机后，负责失效的hregionserver上的regions迁移

![](/hbsase-1.png)

####4.hregionserver

主要负责响应用户的i/o请求，向hdfs中读写数据，==是hbase中最核心的模块==

1）HRegionServer管理了一系列的hregion对象

2）每个hregion对应了Table表中的一个region，一个表可以有多个region

3）hregion中有多个hstore组成，hstore的个数取决于表的列族个数

4）每个hstore对应了table中的一个列簇 columnfamily

新建一个表，默认一个region，达到10g，切分

hbase在Hadoop基础上，生成特定的文件格式hfile。

hbase先写日志，日志写好后，写memstore。日志的读写是hregionserver操控的。WAL

why先写日志？ 因为缓存中的数据容易丢失

列簇：多个列组成的

多少个列簇，就有多少hstore

hstore组成：memstore：（缓存）一般是64M//达到缓存阀值，会触发flush，通过写磁盘生成storefiles

​              	      storefiles：storefiles落地到hdfs的 是hfile 

 	当改数据，或删除时，并没有马上执行，内存中做删除标记，磁盘中并未删除，达到阀值，触发conpat合并，执行小合并（单独的region内）

hbase 其实只有增加数据，更改和删除是在后续的compact过程中进行的。

hbase块的大小是128k，hbase有自己的文件存储块。先落地到hdfs，后续合并，从hdfs拿出数据进行合并，更新和删除。

hadoop不写缓存，直接写入磁盘，所以会产生大量的小文件。

使用场景：

hbase有时候会用做缓存数据库（如联通的话单打印），因为hbase对大规模数据集可以提供很好性能的随机访问（按列）

hive里有多表连接。底层mapreduce，适合做批量的计算。

hbase里一般没有许多表的连接查询。

###HBASE的基本概念

####1.HBASE的数据模型

RowKey 行键，类似于rdbms中的主键

列簇/列族	Context：Name	Context为列簇，Name为列名

列名：

根据RowKey，列簇，列名能够确定一个单元格

HBase的单元格是一个三维立体的，不再是平面的

可以存储同一份数据的多个版本（时间戳）

确定单元格的值？

根据Rowkey、列簇、列名和版本号，就能够确定一个单元格的值

列簇的个数，是表的列数。

RDBMS中要确定列（回忆建表语句），HBase中要确定列簇。

时间戳是一个长整数。

HBase无数据类型，全部都是字节码形式（字节数组）。

单元格cell：

无数据类型，全部是字节码形式

单元格内容是不可以再分割的字节数据

单元格存储着同一份数据的多个版本

不同时间版本是按照时间的倒叙进行排序，时间最新的排在最前

物理模型：

####2.HBase的事务的特性ACID

 ACID：原子性，一致性，隔离性，持久性

NoSQL数据库：Not Only SQL：除了SQL还有更多的检索、查询...等的方式

基于列式存储

仅提供行级别的原子性（有一个列簇写失败，则回滚。要么全成功，要么全失败）

对同一行的所有列操作是原子性的，对该行的put操作要么整体成功，要么整体失败

HBASE：串行执行（排队，根据时间）保证keyValue不会被破坏。

RDBMS：锁机制(打架)

####3.HBase的表结构设计

1）模式设计 Shecmel-任何一个DB都有Schecmel设计

RDBMS模式设计是三范式3NF

第一范式：表中的列 不可再分割

第二范式：解决一行数据能够被独立的识别（主键可以确定一行数据。用户和订单表，用户id可以确定用户信息

​    订单id可以确定订单的信息，-------》拆分成2张表）

第三范式：解决传递函数依赖（表连接时部门id和部门名称都存在。因为部门id就可以确定部门，部门的名称可以放在部门表中）

​    HBase：表 

表名、rowkey、列簇可以唯一确定一张表

表：取名字

rowkey：怎么设计？包含什么内容？

列簇：应该有多少个？

timestamp：一个单元格应该存储同一份数据的多少个版本？

a）Rowkey行健设计

RDBMS中：分为业务主键：跟具体的业务挂钩，学号，身份证号

​             自然主键：数据库中的sequence序列 

hbase中rowkey会自动建立索引，很多数据是通过rowkey来查询的。

rowkey默认是有排序的，由低到高

rowkey是不可以分割的字节数，按照字典的顺序（asc码）由低到高排序

rowkey应该是基于具体的访问模式来进行rowkey的建模

rowkey决定了访问HBase表时可以得到的性能。（按rowkey查询是最快的）

HBase只能在rowkey上建立索引，不清楚rowkey的情况下，可以用扫描全表来查询。

​       注意事项：

​       HBase原生的rowkey只支持字典顺序从小到大排序

  Rowkey = Integer.MAX_VAlue - Rowkey

Rowkey尽量散列设计

  保证散列，就是保证所有的数据都不是在一个region上

Rowkey的长度尽量短

   太长的话，会增加存储开销，会影响存储效率

​      b) Column Family---列族设计

列簇：一些列的集合，一个列簇的所有成员都有共同的前缀

anchor:com.cnn    anchro:cnn.cn2

在物理结构上，列簇的成员在文件系统上都存储在一起。

同一个列簇，存储在一个hstore上。

#####案例

列簇1：用户基本信息

列簇2: 用户财务信息

flush和compassion是针对一个region的，flush（只要）的时候，不相关的列簇也要flush，造成许多没用的I/O负载。

尽量在应用中使用一个列簇。（一般使用一个，复杂的业务可以分2个列簇）

#####案例

行式存储和列示存储的区别：

行式存储：

列式存储：

hbase的文件存储基本单位：128k，（类比java中的字节byte）

hdfs的block块的空间：128M

RDBMS表：通过锁定一行，然后select + 条件 来查询

HBase表： 维度不同，通过锁定一行？，然后 锁定列簇名+列簇值来查询。（效率高）

两者都是要先锁定某条数据。

####----------------------------------复习---------------------------------------

zookeeper 分布式系统的协调器，HBase使用zookeeper做协调

day1

 HBase的安装部署：

1.解压

2.配置环境变量 注：启动hbase命令：start-hbase.sh 查看web 60010

hbase env.sh

JAVA_HOME=/usr/java/jdk1.7.....

3.hbase-site.xml

HBase安装模式：hbase.cluster.distributed true;

hbase.rootdir  hdfs://master:9000/hbase

4.配置regionslaves

配置hbase集群从节点的主机名

5.配置环境变量

export HBASE_HOME=

export PATH=

export HADOOP_CLASSPATH=$HBASE_HOME/lib/*

6.启动hbase

7.验证主从节点 HMaster   HRegionServer

day2

hbase的核心概念：

核心组件：Client	ZooKeeper HMaster	HRegionServer

hbase的数据模型：

列式存储nosql数据库

表设计

行键...

列簇...

65536 2字节

####----------------------------------上课内容---------------------------------------

启动hbase：start-hbase.sh

 Hbase的客户端：

1.hbase shell命令客户端

hbase shell （bin下执行）

####  1)常规命令：

status：查看hbase集群的状态信息

average load：region系数

version：查看hbase集群的版本

 ####   2)DDL命令：数据定义语言命令

#####  建表：create 表名 列簇

create 'member','info','address' ,'phone'''//结尾没有分号！

#####罗列表：list

  查看表信息：

​	      describe 'member' (不要漏引号！)

 disable	'member' 下线表

 enable    'member'  上线表

 is_disabled 'member'  //判断是否是下线表（有下划线！）

 is_enabled 'member'

判断是否存在：exists 'member'

#####删除表：	drop 'member'   //要先下线表再删除

修改表：修改表的列簇的

describe 'member'  //

alter前要先下线表！

#####删除列簇的两种方式：

​	(1）alter 'member' ,{NAME=>'info',METHOD=>'delete'} //凡是通过大括号操作的，左边相当于

​	键，没有引号！右边相当于值，有引号！

​       (2）alter 'member','delete'=>'info' //带引号的才是具体的操作数和具体操作

##### 添加列簇的两种方式：

​       (1）alter 'member',{NAME=>'info'} //可以理解为默认添加列簇

​	(2）alter 'member','NAME'=>'info'

####3) DML语言 数据操控语言,数据的增删改查

#####写数据：put 	'tablename','rowkey','列簇：列名' ，'单元格的值'

put 'member','xueba','info:name','zhangsan'//第一个列簇：含3个列

put 'member','xueba','info:age',20

put 'member','xueba','info:city','beijing'

..........

注：后put的值会覆盖原来的列中数据

scan 'member':扫描查看表

#####读数据：get 'member','xueba'

 get 'member','xueba','info'

 get 'member','xueba','info:name'

  通过时间戳来读：

 get 'member','xueba',{COLUMN=>'info:age',TIMESTAMP=>1128484763} 

 put 'member','xueba','info:age' 27 //执行此操作后生成新的时间戳，如果想获取上一个age，用上一个时间戳的值

  get 'member','xueba',{COLUMN=>‘info:age’,TIMESTAMP=>1128484763}

#####删除数据： delete 'member','xueba','info:age' //其实并没有删除，写入缓存中，所以支持低延迟的数据访问

deleteall 'member','xueba'

count 'member'; //查看行数

清 空 表：  truncate 'member'

#####案例：

将sogou500w 从mysql导入hbase 

bin/sqoop import --connect jdbc:mysql://master:3306/sogou --username hadoop --password hadoop --table sogou_20111230 --hbase-table sogou500w --column-family info --hbase-row-key uid --hbase-create-table -m 1;
