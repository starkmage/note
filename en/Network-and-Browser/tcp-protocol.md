## Five-Layer Network Model

|    Name    |                             Function                             | Transport Unit |          Main Protocols          |
| :--------: | :----------------------------------------------------------: | :------: | :------------------------: |
|   Application Layer   |                    Directly provides services to user processes                    |   Message   | HTTP, FTP, SMTP, POP3, DNS |
|   Transport Layer   | Implements reliable end-to-end communication between two user processes, handles transmission issues such as data packet errors | Segment  |          TCP, UDP          |
|   Network Layer   |                       Performs logical address routing                       |  Datagram  |       IP, ICMP, ARP        |
| Data Link Layer | Physical addressing (MAC address), unique identity of network devices, establishes logical connections and performs hardware address routing |    Frame    |            PPP             |
|   Physical Layer   |              Transmits bit streams, but does not refer to specific transmission media              |  Bit Stream  |                            |

## OSI Seven-Layer Model

Presentation Layer: Provides formatted representation and data conversion services, data compression and decompression, encryption and decryption, etc.

Session Layer: Does not participate in specific transmission, it provides mechanisms for establishing and maintaining communication between applications, including access verification and session management. SSL protocol and TLS protocol are at this layer.

## Differences Between TCP and UDP

1. **TCP is connection-oriented**. The so-called connection refers to the connection between client and server. Before both parties communicate with each other, TCP requires a three-way handshake to establish a connection, **while UDP does not have a corresponding connection establishment process**;

2. **TCP is reliable, UDP is stateless and uncontrollable**;

   TCP precisely records which data has been sent, which data has been received by the other party, which has not been received, and ensures that data packets arrive in sequence, allowing no errors. This is **stateful**.

   When packet loss is detected or the network environment is poor, TCP will adjust its behavior according to the specific situation, controlling its sending speed or retransmitting. This is **controllable**.

3. **TCP is byte-stream oriented, while UDP is message oriented**;

4. TCP header overhead is large (minimum 20 bytes, maximum 60 bytes), UDP header overhead is small, only 8 bytes

## What is the Relationship Between TCP's Byte-Stream Orientation and Segments?

The key to this question is that TCP has a buffer, while UDP, which is message-oriented, does not have a buffer.

When TCP sends messages, it writes application layer data into the TCP buffer, and then the TCP protocol controls the sending of this data. The sending state is in the form of a byte stream, which has nothing to do with the length of the message written down from the application layer, so it is called a stream.

In contrast, UDP has no buffer. The message data written by the application layer will be directly added with a header and passed to the network layer, which is responsible for fragmentation, so it is message-oriented.

## TCP Three-Way Handshake

### Handshake Process

<img src="http://47.98.159.95/my_blog/tcp/001.jpg" style="zoom: 80%;" />

1. The client sends a **connection request segment**, with no application layer data

   SYN = 1, sequence number seq = x (random)

2. The server **allocates buffers and variables** for this TCP connection and returns an **acknowledgment segment** to the client, allowing the connection, with no application layer data

   SYN = 1, ACK = 1, seq = y (random), acknowledgment number ack = x + 1

3. The client **allocates buffers and variables** for this TCP connection and returns an acknowledgment to the server, **can carry data**

   ACK = 1, seq = x + 1, ack = y + 1

As you can see, `SYN` consumes a sequence number, and the next time the corresponding `ack` sequence number is sent, it needs to be incremented by 1. Why? Just remember one rule:

> Anything that needs to be acknowledged by the other end must consume a sequence number in the TCP message.

`SYN` needs to be acknowledged by the other end, while `ACK` does not, so `SYN` consumes a sequence number while `ACK` does not.

### Why Three Times, When Two Seems Enough

Root cause: Unable to confirm the client's receiving capability. Let's look at an example:

The client sends a connection request A, but due to network issues, it times out. At this point, TCP will start the timeout retransmission mechanism and send another connection request B. This request successfully reaches the server, the server responds and establishes the request, then receives data and releases the connection.

Suppose that connection request A finally reaches the server after both ends have closed the connection. At this point, the server would think that the client needs to establish a TCP connection again, so it would respond to the request and enter the **ESTABLISHED** state. But the client is actually in the **CLOSED** state, which would cause the server to wait indefinitely, wasting resources.

