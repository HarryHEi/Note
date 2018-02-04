---
title: 服务器架设学习笔记二
date: 2016-04-08 20:05:32
tags: [linux]
---

# 网络设备
+ **主机硬件**：考虑使用年限、省电、虚拟技术等
+ **Linux操作系统**：考虑稳定性
+ **网卡**：考虑服务器用途、驱动程序
+ **Switch/Hub**：考虑主机数量、带宽、网管功能
+ **网线**：考虑速度相配的等级、线缆形状、配线等
+ **无线网络设备**：考虑速度、标准、安全性

# 网卡
+ 网卡名称：**网卡**(Network Interface Card，NIC)名称以网卡内核模块对应设备名称表示，默认网卡名为**eth0**
+ linux网络参数配置文件：
	+ **ubuntu**配置文件：
		+ **IP地址**配置文件：**/etc/network/interfaces**，设置DHCP(动态主机配置协议)或者手动设置静态IP
		+ 以DHCP方式配置网卡：

			```
			auto eth0
			iface eth0 inet dhcp
			```
		+ 为网卡配置静态IP：

			```
			auto eth0
			iface eth0 inet static
			address 192.168.0.100
			gateway 192.168.0.1
			netmask 255.255.255.0
			```
		+ **/etc/init.d/networking restart**，重启以生效
		+ **/bin/hostname newname**，设置主机名称

	+ **CentOS**配置文件：
	<http://www.cnblogs.com/vicowong/archive/2011/04/23/2025545.html>

+ **无线网络**(Wireless Local Area Network，**WLAN**)：使用无线电波作为传输媒介，IEEE802.11/a/g/b/n/ac标准
+ **WIFI**(Wireless Fidelity，无线保真)：WIFI是WLAN的一个标准
+ **无线接入点**(Wireless Acess Point，简称**AP**)：就是俗称的无线路由器
# linux常用网络命令
+ **ifconfig**：查询、设置网卡的IP等相关参数
+ **ifup**，**ifdown**：两个script文件，启动、关闭网络接口
+ **route**：查看、配置路由表(route table)ping
+ **ip**：整合式命令，可以直接修改上述提到的功能
+ **ping**：通过ICMP检查网络状态
+ **netstat**：查看网络连接和后门
```
-r：列出路由表，如同route
-n：不使用主机名和服务名称，使用IP与port number，如同route -n
-a：列出所有连接状态，包括tcp/udp/unix socket等
-t：仅列出tcp数据包连接
-u：仅列出udp数据包连接
-l：仅列出已在监听(Listen)的服务的网络状态
-p：列出PID与Programe的文件名
-c：设置几秒后自动更新一次

herui@herui-Compaq-511:~$ sudo netstat -tulnp
显示已启动的网络服务
```
+ **host**：查询某个主机ip，(更详细的可以使用**dig**)
```
-a：列出详细设置数据

herui@herui-Compaq-511:~$ host www.baidu.com
www.baidu.com is an alias for www.a.shifen.com.
www.a.shifen.com has address 119.75.218.70
www.a.shifen.com has address 119.75.217.109
```
+ **nslookup**：和host类似，不过还可以由dIP找出主机名
+ **nm-tool**：查询详细网络参数，包括DNS
+ **wget**：文字接口下载器
+ **tcpdump**：文字接口数据包捕获器
# 网络安全
+ **防火墙**
	+ 第一层防火墙：数据包过滤防火墙(Net Filter)，iptables这个软件提供的防火墙功能，针对TCP/IP数据包头部进行过滤
	+ 第二层防火墙：TCP Wrappers，网络数据包接收Super Daemons以及TCP Wrappers的检验

+ **SELinux**(Security Enhanced Linux，安全强化Linux)
针对网络服务的权限设置一些规则，让程序拥有的功能有限
	+ **主体**(Subject)
	SELinux主要管理的就是程序，主体就是程序(Process)
	+ **目标**(Object)
	主题程序访问的目标资源一般是文件系统
	+ **策略**(Policy)
	由于程序和文件数量庞大，SELinux会根据服务制定安全性策略，这些策略会有详细的规则(Rule)来指定资源访问与否
	+ **安全性环境**(Security Context)
	主体和目标的安全性必须一致
	查看安全性环境

		```
		ls -Z
		```
	安全性环境主要用冒号分为三个字段
	
		```
		Identify：Role：Type
		身份识别：角色：类型
		```
	+ **身份识别**(Identify)：相当于帐号方面的身份识别
		+ **root**：表示**root帐号身份**
		+ **system_u**：表示系统程序方面的识别，通常指**程序**
		+ **user_u**：代表**一般用户**帐号相关的身份
	+ **角色**(Role)：通过角色字段，我们可以知道这个数据是代表程序、文件资源还是用户
		+ **object_r**：代表的是**文件或者目录**等文件资源，是最常见的
		+ **system_r**：代表**程序**，不过，**一般用户**也会被指定称为system_r
	+ **类型**(Type)：在默认(targeted)策略中，身份识别(Identify)、角色(Role)基本上是不重要的，**重要的在于这个类型(Type)字段**，基本上，一个程序能不能读取到文件资源，与类型字段有关，类型字段在文件和程序中的定义不太相同
		+ Type：在文件资源中(Object)中称为类型(Type)
		+ Domain：在主体程序(Subject)中称为域(Domain)

		Domain要与Type搭配才能顺利读取文件资源
		
	Identify | Role | 意义
	-----------|--------|---------
	root | system_r | 代表供root帐号登录时所取得的权限
	system_u | system_r | 由于是系统帐号，因此是非交互式的系统运行程序
	user_u | system_r | 一般可登录用户的程序

# WWW服务器
+ **ubuntu apache2配置**
apache 针对服务器环境的配置文件
**/etc/apache2/apache2.conf**
```
ServerRoot  
#服务器设置的最顶层
PidFile ${APACHE_PID_FILE}
#放置pid的文件
Timeout 300
#当持续连接等待300秒后该次连接中断
KeepAlive On
是否允许持续性连接，持续性连接就是一次TCP连接可以具有多个文件资料传送要求
MaxKeepAliveRequests 100
#当KeepAlive设置为On时，这个数值决定该次连接能够传输的最大传输数量
KeepAliveTimeout 5
#允许KeepAlive前提下，该次连接在最后一次传输后等待的秒数
```
VH(虚拟主机)的设置
**/etc/apache2/sites-available/**
设置完之后导入
```
#sudo a2dissite example.com.conf
sudo a2ensite example.com.conf
sudo service apache2 restart
```
针对模块功能的配置
**/etc/apache2/mods-available/mpm_prefork.conf**
```
<IfModule mpm_prefork_module>
        StartServers                     2
        #启动Apache时启动process的数量
        MinSpareServers           6
        MaxSpareServers          12
        #代表最大与最小备用程序数量
        MaxRequestWorkers         30
        #每个程序能够提供的最大传输次数要求
        MaxConnectionsPerChild   3000
</IfModule>

```

参考
<https://segmentfault.com/a/1190000002517469>
<https://www.linode.com/docs/websites/apache/apache-web-server-on-ubuntu-14-04>
<https://www.digitalocean.com/community/tutorials/how-to-configure-the-apache-web-server-on-an-ubuntu-or-debian-vps>
