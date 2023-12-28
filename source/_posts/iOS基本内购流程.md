---
title: iOS基本内购流程
date: 2016-07-19 17:55:41
tags: 
	- iOS
	- 内购
categories: iOS
---
这也是一边移植老文，文章中还有不少纰漏，还有一些应该放出的代码也没有放出，一直拖到今天。以后如果有再用到内购的时候，我想我会把这篇文章补全为iOS内购全解。

<!-- more -->

iOS的内购流程如下

1. 通过产品ID获取产品信息列表
2. 添加监听
3. 把产品包装成SKPayment（支付）发送给苹果服务器
4. 苹果服务器购买成功后会回调监听方法，根据苹果服务器返回信息判断是否购买成功。
5. 购买失败或已经购买过该商品则注销交易。如果购买成功，此时可以向自家服务器发送购买成功的消息，并通过后台向苹果服务器发送验证，然后注销交易。

#### 一般而言，这就是iOS内购的基本过程，看似很简单，但是其实实际操作起来，还是比较麻烦的，因为要考虑到各种意外情况。

下面讲一讲iOS内购的具体过程
1、获取产品信息列表
````objc
if ([SKPaymentQueue canMakePayments]) {
    NSSet *IDSet = [NSSet setWithArray:proID];
    SKProductsRequest *productsRequest = [[SKProductsRequest alloc] initWithProductIdentifiers:IDSet];
    productsRequest.delegate = self;
    [productsRequest start];
} else {
    NSLog(@"用户禁止付费");
}
````
上面代码中的proID就是装有你在开发者后台创建内购产品时输入的产品ID的NSArray。
delegate是指SKProductsRequestDelegate
首先判断用户是否禁止付费，如果没有禁止付费，就想苹果服务器请求产品信息。
请求的信息会在SKProductsRequestDelegate的方法中返回
````objc
- (void)productsRequest:(SKProductsRequest *)request didReceiveResponse:(SKProductsResponse *)response
{
    NSLog(@"%i", response.products.count);
    NSArray *myProducts = response.products;
    if (0 == myProducts.count) {
        NSLog(@"无法获取产品信息列表");
    } else {
        self.products = [myProducts sortedArrayUsingComparator:^NSComparisonResult(id  _Nonnull obj1, id  _Nonnull obj2) {
            SKProduct *pro1 = (SKProduct *)obj1;
            SKProduct *pro2 = (SKProduct *)obj2;
            return pro1.price.integerValue < pro2.price.integerValue ? NSOrderedAscending : NSOrderedDescending;
        }];;
        for (SKProduct *pro in myProducts) {
            NSLog(@"%@", [pro localizedTitle]);
            NSLog(@"%@", [pro localizedDescription]);
            NSLog(@"%@", [pro price]);
            NSLog(@"%@", [pro.priceLocale objectForKey:NSLocaleCurrencySymbol]);
            NSLog(@"%@", [pro.priceLocale objectForKey:NSLocaleCurrencyCode]);
            NSLog(@"%@", [pro productIdentifier]);
        }
    }
}
````
拿到产品信息以后可以进行排序处理，因为请求的时候发送的产品ID是装在一个NSSet中的，所以返回的产品信息也是乱序的，这里需要注意一下。

2、内购监听
拿到产品信息以后要设置监听，因为当你点击购买和购买后苹果服务器会通过监听方法通知应用。
````objc
- (void)startObserver {
    if (!self.isObserver) {
        [[SKPaymentQueue defaultQueue] addTransactionObserver:self];
        NSLog(@"开始监听 ------ 内购");
        self.isObserver = YES;
    }
}

- (void)stopObserver {
    if (self.isObserver) {
        [[SKPaymentQueue defaultQueue] removeTransactionObserver:self];
        NSLog(@"移除监听 ------ 内购");
        self.isObserver = NO;
    }
}
````
监听方法和移除监听的方法一起送上，isObserver是一个判断是否已经监听的BOOL数据。
我的建议是将监听方法和移除监听的方法都在AppDelegate中执行。当App启动时（
application didFinishLaunchingWithOptions）开始监听，当App被关闭时（applicationWillTerminate:）移除监听。至于原因，后面会提到。

3、实现监听方法

