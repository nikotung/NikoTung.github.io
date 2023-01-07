---
title: "Git command a day - checkout remote branch"
description: "checkout command"
categories: "2013-09"
tags: [git,checkout]
---

最近开始要频繁使用git了，这些文章在于纪录使用的一些命令，以及遇到的一些问题。

###checkout 远程分支###

每当我门需要参与一个项目，或者把拿一些开源的项目来用时，第一步总是需要把它们拿到本地，所以我们首先都会clone下来。

当你把一个项目clone下来后，使用git branch，这时你会发现只有一个master的分支，当然这是没有问题的。但是如果你是该团队的一员的话，里面也许会有一些你门正在开发的feature分支，这时你就会奇怪为啥我的分支会不见了。这时请使用如下命令：

	git branch -a

该命令会把项目在远程存在的所有分支都列出来，有remotes/origin前缀的都是远程中的分支。找到你需要的分支，使用如下命令：

	git checkout xxxx  # xxxx就是你想要的分支的名字

该命令就会把xxxx分支checkout下来，并自动切换到该分支。