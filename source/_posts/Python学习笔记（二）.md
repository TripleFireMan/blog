---
title: Python学习笔记（二）
date: 2019-10-17 18:10:32
tags: python
---

今天来学习下Python中的数组、元组和字典

### 一、数组

* **数组的声明**，python的数组可以存放任何类型的数据，数组的声明如下

```python
#数组的声明
list = [1, 2, 'chengyan',['chengyan',1]]
```

* **数组元素的获取**，支持根据下标进行获取

```python
#根据下标获取
print(list[0])
#将会打印1
```

<!--more-->

* 列表常用的方法**

| 方法名称 | 方法功能描述                               |
| -------- | ------------------------------------------ |
| append   | 在列表尾部拼接元素                         |
| clear    | 列表清空（python3.3之后支持）              |
| copy     | 复制生成另一个列表（3.3后支持）            |
| count    | 统计指定元素个数                           |
| extend   | 合并两个列表                               |
| index    | 获取指定元素的下标                         |
| insert   | 在指定位置插入元素                         |
| pop      | 删除指定下标元素                           |
| remove   | 删除某个元素，元素需要在列表中，否则会报错 |
| reverse  | 反转列表                                   |
| sort     | 对列表进行排序                             |

### 二、元组

元组与列表的区别在于：1. 元组不能对其元素进行变动，而列表可以。2. 元组用（）括起来，而列表用[]括起来

* 元组的声明

  * 简单版

  ```python
  #声明一个空元组
  tuple=()
  #简单声明一个元组
  tuple1=('name','age')
  ```

  * 进阶版

  ```python
  #省略（）的元组
  name,age='chengyan',18
  person=(name,age)
  print(person)
  ```

* 元组的基本操作

| 方法名称 | 方法功能描述       |
| -------- | ------------------ |
| count    | 统计指定元素的个数 |
| index    | 返回指定元素的下标 |

| 函数名称 | 函数功能描述           |
| -------- | ---------------------- |
| len      | 统计元组元素个数       |
| max      | 返回元组中最大值的元素 |
| min      | 返回元组中最小值的元素 |
| tuple    | 将列表转换为元组       |
| type     | 返回对象类型           |
| del      | 删除整个元组对象       |
| sum      | 对元组所有元素求和     |

### 三、字典

* **创建字典**

```python
dic={'name':'chengyan','age':18}
```

* 字典的常用方法

| 方法名称   | 方法功能描述                                             |
| ---------- | -------------------------------------------------------- |
| clear      | 字典清空                                                 |
| copy       | 复制生成另一个字典                                       |
| fromkeys   | 使用给定的key建立新的字典，值默认为None                  |
| get        | 返回key对应的值                                          |
| items      | 以元组数组的方式返回字典中的元素                         |
| keys       | 以列表的形式返回字典中的keys                             |
| pop        | 删除指定键的值，并返回该值                               |
| popitem    | 随机返回元素，并删除元素，以元组的形式                   |
| setdefault | 当字典中的键不存在的时候，设置值，如存在时，返回对应的值 |
| update     | 利用一个字典去更新另外一个字典                           |
| values     | 返回字典中的值                                           |

* **一些示例**

```python
dic={'name':'chengyan'}

#新增元素
dic['age']=18
#如果没有，设置默认值，有，不做操作
dic.setdefault('name','wangqi')
#查询
name = dic['name']
name = dic.get('name')
#修改
dic['name']='wangqi'
dic.update('name','wangqi')
#删除
dic.pop('name')
del(dic['name'])
#随机删除一个并返回该元组
dic.popitem()
#遍历
for tuple in dic:
  print(tuple)
for key in dic:
	print(key)
for key in dic:
  print(dic[key])
#fromkeys方式创建字典
dic1={}.fromkeys['name','age','class','school']
  

```

### 四、小结

这一节，基本了解了元组、列表、字典的创建、以及一些常用的方法。下节学习下函数，see u