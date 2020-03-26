---
layout: post
title: Javascript节流和防抖
subtitle: 优化
date: 2020-03-12
author: dukang
header-img: img/js.jpg
catalog: true
tags: 
    - Javascript
---

## 写在前面

​	实际开发中，很多业务都会遇到这样的情况——事件或函数频繁被触发，由此而导致的一系列卡顿，白屏等问题，比较常见的有：

1. 鼠标点击事件 click，用户频繁点击
2. keyup、keydown，用户输入内容，搜索请求数据，
3. window的resize、scroll
4. mousedown 、mousemove

  ​诸如此类的问题，我们需要从代码的角度思考并解决问题，要么时间限制只触发一次，要么每间隔一段时间触发执行一次。而常用的两种解决方案是： 1. 防抖 ；2. 节流





## 防抖（debounce)

#### 定义

事件触发后**n秒内不再触发**则执行事件；若n秒内再次触发，则重新开始计时，直到n秒内不触发,执行事件。

`不恰当比喻`——你坐电梯时，门要关了，突然来了一个人，此时电梯并没有改变楼层，而是再次打开了电梯门。这样电梯延迟了改变楼层的功能，同时也优化了资源（多搭载了一个人）



#### 不同需求实现

###### 普通版

​	触发后必须指定时间到了才触发

```javascript
function debounce(fn, wait) {
  // 维护一个timer
  let timer = null;
  return function () {
    // 通过 ‘this’ 和 ‘arguments’ 获取函数的作用域和变量
    let context = this;
    let args = arguments;
    clearTimeout(timer);//清除最后一次点击之前触发的事件
    timer = setTimeout(function () {
      fn.apply(context, args);
    }, wait);
  }
}

// 注意 fn无返回值（默认undefined）
```

###### 立即执行版

​	触发后立即执行，指定时间内不可再次触发事件，必须时间过了才能再次触发。通过添加一个immediate【Boolean】参数，控制是否立即执行

```javascript
function debounce(fn, wait = 800, immediate = true) {
  let timer;
  return function () {
    let context = this;
    let args = arguments;
+   timer && clearTimeout(timer);
+   if (immediate) {
+     // 首次触发，timer默认为假值。
+     // 若连续触发，则timer为setTimeout返回的最新的ID值[Number]，则runNow为false。
+     let runNow = !timer;
+     // 指定时间wait过后才将定时器清除
+     timer = setTimeout(function () {
+       timer = null;
+     }, wait);
+     if (runNow) fn.apply(context, args)
    } else {
      timer = setTimeout(function () {
        fn.apply(context, args);
      }, wait);
    }
  }
};
// 注意 fn无返回值（默认undefined）
```

###### fn指定了返回值

```javascript
function debounce(fn, wait = 800, immediate = true) {
  let timer;
  return function () {
    let context = this, args = arguments;
    timer && clearTimeout(timer);
    if (immediate) {
      // 首次触发，timer默认为假值。
      // 若连续触发，则timer为setTimeout返回的最新的ID值[Number]，则runNow为false。
      let runNow = !timer;
      // 指定时间wait过后才将定时器清楚
      timer = setTimeout(function () {
        timer = null;
      }, wait);
+     if (runNow) return fn.apply(context, args)
    } 
    else {
      timer = setTimeout(function () {
        return fn.apply(context, args);
      }, wait);
    }
  }
};
```

目前要返回值的话，只能在immediate为true时返回。因为else语句中为异步代码执行，所以一致返回的都是undefined。

###### 可取消

可以在等待执行的过程中取消延迟并立即执行函数fn；

```javascript
function debounce(fn, wait = 800, immediate = true) {
+ let timer, debounced;
+ debounced = function () {
    let context = this, args = arguments;
    timer && clearTimeout(timer);
    if (immediate) {
      // 首次触发，timer默认为假值。
      // 若连续触发，则timer为setTimeout返回的最新的ID值[Number]，则runNow为false。
      let runNow = !timer;
      // 指定时间wait过后才将定时器清楚
      timer = setTimeout(function () {
        timer = null;
      }, wait);
      if (runNow) return fn.apply(context, args)
    }
    else {
      timer = setTimeout(function () {
        return fn.apply(context, args);
      }, wait);
    }
  }
+ debounced.cancel = () => {
+   clearTimeout(timer)
+   timer = null
+ }
+ return debounced
};
```

