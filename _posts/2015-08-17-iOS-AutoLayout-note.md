---
layout: post
title: "AutoLayout 和 VFL-- 还债篇"
description: "iOS的 AutoLayout 理解，以及所带来的DSL －－ VFL"
category: "2015-08"
tags: [AutoLayout]
---

AutoLayout（自动布局） 是 iOS 6 中带入的一个特性，本质就是为了能够解决多设备的多分辨率适配问题了。在其中起关键性作用的就是 *constraint* 。

看看文档对 AutoLayout 的描述

> Auto Layout is a system that lets you lay out your app’s user interface by creating a mathematical description of the relationships between the elements. 

所以 AutoLayout 的约束，你就可以理解为一种数学表达式。

举个例子，比如你要定义一个按钮 button 让它的左边缘于它的容器的左边大20个像素，可以这样描述了，button.left ＝ container.left + 20。如果再多几个这样的表达式，是不是我们高中还是初中时学习过的线性约束很像，根据条件求解。

把表达式修改一下

	y = m*x + b

* x 和 y 是 view 中的一个属性
* m 和 b 是一个常量

x 和 y 的可能值有 left, right, top, bottom, leading, trailing, width, height, centerX, centerY, and baseline。

通过x y 的组合就能满足各种各样的条件了。

用代码就在可以这样写了

	NSLayoutConstraint(item: button, attribute: NSLayoutAttribute.Left, relatedBy: NSLayoutRelation.Equal, toItem: superview, attribute: NSLayoutAttribute.Left, multiplier: 1.0, constant: 20.0)                              

`NSLayoutConstraint`是 iOS 6中新增的一个类，用于创建约束条件的。

创建了约束条件后必须要要把它作用于 view 上面。`UIView`在 iOS 6中新增了下面几个API

    @availability(iOS, introduced=6.0)
    func addConstraints(constraints: [AnyObject]) // This method will be deprecated in a future release and should be avoided.  Instead use +[NSLayoutConstraint activateConstraints:].
    @availability(iOS, introduced=6.0)
    func removeConstraint(constraint: NSLayoutConstraint) // This method will be deprecated in a future release and should be avoided.  Instead set NSLayoutConstraint's active property to NO.
    @availability(iOS, introduced=6.0)
    func removeConstraints(constraints: [AnyObject]) // This method will be deprecated in a future release and should be avoided.  Instead use +[NSLayoutConstraint deactivateConstraints:]

* 同一层级的两个 view 的约束要加到他们最近的父节点中去。
* 有层级关系的两个 view 的约束要到地底的那个 view中去(superview)
* 同样的约束条件，不同的约束实例会叠加

但是在 iOS8 中 上面几个 API 都被废弃掉了。改用`NSLayoutConstraint`的几个 API,还有一个 active 的属性对应remove

	[NSLayoutConstraint activateConstraints:]
	[NSLayoutConstraint deactivateConstraints:]

这两个  API 其实更好用，不再需要考虑 view 的层级关系。想约束生效就直接 active 不想生效就 deactive。

接下来看看 Apple 为了解决新建约束的 API 比较长的问题，引入了一个 DSL，visual formate language 简称 VFL

标准间隔 (button 和 textField 的间隔为 8)，

	[button]-[textField]
	[button]-20－[textField] # 间隔变成了 20
	[button][textField] # 间隔变为 0 ，水平方向两个元素是紧凑的

通过 － 来连接两个元素

宽度约束

	[button(>=50)] # button 的宽度大于等于 50
	[V:button(>=50)] # button 的高度大于等于 50

在元素后面加括号表示宽度/高度约束，默认不加方向(V : H)就是水平方向(H)，V 代表垂直方向(vertical)

跟 superview 建立约束
	
	|-50-[button]-50- | # button 和 superview 的左右间隔为50 
 
`｜`代表是 superview

关系(relation)

	[button1(==button2)] # 两个按钮之间的宽度一样
	[button1(>=100)] # button1 的宽度大于等于100
	[flexibleButton(>=70,<=100)]  # 按钮的宽度大于 70 小于 100

这里的关系可以有 == ，<=，水平方向两个元素是紧凑的>=。如果有多个关系(约束)通过逗号`,`隔开

