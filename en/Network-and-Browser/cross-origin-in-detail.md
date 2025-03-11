## What is Cross-Origin

When a browser sends an Ajax request to a target URL, if the current URL and target URL are not from the same origin, it creates a cross-origin situation, called a `cross-origin request`.

So what is same-origin?

Browsers follow the **same-origin policy** (`scheme(protocol)`, `domain`, and `port` must all be the same to be considered `same-origin`). Non-same-origin sites have these restrictions:

- **DOM Level**. Same-origin policy restricts JavaScript scripts from different origins from reading and writing operations on the current DOM object.

* **Data Level**. Same-origin policy restricts sites from different origins from reading current site's `Cookie`, `IndexDB`, `LocalStorage`, and other data.

* **Network Level**. Cross-origin request **responses** are generally blocked by the browser. Note that it's blocked by the browser, **the response actually successfully reaches the client**.

## What is Cross-Site

As long as two URLs have the same eTLD+1, they are considered same-site, regardless of protocol and port

**eTLD**: (effective top-level domain) registered in Mozilla's Public Suffix List, like `.com`, `.co.uk`, `.github.io`, `.top`, etc.

**eTLD+1**: effective top-level domain + second-level domain, like `taobao.com`, `baidu.com`, `sugarat.top`

Strictly speaking, cookies follow same-site policy.

Same-site also reuses one rendering process.

https://juejin.cn/post/6926731819903631368

## How is Ajax Response Blocked

As mentioned above, when making a cross-origin request, the response successfully reaches the client but is blocked by the browser. So what exactly happens during the process from sending the request to the response being blocked?

First, we know browsers are multi-process, and Ajax requests are constructed in JS code, which is in the rendering process. This means when `xhr.send` is called and the Ajax request is ready to be sent, it's still being processed in the rendering process.

Additionally, we know that to prevent hackers from accessing system resources through scripts, browsers put each rendering process in a sandbox. And to prevent CPU chip's persistent Spectre and Meltdown vulnerabilities, they adopted `site isolation`, giving each different site (different first-level domain) its own sandbox, not interfering with each other. Sandboxes cannot send network requests, they can only send through the network process, which involves inter-process communication (IPC).

Through IPC, data is passed to the network process, and only after the network process receives it does it actually send out the corresponding network request.

After the server processes the data and returns the response, the network process detects cross-origin and no CORS response headers, discards the entire response body, and doesn't send it to the rendering process, thus achieving the goal of intercepting data.

## Cross-Origin Solutions

### JSONP

Although the `XMLHttpRequest` object follows same-origin policy, `script` tags are different - they have no cross-origin restrictions. They can make **GET** requests by filling the target address in `src`, thus achieving cross-origin requests and getting responses. This is the principle of JSONP. Specifically:

- Create a `script` tag, its `src` is the request address
- Insert this `script` tag into DOM, browser accesses server resources based on `src` address
- Returned resource is text, but because it's in `script` tag, browser executes it
- This text happens to be in function call form, i.e., function name (data), browser executes it as JS code calling this function
- As long as this function name is agreed upon beforehand and function exists in `window` object, data can be passed to handling function

Through `<script>` tag pointing to an address that needs to be accessed and providing a callback function to receive data when communication is needed, a key point of this protocol is that it **allows users to pass a `callback` parameter to server side, then when server returns data it will wrap the JSON data with this `callback` parameter as function name, so client can freely customize their function to automatically handle returned data**.

Just looking at frontend, simplest form should be like this:

```html
<script src="http://domain/api?param1=a&param2=b&callback=myFn"></script>
<script>
    function myFn(data) {
        console.log(data)
    }
</script>
```

JSONP is simple to use and has good compatibility, but is limited to `get` requests.

In development, you might encounter multiple JSONP requests with same callback function names. In this case, you need to encapsulate your own JSONP. Here's a simple implementation:

