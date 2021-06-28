 **前  言** 

最近看了一些同学的面经，发现无论什么技术岗位，还是会问到 GET 和 POST 请求的区别，而搜索出来的答案并不能让我们装得一手好逼，那就让我们从 HTTP 报文的角度来撸一波，从而搞明白他们的区别。

------

 **标准答案** 

在开始之前，先看一下标准答案【**来自w3school**】长什么样子来保个底。标准答案很美好，但是在面试的时候把下面的表格甩面试官一脸，问题应该也不大。

![图片](如果面试再问GET和POST区别，就把这篇甩给他.assets/640)

注意，并不是说标准答案有误，上述区别在大部分浏览器上是存在的，因为这些浏览器实现了 HTTP 标准。

所以从标准上来看，GET 和 POST 的区别基本上可以总结如下：

- GET 用于获取信息，无副作用，幂等，且可缓存
- POST 用于修改服务器上的数据，有副作用，非幂等，不可缓存

但是，既然本文从报文角度来说，那就先不讨论 RFC 上的区别，单纯从数据角度谈谈。

------

 **GET和POST报文上的区别** 

先下结论：**GET 和 POST 方法没有本质区别**，仅报文格式不同。

GET 和 POST 只是 HTTP 协议中两种请求方式，而 HTTP 协议是基于 TCP/IP 的应用层协议，无论 GET 还是 POST，用的都是同一个传输层协议，所以在传输上，没有区别。

报文格式上，不带参数时，最大区别仅仅是第一行方法名不同，一个是GET，一个是POST

带参数时报文的区别呢？在约定中，GET 方法的参数应该放在 `url` 中，POST 方法参数应该放在 `body` 中

举个例子，如果参数是 `name=qiming.c`, `age=22`。

GET 方法简约版报文可能是这样的

```
GET /index.php?name=qiming.c&age=22 HTTP/1.1Host: localhost
```

POST 方法简约版报文可能是这样的

```
POST /index.php HTTP/1.1Host: localhostContent-Type: application/x-www-form-urlencoded
name=qiming.c&age=22
```

两种方法本质上是 TCP 连接，没有差别，也就是说，如果我不按规范来也是可以的。我们可以在 URL 上写参数，然后方法使用 POST；也可以在 Body 写参数，然后方法使用 GET。当然，这需要**服务端**支持。

------

 **常见的疑惑问题** 

**一、GET 方法参数写法是固定的吗？**

在约定中，一般我们的参数是写在 `?` 后面，用 `&` 分割。

我们知道，解析报文的过程是通过获取 TCP 数据，用正则等工具从数据中获取 Header 和 Body，从而提取参数。

也就是说，我们**可以自己约定**参数的写法，只要服务端能够解释出来就行，一种比较流行的写法是这样 :

```
http://www.example.com/user/name/yourname/age/22
```

**二、POST 方法比 GET 方法安全？**

按照网上大部分文章的解释，POST 比 GET 安全，因为数据在地址栏上不可见。

然而从传输的角度来说，**他们都是不安全的**，因为 HTTP 在网络上是明文传输，只要在网络节点上抓包，就能完整地获取数据报文。

要想安全传输，就只有加密，也就是 HTTPS。

**三、听说 GET 方法参数长度有限制？**

在网上看到很多关于两者区别的文章都有这一条，提到浏览器地址栏输入的参数是有限的。

首先说明一点，其实HTTP 协议本身倒并没有 Body 和 URL 的长度限制，对 URL 限制的大多是**浏览器**和**服务器端**自己限制的。

浏览器原因就不说了，服务器是因为处理长 URL 要消耗比较多的资源，为了性能和安全（防止恶意构造长 URL 来攻击）考虑，会给 URL 长度加限制。

**四、POST 方法会产生两个TCP数据包？**

有些文章中提到，POST 请求会将 Header 和 Body 分开发送，先发送 Header，服务端返回 100 状态码再发送 Body。

HTTP 协议中**也并没有明确说明** POST 会产生两个 TCP 数据包，而且实际测试(Chrome)发现，Header 和 Body 不会分开发送。

所以，Header 和 Body 分开发送是**部分浏览器**或框架的请求方法，不属于 Post的必然行为。

------

 **代码验证时间** 

如果对 GET 和 POST 请求的报文区别有疑惑，可以直接用Python起一个 Socket 服务端，然后封装简单的 HTTP 处理方法，直接观察和处理 HTTP 报文，就能一目了然。多实验还是有好处的。

```
#!/usr/bin/env python# -*- coding: utf-8 -*-
import socket
HOST, PORT = '', 23333

def server_run():    listen_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)    listen_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)    listen_socket.bind((HOST, PORT))    listen_socket.listen(1)    print('Serving HTTP on port %s ...' % PORT)    while True:        # 接受连接        client_connection, client_address = listen_socket.accept()        handle_request(client_connection)

def handle_request(client_connection):    # 获取请求报文    request = ''    while True:        recv_data = client_connection.recv(2400)        recv_data = recv_data.decode()        request += recv_data        if len(recv_data) < 2400:            break
    # 解析首行    first_line_array = request.split('\r\n')[0].split(' ')
    # 分离 header 和 body    space_line_index = request.index('\r\n\r\n')    header = request[0: space_line_index]    body = request[space_line_index + 4:]
    # 打印请求报文    print(request)
    # 返回报文    http_response = b"""\HTTP/1.1 200 OK
<!DOCTYPE html><html><head>    <title>Hello, World!</title></head><body><p style="color: green">Hello, World!</p></body></html>"""    client_connection.sendall(http_response)    client_connection.close()

if __name__ == '__main__':    server_run()
```

上面代码就是用Python写的简单的打印请求报文然后返回 Hello World 的 html 页面，接着运行起来：

```
[root@chengqm shell]# python httpserver.pyServing HTTP on port 23333 ...
```

然后从浏览器中来请求看看

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

打印出来的报文

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

然后就可以手动验证上面的一些说法，比如说要测试 Header 和 Body 是否分开传输，由于代码没有返回 100 状态码，如果我们 POST 请求成功就说明是一起传输的 (Chrome/postman)。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

又比如 w3school 里面说 URL 的最大长度是 2048 个字符，那我们在代码里面加上一句计算 `uri` 长度的代码即可

```
# 解析首行first_line_array = request.split('\r\n')[0].split(' ')print('uri长度: %s' % len(first_line_array[1]))
```

我们用 Postman 直接发送 >2048 个字符（**比如这里发送2800个字符**）的请求看看：

![图片](如果面试再问GET和POST区别，就把这篇甩给他.assets/640)

很明显可以看到**发2800个字符也都是没问题的**！

然后我们可以得出结论，url 长度限制仅仅是**某些浏览器**和**服务器**的限制，和 HTTP 协议本身并没有关系。

所以有什么想法用Python写个小脚本验证一下就彻底明白了！