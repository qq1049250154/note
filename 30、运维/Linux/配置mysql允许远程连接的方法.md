最近在工作中遇到一个问题，发现在Centos7系统下怎么也不能远程连接mysql，通过查找相关的资料，终于解决了，以下方法就是我在碰到远程连接不到Mysql数据库后试过的方法，最终也是解决掉了问题。所以总结一下分享出来，供同样遇到这个问题的朋友们参考学习，下面话不多说了，来一起看看详细的介绍吧。



有两种原因



● 数据库没有授权



● 服务器防火墙没有开放3306端口



一、数据库没有授权



对于mysql数据库没有授权，只需要用一条命令就可以了。



mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;



//远程连接数据库的时候需要输入用户名和密码



用户名：root



密码:123456



指点ip:%代表所有Ip,此处也可以输入Ip来指定Ip



输入后使修改生效还需要下面的语句



mysql>FLUSH PRIVILEGES;



二、服务器防火墙没有开放3306端口



centos 有两种防火墙 FirewallD和iptables防火墙



centos7 使用的是FirewallD防火墙。



FirewallD 是 iptables 的前端控制器，用于实现持久的网络流量规则。它提供命令行和图形界面，在大多数 Linux 发行版的仓库中都有。与直接控制 iptables 相比，使用 FirewallD 有两个主要区别：



1.FirewallD 使用区域和服务而不是链式规则。



2.它动态管理规则集，允许更新规则而不破坏现有会话和连接。



FirewallD 是 iptables 的一个封装，可以让你更容易地管理 iptables 规则 - 它并不是 iptables 的替代品。虽然 iptables 命令仍可用于 FirewallD，但建议使用 FirewallD 时仅使用 FirewallD 命令。



1.FirewallD防火墙开放3306端口



firewall-cmd --zone=public --add-port=3306/tcp --permanent



命令含义：



--zone #作用域



--add-port=3306/tcp #添加端口，格式为：端口/通讯协议



--permanent  #永久生效，没有此参数重启后失效



重启防火墙



systemctl restart firewalld.service



2.iptables 开发3306端口



/sbin/iptables -I INPUT -p tcp -dport 3306 -j ACCEPT



/etc/rc.d/init.d/iptables save



默认情况下，mysql只允许本地登录，如果要开启远程连接，则需要修改/etc/mysql/my.conf文件。

一、修改/etc/mysql/my.conf

找到bind-address = 127.0.0.1这一行

改为bind-address = 0.0.0.0即可

二、为需要远程登录的用户赋予权限

1、新建用户远程连接mysql数据库

grant all on *.* to admin@'%' identified by '123456' with grant option; 

flush privileges;

允许任何ip地址(%表示允许任何ip地址)的电脑用admin帐户和密码(123456)来访问这个mysql server。

注意admin账户不一定要存在。

2、支持root用户允许远程连接mysql数据库

grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option;

flush privileges;

一、打开Windows的命令行窗口，用dos命令中的ping命令试一下两台电脑是否可以连接通；

在命令行中输入：ping 另一台电脑的IP地址；

二、如果可以用ping通，然后修改MySQL数据库；

打开MySQL数据库中得user表；

把里面的Host字段改为%（允许所有人访问，localhost是本地访问），然后重启MySQL服务

具体的修改方法如下

打开命令行窗口

1:  进入mysql 的安装目录 bin目录下

2:  输入 mysql -hlocalhost -uroot -p 

3:  输入密码 root

4:改变表.可能你的账号不允许远程登录,只能在localhost.这个时候要早localhost 的那台电脑更改mysql数据库里的 user 表里的 Host项, 把localhost 改成 %

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165557.webp)

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165559.webp)