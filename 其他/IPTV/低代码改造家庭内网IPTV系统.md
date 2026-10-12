---
title: 低代码改造家庭内网IPTV系统
source: https://zhuanlan.zhihu.com/p/1901951656851113437
author:
  - "[[IC民工​浙江大学 电子科学与技术博士]]"
published:
created: 2025-09-02
description: 一、动机名下有三条电信宽带，虽然资费相当划算，但只有自己住所的宽带赠送了IPTV服务，父母、岳父母的住所没有IPTV。其实，老年人才是看电视的主力军，咱们做儿女的必须服务好！ 经过调研，有以下几种方案。 A：…
tags:
  - clippings
profileName: 飞牛的wordpress
postId: "13"
postType: post
categories:
  - 1
---
![低代码改造家庭内网IPTV系统](https://picx.zhimg.com/70/v2-29313d1c3c514c2c4ac2b34a4c966979_1440w.avis?source=172ae18b&biz_tag=Post)

低代码改造家庭内网IPTV系统

目录

收起

一、动机

二、原有网络架构

三、改造方案

需求

原理

架构

四、搭建流程

1\. 连接OpenWRT路由器

2\. 组建网桥

3\. 安装必要的工具

4\. 抓包

5\. 创建接口

6\. 配置udpxy

7\. 制作节目源

8\. 导入群晖电视

五、贡献

修订记录

![](https://picx.zhimg.com/v2-0893dd117ca2e0ff97c7e5d2aee5cf1d.png?source=7e7ef6e2&needBackground=1)

图1 原有的网络拓扑

[所属专栏 · 2025-05-17 15:00 更新](https://zhuanlan.zhihu.com/c_1753095854939213824)

[![](https://picx.zhimg.com/v2-c5be1695771c4f9b442b5bde56e5e8e0_720w.jpg?source=172ae18b)](https://zhuanlan.zhihu.com/c_1753095854939213824)

[私有云瞎折腾](https://zhuanlan.zhihu.com/c_1753095854939213824)

[

IC民工

浙江大学 电子科学与技术博士

15 篇内容 · 130 赞同

](https://zhuanlan.zhihu.com/c_1753095854939213824)

[

最热内容 ·

docker编排+低代码部署Headscale VPN打通大内网

](https://zhuanlan.zhihu.com/c_1753095854939213824)

编辑于 2025-05-21 16:27・浙江[IPTV](https://www.zhihu.com/topic/19557219)[OpenWrt](https://www.zhihu.com/topic/19653984)[家庭网络优化](https://www.zhihu.com/topic/28823000)

![](https://picx.zhimg.com/v2-d66392b31e64471b9b99495920239e8f_l.jpg?source=32738c0c&needBackground=1)

理性发言，友善互动

15 条评论

默认

最新

[yc znone](https://www.zhihu.com/people/500b5811aefaf3ed9c482269af021209)

VPN早就被盯上了，再说你的上行带宽够吗？

05-05 · 广西

[IC民工](https://www.zhihu.com/people/4069184dffe8801daff0bd6e75086bd3)

作者

自建自用的Headscale VPN，只为打通自家内网，它盯不上也管不了。

咱不是为了盈利，自家看电视，算上各种设备并行访问最多5路，理论上每路1 MB/s也就是8 Mbps，总共40 Mbps。

我的宽带下行带宽200 Mbps，上行带宽50 Mbps，理论上带宽足够；实际体验也无问题。

05-05 · 浙江

![](https://picx.zhimg.com/v2-d66392b31e64471b9b99495920239e8f_l.jpg?source=32738c0c&needBackground=1)

理性发言，友善互动