map: 针对集合中的每一个元素,进行变换映射出新的集合.

yield: 产生一个数据集

val data = for( line<- fileIter ) yield {

​	val value = line.split(",")

​	LabelPoint(value.last, value.init.map(_.toDouble))

} //返回出的结果 ,还是迭代器! 因为输入是迭代器

print(data.toBuffer) //打印出来是每个labelpoint的地址

要想打印出每个值:

data.toBuffer.foreach(println _) 

打印的结果是:  LabelPoint( Iris-… , 地址 )

如果想把元素打印出来: 

val data1 = data.toBuffer

data1.foreach(x=>println(x.label + x.point.toBuffer))

println(data(0).label +  "$" + data(0).point.toBuffer): 打印第0个元素