### Why Not Four Times

The purpose of the three-way handshake is to confirm both parties' ability to send and receive. Can we do a four-way handshake?

Of course we can, even 100 times would work. But to solve the problem, three times is enough; more would not add much value.

![](http://img.stark.pub/20210321202312.png)

### SYN Flood Attack Principle

SYN Flood is a typical DOS/DDOS attack. Its attack principle is very simple: the client forges a large number of non-existent IP addresses in a short period of time and frantically sends `SYN` to the server. For the server, this produces two dangerous consequences:

1. Processing a large number of `SYN` packets and returning corresponding `ACK` will inevitably result in many connections in the `SYN_RCVD` state, filling up the entire **half-connection queue** and making it impossible to process normal requests.
2. Since the IPs don't exist, the server won't receive `ACK` from the client for a long time, causing the server to continuously retransmit data until the server's resources are exhausted.

### How to Counter SYN Flood Attacks?

- Increase SYN connections, that is, increase the capacity of the half-connection queue.
- Reduce the number of SYN + ACK retries to avoid a large number of timeout retransmissions.
- Use SYN Cookie technology. When the server receives a `SYN`, it doesn't immediately allocate connection resources, but calculates a Cookie based on this `SYN` and sends it back to the client along with the second handshake. When the client replies with `ACK`, it brings this `Cookie` value, and the server only allocates connection resources after verifying that the Cookie is valid.

## TCP Four-Way Handshake (Connection Termination)

### Termination Process

![](http://47.98.159.95/my_blog/tcp/002.jpg)

1. The client sends a **connection release segment**, stops sending data, and actively closes the TCP connection

   FIN = 1, seq = p

2. The server sends back an acknowledgment segment, and the connection from client to server is released—half-closed state, because TCP connections are bidirectional, so the server can still send data to the client

   ACK = 1, ack = p + 1

3. After the server finishes sending data, it sends a connection release segment and actively closes the TCP connection

   FIN = 1, ACK = 1, seq = q, ack = p + 1

4. The client sends back an acknowledgment segment, and after waiting for the time-wait timer set to 2MSL (MSL is the maximum segment lifetime), the connection is completely closed

   ACK = 1, seq = p +1, ack = q + 1

### Significance of Waiting for 2MSL

What would happen if we didn't wait?

To ensure that the server can receive the client's final acknowledgment. If the client enters the CLOSED state immediately after sending the acknowledgment, and if the acknowledgment never reaches the server due to network issues, it would cause the server to not close properly.

So, wouldn't 1 MSL be enough? Why wait for 2 MSL?

- 1 MSL ensures that the client's final ACK packet in the four-way handshake can eventually reach the server
- 1 MSL ensures that the FIN packet retransmitted by the server when it doesn't receive the final ACK can reach the client

### Why Four-Way Handshake Instead of Three

Because when the server receives a `FIN`, it often won't immediately return a `FIN`; it must wait until all of the server's packets have been sent before sending a `FIN`. Therefore, it first sends an `ACK` to indicate that it has received the client's `FIN`, and then sends a `FIN` after a delay. This results in a four-way handshake.

What would be the problem with a three-way handshake?

It would mean that the server combines the sending of `ACK` and `FIN` into one handshake. In this case, a long delay might cause the client to mistakenly believe that the `FIN` hasn't reached the client, causing the client to continuously resend `FIN`.

## TCP Flow Control

http://47.98.159.95/my_blog/tcp/009.html

For the sender and receiver, TCP needs to put the data to be sent into the **send buffer** and the received data into the **receive buffer**.

What flow control needs to do is to control the sender's transmission through the size of the receive buffer.

TCP uses the **sliding window** mechanism to implement flow control.

## TCP Congestion Control

http://47.98.159.95/my_blog/tcp/010.html

```text
Send window size = min(rwnd, cwnd)
// rwnd is the receive window
// cwnd is the congestion window
```

Classic algorithms for TCP congestion control: **Slow Start**, **Congestion Avoidance**, **Fast Retransmit and Fast Recovery**.