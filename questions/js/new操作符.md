```javascript
function _new(Constructor, ...args) {
  const newObj = Object.create(Constructor.prototype);
  const cotrReturn = Constructor.apply(newObj, args);
  const isFun = typeof cotrReturn === 'function';
  const isObj = typeof cotrReturn === 'object' && cotrReturn !== null;
  if (isFun || isObj) {
    return cotrReturn;
  }
  return newObj;
}
```
