---
layout: post
title: Javascript执行机制
subtitle: Event Loop
date: 2020-03-25
author: dukang
header-img: img/js.jpg
catalog: true
tags: 
    - Javascript
---

### 下面代码输出结果

```javascript
setTimeout(() => {
console.log('stt-1');
});
new Promise((resolve,reject)=>{
console.log('pms-1');
resolve();
}).then(()=>{
console.log('then-1');
})
console.log('glo-1');
// 解析
// setTimeout进入队列——宏任务
// new Promise（）立即执行
// promise.then 进入队列——微任务
// log 立即执行

// 所以结果： pms-1 glo-1 then-1 stt-1
```

### 你应该知道的一些知识点

![](http://dukangblog.top/img/JavaScript执行机制.jpg)

##### 图解

1. 执行代码(主线程 | 执行栈)，同步任务依次执行
2. 遇到异步任务，插入到任务队列
3. 主线程执行完毕后，检查任务队列
4. 先执行微任务，再执行宏任务（注意，第一次直接执行的是整体script代码，是宏任务，因为queue为空）
5. 重复1-4步
6. 代码执行完成
7. 用户操作的事件（mouse、onkeydown...）

首先，主线程会去执行所有的同步任务。等到同步任务全部执行完，就会去看任务队列里面的异步任务。如果满足条件，那么异步任务就重新进入主线程开始执行，这时它就变成同步任务了。等到执行完，下一个异步任务再进入主线程开始执行。一旦任务队列清空，程序就结束执行。

##### 思考(QA)

- 如何知道主线程执行栈为空？

  js引擎存在 `monitoring process` 进程，会持续不断的检查主线程执行栈是否为空，一旦为空，就会去Event Queue那里检查是否有等待被调用的函数。

- 宏任务(macro-task)

  `整体代码script` 、[`setTimeout`](#setTimeout) 、[`setInterval` ](#重新认识setInterval)

- 微任务(micro-task)

  `Promise ` 、` process.nextTick（node.js)`

### setTimeout

​	指定某个函数或某段代码，在多少毫秒之后执行。【不完全正确的理解】

- 返回值

  返回一个整数，表示定时器的编号，以后可以用来取消这个定时器。

  setTimeout和setInterval返回的整数值是连续的，也就是说，第二个setTimeout方法返回的整数值，将比第一个的整数值大1。

- 参数

  `第一个参数`：回调函数 或 代码块（必须是字符串格式）；

  `第二个参数`：指定间隔时间 （ms）；

  `之后的参数`：都将作为回调函数的参数。

HTML5标准规定了setTimeout()的第二个参数的最小值（最短间隔），不得低于4毫秒，如果低于这个值，就会自动增加。在此之前，老版本的浏览器都将最短间隔设为10毫秒。另外，对于那些DOM的变动（尤其是涉及页面重新渲染的部分），通常不会立即执行，而是每16.7毫秒执行一次。这时使用requestAnimationFrame()的效果要好于setTimeout()。

你应该经常像下面这样使用。可很多时候，你会发现setTimeout中的业务并不是2秒后就马上执行，会大于2秒。

```javascript
setTimeout(() => {
  //do something
  task()
}, 2000);
```

比如，像下面这样

```javascript
setTimeout(() => {
  //do something
  task()
}, 2000);
function sleep(ms) {
    let start= Date.now();
    while ((Date.now()-start)<ms){}
}
sleep(10000); // 随眠函数
```

分析上述代码执行过程：

1. task进入任务队列并注册，计时器计时开始
2. 执行sleep函数，可能需要10秒
3. 2秒到了，task等待执行，可主线程sleep还未执行完
4. sleep执行完（主线程为空），现在执行task，已过10秒

现在看来，setTimeout函数是`经过指定时间后，把其内部需要执行的任务加入到任务队列中（等待执行），`又因为是单线程任务要一个一个执行，如果前面的任务需要的时间太久，那么只能等着，导致真正的延迟时间远远大于指定时间。

我们可以得到一个结论：`无法确定主线程任务需要多少时间执行完，则不能保证，setTimeout和setInterval指定的任务一定会按照预定时间执行`

##### 注意

如果`回调函数是对象的方法`，那么setTimeout使得方法内部的this关键字指向全局环境（window），而不是定义时所在的那个对象。

```javascript
var x= 1; //这里只能是 var 声明，否则结果为undefined。
var obj = {
    x:2,
    y:function () {
        console.log(this.x);
    }
};
//注意这里的 是 obj.y，而不是obj.y()
setTimeout(obj.y,1000);  // 1

//this指向的解决办法：
//第一种、将对象的方法放在一个函数里
setTimeout(function (){obj.y()},1000); 
//第二种、使用bind方法，将对象的方法（obj.y）绑定在obj上
setTimeout(obj.y.bind(obj),1000);
```

### setTimeout(fn,0)

setTimeout(fn,0)的含义是，`指定某个任务在主线程最早可得的空闲时间执行，也就是说，尽可能早得执行`。它在"任务队列"的尾部添加一个事件，因此要等到同步任务和"任务队列"现有的事件都处理完，才会得到执行。

需要注意的是，setTimeout()只是将事件插入了"任务队列"，必须等到当前代码（执行栈）执行完，主线程才会去执行它指定的回调函数。要是当前代码耗时很长，有可能要等很久，所以并没有办法保证，回调函数一定会在setTimeout()指定的时间执行。

##### 应用

- 网页开发中，某个事件先发生在子元素，然后冒泡到父元素，即子元素的事件回调函数，会早于父元素的事件回调函数触发。如果，想让父元素的事件回调函数先发生，就要用到setTimeout(f, 0)

```html
<input type="button" id="myBtn" value="click">
```

```javascript
var input = document.getElementById('myBtn');
input.onclick = function A() {
  setTimeout(function B() {
    console.log('input')
  },0)
};
document.body.onclick = function C() {
  console.log('body')
}
// body input 
```

- 用户自定义的回调函数，通常在浏览器的默认动作之前触发。比如，用户在输入框输入文本，keypress事件会在浏览器接收文本之前触发。

```html
<input type="text" id="input-box">
```

```javascript
/*下面的代码取不到最新输入的那个字符*/
document.getElementById('input-box').onkeypress = function (event) {
  this.value = this.value.toUpperCase()
}
```

```javascript
// 解决办法
document.getElementById('input-box').onkeypress = function (event) {
  var self = this;
  setTimeout(function () {
    self.value = self.value.toUpperCase()
  },0)
}
```

### setInterval

`setInterval`会每隔指定的时间将注册的函数置入Event Queue，如果前面的任务耗时太久，那么同样需要等待。

对于`setInterval(fn,ms)`来说，我们已经知道不是每过`ms`秒会执行一次`fn`，而是每过`ms`秒，会有`fn`进入Event Queue。一旦**setInterval的回调函数fn执行时间超过了延迟时间ms，那么就完全看不出来有时间间隔了**。

##### 注意

setInterval指定的是`“开始执行”之间的间隔`，并不考虑每次任务执行本身所消耗的时间。因此实际上，两次执行之间的间隔会小于指定的时间。比如，setInterval指定每 100ms 执行一次，每次执行需要 5ms，那么第一次执行结束后95毫秒，第二次执行就会开始。如果某次执行耗时特别长，比如需要105毫秒，那么它结束后，下一次执行就会立即开始。

- 解决办法，在每次执行结束后使用setTimeout指定下次执行的具体时间。

```javascript
var timer = setTimeout(function f() {
    timer = setTimeout(f, 2000);
}, 2000);
```

##### 轮询 URL 的 Hash 值（#）是否发生变化

```javascript
var hash = window.location.hash;
setInterval(function () {
  if (window.location.hash != hash) {
    window.location.reload(); //刷新页面
  }
}, 2000);
```

### setImmediate

这算一个比较新的定时器，目前IE11/Edge支持、Nodejs支持，Chrome不支持，其他浏览器未测试。

这个api的支持性并不是很好。

从API名字来看很容易联想到setTimeout(0)，不过setImmediate应该算是setTimeout(0)的替代版。

在IE11/Edge中，setImmediate延迟可以在1ms以内，而setTimeout有最低4ms的延迟，所以setImmediate比setTimeout(0)更早执行回调函数。不过在Nodejs中，两者谁先执行都有可能，原因是Nodejs的事件循环和浏览器的略有差异。

很明显，setImmediate设计来是为保证让代码在下一次事件循环执行，以前setTimeout(0)这种不可靠的方式可以丢掉了

总之，记住setImmediate会比setTimeout(fn, 0)更快、更及时一点就么有错了。

### 拓展阅读

##### JS为什么是单线程的？

JavaScript语言的一大特点就是单线程，也就是说，同一个时间只能做一件事。那么，为什么JavaScript不能有多个线程呢？这样能提高效率啊。

JavaScript的单线程，与它的用途有关。作为浏览器脚本语言，JavaScript的主要用途是与用户互动，以及操作DOM。这决定了它只能是单线程，否则会带来很复杂的同步问题。比如，假定JavaScript同时有两个线程，一个线程在某个DOM节点上添加内容，另一个线程删除了这个节点，这时浏览器应该以哪个线程为准？

所以，为了避免复杂性，从一诞生，JavaScript就是单线程，这已经成了这门语言的核心特征，将来也不会改变。

为了利用多核CPU的计算能力，HTML5提出Web Worker标准，允许JavaScript脚本创建多个线程，但是子线程完全受主线程控制，且不得操作DOM。所以，这个新标准并没有改变JavaScript单线程的本质。

##### 假死（浏览器无响应）

常见的浏览器无响应（假死），往往就是因为某一段 JavaScript 代码长时间运行（比如死循环），导致整个页面卡在这个地方，其他任务无法执行。JavaScript 语言本身并不慢，慢的是读写外部数据，比如等待 Ajax 请求返回结果。这个时候，如果对方服务器迟迟没有响应，或者网络不通畅，就会导致脚本的长时间停滞。

##### js为什么需要异步？

如果JS中不存在异步，只能自上而下执行，如果上一行解析时间很长，那么下面的代码就会被阻塞。 对于用户而言，阻塞就意味着"卡死"，这样就导致了很差的用户体验。

##### js是单线程的，那么又是如何实现异步的？

事件循环(event loop)，理解了event loop机制，就理解了JS的执行机制。

##### 任务队列

"任务队列"是一个事件的队列（也可以理解成消息的队列），IO设备完成一项任务，就在"任务队列"中添加一个事件，表示相关的异步任务可以进入"执行栈"了。主线程读取"任务队列"，就是读取里面有哪些事件。

"任务队列"中的事件，除了IO设备的事件以外，还包括一些用户产生的事件（比如鼠标点击、页面滚动等等）。只要指定过回调函数，这些事件发生时就会进入"任务队列"，等待主线程读取。

所谓"回调函数"（callback），就是那些会被主线程挂起来的代码。异步任务必须指定回调函数，当主线程开始执行异步任务，就是执行对应的回调函数。

"任务队列"是一个先进先出的数据结构，排在前面的事件，优先被主线程读取。主线程的读取过程基本上是自动的，只要执行栈一清空，"任务队列"上第一位的事件就自动进入主线程。但是，由于存在后文提到的"定时器"功能，主线程首先要检查一下执行时间，某些事件只有到了规定的时间，才能返回主线程。

读取到一个异步任务，首先是将异步任务放进事件表格（Event table）中，当放进事件表格中的异步任务完成某种事情或者说达成某些条件（如setTimeout事件到了，鼠标点击了，数据文件获取到了）之后，才将这些异步任务推入事件队列（Event Queue)中，这时候的异步任务才是执行栈中空闲的时候才能读取到的异步任务。

##### 任务队列的本质

 	1. 所有同步任务都在主线程上执行，形成一个执行栈（execution context stack）。
 	2. 主线程之外，还存在一个”任务队列”（task queue）。只要异步任务有了运行结果，就在”任务队列”之中放置一个事件。
 	3. 一旦”执行栈”中的所有同步任务执行完毕，系统就会读取”任务队列”，看看里面有哪些事件。那些对应的异步任务，于是结束等待状态，进入执行栈，开始执行。
 	4. 主线程不断重复上面的第三步。

##### Event Loop

主线程从"任务队列"中读取事件，这个过程是循环不断的，所以整个的这种运行机制又称为Event Loop（事件循环）。

Event Loop是javascript的执行机制

​	[Event Loop机制（浏览器和Node）]()

##### requestAnimationFrame

requestAnimationFrame并不是定时器，但和setTimeout很相似，在没有requestAnimationFrame的浏览器一般都是用setTimeout模拟。

requestAnimationFrame跟屏幕刷新同步，大多数屏幕的刷新频率都是60Hz，对应的requestAnimationFrame大概每隔16.7ms触发一次，如果屏幕刷新频率更高，requestAnimationFrame也会更快触发。基于这点，在支持requestAnimationFrame的浏览器还使用setTimeout做动画显然是不明智的。

这一点很关键，requestAnimationFrame是跟着屏幕来刷新的，而不会顾及到任务队列的事情。所以会更为及时。

在不支持requestAnimationFrame的浏览器，如果使用setTimeout/setInterval来做动画，最佳延迟时间也是16.7ms。 如果太小，很可能连续两次或者多次修改dom才一次屏幕刷新，这样就会丢帧，动画就会卡；如果太大，显而易见也会有卡顿的感觉。

所以，我们最好就要设置为16.7ms，如果设置的少了，还有可能出现问题，何必呢？

有趣的是，第一次触发requestAnimationFrame的时机在不同浏览器也存在差异，Edge中，大概16.7ms之后触发，而Chrome则立即触发，跟setImmediate差不多。按理说Edge的实现似乎更符合常理。