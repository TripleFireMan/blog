---
title: Python学习笔记（三）
date: 2019-10-30 16:58:13
tags: python
---

今天来继续学习下pytho中的函数

### 学习目标

* 如何定义函数及带不同参数的函数

* 函数模块

* global、nonlocal、lambda

  <!--more-->

### 如何定义函数

1. 无入参的函数

   ```python
   def sayHello():
   	print(hello)	
   ```

2. 带返回值的函数

   ```python
   def sayHello():
   	hello = 'hello'
   	print(hello)
   	return hello
   ```

3. 带参数函数

   ```python
   def saySomething(something):
   	print(something)
   ```

4. 有默认值函数

   ```python
   def sayDefault(something='helloworld'):
   	print(something)
   ```

5. 可变参数* ,**

   ```python
   def sayMore(*morething):
   	for a in morething:
   		print(a)
   def sayMoreDic(**moreDic):
   	for dic in moreDic:
   		print(dic)
   ```

   *修饰的参数，其实是一个元组，**修饰的参数是一个字典，传入方式如下

   ```python
   sayMore(1,2,3)
   sayMoreDic(name='chengyan',age=10)
   ```

6. 传字典、列表和元组

   **传字典和列表进入函数，在函数内修改，也会同步修改外面的值**

   **传数字、字符串、元组进入函数，在函数内修改，并不会同步修改外面的值**

### 函数模块

1. **定义**

   在python里面，新建不同的.py文件，就相当于是创建了函数模块。

2. **引用**

   通过关键字import xx 来引用相同目录下的文件，这么引用到的函数如果想调用，需要这么写

   ```python
   import xx
   xx.sayHello()
   
   ```

3. **from xx import xxx**

   其中“xx”是模块名称，也就是文件名，"xxx"是我们要引用的函数名称，这么写有一个好处，是可以在调用函数的时候，略去"."例如

   ```python
   from xx import sayHello
   sayHello()
   ```

   还有一种写法是from xx import *,这种写法的话，会引入所有的函数

4. **模块搜索路径**

   ```python
   import sys
   sys.path[0] = '函数模块所在路径'
   #通过sys.path函数来指定模块搜索路径，用来解决代码文件不在一个文件夹下的问题
   ```

### global、nonlocal、lambda关键字

* global 关键字用来在局部作用域内，声明变量为全局变量，从而可以**修改**全局变量

* nonlocal 关键字用来修改闭包中的变量，将闭包变量修改为局部变量，从而可以修改闭包变量

* lambda 关键字是用来申明匿名函数的，范例如下

  ```python
  sum1 = lambda x,y: x+y
  print(sum1(2,7))
  ```

  

-----

今天主要是学习了下函数的声明、调用、引用以及几个关于函数的关键字，下一节学习基础部分的最后一节类