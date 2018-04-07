###Linux磁盘及文件系统管理

####使用fdisk进行磁盘管理

#####Fdisk分区工具

1. ​	 fdisk是IBM的老牌分区工具，几乎所有的linux的发行版本都装有fdisk，包括在rescue（救援模式）模式下依然可以使用。
2. ​	(mac, linux,windows系统都有 )
3. ​	 fdisk是一个基于MBR的分区工具，如果需要使用GPT，则无法使用fdisk进行分区。
4. ​	1.fdisk命令只有root可以运行
5. ​	2.fdisk -l可以列出所有安装的磁盘及其分区信息
6. ​	3.fdisk /dev/sda 可以对目标磁盘进行分区操作
7. ​	4.分区之后需要使用partprobe命令让内核更新分区信息，否则需要重启才能识别新的分区
8. ​	5./proc/partitions文件也可以用来查看分区信息

#####步骤：

1. 启动虚拟机之前添加一块硬盘 (/dev/sdb)

​    2 .fdisk -l

​    每个磁盘的大小就是：磁头大小\**磁头数量\*扇区数量\**柱面数量

​    列出的分区标识：Device Boot：分区号

​    Start：起始柱面

​    End：结束柱面

​    Blocks：分区大小 

​    Id：分区类型

​    System：分区类型

3. fdisk /dev/sdb

​         m //查看帮助

​     	 n:添加一个新的分区

​	 p：查看当前的分区 

​	 t：改分区的Id号

​	 w：保存修改

 ......

4. n ---> p(主分区) ---> 1--->+2G--->p （/dev/sdb1）

​    n---> e(扩展分区)--> 2 --->缺省值 263 --->回车(表示剩下的所有)

5. 创建逻辑分区（因为扩展分区是不可以直接使用的）

​    一般来讲，操作系统所在的分区设为主分区，剩下的自动化为逻辑分区（windows里养成的习惯）

​    n ---> l ---> 5(不管现在到哪个分区，逻辑分区的分区号永远从5开始) ---> 263 ---> +2G --->p (/dev/sdb5)

6. 为了方便系统识别，我们还要去指定分区的Id

​    m ---> t ---> 1 ---> L ---> 83(对第一个/dev/sdb1 进行的设置)

7. 至此为止，所有的分区还没有写入硬盘，仅仅存于内存中，如果退出会全部丢失！

​    m ---> w 写入磁盘了

8. ls /dev/sdb 查看（有时候新分区没有出来，需要使用partprobe命令让内核更新分区信息，否则需要重启才能识别新分区）
9. partprobe命令 （最好每次都用一下）
10. cat /proc/partitions ：列出分区信息的另一种方法

###Linux的文件系统

操作系统通过文件管理系统管理文件及数据，磁盘或者分区需要创建文件系统之后才能够为操作系统

使用，创建文件系统的过程又称之为格式化。

1.没有文件系统的设备又称之为裸（raw）设备

2.常见的文件系统有fat32、NTFS、ext2、ext3、ext4、xfs、HFS等

3.文件系统之间的区别：日志、支持的分区大小、支持的单个文件大小、性能等

windows下主流的文件系统是：NTFS

Linux下主流的文件系统是：Ext3、Ext4

​    Linux版本5    版本6

Linux支持的文件系统有：ext2, ext3, ext4, fat(msdos), vfat, nfs, iso9660, proc, gfs, jfs

 

###创建文件系统：

命令：1.  mke2fs -t ext4 /dev/sda3

​               常用参数：-b   blocksize    指定文件系统块大小（2048 4096等字节）

指的是每次文件系统读取的最小单位，默认4096/4k

​    -c	建立文件系统时检查坏损块  

​    -L label      指定卷标

​    -j	 建立文件系统日志（ext3,ext4都是默认带日志的，故不用指定）

​    -t  类型

\2. mkfs 相对比mke2fs简单，但是支持的参数少，不能进行精细化的控制

(如不能指定块大小，不能指定日志，不过一般我们都要手动修改这些参数，故直接用其格式化也可)

mkfs.ext3  /dev/sda3

mkfs.ext4  /dev/sda3

mkfs.vfat  /dev/sda3

####查看分区的文件系统信息：(文件系统已经创建好后)

命令：dumpe2fs  /dev/sda2  非常详细，只在对文件系统在微调的时候用到

JOURNAL日志

