---
layout: post
title: "UNIX下的Socket通信-Socket的连接"
date: 2014-03-20 15:55
comments: true
categories: Tec
---

##一.关于Socket

如我，刚见这个名词的时候不知所云，因为起点就没找对，首先要在基于对UNIX有所了解的情况下才利于开展工作，便是“一切皆文件”。

在UNIX下，一般的文件可以通过open打开，并返回一个小整数作为标记，后续的操作便是针对这个小整数进行读写。这个小整数被称作*描述符*，描述符只是引用file的proc结构中一个数组的某个元素的下标而已，它可以代表该文件，但并不是文件本身，我们通过它来与文件建立联系，方便操作。

以此为前提，Socket便是一种特殊的文件，通过*socket*函数可以创建一个套接字（特殊的文件），它也返回给我们一个小整数，以后所有的函数调用就用该描述符来标示这个套接字。

在UNIX下文件的种类有很多，打开与使用的方式虽然遵循着一定的模式，但也不尽相同。比如想打开一个文本文件需要借助open函数，而且在打开的时候需要告诉open函数该文本的路径位置，以明确打开的目标。

<!--more-->

Socket的打开也与之相似，因为它一定程度上可以理解为是一个联网通信的文件，所以如果想要明确打开的目标，肯定不能传送一个本地路径，而要与之相符地传送一个目标IP地址。当然这只是简单的比喻，Socket的建立要比文件要复杂一些，IP地址的设置只是打开过程的一个步骤，因为涉及到了网络的缘故，还需要设置很多参数。毕竟文本文件是自己的，打开的标准关上门来自己做主，怎么方便都好商量。而Socket则不一样，既然是在各种主机间通信，且走的路线均不一样，这就牵扯到了各种配置。

##二.TCP下IPv4的Socket连接

###1.socket

让我们来看Socket建立的第一步的函数

```c
int socket(int family, int type, int protocol);
```

明显的，单是第一步就比open函数多参数。

其中family参数指明协议族，type指明套接字类型，protocol为某个协议的常值类型，或者设为0。

####socket函数的type常值

| 		type 		  |        说明		  
| ------------   | ------------- 
| SOCK_STREAM    |    字节流套接字   
| SOCK_DGRAM     |     数据报套接字  
|SOCK_SEQPACKET  |    有序分组套接字 
|   SOCK_RAW     |     原始套接字   

####socket函数protocol常值

| 	 protocol	  |        说明		  
| ------------   | ------------- 
| IPPROTO_TCP    |    TCP传输协议   
| IPPROTO_UDP    |     UDP传输协议  
| IPPROTO_SCTP   |    SCTP传输协议  

所以如果

```c
sockfd = socket(AF_INET, SOCK_STREAM, 0);
```

则表示socket函数创建一个网际（AF_INET）字节流（SOCK_STREAM）套接字。

###2.connect

第一步的socket函数只是把一些基本的参数设置好并返回了一个描述符，就像安好了电话，但其实并没有开始扯线连接，connect帮我们搞定这一步。

```c
int connect(int sockfd, const struct sockaddr *servaddr, socklen_t addrlen);
```

socket是由socket返回的套接字描述符，第二个，第三个参数分别是一个指向套接字地址结构的指针和该结构的大小。套接字地址结构必须含有服务器的IP地址和端口号。

关于sockaddr的结构大体如下：

```c
struct sockaddr_in {
    sa_family_t    sin_family; /* address family: AF_INET */
    in_port_t      sin_port;   /* port in network byte order */
    struct in_addr sin_addr;   /* internet address */
};
```

对此结构一般的设置方法如下

```c
bzero(&servaddr, sizeof(servaddr));
servaddr.sin_family = AF_INET;
servaddr.sin_port = htons(13);//服务器端口
inet_pton(AF_INET, "192.168.105.105", &servaddr.sin_addr);//服务器地址
```

