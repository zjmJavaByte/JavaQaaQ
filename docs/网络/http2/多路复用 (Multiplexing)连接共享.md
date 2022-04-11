### 多路复用 (Multiplexing)||连接共享

#### 多路复用允许同时通过单一的 HTTP/2 连接发起多重的请求-响应消息。

众所周知 ，在 HTTP/1.1 协议中 「浏览器客户端在同一时间，针对同一域名下的请求有一定数量限制。超过限制数目的请求会被阻塞」。

Clients that use persistent connections SHOULD limit the number of simultaneous connections that they maintain to a given server. A single-user client SHOULD NOT maintain more than 2 connections with any server or proxy. A proxy SHOULD use up to 2*N connections to another server or proxy, where N is 

the number of simultaneously active users. These guidelines are intended to improve HTTP response times and avoid congestion.

source：[RFC-2616-8.1.4 Practical Considerations](http://rfc-2616-8.1.4 practical considerations/)

比如TCP建立连接时三次握手有1.5个RTT（round-trip time）的延迟，为了避免每次请求的都经历握手带来的延迟，应用层会选择不同策略的http长链接方案。又比如TCP在建立连接的初期有慢启动（slow start）的特性，所以**连接的重用总是比新建连接性能要好**。

下图总结了不同浏览器对该限制的数目。



### ![img](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204071649724.png)来源:[Roundup on Parallel Connections](http://www.stevesouders.com/blog/2008/03/20/roundup-on-parallel-connections/) 

这也是为何一些站点会有多个静态资源 CDN 域名的原因之一

上面协议解析中提到的stream id就是用作连接共享机制的：

一个request对应一个stream并分配一个id，这样一个连接上可以有多个stream，每个stream的frame可以随机的混杂在一起，接收方可以根据stream id将frame再归属到各自不同的request里面。因而 HTTP/2 能多路复用(Multiplexing) ，允许同时通过单一的 HTTP/2 连接发起多重的**请求-响应**消息。

![img](https://cdn.jsdelivr.net/gh/zjmJavaByte/images/img/202204071649422.png)

 

 

因此 HTTP/2 可以很容易的去实现多流并行而不用依赖建立多个 TCP 连接，HTTP/2 把 HTTP 协议通信的基本单位缩小为一个一个的帧，这些帧对应着逻辑流中的消息。并行地在同一个 TCP 连接上**双向交换**消息。

前面还提到过连接共享之后，需要优先级和请求依赖的机制配合才能解决关键请求被阻塞的问题。http2.0里的每个stream都可以设置又优先级（Priority）和依赖（Dependency）。优先级高的stream会被server优先处理和返回给客户端，stream还可以依赖其它的sub streams。优先级和依赖都是可以动态调整的。动态调整在有些场景下很有用，假想用户在用你的app浏览商品的时候，快速的滑动到了商品列表的底部，但前面的请求先发出，如果不把后面的请求优先级设高，用户当前浏览的图片要到最后才能下载完成，显然体验没有设置优先级好。同理依赖在有些场景下也有妙用。