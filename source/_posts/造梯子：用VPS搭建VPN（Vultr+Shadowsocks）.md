---
title: 造梯子：用VPS搭建VPN（Vultr+Shadowsocks）
date: 2018-09-04 10:51:08
tags:
	- 翻墙
categories: 工具
---

最近开始了实验室搬砖的生活，首先要把基础运维工作做好。其中，VPN是必不可少的，对查阅国内外文献资料，学习学习国际前沿知识都是很有帮助的。以前一直用学校和公司的VPN，现在用不了了，于是打算自己搭一个。

本文记录如何使用Vultr（VPS提供商）和Shadowsocks来搭建属于自己的VPN。

<!--- more --->

### Vultr

Vultr是一家VPS服务商，所谓VPS（Virtual Private Server）就是把一台服务器分割成多个虚拟的专有的服务器以供多人使用。

简单来说就是，这家网站搞了一堆服务器，又把服务器分割成一个个小服务器，他们拥有独立的IP、操作系统、磁盘空间等以供大家使用。想想自如的隔断房......

我们要做的就是从Vultr租这么一间“隔断间”。

网站操作十分简单，注册、登录之后，直接点击Server中的加号，选择心仪的服务器，部署即可。

当然，这家网站是需要充钱的，10美元起充。服务器的价格也不算太贵，最低版本的服务器只支持IPv6协议，我个人购买了5美元/月的服务器（1000Gb/月）。

网站支持AliPay，充值完成后就可以部署服务器了。这家网站被“重点关照”过，所以新加坡和日本的服务器的IP有很多被禁了，大家部署之后要先Ping一下试试，如果丢包率过高，尝试换别的服务器。

另外，Vultr提供了脚本测试各个地区服务器的速度，有兴趣的同学可以再尝试。

我部署时测试的速度大概是新加坡、日本最快，美国东部城市的稍慢一些。

然而日本和新加坡的服务器IP被封了，Vultr默认部署同一地区的服务器是同一IP，也就是说你关掉这一台服务器，重新部署之后也还是这个IP，除非你买一个会员，才可以重新配置IP。

于是我选择洛杉矶的服务器，网速还可以接受。

部署很简单，按照步骤操作即可，系统选择CentOS7。

部署好后，会显示你的IP与登录账户名和密码。

---
### 部署Shadowsocks Server

使用Vultr的Console或者在本地使用ssh。

```
> ssh 用户名@IP地址
输入密码
```

进入后，安装bbr加速工具和shadowsocks server（Linux端）

```
> wget –no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh

> chmod +x bbr.sh

> ./bbr.sh

> wget — no-check-certificate -O shadowsocks.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh

> chmod +x shadowsocks.sh

> ./shadowsocks.sh 2>&1 | tee shadowsocks.log
```

接下来配置shadowsocks

```
> vi /etc/shadowsocks.json
```

会出现如下配置：

```
{
    "server":"xxx.xxx.xxx.xxx",
    "local_address":"127.0.0.1",
    "local_port":1080,
    "port_password":{
         "端口":"密码"
    },
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```

更改 `port_password` 和 `method` 即可增加端口和对应密码（多用户），以及更改加密方式。

`port_password`中的书写方式如下：

```
"port_password":{
         "1234":"123456",
         "2345":"234567",
         "3456":"345678"
    },
```

配置好`shadowsocks.json`的文件后，记得要开放对应的端口。

如开放1234端口：

```
/sbin/iptables -I INPUT -p tcp --dport 1234 -j ACCEPT
```

接着，执行如下命令重启shadowsocks。

```
/etc/init.d/shadowsocks restart
```

---

CentOS 7 默认使用firewalld 代替了之前的Iptables

```
启动： systemctl start firewalld

关闭： systemctl stop firewalld

查看状态： systemctl status firewalld 

开机禁用  ： systemctl disable firewalld

开机启用  ： systemctl enable firewalld

查看版本： firewall-cmd --version

查看帮助： firewall-cmd --help

显示状态： firewall-cmd --state

查看所有打开的端口： firewall-cmd --zone=public --list-ports

更新防火墙规则： firewall-cmd --reload

查看区域信息:  firewall-cmd --get-active-zones

查看指定接口所属区域： firewall-cmd --get-zone-of-interface=eth0

拒绝所有包：firewall-cmd --panic-on

取消拒绝状态： firewall-cmd --panic-off

查看是否拒绝： firewall-cmd --query-panic
```

我们需要使用的命令是 

```
开启一个80端口
> firewall-cmd --zone=public --add-port=80/tcp --permanent    
（--permanent永久生效，没有此参数重启后失效，--zone #作用域 ，--add-port=80/tcp #添加端口，格式为：端口/通讯协议 ）

重新载入firewall
> firewall-cmd --reload

查看已开放的端口
> firewall-cmd --list-ports

删除开放80端口
> firewall-cmd --zone= public --remove-port=80/tcp --permanent
```

---

至此，基本的ss搭建完毕，回到本机，打开ss，输入对应的IP、端口、密码及加密方式，打开代理即可。


