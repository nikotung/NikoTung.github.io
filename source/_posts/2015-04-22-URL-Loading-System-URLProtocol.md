---
title: "URL Loading System -NSURLProtocol"
description: "URL加载系统的 NSURLProtocol的基本理解"
categories: "2015-04"
tags: [Objective-C,NSURLProtocol]
---


昨天无意的在[github](https://github.com)上看到一个stub http请求的精简"框架"[OHHTTPStubs](https://github.com/AliSoftware/OHHTTPStubs)。大概看了一下，感觉有点意思的，主要是使用了`NSURLProtocol`去实现stub的。很早之前就看过这个protocol了，现在在这里简单记录一下。

### 使用

在本地把你需要mock的数据准备好，最好是以文件的方式。它里面主要用到`OHHTTPStubs`这个类去stub，返回就用`OHHTTPStubsResponse`。在这个方法中:

		+(id<OHHTTPStubsDescriptor>)stubRequestsPassingTest:(OHHTTPStubsTestBlock)testBlock
                                   withStubResponse:(OHHTTPStubsResponseBlock)responseBlock;


 的testBlock中去指定需要stub的request，在responseBlock中返回stub的数据。如果你没有启动stub，需要调用`+(void)setEnabled:(BOOL)enabled;` 去启用它。就这么简单，你可以把你的http请求stub了。

### 为什么说有意思

 一开始我也是觉得这个该用于单元测试中的，后来发现在ut的作用不大啊。因为ut中，本身就可以自己stub然后返回自己想要的数据(有可能这些数据要自己格式化好，转成model什么的)。再加上在ut在大部分情况下，没有必要把网络的东西这样去做，应该放到集成测试中。
 但是，它在跟服务器并行做开发的时候，就比较有用了。因为我们只需要跟server端定义好接口就可以直接开发，而不需要求server都有数据返回，可以把数据stub掉。而且它还能让你控制接口响应的时间，对于app开发的一些外部依赖可以减轻，我们可以直接写完代码后，一次性跟server端集测。



###   URL Loading System

 在iOS的网络请求中，我们一般用得比较多的应该是:`NSURLConnection`,`NSURLRequest`和`NSURL`(如果只支持iOS7以上的NSURLSession这类应该比较多)。这其中涉及到一个URL Loading System这样的名词。这个可以理解为iOS中获取数据的一个总称吧。它里面包含了一系列的类和一些协议。

 ![](https://developer.apple.com/library/prerelease/ios/documentation/Cocoa/Conceptual/URLLoadingSystem/Art/nsobject_hierarchy_2x.png)

### customer Protocol

 如果你要实现“拦截”url,我们就要实现自定义一个`NSURLProtocol`的子类。只要我们在:

 		+ (BOOL)canInitWithRequest:(NSURLRequest *)request

这个方法中返回yes，后面这个request都会由我们自定的protocol去管理。需要注意的是，你需要用调用`[NSURLProtocol registerClass:CustomerProtocol.class];`去注册。越迟注册，会被越先调用。在这里面你可以干你想做的任何事情(法律允许范围内-.-)。



