—晨测— 

"Hello".reverse(0) //反转后取第一个值

val(first, second, _) = t // _占位符. 表示只关心前两个值

---

## 面向对象

### 类

Scala在架构层面提倡的方法:

1. 小处用函数式编程
2. 大处用面向对象编程

- 声明一个类: class
- 单例对象 : object声明(类似为static)

final: 不能被继承

abstract : 不能实例化

scala默认可见性: public( 不能写public! )

==class Person(var name:String, var age:Int)==

class Helloworld{

​	private var name = "Jack" //不指定默认值时,需要写类型!(否则没办法自动推测)

​	def sayHello( ):Unit={ //括号可以去掉

​        	println("Hello," + name)

​	}

​	def getName = name

}

object HelloWorld{ //main方法必须在object里!  名字随便定义

​	def main(args:Array[String]):Unit ={

​		val hw = new HelloWorld// 也可以加括号

​		hw.sayHello() //括号可以去掉,最好与上面保持一致!

​		print(hw.getName)

​	}

}

统一访问接口: 与java形式不同. 

class Person{

​	var name = "Jack"

​	var age  = 10

​	def sayHello():Unit={

​		println("my name is :" + name)

​	}

}

object Test{

​	def main(args:Array[String]):Unit{

​	val p1 = new Person

​	p1.sayHello()

​	p1.name = "Tom" // 因为上面类型是var

​	println(p1.name)

​	}

}

##### get set方法

private[this]  var name = "Jack" //只能在此访问! 别的类也不行

def getOtherName(other: Person): Unit={

​	println(other.name) //报错!!!

}

class Student{

​	private var sname = "Jack"

​	def name = "your name is " + sname

​	def name_ = (newName:String):Unit=		{

​	sname = newName

​	}

}

//使用java那一套的步骤

import scala.beans.BeanProperty //先导包

class Student{

@BeanProperty //加注解 	

var name:String = _

}

object Test{

​	def ….main….

​	val s1 = new Student()

​        s1.set…..

}

//object的类名 和 class的类名是不同的

如果相同, 可能上面的实验可以相互访问(不飘红)

####主构造器

Scala的主构造器是整个类体，==需要在类名称后面罗列出构造器所需的所有参数，==这些参数被编译成字段，字段的值就是创建对象时传入的参数的值；

主构造器会执行类定义中的所有语句；

如果类名之后没有参数，则该类具备一个无参主构造器，这个构造器仅仅是执行类体中的所有语句。

class Counter(val name:String, val 	mode:Int){

​	println("starting…….")

​	private var value = 0

​	def increament(step:Int)={

​		value+step

​	}

​	def current():Int = {

​		value

​	}

​	def info:Unit = {

​		printf("name" + name + "and mode is" + mode )

​	}

}

main:

val c1 = new Counter("Counter1",10)

####辅助构造器

主构造器没有名字(是整个类体)

辅助构造器是有名字的(必须为this,且第一句话必须调用主构造器!)

class Person{

​	private var name = ""

​	private var age = 0

​	def this(sname:String){

​		this()

​		this.name = sname

​	}

​	def this(sname:String, sage:Int){

​		this(sname)

​		this.age = sage

​	}

​	def info = println("name is " + name + "age is " + age)

}

main:

val p1 = new Person

p1.info

val p2 = new Person("Jack")

p2.info

val p3 = new Person("tom",10)

p3.info

###继承

##### 单例对象

scala并没有提供Java那样的静态方法或者静态字段

object与class非常类似, ==区别: object不能跟参数列表==

object是scala中的一个关键字,对象本质上可以拥有类的所有特性，除了不能提供构造器参数

对于任何在Java中会只用单例对象的地方，在scala中都可以用object实现：

- 作为存放工具函数或常量的地方

- 高效地共享单个不可变实例

- 在java中main是静态方法，用static修饰。

  在scala中没有静态方法的概念，而是用objcet关键字实现java的静态方法和静态字段，所以main方法定义在object中：

##### 伴生对象

当单例对象与某个类具有相同的名称时，它被称为这个类的“伴生对象”

- 类和它的半生对象必须存在于同一个文件中,而且可以相互访问私有成员(字段和方法)
- 伴生对象中定义的newPersonId()实际上就实现了Java中静态（static）方法的功能；
- Scala源代码编译后都会变成JVM字节码。实际上，源代码编译后，Scala里面的class和object在Java层面都会被合二为一，class里面的成员成了实例成员，object成员成了static成员。

##### 应用程序对象

不推荐使用

可以不写main方法, extends App(相当于继承一个接口)

spark里不支持!

```scala
object Hello extends App {
  if (args.length > 0)			// 可以通过args传递命令行参数
    println("Hello, " + args(0))
  else
    println("Hello World!")
}
```

##### apply&update方法 — 可以解释很多的语法现象

通常我们都是在伴生对象, 调用apply方法

由于Scala中的Array对象定义了apply方法，因此，我们就可以采用如下方式初始化一个数组： val myArray = Array(1,2,3,4,5,6)

