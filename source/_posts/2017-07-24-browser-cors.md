---
title: "浏览器跨域"
description: ""
categories: "2017-07"
tags: [http , 浏览器, 跨域]
---

前端时间公司后台接口都加上了Oauth，前端请求都需要加上Authorization 的头。但是前端同学说后台接口访问不了，有跨域问题。根据后端同学说项目已经实现了跨域请求的支持。所以就抽空研究了一下跨域。

先看看后端同学是如何实现跨域的。

	
	@WebFilter
	@Component
	public class FilterResponse implements Filter{
		@Override
		public void destroy() {			
		}
		
		@Override
		public void doFilter(ServletRequest request, ServletResponse response,
				FilterChain chain) throws IOException, ServletException {
			HttpServletResponse rep = (HttpServletResponse)response;
			HttpServletRequest req = (HttpServletRequest)request;
			rep.setHeader("Access-Control-Allow-Methods", "GET, POST, PUT, OPTIONS");
			rep.setHeader("Access-Control-Allow-Origin", "*");
	        rep.setHeader("Access-Control-Allow-Headers", "Content-Type");
	        rep.setHeader("Access-Control-Allow-Credentials","true");
			
			chain.doFilter(request, response);
		}
		@Override
		public void init(FilterConfig filterConfig) throws ServletException {			
		}
	}


粗看一下感觉没有问题。这个写法网上一搜一大堆。

#### cors(Cross-origin resource sharing)定义

我们先看看跨域的定义。

> Cross-origin resource sharing (CORS) is a mechanism that allows restricted resources (e.g. fonts) on a web page to be requested from another domain outside the domain from which the first resource was served.[1] A web page may freely embed cross-origin images, stylesheets, scripts, iframes, and videos.[2] Certain "cross-domain" requests, notably Ajax requests, however are forbidden by default by the same-origin security policy.

> CORS defines a way in which a browser and server can interact to determine whether or not it is safe to allow the cross-origin request.[3] It allows for more freedom and functionality than purely same-origin requests, but is more secure than simply allowing all cross-origin requests. It is a recommended standard of the W3C

来自[wiki](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)

简单来说就是在一个域名下访问另外一个域名的内容。如果单纯的从http 协议上去看是没有问题的，但是由于浏览器有同源策略的限制，访问的内容被浏览器拦截了。所以说这跨域是浏览器的一个安全机制问题，我们需要支持这个跨域就要遵循浏览器的规范。


#### cors 的场景

出现跨域问题主要是没有遵循浏览器的规范，导致请求被拦截。而这其中的场景主要有这样两个:

* 浏览器发出一个跨域请求(simple request)，后台返回的http 头部与请求的头部不匹配，通不过浏览器的跨域验证，返回结果被浏览器拦截
* 浏览器即将发出一个跨域请求，浏览器检测到是一个跨域请求，此时浏览器会提前发送一个预判请求(preflight request,一个 OPTIONS请求)，检测即将发送的请求是否被允许跨域访问。如果预判请求返回的结果通不过跨域验证，则浏览器不会发送真实的请求。

上面的两个场景中，一个是已经发送请求成功，但是结果被浏览器拦截，一个是真实的请求根本没有发送出去。

现在我们就需要知道什么时候会出现上面的两个场景。

##### simple request

同时满足如下条件，则就是一个 simple request

请求方法是如下几个

* GET
* HEAD
* POST

请求头部是如下几个:(除了那些被限制的header)

* Accept
* Accept-Language
* Content-Language
* Content-Type(有额外条件)
* DPR
* Downlink
* Save-Data
* Viewport-width
* Width

Content-Type 是如下几个:

* application/x-www-form-urlencode
* multipart/form-data
* text/plain

同时满足了上面三个条件，浏览器就会直接发请求，而不会发预判请求。最后通不通过跨域验证，要看后台返回的内容(header)。

##### preflight request

只要满足以下其中的任何一个条件

请求方法是如下几个:

* PUT
* DELETE
* CONNECT
* OPTIONS
* TRACE
* PATCH

请求头部是如下几个:(除了那些被限制的header):

* Accept
* Accept-Language
* Content-Language
* Content-Type(有额外条件)
* DPR
* Downlink
* Save-Data
* Viewport-width
* Width

Content-Type 是如下几个:

* application/x-www-form-urlencode
* multipart/form-data
* text/plain

满足了上面其中一个，浏览器就会先发预判请求。通不通过跨域验证，要看后台返回的内容(header)。通过后才能发真实请求。


#### 浏览器如何验证通不通过跨域

在上面的第一种场景下，浏览器发出的请求会带有一个很重要的 header 

	Origin: http://localhost:9292

这个header 表明这个请求是从哪里发出来的，在哪个域名下。

这时候如果服务器返回的响应头中没有下面的 header 就不会通过跨域验证，则结果会被拦截

	Access-Control-Allow-Origin: *

这里返回 `*` 表明允许任何域名下的请求，这样它就会通过跨域验证。

在第二种场景下，options 请求会带上如下的几个header 

	Access-Control-Request-Method:POST
	Host:www.eiko.me
	Origin:https://www.eiko.me/cors.html
	Referer:https://www.eiko.me/cors.html/

后台响应中包含了如下几个 header 

	Access-Control-Allow-Origin: https://www.eiko.me/cors.html
	Vary: Origin
	Access-Control-Allow-Methods: POST
	Access-Control-Allow-Headers: Origin, Accept, Content-Type
	Access-Control-Allow-Credentials: true
	Access-Control-Max-Age: 30

就会通过跨域验证

简单说就是 Access-Control-Allow-* 中需要包含了 Access-Control-Request-* 中的内容了，才能通过验证。

回到我们刚才的 Java 代码中，这样写确实没有什么问题的，覆盖了常用的 header 就没有什么大问题。我们接下来往下看。


#### 带身份信息的请求(Requests with credentials)

在跨域请求时，浏览器默认是不会带上身份信息(cookie,authorization)，除非前端设定了,如:

	withCredentials:true

如果浏览器返回的响应中包含如下 header :

	Access-Control-Allow-Credentials: true

也会通过验证。

但是这里要注意的是，后台响应中对于带有身份信息的请求时，详情的 `Access-Control-Allow-Origin`，就不能通过 `*` 这个符号来指定了，必须要指定一个真实的域名，这样才能通过验证。

这样回到一开始的 Java 代码中就会发现，加了授权后

	rep.setHeader("Access-Control-Allow-Origin", "*");

这段代码成为了问题。

最后看看 spring 中自带的跨域支持一个代码片段

	public String checkOrigin(String requestOrigin) {
		if (!StringUtils.hasText(requestOrigin)) {
			return null;
		}
		if (ObjectUtils.isEmpty(this.allowedOrigins)) {
			return null;
		}

		if (this.allowedOrigins.contains(ALL)) {
			if (this.allowCredentials != Boolean.TRUE) {
				return ALL;
			}
			else {
				return requestOrigin;
			}
		}
		for (String allowedOrigin : this.allowedOrigins) {
			if (requestOrigin.equalsIgnoreCase(allowedOrigin)) {
				return requestOrigin;
			}
		}

		return null;
	}


#### Reference

[fetch](https://fetch.spec.whatwg.org/#http-cors-protocol)

[w3c](https://www.w3.org/TR/cors/#http-access-control-allow-origin)

[mozilla.org](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS)
