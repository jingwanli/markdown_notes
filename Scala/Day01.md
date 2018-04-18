————复习————

Hive暂时没有可替代的产品

重点: Hadoop(hdfs, yarn, mr) ,  Hive, Hbase, kafka

storm有可以替代的产品 (看情况)

hadoop里的mr不重要 —>Spark的wordcount(现在不用mr了,用spark)

Hadoop3本白皮书:(google)

hadoop

mr

bigtable

没有yarn的时候,扩展性收到限制, 可以扩展到千台左右

分布式: 一定是主从架构!(多数情况):中心化(HA用来保证)

kafka:去中心化,没有主从节点(所有元信息都写到zk中了)

2.0才有yarn 主要做资源调度

<<MapReduce与模式>>

java模式: 面试可能会问到

MapReduce: 抽象太单一了.!没有提供一些抽象的方法

mr的不足:

1. 只有2种操作, 复杂的计算实现难度大
2. 作业和作业之间的结果必须要放在hdfs (作业之间没法串起来)
3. 迭代式计算性能比较差
4. 延时高,只适合做批处理. 实时数据处理支持不够

针对mr的不足, 出现了各种改进技术----storm,impala…….

impala:c语言全部重写.闭源

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-11 上午9.31.05.png)

spark解决的是MR的一些缺点:

1. 速度提高.
2. ...

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-11 上午9.33.44.png)

推荐资料: 官方网站

sql , jdbc, sql优化

# Spark

2009年产生

问题的产生—> 网页搜索

解决方案 —> Google论文

工程实现 —> Hadoop

方案优化 —>Spark(优化了MR)

spark仍是使用了Mr的模型, 中间也有shuffle..中间结果也要落地. 但是job之间的结果不会落地.

Spark特点:

1. 快速
2. 容易使用
3. 通用
4. 开放

Spark开发语言: Scala

Spark支持4种语言: java , c/c++, pathon,Scala,R语言(统计), SQL

支持程度: Java=Scala > Python > R

### Scala简介

1. JVM语言: java的优点全部有
2. 简洁优雅: 
3. 类库重用: 可以访问任何Java类库

大数据领域使用scala开发的两个产品: Spark 和  Kafka

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-11 上午10.44.20.png)

IDEA : 使用社区版

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-11 上午10.48.19.png)

spark: 缺省支持hadoop的版本2.7

3台干净的虚拟机: java, hadoop,mysql(先做镜像)

软件装在: /opt/software :软件包

​		/opt/modules:所有的软件

IDEA—>可以写java程序 如果要写scala, 装插件

tar -zxvf xxx.tar.gz -C xxx



### IDEA安装测试

验证是否path生效:

直接的方法判定path有没有写错: echo $PATH

export PATH=…xxx:xxx:xxx:.

加一个点,就可以./ 不用写成 ./idea.sh 而写成idea.sh也可以

## Scala基础

#### HelloWorld!

object 是一个关键字.

object HelloWorld{

​	def main(args:Array[String]){

​	println("Hello World!") //分号可以有, 可以没有

​	}

}

> 注意:两条语句写在一行,一定要用分号!

#### 运行方式

scala语法形式非常灵活.

少于3行代码可以写在命令行里

idea启动非常慢

3种模式执行Scala程序:

1. 脚本模式 
2. 交互模式 vi a.scala; 写程序
3. IDE

#### 命名

区分大小写!

程序文件名和可以不完全匹配. 建议写成一样的

new的时候 —— 选择object ! (main方法一定要写到object里! )

$字符是Scala的保留关键字!

标识符以字母或者下划线开头,后面可以有字母,数字或者下划线.

换行字符: 单行上的分号可以省略.

#### 操作符

操作符在scala里 都是方法!

val i = 1 + 2

val i = 1.+(2) //使用方法的表示形式

无 ++  和  - - 可以有+= 

#### 关键字

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-11 下午2.29.27.png)

