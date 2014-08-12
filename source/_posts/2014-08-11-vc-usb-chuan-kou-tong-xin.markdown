---
layout: post
title: "VC USB 串口通信"
date: 2014-08-11 11:03
comments: true
categories: Tec
---

##楔子

前段时间搞完了聊天服务器后也没有很多事情，因为系统瓶颈停留在数据库读写层面，所以研究了一两周的NOSQL和Cache。NOSQL快是快但key-value型的数据结构实在太过简单，感觉真用起来并不是很方便，Cache倒相对来说可行一些，开多一个服务罢了，其实简单的逻辑自己也能实现。就这样纠结来纠结去的，发现其实现在的服务倒也完全够用，压力测试了下，能撑到1500的并发，想来刚上线估计也最多几百号人能同时聊天就不错了，所以就先研究至此，撑不住了再换，不要过早优化嘛。

<!--more-->

就这样决定先找别的事做做，恰好项目里有需要用到Flash的ANE技术来实现Flash与硬件通信，Flash自然不会写，但用VC进行串口通信打包成DLL给Flash调用这个还是可以研究下的。

ANE这里就不再细谈，大体上就是Flash中调用其它语言的一种技术，通过它可以实现一些Flash做不到的东西，比如说底层通信啥的。这里主要讲USB串口通信这一块儿。

串口通信翻了下书发现还是有很多标准的，而现行最流行的就是USB了，所以在网上关于此的资料还是很好找的。

这篇文章的内容主要由三个部分组成：

1.  串口连接的建立与简单阻塞通讯
2.  通过API调用自动查询当前插入的USB设备并建立连接
3.  区别于简单阻塞的通讯方式，如重叠IO与事件通知

##一，建立连接与阻塞通讯

在WIN下面，对于串口一类的硬件设备也是认定为文件的一种，所以也可以使用文件的方式来创建打开使用：

```c
HANDLE h_com = CreateFile((LPCWSTR)"COM4",   //设备名
					GENERIC_READ || GENERIC_WRITE, //访问模式，可同时读写
					0,                             //共享模式，0表示不共享
					NULL,                          //安全性设置，一般使用NULL
					OPEN_EXISTING,                 //该参数表示该设备必须存在否则创建失败，串口通讯需此设置
					FILE_ATTRIBUTE_NORMAL
					0);
```

首先初始化一个句柄`h_com`，让它指向所要打开设备的串口号，之后对该设备的所有操作均需通过该句柄来执行。

之后便是对此串口的一些基本设置，

####*超时选项*

```c
/*设置串口的超时时间，均设为0，表示不使用超时限制*/
COMMTIMEOUTS  CommTimeouts;
CommTimeouts.ReadIntervalTimeout = 0;
CommTimeouts.ReadTotalTimeoutMultiplier = 0;
CommTimeouts.ReadTotalTimeoutConstant = 0;
CommTimeouts.WriteTotalTimeoutMultiplier = 0;
CommTimeouts.WriteTotalTimeoutConstant = 0;
SetCommTimeouts(h_com, &CommTimeouts);
```

####*DCB参数配置*

因为DCB结构体中选项较多，故一般的配置方式是先通过`GetCommState`获取默认的DCB配置选项，再根据个人需求进行相应的修改。

```c
/*将ANSI字符串转换为UNICODE字符串*/
DWORD dwNum = MultiByteToWideChar(CP_ACP, 0, szDCBparam, -1, NULL, 0);
wchar_t *pwText = new wchar_t[dwNum];
if (!MultiByteToWideChar(CP_ACP, 0, szDCBparam, -1, pwText, dwNum))
{
	bIsSuccess = TRUE;
}
/*获取当前串口配置参数，并且构造自定义DCB参数*/
bIsSuccess = GetCommState(h_com, &dcb) && BuildCommDCB(pwText, &dcb);
/*开启RTS flow控制*/
dcb.fRtsControl = RTS_CONTROL_ENABLE;
dcb.fRtsControl = RTS_CONTROL_ENABLE;
dcb.fBinary = TRUE;
dcb.fParity = TRUE;
dcb.ByteSize = 8;
dcb.Parity = ODDPARITY;
dcb.StopBits = ONESTOPBIT;
delete[] pwText;
SetCommState(h_com, &dcb);
```

####*清空缓冲区*

对句柄进行了一系列操作，保险起见清空一下缓冲区。

