## 双向通信

HTTP 协议有一个缺陷：通信只能由客户端发起。举例来说，我们想了解今天的天气，只能是客户端向服务器发出请求，服务器返回查询结果。

HTTP 协议做不到服务器主动向客户端推送信息。这种单向请求的特点，注定了如果服务器有连续的状态变化，客户端要获知就非常麻烦。

**在WebSocket协议之前，有三种实现双向通信的方式：轮询（polling）、长轮询（long-polling）和iframe流（streaming）**。

### 轮询

轮询是客户端和服务器之间会一直进行连接，每隔一段时间就询问一次。其缺点也很明显：连接数会很多，一个接受，一个发送。而且**每次发送请求都会有Http的Header，会很耗流量，也会消耗CPU的利用率**。

- 优点：实现简单，无需做过多的更改
- 缺点：轮询的间隔过长，会导致用户不能及时接收到更新的数据；轮询的间隔过短，会导致查询请求过多，增加服务器端的负担

``` html
<div id="clock"></div>

<script>
    let clockDiv = document.getElementById('clock');
  	// 定时器，1秒发一次
    setInterval(function(){
        let xhr = new XMLHttpRequest;
        xhr.open('GET','/clock',true);
        xhr.onreadystatechange = function(){
            if(xhr.readyState == 4 && xhr.status == 200){
                console.log(xhr.responseText);
                clockDiv.innerHTML = xhr.responseText;
            }
        }
        xhr.send();
    },1000);
</script>
```

### 长轮询

长轮询是对轮询的改进版，客户端发送HTTP给服务器之后，看有没有新消息，如果没有新消息，就一直等待。当有新消息的时候，才会返回给客户端。在某种程度上减小了网络带宽和CPU利用率等问题。

- 优点：比 Polling 做了优化，有较好的时效性
- 缺点：保持连接会消耗资源; 服务器没有返回有效数据，程序超时。

``` html
<div id="clock"></div>

<script>
let clockDiv = document.getElementById('clock')
function send() {
  let xhr = new XMLHttpRequest()
  xhr.open('GET', '/clock', true)
  xhr.timeout = 2000 // 超时时间，单位是毫秒
  xhr.onreadystatechange = function() {
    if (xhr.readyState == 4) {
      if (xhr.status == 200) {
        //如果返回成功了，则显示结果
        clockDiv.innerHTML = xhr.responseText
      }
      send() //不管成功还是失败都会发下一次请求
    }
  }
  xhr.ontimeout = function() {
    send()
  }
  xhr.send()
}
send()
</script>
```

### iframe

iframe流方式是在页面中插入一个隐藏的iframe，利用其src属性在服务器和客户端之间创建一条长连接，服务器向iframe传输数据（通常是HTML，内有负责插入信息的javascript，所以重点在服务端），来实时更新页面。

- 优点：消息能够实时到达；浏览器兼容好
- 缺点：客户端只请求一次，然而服务端却是源源不断向客户端发送数据，这样服务器维护一个长连接会增加开销；IE、chrome、Firefox会显示加载没有完成，图标会不停旋转。

## WebSocket

### 定义

Websocket 是一个全新的、独立的协议，**基于 TCP 协议**，与 Http 协议兼容、却不会融入 Http 协议。他被设计出来的目的就是要取代轮询和 Comet 技术。

一旦Web服务器与客户端之间建立起WebSocket协议的通信连接，之后所有的通信都依靠这个专用协议进行。通信过程中可互相发送JSON、XML、HTML或图片等任意格式的数据。**由于是建立在HTTP基础上的协议，因此连接的发起方仍是客户端，而一旦确立WebSocket通信连接，不论服务器还是客户端，任意一方都可直接向对方发送报文**。

### HTTP相较WebSocket的局限

- HTTP是半双工协议，也就是说，在同一时刻数据只能单向流动，客户端向服务器发送请求(单向的)，然后服务器响应请求(单向的)。
- 服务器不能主动推送数据给浏览器。这就会导致一些高级功能难以实现，诸如聊天室场景就没法实现。

### WebSocket的特点

- **双向通信**，客户端和服务器都可以随时向对方发送数据。一旦后台有新的数据，就可以立即“推送”给客户端，不需要客户端轮询，**“实时通信”**的效率也就提高了。
- 可以发送文本，也可以发送二进制数据

* WebSocket的首部信息很小，减少通信量

一旦WebSocket连接建立后，后续数据都以帧序列的形式传输。在客户端断开WebSocket连接或服务端端断掉连接前，不需要客户端和服务端重新发起连接请求。**在海量并发和客户端与服务器交互负载流量大的情况下，极大的节省了网络带宽资源的消耗，有明显的性能优势，且客户端发送和接受消息是在同一个持久连接上发起，实时性优势明显**。

### 实现

客户端

``` html
<div id="clock"></div>
<script>
let clockDiv = document.getElementById('clock')
let socket = new WebSocket('ws://localhost:9999')
//当连接成功之后就会执行回调函数
socket.onopen = function() {
  console.log('客户端连接成功')
  //再向服务 器发送一个消息
  socket.send('hello') //客户端发的消息内容 为hello
}
//绑定事件是用加属性的方式
socket.onmessage = function(event) {
  clockDiv.innerHTML = event.data
  console.log('收到服务器端的响应', event.data)
}
</script>
```

服务端

``` js
let express = require('express')
let app = express()
app.use(express.static(__dirname))
//http服务器
app.listen(3000)
let WebSocketServer = require('ws').Server
//用ws模块启动一个websocket服务器,监听了9999端口
let wsServer = new WebSocketServer({ port: 9999 })
//监听客户端的连接请求  当客户端连接服务器的时候，就会触发connection事件
//socket代表一个客户端,不是所有客户端共享的，而是每个客户端都有一个socket
wsServer.on('connection', function(socket) {
  //每一个socket都有一个唯一的ID属性
  console.log(socket)
  console.log('客户端连接成功')
  //监听对方发过来的消息
  socket.on('message', function(message) {
    console.log('接收到客户端的消息', message)
    socket.send('服务器回应:' + message)
  })
})
```

## 参考文章

[Web 实时推送技术的总结](https://github.com/ljianshu/Blog/issues/58)