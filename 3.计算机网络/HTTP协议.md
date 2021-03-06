# HTTP协议

## HTTP 简介

HTTP协议是Hyper Text Transfer Protocol（超文本传输协议）的缩写,是用于从万维网（WWW:World Wide Web ）服务器传输超文本到本地浏览器的传送协议。。

HTTP是一个基于TCP/IP通信协议来传递数据（HTML 文件, 图片文件, 查询结果等）。

------

## HTTP 工作原理

HTTP协议工作于客户端-服务端架构上。浏览器作为HTTP客户端通过URL向HTTP服务端即WEB服务器发送所有请求。

Web服务器有：Apache服务器，IIS服务器（Internet Information Services）等。

Web服务器根据接收到的请求后，向客户端发送响应信息。

HTTP默认端口号为80，但是你也可以改为8080或者其他端口。

**HTTP三点注意事项：**

- HTTP是无连接：无连接的含义是限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开连接。采用这种方式可以节省传输时间。
- HTTP是媒体独立的：这意味着，只要客户端和服务器知道如何处理的数据内容，任何类型的数据都可以通过HTTP发送。客户端以及服务器指定使用适合的MIME-type内容类型。
- HTTP是无状态：HTTP协议是无状态协议。无状态是指协议对于事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就较快。

以下图表展示了HTTP协议通信流程：

