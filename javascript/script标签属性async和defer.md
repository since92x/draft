> # Script 标签属性：async, defer
>
> 现代的网站中，脚本往往比 HTML 更“重”：它们的大小通常更大，处理时间也更长
>
> 浏览器必须等脚本下载完，并执行结束，之后才能继续处理剩余的页面。
>
> 这会导致两个重要的问题：
>
> - 脚本不能访问到位于它们下面的 DOM 元素，因此，脚本不能给它们添加事件等。
>
> - 如果页面顶部有一个庞大的脚本，它会“阻塞页面”。在脚本下载并执行结束前，用户都不能看到页面内容：
>
> ```html
> <p>...content before script...</p>
> <script src="https://javascript.info/article/script-async-defer/long.js?speed=1"></script>
> <!-- 在脚本加载结束前都看不到下面的内容 -->
> <p>...content after script...</p>
> ```
>
> 这里有一些解决办法。例如，我们可以把脚本放在页面底部。此时，它可以访问到它上面的元素，并且不会阻塞页面显示内容
>
> ```html
> <body>
>   <p>...all content is above the script...</p>
>   <script src="https://javascript.info/article/script-async-defer/long.js?speed=1"></script>
> </body>
> ```
>
> 但是这样的方案绝非完美。例如：浏览器只有在下载完整的 HTML 文档后才会注意到脚本（并且开始下载它）。对于长的 HTML 文档来说，这样的延迟必须引起注意。
>
> 于网络连接很快的人来说，这不值一提。但是这个世界上仍然有很多地区的人们他们使用很慢的网络连接，并且使用着远非完美的移动互联网。
>
> 幸运的是，这里有两个 `<script>` 属性可以解决我们的这个问题：`defer` 和 `async`。

## defer

> `defer` 属性告诉浏览器它应该继续处理页面，并在“后台”下载脚本，然后等页面处理完成后才开始执行此脚本。
>
> - 具有 `defer` 属性的脚本不会阻塞页面的加载。
>
> - 有 defer 属性的脚本总是要等到 DOM 解析完毕，但在 `DOMContentLoaded` 事件之前执行。
>
> 下面的例子演示了这一过程：
>
> ```html
> <p>...content before scripts...</p>
> <script>
>   document.addEventListener('DOMContentLoaded', () =>
>     alert('DOM ready after defer!')
>   ); // (2)
> </script>
> <script defer src="https://javascript.info/article/script-async-defer/long.js?speed=1"></script>
> <p>...content after scripts...</p>
> ```
>
> Defer 脚本保持他们的相对顺序，就像常规脚本一样。
>
> 浏览器解析页面找到 script 属性并并行下载它们，以提高性能。
>
> defer 属性会忽略没有 src 属性的 `<script>` 脚本。

## async

> `async` 属性意味着脚本是完全独立的：
>
> - 页面不会等待异步脚本，它会继续处理页面并显示内容。
> - DOMContentLoaded 和 async 脚本不会彼此等待:
>> - `DOMContentLoaded` 可能发生在异步脚本之前（此时异步脚本在页面加载完成后才加载完成）
>> - `DOMContentLoaded` 也可能发生在异步脚本之后（此时异步脚本可能很短或者是从 HTTP 缓存中加载的）
> - 其他脚本不会等待 async 脚本加载完成，同样 async 脚本也不会等待其他脚本。
> 
> 因此，如果我们有几个 async 脚本，它们可能按任意次序执行，总之是先加载完成的就先运行：
>
> ```html
> <p>...content before scripts...</p>
> <script>
>   document.addEventListener('DOMContentLoaded', () => alert("DOM ready!"));
> </script>
> <script async src="https://javascript.info/article/script-async-defer/long.js"></script>
> <script async src="https://javascript.info/article/script-async-defer/small.js"></script>
> <p>...content after scripts...</p>
> ```
>
> * 页面内容立刻显示出来：`async` 脚本不会阻塞页面加载。
> * `DOMContentLoaded` 可以在 `async` 之前或之后触发，不能保证谁在前谁在后。
> * Async 脚本不会等待彼此。一个小的脚本 small.js 放在后面，但是可能会比 long.js 这个长脚本先加载完成，所以尽管 small.js 在后面，但是它可能先运行。这一行为被称为“加载优先（load-first）”顺序。
>
> 当我们将独立的第三方脚本集成到页面的时候，此时采用异步加载方式是非常棒的：计数器，广告等等，因为它们不依赖于我们的脚本，同样我们的脚本也不应该等待它们加载完成：
> ```html
> <!-- Google Analytics 脚本通常是这样嵌入页面的 -->
> <script async src="https://google-analytics.com/analytics.js"></script>
> ```
> 动态脚本（Dynamic scripts）
>
> 我们也可以使用 JavaScript 动态地添加脚本：
> ```javascript
> let script = document.createElement('script');
> script.src = "/article/script-async-defer/long.js";
> document.body.append(script); // (*)
> ```
> 当脚本附加到文档 (*) 时，脚本就会开始加载：
>
> 默认情况下，动态脚本表现为“异步”行为。
>
> 这也就是说：
> * 它们不会等待其他内容，其他的内容也不会等待它们。
> * 先加载完成的脚本先运行（“加载优先” 顺序）
> 我们可以通过将 `async` 属性显示修改为 `false` 以将`加载优先`顺序修改为`文档顺序`（就像常规脚本一样）>
> 例如，这里我们添加了两个脚本。在没有设置 script.async=false 时，它们执行顺序为`加载优先`顺序（即 small.js 可能先运行）。但是当设置了 `script.async=false` 后，脚本执行顺序就是它在文档中的顺序：
>```javascript
> function loadScript(src) {
>   let script = document.createElement('script');
>   script.src = src;
>   script.async = false;
>   document.body.append(script);
> }
> // 由于 async=false 属性存在，long.js 会先运行
> loadScript("/article/script-async-defer/long.js");
> loadScript("/article/script-async-defer/small.js");
> ```

## 总结

> async 和 defer 属性有一个共同点：它们都`不会阻塞页面的渲染`。因此，用户可以立即阅读并了解页面内容。
> 
> 但是它们之间也存在一些本质的区别：
>
> | 类型    | 顺序             | DOMContentLoaded                                                                                             |
> | ------- | ---------------- | ------------------------------------------------------------------------------------------------------------ |
> | `async` | **加载优先顺序** | 无关紧要。可能在文档还未完全下载前加载执行。如果脚本很小或者来自于缓存，同时文档又足够长，就会发生这种情况。 |
> | `defer` | **文档顺序**     | 在 `DOMContentLoaded` 之前且在文档加载解析之后执行（可能需要等待）。                                         |
>
> 没有脚本的页面应该也是可用的:
>
> 请注意，如果你使用的是 `defer`，那么在脚本加载之前页面都是“可见”的。
>
> 因此，用户可以阅读这个页面内容，但是一些图形组件可能没有准备完成。
>
> 所以，这就需要在页面适当位置添加“加载”进度指示，禁用无效的按钮，以清楚地向用户显示什么准备好了什么没有准备好。
>
>  在开发中，通常在脚本需要整个 DOM 文档或者脚本的相对执行顺序很重要的时候，使用 defer 属性。而当脚本之间互相独立，比如计数器或者广告，并且它们相对执行顺序不重要的时候，此时使用 async 属性。


## 参考

[Scripts: async, defer](https://javascript.info/script-async-defer)