```c
PurgeComm(h_com, PURGE_RXCLEAR | PURGE_TXCLEAR | PURGE_RXABORT | PURGE_TXABORT);
```

如此，一个完整的串口句柄就构造完毕了。接下来是进行阻塞读的测试。

首先，我们可以使用`ClearCommError`这个方法获取当前读缓冲区里的数据大小，通过轮询的方式阻塞在这里，当缓冲区里有数据的时候我们再进行下一步的动作。

```c
int get_dirty_len() {
	DWORD dwError = 0;  
	COMSTAT  comstat;   
	memset(&comstat, 0, sizeof(COMSTAT));
	UINT BytesInQue = 0;
	if (ClearCommError(h_com, &dwError, &comstat)) {
		BytesInQue = comstat.cbInQue;
	}
}
```

当有数据到来便可以直接读取了：

```c
while(h_comm->is_working) {
	int len = get_dirty_len();
	if (len == 0) {
		Sleep(SLEEP_TIME_INTERVAL);
		continue;
	}
	char recv[10];
	int recv_len;
	ReadFile(h_com, recv, len, &recv_len, NULL);
}
```

写与读类似

```c
WriteFile(h_com, send, len, &send_len, NULL);
```

如上，一个简单的阻塞串口读写Demo便完成了。

##二，检测查询插入的USB设备信息

因为是Flash直接对DLL调用进行设备连接，所以在C层面需要主动获取连接上的端口名建立连接，并将连接设备的信息返回给Flash调用者，故需要使用另一组工具完成任务。

在WIN下面有一组API可以完成此项功能：

```c
SetUpAPI
```

摸索`setupapi`的使用方法着实费了些力气，归结起来主要有两点

1. 英文阅读能力不足，MSDN上面说明的使用方法并没有深刻理解
2. 整体知识把握不足，摸着石头过河，边试边蒙

说到底还是要多看书，多读英文资料，大体结构把握了也知道从何下手，而且国内博客的资源大多抄来抄去，并没有多关注其所以然，代码贴来贴去，试着心烦远不如一点一点学起来得畅快。

言归正传，获取端口信息依旧要从句柄入手。

####*1.借助API获取某一类设备的相关信息*

```c
HDEVINFO hDevInfo = SetupDiGetClassDevsA(
		(LPGUID)&GUID_DEVCLASS_PORTS,
		0,
		0,
		DIGCF_PRESENT);
```

关于`LPGUID`可以自己初始化，一般的普通设备系统也提供了宏定义，可以看到我这里使用了`GUID_DEVCLASS_PORTS`，指明需要获取的是`PORT`（COM端口）一类的设备信息。

####*2.遍历连接设备，获取设备信息*

```c
char szBuf[MAX_PATH];
ZeroMemory(szBuf, MAX_PATH);
int i = 0;
SP_DEVINFO_DATA   spDevInfoData = { sizeof(SP_DEVINFO_DATA) };
for (i = 0; SetupDiEnumDeviceInfo(hDevInfo, i, &spDevInfoData); i++) {
	char *id = (char *)malloc(10 * sizeof(char));
	char *port = (char *)malloc(10 * sizeof(char));
	//get ID
	if (SetupDiGetDeviceInstanceId(hDevInfo, 
		&spDevInfoData, (PWSTR)szBuf, MAX_PATH, NULL)) {
		char dest[MAX_PATH];
		ZeroMemory(dest, MAX_PATH);
		get_str(szBuf, dest);
		get_id(dest, id);
	}
	ZeroMemory(szBuf, MAX_PATH);
	//get port
	if (SetupDiGetDeviceRegistryProperty(hDevInfo, 
		&spDevInfoData, SPDRP_FRIENDLYNAME, NULL, (PBYTE)szBuf, MAX_PATH, NULL)) {
		char dest[MAX_PATH];
		ZeroMemory(dest, MAX_PATH);
		get_str(szBuf, dest);
		get_com(dest, port);
	}
	strcpy(coms[i], port);
	strcpy(vids[i], id);
	*len = i;
}
```

```c
SetupDiEnumDeviceInfo
```

这个函数通过`Index`进行依次查找设备，若设备不存在则返回空终止遍历。

所以在最后我们可以通过初始化的句柄`hDevInfo`与遍历出来的设备信息`spDevInfoData`来查找我们想要获取的具体信息。

##三，重叠IO与事件通知

###重叠IO

