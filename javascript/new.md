# javascript构造函数和new

## 构造函数

构造函数在技术上是常规函数。不过有两个约定：

* 通常首字母大写
* 使用new操作符执行

eg: 

```javascript
  function User(name) {
    this.name = name;
    this.isAdmin = false;
  }
  let user = new User("Jack");
```
当构造函数用于new操作符时，执行了如下步骤：

* 创建一个empy plain object，并分配给this;
* 函数体执行。通常它会修改 this，为其添加新的属性;
* 若没有返回对象，则返回this。

换句话说，new User(...) 做类似的事情：

```javascript
function User(name) {
  // this = {};（隐式创建）
  // 添加属性到 this
  this.name = name;
  this.isAdmin = false;
  // return this;（隐式返回, 不返回对象则返回this）
}
```
效果类似于 ```let user = { name: "Jack", isAdmin: false };```
但多次创建时，不及构造函数方便

## new.target

检测函数或构造方法是否是通过new运算符被调用。在通过new运算符被初始化的函数或构造方法中，new.target返回一个指向构造方法或函数的引用。在普通的函数调用中，new.target 的值是undefined。

```javascript
function User() {
  alert(new.target);
}
// 不带 new：
User(); // undefined
// 带 new：
new User(); // function User { ... }
```

## 构造函数的return

* 若返回对象，则不返回this；
* 若返回primitive类型，则忽略。

## 实现new操作符 

```javascript
function _new (ctor, ...args) {
    if(typeof ctor !== 'function'){
      throw 'First param must be a function';
    }
    newOperator.target = ctor;
    const newObj = Object.create(ctor.prototype);
    // let newObj = {}; Object.setPrototypeOf(newObj, Con.prototype); // 也可以这样做
    const ctorReturnResult = ctor.apply(newObj, args);
    const isObject = typeof ctorReturnResult === 'object' && ctorReturnResult !== null;
    const isFunction = typeof ctorReturnResult === 'function';
    if(isObject || isFunction){
        return ctorReturnResult;
    }
    return newObj;
}
```

## 参考

[Constructor, operator "new"](https://javascript.info/constructor-new)

[重学 JS 系列：聊聊 new 操作符](https://juejin.im/post/5c7b963ae51d453eb173896e)
