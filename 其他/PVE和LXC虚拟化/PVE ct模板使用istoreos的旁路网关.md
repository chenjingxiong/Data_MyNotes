---
title: "PVE ct模板使用istoreos的旁路网关"
source: "https://www.nodeseek.com/post-147793-1"
author:
  - "[[NodeSeek]]"
published: 2024-08-17
created: 2025-09-17
description: "OpenWRT常规img.gz转化为PVELXCCT模版rootfs.tar.gz以及简单LXCOP的安装与基本设置以旁路网关模式为例转CT模板下载img页面https://fw.koolcenter.com/iStoreOS/x86_64_efi/复制最新的去命令行下载wgethttps://fw0.koolce"
tags:
  - "clippings"
---
OpenWRT常规img.gz转化为PVE LXC CT模版rootfs.tar.gz  
以及简单LXC OP的安装与基本设置以旁路网关模式为例

## 转CT模板

下载img页面

```
https://fw.koolcenter.com/iStoreOS/x86_64_efi/
```

复制最新的去命令行下载

```
wget https://fw0.koolcenter.com/iStoreOS/x86_64_efi/istoreos-22.03.7-2024080210-x86-64-squashfs-combined-efi.img.gz
gzip -d istoreos-22.03.7-2024080210-x86-64-squashfs-combined-efi.img.gz
mv istoreos-22.03.7-2024080210-x86-64-squashfs-combined-efi.img istoreos.img
mkdir /mnt/openwrt
modprobe nbd #加载NBD模块使磁盘镜像可映射到本地使用
qemu-nbd -c /dev/nbd0 -f raw istoreos.img
```

