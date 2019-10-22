# bind

> `bind` 借助`apply/call`实现修改`this`指向; 另外，绑定后的函数可作为`构造函数`,此时传入的上下文被忽略

```javascript
Function.prototype.bind = function(ctx, ...args) {
  const fn = this;
  function bindedFunc(...params) {
    // 作为构造函数时，this 指向实例，instanceof 结果为 true，将绑定函数的 this 指向该实例，可以让实例获得来自绑定函数
    const context = this instanceof bindedFunc ? this : ctx;
    return fn.apply(context, [...args, ...params]);
  }
  // bindedFunc.prototype = Object.create(fn.prototype); // 防止fn.prototype被修改, 同下
  function fnProto() {}
  fnProto.prototype = fn.prototype;
  bindedFunc.prototype = new fnProto();
  return bindedFunc;
};
```

# call

> 借助`obj = { v: 1, f() { alert(this.v) } }; obj.f();`原理实现

```javascript
Function.prototype.call = function(ctx, ...args) {
  const context = [null, undefined].includes(ctx) ? window : Object(ctx);
  context.fn = this;
  const result = context.fn(...args);
  delete context.fn;
  return result;
};
```

# apply

> 和 `call` 差不多

```javascript
Function.prototype.apply = function(ctx, arr = []) {
  const context = [null, undefined].includes(ctx) ? window : Object(ctx);
  context.fn = this;
  const result = context.fn(...arr);
  delete context.fn;
  return result;
};
```

# new

> 本着 `newObj.__proto__ = Constructor.prototype` 的原则

```javascript
function _new(Constructor, ...args) {
  const newObj = Object.create(Constructor.prototype);
  const cotrReturn = Constructor.apply(newObj, args);
  const isFun = typeof cotrReturn === 'function';
  const isObj = typeof cotrReturn === 'object' && o !== null;
  if (isFun || isObj) {
    return cotrReturn;
  }
  return newObj;
}
```

# 参考

[JavaScript 深入之 bind 的模拟实现](https://github.com/mqyqingfeng/Blog/issues/12)
[JavaScript 深入之 call 和 apply 的模拟实现](https://github.com/mqyqingfeng/Blog/issues/11)
[JavaScript 深入之 new 的模拟实现](https://github.com/mqyqingfeng/Blog/issues/13)
