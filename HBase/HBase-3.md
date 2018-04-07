---
typora-root-url: ../../Documents
---

## HBase核心概念—底层持久化

即时查询:hbase + hive

hive做数据的迁移

### HFile

HBase能做即时查询,因为有自己的文件格式-----HFile

![](/img/屏幕快照 2018-03-26 上午8.52.12.png)

HFile附带索引的文件格式 :

从右向左:  

Trailer:指针(维护在内存中) 

 指向Meta Index:数据的元信息 meta的索引

Data Index:data的索引

File info: 文件的基本信息

storefile 底层就是 hfile,对hfile做了轻量级的包装.

长度固定的模块:  trailer ,   file info

Data Index,  Meta index 记录了每个data块和meta块的起始点.

meta存放的是每个datablock的数据的元信息. 与data index 不同,data index存放的是物理块的信息.(针对文件)???

magic :数据保护的机制.  内容:随机数字, 类似hadoop的数据校验.

#### key-value

![](/img/屏幕快照 2018-03-26 上午9.08.45.png)

一个单元格的数据

> 结构 

1. 每一个key-value对简单的byte 数组,包含了很多项,并且有固定的结构

2. key 和value 分别是一个数组

3. 开始是两个固定长度的数值,分别表示key的长度和value的长度

4. key:

   key type: 墓碑标记: 删除某个单元格,key type标记为delete

   ​		小合并的时候,根据keytype的标记去做每一行数据的更新.

#### WAL机制  write ahead log

数据持久化: write ahead log :  write log at first,  then write data.

如果写入失败,日志回滚,重新写.

#### replication机制	(除了hdfs的备份)

实际生产中很少采用! 成本太高!

HBase的备份: 主从备份

zk: 共享日志

主集群 和 从集群 分别有各自的 hdfs

从集群做备机.(这里的备份指的是集群的备份!)

搭建框架: 运维工程师

#### RIT region in transition(不对外提供服务)

 	1. opening region :整个hbase启动的时候,或者做完region的切分之后
		2. closing  region : 正在关闭
		3. splitting region: 下线后才可以切分.

#### Load Balance 负载均衡

balance_switch: shell命令人工控制load balance

Balance_switch  true

Balance_switch  false

#### 写入流程

写入(put) 是HBase最基本,  最核心 ,  效率最高的操作.

**客户端**

- 写请求
- 重构数据: 在原有的基础上做大的改动. rpc请求: 数据的导入
- 提交请求
- 等待结果
- 识别错误并重试

**服务器端**

- 获取region


- 准备写入
- 请求锁
- 更新时间戳
- 写入MemStore : 没有真正写入磁盘
- 更新WAL

#### 查询

主要包括两种:  get和scan( get按照rowkey的方式查询效率最高)

**客户端**

- 构造get对象后,调用htable的get方法发出查询请求

**服务器端**

- 定位region

  - HBase有两个元数据表: root  meta	


  - root 用于保存meta表的region信息
  - meta表用于存放实际用户表的region位置信息
  - 三层的类B+树(适合数据量比较小)	
    - 第一层存储在zk上, 保存root表的region信息
    - 第二层到root表中查找匹配的meta表的region
    - 第二层 到meta表中检索用户表的region信息(直接保存在zk中)

  (注意:**hfile hml**先写内存,写磁盘)

  tip:hbase-daemon.sh  start master  

  ​     hbase-daemon.sh start regionserver(生产过程中,单独的节点,hdfs只用于保存数据)

  ​     yarn-daemon.sh start resourcemanager

  ​     (关机前) stop-all.sh

- 数据校验与转换 

  - 首先检测列族是否合法, 保证Get中的列簇存在表中
  - 如果get中没有列簇,则所有列簇都添加进方法中
  - 将get转化为scan

- region内查询

  - RegionScanner初始化 

  - StoreScanner(包括memstorescanner和storefilescanner)

  - MemStoreScanner 和 StoreFileScanner 

    hbase倾向于读还是写,  设置不同的menstore 和 filescanner大小

