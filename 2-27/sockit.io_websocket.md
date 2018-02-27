## WebSocket

在 WebSocket 没有出现之前，实现与服务端的实时通讯可以通过轮询来完成任务。WebSocket 是一种浏览器和服务器间进行全双工通讯的网络技术

> 工作流程：

WebSocket 是为解决客户端与服务端实时通信而产生的技术。websocket 协议本质上是一个基于 tcp 的协议，是先通过 HTTP/HTTPS 协议发起一条特殊的 http 请求进行握手后创建一个用于交换数据的 TCP 连接，此后服务端与客户端通过此 TCP 连接进行实时通信。在数据传输的稳定性和数据传输量的大小方面，和传统轮询以技术比较，具有很大的性能优势。

为了建立一个 WebSocket 连接，客户端浏览器首先要向服务器发起一个 HTTP 请求，这个请求和通常的 HTTP 请求不同，包含了一些附加头信息，其中附加头信息”Upgrade: WebSocket”表明这是一个申请协议升级的 HTTP 请求，服务器端解析这些附加的头信息然后产生应答信息返回给客户端，客户端和服务器端的 WebSocket 连接就建立起来了，双方就可以通过这个连接通道自由的传递信息，并且这个连接会持续存在直到客户端或者服务器端的某一方主动的关闭连接。

> 协议规范：

一个典型的 websocket 发起请求到响应请求的例子如下

```js
客户端到服务端：
GET / HTTP/1.1
Connection:Upgrade
Host:127.0.0.1:8088
Origin:null
Sec-WebSocket-Extensions:x-webkit-deflate-frame
Sec-WebSocket-Key:puVOuWb7rel6z2AVZBKnfw==
Sec-WebSocket-Version:13
Upgrade:websocket

服务端到客户端：
HTTP/1.1 101 Switching Protocols
Connection:Upgrade
Server:beetle websocket server
Upgrade:WebSocket
Date:Mon, 26 Nov 2013 23:42:44 GMT
Access-Control-Allow-Credentials:true
Access-Control-Allow-Headers:content-type
Sec-WebSocket-Accept:FCKgUr8c7OsDsLFeJTWrJw6WO8Q=
```

这是一个握手的 http 请求，它与普通的 http 请求有一些区别，

* 首先请求和响应的，”Upgrade:WebSocket”表示请求的目的就是要将客户端和服务器端的通讯协议从 HTTP 协议升级到 WebSocket 协议。

* 从客户端到服务器端请求的信息里包含有”Sec-WebSocket-Extensions”、“Sec-WebSocket-Key”这样的头信息。这是客户端浏览器需要向服务器端提供的握手信息

* 服务器端解析这些头信息，并在握手的过程中依据这些信息生成一个 28 位的安全密钥并返回给客户端，以表明服务器端获取了客户端的请求，同意创建 WebSocket 连接。

* 当握手成功后，这个时候 tcp 连接已在经建立了，客户发送上来的时候就是纯纯的数据了。不过服务端要判断什么时候是一次数据请求的开始，什么时候是请求的结束。

* 在 websocket 中，由于浏览端和服务端已经打好招呼，如我发送的内容为 utf-8 编码，如果我发送 0x00,表示包的开始，如果发送了 0xFF，就表示包的结束了。这就解决了黏包的问题。

如果我们要使用 Websocket，我们需要一个实现 Websocket 协议规范的服务器。websocket 是可以和 http 共用监听端口的，也就是它可以公用端口完成 socket 任务。

## Socket.io

socket.io 将 Websocket 和轮询（Polling）机制以及其它的实时通信方式封装成了通用的接口，并且在服务端实现了这些实时机制的相应代码。

> socket.io 实现的 Polling 中的通信机制：

* Adobe® Flash® Socket

  大部分 PC 浏览器都支持的 socket 模式，不过是通过第三方嵌入到浏览器，不在 W3C 规范内，所以可能将逐步被淘汰，况且，大部分的手机浏览器都不支持这种模式。

* AJAX long polling

  所有浏览器都支持这种方式，就是定时的向服务器发送请求，缺点是会给服务器带来压力并且出现信息更新不及时的现象。

* AJAX multipart streaming

  这是在 XMLHttpRequest 对象上使用某些浏览器（比如说 Firefox）支持的 multi-part 标志。Ajax 请求被发送给服务器端并保持打开状态（挂起状态），每次需要向客户端发送信息，就寻找一个挂起的的 http 请求响应给客户端，并且所有的响应都会通过统一连接来写入

  ```js
  var xhr = $.ajaxSettings.xhr();
  xhr.multipart = true;
  xhr.open("GET", "ajax", true);
  xhr.onreadystatechange = function() {
    if (xhr.readyState == 4) {
      processEvents($.parseJSON(xhr.responseText));
    }
  };
  xhr.send(null);
  ```

* Forever Iframe

  涉及了一个置于页面中的隐藏 Iframe 标签，该标签的 src 属性指向返回服务器端事件的 servlet 路径。每次在事件到达时，servlet 写入并刷新一个新的 script 标签，该标签内部带有 JavaScript 代码，iframe 的内容被附加上这一 script 标签，标签中的内容就会得到执行。这种方式的缺点是接和数据都是由浏览器通过 HTML 标签来处理的，因此你没有办法知道连接何时在哪一端已被断开了，并且 Iframe 标签在浏览器中将被逐步取消使用

* JSONP Polling

  JSONP 轮询基本上与 HTTP 轮询一样，不同之处则是 JSONP 可以发出跨域请求

参考列表：

- [SOCKET.IO，理解 SOCKET.IO](https://www.cnblogs.com/xiezhengcai/p/3957314.html)
- [Websocket 协议的学习、调研和实现](https://www.cnblogs.com/lizhenghn/p/5155933.html)
