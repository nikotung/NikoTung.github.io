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

### 建立在TCP/IP协议的HTTP

HTTP是一个主从架构的应用层协议。它主要是运行在一个TCP/IP连接之上，就像下图描述一样。(HTTP 其实不需要运行在TCP/IP连接之上。它只是需要一个可靠的传输。任何能够保证稳定的传输协议都可以)

![](assets/2014-04-27-HTTP_OverTCPIP.png)

TCP/IP是一系列用于设备通过网络交互传输的传输层和网络层的协议。

IP(Internet Protocol)是一个处理网络地址和路由的网络层协议。在一个IP网络，每一个机器都有一个唯一的ip地址(比如。165.1.2.3)，并且IP路由软件就是负责把信息从一个源IP地址发送到目的IP地址。在IPv4，IP地址包含4哥字节，每个从0-255，通过点号隔开，称为四分之一点格式。这种格式包含了4g多个IP地址。最新得IPv6就更多了。由于记住数字是比较难的，一个用英语表示的域名，比如www.test101.com就代替了数字。DNS就是负责把域名转化称为IP地址。一个特别的IP地址127.0.0.1 通常就是表示你自己的机器，它的域名就是localhost可以用于本地回路测试。

TCP是一个传输层的协议，负责在两台机器上建立连接。TCP包括两个协议:TCP和UDP。tcp连接更可靠，它的每一个包都有一个顺序号，和一个应答信号。一个包如果没有被收到，它就会从新发送。包的传输在tcp中是很好的保证了。udp就不能保证每一个包都能发送，所以它是不可靠的。但是upd需要比较少的网络开销，能够用于一些音频和视频的流传输。

TCP支持在同一个IP的机器中多个应用程序传输。对于每一台机器可以有65536个端口，从端口号0到65535。一个应用比如HTTP,FTP会占用一个特定的端口。端口0到1023被一些主流的协议给占用了。比如HTTP用80,FTP用21，Telnet用23 SMTP用25，NNTP用119，DNS用53。在1024以上的我们都能用。

尽管TCP端口80默认是被HTTP使用，但是这不影响用其他可用的端口(1024-65535),比如8080，8000，尤其是那些测试的服务。你也可以在同一台机器上同时使用不同的端口运行HTTP服务。当一个客户端输入没有指定端口号的URL时，比如：http://www.test101.com:8080/docs/index.html,浏览器就会默认连接到80端口。你需要指定端口号，比如：http://www.test101.com/docs/index.html如果服务器监听在8000端口而不是80端口.

简而言之，通过TCP/IP协议连接，你需要了解IP地址，域名和端口号。

### HTTP规范

