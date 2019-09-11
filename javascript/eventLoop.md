# Event Loop

首先我们来看看whatwg说了啥～ 

## 定义

**Event Loop**

> 用户代理使用 Event Loop 来协调事件，用户交互，脚本，渲染，网络等。 每个代理都有一个关联的 Event Loop。

**Task Queue**

> 任务队列是集合（set），而不是队列（queue），因为 Event Loop 处理模型的第一步是从所选队列中获取第一个可运行任务，而不是使第一个任务出队（dequeue）。

> Microtask Queue 不是 Task Queue。

**Event Loop流程**

先来一段伪代码：

```javascript
eventLoop = {
    taskQueues: {
        events: [], // UI events from native GUI framework
        parser: [], // HTML parser
        callbacks: [], // setTimeout, requestIdleTask
        resources: [], // image loading
        domManipulation: []
    },
    microtaskQueue: [],
    nextTask () {
        for (let q of taskQueues)
          if (q.length > 0)
            return q.shift();
        return null;
    },
    executeMicrotasks () {
        if (scriptExecuting)
          return;
        let microtasks = this.microtaskQueue;
        this.microtaskQueue = [];
        for (let t of microtasks) {
          t.execute();
        }
    },
    needsRendering () {
        return vSyncTime() && (needsDomRerender() || hasEventLoopEventsToDispatch());
    },
    render () {
        dispatchPendingUIEvents();
        resizeSteps();
        scrollSteps();
        mediaQuerySteps();
        cssAnimationSteps();
        fullscreenRenderingSteps();
        animationFrameCallbackSteps();
        while (resizeObserverSteps()) {
          updateStyle();
          updateLayout();
        }
        intersectionObserverObserves();
        paint();
    }
}

while(true) {
    task = eventLoop.nextTask();
    if (task) {
        task.execute();
    }
    eventLoop.executeMicrotasks();
    if (eventLoop.needsRendering())
        eventLoop.render();
}
```

<img src="https://user-gold-cdn.xitu.io/2019/8/23/16cbd1e3fc63e5e1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1" style="background:#fff;padding:5px;"/>

Event Loop中的关键点:

* 执行Macrotasks队列中的oldestTask
* 执行Microtasks队列里的所有任务
* 更新渲染


### Macrotask

**任务源**:

DOM 操作任务源：如元素以非阻塞方式插入文档

用户交互任务源：如鼠标键盘事件。用户输入事件（如 click） 必须使用 task 队列

网络任务源：如 XHR 回调

history 回溯任务源：使用 history.back() 或者类似 API

此外 setTimeout、setInterval、IndexDB 数据库操作等也是任务源。总结来说，常见的 task 任务有：

* script（js代码作为一个整体在script tag中也是一个macrotask)
* Using a resource（获取资源）
* Reacting to DOM manipulation（dom操作如元素插入文档时）
* 事件回调（eg: click, mouseover）
* 网络请求回调（eg: XHR回调）
* setTimeout / setInterval
* I/O（eg: IndexDB, 文件读写）
* history API(eg: back)
* postMessage
* requestAnimationFrame
* UI rendering
* setImmediate

需要注意的是：setTimeout 不会自动的把回调放到事件循环队列中。它设置了一个定时器，当定时器过期了，宿主环境会将回调放到事件循环队列中，以便在以后的循环中取走执行它。然而，这个队列中可能会拥有一些早一点添加进来的事件 — 回调将会等待被执行。这就是我们常说的 setTimeout 不准时的根本原因！

### Microtask

**任务源**:

* Promise callback, async/await
* MutationObserver
* Object.observe(已废弃)
* queueMicrotask
* process.nextTick

