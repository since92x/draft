# bind

```javascript
Function.prototype.bind = function(ctx, ...args) {
  const fn = this;
  function bindedFunc(...params) {
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

```javascript
Function.prototype.call = function(ctx, ...args) {
  const fn = this;
};
```

# apply

```javascript
Function.prototype.call = function(ctx, ...args) {
  const fn = this;
};
```

# new

```javascript
function _new() {}
```
