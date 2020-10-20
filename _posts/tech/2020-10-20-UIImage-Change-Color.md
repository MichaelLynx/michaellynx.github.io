---
title: "iOS修改图片颜色的方法"
layout: post
date: 2020-10-20 16:44
image: /assets/images/markdown.jpg
headerImage: false
tag:
- ios
- method
star: false
category: tech
author: Lynx
description: 更改图片颜色的相关方法。
---



# 1. 图片染色

> 将图片染成想要的颜色，适合带有透明度的png图片。

以下为UIImage的类扩展：

.h文件：

~~~objective-c
#import <UIKit/UIKit.h>

NS_ASSUME_NONNULL_BEGIN

@interface UIImage (ChangeColor)

-(UIImage*)imageWithColor:(UIColor*)color;

- (UIImage *)originalImageWithColor:(UIColor *)color;

@end

NS_ASSUME_NONNULL_END
~~~

.m文件：

~~~objective-c
#import "UIImage+ChangeColor.h"

@implementation UIImage (ChangeColor)

//绘图
-(UIImage*)imageWithColor:(UIColor*)color
{
    //获取画布
    UIGraphicsBeginImageContextWithOptions(self.size, NO, 0.0f);
    //画笔沾取颜色
    [color setFill];
    
    CGRect bounds = CGRectMake(0, 0, self.size.width, self.size.height);
    UIRectFill(bounds);
    //绘制一次
    [self drawInRect:bounds blendMode:kCGBlendModeOverlay alpha:1.0f];
    //再绘制一次
    [self drawInRect:bounds blendMode:kCGBlendModeDestinationIn alpha:1.0f];
    //获取图片
    UIImage *img = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return img;
}

- (UIImage *)originalImageWithColor:(UIColor *)color {
    UIImage *img = [self imageWithColor:color];
    img = [img imageWithRenderingMode:(UIImageRenderingModeAlwaysOriginal)];
    return img;
}

@end
~~~

当对作为tabbar的图标做染色时发现图片被显示成系统默认的蓝色了，此时可以调用以下方法对图片做进一层封装，调用时使用以下方法即可：

~~~objective-c
- (UIImage *)originalImageWithColor:(UIColor *)color {
    UIImage *img = [self imageWithColor:color];
    img = [img imageWithRenderingMode:(UIImageRenderingModeAlwaysOriginal)];
    return img;
}
~~~

使用方式：

~~~objective-c
UIImage *image= [UIImage imageNamed:@"tab_home_selected"];
image = [image imageWithColor:[UIColor redColor]];
~~~

