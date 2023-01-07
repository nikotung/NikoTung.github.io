---
title: "iOS控件素材读取"
description: "动态下载iOS标准控件素材"
categories: "2014-03"
tags: [iOS dev,iOS开发]
---

###缘由

自从iPhone4 问世后，iOS开发时的素材我们都会准备两套，一套是用于非高清的屏幕，一套用于高清屏幕(是前者的两倍)。一般来说，这些素材的图片我们都是在开发的阶段就准备好了，设计的同学会随时提供出来。在这种情况下，都不会有什么问题，但是如果这些素材要从网上下载而来的话，会有什么不同呢?最近我的一个项目中就有这样的一个需求,控件的素材都是要下载。现在我只需要一套高清的素材就好了。把遇到的一些问题，记录一下。

###常规做法

在我们的一般开发过程中，读取控件的图片都用UIImage提供的一些API.这里简单的介绍一下。

##### imageNamed:
简单的做法就是用这个了，只要图片的名字作为参数了，到了高清屏，甚至连图片后缀都省了。注意的是该API是有缓存的，如果你的素材很大，而且不会经常用到的话，可以考虑用别的API。

##### imageWithContentsOfFile ; imageWithData:
如果不希望有缓存的话可以用这两个，把图片路径获取到，作为参数就好了。

其实在这些API中，我们都没有刻意的去读取高清和不高清的素材。那么系统是如何帮我们做到读取不同的图片素材呢？

> Discussion

> This method looks in the system caches for an image object with the specified name and returns that object if it exists. If a matching image object is not already in the cache, this method loads the image data from the specified file, caches it, and then returns the resulting object.

>On a device running iOS 4 or later, the behavior is identical if the device’s screen has a scale of 1.0. If the screen has a scale of 2.0, this method first searches for an image file with the same filename with an @2x suffix appended to it. For example, if the file’s name is button, it first searches for button@2x. If it finds a 2x, it loads that image and sets the scale property of the returned UIImage object to 2.0. Otherwise, it loads the unmodified filename and sets the scale property to 1.0. See iOS App Programming Guide for more information on supporting images with different scale factors.
Special Considerations

>On iOS 4 and later, if the file is in PNG format, it is not necessary to specify the .PNG filename extension. Prior to iOS 4, you must specify the filename extension.

>If you have an image file that will only be displayed once and wish to ensure that it does not get added to the system’s cache, you should instead create your image using imageWithContentsOfFile:. This will keep your single-use image out of the system image cache, potentially improving the memory use characteristics of your app.

看到上面Apple文档的解释了吧。其实它主要是根据屏幕的属性去加载不同的图片。如果是高清屏幕，在加载的时候系统会去查找另外的相同名字的图片但是是以@2x结尾。加载时的一个关键点就是如果图片是以@2x结尾的，它就会把UIImage的scale属性设为2表示高清，否则就是非高清了。

虽然这只是对`imageNamed:`这个API的说明，其实其它几个API也是一样的。

### 解决办法

很简单，就是把下载的图片保存时，在后面加多一个@2x的结尾就好了。这样下载的素材就不会出现模糊，有像素点，素材变大等问题了。