---
title: "ESXi 8.0虚拟机必备知识与保姆级安装过程__什么值得买"
source: "https://post.smzdm.com/p/a8x6o5on/"
author:
published: 2023-01-31
created: 2025-09-13
description: "写在前面上面一篇写完了倍控 i3-1115G4 工控小主机开箱折腾，感兴趣的朋友可以前去围观。今天我们开始进入软路由的安装，本篇先介绍如何安装ESX"
tags:
  - "clippings"
---
当前位置：

[社区首页](https://post.smzdm.com/)

文章详情

![](https://qna.smzdm.com/202301/31/63d7ecd87ccc45898.gif_fo742.jpg)

## ESXi 8.0虚拟机必备知识与保姆级安装过程

## 写在前面

上面一篇写完了倍控 i3-1115G4 工控小主机开箱折腾，感兴趣的朋友可以前去围观。

![](https://qna.smzdm.com/202301/30/63d6bbb50aa629928.gif_a200.jpg)

倍控 i3-1115G4 工控小主机开箱折腾

46

63

今天我们开始进入软路由的安装，本篇先介绍如何安装ESXi 8.0虚拟机。iKuai 和 openWrt 的内容放在下一篇。参考了不少大佬的经验，综合下来，打造软路由的话 ESXi 和 PVE 用的比较多，ESXi 稳定性更好，PVE 硬件兼容性好些。

> - 虚拟机（Virtual Machine）指通过软件模拟的具有完整硬件系统功能的、运行在一个完全隔离环境中的完整计算机系统。在实体计算机中能够完成的工作在虚拟机中都能够实现。通过将多台虚拟机放置在一台计算机上，可以仅在一台物理 [服务器](https://www.smzdm.com/fenlei/fuwuqi/) 或 “主机” 上运行多个操作系统和应用。
> - ESXi，是 VMWare vSphere Hypervisor 套件之下重要组件。是一款一款优秀的 [服务器](https://www.smzdm.com/fenlei/fuwuqi/) 级别的虚拟机。前身是 ESX，依赖 Linux 源码，后来抛弃 Linux 源码做成了 ESXi。整个产品商用气息浓重，界面清晰易用，但硬件兼容性较差（主要面向服务器），没什么扩展性，ESXi 不依赖于任何操作系统，直接安装在裸机上，它本身就可以看作一个操作系统，然后可以在它上面安装其他系统。
> - PVE，全称 Proxmox Virtual Environment，是基于 Debian 的 Linux 系统，虚拟机内核为 KVM。硬件兼容性优秀。界面功能不强，很多操作要靠命令行，但扩展能力几乎是无限的。

楼主决定在工控主机内安装ESXi 8.0虚拟机，然后再虚拟机内安装iKuai作为主路由，OpenWrt 作为旁路由，说实话，楼主没有多拨和流控的需求，这样做主要是为了将网络隔离开，iKuai只负责PPOE拨号DHCP功能，其他的都交给openWrt。大家都知道openWrt插件丰富，平时折腾的比较多，经常会出现断网的情况，这样就可以放心折腾了。

### 1.网络拓扑

首先先大致规划一下家里的网络拓扑，因为家里原来用的是华硕的AC88U，像NAS、打印机这些静态IP地址的配置都是192.168.50.XX，这次折腾就懒得改了，索性直接以这个IP段，后面就不用再去重新配置了。大家也可以将50换成其他自己常用的。

> - iKuai ：192.168.50.1账号:admin
> - OpenWrt：192.168.50.2 账号:root
> - ESXi：192.168.50.3 账号:root

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecc7478525552.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_2/)

## 一、制作ventoy镜像安装工具

### 1.下载Ventoy启动盘制作工具

首先准备一个空闲的U盘，用Ventoy 一键制作USB启动盘。打开 [ventoy](https://www.ventoy.net/cn/download.html) 下载安装， Ventoy 是开源软件，下载使用都是免费的，直接去官网或者 GitHub 页面下载即可。

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecc769cef6955.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_3/)

在PC上安装，并运行Ventoy 软件，程序会自动检测当前 USB 设备。如果插入了多个U盘注意识别，别搞错了，造成数据丢失。点击安装开始制作：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecc769ced5529.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_4/)

可以看到 U 盘已经被重命名为 Ventoy。

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecc76d40f998.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_5/)

打开磁盘管理，可以看到 U 盘被细分为 2 个大分区，Ventoy 分区为活动分区，exFAT 文件系统，用于存放 ISO 文件，exFAT 文件系统也能更好的跨平台使用。

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecc770a1d3553.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_6/)

Ventoy 的系统分区里其实还有一个只有 1MB 的空间，存有 Legacy BIOS 模式下的启动文件，可见对于老旧设备来说，Ventoy 一样兼容。

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecc7df8d79746.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_7/) 格式化未NTFS，用于存放 ISO 文件的分区 ，支持 exFAT、FAT32、NTFS、UDF、XFS、Ext2、Ext3、Ext4。

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecc7dee324371.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_8/)

