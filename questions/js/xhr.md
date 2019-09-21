```javascript
var xhr = new XMLHttpRequest();
xhr.onreadystatechange = function () {
  if (xhr.readyState === 4) {
    if (xhr.status === 200){
      
    } else {

    }
  }
}
xhr.open('get', 'http://url', true) // true表示异步
xhr.send(null)
```

readyState: 
0: 未调用open
1: 未调用send
2: 未收到响应
3: 收到部分响应
4: 全部收到响应