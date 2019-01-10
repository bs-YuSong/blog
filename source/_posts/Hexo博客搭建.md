---
title: Hexo博客搭建
date: 2017-12-22 14:03:18
tags: hexo
categories: 杂项
---

从2015年起开始博客写作，到目前为止写下的相关文章应该也有50篇左右，从最开始的CSDN到后来的简书，终于走上了自己搭建博客的道路。所以按照惯例，写下这篇文章，记录自己搭建博客的经过。

<!-- more -->

博客搭建使用了Hexo，基本没有遇到什么难题，总共大概用了一个上午的时间，虽有小坑，但都不算难。

闲话少说，进入正题

前期准备
===

1 Git 安装
	作为一个程序员，这个我就不多说啦

2 Node 安装
	1> 想要安装Node，最好先安装Node的版本管理工具nvm

	wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.1/install.sh | bash

  2> 安装好后新建终端即可使用nvm来安装node

		nvm install stable // 安装最新的稳定版Node

3 hexo 安装
		npm install -g hexo-cli // 全局安装

至此，前期准备全部完成，接下来就是如何使用hexo搭建自己的博客啦

使用Hexo搭建博客
===

正式搭建开始之前先说点废话，虽说是使用Hexo搭建博客，其实是使用了Hexo + Github Pages搭建博客，所以和Github也是息息相关的。
	1 在Github上创建一个代码库，我使用的是blog

  2 使用Hexo创建blog项目  

		hexo init blog; cd blog	//创建一个blog项目文件夹，并跳转进入
		npm install hexo-deployer-git --save	//通过npm添加hexo部署工具

  3 修改blog根目录下的_config.yml文件进行配置

		deploy:
		type: git
		repo: git代码库的地址
		branch: gh-pages // blog静态文件的部署分支

  4 由于我们是要将静态文件部署到gh-pages分支上，所以要特别注意的是，修改_config.yml文件中url的root参数

		url: http://yoursite.com/
		root: /blog

  5 一切准备就绪之后，在命令行输入一下指令

		hexo generate // 重新生成静态文件(很重要，不然新配置的参数不会生效)
		hexo deploy   // 将静态文件部署到指定分支

至此，博客搭建工程基本完成，可以去username.github.io/blog 查看你的博客啦。

PS: 有些博主在通过git pages部署博客的时候，希望将网址直接设为：<username>.github.io，想要达到这种效果的话，就不能使用gh-pages分支，而应该在master分支上部署博客。如果你的项目名就叫做：<username>.github.io，那么在默认情况下，github就会认为你的git pages的分支就是master分支，这时如果你去repo -> setting里查看git pages的分支设置，就会发现，branch锁死的就是master分支，而无法更改。这种情况如要注意。

我就是因为对这一点不了解，在改变网址的时候，明明使用的屎gh-pages的分支，结果想要将域名设置为<username>.github.io，一直出现404错误，于是花了很多时间，才发现这个问题。

<strong>总结一句话，如果你的项目给名叫做<username>.github.io，那么你的git pages就默认实在master分支上，域名也会是<username>.github.io，这种情况，是不会使用gh-pages分支的。如果你把博客文件部署到了gh-pages分支上了，就会出现404错误。因为git pages无法在master分支上找到index.html这个文件！</strong>


参考文章：
https://stackoverflow.com/questions/39978856/unable-to-change-source-branch-in-github-pages
https://stackoverflow.com/questions/39837559/why-isnt-my-page-loading-up-on-github