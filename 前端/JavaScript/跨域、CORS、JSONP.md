# 跨域、CORS、JSONP

## ★跨域关键知识

- 同源策略
  - 浏览器故意设计的一个功能限制
- CORS
  - 突破浏览器限制的一个方法
- JSONP
  - IE 时代的妥协

## ★同源策略

1）何为同源

- 源
  - 源 = 协议 + 域名 + 端口号
  - `window.origin` 或 `location.origin` 可以得到当前源，如它们的返回值是`"https://www.yuque.com"`
- 同源
  - 若两个 url 的协议、域名、端口号都完全一致，则这二者是同源的
- 举例
  - 完全一致才是同源
  - `https://qq.com` 与 `https://baidu.com` 不同源
  - `https://baidu.com` 与 `https://www.baidu.com` 不同源

2）有何策略

3）假如没有同源策略

