##第一章

###可伸展的语言

Scala被设计成可以随着使用者的需求而扩展.

从技术层面上来说, Scala是一种把面向对象和函数式编程理念加入静态类型语言中的混合体.

在可伸展性方面,这两种编程风格具有互补的力量. scala的函数式编程简化了用简单部件搭建实际应用, 它的面向对象特性又使得它便于构造大型系统并使他们适应新的需求.

####增加新的类型

你始终可以裁剪程序以符合所需,因为所有的东西都是==基于库模块==的,可以依照需要选择和修改.

Scala允许用户通过定义感觉像原生语言支持一样的易用库在他们需要的方向上发展和构造语言.

####增加新的控制结构

Java的线程模型: 围绕共享内存和锁机制.

当系统规模和复杂度不断变大的时候,这种模型越发变得难以理解,很难说程序里面没有资源竞争或潜藏的死锁.

目前可认为比较安全的可选方案是消息传递架构, 例如Erlang应用的actor方案

Scala也可以像使用其他Java Api那样用它编程 不过Scala还提供了一个实质上实现了Erlang的actor模型的附加库.

### 是什么让Scala具有可扩展性?

在Scala里, 函数就是对象!

函数类型是能够被子类继承的类.

#### Scala是面向对象的

面向对象: 把数据和操作放进某种形式的容器中. 

面向对象最伟大的思想: 让这些容器完全的通用化, 这样就能像保存数据那也保存操作,并且还可以把这些容器作为值存储到其他容器里, 或作为参数传递给操作.

Scala是纯粹的面向对象语言, 每个值都是对象, 每个操作都是方法调用.

如果说到组装对象, Scala比多数别的语言更胜一筹. Scala的特质(trait)就是其中一例. 特质类似Java中的接口, 但可以有方法实现及字段.

可以通过混入组装(mixin composition)构造对象, 从而带有了类的成员并加入了若干特质的成员.这样的构造方式, 可以让不同用途的类包装不同的特质.

与类不同, 特质可以把一些新的功能加入还未定义的超累中. 这使得特质比类更具有"可加性", 还可以避免在多重继承里,通过若干不同渠道继承相同类时发生经典的"菱形继承"问题.

#### Scala是函数式的

函数式编程的2种指导理念:

- 函数是头等值

- 程序的操作应该把输入值映射为输出值而不是就地修改数据.

  不可变数据结构是函数式语言的一块基石.

  Scala库在Java API之上定义了更多的不可变数据类型,如列表, 元组,  映射表和集

  函数式编程第二种理念的另一种解释是:方法不应有任何副作用(side effect),方法与其所在环境交流的唯一方式应该是获得参数和返回结果.

  	函数式编程鼓励使用不可变数据结构和指称透明的方法.Scala不强迫使用函数式风格,必要的情况下,可以写成指令形式(imperative),用可变数据或有副作用的方法调用. 但是Scala有更好的函数式编程方式做替代,因此可以轻松避免使用它们


#### 为什么选择Scala?

- 兼容性

  Scala的字符串支持类似与toInt 和 toFloat的方法, 可以把字符串转换成整数或浮点数.

  toInt 替代了 Integer.parseInt(str)

  ```txt
  How 做到不打破互操作的基础上的实现?
  Scala有一个通用方案可以解决这种高级库设计和互操作对立的问题.Scala允许定义类型失配或者选用不存在的方法时使用隐式转换(implicit conversion)
  如:在字符串中寻找toInt方法
  Scala编译器发现String无这种方法
  但它会发现 把Java的String对象转换为Scala的RichString类实例的隐式转换,而RichString类中定义了此方法.
  因此,在执行toInt操作前,转换被隐式应用了
  ```

- 简洁

  - 避免了一些束缚Java程序的固定写法
  - 可以类型推断
  - 减少代码的关键(相对Java)或许是现成的代码库.

- 高层抽象(Scala是高级的)

  - Scala可以通过帮助提升接口的抽象级别来帮助管理复杂性.

- 高级的静态类型化

  静态类型系统可以根据保存和计算的值的类型认定变量和表达式类型

  - 类型推断—>避免了冗余性

  - 通过模式匹配和一些新的编写和组织类型的方法获得了灵活性

    优点: 程序抽象的可检验属性, 安全的重构, 及更好的文档

	构成可重用控件接口的成员的类型符号应该是显示给出的, 因为它们构成了控件和它的使用者间契约的重要部分.

## 第二章 Scala入门初探

```scala
scala>1+2
结果: res0:Int=3

resX识别符还能使用在后续代码中.
```

