---
layout: post
title: "Mac下简单ftp使用"
description: ""
category: “2013-10-12”
tags: [Mac,工具杂谈]
---

今天研究了一下iOS的in-house发布方式，发现还是比较简单的。在研究过程中要用到ftp上传文件进行测试，于是发现在mac中已经预置了ftp工具，通过Finder可以简单的连接到ftp服务器。通过命令行可以进行一些简单的操作，如下载，上传等。把这些东西记录一下。

通过Finder连接ftp服务器
--------------------------
我们在Dock中右键一下Finder就可以看到一个 #Connect to Server# 的选项，点击它就在弹出的框中输入ftp地址。这时你就通过Finder中看到server上面的东西

![](/assets/2013-11-13-ftp.png)

简单的ftp命令（在mac下）
-------------------------------

通过命令行登入到ftp后，很类似于在一个unix系统下操作，如删除文件，创建文件/文件夹等。

1. 连接ftp: `ftp` 通过ftp命令加上服务器地址.如 ftp 192.193.0.1 。如果需要用户名和密码，则会提示你输入，按照提示填就可以了，如无意外就可以成功登入。

2. 下载文件: `get` `get remote_file(file path) local_file(file_path)` get 命令就可以把服务器的文件下载到本地，通过设置local_file可以为下载的文件在本地重命名。

3. 上传文件: `put` `put local_file(file_path) remote_file(file_path) ` 通过remote_file同样可以为文件重命名

4.批量上传/下载: mput mget这两个命令可以进行批量上传和下载文件
