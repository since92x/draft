# 实现 Promise.all

- 使用 计数器 和 循环

```javascript
Promise.all = ps =>
  new Promise((resolve, reject) => {
    if (ps.length === 0) return resolve([]);
    let [res, cnt] = [[], 0];
    ps.forEach((p, i) => {
      p.then(r => {
        res[i] = r;
        if (++cnt >= ps.length) resolve(res);
      }).catch(reject);
    });
  });
```

- 递归

```javascript
Promise.all = ps => {
  if (ps.length === 0) return resolve([]);
  const [head, ...tail] = ps;
  return head.then(res => {
    return Promise.all(tail).then(ress => [res, ...ress]);
  });
};
```

- async / await

```javascript
Promise.all = async ps => {
  const results = [];
  for (const p of ps) {
    results.push(await p);
  }
  return results;
};
```

- async / await 递归

```javascript
Promise.all = async ps => {
  if (ps.length <= 0) return [];
  const [head, ...tail] = ps;
  return [await head, ...(await Promise.all(tail))];
};
```
