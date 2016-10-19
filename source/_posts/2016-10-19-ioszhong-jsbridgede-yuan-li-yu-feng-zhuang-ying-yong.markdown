---
layout: post
title: "iOS中jsBridge的原理与封装应用"
date: 2016-10-19 20:46
comments: true
categories: Tec
---


###一，基本原理

*A. Web向Native通信方式：*

客户端Native内置的webView控件可以看做是一个简易的浏览器，可以使用该控件载入Web页面。当Web页面`URL`发生变化时会触发webView控件的`urlReload`函数，并可以拿到变化后的`URL`路径。若在此`URL`上附上一些特殊信息，便实现了Web向Native通信。

当前Web中比较流行的做法是在页面内隐式嵌套一个`iFrame`，通过改变此`iFrame`的`src`，达到触发Native`urlReload`的目的。

*B. Native向Web通信方式：*

Native向Web通信的方式比较粗暴，在iOS端，webView内置了直接执行`js`的方法：

```
[_webView stringByEvaluatingJavaScriptFromString:javascriptCommand]
```

一般情况下，我们约定好Web页面`js`中定义的方法名，则此处便可以直接调用了。

<!--more-->

###二，大致流程：

Native载入H5页面时通过本地内置的`js`代码注入一个供H5页面调用的对象`WebViewJavascriptBridge`，该对象为H5页面提供函数注册方法：`registerHandler(handlerName, handler)`，以供Native调用：

```
bridge.registerHandler('testJavascriptHandler', function(data, responseCallback) {
			log('ObjC called testJavascriptHandler with', data)
			var responseData = { 'Javascript Says':'Right back atcha!' }
			log('JS responding with', responseData)
			responseCallback(responseData)
		})
```

同时也提供调用函数：`callHandler(handlerName, data, responseCallback)`，供H5直接调用Navtive的函数方法：

```
bridge.callHandler('testObjcCallback', {'foo': 'bar'}, function(response) {
				log('JS got response', response)
			})
```

当H5页面载入完成后，H5页面发送`scheme://jsBridge/bridge_loaded`，通知Native页面载入完成，则Native注入`js`对象`WebViewJavascriptBridge`，供H5注册本地函数与调用Native函数。

两方具体通信流程可参见下图：

![image](/images/tec/jsbridge/H5 Call Native.png)

![image](/images/tec/jsbridge/Native Call H5.png)
































