实现数组 map 方法

```javascript
Array.prototype.map = function(fn, context) {
  const result = [];
  for (let i in this) {
    result.push(fn.call(context, this[i], i, this));
  }
  return result;
};
```
