---
title: UITableView如何实现侧滑删除显示图片
tags: iOS
categories: UITableView
date: 2019-09-10 16:40:09
---


<h3>iOS11之后的实现</h3>

在iOS11之后，系统本身就支持了为Action自定义Image的行为，所以实现非常简单。只需要实现UITableViewDelegate中的一个方法：
````objc
- (UISwipeActionsConfiguration *)tableView:(UITableView *)tableView trailingSwipeActionsConfigurationForRowAtIndexPath:(NSIndexPath *)indexPath {
    UIContextualAction *action = [UIContextualAction contextualActionWithStyle:UIContextualActionStyleNormal title:@"" handler:^(UIContextualAction * _Nonnull action, __kindof UIView * _Nonnull sourceView, void (^ _Nonnull completionHandler)(BOOL)) {
    }];
    action.image = [UIImage imageNamed:@"icon_delete"];
    action.backgroundColor = [UIColor colorWithHex:0xff5555];
    return [UISwipeActionsConfiguration configurationWithActions:@[action]];
}
````

<h3>iOS11之前的实现</h3>
在iOS11之前，系统并不支持开发者自动以Aciton的Image，所以想要实现，需要想写别的法子。

首先我们看一下删除按钮在UITableView中的层级关系。
![层级关系](img1.png)

可以看出，UITableCellActionButton是在UITableViewCellDeleteConfirmationView中，而这个UITableCellActionButton就是我们想要设置图片的Button。

所以我们尝试通过UIButton的静态方法：appearanceWhenContainedInInstancesOfClasses来取得这个Button，并设置图片来达到目的。

代码如下：
````objc
- (NSArray<UITableViewRowAction *> *)tableView:(UITableView *)tableView editActionsForRowAtIndexPath:(NSIndexPath *)indexPath {
    UITableViewRowAction *action = [UITableViewRowAction rowActionWithStyle:UITableViewRowActionStyleNormal title:@"       " handler:^(UITableViewRowAction * _Nonnull action, NSIndexPath * _Nonnull indexPath) {
    	// TO DO:
    }];
//    action.backgroundColor = [UIColor colorWithPatternImage:[UIImage imageNamed:@"icon_delete_new"]];
    action.backgroundColor = [UIColor colorWithHex:0xff5555];
    if (LESS_IOS11) {
        Class confirationView = NSClassFromString(@"UITableViewCellDeleteConfirmationView");
        UIButton *deleteBtn = [UIButton appearanceWhenContainedInInstancesOfClasses:@[confirationView.self]];
        [deleteBtn setBackgroundImage:[UIImage imageNamed:@"icon_delete_new"] forState:UIControlStateNormal];
        
    }
    return @[action];
}

````

因为侧滑删除的按钮的层级关系，各个版本的系统中差别比较大，所以具体实现的细节，可能需要根据实际情况进行调整，不过整体思路是不变的。

观察上边的代码，会发现有一句代码被注释掉了，实际上这也是一种实现方法，在某些版本的某些情况下也可以达到设置图片的目的，不过具体细节在此就不展开啦。




相关链接：[UITableViewRowAction title as image icon instead of text](https://stackoverflow.com/questions/27740884/uitableviewrowaction-title-as-image-icon-instead-of-text)