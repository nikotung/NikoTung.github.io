---
layout: post
title: "重载UINavigationController导致iOS7的屏幕滑动返回手势失效"
description: "iOS7 屏幕返回手势失效"
category: “2014-03-31”
tags: [iOS dev,iOS开发]
---

iOS7中苹果对UINavigationController增加了向右滑动返回上一页的的屏幕手势，方便了操作，其实是iPhone5太长了上面的返回按钮不好按啦。

由于在项目中需要需要自定义导航栏的返回按钮，就对UINavigationBar和UINavigationController进行了重载，进本上就能解决问题。但是在后来测试发现iOS7的屏幕返回手势失效了。后来发现是因为自定义了返回的按钮导致的。

###问题
在重载UINavigationController时，我重载了

	- (void)pushViewController:(UIViewController *)viewController animated:(BOOL)animated; // Uses a horizontal slide transition. Has no effect if the view controller is already in the stack.

在这里对每一个进栈的`viewController`自定义自己的返回按钮：

	            [[viewController navigationItem] setLeftBarButtonItem:[viewController backBarButtonItem]];

就这样简单的几句代码，屏幕的返回按钮就失效了。

###原因
查看文档发现在iOS7中，UINavigationController增加了`interactivePopGestureRecognizer`这个属性，该属性就是控制了屏幕的返回操作手势了。我们来看看文档是怎么描述的：

>The gesture recognizer for popping the top item off the navigation stack. (read-only)
The gesture recognizer in this property tracks touch events that indicate the user wants to pop the top view controller off the navigation stack. You can retrieve the object in this property and tie it to one of your own gesture recognizers. For example, if you want to use your own gesture recognizer to signal a pop behavior, you could call its canPreventGestureRecognizer: method and pass the gesture recognizer in this property.

这个手势就是检测用户是否要进行pop操作，只要对这个属性做一下处理就可以解决问题了。


###解决办法

我们只要把手势的委托指向我们自己的`ViewController`，就可以对手势进行操作。在`UINavigationController`的头文件中，我们发现`UINavigationController`并没有显式的实现`UIGetstureRecognizer`的委托(delegate),即看不到`UINavigationController<UIGestureRecognizerDelegate>`这样的代码。所以首先想到的方法就是把手势的delegate指向我们自己的类。

	@interface JVNavigationController : UINavigationController <UINavigationControllerDelegate, UIGestureRecognizerDelegate>
	@end

	@implementation JVNavigationController

	- (void)viewDidLoad
	{

	  if ([self respondsToSelector:@selector(interactivePopGestureRecognizer)])
	  {
	    self.interactivePopGestureRecognizer.delegate = weakSelf;
	    self.delegate = weakSelf;
	  }
	}


就这样屏幕返回的手势又可以工作了。

因为`UINavigationController`的每一次pop和push都是有动画的，如果快速的操作就会由于前一个动画没有完成，后一个动画就开始了就会出错。

	nested pop animation can result in corrupted navigation bar


所以要在每个pop和push完成后才能进行手势的操作，这样就能避免这样的问题了。

我们只要实现`UINavigationControllerDelegate`在每个`ViewController`完全出现了才把手势enable


	- (void)pushViewController:(UIViewController *)viewController animated:(BOOL)animated
	{
	  if ([self respondsToSelector:@selector(interactivePopGestureRecognizer)])
	    self.interactivePopGestureRecognizer.enabled = NO;

	  [super pushViewController:viewController animated:animated];
	}

	#pragma mark UINavigationControllerDelegate

	- (void)navigationController:(UINavigationController *)navigationController
	       didShowViewController:(UIViewController *)viewController
	                    animated:(BOOL)animate
	{

	  if ([self respondsToSelector:@selector(interactivePopGestureRecognizer)])
	    self.interactivePopGestureRecognizer.enabled = YES;
	}


这样问题就好了。

###参考
[iOS 7 Development Tips, Tricks & Hacks](http://stuartkhall.com/posts/ios-7-development-tips-tricks-hacks)