# DOM事件三个阶段
> * 捕获阶段 —— 事件（从 Window）向下走近元素。
>
> * 目标阶段 —— 事件到达目标元素。
>
> * 冒泡阶段 —— 事件从元素上开始冒泡。
> 
# event 对象的属性

> * `event.target` —— 是引发事件的“目标”元素，它在冒泡过程中不会发生变化
>
> * `event.currentTarget（=this)` —— 是“当前”元素，其中有一个`当前正在运行的处理程序`。
>
> * `event.eventPhase` —— 当前阶段（capturing=1，target=2，bubbling=3）

# 冒泡和捕获

## 冒泡

> * 当一个事件发生在一个元素上，它会首先运行在该元素上的处理程序，然后运行其父元素上的处理程序，然后一直向上到其他祖先上的处理程序。
>
> * 几乎 所有事件都会冒泡, 例如: `focus`就不会
> 
> * `event.stopPropagation()`停止冒泡
>
> * `event.stopImmediatePropagation()`停止冒泡，并阻止当前元素上的剩余处理程序运行
>
> 例如：我们试图通过`document.addEventListener('click', listener)`进行分析整个页面的点击行为，但是又一个区域使用了`event.stopPropagation()`，会产生一个死区。一项看似需要阻止冒泡的任务，可以通过其他方法解决。其中之一就是使用自定义事件。还可以通过`event.defaultPrevented`进行过滤解决


# target.addEventListener(type, listener, options / useCapture);

## passive

> `{ passive: true }` 向浏览器发出信号，表明处理程序将不会调用 preventDefault()。然后，浏览器会立即执行默认行为，保证最大的流畅度。如果 listener 仍然调用了preventDefault()，客户端将会忽略它并抛出一个控制台警告。
>
> 移动端touchmove事件可以通过`preventDefault()`阻止滚动。
>
> 浏览器只有等内核线程执行到事件监听器对应的JavaScript代码时，才能知道内部是否会调用`preventDefault`函数来阻止事件的默认行为，所以浏览器本身无法优化这种场景。手势输入事件是由连续的普通输入事件组成的，在这种场景下，无法快速产生，会导致页面无法快速执行滑动逻辑，从而让用户感觉到页面卡顿。
>
> 对于某些浏览器（Firefox，Chrome），默认情况下，touchstart 和 touchmove 事件的 passive 为 true。

## capture

> 如果为 false（默认值），则在冒泡阶段设置处理程序。
>
> 如果为 true，则在捕获阶段设置处理程序。


# 要点

* `passive: true` 告诉浏览器该行为不会被阻止。这对于某些移动端的事件（像 touchstart 和 touchmove）很有用，用以告诉浏览器在`滚动之前不应等待所有处理程序完成`
* 若调用了`preventDefault()`阻止默认行为，`event.defaultPrevented` 的值会变成 true，否则为 false。

# 参考

[passive 的事件监听器](https://www.cnblogs.com/ziyunfei/p/5545439.html)
