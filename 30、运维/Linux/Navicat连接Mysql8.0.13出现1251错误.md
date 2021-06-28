安装了最新版的MySQL 8.0.13数据库，结果Navicat连接Mysql报1251错误，但是window命令进入mysql，账号密码都是正确的。

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165648.png)

在网上查的是,出现这个原因是mysql8 之前的版本中加密规则是mysql_native_password,而在mysql8之后,加密规则是caching_sha2_password, 解决问题方法有两种,一种是升级navicat驱动,一种是把mysql用户登录密码加密规则还原成mysql_native_password. 



我这里说的是第二种方式 

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165651.png)

1.ALTER USER 'root'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER; #修改加密规则 

2.ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password'; #更新一下用户的密码 

3.FLUSH PRIVILEGES; #刷新权限 

\--------------------------------------------------------------------------------------------------------------------------------------------------------

'root'  为你自己定义的用户名

'localhost' 指的是用户开放的IP，可以是'localhost'(仅本机访问，相当于127.0.0.1)，可以是具体的'*.*.*.*'(具体某一IP)，也可以是 '%' (所有IP均可访问)

'password' 是你想使用的用户密码，我这里的密码是123456