# express middleware

express 是基于 connect 开发的。

## connect

## app.use()

```javascript
var express = require('express');

var app = express();

app.listen(3004, function() {
  console.log('listen 3004...');
});

function middlewareA(req, res, next) {
  console.log('middlewareA before next()');
  next();
  console.log('middlewareA after next()');
}

function middlewareB(req, res, next) {
  console.log('middlewareB before next()');
  next();
  console.log('middlewareB after next()');
}

function middlewareC(req, res, next) {
  console.log('middlewareC before next()');
  next();
  console.log('middlewareC after next()');
}

app.use(middlewareA);
app.use(middlewareB);
app.use(middlewareC);
```

在浏览器访问后， 终端的打印结果为:
```shell
middlewareA before next()
middlewareB before next()
middlewareC before next()
middlewareC after next()
middlewareB after next()
middlewareA after next()
```
