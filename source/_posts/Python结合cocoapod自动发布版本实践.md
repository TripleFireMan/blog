---
title: Python结合cocoapod自动发布版本实践
date: 2019-10-18 15:30:29
tags: python
---

### 一、问题

ios开发人员可能对cocoapod比较熟悉，在维护个人的cocoapod版本的时候，会遇到一个问题，那么就是本地代码库修改了文件，一般需要做如下三步操作

1. 代码功能开发完毕，提交代码到git服务器
2. 修改.podspec文件中的版本号
3. 给对应的提交打上tag，以便pod发布时需要
4. 使用pod trunk push 命令，发布到cocoapod

步骤比较多，修改版本和打tag，有时候容易遗漏，如果发布失败的话，还需要重新再修改版本，打tag。很浪费时间，基于此，结合我最近研究的python，写了一个脚本。来实现一键提交代码、发布pod库的功能

<!--more-->

### 二、脚本

```python
#encoding:utf-8
#!usr/bin/python
import os


f = open("CYKit.podspec", "r+")
print ("文件名为: ", f.name)
 
shouldModifire = '' 
content = ''
for line in f.readlines():                          #依次读取每行  
    content = content + line
    line = line.strip()                             #去掉每行头尾空白  
    if 's.version = "0' in line:
    	shouldModifire = line
    


#查找到需要修改的行
print('shouldModifire%s'%shouldModifire)
items = shouldModifire.split('=')
print(items)
versions = items[1].split('.')
versions_last_length = len(versions[2])
versions_last = int(versions[2][:versions_last_length-1])
versions_last_int = (versions_last) + 1
newVersion = str(versions[0])+'.'+ str(versions[1]) + '.' + str(versions_last_int) + '"'
shouldReplaceItem = items[0] + '=' + newVersion

#替换相应的版本号，并写入文件
replaceed = content.replace(shouldModifire,shouldReplaceItem)
print('修改版本号:' + shouldModifire + '=>' +  shouldReplaceItem)

#写入文件
f.seek(0)
f.truncate()	
f.write(replaceed)
f.flush()
#关闭文件
f.close()

#代码提交
os.system('git add .')
os.system('git commit -a -m "【脚本】修改相应版本号"')
os.system('git push origin master')
os.system('git tag -a ' + newVersion + ' -m ' + 'tag版本号')
os.system('git push --tags')
#提交到cocoapods
os.system('pod trunk push --allow-warnings')
```

### 三、遇到的问题

总体来说开发还算顺利，唯一卡壳的地方就是，读取本地文件，修改里面的版本号，这里使用了暴力的写文件方式。

好了，又可以开心的撸代码了，see u