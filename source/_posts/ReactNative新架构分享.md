---
title: ReactNative新架构分享
date: 2022-08-22 15:02:50
tags: RN
---

## RN的前世今生
**React Native是2013年在facebook的内部黑客马拉松中诞生的。到今年2022年新架构，已经整整10年。**
<br/>
### RN适用什么场景
* 业务更新迭代较快的团队与出海团队。
* 既要支持动态更新，又要支持复杂业务的场景。
<br/>

### 为什么要学习 React Native ？

1. React Native 是一个非常流行的跨端框架，开发者认可度很高

![](https://static001.geekbang.org/resource/image/d7/50/d75660fb448113ba4279962f88bc7b50.png?wh=1920x760)
<br/>
2. React Native 是一个跨领域的融合技术，它是你现有技术的自然延伸。
<br/>
3. 更为重要的是，今年RN将会发布新架构。新架构将带来以下三个方面的优势
   1. React Native 新架构的启动性能会有 2 倍左右的提升。
   <br/>
   2. React Native 新架构的通信性能会有 3 倍左右的提升。
   <br/>
   3. React Native 新架构的渲染流水有了很大的变化，这会带来更好的用户体验。


### RN的技术栈
下图是RN会涉及到的知识图谱

![](https://static001.geekbang.org/resource/image/93/17/9396e0ecf7d24b0a7eb84be5445f4017.jpg?wh=1920x1869)

| 序号| 内容|
| - | - |
| 1 | 开发语言、React 框架、开发必备工具这些预备知识 |
| 2 | React Native 本身的知识， 组件等 |
| 3 | 工作流中的实操知识 |

<br/>

### 如何搭建一个RN页面

RN是基于组件来构建应用的，所以构成页面的最小元素是即是组件.
#### 基本原则
**UI稿拆分原则:单一责任原则**
#### 搭建元素
<br/>
通过宿主组件来承载UI稿上面的元素，这些宿主组件包括文本、图片、按钮、列表等等呢个，这些组件都是直接由iOS/Andriod原生平台生成的

整体思路是，从上往下拆出组件，从下往上把拆出来的组件进行逐一实现和拼装

### RN的样式

* 通用样式
* view组件样式
* 其他样式

```jsx
// 文字颜色
<Text style={{color:'red'}}>
// 圆角边框 
<Text style={{borderColor:'green', borderWidth: 1, borderRadius: 5}}>  
```

如何布局？？
Flex：跨平台、高性能、易上手

内联样式和样式表，

stylessheet的三个好处

* 元素结构和样式分离，可维护性更好；
* 样式对象可以复用，能减少重复代码； 
* 样式对象只创建一次，也减少性能的损耗。

### 动态页面
RN的页面，用宿主组件搭建完成之后，还是一个静态的页面，如果要让页面展示不同的内容，还需要使用UseState()来进行页面的刷新,下面的代码是一个简单的控制数量的组件
```jsx
export default function Count() {
  const [count, setCount] = useState(0);

  return (
    <View>
      <Text>{count}</Text>
      <Button title="+" onPress={() => setCount(count + 1)} />
      <Button title="-" onPress={() => setCount(count - 1 >=0? count - 1: 0)} />
    </View>
)};
```
### 图片的加载

* 静态图片加载 

```objc
// 方案一：正确
const dianxinIcon = require('./dianxin.jpg')
<Image source={dianxinIcon}/>
```
* 网络图片加载

```jsx
// 建议
<Image source={{uri: 'https://reactjs.org/logo-og.png'}}
       style={{width: 400, height: 400}} />
```

* base64图片加载 

```jsx
<Image
  source={{
    uri: 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADMAAAAzCAYAAAA6oTAqAAAAEXRFWHRTb2Z0d2FyZQBwbmdjcnVzaEB1SfMAAABQSURBVGje7dSxCQBACARB+2/ab8BEeQNhFi6WSYzYLYudDQYGBgYGBgYGBgYGBgYGBgZmcvDqYGBgmhivGQYGBgYGBgYGBgYGBgYGBgbmQw+P/eMrC5UTVAAAAABJRU5ErkJggg=='
  }}
/>
```

* 宿主应用图片

```jsx
// Android drawable 文件目录
// iOS asset 文件目录
<Image source={{ uri: 'app_icon' }} />
```