![cgiarch](https://gitee.com/stiwen/images_bed/raw/master/img/cgiarch.gif)

## HTTP 消息结构

HTTP是基于客户端/服务端（C/S）的架构模型，通过一个可靠的链接来交换信息，是一个无状态的请求/响应协议。

一个HTTP"客户端"是一个应用程序（Web浏览器或其他任何客户端），通过连接到服务器达到向服务器发送一个或多个HTTP的请求的目的。

一个HTTP"服务器"同样也是一个应用程序（通常是一个Web服务，如Apache Web服务器或IIS服务器等），通过接收客户端的请求并向客户端发送HTTP响应数据。

HTTP使用统一资源标识符（Uniform Resource Identifiers, URI）来传输数据和建立连接。

一旦建立连接后，数据消息就通过类似Internet邮件所使用的格式[RFC5322]和多用途Internet邮件扩展（MIME）[RFC2045]来传送。

------

### 客户端请求消息

客户端发送一个HTTP请求到服务器的请求消息包括以下格式：请求行（request line）、请求头部（header）、空行和请求数据四个部分组成，下图给出了请求报文的一般格式。![img](https://gitee.com/stiwen/images_bed/raw/master/img/2012072810301161.png)

### 服务器响应消息

HTTP响应也由四个部分组成，分别是：状态行、消息报头、空行和响应正文。

![img](https://gitee.com/stiwen/images_bed/raw/master/img/httpmessage.jpg)

------



### 实例

下面实例是一点典型的使用GET来传递数据的实例：

客户端请求：

```
GET /hello.txt HTTP/1.1
User-Agent: curl/7.16.3 libcurl/7.16.3 OpenSSL/0.9.7l zlib/1.2.3
Host: www.example.com
Accept-Language: en, mi
```

服务端响应:

```
HTTP/1.1 200 OK
Date: Mon, 27 Jul 2009 12:28:53 GMT
Server: Apache
Last-Modified: Wed, 22 Jul 2009 19:15:56 GMT
ETag: "34aa387-d-1568eb00"
Accept-Ranges: bytes
Content-Length: 51
Vary: Accept-Encoding
Content-Type: text/plain
```

输出结果：

```
Hello World! My payload includes a trailing CRLF.
```

## HTTP 请求方法

根据 HTTP 标准，HTTP 请求可以使用多种请求方法。

HTTP1.0 定义了三种请求方法： GET, POST 和 HEAD方法。

HTTP1.1 新增了六种请求方法：OPTIONS、PUT、PATCH、DELETE、TRACE 和 CONNECT 方法。

| 序号 | 方法    | 描述                                                         |
| :--- | :------ | :----------------------------------------------------------- |
| 1    | GET     | 请求指定的页面信息，并返回实体主体。                         |
| 2    | HEAD    | 类似于 GET 请求，只不过返回的响应中没有具体的内容，用于获取报头 |
| 3    | POST    | 向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST 请求可能会导致新的资源的建立和/或已有资源的修改。 |
| 4    | PUT     | 从客户端向服务器传送的数据取代指定的文档的内容。             |
| 5    | DELETE  | 请求服务器删除指定的页面。                                   |
| 6    | CONNECT | HTTP/1.1 协议中预留给能够将连接改为管道方式的代理服务器。    |
| 7    | OPTIONS | 允许客户端查看服务器的性能。                                 |
| 8    | TRACE   | 回显服务器收到的请求，主要用于测试或诊断。                   |
| 9    | PATCH   | 是对 PUT 方法的补充，用来对已知资源进行局部更新 。           |

## HTTP状态码

当浏览者访问一个网页时，浏览者的浏览器会向网页所在服务器发出请求。当浏览器接收并显示网页前，此网页所在的服务器会返回一个包含HTTP状态码的信息头（server header）用以响应浏览器的请求。

HTTP状态码的英文为HTTP Status Code。

下面是常见的HTTP状态码：

- 200 - 请求成功
- 301 - 资源（网页等）被永久转移到其它URL
- 404 - 请求的资源（网页等）不存在
- 500 - 内部服务器错误

### HTTP状态码分类

HTTP状态码由三个十进制数字组成，第一个十进制数字定义了状态码的类型，后两个数字没有分类的作用。HTTP状态码共分为5种类型：

| 分类 | 分类描述                                       |
| :--- | :--------------------------------------------- |
| 1**  | 信息，服务器收到请求，需要请求者继续执行操作   |
| 2**  | 成功，操作被成功接收并处理                     |
| 3**  | 重定向，需要进一步的操作以完成请求             |
| 4**  | 客户端错误，请求包含语法错误或无法完成请求     |
| 5**  | 服务器错误，服务器在处理请求的过程中发生了错误 |

HTTP状态码列表:

| 状态码 | 状态码英文名称                  | 中文描述                                                     |
| :----- | :------------------------------ | :----------------------------------------------------------- |
| 100    | Continue                        | 继续。[客户端](http://www.dreamdu.com/webbuild/client_vs_server/)应继续其请求 |
| 101    | Switching Protocols             | 切换协议。服务器根据客户端的请求切换协议。只能切换到更高级的协议，例如，切换到HTTP的新版本协议 |
|        |                                 |                                                              |
| 200    | OK                              | 请求成功。一般用于GET与POST请求                              |
| 201    | Created                         | 已创建。成功请求并创建了新的资源                             |
| 202    | Accepted                        | 已接受。已经接受请求，但未处理完成                           |
| 203    | Non-Authoritative Information   | 非授权信息。请求成功。但返回的meta信息不在原始的服务器，而是一个副本 |
| 204    | No Content                      | 无内容。服务器成功处理，但未返回内容。在未更新网页的情况下，可确保浏览器继续显示当前文档 |
| 205    | Reset Content                   | 重置内容。服务器处理成功，用户终端（例如：浏览器）应重置文档视图。可通过此返回码清除浏览器的表单域 |
| 206    | Partial Content                 | 部分内容。服务器成功处理了部分GET请求                        |
|        |                                 |                                                              |
| 300    | Multiple Choices                | 多种选择。请求的资源可包括多个位置，相应可返回一个资源特征与地址的列表用于用户终端（例如：浏览器）选择 |
| 301    | Moved Permanently               | 永久移动。请求的资源已被永久的移动到新URI，返回信息会包括新的URI，浏览器会自动定向到新URI。今后任何新的请求都应使用新的URI代替 |
| 302    | Found                           | 临时移动。与301类似。但资源只是临时被移动。客户端应继续使用原有URI |
| 303    | See Other                       | 查看其它地址。与301类似。使用GET和POST请求查看               |
| 304    | Not Modified                    | 未修改。所请求的资源未修改，服务器返回此状态码时，不会返回任何资源。客户端通常会缓存访问过的资源，通过提供一个头信息指出客户端希望只返回在指定日期之后修改的资源 |
| 305    | Use Proxy                       | 使用代理。所请求的资源必须通过代理访问                       |
| 306    | Unused                          | 已经被废弃的HTTP状态码                                       |
| 307    | Temporary Redirect              | 临时重定向。与302类似。使用GET请求重定向                     |
|        |                                 |                                                              |
| 400    | Bad Request                     | 客户端请求的语法错误，服务器无法理解                         |
| 401    | Unauthorized                    | 请求要求用户的身份认证                                       |
| 402    | Payment Required                | 保留，将来使用                                               |
| 403    | Forbidden                       | 服务器理解请求客户端的请求，但是拒绝执行此请求               |
| 404    | Not Found                       | 服务器无法根据客户端的请求找到资源（网页）。通过此代码，网站设计人员可设置"您所请求的资源无法找到"的个性页面 |
| 405    | Method Not Allowed              | 客户端请求中的方法被禁止                                     |
| 406    | Not Acceptable                  | 服务器无法根据客户端请求的内容特性完成请求                   |
| 407    | Proxy Authentication Required   | 请求要求代理的身份认证，与401类似，但请求者应当使用代理进行授权 |
| 408    | Request Time-out                | 服务器等待客户端发送的请求时间过长，超时                     |
| 409    | Conflict                        | 服务器完成客户端的 PUT 请求时可能返回此代码，服务器处理请求时发生了冲突 |
| 410    | Gone                            | 客户端请求的资源已经不存在。410不同于404，如果资源以前有现在被永久删除了可使用410代码，网站设计人员可通过301代码指定资源的新位置 |
| 411    | Length Required                 | 服务器无法处理客户端发送的不带Content-Length的请求信息       |
| 412    | Precondition Failed             | 客户端请求信息的先决条件错误                                 |
| 413    | Request Entity Too Large        | 由于请求的实体过大，服务器无法处理，因此拒绝请求。为防止客户端的连续请求，服务器可能会关闭连接。如果只是服务器暂时无法处理，则会包含一个Retry-After的响应信息 |
| 414    | Request-URI Too Large           | 请求的URI过长（URI通常为网址），服务器无法处理               |
| 415    | Unsupported Media Type          | 服务器无法处理请求附带的媒体格式                             |
| 416    | Requested range not satisfiable | 客户端请求的范围无效                                         |
| 417    | Expectation Failed              | 服务器无法满足Expect的请求头信息                             |
|        |                                 |                                                              |
| 500    | Internal Server Error           | 服务器内部错误，无法完成请求                                 |
| 501    | Not Implemented                 | 服务器不支持请求的功能，无法完成请求                         |
| 502    | Bad Gateway                     | 作为网关或者代理工作的服务器尝试执行请求时，从远程服务器接收到了一个无效的响应 |
| 503    | Service Unavailable             | 由于超载或系统维护，服务器暂时的无法处理客户端的请求。延时的长度可包含在服务器的Retry-After头信息中 |
| 504    | Gateway Time-out                | 充当网关或代理的服务器，未及时从远端服务器获取请求           |
| 505    | HTTP Version not supported      | 服务器不支持请求的HTTP协议的版本，无法完成处理               |

# HTTP各个版本特性及区别

作者：jad_design
链接：https://juejin.cn/post/6844903923136856078

## HTTP 0.9

HTTP 0.9 是一个最古老的版本

- 只支持`GET`请求方式：由于不支持其他请求方式，因此客户端是没办法向服务端传输太多的信息
- 没有请求头概念：所以不能在请求中指定版本号，服务端也只具有返回 HTML字符串的能力
- 服务端相响应之后，立即关闭TCP连接

## HTTP 1.0

随着 HTTP 1.0 的发布，这个版本:

- 请求方式新增了POST，DELETE，PUT，HEADER等方式
- 增添了请求头和响应头的概念，在通信中指定了 HTTP 协议版本号，以及其他的一些元信息 (比如: 状态码、权限、缓存、内容编码)
- 扩充了传输内容格式，图片、音视频资源、二进制等都可以进行传输

在这个版本主要的就是对请求和响应的元信息进行了扩展，客户端和服务端有更多的获取当前请求的所有信息，进而更好更快的处理请求相关内容。

#### 请求头

一个简单请求的头信息

```
GET / HTTP/1.0
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_5)
Accept: */*
复制代码
```

可以看到在请求方法之后有 请求资源的位置 + 请求协议版本，之后是一些客户端的信息配置

#### 响应头

一个简单响应的头信息(v1.0)

```
HTTP/1.0 200 OK
Content-Type: text/plain
Content-Length: 137582
Expires: Thu, 05 Dec 1997 16:00:00 GMT
Last-Modified: Wed, 5 August 1996 15:55:28 GMT
// 这是一个空行
...数据内容
复制代码
```

服务端的响应头第一个就是 请求协议版本，后面紧跟着是这次请求的状态码、以及状态码的描述，之后的内容是一些关于返回内容的描述。

**Content-Type**

在 HTTP 1.0 的时候，任何的资源都可以被传输，传输的格式呢也是多种多样的，客户端在收到响应体的内容的时候就是根据这个 `Content-Type` 去进行解析的。所以服务端返回时候必须带着这个字段。

一些常见的 `Content-Type` 可以参考 [对照表](http://tool.oschina.net/commons/)。 这些 `Content-Type` 有一个总称叫做`MIME type`。

关于`MIME type`，这里想播插一个小插曲:

> 在 chrome 浏览器中，当跨域请求回来的数据 MIME type 同跨域标签应有的 MIME type 不匹配时，浏览器会启动 CORB 保护数据不被泄漏。被保护的数据有: html、xml、json。(eg: script、img 标签所支持的 MIME type和他们都不一致)，所以服务端在返回资源的时候一定要对应返回正确的 `Content-Type`，以免浏览器屏蔽返回结果。
>
> 笔者遇到的问题是在 chrome v76 版本之后，跨域图片资源当请求回来的数据 `Content-Type` 不是 `image/*`，图片会被拦截，页面不展示图片。

#### 特性

- 无状态：服务器不跟踪不记录请求过的状态
- 无连接：浏览器每次请求都需要建立tcp连接

##### 无状态

对于无状态的特性可以借助cookie/session机制来做身份认证和状态记录

##### 无连接

无连接导致的性能缺陷有两种：

- 无法复用连接
   每次发送请求，都需要进行一次tcp连接（即3次握手4次挥手），使得网络的利用率非常低
- 队头阻塞
   HTTP 1.0 规定在前一个请求响应到达之后下一个请求才能发送，如果前一个阻塞，后面的请求也给阻塞的

## HTTP 1.1

HTTP 1.1 是在 1.0 发布之后的半年就推出了，完善了 1.0 版本。目前也还有很多的互联网项目基于 HTTP 1.1 在向外提供服务。

#### 特性

- 长连接：新增Connection字段，可以设置keep-alive值保持连接不断开
- 管道化：基于上面长连接的基础，管道化可以不等第一个请求响应继续发送后面的请求，但响应的顺序还是按照请求的顺序返回
- 缓存处理：新增字段cache-control
- 断点传输

##### 长连接

HTTP 1.1默认保持长连接，数据传输完成保持tcp连接不断开,继续用这个通道传输数据

##### 管道化

基于长连接的基础，我们先看没有管道化请求响应：

tcp没有断开，用的同一个通道

```
请求1 > 响应1 --> 请求2 > 响应2 --> 请求3 > 响应3
复制代码
```

管道化的请求响应：

```
请求1 --> 请求2 --> 请求3 > 响应1 --> 响应2 --> 响应3
复制代码
```

即使服务器先准备好响应2,也是按照请求顺序先返回响应1

虽然管道化，可以一次发送多个请求，但是响应仍是顺序返回，仍然无法解决队头阻塞的问题

#### 缓存处理

当浏览器请求资源时，先看是否有缓存的资源，如果有缓存，直接取，不会再发请求，如果没有缓存，则发送请求。 通过设置字段cache-control来控制缓存。

#### 断点传输

在上传/下载资源时，如果资源过大，将其分割为多个部分，分别上传/下载，如果遇到网络故障，可以从已经上传/下载好的地方继续请求，不用从头开始，提高效率

## HTTP 2

特性:

- 二进制分帧
- 多路复用： 在共享TCP链接的基础上同时发送请求和响应
- 头部压缩
- 服务器推送：服务器可以额外的向客户端推送资源，而无需客户端明确的请求

#### 二进制分帧

HTTP 1.x 的解析是基于文本，HTTP 2之后将所有传输的信息分割为更小的消息和帧，并对它们采用二进制格式的编码，提高传输效率

#### 多路复用

在共享TCP链接的基础上同时发送请求和响应，基于二进制分帧，在同一域名下所有访问都是从同一个tcp连接中走，http消息被分解为独立的帧，乱序发送，服务端根据标识符和首部将消息重新组装起来。

#### 头部压缩

由于 HTTP 是无状态的，每一个请求都需要头部信息标识这次请求相关信息，所以会造成传输很多重复的信息，当请求数量增大的时候，消耗的资源就会慢慢积累上去。所以 HTTP 2 可以维护一个头部信息字典，差量进行更新头信息，减少头部信息传输占用的资源，详见 [HTTP/2 头部压缩技术介绍](https://imququ.com/post/header-compression-in-http2.html)。

## HTTPS 和 HTTP

- HTTPS 协议需要申请证书
- HTTP 和 HTTPS 使用端口不一样，前者是80，后者是443
- HTTP 协议运行在 TCP 之上，所有传输的内容都是明文，HTTPS 运行在 SSL/TLS 之上，SSL/TLS运行在TCP之上，所有传输的内容都经过加密的
- HTTPS 可以有效的防止运营商劫持


