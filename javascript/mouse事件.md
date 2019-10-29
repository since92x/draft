# 总结

> 鼠标的`快速移动`可以使 `mouseover, mousemove, mouseout` `跳过`一些中间元素。
>
> mouseover/out 事件和 mouseenter/leave 事件有一个额外的目标：`relatedTarget`。这是作为起点/终点的元素，是对 target 的补充。
> 即使从`父元素转到子元素`时，mouseover/out 也会被触发。它们假设鼠标一次只会移入一个元素 —— 最深的那个。
> `mouseenter/leave 事件不会冒泡`，而且当鼠标进入子元素时也不会被触发。`它们只关注鼠标在整个元素的内部还是外部`。


# 参考

[Moving the mouse: mouseover/out, mouseenter/leave](https://javascript.info/mousemove-mouseover-mouseout-mouseenter-mouseleave)