---
title: 使用MacMini作为打包机+Jenkins进行自动化打包
date: 2024-09-13 16:51:17
tags:
---
# 使用MacMini作为打包机+Jenkins进行自动化打包

### 背景

>团队目前4个人，iOS（2）安卓（2），App打包需要占用研发的电脑，以iOS为例，打包一次司机端的ipa包，大概需要4-5分钟，打包一次货主端的ipa包，大概需要3分钟，基于这种情况，将之前申请的macMini，安装上jenkins的服务，配置上打包任务。这样每次打包就不用占用开发电脑，提高效率


### 最终效果

##### Jenkins配置主页面

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/1wvqr7erzkmeOako/img/e20bf444-d294-4f46-874f-b96ff3d39410.png)

##### 钉钉发送效果图

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/1wvqr7erzkmeOako/img/e041ae92-54e5-4e75-8749-04ae5c680c78.png)

### 思路

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/a/dejDyJAzgT9NMm4m/ae7fdaf54dcd4daea58d89a501051e2d3409.png)

### 踩坑

1.  jdk无法下载
    
    1.  jdk是属于oracle公司的产品，看了之前的文档让下载jdk11，但是jdk11没有找到，只找到了Java11，而且还是需要登录才能下载，于是去注册了oracle的账号，结果注册之后，仍然不能下载报404，最后解决办法是下载了图二所示的JDK22，安装后正常使用
![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/1wvqr7erzkmeOako/img/37c732ae-4931-4144-bf46-e7e9ce7a5094.png)
![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/1wvqr7erzkmeOako/img/3f691c8a-3e1e-4d88-8dc0-bd2b00c37011.png)
2.  局域网无法访问指定ip对用的jenkins
    
    1.  这个是由于jenkins开放ip默认是127.0.0.1.需要将开放ip更改为0.0.0.0，参考资料二解决此问题
        
3.  钉钉插件发送消息报错的问题
    

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/1wvqr7erzkmeOako/img/c7f272ca-7f3e-4567-a4db-be55934d7811.png)

该问题是由于钉钉插件，保存有问题，已经保存的消息未生效，一直发送的是空消息导致，解决办法，钉钉消息先点击应用，再点击保存

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/1wvqr7erzkmeOako/img/9262807c-4386-49d3-a0f2-1b38ef43e7d6.png)

4.  Xcode调参
    

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/1wvqr7erzkmeOako/img/9f7c98ef-6147-45db-870e-be4930075608.png)

我们的项目是使用workspace管理的工程，这里需要配置上工程名称

---

#### 引用资料

1.  [Mac上部署Jenkins和打包集成](https://www.jianshu.com/p/59c0cca2d234)
    
2.  [jenkins局域网无法访问](https://www.jianshu.com/p/20741df76cae)