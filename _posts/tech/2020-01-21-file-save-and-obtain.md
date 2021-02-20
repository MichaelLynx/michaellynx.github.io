---
title: "文件存取方法记录"
layout: post
date: 2020-01-21 21:19
image: /assets/images/markdown.jpg
headerImage: false
tag:
- ios
- method
star: false
category: tech
author: Lynx
description: iOS的数据存取方法(Bundle、沙盒、App间分享)。最新更改时间：2021/02/20
---



# 1.Bundle文件的读取

> Bundle文件直接存储于工程目录下



## 1.1 Bundle文件直接存储在工程项目文件夹内

```swift
//获取直接存放如工程的名为zz.png的图片
let path = Bundle.main.path(forResource: "zz", ofType:"png")
let bImage = UIImage(contentsOfFile: path!)
```



## 1.2 文件存放在工程目录的.Bundle文件内

```swift
let bundlePath = Bundle.main.path(forResource: "MCircleBoard", ofType: "bundle") ?? ""
let bundle = Bundle(path: bundlePath)
let filePath = bundle?.path(forResource: imageName, ofType: "png") ?? ""
let image = UIImage(contentsOfFile: filePath) ?? UIImage()
```



**bundle内图片通用获取方式，无论本工程还是第三方库内的图片都可获取**

```swift
var bundle = Bundle(for: theClass.self)
if let resoucePath = bundle.path(forResource: "bundleName", ofType: "bundle"), let resouceBundle = Bundle(path: resoucePath) {
    bundle = resouceBundle
}
let image = UIImage(named: imageName, in: bundle, compatibleWith: nil) ?? UIImage()
```

- `theClass`：工程项目名称；`bundleName`：对应的bundle的名字



## 1.3 获取工程目录内的文件列表

获取目录下的路径及其文件名

```swift
//文件主路径
let bundlePath = Bundle.main.resourcePath!
let fileManager = FileManager.default
let dirArray = try?fileManager.contentsOfDirectory(atPath: bundlePath)

let predicate = NSPredicate(format: "pathExtension == %@", "bin")
//文件名列表
var fileNameArray:[String] = (dirArray! as NSArray).filtered(using: predicate) as! [String]
```

- 通过predicate筛选出需要的文件，如果不需要筛选，直接使用`dirArray`就行。
- 工程目录下的文件与我们直接在xcode上看到的文件名不一样。`.m`、`.h`、`.swift`、`.storyboard`等文件不会显示出来，`.plist`、`.dylib`等文件会照常显示，`.xib`文件则会显示为`.nib`。

<br>

# 2.沙盒文件的写入与获取

- 沙盒即程序自己的目录，即文件的本地化存储
- `swift`暂未开放自己读写文件的操作，要实现沙盒文件的读取与写入必须将文件转换为`oc`的类
- 文件写入时可以直接加后缀也可以不加，不影响取出后的使用（取出时会转换成可用的样式）

## 2.1  获取沙盒文件目录下的文件名列表

```swift
let path = NSHomeDirectory().appending("/Documents/")
let dirArray: Array = try!fileManager.contentsOfDirectory(atPath: path)

let predicate = NSPredicate(format: "pathExtension == %@", "bin")
let nameArray: [String] = (dirArray as NSArray).filtered(using: predicate) as! [String]
```

- 通过predicate筛选出需要的文件，如果不需要筛选，直接使用`dirArray`就行。
- 得到沙盒文件的文件名后可以直接将其拼接到沙盒的路径下并将其取出



## 2.2 移除文件

```swift
let pathHead:String = NSHomeDirectory().appending("/Documents/")

let arrayName = "theArray"
let path = NSHomeDirectory().appending("/Documents/").appending(arrayName)

//移除文件
try? FileManager.default.removeItem(atPath: path)

//移除所有文件
func deleteAllFile() {
    let enumerator = FileManager.default.enumerator(atPath: pathHead)
    guard enumerator != nil else {return}
    
    for fileName in enumerator!.allObjects as! [String] {
        deleteFile(fileName: fileName)
    }
}
```



## 2.3 图片的存取

保存图片到沙盒

```swift
let image = UIImage(named: "fox")
//给保存在沙盒的文件命名
let imageName = "theFox"

if let imageData =
image.jpegData(compressionQuality: percent) {
    let path = NSHomeDirectory().appending("/Documents/").appending(imageName)
    
    try? imageData.write(to: URL(fileURLWithPath: path), options: Data.WritingOptions.atomic)
    
    print("Image path:\(path)")
}

```

从沙盒中取出图片

```swift
//沙盒中文件的名字
let imageName = "theFox"
let path = NSHomeDirectory().appending("/Documents/").appending(imageName)

if let image = UIImage(contentsOfFile: path) {
  	print("获取图片成功")
    //return image
} else {
    print("获取图片失败")
    //return nil
}

```



## 2.4 字典的存取

