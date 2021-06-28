HTTP响应（Response）：是服务器发给客户端，浏览器根据响应内容进行解析并在界面展现出来。

        HTTP响应报文：响应行、响应头、响应体构成。其结构如下图所示：



        一、Response Line：响应行
    
        一般由协议版本、状态码及其描述组成，比如 HTTP/1.1 200 OK
    
        常见的状态码：
    
                100~199：表示成功接收请求，要求客户继续提交下一次请求才能完成整个处理过程；
    
                200~299：表示成功接收请求并已经完成整个处理过程。常用200（OK）、201（Created）
    
                300~399：表示为完成请求，客户需要进一步细化请求，常用301（Moved Permanently）；
    
           400~499：客户端的请求有误，常用400（Bad Request）、401（Unauthorized）、404（Not Found）、403（Forbidden）；
    
                500~599：服务器出现错误，常用500（Internal Server Error）
    
        二、Response Header： 响应头
    
        响应头：是用于描述服务器的基本信息，以及数据的描述，服务器通过这些数据的描述信息可以知道客户端如何处理等一会儿它回送的数据。其组成如下：

Header	解释	示例
Accept-Ranges	表明服务器是否支持指定范围请求及哪种类型的分段请求	Accept-Ranges: bytes
Age	从原始服务器到代理缓存形成的估算时间（以秒计，非负）	Age: 12
Allow	对某网络资源的有效的请求行为，不允许则返回405	Allow: GET, HEAD
Cache-Control	告诉所有的缓存机制是否可以缓存及哪种类型	Cache-Control: no-cache
Content-Encoding	web服务器支持的返回内容压缩编码类型。	Content-Encoding: gzip
Content-Language	响应体的语言	Content-Language: en,zh
Content-Length	响应体的长度	Content-Length: 348
Content-Location	请求资源可替代的备用的另一地址	Content-Location: /index.htm
Content-MD5	返回资源的MD5校验值	Content-MD5: Q2hlY2sgSW50ZWdyaXR5IQ==
Content-Range	在整个返回体中本部分的字节位置	Content-Range: bytes 21010-47021/47022
Content-Type	返回内容的MIME类型	Content-Type: text/html; charset=utf-8
Date	原始服务器消息发出的时间	Date: Tue, 15 Nov 2010 08:12:31 GMT
ETag	请求变量的实体标签的当前值	ETag: “737060cd8c284d8af7ad3082f209582d”
Expires	响应过期的日期和时间	Expires: Thu, 01 Dec 2010 16:00:00 GMT
Last-Modified	请求资源的最后修改时间	Last-Modified: Tue, 15 Nov 2010 12:45:26 GMT
Location	用来重定向接收方到非请求URL的位置来完成请求或标识新的资源	Location: http://www.zcmhi.com/archives/94.html
Pragma	包括实现特定的指令，它可应用到响应链上的任何接收方	Pragma: no-cache
Proxy-Authenticate	它指出认证方案和可应用到代理的该URL上的参数	Proxy-Authenticate: Basic
refresh	应用于重定向或一个新的资源被创造，在5秒之后重定向（由网景提出，被大部分浏览器支持）	

 

Refresh: 5; url=
http://www.zcmhi.com/archives/94.html
Retry-After	如果实体暂时不可取，通知客户端在指定时间之后再次尝试	Retry-After: 120
Server	web服务器软件名称	Server: Apache/1.3.27 (Unix) (Red-Hat/Linux)
Set-Cookie	设置Http Cookie	Set-Cookie: UserID=JohnDoe; Max-Age=3600; Version=1
Trailer	指出头域在分块传输编码的尾部存在	Trailer: Max-Forwards
Transfer-Encoding	文件传输编码	Transfer-Encoding:chunked
Vary	告诉下游代理是使用缓存响应还是从原始服务器请求	Vary: *
Via	告知代理客户端响应是通过哪里发送的	Via: 1.0 fred, 1.1 nowhere.com (Apache/1.1)
Warning	警告实体可能存在的问题	Warning: 199 Miscellaneous warning
WWW-Authenticate	表明客户端请求实体应该使用的授权方案	WWW-Authenticate: Basic
        三、Response Body：响应体

        响应体：就是响应的消息体。如果是纯数据就是返回纯数据、请求的是HTML页面返回就是HTML代码、JS就是返回JS

   更多内容尽在https://blog.csdn.net/sinat_41898105，欢迎各界大佬前来补充建议！！！
————————————————
版权声明：本文为CSDN博主「lajos182」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/sinat_41898105/article/details/80952338