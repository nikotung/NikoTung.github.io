---
layout: post
title: "NSDate 与时间戳转换时的一些问题"
description: "日期其实都可用用时间戳去表示，只不过你要注意时区的问题"
category: "2016-07"
tags: [NSDate , 时区]
---

前天一位同事问我为啥这样一段代码运行后拿到的对象不是他想要的日期，而且还跟 Android 不一样 

	NSDate *birthday = [NSDate dateWithTimeIntervalSince1970:-2209017600000/1000];
	//输出是 : 1899-12-31 16:00:00 +0000

时间戳`-2209017600000`是后台(Java)返回的，因为日期是数据库表新增的一个字段一个`datetime`类型。没有初始化所以时间都是`1900-01-01 00:00:00`。看输出结果，跟实际结果相差8个小时，很明显的就是时区的问题了。

#### NSDate 的时区

我们先看看文档对 `NSDate` 的描述 

> NSDate objects encapsulate a single point in time, independent of any particular calendrical system or time zone. Date objects are immutable, representing an invariant time interval relative to an absolute reference date (00:00:00 UTC on 1 January 2001).

创建 `NSDate` 对象时，它是跟时区无关的,而且它里面的 timeInterval 是跟 [UTC](https://zh.wikipedia.org/wiki/协调世界时) 相关的。

意思就是你用一个时间戳去初始化一个`NSDate` 时，它会跟根据UTC 这一个时区去计算时间，这样拿到的实例就会与时区相关了。

举个例子:

如果你在伦敦获取当前时间与1970之间的时间间隔，和在北京获取当前时间与1970之间的间隔，得出来的时间是不一样的，这就是时区的原因。你在北京是当前中午十二点，而人间就是在凌晨4点了。


#### java.util.Date 的时区

如果你看Java 中的 Date 源码，你会发现它也没有时区个东西在里面，只是包涵了一个到 1 January 1970, 00:00:00 UTC 时间的间隔，以毫秒为单位。

*那为啥我们拿到的时间戳格式化出来后跟 Android 的不一样，而且还跟后台数据库的不一样呢？*

#### 原因

在调试 Java 的`Date`时,你会发现调用`toString()`方法时，结果会带上时区的信息。因为它会在这时候做一次`normalize`的操作。同样的它在调用`getTime()`方法时，也会有这个操作，所以拿到的时间戳会带上了时区的信息了。

这时后台返回过来的数据的时区信息就是东八区(北京)的信息。

至于 Android 为啥会正确呢？因为同事看到的是格式化后的日期。怎么说了 Android 用了`SimpleDateFormat`，而这个拿到的是当前用户所在的时区(东八区),这就说明为啥他没有问题了。

同样的 在iOS 中通过`NSDateFormatter`去格式化，看到的，我们认为错误的时间，也能格式化成正确的时间，因为它默认拿到用户所在的时区。


关于这个问题，一开始都定义好了要用GMT(UTC),然而后台和Android 的同事都没有留意到这个时区的问题，而因为他们都是用 Java 才没有发现问题。

