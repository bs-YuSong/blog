---
title: iOS根据图片检测点击事件
tags:
  - iOS
  - CGContext
categories: iOS
date: 2018-04-27 13:42:32
---

前言
===
由于项目需要，要实现一个图片点击事件，但是检测局域是个不规则的图片。在网上找了一下实现方法，发现资料都是讲自己绘制不规则图形，然后通过Path来检测Touch Point的是不是在Path内，跟我的需求并不相同，所以就自己想了一个办法。
<!-- more -->
问题：
![地图](map.png)
如图，这是一个UIImageView，里边显示的是一个中国地图。当UIImageView获取touch的时候，它的检测区域是整个正方形，而我们的需求则是希望touch范围在地图内则响应，正方内而地图外的部分则不进行响应。该如何做到呢？

解决方案：
我的思路是通过判断touch point的坐标点是否为透明来判断点击是否在地图内，如果透明，则拦截响应，如果不透明，则响应。

整体思路很简单，接下来是实现方法。

第一步，获取touch point的位置：
```OC
	- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
	    /* 通过本方法可以获取touch point */
	    CGPoint touchPoint = [[touchs anyObject] locationInView:self]; 
	}
```

第二步，检测touch point的位置在图片中是否为透明
```OC
	- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
	    /* 通过本方法可以获取touch point */
	    CGPoint touchPoint = [[touchs anyObject] locationInView:self];

	    / * 检测touch point所在位置是否为透明 */
	    unsigned char pixel[1] = {0};
	    CGContextRef ctx = CGBitmapContextCreate(pixel, 1, 1, 8, 1, NULL, kCGImageAlphaOnly);
	    CGContextSetBlendMode(ctx, kCGBlendModeCopy);
	    UIGraphicsPushContext(ctx);
	    [self.image drawAtPoint:CGPointMake(-touchPoint.x, -touchPoint.y)];
	    UIGraphicsPopContext();
	    CGContextRelease(ctx);
	    CGFloat alpha = pixel[0] / 255.0;
	}
```
到了这一步，通过判断alhpa的值是否小于 0.01既可以判断touch point所在位置是否透明啦。不过这种方法在实际工作中还会遇到各种意外情况，以至于影响判断的准确性。
例如：图片在显示的时候并没有根据自己的size显示，而是被设置了各种contentMode，这会影响我们检测的准确性。
所以当遇到这种情况的时候，我们需要调整整体思路。

所以我在解决方法中追加了一个步骤，那就是将UIImageView实际显示出来的效果渲染成图片，然后在这张新生成的图片上检测touch point所在位置的透明度。

所以最终成品代码如下：
````OC
	/* UIView的一个分类方法 */
	- (UIImage *)createImage {
        UIGraphicsBeginImageContextWithOptions(self.bounds.size, false, 1.0);
        CGContextRef ctx = UIGraphicsGetCurrentContext();
        /* 设置颜色代码很重要，如果缺少的话，新生成的图片默认底色为黑色 */
        [[UIColor clearColor] setFill];
        CGContextFillRect(ctx, self.bounds);
        
        [self.layer renderInContext:ctx];
        UIImage * img = UIGraphicsGetImageFromCurrentImageContext();
        UIGraphicsEndImageContext();
        return img;
	}
````

```OC
	- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
	    /* 通过本方法可以获取touch point */
	    CGPoint touchPoint = [[touchs anyObject] locationInView:self];

	    /* 将self的layer渲染成新的image */
	    UIImage *image = [self createImage];

	    /* 检测touch point所在位置是否为透明 */
	    unsigned char pixel[1] = {0};
	    CGContextRef ctx = CGBitmapContextCreate(pixel, 1, 1, 8, 1, NULL, kCGImageAlphaOnly);
	    CGContextSetBlendMode(ctx, kCGBlendModeCopy);
	    UIGraphicsPushContext(ctx);
	    [image drawAtPoint:CGPointMake(-touchPoint.x, -touchPoint.y)];
	    UIGraphicsPopContext();
	    CGContextRelease(ctx);
	    CGFloat alpha = pixel[0] / 255.0;
	}
	```

效果：
![效果](animation.gif)