重叠IO解决的是受IO性能的影响不能第一时间将预期的字节数全部读完必须时阻塞等待在`ReadFile`，待全部数据传输完毕才返回的问题。

简单说来便是假如我想读1000个字节的数据从串口设备，但当前传输速度只有100K/S，那岂不是要在`ReadFile`上阻塞10s才会返回？正常情况下是这样的，而重叠IO便是解决此一问题的正确方法。

它的大体思路是当主线程第一时间没有从`ReadFile`中读出预期的字节数后便立即返回`FALSE`，继续其它操作，另一方面会在后台单开一个线程执行读取操作，真正读取完毕后再返回主线程进行相应的逻辑处理。

下面看详细步骤：

####*1.句柄设置为可重叠模式*

```c
HANDLE h_com = CreateFileA(port, 
		GENERIC_READ | GENERIC_WRITE, 
		0,                           
		NULL,                         
		OPEN_EXISTING,                
		FILE_ATTRIBUTE_NORMAL | FILE_FLAG_OVERLAPPED/*可重叠模式*/,
		0);
```

####*2.使用可重叠模式进行读取*

```c
DWORD dwRes, deRead;
char cRecved[100];
int BytesRead;
OVERLAPPED ol;
ol.Offset = 0;
ol.OffsetHigh = 0;
ol.hEvent = CreateEvent(NULL, TRUE, FALSE, NULL);
if (ReadFile(m_hComm, cRecved, len, &BytesRead, &ol)) {
	//success!
} else { //无法第一时间读出数据
	dwRes = WaitForSingleObject(ol.hEvent, 5000);//设置5s超时
	if (dwRes == WAIT_OBJECT_0) {
		if (!GetOverLappedResult(h_com, &ol, &dwRead, TRUE)) {
			//操作失败，使用GetLastError获取失败信息
		} else {
			//操作成功，数据读出并存入cRecved数组中
		}
	}
}
```

这便是一个可重叠IO的简单示例，可以看出当`ReadFile`返回`FALSE`后，我们通过`OVERLAPPED`结构体获取该操作的事件，并通过`WaitForSingleObject`来等待异步线程读操作的完成，然后通过`GetOverLappedResult`验证下最终读取的字节数，便算完成了。

###事件通知

在一般的通信情景中，大多是有个线程一直等待消息的到来然后进行读取。在开篇的例子中我使用的是简单的睡眠轮询的方式，但在真实地应用场景中这样做显然是不符合实际的，于是便需要使用基于事件通知的方式。

既然是事件监听，那第一步需要做的便是添加事件监听事件，这一步是通过对句柄的设置来完成的：

```c
SetCommMask(h_com, EV_RXCHAR | EV_TXEMPTY);
```

`EV_RXCHAR`表示一旦有字节到来便触发事件，`EV_TXEMPTY`表示缓冲区为空的时候触发事件。

接下来便是对此事件进行监听，并在事件到来时将数据读出：

```c
	while (is_running) {
		COMSTAT ComStat;
		DWORD dwRes, dwMask;
		ZeroMemory(myChar, 10);
		OVERLAPPED ol;
		//创建等待事件
		ol.hEvent = CreateEvent(NULL, TRUE, FALSE, NULL);
		WaitCommEvent(h_com, &dwMask, &ol);
		//对该事件进行等待
		dwRes = WaitForSingleObject(ol.hEvent, 1000000);
		if (dwRes == WAIT_OBJECT_0) {
			if (dwMask & EV_RXCHAR) {
				rol.hEvent = CreateEvent(NULL, FALSE, FALSE, NULL);
				char str[20];
				ZeroMemory(str, 20);
				DWORD dwRead;
				DWORD dwErrors;
				COMSTAT Rcs;
				int i;
				//获取缓冲区字节数量
				ClearCommError(h_com, &dwErrors, &Rcs);
				if (Rcs.cbInQue > 0) {
					if (ReadFile(h_com, &str, Rcs.cbInQue, &dwRead, &rol)) {
						printf("%d-%d-%d\n", Rcs.cbInQue, dwRead, str[0]);
					}
					else {
						printf("error\n");
					}
				}
			}
		}
		PurgeComm(h_com, PURGE_RXCLEAR | PURGE_TXCLEAR | PURGE_RXABORT | PURGE_TXABORT);
	}
```

以上便是事件触发式读取的基本使用方法，需要注意的是，事件触发IO可以搭配重叠IO进行使用，以获得更好的效果。



























