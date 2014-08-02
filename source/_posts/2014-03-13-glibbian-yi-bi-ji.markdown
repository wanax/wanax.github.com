---
layout: post
title: "GLib编译笔记"
date: 2014-03-13 15:38
comments: true
categories: Tec
---

###缘起

最近做的一个项目是使用C写的，随着项目的深入发现需要通过不同的数据结构来完成需求，自己简单地去编写已经不能满足要求且在健壮性方面也存在着隐患，于是就琢磨着在网上找找看看有没有什么现成的C工具包来用，就这样，发现了GLib。

先来看下维基上的介绍：

>GLib是一个跨平台的、用C语言编写的库，起初是GTK+的一部分，但到了GTK+第二版，开发者决定把跟图形界面无关的代码分开，这些代码于是就组装成了GLib。GLib提供了多种高级的数据结构，如内存块、双向和单向链表、哈希表、动态字符串等。

感觉功能刚好满足需要，类似于C++中的STL，果断搞起。

<!--more-->

###1.下载

从网上搜索GLib找到了这里[Linux From Scratch](http://www.linuxfromscratch.org/blfs/view/svn/general/glib2.html)，下载它的压缩包。

这个网站也把具体的编译安装方式写得很明白。

>###GLib Dependencies
####Required
[libffi-3.0.13](http://www.linuxfromscratch.org/blfs/view/svn/general/libffi.html) and [Python-2.7.6](http://www.linuxfromscratch.org/blfs/view/svn/general/python2.html)
####Recommended
[PCRE-8.34](http://www.linuxfromscratch.org/blfs/view/svn/general/pcre.html)(built with Unicode properties)
####Optional
[attr-2.4.47](http://www.linuxfromscratch.org/blfs/view/svn/postlfs/attr.html), [D-Bus-1.8.0](http://www.linuxfromscratch.org/blfs/view/svn/general/dbus.html) (required to run the tests), and [GTK-Doc-1.20](http://www.linuxfromscratch.org/blfs/view/svn/general/gtk-doc.html)
####Additional Runtime Dependencies
Quoted directly from the INSTALL file: “Some of the mimetype-related functionality in GIO requires the update-mime-database and update-desktop-database utilities”, which are part of [shared-mime-info-1.2](http://www.linuxfromscratch.org/blfs/view/svn/general/shared-mime-info.html) and [desktop-file-utils-0.22](http://www.linuxfromscratch.org/blfs/view/svn/general/desktop-file-utils.html), respectively.

就是说想要安装GLib的话最起码得先把Required和Recommended里的东西装完才行。

但在实际安装过程中还需要另外两个工具[pkg-config](http://www.chenjunlu.com/2011/03/understanding-pkg-config-tool/)和[Gettext](http://zh.wikipedia.org/wiki/Gettext)

对于两者皆是去官网下载最新版本的安装包，执行安装命令就可以了。

[pkg-config](http://pkgconfig.freedesktop.org/releases/)

```c
./configure  --with-internal-glib
make
sudo  make install
```

[Gettext](https://www.gnu.org/software/gettext/)

```c
$ ./configure
$ make
$ make verify   # (optional)
$ sudo make install
```

###2.编译

安装环境配好后就可以正式安装了，顺着那个网页往下走就能看到具体的方法：

```c
./configure --prefix=/usr --with-pcre=system && make
```
再然后

```c
sudo make install
```
GLib在gcc里的使用是借助了一个叫pkg-configure的工具，这玩意儿的作用这位哥儿们说得很详细了，看他的介绍吧：[理解 pkg-config 工具](http://www.chenjunlu.com/2011/03/understanding-pkg-config-tool/)

其实定位到`/usr/lib/pkgconfigure`后会发现它里面放了很多你自己安装的库的.pc的配置文件，用vim随便打开一个发现它的内容是这样的：

```c
   prefix=/usr
   exec_prefix=${prefix}
   libdir=${exec_prefix}/lib
   includedir=${prefix}/include 
&
   glib_genmarshal=glib-genmarshal
   gobject_query=gobject-query
   glib_mkenums=glib-mkenums 
&
   Name: GLib
   Description: C Utility Library
   Version: 2.38.2
   Requires.private: libpcre
   Libs: -L${libdir} -lglib-2.0 -lintl
   Libs.private:   -lpcre  -lintl  -liconv
   Cflags: -I${includedir}/glib-2.0 -I${libdir}/glib-2.0/include
```
看一下就明白了，基本上就是帮你写了一堆编译选项，免得到时候自己编译的时候要敲一坨，而且在不同的电脑上编译的时候也可以动态的调整路径，不用一遍遍的去改Makefile。

比如说自己编译一个使用了GLib的文件，可以这样敲：

```c
gcc `pkg-config --cflags --libs glib-2.0`  hello.c -o hello
```

–cflags 参数可以给出在编译时所需要的选项，而 –libs 参数可以给出连接时的选项。

当然如果不使用pkg-configure的话也可以类似这样子：

```c
gcc  -L/usr/lib -I/usr/lib/glib/include/glib-2.0 hello.c -o hello -liconv -lresolv -lpcre -lintl -lglib-2.0
```
所以基本上还是简化了很多操作的。

###3.测试使用

测试一下GLib中哈希表的使用吧：

```c
#include <stdio.h>
#include <glib.h>
int main(){
		printf("Glib version: %u.%u.%u\n\n",glib_major_version,glib_minor_version,glib_micro_version);
&
		GHashTable* hash = g_hash_table_new(g_str_hash, g_str_equal);
		g_hash_table_insert(hash, "Virginia", "Richmond");
		g_hash_table_insert(hash, "Texas", "Austin");
		g_hash_table_insert(hash, "Ohio", "Columbus");
&
		printf("There are %d keys in the hash\n",g_hash_table_size(hash));
		printf("The capital of Texas is %s\n",g_hash_table_lookup(hash, "Texas"));
		gboolean found = g_hash_table_remove(hash, "Virginia");
		printf("The value 'Virginia' was %sfound and removed\n", found ? "" : "not ");
&
		g_hash_table_destroy(hash);
		return 0;
}
```
可以打印如下信息：

>Glib version: 2.38.2

>There are 3 keys in the hash

>The capital of Texas is Austin

>The value 'Virginia' was found and removed

###4.搭配Eclipse

嫌敲命令行麻烦的话也可以配到Eclipse上用。现在在Eclipse上有了pkg-configure的插件，可以对GLib直接勾选使用，但我安装后没有起到效果，不知道哪里出了问题[pkg-config-support-for-eclipse-cdt](https://code.google.com/p/pkg-config-support-for-eclipse-cdt/)。但反正原理已经知道了，索性自己配一下也就妥了。

选中工程，在Project里找到Properties进入到下图的界面：

![image](/images/tec/GLib/eclipsecon.png)

在Libraries里配置GLib库的位置和名字，在Includes里配置GLib头文件的位置，我的头文件在这里

```c
/usr/include/glib-2.0
```
其实按照教程正常安装的话基本上都会在这个位置，这样的话就可以用Eclipse在GLib里爽起来了~