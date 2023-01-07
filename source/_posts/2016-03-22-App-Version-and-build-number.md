---
title: "App 版本号和build的一些记录"
description: "记录一下版本号和build 的一些问题"
categories: "2016-03"
tags: [version , build]
---

App 的发布难免会跟版本号打交道，怎样才能比较好的处理版本号和build number 之间的关系呢？

#### 命名的样式(scheme)

像`4.1.2` 或者`4.0`，这种格式的版本号是很常见的,`major.minor[.build[.revision]]`,这就是我们常说的版本号。我们用的都是三段式的版本号，第一段会轻易更新，只有当 App 有重大功能发布或者重大改版后才会去修改。第二段是我们的新功能的版本号，一般发布一些小的新功能时候会去改这个。第三段是一个bugfix 版本，发布一个修复bug的版本时会去改动。

#### build number

这个的定义可能就有各种各样的形式，主要时表明每一次编译发布后都会不一样。这个对于用户来说可能没有什么作用，对于内部的开发和bug的追踪就有比较大的作用。

我们希望通过版本号加build number可以快速的定位出现问题的代码。而版本号这个一般变动不会太大，所以在 build number 上下手。

现在来说，build number 的变动主要有下面三种做法

* 每编译一次就加一 (build number 是一个整数)
* 按编译的时间，如201603212337
* 代码管理的版本号，如 Git 的head


一开始我们是用的第一种做法，每一次编译发布就自动加一。加上Apple 本身就有一个agvtool 工具可以轻松的在Xcode 集成，加上一个脚本就基本上可以在编译时自动执行了，需要在 Xcode中配置 `Apple Generic` naming system。

后面发现有两个不好的问题。

* 同一个份代码，但是两次编译会有两个不同的 build 其实是不利于定位问题的
* 在开发阶段需要经常给测试，build number 会变化的很快，而这只是内部测开发测试，没有必要去改变build

后来再加上了基于代码管理的版本号，如`44fa541806927460d3726d9f6b6f982712292de2` 取前面七位。在内部测试时就用`44fa541`基于代码管理系统的版本号，而info.plist 文件中的build number 还是用数字 ，在自动打包发布测试版本时，会自动根据代码的提交记录生成 build number，发布完成后就reset 掉。 在提交 App Store 时就自动加1并且自动提交到 Git 中去。

用了[fastlane](https://github.com/fastlane/fastlane) 这个工具，这些build number的变更就很容易做到了。

#### Reference

[AGVTOOL](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man8/agvtool.8.html)

[Software versioning](https://en.wikipedia.org/wiki/Software_versioning)