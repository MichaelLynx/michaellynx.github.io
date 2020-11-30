---
title: "swift封装Framework静态库SDK"
layout: post
date: 2020-11-30 17:22
image: /assets/images/markdown.jpg
headerImage: false
tag:
- ios
- swift
- framework
- sdk
star: false
category: tech
author: Lynx
description: 开发时我们有时候会用到第三方的SDK，使用的时候直接将framework文件添加进去即可。实际上我们也可以自己封装SDK。
---



> 开发时我们有时候会用到第三方的SDK，使用的时候直接将framework文件添加进去即可。实际上我们也可以自己封装SDK。
>
> 封装framework文件的好处之一是能剥离相关代码，只保留接口，具有高内聚低耦合的优点。更重要的是，一旦封装成SDK，内部的代码就统统不可见了，使用者只能看到暴露出来的方法名，当需要让其他人或者公司接入己方功能和业务的时候，该做法能有效保证己方代码的安全性。
>
> objective-c的封装方法更为简单，此处我们主要讲怎么封装swift的第三方库，以及注意点。



## 一、注意点

### 静态库与动态库的区别

在oc开发中，我们涉及的一般是静态库和动态库，静态库表现为.a和.framework文件，动态库表现为.dylib和.framework文件。

静态库：

链接时完整地拷贝至可执行文件中，被多次使用就有多份冗余拷贝。表现形式为 .a和.framework。

动态库：

链接时不复制，程序运行时由系统动态加载到内存，供程序调用，系统只加载一次，多个程序共用，节省内存。 表现形式为 .dylib和.framework。

本文创建的是静态库。



### .a文件和.framework文件区别

.a文件是纯二进制的文件，使用时还需要有一个.h文件，里面不包含资源文件，如需要添加资源需另外引入。

.framework文件除了二进制文件之外还包含资源文件，封装完成后可直接使用。

使用framework进行开发是较为主流的开发方式，本文使用的就是该种开发方式。



### 注意事项

开发时需要将**Build Settings**里面的**Mach-O Type**选为**Static Library**，即静态库，系统默认选中的是Dynamic Library动态库，需要主动进行修改。

如纯oc项目引入swift开发的静态库，则会出现大量报错，此时创建一个swift文件及桥接文件即可，桥接文件不需要填入framework文件的内容，直接放着就行，生成桥接文件之后swift文件不能删除，必须保证项目内存在swift文件，不然也会报错。

</br>



## 二、开发流程

### 1.新建framework工程

- 打开Xcode，使用`command+shift+N`新建项目；
- 在列表里选中`Cocoa Touch Framework`并点击下一步；
- 填入相关信息，其中生成的framework库默认和项目名一致，并于每次编译后生成在Products文件夹内；
- 新建完成之后需要给Target设置SDK兼容的最低版本，该设置和正常创建项目一样，直接选择最低版本即可；



### 2.开发

#### 2.1 兼容oc

oc的代码可以兼容swift，swift的却不行，需要主动设置`@objc`的标识来让方法可以被oc项目调用。

swift的方法相比oc多了一些特性，所以也导致部分的swift代码无法被oc引用，此时如果在这类型的代码前加`@objc`则会发现代码报错，需要将该类型代码改成和oc兼容。

比如oc的枚举只支持整型，如swift使用了字符串型的枚举类型，则该枚举类型oc无法使用。又或者，swift允许方法名一致，方法内部使用的变量不同，且其传入的变量允许直接赋初值，也允许在调用该方法时不展示该变量，而oc则不支持。这些类型的代码无法转换成oc可以调用的方法，如需要调用此类代码则需要修改swift的代码使其符合oc的语法规范。



#### 2.2 访问修饰符

swift的类和方法如需要暴露出来可以使用`Public`和`open`访问修饰符进行修饰，对于需要暴露出来的类、方法和变量，在前面加上关键字`Public`即可。如需允许`override `和继承，则需要使用`open`关键字。



#### 2.3 生成framework库

当第三方库内容编写完成之后就需要生成对应的SDK了，此时使用`command+B`进行编译或者点击`Product > Build`运行项目之后就会生成对应的framework文件了。

framework文件分为两种类型，分别是真机使用的和模拟器使用的。

当编译或者运行时选择`Generic iOS Device`或者真机时，`Products`文件夹内生成的是真机使用的SDK；当选择模拟器时，生成的是模拟器使用的SDK，生成对应SDK是可以通过右键点击`Show in Finder`拿到对应的SDK。此时生成的SDK已经可以使用了，但如果给项目导入真机的SDK，则该项目无法使用模拟器进行测试。

此时，可以将两个SDK进行合并，合并后生成的SDK即可用于模拟器测试，也可真机运行。



#### 2.4 合并framework

真机和模拟器下分别编译可以得到两个`项目名.framework`的SDK文件，**该文件内部有一个名字和项目名一致的执行文件，我们需要合并的就是该执行文件**，通过命令把真机和模拟器的执行文件进行合并后，将其替换上述framework里的执行文件即可，至此framework封装就完成了。

在终端中输入该命令将两个SDK的执行文件合并并生成真机模拟器都可用的SDK：

~~~objective-c
lipo -create 模拟器framework路径  真机framework路径 -output 新的文件
~~~

示例：

```objective-c
lipo -create /A/B/test.framework/test /a/b/test.framework/test -output /C/D/test
```



### 3.使用

将生成的framework文件添加到项目中，然后在`General->Frameworks, Libraries, and Embedded Content`中将其再次添加进去，此时该文件右边的状态显示为`Embed & Sign`。也可以直接在`General->Frameworks, Libraries, and Embedded Content`中将其右边的状态设置为`Embed & Sign`。不同Xcode版本该目录名称会不一致，旧版的是`General->Embedded Binaries`。

以上方法每次更新都需要重新添加，也有更简便的方法添加SDK：将该SDK放入项目的文件夹内，然后在`General->Frameworks, Libraries, and Embedded Content`中将其引入，其状态自动为`Embed & Sign`，则每次更新SDK只需要直接替换项目文件夹内部的SDK文件即可。

