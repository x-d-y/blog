---
title: linux网络管理神器--NetworkManager
date: 2019-10-25 16:45:48
tags:
---
### （一）背景
&#160;&#160;&#160;&#160;&#160;&#160;Centos8已经发布，本着新东西要试一试的态度安装了Centos8的虚拟机。看看有什么新的功能，在刚安好Centos8后，和Centos7一样，发现不能上网。于是按照Centos7的步骤进行网络设置，最简单的一种方式就是:
```
vi /etc/sysconfig/network-scripts/ifcfg-ens33 将ONBOOT=no 改为 ONBOOT=yes。
然后
systemctl restart network.service
```
但是发现服务不存在。

&#160;&#160;&#160;&#160;&#160;&#160;经过查阅才知道，从Centos7开始，就已经支持networkManager配置网络。但是我（包括身边和网上其他人）在大部分时候都在使用network.service+配置文件 这种方式在配置网络。记得去年在甲方的机房里，因为要配置网络，在配置文件里写一堆，通宵配好多台机器，各种ip、路由、网关、bound等。如果去年就会使用networkManager可能就不用通宵写配置文件了。
&#160;&#160;&#160;&#160;&#160;&#160;趁着这次探索Centos8的机会，了解了networkManager这个神器，先来看看官方的意图。Centos在8已经默认去掉network.service的方式管理网络，但是依然支持network.service方式进行网络管理，需要自己下载相关组件。从下一个大版本(Centos9还是Centos8.1?)将不再支持network.service的方式进行网络管理。看来官方已经决定大力推广networkManager进行网络管理了。再来看看它的优点：
<!--more-->
```
一：
  支持四种方式进行网络管理：
    1.命令行方式(服务器需要)
    2.terminal UI方式(福音)
    3.图形界面
    4.web界面

二：
  支持多种网络设定
    1.有线网络(废话)
    2.无线网络(没有界面，说实话，我不会设置wifi网络，主要是发现不了可连入的网络)
    3.物理网络
    4.虚拟
三：
  可以设置的参数多达200多项。如果这200项全部用配置文件来实现，而且有大量的机器需要配置，不用睡觉了。
四：
  支持主流的linux发行版本，RedHat系、Suse系、Debian/Ubuntu系

```
就不难发现为什么Redhat/Centos在大力推广这玩意儿了。

### (二)本文重点
- nmcli的基础使用
- nmtui的基础使用
- 分别在Centos和Ubuntu上进行实验

### (三)nmcli
nmcli 为network manager client的缩写
```
$ nmcli
 ens33: connected to ens33
        "Intel 82545EM"
        ethernet (e1000), 00:0C:29:E6:31:65, hw, mtu 1500
        ip4 default
        inet4 172.16.168.128/24
        route4 0.0.0.0/0
        route4 172.16.168.0/24
        inet6 fe80::ce5b:ae6b:9954:1bf0/64
        route6 fe80::/64
        route6 ff00::/8

virbr0: connected to virbr0
        "virbr0"
        bridge, 52:54:00:E9:A0:A0, sw, mtu 1500
        inet4 192.168.122.1/24
        route4 192.168.122.0/24

lo: unmanaged
        "lo"
        loopback (unknown), 00:00:00:00:00:00, sw, mtu 65536

virbr0-nic: unmanaged
        "virbr0-nic"
        tun, 52:54:00:E9:A0:A0, sw, mtu 1500

DNS configuration:
        servers: 172.16.168.2
        domains: localdomain
        interface: ens33

Use "nmcli device show" to get complete information about known devices and
"nmcli connection show" to get an overview on active connection profiles.

Consult nmcli(1) and nmcli-examples(5) manual pages for complete usage details.
```
列出所有的硬件设备以及其配置情况
#### nmcli connection命令

&#160;&#160;&#160;&#160;&#160;&#160;通常简写为nmcli c 命令，该命令是关于连接操作的。

- 为网卡添加网络配置

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;一个网卡可以添加多个配置，但每个网卡每次只能执行一个配置，添加配置命令如下：
```
nmcli c add type ethernet con-name ethX ifname ethX ipv4.addr 192.168.1.100/24 ipv4.gateway 192.168.1.1 ipv4.method manual

- type：添加类型
- con-name: 配置名称
- ifname: 网卡名称
- ipv4.addr: ipv4地址
- ipv4.gateway: ipv4网关
- ipv4.method: ipv4网络方式(手动、自动获取)
```
在centos上添加：
```
$ nmcli c add type ethernet con-name ens33-copy ifname ens33 ipv4.addr 192.168.1.100/24 ipv4.gateway 192.168.1.1 ipv4.method manual
```
使用 nmcli c查询详情：
```
$ nmcli c
NAME        UUID                                  TYPE      DEVICE 
ens33       c836911c-237c-46e1-ad05-e39d73af870c  ethernet  ens33  
virbr0      282772fc-e9e5-4280-9df8-602d950a612d  bridge    virbr0 
ens33-copy  10a24608-6849-4599-bbb3-326b54c9c530  ethernet  --     
```
可以发现 TYPE类型中有一个刚刚创建的ens33-copy

使用nmcli查询ens33网卡的状态：
```
$ nmcli

ens33: connected to ens33
        "Intel 82545EM"
        ethernet (e1000), 00:0C:29:E6:31:65, hw, mtu 1500
        ip4 default
        inet4 172.16.168.128/24
```
ip为92.168.85.133
- 开启与停止配置

