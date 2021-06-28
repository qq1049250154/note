## 引言

Nginx作为高性能的web和反向代理服务器，在互联网公司应用广泛。作为一名刚入职的小白，9月底的时候经历了公司站点的HTTPS改造，虽然没有亲手配置nginx, 而且一开始看到Nginx配置还是很懵逼的 （为什么本科学校不专门学一下啊？%>_<% 还是怪自己不够主动去学），本文写给从没接触过Nginx的同学，也算是入门，不会太深入，有兴趣的同学可以买[《深入理解Nginx》](https://link.jianshu.com?t=https://book.douban.com/subject/26745255/), （真的要有兴趣啊，很厚的一本书），个人觉得作为开发人员知道一些基本配置就行了，没必要特别深入。

### Nginx的安装

首先介绍一下Nginx的在不同系统下的安装



```csharp
# CentOS
yum install nginx;
# Ubuntu
sudo apt-get install nginx;
# Mac
brew install nginx;
```

本文主要以Mac下的安装为例。
 通过homebrew，nginx默认被安装在`/usr/local/Cellar/nginx/`目录下。conf安装目录在`/usr/local/etc/nginx/nginx.conf`
 启动、热重启、关闭以及测试配置的命令如下：



```bash
# 启动
nginx -s start;
# 重新启动，热启动，修改配置重启不影响线上
nginx -s reload;
# 关闭
nginx -s stop;
# 修改配置后，可以通过下面的命令测试是否有语法错误
nginx -t;
```

在浏览器中键入[http://localhost:8080](https://link.jianshu.com?t=http://localhost:8080),即可访问到nginx的欢迎界面。那么，为什么会访问到nginx的欢迎界面的呢？
 不妨打开nginx.conf，一起来分析一下这个文件。在nginx的配置中用`#`表示注释



```csharp
#user  nobody;
##定义拥有和运行Nginx服务的Linux系统用户

worker_processes  1;
##定义单进程。通常将其设成CPU的个数或者内核数

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
##定义Nginx在哪里打日志


#pid        logs/nginx.pid;
##Nginx写入主进程ID（PID）

events {
    worker_connections  1024;
    ##通过worker_connections和worker_processes计算maxclients。
    ##max_clients = worker_processes * worker_connections
}


http {
    include       mime.types;
    ##在/opt/nginx/conf/mime.types写的配置将在http模块中解析
    
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    ##如果是为了获取本地存储的静态化文件，sendfile可以加速服务端，但是如果是反向代理，那么该功能就失效了。
    #tcp_nopush     on;
##在 nginx 中，tcp_nopush 配置和 tcp_nodelay "互斥"。它可以配置一次发送数据的包大小。也就是说，它不是按时间累计  0.2 秒后发送包，而是当包累计到一定大小后就发送。在 nginx 中，tcp_nopush 必须和sendfile 搭配使用。
    #keepalive_timeout  0;
    keepalive_timeout  65;
    ##设置保持客户端连接时间

    #gzip  on;
##告诉服务端用gzip压缩
    server {
      ##如果你想对虚拟主机进行配置，可以在单独的文件中配置server模块，然后include进来
        listen       8080;
     ##告诉Nginx TCP端口，监听HTTP连接。listen 80; 和 listen *:80;是一样的
        server_name  localhost;
        ##定义虚拟主机的名字
        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
        ##location模块可以配置nginx如何反应资源请求
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
    include servers/*;
}
```

虽然上面的默认配置很多，但是可以总体归纳为三个模块：



```nginx
#全局模块
events {
    #events模块
}

http 
{

   #http全局模块
 
    server 
    {
    
        #server全局模块
     
        location [PATTERN]{
           #location模块
        }
    }

}  
```

1、全局块：配置影响nginx全局的指令。一般有运行nginx服务器的用户组，nginx进程pid存放路径，日志存放路径，配置文件引入，允许生成worker process数等。

2、events块：配置影响nginx服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。

3、http块：可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type定义，日志自定义，是否使用sendfile传输文件，连接超时时间，单连接请求数等。

4、server块：配置虚拟主机的相关参数，一个http中可以有多个server。

5、location块：配置请求的路由，以及各种页面的处理情况。

## Nginx配置Web服务器

先介绍对一个web服务进行简单配置，然后对各个重要点简单说明。这个案例中关于反向代理的要点将在下一篇中介绍。

### 案列



```csharp
########### 每个指令必须有分号结束。#################
#user administrator administrators;  #配置用户或者组，默认为nobody nobody。
#worker_processes 2;  #允许生成的进程数，默认为1
#pid /nginx/pid/nginx.pid;   #指定nginx进程运行文件存放地址
error_log log/error.log debug;  #制定日志路径，级别。这个设置可以放入全局块，http块，server块，级别以此为：debug|info|notice|warn|error|crit|alert|emerg
events {
    accept_mutex on;   #设置网路连接序列化，防止惊群现象发生，默认为on
    multi_accept on;  #设置一个进程是否同时接受多个网络连接，默认为off
    #use epoll;      #事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport
    worker_connections  1024;    #最大连接数，默认为512
}
http {
    include       mime.types;   #文件扩展名与文件类型映射表
    default_type  application/octet-stream; #默认文件类型，默认为text/plain
    #access_log off; #取消服务日志    
    log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'; #自定义格式
    access_log log/access.log myFormat;  #combined为日志格式的默认值
    sendfile on;   #允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。
    sendfile_max_chunk 100k;  #每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
    keepalive_timeout 65;  #连接超时时间，默认为75s，可以在http，server，location块。

    upstream mysvr {   
      server 127.0.0.1:7878;
      server 192.168.10.121:3333 backup;  #热备
    }
    error_page 404 https://www.baidu.com; #错误页    
    server {
        keepalive_requests 120; #单连接请求上限次数。
        listen       4545;   #监听端口
        server_name  127.0.0.1;   #监听地址       
        location  ~*^.+$ {       #请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
           #root path;  #根目录
           #index vv.txt;  #设置默认页
           proxy_pass  http://mysvr;  #请求转向mysvr 定义的服务器列表
           deny 127.0.0.1;  #拒绝的ip
           allow 172.18.5.54; #允许的ip           
        } 
    }
} 
```

### 域名与端口配置

上述例子中  `listen 4545; #监听端口` 表示监听端口是4545。但是对于一个小白来说有时候看到 `listen [::]:80;`,`listen :80;`,`listen *:80;` 这三种写法还是会很懵逼的，那么他们之间有什么区别啊？

> `listen [::]:80;`表示Nginx会同时监听IPv4和IPv6的80端口，`listen :80;`,`listen *:80;` 这两种写法是一样的，

### location中URL匹配

上述例子中，大家发现location 后面跟着的正则匹配，其实在nginx中，location url 匹配是遵循一定优先级的。



```bash
location = / {
    # 完全匹配  =
    # 大小写敏感 ~
    # 忽略大小写 ~*
}
location ^~ /images/ {
    # 前半部分匹配 ^~
     # 匹配任何以 /images/ 开头的地址，匹配符合以后，停止往下搜索正则，采用这一条。
}
location ~* \.(gif|jpg|jpeg)$ {
    # ~* 表示执行一个正则匹配，不区分大小写
    # ~ 表示执行一个正则匹配，区分大小写
    # 匹配所有以 gif,jpg或jpeg 结尾的请求
}
location / {
    # 如果以上都未匹配，会进入这里
}
```

location中的优先级如下

> (location =) > (location 完整路径) > (location ^~ 路径) > (location ,* 正则顺序) > (location 部分起始路径) > (/)



```csharp
location = / {
#仅仅匹配请求
[ configuration A ]
}
location / {
#匹配所有以 / 开头的请求。但是如果有更长的同类型的表达式，则选择更长的表达式。如果有正则表达式可以匹配，则优先匹配正则表达式。
[ configuration B ]
}
location /documents/ {
# 匹配所有以 /documents/ 开头的请求。但是如果有更长的同类型的表达式，则选择更长的表达式。
#如果有正则表达式可以匹配，则优先匹配正则表达式。
[ configuration C ]
}
location ^~ /images/ {
# 匹配所有以 /images/ 开头的表达式，如果匹配成功，则停止匹配查找。所以，即便有符合的正则表达式location，也
# 不会被使用
[ configuration D ]
}
location ~* \.(gif|jpg|jpeg)$ {
# 匹配所有以 gif jpg jpeg结尾的请求。但是 以 /images/开头的请求，将使用 Configuration D
[ configuration E ]
}
```

## 文件路径定义

在location模块中可以定义文件路径
 比如
 **根目录设置：**



```bash
location / {
    root /home/barret/test/;
}
```

**主页设置：**



```nginx
index /html/index.html /php/index.php;
```

**try_files 设置**
 try_file主要是功能是去检查文件是否存在，使用第一个被找到文件返回。如果没有一个文件找到, 那么重定向到最后一个参数指定的URI。如：



```jsx
location /images/ {
    try_files $uri /images/default.gif;
}

location = /images/default.gif {
    expires 30s;
}
```

ps: $uri 是不带请求参数的当前URI，下面的全局变量中会介绍
 ,最后一个参数也可以是命名的location。如下：



```nginx
try_files $uri $uri.html $uri/index.html @other;
location @other {
    # 尝试寻找匹配 uri 的文件，失败了就会转到上游处理
    proxy_pass  http://localhost:9000;
}

location / {
    # 尝试寻找匹配 uri 的文件，没找到直接返回 502
    try_files $uri $uri.html =502;
}
```

## Rewrite 重定向

如果要把一个URL [http://www.jianshu.com/users/10001](https://www.jianshu.com/users/10001) 重写成  [http://www.jianshu.com/show?user=10001](https://www.jianshu.com/show?user=10001)，可以使用rewrite 规则，参见下面的代码。我在公司站点的改造过程中，遇到了rewrite，重写URL目的是为了更好的SEO。



```ruby
location /users/ {
    rewrite ^/users/(.*)$ /show?user=$1 break;
}
```

rewrite功能就是，使用nginx提供的全局变量或自己设置的变量，结合正则表达式和标志位实现url重写以及重定向。

> rewrite 规则 定向路径 重写类型;

1、规则：可以是字符串或者正则来表示想匹配的目标url
 2、定向路径：表示匹配到规则后要定向的路径，如果规则里有正则，则可以使用$index来表示正则里的捕获分组
 3、重写类型：
 last ：相当于Apache里德(L)标记，表示完成rewrite，浏览器地址栏URL地址不变
 break；本条规则匹配完成后，终止匹配，不再匹配后面的规则，浏览器地址栏URL地址不变
 redirect：返回302临时重定向，浏览器地址会显示跳转后的URL地址
 permanent：返回301永久重定向，浏览器地址栏会显示跳转后的URL地址

### break 与 last的区别

- last一般写在server和if中，而break一般使用在location中
- last不终止重写后的url匹配，即新的url会再从server走一遍匹配流程，而break终止重写后的匹配
- break和last都能组织继续执行后面的rewrite指令
   在location里一旦返回break则直接生效并停止后续的匹配location
   举个例子：



```kotlin
server {
    location / {
        rewrite /last/ /q.html last;
        rewrite /break/ /q.html break;
    }
    location = /q.html {
        return 400;
    }
}
```

- 访问/last/时重写到/q.html，然后使用新的uri再匹配，正好匹配到locatoin = /q.html然后返回了400
- 访问/break时重写到/q.html，由于返回了break，则直接停止了

### if表达式

上面的简单重写很多时候满足不了需求，比如需要判断当文件不存在时、当路径包含xx时等条件，则需要用到if
 **if的语法如下：**



```css
if (表达式) {
}
```

**内置的全局变量：**



```bash
$args ：这个变量等于请求行中的参数，同$query_string
$content_length ： 请求头中的Content-length字段。
$content_type ： 请求头中的Content-Type字段。
$document_root ： 当前请求在root指令中指定的值。
$host ： 请求主机头字段，否则为服务器名称。
$http_user_agent ： 客户端agent信息
$http_cookie ： 客户端cookie信息
$limit_rate ： 这个变量可以限制连接速率。
$request_method ： 客户端请求的动作，通常为GET或POST。
$remote_addr ： 客户端的IP地址。
$remote_port ： 客户端的端口。
$remote_user ： 已经经过Auth Basic Module验证的用户名。
$request_filename ： 当前请求的文件路径，由root或alias指令与URI请求生成。
$scheme ： HTTP方法（如http，https）。
$server_protocol ： 请求使用的协议，通常是HTTP/1.0或HTTP/1.1。
$server_addr ： 服务器地址，在完成一次系统调用后可以确定这个值。
$server_name ： 服务器名称。
$server_port ： 请求到达服务器的端口号。
$request_uri ： 包含请求参数的原始URI，不包含主机名，如：”/foo/bar.php?arg=baz”。
$uri ： 不带请求参数的当前URI，$uri不包含主机名，如”/foo/bar.html”。
$document_uri ： 与$uri相同。
```

**内置的条件判断：**

-f和!-f用来判断是否存在文件
 -d和!-d用来判断是否存在目录
 -e和!-e用来判断是否存在文件或目录
 -x和!-x用来判断文件是否可执行

有时候在配置文件中看到`$http_host`。他和`$host`有什么不同呢？

> `$http_host`和`$host`都是原始的’HOST’字段
>  比如请求的时候HOST的值是`www.csdn.net` 那么反代后还是`www.csdn.net`如果客户端发过来的请求的header中没有有’HOST’这个字段时，
>  建议使用`$host`，这时候的`$host`就等于`server_name`。

if 表达式例子：



```ruby
# 如果文件不存在则返回400
if (!-f $request_filename) {
    return 400;
}
# 如果host不是xuexb.com，则301到xuexb.com中
if ( $host != 'xuexb.com' ){
    rewrite ^/(.*)$ https://xuexb.com/$1 permanent;
}
# 如果请求类型不是POST则返回405
if ($request_method = POST) {
    return 405;
}
# 如果参数中有 a=1 则301到指定域名
if ($args ~ a=1) {
    rewrite ^ http://example.com/ permanent;
}
```

if 通常与location规则搭配使用，如：



```bash
# 访问 /test.html 时
location = /test.html {
    # 默认值为xiaowu
    set $name xiaowu;
    # 如果参数中有 name=xx 则使用该值
    if ($args ~* name=(\w+?)(&|$)) {
        set $name $1;
    }
    # 301
    rewrite ^ /$name.html permanent;
}
```

上面表示：

- /test.html => /xiaowu.html
- /test.html?name=ok => /ok.html?name=ok

本文主要参考以下这个文献

## 参考

http://www.linuxeye.com/configuration/2657.html

https://www.jianshu.com/p/734ef8e5a712