直接下载需要安装的系统镜像放到 U 盘里就可以了。如果U盘够大的话，一些常用的镜像和工具都可以放到里面，以后各个系统装机都可以使用。不管是各版本的 Windows 系统还是 PE 系统甚至是 ubuntu 系统，只要想加载就直接将镜像文件拖入到Ventoy文件夹内即可。

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://am.zdmimg.com/202301/31/63d7ecc8025772743.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_9/)

## 二、下载 ESXi 8.0

### 1.下载镜像

去 [官网下载](https://customerconnect.vmware.com/evalcenter?p=vsphere-eval-8) ，下载前，先注册一个 VMWare 的 customer connect 账号，注册登录完毕后，就可以下载 60 天试用版的 ESXI8.0 了，后面使用密钥激活一下即可，网上神秘代码很多，一搜一堆，这个就不多说了。

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://am.zdmimg.com/202301/31/63d7ecc823b5e8265.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_10/)

下载镜像文件，目前最新的版本是8.0-20513097

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecc8476a11114.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_11/)

## 2.查询网卡驱动

趁着下载的功夫，我们现在 [vmware硬件支持（VMware Compatibility Guide)](https://www.vmware.com/resources/compatibility/search.php?deviceCategory=io&details=1&partner=46&releases=589&keyword=226&deviceTypes=6&page=1&display_interval=10&sortColumn=Partner&sortOrder=Asc) ，查询一下网卡的驱动情况，Intel Corporation i226/ i225网卡都是支持免驱的。

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecc84dea47564.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_12/)

如果网卡不支持，可以去 [【ESXi 的社区网络驱动程序】](https://flings.vmware.com/community-networking-driver-for-esxi#requirements) 下载网卡驱动。

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecc85d7027695.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_13/)

将下载好的镜像拷入Ventoy，ventoy镜像安装工具制作完成。

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecc87a5536172.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_14/)

## 三、安装ESXi8.0

将工控机连接好 [显示器](https://www.smzdm.com/fenlei/xianshiqi/) 、键盘、鼠标。鼠标和键盘最好是有线的，插上 U 盘，准备开机。

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecc8859783785.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_15/)

### 1.主板设置

开机按 del/esc 进入BIOS设置，进入 BIOS 后，首先打开处理器的虚拟化（即 VT-x）。

VT-x是 intel 运用 Virtualization 虚拟化技术中的一个指令集。提供 X86 平台的硬件辅助虚拟化解决方案。能够将单台电脑软件环境分割为多个独立分区，每个分区均可以按照需要模拟电脑的一项技术。将多种系统整合到一台服务器或电脑中，使同一个物理平台能够同时运行多个同类或不同类的操作系统，以分别作为不同业务和应用的支撑平台。

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecc8d4ce13721.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_16/)

#### ①打开VT-d：

英特尔定向 I/O 虚拟化技术 (VT-d) 是英特尔虚拟化技术 (VT) 的扩展，它向硬件提供虚拟化解决方案协助。英特尔 VT-d 能帮助用户提高系统的安全性和可靠性，并改善 I/O 设备在虚拟化环境中的性能。这些都从本质上帮助 IT 管理人员通过减少潜在的停机时间而降低总拥有成本；并通过更充分地利用数据中心资源而增大生产性吞吐量。

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecc8da11a7586.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_17/)

#### ②打开AES：

