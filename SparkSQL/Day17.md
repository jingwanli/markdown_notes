## Day17

find . -name 

select语句:

选: where

投: 投影

连: 连接 join

主节点:做raid , 在操作系统盘做raid1

---

ssh: 用于远程登录(操作系统级别的通信),心跳等要使用rpc传消息,不需要ssh.

国家信息系统:分级

![](/Users/lijingwan/Documents/img/屏幕快照 2018-05-05 上午11.53.30.png)

> 作业 ***(重要)

字段说明：

uid   （用户id）、itemid (商品id)、type  （用户行为分别是1、2、3、4）

ghash  (无用字段)、item_type（商品分类）、vdate（产生时间）

字段类型：uid String, itemid String, type Int, ghash String, item_type String, vdate timestamp

任务：

1、将数据保存到hive中

2、使用sparksql分析hive中的数据：

(1)不重复的用户数是多少

(2)找出UID出现最多的20个用户

(3)找出item_type出现最多的20个

(4)各种type出现的次数

(5)24小时各时间段，用户数统计

(6)按月统计，用户的总数

(7)按星期几，统计用户的总数

3、将全部数据导入hive，hive中保存数据类型为parquet

（选几个查询重新执行一遍，有什么发现）

将上面的查询全部改写为DSL

4、将最后一条sql语句的结果保存到mysql数据库中

5、所有的查询操作同时提供SQL语句与DSL的两种实现

要求：

提交的程序最终写在IDEA的程序中，要有足够的注释

###考察点：

1、数据清理

2、日期函数的应用

3、parquet文件格式

4、DSL语法

---

hive跑起来支持2个计算框架:

1. MR (yarn必不可少)
2. spark作业 : Yarn/ standalone 

在hive上写sql: 

hive on spark(hive + yarn)

sparkSQL(spark)