# 前提

```javascript
function Parent(name) {
  this.name = name || '渣男';
  this.girlFriends = ['girl1', 'girl2'];
}
Parent.prototype.hello = function() {
  alert(this.girlFriends);
};
```

# 原型链继承

```javascript
function Child() {}
Child.prototype = new Parent();
```

问题：

- 引用类型的属性被所有实例共享 (所有实例共享 girl friends ???)

# 借用构造函数

```javascript
function Child() {
  Parent.call(this);
}
```

问题：

- 方法都在构造函数中定义，每次创建实例都会创建一遍方法

# 组合继承

```javascript
```

问题：

- 1

[JavaScript 深入之继承的多种方式和优缺点](https://github.com/mqyqingfeng/Blog/issues/16)