```javascript
function jsonp(url, params, jsonpCallback, success) {
  let script = document.createElement('script')
  params = { ...params, jsonpCallback } // wd=b&callback=show
  let arrs = []
  for (let key in params) {
    arrs.push(`${key}=${params[key]}`)
  }
  script.src = `${url}?${arrs.join('&')}`
  script.async = true
  window[jsonpCallback] = function(data) {
    success && success(data)
  }
  document.body.appendChild(script)
}
// Usage
jsonp('http://xxx', {type: 'zzz'}, 'callback', function(value) {
  console.log(value)
})
```

AJAX and JSONP are essentially different things. **AJAX's core is getting non-page content through `XmlHttpRequest`, while JSONP's core is dynamically adding `<script>` tags to call server-provided js scripts.**

Promise version: Compared to above, difference is it doesn't directly run callback, callback only changes state, then uses to execute callback function

```js
function jsonp({ url, params, callback }) {
  return new Promise((resolve, reject) => {
    let script = document.createElement('script')
    window[callback] = function (data) {
      resolve(data)
      document.body.removeChild(script)
    }
    params = { ...params, callback } // wd=b&callback=show
    let arrs = []
    for (let key in params) {
      arrs.push(`${key}=${params[key]}`)
    }
    script.src = `${url}?${arrs.join('&')}`
    document.body.appendChild(script)
  })
}
jsonp({
  url: 'http://localhost:3000/say',
  params: { wd: 'Iloveyou' },
  callback: 'show'
}).then(data => {
  console.log(data)
})
```

Above code is equivalent to requesting data from `http://localhost:3000/say?wd=Iloveyou&callback=show`, then backend returns `show('I don't love you')`, finally runs show() function, changes Promise state to resolve, thus printing 'I don't love you'

```js
// server.js
let express = require('express')
let app = express()
app.get('/say', function(req, res) {
  let { wd, callback } = req.query
  console.log(wd) // Iloveyou
  console.log(callback) // show
  res.end(`${callback}('I don\'t love you')`)
})
app.listen(3000)
```

### CORS

CORS is actually a W3C standard, full name is `Cross-Origin Resource Sharing`.

- `CORS` requires both browser and backend support. `IE 8` and `9` need to use `XDomainRequest` to implement.
- Browser automatically handles `CORS` communication, key to implementing `CORS` communication is backend - as long as backend implements `CORS`, cross-origin is achieved.
- Server setting `Access-Control-Allow-Origin` enables `CORS`. This property indicates which domains can access resources; if set to wildcard *, means all websites can access resources.

Although setting `CORS` isn't related to frontend, when solving cross-origin problems this way, two situations occur when sending requests: simple requests and non-simple requests (also called preflight requests).

#### Simple Requests

Taking `Ajax` as example, simple requests are triggered when following conditions are met:

1. Uses one of following methods:

   * `GET`

   * `HEAD`

   * `POST`

2. `Content-Type` value limited to these three:

   * `text/plain`

   * `multipart/form-data`

   * `application/x-www-form-urlencoded`

So for simple requests, what does browser do before request is sent?

Browser automatically adds `Origin` field in request header to indicate which `origin` request comes from. After server receives request, correspondingly adds `Access-Control-Allow-Origin` field in response. If `Origin` isn't within this field's range, browser intercepts response.

Supplementary optional field settings in server:

**Access-Control-Allow-Credentials**. This field is boolean, indicates whether sending Cookie is allowed. For cross-origin requests, browser sets this field's default value to false. If need to get browser's `Cookie`, need to add this response header and set to `true`, and frontend also needs to set `withCredentials` property:

```js
let xhr = new XMLHttpRequest();
xhr.withCredentials = true;
```

Note two points:

1. For requests with credentials, server must not set `Access-Control-Allow-Origin` value to *, must be specific domain, otherwise if frontend carries `Cookie`, backend finds it's *, will return error message.

2. If request initiated with `WithCredentials` flag set to `true`, thus sending `Cookie` to server, but if server's response doesn't carry `Access-Control-Allow-Credentials: true`, browser won't return response content to request sender.

