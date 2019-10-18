# bind

```javascript
Function.prototype.bind = function(ctx, ...args) {
  const fn = this;
  return function(...params) {
    return fn.apply(ctx, [...args, ...params]);
  };
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
