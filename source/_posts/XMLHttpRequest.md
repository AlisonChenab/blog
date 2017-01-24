---
title: XMLHttpRequest
date: 2016-12-30 10:28:11
tags:
- http
categories:
- http
---
### Ajax & XMLHttpRequest

ajax是一种技术方案,它依赖的是现有的HTML/CSS/Javascript,而其中最核心的依赖是浏览器提供的XMLHttpRequest对象,是这个对象使得浏览器可以发出HTTP请求和接收HTTP响应。即用XMLHttpRequest对象来发送一个Ajax请求。

![what_is_ajax](https://alisonchenab.github.io/images/what_is_ajax.png)

### XMLHttpRequest发展历程

#### level1缺点：

1. 只支持文本数据的传送，无法用来读取和上传二进制文件。
2. 传送和接收数据时，没有进度信息，只能提示有没有完成。
3. 受到"同域限制"，只能向同一域名的服务器请求数据。

#### level2改进点：

1. 可以设置HTTP请求的时限。
2. 可以使用FormData对象管理表单数据。
3. 可以上传文件。
4. 可以请求不同域名下的数据（跨域请求）。
5. 可以获取服务器端的二进制数据。
6. 可以获得数据传输的进度信息。

### XMLHttpRequest使用

```code

function sendAjax(data) {
    // 创建xhr对象
    var xhr = new XMLHttpRequest();
    // 设置xhr请求的超时时间
    xhr.timeout = 3000;
    // 设置响应返回的超时时间
    xhr.responseType = 'text';
    // 创建一个post请求,采用异步
    xhr.open('POST', '/server', true);
    // 注册相关事件回调处理函数
    xhr.onload = function(e){
        if(this.status == 200 || this.status == 304){
            alert(this.responseText);
        }
    };
    xhr.ontimeout = function(e) {}
    xhr.onerror = function(e) {}
    xhr.upload.onprogress = function(e) {}

    // 发送数据
    xhr.send(data);
}

```


