---
title: 关于如何将私人Pod库发布到pod库的心得
date: 2017-02-19 15:04:38
tags:
---
## 引子
周末闲来无事，琢磨了下如何将git上的代码做成一个开源的库，然后供自己和别人在开发中使用。捣鼓了一个多小时终于是成功了,大家可以在命令行下输入，pod search CYKit ，就会搜索到我这个小demo了。

<!--more-->

![](http://ock9zbzms.bkt.clouddn.com/CYKitPod@2x.png)

## 准备工作
* github准备开源的工程地址，并且有Release的Tag。如我自己的开源地址为：https://github.com/TripleFireMan/CYKit.git,Tag为0.1.
* 注册一个pod trunk 的账号，用来将自己的库push到cocoaPod的master spec中，这样才会能被别人搜索到。注册方式为 命令行输入 pod trunk register ab36474XXX@126.com 'your user name' --description='device name or something other'

## 正文

* <font color=red>在工程目录下创建Podspec文件，cd 到要做成开源库的项目根路径下，执行 pod spec create CYKit。该命令执行之后就会在工程目录下生成一个CYKit.podspec的文件</font>。
* 编辑Podspec文件，刚生成的Podspec文件很多信息都是没有的，需要手动去编辑，打开该文件会看到如下信息

```objc
Pod::Spec.new do |s|

  # ―――  Spec Metadata  ―――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  #
  #  These will help people to find your library, and whilst it
  #  can feel like a chore to fill in it's definitely to your advantage. The
  #  summary should be tweet-length, and the description more in depth.
  #

  s.name         = "CYKit"
  s.version      = "0.1"
  s.summary      = "something useful for daily development"
  s.homepage     = "https://github.com/TripleFireMan"
  # s.screenshots  = "www.example.com/screenshots_1.gif", "www.example.com/screenshots_2.gif"


  s.license      = "MIT"
  s.author       = { "chengyan" => "ab364743113@126.com" }
  s.platform     = :ios, "7.0"
  s.source       = { :git => "https://github.com/TripleFireMan/CYKit.git", :tag => "0.1" }
  
  s.source_files  = "CYKit", "CYKit/**/*.{h,m}"
  #s.resources     = "Resources/*.png"
  s.framework     = "UIKit"
  s.requires_arc  = true

  # s.xcconfig = { "HEADER_SEARCH_PATHS" => "$(SDKROOT)/usr/include/libxml2" }
  # s.dependency "JSONKit", "~> 1.4"

end
```



* <font color=red>**校验当前的podspec文件是否可用，在pod文件的目录下，执行此命令 pod spec lint CYKit.podspec**</font>,如果通过校验，则会有下面的提示
```
localhost:CYKit chengyan$ pod spec lint CYKit.podspec

 -> CYKit (0.1)

Analyzed 1 podspec.

CYKit.podspec passed validation
```
反之就会有错误提示。有错误就解决错误好了


* <font color=red>发布到cocoapod仓库，还是在podspec所在的文件下，执行pod trunk push 命令。</font>
  发布成功的话就有下面的提示了
```objc
localhost:CYKit chengyan$ pod trunk push

[!] Found podspec `CYKit.podspec`
Updating spec repo `master`

CocoaPods 1.2.0 is available.
To update use: `sudo gem install cocoapods`
Until we reach version 1.0 the features of CocoaPods can and will change.
We strongly recommend that you use the latest version at all times.

For more information, see https://blog.cocoapods.org and the CHANGELOG for this version at https://github.com/CocoaPods/CocoaPods/releases/tag/1.2.0

Validating podspec
 -> CYKit (0.1.5)

Updating spec repo `master`

CocoaPods 1.2.0 is available.
To update use: `sudo gem install cocoapods`
Until we reach version 1.0 the features of CocoaPods can and will change.
We strongly recommend that you use the latest version at all times.

For more information, see https://blog.cocoapods.org and the CHANGELOG for this version at https://github.com/CocoaPods/CocoaPods/releases/tag/1.2.0

  - Data URL: https://raw.githubusercontent.com/CocoaPods/Specs/279e29a1cb259157cb329f9bbc2470a167667ee3/Specs/8/a/e/CYKit/0.1.5/CYKit.podspec.json
  - Log messages:
    - February 19th, 04:07: Push for `CYKit 0.1.5' initiated.
    - February 19th, 04:07: Push for `CYKit 0.1.5' has been pushed (0.86529383 s).
    
```
## 小结
通过上述简单几步操作，我们就发布了一个开源的cocoapod版本了，以后有需要就可以往上面添加代码了，解决了重复造轮子的问题。

