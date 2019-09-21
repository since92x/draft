```javascript
function throttle(fn, wait = 500) {
  let prev = 0,
    timer = null;
  return function(...args) {
    let now = Date.now();
    if (now - prev < wait) {
      clearTimeout(timer);
      timer = setTimeout(() => {
        prev = now;
        fn.apply(this, args);
      }, wait);
    } else {
      prev = now;
      fn.apply(this, args);
    }
  };
}
```
