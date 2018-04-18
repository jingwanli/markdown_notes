### 实现一个分数类型的class

> 需求:

​	初始化传入的是Int, Int

1. 能打印分子/分母


2. 创建类型的时候不用写关键字new


3. 能够约分(显示分数的最简形式)


4. 能够实现加减乘除四则运算(加减乘除)

 	5. 实现分数与整数的运算
	6. 能够与分数或Int比较大小, 将多个分数类型放进List中调用sorted方法能够排序
	7. 对象要能进行模式匹配

```scala
class Rational(n:Int, d: Int){

	assert(d!=0) //除数不能为0 否则抛出异常
	private val g = gcd(n.abs,d.abs)
	private val number1 = n/g
	private val number2 = d/g 

    def this(n:Int) = this(n,1)
	def +(that:Rational):Rational = {
	 Rational( number1*that.number2+ that.number1* number2, number2*that.number2)
	}
    
    def +(m:Int):Rational={
      Rational(number1+m*number2,number2)
    }

	override def toString:String = "Rational:"+number1 + "/" +number2

	//能够约分:?
	private  def gcd(a:Int, b:Int):Int ={
		println(s"a=$a, b=$b" ) //a是分子, b是分母
		if(b==0) a 
		else gcd(b, a%b)
	}	
}

object Rational{
	def apply(n:Int, d:Int):Rational = new Rational(n, d)
	def apply(n:Int):Rational = new Rational(n, 1) //重载
	main-----
    val r1 =  new Rational(2, 4)
	println(r1)
	var r2 = Rational(2,6) //apply方法
	println(r2)
	val r3 = new Rational(2)
	println(r3)
	val r4 = Rational(3)
	println(r4)
	//val r5 = r1.add(r2)
	val r5 = r1+r2 //运算符就是方法!!!!
	println(r5)
    
    val r6 = r5+2
    println(r6)
    
    val r7 = 3.+(r6)
    println(r7)

}

/* 	15/100

a	b	a%b
15	100	  15
100	  15  10
15    10	5	  
10	   5	0
5      0 
```

完整版:

```scala
class Rational(n:Int,d:Int)extends Ordered[Rational]{}
```

##函数

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-16 上午10.00.56.png)

###函数字面量

函数字面量可以提现函数式编程的核心理念

函数式编程中, 函数是"头等公民". 

函数的值 : 函数字面量

函数的类型:(Int)=> Int

函数的值: (value) = >{value +1}

==注: 要用=> 而不是\===

1. 单条语句, 可以省略值的大括号

2. 单个类型, 可以省略小括号

3. scala可以对类型进行自动推断, (函数的参数必须带类型! 否则无法自动推断!)

   val counter:(Int) => Int = {(value)=>value+1}

   counter(1)

### 匿名函数

匿名函数: 又被称为Lambda表达式

形式:

==(参数) => 表达式==

最简形式: (x:int) => x+1

(1 to 10).map((x:Int)=>x*2) //传入的是匿名函数

(1 to 10).map(x=>x*2) //自动类型推断

val i:Int = 7

//函数定义方式:

- val i:(Int)=>Int = (x:Int)=>x+1 //方式1
- def i:(Int)=>Int = (x:Int)=>x+1 //方式2
- def max(x:Int,y:Int):Int ={ …..  } //方式3

   //调用方式:

		println(i(3))

- 方式4: 普通的定义方法:(通常情况)

  def max(x:Int, y:Int):Int={

  ​	if(x>y) x else y

  }

### 闭包

闭包是一个函数!

var mode = 100

def add(x:Int) = x + mode +10 //有了上下文,这种写法是正确的

mode是可变的— 自由变量

闭包: 捕获自由变量, 由开放-封闭的过程

### 占位符语法

_: 代表集合的每一个元素

_: 代表匿名函数的第一个参数,第二次出现的\_代表参数列表的第二个参数

(1 to 10).reduce( \_*_ )

占位符只能出现一次!

多个下划线指代多个参数, 而不是单个参数的重复运用!

### 高阶函数

接收一个或多个函数作为输入 或 输出一个函数

函数的参数可以是变量，而函数又可以赋值给变量，即函数和变量地位一样，所以函数参数也可以是函数

常用的高阶函数：map、reduce、flat、filter、sum、count… … 

def a(n:Int) = "*" *n //\*运算符是定义在字符串的,所以不可以反过来写

(1 to  20).map(a(_)).foreach(println _)

### 柯里化

将原来接收两个参数的函数变成新的接收一个参数的函数的过程；

def func1(x:Int, y:Int) = x*y

def func2(x:Int)(y:Int) =  x*y

val a = func2(5) _ 

val a = func2(5)(_)

