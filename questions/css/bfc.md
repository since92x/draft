# 什么是BFC?

* 一个独立渲染区域的盒子

# BFC排列规则？

* 同一bfc 盒子纵向排列
* 垂直距离margin决定
* 同一bfc margin合并
* 同一bfc 盒子的margin-left挨着容器的border-left
* bfc不和浮动元素重叠
* 计算bfc高度时，浮动元素也参与计算

# 触发条件：

* 根body
* float不是none
* 绝对定位
* overflow不是visible
* display: inline-block, inline-flex, flex, table-cell 

# 应用

* bfc不和浮动元素重叠:
> 自适应两栏布局
```scss
.float-bro {
  float: left;
}
.bfc-bro {
  overflow: hidden;
}
```

* 计算bfc高度时，浮动元素也参与计算:
> 高度塌陷, BFC包含块的高度包含浮动元素。

```scss
.bfc-container {
  overflow: hidden;
  .inner-element {
    float: left;
    height: 50px;
  }
}
```