基本概念

Hive 是一个基于hadoop的开源数据仓库工具，用于存储和处理海量结构化数据。

Hive经过对语句进行解析和转换，最终生成一系列hadoop的map/reduce任务，

通过执行这些任务完成数据处理。

特点：

1.可扩展（Hive 可以自由的扩展集群的规模）

2.延展性（Hive支持用户自定义函数）

3.容错性（良好的容错性，节点出现问题SQL仍可完成执行）

hive内使用Hadoop的dfs命令：

dfs -ls / 只需要将hadoop关键字去掉即可。

这种方式比hadoop dfs -更高效，因为后者每次都会启动一个新的JVM实例，而hive会在

同一个进程中执行这些命令。

hive基本数据类型:

tinytint: 1byte

smallint:2byte

int: 4byte

bigint: 8byte

boolean:

float:

double:

string: "",''都可以

timestamp: 整数, 浮点数 或者 字符串

binary:

注意：所有数据类型都是对java接口的实现，因此，这些类型的具体行为细节和java中对应的完全一致。

timestamp的值可以是字符串，格式为：YYYY-MM-DD hh:mm:ss.ffffffff

hive会隐式的将类型转换为值较大的那个类型。

案例：

create table student(id bigint,name string,score double)

row format delimited

fields terminated by ',' ;

==灌数据方式1: load data local inpath '/home/xdl/test.txt' [overwrite] into table student;==

注意这里into后面有table关键字

hive集合数据类型

包括struct, map 和 array.

tip:当处理的数据的数量级是T或者P时，以最少的“头部寻址”来从磁盘上扫描数据是非常必要的。按数据集进行封装的话可以通过减少寻址次数来提供查询的速度。而如果根据外键关系关联的话则需要进行磁盘间的寻址操作，这样会有非常高的性能消耗。

struct:

create table test1(id int, info struct<name:string,age:int>)

row format delimited

fields terminated by ','

collection items terminated by ':' ;

001,zhangsan:20

002,lisi:25

003,wangwu:22

注意. zhangsan:20 整体为字段info，都是张三的信息(与类的属性类似)。

是一组不同类型的集合而已。zhangsan:20:95:shandong:coder

select info.age from test1;

array

create table test2(name string, id_list array<int>)

row format delimited

fields terminated by ','

collection items terminated by ':';

zhangsan,1:2:3:4

lisi,5:6

wangwu,7:8:9:10

注意.为一个相同类型的数组。整体为id_list。用id_list[下标]来查看。

select id_list[3] from class test2; 注：下标从0开始!

map

create table test4(id string,perf map<string,int>)

row format delimited

fields terminated by ','

collection items terminated by '\t' 

map keys terminated by ':';

001,job:80	team:60	person:50

002,Job:60	team:80

003,Job:90	team:70	person:100

select * from test4;

结果：

001	{"job":80,"team":60,"person":50}

002	{"Job":60,"team":80}

003	{"Job":90,"team":70,"person":100}

Time taken: 0.114 seconds, Fetched: 3 row(s)

Select perf['person'] from test4

Select perf['person'] from test4 where perf[‘person’] is not null

结果：

50

NULL

100

注意：MAP<STRNG，FLOAT>表示map中的每个键都是STRING数据类型的，而每个值都是FLOAT数据类型的。对于ARRAY<STRING>，其中的每个条目都是STRING类型的。STRUCT可以混合多种不同的数据类型，

但是STRUCT中一旦声明好结构，那么其位置就不可以再改变

HiveQL数据定义

数据库操作

create database if not exists sogou;

show databases;

show databases like 's.*';  //匹配项为正则表达式

数据库在Hive中的目录：/user/hive/warehouse/stu_test.db/

 为数据库增加描述信息：

create database sogou2 

comment 'sogou2 test database';

drop database if exists sogou2;

drop database if exists sogou2 cascade; //Hive自行先删除数据库的表

表操作

show tables in mydb;

1. 管理表/内部表/临时表 归Hive管，在hive删除时，Hive 也会删除数据。
2. 外部表

     create external table test5(id int,name string)

     row format delimited

     fields terminated by ','

     location '/data/stocks'; //注意：stocks也是目录！

3.分区表

a).创建扩展表 和 分区表

​     create external table sogou.sogou_ext(

ts string,uid string,keyword string,rank int,orders int,url string,

year int,month int,day int,hour int) //order是关键字，不可用

comment 'This is the sogou extends table'

row format delimited

fields terminated by '\t'

location '/sogou_ext';

create external table sogou.sogou_partition(

ts string,uid string,keyword string,rank int,order int,url string)

comment 'This is the sogou search data by partition'

partitioned by(year int,month int,day int,hour int)

row format delimited

fields terminated by '\t'

注意：不指定路径的话，内部表，外部表都默认在：/user/hive/warehouse/sogou.db/..

b)ssh 脚本,向扩展表灌入数据

\#infile=/sogou.500w.utf8

infile=$1

\#outfile=/sogou.500w.utf8.ext

