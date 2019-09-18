<small>最后编辑于2019-09-18</small>

# 前端性能优化 - 网络请求

## DNS Prefetch
```html
<link rel="dns-prefetch" href="//example.com" />
```
## TCP Preconnect
TCP建立连接的时间较长，因此可以在浏览器空闲时预先建立TCP连接，可以通过crossorigin设置是否跨域
```html
<link rel="preconnect" href="//example.com" />
```
## Resouce Prefetch
浏览器会在空闲时请求以后可能用到的资源
```html
<link rel="prefetch" href="xxx.js" />
```
## 预加载 Preload
在网络空闲时提前请求当前页面即将使用到的资源，可以指定优先级，且不会阻塞window的load事件，资源在加载完后不会立即执行，可以用HTTP响应头的方式实现预加载 ```Link: <xx.js>; as="script";rel="preload"```

如果不指定as的值，将会等同于普通的XHR请求，优先级很低，as的值为script、style、image、media、document、font等
```html
<link rel="preload" href="xxx.js" as="script" />
```