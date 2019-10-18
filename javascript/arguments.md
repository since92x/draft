> `arguments` 对象是所有（`非箭头`）函数中都可用的局部变量。你可以使用 `arguments` 对象在函数中引用函数的参数

# arguments.callee

指向当前执行的函数。

# arguments.caller

指向调用当前函数的函数。

# arguments.length

指向传递给当前函数的参数数量。

# arguments[@@iterator]

返回一个新的 Array 迭代器对象，该对象包含参数中每个索引的值。

> **注意:现在在严格模式下，arguments 对象已与过往不同。arguments[@@iterator]不再与函数的实际形参之间共享，同时 caller 属性也被移除。**

即在严格模式下，修改形参和修改 `arguments` 效果不同

通过`arguments` 可以模拟重载

`默认参数`对`arguments`没有影响

# 参考

[Arguments 对象](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/arguments)
