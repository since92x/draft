```javascript
function _new(ctor, ...args) {
  const newObj = Object.create(ctor);
  const ctorResult = ctor.apply(newObj, args);
  const isObject =
    typeof ctorReturnResult === 'object' && ctorReturnResult !== null;
  const isFunction = typeof ctorReturnResult === 'function';
  if (isObject || isFunction) {
    return ctorReturnResult;
  }
  return newObj;
}
```