outfile=$2

awk -F '\t' '{print $0 "\t" substr($1,1,4)"\t"......}' $infile>$outfile

执行脚本：

[xdl@master ~] bash sogou-log-extend.sh /home/xdl/sogou.500w.utf8 

/home/xdl/sogou.500w.utf8.ext

[xdl@master ~] hadoop fs -put ./sogou.500w.utf8.ext /sogou_ext

\>hive select * from sogou.sogou_ext limit 10;

c).开启动态分区功能：

set hive.exec.dynamic.partition = true;

set hive.exec.dynamic.partition.mode=nonstrict;

set hive.exec.max.dynamic.partition.pernode =1000;//最大动态分区个数

d).向分区表灌入数据

==灌数据方式2:insert overwrite table sogou.sogou_partition partition(year,month,day,hour)	 select * from sogou.sogou_ext; //overwrite 可以变成into==

本练习查询结果24个目录：/user/hive/warehouse/sogou.db/

sogou_partition/year=2011/month=12/day=30/hour=1/

/user/hive/warehouse/sogou.db/

sogou_partition/year=2011/month=12/day=31/hour=1/(还有2, 3，4，5...)

相当于建立了分区目录。

删除表：drop table if exists employees;

修改表：改表名：alter table log_message rename to logmsg;

   增加列：alter table log_message add columns(app_name string 

comment 'Application name',session_id long comment 

'The current session id'); 

==灌数据方式3:create table时，定义location //若文件夹中有2个文件，则将数据全部显示==

当列数不同时，将从多的列数，截取建表时的字段。

若列数中有部分是表中定义的列，则将其显示，非表中字段的置为null.(有交叉显示交叉)

//基准是表中字段。

==灌数据方式4:单个查询语句中创建表并加载数据==

create table sogou.sogou500w as select * from sogou.sogou_20111230

HiveQL数据操作

wc -l 文件名：查询文件行数

head 100 源文件 目标文件：截取100条数据

HiveQL查询

案例

1. 查询次数大于2次的用户总数

select count(a.uid) from(select uid,count(*) as cnt from sogou group by uid having cnt>2) a; 

//子查询必须加别名！

2. join语句: inner join; left outer join; right outer join;
3. order by: 对查询结果进行全局排序，既会有一个所有数据都通过一个reducer进行处理的过程。 

​       对大数据集来说，可能会消耗漫长的时间来执行。   

​       sort by: 只会在每个reducer中进行排序，既局部排序，每个reducer的输出是有序的(但并非全局有序)。

可以提高后面进行的全局排序的效率。

   默认两者都是升序。

Hive内置函数

hive>show functions;

select pow(rank,2) from sogou limit 3; //求平方

select pmod(rank+1,2) from sogou limit 10; //取模

select cast(rank as double) from sogou limit 10; //强制类型转换

select concat(uid,keyword) from sogou limit 10; //拼接字符串

select length(uid) from sogou limit 10;//查询字符串长度

select locate("ee",uid,9); //查询uid中，第9个位置之后字符串ee

​    第一次出现的次数

select count(distinct e.uid) as cnt from(select * from sogou where rank<=3 and 

order = 1) e; //表的嵌套

select * from sogou where url like 'http%' limit 10; //模糊查询

select rank,order,count(*) from sogou group by rank,order;//多字段分组

insert overwrite/into table sogou_limit select * from sogou limit 3;

关于insert插入数据的总结：

1).insert into/overwrite命令不能用在列数不同的表内。

2).insert into 是在数据后追加。

3).insert into/overwrite 列数不同时，报错！

4).insert overwrite 是覆盖原数据的写入。

将不同字段拼接成一个json：

select concat("{\"ts\":\"",ts,"\",\"uid\":\"",uid,"\",\"keyword\":\"",keyword,"\",\"rank\":"

,rank,",\"orders\":",orders,",\"url\":\"",url,"\"}") from sogou.sogou500w_100 limit 100;

结果：

{"ts":"20111230000041","uid":"589a702d9ef4e380693815f6f364e905","keyword":"管理人员安全教				    育","rank":8,"orders":1,"url":"http://wenku.baidu.com/view/a48d491efad6195f312ba678.html?from=related&hasrec=1"}

案例：搜索过仙剑奇侠传的用户还搜索过什么内容？

//子查询相当于截取一部分，join相当于在笛卡尔积里过滤数据！

select m.uid,m.keyword from sogou m join (

​     	select distinct uid from sogou where keyword like '%仙剑奇侠传%'

​     	) n on m.uid=n.uid where m.keyword not like '%仙剑奇侠传%';

select m.uid,m.keyword from sogou m join

 	(select distinct uid from   sogou where keyword like '%仙剑奇侠传%') n on m.uid = n.uid where m.keyword rlike '~(.*仙剑奇侠传.* )';

Hive视图

create view 视图名 as select * from 表 where...

describe 视图名

drop view 视图名

解题思路

1、使用临时表	2、多个视图实现	3、复杂的SQL

案例：

