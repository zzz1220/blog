---
title: '教练,我想学webSocket'
layout: post
tags:
  - webSocket
category: code
comments: true
abbrlink: 9ceb5dee
---
> html5里,我最感兴趣的两个部分一个是canvas,因为可以实现在页面上画画,基于webgl的技术在未来网页游戏上一定会大放异彩,而另外一部分,就是webSocket了。**但是实际上websocket不是HTML5 的东西**。它是一个协议,归属于IETF。HTML5只是提供了WebSocket API ，两个规范是独立发布的。

## 什么是webSocket ？    
WebSocket是一种网络通讯协议,目前市面上很多网站都用到了这项技术,比如在线聊天室,在线客服系统,还有一些评论系统,WebIM等。它的特点就是：持久化,实时性,客户端和服务端双向通讯。
为什么需要webSocket呢？相信很多程序员都遇到过这样类似的需求----实时的显示物品的价格、股票走势的图标、网页上实现聊天等交互功能。这些需求如果用http协议,只能客户端发请求,服务端返回结果,就需要不停的"轮询"：每隔一段时候，就发出一个询问，了解服务器有没有新的信息。轮询的效率很低,并且浪费资源,需要服务器有很快的处理速度和资源。另外还有一种长连接的方式（long poll）,是阻塞式的ajax,一次请求直到有结果返回才会进行下一次请求,这种方式需要服务器高并发能力。

而WebSocket就是一个很好的解决方案。


## 怎么做到的 ?    

WebSocket基于HTTP协议,借用HTTP的协议完成一部分握手。
在握手阶段是一样的。

下面是WebSocket的握手（如果你经常看htpp的包,就很容易看懂下面的东西）。

```
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
Origin: http://example.com
```
这里比HTTP多了几个东西
```
Upgrade: websocket
Connection: Upgrade
```
这两行会告诉服务器：这是一个WebSocket协议,请用对应的模块处理。
```
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
```

Sec-WebSocket-Key 是一个Base64 encode的值，这个是浏览器随机生成的密文,要求服务端必须返回一个对应的加密Sec-WebSocket-Accept应答,否则客户端会抛出Error during WebSocket handshake错误，并关闭连接。    
Sec-WebSocket-Protocol 是一个用户定义的字符串，用来区分同URL下，不同的服务所需要的协议。    
Sec-WebSocket-Version 是告诉服务器所使用的Websocket Draft(协议版本),现在都是13。    

然后服务器会返回下列东西，表示已经接受到请求， 成功建立Websocket啦！

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
Sec-WebSocket-Protocol: chat
```

这样就是websocket链接成功,接下来就可以不用request和response愉快的收发数据了。
不同的服务端有不同语言的实现方式,而在客户端通常使用js或者其他封装库。

对比一下HTTP和WebSocket,是这样的~~  
### http:
客户端：现在比特币价格多少！  
服务端：20000刀！  
10s后  
客户端：现在比特币价格多少！  
服务端：18000刀！  
开始轮询~~~  
客户端：现在比特币价格多少！  
0.5s后  
客户端：现在比特币价格多少！  
0.5s后    
客户端：现在比特币价格多少！  
服务端：19000刀  
服务端：19000刀  
服务端：@%#……&@*……*#*（内心os:累死我了，不干了！）  

### WebSocket
奥义--websocket链接！！  
客户端：要是比特币价格变了及时告诉我。  
服务端：ok，没问题。  
服务端：惊天大消息，比特币暴跌！   
客户端：不慌不慌,继续加仓。   
20s后   
服务端：涨了！涨了！  
客户端：还会再涨，加仓，拉满。  


总结就是：  
1  建立在 TCP 协议之上，服务器端的实现比较容易。  
2  与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。   
3  数据格式比较轻量，性能开销小，通信高效。    
4  可以发送文本，也可以发送二进制数据。  
5  没有同源限制，客户端可以与任意服务器通信。  
6  协议标识符是ws（如果加密，则为wss），服务器网址就是 URL。

## 客户端例子
在客户端,webSocket使用起来很简单

```javascript
var ws = new WebSocket("wss://echo.websocket.org");

ws.onopen = function(evt) { 
  console.log("Connection open ..."); 
  ws.send("Hello WebSockets!");
};

