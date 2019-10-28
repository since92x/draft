# 前提

```javascript
function Parent(name) {
  this.name = name || 'User';
  this.colors = ['red', 'green'];
}
Parent.prototype.sayColors = function() {
  alert(this.colors);
};
```

# 原型链继承

```javascript
function Child() {}
Child.prototype = new Parent(); // p1.colors === p2.colors // true
```

缺点：

- `引用类型的属性被所有实例共享`

# 借用构造函数

```javascript
function Child() {
  Parent.call(this); // 每个实例都有自己的colors了
}
```

缺点：

- 只能继承父类的实例属性和方法，`不能继承原型属性/方法`
- 无法实现复用，每个子类都有父类实例函数的副本，影响性能
- 方法都在构造函数中定义，每次创建实例都会创建一遍方法

# 组合继承

```javascript
function Child(name, age) {
  Parent.call(this, name); // 实例属性的继承
  this.age = age;
}
Child.prototype = new Parent(); // 原型属性和方法的继承
Child.prototype.constructor = Child;
Child.prototype.sayAge = function() {
  alert(this.age);
}
```

缺点：

- 第一次调用Parent()：给Child.prototype写入两个属性name，color
- 第二次调用Parent()：给实例写入两个属性name，color

于是，`实例 上的属性 屏蔽了 原型Child.prototype上的同名属性`。

# 原型式继承

```javascript
function create(obj) { // 浅复制
  function F() {};
  F.prototype = obj; // 浅复制
  return new F();  
}
```
缺点: 

- 可以在不必预先定义构造函数的情况下实现继承，其本质是执行对给定对象的浅复制

于是，缺点还是`包含引用类型的属性都会被共享`，且无法传参数

# 寄生式继承

在原型式继承的基础上, 增强实例对象

```javascript
function createAnother(o) {
  var clone = create(o);
  o.sayHi = function () { // 以某种方式，来增强对象
    alert('hi');
  }
  return clone;
}
```
缺点：

- `同原型式继承`

# 寄生组合继承

```javascript
function inheritPrototype(Child, Parent) { // 同寄生式继承
  var prototype = Object.create(Parent.prototype);
  prototype.constructor = Child
  Child.prototype = prototype
}
function Child(name, age) { // 组合式继承
  Parent.call(this, name);
  this.age = age;
}
inheritPrototype(Child, Parent); // 子类原型重写为父类原型
Child.prototype.sayAge = function () { // 对子类添加自己的方法
  alert(this.age);
};

```

这是最成熟的方法，也是现在库实现的方法

# 



[JavaScript 深入之继承的多种方式和优缺点](https://github.com/mqyqingfeng/Blog/issues/16)
[JavaScript常用八种继承方案](https://juejin.im/post/5bcb2e295188255c55472db0#comment)

