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

缺点：

- 引用类型的属性被所有实例共享 (所有实例共享 girl friends ???)

# 借用构造函数

```javascript
function Child() {
  Parent.call(this);
}
```

缺点：

- 只能继承父类的实例属性和方法，不能继承原型属性/方法
- 无法实现复用，每个子类都有父类实例函数的副本，影响性能
- 方法都在构造函数中定义，每次创建实例都会创建一遍方法

# 组合继承

```javascript
function Child(name, age) {
  Parent.call(this, name);
  this.age = age;
}
Child.prototype.constructor = Child;
```

缺点：

- 1

[JavaScript 深入之继承的多种方式和优缺点](https://github.com/mqyqingfeng/Blog/issues/16)
[JavaScript常用八种继承方案](https://juejin.im/post/5bcb2e295188255c55472db0#comment)

