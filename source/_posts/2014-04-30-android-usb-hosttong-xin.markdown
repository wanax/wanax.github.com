---
layout: post
title: "Android USB Host通信"
date: 2014-04-30 11:18
comments: true
categories: Tec
---

##开了个头

一部Android手机加上一块Arduino的板子通过USB通信可以实现很多有趣的东西。

2011年Google推出Android开放配件协议AOA及配件开发工具包ADK提供了Android USB或蓝牙进行通信的API，为基于Android系统的智能设备控制外设提供了条件。利用Android，系统可以连接从家用电器到重型机械、机器人等多种设备 [原文](http://www.chinaaet.com/article/216691)。

<!-- more -->

想来这便是Android的魅力所在，开源的Linux系统给了任何人实现自己的梦想的可能。无需再依靠大的厂商提供的一套套完整的服务。仅仅通过自己的想象力配以几块平淡无奇的板子，便能实现一些之前看起来很高大上的东西。

##利其器

1.Android-[ADT](http://developer.android.com/sdk/index.html)

Android的简单开发依赖于工具包SDK和开发工具IDE的使用。Google为我们提供的ADT打包了完善的开发工具：

>Eclipse + ADT plugin

>Android SDK Tools

>Android Platform-tools

>The latest Android platform

>The latest Android system image for the emulator

这里还不得不提的是一个关于Android模拟器的问题。用过eclipse的都知道，上面自带的模拟器速度奇慢，尤其当适应了iPhone开发后对其更是不能忍受。这里有一篇文章便是专门讲解了下为毛Android的模拟器这么慢-[Simulator or Emulator](http://stackoverflow.com/questions/1584617/simulator-or-emulator-what-is-the-difference)。

然后便发现了这个神器-[Genymotion](http://www.genymotion.com/)，感觉这玩意儿比苹果的模拟器还快。

2.[Arduino](http://www.arduino.cc/)

这个板子还是蛮便宜又好用的，官网的文档也全，基本跟着读一遍就知道大体的开发的流程了，提供的IDE有点坑，只能设为9号字体，倒是可以调大，调大的话光标根本对不上...

![image](/images/tec/androidusb/arduino.png)

示例的代码做了个简单的读数据的功能，每次读一行然后原样返回。

开头的setup那里设置了[波特率](http://en.wikipedia.org/wiki/Baud)，大体上波特率跟每秒发送的信息量有关。手上的这块板子经过简单的测试，有如下数据：

>波特率为9600时，每次可以携带129个字节，间隔极限在150ms

>波特率为14400时，每次可以携带191个字节，间隔极限在260ms

个人推测大概越好的板子每次能携带的信息量越大且时间间隔越短吧，即所谓的可以支持很高的波特率。

有了上面这两套东西基本上就可以开始USB通信了。

##幕前

关于详细的实现Google的教程上有个快速简易的介绍，一般跟着做是可以实现效果的-[USB Host and Accessory](http://developer.android.com/guide/topics/connectivity/usb/index.html)。稍微了解过Android开发的看着玩意儿没啥问题。基本上就是在`manifest`里对指定的activity配置下权限就妥了，它还提供了一个`device_filter`的xml文件，可以过滤指定的USB设备。

因为要在Andorid上做开发然后测试与Arduino，所以免不了要先往Android上写程序再让这俩东西连接，老是这样往复的在几台设备间插拔效率太低，Google提供给我们的解决方案是让Android通过Wifi连电脑，然后Android与Arduino一直连着线就妥了。

照着试了几遍，别人的电脑一连就妥，我的死活不行，也不知是不是苹果机的缘故。无奈实在没有插来插去的兴趣，所以又研究了下Genymotion，因为它是依托在VirtualBox上的，总觉得可以一搞。

多次尝试后发现，可以先将Arduino板插上电脑，如下图会出现这个设备，添加虚拟机对它的过滤，这之后把板子拔下来再插回去的话虚拟机也即Android模拟器便可以接收到Arduino板子的信号了。想要对板子连电脑烧程序的话，则反过来，先取消对板子的过滤，插拔一次便连上电脑了。

![image](/images/tec/androidusb/usbbox.png)

具体的实现大体分为四步：[USB Host and Accessory](http://developer.android.com/guide/topics/connectivity/usb/index.html)

####一，Obtaining permission to communicate with a device

先实例化一个`UsbManager`，通过它来获取USB连接的权限。

```java
UsbManager mUsbManager = (UsbManager) getSystemService(Context.USB_SERVICE);
private static final String ACTION_USB_PERMISSION =
    "com.android.example.USB_PERMISSION";
mPermissionIntent = PendingIntent.getBroadcast(this, 0, new Intent(ACTION_USB_PERMISSION), 0);
IntentFilter filter = new IntentFilter(ACTION_USB_PERMISSION);
registerReceiver(mUsbReceiver, filter);
UsbDevice device;
mUsbManager.requestPermission(device, mPermissionIntent);
```
权限获取：
```java
private static final String ACTION_USB_PERMISSION =
    "com.android.example.USB_PERMISSION";
private final BroadcastReceiver mUsbReceiver = new BroadcastReceiver() {
    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();
        if (ACTION_USB_PERMISSION.equals(action)) {
            synchronized (this) {
                UsbDevice device = (UsbDevice)intent.getParcelableExtra(UsbManager.EXTRA_DEVICE);
                if (intent.getBooleanExtra(UsbManager.EXTRA_PERMISSION_GRANTED, false)) {
                    if(device != null){
                      //call method to set up device communication
                   }
                } 
                else {
                    Log.d(TAG, "permission denied for device " + device);
                }
            }
        }
    }
};
```

####二，Enumerating devices

```java
HashMap<String, UsbDevice> deviceList = manager.getDeviceList();
Iterator<UsbDevice> deviceIterator = deviceList.values().iterator();
while(deviceIterator.hasNext()){
    UsbDevice device = deviceIterator.next()
    //your code
}
```

####三，Communicating with a device

```java
private Byte[] bytes
private static int TIMEOUT = 0;
private boolean forceClaim = true;
UsbInterface intf = device.getInterface(0);
UsbEndpoint endpoint = intf.getEndpoint(0);
UsbDeviceConnection connection = mUsbManager.openDevice(device); 
connection.claimInterface(intf, forceClaim);
connection.bulkTransfer(endpoint, bytes, bytes.length, TIMEOUT); //do in another thread
```

####四，Terminating communication with a device

```java
BroadcastReceiver mUsbReceiver = new BroadcastReceiver() {
    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction(); 
      if (UsbManager.ACTION_USB_DEVICE_DETACHED.equals(action)) {
            UsbDevice device = (UsbDevice)intent.getParcelableExtra(UsbManager.EXTRA_DEVICE);
            if (device != null) {
                // call your method that cleans up and closes communication with the device
            }
        }
    }
};
```

需要注意的是两者间的通讯都是以Byte形式完成的，所以作为在接收与发送的时候需要注意相关的转码工作，而且我这边实现的是一行行的读取，所以发送的时候会隐式地拼接上`\n`。

发送时的转码：

```java
private String changeEscapeSequence(String in) {
    String out = new String();
    try {
        out = unescapeJava(in);
    } catch (IOException e) {
        return "";
    }
    out = out + "\n";
    return out;
}	
private String unescapeJava(String str) throws IOException {
	 if (str == null) {
	     return "";
	 }
	 int sz = str.length();
	 StringBuffer unicode = new StringBuffer(4);
	 StringBuilder strout = new StringBuilder();
	 boolean hadSlash = false;
	 boolean inUnicode = false;
	 for (int i = 0; i < sz; i++) {
	     char ch = str.charAt(i);
	     if (inUnicode) {
	         // if in unicode, then we're reading unicode
	         // values in somehow
	         unicode.append(ch);
	         if (unicode.length() == 4) {
	             // unicode now contains the four hex digits
	             // which represents our unicode character
	             try {
	                 int value = Integer.parseInt(unicode.toString(), 16);
	                 strout.append((char) value);
	                 unicode.setLength(0);
	                 inUnicode = false;
	                 hadSlash = false;
	             } catch (NumberFormatException nfe) {
	                 // throw new NestableRuntimeException("Unable to parse unicode value: " + unicode, nfe);
	                 throw new IOException("Unable to parse unicode value: " + unicode, nfe);
	             }
	         }
	         continue;
	     }
	     if (hadSlash) {
	         // handle an escaped value
	         hadSlash = false;
	         switch (ch) {
	             case '\\':
	                 strout.append('\\');
	                 break;
	             case '\'':
	                 strout.append('\'');
	                 break;
	             case '\"':
	                 strout.append('"');
	                 break;
	             case 'r':
	                 strout.append('\r');
	                 break;
	             case 'f':
	                 strout.append('\f');
	                 break;
	             case 't':
	                 strout.append('\t');
	                 break;
	             case 'n':
	                 strout.append('\n');
	                 break;
	             case 'b':
	                 strout.append('\b');
	                 break;
	             case 'u':
	                 {
	                     // uh-oh, we're in unicode country....
	                     inUnicode = true;
	                     break;
	                 }
	             default :
	                 strout.append(ch);
	                 break;
	         }
	         continue;
	     } else if (ch == '\\') {
	         hadSlash = true;
	         continue;
	     }
	     strout.append(ch);
	 }
	 if (hadSlash) {
	     // then we're in the weird case of a \ at the end of the
	     // string, let's output it anyway.
	     strout.append('\\');
	 }
	 return new String(strout.toString());
}
```

接收时的转码：

```java
void setSerialDataToTextView(int disp, byte[] rbuf, int len, String sCr, String sLf) {
    for (int i = 0; i < len; ++i) {
    	mText.append((char) rbuf[i]);
    }
}
```

##正主

其实最后在各种测试的时候没有用Google的这个原始流程，在网上发现了一份比较完整的USB通讯示例流程-[Android-USB-Serial-Monitor-Lite](https://github.com/ksksue/Android-USB-Serial-Monitor-Lite)，直接用它提供的各种工具做的测试...我的测试代码在这里[android_usb_test](https://github.com/wanax/android_usb_test)



































