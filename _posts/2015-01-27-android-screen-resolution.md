---
layout: post
title: "Android分辨率的一些基础知识和理解"
description: "Android开发中，能够理解好分辨这个东西是非常有用的"
category: "2015-01"
tags: [Android,分辨率]
---

在Android开发过程中，分辨率这个东西非常重要。会影响在不同机型，分辨率下的效果，外观以及对素材的使用。

因为是从iOS开发过来的，一开始也曾不太理解为何Android中有各种各样不同的表示方式，px,sp,ps,dp等等。既然是无法避免要用到的，只能要学习一下了。

### 基本的一些概念

#### 像素&分辨率

这个应该比较好理解。它是显示器上显示的最小单位，图像是由无数的像素点组成的，是非连续的。平常说的320×480分辨率指的就是像素，屏幕物理大小上所具有的像素点。


####  屏幕大小(in,英寸)

平常所说的屏幕大小就是以英寸为单位的，通常是指屏幕对角线的长度，每英寸等于2.54厘米。在Android中根据屏幕的大小分成了四种:(一开始我在想具体的标准是什么，后来发现这是规定的，感性的划分)

* small，小
* normal，中
* large，大
* extra-large，超大

#### dpi & 屏幕密度(density)

dots per inch，每英寸像素点数。屏幕密度就是在一定物理的屏幕大小中所包含的像素点的数量。比如320X480分辨率的手机，宽2英寸，高3英寸, 每英寸包含的像素点的数量为320/2=160dpi（横向）或480/3=160dpi（纵向）。如果横向和纵向的dpi不一样呢？其实就是通过勾股定律去算的，按对角线为准。

在Android中，根据屏幕密度的不同又可以分为6种不同的屏幕(是不是觉得很复杂，前面已经有了四种):

* low ~120dpi
* medium ~160dpi
* high ~240dpi
* extra-high ~320dpi
* extra-extra-high ~480dpi
* and extra-extra-extra-high ~640dpi

这四个值可以通过`DisplayMetrics`读取出来。因为只分为了这六种，如果我们根据对角线算的话就不一定跟这六个值匹配，比如有可能算出140dpi这样的一个值，在low和medium中间，这个算哪个呢？最终的取值应该是向上去的，大于low小于medium取medium，同样的也适用于其它的dpi。

density和dpi的关系为 density = dpi/160


#### dp (device independent pixels)设备独立像素

这是android的一个特有的单位，和设备的硬件有关系，一样的dp在不同的设备上是有可能不一样的。在屏幕密度dpi = 160屏幕上，1dp = 1px (a dp = b *(dpi / 160))。像素和pid的转换:px = dp * (dpi / 160) ＝ dp  * density。

>A virtual pixel unit that you should use when defining UI layout, to express layout dimensions or position in a density-independent way.

>The density-independent pixel is equivalent to one physical pixel on a 160 dpi screen, which is the baseline density assumed by the system for a "medium" density screen. At runtime, the system transparently handles any scaling of the dp units, as necessary, based on the actual density of the screen in use. The conversion of dp units to screen pixels is simple: px = dp * (dpi / 160). For example, on a 240 dpi screen, 1 dp equals 1.5 physical pixels. You should always use dp units when defining your application's UI, to ensure proper display of your UI on screens with different densities.



#### sp (scaled pixels)放大像素

主要处理字体的大小。与dp类似，但是可以根据用户的字体大小首选项进行缩放。主要用于字体显示best for textsize。由此，根据 google 的建议，TextView 的字号最好使用 sp 做单位。


### 多屏幕支持

#### Drawable

在一般的新建项目中的res下面都会有如下的Drawable文件夹:

* drawable
* drawable-ldpi(dpi = 120,density = 0.75)
* drawable-mdpi(dpi = 160,density = 1)
* drawable-hdpi(dpi = 240,density = 1.5)
* drawable-xhdpi(dpi = 320,density = 2)
* drawable-xxhdpi(dpi = 480,density = 3)

同样像素大小的图片在屏幕密度低的手机上反而变大，在屏幕密度高反而变小。这个比较好理解吧，因为同样的在同样的尺寸下包含的像素点不一样(前者少，后者多)，为了显示一样的图片，自然就会被放大和缩小了。

为了能够达到不同的屏幕下一样的显示效果(dip就是为了干这个的)，一般的做法是在不同的drawable目录下放一个同一样的图片不同尺寸。


### 总结

对基本概念的理解就差不多了。





