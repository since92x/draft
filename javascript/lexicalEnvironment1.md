# javascript Lexical Environment 简述

## 问题引入

1. 函数 sayHi 用到 name 这个外部变量 。当函数执行时，它会使用哪一个值呢？

```javascript
let name = "John";
function sayHi() {
  alert("Hi, " + name);
}
name = "Pete";
sayHi(); // 显示 "John" 还是 "Pete" ？
```

2. 函数 makeWorker 生成并返回另一个函数。这个新的函数可以在其他位置进行调用。它访问的是创建位置还是调用位置的外部变量呢，还是都可以？

```javascript
function makeWorker() {
  let name = "Pete";
  return function() {
    alert(name);
  };
}
let name = "John";
// 创建一个函数
let work = makeWorker();
// call it
work(); // 它显示的是什么呢？"Pete"（创建位置的 name）还是"John"（调用位置的 name）呢？
```

## Lexical Environment 

要了解究竟发生了什么，首先我们要来讨论一下『变量』究竟是什么？

在 JavaScript 中，每个运行的函数、代码块或整个程序，都有一个称为 **词法环境（Lexical Environment）** 的关联对象。

词法环境对象由两部分组成：

* **环境记录（Environment Record）** —— 一个把所有局部变量作为其属性（包括一些额外信息，比如 this 值）的对象
* **外部词法环境（outer lexical environment）** 的引用 —— 通常是嵌套当前代码（当前花括号之外）之外代码的词法环境。

> 所以，『变量』只是环境记录这个特殊内部对象的属性。『访问或修改变量』意味着『访问或改变词法环境的一个属性』。<br />
举个例子，这段简单的代码中只有一个词法环境：

<img src="https://javascript.info/article/closure/lexical-environment-global.svg" style="background:#fff;"/><br />

> 在上图中，**矩形表示环境记录（存放变量），箭头表示外部（词法环境）引用**。全局词法环境没有外部（词法环境）引用，所以它指向了 null。

> 这是关于 let 变量（如何变化）的全程展示：

<img src="https://javascript.info/article/closure/lexical-environment-global-2.svg" style="background:#fff;"/><br />

> 右侧的矩形演示在执行期间全局词法环境是如何变化的：<br />1. 执行开始时，词法环境为空。<br />2. let phrase 定义出现。它没有被赋值，所以存为 undefined。<br />3. phrase 被赋予了一个值。<br />4. phrase 引用了一个新值。

总结一下：
* 变量是特定内部对象的属性，与当前执行的（代码）块/函数/脚本有关。
* 操作变量实际上操作的是该对象的属性。

## Function Declaration

函数声明不像变量声明，不是在执行到所在代码时候才去完全初始化，而是更早的，在创建词法环境时。

对于全局词法环境来说，就是脚本启动那一刻。

> 下面的代码的词法环境一开始并不为空。因为 say 是一个函数声明，所以它里面有 say。后面它获得了用 let 定义的 phrase（属性）。

<img src="https://javascript.info/article/closure/lexical-environment-global-3.svg" style="background:#fff;"/><br />

## Inner and outer Lexical Environment

在调用中，say()用到了一个外部变量，那么让我们了解一下其中的细节。

函数运行时，自动创建一个新的函数词法环境。这个词法环境用于存储调用的局部变量和参数。

> 下面是say("John")内执行时词法环境的示意图：

<img src="https://javascript.info/article/closure/lexical-environment-simple.svg" style="background:#fff;"/><br />

在这个函数的执行中，有两个词法环境：内部词法环境（用于函数调用）和外部词法环境（全局）：

* 内部词法环境对应于当前执行的say。它有一个单独的属性name，是函数参数。我们执行 say("John")时，name 值为 "John"。

* 它的外部词法环境就是全局词法环境。其中包含了变量phrase和say函数。

内部词法环境持有外部词法环境的引用

**当代码访问一个变量时 —— 首先会在内部词法环境中进行搜索，然后是外部环境，然后是更外部的词法环境，直到全局词法环境。**

在严格模式下，变量未定义会导致错误。在非严格模式下，为了向后兼容，给未定义的变量赋值会创建一个全局变量。

> 让我们看看例子中的查找是如何进行的：

<img src="https://javascript.info/article/closure/lexical-environment-simple-lookup.svg" style="background:#fff;"/><br />

* 当 say 中的 alert 试图访问 name 时，它立即在函数词法环境中被找到。

* 当它试图访问 phrase 时，然而内部没有 phrase ，所以它追踪外部引用并在全局中找到它。

## 注意

* One call – one Lexical Environment

每次函数运行会都会创建一个新的函数词法环境。

* Lexical Environment is a specification object

『词法环境』是一个规范对象。我们不能在代码中获取或直接操作该对象。但 JavaScript 引擎同样可以优化它，比如清除未被使用的变量以节省内存和执行其他内部技巧等，只要显性行为保持如上所述。

## Nested functions

在函数中创建函数，这就是所谓的『嵌套』。它可以访问外部变量，并可以作为返回值。