带日志的文件系统（ext3、ext4）拥有较强的稳定性，在出现错误时可以进行恢复。

带日志的文件系统会使用一个叫做“两阶段提交”的方式进行磁盘操作

（1）文件系统将准备执行的事务的具体内容写入日志

（2）文件系统进行操作

（3）操作成功后，将事物的具体内容从日志中删除

优点：当事务执行的时候如果出现意外（如断电或者磁盘故障），可以通过查询日志进行恢复操作

缺点：会丧失一定的性能（额外的日志读写操作）

#####E2LABEL

命令：e2label  /dev/sda2  显示sda2的系统标签

​    e2label    /dev/sda2  LINUXCASAT 将sda2的系统标签设置为LINUXCASAT（标签尽量全大写）

#####FSCK 用来检查并修复损坏的文件系统

命令：fsck  /dev/sda2

​    -y  不提示直接进行修复

默认fsck会自动判断文件系统的类型，如果文件系统损坏较为严重，请使用-t参数指定文件系统类型

对于识别为文件的损坏数据（文件系统无记录），fsck会将该文件放入lost+found目录

系统启动时会对磁盘就行fsck操作

注：检查文件系统的时候，文件系统必须是卸载的！

###Linux的文件系统挂载管理

####挂载操作

磁盘或分区创建好文件系统后，需要挂在到一个目录才能够使用

windows或mac系统会进行自动挂载，一旦创建好文件系统后会自动挂载到系统上，windows上称之为C盘、D盘等

Linux需要手工进行挂载操作或者配置系统进行自动挂载

#####MOUNT

linux中我们通过mount命令将格式化好的磁盘或者分区挂载到一个目录上

命令：mount  /dev/sda3（要挂载的分区）	/mnt（挂载点）

​	常用参数：

​	不带参数的mount命令会显示所有已挂载的文件系统

​	-t   指定文件系统的类型

​	-o  指定挂载选项

​	ro , rw     以只读或读写形式挂载，默认是rw

​	sync	代表不使用缓存，而是对所有操作直接写入磁盘（保证数据的准确型，但供电、物理稳定性无法保证的时候使用，内存一断电会丢失数据）

​	async	代表使用缓存，默认是async (读取很快，从内存中读取，系统空闲时才写入硬盘)

​	noatime	代表每次访问文件时不更新文件的访问时间（耗性能）

​	atime	代表每次访问文件时更新文件的访问时间，默认是打开的

​	remount	重新挂载文件系统	

​	mount  -o  remount,ro  /dev/sdb1  /mnt/

#####UMOUNT

命令：umount  文件系统或者挂载点

​    umount  /dev/sda3    ==    unmount    /mnt

如果出现device is busy报错，则表示该文件系统正在被使用，无法卸载

可以用一下命令查看使用文件系统的进程：

fuser   -m  /mnt     --------查看哪些进程在使用fs文件系统，导致不能被卸载

也可以使用命令lsof查看正在被使用的文件

lsof   /mnt     ------查看fs文件系统中有哪些文件被打开了

自动挂载

注：每次都需要先卸载！挂载过程中不允许对文件系统有任何的修改！umount /mnt/

配置文件/etc/fstab用来定义需要自动挂载的文件系统，fstab中每一行代表一个挂载配置

fstab中：每一行代表一个自动挂载配置

格式如下：

/dev/sda3	/mnt	ext4		defaults			0 0

​    文件系统	  	  挂载选项        dump、fsck相关选项

之后reboot系统！

要挂载的设备也可以使用LABEL进行识别，使用LABEL=LINUXCAST取代/dev/sda3

mount  -a命令会挂载所有fstab中睇你的自动挂载项

###Linux下获取帮助

####没有必要记住所有东西！

1. 几乎所有命令都可以使用命令  -h或者--help参数获取使用方法、参数信息等


2. man+命令 //提供更详细的说明

man 1 ls  //man文档分为很多类型1～9

man -k 关键字 //可以用来查询包含该关键字的文档 

3. info 

与man类似，但是提供的信息更加详细深入，以类似网页的形式显示

man 与 info 都可以通过 “/ + 关键字” 的方式进行搜索

​    4.doc

很多程序、命令都带有详细的文档，以TXT、HTML、PDF等方式保存在/usr/share/doc目录中，这些文档

是相当 程序最为详细的文档

###Linux用户基础

####用户、组

