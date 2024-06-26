---
title: Tree命令及简单使用
date: 2022-08-09 11:18:34
tags: 工具
category: 转
---

我用的是阿里云服务器，CentOS7，默认没有tree这个命令，需要安装，用下面的命令就可以安装：
```
sudo yum install tree
```
怎么样使用tree这个命令
其实有个非常简单的办法，就是直接查看关于tree的帮助，输入下面的命令，可以查看关于tree命令的帮助信息
<!-- more -->
```
tree --help
```
最简单的使用办法是直接输入tree命令，就会自动给我们以树形的形式列出当前目录的文件和文件夹，不加任何参数，它会自动列表当前目录下面所有深度级别的文件和目录。

几个比较常规的用法
1. 显示目录结构
```
tree
```

![](https://s2.51cto.com/images/blog/202201/04133345_61d3dc39695e056821.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

2. 包含隐藏文件
```
tree -a
```
![](https://s2.51cto.com/images/blog/202201/04133345_61d3dc39890237557.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

3. 控制深度（假设为2）
```
tree -L 2
```

![](https://s2.51cto.com/images/blog/202201/04133345_61d3dc39c5bd785555.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

更多的选项

| 选项 | 说明 |
| - | - |
|-a|显示所有文件，包含隐藏文件。|
|-d|只显示目录。|
|-l|跟踪符号链接，如果链接的是一个目录，则当成目录处理。|
|-f|显示完整路径。|
|-x|只显示本文件系统。|
|-L level| 控制显示的目录深度。|
|-R|在下级目录中，再次执行 tree 命令并且加上 '-o 00Tree.html’选项，配合-L,-H使用。|
|-P pattern|只显示匹配了 pattern 的文件（不是目录），支持简单的正则表达式。|
|-I pattern|与-P相反，只显示没有匹配 pattern的文件。|
|–ignore-case|当使用了-P或-I选项时，忽略大小写。|
|–matchdirs|当使用了-P选项时，文件名包含完整路径。|
|–prune|不显示空目录，如果经过-P或-I后没有目录下没有需要显示的，也当作空目录。|
|–noreport|不显示最后的统计信息。|
|–charset charset|指定字符集。|
|–filelimit #|过滤掉文件个数超过 # 的目录。|
|–timefmt fmt|按照指定的格式打印文件的修改时间。|
|-o filename |将结果输出到文件。|
|-q| 用问号代替不可打印的字符。|
|-N|用八进制代替不可打印的字符。|
|-Q|用引号将文件名括起来。|
|-p|显示文件的类型和权限。|
|-u|显示文件所属的用户名或者UID。|
|-g|显示文件所属的组或者GID。|
|-s|显示文件的大小，单位：字节|。
|-h|显示文件的大小，使用更人性化的显示。|
|–si|显示文件的大小，类似 -h 但是使用国际公制单位(1k=1000)。|
|–du|对于目录，显示其下所有文件的累计大小。|
|-D|显示文件的最后修改时间。|
|-F|类似 ls -F，对不同的文件类，在末尾加上不同的字符。|
|–inodes|显示文件的索引节点。|
|–device|显示文件所属的设备号。|
|-v|显示的文件列表按照version排序。|
|-t|显示的文件列表按照最后修改时间排序。|
|-c|显示的文件列表按照最后的状态改变时间排序。|
|-U|不进行排序处理。|
|-r|反向输出列表。|
|–dirsfirst|优先显示目录（同一级别）|
|–sort[=name]|指定排序方式，name(default), ctime, mtime, size, version.|
|-i|输出中不要进行缩进。|
|-A|使用ASCII的横线字符表示缩进。|
|-S|使用CP437的横线字符表示缩进。|
|-n|关闭颜色显示。|
|-C|打开颜色显示。|
|-X|使能XML格式输出。|
|-J|使能JSON格式输出。|
|-H baseHREF|使能HTML格式输出，并包含基本http链接地址。|
|-T title|在HTML格式输出中，设置标题和H1标签头|
|–nolinks|在HTML格式输出中，不输出超链接。|

















