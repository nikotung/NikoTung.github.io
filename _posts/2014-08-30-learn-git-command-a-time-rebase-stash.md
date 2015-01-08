---
layout: post
title: "Git command a day - rebase and stash"
description: "checkout command"
category: "2014-08"
tags: [stash,rebase]
---


最近公司出了一些规范，就是如何在并行开发时控制代码的合并。

所以开始要对git有更多的了解，在这过程中发现git真的非常非常强大，不愧为神器啊。而我自己连入门级别都敢说了，真的是很惭愧啊。

这里就说两个命令的基本功能。

### Stash 

相信很多人都有过这样的经历，自己正在开发一个新的feature，突然间被别人打断。没错，就是要修 bug了。这时候，代码写了一半，又不想提交，但是不提交就没法切分支。这时候就是stash发挥作用的时候了。

这时候，您只需要在命令行中敲入，如下:
	
	git stash

这时候你可以随意切换分支了。	

这样你的修改就会被暂存下来了，它只是记录了你的修改状态，而不会对整个git tree有任何的影响，而且你还能随时恢复，还不限分支。

既然暂存了，就必须得可以恢复的。在恢复前，我们肯定希望看看之前有哪些暂存，然后再根据自己想要的恢复。

	git stash list

这样就可以把所有的暂存记录列出来。

	stash@{0}: GitHub: stashing before switching to feature/stash_test
	stash@{1}: WIP on feature/backup: 77d577e 

我这里就有了两条暂存记录了。如果想要恢复暂存的话，就选择自己想要的，然后：

	git stash apply stash@{1} #这里恢复了第二条记录，如果不指定的话会默认恢复最近的一条暂存
	git stash drop stash@{1}  #这里把暂存删掉

如果你想在恢复暂存的同时就把暂存删掉的话，可以
	
	git stash pop 

如果我们在恢复前看到有很多暂存，但是发现自己都忘了每一条暂存都修改了什么内容。所以我们希望可以看到每一条暂存的修改详情

	git stash show -u
	git stash show stash@{0} -u 

这样就可以看到暂存的diff 了

上面的就是我最近用到的关于stash的功能了。

### rebase

关于rebase的功能，是在太强大了。如果你熟练掌握了它，是熟练哦，你就可以为所欲为，偷天换日了。

因为有时候需要强制的串行（就是让commit tree好看），我这边经常要用到另个功能： rev-parse,merge-base

	git rev-parse origin/master
	ed8f0f4760530b0b04b701e523ba35cd8b6490c0

	git merge-base origin/master origin/feature/writting
	ed8f0f4760530b0b04b701e523ba35cd8b6490c0


rev-parse ：找出某一个分支的是基于哪个分支的
merge-base: 找出两个分支的最近的公共基点

如果两个id是一致的，这样提merge request时就能保证串行了。


rebase的基本用法

当你在自己的分支上做开发了，也提交了好几个commit，最后完成了。怎被合到master去，这时候发现master已经被别人合过很多遍了，而你需要你自己的提交嫁接在master分支上最新的commit去，这时候：

	git rebase origin/master

 如果没有冲突，就是一个forward－fast的合并了。

 如果有冲突，当然要自己解决啦。解决完后不需要commit了，只需要这样:

 	git rebase --continue

 如果再用冲突，再解决，再

 	git rebase --continue

 直到完全合并完成。

 如果你rebase到一半，不想干了，怎么办？那就不干吧，简单了。

 	git rebase --abort

 这样rebase就终止了，一切回到解放前了。

 最近用的就是这些了。



 ### Reference(翻墙效果更好)

 [Git-rebase 小筆記](http://blog.yorkxin.org/posts/2011/07/29/git-rebase)

 [How to look at the stash](http://makandracards.com/makandra/11565-git-how-to-look-at-the-stash)