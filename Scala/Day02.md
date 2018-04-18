———晨测——— 

hadoop4个模块: 

1. Common:提供了一些公共的服务,提供一些工具类(如RPC)
2. HDFS
3. YARN
4. MapReduce

spark特点: 快速, 通用, 易用, 开放

jdk1.8—> scala —> idea —> 插件 (安装顺序) idea支持jdk1.8版本 

版本号: scala:2.11 

​	      jdk:1.8

如何确定环境变量是否生效? echo $PATH

3种形式执行scala程序: 命令行, OS命令行(scalac, scala)(脚本), IDE

Scala程序从main()方法开始, main方法应该写在object中

if语句有返回值! (可能为空)

for语句的条件中可以有if语句!

函数: 有返回值的叫函数, 没有返回值的叫过程!

---

## 数组

数组是一种不可变的, 可索引的, 元素具有相同类型的数据集合.

### 定义

##### 定长数组 

#####注意: scala取数组下标用小括号表示!

val a = new Array\[Int](10)

val a = Array(1,2,3,4,5)

(1 to 100).toArray

##### 变长数组

必须导包!

import scala.collection.mutable.ArrayBuffer

object Test{

​	def main(args: Array[String]):Unit={

​	val b = (1 to 10).toArray

​	val a = ArrayBuffer\[Int]() //指定类型

​	a+=1

​	a+=(2,3,4,5) // 可以加多个元素

​        a++=b 

​	a-=10

​	a- -= Array(1,2,3,4,5)

​	//提供了方法

​	a.trimEnd(5) //减掉尾部的5个元素

​	a.insert(2,10) //插入下标2

​        a.insert(2,10,100,1000)



​	println(a) //打印buffer的时候打印值!

​	println(b) //打印数组的时候是打印地址!

​	println(b.toBuffer)

​	}

}

建议: 在尾部进行增删操作!(影响效率)

object Test{

def main(args:Array[String]):Unit

 = {

}

}

##### 多维数组

### 数组的操作

##### ==遍历数组==

var a = (0 to 100 by 5).toArray

1. for(i<- 0 to a.length -1 ) println(a(i)) //下标用小括号访问!

2. for(i<- 0 until a.length) println(a(i))

3. for(i<- 0 until (a.size,2))  print(a(i)) //有步长


   - for(i<-0 until a.size){

   ​	 a(i) = a(i)*2

   ​        }

   ​	println(a.toBuffer)

   - val b = for(i<-0 until a.size) yield {

     ​    a(i)*2 

      //错误写法: a(i) = a(i) *2——赋值语句返回值为空!!

     } 

     println(b.toBuffer)

4. for (i<- a.indices) println(a(i))

//前4种都是类似java写法,for里面维护了下标i

==Tip:==

yield的用法: (一般在for 中使用!)

for 循环中的 yield 会把当前的元素记下来，保存在集合中，循环结束后将返回该集合。Scala 中 for 循环是有返回值的。如果被循环的是 Map，返回的就是  Map，被循环的是 List，返回的就是 List，以此类推。

5. for (item<-a) … //类似增强的for循环

6. 推荐: val b = a.map((x:Int)=>x *2)

   ​     	 println(b.toBuffer)

   ​	 val b = a.map(word=>word*2)

   ​	println(a.toBuffer) //打印原来的(不变性)

7. a.foreach(println _)

	###	常用方法

- 将Range转化为数组: val b = (1 to 10).toArray


- 过滤器

  - var c = a.filter(x=>x>50) 
  -  filterNot

- count(必须写条件!!!)

- println(a.reverse.sorted.toBuffer) //注意要toBuffer否则打印地址!

- println(a.reduce(\_+_))

- println(a.indices) //返回的是一个Range(0,1,2,3,4,…) 运算后显示方法有点歧义,定义的时候要定义成起始,中止,步长

- println(a.count(x=>x>5)) 

- println(a.mkString("$"))//\$为分隔符

  println(a.mkString("[",";","]"))

- println(a.take(4).toBuffer,a.takeRight(4).toBuffer) //取出部分数组

  println(a.takeWhile(_<5)).toBuffer 

  ==注意== toBuffer后面没有括号!

  括号=传参,没有参数就不写括号!

- println(a.drop(4).toBuffer) //删除前4个

- val (b, c) = a.span(_<4)

  println(b.toBuffer)

  println(c.toBuffer)

- (1 to 10).partition(_>4) //一个是满足条件的,另外的是不满足的

- 填充 (1 to10).toArray.padTo(20,4 )// 20表示填充后总长度, 4为填充的值, ==range同样适用!==