我们平时使用最多的Promise，async/await就是microtask。[Promise规范](https://promisesaplus.com/#point-73)中指出Promise的具体实现由平台把握,可以是microtask或macrotask。

### 延伸阅读 async / await

这里说一说async / await。

> 异步函数的真正威力来自 await 表达式，它会暂停函数的执行直到 Promise 状态变为 resolved，并在执行后恢复。await 的值是 Promise 被 fulfilled 的值。表现为await语句执行完之后，会把await语句后面的全部代码加入到 Microtask 队列。

> 就是说：await后面的表达式会先执行一遍，将await后面的代码加入到microtask中，然后就会跳出整个async函数来执行后面的代码。稍后在 await表达式返回的 Promise 状态变为 fulfilled 时恢复了执行。这或多或少等同于将把处理过程写在 await 表达式返回 Promise 的 then 链中。

此外，await 可以使用任何 “thenable”，即任何部署了then方法的对象。基于此特性，我们可以测量setTimeout真正的用时：
```javascript
Class Sleep {
  constructor(ms) {
    this.ms = ms;
  }
  then(resolve, reject) {
    const startTime = Date.now();
    setTimeout(() => resolve(Date.now - startTime), ms);
  }
}

(async () => {
  const actualTime = await new Sleep(1000);
  console.log(actualTime);
})();
```

## microtask执行时机

首先我们解释一下“perform a microtask checkpoint”，伪代码看起来像:
```javascript
let MicrotaskQueue = function(eventLoop) {
    this.eventLoop = eventLoop;
}
MicrotaskQueue.prototype = {
    microtasks: [],
    queueTask: task => {
        this.microtasks.push(task);
    },
    getOldestTask: () => {
        return this.microtasks.shift();
    },
    performCheckpoint: () => {
        if (this.performing)
            return;
        this.performing = true;
        while (let task = this.getOldestTask()) {
            this.eventLoop.currentlyRunningTask = task;
            task.run();
            this.eventLoop.currentlyRunningTask = null;
        }
        // 每一个environment settings object它们的 responsible event loop就是当前的event loop，会给environment settings object发一个 rejected promises 的通知。
        // 清理IndexedDB的事务。
        // 将microtask checkpoint的flag设为flase。
    }
}
```

可见，"perform a microtask checkpoint"所做的就是执行microtask队列里的任务。什么时候会调用microtask checkpoint呢?

在html规范中，我们可以看到很多处写到“perform a microtask checkpoint”，其中包括：

* Calling scripts - Clean up after running script (whatwg 规范)
* Calling scripts - clean up after running a callback (w3c 规范)
* Creating and inserting nodes
* Parsing XML documents
* ...

在whatwg中有规定，在 Calling scripts 的清理阶段，如果 javascript 执行栈为空，也会 perform a microtask checkpoint，即执行 microtask。

**在一个 Event Loop 的 turn 内，并不是把所有同步的任务先全部执行完（如：代码中许多的console.log），再去执行 microtasks；如果执行完一个同步task（如：一个console.log），此时 Execution Context Stack 为空，则会perform a microtask checkpoint（即执行microtask）**

这里我们来看一下ECMA262中对 Job 的描述：

> Execution of a Job can be initiated only when there is no running execution context and the execution context stack is empty.

可见，Job 和 Microtask 很相似。

执行microtask的时机可以总结为：
* 当执行上下文栈(ECS)为空时，perform a microtask checkpoint。
* 在event loop的第八步（Microtasks: Perform a microtask checkpoint），也就是在运行task之后，更新UI-rendering之前。

**我们来看一个ECS为空时的栗子🌰：**

```html
<div class="outer">
  <div class="inner"></div>
</div>
<script>
window.onLoad = function () {
  let outer = document.querySelector('.outer')
  let inner = document.querySelector('.inner')
  function onClick() {
    console.log('click!')
    setTimeout(() => {
      console.log('setTimeout!')
    }, 0)
    Promise.resolve().then(() => {
      console.log('promise!')
    })
  }
  outer.addEventListener('click', onClick)
  inner.addEventListener('click', onClick)
  //inner.click() 第二个测试中，通过程序出发点击
}
</script>
```
* 首先我们点击内部div，让浏览器去派发 click 事件。

<img src="https://github.com/since92x/draft/blob/master/imgs/eventloop-4.png?raw=true" style="background:#fff;"/>

答案显而易见：
```
click!
promise!
click!
promise!
setTimeout!
setTimeout!
```

* 然后我们通过 Javascript 触发 click 事件($inner.click())。猜猜输出结果有什么不同？

<img src="https://github.com/since92x/draft/blob/master/imgs/eventloop-5.png?raw=true" style="background:#fff;"/>

不难看出答案是：
```
click!
click!
promise!
promise!
setTimeout!
setTimeout!
```
## summary

* Event Loop 由三个阶段构成：执行一个macrotask -> 执行microtask队列 -> UI-rendering阶段

* Microtask 中注册的 Microtask 会直接加入到当前turn 的 Microtask 队列（**现做现卖**）。

* Microtask执行时机： (1) ECS为空时会执行;(2)一轮 eventloop 中task之后和UI-rendering之前。一轮Event Loop可能执行多次 microtask。

## 习题巩固

1. 请回答下面代码的运行结果
```html
<div class="outer">
    <div class="inner"></div>
</div>
<script type="text/javascript">
    window.onload = function() {
        let cnt = 0;
        let outer = document.querySelector('.outer');
        let inner = document.querySelector('.inner');
        new MutationObserver(function([record]) {
            console.log('mutate', record.target.dataset.random);
        }).observe(outer, { attributes: true });
        function onClick() {
            console.log('click');
            setTimeout(function() {
                console.log('timeout');
            }, 0);
            Promise.resolve().then(function() {
                console.log('promise');
            });
            outer.setAttribute('data-random', ++cnt);
        }
        inner.addEventListener('click', onClick);
        outer.addEventListener('click', onClick);
    }
  //inner.click() 第二个测试中，通过程序出发点击
</script>
```
(1) 上面的代码当点击div.inner时，控制台输出什么？

(2) 使用inner.click()又是怎样的结果呢？

2. 写出下面程序按ES规范标准的运行结果是？

```javascript
const p = Promise.resolve();
(async () => {
  await p; console.log('after:await');
})();
p.then(() => console.log('tick:a'))
 .then(() => console.log('tick:b'));
```

3. 写出下面程序的运行结果：
```javascript
const promise = new Promise((resolve, reject) => {
  console.log('From promise object')
  resolve('from promise')
})
const ts = async () => {
  console.log('From async function')
  const a = await promise
  console.log(a)
}
ts()
console.log('event loop end')
setTimeout(() => {
  console.log('From setTimeout')
}, 0)
```
4. 写出下面程序的运行结果：
```javascript
async function async1() {
    console.log('async1 start');
    await async2();
    console.log('async1 end');
}
async function async2() {
  //async2做出如下更改：
  new Promise(function(resolve) {
    console.log('promise1');
    resolve();
  }).then(function() {
    console.log('promise2');
  });
}
console.log('script start');
setTimeout(function() {
    console.log('setTimeout');
}, 0)
async1();
new Promise(function(resolve) {
  console.log('promise3');
  resolve();
}).then(function() {
  console.log('promise4');
});
console.log('script end');
```
就题而论的话，这种题我一般就按顺序写，遇到宏/微任务就用括号括号起来,同步代码写完后再从括号中取微/宏任务：
```
script start 
(macrotask: setTimeout)
async1 start 
promise1 
(microtask: promise2, async1 end, promise4)
promise3
script end
promise2
async1 end 
promise4
settimeout
```


## 参考

[Event loops](https://html.spec.whatwg.org/multipage/webappapis.html#event-loops)

[event-loop](https://github.com/atotic/event-loop)

[深入探究 eventloop 与浏览器渲染的时序问题](https://www.404forest.com/2017/07/18/how-javascript-actually-works-eventloop-and-uirendering/)

[从event loop规范探究javaScript异步及浏览器更新渲染时机](https://segmentfault.com/a/1190000009271765)

[异步笔试题](https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/7)

[更快的异步函数和 Promise](https://v8.js.cn/blog/fast-async/)
