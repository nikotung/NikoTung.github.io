---
layout: post
title: "多个git仓库配置多个ssh key"
description: "使用ssh连接git"
category: “2014-09-04”
tags: [ssh key]
---

都知道连接git仓库，一般来说有这么几个方式：

* https
* ssh
* Subversion (这是github特有的吧)

https的话可能要经常输入密码，ssh就是在初次使用的时候配置一下key,以后就不需要再输入密码了。在一些托管平台中，一些私有的项目中默认的都是建议用ssh，这样是会稍微安全一点，当然一个比较好的做法是要定时更新ssh key把一些已经不用的key去掉。


如果我们在一台电脑上第一次使用git时，如果用了ssh的方式去clone一个项目，会出现这样的一个问题：无法建立安全链接。

	Cloning into 'XXX'...
	The authenticity of host 'github.com (207.97.227.239)' can't be established.
	RSA key fingerprint is e3:ee:82:78:fb:c0:ca:24:65:69:ba:bc:47:24:6f:d4.
	Are you sure you want to continue connecting (yes/no)? yes
	Warning: Permanently added 'github.com,207.97.227.239' (RSA) to the list of known hosts.
	Connection closed by 112.124.6.106
	fatal: Could not read from remote repository.



### 解决办法
首先我们在本地生成一个RSA的私钥，然后把公钥取出来放到远程仓库的ssh公钥key中去

	$ ssh-keygen -t rsa -C "your_email@example.com"
	Generating public/private rsa key pair.
	Enter file in which to save the key (/Users/you/.ssh/id_rsa): [Press enter]
	Enter passphrase (empty for no passphrase): [Press enter]
	Enter same passphrase again: [Press enter]
	The key fingerprint is:
	b8:45:68:90:10:b0:61:5e:cf:05:7e:b8:5e:b4:7f:75 Nizh2008@gmail.com
	The key's randomart image is:
	+--[ RSA 2048]----+
	|ooo+.o..         |
	|oo. =.o.         |
	|..   =oo.        |
	|     .+o.        |
	|     ..oS    . E |
	|    . .o.   . .  |
	|     ..  . .     |
	|          .      |
	|                 |
	+-----------------+

把ssh key增加到ssh－agent中去

	ssh-add ~/.ssh/id_rsa

如果没有意外，你就会看到一个identify....的信息

接下来在.ssh目录下面应该多了两个文件：`id_rsa`,`id_rsa.pub`,我们需要的就是把公钥取出来

	cat ~/.ssh/id_rsa.pub

把打印出来的串，一字不漏的拷贝下来放到你的git仓库账号中去。

再接下来，做一下测试

	$ ssh -T git@github.com
	The authenticity of host 'github.com (207.97.227.239)' can't be established.
	# RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
	# Are you sure you want to continue connecting (yes/no)? [enter yes]
	Hi yourusername! You've successfully authenticated, but GitHub does not
	# provide shell access.

不用管输出的什么错误，需要确认的地方直接yes就好了。你看到你的名字就说明成功了。

这样我们就可以用ssh的方式使用git了。

但是这并不是我最近遇到的问题。

### 原因

前两天注册了开源中国的[代码托管平台](http://git.oschina.net/),主要是它免费提供私有仓库的托管，而我用私有仓库又不是非常的频繁，所以就试用一下。当我在上面建立了项目，想要clone下来的时候发现没法建立链接，原来我没有为开源中国的git建立ssh key，而我本地已经有了.ssh下面的id_rsa文件。所以必须要配置多套的ssh key。


### 多个ssh key解决方案

在上面我们提到生成RSA私钥时，需要我们输入保存私钥的文件名，而我是直接就回车键，这样我就用了默认的文件/Users/you/.ssh/id_rsa。所以在生成多个ssh key时要指定不同的文件。这里我就用了/Users/you/.ssh/oschina_rsa

还是一样，要把oschina_rsa加到SSH Agent中

	ssh-add ~/.ssh/oschina_rsa

好了之后我们要在.ssh下面加一个配置文件config，用于配置不同的git服务器。内容如下:

	# github
	Host github.com
	HostName github.com
	User git
	IdentityFile ~/.ssh/id_rsa
	IdentitiesOnly yes

	# oschina 
	Host git.oschina.net
	Host git.oschina.net
	User git
	IdentityFile ~/.ssh/oschina_rsa
	IdentitiesOnly yes

这样我们就可以使用不同的ssh key去访问不同的git仓库。

把配置文件写好后，我还是建议做一下测试(上面有提到)。测试成功后，我们的使用才会比较顺利。



#### Reference

* [多个github帐号切换](http://stormzhang.github.io/other/2013/10/16/github-multiply-ssh-key/)
* [Generating SSH Keys](https://help.github.com/articles/generating-ssh-keys)
* [Using Multiple SSH Public Keys](http://superuser.com/questions/272465/using-multiple-ssh-public-keys)

