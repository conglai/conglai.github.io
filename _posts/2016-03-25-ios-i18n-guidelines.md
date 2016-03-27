---
layout:     	post
title:      	"iOS Internationalization Guidelines"
date:       	2016-03-24 10:00:00
author:     	"无北"
header-img: 	"img/post/bg-alibaba-bus-stop.jpg"
header-mask: 	0.3
catalog: 		true
tags:
    - iOS
---

# iOS Internationalization Guidelines

## 文本

主要使用下述两个宏命令实现文本的国际化。
LocalizedStr用于固定文本的国际化，FormattedString用于需要拼接的文本的国际化。

``` 
#define LocalizedStr(key)   		NSLocalizedString(key, @"")
#define FormattedString(fmt,...) 	[NSString localizedStringWithFormat:LocalizedStr(fmt),##__VA_ARGS__]
```
例如：

*XXX.m*

```
UIButton* btn = [UIButton new];
[btn setTitle:LocalizedStr(@"common.genderPicker.btn.confirm")  forState:UIControlStateNormal];

UIAlertView* alertView = 
  [UIAlertView customAlertViewWithTitle:LocalizedStr(@"com.spaceAlert.title")
                                message:FormattedString(@"com.spaceAlert.prompt",quota,used,quota-used)
                      cancelButtonTitle:LocalizedStr(@"com.btn.cancel")
                                  click:^(NSString *clickBtnTitle) {
                                  ...
                                  }
                      otherButtonTitles:LocalizedStr(@"com.btn.ok"), nil];

```

*Localizable.strings*

```
com.btn.cancel 					= "取消";
com.btn.ok						= "OK";

...

com.spaceAlert.title 			= "存储空间";
com.spaceAlert.message 			= "您的云存储空间目前已使用 %2$.2f GB，剩余 %3$.2f GB。";

```

## 命名


strings文件内的文本标识符使用命名空间的规则进行命名。

> 模块名.组件名.类型(btn,prompt,title,lab...).状态(OR 备注)


## 图片

UI图片统一使用`` [UIImage imageNamed:imageName] ``方法获取图片。

withMe工程已混淆该方法，调用该方法在LocalizedStr(imageName)存在时等价于`` [UIImage imageNamed:LocalizedStr(imageName)] ``。

