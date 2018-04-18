```scala
wordcount:

package cn.it.cn.it.spark.project


import scala.io.Source

object WordCountTest {
  def main(args: Array[String]): Unit = {
    val fileName = "/home/zkpk/scalawordcount.txt"
    val source = Source.fromFile(fileName)
    val lines = source.getLines.toList
   // println("lines is count :" + lines)  //lines is count :List(hello lili lichen yangmi, spark hive storm hello, hello scala spark, hadoop scala hello hadoop storm, hadoop hello)
    val wc = lines.flatMap(_.split(" ")).map((_,1)).groupBy(_._1).map(t=>(t._1,t._2.size)).toList.sortBy(_._2).reverse
    println("wc:" + wc)

  }
}
```

###WordCount

```scala
import scala.io.Source
var lineArr = scala.io.Source.fromFile("/home/xdl/data/scala/a.log").getlines.toArray
lineArr.flatMap(_.split("\\s")).groupBy(x=>x).map(x=>(x._1,x._2.size))
//Array(w1,w2,w2,.....)  
//ws->List(ws,ws,ws)

```

### 晨测

scala没有提供写文件的方法, 用的java的

Option类型的两种状态分别是Some和None

遍历:

```scala
val names = List("Petter","Paul","Mary")
//方式1(map会生成一个新的集合)
names.map(_.toUpperCase)
//方式2 yield也会生成一个新的集合
for(name<-names) yield name.toUpperCase
//方式3
for(name<-names) yield
	for(c<-name) yield c.toUpper
 结果: List[String] = List(PETTER, PAUL, MARY)
//错误写法!!
for(name<-names; c<-name) yield c.toUpperCase
结果:List[Char] = List(P, E, T, T, E, R, P, A, U, L, M, A, R, Y)
```



---

## 样例类与模式匹配

###样例类(case class & case object)

样例类是scala中特殊的类:

当声明样例类时，如下事情会自动发生：

- 构造器中每一个参数都成为val。除非它被显示的声明为var（不建议这样做）
- 提供apply方法。不用new关键字就能够构造出相应的对象
- 提供unapply方法。让模式匹配可以工作
- 将生成toString、equals、hashCode和copy方法。除非你显示的给出这些方法的定义
- 继承了Product和Serializable（implementsProduct, Serializable），也就是说已经序列化和可以应用Product的方法 ( 序列化对象后, 才有io )

case class和其他类型完全一样，可以添加方法和字段，扩展它们；

case class 最大的用处是用于模式匹配, 另外也用来快速的定义简单的类. 

在spark源码中，组件之间传递的消息对象都是case class 或case object

```scala
import scala.math.{pow, sqrt}

case class Point(x : Int, y : Int) {
  def dist(p:Point): Double = {
    sqrt(pow(x-p.x, 2) + pow(y-p.y, 2))
  }
}

val p1 = Point(1, 1)
val p2 = Point(0, 0)
 
println(p1.dist(p2))

p1.toString			// case class提供的toString方法，返回值很漂亮
```

### 模式匹配

与java的switch区别:

- scala有返回值
- 没有break. 完成匹配就结束!(不需要break)
- 如果没有任何一个模式匹配上,会抛出异常(MatchError)

##### 常量模式 & 通用模式

```scala
//常量模式 
//1. 常量匹配
val colNum = 4
val colorStr = colorNum match{
    case 1 =>"red"
    case 2 =>"green"
    case 3 =>"blue"
    case _ =>"no match"
}
println(colorStr)
```

```scala
//通用模式
//2.对表达式的类型进行匹配
val list = List(9,12.3,"Spark","Hadoop",'Hello)
for(elem<-list){
    val str = elem match{
        case i:Int => i + "is an int value"
        case d:Double => d + "is an double value"
        case s:String => s + "is an String"
        case _ => "no match"
    }
    println(str)
}

```

##### 守卫语句

```scala
//3. 可以在模式匹配中添加一些必要的处理逻辑
for(elem<-List(1,2,3,4)){
    elem match{
        case _ if(elem%2==0)=>println(elem + "is even") //在 =>左侧的是匹配语句,只有匹配上才是匹配上
        case _ => println(elem + "is odd")
    }
}
```

##### case class中的模式匹配

```scala
//4. case class中的模式匹配. 匹配的是类
case class SubmitTask(id:String,name:String)
case class HeartBeat(time:Long)
case object CheckTimeOutTask

var arr = Array(CheckTimeOutTask,HeartBeat(12333),SubmitTask("0001", "task-0001"))
for(elem<-arr){
    elem match{
	case SubmitTask(id, name) => println(s"SubmitTask, id is $id, name is $name") //可以分离出类中的元素(消息的具体内容,可以获取到)
    case HeartBeat(time) => println(s"HeartBeat, time is $time")
    case CheckTimeOutTask => println(s"CheckTimeOutTask")
    }
}
```

