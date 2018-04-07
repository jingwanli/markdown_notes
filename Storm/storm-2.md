---
typora-root-url: ../../Documents
---

## Storm进阶

###核心概念和数据流模型

###Tuple元组

#####Tuple概念

tuple就是一个值的列表, tuple中的值的内容可以是任何类型,比如基本类型,  字符串,  字节数组. 

也可以自定义tuple中值的类型,只要实现序列化框架( java.io.Serializable)

如: class Student implements java.io.Serilizable{

}

Values values = new Values(new Student("zhangsan", 23), new Student("lisi", 33));

Values values = new  Values(12, 23, 44, 56);

select uid, name , age from student where student.uid = 'aaaaaa';

一行记录看做一个tuple

uid       name     age

111   zhangsan   23

Values tuple = new Values(111, "zhangsan", 23);

declarer.declear(new Fields("uid","name","age"));

>  tuple可以理解为一个键值对:
>
> 键: 在declareOutputFields方法中声明的Fields对象
>
> 值: 就是在emit方法中发送的Values对象

##### Tuple的生命周期

> collector.emit(Values); 
>
> collector.emit("aaaa", Values) //也可以手动指定id("aaaa");

> spout中, ack 一般不用. 因为回调此方法是有成本的.
>
> bolt是消息的处理者,它会追踪tuple

ISpout{

​	open(); //初始化做好发送tuple的发射器

​	nextTuple();  //用来生成Tuple

​	ack();//对成功发射的tuple确认..由storm框架自动回调

​	fail();// 如果有处理失败的tuple,也由storm框架进行回调

//ack 和 fail重写意义不大, 如果想要将相关信息写入数据库,可以重写此两个方法, 写入出错和成功的个数入数据库

//storm高容错,如果fail,storm会重新发射.

close();//topology被kill的时候,会被storm框架进行回调,释放一定的资源. 

}

当发射的消息被传送到消息处理者bolt时:

​	storm会跟踪:

​		tuple消息的树形结构 是否创建(内存中),(ack后,把id存入树形结构中)

​			==流式框架对内存的要求比较高.(Storm并没有提供回收机制.可能java会回收)==

​		 	根据messageid进行追踪

close方法: 将树形结构中的tuple剔除掉.

状态保存在zookeeper中,如果不kill,将会一直执行.

###Spout数据源

Spout是Storm中Topology的消息的生产者,就是tuple的创造者

Spout主动发射消息,相对Bolt来说它是一个主动的角色

##### Spout发出的消息

>  Spout需要从外部获取数据

​	HDFS文件系统

​	RDBMS系统

​	自己造数据

>  Spout向Topology发射的元组,

​	可靠的: 当storm对某一个元组处理失败, spout负责重新发射处理失败的元组

​	不可靠的: 当tuple处理失败时, spout发射出去元组,就把它遗忘了, 不会在它被处理失败时重新发射.  不指定id的是非可靠的(没法追踪)

> Spout的方法

- nextTuple() : 是Spout中最重要的方法, 负责向Topology发射tuple元组

  不要实现成阻塞的方式,当没有tuple发射的时候,nextTuple方法返回

  ```java
  java中方法意味着线程.??
  ```

- open() : 当Task在初始化的时候会被调用,有多少个task初始化就会调用多少次open方法

  ​	初始化的内容包含:

  -  发射元组tuple的发射器


  - TopologyContext 拓扑作业的上下文

- declareOutputFields() : 用于声明当前Spout的Tuple发送流.(流的单位是Tuple) ,是属于IComponent组件的

- getComponentConfiguration() 用于配置当前组件的参数 是属于IComponent组件的

- ack() :确认

- fail()  :  失败

- close() : 回收

  除了get….和 declare…这两个函数是属于组件的, 其他都是属于Spout本身.

  why多实现,不能多继承?

  继承的类对本身的影响很大.(类中定义的除了方法,还有很多其他元素,) 接口对父接口的污染比较小.


### Bolt

> Bolt简介

Bolt是消息的处理者,它是一个被动的角色,因为它不能主动的发送元组, 只能接受其他Bolt或者Spout发射过来的元组 , 它会对元组进行处理,然后生成新的元组继续发射给下一个Bolt, 或者它处理完元组之后,不再发射. 

> 功能

可以进行过滤,  函数调用,  合并,  写数据库

> 生命周期

首先,客户

- prepare( ) //相当于Spout的open方法,bolt可以设置并发量.有多少个blotTask,该方法就会被调用 初始化的操作
- execute( Tuple input, OutputCollector collector)
- declarerOutputFields( )
- clean( )

### Topology拓扑

>  Topology简介

网络世界里全都是拓扑结构

Storm的拓扑是类似于网络拓扑的一种虚拟.

Storm的拓扑相当于MapReduce的作业,MR中的job运行一段时间就会结束,而Storm的Topology会一直运行下去,除非kill掉.

Topology是由一组Spout( 数据源 ) 和bolt(数据操作)组成的图. Spout和Bolt之间通过tuple流分组进行连接.

```java
TopologyBuilder builder = new TopologyBuilder();
buidler.setSpout("wordspout",new WordSpout(),5);
builder.setSpout("wordspout2",new WordSpout(),6);
builder.setBolt("splitbolt", new WordSplitBolt(),3).shuffleGroupint("wordspout");
builder.setBolt("splitbolt", new WordSplitBolt(),3).shuffleGroupint("wordspout2");----??
builder.setBolt("wordcountbolt2",new WordCountBolt(),3).shuffleGroupint("wordspout2");
```

> Topology组件的并发数

Spout的并发数量

Bolt的并发数量

-  open和prepare 调用次数和并发量相同

- nextTuple 和 execute方法是一直运行的.(不断调用)

- Worker

- Executor: Worker进程内部的线程, 会执行同一个组件的一个或者多个Task.

- Task数量,在Topology中不会变化,一开始就制定了

  但Executor的数量不一定.Executor小于等于Task,默认情况下,二者是相等的.

  Executor: 物理上的数量

  Task: 程序上的的数量

### Stream消息流和String Grouping消息流组

JAVA中 : 字节的序列叫做流. 分为输入流,输出流 (输入输出是按照内存(程序)而言的!)

Storm中: 有向无界的Tuple的序列

> Storm的数据流模型

1. 随机分组(ShuffleGrouping) 尽量保证每个任务获得相等数量的tuple
2. 字段分组(FieldsGrouping) 
3. 全部分组(AllGrouping) 广播发送,将每一个Tuple发送到所有的Task. 谨慎使用(会出现重复,如果做单词统计,就不可以使用此分组方式)
4. 全局分组(GloalGrouping) 所有的Tuple发送到某个bolt的id最小的那个那个Task
5. 无分组(NoneGrouping) :等价于随机分组
6. 直接分组(DirectGrouping) : 直接将Tuple发送到指定的Task来处理

### Worker, Task.Executor三者的关系

见ppt

