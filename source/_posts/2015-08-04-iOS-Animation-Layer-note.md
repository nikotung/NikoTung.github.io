---
title: "iOS 动画 -- layer"
description: "iOS 动画，layer 的简单理解"
categories: "2015-08"
tags: [CALayer ,Animation]
---

## iOS 动画

这只是一个简单的验证了一下在 iOS 的动画中并没有真正的改变`Layer`的一些属性。

在`CALayer`中有一个叫`presentationLayer`的一些属性。
	
>The layer object returned by this method provides a close approximation of the layer that is currently being displayed onscreen. While an animation is in progress, you can retrieve this object and use it to get the current values for those animations.

这就是说这返回这个跟我们实际操作的`layer`极度相似的`layer`对象。并且在动画的作用过程中，我们通过这个对象可以查看到`layer`的实时属性变化。

在一个动画中，通过`CAAnimation`的子类，如：`CAPropertyAnimation`,`CAKeyframeAnimation`和`CABasicAnimation`创建一个动画对象，对这个动画对象的一些属性的设置来定义动画的形式等，最后把这个动画 add 到 layer 上。这样我们就能看到 layer 的动画。能看到了动画，直观来说我们都认为 layer 的某些属性被改变了，所以认为是动画改变了 layer ，然而并不是这样的。

	var animation = CABasicAnimation(keyPath: "position.y")
	animation.toValue = 300.0
	animation.duration = 1.0;
	layer.addAnimation(animation, forKey: "positionAnimation")

上面这样的一个动画，最后完成后，layer 会瞬间返回到原来的位置。这就跟我们的认知对不上了。

> An app using Core Animation has three sets of layer objects. Each set of layer objects has a different role in making the content of your app appear onscreen:

> * Objects in the model layer tree (or simply “layer tree”) are the ones your app interacts with the most. The objects in this tree are the model objects that store the target values for any animations. Whenever you change the property of a layer, you use one of these objects.
* Objects in the presentation tree contain the in-flight values for any running animations. Whereas the layer tree objects contain the target values for an animation, the objects in the presentation tree reflect the current values as they appear onscreen. You should never modify the objects in this tree. Instead, you use these objects to read current animation values, perhaps to create a new animation starting at those values.
* Objects in the render tree perform the actual animations and are private to Core Animation.


看到有 model layer, presentation layer 和 render layer,而 presentation layer是我们在屏幕上看到的动画的 layer 了。所以动画并没有直接修改 layer 的属性。presentation layer ，这个在动画进行的时候，我们可以获取到，然后可以通过它来拿到整个动画过程中，layer 的属性的变化。

所以，上面的动画中只对 layer 的 presentation layer 有影响，并且在动画结束后马上恢复到它原来的状态，这就是我们看到的动画在最后瞬间返回原来的位置的原因了。

再来看看，Apple 文档中对 presentation layer 的一个note: 

> Important: You should access objects in the presentation tree only while an animation is in flight. While an animation is in progress, the presentation tree contains the layer values as they appear onscreen at that instant. This behavior differs from the layer tree, which always reflects the last value set by your code and is equivalent to the final state of the animation.

看最后一句，从正常的动画来说，动画的最后状态应该跟我们直接操作的 layer 属性一致的(可以这么说吧，presentation layer 的最后位置，就是我们期望我们操作动画的 layer 的最后位置是一样的)。

既然想要达到这样，那么最终还是得修改 model layer 的属性。所以上面的动画，最终的状态应该是这样的

	var animation = CABasicAnimation(keyPath: "position.y")
	animation.fromValue = layer.position.y
	animation.duration = 1.0;
	layer.addAnimation(animation, forKey: "positionAnimation")

	layer.position.y = 300.0

还有这样的一种解决方案

	animation.fillMode = kCAFillModeForwards //The receiver remains visible in its final state when the animation is completed.
    animation.removedOnCompletion = false  // whether the animation is removed , when it finishes

但是这种方案有这样的一个问题：在最终的layer中，如果你点击后，它是无法通过`hitTest:`,但是在layer的最初始位置点击就能通过了。

因为之前遇到过这样的问题，我对一个 button 进行一个动画，并使用后面的解决方案，发现按钮点击没有效果，只能最初始位置点击时，才能出发 button 的 target 事件，之前一直没有搞懂，现在终于有点眉目了。


我这里有一个小的 [demo](https://github.com/NikoTung/Layer-Animation-Issue) 可以演示一下

![](/assets/2015-08-04-layer-animation.gif)













### Reference 


[oreAnimation guide](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/CoreAnimation_guide/CoreAnimationBasics/CoreAnimationBasics.html#//apple_ref/doc/uid/TP40004514-CH2-SW13)

[Prevent Layers From Snapping Back to Original Values When Using Explicit CAAnimations](http://oleb.net/blog/2012/11/prevent-caanimation-snap-back/)

[Core Animation基本概念和Additive Animation](http://studentdeng.github.io/blog/2014/06/24/core-animation/)

[Animations Explained](http://www.objc.io/issues/12-animations/animations-explained/)

[Hit testing animating layers](http://ronnqvi.st/hit-testing-animating-layers/)