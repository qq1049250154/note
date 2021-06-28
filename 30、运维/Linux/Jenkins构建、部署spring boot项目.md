## 一、环境搭建

本次实验的环境为Ubuntu 16.04，Jenkins 2.8.3

**1、安装ssh**

```
sudo apt-get update  # 更新软件源
sudo apt-get install openssh-server  # 安装ssh
sudo ps -e |grep ssh ## 查询是否启动 ，如果没有启动 sudo service ssh start 启动
使用gedit修改配置文件”/etc/ssh/sshd_config” 获取远程ROOT权限
打开”终端窗口”，输入”sudo gedit /etc/ssh/sshd_config“–>回车–>把配置文件中的”PermitRootLogin without-password“加一个”#”号,把它注释掉–>再增加一句”PermitRootLogin yes“–>保存，修改成功
```

查看ip：

```
ifconfig
```

**2、安装vim**

```
sudo apt-get install vim
```

**3、本地使用ssh工具或者git bash远程连接**

```
ssh root@47.95.0.243 -p 22
```

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165132.png)

这里推荐一个不错的ssh工具，基于Java开发，叫FinalShell，下载地址http://www.hostbuf.com/。自带加速海外连接功能。

**4、安装jdk**

```
sudo apt-get install openjdk-8-jdk
java -version # 查看是否安装成功
```

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165135.png)



openjdk的安装目录在 /usr/lib/jvm/java-8-openjdk-amd64

**5、安装maven**

最新版本为3.6.0

```
wget http://apache.communilink.net/maven/maven-3/3.6.0/binaries/apache-maven-3.6.0-bin.tar.gz ## 下载
tar vxf apache-maven-3.5.0-bin.tar.gz  ## 解压
mv apache-maven-3.5.0 /usr/local/maven3  ## 移动
```

修改环境变量

在/etc/profile 中添加以下几行

```
MAVEN_HOME=/usr/local/maven3 #此处根据你的maven安装地址修改
export MAVEN_HOME
export PATH=${PATH}:${MAVEN_HOME}/bin
```

执行source /etc/profile使环境变量生效

运行mvn -v验证maven是否安装成功

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165138.png)



**6、安装git**

```
sudo apt-get install git # 安装git
git config --global user.name "zsh"
git config --global user.email "43240825@qq.com"
```

**7、关闭防火墙**

```
sudo ufw status # 查看防火墙状态
sudo ufw disable  #关闭防火墙
```

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165142.png)

active为开启状态。inactive为关闭状态

**8、安装MySQL(非必需)**

```
sudo apt-get update
sudo apt-get install mysql-server
```

在弹出的页面中输入两次数据库root用户的密码即可。

修复数据库中文乱码问题

修改/etc/mysql/my.cnf，加入下面这几行

```
[mysqld]
character_set_server=utf8
[mysql]
default-character-set= utf8
[client]
default-character-set = utf8
```

重启数据库：

```
service mysql restart
```

查询数据库字符编码

```
mysql -uroot -p
show variables like '%character%';
```

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165146.png)

自此中文乱码问题已经修复。

一般阿里云或者腾讯云买的服务器，Ubuntu 16.04 中自带ssh和vim。

## 二、Jenkins 安装

**下载** https://pkg.jenkins.io/debian-stable/

```
sudo apt-get update
sudo apt-get install jenkins
```

或者离线下载之后，上传至服务器，此处我放在了 /opt

**启动服务**

```
默认启动在8080
java -jar jenkins.war &
启动在指定端口可以
nohup java -jar jenkins.war --httpPort=8080 &
```

Jenkins 就启动成功了！它的war包自带Jetty服务器

第一次启动Jenkins时，出于安全考虑，Jenkins会自动生成一个随机的按照口令。**注意控制台输出的口令，复制下来**，然后在浏览器输入密码：



![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165149.png)

因为项目是启动在 ubuntu 系统里，所以我们在外面可以用服务器ip访问

http://47.95.0.243:8080

