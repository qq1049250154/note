**一. 下载压缩包:**

官网地址: http://maven.apache.org/download.cgi

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165846.png)

或者百度网盘链接：https://pan.baidu.com/s/10C3IDcnohJWHbUA-wzBiaA

提取码：2x9h

**二. 上传到linux的/usr/local目录**

cd /usr/local

可以使用rz目录上传

**三. 解压文件**

tar -zxvf apache-maven-3.6.1-bin.tar.gz

**四. 配置环境变量**

vi /etc/profile

export MAVEN_HOME=/usr/local/apache-maven-3.6.1

export PATH=$MAVEN_HOME/bin:$PATH 

![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165848.png)

**五. 刷新环境变量**

source /etc/profile

**六. 检查版本**

mvn -v 