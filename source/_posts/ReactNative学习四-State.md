---
title: 'ReactNative学习四:State'
date: 2022-07-29 10:41:37
tags: ReactNative
---

# 如何让页面动起来？
如图是一个简单的购物车页面的UI，分别用来展示商品的名称、价格和数量。
<!-- more -->
![](https://static001.geekbang.org/resource/image/dd/9e/dd69765bb8fcb1f9dffyy2df4d2b789e.png?wh=1000x784)


## 选择状态
这个页面需要一个标示页面请求正常or失败的状态，这里使用枚举值
```JSX
const RequestStatus = {
    IDLE = 'IDLE',
    PENDING = 'PENDING',
    SUCCESS = 'SUCCESS',
    ERROR = 'ERROR'
}
```
此外还需要一个列表对象用来展示数据,采用
```jsx
const [products, setProduct] = useState([]);
```
即一个枚举和一个数组来展示页面。

## 状态更新
### 基础数据类型的状态更新
还是看下面的代码，以更新数量为例，首先注册了状态 "count",然后在组件的返回函数里面，通过按钮的点击事件，调用setCount(count+1)，来达到更新count值的效果。
```jsx
export default function count(){
    const [count ,setCount] = useState(0);
    return {
        <View>
        <Text>{count}</Text>
        <Button title="+" onPress={()=>setCount(count + 1)}></Button>
        <Button title="-" onPress={()=>setCount((count-1)>0? count-1 : 0)}>
        </View>
    };
}
```

### 引用数据类型的状态更新

引用数据类型的更新， 需要先析构原数据对象，然后再重新赋值，才可达到更新的效果，对象析构的写法
`...countObjects`

```jsx
setCountObject({...countObject, num: countObject.num+1});

const newCountArray = [...newCountArray]
newCountArray[0]++;
setCountArray(newCountArray)
```

## 加载图片
| | 使用方法| 底层scheme | 使用建议|使用场景
| - | - | - | - | - |
| 静态图片资源 | require('./dianxin') | file// | 跟随bundle压缩包下发 | 关键图片|
| 网络图片 | {uri:'https://cdn.dianxin.com/..,.png'} | http 或 https | 自研图片管理工具 | 大部分场景 |
| 宿主应用图片 | uri:'dainxin' | file://或 asset:// | 不建议 | 复用场景 |
| Base64图片 | 'uri':'data:image/jpeg,base64.....'| data:mime/type;base64| 自研图片管理工具 | 小图或者关键图|