更有趣的是，可以返回一个嵌套函数，既可以作为对象的方法，又可以直接作为结果返回。**不论在哪里进行调用，它仍可以访问同样的外部变量。**

* 嵌套函数由构造函数分配给新对象：
```javascript
function User(name) { // 构造函数返回一个新对象
  this.sayHi = function() { // 这个对象方法为一个嵌套函数
    alert(name);
  };
}
let user = new User("John");
user.sayHi(); // 该方法访问外部变量 "name"
```

* 返回函数：
```javascript
function makeCounter() {
  let count = 0;
  return function() {
    return count++; // has access to the outer counter
  };
}
let counter = makeCounter();
alert( counter() ); // 0
alert( counter() ); // 1
alert( counter() ); // 2
```
计数器内部的工作是怎样的呢？当内部函数运行时，count++ 会由内到外搜索该变量。在上面的例子中，步骤应该是：

<img src="https://javascript.info/article/closure/lexical-search-order.svg" style="background:#fff;"/><br />

1. 嵌套函数中。
2. 外部函数的变量。
3. 以此类推直到到达全局变量。

在这个例子中，count 在第二步中被找到。当外部变量被修改时，在找到它的地方被修改。因此 count++ 找到该外部变量并在它从属的词法环境中进行修改。好像我们有 let count = 1 一样。

这里有两个要考虑的问题：

1. 我们可以用某种方式在 makeCounter 以外的代码中修改 counter 吗？比如，在上例中的 alert 调用后。

2. 如果我们多次调用 makeCounter() —— 它会返回多个 counter 函数。它们的 count 是独立的还是共享的同一个呢？

在你继续读下去之前，请先回答这些问题。

…

想清楚了吗？

好吧，我们回答一下。

这是不可能做到的。counter 是一个局部函数的变量，我们不能从外部访问它。

makeCounter() 的每次调用都会创建一个拥有独立 counter 的新词法环境, 其拥有独立的count变量。因此counter 运行结果是独立的。

## Environments in detail

以下是 makeCounter 的执行过程。请注意，在这之前，我们还没有介绍 **[[Environment]]** 属性。

1. 在脚本开始时，只存在全局词法环境：

<img src="https://javascript.info/article/closure/lexenv-nested-makecounter-1.svg" style="background:#fff;"/><br />

**所有的函数在“出生”时都会有一个隐藏的 [[Environment]] 属性，并引用其被创建的词法环境。这是函数知道它是在哪里被创建的原因。**

2. 代码执行，makeCounter() 被执行。下图是当 makeCounter() 内执行第一行瞬间的快照：

<img src="https://javascript.info/article/closure/lexenv-nested-makecounter-2.svg" style="background:#fff;"/><br />

在 makeCounter() 执行时，包含其变量和参数的词法环境被创建。

词法环境中存储着两个东西：

* 一个是环境记录，它保存着局部变量。在我们的例子中 count 是唯一的局部变量（当执行到 let count 这一行时出现）。
* 另外一个是外部词法环境的引用，它被设置为函数的 [[Environment]] 属性。这里 makeCounter 的 [[Environment]] 属性引用着全局词法环境。

所以，现在有了两个词法环境：第一个是全局环境，第二个是 makeCounter 的词法环境，它拥有指向全局环境的引用。

3. 在 makeCounter() 的执行中，创建了一个小的嵌套函数。

不管是使用函数声明或是函数表达式创建的函数都没关系，所有的函数都有 [[Environment]] 属性，该属性引用着所创建的词法环境。新的嵌套函数同样也拥有这个属性。

我们新的嵌套函数的 [[Environment]] 的值就是 makeCounter() 的当前词法环境（创建的位置）。

<img src="https://javascript.info/article/closure/lexenv-nested-makecounter-3.svg" style="background:#fff;"/><br />

4. 随着执行的进行，makeCounter() 调用完成，并且将结果（该嵌套函数）赋值给全局变量 counter。

<img src="https://javascript.info/article/closure/lexenv-nested-makecounter-4.svg" style="background:#fff;"/><br />

5. 当 counter() 执行时，它会创建一个『空』的词法环境。它本身没有局部变量，但是 counter 有 [[Environment]] 作为其外部引用，所以它可以访问前面创建的 makeCounter() 函数的变量。

<img src="https://javascript.info/article/closure/lexenv-nested-makecounter-5.svg" style="background:#fff;"/><br />

如果它要访问一个变量，它首先会搜索它自身的词法环境（空），然后是前面创建的 makeCounter() 函数的词法环境，然后才是全局环境。 当它搜索 count ，它会在最近的外部词法环境 makeCounter 的变量中找到它。

请注意这里的内存管理工作机制。虽然 makeCounter() 执行已经结束，但它的词法环境仍保存在内存中，因为这里仍然有一个嵌套函数的 [[Environment]] 在引用着它。

**通常，只要有一个函数会用到该词法环境对象，它就不会被清理。并且只有没有函数会用到时，才会被清除。**

6. counter() 函数不仅会返回 count 的值，也会增加它。注意修改是『in place』完成的。准确地说是在找到 count 值的地方完成的修改。

