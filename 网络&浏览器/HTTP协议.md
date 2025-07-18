##  GET和POST的区别

- 从**缓存**的角度，GET 请求会被浏览器主动缓存下来，留下历史记录，而 POST 默认不会。
- GET 回退无害，POST 会再次提交。
- 从**编码**的角度，GET 只能进行 URL 编码，只能接收 ASCII 字符，而 POST 没有限制。
- 从**参数**的角度，GET 一般放在 URL 中，因此不安全，POST 放在请求体中，更适合传输敏感信息。
- 从**幂等性**的角度，`GET`是**幂等**的，而`POST`不是。(`幂等`表示执行相同的操作，结果也是相同的)
- 从**TCP**的角度，GET 请求会把请求报文一次性发出去，而 POST 会分为两个 TCP 数据包，首先发 header 部分，如果服务器响应 100(continue)， 然后发 body 部分。(**火狐**浏览器除外，它的 POST 请求只发一个 TCP 包)

* 参数长度的限制：

  HTTP 协议 未规定 GET 和 POST 的长度限制

  GET 的最大长度显示是因为浏览器和服务器限制了 URL 的长度

  不同的浏览器和WEB服务器，限制的最大长度不一样

  要支持IE，则最大长度为2083byte，若只支持Chrome，则最大长度 8182byte

* 使用xhr时

