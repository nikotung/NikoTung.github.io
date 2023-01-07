---
title: "Objective-C Class,Object and MetaClass"
description: "Objective-C运行时的一些特性"
categories: "2014-11"
tags: [Objective-C]
---

#### 背景知识

Objective-C是一个面向对象的语言。任何对象都是从类中实例化过来的，对象的isa指针指向实例化该对象的类，该类包含了这个对象的一些基本信息，比如说一些实例方法，还有一些属性等。

类就是对象的描述。

该类包含的方法就是该实例方法的集合，该实例能响应的SEL。当你用实例调用一个方法时(在Objective-C中，就是向这个实例发送一个消息)，最终会被转化成`objc_msgSend()`,该函数会遍历这个实例的类的所有的方法列表，以及它的父类，如果有的话，最终决定去调用哪一个方法(在Objective-C中，调用方法的正确叫法应该是，sent messaget，发送消息)。

在Objective-C中，有一类方法叫类方法。按照我们对method的定义，有方法的必定会有一个对象，而有对象必定对应有一个类。如此推来，Objective-C的类是不是一个对象呢？那它对应的类又是什么呢？

对的，在Objective-C中，每一个Objective-C类都是一个对象。它也有它的isa指针和它能响应的方法。这就是我们所说的类方法。

看看class的定义:

	struct objc_class {
	    Class isa  OBJC_ISA_AVAILABILITY;

	#if !__OBJC2__
	    Class super_class                                        OBJC2_UNAVAILABLE;
	    const char *name                                         OBJC2_UNAVAILABLE;
	    long version                                             OBJC2_UNAVAILABLE;
	    long info                                                OBJC2_UNAVAILABLE;
	    long instance_size                                       OBJC2_UNAVAILABLE;
	    struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;
	    struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;
	    struct objc_cache *cache                                 OBJC2_UNAVAILABLE;
	    struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;
	#endif

	} OBJC2_UNAVAILABLE;

class就是一个结构体。

#### Meta-Class

既然Objective-C中的每一个类都是一个对象，它必定能够通过一个类实例化过来。这就引出了Objective-C的一个meta-class的概念。我们先看看这个:

	NSUInteger hashValue = [NSObject hash];

这时候的hash方法是发消息到NSObject这个类。因为每一个class都是一个对象，所以这样是可以工作的。

因此，class对象的meta－class必须要包含该class能够响应的所有类方法。因为类是对象的描述，因此这是正确的。

所以meta－class就是class对象所指向的class。

所以我们对Objective-C中，对象和类有这样的两条概括:

* 当你向一个对象发送消息的时候，这条消息会在这个对象所指向的class的方法列表中去查找，以便确定最终invoke哪一个方法。
* 当你向一个类发送消息的时候，这条消息会在这个类所指向的meta-class的方法列表中去查找，以便确定最终哪一个方法。

Meta-Class 大概就是这么个样子了。

#### Meta-Class's class

既然meta-class是一个class，而Objective-C中class也是一个对象，那么问题就来了。meta-class的class又是谁呢？或者这么说吧，meta-class的isa指针指向谁呢？

这里我们会引出一个叫"root class"的东西。

一个meta－class的对象就是root class的meta－class实例化过来的。而root meta－class的class就是root meta-class它本身，就是说root meta-class的对象就是root meta-class实例化过来的。

而且每一个meta－class的isa指针都是指向root meta-class的。是不是非常乱了。其实我们这里的root class就可以直接理解为`NSObject`。这里有一张图，或许会让你觉得清晰一点。来自[Greg Parker](http://www.sealiesoftware.com/blog/archive/2009/04/14/objc_explain_Classes_and_metaclasses.html) 

![](http://www.sealiesoftware.com/blog/class%20diagram.pdf)

我这里做了个简单的[例子](https://gist.github.com/NikoTung/d1a30efeeacc6d1d5df4#file-gistfile1-m)，你会发现通过`object_getClass()`最终会指向`NSObject`



### 参考

[Hamster Emporium archive](http://www.sealiesoftware.com/blog/archive/2009/04/14/objc_explain_Classes_and_metaclasses.html)

[CocoaWithLove](http://www.cocoawithlove.com/2010/01/what-is-meta-class-in-objective-c.html)



