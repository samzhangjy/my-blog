---
title: TCP｜你真的懂 HTTP 吗？
desc: HTTP 相信是每个 Web 开发者都耳熟能详的名词了。但是，新手开发者想要完全理解 HTTP 协议却需要时间。这期视频，我就来带大家入门 HTTP 协议。话不多说，我们直接进入正题！
cover: https://s2.loli.net/2022/03/21/iQkApHORYLGym9o.png
date: 2022.03.20
tags:
  - web
  - http
  - 文字稿
---

此为我在 B 站上发布的视频 [TCP | 你真的懂 HTTP 吗？](https://www.bilibili.com/video/BV1RU4y1R7Kv) 的文字稿。

<iframe src="//player.bilibili.com/player.html?aid=682385745&bvid=BV1RU4y1R7Kv&cid=554442259&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>

## 前言

Hello 大家好，我是 Sam Zhang。

HTTP 相信是每个 Web 开发者都耳熟能详的名词了。但是，新手开发者想要完全理解 HTTP 协议却需要时间。这期视频，我就来带大家入门 HTTP 协议。

话不多说，我们直接进入正题！

## TCP / IP 模型

在真正讲解 HTTP 协议之前，我们先来看一下更为底层的 TCP / IP 模型。

### TCP / IP 模型的四个层次

![TCP / IP 模型](https://tva1.sinaimg.cn/large/e6c9d24ely1h09fe7w3qaj208x08xt90.jpg)

正如大家所看到的，TCP / IP 模型分为四层，由下到上依次是：

1. 链路层（Link Layer）
2. 网络层（Internet Layer）
3. 传输层（Transport Layer）
4. 应用层（Application Layer）

#### 链路层

链路层处于 TCP / IP 模型的最底层。它负责管理数据如何与物理媒介进行交互，例如 Wi-Fi 定义了数据如何进行无线传输。

#### 网络层

链路层只能在局域网中工作，而网络层定义了 IP 协议，主要负责寻址操作。

#### 传输层

传输层主要负责提供应用程序接口，为应用程序提供接入网络的途径。除此以外，这一层还出现了端口，也就是客户端与服务器连接的频道。

最主要的是，这一层有可靠传输的 TCP 协议。通过 TCP 协议传输的数据都能保证按照顺序抵达，内容正确且没有重复。大多数网络请求都是通过 TCP 传输的。

#### 应用层

这一层包括了主要为应用软件使用的协议，例如负责文件传输的 FTP 协议，负责电子邮件的 SMTP 协议和负责域名系统的 DNS 等。最经常出现的 HTTP / HTTPS 协议也是在这一层下的，在浏览器中使用非常广泛。WebSocket 协议也同属于这一层。

## HTTP / HTTPS 协议

### 基础概念

![A Web document is the composition of different resources](https://mdn.mozillademos.org/files/13677/Fetching_a_page.png)

> HTTP（**HyperText Transfer Protocol**）是万维网（World Wide Web）的基础协议。自 Tim Berners-Lee 博士和他的团队在1989-1991年间创造出它以来，HTTP已经发生了太多的变化，在保持协议简单性的同时，不断扩展其灵活性。如今，HTTP已经从一个只在实验室之间交换文件的早期协议进化到了可以传输图片，高分辨率视频和3D效果的现代复杂互联网协议。

HTTP 从 HTTP / 0.9 的单行协议演变到现在 HTTP / 2 的数据帧，HTTP 协议在不断地变化并发展着。这个最初从 CERN 实验室中走出来的小东西，最终被几乎所有人每天每刻地使用着。这，就是我们今天的主角 —— HTTP 协议。

HTTP 是一个 client-server 协议。也就是说，请求由客户端发出，被服务器处理后返回一个响应。在这之间，可能存在着许多代理 - 例如网关。这些代理是建立在应用层上的。代理的作用包括缓存，过滤，负载均衡等功能。

### HTTP 设计理念

#### HTTP 是简单的

HTTP 的设计理念是简单易读 —— 这就使得 HTTP 报文能被人类读懂，而不只是机器。

#### HTTP 是可扩展的

请求头的存在使得扩展 HTTP 变得更简单了。只要服务端和客户端对某一个 header 达成一致的语义，那么这个新 header 的功能就可以很轻松被接入进来。

#### HTTP 是无状态，有会话的

HTTP 本身是无状态的。即使在同一连接中，两个请求之间也是没有关系的。但是，通过 Cookies 创建一个会话，就能够让请求共享同样的上下文信息。

也就是说，HTTP 本身是无状态的，但因为 Cookies ，你可以在 HTTP 中创建有状态的会话。

#### HTTP 和连接

HTTP 在底层使用 TCP 传输协议，而不是 UDP 协议 —— 因为 TCP 是可靠的，不丢失消息的。

#### 数据 URL

数据 URL 是一种特殊的 URL 。他使用 `data:` 协议，允许嵌入小文件。

它的定义形式如下：

```
data:[<mediatype>][;base64],<data>
```

其中 `mediatype` 是一个 MIME 类型的字符串，用于判断数据的类型。默认为 `text/plain;charset=US-ASCII` ，即 ASCII 编码的纯文本文档。

除此以外，你还可以选择使用 `base64` 编码来存储数据。这在存储二进制文件（例如视频和图片）时非常有用。加入 `;base64` 标识符即可存储 base64 数据。

例如，`data:,Hello%20World!` 就是一个纯文本类型数据，而 ``data:text/plain;base64,SGVsbG8sIFdvcmxkIQ%3D%3D`` 则是刚刚 Hello World 的 `base64` 形式。

除此以外，还有几点注意事项：

1. 虽然 Firefox 支持无限长度的 `data` URLs，但是标准中并没有规定浏览器必须支持任意长度的 `data`URIs。比如，Opera 11浏览器限制 URLs 最长为 65535 个字符，这意味着 data URLs 最长为 65529 个字符。
2. 因为数据 URL 是没有结束标记的，所以如果你尝试使用 Query String 添加查询字符串会导致浏览器认为加入的查询字符串也是数据的一部分。
3. data URL 的格式虽然很简单，但 `<data>` 前面的逗号 `,` 却很容易被人忽略，从而导致错误。

#### MIME 类型

MIME（**Multipurpose Internet Mail Extensions**）类型，又称媒体类型，用来表示特定数据的性质和格式。

一个 MIME 类型的结构很简单：

```
type/subtype
```

其中 `type` 代表类别，`subtype` 代表子类别。注意，MIME 类型中不允许空格存在。

虽然 MIME 类型不区分大小写，但传统写法都是小写。

常见的类型有 `text` , `image` , `audio` , `video` , `application` 等。每个类别下还有不同的子类别，可谓种类繁多，绝对能找到你想要的。

刚才说的那几个类型都是**独立**类型，还有另外一种 Multipart 类型。它通常在 HTML 表单中出现，类型是 `multipart/form-data` 。

更详细的 MIME 类型表可以在 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types) 中找到。

### 请求

一个典型的 HTTP 请求应该包括以下几个部分：

1. 请求行
2. 请求头
3. 请求体，如果有的话

#### 请求行

HTTP 请求的启始行叫做请求行。它由下面几个部分组成：

```
<命令> <路径> <版本>
```

例如：`GET /user/register.html HTTP/1.1`

1. `<命令>` ：是 HTTP 请求方式的其中之一，常见的有 `GET` 、`POST` 、`PUT` 等。
2. `<路径>` ：要获取的资源等路径，通常是上下文中就很明显的元素资源的 URL 。*需要注意的是，它没有 `protocol`（`https://`），`domain`（`samzhangjy.github.io`），抑或是 TCP 协议的 `port`（`80`）。*
3. `<版本>` ：HTTP 协议版本号。

#### 请求头

请求头是一行不区分大小写的字符串。它遵循如下规范：

```
<请求头名称>: <请求头值>
```

举个栗子：

```
Accept-Language: zh-CN, en-US
```

这个 header 表示客户端接受简体中文和英文。HTTP 协议支持许多不同的请求头，大致分类有：

- [General headers](https://developer.mozilla.org/zh-CN/docs/Glossary/General_header)：同时适用于请求和响应消息，但与最终消息主体中传输的数据无关的消息头。
- [Request headers](https://developer.mozilla.org/zh-CN/docs/Glossary/Request_header)：包含更多有关要获取的资源或客户端本身信息的消息头。
- [Response headers](https://developer.mozilla.org/zh-CN/docs/Glossary/Response_header)：包含有关响应的补充信息，如其位置或服务器本身（名称和版本等）的消息头。
- [Entity headers](https://developer.mozilla.org/zh-CN/docs/Glossary/Entity_header)：包含有关实体主体的更多信息，比如主体长度 (Content-Length) 或其 MIME 类型。

完整的列表可在 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers) 查看。

#### 请求体

不是所有的请求都必须有请求体：获取数据的请求，例如 `GET` , `HEAD` , `DELETE` , `OPTIONS` 等，通常都不需要请求体。某些请求可能需要上传数据到服务器（最典型的例子就是 `POST` 请求），就需要有请求体来装载数据，例如 `FormData` 。

请求体可以大致分为两种：

- Single-resource bodies，由一个单文件组成。该类型的请求体由两个 header 定义：`Content-Type` 和 `Content-Length` 。
- Multiple-resouce bodies，由多部分 body 组成，每一部分包含不同的信息位，通常和 HTML 表单数据结合在一起。

### 响应

HTTP 响应，跟 HTTP 请求一样，分为三个部分：状态行，响应头和响应体。

#### 状态行

HTTP 响应的第一行就是状态行（status line）。它的模式是：

```
<版本> <状态码> <状态文本>
```

例如：`HTTP/1.1 200 OK` 。

1. `<版本>` ：协议版本，通常情况下是 `HTTP/1.1` 。
2. `<状态码>` ：表明请求的状态，成功还是失败等。常见的有 200 ，404 ，403 等。
3. `<状态文本>` ：一条描述当前状态码的简短明确的文本。

#### 响应头

跟 HTTP 请求头的格式完全一样，HTTP 响应头也不区分大小写并使用逗号和冒号分割数据。整个 header（包括其值）表现为单行形式。

有许多响应头可用，这些响应头可以分为几组：

- *General headers*，例如 [`Via`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Via)，适用于整个报文。
- *Response headers*，例如 [`Vary`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Vary) 和 [`Accept-Ranges`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Accept-Ranges)，提供其它不符合状态行的关于服务器的信息。
- *Entity headers*，例如 [`Content-Length`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Length)，适用于请求的 body。显然，如果请求中没有任何 body，则不会发送这样的头文件。

更多响应头可以在 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers) 找到。

#### 响应体

响应体是可选的，例如有着 HTTP 状态码 201 或 204 的请求大多数情况下不会有响应体。

常用的 Single-resource bodies 响应体可分为两类：

- 由已知长度的单个文件构成的，由 `Content-Type` 和 `Content-Length` 响应头定义。
- 由未知长度的单个文件构成的，通过将 `Transfer-Encoding` 设为 `chunked` 来使用 chunks 编码传输。

属此以外，还有 Multiple-resource bodies ，由多部分响应体组成，每部分包含不同的信息段，但十分少见。

### HTTP / 2

HTTP / 2 修复了 HTTP / 1.1 的许多性能问题：例如在 HTTP / 2 中，可以压缩 header 了。如果你想用 HTTP / 2 协议，你完全不需要更改你的任何 API - 当用户的浏览器和你的服务器都支持 HTTP / 2 时，它会自动取代 HTTP / 1.1 。

如果你想要了解 HTTP / 2 的具体优化，请参考 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Messages#http2_帧) 。

### 认证

![img](https://tva1.sinaimg.cn/large/e6c9d24ely1h0anyizllbj20jq09b3yw.jpg)

一个初始 HTTP 请求是不包括任何认证信息的，可以被认为是一个匿名请求。当服务器接收到此请求且客户端要访问的路由是受保护的，服务器就会返回 401 状态码，并使用 `WWW-Authenticate` 响应头告知客户端需要进行相应的认证。下面是它的语法形式：

```
WWW-Authenticate: <type> realm=<realm>
```

其中，`<type>` 表示验证方案（例如 `Basic` ）。`<realm>` 是一段用于描述进行保护区域的文字，例如 Access to the staging site ，使用户能够知晓他们正在访问的空间。

这时，当客户端（浏览器）接收到此消息后，会弹出密码框，让用户输入密码。输入完成后，客户端将所获的认证信息通过 `Authorization` 请求头传回服务器。

服务器验证当前认证信息后，会根据实际情况返回数据。

一个最基础的 HTTP 认证方式是 Basic 认证方式（基本验证方案）。

#### Basic Authentication

基本验证方案使用用户的 ID 或密码作为认证信息，并使用 base64 算法对其进行编码。

但是，由于 base64 算法的可逆性，基本验证方案并不安全。如果在非 HTTPS 的请求中使用会导致信息泄漏等安全风险。由此，Basic 认证方式不应该用来传输敏感的信息。

### 缓存

HTTP 缓存允许用户复用之前的资源来显著提高性能。

当缓存发现当前请求已经被存储，就会自动拦截请求并返回存储的资源。这跳过了请求源服务器的步骤，大大提升了访问速度。However，不是所有的缓存都是永久不变的：我们需要对缓存的截止时间作出相应的调整，使其不缓存过期的资源。

缓存分为两种，一种是私有的浏览器缓存，另一种是共享的代理缓存。

浏览器缓存，顾名思义，就是将缓存直接存储在浏览器中。而代理缓存，就是在目标服务器和用户之间添加一个代理。它将会把经常访问的资源缓存，并以此减少网络拥堵。

虽然 HTTP 缓存不是必须的，但重用缓存的资源通常是必要的。常见的 HTTP 缓存只能存储 GET 请求的相应，对其他类型的请求稍显无能为力。

客户端可以通过设置 `Cache-Control` 请求头来控制缓存。

`Cache-Control` 的常见取值由 `no-store` , `no-cache` , `max-age=<seconds>` , `must-revalidate` , `private` , `public` 等。

其中，`no-store` 代表不进行任何缓存，`no-cache` 代表缓存但重新验证是否过期，`max-age` 代表缓存保持 fresh 状态的最长时间。

更详细的缓存的信息可以在 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Caching) 找到。

### Cookie

正如我们一开始所说的，HTTP 是无状态的。但是我们可以通过 Cookies 创建有状态的会话。

服务端可以直接使用 `Set-Cookie` 响应头来给客户端发送 Cookie 信息。一个最基础的 Cookie 可以这样设置：

```
Set-Cookie: <name>=<value>
```

客户端会将服务器给定的值存储到本地。Cookies 一旦存储，每次请求客户端都会发送 `Cookies` 请求头给服务器：

```
Cookie: <name1>=<value1>; <name2>=<value2>; ...
```

#### Cookies 的生命周期

Cookie 的生命周期可以通过 `Expires` 或 `Max-Age` 设置。E.g:

```
Set-Cookie: fruit=orange; Expires=Wed, 21 Oct 2022 07:28:00 GMT;
```

更多信息请参考 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies) 。

## 参考资料

### 文档

- [TCP/IP四层模型](https://zhuanlan.zhihu.com/p/57936826), [勤劳的小手](https://www.zhihu.com/people/duan-pan-ykjym), 2021.
- [HTTP概述](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Overview), MDN contributors, 2022.
- [Data URLs](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/Data_URIs), MDN contributors, 2022.
- [HTTP响应](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Messages), MDN contributors, 2022.
- [HTTP基础](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types), MDN contributors, 2022.
- [HTTP缓存](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Caching), MDN contributors, 2022.
- [HTTP Cookies](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies), MDN contributors, 2022.

### BGM

- [Creative Minds](https://www.bensound.com/royalty-free-music/track/creative-minds), *Benjamin Tissot* .
- [Punky](https://www.bensound.com/royalty-free-music/track/punky), *Benjamin Tissot* .
- [Energy](https://www.bensound.com/royalty-free-music/track/energy), *Benjamin Tissot* .