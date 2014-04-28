---
layout: post
title: "HTTP协议"
description: "了解HTTP协议"
category: "2014-04-27"
tags: [HTTP]
---

在iOS开发时，很多时候都要用到网络请求。在我的经验中大部分是HTTP的请求，很看过一些网络请求使用和一些第三方开源库如[AFNetworking](https://github.com/AFNetworking/AFNetworking)，就很快的可以使用HTTP和服务端进行交互了，所以就一直停留在这个层面，没有具体的了解HTTP这个协议。最近需要实现使用HTTP实现上传文件的功能，发现有必要去具体了解一下HTTP这个协议。

所以我就找到南洋理工大学的一篇文章，觉得还是不错的，所以想试着把它翻译一下，希望可以加深自己的理解。

## HTTP(HyperText Transfer Protocal)

### Basic

#### Introduction

### The WEB

网络（或者说网站）是由大量的客户端/服务端分布组成的信息系统，像下面的图描述一样。

![](/assets/2014-04-27-TheWeb.png)

大部分的应用程序同时运行在web上面，如网页浏览，电子邮件，文件传输，音频/视频流等等。为了能够服务端和客户端能够正常通信，这些应用程序必须遵循一些应用成面上的协议，比如HTTP,FTP,SMTP,POP等等。

### HyperText Transfer Protocal(HTTP)

HTTP(HyperText Transfer Protocal)协议也许就是最广泛应用在网络上。

* HTTP是一个对称的：请求－应答；客户端－服务端协议，像下面描述一样。一个HTTP客户端发送一个请求到一个HTTP服务器，服务器就返回一个信息。换句话说，HTTP是一个`pull protocol`,客户端从服务器中拉取信息(服务器往客户端推送消息)。

![](/assets/2014-04-27-HTTP.png)

* HTTP是一个无状态协议。也就是说当前的请求并不知道之前的请求做了什么事。
* HTTP允许自定义数据的类型和表现形式，以便可以让系统独立生成要传输的数据。
* 引用 RFC2616的说明：HTTP协议是一种用于分布，协同以及超媒体发布的应用层协议。它是一种不仅仅用于富文本传输，通过扩展它的请求方法，错误码和传输头(header),还可以用于像域名服务器和分布式对象管理系统，的一般的无状态协议。

### 浏览器

当你在浏览器中输入一个HTTP地址去获取网页资源时，比如:`http://www.test101.com/index.html`,浏览器就会把这个url转变成一个请求把它发送到HTTP服务器中。HTTP服务器解析了请求的信息，并返回了对应的信息，请求的资源或者一个错误的信息。这个过程就是如下：

![](/assets/2014-04-27-HTTP_Steps.png)

### URL(Uniform Resource Locator)

一个URL就是用于唯一定位网页上的资源。一个URL有如下的语法：

* 协议(Protocol):客户端和服务端使用的应用层协议，比如：HTTP,FTP或者远程登录(telnet)
* 主机名(HostName):服务器的域名或者IP地址
* 端口(port):服务器监听来自客户端的请求的TCP端口
* 路径文件名:要请求的资源的文件或者资源

比如在这个URL中`http://www.test101.com/docs/index.html`,其中的交互协议是HTTP，主机名是:www.test101.com.端口号在URL中没有显式表明，使用了默认的80端口，路径和文件名为`/docs/index.html`.

其他的例子：

	ftp://www.ftp.org/docs/test.txt
	mailto:user@test101.com
	news:soc.culture.Singapore
	telnet://www.test101.com/

### HTTP协议

上面中提到只要你在浏览器的地址中输入一个地址，浏览器就会根据对应的协议把它转化成一个请求，并发送到服务器中。

比如浏览器就会把URL`http://www.test101.com/docs/index.html`转化成下面的请求：

	GET /docs/index.html HTTP/1.1
	Host: www.test101.com
	Accept: image/gif, image/jpeg, */*
	Accept-Language: en-us
	Accept-Encoding: gzip, deflate
	User-Agent: Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1)
	(blank line)

当请求到达服务器，服务器就会有如下几个方法去处理：

* 服务器解析收到的请求，把它映射到服务器的文档下的一个文件，并把这个客户端请求的文件返回
* 服务器解析收到的请求，把对应的请求映射到一个段代码中，然后执行对应的代码，把程序的输出结果返回到客户端
* 请求无法解析，服务端返回一个错误

下面是一个HTTP请求的相应信息：

	HTTP/1.1 200 OK
	Date: Sun, 18 Oct 2009 08:56:53 GMT
	Server: Apache/2.2.14 (Win32)
	Last-Modified: Sat, 20 Nov 2004 07:16:26 GMT
	ETag: "10000000565a5-2c-3e94b66c2e680"
	Accept-Ranges: bytes
	Content-Length: 44
	Connection: close
	Content-Type: text/html
	X-Pad: avoid browser bug
	  
	<html><body><h1>It works!</h1></body></html>


	浏览器收到相应的信息，根据相应信息的媒体类型（在相应头的Content-Type），解析相应的信息并把它展示到浏览器的窗口中.常见的媒体类型包括“text/plain”,"text/html","image/gif","image/jpeg","audio/mpeg","video/mpeg","application/msword"和"application/pdf".

在空闲状态，一个HTTP服务器什么事都不干，只是监听会发送请求过来的IP地址和端口。当一个请求过来的时候，服务器使用配置文件的规则分析请求的头，相应不同的处理方法。网页的主要控制只要是在配置文件中，接下来会详细讨论。






















