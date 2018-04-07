### 网络常识

> 什么是计算机网络？

分布在不同区域下的计算机，通过硬件等网络设备使用通信线路互联成一个网络系统。

计算机网络，可以让我们很方便的进行信息传递，共享硬件软件资源等等！

> 接入网流程

联通、电信宽带—》 光纤猫 —〉路由器 —〉分线 —》数字信号

互联网 家    开发板（淘宝）

外部的网线插在交换机上：交换机类似一台电脑

![](/Users/lijingwan/Documents/img/小型的局域网.png)

网卡和ip地址很少重复。

网卡地址：MAC地址，标志了网卡的唯一性

IP地址：标志在局域网中的唯一性

![](/Users/lijingwan/Documents/img/公司局域网.png)

访问不同部门的电脑，比较困难。因为交换机有过滤的作用。

每个交换机也有自身的ip，每个交换机的ip地址不同。（注意：仅仅指的是在父交换机连接的区域内不同！）

只有在一个局域网下，ip地址相同会产生冲突！

访问的网页其实就是一个共享的文件。

防火墙 翻墙

外挂：客户端在欺骗服务器。

dns解析商：将ip和地址进行映射。当输入一个地址，先去找解析商，通过映射关系映射出一个ip，再帮你去访问这个ip。

> 什么是IP和域名？

==IP地址是计算机的唯一标识, 就像人的身份证号码一样! 有一些特殊的字符,可以代替本机IP进行输入!==

###### 如: localhost :本机ip

###### 127.0.0.1: 本机ip

域名就是ip地址的别名，通过dns服务商进行了映射绑定！

通过解析商，可以将域名和一个ip进行绑定，一台计算机有多个别名。

> 什么是网络通信协议

控制计算机之间如何交流的一套标准。对数据的传输的速率v、传输代码、传输接口、传输步骤以及出错控制等指定的一组标准。

> 常见的计算机通信协议：

1. http：超文本传输协议（网页的协议）
2. https： 安全的超文本传输协议（有证书，会卡一点）
3. ftp：文件传输协议
4. tcp：传输控制协议
5. udp：用户数据报协议

> 什么是端口号？

与ip地址很像，ip是用来区分计算机的

端口号是用来在一个电脑中区分不同的程序的！

端口号的范围：0-65535（2个字节）

原理：在进行数据的发送时，可以指定ip：端口号，来确定数据发送到电脑中的某个端口号中！

每个程序都可以选择自己要占用的端口号，一个端口号只允许一个程序同时占用。

>  端口号分类

1. 公认端口/常用端口：0-1024    被系统知名服务占用

		例如：http：80    https：443    ftp：21

2. 注册端口：1025-49151
3. 动态/私有端口：49152-65535

> 数据是如何通过ip+端口号传入另一台计算机的？

OSI模型：（应用层就是软件，物理层：网卡）

![](/Users/lijingwan/Documents/img/osi模型.png)

> 网络编程程序分类

1. B/S  浏览器与服务器，使用浏览器访问的程序，不需要安装客户端，更新方便

    传输的协议是公开的， 很容易被抓包，不安全隐患

   只需要编写一份服务器代码

2. C/S  客户端与服务器，需要开发2套软件。用户需要更新版本，否则使用bug版本。

     协议可以自己定制。数据安全。修改bug需要维护2套，成本高，不可控。

>  注意

​	以后我们的工作，更多的是在进行 B/S中的S开发，Server

​	今天所学习的是 C/S程序  ==C/S==程序中：1. TCP程序	2.UDP程序

​	在Java中，有时网络编程又被称为Socket编程。



### TCP程序

进行TCP程序的开发，需要设计服务器端和客户端！

服务器端：使用ServerSocket 与 Socket进行编写

客  户  端：使用Socket进行编写

#### Socket类 

==socket==：两台计算机通信的端点，是网络驱动提供给应用程序编程的一种接口和一种机制！

> 构造方法

​	两参构造方法: Socket(String ip, int port) ;

​	创建一个Socket对象, 用于指定 ip 下port 端口号的ServerSocket进行连接 ,  并通信 !

> 常用方法

	1. OutputStream  getOutputStream : 得到向通信的另一端点的输出流
	2. InputStream  getInputStream : 得到通信的另一短点的输入流

==Tips:== 

- 如果网络编程传文件,建议使用字节流. 如果传一段文字,建议转化为字符流再传输(PrintWriter BufferedReader)


- 在任何的网络编程中,使用输入流读取客户端或者服务器发送过来的数据时,其实是在等待数据发送并读取!  会造成线程的阻塞( IO )

3.  close( ) :  关闭套接字(相当于断开连接)

> 注意

​	输入输出流的使用:

​		==输入与输出 只是相对的概念 !==

​		例如: 客户端向服务器发送数据:

​				对于客户端来说:  属于输出

​				对于服务器来说:  属于输入

```java
package tcpdemo1;

import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.net.ServerSocket;
import java.net.Socket;

public class Demo1 {

	public static void main(String[] args) throws Exception {
		/**
		 * 创建一个tcp服务器 参数1 要监听的端口号
		 */
		ServerSocket server = new ServerSocket(8099);
		// 这个方法会导致线程的阻塞(连上以后,程序才会往下执行)
		// 等待客户端连接,客户端连接后,会创建一个与客户端进行通信的socket对象
		while(true) {
			System.out.println("正在等待客户端的连接");
			Socket socket = server.accept();
			System.out.println("接收到了客户端的连接" + socket.getInetAddress().getHostName());
			
			//通过与客户端通信的socket,获取输入流,获取用户传过来的内容
			InputStream is = socket.getInputStream();
			//将指向客户端的输入流,转换为了逐行读取流
			BufferedReader br = new BufferedReader(new InputStreamReader(is));
			String text = br.readLine();
			System.out.println("接收到客户端发送的消息:"+ text);
			br.close();
			socket.close(); 
		}
	}

}

```



#### ServerSocket类

ServerSocket用来侦听一个ip地址与端口号下的数据，等待客户端(Socket)的连接

等待连接后，可以获取到一个用来进行通信的Socket对象

> 构造方法

​	一参构造方法：ServerSocket( int port ) : 创建一个TCP程序的服务器，并侦听档期计算机IP下的指定port端口号

> 常用方法

Socket	accept( ) : 此方法会造成线程的阻塞(IO)，等待客户端的连接， 直到客户端连接，线程才会继续行!

```java
package tcpdemo2;

import java.io.IOException;
import java.io.OutputStream;
import java.io.PrintWriter;
import java.net.Socket;
import java.net.UnknownHostException;
import java.util.Scanner;

public class Demo2 {
	    public static Scanner input = new Scanner(System.in);
	public static void main(String[] args) throws UnknownHostException, IOException {
		
		Socket client = new Socket("localhost",8099);
		System.out.println("客户端已经连接到了服务器!");
		System.out.println("请输入要发送给服务器的消息内容:");
		//获取到用户输入的内容
		String message = input.nextLine();
		//获取向服务器输出的流
		OutputStream os = client.getOutputStream();
		//将这个流转换为字符打印流,并向服务器打印一句话
		PrintWriter pw = new PrintWriter(os );
		pw.println(message);
		pw.flush();
		pw.close();
		client.close();
	}

}

```

因为流的读取操作, 会导致线程的阻塞, 客户端与服务器一定要避免同时读取数据!

#### Tcp引入多线程 服务器与多用户交流





