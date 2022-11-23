---
title: "UITextField输入长度限制时，中文输入法导致的输入问题"
layout: post
date: 2022-11-15 23:00
image: /assets/images/markdown.jpg
headerImage: false
tag:
- ios
- method
star: false
category: tech
author: Lynx
description: 输入框设置最大输入长度时，中文占位符导致最后一两个字无法输入的处理方法。
---



> 当我们有UITextField或者UITextView最大输入长度需求的时候，会面临一个问题，那就是如果我们使用系统输入法或者其他输入法的时候，在输入最后几个文字的时候，由于部分输入法会将拼音字母等高亮字符展示在输入框内，这时候输入长度就可能超过最大长度，从而导致最后几个文字输入失败，尽管这时候文字还没到最大输入长度。

# 一、原理
由于部分拼音输入法会把高亮字符展示在输入框内，所以我们在判定输入内容超过最大长度的时候需要做个判断。
**当输入框输入拼音，且高亮字符展示在输入框时，这些占位字符会被高亮选中，等用户选中候选词时，占位符替换成真正输入的字符。**
那么处理方法也就明确了：**监听输入框的输入情况，当输入长度超过最大输入长度时，判断如果存在高亮字符，则不做处理，等拼音输入完毕；如果不存在高亮字符，则进行正常处理。**



# 二、代码实现

## 1.输入框创建
输入框可以是UITextField，也可以是UITextView，两者监听处理方法是一致的。监听方法不一定要用这种方式，只要能监听输入框改变并传入UITextField实例即可。

```objective-c
UITextField *textField = [[UITextField alloc] init];
[textField addTarget:self action:@selector(textFieldChange:)
forControlEvents:UIControlEventEditingChanged];
[self.view addSubview:textField];
```



## 2.监听方法

此处我们需要判断输入框的高亮字符是否存在。
```objective-c
- (void)textFieldChange: (UITextField *)sender {
    NSString *content = sender.text;
    if (content.length > maxLength) {
        UITextRange *selectedRange = [sender markedTextRange];
		// 判断是否存在高亮字符
		UITextPosition *position = [sender positionFromPosition:selectedRange.start offset:0];
		if (!position) {
    		content = [content substringToIndex:8];
    		sender.text = content;
		}
    }
}
```



## 3.语言判断

如果有需要我们也可以对输入内容进行语言判断。
```objective-c
NSString *lang = sender.textInputMode.primaryLanguage;
if ([lang isEqualToString:@"zh-Hans"]) {
	// 判断是否是中文
}
```

