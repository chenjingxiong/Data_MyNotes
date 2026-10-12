---
title: "2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1__什么值得买"
source: "https://post.smzdm.com/p/akle62mk/"
author:
published: 2024-03-14
created: 2025-09-17
description: "PVE镜像下载1、在浏览器输入以下网址下载pve的iso安装镜像。https://www.proxmox.com/en/downloads2、点击d"
tags:
  - "clippings"
---
当前位置：

[社区首页](https://post.smzdm.com/)

文章详情

![](https://a.zdmimg.com/202403/14/65f2693498fe34288.png_fo742.jpg)

## 2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1

## PVE镜像下载

1、在浏览器输入以下网址下载pve的iso安装镜像。

[https://www.proxmox.com/en/downloads](https://www.proxmox.com/en/downloads "https://www.proxmox.com/en/downloads")

2、点击download进行下载。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://qnam.smzdm.com/202403/14/65f268ad44ffe6177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_2/)

3、最新的pve8.1-2镜像大小为1.2GB，接下来制作引导U盘。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://qnam.smzdm.com/202403/14/65f268ad54ed46177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_3/)

## 引导U盘制作

1、首先下载写盘工具，本文使用的写盘工具为Rufus，下载地址为 [https://rufus.ie/downloads/](https://rufus.ie/downloads/ "https://rufus.ie/downloads/") [，选择最新版本的下载。](https://rufus.ie/downloads/ "，选择最新版本的下载。")

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://am.zdmimg.com/202403/14/65f268ad108836177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_4/)

2、下载完成后直接双击运行，无需安装，设备中选择U盘，引导类型选择PVE镜像，然后点击开始，注意U盘中的原有数据会被格式化，重要数据记得备份。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://qnam.smzdm.com/202403/14/65f268ad03f996177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_5/)

3、当状态为准备就绪时，表示U盘写入成功，拔下U盘插入需要安装的机器的USB接口中（ [台式机](https://www.smzdm.com/fenlei/taishiji/) 推荐后置USB接口）

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://am.zdmimg.com/202403/14/65f268ad8bf806177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_6/)

## PVE安装

1、打开主机电源，不断按键盘F12按键（不同主机的U启动按键不同，请自行百度或者询问卖家）进入PVE安装页面，选择第一个 Install Proxmox VE （Graphical) ，按回车。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://qnam.smzdm.com/202403/14/65f268ade00826177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_7/)

2、 页面会进入如下的跑代码阶段，跑码时间看机器配置，跑码期间不要动键盘鼠标。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://am.zdmimg.com/202403/14/65f268ad2c8bf6177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_8/)

3、进入下面页面安装正式开始，首先使用协议页面点击 I agree （我同意）。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://qnam.smzdm.com/202403/14/65f268add0af86177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_9/)

4、选择pve安装的 [硬盘](https://www.smzdm.com/fenlei/yingpan/) ，然后继续点击Next(下一步）。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://qnam.smzdm.com/202403/14/65f268ad45af46177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_10/)

5、设置国家、时区、键盘布局，然后点击Next(下一步）。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://am.zdmimg.com/202403/14/65f268ada340b6177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_11/)

6、设置密码、和邮箱，继续点击Next(下一步）。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://am.zdmimg.com/202403/14/65f268adabb426177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_12/)

7、设置网卡、域（保持默认即可）、PVE管理地址、网关、DNS等，继续点击Next(下一步）。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://am.zdmimg.com/202403/14/65f268ad60d686177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_13/)

8、在配置详情中检查配置信息是否有误，没有问题点击Install（安装）。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://qnam.smzdm.com/202403/14/65f268ad8fb786177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_14/)

9、移除U盘，然后点击Reboot重启机器。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://qnam.smzdm.com/202403/14/65f268adb54cd6177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_15/)

## 初始化

1、重启完成之后出现如下页面，表示安装成功，记录下屏幕中出现的ip地址，接下来的配置无需使用PVE主机显示，单个 [显示器](https://www.smzdm.com/fenlei/xianshiqi/) 的朋友可以将显示器切换到主机了。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://qnam.smzdm.com/202403/14/65f268ad6a91e6177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_16/)

2、在浏览器输入 [https://192.168.2.167:8006](https://192.168.2.167:8006/ "https://192.168.2.167:8006") [进入PVE配置页面，用户名为root，密码为在安装过程中设置的，语言为了方便设置为简体中文，然后点击登录。](https://192.168.2.167:8006/ "进入PVE配置页面，用户名为root，密码为在安装过程中设置的，语言为了方便设置为简体中文，然后点击登录。")

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://qnam.smzdm.com/202403/14/65f268ad614866177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_17/)

3、第一次登录会出现无效订阅不用管，直接点击确定，后续我们将通过设置去掉这个选项。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://qnam.smzdm.com/202403/14/65f268adca0e26177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_18/)

4、首先我们需要配置一下pve的dns来保证pve可以正确访问到网络，这里我使用了阿里云公共的DNS，223.5.5.5，223.6.6.6，然后点击ok。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://qnam.smzdm.com/202403/14/65f268adb183b6177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_19/)

5、使用pve自带的shell工具来ping百度，测试网络是否正常，出现正确的ping值表示外网访问通畅。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://qnam.smzdm.com/202403/14/65f268adac08b6177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_20/)

6、接下来还涉及到pve的换源、系统更新、直通等操作，为了方便小白和第一次接触到pve的朋友们，这里我们将使用第三方工具来实现。

## 优化（PVE\_Source）

pve\_source是老虎大佬开发的pve脚本（ [首发于X86派论坛](https://bbs.x86pi.cn/thread?topicId=20 "首发于X86派论坛") ），可以更方便的初始化PVE，无需使用复杂的命令，使用方法很简单，登录PVE，打开Shell粘贴如下命令。

`wget -q -O /root/pve_source.tar.gz 'https://bbs.x86pi.cn/file/topic/2023-11-28/file/01ac88d7d2b840cb88c15cb5e19d4305b2.gz' && tar zxvf /root/pve_source.tar.gz && /root/./pve_source` [![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://am.zdmimg.com/202403/14/65f268ad42eef6177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_21/)

1、脚本加载成功出现使用协议，输入Y，然后继续按回车。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://qnam.smzdm.com/202403/14/65f268ada44a96177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_22/)

2、进入脚本优化选项，脚本覆盖了PVE的绝大部分必备优化选项，需要使用那个直接输入数字然后按回车即可。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://qnam.smzdm.com/202403/14/65f268ad983436177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_23/)

## 更换 Proxmox VE 源

3、为了更方便了解脚本都干了什么，建议分步执行23456，输入数字2,然后再输入1更换 PVE 软件源 + Debian 源，分别输入列表前面的数字选择PVE版本和软件源镜像站（不清楚的直接都选择清华大学源就行）。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://qnam.smzdm.com/202403/14/65f268ad97bdb6177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_24/)

4、选择软件源以后继续按回车执行，脚本将自动执行更换源成功后会给出提示。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://qnam.smzdm.com/202403/14/65f268ade06866177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_25/)

5、按照以上操作继续完成PVE Ceph 源 、PVE LXC 仓库源 的更新操作（在Shell页面输入./pve\_source即可再次调出pve工具小技巧直接输入./pve然后按tab按键可自动补全后面）。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://am.zdmimg.com/202403/14/65f268adc82926177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_26/)

## 更新软件包系统

更新软件包有两种方法，一种是通过pve工具箱更新，另外一种因为在前面我们已经通过pve工具箱更新了软件源，所以直接在web页面更新也可以

1、点击pve节点→更新→刷新→升级，既可以完成最新软件包和组件的升级。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://am.zdmimg.com/202403/14/65f268adcfc7c6177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_27/)

2、通过pve工具升级，在Shell中输入./pve\_source，然后使用脚本3、4来更新软件包和系统。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://am.zdmimg.com/202403/14/65f268ad597dd6177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_28/)

## 设置系统DNS

1、通过web页面修改，点击节点→DNS→编辑。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://am.zdmimg.com/202403/14/65f268ad4c3966177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_29/)

2、通过pve工具箱修改。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://qnam.smzdm.com/202403/14/65f268adb13e46177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_30/)

## 去除无效订阅源提示

每次在登录pve的时候会弹出企业订阅提示，对于普通用户我们直接去除掉弹窗即可，登录PVE，在Shell窗口中输入./pve\_source,进入pve工具箱，输入6，系统会自动执行去除订阅脚本，脚本运行完成后请强制刷新浏览器缓存。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://qnam.smzdm.com/202403/14/65f268adedc636177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_31/)

## 修改PVE概要信息

1、pve默认概要中不会显示cpu温度、硬盘信息等，通过pve工具箱可以修改信息概要页面，使其信息显示更加完善。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://am.zdmimg.com/202403/14/65f268ada68286177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_32/)

2、登录PVE，在Shell窗口中输入./pve\_source,进入pve工具箱，按7进入pve概要信息定制向导，根据自身喜好选择合适的版本，这里我使用推荐方案1：高大全版本。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://am.zdmimg.com/202403/14/65f268adfdf576177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_33/)

3、修改改完成后可以看到概要信息中显示的非常全面，包括CPU温度核心占用情况，以及硬盘的温度写入量等都可以很直观的显示。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://qnam.smzdm.com/202403/14/65f268ad573756177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_34/)

## 虚拟机安装

pve安装虚拟机还是非常简单的，本篇我们将使用pve安装win10来演示一下安装过程，其他系统安装大同小异。

1、首先需要上传安装镜像，通过local(pve)→ISO镜像→上传，将win10镜像上传到pve中。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://qnam.smzdm.com/202403/14/65f268ad9a3cf6177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_35/)

2、回到pve首页，点击右上角的创建虚拟机，常规中自定义虚拟机的名称

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://am.zdmimg.com/202403/14/65f268ad587816177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_36/)

3、操作系统中选择windows10镜像，然后在类型中选择安装操作系统的类型，注意版本也要选择。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://am.zdmimg.com/202403/14/65f268adda64c6177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_37/)

4、系统保持默认即可。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://am.zdmimg.com/202403/14/65f268acb10ea8851.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_38/)

5、硬盘中总线选择sata，其他保持默认。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://qnam.smzdm.com/202403/14/65f268ad1b0046177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_39/)

6、CPU根据机器配置选择，插槽表示cpu个数单CPU机器输入1，核心根据CPU选择，类别使用HOST系统性能最大化。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://am.zdmimg.com/202403/14/65f268ad5b8536177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_40/)

7、 [内存](https://www.smzdm.com/fenlei/neicun/) 根据机器配置设置。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://qnam.smzdm.com/202403/14/65f268ad420c56177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_41/)

8、网络设置中选择兼容性最高的e1000即可。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://am.zdmimg.com/202403/14/65f268addaca66177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_42/)

9、最后检查配置没有，确认无误后点击完成。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://qnam.smzdm.com/202403/14/65f268adc58706177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_43/)

10、稍等几秒，机器创建成功。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://qnam.smzdm.com/202403/14/65f268adcbe226177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_44/)

11、点击控制台启动，按照正常windows系统安装流程安装即可。

[![2024年PVE8最新安装使用指南|新手入门|安装|优化|Proxmox VE 8.1](https://am.zdmimg.com/202403/14/65f268ad6cfa06177.png_e1080.jpg)](https://post.smzdm.com/p/akle62mk/pic_45/)

## 总结

pve的安装过程还是比较简单的和一般的操作系统安装过程相同，安装完成后通过pve-source工具优化即可，后续我也会出一篇通过命令修改的教程，下一篇会讲一下PVE的 [显卡](https://www.smzdm.com/fenlei/xianka/) 直通，并切在windows和群晖等系统中实现核显直通、解码等，本篇教程到此结束，咱们下一篇见。

作者声明本文无利益相关，欢迎值友理性交流，和谐讨论～

- [让我破费年度数码好物推荐](https://m.smzdm.com/theme/amql7kg/?zdm_feature=%7B%22fs%22%3A1%2C%22ta%22%3A1%7D)
	话题：让我破费年度数码好物推荐

文章很值，打赏犒劳作者一下

4 人已打赏

[![挂件](https://eimg.smzdm.com/202008/05/5f2a7a238c4df3220.png)](https://zhiyou.smzdm.com/member/8053713688/)

## [zwbcc](https://zhiyou.smzdm.com/member/8053713688/)

[

喜欢所有的数码产品，折腾是乐趣，不是目的，合作联系： satr0613

132文章 | 10爆料 | 9819粉丝

](https://zhiyou.smzdm.com/member/8053713688/)

生活家，数码领域作者

发文累计获赞4993，内容被4.1万人收藏

- [![](https://a.zdmimg.com/202504/17/6800b496f7acd9158.png_a320.jpg)](https://post.smzdm.com/p/aoq9oknn/)
	[部署本地对象存储S3服务器MinIO](https://post.smzdm.com/p/aoq9oknn/)
	1 1
	[部署本地对象存储S3服务器MinIO](https://post.smzdm.com/p/aoq9oknn/)
- [![](https://a.zdmimg.com/202507/08/686ca988ba0007374.jpg_a320.jpg)](https://post.smzdm.com/p/a640vgzg/)
	[记录北京联通fttr光宽带安装过程 （ur2s|套餐|改桥接|测试）](https://post.smzdm.com/p/a640vgzg/)
	10 27
	[记录北京联通fttr光宽带安装过程 （ur2s|套餐|改桥接|测试）](https://post.smzdm.com/p/a640vgzg/)
- [![](https://a.zdmimg.com/202508/09/689707012b8496204.png_a320.jpg)](https://post.smzdm.com/p/al4ewr4p/)
	[用 Proxmox Backup Server 备份 PVE 虚拟机|容器教程](https://post.smzdm.com/p/al4ewr4p/)
	3 4
	[用 Proxmox Backup Server 备份 PVE 虚拟机|容器教程](https://post.smzdm.com/p/al4ewr4p/)
- [![](https://a.zdmimg.com/202508/14/689def2754edc7212.png_a320.jpg)](https://post.smzdm.com/p/arl96w6q/)
	[飞牛同步正式上线｜支持多端文件同步｜Windows｜Mac](https://post.smzdm.com/p/arl96w6q/)
	0 0
	[飞牛同步正式上线｜支持多端文件同步｜Windows｜Mac](https://post.smzdm.com/p/arl96w6q/)
- [![](https://a.zdmimg.com/202508/25/68abe088482663923.jpg_a320.jpg)](https://post.smzdm.com/p/ax6wlgkw/)
	[飞牛首款全闪合作硬件NAS｜零刻ME mini上手体验](https://post.smzdm.com/p/ax6wlgkw/)
	2 3
	[飞牛首款全闪合作硬件NAS｜零刻ME mini上手体验](https://post.smzdm.com/p/ax6wlgkw/)

- [![](https://a.zdmimg.com/202509/17/68cab1aa1cb458172.png_a320.jpg)](https://post.smzdm.com/p/adop3dkd/)
	[2025超全推荐：领夹麦哪个品牌音质最好？领夹麦克风十大品牌推荐](https://post.smzdm.com/p/adop3dkd/)
	0 0
	[2025超全推荐：领夹麦哪个品牌音质最好？领夹麦克风十大品牌推荐](https://post.smzdm.com/p/adop3dkd/)
- [![](https://a.zdmimg.com/202509/17/68cab9588066a9438.jpg_a320.jpg)](https://post.smzdm.com/p/aeop0v9z/)
	[华为发布 “4+10+N”中小企业智能化方案，打通迈向智能世界“最后一公里”](https://post.smzdm.com/p/aeop0v9z/)
	0 0
	[华为发布 “4+10+N”中小企业智能化方案，打通迈向智能世界“最后一公里”](https://post.smzdm.com/p/aeop0v9z/)
- [![](https://a.zdmimg.com/202509/17/68cab970592e45992.jpg_a320.jpg)](https://post.smzdm.com/p/a0v94d5w/)
	[iPhone17Pro系性能超M4处理器 标准版ProMax测评](https://post.smzdm.com/p/a0v94d5w/)
	0 0
	[iPhone17Pro系性能超M4处理器 标准版ProMax测评](https://post.smzdm.com/p/a0v94d5w/)
- [![](https://a.zdmimg.com/202509/17/68caaedaea674142.jpg_a320.jpg)](https://post.smzdm.com/p/a2qm3z2p/)
	[孚能科技官宣！2027年推第三代固态电池，能量密度超500Wh/kg！](https://post.smzdm.com/p/a2qm3z2p/)
	0 0
	[孚能科技官宣！2027年推第三代固态电池，能量密度超500Wh/kg！](https://post.smzdm.com/p/a2qm3z2p/)
- [![](https://a.zdmimg.com/202509/17/68cab06cfabd52840.jpg_a320.jpg)](https://post.smzdm.com/p/adop3dod/)
	[大疆DJI Mini 5 Pro评测：1英寸底+主动避障，无人机中的六边形战士](https://post.smzdm.com/p/adop3dod/)
	0 0
	[大疆DJI Mini 5 Pro评测：1英寸底+主动避障，无人机中的六边形战士](https://post.smzdm.com/p/adop3dod/)
- [![](https://a.zdmimg.com/202509/12/68c43f5d4ff5a3982.png_a320.jpg)](https://post.smzdm.com/p/aeo4oml4/)
	[被罗永浩吐槽的西贝，一份炒鸡蛋卖43元，后厨用电磁炉炒菜](https://post.smzdm.com/p/aeo4oml4/)
	266 1197
	[被罗永浩吐槽的西贝，一份炒鸡蛋卖43元，后厨用电磁炉炒菜](https://post.smzdm.com/p/aeo4oml4/)
- [![](https://a.zdmimg.com/202509/12/68c2fb60733876794.png_a320.jpg)](https://post.smzdm.com/p/a3rp092k/)
	[贵不贵 西贝晒出罗永浩消费账单：15个菜收费830元](https://post.smzdm.com/p/a3rp092k/)
	133 1039
	[贵不贵 西贝晒出罗永浩消费账单：15个菜收费830元](https://post.smzdm.com/p/a3rp092k/)
- [![](https://a.zdmimg.com/202509/15/68c7626c0741a6167.jpg_a320.jpg)](https://post.smzdm.com/p/apwoe6ex/)
	[禁止参观后厨！转基因大豆油用于员工餐 西贝称我们不好就没有干净的了](https://post.smzdm.com/p/apwoe6ex/)
	110 769
	[禁止参观后厨！转基因大豆油用于员工餐 西贝称我们不好就没有干净的了](https://post.smzdm.com/p/apwoe6ex/)
- [![](https://a.zdmimg.com/202509/16/68c842087ed914506.png_a320.jpg)](https://post.smzdm.com/p/a507487k/)
	[小米17改名，官方公布真实原因，苹果的“含金量”还在提升](https://post.smzdm.com/p/a507487k/)
	135 718
	[小米17改名，官方公布真实原因，苹果的“含金量”还在提升](https://post.smzdm.com/p/a507487k/)
- [![](https://a.zdmimg.com/202509/11/68c263919ba846748.jpg_a320.jpg)](https://post.smzdm.com/p/al4249w8/)
	[别买彩激了！兄弟多功能打印机，你想要的都有，工作学习全拿捏！](https://post.smzdm.com/p/al4249w8/)
	210 268
	[别买彩激了！兄弟多功能打印机，你想要的都有，工作学习全拿捏！](https://post.smzdm.com/p/al4249w8/)

已收藏  
[去我的收藏夹](https://post.smzdm.com/p/akle62mk/#)

![](https://res.smzdm.com/pc/common/talent-report/talent-report-icon.png)

举报成功

该举报信息「小小值」会优先核实并处理哦！