搜索关键字长度大于256（不区分中英文），并且次数<3的UID

select e.uid,count(*) as cnt  from (select * from sogou where length(keyword)>256) e group by e.uid having cnt<3;

create view sogou_view as select * from sogou where length(keyword)>256;

select uid,count(*) as cnt from sogou_view group by uid having cnt<3;

上午7-9点之间，搜索过“赶集网”的用户，哪些用户直接点击了赶集网的URL.//substr 从1开始

create view sogou_view2 as select * from sogou where cast(substr(ts,9,2) as int)>=7 and 		  

cast(substr(ts,9,2) as int)<9 and keyword like '%赶集网%'；

select * from sogou_view2 where url like '%ganji%';

Rank<3的搜索中，多少用户的搜索次数>2

select count(\*) from (select e.uid,count(*) as cnt from (select * from sogou where rank <3) e group by uid having cnt>2) a;

Tip:语句3：隐式的内连接，没有INNER JOIN，形成的中间表为两个表的笛卡尔积。//语句3,4的表述是相同的

SELECT O.ID,O.ORDER_NUMBER,C.ID,C.NAME

FROM CUSTOMERS C,ORDERS O

WHERE C.ID=O.CUSTOMER_ID;

语句4：显示的内连接，一般称为内连接，有INNER JOIN，形成的中间表为两个表经过ON条件过滤后的笛卡尔积。

SELECT O.ID,O.ORDER_NUMBER,C.ID,C.NAME

FROM CUSTOMERS C INNER JOIN ORDERS O ON C.ID=O.CUSTOMER_ID;

hive中无法制定reduce个数。用mapreduce可以指定。sort by，order by都是mapreduce排序，都是实现comparator，comparable来完成。

Hive & json

select get_json_object(a.j,'$.name') from (select substring(json,2,length(json)-2) as j from sogou.weixin) a

sqoop

通过sqoop将mysql中表的数据导入到hdfs中

bin/sqoop import --connect jdbc:mysql://master:3306/sqoop --username hadoop --password hadoop --table test1 --target-dir /sqoop --fields-terminated-by '\t' -m 1

tip: hive中建表时，指定location(即/sqoop),本次导入，相当于将mysql的数据，导入hdfs文件夹

通过sqoop将hive中的数据导出到mysql数据库中

bin/sqoop export --connect jdbc:mysql://master:3306/sqoop --username hadoop --password hadoop --table test1 --export-dir '/sqoop' --fields-terminated-by '\t'

UDF自定义函数

编写 UDF 函数的时候需要注意以下几点:

a)自定义 UDF 需要继承 org.apache.hadoop.hive.ql.UDF。

b)需要实现 evaluate 函数。

c)evaluate 函数支持重载。

 package hive.connect;

import org.apache.hadoop.hive.ql.exec.UDF;

public final class Add extends UDF {

​		public Integer evaluate(Integer... a) {

​			int total = 0;

​			for (int i = 0; i < a.length; i++){	

​				if(a[i]!=null){

​					total += a[i];

​		    			return total;

​				}

​			}

​		}

}

4、步骤

a)把程序打包放到目标机器上去;

b)进入 hive 客户端，添加 jar 包:hive>add jar /home/xdl/udf_test.jar; 

c)创建临时函数:hive>CREATE TEMPORARY FUNCTION add_example AS '包名.类名';

d)查询 HQL 语句:

SELECT add_example(8, 9) FROM scores;

SELECT add_example(scores.math, scores.art) FROM scores; 

SELECT add_example(6, 7, 8, 6.8) FROM scores; 

e)销毁临时函数:hive> DROP TEMPORARY FUNCTION add_example;

Azkaban

Azkaban-web的启动：

Azkaban-executor的启动：

Azkaban job部署：

1.将azkaban-jobtype-2.5.0.tar.gz 拷贝至azkaban-executor-2.5.0的plugins并解压

2.进入上面的jobtype文件夹，vim common.properties ,配置后需要重启azkaban

3.cd azkaban-executor-2.5.0 , --bin--...shutdown.sh---...start.sh

4.job的书写

1) 家目录下：cd azkaban-->  vim create.sh

​    添加： export HIVE_HOME=/home/xdl/apache-hive-1.2.1-bin

​     export PATH=$HIVE_HOME/bin:$PATH

​     echo $HIVE_HOME

hive -e 'create database if not exists tmall2' 写脚本

......

hive -e '...'  一行命令

hive -f 将命令写成文件，文件结尾是.hql

2)azkaban目录下，vim createTable.job

type=command

command=bash create.sh

dependencies=upload(上次执行的job)

3)打包

注意：azkaban脚本里 hive -e 后面创建表都是hive表。

zip -z a.zip 多个源文件

tar -zcvf 目标 源文件 //.tar.gz

tar -jcvf //.tar.bz2

tar -Jcvf //.tar.xz 

dir=/tmall2;

hadoop fs -ls $dir; 

if [ $? != 0 ];then 

echo hadoop fs -mkdir $dir;

fi //