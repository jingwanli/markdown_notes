-------------------上课内容---------------------

zookeeper最早是yahoo开发的，后被阿帕奇收购

因为zookeeper选举机制：so节点数不能小于3个

hbase对时钟要求非常高

安装zk：

1.解压

2.配置环境变量 ~/.bash_profile

3.修改配置

dataDir修改

server.1 第一个端口用于选举，第二个用于同步

.1 myid

目标：关注功能

Goole Chubby:解决分布式一致性问题

粗粒度：

细粒度：

zookeeper 不只针对hadoop，针对hbase等，hadoop用不到（不支持修改）

分布式锁服务：由zookeeper提供

zk中leader不接受客户端的访问

抢占了锁机制的节点，修改value后通知leader，leader通知所有节点

ajax底层是xml servlet 发起异步请求，回来的数据的形式是

http包，然后解析。然后做大量的javascript的封装，效率很低。jquery,ext,bootstrap等优秀的框架。

做数据的持久化jdbc效率很低。

zookeeper进行了封装，如锁服务，一致协调性的问题，存储数据，也可以将理解为小型的分布式数据库，实时性非常高。

hadoop2.0 提出了yarn---->通用 类比jdbc

now：50M 20M带宽

why zookeeper?

1）分布式系统 HA 需要主控协调器 

2）开发私有的协调程序。缺乏通用机制

3） 锁机制

应用场景：

...

软负载均衡：基于进程的并发

....

storm zookeeper负责传达（中间的媒介）减少耦合

hbase的元数据缓存到zookeeper，缓存至主节点，主节点容易当机，hbase离开zookeeper是无法运行的

角色分类：

Leader：有选举的周期，选举的时间很短10。同步的时间5。（leader消亡不代表服务器宕机，只是一个程序）

Learner：

Follower：与客户端交互，参加投票

Observer：任意一个节点都可以充当观察者，相当于一个线程。内部的后台角色，建立follower和leader之间的沟通机制

client：

工作原理：

zap协议：类http协议，两种模式：

恢复模式：当leader消亡，进入恢复模式选leader

leader选取出以后自动进入广播模式

zk核心：原子广播 

zookeeper存放数据的结构：层次化的树形结构

znode中，同一份数据可以有多个版本（因为有版本号）

znode可以被监控。

zookeeper集群都有存储的责任，共享所有空间。

两大类节点：临时节点：有序的（体现在路径上） 无序的

持久节点：

命令行接口：

zkCli.sh -server master:2181

Tip：此命令是显示zk中的所有数据，不是单单指的是master中的数据。

   如果从slave节点进入zk集群，可以写为zkCli.sh -server slave:2181 或者 zkCli.sh -server master:2181

ls / 

create /zk aaa; //持久性的节点

set  /zk bbbb;

get  /zk;

delete  /zk;

 ls /zookeeper;

------------笔记内容-------------