---
title: "JS基础 函数、对象和原型、原型链的关系"
date: 2021-06-29T16:51:01+08:00
categories: ["前端"]
keywords: ["ES6","javaScript"]
tags: ["ES6","javaScript"]
draft: true
---

# 再次学习JS-函数、对象和原型、原型链的关系

>JS的原型、原型链一直是比较难理解的内容，不少初学者甚至有一定经验的老鸟都不一定能完全说清楚，更多的"很可能"是一知半解，而这部分内容又是JS的核心内容，想要技术进阶的话肯定不能对这个概念一知半解，碰到问题靠“猜”，却不理解它的规则！

## prototype

### 只有函数有prototype属性

```js
let a = {}
let b = function () { }
console.log(a.prototype) // undefined
console.log(b.prototype) // { constructor: function(){...} }
```

#### Object.prototype怎么解释？

其实`Object`是一个全局对象，也是一个构造函数，以及其他基本类型的全局对象也都是构造函数：

```js
function outTypeName(data, type) {
    let typeName =  Object.prototype.toString.call(data)
    console.log(typeName)
}
outTypeName(Object) //[object Function]
outTypeName(String) // [object Function]
outTypeName(Number) // [object Function]
```

### 为什么只有函数有prototype属性

JS通过`new`来生成对象，但是仅靠构造函数，每次生成的对象都不一样。

有时候需要在两个对象之间共享属性，由于JS在设计之初没有类的概念，所以JS使用函数的`prototype`来处理这部分**需要被共享的属性**，通过函数的`prototype`来模拟类：

当创建一个函数时，JS会自动为函数添加`prototype`属性，值是一个有`constructor`的对象。

以下是共享属性`prototype`的栗子：

```js
function People(name) {
    this.name = name
}
People.prototype.age = 23 // 岁数
// 创建两个实例
let People1 = new People('OBKoro1')
let People2 = new People('扣肉')
People.prototype.age = 24 // 长大了一岁
console.log(People1.age, People2.age) // 24 24
```



```js
function isType(data, type) {
    const typeObj = {
        '[object String]': 'string',
        '[object Number]': 'number',
        '[object Boolean]': 'boolean',
        '[object Null]': 'null',
        '[object Undefined]': 'undefined',
        '[object Object]': 'object',
        '[object Array]': 'array',
        '[object Function]': 'function',
        '[object Date]': 'date', // Object.prototype.toString.call(new Date())
        '[object RegExp]': 'regExp',
        '[object Map]': 'map',
        '[object Set]': 'set',
        '[object HTMLDivElement]': 'dom', // document.querySelector('#app')
        '[object WeakMap]': 'weakMap',
        '[object Window]': 'window',  // Object.prototype.toString.call(window)
        '[object Error]': 'error', // new Error('1')
        '[object Arguments]': 'arguments',
    }
    let name = Object.prototype.toString.call(data) // 借用Object.prototype.toString()获取数据类型
    let typeName = typeObj[name] || '未知类型' // 匹配数据类型
    return typeName === type // 判断该数据类型是否为传入的类型
}
console.log(
    isType({}, 'object'), // true
    isType([], 'array'), // true
    isType(new Date(), 'object'), // false
    isType(new Date(), 'date'), // true
)
```

