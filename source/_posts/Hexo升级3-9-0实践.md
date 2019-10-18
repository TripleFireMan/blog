---
title: Hexo升级3.9.0实践
date: 2019-10-17 18:15:44
tags: Hexo
---

### 一、诉求

hexo使用了好几年了，使用的版本一直是3.2.0, node 版本也是4.5.9版本，当前很多的node_models要去的版本都是6.0以上了，所以今天对hexo和node进行一次升级，全部都升级到最新版本。

`hexo->3.9.0`

`node->10.16.3`

具体带来的好处是页面打开速度加快、使用最新的插件，升级完成之后，可以支持文章字数统计和阅读时间统计。此外，还有一点就是以前一直对写的文章的代码区，不是很满意。此次也一并解决。

<!--more-->

### 二、升级方案

* [node升级方案](https://www.jianshu.com/p/504e8227b041)

* [hexo升级方案](https://blog.csdn.net/whjkm/article/details/81088518)

* [hexo文字字数与hexo阅读时间改造](https://blog.csdn.net/mqdxiaoxiao/article/details/93670772)

  > 这里要多说几句，由于有两个方案实现这个功能，一种是通过`npm install hexo-wordcount --save`这种方式，[误导文章](https://blog.csdn.net/ganzhilin520/article/details/79048036)，浪费了很多时间在这里。而正是的方式应该是 `$ npm install hexo-symbols-count-time --save`

最终总算是找到正确的姿势，文章也实现了字数与阅读时间的统计

### 三、意外产物

在升级过程中，意外搞出来一些其他小的东西，如**代码支持暗黑模式**（这个功能以前也有，没有找到地方进行设置）、**尾部贴了一个微信的二维码**（希望能跟网友更多的交流）、另外还意外发现一个**赞赏**的功能，也打开了（如果有网友愿意的话~！~）。

### 四、还没实现，可以继续优化的点

目前我使用的Hexo，Next主题总得来说，已经比较满意了，但是美中不足的是，多说下线之后，没有找到合适的替代方案，所以评论这块还是一个待优化的点。

### 五、其他一些有用的链接

* [Hexo主题网站](https://hexo.io/themes/)

  