当我们使用Linux时，需要以一个用户的身份登入，一个进程也需要以一个用户的身份运行，

用户限制使用者或进程可以使用、不可以使用哪些资源

组用来方便组织管理用户

1）每个用户拥有一个UserID，操作系统实际使用的是用户ID，而非用户名

2）每个 用户属于一个主组，属于一个或多个附属组

3）每个组拥有一个GroupID

4）每个进程以一个用户身份运行，并受该用户可访问的资源限制

5）每个可登陆用户拥有一个指定的shell（命令行或者图形界面）

#####用户

1）用户ID为32位，从0开始，但为了和老式系统兼容，用户ID限制在60000以下

2）用户分为：

###### root用户（ID为0的用户）

系统用户（1～499）没有shell，往往是为某些服务（进程）而创建的，web服务，共享服务，打印服务等

######普通用户（500以上）

3）系统中的文件都有一个所属用户及所属组

4）使用id命令可以显示当前用户的信息

5）使用passwd命令可以修改当前用户密码

######与用户相关的文件

/etc/passwd	保存用户信息

/etc/shadow	保存用户密码（加密的）//root用户才可以查看

/etc/group	保存组信息

######查看登陆的用户

\1. 命令whoami显示当前用户

2.命令who显示有哪些用户已经登陆系统（除了本身）

3.命令w显示有哪些用户已经登陆并且在干什么（除了本身）

######创建用户

命令：useradd zhangsan

参数：-d	       家目录

​    -s	登陆shell

​    -u	userid

​    -g	主组

​    -G	附属组（最多31个，用“，”分隔）

​	useradd命令执行的操作：

​	1）在/etc/passwd中添加用户信息

​	2）如果使用passwd命令创建密码，则将密码加密保存在/etc/shadow中（文件中如果出现!!则表示没有设定密码）

​	3）为用户建立一个新的家目录/home/zhangsan

​	4）将/etc/skel（用户刚建立时的一些初始文件）中的文件复制到用户的家目录中

​	5）建立一个与用户名相同的组，新建用户默认属于这个同名组

也可以通过直接修改 /etc/passwd的方式实现，但不建议 

######修改用户信息：

命令：usermod  参数  username

​    参数：-l  新用户名 （家目录不变）

​	-u  新userid

​	-d  用户家目录位置

​	-g   用户所属主组

​	-G   用户所属附属组

​	-L   锁定用户使其不能登陆

​	-U   解除锁定

######删除用户

命令：userdel    用户名（保留用户的家目录）

 	   userdel    -r    用户名（同时删除家目录）

#####组

几乎所有的操作系统都有组的概念，通过组可以更加方便的归类、管理用户。

一般来讲我们使用部门、职能或地理区域的分类方式来创建使用组

\1. 每个组都有一个组ID

2.组信息保存在/etc/group中

3.每个用户拥有一个主组，同时还可以拥有最多31个附属组

#####创建、修改、删除组

######创建组命令：groupadd  组名

######修改组信息：groupmod -n newname oldname	修改组名

​			groupmod -g newGid	oldGid	   修改组ID

######删   除  组： groupdel	linuxcast

###Linux权限机制

权限是操作系统用来限制对资源访问的机制，权限一般分为读、写、执行。

系统中每个文件都拥有特定的权限、所属用户及所属组，通过这样的机制来限制哪些用户、那些组可以对特定的文件进行什么样的操作

每个进程都是以某个用户的身份运行，所以进程的权限与该用户的权限一样，用户的权限大，该进程拥有的权限就大		

| 权限                   | 对文件的影响     | 对目录的影响           |
| ---------------------- | ---------------- | ---------------------- |
| r（读取）              | 可读取文件内容   | 可列出目录内容         |
| w（写入）              | 可修改文件内容   | 可在目录中创建删除文件 |
| x（执行）/（浏览权限） | 可以作为命令执行 | 可访问目录内容         |

对与读和写权限，文件和文件夹是相同的

对于执行权限：目录必须拥有x权限，否则无法查看其内容！

####UGO

 Linux权限基于UGO模型进行控制：

\1. U代表User， G代表Group， O代表Other

2.每一个文件的权限基于UGO进行设置

3.权限三个一组（rwx），对应UGO分别设置（9个权限位）

4.每一个文件拥有一个所属用户和所属组，对应UG，不属于该文件所属用户或所属组的使用O权限

​	命令：ls -l 可以查看当前目录下文件的详细信息！ 