<img src="https://javascript.info/article/closure/lexenv-nested-makecounter-6.svg" style="background:#fff;"/><br />

7. 接下来的 counter() 调用也是同样的。

至此，文章开头的第二题已经显而易见了。

<img src="https://zh.javascript.info/article/closure/lexenv-nested-work.svg" style="background:#fff;"/><br />

## Closures

> 一个函数保存其外部变量，并能够访问它们，称之为闭包。

> Tips: 在 JavaScript 中函数都是天生的闭包（只有一个例外，请参考 "new Function" 语法）。也就是说，他们会通过隐藏的 [[Environment]] 属性记住创建它们的位置，所以它们都可以访问外部变量。

```javascript
// 再次说明一下: new Function 创建函数，函数的 [[Environment]] 并不指向当前的词法环境，而是指向全局环境。
// let func = new Function ([arg1[, arg2[, ...argN]],] functionBody)
function getFunc() {
  let value = "test";
  let func = new Function('alert(value)');
  return func;
}
getFunc()(); // error：value 未定义
```

面试时，前端通常都会被问到『什么是闭包』，我们可以回答闭包的定义并且解释 JavaScript 中所有函数都是闭包，以及可能的关于 [[Environment]] 属性和词法环境原理。

## Code blocks and loops, IIFE

上面的例子都集中于函数。对于 {...} 代码块，词法环境同样也是存在的。 当代码块中包含块级局部变量并运行时，会创建词法环境。如if，while，for，iife, {}。

**值得注意的是：如果在 for 循环中声明变量，那么它在词法环境中是局部的**

```javascript
for (let i = 0; i < 10; i++) {
  // 每次循环都有其自身的词法环境
  // {i: value}
}
alert(i); // 错误，没有该变量
```
## Garbage collection

* Usually, a Lexical Environment is cleaned up and deleted after the function run.

* 但是如果有一个嵌套函数在 f 结束后仍可达，那么它的 [[Environment]] 引用会继续保持着外部词法环境存在：

```javascript
function f() {
  let value = 123;
  function g() { alert(value); }
  return g;
}
let g = f(); // g 是可达的，并且将其外部词法环境保持在内存中
```

A Lexical Environment object dies when it becomes unreachable (just like any other object). In other words, it exists only while there’s at least one nested function referencing it.

```javascript
function f() {
  let value = 123;
  function g() { alert(value); }
  return g;
}
let g = f(); // 当 g 存在时，对应的词法环境可达
g = null; // 相关的词法环境在内存中被清理
```

## Real-life optimizations

> 正如我们所了解的，理论上当函数可达时，它外部的所有变量都将存在。但实际上，JavaScript 引擎会试图优化它。它们会分析变量的使用情况，如果有变量没被使用的话它也会被清除。

> An important side effect in V8 (Chrome, Opera) is that such variable will become unavailable in debugging. That is not a bug in the debugger, but rather a special feature of V8. Perhaps it will be changed sometime. 

打开 Chrome 浏览器的开发者工具运行下面的代码。当它暂停时，在控制台输入 alert(value)

```javascript
function f() {
  let value = Math.random();
  function g() {
    debugger; // 在控制台中输入 alert( value ); 没有该值！
  }
  return g;
}
let g = f();
g();
```

正如你所见的 ———— 没有该值！理论上，它应该是可以访问的，但引擎对此进行了优化。

这可能会导致有趣的调试问题。其中之一 —— 我们可以看到的是一个同名的外部变量，而不是预期的变量：

```javascript
let value = "Surprise!";
function f() {
  let value = "the closest value";
  function g() {
    debugger; // 在控制台中：输入 alert( value )；Surprise!
  }
  return g;
}
let g = f();
g();
```

## 课后作业

1. Counter object

```javascript
function Counter() {
  let count = 0;
  this.up = function() {
    return ++count;
  };
  this.down = function() {
    return --count;
  };
}
let counter = new Counter();
alert( counter.up() ); // ?
alert( counter.up() ); // ?
alert( counter.down() ); // ?
```

2. Function in if

```javascript
let phrase = "Hello";
if (true) {
  let user = "John";
  function sayHi() {
    alert(`${phrase}, ${user}`);
  }
}
sayHi(); // ?
```

3. Shooters

```javascript
function makeArmy() {
  let shooters = [];
  let i = 0;
  while (i < 10) {
    let shooter = function() { // shooter 函数
      alert( i ); // 应该显示它自己的数字
    };
    shooters.push(shooter);
    i++;
  }
  return shooters;
}
let army = makeArmy();
army[0](); // 第 0 号 shooter 显示 10
army[5](); // 第 5 号也输出 10...
// ... 所有的 shooters 都显示 10 而不是期望的 0, 1, 2, 3...，尝试用两种方法修复它！
```

## 参考
[Lexical Environments](https://tc39.es/ecma262/#sec-lexical-environments)

[Closure](https://javascript.info/closure)

[javascript中词法环境、领域、执行上下文以及作业详解](https://segmentfault.com/a/1190000012162360)

