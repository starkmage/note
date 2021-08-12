## XSS

`XSS` 全称是 `Cross Site Scripting`(即`跨站脚本`)，为了和 `CSS` 区分，故叫它`XSS`。

通常情况，`XSS` 攻击的实现有三种方式——**存储型**、**反射型**和**文档型**。

### 攻击原理

#### 1. 存储型

存储型，顾名思义就是将恶意脚本存储了起来。存储型的 `XSS` 将脚本存储到了服务端的数据库，然后在客户端执行这些脚本，从而达到攻击的效果。

常见的场景是留言评论区提交一段脚本代码，如果前后端没有做好转义的工作，那评论内容存到了数据库，在页面渲染过程中`直接执行`。

#### 2. 反射型

`反射型XSS`指的是恶意脚本作为**网络请求的一部分**。

比如我输入:

```html
http://sanyuan.com?q=<script>alert("你完蛋了")</script>
```

这样，在服务器端会拿到`q`参数,然后将内容返回给浏览器端，浏览器将这些内容作为HTML的一部分解析，发现是一个脚本，直接执行，这样就被攻击了。

之所以叫它`反射型`, 是因为恶意脚本是通过作为网络请求的参数，**经过服务器，然后再反射到HTML文档中，执行解析。**

反射型 XSS 跟存储型 XSS 的区别是：存储型 XSS 的恶意代码存在数据库里，反射型 XSS 的恶意代码存在 URL 里。

#### 3. DOM型

DOM 型 XSS 的攻击步骤：

1. 攻击者构造出特殊的 URL，其中包含恶意代码。
2. 用户打开带有恶意代码的 URL。
3. 用户浏览器接收到响应后解析执行，前端 JavaScript 取出 URL 中的恶意代码并执行。
4. 恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。

DOM 型 XSS 跟前两种 XSS 的区别：DOM 型 XSS 攻击中，取出和执行恶意代码由浏览器端完成，属于前端 JavaScript 自身的安全漏洞，而其他两种 XSS 都属于服务端的安全漏洞。

### 防范措施

#### 1. 输入过滤

输入侧过滤能够在某些情况下解决特定的 XSS 问题，但会引入很大的不确定性和乱码问题。在防范 XSS 攻击时应避免此类方法。

既然输入过滤并非完全可靠，我们就要通过“防止浏览器执行恶意代码”来防范 XSS。

#### 2. 转义HTML

HTML 转义是非常复杂的，在不同的情况下要采用不同的转义规则。如果采用了错误的转义规则，很有可能会埋下 XSS 隐患。

应当尽量避免自己写转义库，而应当采用成熟的、业界通用的转义库。

#### 3. 预防 DOM 型 XSS 攻击

DOM 型 XSS 攻击，实际上就是网站前端 JavaScript 代码本身不够严谨，把不可信的数据当作代码执行了。

在使用 `.innerHTML`、`.outerHTML`、`document.write()` 时要特别小心，不要把不可信的数据作为 HTML 插到页面上，而应尽量使用 `.textContent`、`.setAttribute()` 等。

#### 4. CSP

CSP，即浏览器中的内容安全策略，它的核心思想就是服务器决定浏览器加载哪些资源，具体来说可以完成以下功能：

1. 限制其它域下的资源加载；
2. 禁止向其它域提交数据；
3. 提供上报机制，能帮助我们及时发现 `XSS` 攻击。

通过两种方式来开启 CSP：

1. 设置 `HTTP Header` 中的 `Content-Security-Policy`
2. 设置 `meta` 标签的方式 `<meta http-equiv="Content-Security-Policy">`

#### 5. 利用 HttpOnly

很多 `XSS` 攻击脚本都是用来窃取 `Cookie`, 而设置 `Cookie` 的 `HttpOnly` 属性后，`JavaScript` 便无法通过 `document.cookie`读取 `Cookie` 的值。这样也能很好的防范 `XSS` 攻击。

在服务端的响应头中设置。

``` js
res.setHeader('Set-Cookie', 'test = zsjjj; HttpOnly')
```

如果要设置多个cookie，有些带有`HttpOnly`，有些不带。代码是：

