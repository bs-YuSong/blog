---
title: Hexo博客学习笔记
tags: hexo
categories: 笔记
date: 2019-01-09 11:32:05
---


记录一下Hexo中常用的功能。

<!-- more -->

1 草稿功能
在Hexo中文章可以分文post和draft两种类型，分别存放在source目录下的_drafts和_posts文件夹中。post代表正式发布的文章，draft代表草稿。
如果想要创建draft,只需要在创建文章时指定其layout为draft即可
	
	hexo new draft <title>

如果想要将draft移动到post，只需要执行public命令修改文章layout即可：
	
	hexo publish [layout] <title>

draft布局的文章默认是不会展示的，如果想要预览，需要使用一下命令：
	
	hexo server --draft

如果不想每次都如此，可以修改_config.yml中的参数：render_drafts: ture

