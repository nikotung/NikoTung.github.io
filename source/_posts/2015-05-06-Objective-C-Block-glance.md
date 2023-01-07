---
title: "Block 是什么来的"
description: "Objective-C 中的block是什么东西，有什么用呢"
categories: "2015-05"
tags: [Objective-C,Block]
---

今天早上“复习”了一下Block,发现我竟然无法顺利的写出一个Local Block ,瞬间好吃惊，打击很大。

其实一直都知道Objective-C的Block语法很难看，很恶心。但是没有想到恶心到我都忘记了(哈哈)。记得有一个网站[fuckingblocksyntax](http://fuckingblocksyntax.com)。

所以稍微复习一下这个语法。

主要是在作为参数和local 变量时的定义有点不一样。先看一样苹果文档的图片: 

![](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/Blocks/Art/blocks.jpg)



看看下面的代码


	- (void)sendAsynchronousRequest:(void (^)()) handler
	{
   		void (^aBlock)(id) = ^(id g){
    	};    
    	void ^cBlock = ^{
   		 };   
   		aBlock(@(1));   
    	id (^bBlock)(id) = ^id(id g){
  	      return nil;
   		};
   		id a = bBlock(@(122));
   	}
    
同样是一个无返回类型，不带参数的block，在作为方法的参数定义时也要把那个参数的括号加上，但是在local中的时候就可以忽略了。好像就差不多了。

关于block的好处什么的，就不说了。

















	