查看所挂载的分区  
`lsblk -f /dev/nbd0`  
![31.png](https://roim-picx-bpc.pages.dev/rest/rv4wkHK.png)  
4个分区。选择2号分区挂载

```
mount /dev/nbd0p2 /mnt/openwrt/ # 挂载指定镜像分区到指定文件夹
```

进入挂载的目录，查看文件  
![32.png](https://roim-picx-bpc.pages.dev/rest/30owkHK.png)  
接下来就是打包

```
tar -czvf /var/lib/vz/template/cache/openwrt.rootfs.tar.gz *
```

打包完，卸载镜像

```
cd /mnt
umount /mnt/openwrt
qemu-nbd -d /dev/nbd0
```

## 建立CT容器

pve shell运行

```
pct create 100 local:vztmpl/openwrt.rootfs.tar.gz --rootfs local-lvm:1 --ostype unmanaged --hostname Openwrt-LXC --arch amd64 --Cores 2 --memory 1024 --swap 0 -net0 bridge=vmbr0,name=eth0
```

创建2h1g的100号容器。

## 设置

修改开机自启动  
![33.png](https://roim-picx-bpc.pages.dev/rest/SLcALHK.png)  
修改conf

```
vi /etc/pve/lxc/100.conf
```

末尾添加2行代码，目的为了增加tun设备映射支持

```
lxc.cgroup2/devices/allow: c 10:200 rwm
lxc.mount.entry: /dev/net/tun /dev/net/tun none bind,create-file
```

保存退出。

## 初始化

打开pve 容器点击控制台 开机。很快  
![34.png](https://roim-picx-bpc.pages.dev/rest/flsaLHK.png)  
再回车  
修改网络配置

```
vi /etc/config/network
```

修改lan字段如下  
![35.png](https://roim-picx-bpc.pages.dev/rest/2P3BLHK.png)  
保存退出

```
service network restart
```

重启网络或者reboot重启

## op初始化

打开192.168.31.2登录，密码默认password

1. 初始化网络接口  
	点开左边网络-接口  
	编辑lan  
	![36.png](https://roim-picx-bpc.pages.dev/rest/trWbLHK.png)  
	常规设置-ipv4网关填好 192.168.31.1  
	![37.png](https://roim-picx-bpc.pages.dev/rest/D0obLHK.png)  
	目的是为了访问主路由

高级设置-使用自定义的DNS服务器，填写 192.168.31.1和8.8.8.8

（223.5.5.5和119.29.29.29

目的是为了保证一些插件联网，添加阿里和腾讯的dns，避免主路由dhcp没有给他分配dns。）笔者没有使用两个云的dns。

关闭op的dhcp：  
![39.png](https://roim-picx-bpc.pages.dev/rest/AAH1LHK.png)  
选中 忽略此接口

保存后

再保存并应用，才能生效  
![40.png](https://roim-picx-bpc.pages.dev/rest/KIT1LHK.png)

## bug

如果发现op无法联网。

shell修改/etc/resolv.conf

1. 备份  
	mv /etc/resolv.conf /etc/resolv.conf.bak
2. 自建conf  
	添加8.8.8.8到第一行保存
```
vi /etc/resolv.conf
#添加以下内容
nameserver 8.8.8.8
nameserver 223.5.5.5
```

保存。返回浏览器就有网络了。

因为这个文件有时候会被还原回去，又会断网，我们这样自建的文件权限不同，不会被更新

1. 确认
```
root@Openwrt-LXC:/etc# ls -l /etc/resolv.conf*
-rw-r--r--    1 root     root            40 Aug 15 23:46 /etc/resolv.conf
lrwxrwxrwx    1 root     root            16 Jul 26 14:23 /etc/resolv.conf.bak -> /tmp/resolv.conf
root@Openwrt-LXC:/etc# cat /etc/resolv.conf
nameserver 8.8.8.8
nameserver 223.5.5.5
root@Openwrt-LXC:/etc# cat /etc/resolv.conf.bak
search lan
nameserver 127.0.0.1
nameserver ::1
```

如上/etc/resolv.conf.bak会被还原，修改了没用。

暂时解决断网的dns问题。

## 学习

### 安装pw2（可跳过，笔者卸载了）

参考教程

```
https://github.com/AUK9527/Are-u-ok/tree/main/x86
```

istoreos自带的istore没有自带passwall，去往github下载的Are-u-ok下载。

选择x86\_64平台

下载passwall2

这是一个run后缀的包，打开istore，选择手动安装

选择run包，就会自动跳出命令行界面进行安装

等待右上角原点变绿之后就安装完成

### 配置passwalll2（可跳过，笔者卸载了）

在istoreos后台的服务界面可以找到刚刚安装的passwall2  
![44.png](https://roim-picx-bpc.pages.dev/rest/FdkClHK.png)  
选择节点订阅，选择添加

在该页面中粘贴机场网址后,修改最下方ua为：

```
Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.122 Safari/537.36
```

点击保存并应用  
![45.png](https://roim-picx-bpc.pages.dev/rest/grIkmHK.png)

测试

pw2首页，开启开关，选择节点保存并应用，再点击测试百度谷歌github  
![46.png](https://roim-picx-bpc.pages.dev/rest/lfWLmHK.png)

同时打开op命令行，测试，  
百度可以，谷歌不行，github可以。

可能面板检测百度有问题。

**后续测试，其他设备使用op的网关科学上网，也出现百度打不开，勾了ip伪装，加了防火墙规则，关闭了无效数据包（本来也关闭）等等，实在没辙了，先换用openclash吧**

### 卸载pw2（可跳过）

```
cd /etc/istore/uci-defaults_bak
ls #查看有的名字
is-opkg remove 'luci-i18n-passwall2-zh-cn'
is-opkg remove '*passwall2*'
```

回到浏览器面板，服务中已经没有pw2了

## 拥抱openclash

1. 下载地址

`https://github.com/vernesong/OpenClash/releases`

luci-app-openclash\_0.46.014-beta\_all.ipk 下载备用

1. 安装依赖包  
	进入软路由终端里复制下面的命令安装OpenClash需要的依赖包，可以一条条复制执行，也可以整个复制粘贴按回车执行
```
#iptables
opkg update
opkg install coreutils-nohup bash iptables dnsmasq-full curl ca-certificates ipset ip-full iptables-mod-tproxy iptables-mod-extra libcap libcap-bin ruby ruby-yaml kmod-tun kmod-inet-diag unzip luci-compat luci luci-base
```
1. 安装OpenClash  
	进入软路由系统->系统->软件包->上传软件包，选择步骤1下载的软件包，点击上传。

## 配置OpenClash

1. 设置fake-ip模式  
	进入软路由系统->服务->OpenClash-插件设置

点击中间下方 切换faka-ip模式后

选中 使用meta内核 保存应用  
![51.png](https://roim-picx-bpc.pages.dev/rest/vkiZPHK.png)

1. 更新版本内核  
	进入OpenClash->插件设置->版本更新，点击一键检查更新  
	![52.png](https://roim-picx-bpc.pages.dev/rest/6eCzPHK.png)
2. 开启GEO数据库订阅  
	进入OpenClash->插件设置->GEO数据库订阅 保存-应用  
	![53.png](https://roim-picx-bpc.pages.dev/rest/B7azPHK.png)
3. 切换(更新)Dashboard版本  
	进入OpenClash->插件设置->外部控制

切换前  
![54.png](https://roim-picx-bpc.pages.dev/rest/y4HzPHK.png)

切换后  
![55.png](https://roim-picx-bpc.pages.dev/rest/SELzPHK.png)  
保存 应用

1. 修改github地址  
	进入OpenClash->覆写设置->常规设置  
	![56.png](https://roim-picx-bpc.pages.dev/rest/0mZzPHK.png)  
	保存-应用
2. 配置订阅  
	进入OpenClash->配置订阅

\->新增  
![57.png](https://roim-picx-bpc.pages.dev/rest/O4HapHK.png)  
如上图，添加订阅地址，然后下面的选项修改后，保存配置。

再点击更新配置。

## 订阅bug

首页启动一直失败 测试配置失败

查看运行日志  
![65.png](https://roim-picx-bpc.pages.dev/rest/XrqhpHK.png)  
![63.png](https://roim-picx-bpc.pages.dev/rest/SXxHpHK.png)  
报错找不到proxy

参考网友报错反馈

```
https://github.com/vernesong/OpenClash/issues/874
```

排查问题  
![64.png](https://roim-picx-bpc.pages.dev/rest/QEphpHK.png)  
配置管理页面查看配置（和修改

对比本地clashverge软件订阅的信息  
![62.png](https://roim-picx-bpc.pages.dev/rest/NcBhpHK.png)  
我们发现实际是面板的配置没有获取全，proxy和group没有信息。  
复制两个部分替换回去，然后保存更新

回到首页启动，完美启动  
![66.png](https://roim-picx-bpc.pages.dev/rest/Nz37pHK.png)

1. 添加规则，设置指定域名不走代理  
	进入OpenClash->覆写设置->规则设置

开启自定义规则  
![58.png](https://roim-picx-bpc.pages.dev/rest/c2B0pHK.png)  
填写规则

比如我要将baidu.com加入规则，并让他不走代理。

```
- DOMAIN-SUFFIX,baidu.com,DIRECT
```

![59.png](https://roim-picx-bpc.pages.dev/rest/88M0pHK.png)

```
规则有很多种：
##- SCRIPT,quic,REJECT #shortcuts rule
##- SCRIPT,time-limit,REJECT #shortcuts rule
##- PROCESS-NAME,curl,DIRECT #匹配路由自身进程(curl直连)
##- DOMAIN-SUFFIX,google.com,Proxy #匹配域名后缀(交由Proxy代理服务器组)
##- DOMAIN-KEYWORD,google,Proxy #匹配域名关键字(交由Proxy代理服务器组)
##- DOMAIN,google.com,Proxy #匹配域名(交由Proxy代理服务器组)
##- DOMAIN-SUFFIX,ad.com,REJECT #匹配域名后缀(拒绝)
##- IP-CIDR,127.0.0.0/8,DIRECT #匹配数据目标IP(直连)
##- SRC-IP-CIDR,192.168.1.201/32,DIRECT #匹配数据发起IP(直连)
##- DST-PORT,80,DIRECT #匹配数据目标端口(直连)
##- SRC-PORT,7777,DIRECT #匹配数据源端口(直连)
但主要用到的是域名或者关键字
##- DOMAIN-SUFFIX,google.com,Proxy #匹配域名后缀(交由Proxy代理服务器组)
##- DOMAIN-KEYWORD,google,Proxy #匹配域名关键字(交由Proxy代理服务器组)
Proxy代表要代理
DIRECT代表不走代理
REJECT代表拒绝
根据自己的需求来配置规则
```

保存-应用（在最下方）

查看规则是否生效

运行状态-YACD控制面板-规则，可以查看上面设置的规则  
![67.png](https://roim-picx-bpc.pages.dev/rest/U537pHK.png)

## 试试旁路网关

有两种办法，一种是在设备上手动设置网关为软路由的IP地址，另一种是在主路由上面把 DHCP 默认网关设置为软路由的 IP。为了稳定，我用手动配置的办法。

1. 先测试没有修改，自动dhcp  
	![28.png](https://roim-picx-bpc.pages.dev/rest/FkFSKHK.png)  
	可以看到，直接回到主路由31.1访问
2. 修改网关和dns给旁路网关  
	![29.png](https://roim-picx-bpc.pages.dev/rest/DhlSKHK.png)
3. 测试效果  
	![30.png](https://roim-picx-bpc.pages.dev/rest/AWtsKHK.png)  
	可以看到第一跳、第二跳、第三跳分别是旁路网关、我的硬路由和光猫，然后是联通的网络，最后到达目标地址。

## 国内网站访问速度变慢了？

关掉本地dns劫持

打开面板  
![69.png](https://roim-picx-bpc.pages.dev/rest/XjbsQHK.png)

可以自己感觉下速度。