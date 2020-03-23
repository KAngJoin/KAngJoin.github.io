---
layout: post
title: 面试—Javascript（ES系列）+DOM+BOM（持续更新...）
subtitle: 总结梳理知识点，温故知新
date: 2020-03-20
author: dukang
header-img: img/smile.jpg
catalog: true
tags: 
    - 面试
---

# Ⅰ、JavaScript

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



# Ⅱ、DOM

### 浏览器组成

1. 内核

   - 渲染引擎GUI
   - JS引擎

   **二者互斥**

2. 外壳

| 主流浏览器   | 内核（渲染引擎）               | Js引擎                                     |
| ------- | ---------------------- | ---------------------------------------- |
| IE/Edge | trident-->EdgeHTML(分支) | JScript（IE3.0-IE8.0） / Chakra（IE9+之后）    |
| Chrome  | webkit-->blink(分支)     | V8                                       |
| Safari  | webkit                 | Nitro（4-）                                |
| Opera   | Presto->blink          | SpiderMonkey（1.0-3.0）/ TraceMonkey（3.5-3.6）/ JaegerMonkey（4.0-） |
| Firefox | Gecko                  | Linear A（4.0-6.1）/ Linear B（7.0-9.2）/ Futhark（9.5-10.2）/ Carakan（10.5-） |

### 客户端存储（cookie、localStorage、sessionStorage）

