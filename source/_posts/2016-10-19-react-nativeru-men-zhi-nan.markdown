---
layout: post
title: "React Native入门指南"
date: 2016-10-19 20:56
comments: true
categories: Tec
---

###一，背景

####1. 前言

`React Native`现在的口号是*Learn Once,Write Anywhere*，让我想起了早些年`Jave`曾倡导过的*Write Once, Run Everywhere*，最后虽被人调侃成了*Write Once, Debug Everywhere*，但不失为一次跨平台开发的早期尝试。

这几年游戏开发中很火的`COCOS2D-X`，相较看来确实是个成功的案例了。

现在FaceBook要试水App的跨平台开发，推出不到一年的时间已经火遍世界，但最终能走多远，现在下结论还为时过早。

<!--more-->

####2. 起源

要说`React Native`得先从[ReactJS](http://reactjs.cn/)说起。

`ReactJS`起源于 Facebook 的内部项目，因为该公司对市场上所有 JavaScript MVC 框架，都不满意，就决定自己写一套，用来架设 Instagram 的网站。

由于 React 的设计思想极其独特，属于革命性创新，性能出众，代码逻辑却非常简单。所以，越来越多的人开始关注和使用，认为它可能是将来 Web 开发的主流工具。

这个项目本身也越滚越大，从最早的UI引擎变成了一整套前后端通吃的 Web App 解决方案。衍生的 `React Native` 项目，目标更是宏伟，希望用写 Web App 的方式去写 Native App。如果能够实现，整个互联网行业都会被颠覆，因为同一组人只需要写一次 UI ，就能同时运行在服务器、浏览器和手机[^1]


###二，原理

最基本的Native与WebView通讯原理我之前的一篇文章讲过，在此不多说了，`RN`的实现机制也是基于同样的`JSBridge`规则。

![image](/images/tec/reactNative/rn01.png)

所不同的是，`RN`使用`JSBridge`只是当做一块跳板，最终目的还是调用本地原生API进行UI绘制。

之前与WebView的混合应用需要我们在初始化的时候手动注入方法作为MethodID，在`RN`中，由于每个模块类都实现了RCTBridgeModule接口，可以通过runtime接口objc_getClassList或objc_copyClassList取出项目里所有类，然后逐个判断是否实现了RCTBridgeModule接口，就可以找到所有模块类[^2]。

由此，我们写出来的`JS`代码就可以通过`JSBridge`直接调用本地UI控件了。

`RN`的入口函数为：

```
// Module name
AppRegistry.registerComponent('RNProject', () => RNApp);
```

写在`index.ios.js`文件中。

react native自己实现了一个打包方式packger，打包之后，我们的js代码包括react native的js源码都被打包压缩成了一个.jsbundle文件，我们在index.ios.js里面可以写es6的语法，这些都会在打包的时候去编译解析，生成这个jsbundle文件里面的代码是基于commonJS规范的，便于[JavaScriptCore](http://trac.webkit.org/wiki/JavaScriptCore)解析。[^3]

再由Native本地调用：

```
NSURL *jsCodeLocation = [NSURL URLWithString:@"http://localhost:8081/rn/test.ios.bundle?platform=ios"];
RCTRootView *rootView = [[RCTRootView alloc] initWithBundleURL:jsCodeLocation 
												moduleName:@"RNProject" 
												initialProperties:nil launchOptions:nil];
UIViewController *vc = [[UIViewController alloc] init];
vc.view = rootView;
[self presentViewController:vc animated:YES completion:nil];
```
![image](/images/tec/reactNative/rn02.png)


###三，Hello World

####1. 准备知识

* [NodeJS](https://nodejs.org/zh-cn/)

> Node.js®是一个基于Chrome V8 引擎的 JavaScript 运行时。 Node.js 使用高效、轻量级的事件驱动、非阻塞 I/O 模型。Node.js 之生态系统是目前最大的开源包管理系统。

* [npm](https://zh.wikipedia.org/wiki/Node%E5%8C%85%E7%AE%A1%E7%90%86%E5%99%A8)

> Node包管理器（Node Package Manager）。它是一个以javascript编写的软件包管理系统，默认环境为Node.js，从Node.js0.6.3版本开始，npm被自动附带在安装包中。

类似于`Homebrew`之于Mac，`npm`的使用有时候也受制于*墙*的限制，它的默认源地址为

```
https://registry.npm.org
```

可以使用命令查看：

```
npm get registry
```

现在比较流行的国内镜像地址是淘宝提供的，我们可以直接使用命令设置：

```
npm set registry https://registry.npm.taobao.org
```

[React-Native痛点](http://www.infoq.com/cn/articles/react-native-solution-dev-environment)这篇文章的后半部分还提供了一种自己建立`npm`资源站点的方式，适合团队内部使用。文章前半部分讲解了下`RN`初始化时内部所做的一些事情，有一定的学习意义。

####2. RN与原生App整合

这里我们使用[CocoaPods](https://cocoapods.org/)的方式进行整合。

整合方式很简单，大体分为三步：

1. 使用`npm`下载`RN`相关资源文件。
2. 配置`podfile`，将资源整合。
3. 编写入口文件，进行代码编写。

具体细节可参考[官网文档](https://facebook.github.io/react-native/docs/integration-with-existing-apps.html)。

###四，开发技能数

**UI**

* JS

	`RN`从0.18开始，默认项目转向ES6。

* JSX

	JSX 是一个看起来很像 XML 的 JavaScript 语法扩展。React 可以用来做简单的 JSX 句法转换。
	
* StyleSheet

	CSS的一种实现方式
	
**Net Working**

`RN`内置了`Ajax`网络调用方式，同时也支持其它第三方网络库的使用。

```
var request = new XMLHttpRequest();
request.onreadystatechange = (e) => {
  if (request.readyState !== 4) {
    return;
  }
  if (request.status === 200) {
    console.log('success', request.responseText);
  } else {
    console.warn('error');
  }
};
request.open('GET', 'https://mywebsite.com/endpoint/');
request.send();
```

**热更新**

![image](/images/tec/reactNative/rn03.png)






[^1]: [React 入门实例教程](http://www.ruanyifeng.com/blog/2015/03/react.html)
[^2]: [React Native通信机制详解](http://blog.cnbang.net/tech/2698/)
[^3]: [React Native IOS集成与原理简析](https://www.nihaoshijie.com.cn/index.php/archives/560)





