- hbase支持的压缩算法(倾向于写,设置压缩. 倾向于读,不设置,否则还需要解压缩)

  - LZO 压缩速度快,质量不好 基于apache协议,需要预安装

  - GZIP  压缩率更高但是速度更慢, java会使用Java自带的GZIP

  - SNAPPY  用来压缩和解压缩的C++开发包, 旨在提高压缩率和合理的雅俗速度需要预安装

  - 使用配置: 

    - 首先确认已经安装了相应的压缩算法库 //压缩是针对某个列簇的!列存储

      create 't1' , {NAME => 'c1', COMPRESSION=> 'GZ'}

      describe 't1'

    - 创建表的时候,不指定,默认为none

    - 使用alter修改

      disable 'member_user' //相当于关闭region

      is_enabled 'member_user'

      alter 'member_user', {NAME => 'info',COMPRESSION=>'GZ'} //更新整个region

      describe 'member_user'

      注: 解压缩是自动的

      hbase可以作为缓存数据库,秒级别写入大量数据.

      hbase没有数据类型,不代表hbase没有编码.  data_block_encoding=> "none" 表示存的是二进制字节数据.编码体现在: 外围的系统读数据时,要转化为外围系统能识别的编码!

      blockcache:默认是否开启缓存.

      bloomfilter: 布隆过滤器 

**---------------复习-----------------**

hdfs存两种信息:1. 日志信息 . 2. 数据信息

切分:从中间切. 切分后meta表会有更新

小合并的条件:

大合并:针对集群 默认24小时. close region 

hfile文件结构 : 不是简单的文本,还有很多索引信息,所以才可以快速查询

预写日志: 保证了数据的完整性

hbase有大小合并:所以hbase支持小文件的读写

hbase有memstore :所以支持实时的查询

 - (大公司生产环境中,hive也有集群的备份.但hdfs已经很安全)
 - **hbase的优化 大多是通过配置文件来做的.**

hbase小合并:

每次memstore满的时候都要flush到磁盘,当flush的时候有可能会触发小合并(只要达到3个,就会合并!) 本region内所有的memstore都要flush,优先合并64m的,有小文件会剩下.等待大合并(默认每24小时),所有的都合并成一个.(小合并是时时刻刻存在的)

合并和切分,客户端都不准访问数据库.

hbase做缓存数据库:因为hbase的写入操作非常快.写入操作比读快很多.(有自己的写机制)秒级可以写入十几万条数据.

---



##HBase高级应用

### 优化

####读写优化

- 读/写 共用的同一个jvm的堆内存

  ​	堆内存的分配:    读: 设置30%(缓存)    写:40~50%   两者的和不要超过80%  

  ​	**读的时候先去缓存查,缓存上没有的再去磁盘查**

  ​	倾向于读: hfile.block.cache.size 默认是20% (读的堆内存比例)

  ​	倾向于写: 可以将memstore设置的更大(如64M—>128M)

  ​	hbase.client.write.buffer:写缓存的默认大小

  - hbase-site.xml

    hbase.regionserver.handler.count:  默认是10. 太浪费 regionserver受理的

    RPC的并发量. 一般生产过程中可以改为100以上(要求:双路4核),改后能感到集群的处理速度大幅度提升. 生产过程中需要调试

  - hbase.regionserver.logroll.period:  提交log的间隔时间. 可以去调大.



####split & compact

- 合并的大小可以设定, 文件的个数

- 大合并的时候,对集群的性能有很大的消耗,hbase的region不对外提供服务.

  生产中:  将大合并关闭.  在凌晨3~5点,手动大合并.

- 小合并不会影响对外提供服务.  (除非合并太频繁) memstore调大,则会降低小合并频率

#### 配置文档: <<HBase的默认的配置说明.docx>>

linux配置信息优化包括两类:

1. hbase-site.xml:

2. hbase-env.sh :指定系统运行环境(jdk版本:1.7)

   配置说明中:

		HBase.rootdir
	
		hbase.master.port
	
		hbase.cluster.distributed 设置为true
	
		hbase.tmp.dir  临时目录
	
		hbase.master.info.bindAddress 绑定的外部端口
	
		regionserver绑定的端口
	
		regionserver基本信息绑定的端口
	
		最大重试次数 默认10  可以调大
	
		hbase.balancer.period: master执行负载均衡的周期
	
		hbase.master.logcleaner.ttl   hlog存在的最大时间
	
		hbase.hregion.memstore.flush.size: 调节memstore的值
	
		hbase.hregion.preclose.flush.multiplier: 
	
		hbase.hregion.max.filesize:  最大hstorefile的大小(触发切分). 默认:256M .调为10G. 当某个文件合并后超过storefile的最大值,执行切分
	
		hbase.hstore.compaction.Threshold: 默认3
	
		hbase.hstore.blockingStoreFiles:7

#### JVM OPTS

版本: 1.7 以上

