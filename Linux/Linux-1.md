###SHELL

操作系统底层的内核（kernal）：实现上层应用，上层命令的基本实现功能。如底层磁盘的读写，网络的连接，驱动内存管理，但不是用户可以直接控制的

壳Shell：将用户的请求翻译成kernel可以识别的信息。

SHELL分为：CLI和GUI

Linux操作系统的机器

​      Shell：GUI：GNOME

​     CLI：BASH

###BASH

命令的组成部分：命令、选项、参数

uname：显示系统信息 

uname -r：显示版本号	单字符参数

uname --all：

firefox &：表示命令放在后台运行

history： 历史命令列表

!! 		重复前一个命令

!字符 	重复前一个以‘字符’开头的命令

!num 	按照历史记录的序号执行命令

!?abc 	重复之前包含abc的命令

!-n		重复n个命令之前的那个命令

ctrl + r	历史命令搜索（输入历史命令的任意一部分）

esc + .    调用上一个命令的参数

Bash Shell

###通配符：

*	匹配0个或者多个

		匹配任意一个字符

	0-9]	匹配一个数字范围

	abc]	匹配列表里任何字符

	^abc]	匹配列表以外字符

###切换命令：

su 		切换用户身份但是不切换终端

su -		重新启动一个终端（默认家目录）--常用

sudo+命令	使用管理员身份运行命令

id	获取当前用户信息

passwd		修改当前用户密码

作业管理：

命令+&		在后台运行进程

jobs		查看后台所有作业

ctrl+z	暂停

ctrl+c	中止

bg+编号		让暂停的后台进程继续运行（先jobs查看编号）

fg+编号		将后台程序拉至前台

###LINUX文件系统结构：

\1. LINUX文件系统为一个倒转的单根树状结构

2.文件系统的根为"/"

3.文件系统严格区分大小写(windows中大小写不敏感)

4.路径使用"/"分割。（windows中使用"\"）

pwd：查看当前工作目录

 LINUX文件名称

\1. 文件的名称大小写敏感

\2. 名称最多可以为255个字符

\3. 除了正斜线外，都是有效字符

\4. 通过touch命令可以创建一个空白文件或者更新已有文件的时间

​    touch一个已经存在的文件是更新文件的时间戳。

\5. 以“.”开头的文件为隐藏文件

###列出目录内容

ls

ls -a：	显示所有文件（包括隐藏文件）

ls -l：	显示详细信息（目录内容）

ls -R：	递归显示子目录结构

ls -ld： 	显示目录和链接信息（非目录内容）

注：隐藏文件绝大多数都是配置文件

蓝色的都是目录。

###查看文件类型

file+目录/文件名

#####绝对路径与相对路径

注：工作目录是给当前的shell或者当前进程的概念

.	代表当前目录

～	代表普通用户的家目录

/root 根目录的家目录

-	上一个工作目录

###LINUX系统基础

cp:	    cp 源文件(文件夹)   目标文件(文件夹)

​    常用参数： -r 递归复制整个目录树

​    -v 显示详细信息

​    cp -r -v 和cp -rv等同

mv：	mv 源文件	目标目录

 		mv linuxcast2 linuxcast.net/cast (移动的同时重命名为cast)

​                mv 源文件 新文件名（重命名，不指定目录代表重命名）

rm：	常用参数：-i	交互式（提示信息）    

  -r	递归的删除包括目录中的所有内容

  -f	强制删除，没有警告信息（需要谨慎）

mkdir:	    创建一个目录

rmdir：	    删除一个空目录

rm -r (-f):    删除一个非空目录 

###LINUX系统目录（*代表所有用户都可以使用）

bin：里面存的是可执行文件即各种命令  *

boot：引导目录 vm-linux... 操作系统的内核

dev:	    计算机上所有的硬件设备

​    linux中，所有设备都会被抽象成一个文件

etc：几乎所有的配置文件

home：所有用户的家目录

lib:	所有的库文件（相当于windows的.ll文件）

 media：挂载用（插入u盘，会自动挂载到该目录）

 mnt：常规挂载 

