### Linux网络基础配置

#### 以太网连接

1. 在Linux中，以太网接口被命令为：eth0，eth1等，0、1代表网卡编号

2. 通过lspci命令可以查看网卡硬件信息（如果是usb网卡，则可能需要使用lsusb命令）

3. 命令ifconfig用来查看接口信息

   ifconfig -a  查看所有接口

   ifconfig eth0	查看特定接口

4. 命令ifup、ifdown用来启用、禁用

   ifup eth0

   ifdown eth0

   ls -a —> eth0:第一块网卡 物理网卡

   ​	—>lo 环回接口 饮食127网段

#### 配置网络信息

> 命令：setup 静态手工配置ip地址、子网掩码、dns网关（文本界面）

​	一般网关ip都是1或.254 

​	配置后要使用ifup eth0启动网卡  —> ifconfig

> 网络相关配置文件

 	1. 网卡配置文件 /etc/sysconfig/network-scripts/ifcfg-eth0
		2. DNS配置文件 /etc/resolv.conf
		3. 主机名配置文件 /etc/sysconfig/network
		4. 静态主机名配置文件 /etc/hosts

#### 网络测试命令

> 测试网络连通性 

​	ping 192.168.1.1  ping www.linuxcast.net

​		测试连通后ip地址、子网掩码及硬件配置是ok的

> 测试DNS解析  —— 提供域名到ip地址的解析

​	host www.linuxcast.net			dig www.linuxcast.net

​	只要可以返回值，就表示是正确的	

> 显示路由表  —— 实现路由功能

​	ip route

> 追踪到达目标地址的网络路径 —— 实际上是追踪到达目标地址所经过的路由设备

​	traceroute www.linuxcast.net

> 使用mtr进行网络质量测试（结合了traceroute和ping）

​	mtr  www.linuxcast.net   snt表示发了多少包 可以看到丢包率 

#### 修改主机名

> 实时修改主机名

​	hostname train.linuxcast.net

> 永久性修改主机名

​	/etc/sysconfig/network

​	HOSTNAME=train.linuxcast.net

#### 故障排查

网络故障排查遵循：==从底层到高层、从自身到外部==的流程进行

1. 先查看网络配置信息是否正确：

   -IP地址 -子网掩码 -网关 -DNS

2. 查看到达网关是否连通

    ping 网关IP地址

3. 查看DNS解析是否正常

    host   www.linuxcast.net

   host    www.126.com

   host    www.douban.com

   ​

### Linux多命令协作：管道及重定向

  开源文化的核心理念之一：不要重复发明轮子  《大教堂和集市》

#### 管道和重定向

命令行shell的数据流有以下的定义：

|  名称  |   说明   | 编号 | 默认 |
| :----: | :------: | :--: | :--: |
| STDIN  | 标准输入 |  0   | 键盘 |
| STDOUT | 标准输出 |  1   | 终端 |
| STDERR | 标准错误 |  2   | 终端 |

命令通过STDID接收参数或数据，通过STDOUT输出结果或通过STDERR输出错误。

通过管道和重定向我们可以控制CLI的数据流

> #####重定向

| 关键字 |             定义             |            案例             |
| :----: | :--------------------------: | :-------------------------: |
|   >    | 将STDOUT重定向到文件（覆盖） |       echo  > outfile       |
|  \>\>  | 将STDOUT重定向到文件（追加） |                             |
|   2>   | 将STDERR重定向到文件（覆盖） |                             |
|  2>&1  |     将STDERR与STDOUT结合     |                             |
|   <    |         重定向STDIN          | grep linuxcast </etc/passwd |

> ##### 管道

​	将一个命令的STDOUT作为另一个命令的STDIN

​	案例： ls -l |grep linuxcast

​		    find / -user  linuxcast | grep  video

​		    find  /  -user  linuxcast | grep  video |  tar

​		    find /  -user  linuxcast |  grep video 2>/dev/null    (将标准错误重定向到null，任何重定向到null都被丢弃)	

####小结

> 管道通常用来组合不同的命令，以实现一个复杂的功能

> 重定向通常用来保存某命令的输出信息或者错误信息，可以用来记录执行结果或者保存错误信息到一个指定的文件

### Linux命令行文本处理工具

#### 文件浏览

- cat		查看文件内容
- more	以翻页形式查看文件内容(只能向下翻页)
- less         以翻页形式查看文件内容(可上下翻页)
- head        查看文件的开始10行 (或制定行数)
- tail            查看文件的结束10行(或制定行数)

#### 基于关键字搜索

命令grep用以基于关键字搜索文本(实际上是一个正则表达式的命令 )

grep 'linuxcast' /etc/passwd (passwd是一个纯文本文件)

find / -user linuxcast | grep Video :先查找所属用户为linuxcast的文件,然后查找						     包含Video关键字的文件.

参数:

- -i    在搜索的时候忽略大小写(Linux区分大小写)
- -n    显示结果所在行数
- -v    输出不带关键字的行( 取反操作)
- -Ax  在输出的时候包含结果所在行之后的制定行数
- -Bx  在输出的时候包含结果所在行(关键字所在的行以及之前的x行) 

#### 基于列处理文本

命令cut用以基于列处理文本内容

cut	-d:    -f1  /etc/passwd

grep linuxcast   /etc/passwd|cut -d:  -f3

参数:

- -d    指定分割字符(默认是TAB)
- -f    指定输出的==列号==
- -c    基于==字符==进行切割    cut  -c2-6  /etc/passwd 

#### 文本统计

命令wc用以统计文本信息

wc   linuxcast

参数:

- -l     只统计行数
- -w   只统计单词
- -c    只统计字节数
- -m   只统计字符数 

#### 文本排序

命令sort用以对文本内容进行排序(只支持英文和数字)

sort  linuxcast (默认基于首字母/数字排序)

参数:

- -r      进行倒序排序
- -n     基于数字进行排序
- -f      忽略大小写
- -u     删除重复行
- -t  c   使用c作为分隔符分割为列进行排序 
- -k  x   当进行基于指定字符分割为列的排序时,指定基于哪个列排序
- -o      将结果写入原文件    sort -r number.txt -o number.txt

案例 

​	sort -n  -k  2  -t ' : '  facebook.txt

#### 删除重复行

- 命令sort  -u 可以删除重复行
- 命令 uniq 用以删除重复的相邻行  cat linuxcast  |  uniq

#### 文本比较

命令diff用以比较两个文件的区别

diff  linuxcast  linuxcast-new

参数

- -i	忽略大小写

- -b    忽略空格数量的改变

- -u    统一显示比较信息(一般用以生产patch补丁 文件)

  ​	diff -u linuxcast linuxcast-new > final.patch

#### 检查拼写

命令aspell用以显示检查==英文==拼写

​	aspell check linuxcast

 	aspell  list   <   linuxcast 

#### 处理文本内容

- 删除关键字:  tr -d   'TMD'  < linuxcast(只接受重定向方式,不接受文件,一般处理数据流时)
- 转换大小写:  tr 'a-z'  'A-Z'  < linuxcast

#### 搜索替换

命令sed用以搜索并替换文本(用正则表达式来处理文件)

​	sed  's/linux/unix/g'   linuxcast  //将文件中的linux替换为unix(加g代表替换所有.不加仅仅替换第一个)

​	sed   '1,50s/linux/unix/g'   linuxcast

​	sed -e 's/linux/unix/g'  -e 's/nash/nash_su/g'  linuxcast //指定多个

​	sed -f sededit linuxcast : 将命令存在sededit文件中,使用时用-f 调用就可以.

