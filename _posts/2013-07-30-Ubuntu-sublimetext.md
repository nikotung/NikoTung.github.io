---
layout: post
title: "Ubuntu install sublimetext 2.0小记"
description: "简述ubuntu下安装sublimetext的过程"
tags: [开发工具]
---


今天把笔记本装上了Ubuntu现在才开始用，感觉已经很落后了。安装完后，第一时间把我最喜欢的文本编辑器sublimetext给安装了，安装过程就今天记录以下。

安装Ubuntu的过程就不说了，我用的是wubi安装，非常非常简单。

安装过程遇到的问题基本是在这里找到答案的。[原文](http://www.technoreply.com/how-to-install-sublime-text-2-on-ubuntu-12-04-unity/)

首先到官网中下载最新版的[sublimetext](http://www.sublimetext.com/2)，我这里下载的是2.0.2.

## Step 1
解压下载来的包，其实解压完后就可以直接使用了，但是为了日后的方便使用还是对安装进行一下配置.进入你下载的包所在目录，可以直接用鼠标就可以解压，也可以通过命令行进行。

                tar xf Sublime\ Text\ 2.0.2\ x64.tar.bz2
                #注意，因为目录名字中有空格所以要加上一个反斜杠“\”

##  Step 2
解压后你将会得到一个sublimetext text 2的文件夹，该文件夹包含了所有要用到的信息，因此我们把它移到一些更合适的地方去比如“/opt/”,"usr/local/"等，这里我就把它放到“/opt/”中去。一样可以直接copy过去或者用命令行。

                sudo mv Sublime\ Text\ 2 /opt/

## Step 3
在实际使用过程中或许你想使用更快的方式启动sublimetext，这里就是把它加了一个类似于快捷键一样的功能。

                sudo ln -s /opt/Sublime\ Text\ 2/sublime_text /usr/bin/sublime
                #这样的话我们就可以在终端中直接输入sublimetext就可以打开了。

## Step 4
把sublimetext设置为所有的文本的默认编辑器。

                sudo sublime /usr/share/applications/defaults.list

把所有的gedit.desktop替换成sublime.destop

sublimetext在Ubuntu中输入中文是有问题的，通过以下方法解决，但依然不完美。
sub
菜单栏 -> Preferences -> File Settings - User，在配置文件中增加：
                
                 “font_face”: “WenQuanYi Micro Hei Mono”

通过安装一个插件：inputhelper后就可以在每一次需要输入中文时同时按下Ctrl+Shift+z就可以调用输入框，输入完后直接按Eenter键就可以了。

当然，安装插件最好是通过Package Control ,如果还没安装Package Control,可以通过ctrl+～，把以下代码放进去回车。安装完就重启。

		import urllib2,os; pf='Package Control.sublime-package'; ipp=sublime.installed_packages_path(); os.makedirs(ipp) if not os.path.exists(ipp) else None; urllib2.install_opener(urllib2.build_opener(urllib2.ProxyHandler())); open(os.path.join(ipp,pf),'wb').write(urllib2.urlopen('http://sublime.wbond.net/'+pf.replace(' ','%20')).read()); print('Please restart Sublime Text to finish installation')
