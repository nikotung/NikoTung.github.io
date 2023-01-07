---
title: "Mac OS X 安装phonegap的android和iOS的问题"
description: "Problems ocurred during config phonegap"
categories: "2013-10"
tags: [phonegap]
---


目测以后要用到phonegap进行跨平台开发，今天有空就先把开发环境给搭建起来，来个简单的入门。去到[phonegap](http://docs.phonegap.com/)的官网，看到对初学者的安装是有比较全面的文档，第一感觉还是不错的。

基本的环境要求还是比较容易安装的，只要按照文档进行就可以了，如[node.js](http://nodejs.org),命令行等。

这基本的安装完毕后，只要在命令行中输入`phonegap`，看看输出就能够确定是否安装成功。

在安装过程中，遇到最麻烦的就是多平台的整合，在我的安装过程中主要整合的iOS和android两个平台，另外我的机子是mac的。

因为如果你的本地没有安装你所需要的平台的sdk的话，编译phonegap时会进行远程编译，把你的项目上传到adobe服务器，所以我还是把所需要的sdk都安装了。

#### 整合iOS

由于我是在mac下的，所以整合iOS很简单，只要你安装了xcode和xcode的command tool-lines(如果你是在用4.x，这个你可以在xcode的preference的donwload中下载，如果你用xcode5，很不幸在xcode中找不到，但是你只要在终端中输入`xcode-select --install`,就可以进行安装了)。如无意外，你在你的phonegap项目下增加一个iOS的平台，然后就可以编译一个xcode的项目。

	phonegap build ios

#### 整合Android

整合安卓就是更加的麻烦了，因为我刚升级了OS X 到 Mavericks,Java的sdk全部都被kill了，所以还是需要先把Java环境搭建起来。只要在终端中输入java就可以提示你安装了。这个就不说了。

1.下载Android 的sdk

我这里主要的是用了adt，把sdk和ide的都下载来直接解压就可以用了。[下载地址](http://developer.android.com/sdk/index.html)

解压后需要把你的sdk路径放到path中去。如果根目录下面没有这个文件".bash_profile"的话就新建一个（在系统根目录下），然后输入如下命令：

	export PATH=${PATH}:/Users/dev/AndroidDevelop/adt-bundle/sdk/platform-tools:/Users/dev/AndroidDevelop/adt-bundle/sdk/tools

最后source之，`source .bash_profile`,使其生效。

一般来说这样就可以进行整合到Android中去。但是当我使用如下命令生成安卓项目时却报错了。

	cordova platform add android

抛出如下错误：


	[Error: Please install Android target 17 (the Android 4.2 SDK). Make sure you have the latest Android tools installed as well. Run `android` from your command-line to install/update any missing SDKs or tools.]

应该是我的adt中的Android sdk没有4.2版本，于是在终端中输入an`android`就打开了 Android sdk manager,发现确实只是安装了4.3版本的sdk。但是更令人痛心的是看不到其他版本sdk的安装链接，我了个去，难道要另外下载然后导入，这也太坑爹了吧。遂Google之，发现是由于被墙的原因，获取到其它版本的sdk，WTF.乐观点，只要修改一下hosts文件就可以了，没有完全墙掉。

打开hosts文件，增加下面那一行，保存后，重新打开Android sdk manager就会看到4.2和其它版本的sdk。
	
	 74.125.237.1       dl-ssl.google.com  

下载完sdk后，就可以开始整合到Android了。进入到项目中再次使用如下命令

	cordova platform add android

WTF.还是报错了。说什么 ant jar 编译错误。原来我的apache-ant还没有安装

apache-ant 安装有两种方法，一个是用命令行，一个是下载压缩包；

	sudo port install apache-ant

我是用压缩包安装的，首先去下载.[地址](http://ant.apache.org/srcdownload.cgi)

然后是解压，最后使用命令行把解压后的包放到/urs/local 下，这个需要root权限

	mv apache-ant-1.9.2 /usr/local

最后一步，把包放到path中去。

我还是打开 .bash_profile,放入如下命令，最后source之。

	export ANT_HOME=/usr/local/apache-ant-1.9.2/
	export PATH = ${PATH}:${ANT_HOME}/bin

在命令行中输入`ant`,如果输出如下信息则安装成功

	Buildfile: build.xml does not exist!
	Build failed

这会整合Android应该没有问题了。进入项目中，还是输入如下命令

	cordova platform add android

oh，还是继续报错了。

	An error occured during creation of android sub-project. An unexpected error occurred: "$ANDROID_BIN" create project --target $TARGET --path "$PROJECT_PATH" --package $PACKAGE --activity $ACTIVITY >&/dev/null exited with 1
	Deleting project...

快不行了。都不知道什么原因

最后Google后发现，这是phonegap3.0.0的一个bug，具体请看[这里](https://github.com/phonegap/phonegap-cli/issues/98)

解决方法如下：

打开cordova的一个script：

	vim /Users/dev/.cordova/lib/android/cordova/3.0.0/bin/create

找到如下语句：
	
	"$ANDROID_BIN" create project --target $TARGET --path "$PROJECT_PATH" --package $PACKAGE --activity $ACTIVITY &> /dev/null

把它去掉后面的 `&> /dev/null`改成如下：

	"$ANDROID_BIN" create project --target $TARGET --path "$PROJECT_PATH" --package $PACKAGE --activity $ACTIVITY

保存，然后继续整合。注意到这时config.xml文件中的app的名字不能又空格。

到这里，Android的整合算是完成了。