堆内存:  一般设置为物理内存的二分之一(双路cpu)

GC自动垃圾回收的CMS   默认占比98%   设置为70%(GC会运行多个线程,CMS占比最大,生产中,调小,释放更多的内存空间给别人用)

#### key  design:设计的是否合理,直接影响查询的性能和可扩展性

根据业务需求定义  

- uid
- uid_ts (联通话单,如果此时uid设置为行键,数据量太多! ) 
- ts (例如:年_月)

//联合行键 要带双引号!

### 过滤器

#### BloomFilter 布隆过滤器

链表,  树等数据结构都是基于:将所有元素保存出来,通过比较

随着集合中元素的增加,需要的存储空间越来越大,检索速度也越来越慢

—> 通过散列表(Hash表)

单个元素放入集合前,先计算散列值. ——>布隆过滤器的基本思想——>通过一个hash函数将一个元素映射成一个位阵中的一个点. (有点像索引)

布隆过滤器属于列簇的一个属性

- 不用过滤器是一个很长的二进制向量和一系列随机映射函数

- 可以用于检索一个元素是否在一个集合中

- 优点: 空间效率和查询时间远远超过一般算法(hash值的空间占比很小,可以维护到内存,内存中维护一个矩阵点)

- 布隆过滤器可以每列簇 单独启用(NONE|ROW|ROWCOL)

  - NONE  没有过滤器
  - ROW   row行键的哈希在每次插入行时将被添加到布隆
  - ROWCOL  行键+列簇+列簇修饰的hash值

  ==布隆过滤器里如果没有,磁盘上一定没有!==

  ==布隆过滤器里有,磁盘上不一定有!==

- 使用方法

  - decscibe "member"  查看bloomfilter 默认是true
  - disable  "member"
    - alter "custom" ,{NAME=> "info",  BLOOMFILTER = > 'NONE'}
  - describe "member"

#### 其他过滤器

- HBase提供增删改查,而且提供了更高级的过滤器来查询

- 带有过滤器条件的RPC查询请求会吧过滤器分发到各个RegionServer(服务器端的过滤器) 可以降低网络传输的压力

- 过滤器的两类参数:

  - 抽象的操作符 :−LESS,  LESS_OR_EQUAL,EQUAL, NOT_EQUAL,GREATER_OR,EQUAL,GREATER,NO_OP

  - 比较器

    - 过滤器的核心组件之一 ,用于处理具体的比较逻辑 
    - RegexString (相当于数据库里的sql—>where)

    [ tip: hbase里的二级索引

    phenix:一个开源实现 协处理器

    gface 可以去写真正的sql ,finix将sql转化为java代码放入hbase中执行 ]

    1. RegexStringComparator支持正则表达式的值比较

    RegexStringComparatorcomp = new RegexStringComparator(“my.”);//以my开头的字符串

    SingleColumnValueFilterfilter = new SingleColumnValueFilter（cf，column，CompareOp.EQUAL，comp）

    scan.setFilter(filter)

    2. SubstringComparator用于检测一个子串是否存在于值中，不区分大小写

    SubstringComparatorcomp = new SubstringComparator(childStr);

    SingleColumnValueFilterfilter =new SingleColumnValueFilter(Bytes.toBytes(family),Bytes.toBytes(qualifier),CompareOp.EQUAL,comp);

    3. BinaryPrefixComparator是前缀二进制比较器BinaryPrefixComparatorcomp = new BinaryPrefixComparator(Bytes.toBytes("zha"));

       SingleColumnValueFilterfilter = new SingleColumnValueFilter(Bytes.toBytes(family),Bytes.toBytes(qualifier),CompareOp.EQUAL,comp);

    4. BinaryComparator是二进制比较器，用于按照字典顺序比较Bytes数据值

       BinaryComparatorcomp = new BinaryComparator(Bytes.toBytes(str));

       Filterfilter= new ValueFilter(CompareFilter.CompareOp.NOT_EQUAL,comp);

> 列值过滤器

```java
SingleColumnValueFilter用于测试列值相等（CompareOp.EQUAL）、不等（CompareOp.NOT_EQUAL）、范围(e.g,CompareOp.GREATER)等情况

SingleColumnValueExcludeFilter用于单列值过滤，但并不查询出该列的值。
```

> 键值元数据过滤器

```java
FamilyFilter 用于过滤列族
QualifierFilter 用于列名过滤
ColumPrefixFilter用于列名前缀过滤
```

>行键过滤器

```java
RowFilter 行过滤器
```