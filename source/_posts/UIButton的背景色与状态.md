---
title: UIButton的背景色与状态
date: 2016-07-18 17:30:49
tags: iOS
categories: iOS
---
众所周知，前言很难写，这篇文章的前言我是编不出来啦。

<!-- more -->

在使用UIButton是，由于UIButton具有多种状态，如下：
````objc
typedef NS_OPTIONS(NSUInteger, UIControlState) {
    UIControlStateNormal       = 0,
    UIControlStateHighlighted  = 1 << 0,                  // used when UIControl isHighlighted is set
    UIControlStateDisabled     = 1 << 1,
    UIControlStateSelected     = 1 << 2,                  // flag usable by app (see below)
    UIControlStateFocused NS_ENUM_AVAILABLE_IOS(9_0) = 1 << 3, // Applicable only when the screen supports focus
    UIControlStateApplication  = 0x00FF0000,              // additional flags available for application use
    UIControlStateReserved     = 0xFF000000               // flags reserved for internal framework use
};
````
但是UIButton虽然提供了各种状态下设置图片和设置文字的接口，却没有提供各种状态下设置背景颜色的接口。
所以当各种状态下需要不同的backgroundColor时就会比较麻烦，所以我们需要自定义一个实现这种需求的接口，实现方法很简单，如下：
````objc
UIGraphicsBeginImageContext(CGSizeMake(1, 1));
UIBezierPath *path = [UIBezierPath bezierPathWithRect:CGRectMake(0, 0, 1, 1)];
[backgroundColor set];
[path fill];
UIImage *backgroundImage = UIGraphicsGetImageFromCurrentImageContext();
self.backgroundColor = [UIColor whiteColor];
[self setBackgroundImage:backgroundImage forState:state];
````
这个方法可以装在一个category中来使用，也可以通过继承UIButton来实现一个自己的Button方法。
这样，当遇到UIButton不同状态时需要不同的背景色处理起来就会很方便啦。

方法介绍到这，下面是一个关于神坑的故事。。。。
今天写项目的时候，突然发现UIButton的背景色莫名其妙的变成了半透明状，让我十分诧异，进行了各种检查，却还是没有发现问题所在。直到我再次跳入如上这段代码中，我才发现的一个问题。我的项目里的代码是这样的：
````objc
UIGraphicsBeginImageContext(CGSizeMake(1, 1));
UIBezierPath *path = [UIBezierPath bezierPathWithOvalInRect:CGRectMake(0, 0, 1, 1)];
[backgroundColor set];
[path fill];
UIImage *backgroundImage = UIGraphicsGetImageFromCurrentImageContext();
self.backgroundColor = [UIColor whiteColor];
[self setBackgroundImage:backgroundImage forState:state];
````
那么问题来啦。。。。你们发现不同了嘛？
没错，就是我在制作背景图时，把正方形画成了圆形。。。。这就™很尴尬啦。。。。由于我的画布是1*1的正方形，而画出来的是一个圆形，所以这个背景图会有部分是clearColor，然后当背景图被拉伸时，就莫名弃疗的拉伸出了一个半透明的效果。。。。天哪。。。真神奇。。。。
而就是这个神奇的效果，让我一直没有想到BUG产生的原因居然在这里。。。还以为是哪里的alpha出了问题。。。。
好啦，虽然我觉得这样的BUG大概除了我都不会再有人出现了，但是，我觉得这种情况似乎也蛮有趣，说不定以后会在哪里用到，所以记录一下。