也就是说，不需要new关键字，不用构造器，直接给对象传递参数，Scala就会转换成对apply方法的调用，这里实际上是调用了Array类的伴生对象Array的apply方法，完成了数组的初始化。

尽管在class中和object中都可以定义apply方法，二者还是有区别的：

- 只有class实例化了，才能调用伴生类中的apply方法；
- 没有实例化，调用的是伴生对象中的apply方法

##### Scala中的继承

Scala中的继承与Java有着显著的不同：

- 重写一个非抽象方法必须使用override修饰符；
- 只有主构造器可以调用超类的主构造器；
- 在子类中重写超类的抽象方法时，不需要使用override关键字；
- 可以重写超类中的字段。
- Scala和Java一样，不允许类从多个超类继承。

##### 抽象类

- 定义一个抽象类，需要使用关键字abstract

- 定义一个抽象类的抽象方法，不需要关键字abstract，只要把方法体空着，不写方法体就可以；

- 抽象类中定义的字段，只要没有给出初始化值，就表示是一个抽象字段；

  只有抽象字段可以不初始化! 普通字段声明时必须初始化!

但是，抽象字段必须要声明类型，比如：valcarBrand: String，就把carBrand声明为字符串类型。这个时候，不能省略类型，否则编译会报错。

###特质(接口)

==trait可以同时拥有抽象方法和具体方法==

Trait中没有方法体的方法, 默认就是抽象方法

定义方式:

```scala
trait CarId{
	var id:Int
    def currentId():Int
}
```

##### 将特质混入类中(mix) :使用extends 或 with

即: 类实现了此接口 (多实现用with 接口..with ...)

使用extends关键字混入CarId特质，后面可以反复使用with关键字混入更多特质z 

接口也可以有实现的方法

设计时: 尽量使用简单的接口. 多复用

## 文件操作

### 读文件

```scala
import scala.io.Source
object FileDemo{
    def main.....
    
     val file = Source.formFile("...")
     val lines =  file.getLines().toBuffer //小文件的话可以转(占内存大),大文件用迭代器
     val lines =  file.getLines()//返回的是迭代器
     lines.foreach(println_)
    println(lines.size)//返回行数
}
```

```scala
package scalademos
import scala.io.Source

object IoDemo {
  def main(args: Array[String]): Unit = {
      var source = Source.fromFile("/home/xdl/test") 
      方式1: val lines = source.mkString //将文件读成字符串(保持原样)
      		println(lines)
      方式2:var lines=source.getLines()//返回一个行迭代器
      		lines.foreach(println _)
      		或 for(line<-lines) println(line)
      方式3: var lines = source.getLines().toArray
      		lines.foreach(println _) 
      方式4: var lines = source.getLines().toBuffer
    		lines.foreach(println _) 
      		或 for(line<-lines) println(line)
      		或 println(lines) //打出来是一行字符串(去掉换行)
  }

}


```

###写文件(接上)

```scala
package scalademos
import scala.io.Source
import  java.io.PrintWriter

object IoDemo {
  def main(args: Array[String]): Unit = {
      var source = Source.fromFile("/home/xdl/test")
      var lines = source.mkString
     // println(lines)
     // var lines = source.getLines().toBuffer
    //  lines.foreach(println _)
     // for(line<-lines) println(line)
   //   println(lines)
    val out = new PrintWriter("/home/xdl/output.log")
    for (i<-1 to 10) out.print(i+ "+")
    out.println()
    out.println(lines)
    out.close()
  }

} 

```

### 正则表达式

需要引入: scala.util.matching.Regex

"""…""" :表示原样的字符

```scala
val regex = """([0-9]+)([a-z]+)""".r
val numPattern = "[0-9]+".r
val numberPattern = """\s+[0-9]+\s+""".r
.r()方法: 将字符串转换为正则表达式
模式匹配1: 
	//findAllIn()方法返回遍历所有匹配项的迭代器
	for(matchString<- numPattern.findAllIn("99345 Scala,22298 Spark"))
	println(matchString)
模式匹配2: //找到首个匹配项
	println(numberPattern.findFirstIn("99ss java, 222 spark,333 hadoop"))
模式匹配3://数字和字母的组合的表达式
//分组可以让我们方便地获取正则表达式的子表达式。在想要提取的子表达式两侧加上小括号
	val numitemPattern = """([0-9]+) ([a-z]+)""".r //中间有空格! 不能加大括号括起来!
	val numitemPattern(num, item) = "99 hadoop"
	结果: num:String = 99
		 item:String = hadoop
模式匹配4://数字和字母的组合正则表达式
    val numitemPattern="""([0-9]+) ([a-z]+)""".r
    val line="93459 spark"
    line match{
      case numitemPattern(num,blog)=> println(num+"\t"+blog)
      case _=>println("hahaha...") //_相当于default
    }
模式匹配5:检索字符串的开始部分是否能匹配，使用findPrefixOf
	val pattern = """\s+[0-9]+\s+""".r
	val numPattern = "[0-9]+".r
	pattern.findPrefixOf("99 bottles, 98 bottles")
	numPattern.findPrefixOf("99 bottles, 98 bottles")
模式匹配6:替换首个匹配项或者全部替换
	val numPattern = "[0-9]+".r
	numPattern.replaceFirstIn("99 bottles, 98 bottles","XX")
	numPattern.replaceAllIn("99 bottles, 98 bottles","XX")

```

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-13 下午2.38.33.png)

