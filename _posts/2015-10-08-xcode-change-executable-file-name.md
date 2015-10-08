---
layout: post
title: "Xcode 修改 App 的 executeable file name"
description: "升级了 Xcode 7升级的各种问题"
category: "2015-10"
tags: [Xcode]
---

十一期间，把系统重装了一遍，Xcode 也重新装了一遍。重新打开打开项目的时候发现无法编译安装到模拟器和真机上。原来是修改 info.plist 文件导致的问题。

之前就是因为Xcode 的一次升级导致了各种奇怪的问题，就非常郁闷了。想着十一没啥事情，恰逢 OS X 船长发布，就把机器清理了一遍，重新升级了系统，整个过程除了下载船长很慢之外，就没有什么问题了。系统升级完后，顺带就把 Xcode 给升级了到7.0.1，这过程也没有啥问题。

放完假回来后，打开项目编译时出了一个很奇怪的问题。

![](/assets/2015-10-08.png)

一开始还真以为是权限的问题，因为我真的修改了一下 Mac 的用户，然而并不是这个问题。后来 [Google](http://google.com) 了一下发现有可能是修改了 info.plist 文件中的 executable file name 引起的。重新修改成了 `$(EXECUTABLE_NAME)` 后问题就解决了。

但是，我确实需要修改这个名字的，因为在代码中我要读取这个名字。后来发现在项目文件中其实有一个选项可以修改这个名字的。就是在 `Build Settings` 中有一个叫 **PRODUCT_NAME** 的选项可以修改，在这里修改就可以达到效果了，当然也要在 target 中做同样的修改。












#### Reference

[http://apple.stackexchange.com/questions/159866/project-file-cant-be-build-error-you-dont-have-permission-to-view-it](http://apple.stackexchange.com/questions/159866/project-file-cant-be-build-error-you-dont-have-permission-to-view-it)

[http://www.cocoabuilder.com/archive/xcode/244606-target-properties-executable-field.html](http://www.cocoabuilder.com/archive/xcode/244606-target-properties-executable-field.html)