val : 不变性 如果定义成val , 值就确定了,声明时需要进行初始化.

var: 可变的 ,声明的时候需要初始化

鼓励使用: val

不需要给出值或变量的类型,scala可以做自动类型判断, 必要的时候可以指定类型.

scala可以做自动类型推断

val i:Double = 2 (强制声明)

可以给多个值和变量一起声明:

var i,j,k = 1

var a,b,c:String = null

#### 常用类型

在java的基本数据类型, 在scala中都是类

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-11 下午2.39.29.png)

与java的精度都一样

#### 字面量

## 控制结构和函数

#### 条件表达式

if else表达式与java一样 ,但是 Scala表达式有值!

val z  = if(x<y) 100 else 200

val z = if(x<y) x

z: AnyVal= ( )

val z = if (x<y ) 1 else "100"

z: Any = 100 //找上面两者的共同的父类型

所有的基础类型都继承自: AnyVal

所有的引用类型都继承自: AnyRef

都继承于 Any!

var y= 0

val x =10

val z = if(x>0) y=1 else y = -1 (错误写法!)

z:Unit = ()  //  赋值语句的返回值为空!

val y = if(x>0) 1 else -1 //正确写法!

> scala 没有switch语句

如果想分开写在两行: if(x>0) 1

​				    else if(x==0) 0 else -1

​				   :paster /ctrl+d //写法

#### 输出语句

1. 块表达式 { }

   注意: 快表达式可以包含一些列表达式, 块中最后一个表达式的值就是块的值 (不需要用return)

2. 输出语句

    print println

   printf("hello,%s! you are  %d years old!", "Jack

   ",10);

   %s 代表字符串   %d 代表数字

   字符串插值器: println(s"name = $name")

   ​		结果: name = Jack

   ​			println(f"\$name  is $height%2.2f meters tall")

   println(raw"1\n2")  //是什么就写什么 原样输出

   println("""1\n2""") //原样输出

####for 循环

for(i<-1 to 10) println(i) //1~10

for(i<-1 until 10) println(i) //1~9

for(i<-1 to 5; j<-1 to 5; if(i!=j)) println(10*i + j)

val a = for(…同上) yield (10*i+ j)

for循环中的yield会把当前的元素记下来!(生成数据集)

#### while do循环

注: scala中尽量少用循环!

#### Range对象

数值序列可以采用range ,支持多种数值序列,

包括: int ,  

Range(1,5) 或 1 until 5 //包前不包后

1 to 5 //全包

Range(1,50 10) //10代表步长 

1 to 10 by 2 //同上

(1 to10 by  2).toArray //快速得到一个数组

(1 to10 by  2).toList

('A' to 'Z')

#### 函数

- 只要函数不是递归的, 不需要指定返回类型(类型可以自动推断)
- Scala可以进行自动类型推断

def add(x:Int, y:Int) : Int = {x + y} // 第三个int是返回值,可以不写(但是写上之后可以保证得到你的原意!)

- 递归 必须指定返回值! 

- 变长参数

  def sum(args : Int *) = {

  ​	var result = 0

  ​	for( iterm <- items) result += item

  ​	result

  }  //必须加=  , 返回值为Unit! 

  过程, 不返回值的函数

  sum(1,2,3)

- 懒值 :初始化将被推迟,(用的时候才初始化!!!)

  import scala.io.Source

  lazy val words = fromFile("a.scala").mkString

  可以写为: lazy val words = scala.io.Source.fromFile("a.scala").mkString

  如果调用不存在的文件, 定义写lazy, 不会报错(延迟执行),取消时会报错!

  懒值是开发懒数据结构的基础

#### 异常

- Scala 没有"受检"异常, 所有异常都当做"不受检异常"(或者成为运行时异常)

---

小结:

scala类型的声明 要在变量后面  :类型 ,可以自动匹配类型,所以可以不指定!

