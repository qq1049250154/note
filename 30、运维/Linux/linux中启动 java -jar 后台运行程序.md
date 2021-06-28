直接用java -jar xxx.jar，当退出或关闭shell时，程序就会停止掉。以下方法可让jar运行后一直在后台运行。

1.

java -jar xxx.jar &

说明： 在末尾加入 & 符号

2.

（1）执行java -jar xxx.jar后

（2）ctrl+z 退出到控制台,执行 bg

（3）exit

完成以上3步，退出SHELL后，jar服务一直在后台运行。

3.

nohup java -jar xxxx.jar &

将java -jar xxxx.jar 加入  nohup  &中间，也可以实现