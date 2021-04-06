---
title: "使用hitTest实现点击时UIView自身不响应，底下重叠的视图响应的效果"
layout: post
date: 2021-03-29 05:44
image: /assets/images/markdown.jpg
headerImage: false
tag:
- ios
star: false
category: tech
author: Lynx
description: 通过hitTest方法实现点击视图，自身不响应，底下重叠的视图响应及hitTest原理的介绍。
---



> 视图B和C加载在视图A上，B和C有重叠，重叠部分C在B上面。此时要做到点击重叠部分时，C不响应，而B响应，点击不重叠部分时则正常响应的效果。用UIView的hitTest方法实现了该功能。



# hitTest原理

## hitTest方法

```swift
func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView?
```

- point：在接收器的局部坐标系(界)中指定的点。
- event：系统保证调用此方法的事件。如果从事件处理代码外部调用此方法，则可以指定nil。
- 返回值：返回所能包含point的view和view.subviews中最后的一个view。如果point完全位于视图层次结构之外，则返回nil。



## 调用顺序

触摸事件寻找最佳响应者,即hitTest 的调用顺序大致如下：

```swift
touch(UIEvent)->UIApplication->UIWindow->window.subviews->...->view
```

- 当App接收触摸事件时，主线程的`runloop`被唤醒，触发`source1`回调。`source1`回调又触发了一个`source0`回调，将接收到的触摸事件（`IOHIDEvent`对象）封装成`UIEvent`对象，此时APP将正式开始对于触摸事件的响应。`source0`回调将触摸事件添加到UIApplication的事件队列中。

- `UIApplication`会从事件队列中取出最早的事件进行分发处理，首先将事件传递给窗口对象(`UIWindow`)，如果有多个`UIWindow`对象，则先选择最后加上的`UIWindow`对象。

- `UIWindow`会调用其`hitTest:withEvent:`方法在视图(`UIView`)层次结构中找到一个最合适的`UIView`来处理触摸事件。



## 触摸事件的传递顺序

通过hitTest我们已经找到了最佳响应者，下面要做的事就是让这个最佳响应者响应触摸事件。这个最佳响应者对于触摸事件拥有决定权，它可以决定是自己独自响应这个事件，也可以自己响应之后还把它传递给其他响应者。

事件传递顺序大致为：

```swift
view -> superView ...- > UIViewController.view -> UIViewController -> UIWindow -> UIApplication -> 事件丢弃
```

1、 首先由 view 来尝试处理事件，如果他处理不了，事件将被传递到他的父视图superview

2、superview 也尝试来处理事件，如果他处理不了，继续传递他的父视图
 UIViewcontroller.view

3、UIViewController.view尝试来处理该事件，如果处理不了，将把该事件传递给UIViewController

4、UIViewController尝试处理该事件，如果处理不了，将把该事件传递给主窗口Window

5、主窗口Window尝试来处理该事件，如果处理不了，将传递给应用单例Application

6、如果Application也处理不了，则该事件将会被丢弃。



所以当我们点击该视图时，父视图会依次调用该方法，点击事件皆为点击的视图。其顺序为，当前点击的视图调用方法，返回点击视图。然后父视图也会调用一遍该方法，同样返回点击视图，以此类推。



# 实现

## 实现效果

已知视图B和C分别加载在视图A上，其中B在C底下，当我们点击C与B的重叠部分之后，要求返回的响应结果重叠部位底下的视图，即B视图的，而非默认的C，且其他的地方的点击正常。

![](https://i.niupic.com/images/2021/04/02/9glh.png)

## 实现方法

当点击该视图时，获取其父视图的子视图数组，并依次遍历，判断该子视图是否在该点击坐标范围之内，如在且该视图不是点击的视图时，就判断是重叠的视图，返回该视图，则原本的点击事件就被替换为该视图底下重叠视图的点击视图了。

使用`func convert(_ point: CGPoint, to coordinateSpace: UICoordinateSpace) -> CGPoint`方法将当前的坐标转换为目标视图的坐标。

使用CALayer的`open func contains(_ p: CGPoint) -> Bool`方法，判断该点是否落在对应的视图的layer的范围内。

代码实现：

~~~swift
override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
    let view = super.hitTest(point, with: event)
    guard view == self else {
        return view
    }
    guard let superView = view?.superview else {
        return view
    }
    print(String(format: "self:%p, view:%p", self, view!))
    for inView in superView.subviews {
        if let relatePoint = view?.convert(point, to: inView),
inView.layer.contains(relatePoint) && inView != view {
            ///点击重叠范围时，更改重叠部位底下视图的颜色，并返回底下的视图。
            inView.backgroundColor = fetchColor()
            return inView
        }
    }
    return view
}
~~~



## 注意点

1、hitTest的point参数是调用它的对象对应的坐标，而我们要对点击视图的父视图的子视图进行遍历并对比坐标，判断坐标是否落在视图上，则要保证调用的对象就是我们点击的视图，而不是其父视图。
所以在调用该方法的时候要先判断调用的对象是否是他自己：

```swift
let view = super.hitTest(point, with: event)
guard view == self else {
    return view
}

```

2、使用`view.layer.contains(point)`方法判断坐标的时候，其坐标是以view为准的，其坐标范围为(0, 0, width, height)，需要注意。



## 代码示例

GitHub：[https://github.com/MichaelLynx/HitTest](https://github.com/MichaelLynx/HitTest)

