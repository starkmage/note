## XSS

`XSS` stands for `Cross Site Scripting`, named this way to distinguish it from `CSS`.

Typically, there are three ways to implement an `XSS` attack—**Stored XSS**, **Reflected XSS**, and **DOM-based XSS**.

### Attack Principles

#### 1. Stored XSS

As the name suggests, Stored XSS stores malicious scripts. Stored XSS saves scripts to the server's database, which are then executed on the client side to achieve the attack effect.

A common scenario is submitting a script code in a comment section. If the front and back ends don't properly escape the content, the comment is stored in the database and `directly executed` during page rendering.

#### 2. Reflected XSS

`Reflected XSS` refers to malicious scripts that are part of a **network request**.

For example, if I input:

```html
http://sanyuan.com?q=<script>alert("You're done for")</script>
```

The server will receive the `q` parameter and return the content to the browser. The browser parses this content as part of the HTML, discovers it's a script, and executes it directly, resulting in an attack.

It's called `reflected` because the malicious script is passed as a parameter in a network request, **goes through the server, and then is reflected back into the HTML document for execution.**

The difference between Reflected XSS and Stored XSS is: Stored XSS has malicious code stored in the database, while Reflected XSS has malicious code in the URL.

#### 3. DOM-based XSS

The attack steps for DOM-based XSS are:

1. The attacker constructs a special URL containing malicious code.
2. The user opens the URL with the malicious code.
3. After receiving the response, the user's browser parses and executes it. Frontend JavaScript extracts and executes the malicious code from the URL.
4. The malicious code steals user data and sends it to the attacker's website, or impersonates the user's behavior, calling the target website's interface to perform operations specified by the attacker.

The difference between DOM-based XSS and the other two types: In DOM-based XSS attacks, extracting and executing malicious code is done by the browser, making it a security vulnerability in frontend JavaScript itself, while the other two types of XSS are server-side security vulnerabilities.

### Preventive Measures

#### 1. Input Filtering

Input-side filtering can solve specific XSS problems in certain situations, but it introduces significant uncertainty and encoding issues. This method should be avoided when defending against XSS attacks.

Since input filtering is not completely reliable, we need to prevent XSS by "preventing browsers from executing malicious code".

#### 2. HTML Escaping

HTML escaping is very complex and requires different escaping rules in different situations. If incorrect escaping rules are adopted, XSS vulnerabilities may be introduced.

You should avoid writing your own escaping library and instead use mature, industry-standard escaping libraries.

#### 3. Preventing DOM-based XSS Attacks

DOM-based XSS attacks essentially occur when website frontend JavaScript code is not rigorous enough and executes untrusted data as code.

Be especially careful when using `.innerHTML`, `.outerHTML`, `document.write()`. Don't insert untrusted data as HTML into the page; instead, try to use `.textContent`, `.setAttribute()`, etc.

#### 4. CSP

CSP, or Content Security Policy in the browser, has the core idea that the server decides which resources the browser loads. Specifically, it can accomplish the following functions:

1. Restrict resource loading from other domains;
2. Prohibit data submission to other domains;
3. Provide a reporting mechanism to help us discover XSS attacks promptly.

CSP can be enabled in two ways:

1. Setting `Content-Security-Policy` in the `HTTP Header`
2. Using a `meta` tag: `<meta http-equiv="Content-Security-Policy">`

#### 5. Using HttpOnly

Many `XSS` attack scripts are used to steal `Cookies`. By setting the `HttpOnly` attribute for `Cookies`, `JavaScript` cannot read Cookie values through `document.cookie`. This is also an effective way to prevent `XSS` attacks.

Set it in the server's response header.

``` js
res.setHeader('Set-Cookie', 'test = zsjjj; HttpOnly')
```

If you need to set multiple cookies, some with `HttpOnly` and some without, the code is:

```js
response.setHeader('Set-Cookie', ['foo=bar; HttpOnly', 'x=42; HttpOnly', 'y=88']);
```

https://tech.meituan.com/2018/09/27/fe-security.html

## CSRF

CSRF (Cross-site request forgery) refers to hackers inducing users to click links, open the hacker's website, and then the hacker exploits the user's **current login status** to initiate cross-site requests. It exploits a vulnerability in web login authentication: **Simple identity authentication can only ensure that requests come from the user's browser, but cannot identify whether the request was voluntarily sent by the user.**

### Attack Principles

#### 1. Automatic GET Requests

The hacker's webpage might contain code like this:

```html
<img src="https://xxx.com/info?user=hhh&count=100"></img>
```

After entering the page, a `get` request is automatically sent. **It's worth noting that this request will automatically include Cookie information about xxx.com (assuming you've already logged in to xxx.com).**

If the server doesn't have appropriate verification mechanisms, it might consider the requester to be a normal user because it carries the corresponding `Cookie`, and then perform various operations, which could include fund transfers and other malicious operations.

#### 2. Automatic POST Requests

The hacker might fill out a form and write an auto-submit script:

```html
<form id='hacker-form' action="https://xxx.com/info" method="POST">
  <input type="hidden" name="user" value="hhh" />
  <input type="hidden" name="count" value="100" />
</form>
<script>document.getElementById('hacker-form').submit();</script>
```