​	drwxr-xr--  	2  	nash_su	  training  2.8   Oct 1  13:50  linuxcase.net

​	UGO       链接数量    所属用户       所属组	大小     创建/修改时间     文件名

命令：chown  用以改变文件的所属用户

   参数：-R  递归修改目录下所有文件的所属用户

  chown    nash_su    linuxcast.net //将文件linuxcast.net的所属用户改为nash_su

命令：chgrp  用以改变文件的所属组

   参数： -R

   chgrp   nash_su   linuxcast.net

#####修改权限

命令1：chmod  模式   文件 

模式格式：1.u、g、o分别代表用户、组和其他

​    2.a可以代指ugo

​    \3. + 、 - 代表加入或者删除对应权限

​    4.r 、w、x代表三种权限

#####案例：

chmod u+rw	linuxcast.net

chmod g-x	linuxcast.net

chmod go+r    linuxcast.net

chmod  a-x     linuxcast.net

参数 ：-R 递归修改目录内的文件

命令2：以数字的形式修改

chmod 775 linuxcast.net

r=4

w=2

x=1

案例：

| 组        | 用户           |
| --------- | -------------- |
| trainning | nash_su,   bob |
| marketing | alice,   join  |
| manage    | steve,  david  |

要求为各部门、员工建立工作文件夹，要求：

1 所有目录、文件保存在统一的文件夹下

2 每个部门拥有一个独立的文件夹

3 不同部门之间不可访问各自文件夹

4 每个员工在所在部门文件夹下拥有一个所属文件夹

5  同部门不同员工之间可以查看各自文件夹的内容，但不可以修改，用户仅能修改自己的内容

groupadd	training-->  groupadd marketing  --> groupadd manage

useradd -G training nash_su --> useradd -G training bob

useradd -G marketing alice -->  useradd -G marketing join

useradd -G manage steve -->  useradd -G manage david

cd /  -->mkdir linuxcast.net -->  cd linuxcast

mkdir training --> mkdir marketing --> mkdir manage

 

chgrp manage manage --> chgrp market	market  --> chgro  training training 

chmod  o-rx manage -->  chmod o-rx  market  -->  chmod o-rx  training //删除不同用户之间的o权限

cd training -->  mkdir nash_su   --> mkdir bob 

chown nash_su  nash_su  --> chwon bob  bob

chgrp training  nash_su --> chgrp training bob 

默认同组的用户就只有rx权限，故不用更改

chmod o-rx bob  -->  chmod o-rx nash_su 

其他组也对应配置！此处略过

###默认权限

\1. 每一个终端都拥有一个umask属性，来确定新建文件、文件夹的默认权限

umask使用数字权限方式表示：如 022

\2. 目录的默认权限是：777-umask

文件的默认权限是：666-umask

一般来讲，普通用户的默认umask是002，root用户的默认umask是022（说明root用户创建的文件/目录权限更严格）

即：普通用户：新建文件的权限：664

新建目录的权限：775

3 命令umask用来查看设置的umask值

 umask：返回022（后三位，第一位是存特殊权限的）

 umask 022 ：修改umask值

###特殊权限

| 权限   | 对文件的影响                                 | 对目录的影响                                                 |
| ------ | -------------------------------------------- | ------------------------------------------------------------ |
| suid   | 以文件的所属用户身份执行，而非执行文件的用户 | 无                                                           |
| sgid   | 以文件所属组身份执行                         | 在该目录中创建的任意新文件所属组与该目录的所属组相同         |
| sticky | 无                                           | 对用户拥有写入权限的用户仅可以删除其拥有的文件无法删除其他用户所拥有的文件 |

 注：root是不受任何权限限制的！

设置特殊权限

​	1.设置suid：  chmod u+s linuxcast.net //通常是为可执行文件设置的！红色高亮显示！

​	   设置sgid：chmod g+s  linuxcast.net  //所属组的x位被替换成了s，新建文件/目录时，自动继承父	类的所属组！

​	   设置sticky：chmod o+t linuxcast.net //高亮蓝色！保证文件只能被他的所属用户，也就是创建他的用户删除！

​                          其他用户可以查看

2.也可以使用数字方式表示：

   	 -SUID  = 4

  	  -SGID  = 2

 	   -Sticky = 1

​	chmod  4755  linuxcast.net

​	chmod o+t linuxcast.net 