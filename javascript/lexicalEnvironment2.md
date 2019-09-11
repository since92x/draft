# javascript Lexical Environment (读ecma262)

## Lexical Environment

> Lexical Environment 用于定义特定变量和函数标识符在 ECMAScript 代码的词法嵌套结构上关联关系的规范类型。通常，词法环境与ECMAScript代码的某些特定语法结构相关联，例如FunctionDeclaration，BlockStatement，或TryStatement的Catch子句，每次执行这类代码时都会创建一个新的词法环境。

> 词法环境由 **Environment Record** 和 **reference to an outer Lexical Environment**（以下简称outRef）。

> Environment Record 记录在其关联的词法环境范围内创建的标识符绑定。它被称为Lexical Environment的EnvironmentRecord。

> 外部环境引用对Lexical Environment值的逻辑嵌套进行建模。 （内部）词法环境的外部引用是对逻辑上包围着它的词法环境的引用。一个词法环境可以作为多个内部词法环境的外部环境。

> 全局环境(Global environment) 是一个没有外部环境的词法环境。全局环境的outRef为null。全局环境的EnvironmentRecord可以预先填充标识符绑定，并包括关联的全局对象，其属性提供一些全局环境的标识符绑定。在执行ECMAScript代码时，可以向全局对象添加其他属性，并且可以修改初始属性。

> 模块环境(Module environment)是一个包含模块顶层声明绑定的词法环境。模块环境的外部环境是全局环境(Global environment)。

> 函数环境(Function environment)是与ECMAScript函数对象的调用相对应的词法环境。函数环境可以建立新的this绑定。 函数环境还捕获支持继承(super)方法调用所必需的状态.

## Environment Record

> Environment Record 记录了在它的关联词法环境域内创建的标识符绑定情形。规范中有两种主要的环境记录：**声明性环境记录（declarative Environment Records）** 和 **对象环境记录（object Environment Records）**。

> **声明式环境记录**用于定义那些将 **标识符 与语言值**直接绑定的 ECMAScript 语法元素，例如 函数定义，变量定义 及 Catch 语句

> **对象式环境记录**用于定义那些将 **标识符 与具体对象的属性**绑定的 ECMAScript 元素，例如 With 语句。

> 全局环境记录和函数环境记录是专门用于脚本的全局声明和函数中的顶级声明。

> 出于标准规范的目的，可以将环境记录理解为面向对象中的一个简单继承结构，其中**环境记录**是一个抽象类，有三个具体实现子类，分别为**声明式环境记录**(Declarative Environment Record)， **对象式环境记录**(Object Environment Record)，**全局环境记录**(Global Environment Record)。**函数环境记录**和**模块环境记录**是**声明式环境记录**的子类。抽象类包括在下表中定义的抽象规范方法。 这些抽象方法对于每个具体的子类都有不同的具体算法(由于十分详细和琐碎，本文不再搬砖，直接看ecma262规范).

| Method | Purpose  |
| --- | ---|
| HasBinding(N) | 确定环境记录是否具有String值 N 的绑定。 如果没有，请返回 true ，否则为 false |
| CreateMutableBinding(N, D) (创建可变绑定) | 在环境记录中创建一个新的但未初始化的可变绑定。 String值 N 是绑定名称的文本。 如果布尔参数 D 是 true ，则绑定可能随后被删除. |
| CreateImmutableBinding(N, S) (设置不可变绑定) | 在环境记录中创建一个新的但未初始化的不可变绑定。 String值 N 是绑定名称的文本。 如果 S 是 true ，那么尝试在初始化之前访问绑定的值，或者在初始化之后对其进行设置将始终引发异常， 无论严格模式是否设置，对于该绑定. |
| InitializeBinding(N, V) | 在环境记录中设置已存在但未初始化的绑定的值。 String值 N 是绑定名称的文本。 V 是绑定的值，是任何 ECMAScript语言类型. |
| SetMutableBinding(N, V, S) | 在环境记录中设置已经存在的可变绑定的值。 String值 N 是绑定名称的文本。 V 是绑定的值，可以是任何 ECMAScript语言类型。 S 是一个布尔标志。 如果 S 是 true ，并且绑定不能设置，则抛出一个 TypeError 异常. |
| GetBindingValue(N, S) | 从环境记录中返回已经存在的绑定的值。 String值 N 是绑定名称的文本。 S 用于标识源自严格模式代码(strict mode code) ，否则需要严格的模式引用语义。 如果 S 是 true ，绑定不存在则引发一个 ReferenceError 异常。 如果绑定存在但未初始化，则抛出 ReferenceError ，而不管 S 的值如何. |
| DeleteBinding(N) | 从环境记录中删除绑定。 String值 N 是绑定名称的文本。 如果存在 N 的绑定，则删除绑定并返回 true 。 如果绑定存在但不能被删除返回 false 。 如果绑定不存在返回 true. |
| HasThisBinding() | 确定环境记录是否建立了一个this绑定。 如果没有，请返回 true ，否则为 false. |
| HasSuperBinding() | 确定环境记录是否建立了一个super方法绑定。 如果没有，请返回 true ，否则为 false. |
| WithBaseObject() | 如果此环境记录与with语句相关联，则返回with对象。 否则返回 undefined. |