ws.onmessage = function(evt) {
  console.log( "Received Message: " + evt.data);
  ws.close();
};

ws.onclose = function(evt) {
  console.log("Connection closed.");
};  
```

初中英语都能看懂是什么意思,就不解释了。
另外还有常用的webSocket库,如socket.io。

### api:  
webSocket.readyState readyState属性返回实例对象的当前状态,0:表示正在连接;1,表示连接成功;
2，表示连接正在关闭;3，表示连接已经关闭，或者打开连接失败。  
webSocket.onopen 用于指定连接成功后的回调函数。  
webSocket.onclose 用于指定连接关闭后的回调函数。  
webSocket.onmessage 用于指定收到服务器数据后的回调函数。   
webSocket.send() 用于向服务器发送数据。   
webSocket.bufferedAmount 表示还有多少字节的二进制数据没有发送出去。  
webSocket.onerror  用于指定报错时的回调函数。   
以上api仅简单介绍。详情参照官方 
**[文档](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket)**

## 服务端
基本上支持网络编程的语言都可以创建webSocket服务,比如java,php,python,node等，网上都有解决方案。    
下面是用python写的服务端代码,只用到了一些网络模块,方便熟悉webSocket协议内容和步骤

```python
import simplejson
import socket
import sys
import base64
import hashlib
import time

HOST = '127.0.0.1'
PORT = 9000
MAGIC_STRING = '258EAFA5-E914-47DA-95CA-C5AB0DC85B11'
HANDSHAKE_STRING = "HTTP/1.1 101 Switching Protocols\r\n" \
    "Upgrade:WebSocket\r\n" \
    "Connection: Upgrade\r\n" \
    "Sec-WebSocket-Accept: {1}\r\n" \
    "WebSocket-Location: ws://{2}/chat\r\n" \
    "WebSocket-Protocol:chat\r\n\r\n"

def parse_data(msg):
    v = ord(msg[1]) & 0x7f
    if v == 0x7e:
        p = 4
    elif v == 0x7f:
        p = 10
    else:
        p = 2

    mask = msg[p:p+4]
    data = msg[p+4:]

    return ''.join([chr(ord(v) ^ ord(mask[k%4])) for k, v in enumerate(data)])

def start():
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    try:
        sock.bind((HOST, PORT))
        sock.listen(100)
    except Exception as e:
        print('bind error')
        print(e)
        sys.exit()

    while True:
        conn, add = sock.accept()

        try:
            handshake(conn)
        finally:
            print('finally')
            conn.close()

    sock.close()
    pass

def handshake(conn):
    headers = {}
    shake = conn.recv(1024)

    print shake

    if not len(shake):
        print('len error')
        return False

    header, data = shake.split('\r\n\r\n', 1)
    for line in header.split('\r\n')[1:]:
        key, value = line.split(': ', 1)
        headers[key] = value

    if 'Sec-WebSocket-Key' not in headers:
        print('this is not websocket, client close.')
        print headers
        conn.close()

        return False

    sec_key = headers['Sec-WebSocket-Key']
    res_key = base64.b64encode(hashlib.sha1(sec_key + MAGIC_STRING).digest())

    str_handshke = HANDSHAKE_STRING.replace('{1}', res_key).replace('{2}', HOST + ":" + str(PORT))
    print str_handshke

    conn.send(str_handshke)
    time.sleep(1)
    conn.send('%c%c%s' % (0x81, 6, 'suren1'))
    msg = conn.recv(1024)
    msg = parse_data(msg)
    print('msg : ' + msg)

    time.sleep(1)
    conn.send('%c%c%s' % (0x81, 6, 'suren2'))
    msg = conn.recv(1024)
    msg = parse_data(msg)
    print('msg : ' + msg)

    time.sleep(1)
    conn.send('%c%c%s' % (0x81, 6, 'suren3'))
    msg = conn.recv(1024)
    msg = parse_data(msg)
    print('msg : ' + msg)

    return True

    pass

if __name__ == '__main__':
    try:
        start()
    except Exception as e:
        print(e)
```



参考：   
（1）[WebSocket 教程](http://www.ruanyifeng.com/blog/2017/05/websocket.html)   
（2）[知乎--WebSocket 是什么原理？为什么可以实现持久连接？](https://www.zhihu.com/question/20215561)   
（3）[webApi接口|mdn](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket)