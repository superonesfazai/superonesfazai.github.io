---
layout: post
title: (Linux)在Ubuntu下搭建VPN服务器的方法
---
VPN是什么？中文翻译叫做：虚拟专用网络。功能是，在公用网络上建立专用网络，进行加密通讯。
适用的场合：
1.你的公司网络在一个局域网，不能外部访问。有一天你外出度假了，想访问一下公司的内部网络，外网是不能直接访问的。如果公司的网络有一台主机设置了VPN，你就可以通过连上这台VPN主机，来访问公司内部网络啦。
2.如果你的主机是在国外，你可以在这台主机上配置VPN，然后你的电脑连上VPN之后就可以翻墙啦。
3.某台服务器（如游戏服务器）限制了一些IP连接到它上面，这时你配置VPN，连上VPN之后，就可以继续访问那台服务器咯。
4.etc…
我们以Ubuntu为例，说一下怎样配置VPN服务器。
1、用root账户登陆服务器
2、安装PPTPD
```bash
apt-get install pptpd
```
3、编辑pptpd.conf文件
```bash
vi /etc/pptpd.conf
```
取消注释下面内容
```bash
localip 192.168.0.1
remoteip 192.168.0.234-238,192.168.0.245
```
这几句的意思是：当外部计算机通过pptp联接到vpn后所能拿到的ip地址范围和服务器的ip地址设置。
4、添加用于登陆的账户
```bash
vi /etc/ppp/chap-secrets
```
格式如下：
```bash
# client server secret IP addresses
cqc pptpd 123456 *
```
从左到右依次是用户名，协议(填写pptpd)，密码，自己指定。IP，填*即可。中间用空格分别隔开。
5、设置DNS解析，编辑pptpd-options文件
```bash
vi /etc/ppp/pptpd-options
```
找到ms-dns，取消掉注释，并修改DNS地址，这里我推荐大家用
Google DNS 8.8.8.8 和 8.8.4.4
更改为如下内容
```bash
ms-dns 8.8.8.8
ms-dns 8.8.4.4
```
6、开启转发
```bash
vi /etc/sysctl.conf
```
取消注释以下内容
```bash
net.ipv4.ip_forward=1
```
这句话意思是：打开内核IP转发
更新一下配置
```bash
sudo sysctl -p
```
7、安装iptables并设置
```bash
apt-get install iptables
sudo iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE
```
后面这句话作用是：立刻让LINUX支持NAT(platinum)
或者使用下面这句话来代替
```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```
这种就比较简单，自动伪装ip
如果VPN服务器是用来代理上网的，仅有上面的配置会出现访问网站缓慢的情况，需要手动修改一下转发包的mss
```bash
iptables -A FORWARD -p tcp --syn -s 10.0.0.0/24 -j TCPMSS --set-mss 1356
```
这样就不会出现打不开网页的问题了，最后可以保存一下防火墙。
```bash
iptables-save
```
8、重新启动服务
```bash
/etc/init.d/pptpd restart
```
9、大功告成，VPN服务器就这么配置好啦。
接下来，利用IP地址，刚才设置的VPN账号和密码，就可以连你的VPN啦。
如果有问题，欢迎与我交流~
