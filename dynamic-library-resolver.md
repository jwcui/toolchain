# 要解决的问题

* 给运行时库一个名字，通过名字找到运行时环境里的库
* 在运行时库的多个库里定位一个合适的

# 解决方案

动态链接库不会直接从远程加载，一般都要保留在本地的硬盘上。每个机器的安装动态链接库的位置不同，所以一般是不会用硬盘的文件绝对路径去加载动态链接库的。标识并定位的解决方案就是提供一个安装环境无关的动态链接的过程。

## 构成

* dynmaic library name：动态链接库的名字
* ld library path：加载链接库要在哪里查找。最常用的是动态链接发起的文件所在路径，做为查找相对路径的参照
* resolver：根据 dynamic library name 从 ld library path 里查找动态链接库

## 衍生的问题

* 多个 ld library path 都可以找到同一个动态链接库，应该用哪个?

# 解决方案案例

## 传统浏览器中的 JavaScript

* dynamic library name：http/https 绝对 url 或者相对 url
* ld library path：加载 script 所在网页的 url
* resolver：浏览器自身

相对路径

```html
// http://localhost/a/b.html
<html>
<head>
<script src="c/d.js"></script>
</head>
...
</html>
```

`c/d.js` 相对 `http://localhost/a/b.html` 解析出来就是 `http://localhost/a/c/d.js`

提供 `//url` 做为相对参照。如果当前网页是 https，则 `//` 相当于 `https://`

```html
// https://localhost/a/b.html
<html>
<head>
<script src="//some-cdn.com/c/d.js"></script>
</head>
...
</html>
```

解析出来就是 `https://some-cdn.com/c/d.js`