**AES 指令集：** 高级加密标准指令集（英特尔高级加密标准新指令，简称 AES-NI）是一个 x86 指令集架构的扩展，用于 Intel 和 AMD 微处理器，由 Intel 在 2008 年 3 月提出。该指令集的目的是改进应用程序使用 高级加密标准（AES）执行加密和解密的速度，可以提高出国留学的速度。

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecc8ef6567953.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_18/)

#### ④开启睿频加速技术

在 CPU 技术设置选项中，找到 “Intel Turbo Mode Tech” 选项，有的主板可能会显示 “Intel Turbo Boost Tech”。至于 C-STATE就先不开了。

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecc90451a6991.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_19/)

C-STATE 是 CPU 的电源管理功能，它会根据 CPU 的负载来管理 CPU 的能耗，和睿频加速技术结合，在运行单线程应用时，C-STATE 会关闭或降低其他核心的能耗，把这些能源加到执行程序的核心上提高CPU的执行效率。

C-STATE的模式从 C0 开始，C0 表示 “正常” 的 CPU 运行模式（CPU 已 100% 激活）。C 后的数字越大，表示 CPU 进入休眠模式的程度越深。换言之，关闭的电路和时钟信号越多，完全唤醒 CPU 进入 C0 模式需要的时间越长。

| Mode（模式） | 姓名 | 作用 | CPU |
| --- | --- | --- | --- |
| C0 | 操作状态 | CPU 完全打开 | 所有 CPU |
| C1 | 停止 | 通过软件停止 CPU 内部主时钟；总线接口单元和 APIC 仍然保持全速运行 | 486DX4 及以上 |
| C1E | 增强型停止 | 通过软件停止 CPU 内部主时钟并降低 CPU 电压；总线接口单元和 APIC 仍然保持全速运行 | 所有 socket 775 CPU |
| C1E | — | 停止所有 CPU 内部时钟 | Turion 64、65-nm Athlon X2 和 Phenom CPU |
| C2 | 停止授予 | 通过硬件停止 CPU 内部主时钟；总线接口单元和 APIC 仍然保持全速运行 | 486DX4 及以上 |
| C2 | 停止时钟 | 通过硬件停止 CPU 内部和外部时钟 | 仅限 486DX4、Pentium、Pentium MMX、K5、K6、K6-2、K6-III |
| C2E | 扩展的停止授予 | 通过硬件停止 CPU 内部主时钟并降低 CPU 电压；总线接口单元和 APIC 仍然保持全速运行 | Core 2 Duo 和更高版本（仅限 Intel） |
| C3 | 睡眠 | 停止所有 CPU 内部时钟 | Pentium II、Athlon 以上支持，但 Core 2 Duo E4000 和 E6000 上不支持 |
| C3 | 深度睡眠 | 停止所有 CPU 内部和外部时钟 | Pentium II 以上支持，但 Core 2 Duo E4000、E6000 和 Turion 64 上不支持 |
| C3 | AltVID | 停止所有 CPU 内部时钟和降低 CPU 电压 | AMD Turion 64 |
| C4 | 更深入的睡眠 | 降低 CPU 电压 | Pentium M 以上支持，但 Core 2 Duo E4000、E6000 和 Turion 64 上不支持 |
| C4E/C5 | 增强的更深入的睡眠 | 大幅降低 CPU 电压并关闭内存高速缓存 | Core Solo、Core Duo 和 45-nm 移动版 Core 2 Duo 支持 |
| C6 | 深度电源关闭 | 将 CPU 内部电压降低至任何值，包括 0 V | 仅 45-nm 移动版 Core 2 Duo 支 |

### 2.设置USB启动

在BOOT里把U盘设置为第一启动项。

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecc96bb4a7030.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_20/)

### 3.删除硬盘分区

重启开机后，即可进入 Ventoy 界面，U 盘内保存的系统镜像都在列表中，选择要安装的镜像文件即可，选择启动WinPE64，按Enter键进入PE。

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecc96ec1c8643.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_21/)

进到PE后，用分区工具把软路由硬盘分区全部删除(注意不要删错)，这里有个大坑，如果硬盘存在分区的话，很可能造成无法安装的bug，尤其是对于拆机的硬盘来说，一定要将上面的分区及隐藏分区全部删掉。