#### Vue中使用防抖函数

```vue
<template>
<div @click="handle(params)"></div>
</template>
<script>
  import { Debounce } from "../../common/js/debounce"; //debounce.js 
  methods: {
    handle: Debounce(function(params){
      //将正常处理业务的代码写在这里
    },300)
  }
</script>
```





## 节流（throttle）

#### 定义

持续触发事件或函数，每隔一段时间，只执行一次（时间段内最后那一次）。

`不恰当的比喻`——就好比水积攒到一定重量才会下落。

`核心思想`——**创建计时器，延迟程序的执行 。 **

#### 特点

1. 实现方式多样，时间戳、定时器
2. 首次是否执行？结束是否执行？

#### 优势

1. 程序能否执行是可控的。执行前的某一时刻是否清楚计时器来决定程序是否可以继续执行；
2. 程序是异步的。由于计时器机制，使得程序脱离源程序而异步执行（worker多线程方案），且不影响其它程序正常执行。

#### 使用时间戳

```javascript
//指定时间内触发，时间一到就会触发（不管你执行了多少次）
//思路：定义一个上次的时间，时间触发获取当前时间，满足【当前时间-上次时间>延迟时间】则触发，然后更新上次时间previous。
function throttle(func, delay) {
  let previous = 0;
  return function () {
    let now = new Date();
    if (now - previous > delay) {
      func.apply(this, arguments);
      previous = now;
    }
  }
}
```

注意：一触发即执行，然后等下一次

#### 使用定时器

```javascript
// 思路：事件触发时，设置一个定时器，定时器未执行前再次触发事件则不执行；定时器执行，则执行函数，然后清空定时器，再设置下一个定时器。
function throttle(fn, delay) {
  let timer;
  return function () {
    let context = this, args = arguments;
    if (!timer) {
      timer = setTimeout(() => {
        timer = null;
        fn.apply(context, args)
      }, delay);
    }
  }
}
```

注意：触发了需要等待delay秒才执行，然后定义下一周期

#### 合二为一（有头有尾）

有头——触发时便执行一次，第一次间隔到了再执行一次

有尾——停止触发，会更根据计算条件再执行一次

```javascript
// 第一次触发即执行，然后每隔一段时间执行，最后停止再时间间隔结束时执行一次
function throttle(fn, delay) {
  let timer, context, args,
      // 记录上次触发的时间
      previous = 0;
  return function () {
    let now = +new Date(),
        //下次触发fn剩余的时间
        remaining = delay - (now - previous);
    context = this;
    args = arguments;
    // 无剩余时间，或系统时间改变(人为)
    if (remaining <= 0 || remaining > delay) {
      // 首次触发会立即执行本部分
      if (timer) {
        clearTimeout(timer)
        timer = null;
      }
      previous = now;
      fn.apply(context, args)
    } else if (!timer) {
      // 设置定时器
      timer = setTimeout(() => {
        fn.apply(context, args)
        previous = +new Date();
        timer = null;
      }, remaining);
    }
  }
}
// 正常一次连续触发的执行应该是： 一次if，然后每次都是else if（最后一次也是），
```



## 应用场景

- debounce
  - 搜索框——直到用户一段时间内不在输入才搜索内容
  - window的resize——窗口大小不再变化
- throttle
  - 鼠标点击——
  - 滚动事件——滑动到底部加载更多




## 总结

#### 如何区分防抖和节流？

1. 防抖函数，将多次执行变为**最后一次执行**
2. 节流函数，将多次执行变成**每隔一段时间执行**





## 参考

#### [节流](https://github.com/mqyqingfeng/Blog/issues/26)

#### [防抖](https://github.com/mqyqingfeng/Blog/issues/22)