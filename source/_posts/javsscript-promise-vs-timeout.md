---
title: Javascript 中 Promise 与 setTimeout 的执行顺序
tags: 
 - javascript
 - eventloop
---

考虑以下这种情况, 哪条语句会先打印？

```
setTimeout(()=> {
    console.log("timeout");
})
Promise.resolve().then(()=>{
    console.log("promise");
})
console.log("main");
```
在考虑这个问题之前，我们有必要了解一下 javascript 的运行机制。  

首先我们要明白一个基本原理，Javascript，无论是基于浏览器还是基于 Nodejs 环境，都是基于事件循环（event loop) 而运行的:  
> Javascript 引擎在大多数情况下不做任何事情，直到有脚本、处理程序、事件来激活它, 这里的脚本、处理程序、事件都统称为任务，当任务到来时，如果 javascript 本身在执行其他任务，就会把这个新的任务放到 evet loop 中，直到当前任务执行完毕。  

这样我们可以尝试初步理解开头那个场景会发生什么：
 1. Javascript 执行当前脚本
 2. setTimeout 触发了一个新的事件，这个事件到来时，JS 引擎被当前脚本占据，`console.log("timeout");` 被放到 eventloop 中，脚本继续往下执行
 3. Promise 抛出一个新的任务，`console.log("promise");` 也被放到 eventloop 中（这里的 eventloop 与 2 中的其实并不相同，我们后文在讲）
 4. 运行 `console.log("main");`, `"main"` 被打印出来
 5. 这时当前脚本执行完毕，JS 引擎闲置，则会执行 eventloop 中的任务 `console.log("timeout");` 与 `console.log("promise");`

通过上面的步骤，似乎是 `timeout` 会比 `promise` 先打印？但是这并不与实际 JS 中的运行结果一致，发生了什么呢？ 

这里就涉及到了我们 3 中提到的 eventloop 与 2 中不一致的问题，javascript 中对于 eventloop 或者说任务队列，其实分为两种，  宏任务 (macrotask) 微任务(microtask).
- 宏任务  
   对于常见的事件，例如：setTimeout, ajax 请求， 浏览器事件，都会被放到宏任务中
- 微任务  
   微任务只能有程序创建，例如 promise, async/await(本质上也是 promise), 还有一种特殊的函数 `queueMicrotask(func)` 可以对 `func` 排队使其在 microtask 队列中运行  

**微任务总是优先于宏任务执行**。  

这样我们就可以继续之前的步骤  

 1. Javascript 执行当前脚本
 2. setTimeout 触发了一个新的事件函数，这个事件到来时，JS 引擎被当前脚本占据，`console.log("timeout");` 被放到 宏任务队列 中，脚本继续往下执行
 3. Promise 抛出一个新的任务，`console.log("promise");` 也被放到 微任务队列 中
 4. 运行 `console.log("main");`, `"main"` 被打印出来
 5. 这时当前脚本执行完毕，JS 引擎闲置，由于微任务队列非空，优先执行微任务 `console.log("promise");`, `"promise"` 被打印出来
 6. 这是微任务队列为空，JS 引擎去执行 宏任务队列中的任务 `console.log("timeout");`， `"timeout"` 被打印出来


OK, 到目前，我们搞清楚了setTimeout  与 Promise 的执行顺序是由宏任务与微任务队列的优先级决定的，那么我们在其中考虑一个新的场景
```
document.body.style = 'background:red';

setTimeout(()=> {
    document.body.style = 'background:black';
})

Promise.resolve().then(()=> {
    document.body.style = 'background:white';
})
```
请问目前页面的颜色改变了几次？

要回答这个问题，我们比如引入一个新的知识点**页面渲染**， 我们知道，当 JS 引擎被阻塞是，页面渲染是会被暂停的，只有 JS 引擎空闲时，页面才会完成渲染，这通常也是我们页面优化的一个方面：尽量避免 JS 引擎被长时间占据，这回导致页面是去反应。
> 实际上页面渲染是GUI渲染线程来控制的，二这个线程与 JS 引擎线程互斥，导致了这种现象。  
> 除此之外，浏览器里的线程一共有一下几种  
    - 扩展GUI渲染线程  
    - JS引擎线程   
    - 事件触发线程  
    - 定时触发器线程    
    - 异步http请求线程  

在我们目前的这个例子里, 这里出现了三个任务：
 - 主程序 - macrotask
 - timeout 任务 - macrotask
 - promise 任务 - microtask
通过之前第一个问题，我们知道这三个任务的执行顺序是： 主程序 -> promise 任务 -> timeout 任务  

我们在这里需要判断 UI 渲染会发生在哪一个阶段，实际上，UI 渲染也是一个 macrotask, 所以 UI 渲染会发生在 promise 任务之后，接下来要判断的问题是 UI 渲染任务与 timeout 任务的顺序问题， 这里按照标准 预期的顺序应该是 `'background:white'` 开始 rendering ,然后 setTimeout , `'background:black'` 开始 rendering, 但是实际如果再浏览器中测试，可能看不到 `'background:white'` 的 rendering 过程。

> 参考:  
> - *[事件循环：微任务和宏任务](https://zh.javascript.info/event-loop)*  
> - *[微任务（Microtask）](https://zh.javascript.info/microtask-queue)*  
> - *[MDN - EventLoop](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop)*
> - *[stackoverflow-why-is-settimeoutfn-0-sometimes-useful](https://stackoverflow.com/questions/779379/why-is-settimeoutfn-0-sometimes-useful)*
> - *[为什么js会阻塞页面渲染？](https://lqqjohnny.github.io/2018/07/24/%E4%B8%BA%E4%BB%80%E4%B9%88js%E4%BC%9A%E9%98%BB%E5%A1%9E%E9%A1%B5%E9%9D%A2%E6%B8%B2%E6%9F%93%EF%BC%9F/)*