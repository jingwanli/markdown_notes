## System类

系统类. 主要用于获取系统的属性数据

System类无构造方法, 不能被实例化,方法都是静态方法!

### 常用方法

- arraycopy(Object src, int srcPos, Object dest, int destPos, int length)

- currentTimeMillis( ) 获取当前的系统时间

- exit(int status) 退出jvm. 0:正常退出  非0:异常退出jvm

  无论传0还是1,jvm都退出! 后面的代码都不执行了!

  注: 对于用户而言没有任何区别.

  ​      对于操作系统而言, 是有意义的.(异常退出会发送报告,利于升级)

  对于普通的开发者而言:  try中退出一般用0退出

  ​					catch中退出,一般用非0退出 

- gc( ) 建议jvm赶快启动垃圾回收器回收垃圾

  (一个cpu同一个时间片只能运行一个程序, 所以gc只能说是建议启动,而不能说是马上启动!)

  Object中有一个finalize( )方法: 如果一个对象被垃圾回收器 回收的时候,会先调用对象的finalize( )方法! 

  有时会来不及启动gc就退出jvm虚拟机了!

- getenv(String name) : 根据环境变量的名字获取环境变量.(获取的是路径)

- getProperties( ): 获取系统的所有属性. 返回properties对象.

  - 调用Properties类中的方法可以查看系统属性

    - list(PrintStream out):

      properties.list( System.out )

- getProperty(String s) 根据系统的属性名获取对应的属性值

## RunTime

该类主要代表了应用程序运行的环境.  

无构造方法,

- getRuntime( ) 返回当前应用程序的运行环境对象, 返回值: Runtime类对象

- exec(String command) 根据指定的路径执行对应的可执行文件 

  ​	返回值: 一个process.

  ​	process方法: destroy( )

- freeMemory( ) 返回jvm空闲的内存. 以字节为单位.

- maxMemory( ) 返回java虚拟机试图使用的最大内存量

  - java虚拟机默认管理的内存是64M
  - 从jdk1.7.0开始,虚拟机管理的内存大概是10几~20多M. 不够的话,弹性增长. 

- totalMemory( ) 返回java虚拟机的内存总量

​	