此处注意，如果没有给服务器防火墙打开8080端口，是没法访问的。

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165154.png)

输入上面的密码

进入用户自定义插件界面，建议选择安装官方推荐插件，因为安装后自己也得安装:

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165156.png)



![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165158.png)

等待一段时间之后，插件安装完成，如果有部分插件未安装成功，不比担心，继续配置用户名密码:

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165201.png)



## 三、Jenkins 配置

进入 系统管理 -> 全局工具配置

**1、配置jdk**

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165204.png)

**2、配置git**



![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165211.png)

**3、配置maven**



![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165214.png)

## 四、部署项目



1、首页点击新建：输入项目名称

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165217.png)

如果你没有第二个选项，需要安装 Maven Integration 插件

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165221.png)

2、勾选丢弃旧的构建，选择是否备份被替换的旧包。我这里选择备份最近的10个

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165223.png)

3、源码管理,选择git,配置Git相关信息

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165226.png)



4、构建环境中勾选“Add timestamps to the Console Output”，代码构建的过程中会将日志打印出来

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165229.png)



5、在Build中输入打包前的mvn命令，如：

```
clean install -Dmaven.test.skip=true -Ptest
```

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165232.png)



6、Post Steps 选择 Run only if build succeeds

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165234.png)



7、点击Add post-build step，选择 Excute Shell

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165236.png)



```
cd /home/admin/Jenkins-in #根据自己stop.sh、replace.sh脚本地址写
sh stop.sh
sh replace.sh
BUILD_ID=dontKillMe nohup java -jar /home/admin/workspace/personal-0.0.1-SNAPSHOT.jar &
#根据自己jar包的名称、地址修改
```

stop.sh

```
# 将应用停止
#stop.sh
#!/bin/bash
echo "Stopping SpringBoot Application"
pid=`ps -ef | grep personal-0.0.1-SNAPSHOT.jar | grep -v grep | awk '{print $2}'`
if [ -n "$pid" ]
then
   kill -9 $pid
fi

#此处personal-0.0.1-SNAPSHOT.jar根据自己的jar包名称修改
```

replace.sh

```
#replace.sh 用于将上次构建的结果备份，然后将新的构建结果移动到合适的位置
#!/bin/bash
# 先判断文件是否存在，如果存在，则备份
file="/www/server/workspace/autumn-0.0.1-SNAPSHOT.jar"
if [ -f "$file" ]
then
   mv /home/admin/workspace/personal-0.0.1-SNAPSHOT.jar /home/admin/workspace/personal-0.0.1-SNAPSHOT.jar.`date +%Y%m%d%H%M%S`
fi
mv /root/.jenkins/workspace/hello/target/personal-0.0.1-SNAPSHOT.jar /home/admin/workspace/personal-0.0.1-SNAPSHOT.jar

#此处 /home/admin/workspace/personal-0.0.1-SNAPSHOT.jar根据自己实际jar包名称和路径修改
```

此处如果使用windows的notepad++写好之后再上传上去，有可能出现一个错误



![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165241.png)



```
stop.sh: Syntax error: end of file unexpected (expecting "then")
```

解决方案

在vim中修改下文件的格式就好了，直接输入":"，然后在":"之后输入"set ff"如下图所示

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165244.png)

把格式改为unix，方法是输入":set ff=unix"，也可以输入":set fileformat=unix"如下图所示。

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165246.png)



输入完之后，回车即可完成切换格式。然后我们再输入":set ff"来查看格式，如下图所示，可以看到当前脚本格式变成了我们想要的"unix"了。

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165249.png)



此时就没有问题了。

## 五、构建项目



![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165251.png)



左侧有构建状态，蓝色表示成功，红色表示失败。

点进去可以查看本次构建信息，点击左侧的控制台日志。

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165255.png)



![image](https://cdn.nlark.com/yuque/0/2021/png/12865511/1621043998931-1385eef5-fbc8-4547-94d1-281dc531c5ad.png)

访问项目，成功！

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165258.png)