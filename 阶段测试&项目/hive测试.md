一、数据描述

1)数据参数 用户的历史微博数据 截止到20131215 压缩后244MB，解压后878MB

2)数据类型 整个数据是 json 格式

json 中字段描述: beCommentWeiboId 是否评论 beForwardWeiboId 是否是转发微博 catchTime 抓取时间 commentCount 评论次数

content 内容

createTime 创建时间

info1 信息字段 1

info2 信息字段 2

info3 信息字段 3

mlevel no sure

musicurl音乐链接

pic_list 照片列表(可以有多个) praiseCount 点赞人数

reportCount 转发人数

source 数据来源

userId 用户 id

videourl 视频链接

weiboId 微博 id

weiboUrl 微博网址

二、实操题目

\1. 组织数据(Hive)(10 分)

创建 Hive 表 weibo(json STRING)，表只有一个字段，导入所有数据，并验证查询前 3 条数 据

注意:该表是下面 2、3 题的基表

\2. 统计需求(Hive)(35 分)

(1)统计微博总量和独立用户数

(2)统计用户所有微博被转发的总次数，并输出 TOP-3 用户 (3)统计微博被转发最多的前 3 位用户的 id (4)统计每个用户的发送微博总数，并存储到临时表

(5)统计带图片的微博数

(6)统计使用 iphone 发微博的独立用户数

(7)统计微博中评论次数<1000 的用户 ID 与数据来源信息，将其放入视图中，然后统计视 图中数据来源是“iPad 客户端”的用户数目

3 特殊需求(35 分)

(1)实现 Hive UDF 完成下面的需求:(10 分)

 将微博的点赞人数与转发人数相加求和，并将相加之和降序排列，取前10条记录

(2) 实现 Hive UDF 完成下面的需求:(15 分)

-   微博内容content中的包含某个词的个数，方法返回值是int类型的数值
-   使用该方法统计微博内容中出现“iphone”次数最多的用户，最终结果输出用户ID和

次数

\4. 数据 ETL(20 分)

-   在MySQL数据库中创建一张表(10分)
-   使用Sqoop将上面题目2(4)的结果导入MySQL的新建表中，并使用查询验证导入

是否成功 (10 分)

\------------------------------------------------------------------------------------------------------

1.

create external table weibo(json string)

row format delimited

location '/exam1/weibo';

hadoop fs -ls /exam1;

hadoop fs -put /home/xdl/data/weibo/* /exam1/weibo

create external table weibo_ext(

beCommentWeiboId string,beForwardWeiboId string,catchTime string,commentCount int, content string,createTime string,info1 string,info2 string,info3 string,mlevel string,musicurl string,pic_list string,praiseCount int,reportCount int,source string,userId string,videourl string,weiboId string,weiboUrl string)

row format delimited

fields terminated by '\t'

location '/exam1/weibo_ext';

insert into table weibo_ext select get_json_object(a.j,'$.beCommentWeiboId'),get_json_object(a.j,'$.beForwardWeiboId'),get_json_object(a.j,'$.catchTime'),get_json_object(a.j,'$.commentCount'),get_json_object(a.j,'$.content'),get_json_object(a.j,'$.createTime'),get_json_object(a.j,'$.info1'),get_json_object(a.j,'$.info2'),get_json_object(a.j,'$.info3'),get_json_object(a.j,'$.mlevel'),get_json_object(a.j,'$.musicurl'),get_json_object(a.j,'$.pic_list'),get_json_object(a.j,'$.praiseCount'),get_json_object(a.j,'$.reportCount'),get_json_object(a.j,'$.source'),get_json_object(a.j,'$.userId'),get_json_object(a.j,'$.videourl'),get_json_object(a.j,'$.weiboId'),get_json_object(a.j,'$.weiboUrl') from (select substring(json,2,length(json)-2) as j from exam1.weibo) a;

select json from weibo limit 3;

2.

1) select count(*) from weibo_ext; 

1451868

select distinct userId from weibo_ext;

3924114217

3924244216

3926428816

3928773378

3929303996

3932110045

3932572948

3932784154

3935736330

3936370767

3936451644

3941779768

41915

531080

6002200

608351

77859357

83509471

Time taken: 58.407 seconds, Fetched: 78540 row(s)

2）select userId,sum(reportCount) as cnt from weibo_ext group by userId order by cnt desc limit 3; 

1793285524	76454805

1629810574	73656898

2803301701	68176008

3)select userId from weibo_ext order by reportCount desc limit 3;

4）create table total(uid string,total int) 

row format delimited

fields terminated by '\t';

insert overwrite table total select userId,count(*) as cnt from weibo_ext group by userId;

hive> select * from total limit 10;

OK

1000473355	1

1001024965	1

1001523180	2

1002149043	1

1002280344	2

1002392442	2

1002854697	1

1002861732	1

1002906354	2

1002909672	3

Time taken: 0.098 seconds, Fetched: 10 row(s)

5)select count(*) from weibo_ext where length(pic_list)>2;//如果为空，长度为2--'[]'

​	750512

6)select count(distinct userId) from weibo_ext where source='iPhone客户端';

​	921

7)create view view2 as select b.userId,b.source from weibo_ext b join (select userId,count(commentCount) as cnt from weibo_ext group by userId having cnt<1000) a on a.userId = b.userId; 

//视图存的其实就是一个sql语句

select count(distinct userId) from view2 where source = 'iPad客户端'; 

328

\3. 方法一：

public class Contain_example3 extends UDF{

​	int count = 0;

​	public int evaluate(String str,String se){

​		if(str.indexOf(se)==-1){

​		return 0;

​		}

​		while(str.indexOf(se)!=-1){

​			count++;

​			str = str.substring(str.indexOf(se)+se.length());

​		} 

​		return count;

​	}

}

//此方法substring影响效率

add jar contain_example5.jar;

create temporary function contain_example5 as 'hive.Contain_example3';

select userId,sum(contain_example6(content,"iphone")) as cnt from weibo_ext group by userId order by cnt desc;

方法二：

package hive;

import org.apache.hadoop.hive.ql.exec.UDF;

public class Contain_example6 extends UDF {

​	

​	public int evaluate(String str,String se){

​		int count = 0;

​		int location = 0;

​		while((location = str.indexOf(se, location)) != -1) {

​			count++;

​			location += se.length();

​		}		

​		return count;

​	}	

}

mysql->

4.

create table total2(uid varchar(100),count int(100));

bin/sqoop export --connect jdbc:mysql://master:3306/exam1 --username hadoop --password hadoop --table total2 --export-dir '/user/hive/warehouse/exam1.db/total' --fields-terminated-by '\t'

//此路径一定要写对，如果是内部表，一定要选定exam1.db！