## 安装

本人在安装 echo 模块的时候也是遇到各种坑。如果你的电脑不是Mac, 可以参考[Nginx动态模块安装](https://link.jianshu.com?t=https://www.nginx.com/blog/compiling-dynamic-modules-nginx-plus/) 结合echo 模块 [安装指南](https://link.jianshu.com?t=https://github.com/openresty/echo-nginx-module)。
 如果你用的是mac。那么就用brew吧，一条命令安装。可以用一下命令。



```jsx
brew install nginx-full --with-echo-module
```

具体可以参考这个链接([ttps://github.com/Homebrew/homebrew-nginx](https://link.jianshu.com?t=ttps://github.com/Homebrew/homebrew-nginx))
 如果你之前安装过Nginx，用brew安装可能会遇到冲突，可以用下面命令解决冲突。



```undefined
brew unlink nginx
```

## 使用

大家可以通过下面的[链接](https://link.jianshu.com?t=http://oa2xnsbj0.bkt.clouddn.com/testecho.baidu.com.conf)下载该配置文件。后面echo模块的测试用例根据该文件来讲解。如果你是mac电脑，并且按照之前步骤安装好nginx，那么把下载下来的配置文件放在`/usr/local/etc/nginx/servers`目录下面。在本地电脑配置host



```css
127.0.0.1 testecho.baidu.com
```

### 日志

先说一下怎么看nginx 日志。电脑安装nginx后并没创建nginx日志目录文件。需要根据你的配置文件，手动创建。 我在`/usr/local/etc/nginx/`目录下创建日志目录`logs/nginx`,所有访问`testecho.qidian.com`的访问日志和错误日志都会自动打印到该目录下。

> 下面简单介绍一下几个常用的命令

### echo - 输出字符

- 语法: `echo [options] <string>`...



```bash
输出全局变量$remote_addr
 location /test {
     echo $remote_addr;
     echo $args;
 }
```

curl  testecho.baidu.com:8081/test?123
 输出结果为



```css
127.0.0.1
123
```

这样可以方便查看第一篇文章中介绍的全局变量的值，是不是很方便？

### echo_before_body, echo_after_body - 页面前、后插入内容

- 语法: `echo_before_body [options] [argument]...`



```ruby
# 反向代理添加前置、后置内容
location = /api/proxy_before_after {
    echo_before_body hello before;
    proxy_pass http://127.0.0.1:8081/test;
    echo_after_body world after;
}
```

curl  testecho.baidu.com:8081/api/proxy_before_after?123
 输出结果为：



```css
hello before
127.0.0.1
123
world after
```

### echo_sleep - 请求等待

- 语法: `echo_sleep <seconds>`
   该方法可以使得请求等待指定秒数。该方法不会阻塞整个nginx进程。
   curl  testecho.baidu.com:8081/api/sleep
   输出结果为



```undefined
 1
 2
```

### echo_location_async, echo_location - 请求指定路径

- 语法: `echo_location_async <location> [<url_args>]`

异步跟同步的区别是:

1. 异步会并行的去请求
2. 同步等待当前请求结束才会往下执行

下面这个整个时间为2s, 因为所有路径中最大耗时是2s:



```bash
location /main1 {
     echo_reset_timer;
     echo_location_async /sub1;
     echo_location_async /sub2;
     echo "took $echo_timer_elapsed sec for total.";
 }
 location /sub1 {
     echo_sleep 2;
     echo hello;
 }
 location /sub2 {
     echo_sleep 1;
     echo world;
 }
```

curl  testecho.baidu.com:8081/main1 输出结果为



```css
hello
world
took 0.000 sec for total.
```

之所以输出0s因为main1不会去等待两个子请求sub1和sub2。所以非常快就结束了。
 如果将上面main1中的echo_location_async 改成echo_location。
 curl  testecho.baidu.com:8081/main2 输出结果为



```css
hello
world
took 3.002 sec for total.
```

可以通过第二个参数传参数给子请求 `querystring: echo_location_async /sub 'foo=Foo&bar=Bar';`

### echo_foreach_split - 分隔循环

- 语法: `echo_foreach_split <delimiter> <string>`
   该方法可以将请求中参数根据分隔符分离出来。



```bash
   location /loop {
     echo_foreach_split ',' $arg_list;
       echo "item: $echo_it";
     echo_end;
   }
```

curl  testecho.baidu.com:8081/loop?list=cat,dog,mouse 输出结果为



```undefined
item: cat
item: dog
item: mouse
```

### if语句的调试

通过`arg_val`可以获取到请求参数，方便调试if 语句



```bash
   location ^~ /if {
       set $res miss;
       if ($arg_val ~* '^a') {
           set $res hit;
           echo $res;
       }
       echo $res;
   }
```

访问 curl testecho.baidu.com:8081/if?val=abc
 输出



```undefined
hit
```

访问 curl testecho.baidu.com:8081/if?val=bcd



```undefined
miss
```

## 参考

https://www.jianshu.com/p/a636ca5fa8fa