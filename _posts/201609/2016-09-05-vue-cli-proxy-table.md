---
layout: post
title: Vue-cli 的 proxyTable 解决跨域问题
date: 2016-09-05 10:21:18
categories: javascript
tags: vue
author: Aisin
---

最近，在本地使用 Vue 开发了一个知乎日报的 Web 版，在调用知乎提供的 Api 时，遇到了跨域的问题。于是尝试使用 jsonp 来解决，但是发现并不可行。

后来在看 Vue-cli 的源码时，看到配置文件里有 proxyTable 属性，感觉应该和代理或者跨域有关。在网上查了一下，果然可以解决这个问题。

在 vuejs-templates，也就是 Vue-cli 的使用的模板插件里，有关于 API proxy 的说明，使用的就是这个参数，介绍详情如下：[API Proxying During Development | Introduction](https://vuejs-templates.github.io/webpack/proxy.html)

这个参数主要是一个地址映射表，你可以通过设置将复杂的 url 简化，例如我们要请求的地址是 `example.com/api/lists`，可以按照如下设置：

```javascript
proxyTable: {
  '/api': {
    target: 'http://example.com/api',
    changeOrigin: true,
    pathRewrite: {
      '^/api': '/api'
    }
  }
}
```

这样我们在调用 Api 时，写 url 只需写成 /api/lists 就可以代表 example.com/api/lists

那么又是如何解决跨域问题的呢？其实在上面的 /api 参数里有一个 `changeOrigin` 参数，接收一个布尔值，如果设置为 true ，那么本地会虚拟一个服务端接收你的请求并代你发送该请求，这样就不会有跨域问题了，当然这只适用于开发环境。

Vue-cli 的这个设置来自于其使用的插件 `http-proxy-middleware`

github：https://github.com/chimurai/http-proxy-middleware

深入了解的话可以看该插件配置说明，似乎还支持 websocket，很强大的插件。