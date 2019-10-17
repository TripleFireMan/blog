---
title: Python学习笔记（一）
date: 2019-10-16 21:10:40
tags: python
---

今天开一个专题，来记录下学习python的一些笔记

### 1、 打印输出

```python
#encoding:utf-8
print('hello world')
print('你好，世界')
```

<!--more-->

### 2、变量声明

```python
#声明一个字符串
monday='星期一'
#声明一个int类型的数字
one=1
#声明一个浮点类型的数字
pi=3.14
#声明一个数组
arr=['1','2',2222]
#声明一个字段
dic={'name':'成焱','age':22}
#声明一个bool类型的
boolvalue=False
```

### 3、常用字符串操作函数

* **求字符串长度**

```python
stringlength=len(monday)
```

* **字符串拼接**

```python
string1='你好'
string2='成焱'
string3=string1+string2
```

* **格式化字符串输出**

```python
Python转换说明符
转换说明符	说明
%d，%i	转换为带符号的十进制形式的整数
%o	转换为带符号的八进制形式的整数
%x，%X	转换为带符号的十六进制形式的整数
%e	转化为科学计数法表示的浮点数（e 小写）
%E	转化为科学计数法表示的浮点数（E 大写）
%f，%F	转化为十进制形式的浮点数
%g	智能选择使用 %f 或 %e 格式
%G	智能选择使用 %F 或 %E 格式
%c	格式化字符及其 ASCII 码
%r	使用 repr() 将变量或表达式转换为字符串
%s	使用 str() 将变量或表达式转换为字符串

如
print(monday+'%d'%one)
输出：星期一1
```

### 4、数据类型转换

1. int(x) ： x为数字或字符串型的数字
2. float(x)：x为数字为字符串型的数字
3. complex(x，y)：转换为复数，x，y为数字，浮点数，bool数
4. str(x)：转为字符串
5. bin(x)：转为二进制数
6. oct(x)：转为8进制数
7. hex(x)：转为16进制数
8. chr(x)：将10进制数转为ASCII码
9. ord(x)：将ASCII字符转为10进制数

### 5、条件控制语句

```python
isBoy=True

#单层判断
if isBoy:
  print('男孩')

#双重判断
if isBoy:
	print('男孩')
else:
	print('女孩')
  
#多重判断
if 1:
	print(2)
elif 2:
	print(32)
else:
	print(3)
```

### 6、循环控制语句

* **for循环** 

```python
for i in range(10):
	print(i)

#输出
0
1
2
3
4
5
6
7
8
9
```

* **带步长循环打印**

```python
for i in range(0,10, 2):
	print(i)
#输出
0
2
4
6
8
```

* **循环一个数组**

```python
for var in arr:
	print(var)
#输出
1
2
2222
```

* **while 循环**

```python

i = 0
while i < 3:
	print(i)
	i+=1
#输出  
0
1
2
```

### 7、成员运算符 in，not in

```python
if 2 in range(10):
  print('2在集合中')
else:
  print('2不在集合中')
  
animals=['猫','狗','猪']
if '桌子' not in animals:
  print('桌子不属于动物')
else:
  print('桌子属于动物')
```

### 8、身份运算符 is ，is not

```python
i = 1
j = 1

if i is j:
	print('i 就是 j')
else:
	print('i 不是 j')

#输出：
i 就是 j

name1='成焱'
nickname='chengyan'
if name1 is not nickname:
	print('成焱不是chengyan')
else:
	print('成焱就是chengyan')
  
#输出
成焱不是chengyan
```



### 小结：

本篇就先学习到这里，回顾下，本篇主要学习了python如何打印输出、字符串的常见操作、变量的声明、条件控制语句、数据类型转换、格式化输出字符串、身份变量符和成员变量符，下一节学习列表与元组、字典，see u tomorrow！