# defer

> defer 脚本会在文档渲染完毕后，DOMContentLoaded 事件调用前执行。

> 异步下载,不影响 dom 渲染。

> 如果有多个设置了 defer 的 script 标签存在，则会按照顺序执行所有的 script

# async

> async 的执行，并不会按着 script 在页面中的顺序来执行，而是谁先加载完谁执行。

> async 的设置，会使得 script 脚本异步的加载并在允许的情况下执行。
