---
title: "iOS内购支付"
layout: post
date: 2025-01-10 18:31
image: /assets/images/markdown.jpg
headerImage: false
tag:
- ios
- method
star: false
category: tech
author: Lynx
description: iOS苹果内购支付功能实现流程，即Apple In-App Purchase (IAP) Payment。
---



# 简介

App内购分为四种类型：

- 消耗型项目：只可使用一次的产品，使用之后即失效，必须再次购买。
- 非消耗型项目：只需购买一次，不会过期或随着使用而减少的产品。
- 自动续期订阅：允许用户在固定时间段内购买动态内容的产品。除非用户选择取消，否则此类订阅会自动续期。
- 非续期订阅：允许用户购买有时限性服务的产品。此App 内购买项目的内容可以是静态的。此类订阅不会自动续期。

根据需要选择对应的内购类型，在后台设置并在本地实现。四种内购类型代码上基本一致，唯一需要注意的是自动订阅需要在启动的时候设置监听，用于监听订阅周期结束后的续订回调。

当前苹果已经将内购项目和订阅分开，消耗型项目和非消耗型项目在**App内购买项目**列表下，自动续期订阅和非续期订阅在**订阅**列表下，可根据需要选择对应选项并创建商品。

内购设置完成之后会跟随下一个版本提交上架的时候同步更新上去。在新版本上架的时候，**如有新增App内购，则提交的时候会有对应选项出现**，开发者需要注意并将新的内购商品添加上去。



# 苹果后台设置



## 设置税务信息

税务协议必须配置好，否则无法进行支付相关操作。

具体税务配置此处不做描述。



## 创建商品信息

