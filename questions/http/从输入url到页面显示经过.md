# 神题,涵盖的相当全面，区分度高

## dns 解析

浏览器缓存 -> host -> 系统缓存 -> 路由器缓存 -> 供应商 -> 根 dns 服务器

## 根据 ip 向服务器发起 TCP 连接

## 浏览器发送数据

浏览器设置请求报文：

1.  请求行： GET / HTTP/1.0
2.  请求头： cache-control: no-store cookie: 'XXXXXXXX'
3.  空行
4.  请求体： http://api.com?name=Tom

## 服务器响应

## 通过返回的数据， 渲染页面

1. html -> dom tree
2. css -> cssom tree
3. dom + cssom -> render tree
4. 回流 (位置，大小等几何信息)
5. 重绘 (外观，像素)
6. 展示

## 关闭连接