val b = a(4)

func2(3)(5)

新的函数返回一个原有的函数的第二个参数作为参数的函数。

```scala
def func1(x: Int, y: Int) = x * y

// 定一个接受一个参数，生成另一个接受一个参数的函数
def func2(x: Int) = (y: Int) => x * y
def func2(x: Int)(y: Int) = x * y
// 以上两种写法等价

func2(6)(7) 
func2(6)(y)
```

```scala
Scala支持如下简写来定义这样的柯里化函数：
def f1 (x: Double) (y: Double) = x * y * y
def f2 = f1(3.14)(_)
def f2 = f1(3.14)_
val s1 = f2(5)
```

### 函数式编程

两大核心理念:

1. 函数是一等的值

  在函数式编程语言中，函数值的地位与整数、字符串等是相同的。可以将函数作为参数传递给其他函数，作为返回值返回它们，或者将它们保存在变量里。还可以在函数中定义另一个函数，就像在函数中定义整数那样；也可以在定义函数时不指定名字，就像整数字面量42，允许函数字面量散落在代码中。

2. 程序中的操作应该将输入值映射成输出值, 而不是当场修改数据

   不可变数据结构是函数式编程的基石之一。这个核心理念的另一种表述是方法不应该有副作用（side effect），函数应该只通过接收参数和返回结果这两种方式与外部环境通信。


> 案例:
>
> 错误写法:
>
> def s1(x:Int, y:Array[int]):Int ={
>
> ​	y(0) = 100     //不应该修改y的值. 如果想修改,应该将返回值
>
> ​			    //设置为Array. 同时在函数体中,新建一个Array
>
> }

1. 函数式一等的值(first-class)

2. 程序中的操作应该将输入值映射成输出值, 而不是当初修改数据

   函数应该只通过接收参数和返回结果两种方式与外部通信

def func(n:Int,m:Object,p:Array[Int]):Int{

​	p(0) = 100	

}

def func(n:Int, m:Object, p:Array[Int]):Array[Int]{

​	p(0) = 100

}




## 集合

Scala提供的一套丰富的容器(Collection)库: 包括List,Array, Set, Map. 

3类:

- Seq(序列) 有序
- Set(集) 不可重复
- Map(映射) 

还可以划分成: 可变集合和不可变集合(缺省)

还可以划分: 有序和无序

> 问题

Array, String 分别属于什么类型的集合?

​	Array: 可变 

​	String: 不可变

> Scala使用了3个包来组织容器类:

scala.collection

scala.collection.immutable

scala.collection.mutable

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-16 下午2.07.53.png)

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-16 下午2.08.40.png)

绿色代表: traiter或abstract 

蓝色表示:class

虚线:表示隐式转换

粗线: 缺省的实现 

​	可变:默认实现: ArrayBuffer

​	不可变: 默认实现: List

val a = Traversable(1,2,3,4)

结果: List(1,2,3,4)

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-16 下午2.13.27.png)

需要导包!

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-16 下午7.09.22.png)

### List列表

List 比 Array 多了一个特性: 不可变!

与数组相比, 特点:

- 列表是不可变的。即列表的元素不能通过赋值改变。
- 列表的结构是递归的，而数组是平的；

初始化的时候必须赋值! 否则没有值了

头部: head : 列表第一个==元素值==

尾部:tail: 除了第一个元素之外的( 也由头和尾构成—递归)==新列表==

Nil ——空List对象

> 构造列表的方式:

1. 通过: :在已有列表前端增加元素(: :是右操作符)

   val 1st = 1::2::Nil ———>List[Int] = LIst(1,2)

   :: 右结合的运算符!!— 从Nil开始向左结合

   List 可以相加: 方式1: ++ (表示集合间相加)

				方式2: lst1 : : : lst2
	
		增加元素: 3::lst  (顺序不能反过来!)

2. Nil: 空列表对象.借助Nil, 可以将多个元素用操作符: :串起来初始化一个列表:

   val  mylist = 1::2::3::Nil 结果: List(1,2,3)

   lst::3 ::Nil 结果: List(List(1,2) ,3)

```scala
//求和
object ListDemo{
    main----
    val lst1 = (1 to 10).toList
    lst1.sum //方式1
    //方式2 不推荐此写法
    def sum1(1st:List[Int]):Int={
        if(lst == Nil) 0 else lst.head + sum1(lst.tail)    
    }
    val n1 = sum1(lst1)
    println(s"n1 = $n1")
    
    //方式3 推荐: 模式匹配
    def sum2(lst:List[Int])=lst match{
        case Nil=> 0
        case head::tail => head + sum2(tail)
    }
}

```

> 随机数 + 冒泡算法