```objc
- (void)paymentQueue:(SKPaymentQueue *)queue updatedTransactions:(NSArray<SKPaymentTransaction *> *)transactions
{
    NSLog(@"调用了几次这个方法？");
    SKPaymentTransaction *transaction = transactions.lastObject;
    switch (transaction.transactionState) {
        case SKPaymentTransactionStatePurchased: {
            NSLog(@"购买完成,向自己的服务器验证 ---- %@", transaction.payment.applicationUsername);
            NSData *data = [NSData dataWithContentsOfFile:[[[NSBundle mainBundle] appStoreReceiptURL] path]];
            NSString *receipt = [data base64EncodedStringWithOptions:0];
            [self buySuccessWithReceipt:receipt transaction:transaction];
        }
            break;
        case SKPaymentTransactionStateFailed: {
            NSLog(@"交易失败");
            [[SKPaymentQueue defaultQueue] finishTransaction:transaction];
        }
            break;
        case SKPaymentTransactionStateRestored: {
            NSLog(@"已经购买过该商品");
            [[SKPaymentQueue defaultQueue] finishTransaction:transaction];
        }
            break;
        case SKPaymentTransactionStatePurchasing: {
            NSLog(@"商品添加进列表");
        }
            break;
        default: {
            NSLog(@"这是什么情况啊？");
        }
            break;
    }
}
```

finishTransaction:就是注销方法，如果不注销会出现报错和苹果服务器不停的通知监听方法等等情况。总之，记住要注销交易。
有的同学可能会疑惑，transaction.payment.applicationUsername中的这个applicationUsername属性是干嘛的，先不要急，关于这个属性我们会在后面提到，现在先记住这个点就好。

```objc
NSData *data = [NSData dataWithContentsOfFile:[[[NSBundle mainBundle] appStoreReceiptURL] path]]; 
NSString *receipt = [data base64EncodedStringWithOptions:0];
[self buySuccessWithReceipt:receipt transaction:transaction];
```

关于这三句，要注意，receipt是刚才交易的清单，如果后台需要进行二次验证，需要用到这个数据。
至于最后一句，则是购买成功后向自家的服务器发送的请求。

#### 发送内购请求

完成上面的代码后，就可以进行发送内购请求的部分啦

```objc
SKMutablePayment *payment = [SKMutablePayment paymentWithProduct:product];
payment.applicationUsername = [AppManager sharedInstance].userId.stringValue;
[[SKPaymentQueue defaultQueue] addPayment:payment];
```

内购请求很简单，就是用请求道的产品信息SKProduct创建一个“内购支付”SKPayment，然后添加进支付队列。
#### 以上就是iOS内购的全部核心代码。接下来要讲的，就是一些内购中可能出现的坑和如何跳过这些坑。
细心的同学应该已经发现，在发送内购请求的代码部分，我们又一次见到了applicationUsername这个属性，并将用户的id赋值给了它，那么，这是用来干什么的呢？
答案是：在某些极端情况下，可能出现在发送内购请求的用户和内购成功后通知自家后台的用户可能不是同一个用户的情况（真是奇葩的用户。。。。但是没办法，用户就是上帝嘛。。。）这种情况下，为payment绑定一个appUsername就可以让这次payment有一个固定的发起者，这样当这次payment在苹果后台支付成功后，我们就可以通过监听的回调，将这个发起者的唯一标识符上传给自家后台，使得这次购买能找到一个合适的主人。就算用户在购买的过程中切换账号或者退出，也能够让这次充值验证成功。

既然说到了极端情况，那么我们不如更进一步，让情况更极端一点，那就是当购买请求发送后，直到向苹果后台购买成功的这段时间，如果程序崩溃了！或者程序被用户关闭了！怎么办？！
这种情况下，我们的App自然也就无法进行监听的回调，自然也就无法把购买成功的消息发送给自家后台，用户也就拿不到自己的充值啦。情况很糟糕，但是！不用担心，我们有办法解决。
还记得上边提到的注销交易的方法吗？没错就是：
```objc
[[SKPaymentQueue defaultQueue] finishTransaction:transaction]
```
当购买在苹果后台支付成功时，如果你的App没有调用这个方法，那么苹果就不会认为这次交易彻底成功，当你的App再次启动，并且设置了内购的监听时，监听方法`- (void)paymentQueue:(SKPaymentQueue *)queue updatedTransactions:(NSArray<SKPaymentTransaction *> *)transactions
`就会被调用，直到你调用了上面的方法，注销了这次交易，苹果才会认为这次交易彻底完成。
利用这个特性，我们可以将完成购买后注销方法放到我们向自家后台发送交易成功后调用。
讲到这里，关于内购的大坑我目前遇到的都已经解决啦，当然，你如果实际去操作，可能还会遇到各种各样的小坑，但是没关系，我相信你能够自己解决。。。所以，我就不说啦。
准备爬坑吧！少年！