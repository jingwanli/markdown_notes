共5个job

\#create.sh

export HIVE_HOME=/home/xdl/apache-hive-1.2.1-bin

export PATH=$HIVE_HOME/bin:$PATH

\#echo $HIVE_HOME

hive -e 'create database if not exists tmall2'

hive -e "create external table if not exists tmall2.tmall_ori(uid string,time string,pname string,price double,number int,pid string) row format delimited fields terminated by '\t' location '/tmall2/tmall_ori'"

hadoop fs -mkdir /tmall2

 

\#createTable.job

type=command

command=bash create.sh

 

\#pour.sh

hadoop fs -put /home/xdl/tmall-201412.csv /tmall2/tmall_ori

 

\#pourData.job

type=command

command=bash pour.sh

dependencies=createTable

 

\#clean.sh

hive -e "create table if not exists tmall2.tmall_uid_pid(uid string,pid string) row format delimited fields terminated by '\t'"

hive -e "insert overwrite table tmall2.tmall_uid_pid select uid,pid from tmall2.tmall_ori"

 

hive -e "INSERT OVERWRITE TABLE tmall2.tmall_uid_pid select regexp_extract(uid, '^[0-9]*$', 0),regexp_extract(pid, '^[0-9]*$', 0) from tmall2.tmall_ori where regexp_extract(uid, '^[0-9]*$', 0) is not null and regexp_extract(uid, '^[0-9]*$', 0) != 'NULL' and regexp_extract(uid, '^[0-9]*$', 0) !='' and regexp_extract(uid, '^[0-9]*$', 0) != ' ' and regexp_extract(uid, '^[0-9]*$', 0) != 'null'

and regexp_extract(pid, '^[0-9]*$', 0) is not null and regexp_extract(pid, '^[0-9]*$', 0) != 'NULL' and regexp_extract(pid, '^[0-9]*$', 0) !='' and regexp_extract(pid, '^[0-9]*$', 0) != ' ' and regexp_extract(pid, '^[0-9]*$', 0) != 'null'"

 

\#cleanData.job

type=command

command=bash clean.sh

dependencies=pourData

 

\#alg.sh

/home/xdl/apache-mahout-distribution-0.10.1/bin/mahout recommenditembased --similarityClassname SIMILARITY_COSINE --input /tmall2/tmall2_uid_pid/ --numRecommendations 10 –booleanData

 

\#algData.job

type=command

command=bash alg.sh

dependencies=cleanData

 

\#etl.sh

hadoop fs -get /output0806/part-r-00000 /home/xdl/

\#mysql  -uhadoop -phadoop  -e 'create database if not exists tmall2

\#create table if not exists tmall2.tmall_recommand(uid varchar(50) not null,rlist           varchar(255) not null)'

\#mysql  - uazkaban - pazkaban -e 'select * from tmall2.tmall_recommand limit 10'

mysql -uazkaban -pazkaban << EOF

create database if not exists tmall2;

create table if not exists tmall2.tmall_recommand(uid varchar(50) not null,rlist varchar(255) not null);

load data local infile '/home/xdl/part-r-00000' into table tmall2.tmall_recommand fields terminated by '

\t';

EOF

 

\#etlData.job

type=command

command=bash etl.sh

dependencies algData

 

linux中打包为“recommend.zip”

![img](//note.youdao.com/src/WEBRESOURCEe9c9b5d2a28fe5a80b941fe4a77b753f)

Azkaban中upload recommend.zip 并执行

![img](//note.youdao.com/src/WEBRESOURCE2a19833158e6a1031303a64fc347fac8)

进入mysql

mysql>select * from tmall_recommand;

运行结果：

![img](file:////Users/lijingwan/Library/Group%20Containers/UBF8T346G9.Office/msoclip1/01/clip_image010.png)

 