opt：通常是空的。通常用来装一些大型软件，比如oracle

proc ：系统的实时信息。它存在在内存当中，是一个虚拟的文件系统

里面有cpuinfo，meminfo，进程号..  每次启动系统都会创建一个新的proc

sbin：也是可执行的二进制文件。bin：所有用户都可以执行

​    sbin只有root命令可以操作

selinux：安全机制相关的目录

sys：系统底层的一些信息，如查找硬盘，scsi的串号等...

tmp: 临时目录。tmp目录系统会自动的删除

usr：保存安装的应用软件。应用软件默认装在usr中

var：保存一些经常变化的信息。比如：服务器的信息，log

###LINUX常用命令

#####1.日期和时间：

data查询的是当前系统时间：

 data 回车：中国时间

data -u ：utc格林威治时间

data +%Y--%m--%d： 格式化显示时间

结果为：2012--10--03

data -s "20:20:20":	设置时间(root用户操作)

hwclock和clock查询主板保存的时间；

cal：查看日历

uptime：查看当前系统的运行时间（启动了多少时间，登陆用户数，负载）

#####2.输出查看命令

echo：	用于显示输入的内容

cat：	用于显示文件内容

head：	显示文件的头几行（默认为10）

  -n	指定显示的行树

tail：	用以显示文件的末尾几行（默认10行）

  -n	指定显示的行数

  -f	追踪显示文件更新（一般用于查看日志，命令不会退出，而是持续显示新加入的内容）

more	用于翻页显示文件内容（只能向下翻页）空格键翻页

less		用于翻页显示文件内容（带上下翻页）空格向下翻页 上下键

#####3.查看硬件信息

lspci：	用以查看PCI信息(列出所有pci设备，如网卡，声卡)

  -v	 查看详细信息

lsusb：	用以查看usb设备

  -v	  查看详细信息

lsmod：	用以查看加载的模块（windows的驱动程序）

#####4.关机、重启

shutdown  [关机、重启参数] 时间

-h	关机

-r	重启

如：shutdown -h now

shutdown -h +10 //10分钟后关机

shutdown -h 23:30

shutdown -r now

  poweroff :立即关闭计算机

  reboot: 立即重启计算机

#####5.归档、压缩

\1. zip用以压缩文件

zip linuxcast.zip  myfile

zip    压缩后	     源文件

\2. unzip用以解压缩zip文件

unzip linuxcast.zip

\3. 命令gzip用以压缩文件

gzip  linuxcast.net //压缩到当前目录！文件夹不可以！

自动加后缀.gz 且原文件被删除！ 只能在当前目录下

\4.命令tar 用以归档文件（不会对其进行压缩）

tar -cvf	out.tar linuxcast

tar -xvf	linuxcast.tar //释放归档至当前目录下

tar -zcvf	backup.tar.gz   /etc （先归档，后压缩）

-z 参数归档后的归档文件进行gzip压缩以减少大小

tar -zxvf backup.tar.gz 解压且释放归档到当前目录

#####6.查找

\1. locate keyword:	用以快速查找文件、文件夹

此命令需要预先建立数据库，数据库默认每天更新一次，可用updatadb命令手工建立、更新数据库

\2. find	查找位置	 查找参数 ： 用以高级查找文件、文件夹

例如： find .   -name   *linuxcast*   // 基于文件名

​      find /  -name *.conf

​      find / -perm 777    // 基于权限

find / -type d  // 根据文件类型，此处为目录

find  / -type  l // 查找所有的链接文件

find .  -name  "a*"  -exec ls  -l  {}  \ 

使用查找的结果，指定一些参数作为命令

除了ls -l命令 ，-exec  ......  {} 这部分是固定格式

本条find命令是，查看a开头的文件和目录的详细信息

  find命令中常用的查找条件：

-name	-perm	-user	-group	-type

-ctime: 基于修改时间的查找

-size：以字节计 +n：表示查找大于n的文件，-n为小于n的文件

#####7.VI文本编辑器

类linux系统都默认安装了VIM

VIM拥有三种模式：