##### for表达式中的模式匹配

```scala
//5. for模式匹配
val abbr = Map(("VIP", "very important people"), ("CTO", "chief technology officer"), ("HRD", "human resource director"))
for((k,v)<-abbr) printf("%s:%s\n",k,v)
```

### Option类型

在java中,null是一个关键字

在scala中, none是一个空对象

> 应用场景

Scala中预计到变量或者函数返回值可能不会引用任何值的时候，使用Option类型。

Option类包含一个子类Some，当存在可以被引用的值的时候，就可以使用Some来包含这个值，如Some("Hadoop")。

```scala
val books =Map(("hadoop", 58.8), ("spark", 68.8), ("hive",38.8))
books.get("hadoop")
取值方式:
books("hadoop") 结果: Double=58.8
books.get("hadoop") 结果: 58.8
books("kafka"): //抛异常
books.get("kafka") 结果: none
//get的方法比直接取多包装了一层
//推荐使用get
books("hbase")
val sales = books.get("hbase")
sales
sales.getOrElse("No Such Book"):
//找到返回找到的(类型是Any)
//没找到的时候,返回括号中的
//Option类型提供了getOrElse方法，这个方法在这个Option是Some的实例时返回对应的值，而在是None的实例时返回传入的参数。
```

Option[T]实际上就是一个容器，可以把它看做是一个集合，只不过这个集合中要么只包含一个元素（被包装在Some中返回），要么就不存在元素（返回None）；

既然是一个集合，当然可以对它使用map、foreach或者filter等方法；

```scala
books.get("hadoop").foreach(println _) //58.8
books.get("kafka").foreach(println _) //foreach遍历遇到None的时候，什么也不做，不会执行println操作。
```

## 提升

### 隐式转换和隐式参数

implicit

隐式转换函数只能有一个?!

==隐式转换不是靠名字区分的!!==

建议 见名知意

利用隐式转换可以提供优雅的类库，对类库的使用者隐匿掉那些枯燥乏味的细节。

隐式函数是指那种以implicit关键字声明的带有单个参数的函数。

```scala
val x:Int = 3.5 //报错!! 会丢东西!故不可以自动转换!
//解决方案,添加隐式函数
implicit def double2Int(x:Double)=x.toInt 
val x:Int = 3.5 //当前作用域!直接定义函数完成什么功能即可
```

Scala会考虑如下的隐式转换规则： 

==//隐式转换相当于借用其他函数, 看起来好像是类库中的方法,实际是隐式转换后的类的方法==

1. 位于源或目标类型的==伴生对象==中的隐式函数
2. 位于==当前作用域==可以以单个标示符指代的隐式函数

```scala
import java.io.File
import scala.io.Source

//隐式的增强File类的方法
class RichFile(val commonFile:File){
    def read = Source.fromFile(commonFile.getPath).mkString
}
object RichFile{
  //隐式转换方法，功能是提供“低级类型”到“增强类型”的转换
    implicit def file2RichFile(commonFile:File) = new RichFile(commonFIle)
    //要在伴生对象中定义隐式转换方法! 相当于借用别人的方法!
}
object MainApp{
    def main(args:Array[String]):Unit = {
        import RichFile._ //用前必须导入定义隐式转换的类!!
        println(new File("/home/xdl/a.txt").read)        
    }
}

```

#### 隐式参数

隐式参数有点类似缺省参数，如果在调用方法时没有提供某个参数，编译器会在当前作用域查找是否有符合条件的implicit 对象可以作为参数传入；

不同于缺省参数，隐式参数的值可以在方法调用前的上下文中指定，这使隐式参数更加灵活；

函数或方法可以带有一个标记为implicit的参数列表。这种情况下，编译器将会查找缺省值，提供给该函数或方法。

```scala
object context{
    //1. 在一个类中定义隐式值
    implicit val aa = "guanyu" //隐式值
}
object ImplicitDemo {
  // 隐式参数，给定了缺省值 
  //2. 将隐式值体现在参数列表里
  def say (implicit name:String = "zhangfei"): Unit = println(s"hello $name")
  def main(args: Array[String]): Unit = {
    // 不使用隐式参数
    say()
    say("liubei")
    import cn.itxdl.scala.basic3.context._//使用前必须导入定义隐式值的类!
    say// 使用隐式参数
  }
}
```