This will also carry the user's `Cookie` information, making the server believe it's a normal user operation, enabling various malicious operations.

POST requests initiated by forms are not restricted by CORS.

#### 3. Inducing Clicks to Send GET Requests

On the hacker's website, there might be a link designed to entice you to click:

```html
<a href="https://xxx/info?user=hhh&count=100" taget="_blank">Click to enter the world of immortal cultivation</a>
```

After clicking, a `get` request is automatically sent, and the rest follows the same principle as the `Automatic GET Requests` section.

<img src="https://cdn.jsdelivr.net/gh/starkmage/ImgHosting/starkmage-picgo/20200907194157.png" style="zoom:33%;" />

This is the principle of `CSRF` attacks. Compared to `XSS` attacks, `CSRF` attacks don't need to inject malicious code into the user's current page's HTML document, but instead redirect to a new page, exploiting the server's **verification vulnerabilities** and the user's **previous login status** to simulate user operations.

### About Carrying Cookies

`CSRF` cannot obtain `Cookies`, but it can use them through browser features. For example, a `Cookie` is like an ID card in a box. When the hacker's website initiates a request, it brings this box, but it doesn't know what the ID number inside the box is. This entire process occurs in the browser.

Detailed explanation:

From cross-origin knowledge, we understand that browsers have same-origin restrictions for `Cookies`. That is, for websites with different origins from the `Cookie`, the browser won't allow that website to obtain this `Cookie`. So why do `CSRF` attacks still succeed? This is related to how browsers use `Cookies`.

1. Except for cross-origin `XHR` requests, browsers automatically include qualifying `Cookies` when initiating requests (domain, validity period, path, `SameSite` attribute);
2. The attack methods listed above—`a` tags, `img` `src`, and form submissions—can all carry cross-origin `Cookies` without same-origin policy restrictions. This is determined by actual needs, such as commonly used CDNs;
3. In other words, when a browser sends a request to a domain, the request automatically carries all `Cookies` under that domain. Carrying and obtaining are not the same thing.

### Preventive Measures

#### 1. Using the SameSite Attribute of Cookies

An important link in `CSRF` attacks is the automatic sending of `Cookies` from the target site, which then simulate the user's identity. Therefore, focusing on `Cookies` is the best choice for prevention.

Fortunately, there is a key field in `Cookies` that can place some restrictions on the inclusion of `Cookies` in requests, and this field is `SameSite`.

`SameSite` can be set to three values: `Strict`, `Lax`, and `None`.

* In `Strict` mode, browsers completely prohibit third-party requests from carrying Cookies. For example, requests to the `A.com` website can only carry `Cookies` when requested from the `A.com` domain, not from other websites;
* In `Lax` mode, it's a bit more relaxed, but Cookies can only be carried when submitting forms with the `get` method or sending `get` requests with an `a` tag, not in other situations;
* In `None` mode, which is the default mode, requests automatically carry `Cookies`.

https://www.ruanyifeng.com/blog/2019/09/cookie-samesite.html

#### 2. CSRF Token

**Using CSRF tokens is not recommended because they are most suitable for old applications that use form submissions.**

**When rendering pages using some ancient template engines (such as JSP or Thymeleaf), you can generate a token for each request in a hidden field of the form, and then submit the form with the content type "application/x-www-form-urlencoded". This way, each call to the backend requires first sending a request to obtain a token, sometimes doubling your requests to the backend.**

**However, in modern applications, we don't render forms from server-side template engines. We must first request a token, then store it in localStorage instead of a hidden form field, and then use client-side tools like fetch or axios to convert the form to JSON and submit the request. At this point, the token is in the header, not a form field. For modern applications without form submissions, other simpler solutions can be used. Don't blindly use CSRF tokens without understanding them.**

The CSRF Token protection strategy is divided into three steps:

**1. Output the CSRF Token to the page**

First, when a user opens a page, the server needs to generate a Token for this user. Obviously, the Token can no longer be placed in a Cookie during submission, otherwise it would be misused by attackers again. Therefore, for safety, it's best to store the Token in the server's Session.

Then, each time the page loads, use JS to traverse the entire DOM tree and add the Token to all a and form tags in the DOM. This can solve most requests, but for HTML code dynamically generated after the page loads, this method is ineffective, and programmers need to manually add the Token when coding.

**2. Requests submitted from the page carry this Token**

For GET requests, the Token will be appended to the request address, so the URL becomes [http://url?csrftoken=tokenvalue.](http://url/?csrftoken=tokenvalue.) For POST requests, add the following at the end of the form:

```html
<input type="hidden" name="csrftoken" value="tokenvalue"/>
```

This adds the Token as a parameter to the request.

**3. The server verifies whether the Token is correct.**

The implementation of this method is relatively complex, requiring writing a Token into every page (the frontend cannot use purely static pages), having every Form and Ajax request carry this Token, and having the backend verify every interface, ensuring that the page Token and request Token are consistent.

This makes it impossible for this protection strategy to be uniformly intercepted and processed through general interception, but requires adding corresponding output and verification to each page and interface. This method involves a huge amount of work and may have omissions.