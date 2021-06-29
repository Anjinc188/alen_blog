---
title: "ES6构造函数class与ES5构造函数语法"
date: 2020-11-09 
categories: ["前端"]
keywords: ["ES6","javaScript"]
tags: ["ES6","javaScript"]
draft: true
---

## ES5的构造函数语法1212212

##### 1.创建对象的方法有两种

```javascript
//字面量
const obj = {} //2222

//构造函数
const obj = new Object()
```

构造函数就是JavaScript程序定义好的构造函数,我们直接使用就可以了,构造函数是专门用于生成、定义对象的

##### 2.通过构造函数生成的对象称为实例化对象

- 实例化对象就是通过构造函数,生成的对象,称为实例化对象

##### 3.构造函数一定要和关键词 new一起使用

- new 关键词具有特殊的功能,会自动给构造函数中定义一个对象,并且返回这个对象；我们只要对这个对象设定属性和方法就可以
- 构造函数的语法规范规定：构造函数的函数名称,第一个字母必须大写,使用大驼峰命名法。
- 构造函数给对象定义属性和方法的语法与一般函数不同

**代码展示**

```javascript
function Animal(name, age) {
  this.name = name;
  this.age = age;
  //定义方法  
   this.fun = function(){
      console.log(this.name,this.age)
   }
}

//通过原型
Animal.prototype.call = function () {
	console.log('我可以打电话')
}
//通过自定义构造函数生成实例化对象
const o = new Animal('alen',20);
console.log(o);

// 调用实例化对象中的方法
o.fun();
o.call();
```

### 函数的prototype属性

>不同的实例化对象中,定义的是不同的方法、函数，会占用过多的内存空间。所以，一般将构造函数需要定义给实例化对象的方法,定义在函数的 prototype 属性中

- 定义在 prototype 中的属性,就是写在 prototype 中的,不会写在实例化对象中。所以实例化对象中要添加属性，都是通过 this关键词来指向实例化对象。
- 在 prototype 中定义的方法/函数,也不会定义在实例化对象上，只会出现在函数的prototype 中。

#### __proto__属性

1. 构造函数在生成实例化对象时,会将自己 prototype 这个空间的地址赋值给实例化对象 的__proto__来存储。 实际上，构造函数的 prototype和生成的实例化对象的__proto__指向的是同一个存储空间，可以相互调用数据。
2. 当调用实例化对象中定义在函数本身上的函数/数据时，其实实例化对象本身中是没有这个函数的，会继续在 __proto__中寻找是否有这个方法函数 /数据。__proto__实际上指向的就是构造函数的 prototype。因为构造函数的 prototype 中是有这个方法函数/数据的,所以就可以正常调用。

#### 原型对象、原型属性、原型链三者的联系

1. 原型对象

 每一个函数本身就存在的prototype 属性,称为原型对象；是一个专门用来存储数据、函数等内容的空间。

2. 原型属性

 每一个对象本身就存在的 __proto__属性,称为原型属性。实例化对象的原型属性指向的是创建实例化对象的构造 函数的 prototype。

3. 原型链

 原型链就是将所有相互关联的变量使用__proto__ 属性串联起来， 在调用数据时,会通过__proto__ 将所有相互关联的 变量串联，只要有一个变量中有相应的属性,就会调用成功。

## ES6中新增语法: class 类

- 通过 class 关键字可以定义类，提供了更接近传统语言的写法，引入了 Class （类）这个概念作为对象的模板。

#### 代码展示

```javascript
// class 写法
class Animal {
  constructor (name, age) {
    this.name = name;
    this.age = age;
  }
  fun() {
    console.log('hello')
  }
}
var o = new Animal('alen', '22');
console.log(o)
```

#### 类的所有方法都是定义在类的 prototype 属性上

```javascript
class Animal {
  constructor() { ... };
  toString() { ... };
  getName() { ... };
}

// 等价于
Animal.prototype = {
  constructor() {},
  toString() {},
  getName() {}
}
```

#### 类的所有方法都是定义在类的 prototype 属性上

```javascript
class Animal {
  constructor() { ... };
  toString() { ... };
  getName() { ... };
}

// 等价于
Animal.prototype = {
  constructor() {},
  toString() {},
  getName() {}
}
```

#### 在类的实例上调用方法，其实就是调用原型上的方法。

```javascript
class A {};
let o = new b();
o.constructor === o.prototype.constructor; // true
```

#### 由于类的方法（除 constructor 之外）都定义在 prototype 对象上，所以类的新方法可以添加在 prototype 对象上。Object.assgn() 方法可以很方便的一次向类添加多个方法。

```javascript
class Animal {
  constructor () { ... };
}
Object.assign(Animal.prototype, {
  toString() { ... },
  getName() { ... }
});
```

