### redis主从复制



#### redis主从复制步骤



slaveof 127.0.0.1



redis从2.8开始，使用PSYNC指令来执行复制时的同步操作

整个复制实现的步骤：

1、设置主服务器的地址和端口 slaveof  127.0.0.1 6379

2、建立套接字(socket)连接

3、向master发送Ping命令，如果收到pong，说明收发正常。

4、身份验证，如果从设置了对masterauth验证，就需要验证，不需要则不需要验证。

5、向主发送从监控的端口，以便把数据同步发到这个端口上。

6、真正开始同步

7、命令传播(同步)
![img](https://img-blog.csdnimg.cn/20200516162642921.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3ppemhpbGlhbnFpdQ==,size_16,color_FFFFFF,t_70)