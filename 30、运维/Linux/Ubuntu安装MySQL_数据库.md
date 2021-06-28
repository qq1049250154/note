Ubuntu版本16.0.4

Ubuntu安装比较简单，只需要三条命令

1、sudo apt-get install mysql-server

提示继续执行输入Y

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165820.png)

等待提示设置密码，此密码是以后登录数据库的密码

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165822.png)

重复上一步设置的密码

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165824.png)

2、sudo apt-get install mysql-client

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165827.png)

3、sudo apt-get install libmysqlclient-dev

输入Y继续执行，等待安装成功

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165829.png)

测试数据库

输入mysql -u root -p

提示输入之前第一条命令设置的密码，出现如下界面则安装成功