优先级

	[button1(>=100@100] # button1 的宽度大于等于100,优先级为 100

优先级越大越先满足，最大为1000。

`NSLayoutConstraint`中有一个API，

	    class func constraintsWithVisualFormat(format: String, options opts: NSLayoutFormatOptions, metrics: [NSObject : AnyObject]?, views: [NSObject : AnyObject]) -> [AnyObject]

可以用 VFL 去增加约束。

在 [Apple](https://developer.apple.com/library/prerelease/ios/documentation/UserExperience/Conceptual/AutolayoutPG/VisualFormatLanguage/VisualFormatLanguage.html#//apple_ref/doc/uid/TP40010853-CH3-SW11) 的文档中,描述了 VFL 的语法。里面有一些 metric，number，这样的东西，可以结合这个API来了解。

* 第一个参数`format` 这个不用说了，就是 VFL 所描述的关系
* 第二个参数`options`,是一个枚举类型 `NSLayoutFormatOptions`,主要作用就是使在第一个参数`format`中出现的元素用这个属性去对齐或者方向规定。
* 第三个参数`metrics`,是一个字典，里面的key要跟第一个参数中能够对应上，就是一些额外的参数，写在这里就可以不用 hardcode 在 VFL 上面去了。
* 第四个参数`views`,也是一个字典，包含了第一个参数中所有出现的 view，里面的 key 也要跟它对应。


我们看个例子

	let metrics = ["margin" : 20.0, "priority" : 100]
    let views :[String : AnyObject] = ["imageView" : imageView!,"label" : label!]
    let contraints = NSLayoutConstraint.constraintsWithVisualFormat("H:|-margin-[label]-margin-[imageView]-margin@priority-|", options: NSLayoutFormatOptions.AlignAllCenterY, metrics: metrics, views: views) as! [NSLayoutConstraint]

    //the contraints 
	[<NSLayoutConstraint:0x7fb109c3ae60 H:|-(20)-[UILabel:0x7fb109c35190'This is the text']   (Names: '|':UIView:0x7fb109f36a50 )>, <NSLayoutConstraint:0x7fb109c399d0 H:[UILabel:0x7fb109c35190'This is the text']-(20)-[UIImageView:0x7fb109f36070]>, <NSLayoutConstraint:0x7fb109c3b2d0 H:[UIImageView:0x7fb109f36070]-(20@100)-| priority:100   (Names: '|':UIView:0x7fb109f36a50 )>, <NSLayoutConstraint:0x7fb109c3b280 UILabel:0x7fb109c35190'This is the text'.centerY == UIImageView:0x7fb109f36070.centerY>]

Apple 中会有一个 VFL 的一个解析器，把 `format`里面的东西一个一个的解析成多个约束。

在`metrics`中定义了`margin`和`priority`,分别为20.0 和 100，然后在 vfl 中就可以用这两个 string 代表 这两个数值。

看看`options` 这个属性，用了`NSLayoutFormatOptions.AlignAllCenterY`，会被解析成一条这样的约束

	<NSLayoutConstraint:0x7fb109c3b280 UILabel:0x7fb109c35190'This is the text'.centerY == UIImageView:0x7fb109f36070.centerY>]

这样就比较好理解了，这个参数的作用就是使得在 `format`中出现的每一个元素的该属性一样。

再添加这样一条约束

	NSLayoutConstraint(item: label!, attribute: NSLayoutAttribute.CenterY, relatedBy: NSLayoutRelation.Equal, toItem: self.view , attribute: NSLayoutAttribute.CenterY, multiplier: 1.0, constant: 20.0)
	//label 的 centery 在 superview 的 centery 的下面 20 像素点

	//下面的约束可以跟上面的做到一样的效果
	NSLayoutConstraint(item: imageView!, attribute: NSLayoutAttribute.CenterY, relatedBy: NSLayoutRelation.Equal, toItem: self.view , attribute: NSLayoutAttribute.CenterY, multiplier: 1.0, constant: 20.0)

在上面中，设置了`label`和`imageView`的 centery 的关系。下面只需要把`label`和`imageView` 中的任意一个跟`superview`做关联，这样就能实现`label`和`imageView`联动。



#### Reference

[Auto Layout Guide](https://developer.apple.com/library/prerelease/ios/documentation/UserExperience/Conceptual/AutolayoutPG/Introduction/Introduction.html#//apple_ref/doc/uid/TP40010853-CH13-SW1)

[AutoLayout & VFL](http://objc.co/2014/11/26/AutoLayout-VFL/)
                      
