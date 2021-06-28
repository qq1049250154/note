**1、在 ubuntu 中 输入 uname -a 查看 ubuntu 的版本**



![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165745.png)



**2、下载对应的 jdk8 （懒去 官网下载的，给你个百度云的链接 ---> \**https://pan.baidu.com/s/1EphlSzhD9uyq5XmMu34h6A , \**提取码: 554m\**\**  ）**



![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165747.png)



***\*3、在 ubuntu 中 新建 一个 jdk8 目录，（输入 mkdir jdk8 ）\****



![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165749.png)



**4、将下载的 jdk8 传入 ubuntu的 jdk8 目录 中（ 你可以通过 ftp 服务器 或者 Xftp 工具 方式）**



**Xftp 工具 安装并且进行简单使用的案例链接：https://www.cnblogs.com/oukele/p/10900620.html**



**5、将 传入的压缩包，进行 解压 （输入 tar -zxvf jdk-8u221-linux-x64.tar.gz ）**



![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165751.png)



**6、配置环境变量 （ 输入 vim /etc/profile 进行编辑 ），在文件内容最后加入，注：vim 命令不能使用的，自个去安装个 vim 或者 使用 gedit 命令 ，没有权限的 在 前面 加 sudo**



```
export JAVA_HOME=/home/oukele/jdk8/jdk1.8.0_221 (  这里填的是 当前用户所在的目录下的jkd8目录下的 解压文件夹 路径 ，比如我填的 )
export JRE_HOME=${JAVA_HOME}/jre  
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib  
export PATH=${JAVA_HOME}/bin:$PATH
```



**![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165753.png)**



**7、让 配置文件 立即生效**



![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165757.png)



**8、输入 java -version 查看 jdk的版本**



![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165759.png)



**9、输入 javac 出现有提示**



![image](https://typoralim.oss-cn-beijing.aliyuncs.com/img/20210517165801.png)



**完成 jdk8的 安装了。**