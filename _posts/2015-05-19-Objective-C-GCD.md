---
layout: post
title: "GCD -- Grand Central Dispatch"
description: "什么是GCD,GCD怎么使用...."
category: "2015-05"
tags: [Objective-C,GCD]
---

在用Objective-C 进行多线程编程的话，或多或少都会涉及到GCD。它是在iOS4和OS X 10.6开始引入的，在这里对它做一个概括性的学习了解。

### 什么是GCD

GCD是Grand Central Dispatch的简称，是一个比较低层的并发编程的API。现在已经开源出来了[libdispatch](http://libdispatch.macosforge.org)。GCD有点像NSOperationQueue,但是又不属于Cocoa Framework，允许程序分离到不同的task中去并发或者顺序执行。

GCD的使用大部分都是基于block的，如果你不了解block的语法的话，可以看看Apple的[文档](https://developer.apple.com/library/prerelease/ios/documentation/Cocoa/Conceptual/Blocks/Articles/bxGettingStarted.html#//apple_ref/doc/uid/TP40007502-CH7-SW1)