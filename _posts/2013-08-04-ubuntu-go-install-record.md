---
layout: post
title: "Ubuntu 安装go语言开发环境小记"
description: ""
category: "2013-08-04"
tags: [开发工具]
---

新手上路，最近在学习go语言开发，在配置开发环境时遇到了一些问题，今天就暂时做一下记录。有于是在折腾安装成功后才写这篇文章的，所以不会有太多的案发现场证据，如截图什么之类的。我的安装环境是在ubuntu13.04桌面版。

由于我之前是使用apt-get安装的，安装完后发现版本是1.0.2，并不是我希望的1.1.1版，因此才有了下面的折腾。

###  apt-get安装
用这种方式安装是最简单的，但是安装出来的结果不是最新版的go，所以安装后我把给删除了,而且这种方式默认的安装路径是在/usr/lib/go下面。

                #  may need admin permission
                apt-get install golang 

### 源码安装
##### step1
源代码安装的话，需要自己编译首先需要安装一些编译的工具。

                # admin permission may needed
                apt-get install bison ed gawk gcc libc6-dev make

go语言的源代码获取，很多人都建议使用go的源码管理工具Mercurial去获取。但是因为国内的环境问题，我还是建议你直接上网把适合自己机器的源代码下载来然后自己解压安装。我就是这样做的。

##### step2
下载了源代码后，把源代码解压到你要安装的目录下，我这里安装的是/usr/local/,通过下面的命令就可以把压缩包解压到你想要安装的目录

                tar -C /usr/local -xzf go1.1.1.linux-amd64.tar.gz

##### step 3
解压后最重要的是设置gopath，这是最重要的，在后面项目编译中都需要用到。去到home目录下打开.bashrc文件输入如下变量：

                export GOROOT=/usr/local/go
                export GOARCH=amd64
                export GOOS=linux
                export PATH=${PATH}:/usr/local/go/bin
                export GOPATH=~/Gits/gostudy
其中GOROOT是你安装的路径，GOARCH是根据你下载的包不同而不同的我这里是64位,GOOS根据你系统不一样而不一样其实还是根据包不同了，
PATH,这个是最重要的了，我们需要把安装路径加到PATH中去并在后面加上/bin即把“$GOROOT/bin”加到PATH中去。
GOPATH这个是你项目的路径，这个在后面可以改，可以每一个项目一个GOPATH，但是你要通过：叠加，不然始终还是只有一个的。
我个人的话就是只设置了一个GOPATH，因为目前我的GOPATH目录下面全都是go语言的项目，只需要在GOPATH下面新建一个src文件夹存放项目源代码就可以了。

保存后，需要使用那个如下命令才能使gopath生效

                source .bashrc

如无意外，这是你在终端中输入go就会看到相关的go命令。这就说明你的go是安装成功了。

上面说到我只设置了一个gopath，所有的项目都放在gopath下的src下，这其中我也遇到了一些编译的问题出现，也记录一下。
我一开始是在src下建立一个gotest的项目，但是gotest下面就是一些包或者说是文件夹,我在gotest下面有一个qfile文件夹，该文件夹下面就是程序入口的go文件（只有一个）。这是我就会直接编译gotest，但是这时是编译不成功的。只能通过下面这样才能编译成功。

                go build gotest/qfile 

在实际开发中，我们每写一个bao最后都把该包给install一下，编译是先把每一个包给编译一下，最后才build程序入口的主包，最后通过install就可以在bin目录下生成可执行文件。