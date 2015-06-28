---
layout:     post
title:      "各种通信协议及REST、Socket概述总结"
subtitle:   "就是一些通信协议的自我理解随笔"
date:       2014-10-06 12:00:00
author:     "Qpai"
header-img: "img/cd-background-img.jpg"
---



## TCP/IP

TCP/IP是一个完整的协议系统，存在一个协议集，其中包含了多个协议，并且对这些协议进行了分层。这些协议定义了数据单元的格式和内容。TCP/IP及其相关协议构成了一套在TCP/IP网络中如何处理、传输和接收数据的完整系统。


### UDP和TCP协议

UDP和TCP是TCP/IP协议分层中，传输层的两个协议。

### TCP(传输控制协议)

是一种面向连接，相对可靠稳定的协议，保证了数据正确性。当数据需要传递时，会进行三次握手建立连接，并且在数据传递时，有确认、窗口、重传、拥塞控制机制，数据传完之后还会进行四次握手来断开连接，节约系统资源。

当应用层向TCP层发送用于网间传输的、用8位字节表示的数据流，TCP则把数据流分为适当长度的报文段，最大传输段大小(MSS)通常受该计算机连接的网络数据链路层的最大传送单元(MTU)限制。

但是因为要建立连接，并且进行多种复杂机制的操作，导致了较慢，占用系统资源比较高。

### UDP(用户数据报协议)

无连接，也没有确认，窗口，重传，拥塞控制等机制，同时也不提供数据包分组，排序的功能，所以当数据发送出去之后，发送端并不知道数据是否发送成功还是失败。

UDP的数据包前8个字节是报头，包含了源端口号，目标端口号，数据报长度，校验值。数据报长度是数据长度+报头长度

由于是无连接的，而且操作机制很少，所以较快，但是数据无序，并且丢包可能性大。如果要进行维护，必须依赖于应用层协议。

### 参考

[TCP/IP入门景点]()