```scala
//java中写法:
object BubbleDemo {

  def sort(arr:Array[Int])= {
      val len = arr.length
      for(i<-0 until len-1) {
        for(j<-0 until len-i-1) {
              if(arr(j)>arr(j+1)) {
                val temp = arr(j)
                arr(j) = arr(j + 1)
                arr(j + 1) = temp
              }
        }
      }
    arr.toBuffer
  }

  def main(args: Array[String]): Unit = {
    var brr = Array(1,3,9,2,6,78,90,67,45,43)
    println(sort(brr))
  }
}

```

```scala
val random = new Random()
val lst2 = (1 to 10).map(x=> random.nextInt(10)).toList
println(lst2) //每次的随机数都不同

需求:每次生成固定的随机数
val Random = new Random(100) //100是种子
val lst2 = (1 to 10).map(x=>random.nextInt(1000)).toList
println(lst2)

...
//冒泡排序算法
def sort(list:List[Int]):List[Int] = list match{
    case Nil => List()
    case head:tail = > compute(head,sort(tail)) 
    //sort是针对一个参数(列表)的,compute是针对2个参数的(相当于一个+/*等简单的运算函数)
}
def compute(data:Int, lst:List[Int]):List[Int]=lst match{
      case Nil => List(data)//注意:这里是有data的!!!
      case head:tail = > if(data <= head) data::lst
    					else head :: compute(data, tail)
	                          
}

val lst3 = sour(lst2)
println("lst3 = " + lst3)

```

### 集合Set

不重复元素的容器. 列表中的元素是按照插入的先后顺序来组织的；Set中的元素并不会记录元素的插入顺序，而是以“哈希”对元素的值进行组织，它允许你快速地找到某个元素；

集合包括可变集和不可变集，分别位于scala.collection.mutable包和scala.collection.immutable包，缺省情况下创建的是不可变；

如果要声明一个可变Set，则需要引入scala.collection.mutable.Set；

典型的操作: 交并差

//交集

Set(1,2,3) & Set(2,4)

Set(1,2,3) intersect Set(2,4)

Set(1,2,3).intersect(Set(2,4))

//并集

Set(1,2,3) ++ Set(2,4)

Set(1,2,3) | Set(2,4)

Set(1,2,3) union Set(2,4)

//差集

Set(1,2,3) -- Set(2,4)

Set(1,2,3) &~ Set(2,4) 

Set(1,2,3) diff Set(2,4)

### 序列Seq

具有一定长度的可迭代访问的对象,其中每个元素均带有一个从0开始计数的固定索引位置。

Vector是ArrayBuffer的不可变版本，可通过下标快速的随机访问。Vector是以树形结构的形式实现的，每个节点可以有不超过32个子节点。这样对于有100万的元素的向量而言，只需要4层节点。

Range通常表示一个整数序列，例如0,1,2,3,4,5或10,20,30。Rang对象并不存储所有值而只存储起始值、结束值和增值。

其他常见的Seq类型还包括：String、List、Queue、Stack

### Vector

相当于数组

每层: 放32个数据. 4层可以放百万数据

### 迭代器

在Scala中，迭代器（Iterator）不是一个集合，但是，提供了访问集合的一种方法；迭代器包含两个基本操作：next和hasNext

迭代的常用方式: 

1. for(item <- iter) println(item)
2. iter.toList.foreach(println _) 

//迭代器只能迭代一次!!

```scala
val iter = Iterator("hadoop", "spark", "scala")
while (iter.hasNext){
  println(iter.next())
}

val iter = Iterator("hadoop", "spark", "scala")
for (elem <- iter){
  println(elem)
}

val iter = Iterator("hadoop", "spark", "scala")
while (iter.hasNext){
  println(iter.next())
}
// 这里有坑，迭代器遍历一遍之后指针指到了集合尾部
iter.size
```

## 常用函数

###foreach & map

```scala
//使用foreach进行遍历
val numlist = (1 to 10).toList
numlist.foreach(elem=>print(elem+" "))
numlist.foreach(print _)
numlist.foreach(print)
numlist foreach print

//使用map进行遍历
numlist.map(_>2) //返回boolean值的集合
numlist.map(_*2)

```

##### 比较foreach, map

- foreache、map共同点在于：都可以遍历集合对象，并对每一项执行指定的操作。
- foreach无返回值（准确说返回void）,map返回集合对象；
- foreach无法代替map，而map方法却可以代替foreach；
- foreach用于遍历集合，而map用于将一个集合转换到另一个集合。

### flatten

