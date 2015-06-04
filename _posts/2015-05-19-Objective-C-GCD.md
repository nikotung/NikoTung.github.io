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


### GCD中的基本术语 --- Dispatch Object 和 Dispatch Queue

 **Dispatch Object** GCD虽然是c实现的，但是在Objective-C中也是面向对象的形式的去使用。GCD的对象称为`Dispatch Object`,ARC对其也是有效的的(主要是取决于你deploy版本)。

 **Dispatch Queue**。在GCD中是一个很基础也很重要的概念。它就是一个可以接收提交过来的任务，并且按顺序执行的一个对象，简单来说就可以字面理解一样就是一个队列。提交过来的任务可以串行也可以并行去执行，即有并行队列和串行队列。一个并行的队列可以根据当前系统的运行状态决定并行的数量，而串行每次只能执行一个任务，按顺序进行。

### 三种类型的Queue

 * **The Main Queue**: 这个其实就是主线程了，任何提交到Main Queue 来执行的任务最终就是在主线程中执行，所以这个就是一个串行的queue。可以通过`dispatch_get_main_queue()`取到
 * **The Globle Queue**: 是一个并发的Queue,有三种优先级别:high,default,low and background。可以通过`dispatch_get_global_queue`指定优先级来取得;
 * **The Customer Queue**: 通过`dispatch_queue_create`指定一个`dispatch_queue_attr_s`决定是并行还是串行的队列。

### Queue的基本用法

其实大部分GCD中提供出来的API都提供了这样同步(synchronous)和异步(asynchronous)这样两种使用方式。

同步的用法：

		dipatch_sync(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0) , ^{
			[object dosomethingHere];
			}) ;

异步的用法: 

		dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT , 0) , ^{
			[object dosomethingHere];
			});

非常简单的。


### dispatch_group 

> A dispatch group is a mechanism for monitoring a set of blocks. Your application can monitor the blocks in the group synchronously or asynchronously depending on your needs. By extension, a group can be useful for synchronizing for code that depends on the completion of other tasks.

简单说它就是一个可以监控所有提交到group中的block的执行情况的机制。
	
		dispatch_group_async(dispatch_group_t group,
			dispatch_queue_t queue,
			dispatch_block_t block);

把block提交到queue，并把该block和group关联起来，这样在block执行完成后就可以通知到group，同步或者异步的方式

	dispatch_group_wait(dispatch_group_t group, dispatch_time_t timeout); 
	//同步的话就是直接等待，返回0代表在timeout前所有block已经完成了

	dispatch_group_notify(dispatch_group_t group,
		dispatch_queue_t queue,
		dispatch_block_t block); //异步就通过设置一个block在完成后执行



在dispatch_group中，还有一个`dispatch_group_enter` , `dispatch_group_leave`这个两个API。看文档的时候，以为这两个API也会带一个block的参数。

> Calling this function indicates another block has joined the group through
  a means other than dispatch_group_async(). Calls to this function must be
  balanced with dispatch_group_leave().


就是说你调用了这个enter的方法，就是说明了你以另外一种的方式把group的task计数增加了1，而且你必须要手动在调用leave去把计数减1。当group中的task的数量为0后，还是会调用dispatch_group_notify。

这种使用方法给了我们另外的一种在等待多个异步操作完成后才能进行下一步的操作的情况下的一种实现方式。跟dispatch_group_async很类似。


### Dispatch Source

Dispatch Source是一本比较低层的监控API。监控的事件有很多：

* Mach port send right state changes.
* Mach port receive right state changes.
* External process state change.
* File descriptor ready for read.
* File descriptor ready for write.
* Filesystem node event.
* POSIX signal.
* Custom timer.
* Custom event.

在你初始化一个source的时候传入一个类型进去，就能代表去监控某种类型的事件，然后设置该事件触发后的回调事件，基本的使用方法就是这样，具体会有部分差别。

一个需要主要的是`Dispatch Source`在初始化后是出于一种暂停的状态(suspended),所以要启动它需要手动调用`dispatch_resume(source)`。 

> Dispatch sources are created in a suspended state. After creating the
  source and setting any desired attributes (i.e. the handler, context, etc.),
  a call must be made to dispatch_resume() in order to begin event delivery.


这里面的用法大部分都比较是底层的，可能需要另起一片文章去写了。这里件简单介绍一下。


	