想在解释器中跨行输入的话, 只要一行行写进去即可. 如果输入到行尾还没结束,解释器将在下一行回应一个竖线.

如果你发现了一些错误而解释器仍然在等着更多的输入,你可以通过按两次回车键取消掉

#### 函数定义

函数的每个参数都必须带有前缀冒号的类型标注! 因为Scala编译器无法推断函数的参数类型.

如果想要离开解释器. 输入 :quit 或者 :q

####编写Scala脚本

脚本编写: println("Hello, " + args(0) + "!")

$ scala helloarg.scala planet

Scala和Java一样,必须把while或if的布尔表达式放在括号里

#### 用foreach和for做枚举

```scala
args.foreach(arg=> println(arg))
$ scala  pa.scala  Concise  is  nice
```

这行代码对args调用了foreach方法,并把函数作为参数传入, 在这里是名为arg的函数字面量. 函数体是println(arg)

可以给arg加上类型名, 不过需要把参数部分写在括号里:

- args.foreach((arg:String) => println(arg))

  如果函数字面量只有一行语句并只带一个参数, 那么甚至连指代参数都不需要.  args.foreach(println)

- for(arg<-args) println(arg) :

  左侧的arg是val的名称(不是var) 

  -  这里一定是val, 因此只写arg即可, 无需写成val arg
  - 对于args数组的每一个元素,枚举的时候都会创建并初		始化新的arg值,然后调用执行for的函数体

## 第三章

### 使用类型参数化数组(Array)

Scala里使用new实例化对象(或者叫类实例). 实例化过程中,可以用值和类型使对象参数化(parameterize)

参数化: 值在创建实例的同时完成对它的"设置".

val greetString = new Array\[String](3)

gteetString的类型是 Array[String],而不是Array\[String](3)

```txt
如果用val定义变量, name这个变量就不能被重新赋值,但变量指向的对象内部却仍可以改变.所以,本例中, greetString对象不能被重新赋值成别的数组: 
它将永远指向初始化是指定的那个Array[String]实例.
但Array[String]的内部元素始终能被修改,因此数组本身是可变的.
```

for(i <- 0 to 2 )    print(greetString( i ) )

Scala的另一个规则: 如果方法只有一个参数,调用的时候可以省略点及括号; 

0 to 2 == (0).to(2)

注意: 此语法只有在明确指定方法调用的接受者时才有效. 如不可以写成 println 10 ,—>需写成: Console println 10

从技术层面上讲, Scala没有操作符重载, 因为它根本没有传统意义上的操作符. 取而代之的是, 诸如+ - * /这样的字符, 可以用来做方法名.

```txt
why在Scala里要用括号访问数组?

数组也只是类的实例. 用括号传递给变量一个或多个值参数时, Scala会把它转换为对apply方法的调用.(访问元素相当于方法调用)

不仅仅对数组: 任何对于对象的值参数应用将都被转换为对apply方法的调用.前提是这个类型实例实际定义过apply方法.

与之相似的是:(赋值语句)
当对带有括号并包括一到若干参数的变量赋值时, 编译器将使用对象的update方法对括号里的参数(索引值)和等号右边的对象执行调用.
greetString(0) = "hello" 等价于:
greetString.update(0,"hello")

//apply多用于构造一个对象, update表示赋值,都是调用的伴生对象的apply和update
```

更为简洁的写法: val numNames = Array( "zero","one","two" )

完整写法: val numNames2 = Array.apply("zero","one","two")

### 使用列表(List)

Array[String] :  长度固定,值可变,同类对象序列

List[String] : 长度固定,值不可变,同类对象序列

创建方式:

​	val oneTwoThree = List( 1,2,3 )

==注意== : 对某个列表调用方法时, 似乎这个列表发生了改变, 而实际上只是用新的值重建了列表然后再返回.

: : : 实现叠加功能 (两个list间)

: : 把新元素组合到现有列表的最前端()

​	val twoThree = List( 2,3 )

​	1 : : twoThree (: :是右操作数twoThree的方法)

​	等价于 twoThree. : :(1)

空列表: Nil

```txt
why列表不支持添加(append)操作?
因为随着列表变长, append的耗时将呈线性增长.
而使用::做前缀仅仅耗用固定的时间.
若想在尾部添加:
方法1: 前缀进去后,调用reverse
方法2: 使用ListBuffer,一种提供append操作的可变列表,完成之后调用toList
```

thrill.sort( (s,t)=>s.charAt(0).toLowerCase < t.charAt(0).toLowerCase )

返回thrill列表元素安装第一个字符的字母小写排序之后依次组成的新列表

###使用元组(Tuple)