### 4.加载ESXi8.0镜像

删除之后，重启电脑，选择ESXi启动安装：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecc98f6883711.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_22/)

### 5.修改ESXI的默认空间

在读秒阶段，快速按下Shift+O，调出命令行，来修改ESXI的默认空间大小：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecc98e2069432.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_23/)

在下面命令行输入：autoPartotionOSDataSize=20480。命令注意区分大小写，我这里将默认空间设置为20GB。硬盘空间不足的话推荐设置8192(8GB)即可，大家可以根据自己的情况进行设定：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecc99de743199.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_24/)

回车，开始跑码：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://am.zdmimg.com/202301/31/63d7ecca2914d7846.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_25/)

还在跑码，等待跑码完成

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecca2b4a54427.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_26/)

继续跑码，不用管，等着就行：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecca4fb812112.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_27/)

跑码：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecca6215c1131.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_28/)

跑码：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccabe7bf547.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_29/)

这一步 **Starting service vmtoolsd** 的加载时间会比较长，一定要耐心等待，不是死机了。

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccabcda07186.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_30/)

一直等待到出现如下界面，选择 Continue，按回车，继续下一步：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccabfcfb8128.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_31/)

同意 VMWARE 的条款，按F11，继续下一步：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccb132ed7802.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_32/)

### 6.选择安装位置

接下来会扫描此计算机上的所有存储器(硬盘)，等待扫描结果：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccb19bd61223.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_33/)

选择安装位置，可以看到有两个盘，一个NVMe的 [固态硬盘](https://www.smzdm.com/fenlei/gutaiyingpan/) 和U盘，用键盘上下键调整，选择第一个安装在固态硬盘。一定要看好，不要安装错位置。选择好了回车继续：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccb2af1c1298.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_34/)

选择键盘布局，回车继续下一步：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccb57ed21106.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_35/)

### 7.设置登录密码

弹出密码输入页，这里输入密码，需要输入两遍，输入第一遍之后，按table键切换到第二行输入第二遍确认密码，输入一定要慢一点，因为不显示输入的密码，很容易输错，密码需要大小写带数字。

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccb8f1509283.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_36/)

输入完毕后，回车进行下一步：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccb98e11777.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_37/)

弹出确认安装位置选项，看一下安装位置有没有问题，没选错，按F11继续安装：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccbd87b19675.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_38/)

等待加载完成：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccc1eb454462.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_39/)

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccc2efa48201.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_40/)

到这里拔掉U盘，之后按回车，重启。

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccc810964072.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_41/)

回车继续：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccca7392315.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_42/)

重启后开始跑码：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccce635e294.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_43/)

等待跑码完成：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecccf279e5540.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_44/)

继续等待：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccd33c9f7421.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_45/)

继续加载，下方的安装指示条可以看到安装的进度，全程不用管：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccd423da8908.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_46/)

有以下画面表示已经安装成功，按F2可进入管理后台，F12是关闭或者重启：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccd962697800.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_47/)

## 四、ESXI虚拟机设置

按F2，弹出登录页面，输入刚才设定的密码。回车，进入ESXI管理后台。

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccd951139415.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_48/)

可以看到第一项是修改密码，第三项是设置网络。

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccdcfaa54055.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_49/)

### 1.设置网口

键盘上下键选择第三项，修改网络配置，回车进入：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccde7ff17866.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_50/)

选择第一项，设置网口：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecce2b4b21058.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_51/)

上下键选择你要用来管理ESXI的网口，这里有些工程机的网口是错乱的，可以通过插拔 [网线](https://www.smzdm.com/fenlei/wangxian/) 来确定是不是正确。如果不正确的话，需要通过后面在web端设置进行接口对应。

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecce3ba3d8365.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_52/)

经过插拔网线，四个接口的顺序和ESXi系统显示的网口是一一对应的。选择etho接口作为管理口。通过键盘上下键，移动黑色条框，按空格确定选项。

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecce71bae7116.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_53/)

回车保存，自动退回到【网络配置】页面：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccea73c0996.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_54/)

