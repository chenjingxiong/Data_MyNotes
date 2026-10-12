---
title: "PVE中istoreos的LXC模板安装方法"
source: "http://www.198484.com/?read-163.html"
author:
published:
created: 2025-09-17
description:
tags:
  - "clippings"
---
1. [折腾笔记](http://www.198484.com/?list-1.html)
2. [PVE中istoreos的LXC模板安装方法](http://www.198484.com/?read-163.html "PVE中istoreos的LXC模板安装方法")

[![](http://www.198484.com/view/img/avatar.png)](http://www.198484.com/?user-1.html)

[admin](http://www.198484.com/?user-1.html) 1.6K+

---

使用PVE自带的功能将istoreos的img镜像文件转换为LXC的专用CT模板之后，是无法通过WEB UI进行创建的，会报错，无法创建成功，只能通过命令进行创建。

命令：pct create 102 local:vztmpl/istoreos.rootfs.tar.gz --rootfs local:32 --ostype unmanaged --hostname openwrt-lxc --arch amd64 --cores 2 --memory 2048 --swap 2048 -net0 bridge=vmbr0,name=eth0

pct create就是创建命令

102是创建的虚拟机ID

local:vztmpl/istoreos.rootfs.tar.gz就是模板的路径

istoreos.rootfs.tar.gz就是模板名称

\--rootfs就是格式

local:32是指虚拟机磁盘存放的位置，local如果没有合并的情况下应该是local-lvm，32就是指磁盘的大小，单位是G

\--ostype unmanaged操作系统类型未指定

\--hostname openwrt-lxc虚拟机的名称，或者说是主机名称，这里设置的是openwrt-lxc

\--arch amd64这个不用多说，都知道是64位

\--cores 2核心数量2

\--memory 2048内存容量2048，也就是2G，根据情况自定义

\--swap 2048交换空间2048，也是2G，根据情况自定义

\-net0 bridge=vmbr0,name=eth0网卡桥接到vmbr0，网卡名称eth0

执行完创建命令之后，先不要启动容器，需要进行一些设置。

开机自启动，打开

功能，按需要选择

修改容器配置文件：

命令：vi /etc/pve/lxc/容器ID.conf

添加如下内容，以防用到tun

lxc.cgroup2.devices.allow: c 10:200 rwm  
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file

启动容器之后，配置网络

vi /etc/config/network

如果网络是DHCP的话就需要改为静态

修改：

config interface 'lan'  
option device 'br-lan'  
option proto 'dhcp'

改为：

config interface 'lan'  
option device 'br-lan'  
option proto 'static'  
option ipaddr '192.168.2.11'  
option netmask '255.255.255.0'

保存之后，重启即可使用192.168.2.11访问iStoreOS了。

转载请注明原文地址:http://www.198484.com/?read-163.html

---

## PVE

**最新回复** (0)