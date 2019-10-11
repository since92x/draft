# 前提

```javascript
function Parent(name) {
  this.name = name;
}
Parent.prototype.getName = function() {
  alert(this.name);
};
```

# 原型链继承

```javascript
function Child() {}
Child.prototype = new Parent();
```

> 痛点：