\1. 命令模式（常规模式）：默认或按Esc进入，此模式下直接收特定的命令

 i	在光标前插入文本

o	在当前行的下面插入新行，自动进入插入模式

dd	删除整行

yy	将当前行的内容放入缓冲区（复制当前行）

n+yy	将n行的内容放入缓冲区（复制n行）从当前行开始算

p	将缓冲区中的文本放入光标后（粘贴）

u	撤销上一个操作

r	替换当前字符

/	查找关键字 查到以后按n键可以在不同关键字间移动

​       2.插入模式：在命令模式中按‘i’进入，按Esc返回命令模式

​       3.ex模式：命令模式下按：，进入ex模式。保存修改或者退出

:w	保存当前修改

:q	退出

:q！	强制退出，不保存修改

:x	保存并退出，相当于:wq

:set number / :set nu	显示行号

:! 系统命令	执行一个系统命令并显示结果

:sh	切换到命令行，使用ctrl+d切换回vim

###磁盘、分区、MBR与GPT

####磁盘

现在应用最广泛的仍然是传统的机械硬盘

存储是通过硬盘中的磁质盘片来保存数据

单碟硬盘：只有一个盘片

多碟硬盘：

每个盘片上下都会有一个磁头（盘片中也有）

磁盘转速越高，速度更快，性能也越好，但功率、发热量也越高 

通常：台式机：7200转  笔记本：5400转

固态硬盘：里面使用的是类似U盘的flash存储芯片。速度和性能更快。

一个硬盘有多个盘面，但是是整合到一起进行管理的。

####基本概念

cylinder:柱面  一圈的相同的track（轨道）就叫做一个柱面

sector:	扇区 由中心到边缘扩散出来的扇形区域

head: 磁头 通常是512字节（基本读取单位）

 磁盘在LINUX中的表示：

\1. Linux所有设备都被抽象成一个文件，保存在/dev目录下

2.设备名称一般为hd[a-z]或sd[a-z], ([a-z]为分区号),如hda、hdb、sda、sdb

3.IDE设备的名称为：hd[a-z], 

   SATA,  SCSI,  SAS,  USB等设备的名称为sd[a-z]

### 分区：

#####基本概念：

将一个磁盘逻辑的分为几个区，每个区当作独立磁盘，以方便使用管理

不同分区用：设备名称+分区号 方式表示。如sda1、sda2

主流的分区机制分为：MBR和GPT两种

物理硬盘有一个文件 /dev/sda，

每一个分区还会有一个单独的文件：/dev/sda1、/dev/sda2、/dev/sda3

注：分区并不是硬盘的物理功能，而是一种软件功能

####MBR：Master Boot Record

传统的分区机制，应用于绝大多数使用BIOS的PC设备

1）支持32bit和64bit系统

2）支持分区数量有限

3）只支持不超过2T的硬盘，超过2T的硬盘将只能使用2T空间（有第三方解决方法）

 MBR占据硬盘最开始的512字节：前446是引导代码

​    接下来的64字节是分区表

​    剩下的2字节是启动标识（所有可启动的硬盘，此值为55/AA）

##### MBR分区：

1）主分区：最多创建4个

2）扩展分区：一个扩展分区会占用一个主分区的位置

3）逻辑分区：Linux最多支持63个IDE分区和15个SCSI分区

因为只能创建4个主分区，所以要使用扩展分区和逻辑分区

使用扩展分区之后，逻辑分区是基于扩展分区创建出来的，主分区可以用，扩展分区是不可以用的。

必须基于此创建逻辑分区才可以使用。（具体过程：扩展分区占据一个主分区的位置，再在扩展分区的基础上创建出逻辑分区

逻辑分区是我们最终可以使用的分区）

####GPT：GUID Partition Table

1）支持超过2T的磁盘

2）向后兼容MBR

3）必须在支持UEFI（取代BIOS的新一代引导系统）的硬件上才能使用

4）必须使用64bit系统（因为GPT的寻址空间是64位）

5）Mac、Linux系统都能支持GPT分区格式

6）Windows7 64bit、windowServer2008 64bit支持GPT