---
title: migrate-to-hexo
date: 2023-01-07 22:55:27
categories: "2021-10"
tags: [blog,hexo]
---

把博客的引擎从[Jekyll](https://jekyllrb.com/) 迁移到了[Hexo](https://hexo.io/)，过程还是比较简单，感觉比以往在Jekyll 上换个主题还方便。

按照官方的[Migration](https://hexo.io/docs/migration)文档，正常来说是非常简单的。

> Move all files in the Jekyll _posts folder to the source/_posts folder.Modify the new_post_name setting in _config.yml:


首先直接把以前的post 挪过去，然后就`hexo generate`,总是报Front-matter 的`categories.name` 找不到，发现原来用的是 category。这个好办，直接用`sed '1,7s/category/categories/g' 就统一改过来。

接着就本地部署了一下看看是否正常。执行 `hexo server` 打开[localhost:4000](http://localhost:4000/) 发现页面还是没有渲染好，title 展示不出来，Front-matter 原样展示出来了。

    --- 
	title: "Unit Test Result Macro Reference"
	description: "some macro for ios unit- test"
	categories: "2013-07"
	tags: [ios]
	---

一开始怎么看也没有看出什么问题，也对比了一下官方的文档，根本看不出问题，直接用官方的例子拷过来又能渲染出来，最后通过二分法，把这几行一个一个排除，最后发现第一行的标识`--- `后面多了一个空格。

另外就是主题，这次用的是[hexo-theme-vexo](https://github.com/yanm1ng/hexo-theme-vexo)。跟着文档就可以轻松引进来，而且也没什么问题。主要就是在`_config.yml` 中把自己的信息改过来即可。这里没有用git submodule 的方式，而是直接把主题的文件拷到对应的目录下。

部署就是完全按照官方的文档[github-pages](https://hexo.io/docs/github-pages) 操作，最后用的是gp-pages 这个分支去部署。

主题折腾的次数，比自己写文的次数还多。	