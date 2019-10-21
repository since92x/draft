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
class Draggable {
  constructor(el) {
    this.el = el;
    this.state = {
      isdragging: false,
      position: { x: '', y: '' },
      prevTouch: { clientX: '', clientY: '' },
    };
    this.init();
  }
  checkPosition = () => {
    const position = window
      .getComputedStyle(this.el)
      .getPropertyValue('position');
    if (position !== 'fixed') {
      console.warn(
        `The position of the target element must be fixed or absolute, but it is ${position}, so the program automatically modifies it to 'fixed'`
      );
      this.el.style.position = 'fixed';
    }
  };
  init = () => {
    this.checkPosition();
    this.addListener(this.el, ['mousedown', 'touchstart'], this.handleDown);
    this.addListener(this.el, ['mousemove', 'touchmove'], this.handleMove);
    this.addListener(this.el, ['mouseup', 'touchend'], this.handleEnd);
  };
  addListener = (element, eventTypes, handler, useCapture = false) => {
    eventTypes.forEach(eventType =>
      element.addEventListener(eventType, handler, useCapture)
    );
  };
  checkAxisBounds = (start, len, max, theta = 0) => {
    // theta 为容错阀值
    const minBound = start >= 0 - theta; // 是否超出最低边界
    const maxBound = start + len <= max + theta; // 是否超出最高边界
    return [minBound && maxBound, minBound];
  };
  checkBounds = () => {
    const { clientWidth, clientHeight } = document.documentElement;
    const { x, y } = this.el.getBoundingClientRect();
    const checkXAxis = this.checkAxisBounds(
      x,
      this.el.clientWidth,
      clientWidth
    );
    const checkYAxis = this.checkAxisBounds(
      y,
      this.el.clientHeight,
      clientHeight
    );
    if (!checkXAxis[0]) {
      this.el.style.left = checkXAxis[1]
        ? `${clientWidth - this.el.clientWidth}px`
        : 0;
    }
    if (!checkYAxis[0]) {
      this.el.style.top = checkYAxis[1]
        ? `${clientHeight - this.el.clientHeight}px`
        : 0;
    }
  };
  updatePosition = e => {
    if (this.state.isdragging) {
      let touch = e.touches ? e.touches[0] : e;
      this.el.style.left = `${this.state.position.x +
        touch.clientX -
        this.state.prevTouch.clientX}px`;
      this.el.style.top = `${this.state.position.y +
        touch.clientY -
        this.state.prevTouch.clientY}px`;
    }
  };
  handleDown = e => {
    this.state.isdragging = true;
    let touch = e.touches ? e.touches[0] : e;
    this.state.prevTouch.clientX = touch.clientX;
    this.state.prevTouch.clientY = touch.clientY;
    this.state.position.x = this.el.offsetLeft;
    this.state.position.y = this.el.offsetTop;
  };
  handleMove = e => {
    e.preventDefault();
    let scheduledAnimationFrame = false;
    if (scheduledAnimationFrame) return;
    scheduledAnimationFrame = true;
    window.requestAnimationFrame(() => {
      scheduledAnimationFrame = false;
      this.updatePosition(e);
    });
  };
  handleEnd = () => {
    this.state.isdragging = false;
    this.checkBounds();
  };
}
Vue.directive('drag', {
  drag: {
    inserted (el) {
      new Draggable(el);
    }
  }
})
```