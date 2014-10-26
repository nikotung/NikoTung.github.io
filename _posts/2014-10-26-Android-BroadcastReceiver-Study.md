---
layout: post
title: "Android BroadcastReceiver初试"
description: "在Android中如何简单的使用BroadcastReceiver"
category: “2014-10-26”
tags: [Android,BroadcastReceiver]
---

最近开始要做Android的项目，其实是跟着项目马上就要开发功能了，所以只能边做边学习了。用到啥学啥，纪录啥，没有系统的学习过Android开发。刚开始就用到了广播，所以就先学习一下了。

广播嘛，顾名思义就是广而播之，在它的播送范围内都能知道的。

在Android中，BroadcastReceiver就是接收通过`sendBroadcast()`发送的Intent。

#### 广播的类型

在Android中，有同步和异步的两种广播类型:

 * 同步广播(Normal broadcast)：广播发送后，所有注册为同步广播的接收者都会同时收到广播
 * 异步广播(Ordered broadcast):广播发送后，依次的一个一个传送到接收者，并且能返回一个能影响下一个接收者的结果，也可以中断广播的继续发送`abortBroadcast()`

这样也就对应了两种广播的发送方式:`sendBroadcast()`,`sendOrderedBroadcast()`

####注册接收者

接收者的注册有两种:静态注册和动态注册。

所谓静态就是写在Android的配置文件中(AndroidManifest.xml)以`<receiver>`开头，而且名字中必须要有一个对应的实现类。比如:

	<receiver android:name="com.niko.receivers.BasicBroadcastReceiver"
                  android:label="简单的广播接收者">

在我们的receivers包下面必须要有一个`BasicBroadcastReceiver`实现类继承`BroadcastReceiver`。

所谓动态注册就是通过Java代码来写`registerReceiver()`。如上面的静态注册可以通过`registerReceiver(new BasicBroadcastReceiver,new IntentFilter("niko.basicbroadcastreceiver"))`。这样就能接收到广播了。

在注册和发送的时候，IntentFilter的action必须要一致才能接收。而且在动态注册的时候，`registerReceiver()`和`unregisterReceiver()`需要成对出现。


####跨App接收广播

一旦你注册了一个广播接收者，任何App都有可能发送广播给它。同样的你发送广播的时候，其它的App也可能接收到这个广播。这是一个安全性的问题。如果你不想你的接收者接收外部广播，你可以在静态注册的时候加上`android:exported="false"`。在发送的时候，也可以通过intent的setPackageName()去指定谁可以接收。通过权限的控制，在发送和注册的时候都可以做权限处理。

####接收者的生命周期
如果一个App是已经被关掉的，这时候其它的App发送了一个广播，这时候接收者只有在执行onReceive函数时有效，一旦完成就不再激活状态。
所以任何异步的操作都不应该在这里进行，因为一旦退出函数后系统是随时会把进行杀掉，







