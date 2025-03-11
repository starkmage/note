## XMLHTTPRequest Object

In modern browsers, the initial way to exchange data with servers was through the `XMLHttpRequest` object. It can send and receive data in formats such as JSON, XML, HTML, and plain text.

**It brought us many benefits**:

1. Updating web pages without reloading the entire page
2. Requesting/receiving data from the server after the page has loaded
3. Sending data to the server in the background

**However, it also has some drawbacks**:

1. It's relatively cumbersome to use, requiring many settings to be configured
2. Early IE browsers had their own implementation, necessitating compatibility code

The meanings of the 5 readyState values:

| State Value | Description |
| ------ | ------ |
| 0 | Initialization state. The XMLHttpRequest object has been created or reset by the abort() method. |
| 1 | The open() method has been called, but the send() method has not been called. The request has not been sent yet. |
| 2 | The send() method has been called, and the HTTP request has been sent to the web server. No response has been received yet. |
| 3 | All response headers have been received. The response body has begun to be received but is not complete. |
| 4 | The HTTP response has been completely received. |

In my handwritten code, I've also encapsulated Ajax using both native JavaScript and Promise, which you can refer to.

## jQuery's Ajax

jQuery's Ajax is an encapsulation of the `XMLHttpRequest` object, compatible with various browsers.

``` js
$.ajax({
  type: 'POST',
  url: url, 
  data: data,
  dataType: dataType,
  success: function () {},
  error: function () {}
})
```

**Advantages**:

1. Encapsulation of the native `XHR`, with compatibility handling, simplifying its use
2. Added support for `JSONP`, which can handle some cross-origin requests simply

**Disadvantages**:

1. If there are multiple requests with dependencies, it can easily form a callback hell
2. It was designed for MVC programming, which doesn't align with the current front-end MVVM trend
3. ajax is a method in jQuery. It's unreasonable to import the entire jQuery just to use ajax

## axios

This is good; it's a `promise`-based `HTTP` library that can be used in both browsers and `node.js`. It's essentially an encapsulation of the native `XMLHttpRequest`, but it's a Promise implementation version that conforms to the latest ES specifications.

``` js
axios({
    method: 'post',
    url: '/user/12345',
    data: {
      firstName: 'z',
      lastName: 's'
    }
  })
  .then(res => console.log(res))
  .catch(err => console.log(err))
```

**Advantages**:

1. Creates `XMLHttpRequests` from the browser
2. Creates `http` requests from `node.js`
3. Supports the `Promise` API
4. Intercepts requests and responses
5. Transforms request and response data
6. Cancels requests
7. Automatically transforms `JSON` data
8. Client-side support for defending against `XSRF`

**Disadvantages**:

1. Only supports modern browsers

**What's the principle behind it?**

**There are only two core points:**

**1. The request method,** all external Axios methods are actually calling this one method

**2. The method internally creates a Promise chain,** common functionalities like interceptors, data modifiers, and HTTP requests are executed step by step in this Promise chain. The **request** method returns the Promise chain. What we use is this returned Promise, and the execution result is in the state value of this Promise.

Concurrent requests:

Use axios.all() and axios.spread() together to send multiple requests simultaneously

Reference articles:

[Deep Analysis of Axios Basic Principles](https://juejin.im/post/6844904199302430733#heading-0)

[Understanding the Execution Principle of axios!](https://juejin.im/post/6844903685068161038#heading-1)

[axios.all() for Solving Concurrent Requests](https://segmentfault.com/a/1190000019882188)

## fetch

The `Fetch API` provides a JavaScript interface for accessing and manipulating parts of the HTTP pipeline, such as requests and responses. It also provides a global `fetch()` method that offers a simple, logical way to asynchronously fetch resources across the network.
`fetch` is a low-level API, replacing `XHR`, that can easily handle various formats, including non-text formats. It can be easily used by other technologies, such as `Service Workers`. However, to use `fetch` well, some encapsulation is needed.

**fetch is not a further encapsulation of ajax, but native js, and does not use the XMLHttpRequest object**.

``` js
fetch('http://example.com/movies.json')
  .then(function(response) {
    return response.json();
  })
  .then(function(myJson) {
    console.log(myJson);
  });
```

**Advantage: Cross-origin handling**
In the configuration, adding `mode: 'no-cors'` allows for cross-origin requests

```js
fetch('/users.json', {
    method: 'post', 
    mode: 'no-cors',
    data: {}
}).then(function() { /* handle response */ });
```

**Current issues with `fetch`**:

1. `fetch` only reports errors for network requests; it treats `400`, `500` as successful requests, requiring encapsulation to handle. Only when network errors prevent the request from completing will fetch be rejected
2. `fetch` doesn't carry `cookies` by default, requiring the configuration: fetch(url, {credentials: 'include'})
3. `fetch` doesn't support `abort` (termination) or timeout control. Using `setTimeout` and `Promise.reject` for timeout control doesn't stop the request process from continuing in the background, wasting bandwidth
4. `fetch` has no native way to monitor the progress of a request, while `XHR` can

**The `fetch` specification differs from `jQuery.ajax()` in two main ways, remember:**

1. When receiving an `HTTP status code` that represents an error, the Promise returned from `fetch()` **will not be marked as reject**, even if the HTTP response status code is `404` or `500`. Instead, it will mark the `Promise state` as `resolve` (but will set the `ok` property of the resolved return value to `false`), and will only be marked as `reject` when there's a network failure or the request is blocked

2. By default, `fetch` **will not send or receive any cookies from the server**, which can lead to unauthenticated requests if the site relies on user `sessions` (to send `cookies`, the `credentials` option must be set)