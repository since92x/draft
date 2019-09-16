# 大家来找茬clientHeight、offsetHeight、scrollHeight、offsetTop、scrollTop

以下单位均为px。

先来个height三连：

## clientHeight

<img src="https://github.com/since92x/draft/blob/master/imgs/client.png?raw=true" style="background:#fff;"/>

> element + padding的高度，只读。(不含border、水平滚动条、margin)

## offsetHeight

<img src="https://github.com/since92x/draft/blob/master/imgs/offset.png?raw=true" style="background:#fff;"/>

> element + padding + border + 水平scrollbar的高度，只读。(不含margin)

## scrollHeight

<img src="https://github.com/since92x/draft/blob/master/imgs/scrollHeight.png?raw=true" style="background:#fff;"/>

> 元素的内容高度（包含溢出而导致的视图中不可见内容）
> 即当内容超过了容器的高度，overflow=scroll时，看上图中所指出的高度，包含了隐藏的内容高度。

其中还涉及到scrollTop。

## scrollTop

> scrollTop是滚动条向下滚动的距离，也就是顶部被不可见部分的高度。(可写)

在介绍offsetTop之前，需要知道offsetParent。

## offsetParent

> 指向包含层级上最近的包含该元素的定位元素。如果没有定位的元素，则 offsetParent 为最近的 table, table cell 或根元素。当元素的 style.display 设置为 "none" 时，offsetParent 返回 null。offsetParent 很有用，因为 offsetTop 和 offsetLeft 都是相对于其内边距边界的。

## offsetTop

> 元素相对于其 offsetParent 元素的顶部内边距的距离。(只读)

对块级元素来说，offsetTop、offsetLeft、offsetWidth 及 offsetHeight 描述了元素相对于 offsetParent 的边界框。

> 然而，对于可被截断到下一行的行内元素（如 span），offsetTop 和 offsetLeft 描述的是第一个边界框的位置（使用 Element.getClientRects() 来获取其宽度和高度），而 offsetWidth 和 offsetHeight 描述的是边界框的尺寸（使用 Element.getBoundingClientRect 来获取其位置）。因此，使用 offsetLeft、offsetTop、offsetWidth、offsetHeight 来对应 left、top、width 和 height 的一个盒子将不会是文本容器 span 的盒子边界。


下面这个例子展示了蓝色边框的 div 包含一个长的句子，红色的盒子是一个可以表示包含这个长句子的span标签的边界。

```html
<div style="width: 300px; border-color:blue; border-style:solid; border-width:1;">
  <span>Short span. </span>
  <span id="long">Long span that wraps withing this div.</span>
</div>
<div id="box" style="position: absolute; border-color: red; border-width: 1; border-style: solid; z-index: 10">
</div>
<script type="text/javascript">
  var box = document.getElementById("box");
  var long = document.getElementById("long");
  // long.offsetLeft这个值就是span的offsetLeft.
  // long.offsetParent 返回的是body（在chrome浏览器中测试）
  // 如果id为long的span元素的父元素div，设置了position属性值，只要不为static,那么long.offsetParent就是div
  box.style.left = long.offsetLeft + document.body.scrollLeft + "px";
  box.style.top = long.offsetTop + document.body.scrollTop + "px";
  box.style.width = long.offsetWidth + "px";
  box.style.height = long.offsetHeight + "px";
</script>
```

参考MDN, 猝。。。

