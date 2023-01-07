---
title: "App 提交的一些问题"
description: "app 提交的一些问题"
categories: "2015-12"
tags: [itunesconnect]
---

好久没有提交过App ,最近两年基本上没有用过新版的 iTunes connect。这两天在准备提交，也遇到了一些问题，这里就简单纪录一下吧。以免再踩同样的坑。

尽量不要用 Xcode 上的提交功能，反正没有成功过，每一次都是卡在 "Sending API usage to itunes connect",这个问题Google 一下一堆人在问。各种退出账号，重启Xcode ，重启电脑。反正我是不信了，直接用 "Application Loader"就好了。

**Potential Loss of Keychain Access**。由于我上传的 App 是从别的账号转移过来的，用了新的账号生成证书 provisioning profile 编译上传的。因为这时线上的版本和我现在的Apple 账号的team 是不一样的所以证书的teamID 也是不一样了。所以就会有这个警告，当然这只是一个警告，所以不用太担心。可以用下面的命令去看看 provisioning profile 的信息。

    security cms -D -i /path/to/AppStoreProfile.mobileprovision

当前上传的版本支持的设备不应该比前一个版本少。举例说，你的上一个版本支持了iPad，下一个版本应该是无条件支持iPad。

> An update to an app must work for every customer who has already purchased the app, and is running a current version of iOS.

在支持iPad的时候也要考虑是否要支持多任务。支持多任务就要支持设备旋转，多方向。



Reference 

* [Technical Note TN2318 Troubleshooting Failed Signature Verification](https://developer.apple.com/library/ios/technotes/tn2318/_index.html#//apple_ref/doc/uid/DTS40013777-CH1-ROOTCAUSE)
* [Technical Q&A QA1623 Why am I getting device support errors when uploading my app?](https://developer.apple.com/library/ios/qa/qa1623/_index.html)
* [Technical Q&A QA1726 Resolving the Potential Loss of Keychain Access warning](https://developer.apple.com/library/ios/qa/qa1726/_index.html)




