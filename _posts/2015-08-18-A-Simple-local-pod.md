---
layout: post
title: "一个简单的本地pod"
description: "创建一个简单的本地 pod"
category: "2015-08"
tags: [Cocoapods]
---

如果你是一个 iOS 开发者，却没有听说过 [CocoaPods](http://cocoapods.org) 那么真的需要去了解一下。它是一个 iOS 第三方库的依赖管理工具，可以轻松便捷的管理这些库，而不用整天折腾项目文件。而现在 iOS 的各种各样的开源库遍地花开，基本上能满足你的日常需求，作为一个代码搬运工，这个工具就能帮我们节省太多的时间了。

这里我本着学习如何创建私有 pod 的过程，创建了一个本地的 pod 。这个本地 pod 很简单的，就是一个专门管理支付宝的 SDK 用的，而且我只在本地建立一个 git 的项目，并没有建立远端的 specs repo。

#### 第一步创建 Podspec 文件

Podspec 文件是啥了，看看官方文档怎么说的。

> A Podspec, or Spec, describes a version of a Pod library. One Pod, over the course of time, will have many Specs. It includes details about where the source should be fetched from, what files to use, the build settings to apply, and other general metadata such as its name, version, and description.

一句话，就是一个库的描述文件。很简单就通过`pod spec create <name>` 这个命令，指定 pod 的名字就可以在本地创建一个 Podspec 文件，创建出来的文件是一个模版，需要的信息，模版中都有，只需要把它替换成自己的就行了。	主要就是包括下面的几个模块内容:

* Root specification : 是描述这个库的某一个版本的基本信息，版本，作者，来源等等，里面有些是要必填的(可以验证是否有哪些缺失)
* Platform : 库所支持的平台(iOS OS), 以及最低的发布环境。
* Build settings : 编译时的一些属性设置
* File patterns : 一种匹配模式，决定哪些文件，framwork , library等会引入或不引入到 pod 中
* Subspecs : 所以依赖的子库
* Multi-Platform support : 多平台支持 iOS , OS X，watch OS

#### 为 Podspec 文件添加库的描述

这个其实就可以简单的根据模版去修改。而因为我只是在本地建立的 git 项目，所以在 source 这个里面稍微有点不一样，这个一般来说是指向远端的代码管理服务器，比如

	spec.source = { :git => 'https://github.com/AFNetworking/AFNetworking.git',
                :tag => spec.version.to_s }

而我这里只能这样填

	  spec.source = { :git => "/Users/Niko/Projects/Mobile/AliPay/.git" , :tag => s.version.to_s }

还有一个是，最好给项目打个 tag ，这个在 source 里面用到了。不然会有警告的。
如果有依赖的系统的 library，填写的时候要把 lib 的前缀去掉。

#### 验证 Podspec 文件是否正确

pod 本身就提供了验证的功能，简单运行命令 

	pod spec lint AliPay.podspec 

一般会加上一个参数
	
	pod spec lint AliPay.podspec --verbose

这样出错了，会提示我们具体的原因。


把我的 Podspec 文件内容贴上来

	Pod::Spec.new do |s|

	  s.name         = "AliPay"
	  s.version      = "0.0.1"
	  s.summary      = "支付宝的支付SDK"

	  s.description  = <<-DESC
	                   支付宝的支付SDK,建立一个pod去管理
	                   DESC

	  s.homepage         = "https://github.com/NikoTung"


	  s.license      = { :type => 'MIT' , :file => "LICENSE.md" }

	  s.author             = { "Niko" => "Nizh2008@gmail.com" }
	  s.source           = { :git => "/Users/Niko/Projects/Mobile/AliPay/.git" , :tag => s.version.to_s }


	  s.platform     = :ios, '7.0'

	  #  When using multiple platforms
	  # s.watchos.deployment_target = "2.0"


	  
	  # s.source       = { :git => "http://EXAMPLE/AliPay.git", :tag => "0.0.1" }

	  s.vendored_frameworks = 'Framework/AlipaySDK.Framework'
	  s.resource = "Resource/AlipaySDK.bundle"

	  s.frameworks = "SystemConfiguration", "CoreTelephony"
	  s.libraries = 'z'


	end
		

#### Reference 

[Specs and the Specs Repo](https://guides.cocoapods.org/making/specs-and-specs-repo.html)

[Podspec Syntax Reference](https://guides.cocoapods.org/syntax/podspec.html#specification)

