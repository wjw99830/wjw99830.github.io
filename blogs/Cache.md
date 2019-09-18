<small>最后编辑于2019-09-18</small>

# 前端性能优化 - 缓存
## 本地数据存储（WebStorage）
会话级存储：SessionStorage。在一个窗口关闭后数据清空，多个窗口不会共享数据，即使同源。
持久化存储：LocalStorage。除非手动清空数据，否则数据将会一直存在，以域的级别隔离数据。
两种存储的API基本相同：
```
Storage.get
Storage.set
Storage.clear
...
```
## Memory Cache
浏览器针对同个文档中的多个相同资源请求进行的优化，无相关标准定义其行为。
## Cache API
借助Service Worker对请求进行拦截，在回调函数中可查看请求的资源是否命中缓存，如果没有可以继续发起请求，否则直接返回缓存，该缓存也是持久化的，类似LocalStorage。
## HTTP 缓存
### 强缓存
HTTP响应头中使用Cache-Control字段，其中可以设置max-age指定缓存有效时间，使用Expires指定一个过期时间，浏览器会将其与本地时间进行对比决定是否使用缓存。
### 协商缓存
响应头增加Last-Modified字段表示最后一次资源更新的时间，下次请求时在头部中增加If-Modified-Since字段（其值为Last-Modified的值），若服务端返回304则使用缓存，否则返回200更新资源。
响应头增加E-Tag字段作为资源内容的标识（MD5），请求时将其值作为字段If-None-Match，若值不匹配则更新资源。
## Push Cache
Push Cache是HTTP/2的一个特性，在获取文档的同时发送一些后续会使用到的资源，例如css、js，这种方式的缓存有效时间较短。