---
layout: post
title: "First Response Without Keyboard Present"
description: ""
category: "2013-10-12"
tags: [iOS开发,objective-c]
---


在iOS开发中，我们大多数是通过`UITextField`,`UITextView`来响应输入的，而且这也满足了绝大部分的需求了。由于需要实现一个响应输入（成为firstreponse），而又不显示系统键盘，今天就大概看了一下文档。


文档介绍得很详细，开发者完全可以自己实现自己的输入系统，只要你subclass一个UIView or UIResponder,然后实现UITextInput协议下的三个委托：

	
	- (BOOL)hasText;
	- (void)insertText:(NSString *)text;
	- (void)deleteBackward;

然后通过在界面上通过CoreText,Text Kit之类的底层一点的库把输入的文字draw出来就行了，大概的实现逻辑就是这样。由于我的需要很简单，文档我没有看得很详细啊（真懒啊）。

我只想要的是把一个控件变成第一响应者，而又不显示键盘，最重要的是这样就能够获取到输入的光标可以不停的闪。这个应该是用户体验的需求，如果你要让用户输入，而当用户点击后没能聚焦到输入点，第一感觉可能不会知道可不可以输入，毕竟我没有显示键盘。


在iOS的view中有这样的一个属性：

	@property (readwrite, retain) UIView *inputView;             

这个就当view变成了第一响应者后会自动弹出来的。既然如此第一感觉就是这多简单啊，直接把它置为nil键盘就不显示啦。好吧，这样的话看看苹果的解释：

> Presented when object becomes first responder.  If set to nil, reverts to following responder chain.  If set while first responder, will not take effect until reloadInputViews is called.

然后再看看对UITextField的inputView的描述：

> The custom input view to display when the text field becomes the first responder.
> If the value in this property is nil, the text field displays the standard system keyboard when it becomes first responder. Assigning a custom view to this property causes that view to be presented instead.
The default value of this property is nil.


看到这个应该就知道置为nil是不可行的，然后我就重写了`inputView`

	- (UIView *)inputView
	{
		static UIView *iv = nil;
		if (iv == nil)
		{
			iv = [[UIView alloc] init];
		}
		return iv;
	}

就这样达到了我想要的效果了。

我想应该还有更好的方法的。
