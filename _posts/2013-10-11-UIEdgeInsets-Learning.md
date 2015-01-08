---
layout: post
title: "UIEdgeInsets 小记"
description: "basic understand about UIEdgeInsets"
category: "2013-10"
tags: [UIEdgeInsets]
---


今天出于好奇，想自己写一个UITabBarController,于是就开始动手,于是也就遇到了在tabbar中如何只用一个控件就能把图片和标题显示成标准的baritem一样。

看到了UIButton既有有图片又有标题，于是就从它动手。

如果我们对一个button既设置了图片又设置了标题的话，它是如下显示的：

![](/assets/2013-10-11-UIEdgeInsets-1.png)

看到button中有一个`titleEdgeInsets`和`imageEdgeInsets`.

注：在iOS中，UIEdgeInsets跟web中的padding一样的分top，left，bottom和right，正数就表示与周围的距离变大，例如：默认在是0，我把top变成了10，这时该元素与顶端的距离就会增加10，即元素下移，距离变大。如果是负数的话效果就相反了，距离拉近。

通过上面的图，可以看到图片和标题是在一起水平的，这说明button中的图片和显示标题的label在同一个水平线上，看到这个，写过前端css的应该很容易就把它适配成我们想要的效果吧。

注意一点的是，这是我们的整个窗口是在button里面，就button这么大。

通过下面的代码

	   [_nextButton setImageEdgeInsets:UIEdgeInsetsMake(0, 0, 14, -_nextButton.titleLabel.bounds.size.width+10)];
    	[_nextButton setTitleEdgeInsets:UIEdgeInsetsMake(30, -[_nextButton currentImage].size.width+3, 0, 0)];

简单说明一下：

由于图片和标题原本是水平的，它们是压缩在同一个宽度中的，因此，对于图片的话我们需要把它向右边拉长到标题label的宽度，而对于标题则需要把它向左拉到图片的宽度。

效果图如下：(更精确的对齐，可能还是需要再微调一下)



![](/assets/2013-10-11-UIEdgeInsets-2.png)