登录[appstoreconnect](https://appstoreconnect.apple.com)，选中对应App，在功能列表里的`营利`分类里根据需要选择`App内购买项目`或者`订阅`。其中App内购买项目包含消耗型项目和非消耗型项目，订阅包含自动续期订阅和非续期订阅。

**App内购买项目**进入后点击新增，可选择消耗型项目和非消耗型项目，并填入产品ID，参考名称仅用于后台展示，根据需要填写名称。

**订阅**进入后默认是自动续期订阅的类型，使用之前需要先创建群组，群组内的订阅用户只能选择一个，用于长期订阅，同样需要填写产品ID和参考名称。在订阅主页面的下方有一个非续期订阅的选项，点击管理可以创建非续期订阅，功能与其他购买功能类似。

参考名称和描述用于AppStore展示，用户支付时看不到，显示名称在用户支付时会显示在支付弹窗。

产品ID用于App购买时传入，需要记录下来开发时使用。



## 创建沙盒账号

当使用xcode进行支付功能的时候，会默认使用沙盒环境实现支付。所以我们需要事先创建沙盒账号用来实现支付功能。



### 沙盒账号创建

登录苹果开发者后台，依次选择用户和访问、沙盒，并点击测试账号的添加按钮，添加测试账号，输入可用邮箱并为该邮箱设置一个密码。注意，该邮箱不能是已被注册的Apple ID，因为生成的测试账号是虚拟的。

创建完成之后就可以进入测试机的`设置`—`App Store`里添加`沙盒账户`了，此处同时可以取消已经购买的订阅等功能。

沙盒账号创建完成之后，就可以正式开始测试支付功能了。

注意：

- 测试需要真机，不能使用模拟器，而且必须不能是越狱的机子，不然不能用；
- 初次集成内购功能时，必须填写税务协议，不然内购的商品ID会无效。



# 代码设置

使用苹果内购功能相关代码需要引入头文件：

```objective_c
#import <StoreKit/StoreKit.h>
```



### 付款请求

此代码调起支付功能：

```objective_c
- (void)subscribeYearlyMemberWithResult:(void(^)(void))result {
    self.Subscribe = result;
    self.verifyVip = NO;
    NSString *proID = XXX;
    if ([SKPaymentQueue canMakePayments]) {
        self.productID = proID;
        [self requestProductData:proID];
    }else{
        dispatch_async(dispatch_get_main_queue(), ^{
            HUD(@"不允许程序内付费")
        });
    }
}

// 收到请求信息
- (void)requestProductData:(NSString *)productID{
    NSLog(@"-------------请求对应的产品信息----------------\nproductID:%@", productID);
    HUD(@"支付中...")
    NSArray *product = [[NSArray alloc] initWithObjects:productID,nil];
    NSSet *nsset = [NSSet setWithArray:product];
    _request = [[SKProductsRequest alloc]initWithProductIdentifiers:nsset];
    _request.delegate = self;
    [_request start];
}
```



### 支付请求回调

调起支付功能之后会执行支付请求的回调，可根据回调结果进行对应处理：

```objective_c
// 收到返回信息
- (void)productsRequest:(SKProductsRequest *)request didReceiveResponse:(SKProductsResponse *)response{
    NSArray *product = response.products;
    if (product.count == 0) {
        dispatch_async(dispatch_get_main_queue(), ^{
            HUD(@"购买失败")
        });
        return;
    }
    SKProduct *prod = nil;
    for (SKProduct *pro in product) {
        if ([pro.productIdentifier isEqualToString:self.productID]) {
            prod = pro;
        }
    }
    // 发送购买请求
    if (prod != nil) {
        SKPayment *payment = [SKPayment paymentWithProduct:prod];
        [[SKPaymentQueue defaultQueue] addPayment:payment];
    }
}

// 失败回调
- (void)request:(SKRequest *)request didFailWithError:(NSError *)error{
    dispatch_async(dispatch_get_main_queue(), ^{
        HUD(@"购买失败")
    });
}

// 支付后的反馈信息
- (void)requestDidFinish:(SKRequest *)request{
}
```



### 交易监听

支付请求完成之后，需要调用交易监听的回调，监听交易的支付情况，并做对应处理。

正常来说支付完成之后需要将收据发送给后台，后台再向苹果进行校验，避免虚假购买。同时沙盒如果不执行校验的话，交易监听有概率回调错误，所以如有可能最好由后台执行校验操作。

调用监听：

```objective_c
 [[SKPaymentQueue defaultQueue] addTransactionObserver:self];
```

- 该功能需要添加协议：`SKPaymentTransactionObserver`。
- 添加协议后需要添加协议方法：`- (void)paymentQueue:(SKPaymentQueue *)queue updatedTransactions:(NSArray<SKPaymentTransaction *> *)transactions`，该方法用于监听购买结果。



协议方法：

```objective_c
// 监听购买结果
- (void)paymentQueue:(SKPaymentQueue *)queue updatedTransactions:(NSArray<SKPaymentTransaction *> *)transactions {
    for (SKPaymentTransaction *tran in transactions) {
        switch (tran.transactionState) {
            case SKPaymentTransactionStatePurchased:    // 交易完成
                if (tran.originalTransaction) { // 如果是自动续费的订单originalTransaction会有内容
                    NSLog(@"自动续费的订单originalTransaction会有内容");
                } else {  // 普通购买，以及 第一次购买 自动订阅
                    NSLog(@"普通购买，以及 第一次购买 自动订阅");
                }
                HUD(@"支付成功")
                [self verifyPurchaseWithPaymentTransactionWith:tran];
                break;
            case SKPaymentTransactionStatePurchasing:   // 正在购买
                NSLog(@"商品已经添加进列表");
                break;
            case SKPaymentTransactionStateRestored:     // 恢复购买
                NSLog(@"已经购买过商品");
                if (!self.verifyVip) {  // 验证一次，已购商品会有多个，避免重复验证
                    self.verifyVip = YES;
                    [self verifyPurchaseWithPaymentTransactionWith:tran];
                } else {
                    [[SKPaymentQueue defaultQueue] finishTransaction:tran];
                }
                break;
            case SKPaymentTransactionStateFailed:
                NSLog(@"购买失败");
                [[SKPaymentQueue defaultQueue] finishTransaction:tran];
                break;
            default:
                break;
        }
    }
}

- (void)paymentQueue:(SKPaymentQueue *)queue restoreCompletedTransactionsFailedWithError:(NSError *)error API_AVAILABLE(ios(3.0), macos(10.7), watchos(6.2)) {
    HUD(@"恢复购买失败")
}

- (void)paymentQueueRestoreCompletedTransactionsFinished:(SKPaymentQueue *)queue API_AVAILABLE(ios(3.0), macos(10.7), watchos(6.2)) {
}

/// 验证购买，避免越狱软件模拟苹果请求达到非法购买问题
-(void)verifyPurchaseWithPaymentTransactionWith:(SKPaymentTransaction *)tran {
    //HUD(@"验证内购信息")
    NSURL *receiptUrl=[[NSBundle mainBundle] appStoreReceiptURL];
    NSData *receiptData=[NSData dataWithContentsOfURL:receiptUrl];
    NSString *receiptString = [receiptData base64EncodedStringWithOptions:NSDataBase64EncodingEndLineWithLineFeed];
    if (!receiptString) {
        return;
    }
    // 网络请求同步购买结果并由后台进行校验操作.
}
```



### 恢复购买

调起苹果内购恢复接口:

```objective_c
[[SKPaymentQueue defaultQueue] restoreCompletedTransactions];
```



### 自动订阅处理

自动订阅和其他购买流程差不多，唯一有区别的是，订阅除了第一次购买是由用户主动触发，后续的续订都由苹果自动完成。在订阅快过期的前24小时开始，苹果会尝试进行扣费。扣费成功则下次App启动的时候会自动推送给App，所以在App启动时需要添加`addTransactionObserver`监听方法，从而及时将购买记录同步到后台。

所以如果是自动订阅，交易监听的方法必须在App启动时调用，正常来说可以直接放在`AppDelegate`里的`didFinishLaunchingWithOptions`方法里。

```objective_c
 [[SKPaymentQueue defaultQueue] addTransactionObserver:self];
```

- 由于在调用支付功能的时候需要调用该方法，自动订阅的时候也需要再App启动时调用，建议将支付功能封装成单例，方便使用。