HTTP规范是由[W3C](http://www.w3.org/)来维护，在[这里](http://www.w3.org/standards/techs/http)可以看到。目前来说，只有两个版本的HTTP规范，HTTP/1.0 和HTTP/1.1。最初的版本，HTTP/0.9(1991年)，是由蒂姆·伯纳斯-李写的，是一个简单的通过网络传输元数据的协议。HTTP/1.0(1996年)通过允许MIME-like的信息来提升了。1.0版本没有说明代理，缓存，持久连接，虚拟host和断点续传，这些功能都在1.1版给完善了。

### Apache HTTP Server or Apache Tomcat Server

建立一个http服务，是由必要知道http协议的。

apache HTTP 服务是一个流程的工业级的服务，是由Apache Software Foundation(ASF)@[www.apache.org](www.apache.org)提供。ASF是一个开源的的软件基础服务，就是说apache http 服务是免费的并且提供了源代码。

阅读[Apache How to](http://www3.ntu.edu.sg/home/ehchua/programming/howto/Apache_HowToInstall.html) 关于如何暗转和配置apache http 服务；或者[Tomcat How-to](http://www3.ntu.edu.sg/home/ehchua/programming/howto/Tomcat_HowTo.html)关于如何安装并开始使用apache tomcat服务。

### HTTP请求和相应信息

HTTP客户端和服务器通过发送文本信息来通信。客户端发送请求信息到服务端，接着服务端就会返回一个相应信息。

一个HTTP信息包括信息的"头"和一个可选的信息体，通过一个空行隔开，像下面描述一样：

![](assets/2014-04-27-HTTP_MessageFormat.png)

#### HTTP请求信息

HTTP请求信息的格式如下：

![](assets/2014-04-27-HTTP_RequestMessage.png)

###### 请求线(Request Line)

请求头的第一行称为请求线，接着可选的请求头

请求线有如下的语法：

	request-method-name request-URI HTTP-version

* request-method-name: HTTP 协议定义了一系列的请求方法，比如，GET,POST,HEAD,and OPTIONS.客户端可以使用其中的一个去发起请求到服务器中去。
* request-URI:指定了要请求的资源
* HTTP-version: 当前有两个版本在使用：HTTP/1.0 和 HTTP/1.1.

下面是一个请求线的样例：

	GET /test.html HTTP/1.1
	HEAD /query.html HTTP/1.0
	POST /index.html HTTP/1.1

###### 请求头(request-header)

请求头是通过key－value的形式的，多个值就通过逗号隔开。
	
	request-header-name: request-header-value1, request-header-value2, ...

下面是一个请求头的样例：

	Host: www.xyz.com
	Connection: Keep-Alive
	Accept: image/gif, image/jpeg, */*
	Accept-Language: us-en, fr, cn

##### 样例

下面是一个HTTP请求的样例：

![](assets/2014-04-27-HTTP_RequestMessageExample.png)

#### HTTP相应信息

HTTP相应信息的格式如下：

![](assets/2014-04-27-HTTP_ResponseMessage.png)

##### 状态线(status line)

第一行称为状态线，接着一下可选的请求头

状态线有如下的语法：

	HTTP-version status-code reason-phrase

* HTTP-version: 使用在这次会话的HTTP版本，HTTP/1.0 或者 HTTP/1.1。
* status-code: 服务端根据请求的输出而生成的一个三位数。
* reason-phrase:一个对status-code的简单描述。
* 常见的status-code是“200 OK”.

状态线的样例:

	HTTP/1.1 200 OK
	HTTP/1.0 404 Not Found
	HTTP/1.1 403 Forbidden 


##### 相应头(response headers)

相应头是一个键值对(key-value)的形式：

	response-header-name: response-header-value1, response-header-value2, ...

相应头的样本：

	Content-Type: text/html
	Content-Length: 35
	Connection: Keep-Alive
	Keep-Alive: timeout=15, max=100

相应信息的样例：

![](assets/2014-04-27-HTTP_ResponseMessageExample.png)

### HTTP 请求方法

HTTP协议定义了一系列的请求方法。客户端可以使用其中的一个去发起请求到服务器。可以使用的请求方法有:

* GET: 客户端使用该方法可以获取一些网页资源
* HEAD: 去获取一个get请求的头。这个头包含了last-modified 的日期信息，可以用于跟本地缓存做对比。
* POST:post数据到服务器
* PUT: 请求服务器去保存数据
* DELETE:请求服务器去删除数据
* TRACE:返回整个请求过程的信息，如一开始的请求信息
* OPTIONS:返回服务器支持哪些请求方法
* CONNECT:
* 其它扩展方法。


### GET请求方法

GET是最常见的HTTP请求方法。客户端可以用GET方法去获取一些资源。一个GET请求需要如下的语法：

	GET request-URI HTTP-version
	(optional request headers)
	(blank line)
	(optional request body)

* GET 关键词是大小写敏感的，并且必须要大写
* request-URI:指定了请求资源的路径
* HTTP-verion:请求的版本
* 客户端可以使用一些可选的请求头(比如Accept,Accept-Language等等)去和服务器交换并要求返回指定的内容
* GET 请求有一些可选的请求内容包括一些查询串（稍后会提到）




#### http/1.0 GET请求
















