**Access-Control-Expose-Headers**. This field empowers `XMLHttpRequest` object, letting it not only get basic 6 response header fields (including `Cache-Control`, `Content-Language`, `Content-Type`, `Expires`, `Last-Modified` and `Pragma`), but also get **response header fields** declared by this field. For example, set like this:

```http
Access-Control-Expose-Headers: aaa
```

Then frontend can get value of `aaa` field through `XMLHttpRequest.getResponseHeader('aaa')`.

#### Non-Simple Requests

Non-simple requests differ in two aspects: **preflight requests** and **response fields**.

Taking PUT method as example:

```js
var url = 'http://xxx.com';
var xhr = new XMLHttpRequest();
xhr.open('PUT', url, true);
xhr.setRequestHeader('X-Custom-Header', 'xxx');
xhr.send();
```

First step, browser must first send preflight request using **OPTIONS** method to know if server allows this cross-origin request.

This preflight request's request line and body are in following format:

```http
OPTIONS / HTTP/1.1
Origin: current address
Host: xxx.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: X-Custom-Header
```

Preflight request method is `OPTIONS`, also adds `Origin` source address and `Host` target address, this is simple. Also adds two key fields:

- `Access-Control-Request-Method`, lists **which HTTP method CORS request uses**
- `Access-Control-Request-Headers`, specifies what request headers CORS request will add

Next are **response fields**, which also divide into two parts: response to **preflight request** and response to **CORS request**.

**Preflight request response**, in following format:

```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 1728000
Content-Type: text/html; charset=utf-8
Content-Encoding: gzip
Content-Length: 0
```

It has these key **response header fields**:

- `Access-Control-Allow-Origin`: Indicates allowed request origins, can fill specific origin name or `*` to allow any origin request.
- `Access-Control-Allow-Methods`: Indicates allowed request method list.
- `Access-Control-Allow-Credentials`: Already introduced in simple requests. Shows server can notify client in preflight request response whether credentials need to be carried.
- `Access-Control-Allow-Headers`: Indicates allowed request header fields
- `Access-Control-Max-Age`: Preflight request's validity period, during which no need to send another preflight request.

Third step, after preflight request response returns, if request doesn't meet response header conditions, triggers `XMLHttpRequest`'s `onerror` method, and of course real **CORS request** won't be sent.

If server allows, will send real CORS request, now it's same as **simple request** situation. Browser automatically adds `Origin` field, server response header returns **Access-Control-Allow-Origin**. Can refer to above simple request section content.

#### Node.js Configuration Methods

##### CORS Module

This method is simple and forceful, solves all request header and method setting hassles, drawback is if need to carry cookie this method obviously isn't suitable

```js
const express = require('express')
const cors = require('cors')
const app = express()
//Allow cross-origin, set *
app.use(cors())
```

##### Set Single Domain

`res.setHeader()` is Node.js built-in method, `res.header()` is alias of express framework's `res.set()` method.

These two methods are completely same, set HTTP response headers. Only difference is `res.setHeader()` only allows you to **set one single header** while `res.header()` will allow you to **set multiple headers**.

```js
const express = require('express')
const app = express()

app.use(function(req, res, next) {
  res.set({
    'Access-Control-Allow-Origin': 'http://a.com',	// or *
    'Access-Control-Allow-Methods': 'GET, POST, PUT, OPTIONS',
    'Access-Control-Allow-Headers': 'Content-Type, Authorization',
    'Access-Control-Allow-Credentials': true
  })
  // Remember next()
  return next()
})
```

##### Set Multiple Domains - Whitelist

To set multiple domains, in Node can't directly do this:

```js
res.setHeader('Access-Control-Allow-Origin', 'http://a.com, http://b.com')
```

Can only do this:

```js
const express = require('express')
const app = express()

app.use(function(req, res, next) {
  const allowOrigins = ['http://a.com', 'http://b.com', 'http://c.com']
  const origin = req.headers.Origin
  if (allowOrigins.indexOf(origin) !== -1) {
    res.header('Access-Control-Allow-Origin', origin)
  }
  res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, OPTIONS')
  res