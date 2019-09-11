# js正则表达式【前/后查找】【lookahead and lookbehind】

## tl;dr
* 目的: 匹配这样一个位置，它前面（不）是啥啥啥，后面（不）是啥啥啥

### Lookahead

* Postive lookahead: 正向前查找 
> **[x(?=y)** look for x, but match only if followed by y

```javascript
  let str = "1 turkey costs 30€";
  alert( str.match(/\d+(?=€)/) ); // 30
```

* Negtive lookahead: 负向前查找
> **x(?!y)** search x, but only if not followed by y

```javascript
  let str = "2 turkeys cost 60€";
  alert( str.match(/\d+(?!€)/) ); // 2
```

### Lookbehind

* Postive lookbehind: 正向后查找
> **(?<=y)x** matches x, but only if it follows after y
```javascript
  let str = "1 turkey costs $30";
  alert( str.match(/(?<=\$)\d+/) ); // 30
```

* Negtive lookbehind: 负向后查找
> **(?<!y)x** matches x, but only if there’s no y before
```javascript
  let str = "2 turkeys cost $60";
  alert( str.match(/(?<!\$)\d+/) ); // 2
```

### Capture groups

* lookaround自身不会出现在match结果中，使用括号得以捕获

```javascript
  let str = "1 turkey costs 30€";
  let reg = /\d+(?=(€))/;
  alert( str.match(reg) ); // 30, €
```
```javascript
  let str = "1 turkey costs $30";
  let reg = /(?<=(\$))\d+/;
  alert( str.match(reg) ); // 30, $
```

### Summary

|Pattern|type               |matches               |
|-------|-------------------|----------------------|
|x(?=y) |Positive lookahead |x if followed by y    |
|x(?!y) |Negative lookahead |x if not followed by y|
|(?<=y)x|Positive lookbehind|x if after y          |
|(?<!y)x|Negative lookbehind|x if not after y      |

### 参考

[Lookahead and lookbehind](https://javascript.info/regexp-lookahead-lookbehind)
