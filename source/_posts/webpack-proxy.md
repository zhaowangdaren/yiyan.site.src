---
title: webpack devServer代理解决跨域失败的情况：get请求无origin，而post请求有
date: 2019-02-13 10:49:33
tags:
---

前后后端分离开发，普遍会存在跨域的问题。前端在使用webpack开发时，通常采用代理的方式来解决跨域问题。

通常在webpack的devServer进行如下配置
```js
devServer: {
  ...
  proxy: {
    '/api': {
      target: "http://target.api.domain.com",
      changeOrigin: true,
    }
  },
}
```

如此配置基本可以解决大部分的跨域问题。

但是，在后台将允许的跨域地址指定为一个后，上述配置便对post请求不生效了，也就是post请求跨域失败，而get请求成功。

对比get和post请求的不同，发现get的请求头里面没有`origin`字段，而post请求里面有`origin`字段。**为什么get请求头部没有`origin`字段，而post请求头部有`origin`字段**

在命令行中使用curl工具请求接口，header中不设置`origin`字段请求成功，一旦设置为本机开发服务IP地址和端口，post请求就失败。

在webpack的devServer采用`http-proxy-middleware`来代理转发请求，配置中`changeOrigin`的作用是： changes the origin of the host header to the target URL。

`changeOrigin: true`为什么没有生效？没有进行抓包分析。

`http-proxy-middleware`的配置中还有一个`headers`的配置，将上述配置修改为：
```js
devServer: {
  ...
  proxy: {
    '/api': {
      target: "http://target.api.domain.com",
      changeOrigin: true,
      headers: {
        host: "http://target.api.domain.com",
        origin: "http://target.api.domain.com"
      }
    }
  },
}
```
成功

那么问题来了：**为什么get请求头部没有`origin`字段，而post请求头部有`origin`字段？**