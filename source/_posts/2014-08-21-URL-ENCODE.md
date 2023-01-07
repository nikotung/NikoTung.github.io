---
title: "URL编码和解码的问题"
description: "了解HTTP协议"
categories: "2014-08"
tags: [URL]
---


这个问题其实很早之前都有遇到过，一直没有记录下来。恰好今天有被我遇到了，看来我是要跟它耗上了。


### URL编码

我们都知道URL只能使用英文字，阿拉伯数字和一些标点符号。其它的必须要经过编码后才能使用。这就是URL编码了。

因此，我们使用URL时都会对其进行一个编码，这样即是为了可控，也是为了安全起见。为了安全也不应该重复编码。（问题就是出现在这里）


### App前端

我们现在的app中的架构中是通过一个自定义的URL scheme去打开一个页面（Scheme这个技术无论是在iOS还是Android中都是支持的），因此我们要打开一个新的页面前都是需要构造一个我们的架构中支持的的URL。这个跟一般的URL没什么区别，其中这个URL可以传递参数，app的架构把这些参数通过key-value的形式单个取出来。

因为是通过URI的方式传值，因此App中每取一个值都会对其进行URL解码。

其中这里面就涉及到一些URL的解析过程了，这里就不去细说了。

在我们自定义的URL中有可能传递一个参数，就叫url，是一个http的url，而且这个url后面还带有自己的参数。这时候的url解析就要注意了。

这里我举个例子

比如这样的一个url

niko://niko?user=niko&gendor=1&url=http://www.baidu.com?kw=query&user=niko

从我们自己的理解来说应该对这个url的query参数解析出来是这样的

	user=niko
	gendor=1
	url=http://www.baidu.com?kw=query&user=niko

为了能够达到我们的期望，做法就是把url这个参数对应的值进行URL编码（这个编码是全局的编码即任何一个字符都会被编码，就像js中的encodeURIComponent），app中去url的值时进行一个解码，这样一个编码一个解码相对应，就不会影响我们的取值使用了。




### App后端

由于我们的URL是由后端传过来的，这里的URL是指符合我们架构的URL。后端在传过来时，为了App能够正常解析出来，就会对url这样的参数进行全局的编码。

这样我们的前后，我们前后端一直相安无事。

直到最近。

### App升级

最近App的架构做了一些修改，即App中取出了url要使用时，比如用来打开一个webview，就会对url进行一个编码（这个编码只会对常见的字符编码，就像js中的encodeURI），然后再去请求。

PS：在iOS中，如果一个url的string中有中文等字符是无法通过[NSURL URLWithString:xxxx]，生成一个NSURL对象的。

然后我们发现，我们对应业务的有一些打开url的页面无法正常加载。

后来通过调试发现，原来后端在生成传给App的URL时，做了这样的一个处理：在生成URL的url的参数时，对这些参数进行了一个编码（encodeURI），然后在生成了整个url时又进行了一次编码（encodeURIComponent）。就这样url的参数被进行了两次编码。

这时后App中取出url来后再次编码才去使用，最终url的参数还是被编码了两次，导致了某些url无法加载。


### 结论

* 对于这种url中嵌套url的做法，尽量不要，实在需要的话一定要两边订好如何编码解码；
* 避免对url重复编码，避免摆乌龙；




### 参考

[关于URL编码](http://www.ruanyifeng.com/blog/2010/02/url_encoding.html)
