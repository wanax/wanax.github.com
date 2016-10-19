---
layout: post
title: "iOS图形原理与离屏渲染"
date: 2016-10-19 20:28
comments: true
categories: Tec
---

*Outline*:

一，[iOS图形显示基本知识](#anchor1.0)

------ 1.1 [图形显示原理](#anchor1.1)

------ 1.2 [iOS的显示架构](#anchor1.2)

------ 1.3 [iOS图形显示流程](#anchor1.3)

------ 1.4 [补充知识](#anchor1.4)

---------- 1.4.1 [图像多层次的合成--为何设置透明会增加GPU工作量](#anchor1.4.1)

---------- 1.4.2 [图层对其--为何图片缩放会增加GPU工作量](#anchor1.4.2)

---------- 1.4.3 [关于卡顿的简单原理解释](#anchor1.4.3)

二，[iOS中离屏渲染相关知识](#anchor2.0)

------ 2.1 [什么是离屏渲染](#anchor2.1)

------ 2.2 [为什么要谨慎避免离屏渲染](#anchor2.2)

------ 2.3 [离屏渲染与光栅化](#anchor2.3)

------ 2.4 [一些触发离屏渲染的基本case与替换方式](#anchor2.4)

三，[iOS的一些显示性能检测方法的简单介绍](#anchor3.0)

------ 3.1 [显示性能检测一些小Tip](#anchor3.1)

------ 3.2 [Instruments - Core Animation](#anchor3.2)

------ 3.3 [Instruments - OpenGL ES](#anchor3.3)

四，[AsyncDisplayKit分享](#anchor4.0)

------ 4.1 [AsyncDisplayKit是干什么的](#anchor4.1)

------ 4.2 [AsyncDisplayKit的一些性能上的优势](#anchor4.2)

###[一，iOS图形显示基本知识](id:anchor1.0)

从一个像素点到真正显示在屏幕上，iOS到底在里面做了哪些工作，涉及到哪些Frameworks与Libraries？这是这一章想搞明白的问题。

<!--more-->

####1.1 [图形显示原理](id:anchor1.1)

图像想显示到屏幕上使人肉眼可见都需借助像素的力量。简单地说，每个像素由红，绿，蓝三种颜色组成，它们密集的排布在手机屏幕上，将任何图形通过不同的色值表现出来。

计算机显示的流程大致可以描述为将图像转化为一系列像素点的排列然后打印在屏幕上，由图像转化为像素点的过程又可以称之为[光栅化](https://www.zhihu.com/question/29163054)，就是从矢量的点线面的描述，变成像素的描述。

![image](/images/tec/offscreen/offscreen011.png =250x)

回溯历史，可以从过去的 CRT 显示器原理说起。CRT 的电子枪按照上面方式，从上到下一行行扫描，扫描完成后显示器就呈现一帧画面，随后电子枪回到初始位置继续下一次扫描。为了把显示器的显示过程和系统的视频控制器进行同步，显示器（或者其他硬件）会用硬件时钟产生一系列的定时信号。当电子枪换到新的一行，准备进行扫描时，显示器会发出一个水平同步信号（horizonal synchronization），简称 HSync；而当一帧画面绘制完成后，电子枪回复到原位，准备画下一帧前，显示器会发出一个垂直同步信号（vertical synchronization），简称 VSync。显示器通常以固定频率进行刷新，这个刷新率就是 VSync 信号产生的频率。尽管现在的设备大都是液晶显示屏了，但原理仍然没有变。[^1]

如在 iPhone5 的上就有1,136×640=727,040个像素，而在15寸Retain的MBP上，这一数字达到15.5百万以上，当你滚动整个屏幕的时候，数以百万计的颜色单元必须以每秒60次的速度刷新，计算量可想而知。

####1.2 [iOS的显示架构](id:anchor1.2)

从软件层面上，iOS借助`Core Graohics`，`Core Animation`，`Core Image`完成图形的处理，它们又都是借助`OpenGL ES`来完成底层的工作，其结构如下图所示：

![image](/images/tec/offscreen/arch01.png =500x)

Display 的上一层便是图形处理单元 GPU，GPU 是一个专门为图形高并发计算而量身定做的处理单元。这也是为什么它能同时更新所有的像素，并呈现到显示器上。它并发的本性让它能高效的将不同纹理合成起来。因为涉及到各种图形矩阵的计算，它跟CPU最直观的区别在于浮点计算能力要超出CPU很多。所以在开发中，**我们应该尽量让CPU负责主线程的UI调动，把图形显示相关的工作交给GPU来处理**，因为涉及到光栅化等一些工作时，CPU也会参与进来，这点在后面再详细描述。

`GPU Driver` 是直接和 GPU 交流的代码块。不同的GPU是不同的性能怪兽，但是驱动使他们在下一个层级上显示的更为统一，典型的下一层级有 OpenGL/OpenGL ES.

`OpenGL`(Open Graphics Library) 是一个提供了 2D 和 3D 图形渲染的 API。GPU 是一块非常特殊的硬件，OpenGL 和 GPU 密切的工作以提高GPU的能力，并实现硬件加速渲染。

OpenGL 之上扩展出很多东西。在 iOS 上，几乎所有的东西都是通过 Core Animation 绘制出来，然而在 OS X 上，绕过 Core Animation 直接使用 Core Graphics 绘制的情况并不少见。[^2]

在硬件层面的调度我们可以看下图所示：

![image](/images/tec/offscreen/arch03.png =500x)

计算机系统中 CPU、GPU、显示器是以上面这种方式协同工作的。CPU 计算好显示内容提交到 GPU，GPU 渲染完成后将渲染结果放入帧缓冲区，随后视频控制器会按照 VSync 信号逐行读取帧缓冲区的数据，经过可能的数模转换传递给显示器显示。

在最简单的情况下，帧缓冲区只有一个，这时帧缓冲区的读取和刷新都都会有比较大的效率问题。为了解决效率问题，显示系统通常会引入两个缓冲区，即双缓冲机制。在这种情况下，GPU 会预先渲染好一帧放入一个缓冲区内，让视频控制器读取，当下一帧渲染好后，GPU 会直接把视频控制器的指针指向第二个缓冲器。如此一来效率会有很大的提升。

####1.3 [iOS图形显示流程](id:anchor1.3)

我们可以再从上层看一下iOS中不同的Frameworks和Libraries之间的一些联系：

![image](/images/tec/offscreen/offscreen06.png =500x)

在最顶层的就是UIKit，一个在iOS中用来管理用户图形交互的Objc高级的框架，它由一系列的集合类构成，例如UIButton、UILabel，每一个都负责他们指定的UI Control角色。UIKit本身构建在一个叫Core Animation的框架之上。最后一部分是Core Graphics，曾经在Quartz（一个基于CPU的绘制引擎，在OS X系统上初次露脸）中被引入。这两个较为底层的框架都是用C语言编写的。

我们经常说到的硬件加速其实是指OpenGL,Core Animation/UIKit基于GPU之上对计算机图形合成以及绘制的实现，直到目前为止，iOS上的硬件加速能力还是大大领先与android，后者由于依赖CPU的绘制，绝大多数的动画实现都会让人感觉明显的卡顿[^3]

CoreAnimation的渲染流程可以用下图来概括:

![image](/images/tec/offscreen/offscreen07.png =500x)

在GPU的渲染过程中,我们能看到顶点着色器与像素着色器参与到图像的处理。

在objc.io中有一篇文章进一步地阐明了顶点着色器与像素着色器 ([GPU 加速下的图像处理](http://objccn.io/issue-21-7/))[^4]

####1.4 [补充知识](id:anchor1.4)

#####1.4.1 [图像多层次的合成--为何设置透明会增加GPU工作量](id:anchor1.4.1)[^5]

*合成 | Blended*

在图形世界中，合成是一个描述不同位图如何放到一起来创建你最终在屏幕上看到图像的过程。

一个不透明的红色盖在蓝色上那我们看到的就是一个蓝色，但一个半透明的红色盖在蓝色让我们得到的却是一个紫色，这便是合成所要做的工作。

我们可以用下面这个公式来计算每一个像素：

	R = S + D * ( 1 – Sa )
	
结果的颜色是源色彩(顶端纹理)+目标颜色(低一层的纹理)*(1-源颜色的透明度)。在这个公式中所有的颜色都假定已经预先乘以了他们的透明度。

假定两个纹理都完全不透明，比如 alpha=1.如果目标纹理(低一层的纹理)是蓝色(RGB=0,0,1)，并且源纹理(顶层的纹理)颜色是红色(RGB=1,0,0)，因为 Sa 为1，所以结果为：

	R = S
	
如果源颜色层为50%的透明，比如 alpha=0.5，既然 alpha 组成部分需要预先乘进 RGB 的值中，那么 S 的 RGB 值为(0.5, 0, 0)，公式看起来便会像这样:

![image](/images/tec/offscreen/arch04.png =800x)

所以当源纹理是完全不透明的时候，目标像素就等于源纹理。这可以省下 GPU 很大的工作量

这也是为什么 CALayer 有一个叫做 opaque 的属性了。如果这个属性为 NO，GPU 将不会做任何合成，而是简单从这个层拷贝，不需要考虑它下方的任何东西(因为都被它遮挡住了)。

#####1.4.2 [图层对齐--为何图片缩放会增加GPU工作量](id:anchor1.4.2)

当所有的像素是对齐的时候我们得到相对简单的计算公式。每当 GPU 需要计算出屏幕上一个像素是什么颜色的时候，它只需要考虑在这个像素之上的所有 layer 中对应的单个像素，并把这些像素合并到一起。或者，如果最顶层的纹理是不透明的(即图层树的最底层)，这时候 GPU 就可以简单的拷贝它的像素到屏幕上。

当一个 layer 上所有的像素和屏幕上的像素完美的对应整齐，那这个 layer 就是像素对齐的。主要有两个原因可能会造成不对齐。第一个便是滚动；当一个纹理上下滚动的时候，纹理的像素便不会和屏幕的像素排列对齐。另一个原因便是当纹理的起点不在一个像素的边界上。

在这两种情况下，GPU 需要再做额外的计算。它需要将源纹理上多个像素混合起来，生成一个用来合成的值。当所有的像素都是对齐的时候，GPU 只剩下很少的工作要做。

Core Animation 工具和模拟器有一个叫做 color misaligned images 的选项，当这些在你的 CALayer 实例中发生的时候，这个功能便可向你展示。

关于iOS设备的一些尺寸限制可以看这里：[iOSRes](http://iosres.com/)

#####1.4.3 [关于卡顿的简单原理解释](id:anchor1.4.3)[^6]

![image](/images/tec/offscreen/offscreen022.png =500x)

在 VSync 信号到来后，系统图形服务会通过 CADisplayLink 等机制通知 App，App 主线程开始在 CPU 中计算显示内容，比如视图的创建、布局计算、图片解码、文本绘制等。随后 CPU 会将计算好的内容提交到 GPU 去，由 GPU 进行变换、合成、渲染。随后 GPU 会把渲染结果提交到帧缓冲区去，等待下一次 VSync 信号到来时显示到屏幕上。由于垂直同步的机制，如果在一个 VSync 时间内，CPU 或者 GPU 没有完成内容提交，则那一帧就会被丢弃，等待下一次机会再显示，而这时显示屏会保留之前的内容不变。这就是界面卡顿的原因。

从上面的图中可以看到，CPU 和 GPU 不论哪个阻碍了显示流程，都会造成掉帧现象。所以开发时，也需要分别对 CPU 和 GPU 压力进行评估和优化。

###[二，iOS中离屏渲染相关知识](id:anchor2.0)

####[2.1 什么是离屏渲染](id:anchor2.1)

* On-Screen Rendering[^7]

	意为当前屏幕渲染，指的是GPU的渲染操作是在当前用于显示的屏幕缓冲区中进行。
	
* Off-Screen Rendering

	意为离屏渲染，指的是GPU在当前屏幕缓冲区以外新开辟一个缓冲区进行渲染操作。

当图层属性的混合体被指定为在未预合成之前不能直接在屏幕中绘制时，屏幕外渲染就被唤起了。屏幕外渲染并不意味着软件绘制，但是它意味着图层必须在被显示之前在一个屏幕外上下文中被渲染（不论CPU还是GPU）[^8]。

离屏渲染可以被 Core Animation 自动触发，或者被应用程序强制触发。屏幕外的渲染会合并/渲染图层树的一部分到一个新的缓冲区，然后该缓冲区被渲染到屏幕上。[^9]

这里提到的offscreen rendering主要讲的是通过GPU执行的offscreen,事实上还有的offscreen rendering是通过CPU来执行的。

如果我们重写了drawRect方法，并且使用任何Core Graphics的技术进行了绘制操作，就涉及到了CPU渲染。整个渲染过程由CPU在App内同步地完成，渲染得到的bitmap最后再交由GPU用于显示。其它类似cornerRadios, masks, shadows等触发的offscreen是基于GPU的。

***PS：CoreGraphic通常是线程安全的，所以可以进行异步绘制，显示的时候再放回主线程***

许多人有误区,认为offscreen rendering就是software rendering,只是纯粹地靠CPU运算。实际上并不是的,offscreen rendering是个比较复杂,涉及许多方面的内容。我们在开发应用,提高性能通常要注意的是避免offscreen rendering。不需要纠结和拘泥于它的定义。[^10]


####[2.2 为什么要谨慎避免离屏渲染](id:anchor2.2)

[WWDC 2011 Understanding UIKit Rendering](https://developer.apple.com/videos/#121)指出一般导致图形性能的问题大部分都出在了offscreen rendering,因此如果我们发现列表滚动不流畅,动画卡顿等问题,就可以想想和找出我们哪部分代码导致了大量的offscreen 渲染。

离屏渲染主要在两个地方开销较大：

1. 创建新缓冲区

	要想进行离屏渲染，首先要创建一个新的缓冲区。

2. 上下文切换

	离屏渲染的整个过程，需要多次切换上下文环境：先是从当前屏幕（On-Screen）切换到离屏（Off-Screen）；等到离屏渲染结束以后，将离屏缓冲区的渲染结果显示到屏幕上有需要将上下文环境从离屏切换到当前屏幕。而上下文环境的切换是要付出很大代价的。

####[2.3 离屏渲染与光栅化](id:anchor2.3)

光栅化的概念前文有介绍，这里再次跟离屏渲染同时提出来是因为在看过的文章中对这两者的区分有些混淆，这里坐下记录。

*光栅化*：

我们屏幕上显示的画面都是由像素组成，而三维物体都是点线面构成的。要让点线面，变成能在屏幕上显示的像素，就需要Rasterize这个过程。就是从矢量的点线面的描述，变成像素的描述。

* 光栅化概念：将图转化为一个个栅格组成的图像
* 光栅化特点：每个元素对应帧缓冲区中的一像素[^11]

![image](/images/tec/offscreen/rasterize01.png =500x)

iOS中，光栅化的过程是将图形转化为可以存储的bitmap，放在缓存中，以被程序循环使用，减少渲染的频率。

我的理解是光栅化是一种特殊的离屏渲染，它的主要工作量集中在CPU上，而不是前文介绍的那种GPU单独开辟缓存进行图形生成计算，并且CPU光栅化完成后会将该bitmap缓存于本地，以便重复利用，它在形式上也是一种离屏渲染，但不属于`OpenGL`名字中特指的那种GPU新开buffer生成图形的过程。

在`CALayer`中，设置`shouldRasterize = YES`便会触发光栅化，且会将光栅化后的内容缓存起来。相当于光栅化是把GPU的操作转到CPU上了，生成位图缓存，直接读取复用。

因为离屏渲染本身开销较大，所以对于是否需要光栅化，应该因地制宜地使用。且系统设置了对这个光栅化的内存使用限制，有两点需要注意：

1. 不要过度使用,系统限制了缓存的大小为2.5X Screen Size.

	如果过度使用,超出缓存之后,同样会造成大量的offscreen渲染。

2. 被光栅化的图片如果超过100ms没有被使用,则会被移除

	因此我们应该只对连续不断使用的图片进行缓存。对于不常使用的图片缓存是没有意义,且耗费资源的。

####[2.4 一些触发离屏渲染的基本case与替换方式](id:anchor2.4)

除了上面介绍的光栅化可以触发离屏渲染，还有很多种情况可以触发：

* 圆角（当和maskToBounds一起使用时）
* 图层蒙版
* 阴影

对于那些需要动画而且要在屏幕外渲染的图层来说，你可以用`CAShapeLayer`，`contentsCenter`或者`shadowPath`来获得同样的表现而且较少地影响到性能[^12]。

###三，[iOS的一些显示性能检测方法的简单介绍](id:anchor3.0)

####3.1 [显示性能检测一些小Tip](id:anchor3.1)

* 关注FPS

	FPS-Frame per Second,帧率或画面更新率是用于测量显示帧数的量度。，一般来说FPS用于描述视频、电子绘图或游戏每秒播放多少帧，而赫兹则描述显示器的画面每秒更新多少次。在实际体验中，60帧相对于30帧有着更好的体验。[^13]
	
	在开发中，我们应该对FPS的数值保持关注，若发现掉帧严重则可以进一步使用Instruments分析是哪里出现的问题。现在工程里的beta版本的debug包已经附加了此功能。
	
* CPU && GPU[^14]

	CPU，中央处理器。GPU，图形处理器。两者都有总线和外界联系，有自己的缓存体系，以及数字和逻辑运算单元。一句话，两者都为了完成计算任务而设计。
	
	两者的区别在于存在于片内的缓存体系和数字逻辑运算单元的结构差异：CPU虽然有多核，但总数没有超过两位数，GPU的核数远超CPU，被称为众核（NVIDIA Fermi有512个核）。从结果上导致CPU擅长处理具有复杂计算步骤和复杂数据依赖的计算任务，GPU的众核架构非常适合把同样的指令流并行发送到众核上，采用不同的输入数据执行。
	
	并且GPU拥有为视频运算专门设计的运算单元: 光栅单元和纹理填充单元。是专为图形而生的。

	![image](/images/tec/offscreen/opti04.png =500x)

	所以在开发过程中，心中应该有个尺度，对于主线程的UI响应等一些逻辑工作，我们尽量交给CPU来完成，而图形渲染的工作则多交给GPU搞定，检查有没有做无必要的CPU渲染，例如有些地方我们重写了drawRect或开启了光栅化，而其实是我们不需要也不应该的。

* 离屏渲染的消耗

	这会耗费GPU的资源，像前面已经分析的到的。offscreen 渲染会导致GPU需要不断地onScreen和offscreen进行上下文切换。
	
* Blended Layers | Misaligned Images
	
	检查我们有无过多的合成 | Blending，图片的格式是否为常用格式，大小是否正常。如果一个图片格式不被GPU所支持，则只能通过CPU来渲染。

####3.2 [Instruments - Core Animation](id:anchor3.2)

使用Core Animation可以帮助我们通过观察FPS来定位问题所在。如下图所示，两个红框处便是掉帧比较严重的地方，分别是股票详情页与牛圈首页，然后我们在选取此处，观察调用栈，便可以找出哪里吃性能比较严重了。

![image](/images/tec/offscreen/opti01.png =500x)

又如下图，可以通过勾选不同的选项，观察页面中是否存在，*Blended Layers*，*Misaligned Images*等一系列前文提到的可优化点。

![image](/images/tec/offscreen/opti03.png =500x)

* `Color Blended Layers`，这个选项选项基于渲染程度对屏幕中的混合区域进行绿到红的高亮显示，越红表示性能越差，会对帧率等指标造成较大的影响。红色通常是由于多个半透明图层叠加引起。[^15]

* `Color Offscreen-Rendered Yellow`，这个选项会把那些离屏渲染的图层显示为黄色。黄色越多，性能越差。这些显示为黄色的图层很可能需要用 shadowPath 或者 shouldRasterize 来优化。

* `Flash Updated Regions`，这个选项会把重绘的内容显示为黄色。不该出现的黄色越多，性能越差。通常我们希望只是更新的部分被标记完黄色。

####3.3 [Instruments - OpenGL ES](id:anchor3.3)

OpenGL ES驱动工具可以帮你测量GPU的利用率，同样也是一个很好的来判断和GPU相关动画性能的指示器。它同样也提供了类似Core Animation那样显示FPS的工具

![image](/images/tec/offscreen/opti05.png =500x)

Renderer Utilization - 如果这个值超过了~50%，就意味着你的动画可能对帧率有所限制，很可能因为离屏渲染或者是重绘导致的过度混合。

Tiler Utilization - 如果这个值超过了~50%，就意味着你的动画可能限制于几何结构方面，也就是在屏幕上有太多的图层占用了。

###四，[AsyncDisplayKit分享](id:anchor4.0)

####4.1 [AsyncDisplayKit是干什么的](id:anchor4.1)

AsyncDisplayKit is an iOS framework that keeps even the most complex user interfaces smooth and responsive. It was originally built to make Facebook's Paper possible, and goes hand-in-hand with pop's physics-based animations — but it's just as powerful with UIKit Dynamics and conventional app designs.[^16]

####4.2 [AsyncDisplayKit的一些性能上的优势](id:anchor4.2)

AsyncDisplayKit Nodes are a thread-safe abstraction layer over UIViews and CALayers:

![image](/images/tec/offscreen/asdk01.png)

If you know how to use views, you know how to use nodes. ASImageNode and the Text Kit-powered ASTextNode can be used just like their UIKit counterparts. Unlike UIKit view hierarchies, node hierarchies for entire screenfuls of content can be initialized and laid out on background threads — and nodes make it easy to take advantage of the multicore CPUs in all current iOS devices.

Nodes have many advantages over views. For example, you can often improve performance by replacing views with layers. Unfortunately, doing so requires the tedious process of porting view-based code to the different API and inevitably risks regressions. With nodes, it’s as easy as:[^17]

![image](/images/tec/offscreen/asdk02.png =250x)

If you later need to switch from layers back to views, it’s a one-line change! This is a transformational difference. Instead of being cautious of layer-backed UI code, you can use it by default whenever you don’t need touch handling.

[^1]: [屏幕显示图像的原理](http://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)

[^2]: [Getting Pixels onto the Screen](http://objccn.io/issue-3-1/)

[^3]: [iOS图形处理和性能](http://www.cocoachina.com/industry/20130821/6841.html)

[^4]: [WWDC心得与延伸:iOS图形性能 -- 方秋枋](https://github.com/100mango/zen/blob/master/WWDC%E5%BF%83%E5%BE%97%EF%BC%9AAdvanced%20Graphics%20and%20Animations%20for%20iOS%20Apps/Advanced%20Graphics%20and%20Animations%20for%20iOS%20Apps.md)

[^5]: [Compositing](https://www.objc.io/issues/3-views/moving-pixels-onto-the-screen/)

[^6]: [卡顿产生的原因和解决方案](http://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)

[^7]: [离屏渲染学习笔记](http://foggry.com/blog/2015/05/06/chi-ping-xuan-ran-xue-xi-bi-ji/)

[^8]: [ios核心动画高级技巧](https://zsisme.gitbooks.io/ios-/content/chapter15/offscreen-rendering.html)

[^9]: [离屏渲染(Offscreen Rendering)](http://objccn.io/issue-3-1/)

[^10]: [关于offscreen rendering -- 方秋枋](https://github.com/100mango/zen/blob/master/WWDC%E5%BF%83%E5%BE%97%EF%BC%9AAdvanced%20Graphics%20and%20Animations%20for%20iOS%20Apps/Advanced%20Graphics%20and%20Animations%20for%20iOS%20Apps.md)

[^11]: [iOS 离屏渲染的研究 -- 齐滇大圣](http://www.jianshu.com/p/6d24a4c29e18)

[^12]: [CAShapeLayer使用方式的介绍](https://zsisme.gitbooks.io/ios-/content/chapter15/offscreen-rendering.html)

[^13]: [帧率 - wiki](https://zh.wikipedia.org/wiki/%E5%B8%A7%E7%8E%87)

[^14]: [CPU 和 GPU 的区别是什么？ - 王洋子豪](https://www.zhihu.com/question/19903344)

[^15]: [使用 Instruments 做 iOS 程序性能调试](http://www.samirchen.com/use-instruments/)

[^16]: [facebook/AsyncDisplayKit](https://github.com/facebook/AsyncDisplayKit)

[^17]: [Introducing AsyncDisplayKit: For smooth and responsive apps on iOS](https://code.facebook.com/posts/721586784561674/introducing-asyncdisplaykit-for-smooth-and-responsive-apps-on-ios/)



















