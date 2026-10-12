---
title: "PVE+iKuai+OpenWrt+fnOS折腾记录"
source: "https://mister-kin.github.io/software/pve/"
author:
  - "[[Mr. KinSince you have chosen to chase]]"
  - "[[don't give up.]]"
  - "[[Mr. Kin]]"
published: 2023-09-11
created: 2025-09-17
description: "PVE官方文档 PBS官方文档 PMG官方文档    1   背景早期因联通赠送的光猫性能不支持双LAN口长时间同时上网（存在间歇性断流现象），故从海鲜市场（小黄鱼）上淘了一个N3530小工控机（准系统，无内存条），计划用来安装PVE虚拟机系统，以便搭建软路来处理宽带拨号和作为DHCP服务器分配管理局域网IP，而光猫仅保留作为一个光电信号转换器。 后期陆续购置新硬件摸索各类方"
tags:
  - "clippings"
---
| [PVE 官方文档](https://pve.proxmox.com/pve-docs/) | [PBS 官方文档](https://pbs.proxmox.com/docs/) | [PMG 官方文档](https://pmg.proxmox.com/pmg-docs/) |
| --- | --- | --- |

## 1 背景

早期因联通赠送的光猫性能不支持双 LAN 口长时间同时上网（存在间歇性断流现象），故从海鲜市场（小黄鱼）上淘了一个 N3530 小工控机（准系统，无内存条），计划用来安装 PVE 虚拟机系统，以便搭建软路来处理宽带拨号和作为 DHCP 服务器分配管理局域网 IP，而光猫仅保留作为一个光电信号转换器。

后期陆续购置新硬件摸索各类方案，目前本文已总结了笔者探索使用 PVE 的各种经验所得。

## 2 机器配置概览及 BIOS 优化设置

## 2.1 N3530 工控机配置概览

| 配件类别 | 配件型号 | 数量 | 品牌 | 备注 |
| --- | --- | --- | --- | --- |
| CPU | Intel(R) Pentium(R) CPU N3530 | 1 | 英特尔 Intel |  |
| 内存 | Hynix DDR3L-8GB | 1 | 海力士 Hynix | 从老笔记本电脑上拆出来的一根 DDR3L 条子 |
| 硬盘 | Hynix SH920 mSATA 256GB | 1 | 海力士 Hynix |  |
| 千兆网卡 | Intel I211   Realtek Mini-PCIe RTL8111E/F | 2   1 | 英特尔 Intel   瑞昱 Realtek | Realtek 螃蟹卡是后期购买加装至 Mini-PCIe 无线网卡口位置   通过拆掉主控板的一个 COM 接口，用锉刀将外壳挫出一个 RJ45 网口的尺寸孔实现加装 |
| 主板 | 工控机主板 | 1 | 未知品牌 |  |
| 外围接口 | RJ45 网线接口   USB-A 3.0   USB-A 2.0   HDMI   VGA   麦克风 3.5mm   耳机 3.5mm   COM 口   SIM 卡槽 | 3   2   4   1   1   1   1   1   1 | 无 |  |
| 内围接口 | mSATA   Mini-PCIe   SATA   SATA PWR 供电口 | 2   1   1   1 | 无 |  |

## 2.2 N3530 工控机 BIOS 优化设置

> 快速启动菜单按键：F11 键  
> 进入 BIOS 模式按键：ESC 或者 DEL 键

该 N3530 工控机 BIOS 是采用经典的 `AMI(American Megatrends)` 方案。

- 设置来电自启： `Chipset->South Bridge->Restore AC Power Loss->Power on`
- 启用 CPU 内置温度传感器： `Advanced->CPU Configuration->CPU Thermal Configuration->DTS->Enabled`
- 启用 VT-x： `Advanced->CPU Configuration->Intel Virtualization Technology->Enabled`
- 设置自定义功率模式： `Advanced->CPU Configuration->Power Technology->Custom` 。（默认是 Efficient 节能模式）

## 2.3 i5-12400+ 云星 Matx-Z690 NAS 主板配置

## 2.4 云星 Matx-Z690 NAS 主板 BIOS 优化设置

> 快速启动菜单按键：F11 键

## 3 安装 PVE

## 3.1 PVE 安装步骤

1. 下载 PVE 系统镜像： [点击跳转](https://www.proxmox.com/en/downloads/proxmox-virtual-environment/iso) （我的情况就是需要下载 pve 7.4 的 iso 镜像文件，因为安装 pve 8.x 会引起 `bootloader setup errors` 报错而导致安装 PVE 系统失败）
2. 制作 U 盘启动项： [ventoy](https://github.com/ventoy/Ventoy/releases) 。拷贝镜像 iso 文件到 U 盘，启动选择 pve 镜像进入，选择图形安装界面。
3. 同意协议。
4. 点击 option 按钮，选择硬盘和 ext4 格式：
	1. swapsize = 8GB（交换分区请根据自己的硬盘容量和使用情况设置）
	2. maxroot = 50GB（该分区对应 PVE 系统的 local 存储空间，主要用于存放 ISO 镜像和 LXC 模板（CT Templates），请根据自己的硬盘和使用情况设置）
	3. minfree = 0GB（该分区是定义划分 LVM 卷组 “pve” 后剩余的可用空间量。当可用存储空间超过 128GB 时，其默认值为 16GB。）
		- 一般是留出这闲置空间用于手动 lvcreate 逻辑分卷 snapshot 快照这种操作。但 PVE 是使用 lvmthin，无需闲置空间作为 lvmthin 的快照存储，因此设置为 0 即可。
		- 如果自己的需求有自行创建逻辑卷快照的需求，请留空使用默认值 16GB 即可。
		- LVM 需要 VG 中的可用空间来创建快照，而 lvmthin 快照则不需要。
	4. maxvz = 留空（该分区是定义数据卷（data）的最大大小。计算公式：datasize = hdsize - rootsize - swapsize - minfree）
5. 国家选择 china，时区会自动修改为上海，键盘布局不改。
6. 设置密码，邮箱随意。
7. 管理网口，默认第一个做管理（enp1s0，pve 系统里面识别的第一个网口，N3530 工控机中靠近 hdmi 接口的 rj45），第二个做 wan 口（方便后续维护）。域名 hostname 设置为 pve.lan（这项随便填写都行，有能用的域名可填写上去，可解析到机器用域名管理 pve 虚拟机）；ip：192.168.1.2；掩码：255.255.255.0；网关：192.168.1.1（留给 iKuai 的地址），DNS：223.5.5.5。
8. 查看确认已设置的内容（弹窗），确认安装。
9. 访问 WEB 管理地址并登录： [https://192.168.1.2:8006](https://192.168.1.2:8006/) 。账户：root。密码：\[前面步骤设置的密码\]。
10. 设置网卡并确保 pve 能联网。

## 3.2 PVE 升级步骤（7.4->8.x）

### 3.2.1 替换 PVE 7.4 的 Debian 系统软件源和 PVE 软件源

注：推荐中科大源，清华源可能会遇到大型 NAT 局域网宽带频繁访问而导致被禁用 IP，从而导致出现 403 Forbidden。

修改 `/etc/apt/sources.list` 文件内容

```bash
BASHmv /etc/apt/sources.list /etc/apt/sources.list.bak

nano /etc/apt/sources.list
```

```bash
BASH#deb http://ftp.debian.org/debian bullseye main contrib

deb https://mirrors.ustc.edu.cn/debian/ bullseye main contrib non-free

#deb http://ftp.debian.org/debian bullseye-updates main contrib

deb https://mirrors.ustc.edu.cn/debian/ bullseye-updates main contrib non-free

deb https://mirrors.ustc.edu.cn/debian/ bullseye-backports main contrib non-free

# security updates

#deb http://security.debian.org bullseye-security main contrib

deb https://mirrors.ustc.edu.cn/debian-security bullseye-security main contrib non-free
```

```bash
BASH#deb http://ftp.debian.org/debian bullseye main contrib

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye main contrib non-free

#deb http://ftp.debian.org/debian bullseye-updates main contrib

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-updates main contrib non-free

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-backports main contrib non-free

# security updates

#deb http://security.debian.org bullseye-security main contrib

deb https://mirrors.tuna.tsinghua.edu.cn/debian-security bullseye-security main contrib non-free
```

删除 pve-enterprise.list，新建 pve-no-subscription.list。

```bash
BASHmv /etc/apt/sources.list.d/pve-enterprise.list /etc/apt/sources.list.d/pve-enterprise.list.bak
```

```bash
BASHecho deb https://mirrors.ustc.edu.cn/proxmox/debian/pve bullseye pve-no-subscription > /etc/apt/sources.list.d/pve-no-subscription.list
```

```bash
BASHecho deb https://mirrors.tuna.tsinghua.edu.cn/proxmox/debian/pve bullseye pve-no-subscription > /etc/apt/sources.list.d/pve-no-subscription.list
```

### 3.2.2 升级到 PVE 7.4 最新的版本

```bash
BASHapt update -y && apt dist-upgrade -y

pveversion
```

### 3.2.3 检查版本升级的兼容性

```bash
BASHpve7to8
```

### 3.2.4 替换 PVE 8.x 的 Debian 系统软件源和 PVE 软件源（国内镜像）

注：推荐中科大源，清华源可能会遇到大型 NAT 局域网宽带频繁访问而导致被禁用 IP，从而导致出现 403 Forbidden。

修改 /etc/apt/sources.list 文件内容

```bash
BASHmv /etc/apt/sources.list /etc/apt/sources.list.bak

nano /etc/apt/sources.list
```

```bash
BASH# deb http://ftp.debian.org/debian bookworm main contrib

deb https://mirrors.ustc.edu.cn/debian/ bookworm main contrib non-free non-free-firmware

# deb http://ftp.debian.org/debian bookworm-updates main contrib

deb https://mirrors.ustc.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware

deb https://mirrors.ustc.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware

# security updates

# deb http://security.debian.org bookworm-security main contrib

deb https://mirrors.ustc.edu.cn/debian-security bookworm-security main contrib non-free non-free-firmware
```

```bash
BASH# deb http://ftp.debian.org/debian bookworm main contrib

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware

# deb http://ftp.debian.org/debian bookworm-updates main contrib

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware

# security updates

# deb http://security.debian.org bookworm-security main contrib

deb https://mirrors.tuna.tsinghua.edu.cn/debian-security bookworm-security main contrib non-free non-free-firmware
```

删除 pve-enterprise.list，新建 pve-no-subscription.list。

```bash
BASHmv /etc/apt/sources.list.d/pve-enterprise.list /etc/apt/sources.list.d/pve-enterprise.list.bak
```

```bash
BASHecho deb https://mirrors.ustc.edu.cn/proxmox/debian/pve bookworm pve-no-subscription > /etc/apt/sources.list.d/pve-no-subscription.list
```

```bash
BASHecho deb https://mirrors.tuna.tsinghua.edu.cn/proxmox/debian/pve bookworm pve-no-subscription > /etc/apt/sources.list.d/pve-no-subscription.list
```

### 3.2.5 替换 LXC 模板（CT Templates）镜像源

修改 `/usr/share/perl5/PVE/APLInfo.pm` 文件内容， **针对 `/usr/share/perl5/PVE/APLInfo.pm` 文件的修改，重启后生效。**

### 3.2.6 开始正式升级

```bash
BASHapt update -y && apt dist-upgrade -y
```

升级若遇到提示 `Upgrade wants to remove package 'proxmox-ve'` 的话，在升级前尝试执行 `apt remove linux-image-amd64` 。

升级中会出现一些交互界面，下面是官网的建议（不清楚的选项，可以选择推荐参数）

```bash
BASH/etc/issue -> Proxmox VE will auto-generate this file on boot, and it has only cosmetic effects on the login console.

Using the default "No" (keep your currently-installed version) is safe here.

/etc/lvm/lvm.conf -> Changes relevant for Proxmox VE will be updated, and a newer config version might be useful.

If you did not make extra changes yourself and are unsure it's suggested to choose "Yes" (install the package maintainer's version) here.

/etc/ssh/sshd_config -> If you have not changed this file manually, the only differences should be a replacement of ChallengeResponseAuthentication no with KbdInteractiveAuthentication no and some irrelevant changes in comments (lines starting with #).

If this is the case, both options are safe, though we would recommend installing the package maintainer's version in order to move away from the deprecated ChallengeResponseAuthentication option. If there are other changes, we suggest to inspect them closely and decide accordingly.

/etc/default/grub -> Here you may want to take special care, as this is normally only asked for if you changed it manually, e.g., for adding some kernel command line option.

It's recommended to check the difference for any relevant change, note that changes in comments (lines starting with #) are not relevant.

If unsure, we suggested to selected "No" (keep your currently-installed version)
```

### 3.2.7 去除未订阅提示

**完成如下文件内容修改后，注销浏览器当前的 PVE 登录信息，并重新登录就没有提示弹窗了。**

- `/usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js`

[![](https://mister-kin.github.io/images/pve/no_subscribe.webp)](https://mister-kin.github.io/images/pve/no_subscribe.webp) 「无有效订阅」的提示弹窗

### 3.2.8 网卡 mac 地址绑定网卡名称

如果 PVE 宿主机经常要变更硬件的话，这个步骤的操作很重要，详看 [PVE 宿主机更替硬件后导致失联无法访问网页后台](https://mister-kin.github.io/software/pve/#PVE%E5%AE%BF%E4%B8%BB%E6%9C%BA%E6%9B%B4%E6%9B%BF%E7%A1%AC%E4%BB%B6%E5%90%8E%E5%AF%BC%E8%87%B4%E5%A4%B1%E8%81%94%E6%97%A0%E6%B3%95%E8%AE%BF%E9%97%AE%E7%BD%91%E9%A1%B5%E5%90%8E%E5%8F%B0)

### 3.2.9 后续 PVE 系统日常更新：

```bash
BASHapt update -y && apt dist-upgrade -y
```

## 4 PVE 安装和使用遇到的坑

## 4.1 PVE 安装到 99% 时出现 unable to install the EFI boot loader on '/dev/sda'

报错信息：

```
PLAINTEXTbootloader setup errors:

- failed to prepare EFI boot using Grub on '/dev/sda2': unable to install the EFI boot loader on '/dev/sda'
```

原因：PVE 8.0 版本镜像īso 不兼容 N3530 工控机，估计是 bug。参考其他帖子说的只保留 usb 的 uefi 设备，依旧也会报错。并且因为机器的 bios 设置也比较繁琐，我反复尝试将其设置为 csm 模式，机器始终无法以 csm 模式启动，估计该主板的 bios 也是魔改过有 bug 的。因此导致也无法以 csm 模式安装 pve。

解决方法：安装 pve 7.4 版本，再升级到 pve 8.x。

## 4.2 USB 键盘一直插着时会无法响应的，无法进入并操控 BIOS，只有 CTRL + ALT + DELETE 可以响应

原因：推测为 N3530 工控机 BIOS 的驱动 BUG。

解决方法：先不插键盘启动机器，看屏幕右下角等待主板自检代码走到 99 阶段，快速插上键盘，摁 DEL 键。如果一次不成功，就使用 `CTRL+ALT+DELETE` 强制重启，继续尝试该操作。

## 4.3 hdmi 屏幕不显 bios

N3530 工控机主板 BIOS 内置驱动对普通 HDMI 小屏幕兼容性不好，只有成功启动 PVE 系统后才会正常显示画面，而在自检阶段和 BIOS 界面只会显示白屏或者花屏，接大屏驱动板兼容性好点。

## 4.4 后面加装的 RTL8211 网卡不识别

原因：因为安装 pve 系统之前未加装好螃蟹卡 RTL8111，因此后续加装之后，pve 系统并未识别到。

解决方法：重装 pve 系统。（驱动源码编译方案，也有编译报错，不好解决，推荐是重装 pve，让 pve 自动打驱动）

## 4.5 工控机重启 bug，看门狗导致重启很长时间

原因：未查明，只要一重启，就会长时间卡在看门狗的错误提示。

错误提示：

```bash
BASHwatchgod: watchdog0: watchdog did't did not stop!

systemd-shutdown: Failed to finalzie DM services, ignorening.
```

解决方法：无法解决，只能长时间等待完成重启或者强制关机重开。

**P.S. 尝试在主板 BIOS 中主动启用看门狗计时器设置 `Advanced->SIO MISC Configuration->Watchdog Dog Timer->Enabled` 仍无法解决，该问题暂未查明原因。**

## 4.6 Repository 'Debian bookworm' changed its 'non-free component' value from 'non-free' to 'non-free non-free-firmware'

问题现象：运行 `apt update` 出现报错：Repository 'Debian bookworm' changed its 'non-free component' value from 'non-free' to 'non-free non-free-firmware'

问题原因：Debian 12 版本（bookworm）的非自有固件包（如硬件驱动程序）不再放在 non-free 目录中，而是 non-free-firmware 目录。

解决方法 1（修正 `/etc/apt/sources.list` 软件源）：

```bash
BASH# 确保软件源中添加了non-free-firmware目录，而不仅仅只有non-free目录

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware
```

解决方法 2（直接屏蔽该错误）：  
创建一个文件 `/etc/apt/apt.conf.d/no-bookworm-firmware.conf` ，写入内容如下：

```bash
BASHAPT::Get::Update::SourceListWarnings::NonFreeFirmware "false";
```

## 4.7 安装独立显卡的机器中 PVE 系统里面识别不到核显设备

问题现象：华硕 Z690-A Gaming WiFi D4 主板在安装独立显卡后，运行 `ls pci | grep VGA` 找不到核显的 VGA 设备。

问题原因：主板 BIOS 未有开启核显。

解决方法：

- 开启 iGPU 设置： `Advanced->System Agent (SA) Configuration->Graphics Configuration->iGPU Multi-Monitor->Enabled`
	- Intel 11 代及以后的 CPU 核显不支持传统 Legacy 显示方式，如果看不到在主板 BIOS 中找不到 iGPU 设置选项的话，请设置关闭 CSM： `BOOT->CSM (Compatibility Support Module)->Launch CSM->Disabled`
- 设置首选显示设备为核显： `Advanced->System Agent (SA) Configuration->Graphics Configuration->Primary Display->CPU Graphics`
	- 一般其他主板型号可能有的选项值：
		- Auto
		- CPU Graphics：核显
		- PEG Slot (PCI-e Graphics Device)：独立显卡
		- PCIE (PCI-e Graphics Device)
		- IGD (Internal Graphics Device)：一般多指核显，除非主板自带有北桥显示芯片
		- PCI (PCI Graphics Device)

## 4.8 PVE 概要页面利用率图标信息无法显示

问题原因：rrdcached 服务异常，某些情况下也可能会导致所有虚拟机显示「状态: unknown」。

解决方法：

```bash
BASHsystemctl stop rrdcached

rm -rf /var/lib/rrdcached/*

# 或者mv /var/lib/rrdcached /var/lib/rrdcached.bck

systemctl start rrdcached
```

## 4.9 PVE 面板节点和所有虚拟机显示「状态: unknown」，虚拟机正常运行

问题原因：pve 的节点状态守护程序 `pvestatd` 异常。

解决方法：

```bash
BASH#查看状态和重启服务

systemctl status pvestatd

systemctl restart pvestatd
```

## 4.10 使用 ventoy 引导安装 PVE 后，无法正常启动 PVE

问题：使用 ventoy 引导安装 PVE 后，安装完成重启，初次开机就卡在 `loading initial ramdisk` 或者黑屏

原因：因为 ventoy 偷偷给 PVE 添加启动内核参数 `rdinit=/vtoy/vtoy` 。据反复实验，在 ventoy 使用 grub 模式加载 PVE 镜像引导的话，这个参数百分百会被偷偷加入，使用正常 UEFI 模式加载偶尔会出现这种情况。测试使用的 ventoy 版本为 v1.1.05。

解决方法：

1. 强制重启主机，在出现 PVE 引导菜单时，按 E 键，进入 grub 编辑模式
2. 方向键定位到 `linux` 开头一行，删除 `rdinit=/vtoy/vtoy` 参数，Ctrl + X 启动配置，进入系统引导加载启动。
3. 登录 PVE 终端后台
4. 查看并修改所有涉及内核参数的文件
```bash
BASHnano /etc/default/grub

# 若有修改，执行更新

update-grub

nano /etc/kernel/cmdline

# 若有修改，执行更新

proxmox-boot-tool refresh

# 如果上述两个文件都不存在该参数，尝试检查以下文件

/boot/grub/grub.cfg

/etc/default/grub.d/init-select.cfg

/etc/default/grub.d/installer.cfg

/etc/default/proxmox-ve.cfg

/etc/default/grub.ucf-dist
```
1. 移除参数后，重启机器，PVE 启动应该就能恢复正常

## 4.11 KVM 和 LXC 被锁定 lock 无法启动

原因：回滚快照或者备份被异常中断都可能导致出现这种情况，对应的虚拟机状态可以看见一个被锁的图标。

解决方法：

- KVM 虚拟机解锁命名： `qm unlock vmid` ，例如 `qm unlock 100` 。
- LXC 虚拟机解锁命名： `pct unlock vmid` ，例如 `pct unlock 101` 。

## 4.12 Intel 13 / 14 代 CPU 搭建的 PVE 机器频繁宕机

问题现象：Intel 13 / 14 代 CPU 搭建的 PVE 机器经常会出现宕机现象，基本每隔两天就会出现失联，ssh 也无法正常连接到 PVE。此外还有一个很严重的现象，即未设置任何硬件直通的内核启动参数情况下，仍然会导致十分随机的 PVE 系统启动卡住现象或者出现随机启动报错，包括并不仅限于：

- [PVE 启动卡在 loading initial ramdisk](https://blog.csdn.net/Magic_Engine/article/details/144424476)
- [Proxmox stuck in /dev/mapper/pve-root: clean](https://forum.proxmox.com/threads/proxmox-stuck-in-dev-mapper-pve-root-clean.154567/) ：光标在下一行，并且光标不会闪烁，就显示一个下划线 `_` 。
- `BUG: Bad page state in process init pfn:11386d`
- 启动虚拟机经常遇到： [Failed to create the configured network](https://github.com/lxc/lxc/issues/2678) ，从而导致无法启动
- 无法正常卸载分区，reboot 系统也会卡住在 `unmount filesystem xxxx` （xxxx 指 UUID）
- 导致 PBS 备份失败： [ERROR: Backup of VM 102 failed - lvcreate snapshot 'pve/snap\_vm-102-disk-0\_vzdump' error](https://mister-kin.github.io/software/pve/#ERROR-Backup-of-VM-102-failed-lvcreate-snapshot-pve-snap-vm-102-disk-0-vzdump-error)

问题原因：CPU 缩缸（辣鸡 Intel (ಥ﹏ಥ)）。相信我，修改 grub 引导参数设置 `nomodeset` 什么的都不管用的，仍然是随机崩溃宕机或者开机随机卡住。

解决方法：为 CPU 设置 BIOS 锁频和稳压，确保 CPU 可以稳定运行工作。

如果是 1265u 的机器，可能是 C 状态的问题。编辑 `/etc/default/grub` ，在 `GRUB_CMDLINE_LINUX_DEFAULT` 追加 `intel_idle.max_cstate=2` 参数，然后 `update-grub` ，最后在 BIOS 中设置关闭 C 状态。

## 4.13 PVE 宕机故障的排查手段

- PVE 管理后台 ->PVE 节点 -> 系统 -> 系统日志
- 排障思路：
```bash
BASH# 查看系统版本

uname -a

# 查看PVE版本

pveversion -v

# 查看PVE启动日志

cd /var/log

last
```

## 4.14 PVE 宿主机更替硬件后导致失联无法访问网页后台

问题现象：PVE 更替硬件，重新开机后，直连 PVE 主机显示器显示启动正常，但是无法通过 IP 地址访问 PVE 管理后台。

原因：PVE 更替硬件，例如加装或者拆卸网卡/硬盘/显卡等设置，会导致 PCI 设备变动，对应的 PCI 编号变更。这样会导致网卡名称发生变动，但是 PVE 自身没法自动识别和更新这些信息，网络接口信息 `/etc/network/interfaces` 仍然是记录着旧的网卡名称信息，因此重启开机后就失联无法访问。

解决方法：

- 永久解决方案（推荐）：使用.link 文件固定网卡名字
	1. 查看当前所有物理网卡的 MAC 地址
		```bash
		BASHip addr
		# 输出信息中lan开头设备就是物理网卡，其中00:03:ee:00:xx:dc就是lan1的MAC地址
		# lan1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq master vmbr3 state DOWN group default qlen 1000
		# link/ether 00:03:ee:00:xx:dc brd ff:ff:ff:ff:ff:ff
		```
	2. 在 `/etc/systemd/network/` 路径下创建网卡对应的.link 文件 `10-persistent-net.link` ：
		```bash
		BASH[Match]
		MACAddress=00:03:ee:00:xx:dc
		[Link]
		Name=lan1
		```
	3. 其余网卡就按照 `1[x]-persistent-net.link` 和 `lan[x]` 的命名方式创建.link 文件和设置网卡名称即可。
- 临时解决方案（不推荐）：显示器直连 PVE 宿主机器，登录帐号。但这个方法不推荐，如果机器网卡很多的话，压根无法知道哪个网卡是以前的管理口。
	```bash
	BASH# 查看当前所有网卡名称
	ip addr
	# 把查看到新的网卡名称，更新到/etc/network/interfaces文件中
	nano /etc/network/interfaces
	systemctl restart network
	```

## 5 PVE 硬件直通

如果机器本身就支持 VT - X 和 VT-d 等 CPU 虚拟化和直通硬件功能，则 PVE 启用 IOMMU 方面不需要额外设置。只需要设置好主板 BIOS 对应的 CPU 虚拟化和直通硬件功能即可，因为安装 PVE 系统阶段就会自动设置好启动 IOMMU。

IOMMU 允许系统设备在虚拟内存中进行寻址，也就是将虚拟内存地址映射为物理内存地址，让实体设备可以在虚拟的内存环境中工作，这样可以帮助系统扩充内存容量，提升性能。

换而言之，IOMMU 可以使 VM 虚拟机能够接入一些物理设备，比如 PCIe 中的网卡、声卡、显卡，VM 虚拟机可以直接或间接使用这些设备。

> 在创建虚拟机时，芯片组一定要 q35。因为 Q35，才能 PCIE 直通，否则就是 PCI 直通。

## 5.1 检查硬件是否支持直通（BIOS 内查看 Advanced 高级选项）

1. 开启 CPU 虚拟化：Intel: VT - X (Intel Virtual Technology)。AMD: AMD - V (SVM, Secure Virtual Machine)。
2. 开启硬件直通功能：Intel: Intel VT-d。AMD: IOMMU。
3. 部分主板会有和 Intel VT-d/IOMMU 相关联的 `Memory Remap Feature` 设置。

如果主板 BIOS 中没有 VT-d 或者 IOMMU 功能选项，则很可能不支持直通硬件功能。一般确认 CPU 是否支持该特性即可，例如确认 Intel CPU 是否支持 VT-d，请查询 Intel 官方 CPU SKU 规格页面资料即可。

若机器 CPU 明确支持 VT-d 特性，但在主板 BIOS 中却找不到该选项的话，则很可能是机器主板 BIOS 默认设置关闭，并隐藏了该设置选项（这种情况常见于笔记本主板或者工控机主板）。

解决方法：使用 `UniversalAMDFormBrowser` 工具查看隐藏设定并开启。

1. 下载 `UniversalAMDFormBrowser` 工具
	- **方案 1（推荐）** ： [UniversalAMDFormBrowser-Ventoy](https://github.com/mio-19/UniversalAMDFormBrowser-Ventoy) 的 `UniversalAMDFormBrowser.img` 文件：该方案只需要将 img 文件拷贝到 ventoy 引导盘即可。
	- 方案 2： [Smokeless\_UMAF](https://github.com/DavidS95/Smokeless_UMAF) 的 `UniversalAMDFormBrowser.zip` 文件：该方案需要制作 FAT32 分区格式的 U 盘制作引导盘。
2. 重启机器，选择 U 盘启动项，引导加载 `UniversalAMDFormBrowser` 镜像。
3. 在 `UniversalAMDFormBrowser` 中查看隐藏设定，例如隐藏的 VT-d 选项或者解除功耗限制选项等。

## 5.2 PVE 提示No IOMMU detected

当 pve 未启用 IOMMU 或者硬件不支持直通（即使手动设置 PVE 开启 IOMMU）时，在 ProxmoxVE (PVE) 的 VM / VPS 中添加 PCI 设备时候提示： `No IOMMU detected, please activate it.See Documentation for further information`.

解决方法：请确认自己的机器是否支持 VT - X 和 VT-d 等 CPU 虚拟化和直通硬件功能，并是否已开启 IOMMU。若硬件不支持，建议更换支持这些功能特性的硬件设备。

## 5.3 PVE 启用 IOMMU

内核版本为 5.15 或之前的，在 ProxmoxVE (PVE) 系统内核中，需要手动启用 IOMMU。5.15 版本以后的 ProxmoxVE (PVE) 系统内核自带默认开启了 IOMMU 支持。

但我的硬件（N3530 工控机）本身不支持 VT-d 功能（ [N3530 SKU 规格页面明确写到不支持 VT-d](https://www.intel.cn/content/www/cn/zh/products/sku/81074/intel-pentium-processor-n3530-2m-cache-up-to-2-58-ghz/specifications.html) ），因此安装 PVE 时并未自动启用 IOMMU。

1. 编辑 grub 文件： `nano /etc/default/grub`
2. 找到 GRUB\_CMDLINE\_LINUX\_DEFAULT 这一行：
```
PLAINTEXT...

GRUB_DEFAULT=0

GRUB_TIMEOUT=5

GRUB_DISTRIBUTOR=\`lsb_release -i -s <strong>2</strong>> /dev/null || echo Debian\`

GRUB_CMDLINE_LINUX_DEFAULT="quiet"

GRUB_CMDLINE_LINUX=""

...
```
1. 根据 CPU 类型修改

```
PLAINTEXT...

GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"

...
```

```
PLAINTEXT...

GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on"

...
```

4\. 更新 grub：\`update-grub\` 5. 重启 PVE

1. 编辑 cmdline： `nano /etc/kernel/cmdline` 。（若不存在，则新建该文件）
2. 根据 CPU 类型在第一行末尾添加：

```
PLAINTEXTquiet intel_iommu=on
```

```
PLAINTEXTquiet amd_iommu=on
```

## 5.4 验证 PVE 的 IOMMU 是否启用

1. 查询命令： `dmesg | grep -e DMAR -e IOMMU` 。输出显示 `DMAR: IOMMU enabled` 或者 `DMAR: Intel(R) Virtualization Technology for Directed I/O` 证明已启用。
2. 实操验证：虚拟机分配 PCI 设备时可见。

## 5.5 PVE 启用 PT 模式

PT 模式：会在 IOMMU 需要使用时候启动，适配器不需要使用 DMA 转换到内存，因此可以提高其他没有分配过设备的性能。

1. 编辑 grub 文件： `nano /etc/default/grub`
2. 找到 GRUB\_CMDLINE\_LINUX\_DEFAULT 这一行：
```
PLAINTEXT...

GRUB_DEFAULT=0

GRUB_TIMEOUT=5

GRUB_DISTRIBUTOR=\`lsb_release -i -s <strong>2</strong>> /dev/null || echo Debian\`

GRUB_CMDLINE_LINUX_DEFAULT="quiet"

GRUB_CMDLINE_LINUX=""

...
```
1. 附加参数 iommu = pt，Intel 和 AMD 芯片均可使用这个参数。
2. 添加模块设置（5.4 内核需自行添加，现行版本自带有添加）： `nano /etc/modules` 。没有如下内容需添加。
```
PLAINTEXTvfio

vfio_iommu_type1

vfio_pci

vfio_virqfd
```
1. 刷新 initramfs： `update-initramfs -u -k all`
2. 更新 grub： `update-grub`

## 5.6 直通 i9-13900K 核显 UHD770 给虚拟机 Windows11

### 5.6.1 Intel 核显直通 optionROM 方案

QEMU 9.2.0 后的版本无法使用 igd 的 ROM 直通核显。例如 PVE8.3.1 版本的 QEMU 就不支持。回退到 9.1.3 版本再操作直通核显： `apt install pve-qemu-kvm=9.1.2-3` 。

注意 BIOS 设定：DVMT pre allocated，不要大过 64M，64M 对应 x-igd-gms=0x2，如果超过 64M,x-igd-gms 要加大！

1. 访问 [igd](https://github.com/gangqizai/igd) 仓库，在仓库代码根路径下，下载 [`gen12_gop.rom`](https://github.com/gangqizai/igd/blob/main/gen12_gop.rom) 和 [`gen12_igd.rom`](https://github.com/gangqizai/igd/blob/main/gen12_igd.rom) 两个文件。
2. 将这两个文件复制到 `/usr/share/kvm/` 目录下。
3. 编辑 Windows 虚拟机配置文件（示例 Windows 虚拟机 ID 为 103）：
```bash
BASHnano /etc/pve/qemu-server/103.conf

# 添加下面三行

args: -set device.hostpci0.addr=02.0 -set device.hostpci0.x-igd-gms=0x2 -set device.hostpci0.x-igd-opregion=on -debugcon fil>bios: ovmf

hostpci0: 0000:00:02.0,legacy-igd=1,romfile=gen12_igd.rom

hostpci1: 0000:00:1f.3,romfile=gen12_gop.rom

# 以下是测试工作正常的完整配置

args: -set device.hostpci0.addr=02.0 -set device.hostpci0.x-igd-gms=0x2 -set device.hostpci0.x-igd-opregion=on

bios: ovmf

boot: order=sata0

cores: 16

cpu: host

efidisk0: local-lvm:vm-103-disk-0,efitype=4m,size=4M

hostpci0: 0000:00:02.0,legacy-igd=1,romfile=gen12_igd.rom

hostpci1: 0000:00:1f.3,romfile=gen12_gop.rom

machine: pc-i440fx-8.0

memory: 20480

meta: creation-qemu=9.2.0,ctime=1744108457

name: windows

net0: virtio=BC:24:11:C9:53:72,bridge=vmbr1,firewall=1

numa: 0

onboot: 1

ostype: win11

sata0: local-lvm:vm-103-disk-1,size=200G,ssd=1

sata1: win-hdd:103/vm-103-disk-0.raw,backup=0,size=1007G

scsihw: virtio-scsi-single

smbios1: uuid=eddc911c-c81f-4d48-9ae2-f8c7f6cee2c5

sockets: 1

startup: order=4,up=1

tpmstate0: local-lvm:vm-103-disk-2,size=4M,version=v2.0

vga: none

vmgenid: dc473f54-0098-45f9-92b8-d99320460696
```
1. 注意在启动前，将虚拟机 -> 硬件 -> 显示 -> 设置为「无 (none)」，即对应虚拟机配置文件的参数： `vga: none`
2. 通过远程桌面连接 RDP 接入虚拟机系统，下载并安装最新的 Intel 的核显驱动（已测试正常的驱动版本 `32.0.101.6732` ，驱动程序日期：2025/4/1）。

## 6 创建 PVE 虚拟机 / LXC 容器

从配置文件中修改虚拟机或者 LXC 的配置参数（请自行修改对应 ID）：

- 虚拟机：nano /etc/pve/qemu-server/103.conf
- LXC：nano /etc/pve/lxc/101.conf

> 若机器硬件性能不太足够的话，不建议安装 win 系统，win 系统极度消耗硬件性能（即便是空载待机状态也消耗很大），而这影响 openwrt 的性能。比如我的情况：安装了 tiny10 系统，千兆网线，宽带 500M 实测不到 300Mbp / s（这里是指用 [广东联通宽带测速平台](http://speed.unicomgd.com:8080/) 测试的结果，实际 iKuai 主路由测试速率却又正常），局域网速率最高只能到 700Mbp/s 的速率。关闭后 tiny10 后，实测宽带速率恢复正常，局域网速率跑到 900Mbp/s 出头）

## 6.1 PVE 基础设置

1. 访问 WEB 管理地址并登录： [https://192.168.1.2:8006](https://192.168.1.2:8006/) ，账户：root。密码：\[前面安装步骤设置的密码\]。
2. 根据硬件网卡情况，添加并设置好虚拟网桥 Linux Bridge：创建 > Linux Bridge > 桥接接口填写物理网口地址名称。以我的机器为例，机器三个物理网口，PVE 新建 5 个网桥，前三个网桥和物理网口绑定，第四个网桥用作 iKuai 和 OpenWrt 之间的传输，第五个网桥用作其他虚拟机系统的传输。

## 6.2 安装 LXC 容器后的基础设置

### 6.2.1 适用于 CT 模板（debian-12-standard\_12.7-1\_amd64）

```bash
BASHmv /etc/apt/sources.list /etc/apt/sources.list.bak

nano /etc/apt/sources.list

# deb http://ftp.debian.org/debian bookworm main contrib

deb https://mirrors.ustc.edu.cn/debian/ bookworm main contrib non-free non-free-firmware

# deb http://ftp.debian.org/debian bookworm-updates main contrib

deb https://mirrors.ustc.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware

deb https://mirrors.ustc.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware

# security updates

# deb http://security.debian.org bookworm-security main contrib

deb https://mirrors.ustc.edu.cn/debian-security bookworm-security main contrib non-free non-free-firmware

timedatectl set-timezone Asia/Shanghai

apt install locales -y

dpkg-reconfigure locales

init 6
```

## 6.3 安装爱快 iKuai（KVM 虚拟机)

1. 下载 iKuai 镜像： [点击跳转](https://www.ikuai8.com/component/download)
2. local - 内容 - 上传：iKuai 的安装镜像文件。
3. 创建虚拟机，命名虚拟机名称 iKuai，VM 编号 100。
4. CD / DVD 光盘镜像文件选择刚上传的 ISO 镜像。
5. 主硬盘分配 2G，额外添加 EFI 磁盘（不勾选添加密钥），CPU 给 4 核（host），内存分配 4G（64 位 iKuai 要求 4G 内存），网卡模型选择 VirtIO（半虚拟化）。
6. 完成新建后，在硬件标签页中继续完成添加所有网桥，网桥的防火墙全部关掉（使用 iKuai 的），模型都选择半虚拟化。
7. 在选项标签页中把「开机自启动」开启，启动顺序设置为 1，引导顺序只开 disk‘xxxx’磁盘，把网络启动关掉，CD 启动上移到第一位。
8. 点击启动虚拟机，完成安装即可。
9. iKuai 内识别的第一个网卡设置为 lan1。
10. 输入字母 o 回车进入其「其他选项」，开启外网访问 web。
11. 访问 WEB 管理地址并登录： [http://192.168.1.1:80](http://192.168.1.1/) 。默认账户：admin。默认密码：admin。

## 6.4 安装 OpenWrt（KVM 虚拟机）

### 6.4.1 OpenWrt 镜像的选择

- [Bleach OpenWrt](https://openwrt.mpdn.fun:8443/?dir=lede/x86_64) ：推荐（本教程使用该固件）。内置 SmartDNS，软件包空间设置为 1G，剩余五百多兆。
- [eSir GDQ 高大全](https://drive.google.com/drive/folders/1PsS3c0P7a4A4KY8plQg4Fla8ZI-PGBb1) ：eSir 的高大全固件。不推荐。没有内置 SmartDNS 拓展（我测试时安装 SmartDNS 并未出现有服务项，无效）。并且相较于 Blench 版本，较为繁琐。软件包空间只剩余几兆，需自己手动扩盘。

### 6.4.2 安装步骤

1. 创建虚拟机，命名虚拟机名称 openwrt，VM 编号 101。
2. CD / DVD 光盘镜像文件设置为无介质。
3. 主硬盘随便分配（后续会删除），额外添加 EFI 磁盘（不勾选添加密钥），CPU 给 4 核（KVM64），内存分配 4G，网卡模型选择 VirtIO（半虚拟化）。
4. 网桥只选择第四个网桥（用作 iKuai 和 OpenWrt 之间的传输），网卡的防火墙全部关掉（使用 iKuai 的），模型选择半虚拟化。
5. 下载 OpenWrt 镜像： [见上一小节](https://mister-kin.github.io/software/pve/#OpenWrt%E9%95%9C%E5%83%8F%E7%9A%84%E9%80%89%E6%8B%A9)
6. local - 内容 - 上传：OpenWrt 的镜像文件。
7. 向虚拟机导入 OpenWrt 镜像： `qm importdisk 101 /var/lib/vz/template/iso/bleach-plus-20230826-openwrt-x86-64-generic-squashfs-combined-efi.img local-lvm` 。（直接写盘即可，无需安装）。
8. 在 pve 管理页面中，加载刚才导入镜像生成的磁盘。
9. 在选项标签页中把「开机自启动」开启，启动顺序设置为 2，引导顺序只开 disk‘xxxx’磁盘，把网络启动关掉。
10. 点击启动虚拟机。
11. pve 中用 shell： `vi /etc/config/network` ，将 lan 口 ip 改为 `192.168.1.3` ， `reboot` 重启。vim 编辑：按 i 进入修改模式，按 esc 退出编辑模式，输入:wq 回车保存修改。
12. 访问 WEB 管理地址并登录： [http://192.168.1.3](http://192.168.1.3/) 。默认账户：root。默认密码：password。

## 6.5 安装 OpenWrt（LXC 容器）

### 6.5.1 OpenWrt 根文件镜像的选择

- [ImmortalWrt](https://firmware-selector.immortalwrt.org/?target=x86/64&id=generic) ：推荐（本教程使用该固件）。选择 `ROOTFS (SQUASHFS)` 类型下载，编写教程时最新版本为 `immortalwrt-24.10.1-x86-64-generic-squashfs-rootfs.img.gz` 。
- [OpenWrt 官方固件](https://downloads.openwrt.org/)

### 6.5.2 安装步骤

1. 在 PVE 宿主机上安装 `squashfs-tools` 软件包： `apt install squashfs-tools` 。
2. 下载 OpenWrt 根文件镜像： [见上一小节](https://mister-kin.github.io/software/pve/#OpenWrt%E6%A0%B9%E6%96%87%E4%BB%B6%E9%95%9C%E5%83%8F%E7%9A%84%E9%80%89%E6%8B%A9)
3. 解压 gz 包： `gunzip immortalwrt-24.10.1-x86-64-generic-squashfs-rootfs.img.gz` （gunzip 解压 gz 包之后会自动删除 gz 包的）。
4. 解压 img 镜像文件： `unsquashfs openwrt-x86-64-generic-squashfs-rootfs.img` ，解压路径为当前路径下的 `squashfs-root` 文件夹。
5. 打包 tar.gz 格式，作为 LXC 模版：
```bash
BASHcd squashfs-root

tar zcf /var/lib/vz/template/cache/immortalwrt-24.10.1-x86-64-generic-rootfs.tar.gz ./*

# 完成之后PVE的local的CT模板路径中就可以看到immortalwrt-24.10.1-x86-64-generic-rootfs.tar.gz文件了
```
1. 手动创建 LXC 容器（这里不能用网页端选择 LXC 模板来创建，会报错），若需要后期编辑 LXC 虚拟机的配置项，使用命令编辑 `nano /etc/pve/lxc/101.conf` 。
```bash
BASHpct create 101 /var/lib/vz/template/cache/immortalwrt-24.10.1-x86-64-generic-rootfs.tar.gz --rootfs local-lvm:2 --ostype unmanaged --hostname openwrt --arch amd64 --cores 4 --memory 2048 --swap 0 -net0 bridge=vmbr4,name=eth0 -net1 bridge=vmbr4,name=eth1 --features nesting=1 --unprivileged 0
```

参数说明：

- `pct create 100`: 创建一个 ID 为 100 的容器。
- `/var/lib/vz/template/cache/openwrt-x86-64-generic-rootfs.tar.gz`: 指定要使用的根文件系统模板的路径。这里是 OpenWRT 的根文件系统。
- `--rootfs local-lvm:2`: 设置容器的根文件系统存储位置。local-lvm 表示使用 local-lvm 的空间，2 表示分配 2GB 的存储空间。
- `--ostype unmanaged`: 指定操作系统类型为 "unmanaged"，表示 Proxmox 不会对已知操作系统进行特定优化。
- `--hostname openwrt`: 将容器的主机名设置为 openwrt。
- `--arch amd64`: 指定容器的架构为 64 位（amd64）。
- `--cores 4`: 分配 4 个 CPU 核心给容器。
- `--memory 2048`: 分配 2048 MB 的内存给容器。
- `--swap 0`: 分配 0 MB 的交换空间给容器。
- `--net0 bridge=vmbr0,name=eth0`: 配置第一个网络接口，连接到 vmbr0 桥接，并将接口命名为 eth0。
- `--features nesting=1`: 启用特定功能。nesting = 1 允许嵌套虚拟化。
- `--unprivileged 0` ：表示使用特权模式（该选项一定要开启，否则 OpenWrt 的 DNSMasq 服务无法正常启动）
1. 为容器追加额外的参数设置 `nano /etc/pve/lxc/101.conf`
```bash
BASHlxc.cgroup2.devices.allow: a

lxc.cap.drop:

lxc.mount.auto: proc:mixed sys:ro cgroup:mixed

lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

参数说明：这些配置项主要用于确保容器能够正常访问网络设备，并设置适当的权限和挂载点，以便 OpenWRT 能够正常工作。

- `lxc.cgroup2.devices.allow: a` ：允许容器访问所有设备。a 表示所有设备权限。
- `lxc.cap.drop:`：该参数用于指定要从容器中删除的 Linux 功能（capabilities）。如果没有指定任何功能，表示不删除任何功能。此项通常用于限制容器的特权。
- `lxc.mount.auto: proc:mixed sys:ro cgroup:mixed` ：
	- `lxc.mount.auto` 用于自动挂载文件系统。
	- `proc:mixed` 表示以混合模式挂载 `/proc` 文件系统，允许容器访问进程信息。
	- `sys:ro` 表示以只读模式挂载 `/sys` 文件系统，容器可以读取系统信息但不能修改。
	- `cgroup:mixed` 表示以混合模式挂载 `cgroup` 文件系统，用于管理资源限制。
- `lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file` ：该参数用于手动挂载设备。
	- `/dev/net/tun` 是 TUN / TAP 虚拟网络设备的路径。
	- `dev/net/tun` 是容器内的挂载点。
	- `none` 表示不指定文件系统类型。
	- `bind` 表示将主机的设备绑定到容器中。
	- `create=file` 表示如果挂载点不存在则创建它。
1. 在 PVE 管理网页打开进入 LXC 的 shell，修改固件的 LAN 口 IP，让 OpenWrt 可以访问外网：
```bash
BASHvim /etc/config/network

service network restart

# 配置文件内容（请按自己的网络拓扑需求进行修改）

config interface 'loopback'

        option device 'lo'

        option proto 'static'

        option ipaddr '127.0.0.1'

        option netmask '255.0.0.0'

config globals 'globals'

        option ula_prefix 'fdbb:d148:dd0e::/48'

        option packet_steering '1'

config device

        option name 'br-lan'

        option type 'bridge'

        list ports 'eth0'

config interface 'lan'

        option device 'br-lan'

        option proto 'static'

        option ipaddr '10.10.11.1'

        option netmask '255.255.255.0'

        option ip6assign '60'

config interface 'wan'

        option device 'eth1'

        option proto 'static'

        option ipaddr '10.10.10.3'

        option netmask '255.255.255.0'

        option gateway '10.10.10.1'

        list dns '114.114.114.114'
```
1. 安装软件包：
	1. 美化皮肤： `luci-i18n-argon-config-zh-cn`
	2. OpenClash： `luci-app-openclash`

## 6.6 安装飞牛私有云 fnOS（KVM 虚拟机）

1. 下载飞牛私有云 fnOS 镜像： [点击跳转](https://www.fnnas.com/download)
2. local - 内容 - 上传：fnOS 的安装镜像文件。
3. 创建虚拟机，命名虚拟机名称 fnOS，VM 编号 102。
4. CD / DVD 光盘镜像文件选择刚上传的 ISO 镜像。
5. 主硬盘分配 64G，额外添加 EFI 磁盘（不勾选添加密钥），CPU 给 4 核（host），内存分配 6G（fnOS 系统内存需求还是有点大的），网卡模型选择 VirtIO（半虚拟化）。
	- 其他硬盘后续自行根据需求添加。
6. 网桥只选择第五个网桥（用作 iKuai 和 fnOS 之间的传输），网卡的防火墙全部关掉（使用 iKuai 的），模型选择半虚拟化。
7. 在选项标签页中把「开机自启动」开启，启动顺序设置为 3，引导顺序只开 disk‘xxxx’磁盘，把网络启动关掉，CD 启动上移到第一位。
8. 点击启动虚拟机。
9. 选择 `Graphical Install` 进行图形化安装流程。
10. 选择系统主硬盘进行安装。
11. 系统分区大小：64GB；Swap 大小：0GB。（有论坛帖子反馈开启 `Swap cache` 时，某些下载应用例如 `QBittorrent` 可能会引导硬盘持续读取和 CPU 的异常占用。永久禁用 Swap 还需要修改 fstab 文件。）
12. 点击确认安装，选择网卡界面，直接默认选项确认即可。
13. 安装完成后，点击确认开始重启。摁 `ESC` 键，进入虚拟机 BIOS 设置界面，重新选择硬盘分区作为启动项启动。（否则默认启动 CD）
14. 出现 `Debian GNU/Linux` 系统引导选项就是安装成功了
15. 进入到传统 Linux 登录界面（无 GUI），这里能看到 WEB 管理地址。若需要固定 IP 地址，在 `iKuai` 中找到对应 MAC 地址加入静态分配表即可。
16. 访问 WEB 管理地址并登录： [http://192.168.1.5:5666/](http://192.168.1.5:5666/) 。首次访问会提示创建 NAS `设备名称``超级管理员账号``密码``确认密码` 。

### 6.6.1 飞牛私有云 fnOS 的硬件直通设置

新版 PVE 无需额外设置什么直通参数，系统内核自带默认开启了 IOMMU 支持。

需要额外注意的一点是：安装飞牛系统时先不要加 PCI 设备，照常安装飞牛 fnOS 系统，正常设置完成之后，再关闭虚拟机添加 PCI 设备，例如核显和 sata 控制器（添加 PCI 设备 -> 原始设备时，按默认参数即可，也无需勾选「所有功能」）。

- 核显设备：Alder Lake-S GT1 \[UHD Graphics 730\]
- SATA 控制器：Alder Lake-S PCH SATA Controller \[AHCI Mode\]

## 6.7 安装 Proxmox Backup Server（PBS 备份服务器）（LXC 容器）

1. 创建 LXC 容器配置（debian-12-standard\_12.7-1\_amd64）
	- 核心：2
	- 内存：512MB
	- SWAP：512MB
	- 磁盘：10GB
	- 网桥和网卡：eth0，vmbr4， `10.10.10.8/24` ，10.10.10.1（请根据自行实际情况设置）
2. 替换 debian 系统镜像源和配置系统时区。
3. 添加 Proxmox Backup Server 的软件源
	- 方案一：配置 `pbs-enterprise.list`
		```bash
		BASHecho deb http://mirrors.ustc.edu.cn/proxmox/debian/pbs bookworm pbs-no-subscription > /etc/apt/sources.list.d/pbs-enterprise.list
		```
	- 方案二： `nano /etc/apt/sources.list`
		```bash
		BASHdeb http://mirrors.ustc.edu.cn/proxmox/debian/pbs bookworm pbs-no-subscription
		```
4. 添加 Proxmox 的 GPG Key，使用 APT 安装 proxmox-backup-server 包（可以尝试直接安装看看，若报错再下载 GPG KEY）
```bash
BASHwget https://enterprise.proxmox.com/debian/proxmox-release-bookworm.gpg -O /etc/apt/trusted.gpg.d/proxmox-release-bookworm.gpg

apt update && apt install proxmox-backup-server -y
```
1. 提示 `pbs-enterprise.list` 配置文件修改意见，使用默认选项 NO，即保持当前配置文件，不作任何修改。
```bash
BASHConfiguration file '/etc/apt/sources.list.d/pbs-enterprise.list'

 ==> File on system created by you or by a script.

 ==> File also in package provided by package maintainer.

   What would you like to do about it ?  Your options are:

    Y or I  : install the package maintainer's version

    N or O  : keep your currently-installed version

      D     : show the differences between the versions

      Z     : start a shell to examine the situation

 The default action is to keep your current version.

*** pbs-enterprise.list (Y/I/N/O/D/Z) [default=N] ?
```
1. 访问 WEB 管理地址并登录： [https://10.10.10.8:8007/](http://10.10.10.8:8007/) 。（root 账户登录，领域为 `Linux PAM standard authentication` 。注意别选错为 `Proxmox Backup authentication server` ，这个是给普通 pbs 用户使用）。
2. 去除 Proxmox Backup Server 未订阅提示（和 PVE 操作类似，但只需要修改 `/usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js` 文件），注销浏览器当前的 PBS 登录信息，并重新登录就没有提示弹窗了。
1. 查看并禁用不需要的 failed 服务（ `systemctl` 命令会输出服务状态情况）

### 6.7.1 挂载新硬盘到宿主机并设置 PVE 的 PBS 备份

1. 安装新硬盘用于专门存储 PBS 的备份数据，这里对应的是 `/dev/sdc` 。
2. 在宿主机 PVE 中，直接格式化整个硬盘为 ext4，并挂载到 `/mnt/backup-server-data` 目录。
```bash
BASHmkfs.ext4 /dev/sdc

# 查看并复制uuid，编辑/etc/fstab文件，以便开机自动挂载硬盘

blkid | grep /dev/sdc

mkdir /mnt/backup-server-data

nano /etc/fstab

UUID=复制的uuid值 /mnt/backup-server-data ext4 defaults 0 2

# 挂载/etc/fstab中列出的所有文件系统

mount -a

# PVE可能会提示如下信息要求重载

# mount: (hint) your fstab has been modified, but systemd still uses

#        the old version; use 'systemctl daemon-reload' to reload.

systemctl daemon-reload

# 查看挂载情况

lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINT
```
1. 将宿主机 PVE 的挂载路径映射给 PBS 的 LXC 容器。
```bash
BASH# 将宿主机的/mnt/backup-server-data映射到LXC容器106的客户机 /mnt/data 路径中

pct set 106 -mp0 /mnt/backup-server-data,mp=/mnt/data
```
1. 设置数据目录的属主 / 属组（Unprivileged 非特权容器一定要设置这个操作，否则在下一步的添加数据存储就会报错： `TASK ERROR: EPERM: Operation not permitted` ）
```bash
BASH# 查看proxmox-backup-api的属主/属组

stat -c "%u %g" /proc/$(pgrep -f proxmox-backup-api)

# 命令输出为100000 100000

# 修改属主/属组

chown 100000:100000 /mnt/backup-server-data
```
1. 在 PBS 后台管理界面中，创建数据存储：数据存储 -> 添加数据存储。
	- 常规：名称：HDD，备份路径：/mnt/data，GC 时间表：daily，精简时间表：daily。（默认的 daily 表示每天 00:00）
	- 精简选项：参见 [PBS 服务器备份精简计划](https://mister-kin.github.io/software/pve/#PBS%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%A4%87%E4%BB%BD%E7%B2%BE%E7%AE%80%E8%AE%A1%E5%88%92)
2. 创建命名空间：点击 `HDD` 这个数据存储 -> 内容 -> 新增命名空间 -> `pve-backup`
	- 每个数据存储中可以包含多个命名空间，命名空间并不隔离访问权限与存储设备，仅隔离备份名称。这样子可以实现，多个不同 PVE 节点可以备份存储在不同的命名空间。
3. 创建普通账户给 PVE 设置备份使用：配置 -> 访问控制 -> 添加 -> 用户名：admin，领域： `Proxmox Backup authentication server` ，密码：自己设置。
	- 除仅用于登录管理界面的 root 用户使用 PAM 认证外（即 `Linux PAM standard authentication` ），其他管理员用户或备份客户端均使用 Proxmox Backup Server 自带的用户管理系统进行认证（即 `Proxmox Backup authentication server` ）。
	- 如果需要设置邮件通知，双击对应帐号，在弹窗里填写 `E-Mail` 一栏即可。
4. 为上一步创建的帐号设置权限：配置 -> 访问控制 -> 权限 -> 用户权限 -> 路径：/datastore/HDD，用户：admin@pbs，角色：DatastoreAdmin。（该角色方便在 PVE 中管理备份）
5. 回到宿主机 PVE 后台管理界面中，将 Proxmox Backup Server 添加为存储（Storage）：数据中心 -> 存储 -> 添加 ->Proxmox Backup Server
	- 常规：ID：pve-pbs，服务器：10.10.10.8，用户名：admin@pbs，密码： ，Datastore：HDD，命名空间：pve-backup，指纹：\[在 PBS 管理界面首页中查看：仪表板 -> 显示指纹\]
	- 备份保留：默认设置，该功能直接由 PBS 接管即可，无需在这里设置。
	- 加密：如果是首次添加 PBS 存储，选择自动生成客户端加密密钥。不是首次添加，可以选择上传密钥。注意保存好密钥。
6. 在 PVE 中设置备份任务：数据中心 -> 备份 - 添加：自行选择需要备份的虚拟机 ID。节点：PVE 节点，存储：pve-pbs，计划：\*/1:00，选择模式：所有，压缩：灰色选项（因为设置了 pve-pbs 备份存储空间，该选项无法调整），模式：快照。
	- 如果是生产数据，比较重要，建议设置备份时间为 30 分钟一次或者更短时间。
	- 如果需要不同的备份时间频率，可以设置多个备份任务。
7. 点击备份任务计划，点击 `现在运行` ，查看备份效果。

### 6.7.2 配置 PBS 日志轮转清理

> 这一步很重要，否则长期备份的日志文件会填满的机器根目录。  
> 如果设置备份间隔半小时的话，设置 60 天轮转一次，PBS 占用大约在 9.21GB；设置 30 天轮转一次，大约占用是 6.03GB。请根据自己 PBS 本体安装时的硬盘大小设置合适的轮转周期。

配置方法（保留 60 天日志文件，一般是凌晨 00:00 执行轮转任务）：

```bash
BASHproxmox-backup-manager node update --task-log-max-days 60
```

日志轮转成功执行后，在 PBS 的网页后台管理菜单 -> 任务：日志轮换，可以查看到相关日志信息

```bash
BASH2025-07-03T00:00:00+08:00: starting task log rotation

2025-07-03T00:00:00+08:00: task log archive was rotated

2025-07-03T00:00:00+08:00: API access log was not rotated

2025-07-03T00:00:00+08:00: API authentication log was not rotated

2025-07-03T00:00:00+08:00: cleaning up old task logs

2025-07-03T00:00:00+08:00: TASK OK
```

#### 6.7.2.1 Linux 日志轮转 logrotate 服务

logrotate 是基于 crontab 运行的 (准确来说是基于 timer 和 crontab 定时机制)：

- 配置文件
	- 主配置文件： `/etc/logrotate.conf` ，定义全局轮转规则。
	- 子配置文件： `/etc/logrotate.d/` 目录下的文件，定义特定日志的轮转规则。
		- 创建自定义的轮转任务 `mylog` ：
			```bash
			BASH# 创建mylog文件
			nano /etc/logrotate.d/mylog
			# 填入如下内容
			/var/log/mylog.log { # 要轮转的日志文件路径，支持通配符（如 /var/log/mylog/*.log）
			    daily # 每日轮转
			    rotate 10 # 保留7个备份，mylog.log.1.gz 到 mylog.log.7.gz
			    size 10M # 当日志文件超出10M时，轮转
			    compress # 压缩旧的日志文件
			    missingok # 若日志文件不存在，不报错
			    notifempty # 若日志文件为空，不轮转
			}
			# 修改文件权限，只有拥有者有读写权限；而属组用户和其他用户只有读权限。
			chmod 644 /etc/logrotate.d/mylog
			# 模拟轮转任务，-d是调试模式
			logrotate -d /etc/logrotate.conf
			# 模拟mylog的轮转任务
			logrotate -d /etc/logrotate.conf/mylog
			# 强制执行轮转任务
			logrotate -f /etc/logrotate.conf
			```

查看相关轮转任务后台服务的命令：

```bash
BASH# 查看logrotate版本

logrotate --version

systemctl status cron.service

systemctl status logrotate.service

systemctl status logrotate.timer

# 查看Linux系统内置的logrotate服务轮转状态

cat /var/lib/logrotate/status
```

## 7 创建 PVE 虚拟机 LXC 容器遇到的坑

## 7.1 使用debian-12的 CT 模板创建 LXC 容器 SSH 终端中文乱码

```bash
BASHapt install locales -y

# 需要重新设置语言的，也可以重新执行该命令

dpkg-reconfigure locales

# 使用Page Up和Page Down翻页查找这两种语言，使用空格确认复选框，tab键切换「确认」，然后回车执行

# en_US.UTF-8 UTF-8

# zh_CN.UTF-8 UTF-8

# 然后重启机器

init 6
```

## 7.2 非特权（unprivileged）LXC 容器无法设置vm.max\_map\_count内核参数

报错提示： `error: permission denied on key 'vm.max_map_count'`

报错原因：因为非特权的 LXC 容器的 `/proc/sys` 是以只读方式挂载到容器里面，无法修改 sysctl 内核参数。

解决方法：在 PVE 宿主机上设置 `vm.max_map_count` 内核参数， `max_map_count` 默认值是 262144，而我们需要设置为 2000000。

```bash
BASH# 持久化设定，永久生效，推荐使用该方式

nano /etc/sysctl.conf

# 在/etc/sysctl.conf最底部添加如下参数，保存文件

vm.max_map_count=2000000

# 从文件中加载参数，不指定文件默认加载读取 /etc/sysctl.conf 的内容，会立即应用其中的内核参数（无需重启系统）

sysctl -p

# 临时设置生效，重启PVE后会恢复到默认值，而不是2000000

sysctl -w vm.max_map_count=2000000

# 查看参数值

sysctl vm.max_map_count
```

### 7.2.1 补充知识点：namespace 化的 sysctl 内核参数

`vm.max_map_count` 属于内核参数，而 `vm.*` 参数并没有 namespace 化。只有 namespace 化的 sysctl 内核参数在容器内直接修改设置的时候，不会影响宿主机和其他容器。

已经 namespace 化的 sysctl 内核参数：

- kernel.shm\*
- kernel.msg\*
- kernel.sem
- fs.mqueue.\*
- net.\*
	- net.netfilter.nf\_conntrack\_max（Linux 5.12.2 之前并未 namespace 化）
	- net.netfilter.nf\_conntrack\_expect\_max（Linux 5.12.2 之前并未 namespace 化）

给 LXC 容器单独设置 sysctl 内核参数的方式：

- 为 LXC 容器开启特权选项 `--privileged` ，对应 `/proc` 目录的挂载方式为 `rw` ，可以直接在容器内使用命令修改参数了。若修改的是 `namespaced` 参数，则不会影响宿主机和其他容器。
- 把 `/proc/sys` bind 到容器里面，该方式和 `--privileged` 效果类似。

给 docker 容器单独设置 sysctl 内核参数的方式：

- `--privileged` ： `docker run --privileged -it ubuntu bash` ，该方式启动后，可以通过命令任意修改 sysctl 参数。
- 把 /proc/sys bind 到容器里面，再具体写值，和上面的效果类似。
	1. `docker run -v /proc/sys:/writable-sys -it ubuntu bash`
	2. `echo 62 > /writable-sys/net/ipv4/ip_default_ttl`
- `--sysctl` ： `docker run -it --sysctl 'net.ipv4.ip_default_ttl=63' ubuntu sysctl net.ipv4.ip_default_ttl` 。
	- 只有 namespaced 参数才可以。否则会报错 "invalid argument..."
	- 这种方式只是在容器初始化过程中完成内核参数的修改，容器运行起来以后，/proc/sys 仍然是以只读方式挂载的，在容器中不能再次修改 sysctl 内核参数。

## 7.3 创建的 LXC 容器debian-12系统时间与宿主机时间不一致

使用 `debian-12` 的 CT 模板默认使用的是世界时设置，比东八区时间慢八小时，因此需要手动设置时区。

- 方法 1： `timedatectl set-timezone Asia/Shanghai`
- 方法 2： `dpkg-reconfigure tzdata`

验证修改是否成功： `timedatectl`

## 7.4 在 PVE 管理页面工具选择 OpenWrt 的自建 LXC 模板来创建 LXC 会报错

报错提示：

```bash
BASH  Logical volume "vm-101-disk-0" created.

Creating filesystem with 524288 4k blocks and 131072 inodes

Filesystem UUID: 0d6d0789-09cb-4564-95be-c04ba88481bb

Superblock backups stored on blocks:

    32768, 98304, 163840, 229376, 294912

extracting archive '/var/lib/vz/template/cache/immortalwrt-24.10.1-x86-64-generic-rootfs.tar.gz'

Total bytes read: 39229440 (38MiB, 176MiB/s)

unknown ID 'immortalwrt' in /etc/os-release file, trying fallback detection

  Logical volume "vm-101-disk-0" successfully removed.

TASK ERROR: unable to create CT 101 - unable to detect OS distribution
```

报错原因：PVE 管理页面工具无法识别该 LXC 模板的系统类型。

解决方法：使用 `pct create` 手动创建 LXC 容器。

## 7.5 非特权模式的 OpenWrt 的 LXC 容器网络访问问题

使用 `unprivileged 1` 非特权模式创建的 LXC 容器，OpenWrt 的 DNSMasq 服务无法正常启动，这种模式下网络可以 ping 通 IP，但无法解析域名，提示报错 `ping: bad address 'qq.com'` ，因为 DNSMasq 服务异常。

临时解决方法：若需要在这种情况下临时访问外网，直接修改 `/etc/resolv.conf` 文件即可，追加可用的 DNS 服务器 IP 即可。

```bash
BASHvim /etc/resolv.conf

# 配置文件内容

search lan

nameserver 127.0.0.1

nameserver ::1

# 追加可用的DNS服务器

nameserver 8.8.8.8
```

## 7.6 LXC 容器挂载新路径，共享宿主机目录

另一种为非特权 LXC 容器挂载路径的方式： [https://forum.proxmox.com/threads/tutorial-unprivileged-lxcs-mount-cifs-shares.101795/](https://forum.proxmox.com/threads/tutorial-unprivileged-lxcs-mount-cifs-shares.101795/)

## 8 创建 PVE 虚拟机 VM 遇到的坑

## 8.1 添加 EFI 磁盘后，启动无法引导磁盘，出现 shell 界面。

原因：创建 EFI 磁盘时，勾选了添加密钥。

解决方法：创建 EFI 磁盘时，取消勾选添加密钥。

## 8.2 安装 windows 镜像，无法识别找到磁盘

原因：scsi 不适合 windows 镜像，windows 安装解决识别不到硬盘。

解决方法：创建硬盘时，总线 / 设备选择：SATA 硬盘或者 IDE。

## 8.3 安装 windows 镜像，网卡无驱动

现象：当网卡设置为 virtio 模型时，安装并进入到 windows 系统后，网卡无驱动。

原因：windows 系统自带的驱动无此虚拟硬件的驱动。

解决方式：加载 virtio 镜像，并运行 x64 程序安装驱动即可。

VirtIO 镜像下载地址： [点击跳转](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/)

## 8.4 创建的 Windows 虚拟机运行一段时间自动挂起进入 suspended 状态

问题：创建的 Windows 虚拟机运行一段时间后，会自动挂起进入 suspended 状态，并且恢复 resume 系统状态会导致虚拟机内存占用瞬间吃满，然后无响应。

原因：虚拟机 Windows 系统自身有自动睡眠和关闭屏幕等设置，导致 PVE 的 QEMU 自动进入到挂起状态。

解决方法：关闭虚拟机 Windows 系统内电源设置的自动睡眠和自动关闭屏幕功能。

## 8.5 虚拟机挂载新路径，共享宿主机目录

[https://blog.209902.xyz/archives/1715428748533](https://blog.209902.xyz/archives/1715428748533)

## 9 路由方案一：iKuai 主路由 + OpenWRT 传统旁路由的网络拓扑

iKuai 作为主路由，负责拨号及 DHCP，OpenWRT 做旁路由。SmartDNS + AdGuardHome 设置分流与去广告。SmartDNS 作为 DNS 管理并提供 DNS 缓存，实现国内国外 DNS 分流，彻底解决 DNS 污染问题、实现秒开网页。同时搭配 AdGuardHome 实现整个局域网去广告。

DNS 转发流程：设置最核心的部分就是 DNS 转发端口的衔接，就是把 DNSMASQ、Adguardhome、SmartDNS 三个插件里的 DNS 服务器功能分成三个层级，实现层层转发。依次是第一级 DNSMASQ，第二级 Adguardhome，第三级 SmartDNS，第四级 OpenClash。

## 9.1 iKuai 的设置

- ~~系统设置 > 重启关机 > 添加一个重启计划：每天 05:00 时重启。~~
- 网络设置 > 内外网设置
	- 外网接口（选择 iKuai 识别到的第二个网口）：填入宽带拨号信息。
	- 内网接口（iKuai 识别到的第一个网口，IP 地址 192.168.1.1）：链路桥接（选择剩余的其他全部接口）
- DHCP 设置 > DHCP 服务端
	- 单 iKuai 版：
		- 客户端地址：192.168.1.10-192.168.1.254（1-9 留给专用设备的管理地址）
		- 子网掩码：255.255.255.0
		- 网关：192.168.1.1
		- 首选 DNS：223.5.5.5
		- 备选 DNS：114.114.114.114
	- iKuai + OpenWrt 版本：
		- 客户端地址：192.168.1.10-192.168.1.254（1-9 留给专用设备的管理地址）
		- 子网掩码：255.255.255.0
		- 网关：192.168.1.3
		- 首选 DNS：192.168.1.3
		- 备选 DNS：192.168.1.3
- DNS 设置 > DNS 设置
	- 单 iKuai 版：
		- 首选 DNS：223.5.5.5
		- 备选 DNS：114.114.114.114
	- iKuai + OpenWrt 版本：
		- 首选 DNS：192.168.1.3
		- 备选 DNS：192.168.1.3
- UPnP 设置 > UPnP 设置
	- UPnP 即插即用服务：开启
	- 允许内网 IP 映射：0.0.0.0-255.255.255.255
	- 默认线路设置：任意
	- 掉线检测：开启
	- 检测周期：2
	- 定时重启：开启
	- 重启周期：全选
	- 重启时间：05:00。

## 9.2 OpenWrt 设置

### 9.2.1 定时重启

~~系统 > 定时重启：启用，设置每天 05:00。~~

### 9.2.2 网络

#### 9.2.2.1 接口 > LAN

- IPv4 地址：192.168.1.3。管理 OpenWrt 的地址。
- 和 iKuai 一致保证同一个网段。
- IPv4 网关：指向 iKuai 的网关。保证 OpenWrt 的上网数据传输到 iKuai 再传输到外网。
- 使用自定义的 DNS 服务器：223.5.5.5（初始先设置阿里云 DNS 保证设置过程的上网）。后面等设置好 SmartDNS 之后改由 OpenWrt 代理，即输入 192.168.1.3。
- 关闭 DHCP 服务，统一由 iKuai 分配。
- 禁用掉 IPv6 服务。
- 物理设置：桥接接口。（因为选用的也是）
- lan 高级设置：不勾选 IPv6，勾选【开启开机自动运行】和【强制链路】。

#### 9.2.2.2 Turbo ACC 加速

只开启前三个即可，DNS 缓存后面由 SmartDNS 来管理。

#### 9.2.2.3 防火墙设置

- SYN-flood 防御：关闭（此项开启关闭都行）
- 丢弃无效数据包：启用（此项开启关闭都行）
- 启用 FullCone-NAT：高性能模式
- 入站数据、出站数据、转发都设置为接受。
- lan 口开启：IP 动态伪装（只保留 lan 口的规格，其他接口全部删除）
- 防火墙自定义规则（一般默认就有前四条规则，没有需加上）
```
PLAINTEXTiptables -t nat -A PREROUTING -p udp —dport 53 -j REDIRECT —to-ports 53

iptables -t nat -A PREROUTING -p tcp —dport 53 -j REDIRECT —to-ports 53

[ -n “$(command -v ip6tables)” ] && ip6tables -t nat -A PREROUTING -p udp —dport 53 -j REDIRECT —to-ports 53

[ -n “$(command -v ip6tables)” ] && ip6tables -t nat -A PREROUTING -p tcp —dport 53 -j REDIRECT —to-ports 53

iptables -t nat -I POSTROUTING -o eth0 -j MASQUERADE
```

### 9.2.3 SmartDNS

Bleach OpenWrt 固件自带 SmartDNS，eSir GDQ 高大全固件需自行手动安装。

- 基本设置
	- 启用
	- 本地端口 6053
- 高级设置
	- 开启 TCP 服务器
	- 勾选域名预加载
	- 勾选缓存过期服务
	- 缓存大小设为：1000000
	- 域名 TTL 最大值设为：3600
- 第二 DNS 服务器
	- 启用
	- 本地端口 5335
	- TCP 服务器
	- 服务器组：oversea
	- 跳过测速
	- 跳过 address 规则
	- 跳过 address SOA (#) 规则
	- 跳过双栈优选
	- 跳过 cache
	- 停用 IPV6 地址解析
- 上游服务器（添加 DNS 的设置里面可以设置服务器组）
	- 服务器组名称：china（10 条左右，包含 iKuai 路由宽带拨号返回的两个宽带供应商提供的 DNS）
		1. iKuai 路由宽带拨号返回的宽带供应商提供的 DNS1，udp 协议
		2. iKuai 路由宽带拨号返回的宽带供应商提供的 DNS2，udp 协议
		3. 114dns，114.114.114.114，udp 协议
		4. 114dns，114.114.115.115，udp 协议
		5. opener dns，42.120.21.30，udp 协议
		6. 阿里 dns，223.5.5.5，udp 协议
		7. 阿里 dns，223.6.6.6，udp 协议
		8. 腾讯 dns，119.29.29.29，udp 协议
		9. 百度 dns，180.76.76.76，udp 协议
		10. 华为 dns，122.112.208.1，udp 协议
	- 服务器组名称：oversea（10 条左右）
		1. google dns，8.8.8.8，udp 协议
		2. google dns，8.8.4.4，tls 协议
		3. CloudFlare，1.1.1.1，udp 协议
		4. IBM Quad9，9.9.9.9，udp 协议
		5. Norton ConnectSafe，199.85.126.10，udp 协议
		6. Norton ConnectSafe，199.85.127.10，udp 协议
		7. OpenDNS，208.67.222.222，tls 协议
		8. OpenDNS，208.67.220.220，udp 协议
		9. V2EX DNS，199.91.73.222，tls 协议
		10. V2EX DNS，178.79.131.110，tls 协议
- 域名规则
	- 服务器组：china
	- 域名分流设置：跳过测试
- 自定义设置（只设置两条，其他全部注释掉）
```
PLAINTEXTbind:6053 -group china

bind:5335 -group oversea
```
- 保存 & 应用

### 9.2.4 AdGuard Home

#### 9.2.4.1 手动更新内核

1. 下载最新的 AdGuardHome 内核（AdGuardHome\_linux\_amd64.tar.gz）： [点击跳转](https://github.com/AdguardTeam/AdGuardHome/releases)
2. 解压获取「AdGuardHome」文件。
3. 使用 WinSCP 登录 openwrt 虚拟机，进入到路径 `/usr/bin/AdGuardHome/` ，上传「AdGuardHome」文件。右击属性，分配权限 0755（rwxr-xr-x）。

能科学上网时，点击检查更新，直接更新核心即可。

#### 9.2.4.2 设置 AdGuard Home

- 启用
- AdGuardHome 重定向模式：作为 dnsmasq 的上游服务器
- 详细日志
- 开机后网络准备好时重启
- 在关机时备份工作目录文件（所有的选项）
- 打开 192.168.1.3:3000 地址并配置
- 网页管理界面 > 监听接口 > 所有接口，端口号设置为默认的 3000
- DNS 服务器 > 监听接口 > 所有接口，端口号设置为 5351
- 网页管理界面登录密码
- 设置 > 常规设置 > 使用过滤器和 Hosts 文件以拦截指定域名（其他选项全部关闭）
- 设置 > DNS 设置
	- 上游 DNS 服务器
		```
		PLAINTEXT127.0.0.1:6053
		127.0.0.1:5335
		```
		- 并行请求
		- Bootstrap DNS： `127.0.0.1:6053`
		- 应用
	- DNS 服务配置
		- 速度限制：0
		- DNS 缓存配置：都空着（采用上游 DNS 服务器，让 SmartDNS 来管理）
- 设置 > 过滤器 > DNS 黑名单设置（将阻止匹配 DNS 拦截清单的域名）：综合性的规则列表启用几个即可。

### 9.2.5 DNSMASQ 设置

打开网络 > DHCP / DNS。主要是解除 DNSMASQ 的 DNS 功能，只保留转发功能，让 DNSMASQ 作为 AdGuard Home 的下级服务生效。

- DNS 转发： `127.0.0.1#5351` 。如果未自动设置成这个，手动强制改成这个。
- HOSTS 和解析文件：忽略掉解析文件，不用 DNSMASQ 的解析。
- 高级设置
	- DNS 服务器端口：53
	- DNS 查询缓存的大小：0
	- 最大并发查询数：1500
- 保存 & 应用

### 9.2.6 OpenClash

实测，Meta 内核因 DNS 转发问题，导致无法正常上网。例如 GOOGLE 可能会出现证书异常的问题，YOUTUBE 点击视频后会存在一直转圈圈的情况，无法加载视频。现阶段稳定工作的模式为：Fake-IP 的普通 TUN 模式，模型模式可调 TUN 或者混合（TUN 模式需要有 TUN 内核支持）。代理模式使用规则上网即可。

是最后一版支持 TUN 内核的版本，往后一版的版本就开始移除 Dev 和 TUN 内核的支持，如果手动操作更新 OpenClash 版本，会自动移除 Dev 和 TUN 内核文件。即使手动通过 WinSCP 上传 Dev 和 TUN 内核文件，软件内也是无法识别到的。

#### 9.2.6.1 更新客户端

1. 下载客户端安装包： [点击跳转](https://github.com/vernesong/OpenClash/releases)
2. 使用 WinSCP 登录 openwrt 虚拟机，进入到路径家目录 home，上传安装包，运行安装 `opkg install ./luci-app-openclash_0.46.014-beta_all.ipk` 。如果报错，尝试 `opkg update` 。

#### 9.2.6.2 手动更新内核

内核下载地址：

- 老的 Release 页面
- 当前发布的最新内核
	- [Dev 内核](https://github.com/vernesong/OpenClash/tree/core/master/dev)
	- [Tun 内核](https://github.com/vernesong/OpenClash/tree/core/master/premium)
	- [Meta 内核](https://github.com/vernesong/OpenClash/tree/core/master/meta)

新的内核只支持 fake-ip，需要上传 Meta 内核。

能科学上网时，点击检查并更新，直接更新核心即可。

更新步骤：

1. 下载内核。
2. 使用 WinSCP 登录 openwrt 虚拟机，进入到路径 `/etc/openclash/core` ，上传内核文件。右击属性，分配权限 0755（rwxr-xr-x）。

内核对应的名称（压缩包解压出来的名称不一定对应，需手动修改）：

- Dev 内核: clash
- Tun 内核: clash\_tun
- Meta 内核: clash\_meta

#### 9.2.6.3 OpenClash 设置

- 运行状态：启动 OPENCLASH
- 插件设置
	- 模式设置 > 运行模式：Fake-IP（TUN - 混合）模式【UDP - TUN，TCP - 转发】
	- DNS 设置 > 本地 DNS 劫持：停用
	- GEO 数据库订阅：设置自动更新
	- 大陆白名单订阅：设置自动更新
- 配置订阅：设置自动更新，更新间隔 60 分钟

### 9.2.7 扩容 Overlay 软件安装空间

1. 关闭 openwrt 虚拟机。
2. 硬件 > 硬盘 > 更多，调整磁盘大小（只能增加大小，按需增加所需硬盘的空间大小）。
3. 打开 openwrt 虚拟机。
4. 系统 > 磁盘管理 L：将新增磁盘大小新建一个分区并格式化为 ext4，保存并应用
5. 挂载刚才新增的分区，举例 sda3： `mount /dev/sda3 /mnt/sda3` ，保存并应用。
6. 拷贝源 overlay 目录的文件到新分区内： `cp -r /overlay/* /mnt/sda3` 。
7. 系统 > 挂载点：添加 sda3 挂载点为「作为外部 overlay 使用（/overlay）」，保存并应用。
8. 重启 openwrt 虚拟机，检查扩容情况。
9. 根据情况可能需手动移除之前挂载的 `/mnt/sda3` 状态。

## 10 路由方案二：iKuai 主路由 + OpenWrt 旁路由分流的网络拓扑

## 10.1 iKuai 主路由设置

[大陆地区 IP 列表](https://github.com/17mon/china_ip_list)

## 10.2 OpenWrt 旁路有设置

TODO

## 10.3 构建本地机场订阅转换服务

相关核心项目：

- 后端项目： [tindy2013/subconverter](https://github.com/tindy2013/subconverter)
- 前端项目： [CareyWang/sub-web](https://github.com/CareyWang/sub-web)
- 短链服务： [CareyWang/MyUrls](https://github.com/CareyWang/MyUrls)

### 10.3.1 PVE 新建一个 debian 虚拟机

使用 LXC 模板（debian-12-standard），完成创建之后，并登录该 debian 虚拟机（可设置 SSH 连接）

- 容器名称：sub-converter
- 内存：2GB（别分配 1.5GB 以下的内存，否则不够内存完成构建 docker 镜像）（构建完成 docker 镜像后，内存可以调整为 512MB）
- Swap：0B
- 核心：4 核心（构建完成 docker 镜像后，核心可以调整为 1）
- 根磁盘：5GB（别分配 2GB 以下的磁盘，否则不够空间完成构建 docker 镜像）
- 网络：选择已分配给主路由 LAN 口的网桥（我的情况是：vmbr4，IP 静态地址：10.10.10.6 / 24，网关：10.10.10.1）

### 10.3.2 设置优化 debian 虚拟机

1. 为 debian 修改 apt 镜像源： [替换 PVE-8-x 的 Debian 系统软件源和 PVE 软件源（国内镜像）](https://mister-kin.github.io/software/pve/#%E6%9B%BF%E6%8D%A2PVE-8-x%E7%9A%84Debian%E7%B3%BB%E7%BB%9F%E8%BD%AF%E4%BB%B6%E6%BA%90%E5%92%8CPVE%E8%BD%AF%E4%BB%B6%E6%BA%90%EF%BC%88%E5%9B%BD%E5%86%85%E9%95%9C%E5%83%8F%EF%BC%89)
2. 为 debian 安装 docker： [使用 apt 仓库为 Debian 安装 docker](https://mister-kin.github.io/code/docker/#%E4%BD%BF%E7%94%A8apt%E4%BB%93%E5%BA%93%E4%B8%BADebian%E5%AE%89%E8%A3%85docker)
3. 为 debian 系统设置时区： [创建的 LXC 容器 debian-12 系统时间与宿主机时间不一致](https://mister-kin.github.io/software/pve/#%E5%88%9B%E5%BB%BA%E7%9A%84LXC%E5%AE%B9%E5%99%A8debian-12%E7%B3%BB%E7%BB%9F%E6%97%B6%E9%97%B4%E4%B8%8E%E5%AE%BF%E4%B8%BB%E6%9C%BA%E6%97%B6%E9%97%B4%E4%B8%8D%E4%B8%80%E8%87%B4)
4. 为 debian 系统设置 UTF-8 语言环境： [使用 debian-12 的 CT 模板创建 LXC 容器 SSH 终端中文乱码](https://mister-kin.github.io/software/pve/#%E4%BD%BF%E7%94%A8debian-12%E7%9A%84CT%E6%A8%A1%E6%9D%BF%E5%88%9B%E5%BB%BALXC%E5%AE%B9%E5%99%A8SSH%E7%BB%88%E7%AB%AF%E4%B8%AD%E6%96%87%E4%B9%B1%E7%A0%81)

### 10.3.3 手动重新构建前端 docker 镜像

1. 克隆前端项目
```bash
BASHgit clone https://github.com/CareyWang/sub-web.git
```
1. 进入对应仓库路径 `cd sub-web`
2. 修改`.env` 文件内容： `nano .env`
```bash
BASH# 找到以下三行内容，并修改自己的域名或者IP

# VUE_APP_SUBCONVERTER_DEFAULT_BACKEND = "https://api.wcc.best"

# VUE_APP_MYURLS_API = "https://suo.yt/short"

# VUE_APP_CONFIG_UPLOAD_API = "https://oss.wcc.best/upload"

# 这是笔者修改的配置

VUE_APP_SUBCONVERTER_DEFAULT_BACKEND = "http://10.10.10.6:25500"
```
1. 修改 `Subconverter.vue` 文件内容： `nano src/views/Subconverter.vue`
```bash
BASH# 搜索127.0.0.1内容，并修改为10.10.10.6（该IP是LXC容器debian使用的IP）

# placeholder="动动小手，（建议）自行搭建后端服务。例：http://127.0.0.1:25500/sub?">

# backendOptions: [{ value: "http://127.0.0.1:25500/sub?" }],

# 这是笔者修改的配置

placeholder="动动小手，（建议）自行搭建后端服务。例：http://10.10.10.6:25500/sub?">

backendOptions: [{ value: "http://10.10.10.6:25500/sub?" }],

# 搜索remoteConfig: [

# 在下一行添加ACL4SSR的规则配置。

# 效果如下：

remoteConfig: [

  {

      label: "ACL4SSR",
```

ACL4SSR 规则配置：

```bash
BASH{

    label: "ACL4SSR",

    options: [

      {

        label: "ACL4SSR_Online 默认版 分组比较全 (与Github同步)",

        value:

          "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online.ini"

      },

      {

        label: "ACL4SSR_Online_AdblockPlus 更多去广告 (与Github同步)",

        value:

          "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_AdblockPlus.ini"

      },

      {

        label: "ACL4SSR_Online_NoAuto 无自动测速 (与Github同步)",

        value:

          "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_NoAuto.ini"

      },

      {

        label: "ACL4SSR_Online_NoReject 无广告拦截规则 (与Github同步)",

        value:

          "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_NoReject.ini"

      },

      {

        label: "ACL4SSR_Online_Mini 精简版 (与Github同步)",

        value:

          "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini.ini"

      },

      {

        label: "ACL4SSR_Online_Mini_AdblockPlus.ini 精简版 更多去广告 (与Github同步)",

        value:

          "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_AdblockPlus.ini"

      },

      {

        label: "ACL4SSR_Online_Mini_NoAuto.ini 精简版 不带自动测速 (与Github同步)",

        value:

          "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_NoAuto.ini"

      },

      {

        label: "ACL4SSR_Online_Mini_Fallback.ini 精简版 带故障转移 (与Github同步)",

        value:

          "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_Fallback.ini"

      },

      {

        label: "ACL4SSR_Online_Mini_MultiMode.ini 精简版 自动测速、故障转移、负载均衡 (与Github同步)",

        value:

          "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_MultiMode.ini"

      },

      {

        label: "ACL4SSR_Online_Full 全分组 重度用户使用 (与Github同步)",

        value:

          "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full.ini"

      },

      {

        label: "ACL4SSR_Online_Full_NoAuto.ini 全分组 无自动测速 重度用户使用 (与Github同步)",

        value:

          "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full_NoAuto.ini"

      },

      {

        label: "ACL4SSR_Online_Full_AdblockPlus 全分组 重度用户使用 更多去广告 (与Github同步)",

        value:

          "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full_AdblockPlus.ini"

      },

      {

        label: "ACL4SSR_Online_Full_Netflix 全分组 重度用户使用 奈飞全量 (与Github同步)",

        value:

          "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full_Netflix.ini"

      },

      {

        label: "ACL4SSR 本地 默认版 分组比较全",

        value: "config/ACL4SSR.ini"

      },

      {

        label: "ACL4SSR_Mini 本地 精简版",

        value: "config/ACL4SSR_Mini.ini"

      },

      {

        label: "ACL4SSR_Mini_NoAuto.ini 本地 精简版+无自动测速",

        value: "config/ACL4SSR_Mini_NoAuto.ini"

      },

      {

        label: "ACL4SSR_Mini_Fallback.ini 本地 精简版+fallback",

        value: "config/ACL4SSR_Mini_Fallback.ini"

      },

      {

        label: "ACL4SSR_BackCN 本地 回国",

        value: "config/ACL4SSR_BackCN.ini"

      },

      {

        label: "ACL4SSR_NoApple 本地 无苹果分流",

        value: "config/ACL4SSR_NoApple.ini"

      },

      {

        label: "ACL4SSR_NoAuto 本地 无自动测速 ",

        value: "config/ACL4SSR_NoAuto.ini"

      },

      {

        label: "ACL4SSR_NoAuto_NoApple 本地 无自动测速&无苹果分流",

        value: "config/ACL4SSR_NoAuto_NoApple.ini"

      },

      {

        label: "ACL4SSR_NoMicrosoft 本地 无微软分流",

        value: "config/ACL4SSR_NoMicrosoft.ini"

      },

      {

        label: "ACL4SSR_WithGFW 本地 GFW列表",

        value: "config/ACL4SSR_WithGFW.ini"

      }

    ]

  },
```
1. 重新构建 docker 本地镜像
```
PLAINTEXTdocker build -t subweb-local:latest .
```

### 10.3.4 在 root 账户用户路径下新建docker-compose.yml：nano docker-compose.yml

```bash
BASHservices:

  subc:

    hostname: subc

    restart: always

    container_name: subc

    image: tindy2013/subconverter:latest

    environment:

      - TZ=Asia/Shanghai

    ports:

      - "25500:25500"

  subweb:

    hostname: subweb

    restart: always

    container_name: subweb

    image: subweb-local:latest

    environment:

      - TZ=Asia/Shanghai

    ports:

      - "80:80"
```

### 10.3.5 运行 docker compose

```bash
BASHdocker compose -f docker-compose.yml up -d
```

### 10.3.6 验证服务是否运行成功

访问前端页面： [http://10.10.10.6](http://10.10.10.6/)

通过 tcping 验证各个端口是否都正常：

- `tcping 10.10.10.6 80`
- `tcping 10.10.10.6 25500`

## 11 PVE 总结

## 11.1 PVE 服务

[https://pve.proxmox.com/wiki/Service\_daemons](https://pve.proxmox.com/wiki/Service_daemons)

## 11.2 PVE (Proxmox VE) 虚拟机各硬件模型总结

TODO

## 11.3 PVE 服务器的网络拓扑思路

一般而言，TODO

`vmbr0` 为 PVE 初始创建的默认网桥，一般设置如下参数：

- 固定 ip 地址 `192.168.2.2/24`
- 网关 `192.168.2.1`
- 绑定到一个物理网口

点击创建第二个网桥 `vmbr1` ，设置如下参数：

- 固定 ip 地址 `10.10.10.1/24`
- 网关留空
- 不绑定物理网卡

编辑 PVE 宿主机的 `/etc/network/interfaces` 文件，添加如下语句实现 `vmbr1` 数据转发 `vmbr0` ： `nano /etc/network/interfaces`

```bash
BASHauto vmbr1

iface vmbr1 inet static

        # address 10.10.10.1

        # netmask 255.255.255.0

        address 10.10.10.1/24

        bridge-ports none

        bridge-stp off

        bridge-fd 0

        post-up echo 1 > /proc/sys/net/ipv4/ip_forward

        post-up iptables -t nat -A POSTROUTING -s '10.10.10.0/24' -o vmbr0 -j MASQUERADE

        post-down iptables -t nat -D POSTROUTING -s '10.10.10.0/24' -o vmbr0 -j MASQUERADE
```

启用并验证

```bash
BASH# 启用网桥

ifup vmbr1

# 查看网桥状态

ip address show dev vmbr1

# 查看iptables路由表

iptables -L -t nat

# 重启网络服务

systemctl restart networking

# 注意post-up和post-down是在vmbr1网络接口启动时和关闭时才会执行后面的指令。systemctl restart networking是不会执行这些命令的
```

### 11.3.1 创建 DHCP 服务器，为 vmbr1 网桥的设备自动设置 ip 地址

```bash
BASHapt install isc-dhcp-server -y

# 配置DHCP服务器监听的网络接口，就是用来分发ip地址的网络接口

nano /etc/default/isc-dhcp-server

INTERFACESv4="vmbr1"

# 定义NAT子网，例如DHCP的ip范围，

nano /etc/dhcp/dhcpd.conf

subnet 10.10.10.0 netmask 255.255.255.0 {

        range 10.10.10.10 10.10.10.254;

        option subnet-mask 255.255.255.0;

        option domain-name-servers 223.5.5.5,223.6.6.6;

        option routers 10.10.10.1;

        option netbios-name-servers 10.10.10.1;

        option netbios-node-type 8;

        get-lease-hostnames true;

        use-host-decl-names true;

        default-lease-time 600;

        max-lease-time 7200;

        interface vmbr0;

}

# 重启DHCP服务

systemctl restart isc-dhcp-server

# 查看DHCP服务运行状态

systemctl status isc-dhcp-server

# 检查DHCP服务器的日志文件

tail -f /var/log/syslog | grep isc-dhcp-server
```

NAT 网络转发：转发 3389 端口

```bash
BASHnano /etc/network/interfaces

auto vmbr0

iface vmbr0 inet static

        address 192.168.2.2/24

        gateway 192.168.2.1

        bridge-ports enp6s0

        bridge-stp off

        bridge-fd 0

        post-up iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 3389 -j DNAT --to 10.10.10.10:3389

        post-down iptables -t nat -D PREROUTING -i vmbr0 -p tcp --dport 3389 -j DNAT --to 10.10.10.10:3389
```

iptables 命令设置：

```bash
BASH#新增

iptables -t nat -A PREROUTING -p tcp -m tcp --dport 10022 -j DNAT --to-destination 10.10.10.100:22

#删除 （即把新增映射的-A改成-D）

iptables -t nat -D PREROUTING -p tcp -m tcp --dport 10022 -j DNAT --to-destination 10.10.10.100:22

#查看NAT规则，并显示行号

iptables -t nat --list --line-number

#删除指定行号的iptables规则

iptables -t nat -D POSTROUTING 10
```

## 11.4 PVE 虚拟机磁盘容量调整

- LXC 容器的磁盘，在网页控制台设置增加大小，容器会自动增加对应的磁盘分区大小，无需手动在虚拟机系统格式化和挂载。

## 11.5 PVETOOLS 脚本程序推荐

> 新硬件主机不太推荐安装该脚本，CPU 省电相关设置不一定能正常设置成功，恢复时也容易出问题。

- PVETOOLS 仓库地址： [点击跳转](https://github.com/ivanhao/pvetools)
- 安装步骤（PVE SHELL 执行命令）：
```bash
BASHexport LC_ALL=en_US.UTF-8

apt update && apt -y install git && git clone https://github.com/ivanhao/pvetools.git

cd pvetools

./pvetools.sh
```
- 执行脚本程序后，可以一键执行「安装配置 CPU 省电」「配置 PVE 的 web 界面显示传感器温度」「去除订阅提示」等功能。

## 12 PBS (Proxmox Backup Server) 总结

## 12.1 PBS 常用命令

```bash
BASH# 查看PBS版本

proxmox-backup-manager versions

# 查看PBS节点配置信息

proxmox-backup-manager node show

# 查看节点配置文件

nano /etc/proxmox-backup/node.cfg

# 配置PBS日志文件的轮转日期，例如只保留最近30天的日志文件

proxmox-backup-manager node update --task-log-max-days 30
```

## 12.2 PBS 备份计划

重要服务器虚拟机，可以将备份时间设置为半小时或者更短时间。

## 12.3 PBS 服务器备份精简计划

PBS 备份精简计划设置效果模拟器： [https://pbs.proxmox.com/docs/prune-simulator/index.html](https://pbs.proxmox.com/docs/prune-simulator/index.html)

保留设置：

- keep-last（保留上次）： 3 - 保留最近 3 次的备份。
- keep-hourly（保留每小时）: not set - 若设置了每日备份，这个选项一般无需设置
- keep-daily（保留每天）： 13 - 与 keep-last 一起使用，确保至少有两周的每日备份
- keep-weekly（保留每周）： 8 - 确保至少有两个月的每周备份
- keep-monthly（保留每月）： 11 - 与之前的保留设置一起，这确保至少有一年的每月备份
- keep-yearly（保留每年）： 9 - 确保至少 10 年的覆盖

## 12.4 PBS 使用遇到的坑

问题现象：PBS 备份服务器或者 PVE 备份 LXC 虚拟机容器时出现失败，具体报错信息为

```bash
BASHINFO: Starting Backup of VM 102 (lxc)

INFO: Backup started at 2025-04-25 00:30:36

INFO: status = running

INFO: CT Name: test

INFO: including mount point rootfs ('/') in backup

INFO: backup mode: snapshot

INFO: ionice priority: 7

INFO: create storage snapshot 'vzdump'

snapshot create failed: starting cleanup

no lock found trying to remove 'backup'  lock

ERROR: Backup of VM 102 failed - lvcreate snapshot 'pve/snap_vm-102-disk-0_vzdump' error:   Logical Volume "snap_vm-102-disk-0_vzdump" already exists in volume group "pve"

INFO: Failed at 2025-04-25 00:30:36

INFO: Backup job finished with errors

INFO: notified via target \`mail-to-root\`

TASK ERROR: job errors
```

问题原因：有残留的同名逻辑卷 `/dev/pve/snap_vm-102-disk-0_vzdump` ，一般是上次备份成功之后的清理操作失败导致残留下来的逻辑卷，例如卸载操作失败 `command 'umount -l -d /mnt/vzsnapo/ failed: got signal 11` 。

解决方法：

1. 查看当前 PVE 系统中存在的逻辑卷： `lvdisplay` ，可以查看到 `/dev/pve/snap_vm-102-disk-0_vzdump` 逻辑卷属性 `LV Write Access read only` 是只读。其他查看命令 `lvs` ， `lvscan` 。
2. 清理残留的逻辑卷：
	- 方法一： `lvremove /dev/pve/snap_vm-102-disk-0_vzdump`
	- 方法二：PVE 后台网页控制面板中，对应的虚拟机选项 -> 快照，可以选中对应的 vzdump 快照删除。

若遇上报错： `lvremove /dev/pve/snap_vm-102-disk-0_vzdump' error: Logical volume pve/snap_vm-102-disk-0_vzdump contains a filesystem in use.`，请尝试

- 方案 1：重启 PVE 宿主机
- 方案 2：添加 `-f` 参数强制删除， `lvremove -f /dev/pve/snap_vm-102-disk-0_vzdump`

#### 12.4.1.1 PBS 宿主机被日志文件填满根目录

详看 [配置 PBS 日志轮转清理](https://mister-kin.github.io/software/pve/#%E9%85%8D%E7%BD%AEPBS%E6%97%A5%E5%BF%97%E8%BD%AE%E8%BD%AC%E6%B8%85%E7%90%86)

## 13 iKuai 总结

## 13.1 iKuai 爱快内网测试

iKuai 爱快内置支持 iperf3 内网测试工具，但服务端默认并未开启。

开启服务端设置和测试的步骤

1. 在 iKuai 爱快中设置：应用工具 -> 吞吐测试 -> 服务端功能开关（勾选启用）
2. 下载 [iperf3](https://iperf.fr/iperf-download.php) 到客户端机器。
3. 运行测试命令： `iperf3 -c 192.168.1.1`

客户端机器作为服务端的设置步骤：

1. 下载 [iperf3](https://iperf.fr/iperf-download.php) 到客户端机器。
2. 运行测试命令： `iperf3 -s -p 5201`
3. 在 iKuai 爱快的客户端工具连接客户端 IP 测试。

## 13.2 iKuai 报错提示：写入数据失败, 请检查你的配置!

问题：调整 iKuai 设置，无法保存设置，并报错提示：写入数据失败，请检查你的配置！具体变现为修改配置后，刷新页面后发现设置跟以前一样，并且无法记录流量数据等日志。

原因：磁盘所有分区变成只读模式，没有读写权限，数据无法正常保存。

排查确认方法：iKuai 管理后台 -> 应用工具 -> 健康检测。

解决方法：强制断电重启机器。若仍然无法识别到硬盘或者反复出现这种只读权限问题，很可能硬盘出故障，请尽快备份设备文件并更换新硬盘。

## 13.3 局域网内双 iKuai 主路由的 DHCP 的设置

局域网内若有两个 iKuai 主路由同时提供 DCHP 服务，并且两个主路由物理上联通的话（两接口分别归属于各自的 LAN 口），可以在网络设置 ->DHCP 设置 ->DHCP 服务端 -> 关联接口 -> 取消勾选两个路由互联的接口。避免两主路由互相跨网段提供 DHCP 服务，否则会有潜在的广播风暴问题。（该问题在 iKuai 路由体检功能也能发现）

## 13.4 局域网内双 iKuai 主路由跨网段互相访问（静态路由）

背景假设：一个 iKuai 地址为 `192.168.1.1` ，另一个 iKuai 地址为 `192.168.2.1`

1. 为网段 1 的 iKuai 新增扩展 IP：网络设置 -> 内外网设置 ->LAN1-> 高级设置 -> 扩展 IP-> 新增一个不归属于两个主路由网段的 IP，例如 `192.168.5.1` 。
	- 同理，另外一个 iKuai 新增 IP 为 `192.168.5.2` 。
2. 为网段 1 的 iKuai 设置静态路由表：网络设置 -> 静态路由 -> 添加 -> 目的地址： `192.168.2.0` ，子网掩码 `255.255.255.0 (24)` ，网关： `192.168.5.1` 。
	- 同理，另外一个 iKuai 置静态路由表：目的地址： `192.168.1.0` ，子网掩码 `255.255.255.0 (24)` ，网关： `192.168.5.2` 。
	- iKuai 静态路由的网关就是指下一跳。

## 13.5 iKuai 设定时间 NTP 服务器

若在网络拓扑的下游中，存在个别子路由是当作交换机使用，没有接 WAN 口，无法联网获取时间，导致定时重启等需要时间参数的功能无法使用的话，则可以将 iKuai 设定为 NTP 服务器。 `系统设置->基础设置->时间设置->NTP服务->勾选开启（网关作为NTP服务器）` 。

## 14 OpenWrt 总结

## 14.1 OpenClash 报错all DNS requests failed或者dns resolve failed

问题：通过机场订阅配置后，发现部份被分流到海外的 IP 和域名无法正常访问。查看 OpenClash 后台的内核日志，是因为有大量的 `all DNS requests failed` 或者 `dns resolve failed` 错误提示。并且该现象只会出现在个别机场订阅上，有些机场订阅是正常工作。

原因：部份机场比较挑剔 OpenClash 的 DNS 设置。

解决方法：OpenClash-> 覆写设置 ->DNS 设置  
FallBack 和 Default-NameServer 的 DNS 设置比较重要，对于笔者而言， `8.8.4.4, UDP` 是能有效解决上述问题。

- 【勾选】自定义上游 DNS 服务器
- 【勾选】追加上游 DNS
- 【勾选】追加默认 DNS
- 【勾选】Fallback-Filter
- 设置自定义上游 DNS 服务器（在上方设置中启用本功能后生效）
	- NameServer
		- 223.5.5.5/dns-query, HTTPS
		- 223.6.6.6/dns-query, HTTPS
		- 8.8.8.8, UDP
	- FallBack
		- 8.8.4.4, UDP
	- Default-NameServer
		- 114.114.114.114, UDP
		- 223.5.5.5, UDP
		- 8.8.8.8, UDP
		- 8.8.4.4, UDP

## 15 飞牛私有云 fnOS（NAS 总结）

## 15.1 EMBY + MetaTube Server 实现影视刮削

1. 在飞牛的应用中心中安装 `EMBY` 并通过 `wizardstart` 配置好帐号。
2. 打开 `docker` 应用，点击 `compose` 页面，在飞牛的存储中，创建 `metatube` 文件夹路径存放 `docker-compose.yaml` 的文件（以下卷映射也是基于该路径的相对路径），输入如下内容保存运行： `metatube` 的 `docker-compose.yaml`
	```yaml
	YAMLservices:
	  metatube:
	    image: ghcr.io/metatube-community/metatube-server:latest
	    container_name: metatube
	    ports:
	      - "8080:8080"
	    restart: unless-stopped
	    depends_on:
	      - postgres
	    environment:
	      - HTTP_PROXY=
	      - HTTPS_PROXY=
	      - TZ=Asia/Shanghai
	    volumes:
	      - run:/var/run
	    command: -dsn "postgres://metatube:metatube@/metatube?host=/var/run/postgresql" -port 8080 -db-auto-migrate -db-prepared-stmt
	  postgres:
	    image: postgres:15-alpine
	    container_name: metatube-postgres
	    restart: unless-stopped
	    environment:
	      - POSTGRES_USER=metatube
	      - POSTGRES_PASSWORD=metatube
	      - POSTGRES_DB=metatube
	      - TZ=Asia/Shanghai
	    volumes:
	      - ./db:/var/lib/postgresql/data
	      - run:/var/run
	    command: "-c TimeZone=Asia/Shanghai -c log_timezone=Asia/Shanghai -c listen_addresses='' -c unix_socket_permissions=0777"
	volumes:
	  run:
	```
3. 下载 [Metatube](https://github.com/metatube-community/jellyfin-plugin-metatube/releases) 的插件，放置于 `/vol1/@appdata/EmbyServer4-9/plugins` （应用 -> 文件管理 -> 管理员视角），重启 EMBY 应用。
4. 在 `EMBY` 的控制台， `高级->插件` ，找到 `Metatube` ，点击 `设置` ，填入第二步的部署 docker 的 ip + 端口：就是 `http://飞牛IP:8080` ，点击底部的 `保存` 。
5. 然后在 `EMBY` 的控制台， `Emby Server->媒体库` ，这里可以新增和调整元数据的设置参数。

## 15.2 Komga 服务端图书馆

- 不支持 txt 书籍！！！
- 支持 mango，PDF，epub。

1. 打开 `docker` 应用，点击 `compose` 页面，在飞牛的存储中，创建 `komga` 文件夹路径存放 `docker-compose.yaml` 的文件（以下卷映射也是基于该路径的相对路径），输入如下内容保存运行： `komga` 的 `docker-compose.yaml`
	```yml
	YMLservices:
	  komga:
	    image: gotson/komga
	    container_name: komga
	    volumes:
	      - ./config:/config
	      - ./data:/data
	      - /etc/timezone:/etc/timezone:ro
	    ports:
	      - 25600:25600
	    user: "1000:1000"
	    # remove the whole environment section if you don't need it
	    # environment:
	      # - <ENV_VAR>=<extra configuration>
	    restart: unless-stopped
	```
2. 访问地址： `http://飞牛IP:25600` ，配置帐号 / 密码（帐号要求为邮箱地址）。

## 15.3 Talebook 服务端图书馆

- Talebook 配合手机 APP `静读天下-moonreader` ，实测无法在线阅读 epub 电子书，只能下载后再阅读，阅读记录可被保留（毕竟都下载本地了 (lll￢ω￢)）。
- Talebook 服务端批量操作没有删除选项，只能单独删除。

1. 打开 `docker` 应用，点击 `compose` 页面，在飞牛的存储中，创建 `talebook` 文件夹路径存放 `docker-compose.yaml` 的文件（以下卷映射也是基于该路径的相对路径），输入如下内容保存运行： `talebook` 的 `docker-compose.yaml`
	```yaml
	YAMLservices:
	  # main service
	  talebook:
	    restart: always
	    image: talebook/talebook
	    volumes:
	       - ./data:/data
	    ports:
	       - "8090:80"
	    #- "8443:443"
	    environment:
	      - PUID=1000
	      - PGID=1000
	      - TZ=Asia/Shanghai
	    depends_on:
	      - douban-rs-api
	  # optional, for meta plugins
	  # please set "http://douban-rs-api" in settings
	  douban-rs-api:
	    restart: always
	    image: ghcr.io/cxfksword/douban-api-rs
	```
2. 访问地址： `http://飞牛IP:8090` ，配置帐号/密码/邮件。
3. 手机 APP `静读天下-moonreader` 右上角，增加新书库，书库地址： `http://飞牛IP:8090/opds/` 。

## 15.4 下载应用例如QBittorrent在后台上传时可能会引导硬盘持续读取和 CPU 的异常占用。

问题原因：系统 Swap cache 功能引起引起硬盘持续读取和 CPU 的异常占用。

解决方法：永久禁用 `Swap cache` 功能。

1. 访问 PVE 的 fnOS 虚拟机控制台或者开启 SSH 远程访问权限。
2. 登录 fnOS 账户。
3. `sudo vim /etc/fstab`
4. 摁 `I` 键进入 vim 编辑器编辑模式
5. 找到 `/swapfile none swap sw 0 0` 一行内容并注释掉，即在行头加上 `#` 。
6. 摁 `ESC` 键，摁`:`键然后输入 `wq!`保存退出。
7. `cat /etc/fstab` 可以查看文件内容是否修改成功，输出内容如下：
```bash
BASHUUID=ea545bcc-cd3f-479f-902a-ab2b02db25 / ext4 errors=remount-ro 0 1

# /swapfile none swap sw 0 0

UUID=676D-A1AF /boot/efi vfat umask=0077 0 1
```

## 15.5 应用中心安装AutoBangumi跳过首次配置页面之后，无法再次访问

问题现象：在应用中心安装 `AutoBangumi` 之后，首次访问 `http://ip:17892/` 端口的配置页面时，若不配置 qbit 的下载端口时。关闭页面，尝试二次访问该端口时，页面被拒绝访问，连接失败。通过 Docker 应用查看容器的运行日志：

```bash
BASH[2025-09-02 00:30:13] ERROR     [Checker] Downloader connect failed.

[2025-09-02 00:30:13] WARNING:  Downloader is not running.

[2025-09-02 00:30:13] INFO:     Waiting for downloader to start.
```

问题原因：没有配置 qbit 的下载端口， `AutoBangumi` 未能检测到下载器。

解决方法：

- 方法一：停用 `AutoBangumi` 后再启用。
- 方法二：卸载删除数据并重装应用
	1. 卸载 `AutoBangumi` 应用。
	2. 打开 `文件管理` ，定位到 `应用文件` 路径，删除 `AutoBangumi` 文件夹
	3. 重新安装 `AutoBangumi` 应用
	4. 访问 `http://ip:17892/` 端口即可访问成功

## 16 宽带网络总结

## 16.1 会话数（连接数）

会话数（连接数）在线测试工具： [https://qps.itzmx.com](https://qps.itzmx.com/)

连接数为应用产生的网络连接数，例如在爱快首页上看到连接数。并发连接数为 qps ，即每秒发起请求，等于在同一秒内，产生了 250 个请求。

会话数（连接数）的限制，一般普通线路带宽会有会话数限制，专线则没有该限制。触发限制的常见现象为：超过并发连接数上限后，此前已建立连接数的软件和网络请求可正常通信，并且期间 ping 一切正常，延迟没有任何抖动现象，但是新开软件则连不上网络。

## 17 机场相关

## 17.1 自动签到领取机场流量

只要机场基于 `Powered by SSPANEL` 技术构建的，就可以利用这个方案实现。

1. Fork 仓库 `https://github.com/bighammer-link/jichang_checkin` 到自己的 GitHub 帐号
2. `Settings` -> `Secrets and variables` → `Actions` ： `New repository secret`
	- NAME： `URL` ，值填写机场域名网址，例如 `https://example.com` ，尾部不要加 `/` 。
	- NAME：\`CONFIG，值填写机场的帐号（一般是邮箱）和密码。一行账号一行密码。
		- 第一行：帐号，即邮箱
		- 第二行：密码
3. 在 `Actions` 中创建一个 `workflow` ，手动运行一次，后续会每日自动运行。

## 18 后记

~~这光猫还是有断流情况，估计是过热导致掉线，后续考虑下加装小风扇。~~

断流降速故障已查明：虚拟网卡模型设置为 `Intel E1000` 导致的故障，重设为 `VirtIO (半虚拟化)` 即可解决。

在这台工控机上实际使用 openwrt 内网测速工具发现，虚拟机网卡模型设置为 `Intel E1000` 或者 `Realtek RTL8139` 都无法正常发挥网卡的性能， `Intel E1000` 大约测试性能为 100Mbps~300Mbps 不等， `Realtek RTL8139` 模型是最为拉跨的，直接为 10Mbps。使用 `VirtIO (半虚拟化)` 模型内网测速才能恢复到 900Mbps 以上的正常速度。

## 19 All in One (All in Boom) 机器硬件选型总结

## 19.1 CPU 选型对比

All in One 用途大多是使用虚拟机方案实现各子系统，其中虚拟化直通硬件方案需要 CPU 中一个很重要的功能才能实现。对于 Intel CPU 来说，就是 VT-d（该功能可以在 Intel 官方每个 CPU SKU 规格页面查看是否支持）；对于 AMD CPU 来说，就是 IOMMU，也称 AMD-Vi（暂未找到 AMD 官方资料特性说明，根据维基百科资料所言，所有 Ryzen 处理器均支持该功能）。

| CPU 型号 | 内核 / 线程 | 功耗 / W | 频率 / GHz | 核显最大分辨率 | 最大内存规格 / GB | VT-x | VT-d |
| --- | --- | --- | --- | --- | --- | --- | --- |
| [赛扬 ®J1900](https://www.intel.cn/content/www/cn/zh/products/sku/78867/intel-celeron-processor-j1900-2m-cache-up-to-2-42-ghz/specifications.html) | 4/4 | 10 | 2~2.42 | 未知 | 2×8 DDR3L 1333MT/s | 支持 | 不支持 |
| [奔腾 ®N3530](https://www.intel.cn/content/www/cn/zh/products/sku/81074/intel-pentium-processor-n3530-2m-cache-up-to-2-58-ghz/specifications.html) | 4/4 | 4.5~7.5 | 2.16~2.58 | 未知 | 2×8 DDR3L 1333MT/s | 支持 | 不支持 |
| [奔腾 ®J4205](https://www.intel.cn/content/www/cn/zh/products/sku/95591/intel-pentium-processor-j4205-2m-cache-up-to-2-60-ghz/specifications.html) | 4/4 | 10 | 1.5~2.6 | 未知 | 2×8 LPDDR4 2400 MT/s   DDR3L/LPDDR3 1866 MT/s; | 支持 | 支持 |
| [赛扬 ®J4125](https://www.intel.cn/content/www/cn/zh/products/sku/197305/intel-celeron-processor-j4125-4m-cache-up-to-2-70-ghz/specifications.html) | 4/4 | 10 | 2~2.7 | DP：4096x2160@60Hz   HDMI：4096x2160@30Hz   eDP：4096x2160@60Hz | 2×8 DDR4/LPDDR4 2400MT/s | 支持 | 支持 |
| [赛扬 ®J6412](https://www.intel.cn/content/www/cn/zh/products/sku/214758/intel-celeron-processor-j6412-1-5m-cache-up-to-2-60-ghz/specifications.html) | 4/4 | 10 | 2~2.6 | DP：4096 x 2160@60Hz   eDP：4096 x 2160@60Hz | 4x32 LPDDR4/x 3733MT/s   2x64 DDR4 3200MT/s | 支持 | 支持 |
| [N100](https://www.intel.cn/content/www/cn/zh/products/sku/231803/intel-processor-n100-6m-cache-up-to-3-40-ghz/specifications.html) | 4/4 | 6 | 3.4 | DP：4096 x 2160@60Hz   HDMI：4096 x 2160@60Hz | 16 DDR5 4800 MT/s   16 DDR4 3200 MT/s   16 LPDDR5 4800 MT/s | 支持 | 支持 |
| [酷睿™ i7-1255U](https://www.intel.cn/content/www/cn/zh/products/sku/226259/intel-core-i71255u-processor-12m-cache-up-to-4-70-ghz/specifications.html) | 10/12 | 12~55 | E 核 3.5   P 核 4.7 | DP：7680 x 4320 @ 60Hz   HDMI：4096 x 2304 @ 60Hz   eDP：4096 x 2304 @ 120Hz | 64 LDDR5 4800 MT/s   64 DDR4 3200 MT/s   64 LPDDR5 5200 MT/s   64 LPDDR4x 4267 MT/s | 支持 | 支持 |
| [酷睿™ i3-12100](https://www.intel.cn/content/www/cn/zh/products/sku/134584/intel-core-i312100-processor-12m-cache-up-to-4-30-ghz/specifications.html) | 4/8 | 60~89 | 无 E 核   P 核 3.3~4.3 | DP：7680 x 4320 @ 60Hz   HDMI：4096 x 2160 @ 60Hz   eDP：5120 x 3200 @ 120Hz | 128 DDR5 4800 MT/s   128 DDR4 3200 MT/s | 支持 | 支持 |
| [酷睿™ i3-12300](https://www.intel.cn/content/www/cn/zh/products/sku/223095/intel-core-i312300-processor-12m-cache-up-to-4-40-ghz/specifications.html) | 4/8 | 60~89 | 无 E 核   P 核 3.5~4.4 | DP：7680 x 4320 @ 60Hz   HDMI：4096 x 2160 @ 60Hz   eDP：5120 x 3200 @ 120Hz | 128 DDR5 4800 MT/s   128 DDR4 3200 MT/s | 支持 | 支持 |
| [酷睿™ i5-12400](https://www.intel.cn/content/www/cn/zh/products/sku/134586/intel-core-i512400-processor-18m-cache-up-to-4-40-ghz/specifications.html) | 6/12 | 65~117 | 无 E 核   P 核 2.5~4.4 | DP：7680 x 4320 @ 60Hz   HDMI：4096 x 2160 @ 60Hz   eDP：5120 x 3200 @ 120Hz | 128 DDR5 4800 MT/s   128 DDR4 3200 MT/s | 支持 | 支持 |
| [酷睿™ i3-13100](https://www.intel.cn/content/www/cn/zh/products/sku/230575/intel-core-i313100-processor-12m-cache-up-to-4-50-ghz/specifications.html) | 4/8 | 60~110 | 无 E 核   P 核 3.4~4.5 | DP：7680 x 4320 @ 60Hz   HDMI：4096 x 2160 @ 60Hz   eDP：5120 x 3200 @ 120Hz | 192 DDR5 4800 MT/s   192 DDR4 3200 MT/s | 支持 | 支持 |
| [酷睿™ i3-14100](https://www.intel.cn/content/www/cn/zh/products/sku/236774/intel-core-i3-processor-14100-12m-cache-up-to-4-70-ghz/specifications.html) | 4/8 | 60~110 | 无 E 核   P 核 3.5~4.7 | DP：7680 x 4320 @ 60Hz   HDMI：4096 x 2160 @ 60Hz   eDP：5120 x 3200 @ 120Hz | 192 DDR5 4800 MT/s   192 DDR4 3200 MT/s | 支持 | 支持 |
| [酷睿™ i7-12700K](https://www.intel.cn/content/www/cn/zh/products/sku/134594/intel-core-i712700k-processor-25m-cache-up-to-5-00-ghz/specifications.html) | 12/20 | 125-190 | E 核 2.7~3.8   P 核 3.6~4.9 | DP：7680 x 4320 @ 60Hz   HDMI：4096 x 2160 @ 60Hz   eDP：5120 x 3200 @ 120Hz | 128 DDR5 4800 MT/s   128 DDR4 3200 MT/s | 支持 | 支持 |
| [酷睿™ i7-1265U](https://www.intel.cn/content/www/cn/zh/products/sku/226258/intel-core-i71265u-processor-12m-cache-up-to-4-80-ghz/specifications.html) | 10/12 | 12~55 | E 核 3.6   P 核 4.8 | DP：7680 x 4320 @ 60Hz   HDMI：4096 x 2304 @ 60Hz   eDP：4096 x 2304 @ 120Hz | 64 DDR5 4800 MT/s   64 DDR4 3200 MT/s   64 LPDDR5 5200 MT/s   64 LPDDR4x 4267 MT/s | 支持 | 支持 |

BUG 处理器的折腾性思考：小黄鱼上面流通不少 Intel 12 代 ES 测试版处理器，不同步进版本存在不同问题，例如 CPU 没有内建直连独立显卡的 PCI - E 通道，会导致所有主板上第一根直连独立显卡的 PCI - E X16 插槽无效。这类 BUG CPU 虽然价格看着很诱惑，但可能实际运行不稳定，因此不纳入 All in One 硬件选型考虑范围。

## 19.2 主板选型对比

| 主板型号 | 内存插槽 | 供电 | 网口情况 | SATA 支持 | M.2 支持 | PCIe 通道 | 显示接口 | PCB 设计板层 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 云星 Matx-Z690 NAS 主板 | 4 x DDR4 内存槽   最大支持 4 x 32G = 128G | 8 + 1+1 相 | 4 x Intel I226-V 2.5G | 8 x SATA 3.0 | 2 x PCIe4.0 x4 正面   2 x PCIe3.0 x4 反面 | 2 x PCIe4.0 x16 插槽（x8 带宽）   PCIe4.0 x4 插槽（x4 带宽） | HDMI2.0   DP1.4 | 6 层 |

## 20 参考文献

\[1\] Installation Failing: "Failed to prepare EFI boot using Grub"\[EB/OL\]. [https://forum.proxmox.com/threads/installation-failing-failed-to-prepare-efi-boot-using-grub.122002/](https://forum.proxmox.com/threads/installation-failing-failed-to-prepare-efi-boot-using-grub.122002/).  
\[2\] 软路由科普系列 篇二：PVE 安装 iKuai OpenWrt 旁路由 基础设置 保姆级全教程 \[EB / OL\]. [https://post.smzdm.com/p/awrx4lxm/](https://post.smzdm.com/p/awrx4lxm/).  
\[3\] ProxmoxVE 7.4 升级到 8.0，详细步骤 \[EB / OL\]. [https://blog.margrop.net/post/pve-7-upgrade-to-8/](https://blog.margrop.net/post/pve-7-upgrade-to-8/).  
\[4\] PVE 联网及更换国内源 \[EB / OL\]. [https://www.cnblogs.com/pdblogs/p/16218543.html](https://www.cnblogs.com/pdblogs/p/16218543.html).  
\[5\] Force update from unsigned repository\[EB/OL\]. [https://askubuntu.com/questions/732985/force-update-from-unsigned-repository](https://askubuntu.com/questions/732985/force-update-from-unsigned-repository).  
\[6\] ProxmoxVE (PVE) 启用 IOMMU \[EB / OL\]. [https://www.insilen.com/post/501.html](https://www.insilen.com/post/501.html).  
\[7\] Enable IOMMU or VT-d in your motherboard BIOS\[EB/OL\]. [https://us.informatiweb.net/tutorials/it/bios/enable-iommu-or-vt-d-in-your-bios.html](https://us.informatiweb.net/tutorials/it/bios/enable-iommu-or-vt-d-in-your-bios.html).  
\[8\] ProxmoxVE (PVE) 使用 IMG 镜像文件，img 转 qcow2 \[EB / OL\]. [https://www.lxtx.tech/index.php/archives/65/](https://www.lxtx.tech/index.php/archives/65/).  
\[9\]『软路由踩坑指南』篇三：ESXi 8.0 虚拟机安装 iKuai 主路由及保姆级配置 \[EB / OL\]. [https://post.smzdm.com/p/a5op28x7/](https://post.smzdm.com/p/a5op28x7/).  
\[10\]『软路由踩坑指南』篇四：ESXi 8.0 虚拟机安装 openWrt 路由系统终极指南 \[EB / OL\]. [https://post.smzdm.com/p/a7ngxeel/](https://post.smzdm.com/p/a7ngxeel/).  
\[11\]『软路由踩坑指南』篇五：OpenWrt 旁路由进阶篇 SmartDNS + AdGuardHome 设置 DNS 分流、秒开网页、去广告 \[EB / OL\]. [https://post.smzdm.com/p/axz6z7w9/](https://post.smzdm.com/p/axz6z7w9/).  
\[12\] \[openwrt (x86)\] OPenWRT 旁路由 +MosDNS + OpenClash + AdGuard Home 傻瓜配置图文教程 \[EB / OL\]. [https://www.right.com.cn/forum/thread-8284982-1-1.html](https://www.right.com.cn/forum/thread-8284982-1-1.html).  
\[13\] OpenWrt 扩容 Overlay 和 Docker 软件安装空间教程（内置硬盘版）附：Samba 网络共享设置 \[EB / OL\]. [https://www.right.com.cn/forum/thread-7470757-1-1.html](https://www.right.com.cn/forum/thread-7470757-1-1.html).  
\[14\] \[OpenWrt\] 使用 OpenClash 科学上网 \[EB / OL\]. [http://suyu0925.github.io/blog/2022/07/25/openwrt-openclash/](http://suyu0925.github.io/blog/2022/07/25/openwrt-openclash/).  
\[15\] 镜像站使用帮助 - Proxmox 软件仓库 \[EB / OL\]. [https://mirrors.tuna.tsinghua.edu.cn/help/proxmox/](https://mirrors.tuna.tsinghua.edu.cn/help/proxmox/).  
\[16\] 开启直通 \[EB / OL\]. [https://skyao.io/learning-pve/docs/pass-through/enable/](https://skyao.io/learning-pve/docs/pass-through/enable/).  
\[17\] \[经验分享\] 在线连接数测试网页，简单一键测试宽带并发连接数限制 \[EB / OL\]. [https://bbs.ikuai8.com/thread-87196-1-1.html](https://bbs.ikuai8.com/thread-87196-1-1.html).  
\[18\] 禁用 Swap cache \[EB / OL\]. [https://club.fnnas.com/forum.php?mod=viewthread&tid=2492](https://club.fnnas.com/forum.php?mod=viewthread&tid=2492).  
\[19\] 镜像站使用帮助 - Debian 软件源 \[EB / OL\]. [https://mirrors.tuna.tsinghua.edu.cn/help/debian/](https://mirrors.tuna.tsinghua.edu.cn/help/debian/).  
\[20\] 第 5 章 bookworm 中需要注意的问题 \[EB / OL\]. [https://www.debian.org/releases/bookworm//amd64/release-notes/ch-information.zh-cn.html](https://www.debian.org/releases/bookworm//amd64/release-notes/ch-information.zh-cn.html).  
\[21\] 【装机帮扶站】第 881 期：BUG 处理器能选否？聊聊 12 代 ES 处理器 \[EB / OL\]. [https://zhuanlan.zhihu.com/p/458166029](https://zhuanlan.zhihu.com/p/458166029).  
\[22\] 爱快 ikuai 和 windows 电脑使用 iperf3 测试内网传输速度 \[EB / OL\]. [https://www.mulingyuer.com/archives/1073/](https://www.mulingyuer.com/archives/1073/).  
\[23\] PVE8.3.0 折腾记（一）——PVE 初次安装后更换国内源 \[EB / OL\]. [https://blog.csdn.net/weixin\_52626278/article/details/144147244](https://blog.csdn.net/weixin_52626278/article/details/144147244).  
\[24\] 解决 PVE 安装独显后，核显消失，无法直通的问题。\[EB / OL\]. [http://157.0.230.2:8090/archives/owKXsHkN](http://157.0.230.2:8090/archives/owKXsHkN).  
\[25\] PVE 下 LXC ubuntu 中文乱码解决 \[EB / OL\]. [https://blog.csdn.net/zyr920425/article/details/131126045](https://blog.csdn.net/zyr920425/article/details/131126045).  
\[26\] Proxmox VE 常见问题以及解决办法（持续更新）\[EB / OL\]. [https://blog.csdn.net/qq\_40858216/article/details/106784329](https://blog.csdn.net/qq_40858216/article/details/106784329).  
\[27\] Proxmox VE 面板节点和虚拟机提示问号，节点状态 unknown, 储存一直打不开，虚拟机正常的诊断思路 \[EB / OL\]. [https://www.wjxy.net.cn/?p=2629](https://www.wjxy.net.cn/?p=2629).  
\[28\] 给容器设置内核参数 \[EB / OL\]. [https://cloud.tencent.com/developer/article/1583736](https://cloud.tencent.com/developer/article/1583736).  
\[29\] Using sysctls in a Kubernetes Cluster\[EB/OL\]. [https://kubernetes.io/docs/tasks/administer-cluster/sysctl-cluster/#listing-all-sysctl-parameters](https://kubernetes.io/docs/tasks/administer-cluster/sysctl-cluster/#listing-all-sysctl-parameters).  
\[30\] setting key "vm.max\_map\_count": Read-only file system\[EB/OL\]. [https://forum.proxmox.com/threads/setting-key-vm-max\_map\_count-read-only-file-system.28720/](https://forum.proxmox.com/threads/setting-key-vm-max_map_count-read-only-file-system.28720/).  
\[31\] After installation with Ventoy rdinit=/vtoy/vtoy\[EB/OL\]. [https://forum.proxmox.com/threads/after-installation-with-ventoy-rdinit-vtoy-vtoy.153979/](https://forum.proxmox.com/threads/after-installation-with-ventoy-rdinit-vtoy-vtoy.153979/).  
\[32\] 升级至 QEMU 9.2.0 后直通失败 \[EB / OL\]. [https://github.com/gangqizai/igd/issues/27](https://github.com/gangqizai/igd/issues/27).  
\[33\] 每过段时间修改编辑配置那些失效 \[EB / OL\]. [https://bbs.ikuai8.com/forum.php?mod=viewthread&tid=146800&extra=page%3D1%26filter%3Dtypeid%26typeid%3D20](https://bbs.ikuai8.com/forum.php?mod=viewthread&tid=146800&extra=page=1&filter=typeid&typeid=20).  
\[34\] proxmox 安装定义磁盘分区大小 \[EB / OL\]. [https://www.cnblogs.com/weihua2020/p/13723466.html](https://www.cnblogs.com/weihua2020/p/13723466.html).  
\[35\] minfree for lvm snapshots\[EB/OL\]. [https://forum.proxmox.com/threads/minfree-for-lvm-snapshots.28977/](https://forum.proxmox.com/threads/minfree-for-lvm-snapshots.28977/).  
\[36\] Backup fails with Logical Volume already exists in volume group\[EB/OL\]. [https://forum.proxmox.com/threads/backup-fails-with-logical-volume-already-exists-in-volume-group.42377/](https://forum.proxmox.com/threads/backup-fails-with-logical-volume-already-exists-in-volume-group.42377/).  
\[37\] PVE8.3.1 LXC 下安装 OpenWRT 记录 \[EB / OL\]. [https://www.cnblogs.com/airoot/p/18715211](https://www.cnblogs.com/airoot/p/18715211).  
\[38\] LXC 容器中安装 openwrt 作为主路由 \[EB / OL\]. [https://blog.jmal.top/s/lxc-openwrt-main-router](https://blog.jmal.top/s/lxc-openwrt-main-router).  
\[39\] ping: bad address ‘openwrt.org’\[EB/OL\]. [https://blog.axiaoke.cn/archives/1173.html](https://blog.axiaoke.cn/archives/1173.html).  
\[40\] Proxmox VE 容器或虚拟机被锁定解决办法 \[EB / OL\]. [https://www.wort.cloud/post/notes/proxmox-ve-lxc%e5%ae%b9%e5%99%a8%e8%a2%ab%e9%94%81%e5%ae%9a%ef%bc%8c%e7%8a%b6%e6%80%81%e4%b8%barollback%e8%a7%a3%e5%86%b3%e5%8a%9e%e6%b3%95/](https://www.wort.cloud/post/notes/proxmox-ve-lxc%e5%ae%b9%e5%99%a8%e8%a2%ab%e9%94%81%e5%ae%9a%ef%bc%8c%e7%8a%b6%e6%80%81%e4%b8%barollback%e8%a7%a3%e5%86%b3%e5%8a%9e%e6%b3%95/).  
\[41\] kyleflower/clash\_rules\[EB/OL\]. [https://github.com/kyleflower/clash\_rules](https://github.com/kyleflower/clash_rules).  
\[42\] docker 搭建 MyUrls 短链服务, 可配合 sub-web 食用 \[EB / OL\]. [http://213.35.110.223:8000/2024/08/1724228021116](http://213.35.110.223:8000/2024/08/1724228021116).  
\[43\] 自建 Clash 订阅转换 - Subconverter + Subweb + MyUrls 搭建教程 (Docker 版本)\[EB / OL\]. [https://crossbell.io/notes/59373-1](https://crossbell.io/notes/59373-1).  
\[44\] 自建 Clash 订阅转换 - Subconverter + Subweb + MyUrls 搭建教程 - 全 docker 完成 - mihomo 内核 避坑指南 \[EB / OL\]. [https://imgki.com/archives/718.html](https://imgki.com/archives/718.html).  
\[45\] 自建 Clash 订阅转换 \[EB / OL\]. [https://lwnlh.com/14\_Self-built\_Clash\_Subscription\_Conversion.html](https://lwnlh.com/14_Self-built_Clash_Subscription_Conversion.html).  
\[46\].env 配置在 Docker 中不起作用 \[EB / OL\]. [https://github.com/CareyWang/sub-web/issues/68](https://github.com/CareyWang/sub-web/issues/68).  
\[47\] \[BUG\] 使用 Clash Meta 内核开启 tun 模式出现 connect error: dns resolve failed: context deadline exceeded \[EB / OL\]. [https://github.com/clash-verge-rev/clash-verge-rev/issues/104](https://github.com/clash-verge-rev/clash-verge-rev/issues/104).  
\[48\] openclash "all DNS requests failed, first error: dns: bad rdata" 错误解决 \[EB / OL\]. [https://zhuanlan.zhihu.com/p/670131605](https://zhuanlan.zhihu.com/p/670131605).  
\[49\] 我的家里 pve 虚拟机经常死机原来是原因这导致的 \[EB / OL\]. [https://blog.csdn.net/u010042660/article/details/139935170](https://blog.csdn.net/u010042660/article/details/139935170).  
\[50\] pve 节点频繁宕机问题排查 \[EB / OL\]. [https://www.cnblogs.com/nf01/p/16296724.html](https://www.cnblogs.com/nf01/p/16296724.html).  
\[51\] pve8.0 cpu 是 1265u 经常死机，进不去，需要断电重启 \[EB / OL\]. [https://tieba.baidu.com/p/8488486698](https://tieba.baidu.com/p/8488486698).  
\[52\] PBS 维护任务 \[EB / OL\]. [https://www.cnblogs.com/varden/p/15251582.html](https://www.cnblogs.com/varden/p/15251582.html).  
\[53\] 使用 Proxmox Backup Server 备份 Proxmox VE 的客户机与宿主机 \[EB / OL\]. [https://blog.men.ci/backup-pve-with-pbs/](https://blog.men.ci/backup-pve-with-pbs/).  
\[54\] Installation\[EB/OL\]. [https://pbs.proxmox.com/docs/installation.html](https://pbs.proxmox.com/docs/installation.html).  
\[55\] CUSTOM SCHEMES USING.LINK FILES\[EB/OL\]. [https://wiki.debian.org/NetworkInterfaceNames#CUSTOM\_SCHEMES\_USING\_.LINK\_FILES](https://wiki.debian.org/NetworkInterfaceNames#CUSTOM_SCHEMES_USING_.LINK_FILES).  
\[56\] ProxmoxVE 操作指南 (已更新至 pve8.0)\[EB / OL\]. [https://ifwww.com/posts/61099.html](https://ifwww.com/posts/61099.html).  
\[57\] Is there any purge facility to free up disk space?\[EB/OL\]. [https://forum.proxmox.com/threads/is-there-any-purge-facility-to-free-up-disk-space.110575/](https://forum.proxmox.com/threads/is-there-any-purge-facility-to-free-up-disk-space.110575/).  
\[58\] Proxmox-backup logs\[EB/OL\]. [https://forum.proxmox.com/threads/proxmox-backup-logs.98220/](https://forum.proxmox.com/threads/proxmox-backup-logs.98220/).  
\[59\] How does task-log-max-days work?\[EB/OL\]. [https://forum.proxmox.com/threads/how-does-task-log-max-days-work.160627/](https://forum.proxmox.com/threads/how-does-task-log-max-days-work.160627/).  
\[60\] Linux 日志切割神器 logrotate 原理介绍和配置详解 \[EB / OL\]. [https://blog.csdn.net/wangliang888888/article/details/134575630](https://blog.csdn.net/wangliang888888/article/details/134575630).  
\[61\] Linux 自带神器 logrotate 详解 \[EB / OL\]. [https://www.cnblogs.com/txlsz/p/13126723.html](https://www.cnblogs.com/txlsz/p/13126723.html).  
\[62\] 使用 logrotate 实现日志轮转 \[EB / OL\]. [https://blog.csdn.net/LONG\_Yi\_1994/article/details/147464654](https://blog.csdn.net/LONG_Yi_1994/article/details/147464654).  
\[63\] 年轻人折腾的第一台独服 – PVE 7.4 + NAT 内网 + DHCP + K8S 多节点集群 + Istio + CertManager (letsencrypt)\[EB / OL\]. [https://doraemonext.com/archives/347.html](https://doraemonext.com/archives/347.html).  
\[64\] AutoBangumi 安装问题 \[EB / OL\]. [https://club.fnnas.com/forum.php?mod=viewthread&tid=18727](https://club.fnnas.com/forum.php?mod=viewthread&tid=18727).  
\[65\] MetaTube Sever deployment (后端部署)\[EB / OL\]. [https://metatube-community.github.io/wiki/server-deployment/#docker-compose](https://metatube-community.github.io/wiki/server-deployment/#docker-compose).  
\[66\] 【Docker】搭建一个媒体服务器插件后端 API 服务 - MetaTube \[EB / OL\]. [https://blog.mvui.cn/detail/7100.html](https://blog.mvui.cn/detail/7100.html).  
\[67\] 飞牛 NAS 搭建私人图书馆，实现网页、手机看电子书 \[EB / OL\]. [https://club.fnnas.com/forum.php?mod=viewthread&tid=16419](https://club.fnnas.com/forum.php?mod=viewthread&tid=16419).  
\[68\] proxmox container uid/gid mapping\[EB/OL\]. [https://forum.proxmox.com/threads/proxmox-container-uid-gid-mapping.129544/](https://forum.proxmox.com/threads/proxmox-container-uid-gid-mapping.129544/).  
\[69\] PVE 使用 lxc.mount.entry，让 LXC 容器共享宿主机目录 \[EB / OL\]. [https://blog.209902.xyz/archives/1715429836621](https://blog.209902.xyz/archives/1715429836621).

R.I.P