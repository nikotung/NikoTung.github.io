---
layout: post
title: "iOS 8 Launch Screen"
description: "在iOS 8 中使用xib或者storyboard文件创建启动画面"
category: “2014-10-10”
tags: [iOS8]
---


在iOS开发中，Apple为了能够让App启动时第一时间给用户留下一个好"印象"，用静态的图片做了一个启动画面(Launch Image)。这个静态只要按照一定的命名规则和尺寸大小，就能自动的适配不同的机器。随着Apple不断推出新的机器，如果还是按照这种方式的话，在App的Bundle中这样的静态图片就会越来越多，而且是随着App编译后发布出去的。按照目前的机器来说大概就有:320x480,640x960,640x1136,750x1334,1242x2208（不算横屏），理论上来说如果要全部支持的话就要这么多张Launch Images。


为了解决这个问题，从iOS 8开始支持通过xib或者storyboard文件来创建Launch Image的方式。我觉得这个可能是Apple是为了后面不同屏幕的设备做准备吧，可谓远见。

在Xcode 6中，只要你新建一个项目，里面的模板就会自动的帮你创建一个叫LaunchScreen.xib的文件，用来做启动画面。当然你也可以自己创建一个：通过New File->User Interface,然后选择Launch Screen。

其实也可以创建一个storyboard来当作启动画面。

创建完文件后，在项目的target中的General下面有一个App Icons and Launch Images。在这里可以选择我们的启动画面：xib文件或者storyboard文件。

创建完后在info.plist文件中会多一个:Launch Screen interface file base name 的key

这样我们就可以在对应文件中拖一些控件去启动画面中，开发者也可以做启动画面了。（在storyboard中还是要拖一个viewcontroller过去的）

在Launch Screen中通过autolayout的作用就可以去适配不同的设备(其实就是6以上的)。

如果通过这样的方式去创建启动画面，是不是意味着我们也可以写代码去控制不同的启动画面呢？

答案是不可以的。我们还是不可以自己写一个自定义的view然后替换掉xib或者storyboard中的view。其实这时候app的还没有真正启动起来，代码还没有动。而且这样做也是没有效果的。你想想我们把这个view改成一个UIWebView,这样启动岂不是很坑爹。这应该也是Apple的这种方式的一个限制吧。


虽然xib是可以向后兼容的，但是通过这样的方式创建的启动画面在iOS8以下，还是起不了作用了，如果你的app要支持8以下的话，还是要像原来那样添加Launch Images.
