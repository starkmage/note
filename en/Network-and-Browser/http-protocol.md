## Differences between GET and POST

- From a **caching** perspective, GET requests are actively cached by browsers and leave historical records, while POST requests are not cached by default.
- GET rollback is harmless, while POST will submit again.
- From an **encoding** perspective, GET can only use URL encoding and can only receive ASCII characters, while POST has no such limitations.
- From a **parameter** perspective, GET parameters are generally placed in the URL, making them less secure, while POST parameters are placed in the request body, making them more suitable for transmitting sensitive information.
- From an **idempotency** perspective, `GET` is **idempotent**, while `POST` is not. (`Idempotent` means performing the same operation will yield the same result)
- From a **TCP** perspective, GET requests send the request message in one go, while POST divides it into two TCP packets: first sending the header part, and if the server responds with 100 (continue), then sending the body part. (**Firefox** browser is an exception, as it sends POST requests in a single TCP packet)

* Parameter length limitations:

  The HTTP protocol does not specify length limits for GET and POST

  The maximum length of GET appears to be limited by browsers and servers

  Different browsers and web servers have different maximum length restrictions

  To support IE, the maximum length is 2083 bytes; for Chrome only, the maximum length is 8182 bytes

* When using xhr:

![](https://camo.githubusercontent.com/ef36bff2a3d3d0b12e96d481c20894fbaafb52f6/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f31322f31382f313637626366383365613762336662623f773d32393826683d31363926663d706e6726733d3335383634)

![](https://camo.githubusercontent.com/09f65a118cd0b462bbbc84928b8038be87f5bcaa/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f31322f31382f313637626366376264363564623737303f773d33343626683d31343926663d706e6726733d3431343234)

## HTTP Status Codes

### Five Categories

- 1XX: Informational
  - **100 Continue** Client should resend request
  - **101 Switching Protocols** Client requests server to switch protocols, if server agrees to change, it will send status code 101
- 2XX: Success
  - **200 OK** Operation successful
  - **201 Created** A new resource was created according to client request
  - **202 Accepted** Request cannot or will not be processed in real-time
  - **204 No Content** Request successful, but response contains no entity body
  - **205 Reset Content** Request successful, but response contains no entity body, requires client to reset content
  - **206 Partial Content** Used for range requests, indicates partial content, used in HTTP chunked downloads and resume broken downloads
- 3XX: Redirection
  - **301 Moved Permanently** Permanent redirection, resource has been assigned to a new URL
  - **302 Found** Temporary redirection, resource temporarily assigned to a URL (some clients treat it as 303)
  - **303 See Other** Indicates resource exists at another URL. Should use GET to retrieve resource
  - **304 Not Modified** Returned when cache negotiation hits
  - **307 Temporary Redirect** Temporary redirection, resource temporarily assigned to URL, but hopes client maintains method when requesting new address (solves issue of 302 being treated as 303)
- 4XX: Client Error
  - **400 Bad Request** Developers often see this vague error, which only generally indicates an error without specifying where
  - **401 Unauthorized** Request requires authentication, client attempts to operate on a protected resource without credentials
  - **403 Forbidden** Resource exists but access is denied, commonly used when a resource is only accessible during specific time periods (can report 404 if you don't want to reveal this)
  - **404 Not Found** Requested resource not found
  - **405 Method Not Allowed** Resource doesn't support the request method, e.g., only supports GET but received POST request
- 5XX: Server Error
  - **500 Internal Server Error** Only tells you server error occurred, specific error unknown
  - **501 Not Implemented** Server doesn't support this request method (differs from 405 in that 405 means the accessed resource doesn't support it, while 501 means server can't handle this method)
  - **502 Bad Gateway** Problem between proxy and upstream server, server itself is normal
  - **503 Service Unavailable** Server temporarily busy or under maintenance
  - **504 Gateway Timeout** Request timeout

### Several Distinctions

#### Possible Reasons for 204 Status Code

- Server refuses request and returns
- GET resource exists but indicates it's empty
  Server uses this response code to tell client: client's input has been accepted, but client should not change any UI elements

#### Difference between Status Codes 204 and 205

The difference between 204 and 205 is that **205 requires reset**!

Taking a form as an example, if it returns 204 after submission, the field values in the form remain unchanged and can continue to be modified; but if response code 205 is received, the fields in the form will be reset to their initial values.

## HTTP Characteristics

### Advantages

1. Flexible and extensible. This is mainly reflected in two aspects. One is semantic freedom, only basic formats are specified, such as spaces separating words, line breaks separating fields, with no strict syntax restrictions for other parts. The other is the diversity of transmission forms, not only can text be transmitted, but also images, videos, and any other data, which is very convenient.
2. Reliable transmission. HTTP is based on TCP/IP, so it inherits this characteristic. This belongs to TCP's features, no need for detailed introduction.
3. Request-response. That is `one send one receive`, `give and take`. Of course, the requester and responder don't only refer to the client and server; if a server acts as a proxy to connect to the backend server, that server will also play the role of **requester**.
4. Stateless. Each request is mutually independent and unrelated, the protocol doesn't require clients or servers to record request-related information.

### Disadvantages

#### Stateless

The so-called advantages and disadvantages still need to be viewed according to scenarios. For HTTP, the most controversial aspect is its **statelessness**.

In scenarios requiring long connections, a lot of context information needs to be saved to avoid transmitting large amounts of repeated information, in which case being stateless becomes HTTP's disadvantage.

However, at the same time, some applications only need to obtain some data and don't need to save connection context information, in which case being stateless reduces network overhead and becomes HTTP's advantage.

#### Plain text transmission, not secure

This means the messages in the protocol (mainly referring to the headers) don't use binary data but text form. **This "text" is different from that "text", what is hypertext? It's the form of binary files in the HTTP protocol, or encoding might be more appropriate** https://segmentfault.com/q/1010000006670932

This certainly provides convenience for debugging, but at the same time also exposes HTTP's message information to the outside world, providing convenience for attackers. `WIFI traps` exploit HTTP's plain text transmission weakness, inducing you to connect to hotspots, then frantically capturing all your traffic to obtain your sensitive information.

#### Head-of-line blocking

When HTTP enables long connections, sharing one TCP connection, only one request can be processed at a time. When the current request takes too long, other requests can only be in a blocked state, which is the famous **head-of-line blocking** problem.

## Differences between HTTP 1.1 and 1.0

HTTP 1.0:

1. Doesn't support long connections by default, needs to set keep-alive parameter to specify
2. Strong cache expires and negotiation cache last-modified/if-modified-since have certain defects

HTTP 1.1:

1. Long connections (keep-alive) by default, HTTP requests can reuse TCP connections, but only one HTTP request can correspond at a time (HTTP requests are serial in one TCP)
2. Added strong cache control-cache and negotiation cache Etag/if-none-match as optimizations for HTTP/1 caching

## Common HTTP Headers

|              |                Request                |                           Response                            |
| :----------: | :--------------------------------: | :-------------------------------------------------------: |
|    Start line    |         GET /home HTTP/1.1         |                 HTTP/1.1 200 OK (Status line)                 |
|   Data format   |         Accept: text/html          |                  Content-Type: text/html                  |
|   Compression   |       Accept-Encoding: gizp        |                  Content-Encoding: gzip                   |
|   Language   |   Accept-Language: zh-CN, zh, en   |              Content-Language: zh-CN, zh, en              |
|    Charset    |   Accept-Charset: charset=utf-8    |          Content-Type: text/html; charset=utf-8           |
| Resource modification time |     If-Modified-Since: timestamp      |                   Last-Modified: timestamp                   |
|  Resource identifier  |     If-None-Match: hash string      |                     Etag: hash string                      |
|   Cache control   | cache-control: max-age=0(refresh data) | cache-control: public/private/no-cache/no-store/max-age=X |
|    Long connection    |    connection: keep-alive/close    |               connection: keep-alive/close                |

### Common content-type Types

Common media format types are as follows:

- text/html: HTML format
- text/plain: Plain text format
- text/xml: XML format
- image/gif: GIF image format
- image/jpeg: JPG image format
- image/png: PNG image format

Media format types starting with application:

- application/xhtml+xml: XHTML format
- application/xml: XML data format
- application/atom+xml: Atom XML aggregation format
- application/json: JSON data format
- application/pdf: PDF format
- application/msword: Word document format
- application/octet-stream: Binary stream data (such as common file downloads)
- application/x-www-form-urlencoded: Default encType in <form encType="">, form data is encoded as key/value format sent to server (default format for submitting form data)

Another common media format used when uploading files:

- multipart/form-data: Needed when files need to be uploaded in forms

## HTTP Fixed-length and Variable-length Body Transmission

http://47.98.159.95/my_blog/http/007.html#%E5%AE%9A%E9%95%BF%E5%8C%85%E4%BD%93

### Fixed-length Body

```http
content-length: 20(body length)
```

`Content-Length` plays a crucial role in HTTP transmission process. If set incorrectly, it can directly lead to transmission failure: if set too small, only part of the data can be transmitted; if set too large, it directly leads to transmission failure.

### Variable-length Body

```http
Transfer-Encoding: chunked
```

Indicates chunked data transmission. Setting this field automatically produces two effects:

- Content-Length field will be ignored
- Continuous push of dynamic content based on long connection

## HTTP Handling Large File Requests

http://47.98.159.95/my_blog/http/008.html#%E5%A6%82%E4%BD%95%E6%94%AF%E6%8C%81

For this scenario, HTTP adopted the `range request` solution, allowing clients to request only a portion of a resource. Uses the Ranges field.

## HTTP Handling Form Data Submission

Since form submissions are generally `POST` requests, rarely considering `GET`, here we'll assume the submitted data is in the request body by default.

### application/x-www-form-urlencoded

For form content in `application/x-www-form-urlencoded` format, there are following characteristics:

- Data will be encoded as key-value pairs separated by `&`
- Characters are encoded using **URL encoding** method

For example:

```text
// Conversion process: {a: 1, b: 2} -> a=1&b=2 -> as follows (final form)
"a%3D1%26b%3D2"
```

### multipart/form-data

For `multipart/form-data`:

- The `Content-Type` field in request header will contain `boundary`, and the value of `boundary` is specified by browser by default. Example: `Content-Type: multipart/form-data;boundary=----WebkitFormBoundaryRRJKeWfHPGrS4LKe`.
- Data will be divided into multiple parts, separated by separators between every two parts, each part description has HTTP header describing sub-body, such as `Content-Type`, and `--` will be added to the last separator to indicate the end.

The corresponding `request body` looks like this:

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
### Comparison

It's worth noting that the biggest feature of the `multipart/form-data` format is: **each form element is an independent resource representation**. No encoding is performed, binary data is sent.

Additionally, in our business development process, we haven't noticed the existence of `boundary`. If you open packet capture tools, you can indeed see different form elements being split apart. The reason we don't feel it in daily use is because browsers and HTTP have encapsulated this series of operations for us.

Moreover, **in actual scenarios, for uploading files like images, `multipart/form-data` is generally used instead of `application/x-www-form-urlencoded`, because there's no need for URL encoding, which not only brings huge time consumption but also occupies more space.**

## HTTP Long Connection

The definition of a long connection simply means that after establishing a TCP connection once, multiple HTTP requests and responses will be made.

Since long connections significantly improve performance, **all connections in HTTP/1.1 enable long connections by default. No special header fields are needed**. Once you send the first request to the server, subsequent requests will reuse the first opened TCP connection, which is the long connection, to send and receive data.

Of course, we can also explicitly request the use of long connection mechanism in the request header using the Connection field with value "keep-alive".

However, regardless of whether the client explicitly requests a long connection, if the server supports long connections, it will always put a "Connection: keep-alive" field in the response message, telling the client: "I support long connections, let's keep using this TCP to send and receive data."

### Issues with Long Connections

Because TCP connections remain open for a long time, the server must keep their states in memory, which consumes server resources. If there are many idle long connections that only connect but don't send data, it will quickly exhaust server resources, preventing the server from serving users who really need it.

Therefore, long connections also need to be closed at appropriate times, they can't maintain connections with the server forever, and this can be done on either client or server side.

On the client side, you can add the "Connection: close" field in the request header, telling the server: "Close the connection after this communication." When the server sees this field, it knows the client wants to actively close the connection, so it also adds this field in the response message and calls the Socket API to close the TCP connection after sending.

> Servers usually don't actively close connections, but they can use some strategies. Taking Nginx as an example, it has two methods:

Use the "keepalive_timeout" directive to set the timeout for long connections. If there's no data transmission on the connection for a period of time, it will actively disconnect to avoid idle connections occupying system resources.

Use the "keepalive_requests" directive to set the maximum number of requests that can be sent on a long connection. For example, if set to 1000, Nginx will actively disconnect after processing 1000 requests on this connection. Additionally, both client and server can attach the general header field "Keep-Alive: timeout=value" in messages to limit the timeout of long connections. However, this field's constraint isn't strong, both parties in communication might not comply, so it's not very common.

### Pipelining

Pipelining is a method to further improve communication efficiency based on persistent connections. Multiple HTTP requests on one TCP connection no longer proceed sequentially with request-response, where the next request can only be sent after receiving the response to the previous request. Pipelining technology allows **clients to sequentially send other requests before receiving responses**. This can improve communication efficiency in high-latency situations.

Because HTTP messages don't have sequence numbers, responses must be sent in the order requests came in. If the order is mixed up, the client can't match them.

This is different from multiplexing in HTTP2.0, where there are IDs that ensure order won't be mixed up.

## Head-of-line Blocking

"Head-of-line blocking" is unrelated to short or long connections, but is caused by HTTP's basic "request-response" model.

Because HTTP stipulates that messages must be "one send one receive", this forms a first-in-first-out "serial" queue. Requests in the queue don't have priority levels of urgency, only the order of entry, and requests at the front of the queue are processed first.

If the request at the front of the queue takes too long to process, all requests behind it in the queue have to wait together, resulting in other requests bearing undue time costs.

### Concurrent Connections

Allowing multiple long connections for one domain name is equivalent to increasing task queues, preventing tasks in one queue from blocking all other tasks. RFC2616 stipulated that clients could have at most 2 concurrent connections, but in fact, in current browser standards, this limit is much higher, with Chrome having 6.

However, even with increased concurrent connections, it still can't meet people's performance demands.

### Domain Sharding

If a domain name can have 6 concurrent long connections, then I'll just split into several domain names.

For example, content1.sanyuan.com, content2.sanyuan.com.

This way, a `sanyuan.com` domain can split into many subdomains, and they all point to the same server, allowing for more concurrent long connections, which actually better solves the head-of-line blocking problem.

## Cookie

As mentioned earlier, HTTP is a stateless protocol, each HTTP request is independent and unrelated, by default not needing to retain state information. But sometimes we need to save some states, what to do?

HTTP introduced Cookie for this. Cookie is essentially a very small text file stored in the browser, internally storing key-value pairs (can be seen in the Application tab of chrome developer panel). Requests sent to the same domain name will carry the same Cookie, the server can parse the Cookie to get the client's state. And the server can write Cookie to the client through the `Set-Cookie` field in the response header. Example as follows:

```http
// Request header
Cookie: a=xxx;b=xxx
// Response header
Set-Cookie: a=xxx
set-Cookie: b=xx
```

### Cookie Attributes

#### Lifecycle

Cookie's validity period can be set through two attributes: **Expires** and **Max-Age**.

- **Expires** is the `expiration time`
- **Max-Age** uses a time interval in seconds, calculated from when the browser receives the message.

If a Cookie expires, it will be deleted and not sent to the server.

#### Scope

Regarding scope, there are also two attributes: **Domain** and **path**. These bind the **Cookie** to a domain name and path. Before sending a request, if the domain name or path doesn't match these two attributes, the Cookie won't be included. Note that for paths, `/` means the Cookie is allowed to be used under any path of the domain name.

#### Security Related

If the cookie field includes `HttpOnly`, it means it can only be transmitted through HTTP protocol and can't be accessed through JS, which is an important means of preventing XSS attacks.

Correspondingly, for preventing CSRF attacks, there's also the `SameSite` attribute.

`SameSite` can be set to three values: `Strict`, `Lax`, and `None`.

**a.** In `Strict` mode, browsers completely prohibit third-party requests from carrying Cookies. For example, requests to `a.com` website can only carry Cookies when requested within the `a.com` domain, not from other websites.

**b.** In `Lax` mode, it's a bit more lenient, but Cookies can only be carried in cases of `submitting forms with get method` or `a tags sending get requests`, not in other cases.

**c.** In `None` mode, which is the default mode, requests will automatically carry Cookies.

### Cookie Disadvantages

1. Capacity limitations. Cookie's volume limit is only `4KB`, can only be used to store small amounts of information.
2. Performance limitations. Cookies follow domain names, regardless of whether a certain address under the domain needs this Cookie or not, requests will carry the complete Cookie, which actually causes huge performance waste as the number of requests increases, because requests carry a lot of unnecessary content. However, this can be solved by specifying **scope** through `Domain` and `Path`.
3. Security limitations. Since Cookies are transmitted in plain text between browsers and servers, they can easily be intercepted by unauthorized users, who can then make a series of modifications and resend to the server within the Cookie's validity period, which is quite dangerous. Additionally, when `HttpOnly` is false, Cookie information can be read directly through JS scripts.

## HTTP Caching

### Strong Cache

When initiating a network request, the network process first checks the strong cache, this phase `does not need` to send an HTTP request.

In the early `HTTP/1.0` era, **Expires** was used, while `HTTP/1.1` uses **Cache-Control**. When both **Expires** and **Cache-Control** exist, **Cache-Control** takes precedence.

#### Expires

`Expires` is the expiration time, existing in the response header returned by the server, telling the browser that before this expiration time it can directly get data from the cache without requesting again.

```http
Expires: Wed, 22 Nov 2019 08:41:00 GMT
```

This approach seems fine and reasonable, but actually hides a pitfall, which is that **server time and browser time might not be consistent**, so this expiration time returned by the server might be inaccurate.

#### Cache-Control

Its essential difference from `Expires` is that it doesn't use the approach of `specific expiration time point`, but uses expiration duration to control caching, corresponding to the **max-age** field. For example:

```http
Cache-Control:max-age=3600
// means within 3600 seconds, or one hour, after this response returns, cache can be used directly
```

Many directives can be combined: public, private, no-store, no-cache, s-maxage, must-revalidate (if time exceeds `max-age`, browser must send request to server to verify if resource is still valid)

#### Pragma

`Pragma` is a historical legacy field retained from versions before HTTP/1.1, defined only for backward compatibility with HTTP/1.0. The specification defines **only one form**, as shown below:

```http
Pragma: no-cache
```
### Negotiation Cache

After strong cache expires, the browser carries corresponding `cache tags` in the request header to send requests to the server. The server decides whether to use cache based on these tags, this is **negotiation cache**.

Specifically, these cache tags are divided into two types: **Last-Modified** and **ETag**. These two have their own advantages and disadvantages, there is no `absolute advantage` of one over the other, unlike the two tags in strong cache mentioned above.

#### Last-Modified

After the browser sends the first request to the server, the server will add this Last-Modified field in the response header.

After the browser receives it, if it requests again, it will carry the `If-Modified-Since` field in the request header, and the value of this field is the last modification time sent by the server.

After the server gets the `If-Modified-Since` field from the request header, it will compare it with `the last modification time of this resource` on the server:

- If this value in the request header is less than the last modification time, it means it's time to update. Return new resource, same as the regular HTTP request response process.
- Otherwise return 304, telling the browser to use cache directly.

#### ETag

ETag is the abbreviation of "Entity Tag", it's a unique identifier for resources, mainly used to solve the problem that modification time cannot accurately distinguish file changes.

For example, if a file is modified multiple times within one second, because the modification time is in seconds, new versions within this second cannot be distinguished.

Another example, if a file is updated regularly but sometimes has the same content, there's actually no change, using modification time would mistakenly think changes occurred, sending it to the browser would waste bandwidth.

Using ETag can precisely identify resource changes, allowing browsers to use cache more effectively.

The server sends this value to the browser through the `response header`.

After the browser receives the `ETag` value, in the next request, it will use this value as the content of the `If-None-Match` field and put it in the request header, then send it to the server.

#### Comparison of the Two

1. In terms of `precision`, `ETag` is better than `Last-Modified`. ETag labels resources according to content, so it can accurately detect resource changes. Last-Modified is different, it cannot accurately detect resource changes in some special cases, mainly in two situations:
   * Edited the resource file, but file content hasn't changed, this will also cause cache invalidation;
   * Last-Modified can only sense time units in seconds, if file changed multiple times within 1 second, then Last-Modified doesn't reflect the changes.

2. In terms of performance, `Last-Modified` is better than `ETag`, which is also easy to understand, `Last-Modified` only records a time point, while `Etag` needs to generate hash values based on specific file content.

Additionally, if both methods are supported, the server will prioritize `ETag`.

### Proxy Cache

For HTTP caching, if client cache invalidation always requires fetching from the origin server, that puts a lot of pressure on the origin server.

This introduced the mechanism of **cache proxy**. Let `proxy servers` take over part of the server-side HTTP caching, when client cache expires, get from nearby proxy cache, only request from origin server when proxy cache expires, this can significantly reduce pressure on origin servers when traffic is huge.

So how does cache proxy achieve this?

Overall, cache proxy control is divided into two parts, one part is **origin server** control, another part is **client** control.

#### Origin Server Control

1. private and public

   In the origin server's response header, the `Cache-Control` field will be added for cache control, and its value can include `private` or `public` to indicate whether proxy server caching is allowed, former prohibits, latter allows.

2. s-maxage

   `s` means `share`, it limits how long cache can stay in proxy servers, and doesn't conflict with `max-age` which limits client cache time.

   Here's a small example, origin server adds such a field in response header:

   ```text
   Cache-Control: public, max-age=1000, s-maxage=2000
   ```

   This means the origin server is saying: this response allows proxy server caching, client should get from proxy when client cache expires, and client cache time is 1000 seconds, proxy server cache time is 2000 seconds.

3. proxy-revalidate

   `must-revalidate` means **client** gets from origin server when cache expires, while `proxy-revalidate` means **proxy server** gets from origin server when cache expires.

#### Client Control

1. max-stale and min-fresh

   ```text
   max-stale: 5
   // means when client gets cache from proxy server, it's okay even if proxy cache has expired, as long as expiration time is within 5 seconds, it can still be obtained from proxy.
   min-fresh: 5
   // means proxy cache needs certain freshness, don't wait until cache is about to expire to get it, must get it at least 5 seconds before expiration, otherwise can't get it.
   ```

2. only-if-cached

   Adding this field means client will only accept proxy cache, and won't accept origin server's response. If proxy cache is invalid, it directly returns `504 (Gateway Timeout)`.

### Cache Location

From cache location perspective, there are four types, and they each have **priority**. Only when **cache is searched sequentially and none hit**, will it request network. The order is:

1. Service Worker
2. Memory Cache
3. Disk Cache
4. Push Cache
5. Network Request

#### Service Worker

- `Service Worker` is an independent thread running in the background of browser, generally can be used to implement caching functionality. To use `Service Worker`, transmission protocol must be `HTTPS`. Because `Service Worker` involves request interception, `HTTPS` protocol must be used to ensure security
- Implementing cache functionality with `Service Worker` generally involves three steps: first need to register `Service Worker`, then after listening to `install` event can cache needed files, then next time user visits can query whether cache exists through request interception method, if cache exists can directly read cache files, otherwise request data.

- `Service Worker` caching is different from other built-in browser caching mechanisms, it lets us freely control which files to cache, how to match cache, how to read cache, and cache is persistent.
- When `Service Worker` doesn't hit cache, we need to call `fetch` function to get data. That is, if we don't hit cache in `Service Worker`, will search data according to cache search priority. But no matter whether we get data from `Memory Cache` or network request, browser will show we got content from `Service Worker`.

#### Memory Cache

`Memory Cache` refers to memory cache, from efficiency perspective it's fastest. But from survival time perspective it's shortest, when rendering process ends, memory cache no longer exists. That is, once we close Tab page, cache in memory is also released.

#### Disk Cache

`Disk Cache` is cache stored on hard disk, reading speed is slower, but anything can be stored to disk, compared to `Memory Cache` **wins in capacity and storage time effectiveness.**

Among all browser caches, `Disk Cache` basically has largest coverage. It will judge which resources need caching, which resources can be used directly without requesting, which resources have expired and need re-requesting based on HTTP Header fields. **And even in cross-site situations, once resources of same address are cached on disk, won't request data again.**

How does browser decide whether to put resources in memory or disk? Main strategies are:

- Larger JS, CSS files will be thrown directly to disk, opposite thrown to memory
- When memory usage is high, files prioritize going to disk

#### Push Cache

`Push Cache` is content in HTTP/2, namely push cache. It's only used when above three caches all miss, **and cache time is very short, only exists in session, once session ends it's released.**

### Browser Behavior

What cache strategies are triggered when users operate in browser. Mainly three types:

- Open webpage, enter address in address bar: Search if there's match in disk cache. If yes use it; if no send network request.
- Normal refresh (F5): Because TAB hasn't closed, memory cache is available, will be used first (if matches). Then disk cache.
- Force refresh (Ctrl + F5): Browser doesn't use cache, so request headers all carry `Cache-control: no-cache`(for compatibility, also carries `Pragma: no-cache`). Server directly returns 200 and latest content.

Reference articles:

[Understanding Frontend Cache in One Article](https://juejin.cn/post/6844903747357769742?utm_source=gold_browser_extension%3Futm_source%3Dgold_browser_extension#heading-0)

## HTTPS

`HTTPS` isn't a new protocol, but an enhanced version of `HTTP`. Its principle is establishing an intermediate layer between `HTTP` and `TCP`, when `HTTP` and `TCP` communicate they don't communicate directly like before, but go through an intermediate layer for encryption, passing encrypted data packets to `TCP`, correspondingly, `TCP` must decrypt data packets before passing to `HTTP` above. This intermediate layer is also called `security layer`. The core of `security layer` is data `encryption and decryption`.

Simply put, **HTTPS = HTTP + SSL/TLS**. **So-called HTTPS is actually HTTP wrapped in SSL protocol shell**.

So what is SSL/TLS?

SSL means Secure Sockets Layer, located in session layer (Layer 5) in OSI seven-layer model. Previously SSL had three major versions, when it developed to third major version it was standardized, became TLS (Transport Layer Security), and was treated as TLS1.0 version, accurately speaking, **TLS1.0 = SSL3.1**. Current mainstream version is TLS/1.2.

### Why is Http not secure?

1. Data transmitted in plaintext, risk of being eavesdropped
2. Received messages can't prove they're same as sent messages, can't guarantee integrity, so messages risk being tampered
3. Doesn't verify identities of both communication ends, requests or responses risk being forged

In contrast, Https:

- Data privacy: Content undergoes symmetric encryption, each connection generates unique encryption key
- Data integrity: Content transmission undergoes integrity check—hash function, digital signature
- Identity authentication: Third parties can't forge server (client) identity—digital certificate

### What are differences between Http and Https?

1. HTTP is hypertext transfer protocol, information is transmitted in **plaintext**, HTTPS is SSL encrypted transmission protocol with security
2. HTTP and HTTPS use completely different connection methods, use different **ports, former is 80, latter is 443**
3. HTTPS protocol needs CA certificate application, generally few free certificates, thus needs certain cost
4. HTTP connection is simple, stateless; HTTPS protocol is encrypted by **TSL** protocol, safer than HTTP protocol, but also stateless

### What are disadvantages of Https?

1. Both communication ends need to perform encryption and decryption, consuming large amounts of CPU, memory and other resources, **increasing server pressure**
2. Encryption calculations and multiple handshakes **reduce access speed**
3. During development phase, **increases page debugging difficulty**. Because information is all encrypted, when using proxy tools, need to decrypt first then can see real information
4. Pages accessed using HTTPS, external resources in page must all use HTTPS requests, including AJAX requests in scripts

### Symmetric Encryption and Asymmetric Encryption

`Symmetric encryption` is simplest way, meaning `encryption` and `decryption` use **same key**.

As for `asymmetric encryption`, if there are two keys A and B, if data packet encrypted with A can only be decrypted with B, conversely, if data packet encrypted with B can only be decrypted with A.

### Traditional RSA Handshake

http://47.98.159.95/my_blog/browser-security/003.html#%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86%E5%92%8C%E9%9D%9E%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86

Reason it's called RSA version is because it uses RSA algorithm when encrypting and decrypting `pre_random`.

This is combination of symmetric and asymmetric encryption,

1. Browser sends `client_random` and encryption method list to server.
2. Server receives, returns `server_random`, encryption method and public key.
3. Browser receives, then generates another random number `pre_random` through RSA algorithm, and encrypts with public key, sends to server. (Key operation!)
4. Server decrypts this encrypted `pre_random` with private key.

Now browser and server have three same credentials: `client_random`, `server_random` and `pre_random`. Then both use same pseudorandom function to mix these three random numbers, generating final `key`.

Then browser and server just use same key to communicate, namely use `symmetric encryption`.

![](http://47.98.159.95/my_blog/week12/1.jpg)

You'll find there's no public key in image but digital certificate, in fact `HTTPS` added `digital certificate authentication` step on basis of above `combining symmetric and asymmetric encryption`. Its purpose is to let server prove its identity.

To obtain this certificate, server operator needs to get authorization from third-party certification authority, this third-party institution is also called `CA`(`Certificate Authority`), after certification passes CA will issue **digital certificate** to server.

This digital certificate has two purposes:

1. **Server proves its identity to browser.**
2. **Pass public key to browser.**

When does this verification process happen?

When server sends `server_random`, encryption method, it will also bring along `digital certificate`(contains `public key`), then browser starts verifying digital certificate after receiving. If verification passes, then following process proceeds normally, otherwise refuses to execute.

### TLS1.2 Handshake

http://47.98.159.95/my_blog/http/015.html

In **TLS handshake phase, both ends use asymmetric encryption method to handshake**, but because asymmetric encryption consumes more performance than symmetric encryption, so **when formally transmitting data, both ends actually use symmetric encryption method to communicate**.

![](http://47.98.159.95/my_blog/http/010.jpg)

#### step 1: Client Hello

First, browser sends client_random, TLS version, encryption suite list.

What is client_random? A parameter used for final secret.

What is encryption suite list? I'll give an example, encryption suite list generally looks like this:

```text
TLS_ECDHE_WITH_AES_128_GCM_SHA256
```
This means during `TLS` handshake process, using `ECDHE` algorithm to generate `pre_random` (this number will be introduced later), 128-bit `AES` algorithm for symmetric encryption, using mainstream `GCM` grouping mode during symmetric encryption process, because how to group is a very important issue in symmetric encryption. The last one is hash digest algorithm, using `SHA256` algorithm.

Worth explaining is this hash digest algorithm. Imagine such a scenario: server now sends a message to client, client doesn't know whether this message is sent by server or forged by man-in-the-middle? Now introducing this hash digest algorithm, generating a digest (can be understood as a `relatively short string`) through **this algorithm** from server's certificate information, used to **identify** this server's identity, encrypting with private key then sending **encrypted identifier** and **its own public key** to client. Client uses **this public key** to decrypt, generates another digest. Compare the two digests, if they're same then can confirm server's identity. This is the principle of so-called **digital signature**. Among these, besides hash algorithm, most important process is **private key encryption, public key decryption**.

#### step 2: Server Hello

We can see server replied to client with lots of content at once.

`server_random` is also a parameter for finally generating `secret`, while confirming TLS version, encryption suite to use and its own certificate, these are not hard to understand. So what is remaining `server_params` for?

#### step 3: Client Verifies Certificate, Generates secret

Client verifies whether server's sent `certificate` and `signature` pass, if verification passes, then **encrypts and transmits `client_params` parameter to server using public key.**

Then client calculates `pre_random` through `ECDHE` algorithm, with two parameters input: **server_params** and **client_params**. Now you should understand what these two parameters are for, since `ECDHE` is based on `elliptic curve discrete logarithm`, these two parameters are also called `elliptic curve public keys`.

Client now has `client_random`, `server_random` and `pre_random`, next will calculate final `secret` through a pseudorandom function using these three numbers.

#### step4: Server Generates secret

Didn't client just send `client_params` over? Server decrypts with private key.

Now server starts using `ECDHE` algorithm to generate `pre_random`, then uses same pseudorandom function as client to generate final `secret`.

#### Notes

TLS process is basically explained, but there are two points to note.

**First**, actually TLS handshake is a **bidirectional authentication** process. From step1 we can see client has ability to verify server's identity, so can server verify client's identity?

Of course it can. Specifically, in `step3`, when client sends `client_params`, it's actually sending a verification message to server, letting server go through same verification process (hash digest + private key encryption + public key decryption) to confirm client's identity.

**Second**, after client generates `secret`, it will send a finishing message to server, telling server to use symmetric encryption afterwards, using encryption algorithm agreed upon first time. After server generates `secret` it will also send a finishing message to client, telling client to directly use symmetric encryption for communication from now on.

This finishing message includes two parts, one part is `Change Cipher Spec`, meaning encrypted transmission follows, another is `Finished` message, this message is a **digest** of all previously sent data, encrypting the digest, letting other party verify.

Only after both parties verify successfully does handshake formally end. Following HTTPS formally starts transmitting encrypted messages.

### Digital Signature Process

Server first uses Hash function to generate message digest from a text, then uses sender's private key to encrypt generating digital signature, sending together with original text to receiver. Next is receiver's process of verifying digital signature.

Receiver can only decrypt encrypted digest information using sender's public key, then uses HASH function to generate a digest information from received original text, comparing with digest information obtained in previous step. If same, means received information is complete, hasn't been modified during transmission, otherwise means information has been modified, therefore digital signature can verify information integrity.

https://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html

### HTTPS One-way and Two-way Authentication

**One-way Authentication**

1. Client stores and trusts server's certificate
2. HTTPS generally uses one-way authentication, this allows vast majority of people to access your site

**Two-way Authentication**

1. Prerequisite is having 2 or more certificates, one server certificate, others are client certificates
2. Server stores and trusts client's certificate, client stores and trusts server's certificate. This way, request response can be completed when certificate verification succeeds
3. Two-way authentication generally used for enterprise application connection (like bastion host hh)

### How Client Verifies Certificate Legitimacy During HTTPS Handshake

1. First browser reads certificate owner, validity period and other information from certificate for verification, **verifies whether certificate's website domain matches certificate-issued domain, verifies whether certificate is within validity period**
2. Browser starts searching for trusted certificate authorities CA pre-installed in operating system, comparing with issuer CA in server-sent certificate, used to **verify whether certificate was issued by legitimate institution**
3. If not found, browser will report error, indicating server-sent certificate is untrustworthy
4. If found, then browser will extract issuer CA's public key from operating system (most browser developers pre-embed commonly used certification authorities' public keys when releasing versions), then decrypt signature in server-sent certificate
5. Browser uses same hash algorithm to calculate hash value of server-sent certificate, compares this calculated hash value with signature in certificate
6. If comparison results match, proves server-sent certificate is legitimate, hasn't been impersonated

## HTTP2.0

Since HTTPS has done very well in security aspect, HTTP improvements focused on performance aspect.

### Features

1. Header Compression

   First establish hash table between server and client, storing used fields in this table, then during transmission for previously appeared values, only need to pass **index** (like 0, 1, 2, ...) to other party, other party can look up table with index. This way of **passing index** can be said to have greatly simplified and reused request header fields.

   Secondly is performing **Huffman coding** on integers and strings, principle of Huffman coding is first establishing an index table for all appearing characters, then making indexes corresponding to frequently appearing characters as short as possible, during transmission also transmitting such **index sequences**, can achieve very high compression rate

2. Multiplexing

   Previously we mentioned using **concurrent connections** and **domain sharding** to solve HTTP head-of-line blocking problem, but this didn't really solve problem from HTTP's own level, just increased TCP connections, sharing risk only.

   HTTP2.0 implemented **multiplexing**, using **one TCP for connection sharing, one request corresponding to one id, this way can send multiple requests**, receiver responds to different requests through id, solving http1.1's head-of-line blocking and too many connections problems. Because http2.0 **has only one connection regardless of how many files accessed under same domain name**, so for server, improvement in concurrency is very large.

3. Binary Framing

   **Frame:** Smallest unit message of HTTP/2 data communication, referring to logical HTTP messages in HTTP/2. For example requests and responses, messages consist of one or more frames.

   **Stream:** A virtual channel existing in connection. Stream can carry bidirectional messages, each stream has a unique integer ID.

   HTTP/2 adopts binary format to transmit data, rather than HTTP 1.x's text format, binary protocol is more efficient to parse. HTTP/1's request and response messages all consist of start line, header and entity body (optional), parts separated by text line breaks. HTTP/2 divides request and response data into smaller frames, and they adopt binary encoding.

   **In HTTP/2, all communication under same domain name is completed on single connection, this connection can carry arbitrary number of bidirectional data streams.** Each data stream is sent in form of messages, and messages consist of one or more frames. Multiple frames can be sent out of order, can be reassembled according to stream identifier in frame header.

   Adding a binary framing layer between application layer (HTTP/2) and transport layer (TCP or UDP), in binary framing layer, HTTP/2 will divide all transmitted information into smaller messages and frames, and encode them in binary format.

4. Server Push

   Also worth mentioning is HTTP/2's server push (Server Push). In HTTP/2, server is no longer completely passive in receiving requests and responding to requests, it can also create new stream to send messages to client, when TCP connection is established, for example when browser requests an HTML file, server can return other resource files referenced in HTML along with HTML to client, reducing client's waiting, needs to be set on server side

### Issues

HTTP2.0 uses multiplexing, generally speaking only needs to use one TCP connection under same domain name.

But when packet loss occurs in connection, entire TCP has to start waiting for retransmission, data behind also gets blocked. While http1.0 can open multiple connections, will only affect one, won't affect others.

So **in packet loss situations, HTTP2.0's situation is actually worse than HTTP1.0**.

## HTTP3.0

To solve 2.0's packet loss performance problem, Google proposed QUIC protocol based on UDP.

Underlying support protocol in HTTP3.0 is QUIC. So HTTP3.0 is also called HTTP-over-QUIC.

#### QUIC Protocol

UDP protocol is efficient but unreliable. QUIC based on UDP, combining essence of tcp and http on original basis makes it reliable.