---
layout: post
title: "Git command a day - gitignore"
description: "checkout command"
category: "2014-09"
tags: [gitignore]
---

今天在使用git新建项目时犯了一个错误，折腾了些许时间，所以在这里记录一下。

如果我们在github，gitcafe等托管平台的时候，一般新建项目时都会建议我们加入readme和.gitignore这两个文件。我的错误就是在于没有建立.gitignore文件。

我在后台建立一个个项目后，直接就clone下载，然后在本地把项目等搭建起来。在项目搭建起来后，我们的习惯一般是提一个init的commit。

在提这个commit前我看了一下当前的状态，发现很多本来就不需要加入版本控制的文件都加进去了，仔细看了一下后发现我还没有加.gitignore文件。接下来就把.gitignore文件加上。再看了一下`git status`,除了多一个.gitignore文件外，没有任何改变，让我感到非常奇怪。先看看我的.gitignore文件的内容吧。

	# Xcode
	.DS_Store
	*/build/*
	*.pbxuser
	!default.pbxuser
	*.mode1v3
	!default.mode1v3
	*.mode2v3
	!default.mode2v3
	*.perspectivev3
	!default.perspectivev3
	xcuserdata
	project.xcworkspace
	profile
	*.moved-aside
	DerivedData
	.idea/
	.svn/
	

再看看的我项目状态：

	Niko@NikoTungs-MacBook-Pro Bicycle (master) $ git status
	On branch master
	Your branch is up-to-date with 'origin/master'.

	Changes to be committed:
	  (use "git reset HEAD <file>..." to unstage)

		new file:   Bicycle/Bicycle.xcodeproj/project.pbxproj
		new file:   Bicycle/Bicycle.xcodeproj/project.xcworkspace/contents.xcworkspacedata
		new file:   Bicycle/Bicycle.xcodeproj/project.xcworkspace/xcuserdata/Niko.xcuserdatad/UserInterfaceState.xcuserstate
		new file:   Bicycle/Bicycle.xcodeproj/xcuserdata/Niko.xcuserdatad/xcschemes/Bicycle.xcscheme
		new file:   Bicycle/Bicycle.xcodeproj/xcuserdata/Niko.xcuserdatad/xcschemes/xcschememanagement.plist
		new file:   Bicycle/Bicycle/BRAppDelegate.h
		new file:   Bicycle/Bicycle/BRAppDelegate.m
		new file:   Bicycle/Bicycle/Bicycle-Info.plist
		new file:   Bicycle/Bicycle/Bicycle-Prefix.pch
		new file:   Bicycle/Bicycle/Images.xcassets/AppIcon.appiconset/Contents.json
		new file:   Bicycle/Bicycle/Images.xcassets/LaunchImage.launchimage/Contents.json
		new file:   Bicycle/Bicycle/en.lproj/InfoPlist.strings
		new file:   Bicycle/Bicycle/main.m
		new file:   Bicycle/BicycleTests/BicycleTests-Info.plist
		new file:   Bicycle/BicycleTests/BicycleTests.m
		new file:   Bicycle/BicycleTests/en.lproj/InfoPlist.strings

	Changes not staged for commit:
	  (use "git add <file>..." to update what will be committed)
	  (use "git checkout -- <file>..." to discard changes in working directory)

		modified:   Bicycle/Bicycle.xcodeproj/project.pbxproj
		modified:   Bicycle/Bicycle.xcodeproj/project.xcworkspace/xcuserdata/Niko.xcuserdatad/UserInterfaceState.xcuserstate

	Untracked files:
	  (use "git add <file>..." to include in what will be committed)

		.gitignore
		Bicycle/Bicycle.xcworkspace/
		Bicycle/Podfile
		Bicycle/Podfile.lock


奇怪吧，我已经把xcuserdata加到.gitignore中了，可以Bicycle/Bicycle.xcodeproj/project.xcworkspace/xcuserdata/Niko.xcuserdatad/UserInterfaceState.xcuserstate折让的文件依然还在，让我感到有点意外。一时也没有想到是为什么，只能用最简单的方法把一个一个这样的文件移除掉。

	Niko@NikoTungs-MacBook-Pro Bicycle (master) $ git reset HEAD Bicycle/Bicycle.xcodeproj/xcuserdata/Niko.xcuserdatad/xcschemes/Bicycle.xcscheme
	Unstaged changes after reset:
	M	.gitignore
	M	Bicycle/Bicycle.xcodeproj/project.pbxproj
	M	Bicycle/Bicycle.xcodeproj/project.xcworkspace/xcuserdata/Niko.xcuserdatad/UserInterfaceState.xcuserstate

突然间发现，原来为什么gitignore没有生效，因为那些要忽略的文件已经加到了stage了，所以后面才加上去的gitignore不生效。所以要把那些文件unstage掉。

所以，在新建项目时，如果要加入版本控制的话，务必要第一时间加上.gitignore文件。

这里还有一个关于CocoaPods的问题。

我的新项目是用了CocoaPods管理依赖，我加了[OCMock](http://ocmock.org/)这个测试框架进去。然后我简单的在项目自带的test target中测试一下看能不能用。一运行发现test target根本无法访问pods中的文件

	OCMock.h  ...file not found


后来查了一下[CocoaPods](http://cocoapods.org/)的官网后发现，原来我们用简单的podfile模版的话，只能在我们的product的target中才能访问引用的第三方库。如果要在测试的框架中使用引用的需要在podfile中配置一下。

	target 'xxxxtarget', :exclusive => true do
	pod 'OCMock' 
	end

这样你的的测试或者新加的target才能访问引用的第三方库。


##### Reference

[CocoaPods](http://guides.cocoapods.org/syntax/podfile.html#target)