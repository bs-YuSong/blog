---
title: iOS地图和CoreLocation学习笔记
tags: 
	- iOS
	- CoreLocation
	- Map
categories: iOS
---

前言
===

要做一个地图应用，第一次做，做个笔记

<!-- more -->

1. 定位通过CoreLocation框架来获取当前的经纬度

````objective-c
	if ([CLLocationManager locationServicesEnabled]) {
        self.locationManager.delegate = self; /* 为LocationManager设置代理 */
        self.locationManager.desiredAccuracy = kCLLocationAccuracyBest;	/* 选择Location获取经度 */
        self.locationManager.distanceFilter = 1.0f;	/* 可以设置多少米更新一次Location */
        [self.locationManager requestWhenInUseAuthorization]; /* Location获取的模式，App开启时还是一直可以获取 */
        [self.locationManager startUpdatingLocation];	/* 开始获取Location */
	}
````

 根据两点绘制线路：http://swift.gg/2017/05/02/draw-route-mapkit-tutorial/
 根据一组坐标绘制路线：https://www.jianshu.com/p/fd47eab5e098
 MapKit介绍：http://www.cnblogs.com/chglog/p/4836715.html
 iOS原生渐变轨迹：https://www.jianshu.com/p/2d71cc8dd035
 MKCoordinateSpan：https://cnbin.github.io/blog/2015/11/18/mkcoordinatespan/
 CoreLocation定位：http://www.hangge.com/blog/cache/detail_783.html
 iOS模拟器模拟Location信息：http://foggry.com/blog/2014/05/25/iosmo-ni-qi-custom-locationbei-zhong-zhi-jie-jue-fang-an/


 https://www.jianshu.com/p/ddbb722066f6
 https://www.jianshu.com/p/964b19ef0225
 http://www.hangge.com/blog/cache/detail_783.html
 http://www.cnblogs.com/congli0220/p/5078187.html

iOS经纬度偏移： https://my.oschina.net/gamecubategc1/blog/148592
如何判断当前GPS信号好坏： https://stackoverflow.com/questions/10246662/how-can-i-evaluate-the-gps-signal-strength-iphone

