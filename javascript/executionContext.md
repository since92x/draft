# Execution context

## 执行上下文

> 执行上下文(Execution contex)是由ECMAScript实现的一种用来追踪代码运行的机制。在任何时刻，最多只有一个执行上下文正在执行代码，被称为运行时执行上下文(Running execution context)。

> 执行上下文栈(Execution context stack)是用来追踪执行上下文的。运行时执行上下文总是这个栈的栈顶元素。

> 任何时候，当控制器从当前运行的执行环境相关的可执行代码转入与该执行环境无关的可执行代码时，会创建一个新的执行环境。新建的这个执行环境会推入栈中，成为当前的运行时执行上下文。

> 执行上下文包含跟踪其关联代码的执行进度所需的任何实现特定状态。 每个执行上下文至少具有下表中列出的**状态组件**。

| 组件| 用途 |
| --- | --- |
| code evaluation state | 执行，暂停和恢复与该执行上下文相关联的代码的执行所需的任何状态。 |
| Function | 如果这个执行环境正在执行函数对象的代码，那么这个组件的值就是该函数对象。 如果上下文执行脚本或模块的代码，则该值为null。 |
| Realm | 相关代码访问ECMAScript资源的Realm Record。 |
| ScriptOrModule | 相关代码所源自的Module Record或Script Record。 若没有所源自的脚本或模块，就像在[InitializeHostDefinedRealm](https://tc39.es/ecma262/#sec-initializehostdefinedrealm)中创建的原始执行上下文的情况一样，值为null。 |

运行时执行上下文的Realm组件的值也被称为当前领域记录(current Realm Record)。运行时执行上下文的Function组件的值也被称为活动函数对象(active function object)。 
ECMAScript代码的执行上下文具有下表列出的其他状态组件。

ECMAScript代码的执行上下文具有下表列出的其他**状态组件**。

| 组件 | 用途 |
| --- | --- |
| LexicalEnvironment | 标识用于解析此执行上下文中的代码所做的标识符引用的词法环境。 |
| VariableEnvironment | 标识在此执行上下文中的词法环境，它的环境记录保存了由VariableStatements创建的绑定。 |

> 当创建执行上下文时，它的LexicalEnvironment组件和VariableEnvironment组件在大多数情况下都指向相同的词法环境（Lexical Environment）。

当执行上下文正在执行generator对象时，会有一个额外的**组件**如下表所示

| 组件 | 目的 |
| --- | --- |
| Generator	| 此执行上下文正在评估的GeneratorObject。 |


## 执行上下文的创建、入栈及出栈

> ECMAScript可执行代码有四种类型：全局代码，函数代码，模块代码和eval。
>> 这里虽然说是全局代码，但是JavaScript引擎其实是按照script标签来解析执行的，也就是说script标签按照它们出现的顺序解析执行，这也就是为什么我们平时要将项目依赖js库放在前面引入的原因。

> JavaScript引擎是按可执行代码块来执行代码的，在任意的JavaScript可执行代码被执行时，执行步骤可按如下理解：
> 1. 创建一个新的执行上下文（Execution Context）
> 2. 创建一个新的词法环境（Lexical Environment）
> 3. 将该执行上下文的 变量环境组件（VariableEnvironment） 和 词法环境组件（LexicalEnvironment） 都指向新创建的词法环境
> 4. 将该执行上下文 推入执行上下文栈 并成为 运行时执行上下文（Running Execution Context）
> 5. 对代码块内的 标识符进行实例化及初始化
> 6. 运行代码
> 7. 运行完毕后执行上下文出栈

## Hoisting 和 temporal dead zone

> 我们平常所说的变量提升就发生在上述执行步骤的第四步，对代码块内的标识符进行实例化及初始化的具体表现如下：
> 1. 执行代码块内的let、const和class声明的标识符合集记录为lexNames
> 2. 执行代码块内的var和function声明的标识符合集记录为varNames
> 3. 如果lexNames内的任何标识符在varNames或lexNames内出现过，则报错SyntaxError
>> 这就是为什么可以用var或function声明多个同名变量，但是不能用let、const和class声明多个同名变量。
> 4. 将varNames内的var声明的标识符实例化并初始化赋值undefined，如果有同名标识符则跳过
>> 这就是所谓的变量提升，我们用var声明的变量，在声明位置之前访问并不会报错，而是返回undefined
> 5. 将lexNames内的标识符实例化，但并不会进行初始化，在运行至其声明处代码时才会进行初始化，在初始化前访问都会报错。
>> 这就是我们所说的暂时性死区，let、const和class声明的变量其实也提升了，只不过没有被初始化，初始化之前不可访问。
> 6. 最后将varNames内的函数声明实例化并初始化赋值对应的函数体，如果有同名函数声明，则前面的都会忽略，只有最后一个声明的函数会被初始化赋值。
>> 函数声明会被直接赋值，因此我们在函数声明位置之前也可以调用函数。

## 为什么需要两个环境组件
即 VariableEnvironment 组件和 LexicalEnvironment 组件

> 首先明确这两个环境组件的作用，变量环境组件（VariableEnvironment）用于记录var声明的绑定，词法环境组件（LexicalEnvironment）用于记录其他声明的绑定（如let、const、class等）。

> 一般情况下一个Exexution Context内的VariableEnvironment和LexicalEnvironment指向同一个词法环境，之所以要区分两个组件，主要是为了**实现块级作用域的同时不影响var声明及函数声明**。

> 众所周知，ES6之前并没有块级作用域的概念，但是ES6及之后我们可以通过新增的let及const等命令来实现块级作用域，并且不影响var声明的变量和函数声明，那么这是怎么实现的呢？
> 1. 首先在一个正在运行的执行上下文（running Execution Context）内，词法环境由VariableEnvironment和LexicalEnvironment构成，此执行上下文内的所有标识符的绑定都记录在两个组件的环境记录内。
> 2. 当运行至块级代码时，会将LexicalEnvironment记录下来，我们将其记录为oldEnv。
> 3. 然后创建一个新的LexicalEnvironment（外部词法环境outer指向oldEnv），我们将其记录为newEnv，并将newEnv设置为running Execution Context的LexicalEnvironment。
> 4. 然后块级代码内的let、const等声明就会绑定在这个newEnv上面，但是var声明和函数声明还是绑定在原来的VariableEnvironment上面。
>> 块级代码内的函数声明会被当做var声明，会被提升至外部环境，块级代码运行前其值为初始值undefined
> 5. 块级代码运行完毕后，又将oldEnv还原为running Execution Context的LexicalEnvironment。

> 目前包括块级代码（在一对大括号内的代码）、for循环语句、switch语句、TryCatch语句中的catch从句以及with语句（with语句创建的新环境为对象式环境，其他皆为声明式环境）都是这样来实现块级作用域的。

## 回顾ES3时代的EC

在ES3时代，EC主要有三部分：变量对象、作用域链、this。

VO: 变量对象是与执行上下文相关的数据作用域，存储了在上下文中定义的变量和函数声明。
```
ECStack = [EC1, EC2, ...]

EC = {
    VO:  [...],  // 保存变量和函数声明
    scope:  [...];  // 作用域
    this:  {...};  // this的值
}
```
eg: 
``` javascript
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f() {
        var inner = 'inner';
        return scope;
    }
    return f();
}
checkscope();
```
上述代码执行过程可描述为:
```
1. 执行全局代码，创建全局EC，全局EC被压入EC栈

ECStack = [
  globalContext
];

2. 全局EC初始化

globalContext = {
  VO: [global, scope, checkscope],
  Scope: [globalContext.VO],
  this: globalContext.VO
}

初始化的同时，checkscope函数被创建，保存作用域链到[[scope]]

checkscope.[[scope]] = [
  globalContext.VO
];

3. 执行checkscope函数，创建checkscope函数执行上下文，checkscope函数执行上下文被压入执行上下文栈

ECStack = [
  checkscopeContext,
  globalContext
];

4. checkscope函数执行上下文初始化：

  复制函数 [[scope]] 属性创建作用域链，
  用 arguments 创建活动对象，
  初始化活动对象，即加入形参、函数声明、变量声明，
  将活动对象压入 checkscope 作用域链顶端。

同时 f 函数被创建，保存作用域链到 f 函数的内部属性[[scope]]

checkscopeContext = {
  AO: {
    arguments: {
      length: 0
    },
    scope: undefined,
    f: reference to function f(){}
  },
  Scope: [AO, globalContext.VO],
  this: undefined
}

5. 执行 f 函数，创建 f 函数执行上下文，f 函数执行上下文被压入执行上下文栈

ECStack = [
  fContext,
  checkscopeContext,
  globalContext
];

6. f 函数执行上下文初始化, 以下跟第 4 步相同：

fContext = {
  AO: {
    arguments: {
      length: 0
    }
  },
  Scope: [AO, checkscopeContext.AO, globalContext.VO],
  this: undefined
}
7. f 函数执行，沿着作用域链查找 scope 值，返回 scope 值

8. f 函数执行完毕，f 函数上下文从执行上下文栈中弹出

9. checkscope 函数执行完毕，checkscope 执行上下文从执行上下文栈中弹出
ECStack = [
  globalContext
];

```


## 相关阅读

让我们来读一读社区和论坛内的相关解读：
1. 执行上下文
> 一个执行上下文可以激活另一个上下文，就好比一个函数调用了另一个函数(或者全局的上下文调用了一个全局函数)，然后一层一层调用下去。逻辑上来说，这种实现方式是栈，我们可以称之为上下文栈。

> 激活其它上下文的某个上下文被称为 调用者(caller) 。被激活的上下文被称为被 调用者(callee) 。被调用者同时也可能是调用者(比如一个在全局上下文中被调用的函数调用某些自身的内部方法)。

> 当一个 caller 激活了一个 callee，那么这个 caller 就会暂停它自身的执行，然后将控制权交给这个 callee . 于是这个 callee 被放入堆栈，称为进行中的上下文 [running/active execution context] . 当这个 callee 的上下文结束之后，会把控制权再次交给它的 caller，然后caller会在刚才暂停的地方继续执行。在这个caller结束之后，会继续触发其他的上下文。一个 callee 可以用返回（return）或者抛出异常（exception）来结束自身的上下文。

2. 下面我们再举一例，感受一下

```javascript
let a = 20;
const b = 30;
var c;
function multiply(e, f) {
  var g = 20;
  return e * f * g;
}
c = multiply(20, 30);
```

全局EC在创建阶段, 上面过程的伪代码看起来像：

``` javascript
GlobalExectionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      // Identifier bindings go here
      a: < uninitialized >,
      b: < uninitialized >,
      multiply: < func >
    }
    outer: <null>,
    ThisBinding: <Global Object>
  },
  VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      // Identifier bindings go here
      c: undefined,
    }
    outer: <null>, 
    ThisBinding: <Global Object>
  }
}
```

在运行阶段，变量赋值已经完成。全局EC在执行阶段看起来像：

``` javascript
GlobalExectionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      // Identifier bindings go here
      a: 20,
      b: 30,
      multiply: < func >
    }
    outer: <null>,
    ThisBinding: <Global Object>
  },
  VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Object",
      // Identifier bindings go here
      c: undefined,
    }
    outer: <null>,
    ThisBinding: <Global Object>
  }
}
```

当遇到 multiply(20，30) 函数调用时，一个新的函数执行环境被创建并执行函数中的代码。因此函数执行环境在创建阶段看起来像：

``` javascript
 FunctionExectionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // Identifier bindings go here
      Arguments: {0: 20, 1: 30, length: 2},
    },
    outer: <GlobalLexicalEnvironment>,
    ThisBinding: <Global Object or undefined>,
  },
  VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // Identifier bindings go here
      g: undefined
    },
    outer: <GlobalLexicalEnvironment>,
    ThisBinding: <Global Object or undefined>
  }
}
```

在这以后，执行环境会经历执行阶段，这意味着在函数内部赋值给变量的过程已经完成。因此此函数执行环境在执行阶段看起来就像这样的：

``` javascript
FunctionExectionContext = {
  LexicalEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // Identifier bindings go here
      Arguments: {0: 20, 1: 30, length: 2},
    },
    outer: <GlobalLexicalEnvironment>,
    ThisBinding: <Global Object or undefined>,
  },
  VariableEnvironment: {
    EnvironmentRecord: {
      Type: "Declarative",
      // Identifier bindings go here
      g: 20
    },
    outer: <GlobalLexicalEnvironment>,
    ThisBinding: <Global Object or undefined>
  }
}
```

在函数执行完成以后，返回值会被存储在c里。因此全局词法环境被更新。在这之后，全局代码执行完成，程序运行终止。

**注意：正如你所注意到的，let和const在创建阶段定义的变量没有值与他们相关联，但是var定义变量会设置为undefined。**

**这是因为，在创建阶段，扫描代码以查找变量和函数声明，而函数声明的全部都存储到环境中；变量首先会被初始化为undefined（用var声明时），或是未初始化状态（用let和const声明时）。**

这就是我们所说的提升（hoisting）。

注意 - 在执行阶段，如果javascript引擎在源代码中声明的实际位置找不到let变量的值，那么将会分配为undefined。


## 参考

[Execution Contexts](https://tc39.es/ecma262/#sec-execution-contexts)

[执行上下文](https://github.com/logan70/Blog/issues/2)

[Understanding Execution Context and Execution Stack in Javascript](https://blog.bitsrc.io/understanding-execution-context-and-execution-stack-in-javascript-1c9ea8642dd0)