&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;刚刚在ens33网卡上创建了ens33-copy配置，此时ens33网卡有两个配置，配置名分别为ens33和ens33-copy。使用nmcli c up <配置名> 和 nmcli c down <配置名>启动与停止配置。当一张网卡有两个配置时，如果停止其中正在使用的配置，会默认启动停止前为使用的配置。
```
$ nmcli c down ens33
$ nmcli c

NAME        UUID                                  TYPE      DEVICE 
ens33-copy  10a24608-6849-4599-bbb3-326b54c9c530  ethernet  ens33   
virbr0      282772fc-e9e5-4280-9df8-602d950a612d  bridge    virbr0 
ens33       c836911c-237c-46e1-ad05-e39d73af870c  ethernet  --  
```
可以看到关闭ens33配置后ens33网卡自动切换到ens33-copy上
```
$ nmcli c up ens33
$ nmcli c

NAME        UUID                                  TYPE      DEVICE 
ens33       c836911c-237c-46e1-ad05-e39d73af870c  ethernet  ens33  
virbr0      282772fc-e9e5-4280-9df8-602d950a612d  bridge    virbr0 
ens33-copy  10a24608-6849-4599-bbb3-326b54c9c530  ethernet  --     

```
ens33网卡切换回ens33配置

- 删除网络配置
```
$ nmcli c delete ens33-copy
$ nmcli c

NAME    UUID                                  TYPE      DEVICE 
ens33   c836911c-237c-46e1-ad05-e39d73af870c  ethernet  ens33  
virbr0  282772fc-e9e5-4280-9df8-602d950a612d  bridge    virbr0 

```
- 修改网络(非交互式)
```
$ nmcli c modify ens33 ipv4.addr '192.168.85.133/24'
$ nmcli c

ens33: connected to ens33
        "Intel 82545EM"
        ethernet (e1000), 00:0C:29:E6:31:65, hw, mtu 1500
        ip4 default
        inet4 192.168.85.133/24

```
- 修改网络(交互式)
```
$nmcli c edit ens33

===| nmcli interactive connection editor |===

Editing existing '802-3-ethernet' connection: 'ens33'

Type 'help' or '?' for available commands.
Type 'print' to show all the connection properties.
Type 'describe [<setting>.<prop>]' for detailed property description.

You may edit the following settings: connection, 802-3-ethernet (ethernet), 802-1x, dcb, sriov, ethtool, match, ipv4, ipv6, tc, proxy

nmcli> goto ipv4.address

nmcli ipv4.addresses> change

Edit 'addresses' value: 192.168.85.134/24

Do you also want to set 'ipv4.method' to 'manual'? [yes]: yes

nmcli ipv4.addresses> back

nmcli ipv4> save
Connection 'ens33' (c836911c-237c-46e1-ad05-e39d73af870c) successfully updated.

nmcli ipv4> activate

```
 - 对硬件设备的操作

```
$ nmcli d show

GENERAL.DEVICE:                         ens33
GENERAL.TYPE:                           ethernet
GENERAL.HWADDR:                         00:0C:29:E6:31:65
GENERAL.MTU:                            1500
GENERAL.STATE:                          100 (connected)
GENERAL.CONNECTION:                     ens33
GENERAL.CON-PATH:                       /org/freedesktop/NetworkManager/ActiveC>
WIRED-PROPERTIES.CARRIER:               on
IP4.ADDRESS[1]:                         192.168.85.133/24
IP4.GATEWAY:                            --
IP4.ROUTE[1]:                           dst = 192.168.85.0/24, nh = 0.0.0.0, mt>
IP6.ADDRESS[1]:                         fe80::ce5b:ae6b:9954:1bf0/64
IP6.GATEWAY:                            --
IP6.ROUTE[1]:                           dst = fe80::/64, nh = ::, mt = 100
IP6.ROUTE[2]:                           dst = ff00::/8, nh = ::, mt = 256, tabl>

$ nmcli d connect ethX --连接网卡设备
$ nmcli d disconnect ethX --与网卡设备断开连接

以上两条命令分别对应的就是图形界面里的启用网卡设备与禁用网卡设备
```

### nmtui

&#160;&#160;&#160;&#160;&#160;&#160;这个就是对应的ternimal UI,个人比较倾向与这种配置方法，命令行有时候记不住命令，使用这个完美解决健忘症的烦恼。
```
$ nmtui
```


选择Edit a connection:


再选择编辑ens33:

添加一条ens33网卡的配置ens33-copy:

可以看到可添加的类型很多，还有网卡bound,如果没有这个配置，采用配置文件配置bound是相当麻烦的：

按esc后退选择 Activate a connection:

打星号为已经选择的配置

在Centos7中打开网络需要修改配置文件，再重启网络，使用nmtui只需要选择activate就可以使用网络，不再需要繁琐的编辑文件。使用nmtui激活ens33后打开ifcfg-ens33显示ONBOOT=no，并不是yes
```
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=c836911c-237c-46e1-ad05-e39d73af870c
DEVICE=ens33
ONBOOT=no

```
### (四) 总结

经过短时间的学习，个人觉得networkmanager的学习成本相比较于学习修改配置文件然后再重启网络服务而言是不高的，而且nmtui 简直就是神器，可以直接在terminal进行使用，所以桌面配置和web配置就不做介绍了。再次感谢redhat提供了该软件并进行大力推广。




