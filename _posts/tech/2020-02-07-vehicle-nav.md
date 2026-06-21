---
title: "iOS调用第三方地图App进行导航方法"
layout: post
date: 2020-02-07 14:20
image: /assets/images/markdown.jpg
headerImage: false
tag:
- method
- 导航
star: false
category: tech
author: Lynx
description: oc跳转到第三方地图进行导航的方法。
---



# 介绍

- App内根据手机上装载的地图App将其显示在弹出的选择框，选择对应地图跳转进入地图导航。需要用到``- (BOOL)canOpenURL:(NSURL *)url NS_AVAILABLE_IOS(3_0);``方法判断手机是否已安装相应地图App。
- 要进行跳转需要先在xcode的plist文件内将目标App的url Scheme加入白名单(LSApplicationQueriesSchemes)。

## 常见第三方地图App的url Scheme

- 百度地图：baidumap
- 高德地图：iosamap
- 谷歌地图：comgooglemaps
- 腾讯地图：qqmap

## plist白名单设置

plist文件新增``LSApplicationQueriesSchemes``关键字，类型为NSArray，并在其下添加子目录，类型为NSString，内容为各地图对应的url Scheme。

![白名单LSApplicationQueriesSchemes](http://upload-images.jianshu.io/upload_images/3820289-92f9f67b5106c0cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 代码

```objective_c

//导航只需要目的地经纬度，endLocation为纬度、经度的数组
-(void)doNavigationWithEndLocation:(NSArray *)endLocation {
    
    //NSArray * endLocation = [NSArray arrayWithObjects:@"26.08",@"119.28", nil];
    
    NSMutableArray *maps = [NSMutableArray array];
    
    //苹果原生地图-苹果原生地图方法和其他不一样
    NSMutableDictionary *iosMapDic = [NSMutableDictionary dictionary];
    iosMapDic[@"title"] = @"苹果地图";
    [maps addObject:iosMapDic];
    
    //百度地图
    if ([[UIApplication sharedApplication] canOpenURL:[NSURL URLWithString:@"baidumap://"]]) {
        NSMutableDictionary *baiduMapDic = [NSMutableDictionary dictionary];
        baiduMapDic[@"title"] = @"百度地图";
        NSString *urlString = [[NSString stringWithFormat:@"baidumap://map/direction?origin={{我的位置}}&destination=latlng:%@,%@|name=北京&mode=driving&coord_type=gcj02",endLocation[0],endLocation[1]] stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
        baiduMapDic[@"url"] = urlString;
        [maps addObject:baiduMapDic];
    }
    
    //高德地图
    if ([[UIApplication sharedApplication] canOpenURL:[NSURL URLWithString:@"iosamap://"]]) {
        NSMutableDictionary *gaodeMapDic = [NSMutableDictionary dictionary];
        gaodeMapDic[@"title"] = @"高德地图";
        NSString *urlString = [[NSString stringWithFormat:@"iosamap://navi?sourceApplication=%@&backScheme=%@&lat=%@&lon=%@&dev=0&style=2",@"导航功能",@"nav123456",endLocation[0],endLocation[1]] stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
        gaodeMapDic[@"url"] = urlString;
        [maps addObject:gaodeMapDic];
    }
    
    //谷歌地图
    if ([[UIApplication sharedApplication] canOpenURL:[NSURL URLWithString:@"comgooglemaps://"]]) {
        NSMutableDictionary *googleMapDic = [NSMutableDictionary dictionary];
        googleMapDic[@"title"] = @"谷歌地图";
        NSString *urlString = [[NSString stringWithFormat:@"comgooglemaps://?x-source=%@&x-success=%@&saddr=&daddr=%@,%@&directionsmode=driving",@"导航测试",@"nav123456",endLocation[0], endLocation[1]] stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
        googleMapDic[@"url"] = urlString;
        [maps addObject:googleMapDic];
    }
    
    //腾讯地图
    if ([[UIApplication sharedApplication] canOpenURL:[NSURL URLWithString:@"qqmap://"]]) {
        NSMutableDictionary *qqMapDic = [NSMutableDictionary dictionary];
        qqMapDic[@"title"] = @"腾讯地图";
        NSString *urlString = [[NSString stringWithFormat:@"qqmap://map/routeplan?from=我的位置&type=drive&tocoord=%@,%@&to=终点&coord_type=1&policy=0",endLocation[0], endLocation[1]] stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
        qqMapDic[@"url"] = urlString;
        [maps addObject:qqMapDic];
    }
    
    //选择
    UIAlertController * alert = [UIAlertController alertControllerWithTitle:@"选择地图" message:nil preferredStyle:UIAlertControllerStyleActionSheet];
    
    NSInteger index = maps.count;
    for (int i = 0; i < index; i++) {
        NSString * title = maps[i][@"title"];
        
        //苹果原生地图方法
        if (i == 0) {
            UIAlertAction * action = [UIAlertAction actionWithTitle:title style:(UIAlertActionStyleDefault) handler:^(UIAlertAction * _Nonnull action) {
                [self navAppleMap];
            }];
            [alert addAction:action];
            continue;
        }
        
        UIAlertAction * action = [UIAlertAction actionWithTitle:title style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
            NSString *urlString = maps[i][@"url"];
            [[UIApplication sharedApplication] openURL:[NSURL URLWithString:urlString]];
        }];
        [alert addAction:action];
    }
    
    [self presentViewController:alert animated:YES completion:nil];
}



//苹果地图
- (void)navAppleMap
{
//    CLLocationCoordinate2D gps = [JZLocationConverter bd09ToWgs84:self.destinationCoordinate2D];
    
    //终点坐标
    CLLocationCoordinate2D loc = CLLocationCoordinate2DMake(26.08, 119.28);
    
    //用户位置
    MKMapItem *currentLoc = [MKMapItem mapItemForCurrentLocation];
    //终点位置
    MKMapItem *toLocation = [[MKMapItem alloc]initWithPlacemark:[[MKPlacemark alloc]initWithCoordinate:loc addressDictionary:nil] ];
    
    NSArray *items = @[currentLoc,toLocation];
    //第一个
    NSDictionary *dic = @{
                          MKLaunchOptionsDirectionsModeKey : MKLaunchOptionsDirectionsModeDriving,
                          MKLaunchOptionsMapTypeKey : @(MKMapTypeStandard),
                          MKLaunchOptionsShowsTrafficKey : @(YES)
                          };
    //第二个，都可以用
//    NSDictionary * dic = @{MKLaunchOptionsDirectionsModeKey: MKLaunchOptionsDirectionsModeDriving,
//                           MKLaunchOptionsShowsTrafficKey: [NSNumber numberWithBool:YES]};
    
    [MKMapItem openMapsWithItems:items launchOptions:dic];
}
```

<br>

# 附录-相关链接

> 参考链接：[IOS实现应用内打开第三方地图app进行导航](http://blog.csdn.net/a416863220/article/details/51220739)
> 相关链接：[iOS 跳转方式实现地图导航功能](http://www.jianshu.com/p/1277346a8bb4)
> 相关链接：[Xcode常用设置](http://www.jianshu.com/p/fc3677f2cd1a)