一、判断[Ubuntu](http://www.linuxidc.com/topicnews.aspx?tid=2)是否开启防火墙



```
sudo ufw status
```



开放防火墙3306端口



```
sudo ufw allow 3306
```



二、查看3306端口是否打开



![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165723.png)

注意：红色框框表示3306绑定的ip地址–>未修改前为：127.0.0.1:3306–>即mysql默认绑定localhost，远程访问不了

*如果是绑定了127.0.0.1则继续看第三步，否则请跳过第三步



三、修改mysql配置文件，将bind-address = 127.0.0.1注释，开放所有连接



```
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
```



![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165724.png)



重启ubuntu，再次查看3306端口状态，同第二步



四、通过telnet尝试连接mysql



```
telnet your-remote-ip-address 3306
```



如果不能连通，继续下一步



五、将root用户授权给所有连接

step1：进入mysql

step2：

法一>改表法：进入mysql数据库，查看里面user表，搜索User=’root’的记录

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165727.png)

注：此处为修改后的记录

修改Host=’localhost’的记录：



```
mysql> UPDATE user SET Host = ‘%’ WHERE User = ‘root’ AND Host=’localhost’;
```



使修改生效：



```
mysql> FLUSH PRIVILEGES;
```



法二>授权法：

例子：允许root用户使用密码password从任何主机连接到mysql：



```
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;
```



使修改生效：



```
mysql> FLUSH PRIVILEGES;
```



最后，可再通过第四步进行测试验证能否远程连接上mysql~