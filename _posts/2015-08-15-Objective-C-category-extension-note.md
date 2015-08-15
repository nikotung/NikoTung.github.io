---
layout: post
title: "Objective-C 的 category 和 extension"
description: "Objective-C 学习笔记"
category: "2015-08"
tags: [Objective-C]
---

我们都知道 Objective-C 中有 category 这个语言特性。能够让你在即使没有源代码的情况下，对一个已经存在的 class 增加一些新的方法。

#### Category

这个 feature 其实是非常有用的。因为在面向对象的语言中，一般需要增加新功能的话，常规的做法是子类化一下，然后新增的方法。然而在一般情况下，子类化一下作用不是很大的。有了 category 这个功能后可以让我们实现这样的功能：

* 避免子类化
* 一次增加所有子类都能用
* 模块化功能

一般来说，增加 category 都是在一个独立的文件中进行的。这样就能把一个类的大小减少了。

#### 一个要注意的地方

在 category 中增加的方法，要避免方法名重复。因为 Objective-C 的方法调用并不是在编译期决定的，而是在运行时决定的，是 message send 的，如果是同样的方法，就没法确认了。


#### Extension

其实当时是看到 Yelp 公司开源的一个 tableview 的框架[YLTableView](https://github.com/Yelp/YLTableView)时才注意到的一个点，才想起写这些东西的。


一般来说，我们的习惯做法是在实现文件中作 extension 的定义，在这里定义的一些 property 和 method 都是不会暴露到外面给别的类去调用的，实现一个私有的功能。而且这里面定义的方法，必须在类的实现文件中去实现。

而在这个框架中，它把 extension 放到了一个独立的头文件中去，然后在实现文件中引入，再实现。再然后又在别的类中引入，去调用。一开始觉得有点违背了，之前的了解，然后就去翻 Apple 的文档。发现这种做法也是对的，只不过都不太推荐。

> If you intend to make “private” methods or properties available to select other classes, such as related classes within a framework, you can declare the class extension in a separate header file and import it in the source files that need it. It’s not uncommon to have two header files for a class, for example, such as XYZPerson.h and XYZPersonPrivate.h. When you release the framework, you only release the public XYZPerson.h header file.


就这样了。




#### Reference

[Programming with Objective-C](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/CustomizingExistingClasses/CustomizingExistingClasses.html)