### 声明式环境记录

> 每个声明式环境记录都和ECMAScript程序作用域内的变量、常量、let、class、module、import或者函数声明相关联。一个声明式环境记录绑定了一系列在其范围内的声明的标识符。

eg: 

```javascript
import x from 'constants.js';
var a=1;
let b=1;
const c=1;
function foo(){};
class Bar{};
// 这时声明性环境记录中就有了«x,a,b,c,foo,Bar»这样一组标识符，当然实际存放的结构肯定不是这个样子的，还要复杂。
```
由于**函数环境记录**和**模块环境记录**是**声明式环境记录**的子类，下面我们将其归入此模块介绍。

#### 函数环境记录

> 函数环境记录是一个声明性环境记录，它用来表示function中的顶级作用域，此外如果函数不是一个箭头函数（ArrowFunction），则为这个函数提供一个this绑定。如果一个函数不是一个ArrowFunction函数并引用了super，则它的函数环境记录还包含从该函数内执行super方法调用的状态。

下表列出了Function Environment Records 的附加字段

| Field Name | Value | Meaning |
| --- | --- | --- |
| [[ThisValue]] | Any | 这是用于此函数调用的this值. |
| [[ThisBindingStatus]] | "lexical" |  "initialized" |  "uninitialized" | 如果值为"lexical"，则这是 ArrowFunction ，并且没有本地this值。 |
| [[FunctionObject]] | Object | 函数对象被调用导致这个Environment Record 被创建。 |
| [[HomeObject]] | Object \| undefined | 如果相关函数具有super属性访问，并且不是 ArrowFunction ，[[HomeObject ]]是函数绑定到一个方法的对象。 [[HomeObject]]的默认值为 undefined 。 |
| [[NewTarget]] | Object \| undefined | 如果这个Environment Record 是由[[Construct]]内部方法创建，[[NewTarget]]是[[Construct]] newTarget 参数的值。 否则，其值为 undefined 。 |

> 函数环境记录支持所有声明式环境记录列出的方法，除了HasThisBinding和HasSuperBinding之外，共享所有这些方法的相同规范。 另外，环境记录功能支持下表中列出的方法:

| Method | Purpose |
| --- | --- |
| BindThisValue(V) | 设置[[ThisValue]]并记录它已被初始化。 |
| GetThisBinding() | 返回此Environment Recordthis绑定。 如果this绑定尚未初始化，则抛出 ReferenceError。 |
| GetSuperBase() | 返回在Environment Record中作为 super 属性访问绑定的基础的对象。该对象源于此Environment Record的[[ HomeObject]]字段。 值 undefined 表示super属性访问将产生运行时错误. |


#### 模块环境记录

> 模块环境记录是一个声明性环境记录，用于表示ECMAScript模块的外部作用域。除了正常的可变和不可变绑定之外，模块环境记录还提供了不可变的导入绑定，这些绑定提供间接访问另一个环境记录中存在的目标绑定。

### 对象式环境记录

> 每个对象环境记录都有一个关联的对象，这个对象被称作 绑定对象。对象环境记录直接将一系列标识符与其绑定对象的属性名称建立一一对应关系。不符合IdentifierName 的属性名不会作为绑定的标识符使用。无论是对象自身的，还是继承的属性都会作为绑定，无论该属性的 [[Enumerable]] 特性的值是什么。由于对象的属性可以动态的增减，因此对象式环境记录项所绑定的标识符集合也会隐匿地变化，这是增减绑定对象的属性而产生的副作用。通过以上描述的副作用而建立的绑定，均被视为可变绑定，即使该绑定对应的属性的 Writable 特性的值为 false。对象式环境记录项没有不可变绑定。

eg: 

```javascript
const o ={
    cnt: 0,
    show: function() {
      console.log(this.cnt);
    }
}
with(o){
    a = a+1;
    show(); // 1
}
```
在js代码执行到with语句的时：

1. 创建新的词法环境。
2. 接着创建了一个对象环境记录即为OER，OER包含withObject这个绑定对象，OER中的字符串标识符名称列表为 o 中的属性«cnt, show»，在with语句中的变量操作默认在绑定对象中的属性中优先查找。
3. 为OER设置外部词法环境引用。

> 注意：对象环境记录不是指Object里面的环境记录。普通的Object内部不存在新的环境记录，它的环境记录就是定义该对象所在的环境记录。


### 全局环境记录

> 全局环境记录用于表示在共同领域（Realms）中处理所有共享最外层作用域的ECMAScript Script元素。全局环境记录提供了内置全局绑定，全局对象属性以及所有在脚本中发生的顶级声明。

