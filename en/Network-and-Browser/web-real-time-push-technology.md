## Bidirectional Communication

HTTP protocol has a flaw: communication can only be initiated by the client. For example, if we want to know today's weather, the client must send a request to the server, and the server returns the query result.

HTTP protocol cannot achieve server-initiated push information to the client. This one-way request characteristic means that if the server has continuous state changes, it is very troublesome for the client to be informed.

**Before the WebSocket protocol, there were three ways to implement bidirectional communication: polling, long-polling, and iframe streaming**.

### Polling

Polling involves continuous connections between the client and server, with inquiries made at regular intervals. Its disadvantages are obvious: there will be many connections, one for receiving and one for sending. Moreover, **each request sent includes HTTP headers, which consumes bandwidth and CPU utilization**.

- Advantage: Simple implementation, no need for many changes
- Disadvantage: If the polling interval is too long, users cannot receive updated data in a timely manner; if the polling interval is too short, it will cause too many query requests, increasing the server's burden

``` html
<div id="clock"></div>

<script>
    let clockDiv = document.getElementById('clock');
  	// Timer, sends once per second
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

### Long Polling

Long polling is an improved version of polling. After the client sends an HTTP request to the server, it checks if there are new messages. If there are no new messages, it continues to wait. When there are new messages, it returns them to the client. This reduces network bandwidth and CPU utilization issues to some extent.

- Advantage: Optimized compared to Polling, with better timeliness
- Disadvantage: Maintaining connections consumes resources; the program times out if the server does not return valid data

``` html
<div id="clock"></div>

<script>
let clockDiv = document.getElementById('clock')
function send() {
  let xhr = new XMLHttpRequest()
  xhr.open('GET', '/clock', true)
  xhr.timeout = 2000 // Timeout in milliseconds
  xhr.onreadystatechange = function() {
    if (xhr.readyState == 4) {
      if (xhr.status == 200) {
        //If the return is successful, display the result
        clockDiv.innerHTML = xhr.responseText
      }
      send() //Send the next request regardless of success or failure
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

The iframe streaming method inserts a hidden iframe into the page, using its src attribute to create a long connection between the server and client. The server transmits data to the iframe (usually HTML, containing JavaScript responsible for inserting information, so the focus is on the server side) to update the page in real-time.

- Advantage: Messages can reach in real-time; good browser compatibility
- Disadvantage: The client only requests once, but the server continuously sends data to the client, which increases overhead for the server to maintain a long connection; IE, Chrome, and Firefox will show that loading is not complete, with icons continuously spinning

## WebSocket

### Definition

WebSocket is a completely new, independent protocol, **based on the TCP protocol**, compatible with the HTTP protocol but not integrated into it. It was designed to replace polling and Comet technologies.

Once a WebSocket communication connection is established between the web server and client, all subsequent communication relies on this dedicated protocol. During communication, they can exchange data in any format such as JSON, XML, HTML, or images. **Since it is built on the TCP protocol, the connection initiator is still the client, but once the WebSocket communication connection is established, either the server or the client can send messages directly to the other party**.

### Limitations of HTTP Compared to WebSocket

- HTTP is a half-duplex protocol, meaning that data can only flow in one direction at a time: the client sends a request to the server (one-way), and then the server responds to the request (one-way).
- The server cannot actively push data to the browser. This makes it difficult to implement some advanced features, such as chat room scenarios.

### WebSocket Features

- **Bidirectional communication**, both client and server can send data to each other at any time. Once the backend has new data, it can immediately "push" it to the client without requiring client polling, improving **"real-time communication"** efficiency
- Can send text as well as binary data

* WebSocket headers are small, reducing communication volume

Once a WebSocket connection is established, subsequent data is transmitted in the form of frame sequences. Before the client disconnects the WebSocket connection or the server terminates the connection, the client and server do not need to reinitiate connection requests. **In cases of massive concurrency and high load traffic between client and server interactions, it greatly saves network bandwidth resource consumption, has obvious performance advantages, and client sending and receiving messages are initiated on the same persistent connection, with obvious real-time advantages**.

### Implementation

Client

``` html
<div id="clock"></div>
<script>
let clockDiv = document.getElementById('clock')
let socket = new WebSocket('ws://localhost:9999')
//The callback function will be executed after the connection is successful
socket.onopen = function() {
  console.log('Client connected successfully')
  //Send a message to the server
  socket.send('hello') //The client's message content is hello
}
//Binding events is done by adding attributes
socket.onmessage = function(event) {
  clockDiv.innerHTML = event.data
  console.log('Received server response', event.data)
}
</script>
```

Server

``` js
let express = require('express')
let app = express()
app.use(express.static(__dirname))
//http server
app.listen(3000)
let WebSocketServer = require('ws').Server
//Start a websocket server using the ws module, listening on port 9999
let wsServer = new WebSocketServer({ port: 9999 })
//Listen for client connection requests. When a client connects to the server, it triggers the connection event
//socket represents a client, not shared by all clients, but each client has its own socket
wsServer.on('connection', function(socket) {
  //Each socket has a unique ID property
  console.log(socket)
  console.log('Client connected successfully')
  //Listen for messages from the other party
  socket.on('message', function(message) {
    console.log('Received client message', message)
    socket.send('Server response:' + message)
  })
})
```

## Reference Article

[Summary of Web Real-time Push Technologies](https://github.com/ljianshu/Blog/issues/58)