---
layout: post
title: "IOS下單元測試"
description: "簡述一些ios下的單元測試庫"
category: “2013-07-22”
tags: [单元测试]
---

自从xcode4整合的单元测试后，我才知道要在ios下也要做单元测试的，之前也是完全没有接触过这个测试。其实一开始用xcode4的时候，就看到有了单元测试这个功能了，但是一直都没有留意，知道最近项目后期的一些空闲时，才留意了一下，现在顺便整理一下。

在ios下除了xcode中自带的OCUnit外，还有其它的一些第三方的选择如：
+ [GHUnit](https://github.com/gabriel/gh-unit/downloads)
+ [OCMock](http://ocmock.org/#download)

如果之前有做过单元测试的，可以按照之前的习惯去做，感觉区别不是很大。如果还没有做过单元测试的话，其实个人建议就直接使用OCUnit,使用起来很方便，既可以在项目初建时直接勾选加入，也可以在你写完代码后，再添加测试。

很多人觉得ios自带的OCUnit的控制台输出太乱了，很难看清楚。一开始我也是这么觉得的，然后就试了一下*GHUnit*,因为觉得它有界面可以看，其实发现，也就是在界面中显示了哪些测试用例没有通过，所以我才说其实区别都是不大的。当然，一定要承认的是[GHUnit](https://github.com/gabriel/gh-unit/downloads)和[OCMock](http://ocmock.org/#download)提供的是比较多的功能。

接下来，当然我是不会手把手的截图，贴图，一步一步的说怎么做。只要你看过官方的文档，你就会发现是很简单的，而且里面有更详细的介绍你怎么做。这里我主要的是讲到OCUnit。

我把官方的文档地址贴上来吧。

+ [OCUnit](https://developer.apple.com/library/mac/#documentation/DeveloperTools/Conceptual/UnitTesting/00-About_Unit_Testing/about.html)

+ [GHUnit](http://gabriel.github.io/gh-unit/docs/index.html)

+ [OCMock](http://ocmock.org/tutorials/)

### 使用的一些注意事情
+ OCUnit 提供了 Logic tests 和Application tests.logic test只能在模拟器上运行。如果你在项目初建的时候就添加了单元测试，这时添加的就是application tests。

+ test case 命名：test < test_case_name > ,并且还是需要无参无返回.

+ 执行顺序。程序员嘛，都是一步一步走的，我写了一个测试用例，怎么调用它呢。在添加单元测试的时候，其实就已经有了`setup`和`teardown`两个函数，这两个函数有点类似于`init` 和 `dealloc`。所以说每一个test case在进行测试前都会先调用`setup`完毕后就会调用`teardown`

网上看到一篇[文章][1]，说作者为何使用ocunit而不是ghunit的，观点挺有意思。




[1]: http://longweekendmobile.com/2011/04/15/unit-testing-in-xcode-4-use-ocunit-and-sentest-instead-of-ghunit/




