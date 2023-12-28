---
title: iOS屏幕旋转总结
date: 2019-01-09 11:33:36
tags: iOS
categories: iOS
---

屏幕旋转这个功能点，因为受到的影响条件比较多，所以在我看来，是个比较容易搞混的知识点，借此机会，总结一下，希望对大家理解iOS系统的屏幕旋转有所帮助。

<!-- more -->

<h4>1. 设备的物理方向</h4>
![设备物理方向](device_orientation.png)

在iOS系统中，可以通过代码获取设备当前的屏幕状态：

	UIDevice.current.orientation    //Swift
	[UIDevice currentDevice].orientation    //Object-C

相较于界面显示方向，设备物理方向多了FaceUp和FaceDown两个状态，分别是，屏幕朝上跟屏幕朝下，这个是界面显示方向所不具备的。

关于物理设备方向有一点需要注意，那就是当屏幕方向被强制修改的时候，有可能会出现unknow状态，这种情况下，需要设备状态再次更改之后，设备才能获取到正确的状态。


<h4>2. 界面的显示方向</h4>
<center>![屏幕方向](interface_orientation.png)
屏幕方向</center>
</br>
<center>![屏幕支持方向](interface_orientation_mask.png)
屏幕支持方向</center>

屏幕支持方向可以自由组合，是OptionSet类型。

<h4>3. 有哪些点会影响到屏幕的旋转</h4>
<center>![Xcode中设置屏幕支持方向](Xcode_orientation.png)
Xcode中设置屏幕支持方向</center>

在Xcode中设置完屏幕支持方向之后，如果在AppDelegate中，实现如下方法，则可以覆盖掉Xcode中设置的屏幕支持方向，在某些情况下会很有用
	
	func application(_ application: UIApplication, supportedInterfaceOrientationsFor window: UIWindow?) -> UIInterfaceOrientationMask {
        return .all
    }

除了在Xcode和AppDelegate之中设置屏幕支持方向之外，我们最常用的其实是在ViewController中来控制屏幕是否支持旋转，屏幕支持旋转方向和默认屏幕方向，要实现屏幕旋转的精准控制，就需要实现以下方法：

	override var shouldAutorotate: Bool {    // 是否支持屏幕自动旋转
        var result = true
        return result
    }

    override var supportedInterfaceOrientations: UIInterfaceOrientationMask {    // 屏幕支持旋转的方向
        return IS_IPAD ? .all : .allButUpsideDown
    }
    
    override var preferredInterfaceOrientationForPresentation: UIInterfaceOrientation {    // 屏幕当前默认方向
        return .portrait
    }

<strong>这里有两点需要注意，屏幕实际上可以支持的方向，是AppDelegate中支持的和supportedInterfaceOrientations方法中支持的并集。
还有，设备方向被改变的时候，shouldAutorotate都会被调用，这为我们的灵活操作提供了很大的空间啊！</strong>

理想来说，通过这3个方法，就可以很灵活的来控制屏幕旋转相关的操作。

但是,现实很骨感，这3个方法其实还受很多东西的影响。

<strong>这3个方法，只有当Controller是window的第一个Controller时才会被调用，也就是只有window的rootViewController才会调用，所以如果想要自由操作，就需要重新rootViewController的这三个方法，将当前viewController的这三个方法返回，达到灵活控制的目的。常见的就是在UITabBarController或者UINavigationController中实现这三个方法。

或者，使用Modal，在Present出来的ViewController中来实现这三个方法，也可以自由控制屏幕旋转。</strong>

一般来说，到目前为止的知识点就足够啦，不过实际上，由于iOS系统的版本不同，还会出现各种问题。

<h4>4. 各种设备上屏幕旋转的微妙不同</h4>
这里我就说一下，我遇到的一些问题，并讲一下如何解决。

从iOS9开始，在Xcode中如果没有勾选Requires full screen选项，那么，在iPad上，即使实现了shouldAutorotate方法，也不会被正常调用，因为在这种情况下，iPad默认是支持所有的屏幕方向的。

<strong>那么如何解决呢？答案前边已经写了，就是勾选上Requires full screen选项。</strong>
<center>![rquires](rquires.png)
勾选requires full screen</center>

但是要注意，一旦勾选了requires full screen选项，就说明这个App将不再具备分屏功能，必须是全屏App啦。

<h4>5. 屏幕旋转的流程</h4>
屏幕旋转的流程如下：
1>、加速计来识别设备的旋转方向。
发送 UIDeviceOrientationDidChangeNotification 设备旋转的通知。
2>、app 接收到旋转事件（通知事件）。
2>、app 通过AppDelegate通知当前程序的KeyWindow。
3>、Window 会知会它的 rootViewController，判断该view controller所支持的旋转方向，完成旋转。
4>、如果存在 modal 的view controller的话，系统则会根据 modal 的view controller，来判断是否要进行旋转。

<h4>6. 如何让屏幕不旋转，而屏幕中的控件来旋转</h4>
这是最近遇到的项目里的一个功能，参考iOS12系统上，iPhone的相机，当设备旋转的时候，屏幕并没有旋转，而屏幕上的按钮却发生了旋转。
我想到的思路是，关闭屏幕旋转功能，即：shouldAutorotate返回false，然后通过NotificationCenter来监听设备的旋转：
    
    NotificationCenter.default.addObserver(self, selector: #selector(rotationCall(noti:)), name: UIDevice.orientationDidChangeNotification, object: nil)

然后在监听方法中实现，将控件进行旋转就可以啦。

参考：
https://www.jianshu.com/p/e473749f1c30
http://www.cocoachina.com/ios/20170711/19808.html
http://foggry.com/blog/2014/08/08/ping-mu-xuan-zhuan-xue-xi-bi-ji/
https://www.jianshu.com/p/62431e148e68
https://www.jianshu.com/p/62431e148e68
