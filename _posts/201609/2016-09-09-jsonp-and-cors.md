---
layout: post
title: 关于 jsonp 和 CORS 跨域
date: 2016-09-09 17:11:58
categories: javascript
author: Aisin
---

##什么是跨域

跨域问题是因为**同源策略**的存在，同源策略限制了一个源（origin）中加载文本或脚本与来自其它源（origin）中资源的交互方式。（详细参见：[同源策略](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)）

举个例子：

地址：http://news.baidu.com:80/article/2016-09/9423903 从左至右，依次为协议（http）、子域名（news）、主域名（主域名）、端口号（8080）、请求资源地址（article/2016-09/9423903）。而当协议、子域名、主域名、端口号有任意一个不相同，就算是不同域，即**跨域**。这样理解起来就简单多了吧。

##JSONP

`JSONP` 和 `JSON` 不同，`JSON` 是一种数据交互格式，而 `JSONP` 是为了解决跨域问题搞出来的一种获取数据的方式。

JSONP 的原理非常简单，为了克服跨域问题，利用没有跨域限制的 script 标签加载预设的 callback 将内容传递给 js。一般来说我们约定通过一个参数来告诉服务器 JSONP 返回时应该调用的回调函数名，然后拼接出对应的 js。

* jsonp 利用的是 script 标签来实现（毕竟 script 不存在跨域问题）
* 只支持 GET 请求，不支持 POST 请求
* 一般在 URL 中传入一个 callback 的参数，用来调用函数

###实例

我在本地使用 Node.js 搭建了两个不同端口号的环境，用来实现两个不同的域。

请求数据的客户端：

```javascript
// 回调函数
function jsonpCallback(data) {
    console.log("jsonpCallback: " + data.name);
}

$("#submit").click(function() {
    var data = {
        name: $("#name").val(),
        id: $("#id").val()
    };
    $.ajax({
        url: 'http://localhost:9988/api/get-user',
        data: data,
        dataType: 'jsonp',
        cache: false,
        timeout: 5000,
        // jsonp 字段含义为服务器通过什么字段获取回调函数的名称
        jsonp: 'cb',
        // 声明本地回调函数的名称，jquery 默认随机生成一个函数名称
        jsonpCallback: 'jsonpCallback',
        success: function(data) {
            console.log("ajax success callback: " + data.name);
        },
        error: function(jqXHR, textStatus, errorThrown) {
            console.log(textStatus + ' ' + errorThrown);
        }
    });
});
```

提供数据的服务端：

```javascript
exports.getUser = function (req, res, next) {
    // 表示服务端对接收到的数据进行处理
    var nameSer = req.query.name + ' - from Server 9988';
    var idSer = req.query.id + ' - from Server 9988';
    
    var data = "{" + "name: '" + nameSer + "'," + "id: '" + idSer + "'}";
    var cb = req.query.cb;
    var result = cb + '(' + data + ')';

    res.send(result);
    res.end();
}
```

这里一定要注意 data 中字符串拼接，不能直接将 JSON 格式的 data 直接传给回调函数，否则会发生编译错误。

经过上面的事件，你是不是觉得 JSONP 的实现和 Ajax 大同小异？

其实，由于实现的原理不同，由 JSONP 实现的跨域调用不是通过 XmlHttpRequset 对象，而是通过 script 标签，所以在实现原理上，JSONP 和 Ajax 已经一点关系都没有了。看上去形式相似只是由于 jQuery 对 JSONP 做了封装和转换。

比如在上面的例子中，我们假设要传输的数据 data 格式如下：

```javascript
{
    name: "Tim",
    id": "3001"
}
```

那么数据是如何传输的呢？HTTP 请求头的第一行如下：

```javascript
GET /ajax/deal?cb=jsonpCallback&name=Tim&id=3001&_=1473164168520 HTTP/1.1
```

可见，即使形式上是用 POST 传输一个 JSON 格式的数据，其实发送请求时还是转换成 GET 请求。

##CORS

###什么是 CORS？

Cross-Origin Resource Sharing（CORS）跨域资源共享是一份浏览器技术的规范，提供了 Web 服务从不同域传来沙盒脚本的方法，以避开浏览器的同源策略，是 JSONP 模式的现代版。与 JSONP 不同，CORS 除了 GET 要求方法以外也支持其他的 HTTP 要求。用 CORS 可以让网页设计师用一般的 XMLHttpRequest，这种方式的错误处理比 JSONP 要来的好。另一方面，JSONP 可以在不支持 CORS 的老旧浏览器上运作。现代的浏览器都支持 CORS。

请求的客户端使用普通的 GET 或者 POST 请求数据即可，而客户端需要类似这样写：

```javascript
app.post('/cors', function(req, res) {
    
    res.header("Access-Control-Allow-Origin", "*");
    res.header("Access-Control-Allow-Headers", "X-Requested-With");
    res.header("Access-Control-Allow-Methods", "PUT,POST,GET,DELETE,OPTIONS");
    res.header("X-Powered-By", ' 3.2.1')
    res.header("Content-Type", "application/json;charset=utf-8");
    
    var data = {
        name: req.body.name + ' - from Server 9988',
        id: req.body.id + ' - from Server 9988'
    };
    console.log(data);
    res.send(data);
    res.end();
})
```

**CORS 与 JSONP 对比**

* CORS 除了 GET 方法外，也支持其它的 HTTP 请求方法如 POST、 PUT 等。
* CORS 可以使用 XmlHttpRequest 进行传输，所以它的错误处理方式比 JSONP 好。
* JSONP 可以在不支持 CORS 的老旧浏览器上运作。