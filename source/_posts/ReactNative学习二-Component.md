---
title: ReactNative学习二-Component
date: 2022-07-27 16:43:09
tags: ReactNative
---
# 搭建静态页面的原则
## 单一责任原则
    每个组件都应该只有一个单一的功能，并且这个组件和其他组件没有相互依赖
<!-- more -->
### 分析一段代码来学习一个组件的基本构成
![](https://static001.geekbang.org/resource/image/e2/94/e22a8ff50c7bbdb637ed6eb42892dd94.png?wh=1000x594)

```jsx
export default function Product({product = {name: '苹果', price: '1元'} }) {
  return (
    <View style={{flexDirection: 'row', marginTop: 5}}>
      <Text style={{flex: 1}}>{product.name}</Text>
      <Text style={{width: 50}}>{product.price}</Text>
    </View>
  );
}
```

1. 第一步，导出组件。还记得单一责任原则吗？一个组件的责任要单一，一个文件的责任也要单一。因此通常一个文件中只有一个组件，用export default就可以将它导出，让其他文件import引入使用。
2. 第二步，定义函数。组件是一种特殊的函数。组件名字的首字母一定是大写的，示例中的Product是组件，因此它的 P是大写的（当然，还有类组件，但用得会越来越少，这里我们不探讨，你可以自己额外搜些资料）。
3. 第三步，接收入参。组件能从其父组件中接参数，而且组件是函数，因此该参数就是函数的入参，通常命名为属性 props。props 是一个对象，因此也可以直接对它进行解构，直接获取对象中的值。示例代码中用的就是用解构的方式来获取参数的，它直接获取了product参数，这里的product 是数据因此p是小写的。
4. 第四步，返回 JSX。组件的返回值就是 JSX，我们前面也提到过，它是用来描述 UI 页面的，JSX 最终生成的是视图元素、文字元素。这里我们初始化了一个元素，和两个元素。

### 组件的文件结构
```jsx

// index.js
AppRegistry.registerComponent('appName', () => App);



// App.js
const PRODUCTS = [
  {category: '水果', price: '￥1', name: 'PingGuo'},
];

export default function App() {
  return (
    <SafeAreaView style={{marginHorizontal: 30}}>
      <ProductTable products={PRODUCTS} />
    </SafeAreaView>
  );
}

// ProductTable.js
import Category from './Category';
import Product from './Product';

export default function ProductTable({products}){
  // ...
  <Category category={products[i].category}
  // ...
  <Product product={products[i]} 
  // ...  
}

// Category.js
export default function Category({category}){}

// ProductTable.js
export default function Product({product}) {}
```

1. index.js 文件：它是根文件，在该文件中registerComponent方法，会调用根组件 App，然后开始逐级调用，渲染应用；
2. App 组件：在 App 组件中，用于表示商品信息的数据变量 PRODUCTS，在被调用时会通过 ProductTable 组件的 products 属性传递下去；
3. ProductTable 组件：它被 App 组件调用后，它的调用入参就是 products。products 是一个数组，数组中的每一项就是 Product组件的入参product。每一项中的分类，就是Category 组件的入参 category。还是一样，组件首字母是大写的，属性、入参的首字母是小写的；
4. Category 组件：它会被 ProductTable 组件调用两次，第一次调用接收的入参category是“水果”，第二次是“蔬菜”；
5. Product 组件：它会被 ProductTable 组件调用 6 次，生成 6 个不同的商品元素，展示在手机屏幕上。


**简而言之，组件间的数据是单向流动的，是逐层往下传递**