---
layout: post
title: "iOS7多任务和Background Fetch"
description: "iOS7 后台运行"
category: “2014-04-10”
tags: [iOS7,iOS开发]
---

iOS7增加了很多新的特性，这么多的特性我也是最近才开始研究起来，是不是有点慢了，传说中的iOS8都要出来了。

一般来说App进入了后台后都会被系统挂起，不会执行任何的代码，而且如果当前在前台运行的app需要更多的内存时，在后台挂起的App随时都会被干掉。在iOS4起其实就已经支持了多任务的特性了，可以让App在后台长时间运行，当时Apple要求是：
* VoIP
* Audio
* Location
* Newsstand
这四种类型的App才应该这样做。而一般的App只需要在App进入后台申请后台运行一段时间，然后等着被系统挂起。

在iOS7中增加了一个后台fetch的功能，系统会不定时的把你的App唤醒给你30秒的后台运行时间，你必须要在30秒内完成操作，不然你的App接下来获得后台运行的机会越来越少，甚至会被挂起。

###App的状态
* Not running：app还没有被启动或者app已经启动过了但是被系统干掉；
* inactive：app运行在前台，但是没有接收任何事件，这个状态是很短暂的，可能是由于app将要切换到不同的状态；
* active：app运行在前台也能接收所有的事件，这是在前台运行的app的正常状态；
* Background：app进入到后台并在执行一些代码，这可能是短暂的由前台切换到后台时的一个状态，这是大部分app进入到后台并在将被系统挂起前的一个状态。一些需要长时间后台运行的app这个状态的时间就会比较长。一个被用户手动启动的app当按home键后会直接进入到background而不是inactive；
* Suspended：app进入到后台，但是不会执行任何代码，这是app只是被挂起了，但还是会占有内存，但是一旦系统的内存不足时，app随时会被干掉


###检查当前设备是否支持多任务(Determining Whether Multitasking Is Available)
在`UIDevice`有这么一个属性`multitaskingSupported`可以标明是否支持多任务

	UIDevice* device = [UIDevice currentDevice];
	if ([device respondsToSelector:@selector(isMultitaskingSupported)])
	   backgroundSupported = device.multitaskingSupported;


###有限时间长的后台运行（finite－length）
当用户按了home键后，App的delegate`applicationDidEnterBackground:` 方法就会被调用。该方法有5秒钟时间去运行，该方法尽量要在5秒钟内退出。如果我们有更长时间的事情操作，这时我们就应该向系统申请更长的后台运行时间。下面的两个方法都可以申请后台运行时间：

	- (UIBackgroundTaskIdentifier)beginBackgroundTaskWithExpirationHandler:(void(^)(void))handler  NS_AVAILABLE_IOS(4_0);
	- (UIBackgroundTaskIdentifier)beginBackgroundTaskWithName:(NSString *)taskName expirationHandler:(void(^)(void))handler NS_AVAILABLE_IOS(7_0);

一个是iOS4以上可以用，一个是iOS7以上可以用。这两个方法都带有一个`block`的参数，这个`block`就是当这个后台运行结束后就会被调用，在这个`block`中必须要调用`- (void)endBackgroundTask:(UIBackgroundTaskIdentifier)identifier NS_AVAILABLE_IOS(4_0);`,因为这个`block`如果是被系统调用，说明这个后台运行的时间快要结束了，防止你的任务还没有完成，为了保险需要在`block`中调用。

Sample

	- (void)applicationDidEnterBackground:(UIApplication *)application
	{
	    bgTask = [application beginBackgroundTaskWithExpirationHandler:^{
	        
	        [application endBackgroundTask:bgTask];
	        bgTask = UIBackgroundTaskInvalid;
	    }];
	 
	    // Start the long-running task and return immediately.
	    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
	 
	        // Do the work associated with the task, preferably in chunks.
	 
	        [application endBackgroundTask:bgTask];
	        bgTask = UIBackgroundTaskInvalid;
	    });
	}


###长时间后台运行(Long-Running Background Tasks)
对于那些需要长时间在后台运行的App，需要向系统申请对应的权限，一般来说我们都是在info.plist文件中的`UIBackgroundModes`增加对应的值就好了，如audio，location。

这个主要说一个iOS7新增的Background Fetch，因为其它的都还没有真正用过。

#####Fetching Small Amounts of Content Regularly

在iOS7中，App可以在后台定时的去获取网络信息，但是需要申请权限，其实就是在info.plist的`UIBackgroundModes`中增加一个`fetch`的值就好了，如果是用xcode5的话可以直接在xcode的targets中的Capabilities中直接勾选。

首先我们要在app启动的时候设置后台获取的间隔`[[UIApplication sharedApplication] setMinimumBackgroundFetchInterval:UIApplicationBackgroundFetchIntervalMinimum];`。看起来是可以自己设置自己需要的间隔，实际上我们根本没法确定系统让我们后台激活的间隔是什么。

当App进入后台后，获得后台执行权限的时候，UIApplication的delegate就会被调用。

	//ios7 background fetch feature
	- (void)application:(UIApplication *)application performFetchWithCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler
	{
	    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0), ^{
	        [[JVMessageManager sharedInstance] fetchMessagesInBackgroundModeWith:completionHandler];
	        //handler everything here 
	    });
	}

这个后台运行的时间最多也就只有三十秒，所以必须要在三十秒内完成，不然App就很有可能被挂起了。当我们的后台任务完成后必须要显式的调用completeHandler。这是一个block接收一个`UIBackgroundFetchResult`的类型参数。

	typedef NS_ENUM(NSUInteger, UIBackgroundFetchResult) {
    UIBackgroundFetchResultNewData,
    UIBackgroundFetchResultNoData,
    UIBackgroundFetchResultFailed
	} NS_ENUM_AVAILABLE_IOS(7_0);

不确定这三个值对下一次后台运行的时间有什么影响。

这样我们就能够完成了一个iOS7的后台获取的功能了。

在调试的时候，我们可以通过Xcode的Debug菜单下的Simulate Background Fetch 来触发，当然也可以通过在edit scheme中Duplicate Scheme然后勾上Backgroung Fetch这个选项就好了。




