```js
response.setHeader('Set-Cookie', ['foo=bar; HttpOnly', 'x=42; HttpOnly', 'y=88']);
```

https://tech.meituan.com/2018/09/27/fe-security.html

## CSRF

CSRF(Cross-site request forgery)，即**跨站请求伪造**，指的是黑客诱导用户点击链接，打开黑客的网站，然后黑客利用用户**目前的登录状态**发起跨站请求。利用了web登录身份认证的一个漏洞：**简单的身份认证只能保证请求来自用户的浏览器，但不能识别请求是用户自愿发出的。**

### 攻击原理

#### 1. 自动发 GET 请求

黑客网页里面可能有一段这样的代码：

```html
<img src="https://xxx.com/info?user=hhh&count=100"></img>
```

进入页面后自动发送 `get` 请求，**值得注意的是，这个请求会自动带上关于 xxx.com 的 `Cookie` 信息(这里是假定你已经在 xxx.com 中登录过)。**

假如服务器端没有相应的验证机制，它可能认为发请求的是一个正常的用户，因为携带了相应的 `Cookie`，然后进行相应的各种操作，可以是转账汇款以及其他的恶意操作。

#### 2. 自动发 POST 请求

黑客可能自己填了一个表单，写了一段自动提交的脚本：

```html
<form id='hacker-form' action="https://xxx.com/info" method="POST">
  <input type="hidden" name="user" value="hhh" />
  <input type="hidden" name="count" value="100" />
</form>
<script>document.getElementById('hacker-form').submit();</script>
```

同样也会携带相应的用户 `Cookie` 信息，让服务器误以为是一个正常的用户在操作，让各种恶意的操作变为可能。

form 发起的 POST 请求并不受到 CORS 的限制

#### 3. 诱导点击发送 GET 请求

在黑客的网站上，可能会放上一个链接，驱使你来点击：

```html
<a href="https://xxx/info?user=hhh&count=100" taget="_blank">点击进入修仙世界</a>
```

点击后，自动发送 `get` 请求，接下来和`自动发 GET 请求`部分同理。

<img src="https://cdn.jsdelivr.net/gh/starkmage/ImgHosting/starkmage-picgo/20200907194157.png" style="zoom:33%;" />

这就是`CSRF`攻击的原理，和`XSS`攻击对比，`CSRF` 攻击并不需要将恶意代码注入用户当前页面的html文档中，而是跳转到新的页面，利用服务器的**验证漏洞**和**用户之前的登录状态**来模拟用户进行操作。

### 关于携带Cookie

`CSRF` 是不能获取到 `Cookie `的，但是可以利用浏览器的特性去使用，比如说 `Cookie` 是一张放在盒子里的身份证，黑客网站在发起请求的时候会带上这个盒子，但是它是不知道盒子里的身份证号码是多少的，这整个过程都是在浏览器进行的。

详细解释：

从跨域知识中，我们了解到浏览器对于 `Cookie` 是存在同源限制的，也就是与 `Cookie`处于不同源的网站，浏览器是不会让该网站获取到这个 `Cookie`，那为什么`CSRF` 攻击还会成功呢？其实这个与浏览器使用 `Cookie` 的方式有关。

1. 除了跨域 `XHR` 请求情况下，浏览器在发起请求的时候会把符合要求的 `Cookie` 自动带上(域名，有效期，路径，`SameSite` 属性)；
2. 也就是上面列举的几种攻击方式，`a` 标签、`img` 的 `src` 以及表单提交，都是可以携带跨域的`Cookie`，没有同源策略的限制，这也是实际需求决定的，比如常用的CDN；
3. 也就是说，浏览器向某个域名发送请求时，其请求都会自动携带该域名下的所有 `Cookie`，携带和获取不是一回事。

### 防范措施

#### 1. 利用Cookie的SameSite属性

`CSRF` 攻击中重要的一环就是自动发送目标站点下的 `Cookie`，然后就是这一份 `Cookie` 模拟了用户的身份，因此在`Cookie`上面下文章是防范的不二之选。