因为具体的连接动作是使用网络字节序的，并不是我们所见到的诸如`192.168.105.105:13`之类的表达，所以需要htons与inet_pton进行相应的转换。*htons()主机到网络短整数，转换二进制端口号*，地址转换函数在地址的文本表达式和它们存放在套接字地址结构中的二进制值之间转换。

如果是写一个客户端的请求程序，那么前两个函数就可以满足要求建立了套接字，下一步的动作便是针对该套接字返回的描述符进行相应的读写便可以了。但要是想做一个服务器端的进程具有监听功能的话则还需要另外下面这三个函数。

###3.bind，listen，accept

>打个简单的比喻，建立TCP连接就好比一个电话系统。socket函数等同于有电话可用。connect要求我们知道对方的电话号码并拨打它。bind函数是在告诉别人你的电话号码，这样他们可以呼叫你。listen是打开电话振铃，这样当有个外来呼叫到来时，你可以听到。accept则相当于接电话。

对于服务器端的socket addr的初始化如下

```c
bzero(&servaddr, sizeof(servaddr));
servaddr.sin_family      = AF_INET;
servaddr.sin_port        = htons(SERV_PORT);
servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
```

它区别于客户端的套接字初始化在于第四行，并没有指定目标IP地址，代之的是一个INADDR_ANY的宏，它可以表示我接收来自任何地址的请求。
如果想要真正实现这一目的，我们需要调用bind来告知UNIX系统我这个套接字将被用来接收信号：

```c
int bind(int sockfd, const stuct sockaddr *myaddr, socklen_t addrlen);
```

继而取消电话的静音状态：

```c
int listen(int sockfd, int backlog);
```

此函数通常应该在调用socket和bind这两个函数之后，并在调用accept之前调用。

关于backlog，是一个设置未完成连接队列与已完成连接队列的参数，这里暂不做细致的探讨了。

最后一步，我们的进程将阻塞于accept的调用中。accept函数由TCP服务器调用，用于从已完成的连接队列头返回下一个已完成连接。关于阻塞函数，看下面这段的讲解：

>永远阻塞的系统调用是指调用有可能永远无法返回，也可称为慢系统调用（slow system call），多数网络支持函数都属于这一种（accept）。适用于慢系统调用的基本规则是，当阻塞于某个慢系统调用的一个进程捕获某个信号且相应信号处理函数返回时，该系统调用可能返回一个EINTR错误。

对于此类错误我们可以通过以下函数进行绕过，

```c
for ( ; ; ) {
		clilen = sizeof(cliaddr);
		if ( (connfd = accept(listenfd, (SA *) &cliaddr, &clilen)) < 0) {
			if (errno == EINTR)
				continue;		/* back to for() */
			else
				err_sys("accept error");
		}
	}
```

以上便是一个服务器端的监听套接字的建立过程。

###4.close，shutdown

有开就有关，关于套接字的关闭我们一般使用close与shutdown来进行控制。

正如我们前面所述，我们通过描述符对套接字进行操作，描述符不过是对file的一个下标而已，如果有多个进程同时使用描述符，则均可以表示为对此file的一个引用。初始的引用值为1，每当调用fork以派生子进程或对打开操作返回的描述符（或其复制品）调用dup以复制描述符时，该file结构的引用计数就递增。相应的close()使相应描述符的引用计数减1，当该描述符引用计数为0时则引发正常TCP连接终止序列：每个方向上发送一个FIN，每个FIN又由各自的对端确认。如想确实在某个TCP连接上发送FIN，可以改用shutdown。

###5.最后无图无真相:

![image](/images/tec/Socket/funtime.jpg)

##下一步

这部分具体介绍了Socket的基本概念与一个客户端和服务端的套接字是怎样一步步建立起来的。但文章中也出现了一些诸如FIN之类的关键字并没有进行详细的解释，在下一章，我会对TCP/IP的一些基本概念进行解释。
