保存字典到沙盒

```swift
let dic = Dictionary()
let dicName = "theDic"
let path = NSHomeDirectory().appending("/Documents/").appending(dicName)

let nDic = NSDictionary(dictionary: dic)
nDic.write(to: URL(fileURLWithPath: path),atomically: true)
print("Dictionary path:\(path)")

```

从沙盒中取出字典

```swift
let dicName = "theDic"
let path = NSHomeDirectory().appending("/Documents/").appending(dicName)

if let dic = NSDictionary(contentsOfFile: path) {
  	print("获取字典成功")
    //return dic as? Dictionary
} else {
    print("获取字典失败")
    //return nil
}

```



## 2.5 数组的存取

保存数组到沙盒

```swift
let dic = Dictionary()
let dicName = "theDic"
let path = NSHomeDirectory().appending("/Documents/").appending(dicName)

nArray.write(to: URL(fileURLWithPath: path), atomically: true)
print("Array path:\(path)")

```

从沙盒中取出数组

```swift
let arrayName = "theArray"
let path = NSHomeDirectory().appending("/Documents/").appending(arrayName)

if let array = NSArray(contentsOfFile: path) {
  	print("获取数组成功")
    //return array as Array
} else {
    print("获取数组失败")
    //return nil
}

```

<br>



# 3.App间分享文件

- iOS的每个App之间相互独立，如同一个个沙盒，所以无法直接相互分享文件
- App间分享文件必须利用iOS本身的文件共享功能



## 3.1 接收分享文件

- 需要先再`info.plist`添加新属性`CFBundleDocumentTypes`(实际显示为`Document type`)
- `Document type`是一个数组类型的属性，可以在里面同时注册多个类型，[详细属性列表](https://developer.apple.com/library/archive/documentation/Miscellaneous/Reference/UTIRef/Articles/System-DeclaredUniformTypeIdentifiers.html)如下。
- 设置完成后需要在`AppDelegate`文件添加`func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool`方法，并导入对应代码处理接收到的文件



### 3.1.1 CFBundleDocumentTypes的常用属性

可以直接在plist里添加属性，也可以

- CFBundleTypeName

字符串类型，指定某种类型的别名，也就是用来指代我们规定的类型的别称，一般为了保持唯一性，我们使用UTI来标识。

- CFBundleTypeIconFiles

数组类型，包含指定的png图标的文件名，指定代表某种类型的图标，而图标有具体的尺寸标识：`iPad`、`iPhone and iPod touch`

- LSItemContentTypes

数组类型，包含UTI字符串，指定我们的应用程序所有可以识别的文件类型集合 

- LSHandlerRank 

字符串类型，包含`Owner`、`Default`、`Alternate`、`None`四个可选值，指定对于某种类型的优先权级别，而Launcher Service会根据这个优先级别来排列显示的App的顺序。优先级别从高到低依次是`Owner`、`Alternate`、`Default`。`None`表示不接受这种类型。


可以直接存放进plist，也可以以`Source code`的方式编辑保存：

```
<key>CFBundleDocumentTypes</key>
<array>
    <dict>
        <key>CFBundleTypeName</key>
        <string>PDF</string>
        <key>LSHandlerRank</key>
        <string>Owner</string>
        <key>LSItemContentTypes</key>
        <array>
            <string>com.adobe.pdf</string>
        </array>
    </dict>
    <dict>
        <key>CFBundleTypeName</key>
        <string>Image</string>
        <key>LSHandlerRank</key>
        <string>Alternate</string>
        <key>LSItemContentTypes</key>
        <array>
            <string>public.image</string>
        </array>
    </dict>
</array>

```



### 3.1.2 接收并处理共享文件

- 需要在`AppDelegate`的`func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool`方法里添加代码，对接收到的文件进行处理保存
- iOS9之前的回调是：`func application(_ application: UIApplication, open url: URL, sourceApplication: String?, annotation: Any) -> Bool`，如有需要则需要根据版本分别做处理

当前保存在`Documents`文件夹，需注意的是添加到`Documents`文件夹的文件会被存一份到`Inbox`文件夹，而如果存入的文件含有中文，则该文件只会出现在`Inbox`文件夹。

```swift
func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
    
    var path = url.absoluteString
    if path.contains("file:///private") {
        path = path.replacingOccurrences(of: "file:///private", with: "")
    }
    let tmpArray = path.components(separatedBy: "/")
    guard tmpArray.count > 0 else {
        return true
    }
    let fileName = tmpArray.last
    let filePath = NSHomeDirectory().appending("/Documents/").appending(fileName!)
    
    try? FileManager.default.copyItem(atPath: path, toPath: filePath)
    if FileManager.default.fileExists(atPath: filePath) {
        print("The file has been shared successfully.")
    } else {
        print("The file has not been added to the directory.")
    }
    
    return true
}

```

<br>

