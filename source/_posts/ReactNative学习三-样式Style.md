---
title: 'ReactNative学习三:样式Style'
date: 2022-07-29 08:57:38
tags: ReactNative
---
# 前言

组件只是页面的架子，如果不使用样式，就会平铺在页面上，没有什么体验可言。
<!-- more -->
> React Native的组件样式有哪些?
> React Native的Flex布局有哪些特点?
> React Natice的样式代码怎么管理?


# 一、组件样式
## 1.1 通用样式
大部分的React组件都有样式属性，style，例如改变文字
```JSX
// 文字颜色
<Text style={{color:'red'}}>
// 圆角边框
<Text style={{borderColor:'gray',borderWidth:1,borderRadius:5}}>
```
下图是常见的通用样式和组件样式的一些继承关系
![](https://static001.geekbang.org/resource/image/2d/9c/2d0dbe2764f676b3bac28330b7ba969c.jpg?wh=1920x1047)

## 1.2 私有样式
View的样式，可以认为是通用样式，如Layout、Transform、Shadow、backgroundColor、opacity。
像Text、Image、除了继承自View的通用样式以外，还有如font、color、lineHeight等等这些样式，都可以认为是私有样式

## 二、Flex弹性布局
* <font color=red style="background-color:yellow;">跨平台</font>
  * 第一层含义是 Flex 布局并不是 React Native 所独有的，在<font color=red> Web、Android、iOS 平台也都在用</font>
  * 跨平台的第二层含义是，React Native 的布局引擎 <font color=red>Yoga 是 Android、iOS 通用的</font>
* <font color=red style="background-color:yellow">高性能</font>
  * 使用 Yoga 实现的 FlexLayout 布局引擎比苹果官方提供了 UIStackViews 和 Auto layout 布局引擎，耗时减少了将近一个量级
  ![](https://static001.geekbang.org/resource/image/61/f7/612209db97553841a1d49bf207e7eef7.png?wh=1000x736)
* <font color=red style="background-color:yellow;">易上手</font>
  * 从上往下的布局样式，父容器view默认是```{display:"flex",flexDirection:'column'}```。示例如下：
    ```JSX
    <View>
    <View style={{height: 50, backgroundColor:   'powderblue'}} />
    <View style={{height: 50, backgroundColor: 'skyblue'}} />
    <View style={{height: 50, backgroundColor: 'steelblue'}} />
    </View>
    ```
  * 左图右文布局。
    ```jsx
    <View style={{flexDirection: 'row'}}>
      <Image
        style={{width: 100, height: 100}}
        source={{
        uri: 'https://placeimg.com/640/480/cats',
      }}
      />
      <Text style={{flex: 1,fontSize: 18}}>我是文字</Text>
    </View>
    ```
   * 文字居中布局
      ```jsx
      <View
          style={{
            alignItems: 'center',
            justifyContent: 'center',
            // 高度确定
            height: 60,
            borderWidth: 1,
          }}>
          <Text
            style={{
              fontSize: 18,
              // 文字默认内边距，会导致垂直居中偏下
              includeFontPadding: false,
              // 文字默认基于基线对齐，会导致垂直居中偏下
              textAlignVertical: 'center',
            }}>
          我是文字1
          </Text>
      </View>
      ```

# 三、StyleSheet
## 3.1 使用Stylesheet的好处。
  * **元素结构和样式分离、可维护性好**
  * **样式对象可以复用，减少重复代码**
  * **样式代码只创建一次，减少性能的损耗**
## 3.2 没有使用stylesheet的样式
```jsx
// 各种内联，导致 JSX 结构不清楚。
<View
      // 普通属性
      hitSlop={
      top: 10,
      bottom: 10,
      left: 0,
      right: 0
    }
      // 事件属性
      onLayout={() => {
      // 事件逻辑
      }}
      // 样式属性
    style={{
      alignItems: 'center',
      justifyContent: 'center',
      height: 60,
      borderWidth: 1,
    }}>
    <Text
      style={{
        fontSize: 18,
        includeFontPadding: false,
        textAlignVertical: 'center',
      }}>
    我是文字1
    </Text>
    <Text
      style={{
        fontSize: 18,
        includeFontPadding: false,
        textAlignVertical: 'center',
      }}>
    我是文字2
    </Text>
</View>
```
### 3.3 使用StyleSheet的样式代码
```jsx
// JSX 结构
<View
      hitSlop={hitSlop}
      onLayout={handleLayout}
    style={styles.container}>
    <Text style={styles.texts}>我是文字1</Text>
    <Text style={styles.texts}>我是文字2</Text>
</View>

// 样式表
const styles = StyleSheet.create({
  container: {
    alignItems: 'center',
    justifyContent: 'center',
    height: 60,
    borderWidth: 1,
  },
  texts: {
    fontSize: 18,
    includeFontPadding: false,
    textAlignVertical: 'center',
  }
});

```