- val c = (1 to 10).toArray.slice(10, 33) //切片

  从哪里开始和到哪里结束(包前不包后!!!)

- val c = (1 to 10).toArray.sliding(5) //返回一个迭代器,5代表每个小单元的个数 如果是2个参数, 第二个是步长

  c.foreach(x=>println(x.toBuffer)) //迭代器只能迭代一次!!

  ==如果再次输入上面的语句, 则没有输出结果!!!!==

  输出结果:

  ![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-12 下午2.18.14.png)

- val c = (1 to 10).toArray

  - c(5) = 1000 //修改数组的值 可以为字符->转化为asc码,字符串不可以!!


  - c.update(5,100000)

- val a = (1 to 10).toArray

  val  b = (10 to 100 by 10).toArray

  a.zip(b) //拉链操作 

  案例:

  ​    val a = Array(10,20)

  ​    val b = Array(2,3)

  ​    val c = a.zip(b) //相同的key组成一组,组的长度取决于数组的长度, 组的维度取决于变量的个数

  ​     c(0)// (10,2) //元组

  ​     c(0)._1 //10

  ​     c(0)._2 //2

  ​     c.map(x=>(x._1-x_2))

  ​    math.sqrt( c.map(x=>(x._1-x_2)).map(x=>x*x).sum)

- val z4 = z1.

- val a = (1 to 10).toArray

  a.head //第一个元素

  ​a.tail //除了第一个元素剩下的元素

  a.last

  a.init //除了最后一个元素

  a.init.tail //也可以切片来做 (除了第一个和最后一个的元素)


说明: 上述的很多方法不仅仅对Array适用, 一般情况下对其他集合类型同样适用

## Map和Tuple

### Map

分为可变 和 不可变 map

key不可以重复

#####构造方式:

- val a = Map(("a","1"),("b","2"))
- val b= Map(("a"->"1"),("b"->"2"))

##### 常用方法

- a.keys —返回值为类似迭代器set

- a.values — 返回值为类似迭代器MapLike

- a.getOrElse("aa", "Error!")

- a.get("a") //一定存在的key

- a("a") //一定存在的key可以这样写

- import scala.collection.mutable.Map

  val b = Map((xxxxxxxx));

  b("a") = "111" //改变值

  val c =  b + (("d","4"))// 必须是双扩号! 一个括号为tuple

  或者写为:val c =  b + ("d"->"4")

  减的时候指定key就可以了

##### Map增删(针对可变Map)

==增加元素/删除元素==

- b("d") = "a" //b为可变Map
- b += ("e"->1, "v"->2) ———— b-="a"
- val c = b + ("a"->10, "b"->20)


- 迭代:

  val a = Map(("a","1"),("b","2"))

  - c.map(println()) //一般map有返回值! 没有返回值的时候用foreach!
  - c.foreach(println())
  - a.map(x=>(x._2,x.\_1)) //相当于交换了k, v,x代表键值对, x.\_2 表示取value 

- 拉链操作 (可以对数组, 也可以对map!!!)

  val a = Array(1,2,3)

  val b = Array("a","b","c")

  val c = a.zip(b)

  val c = a.zip(b).toMap //这步要注意!!!

  ​

### Tuple

元组可以包含不同的元素!!!!不会去找父类!!! 对偶(Map)是元组的最简单形态!

访问 a._1 //元组的下标从1开始

val a = (1, 1.2, "ad", 'd')

val(a1,a2,a3,a4),a5 = a //相当于赋值!!

结果为:

a1=1 a2=1.2 a3="ad" a4='d'

a5=(1,1.2,"ad",'d')



## π的算法 蒙特卡洛算法

import scala.util.Random

val rand = new Random()

rand.nextFloat

(1 to  1000).foreach(println(rand.nextFloat))

```scala
package scalademos
import scala.math._
import scala.util.Random

object PiCaculate {

  def dist(x1:Float,x2:Float):Double = {

    val arr = Array(x1,x2)
    var distance = math.sqrt((for(x<-arr) yield x*x).sum)
    distance
  }

  def main(args:Array[String]){
    var rand = new Random()
    var total:Int = 0
    for(i<- 1 to 100000) {
      var a1 = rand.nextFloat()
      var a2 = rand.nextFloat()
      if (dist(a1, a2) <= 1) total += 1
    }
    print(total*4.0f/100000)
  }
}

```



## 总结

>  遍历数组: 5种

1. for(i<-1 until arr.size)
2. for(i<- arr.indices)
3. for(elem<-arr)
4. arr.map
5. arr.foreach(..)

数组定长和变长转换:

toArray  toBuffer

>  map的迭代: (map没有下标!)

for , map , foreach

> Tuple

==Tuple下标从1开始!==