参考文章：[iOS改变图片的颜色](https://www.jianshu.com/p/10047407463c)



# 2. 更改图片中的特定颜色

> 有时候如果要根据主题或者主色调对图片进行更改时，可调用以下方法，将一定范围内的颜色更改成指定的颜色。

以下为UIImage的类扩展：

.h文件：

~~~objective-c
#import <UIKit/UIKit.h>

NS_ASSUME_NONNULL_BEGIN

@interface UIImage (Rander)

- (UIImage *)translatePixelColorByTargetNearBlackColor:(UIColor *)nearBlackColor nearWhiteColor:(UIColor *)nearWhiteColor transColor:(UIColor *)transColor;

- (UIImage *)translatePixelColorByTargetNearBlackColorHex:(UInt32)nearBlackRGB nearWhiteColorHex:(UInt32)nearWhiteRGB transColorHex:(UInt32)transRGB;

- (UIImage *)translatePixelColorByTargetNearBlackColorRGBA:(UInt32)nearBlackRGBA nearWhiteColorRGBA:(UInt32)nearWhiteRGBA transColorRGBA:(UInt32)transRGBA inRect:(CGRect)rect;

@end

NS_ASSUME_NONNULL_END
~~~

.m文件

~~~objective-c
#import "UIImage+Rander.h"
#import "UIColor+RGB.h"

@implementation UIImage (Rander)

- (UIImage *)translatePixelColorByTargetNearBlackColor:(UIColor *)nearBlackColor
                                        nearWhiteColor:(UIColor *)nearWhiteColor
                                            transColor:(UIColor *)transColor {
    CGRect rect = CGRectMake(0, 0, self.size.width, self.size.height);
    return [self translatePixelColorByTargetNearBlackColor:nearBlackColor nearWhiteColor:nearWhiteColor transColor:transColor inRect:rect];
}

- (UIImage *)translatePixelColorByTargetNearBlackColor:(UIColor *)nearBlackColor
                                        nearWhiteColor:(UIColor *)nearWhiteColor
                                            transColor:(UIColor *)transColor
                                                inRect:(CGRect)rect {
    // UIColor 转 RGBA
    UInt32 nearBlackRGBA = nearBlackColor.RGBA;
    UInt32 nearWhiteRGBA = nearWhiteColor.RGBA;
    UInt32 transRGBA = transColor.RGBA;

    return [self translatePixelColorByTargetNearBlackColorRGBA:nearBlackRGBA nearWhiteColorRGBA:nearWhiteRGBA transColorRGBA:transRGBA inRect:rect];
}


- (UIImage *)translatePixelColorByTargetNearBlackColorHex:(UInt32)nearBlackRGB
                                        nearWhiteColorHex:(UInt32)nearWhiteRGB
                                            transColorHex:(UInt32)transRGB {
    CGRect rect = CGRectMake(0, 0, self.size.width, self.size.height);
    return [self translatePixelColorByTargetNearBlackColorHex:nearBlackRGB nearWhiteColorHex:nearWhiteRGB transColorHex:transRGB inRect:rect];
}


- (UIImage *)translatePixelColorByTargetNearBlackColorHex:(UInt32)nearBlackRGB
                                        nearWhiteColorHex:(UInt32)nearWhiteRGB
                                            transColorHex:(UInt32)transRGB
                                                   inRect:(CGRect)rect {
    // RGB 转 RGBA
    UInt32 nearBlackRGBA = (nearBlackRGB << 8) + 0xFF;
    UInt32 nearWhiteRGBA = (nearWhiteRGB << 8) + 0xFF;
    UInt32 transRGBA = (transRGB << 8) + 0xFF;
    
    return [self translatePixelColorByTargetNearBlackColorRGBA:nearBlackRGBA nearWhiteColorRGBA:nearWhiteRGBA transColorRGBA:transRGBA inRect:rect];
}

/**
解释一下前两个参数的含义：
想象一个数轴，最左边是黑色（RGBX：0x000000FF），最右边是白色（0xFFFFFFFF），
nearBlackColor是靠近左边边界的色值，nearWhiteColor是靠近右边边界的色值，
它们中间则是需要被修改的色值范围
*/
- (UIImage *)translatePixelColorByTargetNearBlackColorRGBA:(UInt32)nearBlackRGBA
                                        nearWhiteColorRGBA:(UInt32)nearWhiteRGBA
                                            transColorRGBA:(UInt32)transRGBA
                                                   inRect:(CGRect)rect {
    // 第一步：判断传入的rect是否在图片的bounds内
    CGRect canvas = CGRectMake(0, 0, self.size.width, self.size.height);
    if (!CGRectContainsRect(canvas, rect)) {
        if (CGRectIntersectsRect(canvas, rect)) {
            rect = CGRectIntersection(canvas, rect);    // 取交集
        } else {
            return self;
        }
    }
    
    UIImage *transImage = nil;
    
    int imageWidth = self.size.width;
    int imageHeight = self.size.height;
    
    // 第二步：创建色彩空间、画布上下文，并将图片以bitmap（不含alpha通道）的方式画在画布上。
    size_t bytesPerRow = imageWidth * 4;
    uint32_t *rgbImageBuf = (uint32_t *)malloc(bytesPerRow * imageHeight);
    CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();
    
    CGContextRef context = CGBitmapContextCreate(rgbImageBuf, imageWidth, imageHeight, 8, bytesPerRow, colorSpace,
                                                 kCGBitmapByteOrder32Little | kCGImageAlphaNoneSkipLast);
    
    CGContextDrawImage(context, CGRectMake(0, 0, imageWidth, imageHeight), self.CGImage);
    
    // 第三步：遍历并修改像素
    uint32_t *pCurPtr = rgbImageBuf;
    pCurPtr += (long)(rect.origin.y*imageWidth);    // 将指针移动到初始行的起始位置
    
    // 空间复杂度：O(rect.size.width * rect.size.height)
    for (int i = rect.origin.y; i < CGRectGetMaxY(rect); i++) {                     // row
        pCurPtr += (long)rect.origin.x;             // 将指针移动到当前行的起始列
        
        for (int j = rect.origin.x; j < CGRectGetMaxX(rect); j++, pCurPtr++) {      // column
            if (*pCurPtr < nearBlackRGBA || *pCurPtr > nearWhiteRGBA) { continue; }
            
            // 将图片转成想要的颜色
            uint8_t *ptr = (uint8_t *)pCurPtr;
            ptr[3] = (transRGBA >> 24) & 0xFF;              // R
            ptr[2] = (transRGBA >> 16) & 0xFF;              // G
            ptr[1] = (transRGBA >> 8)  & 0xFF;              // B
        }
        
        pCurPtr += (long)(imageWidth - CGRectGetMaxX(rect));    // 将指针移动到下一行的起始列
    }
    
    // 第四步：输出图片
    CGDataProviderRef dataProvider = CGDataProviderCreateWithData(NULL, rgbImageBuf, bytesPerRow * imageHeight, providerReleaseDataCallback);
    CGImageRef imageRef = CGImageCreate(imageWidth, imageHeight, 8, 32, bytesPerRow, colorSpace,
                                        kCGImageAlphaLast | kCGBitmapByteOrder32Little, dataProvider,
                                        NULL, true, kCGRenderingIntentDefault);
    CGDataProviderRelease(dataProvider);
    transImage = [UIImage imageWithCGImage:imageRef];
    
    // end：清理空间
    CGImageRelease(imageRef);
    CGContextRelease(context);
    CGColorSpaceRelease(colorSpace);
    
    return transImage ? : self;
}

void providerReleaseDataCallback (void *info, const void *data, size_t size) {
    free((void*)data);
}

@end
~~~

该方法调用了UIColor的类扩展，以下是扩展内容

.h文件

~~~objective-c
#import <UIKit/UIKit.h>

NS_ASSUME_NONNULL_BEGIN

@interface UIColor (RGB)

- (UInt32)RGBA;

@end

NS_ASSUME_NONNULL_END
~~~

.m文件

~~~objective-c
#import "UIColor+RGB.h"

@implementation UIColor (RGB)

- (UInt32)RGBA {
    CGFloat red = 0;
    CGFloat green = 0;
    CGFloat blue = 0;
    CGFloat alpha = 0;
    
    BOOL succ = [self getRed:&red green:&green blue:&blue alpha:&alpha];
    
    UInt32 r = round(red*255);
    UInt32 g = round(green*255);
    UInt32 b = round(blue*255);
    UInt32 a = round(alpha*255);

    r = (r << 24);
    g = (g << 16);
    b = (b << 8);
    
    UInt32 rgba = r + g + b + a;
    return succ ? rgba : 0x00000000;
}

@end
~~~

参考文章：[iOS修改图片颜色（修改像素色值）](https://www.jianshu.com/p/619ef8423895)



