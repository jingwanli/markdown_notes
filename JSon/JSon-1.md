### JSON

轻量级的数据交换格式；一般应用于网络接口的数据返回；

存储的数据，key都通过引号引住；value：数字和boolean值可以不用引号，字符串必须用引号引住！

JSON中包含两种格式: ，

####1.JSON对象

{ } {"name":"高飞","age":20} //对象中不能有重复的key

####2.JSON数组

使用[]封装数据，分为下标和值，值与值之间使用，分割；

案例：

描述一组成绩：

[100,50,10,100]

JSON对象和JSON数组是可以互相嵌套的：

{"name":"高飞",

 "age":18,

 "friends":[ 

{"name":"A","age":18},

{"name":"B","age":25},

{"name":"C","age":28}

​    ],

 "lobby":  {

"name":"football"

​              }

 }

####JS中解析JSON

格式：var obj = JSON格式数据； /*数据不要给引号*/

案例：

var obj = {

"name":"xx",

"id":10001,

"sex":"男",

"age":18

};

想要解析JSON格式的数据，首先要将JSON格式的数据，转换为JSON格式的对象，然后再操作它的属性。

将string--->JSON

格式1：   var obj = eval('('+JSON格式的字符串+')')；

格式2:    var obj = JSON.parse(JSON格式的字符串);

JSON格式的对象中，取出数据

var value = 对象.key;

var value = 数组[下标];