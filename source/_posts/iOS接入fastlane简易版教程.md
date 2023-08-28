---
title: iOS接入fastlane简易版教程
date: 2022-07-28 08:41:45
tags: iOS
---
## fastlane 是什么？
一套使用Ruby写的自动化工具集，旨在简化Android和iOS的部署过程，自动化你的工作流。它可以简化一些乏味、单调、重复的工作，像截图、代码签名以及发布App
<!-- more -->
## fastlane 可以干什么
|插件|功能|
| - | - |
|gym|是fastlane提供的打包工具。|
|snapshot| 生成多个设备的截图文件|
|frameit |对截图加一层物理边框|
|increment_build_number|自增build number 然后与之对应的get_build_number。Version number同理。
|cert：|创建一个新的代码签名证书
|sigh：|生成一个provisioning profile并保存打当前文件
|pem：|确保当前的推送证书是活跃的，如果没有会帮你生成一个新的
|match：|在团队中同步证书和描述文件。(这是一种全新的管理证书的方式)
|testflight：|上传ipa到testflight
|deliver：|上传ipa到AppStore

## fastlane 安装指南
gem 安装fastlane
```ruby
sudo gem install fastlane
```

## 新项目安装fastlane
```ruby
fastlane init
```

## fastlane打包
  ### 打包需要条件
  |条件|名称| 是否必须
  | - | - | - |
  | 工程名|DailyClock|是|
  | 包名 | 打卡 | 是|
  | 打包环境 | Debug | 是|
  | teamID | M99LZTCK8N | 是 |
  | provisioningProfiles | {"com.chengyan.DailyClock" => "DailyClockAppStore"} | 是
  |<font color=red style="background-color:yellow;">以下是上传蒲公英需要的参数</font>
  | 蒲公英的appid|****|否
  |蒲公英的userkey|******|否
  |<font color=red style="background-color:yellow;">以下是上传appstore需要准备的参数</font>|
  |用户名|xxxxxx@xx.com|是|
  |密码|xxxxx|是|
  |FASTLANE_APPLE_APPLICATION_SPECIFIC_PASSWORD|xxxx|是|
  |FASTLANE_SESSION|xxx|是|
  
<br/>

### 可能遇到的问题

> 1. 怎么获取 FASTLANE_APPLE_APPLICATION_SPECIFIC_PASSWORD？<br/>
> 访问[苹果账号管理](https://www.appleid.apple.com)添加app专用的密码
> 
> 2. 怎么获取FASTLANE_SESSION？<br/>
> 命令行执行如下命令：fastlane spaceauth -u yourappleid
> 3. command timed out after 10 seconds on try 1 of 4
> 原因： 很大可能是机器不给力了，使xcodebuild -showBuildSettings ?
> -workspace ./XX.xcworkspace -scheme XX -configuration Release 
> 命令执行超时
> 解决方法: 在Fastfile的 before_all方法中，添加 :
>```ruby
>    ENV["FASTLANE_XCODEBUILD_SETTINGS_TIMEOUT"] = "120"
>    ENV["FASTLANE_XCODEBUILD_SETTINGS_RETRIES"] = "4"
>```



### 一些范例
  ---
  **范例一、打包app并上传到蒲公英平台**
```ruby
  desc "Description of what the lane does"
  lane :dk do
	ipa_dir = "fastlane_build/"
	ipa_name = "打卡"+Time.new.strftime('%Y-%m-%d_%H:%M')
	archive_path = "./build/ipa/tem/archive/DailyClock.xcarchive"
	
	increment_build_number(xcodeproj:"DailyClock.xcodeproj")
	build_app(workspace:"DailyClock.xcworkspace",
	scheme:"DailyClock",
	configuration:"Debug",
	clean: true,
	output_directory:ipa_dir,
	archive_path:archive_path,
	export_options:{
	method:"development",
      	destination: "export",
      	stripSwiftSymbols: true,
      	teamID: "M99LZTCK8N",
      	uploadBitcode: false,
      	uploadSymbols: true,
      	provisioningProfiles: {"com.chengyan.DailyClock" => "DailyClockDev"}
	}
	)
        # 发布到蒲公英平台
    pgyer(api_key:"替换为你的蒲公英平台的appkey",user_key:"替换为你自己的userkey")

  end
```
**范例二、打包app上传到appstore**
```ruby
desc "app store"
	lane:release do
	ENV["FASTLANE_USER"] = "your appleid"
	ENV["FASTLANE_PASSWORD"] = "password"
	ENV["FASTLANE_APPLE_APPLICATION_SPECIFIC_PASSWORD"] = "yourpasssword"
	ENV["FASTLANE_SESSION"] = '''yoursession'''
	ipa_dir = "fastlane_build/"
	ipa_name = "测一测"+Time.new.strftime('%Y-%m-%d_%H:%M')
	archive_path = "./build/ipa/tem/archive/xxx.xcarchive"
	
	increment_build_number(xcodeproj:"xxx.xcodeproj")
	build_app(workspace:"xxx.xcworkspace",
	scheme:"xxx",
	configuration:"Release",
	clean: true,
	output_directory:ipa_dir,
	archive_path:archive_path,
	export_options:{
	method:"app-store",
      	destination: "export",
      	stripSwiftSymbols: true,
      	teamID: "xxx",
      	uploadBitcode: false,
      	uploadSymbols: true,
      	provisioningProfiles: {"xxxx"}
	}
	)
	upload_to_testflight(
	ipa: ipa_dir+'/grh.ipa',
	skip_waiting_for_build_processing: true
	)

end
```

### 成功截图
![](iOS%E6%8E%A5%E5%85%A5fastlane%E7%AE%80%E6%98%93%E7%89%88%E6%95%99%E7%A8%8B/20220728154127.jpg)


### 命令执行
```ruby
fastlane release # 打包并发布appstore
fastlane dk #打包并发布到蒲公英
```

### 更多
结合alfred效率神器，打包将更香。最终效果
![](iOS%E6%8E%A5%E5%85%A5fastlane%E7%AE%80%E6%98%93%E7%89%88%E6%95%99%E7%A8%8B/20220728155036.jpg)
![
](iOS%E6%8E%A5%E5%85%A5fastlane%E7%AE%80%E6%98%93%E7%89%88%E6%95%99%E7%A8%8B/20220728155448.jpg)
真香！！！