### 2.设置IPV4选项

选择设置IPV4选项：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecceb38f31961.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_55/)

键盘上下键，移动黑色条框到第三项，可以看到第三项前面的括号里有个圆圈(○)，按空格确定选项。

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccee27a38527.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_56/)

设置下面三个选项【IPV4 Address(ESXI管理地址)】、【Subnet Mask(子网掩码)】、【Default Gateway(默认网关)】：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecceee98b1008.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_57/)

> - IPV4 Address【ESXI管理地址】：192.168.50.3
> - Subnet Mask【子网掩码】：255.255.255.0
> - Default Gateway【默认网关】：192.168.50.1

这个管理地址是后面我们在web端进行访问的地址，千万不要和ikuai以及openwrt冲突。Default Gateway【默认网关】就设置为主路由的网管即可。我以ikuai作为主路由。这里就直接将网关设置为主路由ikuai的地址了。按回车保存：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccf137395964.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_58/)

退出来之后，在页面右上角就看到我们设置的信息了，按ESC，退出：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccf1a3676537.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_59/)

在弹出的页面，按Y，并自动重启网络：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccf63a5a5344.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_60/)

网络重启完后，可以看到，已经设置成功了，按ESC，回到首页：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://am.zdmimg.com/202301/31/63d7eccf9c5a75954.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_61/)

返回首页之后，也可以看到已经设置完成：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccfccdc9988.jpg_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_62/)

## 五、连接ESXi 8.0网页端设置

### 1.给电脑分配IP

我们将网线一头插到软路由的eth0口，另外一端插到电脑上，因为ESXi没有DHCP(Dynamic Host Configuration Protocol, 动态主机配置协议) 功能，没办法给电脑自动分配IP地址，这里要手动修改一下：

打开控制面板—网络和internet——更改适配器选项

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccfce8333831.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_63/)

这时候电脑显示未识别的网络：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7eccfd10228953.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_64/)

右击网络-【属性】：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://am.zdmimg.com/202301/31/63d7eccfe0a043310.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_65/)

双击IPV4协议，在弹出的属性内，填入相同的IP段：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecd04871a341.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_66/)

### 2.访问web端

打开浏览器，在浏览器内输入：192.168.50.3访问，用账户名：root和前面安装系统时设置的密码登录：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecd07099d5023.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_67/)

弹出的页面选择不加入改进计划，确认下一步：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://am.zdmimg.com/202301/31/63d7ecd07709d406.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_68/)

### 3.激活

在【管理】——【许可】看下激活状态，进来之后可以看到系统分配的60天的密钥：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://am.zdmimg.com/202301/31/63d7ecd07f2f8369.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_69/)

然后点分配许可证，输入密钥激活即可：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecd0bf1d53112.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_70/)

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecd0ecaf96011.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_71/)

## 六、开启直通网卡

选择【管理】-【硬件】-【PCI设备】，查看可以直通的网卡，这里网卡的顺序就是按照我们物理接口的etho、eth1、eth2、eth3顺序来的，第一个网卡千万不要直通，也就是我们前面设置的ESXi管理口，不要设置直通。如果把管理口也做了直通，ESXI 重启后 WEB 管理界面就进入不去了。

### 直通的好处？

直通是指虚拟化时将硬件管理权、使用权完全交给一个虚拟出的系统(使用这个网卡的虚拟机)，不经过虚拟系统的虚拟封装，以直接以原生方式使用硬件，达到最高性能。直通的效率更好，因为直接直通给了虚拟机使用，硬件使用中产生的数据不会再经过ESXI进行二次处理，可以节约一部分系统性能，不直通的话虚拟机在运行中会占用更多的cpu。

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://am.zdmimg.com/202301/31/63d7ecd10d9ac3888.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_72/)

选择要直通的设备，然点击切换直通，将三个网卡的直通全部开启：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecd12447c9216.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_73/)

看到网卡直通状态变为活动后，就是直通开通成功了，这样后面我们安装路由虚拟机时，直接添加PCl设备就可以直通了，同时注意内存设置全部保留。这个放到安装ikuai和openwrt时具体再讨论：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecd129f499041.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_74/)