![](https://camo.githubusercontent.com/ef36bff2a3d3d0b12e96d481c20894fbaafb52f6/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f31322f31382f313637626366383365613762336662623f773d32393826683d31363926663d706e6726733d3335383634)

![](https://camo.githubusercontent.com/09f65a118cd0b462bbbc84928b8038be87f5bcaa/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f31322f31382f313637626366376264363564623737303f773d33343626683d31343926663d706e6726733d3431343234)

## HTTP状态码

### 五大类

- 1XX: 通知
  - **100 Continue** 客户端应重新发请求
  - **101 Switching Protocols** 客户端请求服务器更换协议，如果服务器同意变更，就会发送状态码 101
- 2XX：成功
  - **200 OK** 操作成功
  - **201 Created**按照客户端请求创建了一个新资源
  - **202 Accepted** 请求无法或不被实时处理
  - **204 No Content** 请求成功，但是报文不含实体的主体部分
  - **205 Reset Content** 请求成功，但是报文不含实体的主体部分，要求客户端重置内容
  - **206 Partial Content** 进行范围请求，表示部分内容，它的使用场景为 HTTP 分块下载和断电续传
- 3XX：重定向
  - **301 Moved Permanently**永久性重定向，资源已经被分配到了新的URL
  - **302 Found** 临时重定向，资源临时分配了URL 实际上发部分客户端把它当成303处理
  - **303 See Other** 表示资源存在另一个URL。应用Get获取资源
  - **304 Not Modified** 当协商缓存命中时会返回这个状态码
  - **307 Temporary Redirect** 临时重定向，资源临时分配了URL，但是希望客户端能够保持方法不变请求新地址（解决302被当成303处理的问题）
- 4XX：客户端错误
  - **400 Bad Request** 开发者经常看到一头雾水，只是笼统地提示了一下错误，并不知道哪里出错了
  - **401 Unauthorized** 发送的请求需要通过验证，客户端试图对一个受保护的资源操作但没有认证证书
  - **403 Forbidden** 请求资源存在但被拒绝，常用于一个资源只允许在特定时间段内访问（如果不想透露可以谎报404）
  - **404 Not Found** 找不到请求的资源
  - **405 Method Not Allowed** 资源不支持的请求方法，比如只支持Get，但是收到了Post请求
- 5XX：服务端错误
  - **500 Internal Server Error** 仅仅告诉你服务器出错了，出了啥错咱也不知道
  - **501 Not Implemented** 服务器不支持此请求方法（和405区别在于，405是访问的资源不支持，而501表示服务器不能操作此方法）
  - **502 Bad Gateway** 代理与上行服务器之间出现问题，服务器自身是正常的
  - **503 Service Unavailable** 服务器暂时很忙或者维护中
  - **504 Gateway Timeout** 请求超时

### 几个区别

#### 状态码返回204可能的原因

- 服务器拒绝请求返回
- Get资源存在但表示是空的
  服务器通过这个响应代码告诉客户端：客户端的输入已被接受，但客户端不应该改变任何UI元素

#### 状态码204和205的区别

204和205的区别在于**205要求了重置**！

用一个表单为例，如果提交后返回204，那么表单里的各个字段值不变，可以继续修改它们；但假如得到的响应代码205，那么表单里的各个字段将被重置为它们的初始值。

## HTTP特点

### 优点

1. 灵活可扩展。主要体现在两个方面。一个是语义上的自由，只规定了基本格式，比如空格分隔单词，换行分隔字段，其他的各个部分都没有严格的语法限制。另一个是传输形式的多样性，不仅仅可以传输文本，还能传输图片、视频等任意数据，非常方便。
2. 可靠传输。HTTP 基于 TCP/IP，因此把这一特性继承了下来。这属于 TCP 的特性，不具体介绍了。
3. 请求-应答。也就是`一发一收`、`有来有回`， 当然这个请求方和应答方不单单指客户端和服务器之间，如果某台服务器作为代理来连接后端的服务端，那么这台服务器也会扮演**请求方**的角色。
4. 无状态。每个请求都是互相独立、毫无关联的，协议不要求客户端或服务器记录请求相关的信息。

### 缺点

#### 无状态

所谓的优点和缺点还是要分场景来看的，对于 HTTP 而言，最具争议的地方在于它的**无状态**。

在需要长连接的场景中，需要保存大量的上下文信息，以免传输大量重复的信息，那么这时候无状态就是 http 的缺点了。

但与此同时，另外一些应用仅仅只是为了获取一些数据，不需要保存连接上下文信息，无状态反而减少了网络开销，成为了 http 的优点。

#### 明文传输，不安全

即协议里的报文(主要指的是头部)不使用二进制数据，而是文本形式。**此文本非彼“文本”，超文本是啥，就是二进制文件在http协议中的存在形式，或者叫编码更合适**https://segmentfault.com/q/1010000006670932

这当然对于调试提供了便利，但同时也让 HTTP 的报文信息暴露给了外界，给攻击者也提供了便利。`WIFI陷阱`就是利用 HTTP 明文传输的缺点，诱导你连上热点，然后疯狂抓你所有的流量，从而拿到你的敏感信息。

#### 队头阻塞

当 http 开启长连接时，共用一个 TCP 连接，同一时刻只能处理一个请求，那么当前请求耗时过长的情况下，其它的请求只能处于阻塞状态，也就是著名的**队头阻塞**问题。

## HTTP1.1 与 1.0 的区别

http 1.0 :

1. 默认不支持长连接，需要设置keep-alive参数指定
2. 强缓存expires、协商缓存last-modified\if-modified-since 有一定的缺陷

http 1.1 :

1. 默认长连接(keep-alive)，http请求可以复用Tcp连接，但是同一时间只能对应一个http请求(http请求在一个Tcp中是串行的)
2. 增加了强缓存cache-control、协商缓存Etag\if-none-match 是对http/1 缓存的优化

## HTTP常用首部

|              |                请求                |                           响应                            |
| :----------: | :--------------------------------: | :-------------------------------------------------------: |
|    起始行    |         GET /home HTTP/1.1         |                 HTTP/1.1 200 OK（状态行）                 |
|   数据格式   |         Accept: text/html          |                  Content-Type: text/html                  |
|   压缩方式   |       Accept-Encoding: gizp        |                  Content-Encoding: gzip                   |
|   支持语言   |   Accept-Language: zh-CN, zh, en   |              Content-Language: zh-CN, zh, en              |
|    字符集    |   Accept-Charset: charset=utf-8    |          Content-Type: text/html; charset=utf-8           |
| 资源修改时间 |     If-Modified-Since: 时间戳      |                   Last-Modified: 时间戳                   |
|  资源标识符  |     If-None-Match: 哈希字符串      |                     Etag: 哈希字符串                      |
|   缓存控制   | cache-control: max-age=0(刷新数据) | cache-control: public/private/no-cache/no-store/max-age=X |
|    长连接    |    connection: keep-alive/close    |               connection: keep-alive/close                |

### content-type常用类型

常见的媒体格式类型如下：

- text/html ： HTML格式
- text/plain ：纯文本格式
- text/xml ： XML格式
- image/gif ：gif图片格式
- image/jpeg ：jpg图片格式
- image/png：png图片格式

以application开头的媒体格式类型：

- application/xhtml+xml ：XHTML格式
- application/xml： XML数据格式
- application/atom+xml ：Atom XML聚合格式
- application/json： JSON数据格式
- application/pdf：pdf格式
- application/msword ： Word文档格式
- application/octet-stream ： 二进制流数据（如常见的文件下载）
- application/x-www-form-urlencoded ： <form encType="">中默认的encType，form表单数据被编码为key/value格式发送到服务器（表单默认的提交数据的格式）

另外一种常见的媒体格式是上传文件之时使用的：

- multipart/form-data ： 需要在表单中进行文件上传时，就需要使用该格式

## HTTP传输定长包体与不定长包体

http://47.98.159.95/my_blog/http/007.html#%E5%AE%9A%E9%95%BF%E5%8C%85%E4%BD%93

### 定长包体

```http
content-length: 20(包体长度)
```

`Content-Length`对于 HTTP 传输过程起到了十分关键的作用，如果设置不当可以直接导致传输失败：设置小了，数据只能传输部分，设置大了，直接导致传输失败。

### 不定长包体

```http
Transfer-Encoding: chunked
```

表示分块传输数据，设置这个字段后会自动产生两个效果:

- Content-Length 字段会被忽略
- 基于长连接持续推送动态内容

## HTPP处理大文件请求

http://47.98.159.95/my_blog/http/008.html#%E5%A6%82%E4%BD%95%E6%94%AF%E6%8C%81

HTTP 针对这一场景，采取了`范围请求`的解决方案，允许客户端仅仅请求一个资源的一部分。Ranges字段。

## HTTP处理表单数据提交

由于表单提交一般是`POST`请求，很少考虑`GET`，因此这里我们将默认提交的数据放在请求体中。

### application/x-www-form-urlencoded

对于`application/x-www-form-urlencoded`格式的表单内容，有以下特点:

- 其中的数据会被编码成以`&`分隔的键值对
- 字符以**URL编码方式**编码

如：

```text
// 转换过程: {a: 1, b: 2} -> a=1&b=2 -> 如下(最终形式)
"a%3D1%26b%3D2"
```

### multipart/form-data

对于`multipart/form-data`而言:

- 请求头中的`Content-Type`字段会包含`boundary`，且`boundary`的值有浏览器默认指定。例: `Content-Type: multipart/form-data;boundary=----WebkitFormBoundaryRRJKeWfHPGrS4LKe`。
- 数据会分为多个部分，每两个部分之间通过分隔符来分隔，每部分表述均有 HTTP 头部描述子包体，如`Content-Type`，在最后的分隔符会加上`--`表示结束。

相应的`请求体`是下面这样:

```text
Content-Disposition: form-data;name="data1";
Content-Type: text/plain
data1
----WebkitFormBoundaryRRJKeWfHPGrS4LKe
Content-Disposition: form-data;name="data2";
Content-Type: text/plain
data2
----WebkitFormBoundaryRRJKeWfHPGrS4LKe--
```

### 对比

值得一提的是，`multipart/form-data` 格式最大的特点在于：**每一个表单元素都是独立的资源表述**。不做编码，发送二进制数据。

另外，我们在写业务的过程中，并没有注意到其中还有`boundary`的存在，如果打开抓包工具，确实可以看到不同的表单元素被拆分开了，之所以在平时感觉不到，是以为浏览器和 HTTP 给你封装了这一系列操作。

而且，**在实际的场景中，对于图片等文件的上传，基本采用`multipart/form-data`而不用`application/x-www-form-urlencoded`，因为没有必要做 URL 编码，带来巨大耗时的同时也占用了更多的空间。**

## HTTP长连接

长连接的定义简单来说就是建立一次 TCP 连接后，会进行多次 HTTP 的请求和响应。

由于长连接对性能的改善效果非常显著，所以**在 HTTP/1.1 中的连接都会默认启用长连接。不需要用什么特殊的头字段指定**，只要向服务器发送了第一次请求，后续的请求都会重复利用第一次打开的 TCP 连接，也就是长连接，在这个连接上收发数据。

当然，我们也可以在请求头里明确地要求使用长连接机制，使用的字段是 Connection，值是“keep-alive”。

不过不管客户端是否显式要求长连接，如果服务器支持长连接，它总会在响应报文里放一个“Connection: keep-alive”字段，告诉客户端：“我是支持长连接的，接下来就用这个 TCP 一直收发数据吧”。

### 长连接的问题

因为 TCP 连接长时间不关闭，服务器必须在内存里保存它的状态，这就占用了服务器的资源。如果有大量的空闲长连接只连不发，就会很快耗尽服务器的资源，导致服务器无法为真正有需要的用户提供服务。

所以，长连接也需要在恰当的时间关闭，不能永远保持与服务器的连接，这在客户端或者服务器都可以做到。

在客户端，可以在请求头里加上“Connection: close”字段，告诉服务器：“这次通信后就关闭连接”。服务器看到这个字段，就知道客户端要主动关闭连接，于是在响应报文里也加上这个字段，发送之后就调用 Socket API 关闭 TCP 连接。

> 服务器端通常不会主动关闭连接，但也可以使用一些策略。拿 Nginx 来举例，它有两种方式：

使用“keepalive_timeout”指令，设置长连接的超时时间，如果在一段时间内连接上没有任何数据收发就主动断开连接，避免空闲连接占用系统资源。

使用“keepalive_requests”指令，设置长连接上可发送的最大请求次数。比如设置成 1000，那么当 Nginx 在这个连接上处理了 1000 个请求后，也会主动断开连接。 另外，客户端和服务器都可以在报文里附加通用头字段“Keep-Alive: timeout=value”，限定长连接的超时时间。但这个字段的约束力并不强，通信的双方可能并不会遵守，所以不太常见。

### 管线化

管线化是在持久连接的基础上再次提升通信效率的方法。将一个TCP上的多个HTTP请求，不再以请求-响应顺次进行，上次请求得到响应后，才能发送下次请求。管线化技术就是**客户端在收到响应之前就顺次发送其他请求**。可以在较高时延的情况下，提升通信效率。

因为HTTP报文没有序列号，响应必须按照请求发过来的顺序，进行发送，如果顺序错乱，客户端就没办法匹配。

这与 HTTP2.0 中的多路复用不一样，多路复用中是有 id 的，会保证不会顺序错乱。

## 队头阻塞

“队头阻塞”与短连接和长连接无关，而是由 HTTP 基本的“请求 - 应答”模型所导致的。

因为 HTTP 规定报文必须是“一发一收”，这就形成了一个先进先出的“串行”队列。队列里的请求没有轻重缓急的优先级，只有入队的先后顺序，排在最前面的请求被最优先处理。

如果队首的请求因为处理的太慢耽误了时间，那么队列里后面的所有请求也不得不跟着一起等待，结果就是其他的请求承担了不应有的时间成本。

### 并发连接

对于一个域名允许分配多个长连接，那么相当于增加了任务队列，不至于一个队伍的任务阻塞其它所有任务。在RFC2616规定过客户端最多并发 2 个连接，不过事实上在现在的浏览器标准中，这个上限要多很多，Chrome 中是 6 个。

但其实，即使是提高了并发连接，还是不能满足人们对性能的需求。

### 域名分片

一个域名不是可以并发 6 个长连接吗？那我就多分几个域名。

比如 content1.sanyuan.com 、content2.sanyuan.com。

这样一个`sanyuan.com`域名下可以分出非常多的二级域名，而它们都指向同样的一台服务器，能够并发的长连接数更多了，事实上也更好地解决了队头阻塞的问题。

## Cookie

前面说到了 HTTP 是一个无状态的协议，每次 http 请求都是独立、无关的，默认不需要保留状态信息。但有时候需要保存一些状态，怎么办呢？

HTTP 为此引入了 Cookie。Cookie 本质上就是浏览器里面存储的一个很小的文本文件，内部以键值对的方式来存储(在chrome开发者面板的Application这一栏可以看到)。向同一个域名下发送请求，都会携带相同的 Cookie，服务器拿到 Cookie 进行解析，便能拿到客户端的状态。而服务端可以通过响应头中的`Set-Cookie`字段来对客户端写入`Cookie`。举例如下:

```http
// 请求头
Cookie: a=xxx;b=xxx
// 响应头
Set-Cookie: a=xxx
set-Cookie: b=xx
```

### Cookie 属性

#### 生命周期

Cookie 的有效期可以通过**Expires**和**Max-Age**两个属性来设置。

- **Expires**即`过期时间`
- **Max-Age**用的是一段时间间隔，单位是秒，从浏览器收到报文开始计算。

若 Cookie 过期，则这个 Cookie 会被删除，并不会发送给服务端。

#### 作用域

关于作用域也有两个属性: **Domain**和**path**, 给 **Cookie** 绑定了域名和路径，在发送请求之前，发现域名或者路径和这两个属性不匹配，那么就不会带上 Cookie。值得注意的是，对于路径来说，`/`表示域名下的任意路径都允许使用 Cookie。

#### 安全相关

如果 cookie 字段带上`HttpOnly`，那么说明只能通过 HTTP 协议传输，不能通过 JS 访问，这也是预防 XSS 攻击的重要手段。

相应的，对于 CSRF 攻击的预防，也有`SameSite`属性。

`SameSite`可以设置为三个值，`Strict`、`Lax`和`None`。

**a.** 在`Strict`模式下，浏览器完全禁止第三方请求携带Cookie。比如请求`a.com`网站只能在`a.com`域名当中请求才能携带 Cookie，在其他网站请求都不能。

**b.** 在`Lax`模式，就宽松一点了，但是只能在 `get 方法提交表单`或者`a 标签发送 get 请求`的情况下可以携带 Cookie，其他情况均不能。

**c.** 在`None`模式下，也就是默认模式，请求会自动携带上 Cookie。

### Cookie 的缺点

1. 容量缺陷。Cookie 的体积上限只有`4KB`，只能用来存储少量的信息。
2. 性能缺陷。Cookie 紧跟域名，不管域名下面的某一个地址需不需要这个 Cookie ，请求都会携带上完整的 Cookie，这样随着请求数的增多，其实会造成巨大的性能浪费的，因为请求携带了很多不必要的内容。但可以通过`Domain`和`Path`指定**作用域**来解决。
3. 安全缺陷。由于 Cookie 以纯文本的形式在浏览器和服务器中传递，很容易被非法用户截获，然后进行一系列的篡改，在 Cookie 的有效期内重新发送给服务器，这是相当危险的。另外，在`HttpOnly`为 false 的情况下，Cookie 信息能直接通过 JS 脚本来读取。

## HTTP缓存

### 强缓存

发起网络请求的时候，网络进程首先检查强缓存，这个阶段`不需要`发送HTTP请求。

在早期`HTTP/1.0`时期，使用的是**Expires**，而`HTTP/1.1`使用的是**Cache-Control**。当**Expires**和**Cache-Control**同时存在的时候，**Cache-Control**会优先考虑。

#### Expires

`Expires`即过期时间，存在于服务端返回的响应头中，告诉浏览器在这个过期时间之前可以直接从缓存里面获取数据，无需再次请求。

```http
Expires: Wed, 22 Nov 2019 08:41:00 GMT
```

这个方式看上去没什么问题，合情合理，但其实潜藏了一个坑，那就是**服务器的时间和浏览器的时间可能并不一致**，那服务器返回的这个过期时间可能就是不准确的。

#### Cache-Control

它和`Expires`本质的不同在于它并没有采用`具体的过期时间点`这个方式，而是采用过期时长来控制缓存，对应的字段是**max-age**。比如这个例子:

```http
Cache-Control:max-age=3600
// 代表这个响应返回后在 3600 秒，也就是一个小时之内可以直接使用缓存
```

可以组合很多指令：public、private、no-store、no-cashe、s-maxage、must-revalidate（如果超过了 `max-age` 的时间，浏览器必须向服务器发送请求，验证资源是否还有效）

#### Pragma

`Pragma` 是HTTP/1.1 之前版本保留的历史遗留字段，仅作为与HTTP/1.0 的向后兼容而定义。规范定义的**形式唯一**，如下所示：

```http
Pragma: no-cache
```

### 协商缓存

强缓存失效之后，浏览器在请求头中携带相应的`缓存tag`来向服务器发请求，由服务器根据这个tag，来决定是否使用缓存，这就是**协商缓存**。

具体来说，这样的缓存tag分为两种: **Last-Modified** 和 **ETag**。这两者各有优劣，并不存在谁对谁有`绝对的优势`，跟上面强缓存的两个 tag 不一样。

#### Last-Modified

在浏览器第一次给服务器发送请求后，服务器会在响应头中加上这个 Last-Modified 字段。

浏览器接收到后，如果再次请求，会在请求头中携带`If-Modified-Since`字段，这个字段的值也就是服务器传来的最后修改时间。

服务器拿到请求头中的`If-Modified-Since`的字段后，其实会和这个服务器中`该资源的最后修改时间`对比:

- 如果请求头中的这个值小于最后修改时间，说明是时候更新了。返回新的资源，跟常规的HTTP请求响应的流程一样。
- 否则返回304，告诉浏览器直接用缓存。

#### ETag

ETag 是“实体标签”（Entity Tag）的缩写，是资源的一个唯一标识，主要是用来解决修改时间无法准确区分文件变化的问题。

比如，一个文件在一秒内修改了多次，但因为修改时间是秒级，所以这一秒内的新版本无法区分。

再比如，一个文件定期更新，但有时会是同样的内容，实际上没有变化，用修改时间就会误以为发生了变化，传送给浏览器就会浪费带宽。

使用 ETag 就可以精确地识别资源的变动情况，让浏览器能够更有效地利用缓存。

服务器通过`响应头`把这个值给浏览器。

浏览器接收到`ETag`的值，会在下次请求时，将这个值作为`If-None-Match`这个字段的内容，并放到请求头中，然后发给服务器。

#### 两者对比

1. 在`精准度`上，`ETag`优于`Last-Modified`。 ETag 是按照内容给资源上标识，因此能准确感知资源的变化。而 Last-Modified 就不一样了，它在一些特殊的情况并不能准确感知资源变化，主要有两种情况:
   * 编辑了资源文件，但是文件内容并没有更改，这样也会造成缓存失效；
   * Last-Modified 能够感知的单位时间是秒，如果文件在 1 秒内改变了多次，那么这时候的 Last-Modified 并没有体现出修改了。

2. 在性能上，`Last-Modified`优于`ETag`，也很简单理解，`Last-Modified`仅仅只是记录一个时间点，而 `Etag`需要根据文件的具体内容生成哈希值。

另外，如果两种方式都支持的话，服务器会优先考虑`ETag`。

### 代理缓存

对于 HTTP 缓存来说，如果每次客户端缓存失效都要到源服务器获取，那给源服务器的压力是很大的。

由此引入了**缓存代理**的机制。让`代理服务器`接管一部分的服务端HTTP缓存，客户端缓存过期后**就近**到代理缓存中获取，代理缓存过期了才请求源服务器，这样流量巨大的时候能明显降低源服务器的压力。

那缓存代理究竟是如何做到的呢？

总的来说，缓存代理的控制分为两部分，一部分是**源服务器**端的控制，一部分是**客户端**的控制。

#### 源服务器端的控制

1. private 和 public

   在源服务器的响应头中，会加上`Cache-Control`这个字段进行缓存控制字段，那么它的值当中可以加入`private`或者`public`表示是否允许代理服务器缓存，前者禁止，后者为允许。

2. s-maxage

   `s`是`share`的意思，限定了缓存在代理服务器中可以存放多久，和限制客户端缓存时间的`max-age`并不冲突。

   举个小例子，源服务器在响应头中加入这样一个字段:

   ```text
   Cache-Control: public, max-age=1000, s-maxage=2000
   ```

   相当于源服务器说: 我这个响应是允许代理服务器缓存的，客户端缓存过期了到代理中拿，并且在客户端的缓存时间为 1000 秒，在代理服务器中的缓存时间为 2000 s。

3. proxy-revalidate

   `must-revalidate`的意思是**客户端**缓存过期就去源服务器获取，而`proxy-revalidate`则表示**代理服务器**的缓存过期后到源服务器获取。

#### 客户端的控制

1. max-stale 和 min-fresh

   ``` text
   max-stale: 5
   // 表示客户端到代理服务器上拿缓存的时候，即使代理缓存过期了也不要紧，只要过期时间在5秒之内，还是可以从代理中获取的。
   min-fresh: 5
   // 表示代理缓存需要一定的新鲜度，不要等到缓存刚好到期再拿，一定要在到期前 5 秒之前的时间拿，否则拿不到。
   ```

2.  only-if-cached

   这个字段加上后表示客户端只会接受代理缓存，而不会接受源服务器的响应。如果代理缓存无效，则直接返回`504（Gateway Timeout）`。

### 缓存位置

从缓存位置上来说分为四种，并且各自有**优先级**，当**依次查找缓存且都没有命中的时候**，才会去请求网络。顺序是：

1. Service Worker
2. Memory Cache
3. Disk Cache
4. Push Cache
5. 网络请求

#### Service Worker

- `Service Worker` 是运行在浏览器背后的独立线程，一般可以用来实现缓存功能。使用 `Service Worker`的话，传输协议必须为 `HTTPS`。因为 `Service Worker` 中涉及到请求拦截，所以必须使用 `HTTPS` 协议来保障安全
- `Service Worker` 实现缓存功能一般分为三个步骤：首先需要先注册 `Service Worker`，然后监听到 `install` 事件以后就可以缓存需要的文件，那么在下次用户访问的时候就可以通过拦截请求的方式查询是否存在缓存，存在缓存的话就可以直接读取缓存文件，否则就去请求数据。

- `service Worker` 的缓存与浏览器其他内建的缓存机制不同，它可以让我们自由控制缓存哪些文件、如何匹配缓存、如何读取缓存，并且缓存是持续性的。
- 当 `Service Worker` 没有命中缓存的时候，我们需要去调用 `fetch` 函数获取数据。也就是说，如果我们没有在 `Service Worker` 命中缓存的话，会根据缓存查找优先级去查找数据。但是不管我们是从 `Memory Cache` 中还是从网络请求中获取的数据，浏览器都会显示我们是从 `Service Worker` 中获取的内容。

#### Memory Cache

`Memory Cache`指的是内存缓存，从效率上讲它是最快的。但是从存活时间来讲又是最短的，当渲染进程结束后，内存缓存也就不存在了。 也就是说，一旦我们关闭 Tab 页面，内存中的缓存也就被释放了。

#### Disk Cache

`Disk Cache`是存储在硬盘中的缓存，读取速度慢点，但是什么都能存储到磁盘中，比之 `Memory Cache` **胜在容量和存储时效性上。**

在所有浏览器缓存中，`Disk Cache` 覆盖面基本是最大的。它会根据 HTTP Herder 中的字段判断哪些资源需要缓存，哪些资源可以不请求直接使用，哪些资源已经过期需要重新请求。**并且即使在跨站点的情况下，相同地址的资源一旦被硬盘缓存下来，就不会再次去请求数据。**

浏览器如何决定将资源放进内存还是硬盘呢？主要策略如下：

- 比较大的JS、CSS文件会直接被丢进磁盘，反之丢进内存
- 内存使用率比较高的时候，文件优先进入磁盘

#### Push Cache

`Push Cache` 是 HTTP/2 中的内容，即推送缓存。当以上三种缓存都没有命中时，它才会被使用，**并且缓存时间也很短暂，只在会话（Session）中存在，一旦会话结束就被释放。**

### 浏览器的行为

用户在浏览器如何操作时，会触发怎样的缓存策略。主要有 3 种：

- 打开网页，地址栏输入地址： 查找 disk cache 中是否有匹配。如有则使用；如没有则发送网络请求。
- 普通刷新 (F5)：因为 TAB 并没有关闭，因此 memory cache 是可用的，会被优先使用(如果匹配的话)。其次才是 disk cache。
- 强制刷新 (Ctrl + F5)：浏览器不使用缓存，因此发送的请求头部均带有 `Cache-control: no-cache`(为了兼容，还带了 `Pragma: no-cache`)。服务器直接返回 200 和最新内容。

参考文章：

[一文读懂前端缓存](https://juejin.cn/post/6844903747357769742?utm_source=gold_browser_extension%3Futm_source%3Dgold_browser_extension#heading-0)

## HTTPS

`HTTPS`并不是一个新的协议, 而是一个加强版的`HTTP`。其原理是在`HTTP`和`TCP`之间建立了一个中间层，当`HTTP`和`TCP`通信时并不是像以前那样直接通信，直接经过了一个中间层进行加密，将加密后的数据包传给`TCP`, 响应的，`TCP`必须将数据包解密，才能传给上面的`HTTP`。这个中间层也叫`安全层`。`安全层`的核心就是对数据`加解密`。

简单的讲，**HTTPS = HTTP + SSL/TLS**。**所谓HTTPS，其实就是身披SSL协议这层外壳的HTTP**。

那什么是 SSL/TLS 呢？

SSL 即安全套接层（Secure Sockets Layer），在 OSI 七层模型中处于会话层(第 5 层)。之前 SSL 出过三个大版本，当它发展到第三个大版本的时候才被标准化，成为 TLS（传输层安全，Transport Layer Security），并被当做 TLS1.0 的版本，准确地说，**TLS1.0 = SSL3.1**。现在主流的版本是 TLS/1.2。

### Http为什么不安全？

1. 数据以明文传递，有被窃听的风险
2. 接收到的报文无法证明是发送时的报文，不能保证完整性，因此报文有被篡改的风险
3. 不验证通信两端的身份，请求或响应有被伪造的风险

反观 Https：

- 数据隐私性：内容经过对称加密，每个连接生成一个唯一的加密密钥
- 数据完整性：内容传输经过完整性校验——散列函数，数字签名
- 身份认证：第三方无法伪造服务端（客户端）身份——数字证书

### Http和Https有什么区别？

1. HTTP是超文本传输协议，信息是**明文传输** ，HTTPS则是具有安全性的SSL加密传输协议
2. HTTP和HTTPS使用的是完全不同的连接方式，用的**端口也不一样，前者是80，后者是443**
3. HTTPS协议需要CA申请证书，一般免费证书比较少，因而需要一定费用
4. HTTP的连接很简单，是无状态的；HTTPS协议是由**TSL**协议进行了加密，比HTTP协议安全，但也是无状态的

### Https的缺点？

1. 通信两端都需要进行加密和解密，会消耗大量的CPU、内存等资源 ，**增加服务器的压力**
2. 加密运算和多次握手**降低了访问速度**
3. 在开发阶段，**加大了页面调试难度** 。由于信息都被加密了，所以用代理工具的话，需要先解密然后才能看到真实信息
4. 用HTTPS访问的页面，页面内的外部资源都得用HTTPS请求，包括脚本中的AJAX请求

### 对称加密和非对称加密

`对称加密`是最简单的方式，指的是`加密`和`解密`用的是**同样的密钥**。

而对于`非对称加密`，如果有 A、 B 两把密钥，如果用 A 加密过的数据包只能用 B 解密，反之，如果用 B 加密过的数据包只能用 A 解密。

### 传统RSA握手

http://47.98.159.95/my_blog/browser-security/003.html#%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86%E5%92%8C%E9%9D%9E%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86

之所以称它为 RSA 版本，是因为它在加解密`pre_random`的时候采用的是 RSA 算法。

这是对称加密和非对称加密的结合，

1. 浏览器向服务器发送`client_random`和加密方法列表。
2. 服务器接收到，返回`server_random`、加密方法以及公钥。
3. 浏览器接收，接着通过 RSA 算法 生成另一个随机数`pre_random`, 并且用公钥加密，传给服务器。(重点操作！)
4. 服务器用私钥解密这个被加密后的`pre_random`。

现在浏览器和服务器有三样相同的凭证:`client_random`、`server_random`和`pre_random`。然后两者用同样的伪随机数函数去混合这三个随机数，生成最终的`密钥`。

然后浏览器和服务器尽管用一样的密钥进行通信，即使用`对称加密`。

![](http://47.98.159.95/my_blog/week12/1.jpg)

你会发现图中没有公钥，而是数字证书，事实上`HTTPS`在上述`结合对称和非对称加密`的基础上，又添加了`数字证书认证`的步骤。其目的就是让服务器证明自己的身份。

为了获取这个证书，服务器运营者需要向第三方认证机构获取授权，这个第三方机构也叫`CA`(`Certificate Authority`), 认证通过后 CA 会给服务器颁发**数字证书**。

这个数字证书有两个作用:

1. **服务器向浏览器证明自己的身份。**
2. **把公钥传给浏览器。**

这个验证的过程发生在什么时候呢？

当服务器传送`server_random`、加密方法的时候，顺便会带上`数字证书`(包含了`公钥`), 接着浏览器接收之后就会开始验证数字证书。如果验证通过，那么后面的过程照常进行，否则拒绝执行。

### TLS1.2握手

http://47.98.159.95/my_blog/http/015.html

在**TLS 握手阶段，两端使用非对称加密的方式来握手** ，但是因为非对称加密损耗的性能比对称加密大，所以**在正式传输数据时，两端传输其实是使用对称加密的方式通信** 。

![](http://47.98.159.95/my_blog/http/010.jpg)

#### step 1: Client Hello

首先，浏览器发送 client_random、TLS版本、加密套件列表。

client_random 是什么？用来最终 secret 的一个参数。

加密套件列表是什么？我举个例子，加密套件列表一般张这样:

```text
TLS_ECDHE_WITH_AES_128_GCM_SHA256
```

意思是`TLS`握手过程中，使用`ECDHE`算法生成`pre_random`(这个数后面会介绍)，128位的`AES`算法进行对称加密，在对称加密的过程中使用主流的`GCM`分组模式，因为对称加密中很重要的一个问题就是如何分组。最后一个是哈希摘要算法，采用`SHA256`算法。

其中值得解释一下的是这个哈希摘要算法，试想一个这样的场景，服务端现在给客户端发消息来了，客户端并不知道此时的消息到底是服务端发的，还是中间人伪造的消息呢？现在引入这个哈希摘要算法，将服务端的证书信息通过**这个算法**生成一个摘要(可以理解为`比较短的字符串`)，用来**标识**这个服务端的身份，用私钥加密后把**加密后的标识**和**自己的公钥**传给客户端。客户端拿到**这个公钥**来解密，生成另外一份摘要。两个摘要进行对比，如果相同则能确认服务端的身份。这也就是所谓**数字签名**的原理。其中除了哈希算法，最重要的过程是**私钥加密，公钥解密**。

#### step 2: Server Hello

可以看到服务器一口气给客户端回复了非常多的内容。

`server_random`也是最后生成`secret`的一个参数, 同时确认 TLS 版本、需要使用的加密套件和自己的证书，这都不难理解。那剩下的`server_params`是干嘛的呢？

#### step 3: Client 验证证书，生成secret

客户端验证服务端传来的`证书`和`签名`是否通过，如果验证通过，则**通过公钥加密传递`client_params`这个参数给服务器。**

接着客户端通过`ECDHE`算法计算出`pre_random`，其中传入两个参数:**server_params**和**client_params**。现在你应该清楚这个两个参数的作用了吧，由于`ECDHE`基于`椭圆曲线离散对数`，这两个参数也称作`椭圆曲线的公钥`。

客户端现在拥有了`client_random`、`server_random`和`pre_random`，接下来将这三个数通过一个伪随机数函数来计算出最终的`secret`。

#### step4: Server 生成 secret

刚刚客户端不是传了`client_params`过来了吗？服务端用私钥解密。

现在服务端开始用`ECDHE`算法生成`pre_random`，接着用和客户端同样的伪随机数函数生成最后的`secret`。

#### 注意事项

TLS的过程基本上讲完了，但还有两点需要注意。

**第一**、实际上 TLS 握手是一个**双向认证**的过程，从 step1 中可以看到，客户端有能力验证服务器的身份，那服务器能不能验证客户端的身份呢？

当然是可以的。具体来说，在 `step3`中，客户端传送`client_params`，实际上给服务器传一个验证消息，让服务器将相同的验证流程(哈希摘要 + 私钥加密 + 公钥解密)走一遍，确认客户端的身份。

**第二**、当客户端生成`secret`后，会给服务端发送一个收尾的消息，告诉服务器之后的都用对称加密，对称加密的算法就用第一次约定的。服务器生成完`secret`也会向客户端发送一个收尾的消息，告诉客户端以后就直接用对称加密来通信。

这个收尾的消息包括两部分，一部分是`Change Cipher Spec`，意味着后面加密传输了，另一个是`Finished`消息，这个消息是对之前所有发送的数据做的**摘要**，对摘要进行加密，让对方验证一下。

当双方都验证通过之后，握手才正式结束。后面的 HTTPS 正式开始传输加密报文。

### 数字签名过程

服务端将一段文本先用Hash函数生成消息摘要，然后用发送者的私钥加密生成数字签名，与原文文一起传送给接收者。接下来就是接收者校验数字签名的流程了。

接收者只有用发送者的公钥才能解密被加密的摘要信息，然后用HASH函数对收到的原文产生一个摘要信息，与上一步得到的摘要信息对比。如果相同，则说明收到的信息是完整的，在传输过程中没有被修改，否则说明信息被修改过，因此数字签名能够验证信息的完整性。

https://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html

### HTTPS 的单向认证和双向认证

**单向认证**

1. 客户端保存着服务器的证书并信任该证书
2. HTTPS 一般是单向认证，这样可以让绝大部分人都可以访问你的站点

**双向认证**

1. 先决条件是有2个或者2个以上的证书，一个服务器证书，其它是客户端证书
2. 服务器保存着客户端的证书并信任该证书，客户端保存着服务器的证书并信任该证书。这样，在证书验证成功的情况下即可完成请求响应
3. 双向认证一般用于企业应用对接（比如说堡垒机hh）

### HTTPS 握手过程中，客户端如何验证证书的合法性

1. 首先浏览器读取证书中的证书所有者、有效期等信息进行校验，**校验证书的网站域名是否与证书颁发的域名一致，校验证书是否在有效期内**
2. 浏览器开始查找操作系统中已内置的受信任的证书发布机构CA，与服务器发来的证书中的颁发者CA比对，用于**校验证书是否为合法机构颁发**
3. 如果找不到，浏览器就会报错，说明服务器发来的证书是不可信任的
4. 如果找到，那么浏览器就会从操作系统中取出颁发者CA 的公钥(多数浏览器开发商发布版本时，会事先在内部植入常用认证机关的公开密钥)，然后对服务器发来的证书里面的签名进行解密
5. 浏览器使用相同的hash算法计算出服务器发来的证书的hash值，将这个计算的hash值与证书中签名做对比
6. 对比结果一致，则证明服务器发来的证书合法，没有被冒充

## HTTP2.0

由于 HTTPS 在安全方面已经做的非常好了，HTTP 改进的关注点放在了性能方面。

### 特性

1. 头部压缩

   首先在服务器和客户端之间建立哈希表，将用到的字段存放在这张表中，那么在传输的时候对于之前出现过的值，只需要把**索引**(比如0，1，2，...)传给对方即可，对方拿到索引查表就行了。这种**传索引**的方式，可以说让请求头字段得到极大程度的精简和复用。

   其次是对于整数和字符串进行**哈夫曼编码**，哈夫曼编码的原理就是先将所有出现的字符建立一张索引表，然后让出现次数多的字符对应的索引尽可能短，传输的时候也是传输这样的**索引序列**，可以达到非常高的压缩率

2. 多路复用

   前面我们提到用**并发连接**和**域名分片**的方式来解决HTTP队头阻塞的问题，但这并没有真正从 HTTP 本身的层面解决问题，只是增加了 TCP 连接，分摊风险而已。

   HTTP2.0 实现了**多路复用**，用**一个TCP进行连接共享，一个请求对应一个id，这样就可以发送多个请求**，接收方通过id来响应不同的请求，解决了http1.1队首阻塞和连接过多的问题。因为http2.0**在同一域名不论访问多少文件都只有一个连接**，所以对服务器而言，提升的并发量是很大的。

3. 二进制分帧

   **帧(Frame)：**HTTP/2 数据通信的最小单位消息，指 HTTP/2 中逻辑上的 HTTP 消息。例如请求和响应等，消息由一个或多个帧组成。

   **流：**存在于连接中的一个虚拟通道。流可以承载双向消息，每个流都有一个唯一的整数ID。

   HTTP/2 采用二进制格式传输数据，而非 HTTP 1.x 的文本格式，二进制协议解析起来更高效。 HTTP / 1 的请求和响应报文，都是由起始行，首部和实体正文（可选）组成，各部分之间以文本换行符分隔。HTTP/2 将请求和响应数据分割为更小的帧，并且它们采用二进制编码。

   **HTTP/2 中，同域名下所有通信都在单个连接上完成，该连接可以承载任意数量的双向数据流。**每个数据流都以消息的形式发送，而消息又由一个或多个帧组成。多个帧之间可以乱序发送，根据帧首部的流标识可以重新组装。

   在应用层(HTTP/2)和传输层(TCP or UDP)之间增加一个二进制分帧层，在二进制分帧层中，HTTP/2 会将所有传输的信息分割为更小的消息和帧（frame），并对它们采用二进制格式的编码。

4. 服务器推送

   另外值得一说的是 HTTP/2 的服务器推送(Server Push)。在 HTTP/2 当中，服务器已经不再是完全被动地接收请求，响应请求，它也能新建 stream 来给客户端发送消息，当 TCP 连接建立之后，比如浏览器请求一个 HTML 文件，服务器就可以在返回 HTML 的基础上，将 HTML 中引用到的其他资源文件一起返回给客户端，减少客户端的等待，需要在服务端进行设置

### 问题

HTTP2.0 使用了多路复用，一般来说同一域名下只需要使用一个TCP 连接。

但是当连接中出现丢包时，整个TCP都要开始等待重传，后面的数据也都被阻塞了。而http1.0可以开启多个连接，只会影响一个，不会影响其他的。

所以**在丢包情况下，HTTP2.0 的情况反而不如 HTTP1.0**。

## HTTP3.0

为了解决2.0丢包性能的问题，Google基于UDP提出了QUIC协议。

HTTP3.0中的底层支撑协议就是QUIC。所以HTTP3.0也叫HTTP-over-QUIC。

#### QUIC协议

UDP协议高效，但不可靠。QUIC基于UDP，在原来的基础上结合了tcp和http的精华使它可靠。