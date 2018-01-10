---
title: Xcode调试技巧总结
tags: 
	- Debug
	- Xcode
categories: iOS
---
在iOS开发当中总会遇到各种各样的奇葩BUG，很多时候Debug是一个难题。很多时候由于种种原因，单一的Debug手段会失去作用，无法达到我们想要的预期，所以特此写下这篇文章，将我知道的Debug技巧罗列出来，方便大家学习。

<!-- more -->

1. [断点Debug](#breakpoint)
2. [Log调试](#log)
3. [层级透视](#layer)
4. [LLDB调试](#command)


<h5 id="breakpoint">1. 断点调试</h5>
1. 自定义断点
2. 为某一种方法添加断点
3. 全局断点

<h5 id="log">2. Log调试</h5>
1. 使用System Logs打印Log

<h5 id="layer">3. 层级透视</h5>
1. 层级透视观察控件内部实现

<h5 id="command">4. LLDB调试</h5>
1. po（print object）的使用技巧
2. p
3. call
4. expr
5. bt
6. image


参考：[iOS开发之Xcode常用调试（Debug）技巧总结](http://www.yangshebing.com/2016/10/27/iOS%E5%BC%80%E5%8F%91%E4%B9%8BXcode%E5%B8%B8%E7%94%A8%E8%B0%83%E8%AF%95%EF%BC%88Debug%EF%BC%89%E6%8A%80%E5%B7%A7%E6%80%BB%E7%BB%93/)
