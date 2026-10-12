---
title: "Esxi_FAQ_2024.6.3"
source: "https://blog.csdn.net/qq_41210783/article/details/139413313?spm=1001.2101.3001.4242.1&utm_relevant_index=2"
author:
  - "[[qq_41210783]]"
published: 2024-06-03
created: 2025-09-13
description: "文章浏览阅读3.6k次，点赞8次，收藏10次。问题内存设置无效: 内存预留 (sched.mem.min) 应该等于内存大小 (32768设置的内存容量)原因是因为直通了nvme硬盘，在没有直通nvme硬盘时，不预留内存开机也不会报错。解决需要全部预留内存勾上。_内存设置无效: 内存预留 (sched.mem.min) 应该等于内存大小"
tags:
  - "clippings"
---
### 问题1-内存设置无效

**问题：** 内存设置无效: 内存预留 (sched.mem.min) 应该等于内存大小 (32768设置的内存容量)

![image](https://i-blog.csdnimg.cn/blog_migrate/35bebad82798cfe50974e52c4a01c577.png)

**原因：** 是因为直通了nvme硬盘，在没有直通nvme硬盘时，不预留内存开机也不会报错。

**解决：** 需要全部预留内存勾上。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a76ff7227df9a3c842c4654bba9ec882.jpeg)

### 问题2-模块“DevicePowerOn”打开电源失败

**问题** 已经直通了的nvme硬盘在别的虚拟机添加pci设备仍能添加选择直通。但开机会报错，失败 - 模块“DevicePowerOn”打开电源失败。

**原因：** 点详细信息看见 键：haTask-2-vim.VirtualMachine.powerOn-3867480738，没有提到直通。

**解决：** 虚拟交换机问题。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/75e340d92f3ae20a32accf9c697b234f.png)

要这样设置才可以透过vlan交换机拨号。

### 问题3-在当前状况下不允许执行此操作

**问题：** 删除虚拟机提示“在当前状况下不允许执行此操作”

**原因：** 引起此故障的原因是 vCenter Server 没有正常关机导致。（环境为VC部署在esxi主机上，在关机进，没有先关闭vcenter server，而是直接在Esxi主机上执行关机操作导致。）

**解决：** 将Esxi主机从VC中断开链接，再重新连接即可。

在vSphere中，可以通过使用vSphere Web客户端（HTML5界面）或SSH方式来从vCenter服务器中断开ESXi主机的连接。以下是使用SSH方式的示例步骤：

1. 使用SSH客户端连接到ESXi主机。
2. 登录到ESXi主机的shell。
3. 使用以下命令断开与vCenter的连接：
	/etc/init.d/hostd stop  
	/etc/init.d/hostd start