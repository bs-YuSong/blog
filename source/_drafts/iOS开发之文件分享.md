---
title: iOS开发之文件分享
tags: 
	- 文件
	- UTI
	- UTL
categories: iOS 
---

通过Document Types添加可接收文件

![如何设置Document Types](img1.png)

设置好Document Types后，只需要在一下方法中实现相关处理即可:
````objc
- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options {
    NSString *content = [[NSString alloc] initWithContentsOfFile:url.path encoding:NSUTF8StringEncoding error:nil];
    NSLog(@"url: %@, option: %@, content: %@", url, options, content);
    return YES;
}
````

参考：
[UTL查询表](https://developer.apple.com/library/archive/documentation/Miscellaneous/Reference/UTIRef/Articles/System-DeclaredUniformTypeIdentifiers.html)
[详解苹果提供的UTI(统一类型标识符)](https://www.jianshu.com/p/d6fe1e7af9b6)
[UIDocumentInteractionController之程序间文档共享](http://superdanny.link/2015/12/26/iOS-UIDocumentInteractionController/)
[UIDocumentInteractionController Does Not Show Mail Option](https://stackoverflow.com/questions/23336363/uidocumentinteractioncontroller-does-not-show-mail-option)
[UIDocumentInteractionController](https://developer.apple.com/documentation/uikit/uidocumentinteractioncontroller/1616811-presentoptionsmenufrombarbuttoni)