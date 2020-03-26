---
layout: post
title: 复习—Javascript（ES系列、持续更新...）
subtitle: 总结梳理知识点，温故知新
date: 2020-03-20
author: dukang
header-img: img/smile.jpg
catalog: true
tags: 
    - 复习（面试）
---

### JS基本数据类型

**7大数据类型**

- undefined、null、String、Number、Boolean、Symbol、（BigInt 【 ES10】）  +  Object【引用的】

**两大细分**

- 基本数据类型：number、string、null、undefined、boolean、symbol -- 栈


- 引用数据类型：object、array、function -- 堆

**区别与特性**

1. 两种数据类型**存储位置**不同
2. **原始数据类型**是直接存储在栈(stack)中的简单数据段，占据空间小、大小固定，属于被频繁使用数据；
3. **引用数据类型**存储在堆(heap)中的对象，占据空间大、大小不固定，如果存储在栈中，将会影响程序运行的性能；
4. 引用数据类型在栈中存储了指针，该指针指向堆中该实体的起始地址。
5. 当解释器寻找引用值时，会首先检索其在栈中的地址，取得地址后从堆中获得实体。

### == 和 === 的区别

`==`	先比较类型，类型不同先进行类型转换；再比较值；

`===` 先比较类型，不同则返回false；类型相同才比较值。

### Map和Set的区别

### 继承和ES5与ES6继承的区别

- **思路**

> ES5 的继承使用借助构造函数实现，实质是`先创造子类的实例对象this`，然后再将父类的方法添加到`this`上面。ES6 的继承机制完全不同，实质是`先创造父类的实例对象this`（所以必须先调用`super`方法），然后再用子类的构造函数修改`this` ;
>
> ES6 在继承的语法上不仅继承了类的原型对象，还继承了类的静态属性和静态方法
>
> ...

### 对原生JavaScript的理解

- 思路

>JavaScript 实现包含的几个部分;
>
>JavaScript 的语言类型特性;
>
>解释性脚本语言（对标编译性脚本语言）;
>
>面向对象（面向过程）;
>
>事件驱动 / 异步 IO ;
>
>缺少的关键性功能等（块级作用域 、模块、子类型等）;
>
>自由...

### 如何提升 JavaScript 变量的存储性能？

- 思路

> 访问字面量和局部变量的速度最快，访问数组元素和对象成员相对较慢;
>
> 由于局部变量存在于作用域链的起始位置，因此访问局部变量比访问跨作用域变量更快，全局变量的访问速度最慢;
>
> 避免使用`with`和`catch`，除非是有必要的情况下;
>
> 嵌套的对象成员会明显影响性能，尽量少用，例如`window.loacation.href`;
>
> 属性和方法在原型链中的位置越深，则访问它的速度也越慢;
>
> 通常来说，需要访问多次的对象成员、数组元素、跨作用域变量可以保存在局部变量中从而提升 JavaScript 执行效率;

### null和undefined的区别

- `null`表示"没有对象"，即该处不应该有值。典型用法是：
  - 作为函数的参数，表示该函数的参数不是对象。
  - 作为对象原型链的终点。


- `undefined`表示"缺少值"，就是此处应该有一个值，但是还没有定义。典型用法是：
  - 变量被声明了，但没有赋值时，就等于undefined。
  - 调用函数时，应该提供的参数没有提供，该参数等于undefined。
  - 对象没有赋值的属性，该属性的值为undefined。
  - 函数没有返回值时，默认返回undefined。

### 箭头函数与普通函数的区别

1. `对this的关联`。内置this的值，取决于箭头函数在哪里定义，而非箭头函数执行的上下文环境。
2. `new 不可用`。箭头函数不能用new关键字来实例化对象，否则报错。
3. `this指向不会改变`。函数内置this指向不可改变，this在函数体内整个执行环境中为常量。有利于JavaScript引擎优化处理。
4. `没有arguments对象`。不能通过arguments对象访问传入的实参。只能使用显示命名或者其它新特性完成。

###  void

```javascript
console.log(void 0 === undefined) // true
console.log(void 1 === undefined) // true

function fn(name: string, add: string = '湖南') {} 
//解析为：
function fn(name, add) {
  if (add === void 0) { add = '湖南'; } // 注意：函数参数默认值的转换 
  // if (typeof add === 'undefined' ) { add = '湖南'; }
}
```
**注意**： void 返回的都是 undefined 类型

### js判断对象是否为空对象

- 将js对象转换为json字符串，再判断该字符串是否为 "{}"

```javascript
let obj = {};
let b = (JSON.stringify(obj) == '{}');
console.log(b);	// true
//注意的是：obj是对象类型，“{}”是字符串类型。

//所以 不能使用 toString（） // 错误的做法
let obj = {};
let b = (obj.toString() == '{}'); 
console.log(b);  // false   
```

- for in 遍历对象 

```javascript
let obj = {};
let b = function () {    
  for (let key in obj) { 
    return false    
  }   
  return true
};
console.log(b()); // true
```

- jQuery的 isEmptyObject方法

```javascript
//该方法是对 （for in）的封装；
let obj = {};
let b = $.isEmptyObject(obj);
alert(b); // true
```

- Object.getOwnPropertyNames()方法

  使用Object对象的getOwnPropertyNames方法，获取到对象中的属性名，存到一个数组中，返回数组对象，我们可以通过判断数组的length来判断此对象是否为空

```javascript
let obj = {};
let b = Object.getOwnPropertyNames(obj);
console.log(b.length); // 0 
```

- 使用ES6的Object.keys()方法

```javascript
//该方法返回的同样是属性名组成的数组对象。
let obj = {};
let arr = Object.keys(obj);
console.log(arr.length); // 0	
```

