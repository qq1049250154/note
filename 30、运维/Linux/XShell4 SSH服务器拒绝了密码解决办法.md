使用XShell4连接虚拟机，以root账户登录，输入密码时服务器拒绝：



![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165617.png)



安装open ssh： sudo apt-get install openssh-server



启动ssh服务：  sudo /etc/init.d/ssh restart



sshd的设置不允许root用户用密码远程登录



修改 vim /etc/ssh/sshd_config



找到# Authentication:



LoginGraceTime 120



PermitRootLogin without passwd



StrictModes yes



改成



\# Authentication:



LoginGraceTime 120



PermitRootLogin yes



StrictModes yes



重启虚拟机