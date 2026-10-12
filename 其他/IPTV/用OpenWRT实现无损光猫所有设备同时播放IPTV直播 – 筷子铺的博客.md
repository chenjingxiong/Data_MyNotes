---
title: "用OpenWRT实现无损光猫所有设备同时播放IPTV直播 – 筷子铺的博客"
source: "https://kzpu.com/archives/5519.html"
author:
  - "[[admin                                                         766RSS订阅]]"
  - "[[admin]]"
  - "[[766]]"
  - "[[RSS订阅]]"
published: 2023-10-06
created: 2025-09-01
description:
tags:
  - "clippings"
---
- 发表于： [2023年10月6日 2024年03月15日](https://kzpu.com/archives/5519.html)
- 分类： [分享](https://kzpu.com/archives/category/%e5%88%86%e4%ba%ab), [学习笔记](https://kzpu.com/archives/category/%e5%ad%a6%e4%b9%a0%e7%ac%94%e8%ae%b0)

### 前情提要  

博主家用的是广东电信的宽带，送了一个IPTV的机顶盒，平时放在老人房间播放。时不时的总能在网上看到有人折腾IPTV机顶盒和光猫，实现多设备同时观看IPTV的直播，博主我也折腾过多次，由于都要进去光猫里进行设置，担心搞的不好，把IPTV搞坏了，老人家看不了电视，又被说。但是，我还是收集了各种单播组播的信号源，就等有一天能实现多设备同时观看IPTV。

机会来了。国庆期间，在油管上刷到一哥们无损光猫，也就是光猫不做任何更改，利用OpenWRT来接管机顶盒的办法，实现IPTV信号源的转发，说干就干，一次成功，记录在此。

广东电信测试成功，其它运营商或地区，也可以试一下，是否能成功。

油管的视频：https://www.youtube.com/watch?v=z4gWgJPwHX4

### 功能实现  

1，光猫不做任何更改

2，原始的IPTV盒子照样可以看直播

3，内网所有设备都可以同时看直播，互不干涉

4，内网看直播不占用宽带流量，走IPTV专线

### 搭建环境及条件  

1，家里能正常上网，IPTV盒子能正常观看电视。

2，路由器至少空一个口。

3，OpenWRT的固件路由器一台。如果你是软路由，则空一个口出来接光猫的iTV。

4，也就是说，要有两台路由器，一台是平时拨号上网的，一台是OpenWRT的固件。如果你是软路由，只需一台就行了。

### 准备工作  

1，打开IPTV盒子，看一下拨号得到的IP地址，拍照或记录，IP地址和网关，后面会用到。如下图

[![](https://kzpu.com/wp-content/uploads/2023/10/2023100602.png)](https://kzpu.com/wp-content/uploads/2023/10/2023100602.png)

2，IPTV机顶盒底部，记录mac地址，后面会用到。

![](https://kzpu.com/wp-content/uploads/2023/10/2023100603.png)

### 连线图  

OpenWRT连接光猫的iTV接口，就是连iptv机顶盒的那个口。在OpenWRT里面这个口要直通，OpenWRT的另一个口，直接连路由器的LAN口即可。

如果你有软路由，直接直通一个网口出来，接光猫的iTV也是可以的。

博主用了一个8口交换机接在光猫的iTV口上，然后交换机接OpenWRT直通口、接原始的IPTV机顶盒，也就是说，把iTV多扩了几个，方便原始的IPTV盒子使用。

[![](https://kzpu.com/wp-content/uploads/2023/10/2023100601.png)](https://kzpu.com/wp-content/uploads/2023/10/2023100601.png)

如果是多网口的软路由，可以直接把iTV插到软路由的某一个网口上即可，然后在软路由的OpenWRT里进行设置。

### OpenWRT设置  

1，新建一个接口

[![](https://kzpu.com/wp-content/uploads/2023/10/2023100604.png)](https://kzpu.com/wp-content/uploads/2023/10/2023100604.png)

2，新建接口名填IPTV，设置见下图。

协议：静态地址

设备：这里选直通的那个网口，也就是接iTV的那个口

开机自动运行：打勾

IPv4地址：填入之前我们拍的机顶盒的那个IP地址

IPv4网关：填入之前我们拍的机顶盒的那个IP网关

[![](https://kzpu.com/wp-content/uploads/2023/10/2023100605.png)](https://kzpu.com/wp-content/uploads/2023/10/2023100605.png)

高级设置里面，把使用网关跃点改为200

![](https://kzpu.com/wp-content/uploads/2023/10/2024030501.png)

3，防火墙这里，选择IPTV防火墙

[![](https://kzpu.com/wp-content/uploads/2023/10/2024030502.png)](https://kzpu.com/wp-content/uploads/2023/10/2024030502.png)

4，DHCP这里也不用设置，如果有请关闭。

[![](https://kzpu.com/wp-content/uploads/2023/10/2023100607.png)](https://kzpu.com/wp-content/uploads/2023/10/2023100607.png)

5，编辑直通的那个网口

[![](https://kzpu.com/wp-content/uploads/2023/10/2023100608.png)](https://kzpu.com/wp-content/uploads/2023/10/2023100608.png)

6，在直通网口的mac地址这里，填入机顶盒的mac地址

[![](https://kzpu.com/wp-content/uploads/2023/10/2023100609.png)](https://kzpu.com/wp-content/uploads/2023/10/2023100609.png)

7，在OpenWRT安装udpxy插件，并进行以下设置

启用：打勾

刷新：打勾

状态：打勾

绑定IP/接口：填写本OpenWRT的地址，我这个的地址是10.0.0.22，所以就填这个

端口：任何设置，默认也可以

源IP/接口：填写之前记录的机顶盒的IP地址

[![](https://kzpu.com/wp-content/uploads/2023/10/2023100610.png)](https://kzpu.com/wp-content/uploads/2023/10/2023100610.png)

8，打开：http://10.0.0.22:4022/status

看一直转发状态，可以看到，一台设备已连上了，并且有速度显示。

![](https://kzpu.com/wp-content/uploads/2023/10/2023100611.png)

9，整理IPTV的源

github上最新最快的rtp源，广东电信：https://github.com/Tzwcard/ChinaTelecom-GuangdongIPTV-RTP-List

这里的源是rtp的，我们还需要稍做更改

#EXTINF:-1 tvg-name="广东卫视" group-title="IPTV-广东电视台",广东卫视高清
rtp://239.77.1.19:5146

更改为

#EXTINF:-1 tvg-name="广东卫视" group-title="IPTV-广东电视台",广东卫视高清
http://10.0.0.22:4022/udp/239.77.1.19:5146

然后保存为m3u格式，这里提供一份我整理去重的版本：[GuangdongIPTV\_http\_udpxy\_HD](https://kzpu.com/wp-content/uploads/2023/10/GuangdongIPTV_http_udpxy_HD.zip)

成功！！