| Field Name | Value | Meaning |
| --- | --- |--- |
| [[ObjectRecord]] | Object Environment Record | 绑定对象是global object。 它包含全局内置绑定以及关联领域的全局代码中FunctionDeclaration, GeneratorDeclaration, and VariableDeclaration绑定. |
| [[GlobalThisValue]] | Object | 在全局作用域内返回的this值。宿主可以提供任何ECMAScript对象值。 |
| [[DeclarativeRecord]] | Declarative Environment Record | 包含在关联领域的全局代码中除了FunctionDeclaration，GeneratorDeclaration，AsyncFunctionDeclaration和VariableDeclaration绑定之外的所有声明的绑定。 |
| [[VarNames]] | List of String | 由相关领域的全局代码中的FunctionDeclaration，GeneratorDeclaration和VariableDeclaration声明绑定的字符串名称。 |

> 这里提一下,在全局环境记录中，FunctionDeclaration，GeneratorDeclaration，AsyncFunctionDeclaration和VariableDeclaration不在Declarative Environment Record中，而是在Object Environment Record中，这也解释了为什么在全局代码中用var、function声明的变量自动的变为全局对象的属性而let、const、class等声明的变量却不会成为全局对象的属性。


## Realm

> 在执行ECMAScript代码之前，所有ECMAScript代码都必须与一个领域相关联。从概念上讲，领域由一组内部对象，ECMAScript全局环境，在该全局环境范围内加载的所有ECMAScript代码以及其他关联的状态和资源组成。规范中，领域表示为领域记录（Realm Record）。有如下字段

| 字段名称 | 值 | 含义 |
| --- | --- | --- |
| [[Intrinsics]] | 一个记录,它的字段名是内部键，其值是对象 | 与此领域相关的代码使用的内在值。                    |
| [[GlobalObject]] | Object | 这个领域的全局对象。 |
| [[GlobalEnv]] | Lexical Environment | 这个领域的全局环境。 |
| [[TemplateMap]] | 一个记录列表 { [[Strings]]: List, [[Array]]: Object}. | 模板对象使用Realm Record的[[TemplateMap]]分别对每个领域进行规范化。 |
| [[HostDefined]] | Any, 默认值是undefined. | 保留字段以供需要将附加信息与Realm Record关联的宿主环境使用。 |

**如何理解呢?**

让我们来看看有关论坛和社区内的回答: 

> In web browsers, each frame and window has its own realm with separate global variables. That prevents instanceof from working for objects that cross realms. --- << Speaking JavaScript >>

> 由于javascript环境差异可能很大，所以规范使用较为抽象的描述。在浏览器中，一个window（一个frame，一个用window.open（）打开的window，或者只是一个普通的浏览器tab）是一个realm。一个web worker是一个realm。一个service worker是一个realm。

> 对象可能跨越realm边界，例如：从一个窗口打开的新的窗口，可以通过函数调用和变量引用进行相互通信。下面我们以iframe为例，思考下面代码：

```javascript
// child.html
  window.parent.parentFunc(globalThis, ["I'm child's Array instance"]);

// parent.html
  function parentFunc(subGlobalThis, arr) {
      console.debug(subGlobalThis === globalThis); // false
      console.debug(arr instanceof globalThis.Array); // false
  }
```
## Job and Job Queue

> Job是一个抽象操作，当没有其他ECMAScript计算正在进行时，启动的一个ECMAScript计算, 称为Job。 一个Job抽象操作以接受任意一组作业参数。

> 只有在没有运行时执行上下文且执行上下文栈为空时，才能启动执行Job。 PendingJob是一个未来执行的Job，它是个内部Record，其字段在下表中指定。一旦Job启动了执行，一定会执行完成，在这期间不会有其他的Job启动。 然而，当前值执行的Job或者外部的事件可能会加入额外的PendingJobs，他们会在执行完当前的Job后的某个时机启动执行。

Table: PendingJob Record Fields

| 字段名 | 值 | 意义 |
| --- | --- | ---|
| [[Job]]| Job抽象操作的名字 | 这是在启动此PendingJob的执行时执行的抽象操作。 |
| [[Arguments]] | 一个List | [[Job]]被激活的时候，传给它的参数列表 |
| [[Realm]] | 一个Realm Record | 启动此PendingJob时初始执行上下文的领域记录。 |
| [[ScriptOrModule]] | 一个 Script Record 或 Module Record | 启动此PendingJob时初始执行上下文的脚本或模块。 | 
| [[HostDefined]] | 任意值，默认是undefined | 保留的一些宿主环境需要使用的字段，为了给pending Job附加额外信息。 |

Job Queue 是PendingJob记录的FIFO队列。每种ECMAScript的实现都至少具有下表定义的作业队列。

| 名字 | 用途 |
| --- | --- |
| ScriptJobs | Jobs that validate and evaluate ECMAScript Script and Module source text. |
| PromiseJobs | Jobs that are responses to the settlement of a Promise. | 

## TO CONTINUE

ecma262规范好难读哦～，抽象，晦涩，还需要慢慢咀嚼。OTL

## 参考

[Lexical Environments](https://tc39.es/ecma262/#sec-lexical-environments)

[javascript中词法环境、领域、执行上下文以及作业详解](https://segmentfault.com/a/1190000012162360)
