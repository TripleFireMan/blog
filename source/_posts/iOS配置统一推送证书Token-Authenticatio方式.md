---
title: iOS配置统一推送证书Token Authenticatio方式
date: 2022-07-26 17:55:22
tags: iOS
---

## 背景

> 由于开发者账号下面配置的app越来越多，对于推送证书的配置有了新的要求，证书尽可能只需要配置一次就可以。
> 
<!-- more -->
---
## 方案
Apple提供了一种新的方案，在开发者网站上，可以创建keys，极光推送目前也已经对这种方式进行了支持，创建好的key，保存到本地会是一个p8类型的文件，创建流程如下123

<br/>

第一步: 点击key盘边的“+”号按钮

<br/>

<center>
<img src=iOS配置统一推送证书Token-Authenticatio方式/20220727150128.jpg width=2186 alt="9">
<br/>
<i>图一</i>
</center>
<br/>

第二步：然后勾选apple push notification service 

<br/>
<center>
<img src=iOS%E9%85%8D%E7%BD%AE%E7%BB%9F%E4%B8%80%E6%8E%A8%E9%80%81%E8%AF%81%E4%B9%A6Token-Authenticatio%E6%96%B9%E5%BC%8F/20220727150216.jpg width=300>
<br/><font color=white style="background-color:clear"><i>图二</i></font>
</center>

<br/>
第三步：输入要创建的keys名称，然后点击右侧的regiest按钮

<br/>

<center>
<img src=iOS%E9%85%8D%E7%BD%AE%E7%BB%9F%E4%B8%80%E6%8E%A8%E9%80%81%E8%AF%81%E4%B9%A6Token-Authenticatio%E6%96%B9%E5%BC%8F/20220727150327.jpg width=300>
<br/>

<i>图三</i>
</center>
<br/>
第四部：登入极光平台，然后选中刚才生成的P8文件，之后，挨个填入对应的信息

<br/>
<center>
<img src=iOS%E9%85%8D%E7%BD%AE%E7%BB%9F%E4%B8%80%E6%8E%A8%E9%80%81%E8%AF%81%E4%B9%A6Token-Authenticatio%E6%96%B9%E5%BC%8F/20220727154208.jpg width=300>
<br/>
<i>图四</i>
</center>

----
<br/>
备注

 * [极光推送网站](https://www.jiguang.cn/)
 * [apple配置地址](https://developer.apple.com/account/resources/authkeys/list)
