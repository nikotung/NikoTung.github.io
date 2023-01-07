---
title: "git 删除远程master分支"
description: "怎样删除远程仓库中的master分支"
categories: "2015-01"
tags: [git,master]
---

分支在git中有多么的重要，我就不说了。用到了分支就一定会有创建和删除的操作了。今天因为想把blog中的主题换一下，折腾了一下。在删除远程仓库中的master分支的时候遇到了点问题，顺便记录一下。

在git中，如果你的分支已经push出去了，但是你想要改名字的话你需要这样做：
* 在你需要删除的分支中checkout一个新的分支出来，并且命名为你想要的分支名字；
* 把你想要改名字的远程分支删掉；
* 把刚刚checkout出来的分支推送到远程分支

这样一个远程分支改名字的工作就完成了。

原理上说按照这个流程，我们的删除，重命名以及创建远程分支都不会有问题的。但是我通过这样的方式更改远程仓库的master分支的时候却出现了错误：

	$ git push origin :master
	remote: error: refusing to delete the current branch: refs/heads/master
	To git@github.com:NikoTung/NikoTung.github.io.git
	remote rejected master (deletion of the current branch prohibited)
	error: failed to push some refs to 'git@github.com:NikoTung/NikoTung.github.io.git'

远程仓库拒绝了删除master分支的push。

这是为什么呢？

因为git项目初始化的时候默认就有一个master分支，当你把项目推送到远程仓库的时候，github会把master分支设为默认的display分支。什么意思呢？就是用户打开你这个项目的时候马上展示给用户看的意思，不然它不知道用哪个分支给用户看。

所以如果你想删除一个设为默认展示分支的时候，远程仓库是不允许的。这是需要在项目的设置中更换默认展示分支，然后才能把原来默认的展示分支删除掉。

知道了这个就知道了怎么删除远程分支了。

就这么多了。