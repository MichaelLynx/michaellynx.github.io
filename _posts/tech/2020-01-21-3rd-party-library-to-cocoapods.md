---
title: "上传第三方库到cocoapods"
layout: post
date: 2020-01-21 16:51
image: /assets/images/markdown.jpg
headerImage: false
tag:
- ios
- cocoapods
star: false
category: tech
author: Lynx
description: 之前需要实现一个功能，网上找了一圈没看到别人做过，就自己实现了，完成后突发奇想将其开源到cocoapods，说做就做。
---



## 1.1 在GitHub创建项目

- 填写项目名

- 填写项目描述
- 设置库公开(Public)
- 添加证书及README文件，证书也可以在项目创建完成后点击`Create new file`并输入`LICENSE`，点击右上角出现的`Choose a license template`选择需要的证书并添加进去

设置完成后直接克隆到本地，并将工程代码复制进该文件夹

## 1.2 创建podspec文件

### 1.2.1 通过终端进克隆到本地的文件夹

```
cd xxx
```

### 1.2.2 创建podspec文件

```
pod spec create xxx
```

- xxx为自己创建的库的名字，如`Alamofire`

### 1.2.3 配置podspec文件

```ruby
@version = "0.0.1"
Pod::Spec.new do |s|
s.name = "MCircleBoard"
s.version = @version
s.summary = "MCircleBoard"
s.description = "MCircleBoard is a dash board."
s.homepage = "https://github.com/MichaelLynx/MCircleBoard"
s.license = { :type => "MIT", :file => "LICENSE" }
s.author = { "xxx" => "xxxx@xx.com" }
s.platform = :ios
s.ios.deployment_target = "8.0"
s.source = { :git => "https://github.com/MichaelLynx/MCircleBoard.git", :tag => "v#{s.version}" }
s.source_files = "Code/*.swift"
s.resources = ["Code/*.xcassets"]
s.swift_version = "4.0"
s.requires_arc = true
s.framework = "UIKit"

end
```

- 如果有依赖库则增加如：`s.dependency "AFNetworking", '~> 3.0'`
- 比较重要的字段有：`name`、`version`、`summary`、`homepage`、`license`、`author`、`deployment_target`、`source`、`source_files`、`requires_arc`
- `source_files`：最好把要上传的文件放在专门的文件夹下
- 往库里添加资源：添加bundle然后设置属性`resource`\\`resources`\\`resource_bundles`



spec里添加资源：

```
spec.resource = 'Code/MCircleBoardIcon.bundle'
spec.resources = ['Images/*.png', 'Sounds/*']
spec.resource_bundles = {
    'YourBundleName' => ['MapView/Map/Resources/*.png'],
    'OtherResources' => ['MapView/Map/OtherResources/*.png']
  }
```

bundle内图片的调用方式：

```swift
var bundle = Bundle(for: theClass.self)
if let resoucePath = bundle.path(forResource: "bundleName", ofType: "bundle"), let resouceBundle = Bundle(path: resoucePath) {
    bundle = resouceBundle
}
let image = UIImage(named: imageName, in: bundle, compatibleWith: nil) ?? UIImage()
```

- 要设置成为第三方库，获取图片只能通过该方式，而不能像主工程项目里的获取方式
- `theClass`：工程项目名称；`bundleName`：对应的bundle的名字



podspec字段说明：

| 字段              | 作用                                                       |
| ----------------- | ---------------------------------------------------------- |
| name              | 库的名称                                                   |
| version           | 版本号，该号一定要和GitHub上的对应，参见下面的步骤         |
| summary           | 该库的一个简介                                             |
| homepage          | 这个库的主页，对应Github上的地址                           |
| license           | 采用何种license，和创建GitHub时候选择的那个要对应          |
| author            | 作者                                                       |
| deployment_target | 可以使用该库的iOS的最低版本                                |
| source            | 数据源，对应的是你`clone`的时候使用的那个`HTTPS`的链接地址 |
| source_files      | 哪些文件是要上传到pods的                                   |
| requires_arc      | 是否在ARC环境下使用                                        |
| dependency        | 该库还引用了哪些第三方库                                   |



## 1.3 GitHub上创建发布版本

- podspec文件验证无误后需要先将项目更新到GitHub上
- 更新完毕后再release创建新版本，`Tag version`以v开头，后接版本号，标题`release title`需要填写
- 如果操作过程中有对代码进行了更改，则需要删除release，上传代码并重新创建该release



## 1.4 注册Trunk账号及上传项目

### 1.4.1 注册cocoapods账号

> 如果已经注册成功账号，则这步跳过

在终端输入指令：

```ruby
pod trunk register 邮箱地址 '用户名' —description='描述信息'
或
pod trunk register 邮箱地址 '用户名'  --verbose
```

- 用户名后面的信息可不写
- 注册成功后会提示去邮箱验证信息
- 注册的信息最好和GitHub保持一致，方便记忆
- **这里注册的邮箱地址和用户名即podspec的author**



验证成功后在终端输入以下命令确认是否注册成功

```ruby
pod trunk me
```

- 出现用户信息则注册成功



### 1.4.2 验证podspec文件

>  如果确认podspec文件没问题则可以不进行验证
>
>  如果配置得正确，则输入验证命令后会提示：`passed validation.`

podspec文件配置完成之后可以在终端输入指令进行验证

```ruby
pod spec lint XXX.podspec
```

- 如果验证的结果有error则必须按要求进行处理；如果有warning则可以选择处理也可以忽视
- 后面的`XXX.podspec`可省略
- `pod lib lint`只从本地验证你的pod能否通过验证；`pod spec lint`从本地和远程验证你的pod能否通过验证。`pod lib lint`可省略，直接用`pod spec lint`进行验证



### 1.4.3 上传cocoapods

输入以下指令将代码上传到cocoapods：

```ruby
pod trunk push
```

如果之前验证的结果里有`warning`，则可以用`--allow-warnings`忽略警告：

```ruby
pod trunk push --allow-warnings
```



完成后出现`Tell your friends`字样则证明上传完毕，可以通过`pod search XXX`指令来验证是否上传成功。

如果没搜索出来则可以通过`pod repo update`更新本地的pod库，然后再进行搜索。

如果事先已经下载了旧版的库，可以直接cd到该工程，并`pod update 第三方库名`，哪怕暂时无法搜索出来，`pod update`依然可以有效将其更新到最新版本。

自己上传的第三方库，欢迎点个star：[MCircleBoard](https://github.com/MichaelLynx/MCircleBoard)



## 参考链接

[iOS开发之将自己的项目上传到Cocoapods](https://rakuyomo.github.io/2017/08/21/28-iOS%E5%BC%80%E5%8F%91%E4%B9%8B%E5%B0%86%E8%87%AA%E5%B7%B1%E7%9A%84%E9%A1%B9%E7%9B%AE%E4%B8%8A%E4%BC%A0%E5%88%B0CocoaPods/)

[将自己写的库上传到cocoapods（2015）](https://www.bbsmax.com/A/pRdBjmA1dn/)