1. **[Cookie](https://www.nowcoder.com/ta/front-end-interview/review?tpId=10&tqId=11077&query=&asc=true&order=&page=1)**

   **限制**

   - 大多数浏览器支持最大为4096字节（4kb）的Cookie；
   - 浏览器还会限制站点可以在用户计算机上存储的Cookie数量，大部分浏览器只允许每个站点存储20个Cookie。若试图存储更多的Cookie，则最老的Cookie便会被删除；
   - 部分浏览器会对所有站点的Cookie总数作出绝对限制，一般为300 个；
   - 默认情况下，Cookie会随着Http请求发送到后台服务器，可是并不是所有请求都需要Cookie，如：js、css、图片等

2. **会话级别的 sessionStorage**  （对象）

   会话级别：与浏览器当前会话相关，页面关闭，数据会自动消除。

   操作方法：

   - setItem(key,value)：添加本地存储数据；
   - getItem(key)：通过key获取相应的Value；
   - removeItem(key)：通过key删除本地数据；
   - clear()：清空数据。

3. **永久性的 localStorage ** （对象）

   数据不会随着Http 请求发送到后台服务器;

   存储数据的大小几乎不用考虑，HTML5的标准中要求浏览器至少要支持到**4MB**;

   **操作方法：同上**

> **注意**：浏览器获取ocal storage 和session storage 的值为<u>字符串类型</u>。

### [从输入URL到页面展示的过程](https://juejin.im/post/5c773dd251882519610194c1)

1. **URL解析**：首先判断你输入的是一个合法的 URL 还是一个待搜索的关键词，并且根据你输入的内容进行自动完成、字符编码等操作。

2. **DNS查询**：浏览器缓存->操作系统缓存->路由器缓存->ISP DNS缓存->根域名服务器查询

3. **TCP连接**：按我课本中TCP/IP网络模型来说

   1.应用层：发起http请求

   ```
   请求报头(Headers),请求方法,目标地址、遵循协议
   请求body
   ```

   2.传输层：TCP报文传输（TCP三次握手的建立）
   3.网络层：IP协议查询Mac地址
   4.链路层：以太网协议
   在发送请求时从应用层层层包裹数据，每一层都要对数据进行封装。

4. **处理请求**：接受 TCP 报文后，会对连接进行处理，对HTTP协议进行解析（请求方法、域名、路径等），并且进行一些验证。

5. **接受响应**：浏览器接收到来自服务器的响应资源后，会对资源进行分析。首先查看 Response header，根据不同状态码做不同的事（比如上面提到的重定向）

6. **渲染页面**：不同的浏览器内核，渲染过程也不完全相同。

### 浏览器如何解析渲染网页的？

1. 根据 HTML 解析 DOM 树

- 根据 HTML 的内容，将标签按照结构解析成为 DOM 树，DOM 树解析的过程是一个深度优先遍历。即先构建当前节点的所有子节点，再构建下一个兄弟节点。
- 在读取 HTML 文档，构建 DOM 树的过程中，若遇到 script 标签，则 DOM 树的构建会暂停，直至脚本执行完毕。

2. 根据 CSS 解析生成 CSS 规则树

- 解析 CSS 规则树时 js 执行将暂停，直至 CSS 规则树就绪。
- 浏览器在 CSS 规则树生成之前不会进行渲染。

3. 结合 DOM 树和 CSS 规则树，生成渲染树

- DOM 树和 CSS 规则树全部准备好了以后，浏览器才会开始构建渲染树。
- 精简 CSS 并可以加快 CSS 规则树的构建，从而加快页面相应速度。

4. 根据渲染树计算每一个节点的信息（布局）

- 布局：通过渲染树中渲染对象的信息，计算出每一个渲染对象的位置和尺寸
- 回流：在布局完成后，发现了某个部分发生了变化影响了布局，那就需要倒回去重新渲染。

5. 根据计算好的信息绘制页面

- 绘制阶段，系统会遍历呈现树，并调用呈现器的“paint”方法，将呈现器的内容显示在屏幕上。
- 重绘：某个元素的背景颜色，文字颜色等，不影响元素周围或内部布局的属性，将只会引起浏览器的重绘。
- 回流：某个元素的尺寸发生了变化，则需重新计算渲染树，重新渲染。

### 回流和重绘

- 页面的显示过程的几个阶段
  - 生成DOM树(**包括display:none的节点**) 
  - 在DOM树的基础上根据节点的集合属性(margin,padding,width,height等)生成render树(**不包括display:none，head节点，但是包括visibility:hidden的节点**)
  - 在render树的基础上继续渲染颜色背景色等样式

`reflow` : **当render树的一部分或者全部因为大小边距(结构)等问题发生改变而需要重建的过程，叫做回流**

`repaint `: 当页面中元素样式的改变并不影响它在文档流中的位置，而**只需要重新渲染**的过程，叫做重绘

`注意` **回流必然引起重绘，但重绘不一定引起回流。**

##### 何时发生回流

- 页面渲染初始化
- 添加或删除可见的DOM元素 ( DOM结构变化 )
- 元素的位置发生变化
- 改变`字体大小`会引发回流
- 元素的尺寸发生变化（包括外边距、内边框、边框大小、高度和宽度等）
- 内容发生变化，比如文本变化或图片被另一个不同尺寸的图片所替代
- 获取某些属性，（很多浏览器会对回流做优化，等到足够数量的变化发生，才做一次批处理回流）
- 浏览器的窗口尺寸变化(resize事件)【因为**回流是根据视口的大小来计算元素的位置和大小的**】
- 除了render树的直接变化。 当获取一些属性时，浏览器为了获得正确的值也会触发回流。这样就使得浏览器的优化失效了，这些属性包括以下

```javascript
1. offsetTop, offsetLeft, offsetWidth, offsetHeight
2. scrollTop/Left/Width/Height
3. clientTop/Left/Width/Height
4. width,height
5. 调用了getComputedStyle(), 或者 IE的 currentStyle
```

**示例**

```css
var s = document.body.style;
s.padding = "2px"; // 回流+重绘
s.border = "1px solid red"; // 再一次 回流+重绘
s.color = "blue"; // 再一次重绘
s.backgroundColor = "#ccc"; // 再一次 重绘
s.fontSize = "14px"; // 再一次 回流+重绘
// 添加node，再一次 回流+重绘
document.body.appendChild(document.createTextNode('abc!'));
```

##### 何时发生重绘

- 样式改变
- 初始化页面
- 回流必然引起重绘

##### 如何避免触发回流和重绘

CSS：

- 避免使用table布局（table元素一旦触发回流就会导致table里所有的其他元素回流）。
- 尽可能在DOM树的最末端改变class。
- 避免设置多层内联样式。
- 尽量使用css属性简写
- 将动画效果应用到`position`属性为`absolute`或`fixed`的元素上
- 避免使用CSS表达式（例如：`calc()` , 每次调用都会重新计算值、会重新加载页面）
- CSS3硬件加速（GPU加速）

JavaScript：

- 避免频繁操作样式，最好一次性重写style属性，或者将样式列表定义为class并一次性更改class属性；
- 避免频繁操作DOM，创建一个`documentFragment`（包含所有操作），最后再把它添加到文档中；
- 在display属性为none的元素上进行的DOM操作不会引发回流和重绘（先为元素设置`display: none`，操作结束后再把它显示出来）；
- 缓存layout属性值，减少回流次数，如const offsetLeft=element.offsetLeft；
- 对具有复杂动画的元素使用绝对定位，使它脱离文档流，否则会引起父元素及后续元素频繁回流；
- 需要创建多个DOM节点时，使用DocumentFragment创建完成后一次性的加入document。

##### `display:none`和`visibility：hidden`会产生回流与重绘吗

- display：none指的是元素完全不陈列出来，不占据空间，涉及到了DOM结构，故产生`eflow与repaint`
- visibility：hidden指的是元素不可见但存在，保留空间，不影响结构，故`只产生repaint`

##### 注意

有时即使仅仅回流一个单一的元素，它的父元素以及任何跟随它的元素也会产生回流。现代浏览器会对频繁的回流或重绘操作进行优化，浏览器会维护一个队列，把所有引起回流和重绘的操作放入队列中，如果队列中的任务数量或者时间间隔达到一个阈值的，浏览器就会将队列清空，进行一次批处理，这样可以把多次回流和重绘变成一次。你访问以下属性或方法时，浏览器会立刻清空队列：

- `clientWidth`、`clientHeight`、`clientTop`、`clientLeft`
- `offsetWidth`、`offsetHeight`、`offsetTop`、`offsetLeft`
- `scrollWidth`、`scrollHeight`、`scrollTop`、`scrollLeft`
- `width`、`height`
- `getComputedStyle()`
- `getBoundingClientRect()`

以上属性和方法都需要返回最新的布局信息，因此浏览器不得不清空队列，触发回流重绘来返回正确的值。因此，我们在修改样式的时候，**最好避免使用上面列出的属性，他们都会刷新渲染队列。**如果要使用它们，最好将值缓存起来。