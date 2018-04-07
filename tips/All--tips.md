### All-tips

1. mysql没有全外连接。

2. mahout里面就是用mapreduce写出来的一个个job，因此需要集群正常的情况下安装

3. alter database 数据库名 character set=utf8；//导数据之前修改

4. 修改mysql字符集后，要重启mysql才会生效

5. hadoop-daemon.sh start datanode 单独启动datanode

   ​	start-hbase.sh

    	zkServer.sh start

   ​	zkCli.sh -server master:2181 命令行

   ​	hbase-daemon.sh start regionserver

6. shell概念：

   shell将我们输入的命令与内核进行通信，它的功能是给用户提供了一个操作 系统的接口，因此在shell里面经常调用其它的程序。比如，在shell中输入：man cd  即是调用man程序去显示cd命令的介绍。

   也就是说，只要能操作应用程序的接口都能称为shell，狭义的shell指的是命令行方面的软件，如：bash

   广义的shell包括图形界面软件，因为图形界面也能操作各种应用程序来调用内核工作。

   登陆shell：

   ​                  取得bash需要完整的登陆流程，比如从tty1－tty6进行登陆，需要输入完整的用户名及密码，此时取得的bash称为登陆shell          

   ​                 登陆shell会读取/etc/profile 的配置文件内容，它是整个系统的整体配置。然后登陆shell还会再去读取用户的个人配置文件：~/.bash_profile    ~/.bash_login    ~/.profile  按照顺序读取上面三个文件中的一个。

   非登陆shell：

   ​               取得bash不需要重复的登陆，比如：在X window 登录linux后，再以X的图形界面启动终端机，此时的那个终端接口为非登陆shell。非登陆shell只会读取~/.bashrc配置文件。