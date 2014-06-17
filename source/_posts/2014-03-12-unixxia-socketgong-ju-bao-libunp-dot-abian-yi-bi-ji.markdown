---
layout: post
title: "UNIX下Socket工具包libunp.a编译笔记"
date: 2014-03-12 16:23
comments: true
categories: Tec 
---
1.README很重要，基本上跟着README走一遍就妥了，报错的话一定要仔细得去读错误信息，可以对错误的解决提供很大的帮助。我遇到的错误比较简单，是电脑里没有ar的打包指令，因为刚开始接触makefile，所以在这个不大的坑上也花了点时间才发现，还是在别人的机子上编译成功的，一对比才发现自己电脑上没有装...一定不要太自信，当时压根就没往这个方向想。

```c
Execute the following from the src/ directory:
&
./configure    # try to figure out all implementation differences
cd lib         # build the basic library that all programs need
make           # use "gmake" everywhere on BSD/OS systems
&
cd ../libfree  # continue building the basic library
make
&
cd ../libroute # only if your system supports 4.4BSD style routing sockets
make           # only if your system supports 4.4BSD style routing sockets
&
cd ../libxti   # only if your system supports XTI
    make           # only if your system supports XTI
cd ../intro    # build and test a basic client program
make daytimetcpcli
&
./daytimetcpcli 127.0.0.1
&
If all that works, you're all set to start compiling individual programs.
Notice that all the source code assumes tabs every 4 columns, not 8.
```
<!--more-->

2.开始蛮兴奋地在sublime上敲，觉得脱离了IDE后怎么看怎么高大上，但慢慢发现每更改了服务端客户端的程序后，想要运行测试要在三个终端窗口敲五次命令才可以，对于习惯了一个快捷键编译运行的我实在是一场灾难，果断还是要把服务端的开发迁移到eclipse上才是上上之道。因为不熟悉Makefile的语法，对于编译的配置又是一场灾难，折腾了好久才日出来。

首先要配置-L选项，定位于lib所在文件路径

```c
/Users/workingspace/Demo/src/
```
再之后配置-l（小写L）选项，定位到具体的库名，虽然全名为libunp.a，但在配置的时候只写unp就够了,另外还要添加resolv，pthread这两个库

```c
unp
resolv
pthread
```

本来以为到这里就结束了，所以也就开始了悲剧的折腾之路，后来仔细对比了Makefile的显示信息，发现还要对-I（大写i）进行配置

```c
/Users/src/lib
```
这个路径是编译lib时所在的路径，三个配置齐全，用的时候记得include就可以使用了。