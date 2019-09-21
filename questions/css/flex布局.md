# 容器属性：

```css
.container {
  justify-content: space-around;
  align-content: center;
  align-items: flex-start;
  flex-direction: column-reverse;
  flex-wrap: nowrap;
  /* flex-flow 是 direction 和 wrap 的缩写 */
}
```

# item 的属性：

```css
.item {
  order: -1;
  flex-grow: 1;
  flex-shrink: 0;
  flex-basis: 100px;
  /* flex 是 放大， 缩小， basis 的缩写 */
  align-self: flex-end;
}
```
