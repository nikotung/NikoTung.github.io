---
title: "Git command a day - Git 迁移保留提交记录和分支"
description: "checkout command"
categories: "2014-02"
tags: [git,migration]
---

最近的项目需要进行一个迁移，然后希望迁移的时候保留所有的提交记录和分支，迁移后新的项目还要进行目录结构的调整。

一开始以为很简单的只是需要把remote origin的URL重新指向新的项目就可以了，但是发现比想象中麻烦多了。

#### 初试
一开始我真的就这么做了，把remote origin的URL修改后直接push上去，以为就可以了。因为这样把所有的commit记录和分支都保留了。接下来就要改目录结构，问题来了。因为原来的项目中有太多的东西是不需要了，可以删掉。由于这时的每一个操作的是在每一个分支上操作的，如果在一个分支上删除了，其它分支跟着这么做到时merge时就会出现各种问题，最痛苦莫过于项目文件有冲突。后来再想想，这样做不好吧，再找找其它办法。


如果能够把我要的文件全部通过git 命令取出来，其它不需要的直接删掉或者去掉git跟踪，这样就最好了。

#### 把需要的文件从git中抽出来

第一件事要做的当然是备份啦，不备份后果自负哈。

由于我很多feature的分支是在本地的，我又不想删掉它们。所以我的办法有点笨，我在本地新建一个文件夹A，把项目拷贝到那边去，因为我还是在试验阶段，当然不会随便动原来的项目。拷贝完后通过命令行进入A下的项目。

	git remote rm origin //其实是为了保险起见，不会污染远程仓库
	git filter-branch --subdirectory-filter <directory > -- --all   //最重要就是这句了


输出结果大概如下：

	Rewrite 3b61e04975f6d4875137434028b13edf981458a1 (125/125)
	Ref 'refs/heads/xx1' was rewritten
	Ref 'refs/heads/xx2' was rewritten
	Ref 'refs/heads/xx3' was rewritten
	Ref 'refs/heads/feature/xx4' was rewritten
	Ref 'refs/heads/feedback/xx5 was rewritten
	Ref 'refs/heads/master' was rewritten
	Ref 'refs/heads/xx6' was rewritten
	Ref 'refs/heads/xx7' was rewritten
	Ref 'refs/heads/xx8' was rewritten
	Ref 'refs/heads/xx9' was rewritten
	Ref 'refs/heads/xx10' was rewritten
	Ref 'refs/heads/xx.0.1' was rewritten
	Ref 'refs/tags/xx11' was rewritten
	Ref 'refs/tags/xx12' was rewritten
	Ref 'refs/tags/xx13' was rewritten
	Ref 'refs/tags/RELEASE-0.0.1' was rewritten
	Ref 'refs/tags/RELEASE-0.0.2' was rewritten
	

第二句命令通过filter-branch 把git中的tree进行处理--subdirectory-filter是一个重写条件指定是根据目录(directory)去重新 --all就是应用于所有的分支。通过这个命令就可以把不在< directoy >目录下的文件全部删掉并把该目录下的的文件全部移到了git的根目录下。这时候如果你发现了有一些目录不在 < directory >下但是却还没有被删去，不用担心，在终端跑一下`git status` 你会发现那些文件还没有被git track进来，所以直接把它们删掉也没关系了。


#### 重新设置remote url

由于我在上面把origin删掉了，所以这里就是直接的 `git remote add origin <your new repo>`

如果你没有删掉origin的话，这时可以直接 `git remote set-url origin <your new repo>`


到最后按照正常的git flow来操作就好了。



### 参考

[Git Reference](http://gitref.org/remotes/)

[Moving Files From One Git Repository to Another,Preserving History](http://gbayer.com/development/moving-files-from-one-git-repository-to-another-preserving-history/)