## 七、设置网络

在网络内对虚拟机VM Network进行设置，选择【编辑设置】——安全内的混杂模式、mac地址改变、伪传输都改为接受：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://am.zdmimg.com/202301/31/63d7ecd156689962.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_75/)

## 八、修改电源模式

一定要电源模式改为高性能，否则会 [路由器](https://www.smzdm.com/fenlei/luyouqi/) 会自动限制性能，百度网盘，迅雷等软件下载速度会限制在10MB/S。

选择【管理】-【硬件】-【电源管理】-【更改策略】-高性能：

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecd1866055445.png_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_76/)

## 小结：

本期的经验分享就到这里结束了，本期主要是安装ESXi8.0虚拟机和基本的配置，在安装过程中碰到的一些问题写出来分享给大家，下一步就可以安装路由虚拟机了，后面我们来安装ikuai及openwrt，继续折腾。

---

### 以上就是本期的全部内容了，感兴趣的朋友，关注我不迷路★

[![ESXi 8.0虚拟机必备知识与保姆级安装过程](https://qnam.smzdm.com/202301/31/63d7ecd1930557944.gif_e1080.jpg)](https://post.smzdm.com/p/a8x6o5on/pic_77/)

## 往期热文推荐

![](https://qna.smzdm.com/202301/30/63d6bbb50aa629928.gif_a200.jpg)

倍控 i3-1115G4 工控小主机开箱折腾

46

63

  

作者声明本文无利益相关，欢迎值友理性交流，和谐讨论～

文章很值，打赏犒劳作者一下

6 人已打赏## [纵笔浮生](https://zhiyou.smzdm.com/member/2279839416/)

[

v-x：knudyinger 同名VB\\公众号：纵笔浮生 再见少年拉满弓，不惧岁月，不惧风。 欲买桂花同载酒，终不似，少年游。

866文章 | 172爆料 | 61636粉丝

](https://zhiyou.smzdm.com/member/2279839416/)

## 纵笔浮生

v-x：knudyinger 同名VB\\公众号：纵笔浮生 再见少年拉满弓，不惧岁月，不惧风。 欲买桂花同载酒，终不似，少年游。

首席生活家，数码领域作者

发文累计获赞17.2万，内容被51.0万人收藏

微博：纵笔浮生

- [![](https://a.zdmimg.com/202507/26/688426e23241e6164.jpg_a320.jpg)](https://post.smzdm.com/p/amvr7xqz/)
	[让丐版不是丐版，让Mac mini起飞！零刻Mate mini拓展坞评测](https://post.smzdm.com/p/amvr7xqz/)
	83 4
	[让丐版不是丐版，让Mac mini起飞！零刻Mate mini拓展坞评测](https://post.smzdm.com/p/amvr7xqz/)
- [![](https://a.zdmimg.com/202508/20/68a54163f369d5119.gif_a320.jpg)](https://post.smzdm.com/p/aoglek8r/)
	[从幼儿园到成人办公，大将作UP椅让孩子“坐姿一路正到底”](https://post.smzdm.com/p/aoglek8r/)
	90 15
	[从幼儿园到成人办公，大将作UP椅让孩子“坐姿一路正到底”](https://post.smzdm.com/p/aoglek8r/)
- [![](https://a.zdmimg.com/202508/24/68aac817016252643.gif_a320.jpg)](https://post.smzdm.com/p/aeognnrk/)
	[以速破局，610Hz 超高刷FPS对战专属：ROG绝神S TN电竞显示器评测](https://post.smzdm.com/p/aeognnrk/)
	118 5
	[以速破局，610Hz 超高刷FPS对战专属：ROG绝神S TN电竞显示器评测](https://post.smzdm.com/p/aeognnrk/)
- [![](https://a.zdmimg.com/202508/25/68ac6d398fcf41086.jpg_a320.jpg)](https://post.smzdm.com/p/amog359d/)
	[一次旅行、一场通勤、一个好睡眠：5款让我惊喜的生活好物合集](https://post.smzdm.com/p/amog359d/)
	133 18
	[一次旅行、一场通勤、一个好睡眠：5款让我惊喜的生活好物合集](https://post.smzdm.com/p/amog359d/)
- [![](https://a.zdmimg.com/202508/25/68ac6fb2730847299.jpg_a320.jpg)](https://post.smzdm.com/p/awmgdwo4/)
	[千元电竞屏天花板！27寸2K300Hz——HKC G27H4PLUS全方位评测](https://post.smzdm.com/p/awmgdwo4/)
	108 10
	[千元电竞屏天花板！27寸2K300Hz——HKC G27H4PLUS全方位评测](https://post.smzdm.com/p/awmgdwo4/)

- [![](https://a.zdmimg.com/202509/13/68c50aa40e39f104.png_a320.jpg)](https://post.smzdm.com/p/al4g3p7e/)
	[好用的抗皱面霜有哪些？未来抗衰老新趋势！提拉紧致抗皱哪个好](https://post.smzdm.com/p/al4g3p7e/)
	0 0
	[好用的抗皱面霜有哪些？未来抗衰老新趋势！提拉紧致抗皱哪个好](https://post.smzdm.com/p/al4g3p7e/)
- [![](https://a.zdmimg.com/202509/13/68c50a8cf40c26914.jpg_a320.jpg)](https://post.smzdm.com/p/ako58mpr/)
	[开放式耳机实测推荐！音质不将就、价格不肉疼，按需选不踩坑](https://post.smzdm.com/p/ako58mpr/)
	0 0
	[开放式耳机实测推荐！音质不将就、价格不肉疼，按需选不踩坑](https://post.smzdm.com/p/ako58mpr/)
- [![](https://a.zdmimg.com/202509/13/68c5078f33ebc724.png_a320.jpg)](https://post.smzdm.com/p/azzn8kpn/)
	[王依诺 YiNuo 秋季款高腰显瘦气质短裤：解锁秋日时尚密码](https://post.smzdm.com/p/azzn8kpn/)
	0 0
	[王依诺 YiNuo 秋季款高腰显瘦气质短裤：解锁秋日时尚密码](https://post.smzdm.com/p/azzn8kpn/)
- [![](https://a.zdmimg.com/202509/13/68c5054f3c5f78283.png_a320.jpg)](https://post.smzdm.com/p/awmew92m/)
	[深圳妙感成人用品技术创新，成人赛道新势力](https://post.smzdm.com/p/awmew92m/)
	0 0
	[深圳妙感成人用品技术创新，成人赛道新势力](https://post.smzdm.com/p/awmew92m/)
- [![](https://a.zdmimg.com/202509/13/68c505e045eea7043.png_a320.jpg)](https://post.smzdm.com/p/al4g3nz0/)
	[蓝白渐变证件照背景怎么弄？一键生成超自然](https://post.smzdm.com/p/al4g3nz0/)
	0 0
	[蓝白渐变证件照背景怎么弄？一键生成超自然](https://post.smzdm.com/p/al4g3nz0/)
- [![](https://a.zdmimg.com/202509/05/68ba8a34a316f3737.jpg_a320.jpg)](https://post.smzdm.com/p/ax6wvw5d/)
	[到站秀：家庭清洁全场景覆盖 科沃斯T80S 重新定义3000+价位扫地机器人天花板](https://post.smzdm.com/p/ax6wvw5d/)
	2587 2606
	[到站秀：家庭清洁全场景覆盖 科沃斯T80S 重新定义3000+价位扫地机器人天花板](https://post.smzdm.com/p/ax6wvw5d/)
- [![](https://a.zdmimg.com/202509/09/68c04d2ecf1a57395.jpg_a320.jpg)](https://post.smzdm.com/p/aeo4pve3/)
	[【评论有奖】这一次，曹操是主角！曹操头号拥趸易中天力挺《三国的星空》](https://post.smzdm.com/p/aeo4pve3/)
	158 565
	[【评论有奖】这一次，曹操是主角！曹操头号拥趸易中天力挺《三国的星空》](https://post.smzdm.com/p/aeo4pve3/)
- [![](https://a.zdmimg.com/202509/12/68c43f5d4ff5a3982.png_a320.jpg)](https://post.smzdm.com/p/aeo4oml4/)
	[被罗永浩吐槽的西贝，一份炒鸡蛋卖43元，后厨用电磁炉炒菜](https://post.smzdm.com/p/aeo4oml4/)
	129 597
	[被罗永浩吐槽的西贝，一份炒鸡蛋卖43元，后厨用电磁炉炒菜](https://post.smzdm.com/p/aeo4oml4/)
- [![](https://a.zdmimg.com/202509/12/68c2fb60733876794.png_a320.jpg)](https://post.smzdm.com/p/a3rp092k/)
	[贵不贵 西贝晒出罗永浩消费账单：15个菜收费830元](https://post.smzdm.com/p/a3rp092k/)
	66 668
	[贵不贵 西贝晒出罗永浩消费账单：15个菜收费830元](https://post.smzdm.com/p/a3rp092k/)
- [![](https://a.zdmimg.com/202509/10/68c07a46ded1a1272.png_a320.jpg)](https://post.smzdm.com/p/a9kzxmop/)
	[iPhone 17系列发布：120Hz高刷终于普及，A19系列芯片，以及全新的iPhone Air](https://post.smzdm.com/p/a9kzxmop/)
	157 437
	[iPhone 17系列发布：120Hz高刷终于普及，A19系列芯片，以及全新的iPhone Air](https://post.smzdm.com/p/a9kzxmop/)

## 纵笔浮生

v-x：knudyinger 同名VB\\公众号：纵笔浮生 再见少年拉满弓，不惧岁月，不惧风。 欲买桂花同载酒，终不似，少年游。

首席生活家，数码领域作者

发文累计获赞17.2万，内容被51.0万人收藏

微博：纵笔浮生

- [![](https://a.zdmimg.com/202507/26/688426e23241e6164.jpg_a320.jpg)](https://post.smzdm.com/p/amvr7xqz/)
	[让丐版不是丐版，让Mac mini起飞！零刻Mate mini拓展坞评测](https://post.smzdm.com/p/amvr7xqz/)
	83 4
	[让丐版不是丐版，让Mac mini起飞！零刻Mate mini拓展坞评测](https://post.smzdm.com/p/amvr7xqz/)
- [![](https://a.zdmimg.com/202508/20/68a54163f369d5119.gif_a320.jpg)](https://post.smzdm.com/p/aoglek8r/)
	[从幼儿园到成人办公，大将作UP椅让孩子“坐姿一路正到底”](https://post.smzdm.com/p/aoglek8r/)
	90 15
	[从幼儿园到成人办公，大将作UP椅让孩子“坐姿一路正到底”](https://post.smzdm.com/p/aoglek8r/)
- [![](https://a.zdmimg.com/202508/24/68aac817016252643.gif_a320.jpg)](https://post.smzdm.com/p/aeognnrk/)
	[以速破局，610Hz 超高刷FPS对战专属：ROG绝神S TN电竞显示器评测](https://post.smzdm.com/p/aeognnrk/)
	118 5
	[以速破局，610Hz 超高刷FPS对战专属：ROG绝神S TN电竞显示器评测](https://post.smzdm.com/p/aeognnrk/)
- [![](https://a.zdmimg.com/202508/25/68ac6d398fcf41086.jpg_a320.jpg)](https://post.smzdm.com/p/amog359d/)
	[一次旅行、一场通勤、一个好睡眠：5款让我惊喜的生活好物合集](https://post.smzdm.com/p/amog359d/)
	133 18
	[一次旅行、一场通勤、一个好睡眠：5款让我惊喜的生活好物合集](https://post.smzdm.com/p/amog359d/)
- [![](https://a.zdmimg.com/202508/25/68ac6fb2730847299.jpg_a320.jpg)](https://post.smzdm.com/p/awmgdwo4/)
	[千元电竞屏天花板！27寸2K300Hz——HKC G27H4PLUS全方位评测](https://post.smzdm.com/p/awmgdwo4/)
	108 10
	[千元电竞屏天花板！27寸2K300Hz——HKC G27H4PLUS全方位评测](https://post.smzdm.com/p/awmgdwo4/)

已收藏  
[去我的收藏夹](https://post.smzdm.com/p/a8x6o5on/#)

![](https://res.smzdm.com/pc/common/talent-report/talent-report-icon.png)

举报成功

该举报信息「小小值」会优先核实并处理哦！