恰好，在 `Cookie` 当中有一个关键的字段，可以对请求中 `Cookie` 的携带作一些限制，这个字段就是`SameSite`。

`SameSite`可以设置为三个值，`Strict`、`Lax`和`None`。

* 在`Strict`模式下，浏览器完全禁止第三方请求携带Cookie。比如请求`A.com`网站只能在 `A.com`域名当中请求才能携带 `Cookie`，在其他网站请求都不能；

* 在`Lax`模式，就宽松一点了，但是只能在 `get` 方法提交表单或者 `a` 标签发送 `get` 请求的情况下可以携带 `Cookie`，其他情况均不能；

* 在`None`模式下，也就是默认模式，请求会自动携带上 `Cookie`。

#### 2.csrf token

**不建议使用CSRF token，因为它最适合使用表单提交的旧应用程序。** 

**当使用某些古老的模板引擎（例如JSP或Thymeleaf）渲染页面时，可以在表单的隐藏字段中生成每个请求令牌，然后提交内容类型为“ application/x-www-form-urlencoded”的表单。这样每次调用后端都要先发个请求获取令牌，有时候您对后端的请求会加倍。**

**但是，在现代应用程序中，我们不从服务器端的模板引擎渲染表单，我们必须首先请求获得令牌，然后将其存到localStorage里面，而不是隐藏的表单字段，然后使用fetch或axios之类的客户端把form转为JSON再提交请求，令牌这时候是在header上的，而不是一个form field。对于没有表单提交的现代应用程序，可以使用其他更简单的解决方案，不要再不明就里的用CSRF token了。**

CSRF Token的防护策略分为三个步骤：

**1. 将CSRF Token输出到页面中**

首先，用户打开页面的时候，服务器需要给这个用户生成一个Token，显然在提交时Token不能再放在Cookie中了，否则又会被攻击者冒用。因此，为了安全起见Token最好还是存在服务器的Session中。

之后在每次页面加载时，使用JS遍历整个DOM树，对于DOM中所有的a和form标签后加入Token。这样可以解决大部分的请求，但是对于在页面加载之后动态生成的HTML代码，这种方法就没有作用，还需要程序员在编码时手动添加Token。

**2. 页面提交的请求携带这个Token**

对于GET请求，Token将附在请求地址之后，这样URL 就变成 [http://url?csrftoken=tokenvalue。](http://url/?csrftoken=tokenvalue。) 而对于 POST 请求来说，要在 form 的最后加上：

```html
<input type=”hidden” name=”csrftoken” value=”tokenvalue”/>
```

这样，就把Token以参数的形式加入请求了。

**3. 服务器验证Token是否正确**。

此方法的实现比较复杂，需要给每一个页面都写入Token（前端无法使用纯静态页面），每一个Form及Ajax请求都携带这个Token，后端对每一个接口都进行校验，并保证页面Token及请求Token一致。

这就使得这个防护策略不能在通用的拦截上统一拦截处理，而需要每一个页面和接口都添加对应的输出和校验。这种方法工作量巨大，且有可能遗漏。

#### 4. 双重cookie

利用CSRF攻击不能获取到用户Cookie的特点，我们可以要求Ajax和表单请求携带一个Cookie中的值。

双重Cookie采用以下流程：

- 在用户访问网站页面时，向请求域名注入一个Cookie，内容为随机字符串（例如`csrfcookie=v8g9e4ksfhw`）。
- 在前端向后端发起请求时，取出Cookie，并添加到URL的参数中（接上例`POST https://www.a.com/comment?csrfcookie=v8g9e4ksfhw`）。
- 后端接口验证Cookie中的字段与URL参数中的字段是否一致，不一致则拒绝。

#### 5. 验证来源站点

这就需要要用到请求头中的两个字段: `Origin` 和 `Referer`。

其中，`Origin` 只包含域名信息，而 `Referer` 包含了具体的 `URL` 路径。

当然，这两者都是可以伪造的，通过 `Ajax` 中自定义请求头即可，安全性略差。

https://tech.meituan.com/2018/10/11/fe-security-csrf.html

https://danielw.cn/web-security-xss-csrf-cn