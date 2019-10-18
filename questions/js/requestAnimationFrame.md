# requestAnimationFrame()

背景：屏幕刷新率 60HZ（即每 16.7ms 刷新一次屏幕）， 定时器 timer 每 10ms 移动元素向右 1px（元素初始 style.right=0px）

步骤：

- 第 0ms： 屏幕不刷新，timer 不执行
- 第 10ms： 屏幕不刷新，timer 执行，增加元素的 right = 1px
- 第 16.7ms： 屏幕刷新，元素向右移动到 1px，timer 不执行
- 第 20ms： 屏幕不刷新，timer 执行，right = 2px
- 第 30ms：屏幕不刷新，timer 执行，right = 3px
- 第 33.4ms： 屏幕刷新，元素向右移动到 3px，timer 不执行
- ...

可见，屏幕并没有出现过 right=2px 那一帧，直接从 1px 到 3px，这就是丢帧，会造成卡顿

> requestionAnimationFrame **由系统决定回调函数的执行时机**， 它能保证回调函数在屏幕每一次的刷新间隔中只被执行一次，这样就不会引起丢帧现象，也不会导致动画出现卡顿的问题。

优点：

- CPU 节能: 页面隐藏时候，setTimeout 仍然执行，requestAnimationFrame 会被暂停，页面被激活时恢复
- 函数节流: 保证刷新间隔内，如：16.7ms 内只执行一次 resize / scroll

```javascript
let progress = 0;
let requestId = null;

function render() {
  progress += 1;
  el.style.right = progress + 'px';
  if (progress < 100) {
    requestId = requestAnimationFrame(render);
  }
}

requestId = requestAnimationFrame(render);

cancelAnimationFrame(requestId);
```

兼容:

```javascript
window.requestAnimFrame = window.requestAnimationFrame || function(callback) {
  window.setTimeout(callback, 1000 / 60);
};
```

vue 可拖动指令, 其中处理移动移动事件时，使用`requestAnimationFrame`
```javascript
Vue.directive('drag', {
  drag: {
    inserted (el) {
      let prevTouch = { clientX: 0, clientY: 0 }
      let self = { x: '', y: '' }
      el.isdragging = false
      function addListener (element, eventTypes, handler, useCapture = false) {
        eventTypes.forEach(eventType => element.addEventListener(eventType, handler, useCapture))
      }
      function checkAxisBounds (start, len, max, theta = 50) { // theta 为容错阀值
        const minBound = (start >= (0 - theta)) // 是否超出最低边界
        const maxBound = (start + len) <= (max + theta) // 是否超出最高边界
        return [minBound && maxBound, minBound]
      }
      function checkBounds () {
        const { clientWidth, clientHeight } = document.documentElement
        const { x, y } = el.getBoundingClientRect()
        const checkXAxis = checkAxisBounds(x, el.clientWidth, clientWidth)
        const checkYAxis = checkAxisBounds(y, el.clientHeight, clientHeight)
        if (!checkXAxis[0]) {
          el.style.left = checkXAxis[1] ? `${clientWidth - el.clientWidth}px` : 0
        }
        if (!checkYAxis[0]) {
          el.style.top = checkYAxis[1] ? `${clientHeight - el.clientHeight}px` : 0
        }
      }
      function updatePosition (e) {
        if (el.isdragging) {
          let touch = e.touches ? e.touches[0] : e
          el.style.left = `${self.x + touch.clientX - prevTouch.clientX}px`
          el.style.top = `${self.y + touch.clientY - prevTouch.clientY}px`
        }
      }
      function handleDown (e) {
        el.isdragging = true
        let touch = e.touches ? e.touches[0] : e
        prevTouch.clientX = touch.clientX
        prevTouch.clientY = touch.clientY
        self.x = el.offsetLeft
        self.y = el.offsetTop
      }
      function handleMove (e) {
        e.preventDefault()
        let scheduledAnimationFrame = false
        if (scheduledAnimationFrame) return
        scheduledAnimationFrame = true
        window.requestAnimationFrame(() => {
          scheduledAnimationFrame = false
          updatePosition(e)
        })
      }
      function handleEnd () {
        el.isdragging = false
        checkBounds()
      }
      addListener(el, ['mousedown', 'touchstart'], handleDown)
      addListener(el, ['mousemove', 'touchmove'], handleMove)
      addListener(el, ['mouseup', 'touchend'], handleEnd)
    }
  }
})
```