```scala
case class Delimiters(left:String, right:String)
object FrenchPunctuation{
    implicit val quoteDelimiters = Delimiters("<<",">>")
}
object MyApp{
    def quote(what:String)(implicit delims:Delimiters)=
    delims.left + what + delims.right
   
    def main(args:Array[String]):Unit={
        val a = quote("Jack")(Delimiters("<",">"))
        println(a)
        
        import FrenchPunctuation._ //引入隐式值
        println(quote("Jack"))
    }
}
```

####递归和尾递归

递归的缺点: 效率低

因为一直在进栈出栈,如果调用的深度过深,有可能会栈溢出.

尾递归: 自动转化为while循环的效率

```scala
// 递归求List的和
// List、Array 都可以看成是 头+尾，尾部又是头+尾 的结构
def sum(xs: List[Int]): Int =
  if (xs.isEmpty)
    0
  else
    xs.head + sum(xs.tail)

```

尾递归是指递归调用是函数的最后一个语句，而且其结果被直接返回，这是一类特殊的递归调用。由于递归结果总是直接返回，尾递归比较方便转换为循环，因此编译器容易对它进行优化。

### Ordered & Ordering

Scala提供两个特质（trait）Ordered与Ordering用于比较:

- Ordered混入（mix）Java的Comparable接口
- 而Ordering则混入Comparator接口

实现Comparable接口的类，其对象具有了可比较性；

实现comparator接口的类，则提供一个外部比较器，用于比较两个对象。

Comparable是排序接口。若一个类实现了Comparable接口，就意味着该类支持排序。实现了Comparable接口的类的对象的列表或数组可以通过Collections.sort或Arrays.sort进行自动排序。

Comparator是比较接口。通过实现Comparator来新建一个比较器，然后通过这个比较器对类进行排序。

Ordered与Ordering的区别与之相类似：

- Ordering特质定义了相同类型间的比较方式，但这种内部比较方式是单一的
- Ordered则是提供比较器模板，可以自定义多种比较方式

Ordered特质更像是rich版的Comparable接口，除了compare方法外，丰富了比较操作（<,

\>, <=, >=）定义compare方法，Ordered特质会利用这个方法定义<、>、<=、>=。从而，Ordered特质让你可以通过仅仅实现一个compare方法，使类具有了全套的比较方法：

```java
case class Person1(name: String)
case class Person2(name: String) extends Ordered[Person2] {
  def compare(other: Person2) = name compare other.name//定义比较的字段
}
val p1 = Person1("Tom")
val p2 = Person1("Bob")
val p3 = Person2("Tom")
val p4 = Person2("Bob")

// p1、p1 不能比较，因为没有混入Ordered接口
p1 < p2
p3 < p4
```

//两种等价的定义方式(数字类型有减法, 字符串没有减法操作)

case class Person2(age: Int)extends Ordered[Person2] {

 def compare(other: Person2) = age compare other.age }

case class Person2(age: Int)extends Ordered[Person2] {

 def compare(other: Person2) = this.age - other.age }

### sorted, sortWith, sortBy

1. 基于单集合单字段的排序

```scala
val xs = Seq(1, 5, 3, 4, 6, 2)
xs.sorted			// 升序
xs.sorted.reverse		// 降序

xs.sortBy(d=>d)
xs.sortBy(d=>d).reverse

xs.sortWith(_<_)
xs.sortWith(_>_)
```

2. ​











```scala
case class LabelPoint(label:String,point:Array[Double])
object Test1{
    def main(args:Array[String]):Unit={
        val fileIter = Source.fromFile("/home/xdl/scala/iris.dat").getLines
       val data = for(line<-fileIter) yield {
            val value = line.split(",") //返回的是Array[String]
            LabelPoint(value.last,value.init.map(_.toDouble))
        }
        println(data) //结果为:non-empty iterator 因为输入是迭代器,故输出也是迭代器
        //如果想要fileIter是数组类型,在getLines后面加toArray
        //这样的话,println(data)打印出的是一个地址
        
        //参数变为data.toBuffer,label可以打印出值,point打印的仍然是数组的地址(是全部的数据)
        //打印结果为: ArrayBuffer(LabelPoint(xx,xxx),LabelPoint(xx,xxx)...)

        //上一个语句变为:
        val data1 = data.toBuffer 
        data1.foreach(x=>println(x.label + " " + x.point.toBuffer))
        //打印结果为:xx,ArrayBuffer(xx,xx,xx,xx)
        
        //还可以: 前提getLines后面是跟了.toArray
       	println(data(0).label + "$" + data(0).point.toBuffer)
    }
}

```

