以下过程是CentOS1.打开此脚本（使用VIM或其他编辑器）：

vim /etc/sysconfig/jenkins

2.找到$JENKINS_USER并更改为“root”：

$JENKINS_USER="root"

3.然后更改Jenkins主页，webroot和日志的所有权：

```
chown -R root:root /var/lib/jenkins
chown -R root:root /var/cache/jenkins
chown -R root:root /var/log/jenkins
```

4）重新启动Jenkins并检查用户是否已更改：

service jenkins restart

ps -ef | grep jenkins

现在你应该能够以root用户身份运行Jenkins作业，并且所有的shell命令将被执行root。