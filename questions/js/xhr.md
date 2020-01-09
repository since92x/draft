#  xhr usage
```javascript
var xhr = new XMLHttpRequest();
xhr.onreadystatechange = function () {
  if (xhr.readyState === 4) {
    if (xhr.status === 200){
      
    }
  }
}
xhr.open('get', 'http://url', true) // true表示异步
xhr.send(null)
```
# promise wrap xhr
```javascript
function $request(url, method) {
	const request = new XMLHttpRequest();
	return new Promise(function (resolve, reject) {
		request.onreadystatechange = function () {
			if (request.readyState !== 4) return;
			if (request.status >= 200 && request.status < 300) {
				resolve(request);
			} else {
				reject({
					status: request.status,
					statusText: request.statusText
				});
			}
		};
		request.open(method || 'GET', url, true);
		request.send();
	});
};
```

# xhr readyState
readyState: 
0: 未调用open
1: 未调用send
2: 未收到响应
3: 收到部分响应
4: 全部收到响应