---
layout: post
title: Javascript内存泄漏
subtitle: 内存问题
date: 2020-03-14
author: dukang
header-img: img/js.jpg
catalog: true
tags: 
    - Javascript
---

# 内存泄漏

​	内存泄漏指的是：**任何对象在你不再拥有或不再需要之后任然存在**。

- 不再拥有——（无法获取）
- 不再需要——（任存在隐藏的引用）

# 常见的内存泄漏

1. 闭包
2. 控制台日志
3. 循环（两对象彼此引用且彼此保留）
4. 事件监听，addEventListener需要removeEventListener移除（**传递给两者的函数必须一致**）
5. setTimeout/setInterval ，对应的使用clearTimeout/clearInterval清空
6. **注意，使用setTimeout模拟setInterval 循环调用会造成内存泄漏**
7. 如Promise、rxjs的Observables、node的EventEmitters这些方法，无回调函数或未取消监听都会造成内存泄漏
8. Promise如果没有resolved或者rejected，会连同then()中的代码一起造成内存泄漏
9. 在没有虚拟dom的计算下实现了无无限滚动，那么dom节点的数量将无限增加
10. [IntersectionObserver](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver), [ResizeObserver](https://developer.mozilla.org/zh-CN/docs/Web/API/ResizeObserver), [MutationObserver](https://developer.mozilla.org/zh-CN/docs/Web/API/MutationObserver) 这些新的事件监听Api，都必须使用对应的disconnect取消监听
11. 同redux、vuex这样**挂载在全局的状态管理，如果不注意内存的占用，则会持续增加不会被释放**