字符串中: \\ \ 代表 一个 \

####捕获组

/(a) (b) (c)/

百度搜: 正则表达式解析器

```java
package scalademos

import java.util.regex.Pattern

object RegexDemo {
  def main(args: Array[String]): Unit = {
      val str ="""10.1.1.95 - - [18/Mar/2005:12:21:42 +0800] "GET /stats/awstats.pl?config=e800 HTTP/1.1" 200 899 "http://10.1.1.1/pv/" "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; Maxthon)""""
      val p1 = """(\S+) .+ \[(.+)\] "(.+)" (\d+ \d+) "(.+)" "(.+)"""
      val m1 = Pattern.compile(p1)
      val k1 = m1.matcher(str)
      println(k1.matches())
      println(k1.group(1))
      println(k1.group(2))
      println(k1.group(3))
      println(k1.group(4))
      println(k1.group(5))
      println(k1.group(6))
  }
}
```



---

作业点评:

object Array01{

​	def main (args:Array[String]):Unit={

​		//得到一系列的点 

​		//计算与远点的距离

​		//计算pi

​		val rand = new Random()

​		def calcPi(N:Int=10000):Double={

​	 	val points = for(i<-1 to N)   yield (rand.nextDouble(), rand.nextDouble())

​		val n = points.map(point=>sqrt(pow(point._1,2) + pow(point._2,2))).count(dist=>dist<=1)

​		4*n.toDouble/N

​	}

​	}

}

//需要修改!!!因为内存可能溢出(可以先算 算完就和计算的部分没有关系了)

修改方法: 边生成边计算

val result = for(i<-1 to N) yield{

val(x,y) = (rand.nextDouble(), rand.nextDouble())

val dist =  sqrt(pow(x,2) + pow(y,2))

if(dist<=1) 1 else 0

}

4*result.sum.toDouble/N

//Linux下检查分配给Scala的内存

开启scala

jps —> 拿到进程号

ps -ef | grep 2748

xmx 256m :给堆内存分配了256m的最大内存

scala启动时可以跟参数:  scala -help

将内存值调大

--------

###回顾

类的声明: 可以跟参数列表

主构造器: 整个类体

辅助构造器: 名称为this 必须以调用其他的辅助构造器或主构造器开始.

应用程序对象: object可以继承接口, 可以省略main方法, 可以传递程序运行时的参数



---

拓展:

> 关于private 和 private[this]

```scala
package scalademos

class Person{
  var varName = "Spark"
  private var priName = "GreenPlum"
  private[this] var prithisName = "Oracle"

  def getName(): Unit ={
    prithisName = "ss"
    println(varName + "," + priName + "," + prithisName)
  }

  def getInsName(p:Person): Unit ={
  //  println(p.varName + "," + p.priName +"," p.prithisName)
    varName = "s1"
    p.varName = varName
    priName = "g1"
    p.priName = priName
    prithisName = "o1"
   // p.prithisName = prithisName
    println(p.varName + " " + p.priName )

  }
}

object Test{
  def main(args: Array[String]): Unit = {
    var p = new Person;
    p.varName = "zhangsan"
  //  p.priName = "lisi"
  //  p.prithisName = "wangwu"
    p.getName()
    p.getInsName(p)

  }
}

```

> 自定义get set方法

```scala
package scalademos

class Student{

  private var sname = "bell"
  def name  = "your name is " + sname
  def name_(newValue:String):String={
    sname = newValue
    sname
  }
}

object Demo1 {
  def main(args: Array[String]): Unit = {
    val s1 = new Student
    println(s1.name)
    println(s1.name_("ww"))
  }
}
```

- 如果不希望get和set方法暴露，但是希望其他类可以修改某一个字段的值，可以将字段定义成private，然后提供一个方法修改该字段如果不希望get和set方法暴露，但是希望其他类可以修改某一个字段的值，可以将字段定义成private，然后提供一个方法修改该字段

- 如果不希望当前 private字段被其他类的方法调用时访问，可以使用private[this]。

  当前类Student，定义一个private字段 sname：

  如果希望两个实例的name进行比较，private可以实现

  如果希望不同实例间不能访问该字段，则需要使用private[this]

> scala 生成java风格的get,set方法

```scala
package scalademos

import scala.beans.BeanProperty

class Student1{
  @BeanProperty
  var name : String= _
}
object Demo2 {
  def main(args: Array[String]): Unit = {
    val s = new Student1
    s.setName("nicky")
    print(s.getName)
  }
}
```

