---
title: MacOS软件开发日记
tags:
categories:
---

突发奇想的想学习macOS的软件开发，所以单开一个日记系列来记录。

<!-- more -->

1. macOS开发一样可是使用cocosPods，不过要指定platform为osx，同时需要指定最低的版本号如"10.9"，不然cocosPods在导入框架时会按照框架的最低版本来导入，可能因为框架最低版本不支持macOS而失败。

2. macOS的UI布局系统跟iOS差别其实很大，正在学习中。

3. NSOpenPanel是一个文件选择类，可以创建出一个文件选择窗口，支持文件选择、文件夹选择、创建文件夹、多选等功能。

4. 目前发现，AVAudioPlayer对.ape格式的文件好像无法播放，确实不支持.ape文件的播放

5. 想要获取ID3信息，不过mac不支持原始的方法，找到了一个C++框架taglib,但是不会打包framework，很烦。顺藤摸瓜找到了一个叫CMake的工具，可是完全不知道是干嘛的，很烦。基础不牢啊。

6. 