```scala
val lst = List(List(1,2,3,4),List(4,5,6,7))
println(lst.flatten)
//结果: List(1,2,3,4,4,5,6,7)
println(lst.flatMap(_.map(_*2)))
等价于: println(lst.flatten.map(_*2))
//这里不能倒过来. 必须先压平后map!

案例:
var a = Array("hello jd","hi jd")

flatMap是map的扩展；
在flatMap中传入一个函数，该函数对每个输入都返回一个集合（而不是一个元素），最后flatMap把生成的多个集合“拍扁”成为一个集合。
var arr1 = a.flatMap(_.split(" ")).toBuffer
//打印结果: hello,jd,hi,jd

val arr2 = a.flatten.toBuffer
//结果为: ArrayBuffer(h, e, l, l, o,  , j, d, h, i,  , j, d)

var arr3 = a.map(_.split("\\s+")).flattern.toBuffer
//和arr1结果相同

flatMap = flatten + map 或
flatMap = map + 
```

### groupBy

```scala
val data = List(("HomeWay", "Male"), ("XSDYM", "Female"), ("Mr.Wang","Male"))
// 以下两句结果等价
val group1 = data.groupBy(_._2) //按什么分组,相当于用什么key
结果: Map[String,List[(String, String)]] = Map(Male -> List((HomeWay,Male), (Mr.Wang,Male)), Female -> List((XSDYM,Female)))

val group2 = data.groupBy{case (name,sex) => sex} 
结果:scala.collection.immutable.Map[String,List[(String, String)]] = Map(Male -> List((HomeWay,Male), (Mr.Wang,Male)), Female -> List((XSDYM,Female)))


val z = Seq("a", "b", "c", "d").groupBy { x => 
  x match {
    case "a" => 1
    case "b" => 1
    case _ => 2
  }
}
结果:scala.collection.immutable.Map[Int,Seq[String]] = Map(2 -> List(c, d), 1 -> List(a, b))

List((1,2),(1,3),(2,3),(3,3),(3,4)).groupBy(_._1)
结果:scala.collection.immutable.Map[Int,List[(Int, Int)]] = Map(2 -> List((2,3)), 1 -> List((1,2), (1,3)), 3 -> List((3,3), (3,4)))
```

### reduce

```scala
reduce包含reduceLeft和reduceRight两种操作，前者从集合的头部开始操作，后者从集合的尾部开始操作。
val lst = (1 to 5).toList
val n1 = lst.reduce(_+_)
val n2 = lst.reduce(_*_)
写全:(验证方法)
lst.reduce((x,y)=>{
    println(s"x=$x,y=$y")
    x+y}) //想办法去推断它的意思!

// 下面两个操作是等价的
list.reduce(_+_)
list.reduce((x,y)=>x+y)

```

reduceLeft(等价于不加Left)

reduceRight(从右往左)

reduce: 默认需要满足交换律和结合律(以免出现两个不同结果)

### fold

fold操作和reduce操作有些类似。

fold操作需要从一个初始的“种子”值开始，并以该值作为上下文，处理集合中的每个元素。

```scala
val list = List(1, 2, 3, 4, 5)
list.fold(10)(_*_)
list.fold(10)((x,y)=>{println(s"x=$x, y=$y");x*y})

val n3 = lst.fold(10)(_+_)
//结果: 25 多加一个10
写全:
lst.fold(10)
```

与reduce类似，fold也有两个变体：foldLeft()和foldRight()

foldLeft()，第一个参数为累计值，集合遍历的方向是从左到右；foldRight()，第二个参数为累计值，集合遍历的方向是从右到左；

### filter

使用filter的两个关键点：

算法要能正确判断出所需要的元素，并返回true，对于不需要的数据则返回false

用一个新的变量指向filter方法返回的集合，因为filter方法并不会对原集合做改变

```scala
val lst4 = lst.filter(_>4)
//filter返回的是boolean值,所以一定要用一个变量来接收!
println(lst.toBuffer) //ArrayBuffer(1,2,3,4,5)
println(lst4.toBuffer)//ArrayBuffer(5)

val x = List.range(1, 10)
val evens = x.filter(_ % 2 == 0)

val fruits = Set("orange", "peach", "apple", "banana")
val x = fruits.filter(_.startsWith("a"))
val x = fruits.filter(_.length > 5)

// 提取List中的 String 元素
val list = "apple" :: "banana" :: 1 :: 2 :: Nil
val strings = list.filter{
  case s:String => true
  case _ => false
}
```

> 案例(机器学习的一个算法)

KNN算法: K近邻算法

参考的是 : 临近的数据点

鸢尾花数据集

算法过程:

- 读2个文件(样本, 要求分类的数据)
- 每一个测试数据与样本数据求距离, 只关心k个距离最小的点(奇数个)
- wordcount



> WordCount