元组与List一样,也是不可变的.但是,元组可以包含不同类型的元素.

> 需求: 在方法里返回多个对象

Java里: 创建JavaBean以包含多个返回值

Scala里: 可以仅返回元组.

实现方法: 把元组实例化需要的对象放在括号里, 并用逗号分隔即可. 元组实例化之后, 可以用点号, 下划线 和 ==基于1==的索引访问其中的元素.

```scala
val  pair = (99, "Luftballons")
println(pair._1)
println(pair._2)
```

> Q: why不用访问列表的方法访问元组?

A: 因为列表的apply方法始终返回同样的类型,但元组里的类型不尽相同. \_1的结果类型与_2的不一致,因此访问方法也不同.

并且元组的索引是基于1而非0

### 使用集(set) 和 映射(map)

对于set和map来说,Scala同样有可变的和不可变的, 不过并非各提供两种类型, 而是通过类继承的差别把可变性差异蕴含其中.

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-17 下午9.42.10.png)

```txt
Scala的API包含set的基本特质
两个子特质: 分别属于不同的包 因此3个特质共享同样的简化名set,但是全称不同.
HashSet类,各有一个扩展了可变的和另一个扩展了不可变的Set特质(默认是不可变的)
```

set的构造方法:

var jetSet=Set("Boeing","Airbus")

jetSet += "Lear"

println(jetSet.contains("Cessna"))

- 通过调用Set伴生对象的apply工厂方法构造对象.

- 对于加入新变量 +  , 可变和不可变集都提供了+方法, 

  可变集把元素加入自身, 不可变集则创建并包含了添加元素的新集. 故只有可变集提供的才是真正的+=方法.

  上面的语句等价为: jetSet = jetSet + "Lear"

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-17 下午10.12.47.png)

构造方法:

​	import scala.collection.mutable.Map

​	val treasureMap = Map\[Int, String]( )

​	treasureMap += (1 -> "Go to island")

​	treasureMap += (2 -> "Find big X on ground")

​	treasureMap += (3 -> "Dig")

​	println(treasureMap(2))

Scala的任何对象都可以调用 -> 方法, 并返回包含键值对的二元组.

### 学习识别函数式风格

大致可以说: 如果代码包含了任何var变量,那它可能就是指令式的风格,如果代码根本就没有var, 即仅仅包含val,那或许是函数式的风格.

因此, 向函数式风格转变的方式之一,就是尝试不用任何var编程.

识别函数是否有副作用的地方就在于其结果类型是否为Unit.

如果某个函数不返回任何有用的值, 也就是说如果返回类型为Unit,那么这个函数唯一能产生的作用就只能是通过某种副作用.

```scala
//完全没有副作用或var的mkString方法
//无副作用的代码,有助于程序更容易调试
def formatArgs(args:Array[String]) = args.mkString("\n")
```

> Scala程序员的平衡感

崇尚val, 不可变对象 和 没有副作用的方法.

首先想到他们. 只有在特定需要和并加以权衡之后才选择var,可变对象和有副作用的方法.

### 从文件里读取文本行

方式1 :

```scala
import scala.io.Source
if(args.length>0){
    for(line<-Source.fromFile(args(0)).getLines)
//getLines后返回Iterator(String)
    print(line.length + " " + line)
}
else
	Console.err.println("Please enter filename")
```

方式2 :

调整格式为: 

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-17 下午11.11.21.png)

![](/Users/lijingwan/Documents/img/屏幕快照 2018-04-17 下午11.11.59.png)

```scala
import scala.io.Source
def widthOfLength(s:String) = s.length.toString.length
if(args.length>0){
    val lines = Source.fromFile(args(0)).getLines.toList
//toList必须加.因为getLines方法返回的是枚举器.一旦完成遍历,枚举器就失效了; 而通过调用toList把它转换为List,我们把文件中的所有行全部贮存在内存中,因此可以随时使用.lines变量因此就指向着包含了指定文件的文本字符串列表
    val longestLine = lines.reduceLeft(
        (a,b)=>if(a.length>b.length) a else b
    )
    val maxWidth = widthOfLength(longestLine)
    for(line<-lines){
        val numSpaces = maxWidth-widthOfLength(line)
        val padding = " "*numSpaces
        print(padding + line.length + "|" + line)
    }
}
else 
	Console.err.println("Please enter filename")


//Tip:计算字符串的最大长度 方式2:
	var maxWidth = 0
	for(line<-lines)
		maxWidth = maxWidth.max(widthOfLength(line))
//注: max 任何Int上都可以调用,返回被调用者和传入参数中较大的值, 本方法使用了var
//方式1的方法没有用var来实现.
```

