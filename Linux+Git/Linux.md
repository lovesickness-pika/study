### 环境

- 查看环境：

```shell
export
cat /etc/redhat-release		//查看centos版本

//防火墙
firewall-cmd --help		//查看防火墙相关
firewall-cmd --state	//防火墙运行状态
systemctl status firewalld		//防火墙状态
firewall-cmd --list-ports		//防火墙开放端口
```

- 配置环境

修改/etc/profile文件，然后export PATH=$PATH:指定路径



```shell
systemctl | grep mysql	//查看指定服务
systemctl list-units --type service		//查看所有服务进程
systemctl | more //查看所有正运行的服务
netstat -pnltu		//查看服务以及监听他们的端口
```





#### 安装

##### RPM包安装