[TCP(传输控制协议)百度百科](http://baike.baidu.com/subview/32754/8048820.htm?fr=aladdin)

[UDP(用户数据报协议)百度百科](http://baike.baidu.com/view/30509.htm?fr=aladdin)

[TCP和UDP有什么区别，TCP和UDP各有什么优缺点？](http://www.ctowhy.com/132.html)

## HTTP(超文本传送协议)

HTTP是TCP/IP协议组的的一个应用层协议，而且基于TCP协议（上面已经说明白了TCP协议和TCP/IP的关系），其由请求和响应构成，是一个标准的客户端服务端模型。

HTTP协议是无状态协议，对于事务的处理没有记忆能力，假如后续处理需要前面的信息，它必须重新传输。

目前使用的HTTP/1.1协议和HTTP1.0及以前版本的区别是：当一个获取一个web页面的时候，以前版本可能需要建立多个TCP连接，而1.1版本只需要建立一个连接，重复使用。

目前的绝大部分防火墙可以允许HTTP协议请求的通过。

REST(表述性状态转移)，这是一种架构风格，是一组架构约束条件和原则。如果你的应用满足了这些约束条件和原则，那么你的应用就是RESTful。

* 每个资源都应该定义为URI。
* 将所有事物链接在一起
* 使用标准的HTTP方法来进行操作。
* 资源多重表述(即同一个资源不仅可以表述成XML的形式，也可以表述成Json，只需要在Header中进行配置，就可以读取不同表述的同一个资源)
* 无状态通信

### 参考

[深入浅出REST](http://kb.cnblogs.com/page/132129/)
[REST百度百科](http://baike.baidu.com/view/1077487.htm?fr=aladdin)
[HTTP百度百科](http://baike.baidu.com/view/1628025.htm)


## SIP

SIP(会话发起协议)是一种应用层的，基于文本的协议，类似于HTTP协议，使用UDP和TCP协议，还是用了多种其他协议，比如负责语音质量的RSVP(资源预留协议)，负责定为的LDAP(轻型目录访问协议)，负责身份验证的RADIUS(远程身份验证拨入用户服务)，负责实时传输的RTP等多个协议。

![](http://images.cnblogs.com/cnblogs_com/gnuhpc/201201/201201161517218838.png)

SIP有类似于HTTP的Method，比如INVITE，ASK，CANCEL等。同时也有和HTTP类似的状态码和短语，比如1xx，2xx之类

定义了用户标识后，我们就可以定义请求的标识了——SIP中称为方法（Method）。其他的扩展方法在后续的RFC中都有定义。


SIP仅仅是建立和拆除了会话，但是又不定义会话内容类型，究竟是文本，还是语音，还是视频，什么格式的语音，这些信息都需要发起者在INVITE这个命令中带有，这些协议遵守SDP(会话描述协议)

SDP能提供以下信息：

* 会话名和目的
* 会话激活的时间
* 构成会话的媒体
* 怎么样接收这些媒体（地址，端口号，格式）
及其他：
* 会议使用的带宽
* 负责这个会话的那个人的联系方式
等等

SIP消息和具体的媒体流并不是在一个层面运作的。SIP的根本作用是完成点对点(或多点)的媒体流传输的前序工作。

### 参考

[【SIP协议】学习初学笔记](http://www.cnblogs.com/gnuhpc/archive/2012/01/16/2323637.html)

[SIP学习之旅【环境搭建篇】](http://www.cnblogs.com/zhaobl/archive/2011/12/31/2282534.html)

##  XMPP

XMPP(可扩展通讯和表示协议)是一种基于XML的、可扩展的、开放的协议，客户端服务器端协议。它目前执行的标准专门应用于即时通信，为即时通信

因为是基于XML的协议，使其可以进行扩展，增加更多自定义的信息传输的数据中，而这个协议已经被IETF定为标准协议，那么只要是支持XMPP的应用，他们之间就可以互动，使开放成为可能。

XMPP协议基于TCP协议，通过TCP协议传输XML数据。其也可以通过HTTP连接轮询(估计就是我们现在用的HTTP长连接)方式实现，但是在XMPP的`RFC 3920`标准协议文档中只使用TCP。

XMPP中定义了三个角色，客户端，服务器，网关。通信能够在这三者的任意两个之间双向发生。服务器同时承担了客户端信息记录，连接管理和信息的路由功能。网关承担着与异构即时通信系统的互联互通，异构系统可以包括SMS（短信），MSN，ICQ等。基本的网络形式是单客户端通过TCP/IP连接到单服务器，然后在之上传输XML。

客户端可以使用一个TCP连接来发送和接收XML数据，但是服务器必须使用两个TCP连接分别发送和接收XML数据。

  	C: <?xml version='1.0'?>
      <stream:stream to='example.com' xmlns='jabber:client' xmlns:stream='http://etherx.jabber.org/streams' version='1.0'>
 
   	S: <?xml version='1.0'?>
      <stream:stream from='example.com' id='someid' xmlns='jabber:client' xmlns:stream='http://etherx.jabber.org/streams' version='1.0'>
   	...  encryption, authentication, and resource binding ...
 
   	C:   <message from='juliet@example.com' to='romeo@example.net' xml:lang='en'>
 
   	C:     <body>Art thou not Romeo, and a Montague?</body>
 
   	C:   </message>
 
   	S:   <message from='romeo@example.net' to='juliet@example.com' xml:lang='en'>
 
   	S:     <body>Neither, fair saint, if either thee dislike.</body>
 
   	S:   </message>
 
   	C: </stream:stream>
 
   	S: </stream:stream>


以上就是一个客户端和服务器端相互发消息的简化例子，无论是c2s，还是s2c都是<stream:stream></stream:stream>包着内容，每一个<stream:stream>都是一个TCP连接。

XMPP的问题就在于XMPP协议使用的流量太多了，而且不断被重复转发，在移动IM的环境下有了臃肿的感觉。其大部分结构描述比信息都大，因为数据中为了提供路由效率，增加了现场和敏感信息标识，是数据高效路由至最合适的请求资源。


### 参考

[XMPP百度百科](http://baike.baidu.com/view/189676.htm?fr=aladdin)

[XMPP标准文档，RFC3920](http://wiki.jabbercn.org/RFC3920)

[XMPP 协议适合用来做移动 IM 么](http://www.v2ex.com/t/131245)

SOAP
RTSP
H323

## RTP/RTCP

### RTP(实时传输协议)和RTCP(实时传输控制协议)。

RTP是基于UDP协议的传输层的子协议(也可以认为是应用层协议)，定义了流媒体数据包的格式，而RTCP是RTP的控制协议，一般情况下两个协议一起使用。

RTP一般用于流媒体传输，一般的应用场景是IP电话，视频会议等，我估计QQ的视频聊天也是基于RTP或者类似协议。因为基于UDP，所以RTP可以应用于单播或者多播网络。


流媒体的最重要形式就是媒体数据依据相对的时间进行顺序播放，而网络传输来的数据包经常是乱序的。所以RTP定义流媒体数据包中要包含，数据的`序列号`和`时间标签`，这样接收端就可以依据这些数据来进行重组排序，使流媒体可以以正确的顺序播放。

而解包和重组排序需要花费时间，所以一般的视频应用就会进行预加载，和播放缓冲.

RTP只是定义了数据包的结构框架，只是提供了数据包的时间信息和流数据的同步。而传输让UDP负责，而服务质量，流量监控等用RTCP负责。

### RTP会话过程

当应用程序建立一个RTP会话时，应用程序将确定一对目的传输地址。目的传输地址由一个网络地址和一对端口组成，有两个端口：一个给RTP包，一个给RTCP包，使得RTP/RTCP数据能够正确发送。RTP数据发向偶数的UDP端口，而对应的控制信号RTCP数据发向相邻的奇数UDP端口（偶数的UDP端口＋1），这样就构成一个UDP端口对。 RTP的发送过程如下，接收过程则相反。
1)        RTP协议从上层接收流媒体信息码流（如H.263），封装成RTP数据包；RTCP从上层接收控制信息，封装成RTCP控制包。
2)        RTP将RTP 数据包发往UDP端口对中偶数端口；RTCP将RTCP控制包发往UDP端口对中的接收端口。

### 参考

[RTP协议分析](http://blog.csdn.net/bripengandre/article/details/2238818)

[RTP与RTCP协议介绍](http://zhangjunhd.blog.51cto.com/113473/25481/)

## MQTT

MQTT（Message Queuing Telemetry Transport，消息队列遥测传输），是IBM推出的一个即时通信协议。是轻量级的、基于代理的发布/订阅的协议。怎么感觉更像苹果的APNS。

使用TCP/IP系统进行网络请求，但是不确定MQTT在TCP/IP的哪一层。

相对于XMPP，结构描述信息变得特别小巧，只有两字节，所有消息都存在这两字节的位数据中。不要以为两字节很小，存不了多少数据，其实MQTT最多可以实现256M的数据，可谓能伸能缩。

而且很适用于移动互联网的网络不稳定的特点，因为MQTT定义了三种发布服务质量：`至多一次、至少一次，只有一次`。如果网络环境差，那么至少一次绝对能确保消息到达。网络环境非常好，只有一次就够了。但是不只是发送多次的问题，还有根据不同的选择配合不同的协议进来

### 参考

[IBMMQTT文档](http://public.dhe.ibm.com/software/dw/webservices/ws-mqtt/mqtt-v3r1.html)
[MQTT协议详解一](http://blog.csdn.net/king_of_fighter/article/details/17401133)
[MQTT协议详解二](http://blog.csdn.net/king_of_fighter/article/details/17415277)


## Socket

### Socket在TCP/IP中属于什么位置？

![](http://images.cnblogs.com/cnblogs_com/goodcandle/socket2.jpg)

### socket简介

Socket是应用层和TCP/IP协议族通信的软件抽象层，其设计成了门面模式。它把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。

### 参考

[TCP/IP、Http、Socket的区别](http://jingyan.baidu.com/article/08b6a591e07ecc14a80922f1.html)

[揭开Socket编程的面纱](http://goodcandle.cnblogs.com/archive/2005/12/10/294652.aspx)