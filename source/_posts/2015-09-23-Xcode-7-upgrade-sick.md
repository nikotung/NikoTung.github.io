---
title: "Xcode 7 闹心一次的升级"
description: "升级了 Xcode 7升级的各种问题"
categories: "2015-09"
tags: [Xcode,Exception BreakPoint]
---


事情源自于在家无聊，打开 App Store 看到 Xcode 可以升级到7,点了一下 Get ,然后发现，速度很不错，这么难得的机会当然是捉紧时间升级啦，果然很快就升级好，然后欢天喜地的出门了。没想到隐藏了这么多问题在这 Xcode 7里面。

在用新的 Xcode 时，偶尔会卡顿，新版本嘛，这个你懂得，忍忍就算了。偶尔还会 crash ，忍忍也就算了。

接下来在调试代码的时候，竟然出现了这么一个异常，

	terminate_handler unexpectedly threw an exception

完全没有任何提示，更别说定位在哪个代码里面，当时有点心慌啊。还好习惯了每一段代码成功了就 commit 一次。在这个异常下，新增的代码不多，然后就试着回滚，一个一个去调试。发现了是一个 autolayout 的 VFL 中的一个拼写错误，导致无法解析。厉害吧，很快就定位到问题。

等等，好像哪里不对的样子啊，用过 autolayout 的同学都知道，Xcode 会有很明确的错误原因 log 打出来，就算那些有歧义的，不满足条件的 autolayout 警告也会提示出来的。怎么这次，抛这么一个 谁也看不懂的异常出来。

深深的陷入了沉思啊，我特么做了什么啊。

* 升级了Xcode 7 后，发现之前的插件，无效了，然后就重新手动装了一遍；
* 想在Xcode 7中增加，iOS 8的模拟器，在Xcode 目录下放了一个.sdk的文件，后面发现无效；

一开始，我是断定是手动装插件时，因为用 cp 和tar 命令时报了一个错，各种乱码信息，各种乱七八糟的log 。

接下来，我就把 所有的插件删除，连那个iOS 8的模拟器也删了。

发现还是不行。那是不是不是我导致的问题呢？

这时候，又怀疑，是不是电脑的问题，以及升级 Xcode 的时候有问题呢，之前 6.4版本是通过下载 dmg 文件安装的，更新到 7 时是通过 Mac App Store 去更新安装的。

手写 autolayou ，没有这种提示，是很坑的一件事情啊，没法忍受啊。

我还是重新下一个 Xcode，把现有的彻彻底底的删掉，然后重新安装一个。别问我怎么彻彻底底删掉的，都是泪。

重新安装 Xcode 7 后，发现还是不行啊。天啊，好坑。

这样的话，是不是 Xcode 7 真的就已经把这个错误信息，和歧义警告不打印出来呢？ 就差把电脑重装，去验证了。

不知道有没有谁也有同样的问题。

这种 autolayout 的异常，没有错误提示，不能忍啊。有什么办法能把异常打印出来呢？

有了，`Exception BreakPoint`,应该可以把它打印出来。

#### 添加 Excepiton BreakPoint 

于是兴高采烈的加了一个 Exception BreakPoint ,在 Action 中用 Debugger Command 这个命令 `po $arg1` 打印。

没想到，出来这么一个 错误提示 

	error: use of unresolved identifier '$arg1'

这个在Xcode 6 中明明是好的，不要问我明明是谁。

这能在 `objc_exception_throw` 的这个断点中的符号一个一个试试了。看看下面的图片

![](/assets/2015-09-23-xcode-7.png)

调试后发现在 Xcode 7 中要用 `po $eax` 这个去打印，这样就能把异常打印出来了 。

	Unable to parse constraint format: 
    Encountered metric with name "margiin", but value was not specified in metrics or views dictionaries 
    H:|-margiin-[_iconImageView(width)]-margin-[_shopNameLabel]-margin-[_startingFeeLabel]-margin-| 

久违的异常警告啊。


不过这种方式，只能在导致 crash 的情况下才能把生效，那种约束有歧义，条件不满足的警告还是没法打印出来。




























