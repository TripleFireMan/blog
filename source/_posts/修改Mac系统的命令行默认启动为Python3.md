---
title: 修改Mac系统的命令行默认启动为Python3
date: 2019-10-30 10:52:54
tags:
---

### 问题

mac系统的默认python环境为2.7.10，当前python的主流库都升级到3.7版本了。因此需要对python进行升级.升级方法这里不在介绍，[参考这里](https://blog.csdn.net/huhaoxuan2010/article/details/80507688)，

当将本机的python升级到3.7之后，在mac自带终端，输入`python -V`之后会发现版本仍然是之前的2.7.10

<!--more-->

### 解决办法



1. 打开命令行，执行如下命令

   ```bash
   ➜  ~ open ~/.bash_profile
   ```

2. 在该文件的最下面，添加如下命令

   ```
   export PATH=${PATH}:/Library/Frameworks/Python.framework/Versions/3.7/bin
   alias python="/Library/Frameworks/Python.framework/Versions/3.7/bin/python3.7" 
   ```

3. 最后执行

   ```
   ➜  ~ source ~/.bash_profile
   ➜  ~ python -V
   Python 3.7.4
   ```

4. 但是这么解决还有一个问题，当重启命令行之后，又会变为2.7.10版本。接下来需要继续执行

   ```
   open /etc/bashrc 
   ```

5. 在该文件的最后面添加如下命令`source ~/.bash_profile`

   ```
   # System-wide .bashrc file for interactive bash(1) shells.
   if [ -z "$PS1" ]; then
      return
   fi
   
   PS1='\h:\W \u\$ '
   # Make bash check its window size after a process completes
   shopt -s checkwinsize
   
   [ -r "/etc/bashrc_$TERM_PROGRAM" ] && . "/etc/bashrc_$TERM_PROGRAM"
   
   source ~/.bash_profile
   ```

6. 大部分时候，问题到这就解决了。但是如果使用了iTerm2,那么还需要打开

   ```
   open /etc/zshrc
   ```

7. 在该文件的最后面添加`source ~/.bash_profile`

   ```
   # Correctly display UTF-8 with combining characters.
   if [ "$TERM_PROGRAM" = "Apple_Terminal" ]; then
   	setopt combiningchars
   fi
   
   disable log
   
   [ -r "/etc/zshrc_$TERM_PROGRAM" ] && . "/etc/zshrc_$TERM_PROGRAM"
   source ~/.bash_profile
   ```

问题解决!