---
title: "类方法"
layout: post
date: 2020-01-21 20:51
image: /assets/images/markdown.jpg
headerImage: false
tag:
- ios
- method
star: false
category: tech
author: Lynx
description: 该文档用于记录iOS中常用的类方法。最新更改时间：2020/04/03
---

> 该文档用于记录iOS中常用的类方法



# 1.[String]字符串方法

## 1.1 获取某字符的位置及其富文本设置

- 由于`swift`原生的字符串处理方法依然不方便，当前对字符串的部分内容进行处理依然采用`NSString`的方法
- `NSRange`的结构里包含`location`和`length`，可以较为方便获取某个字符的当前位置

```swift
let str = "abcd 1234"
let attrStr = NSMutableAttributedString(string: str)
//获取空格处的位置信息
let midIndex = (str as NSString).range(of: " ").location
let range1 = NSRange(location: 0, length: midIndex)
let range2 = NSRange(location: midIndex, length: str.count - midIndex)

//设置富文本
attrStr.setAttributes([NSAttributedString.Key.font : UIFont.systemFont(ofSize: 28)], range: range1)

//当设置为UILabel的富文本时，未进行设置的富文本按默认格式展示
label.attributedText = attrStr

```

## 1.2 字符串富文本文字上下方向上的位移

- 当我们设置富文本文字大小不一致时，小号字体文字默认位于下方
- 如果使用的字体格式比较特别甚至会出现小号的字体位于下方偏下的位置，无法和大号字体对齐
- 文本框(`UILabel`)只有水平位置上的对齐，没有文字垂直方向上的调整，故要实现只能借助富文本的方法对其位置进行移动
- `NSAttributedString`的`baselineOffset`方法可以使富文本的文字位置以底端为基准进行移动，可以通过文字大小大致计算并实现底端对齐或者垂直居中对齐

```swift
attrStr.setAttributes([NSAttributedString.Key.baselineOffset: 5], range: range1)
```

<br>



# 2.UIColor

## 2.1 颜色十六进制转换

> 将颜色的十六进制字符串转换为具体颜色样式



```swift
extension UIColor{
    convenience init(valueStr:String) {
        let scanner:Scanner = Scanner(string:valueStr)
        var valueRGB:UInt32 = 0
        if scanner.scanHexInt32(&valueRGB) == false {
            self.init(red: 0,green: 0,blue: 0,alpha: 0)
        }else{
            self.init(
                red:CGFloat((valueRGB & 0xFF0000)>>16)/255.0,
                green:CGFloat((valueRGB & 0x00FF00)>>8)/255.0,
                blue:CGFloat(valueRGB & 0x0000FF)/255.0,
                alpha:CGFloat(1.0)
            )
        }
    }
}

```

<br>



# 3.时间及时间戳处理

- 正常的调用`Date()`方法获取的是手机里显示的时间，而非标准时间

## 3.1 获取时间戳

```swift
//获取当前手机设置的时间，而非标准时间
let now = Date()
let timeInterval = now.timeIntervalSince1970
print("当前的时间戳：\(timeInterval)")

```

## 3.2 将时间戳转换为具体时间

```swift
let date = Date(timeIntervalSince1970: timeInterval)
let formatter = DateFormatter()
formatter.dateFormat = "yyyy-MM-dd HH:mm:ss"
print("时间戳对应的时间：\(formatter.string(from: date))")

```

## 3.3 将Date直接转换为具体时间

```swift
let now = Date()
let formatter = DateFormatter()
formatter.dateFormat = "yyyy-MM-dd HH:mm:ss"
print("时间戳对应的时间：\(formatter.string(from: now))")
```

## 3.4设置当天整点时间

- 对设置以外的较小时间单位默认缺省置零，如只设置到日，则时分秒皆置零

```objective-c
NSDate *now = [NSDate date];
NSCalendar *calendar = [NSCalendar currentCalendar];
NSDateComponents *components = [calendar components:NSCalendarUnitYear|NSCalendarUnitMonth|NSCalendarUnitDay|NSCalendarUnitHour|NSCalendarUnitMinute fromDate:now];
NSDate *startDate = [calendar dateFromComponents:components];
NSInteger timeStamp = [startDate timeIntervalSince1970];

NSLog(@"now:%@, date:%@, stamp:%ld", now, startDate, timeStamp);
```



<br>



# 4.Data

## 4.1 data转换为十六进制字符串

```swift
extension Array where Element == UInt8 {
    var hexString: String {
        return self.compactMap { String(format: "%02x", $0).uppercased() }
            .joined(separator: "")
    }
}
```

<br>



# 5.Dictionary

## 5.1 重载操作符实现字典添加字典

```swift
func += <KeyType, ValueType> ( left: inout Dictionary<KeyType, ValueType>, right: Dictionary<KeyType, ValueType>) {
    for (k, v) in right {
        left.updateValue(v, forKey: k)
    }
}
```

- 直接添加进swift文件内，需要时直接使用`+=`操作符将另一个字典载入当前字典内

<br>

