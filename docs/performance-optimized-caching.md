# 要解决的问题

	如何从缓存方面性能优化

# 解决方案
	缓存是web提高性能重要的一项技术.缓存可以有效的降低网络的加载时长,减轻服务器的压力. 作为一名工程师, 首要的责任是给用户带来极好的体验. 有效利用缓存是提高用户体验的一种有效方法.

# 缓存内容讲解
	
## cdn 缓存

### 原理

CDN的基本原理是将这些缓存服务器分不到用户访问相对集中的地区或网络中, 在用户访问网站时,利用全局负载技术将用户的访问指向距离最近的缓存服务器上,由缓存服务器直接响应用户请求. 

### 过程

 * 用户在浏览器中输入要访问的域名。
 * 浏览器向DNS服务器请求对域名进行解析。由于CDN对域名解析进行了调整，DNS服务器会最终将域名的解析权交给CNAME指向的CDN专用DNS服务器。
 * CDN的DNS服务器将CDN的负载均衡设备IP地址返回给用户。
 * 用户向CDN的负载均衡设备发起内容URL访问请求。
 * CDN负载均衡设备会为用户选择一台合适的缓存服务器提供服务。

```
	选择的依据包括：根据用户IP地址，判断哪一台服务器距离用户最近；根据用户所请求的URL中携带的内容名称，判断哪一台服务器上有用户所需内容；查询各个服务器的负载情况，判断哪一台服务器的负载较小。
	基于以上这些依据的综合分析之后，负载均衡设置会把缓存服务器的IP地址返回给用户。
```
* 用户向缓存服务器发出请求。
* 缓存服务器响应用户请求，将用户所需内容传送到用户。


## 浏览器缓存

### 原理: 

> 浏览器在加载资源时，先根据这个资源的一些http header判断它是否命中强缓存，如果命中，浏览器直接从自己的缓存中读取资源，不会发请求到服务器.
> 当强缓存没有命中的时候，浏览器一定会发送一个请求到服务器，服务器端依据资源的另外一些http header验证这个资源是否命中协商缓存，
> 如果命中，服务器会将这个请求返回，但是不会返回这个资源的数据，而是告诉客户端可以直接从缓存中加载这个资源，于是浏览器还是从自己的缓存中加载资源，当协商缓存也没有命中的时候，浏览器直接从服务器加载资源数据。 

### 协商缓存
* If-Modified-Since/Last-modified: 服务器程序检查请求头(request header)里面的(If-modified-Since)，如果最后修改时间相同(例如静态文件的Modified time 通过shell ls -l可以查看)则返回304，否则给返回头(response header)添加last-Modified并且返回数据(response body)。
	
* If-None-Match/Etag：服务器程序检查检查请求头(request header)里面的if-none-match的值与当前文件的内容通过hash算法（例如 nodejs: cryto.createHash('sha1')）生成的内容摘要字符对比，相同则直接返回304，否则给返回头(response header)添加etag属性为当前的内容摘要字符，并且返回内容。(if-none-match的值是上一次浏览器响应头返回的ETag的值.)

### 强缓存

* request header cache-control：存在以下值
	1. no-cache: 请求不会直接使用缓存
	2. no-store: 指示缓存不存储此次请求的响应部分
	3. max-age: 表示此次请求成功后N秒之内发送同样请求不会去服务器重新请求，而是使用本地缓存
* response header expires: 跟cache-control中的max-age作用一样，不过在碰见max-age之后，该值会被覆盖从而被max-age替代

> expires/cache-control 虽然是强缓存，但用户主动触发的刷新行为，还是会采用缓存协商的策略，主动触发的刷新行为包括点击刷新按钮、右键刷新、f5刷新、ctrl+f5刷新等。
> 当然如果在控制台里面选中了disable cahce则无论如何都会请求最新内容(304协商缓存、强缓存都无效)，因为
> 1.不会检查本地是否有缓存。
> 2.请求头信息(request header)既没有If-Modified-Since也没有If-None-Match来让服务端判断。
> 地址栏输入的地址按下回车键，该页面请求（仅仅是该url）的request header都会带上cache-contro:max-age=0，所以不会命中缓存。
> 但是通过链接点击的地址会命中缓存。

### 如何清理缓存呢？
 因为浏览器根据url来判断进行缓存，1、修改文件名称。2、加时间戳。

## 实现
* 静态文件（非HTML/php等页面文件）cache-control设为31536000
* 静态文件（非HTML/php等页面文件）文件通过md5码形式文件名或者时间戳query来更新文件
* 静态文件（非HTML/php等页面文件）接入cdn缓存

## 收益
* 服务端：降低了静态服务器的压力(需OP统计)
* 客户端：提升页面加载渲染速度(节省304协商询问时间)





