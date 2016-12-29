---
title: 如何使用subline-text2-作为markdown的编辑工具
date: 2016-12-29 11:37:48
tags: 点点滴滴
---

>今年8月份的时候，在[二代](http://kaisayoung.github.io)的带领下，接触了Hexo+Github搭建
>个人的博客空间的新姿势，在此基础上。我个人选择了sublinetext2 作为markdown的编辑工具，实测还可以，采用subline能够做到
>
+ *在线预览*  (通过安装[OmniMarkupPreviewer](https://github.com/timonwong/OmniMarkupPreviewer)实现)
+ *语法高亮*  (通过安装[MarkdownEditing](https://github.com/SublimeText-Markdown/MarkdownEditing)实现)
<!--more-->
单纯的subline虽然也可以打开markdown文件，也可以编辑，但是效果和体验总是不太好，我今天介绍的这俩个插件能够大大提高书写效率和提升书写体验。先附俩张图体验一下最终的效果。

![在线预览](http://ock9zbzms.bkt.clouddn.com/1D8AC7FE-531C-4E47-B744-F1DBDBA8C307.png)
![图片高亮](http://ock9zbzms.bkt.clouddn.com/subline_online_view1B9EB750-1049-409B-BEBA-385DA4594819.png)
基本可以实现，在写的过程中就可以看到最终显示在网页上是什么效果。不用等写完之后再去调整展示的界面，还是挺方便的。

这里有几个点需要注意一下。

1. 将OmniMarkupPreviewer，MarkdownEditing从github上clone到subline的packages下需要重启下subline。
2. 如果是代码的话，高亮需要使\```将代码包起来  ，形如\```objc 这里添写需要写的代码\```
3. 在线预览功能，可以通过在subline右键点击preview markdown on browser，也可以通过快捷键command + Alt + O 在线预览。
---
好久没有写东西了，写了这么点东西居然写了改，改了写，以后还是要常写常更，争取回到每周一更的正确轨道上来。
