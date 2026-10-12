---
title: "从 ESXi/ESX 命令行重新挂载已断开连接的 NFS 数据存储_esxi nfs 掉线-CSDN博客"
source: "https://blog.csdn.net/oLanTianZhiYi/article/details/105577501"
author:
  - "[[成就一亿技术人!]]"
  - "[[hope_wisdom 发出的红包]]"
published:
created: 2025-09-14
description: "文章浏览阅读6k次。本文详细介绍了当NFS存储与ESXi主机断开连接后，如何通过esxcli命令来移除并重新挂载NFS数据存储，解决无法重新挂载或完全卸载的问题。"
tags:
  - "clippings"
---
AI 搜索

## 从 ESXi/ESX 命令行重新挂载已断开连接的 NFS 数据存储

最新推荐文章于 2025-07-27 11:58:12 发布

[蓝天之翼](https://blog.csdn.net/oLanTianZhiYi "蓝天之翼") 于 2020-04-17 13:15:35 发布

CC 4.0 BY-SA版权

版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

本文链接： [https://blog.csdn.net/oLanTianZhiYi/article/details/105577501](https://blog.csdn.net/oLanTianZhiYi/article/details/105577501)

本文详细介绍了当NFS存储与ESXi主机断开连接后，如何通过esxcli命令来移除并重新挂载NFS数据存储，解决无法重新挂载或完全卸载的问题。

> https://editor.csdn.net/md/?not\_checkout=1

由于某些原因NFS存储会和ESXi主机断开连接  
当NFS恢复后无法重新挂载或完全卸载

要移除数据存储，请执行以下操作：  
运行以下命令以列出主机上挂载的数据存储：

```bash
esxcli storage nfs list
bash1
```

记录步骤 1 中的 NFS 数据存储。运行以下命令以删除 NFS 挂载：

```bash
esxcli storage nfs remove -v NFS_Datastore_Name
bash1
```

注意：此操作不会删除共享上的信息，它会从主机中卸载共享。如果 NFS 数据存储正由虚拟机或第三方脚本使用，您可能会看到以下错误：

执行操作时出错: NFS 错误: 无法卸载文件系统: `忙碌 (Error performing operation: NFS Error: Unable to Unmount filesystem: Busy)。`  
要添加数据存储，请执行以下操作：

运行以下命令以挂载 NFS 数据存储：

```bash
esxcli storage nfs add -H NFS_IP|NFS_HOSTNAME -s Share_mount_point_on_the_NFS -v DatastoreName
bash1

esxcli storage nfs41 add -H 192.168.9.6 -s /nfs/ -v NAS-NFS

bash1


```[手动配置 *ESXi* 主机 *挂载* *NFS* 的最大值](https://is-cloud.blog.csdn.net/article/details/53995394)

[烟云的计算](https://blog.csdn.net/Jmilk)

01-03 4555[目录目录 资料 手动配置 *ESXi* 主机 *挂载* *NFS* 的最大值资料官方 KB 地址手动配置 *ESXi* 主机 *挂载* *NFS* 的最大值 Open: 清单 ==> *ESXi* 主机 ==> 配置 ==> 高级配置 Set 1: *NFS* ==> *NFS*.MaxVolumes 限制 vSphere *ESXi* / *ESX* 主机可同时 *挂载* 的 *NFS* *数据存储* 数量。 *ESXi* / *ESX* 3.x:将 *NFS*.MaxVolume](https://is-cloud.blog.csdn.net/article/details/53995394)[*Esxi* 5.1 *挂载* *NFS* 存储遇到的问题及解决办法\_mount failed: unable to...](https://blog.csdn.net/u013385554/article/details/97830165)

8-27[1、使用 VI/vSphereClient *连接* 到 Virtual Center/vCenterServer。 2、选择 *ESX* / *ESXi* 主机。 3、单击配置选项卡。 4、单击网络。 5、查看 VMKernel 的网络图,或单击属性 > 端口 > VMKernel。如果 VMKernel 未列出,必须添加它。 手动 *挂载* 手动 *挂载* 命令: *esx* cli storage *nfs* add -H *NFS* \_IP| *NFS* \_HOSTNAME -s...](https://blog.csdn.net/u013385554/article/details/97830165)[VMware *ESXI* *连接* 群晖NAS\_ *esxi* *挂载* *nfs* 存储](https://blog.csdn.net/weixin_44164294/article/details/143992202)

9-11[VMware *ESXI* *连接* 群晖NAS 登录 *ESXI* 选择存储- *数据存储* -新建数据储存 新建- *挂载* *NFS* *数据存储* 这个页面先放着,然后登录群晖Web管理页面,进入控制面板,选择文件服务 *NFS* 服务,作如下配置: 然后选择共享文件件,进行如下选择 在新增页面里进行填写 最重要的是 *ESXI* 的IP地址要正确...](https://blog.csdn.net/weixin_44164294/article/details/143992202)[6、VMware *ESXi* 与vCenter Server安装及配置指南](https://blog.csdn.net/gaochao/article/details/149386476)

[gaochao的博客](https://blog.csdn.net/gaochao)

05-09 88[本文详细介绍了VMware *ESXi* 和vCenter Server的安装、配置及管理方法。内容涵盖 *ESXi* 有状态模式配置、安装后必要的设置（如vSphere客户端安装、网络重配置、时间同步和名称解析）、vCenter Server的核心服务与部署规划，以及其主要功能如资源管理、模板管理、虚拟机管理和监控任务等的使用。此外，文章还总结了关键操作建议、常见问题解决方法，并展望了未来虚拟化技术的发展趋势。适用于希望构建高效稳定虚拟环境的企业管理员参考学习。](https://blog.csdn.net/gaochao/article/details/149386476)[迷你主机 *Esxi* 6.7 *挂载* 新硬盘](https://blog.csdn.net/m0_46246804/article/details/139329066)

[iamblackcat's blog](https://blog.csdn.net/m0_46246804)

05-31 4671[在mini主机上的 *ESXi* 6.7 *挂载* 新硬盘](https://blog.csdn.net/m0_46246804/article/details/139329066)[*ESXi* *挂载* *NFS* 共享存储](https://blog.csdn.net/OliverLion/article/details/102584819/)

8-29[*ESXi* *挂载* *NFS* 共享存储 本文详述了如何在VMware *ESXi* 主机上通过 *NFS* 协议 *挂载* 共享存储卷的过程,包括配置步骤与常见错误排查方法。 摘要生成于C知道,由 DeepSeek-R1 满血版支持,前往体验 > 通常VMware的整体架构由三个部分组成,虚拟化环境(包括 *ESXi* 与vCenter以及VM),交换机(通常为万兆交换机或光纤交换机),存储(netap、...](https://blog.csdn.net/OliverLion/article/details/102584819/)[VMware *ESXi* 5.5主机无法 *挂载* RHEL6.5 *NFS* 存储](https://blog.csdn.net/weixin_34357887/article/details/85030688)

9-10[这是由于 *NFS* 服务器上的服务用的是默认的UDP协议方式传输,而VMware要求必须用TCP,因此修改配置文件/etc/ *nfs* mount.conf,取消Defaultproto=tcp行注释即可。当然有的时候,如果你修改了这项还不能成功 *挂载* 的话,请检查如下: 一、防火墙是否关闭 service iptables stop...](https://blog.csdn.net/weixin_34357887/article/details/85030688)[*NFS* 与自动 *挂载*](https://blog.csdn.net/qq_41904732/article/details/104175169)

[qq\_41904732的博客](https://blog.csdn.net/qq_41904732)

02-04 545[1.*NFS* 共享文件server端的配置（server ip：172.16.8.11） \[root@localhost ~\]# getenforce Enforcing \[root@localhost ~\]# setenforce 0 \[root@localhost ~\]# getenforce Permissive \[root@localhost ~\]# firewall-cmd --perm...](https://blog.csdn.net/qq_41904732/article/details/104175169)[解决：VMware *ESXI* *挂载* 文件至虚拟机](https://blog.csdn.net/qq_36813879/article/details/149686880)

[

最新发布

](https://blog.csdn.net/qq_36813879/article/details/149686880)

[qq\_36813879的博客](https://blog.csdn.net/qq_36813879)

07-27 574[可以使用软碟通（UltraISO）完成。打开> 选择SSH文件 > 另存 > 即可变成iso文件。在 *ESXI* 界面，点击左侧 > 存储 > *数据存储* 浏览器 > 上传 >将镜像上传平台。找到虚拟机 > 配件配置 > CD/DVD驱动器1 > 选择镜像文件。开机，进入到虚拟机桌面，左侧会看到光驱图标，点击里面便是打包的内容。关掉虚拟机 > 操作 > 编辑设置 > 勾选 CD/DVD驱动器。](https://blog.csdn.net/qq_36813879/article/details/149686880)[*ESXi* 存储\_ *esxi* *nfs*](https://blog.csdn.net/omaidb/article/details/131914303)

9-2[1.安装 *NFS* 服务端 https://blog.csdn.net/omaidb/article/details/120926883 2.*ESXi* *连接* *NFS* 存储 *ESXi* 想要 *连接* *NFS* 服务器,需要为 *NFS* 服务器配置一个vmkernel的ip地址,这样 *NFS* 服务器才能和 *ESXi* 正常网络互通。 # 查看 *nfs* 版本 *nfs* stat-m *nfs* 服务器 AI生成项目shell...](https://blog.csdn.net/omaidb/article/details/131914303)[kubesphere 问题处理:*nfs* \_a lockfile from a previous execution...](https://blog.csdn.net/qq_47767419/article/details/143767212)

9-7[kubesphere 问题处理:*nfs* 一 问题报错level=error err="opening storage failed: lock DB directory: no locks available 一、背景介绍 首先说明,该问题应该是一个小众问题 在学习K8S过程中我使用位于 *esxi* 上的三台Ubuntu虚拟机和kubekey进行了集群的搭建,使用了另外一台TrueNAS虚拟机作为 *NFS* 服务器,作为集群的高可用搭建...](https://blog.csdn.net/qq_47767419/article/details/143767212)[*ESXI* 网卡组故障切换、故障恢复或 vMotion 导致网络 *连接* 丢失](https://blog.csdn.net/wl137881454/article/details/106396038)

[wl137881454的博客](https://blog.csdn.net/wl137881454)

05-28 8077[*ESXI* 中，当利用网卡组配置且组中的某个网络适配器发生故障时，进行故障切换或故障恢复后虚拟交换机上的所有端口组的网络 *连接* 都将丢失。在 vMotion 至其他 *ESX* / *ESXi* 主机后，虚拟机上的网络 *连接* 丢失。造成该类现象的原因是：故障切换顺序中位居收尾的物理网卡遇到间歇性故障，故障恢复策略可能导致平凡更换使用的网卡，上层物理交换机MAC表地址频繁更改，造成网络丢失。该丢失的具体解决办法为：https://kb.vmware.com/s/article/1003969?lang=zh\_CN。...](https://blog.csdn.net/wl137881454/article/details/106396038)[linux *挂载* VMFS硬盘,*esxi* 4.i *重新* *挂载* vmfs](https://blog.csdn.net/weixin_26732855/article/details/116837410)

[weixin\_26732855的博客](https://blog.csdn.net/weixin_26732855)

05-11 551[一台 *esxi* 4.1　测试主机，本地硬盘sata 500G(*esxi* 系统跟VMFS)　本地硬盘sata 500G(VMFS)某天因为主机硬件需要更换，于是把两块本地硬盘放到另一台主机上，正常开机发现第二块硬盘没有 *挂载* 上来，在vcenter里添加存储器无法添加硬盘，提示如下选定的磁盘已具有VMFS *数据存储* ，请选择另一磁盘这种问题通常由于下列原因导致：多台 *ESXi* / *ESX* 主机由同一台vCenter...](https://blog.csdn.net/weixin_26732855/article/details/116837410)[手动设置 *ESXi* 主机 *挂载* *NFS* 的最大值\_ *esxi* *挂载* *nfs*](https://blog.csdn.net/XsiJava/article/details/133474903)

9-6[*ESXi* 主机 *挂载* *NFS* 的最大值默认情况下是 32。如果你的应用场景中需要更大的最大值,你可以按照以下步骤手动配置。 步骤一:登录 *ESXi* 主机 首先,使用 SSH 客户端登录到你的 *ESXi* 主机。你需要具有管理员权限才能进行以下操作。 步骤二:编辑 *NFS* 服务配置文件...](https://blog.csdn.net/XsiJava/article/details/133474903)[*ESXi* 6.5挂在外置移动硬盘或U盘作为存储](https://devpress.csdn.net/v1/article/detail/106424058)

[

热门推荐

](https://devpress.csdn.net/v1/article/detail/106424058)

[syh6610的博客](https://blog.csdn.net/syh6610)

05-29 1万+[进入 *ESXi* 单节点控制台开启ssh 使用xshell或secureCRT等终端工具远程进入系统中。 在不接入移动硬盘或U盘的情况下输入一下指令 关闭usbarbitrator服务 /etc/init.d/usbarbitrator stop 关闭usbarbitrator服务开机自启 chkconfig usbarbitrator off 接入你的USB硬盘，查找usb存储设备，这时我们可以看到（Is USB:true）说明我们的usb硬盘已经被识别出来 *esx* cli storage co.](https://devpress.csdn.net/v1/article/detail/106424058)[vmware 6.5 主机卸载 *NFS* 存储失败故障处理](https://blog.csdn.net/hyonex/article/details/138205796)

[hyonex的博客](https://blog.csdn.net/hyonex)

04-26 864[参考在 “vSphere 7.0 主机卸载 *NFS* 存储失败故障处理” 文中的方法并未真正解决 *nfs* 失败问题。其中，vol\_ *nfs* -2 是 *NFS* *数据存储* 名称。真正干净暴力解决问题。](https://blog.csdn.net/hyonex/article/details/138205796)[*ESXi* 主机失去与 *ESXi* 和 VMFS5 *数据存储* 的 *连接*](https://blog.csdn.net/weixin_34221112/article/details/85126272)

[weixin\_34221112的博客](https://blog.csdn.net/weixin_34221112)

09-22 2016[在虚拟化环境中使用 VAAI ATS 检测信号时， *ESXi* 5.5 Update 2 或 *ESXi* 6.0 主机失去与 VMFS5 *数据存储* 的 *连接* ，会造成 *ESXI* 失联，但其上的虚拟机仍可以PING通，业务不受影响，但有一定延时。 要解决此问题，可以通过在 *ESXi* 内核中禁用 ATS 检测信号，将检测信号相关的活动恢复为旧方法。对于 VMFS5 *数据存储* ：1、首先登陆exsi的 *命令行* 执...](https://blog.csdn.net/weixin_34221112/article/details/85126272)[让 *ESX* *连接* 上 *NFS* Server](https://blog.csdn.net/flexitime/article/details/6182104)

[flexitime的专栏](https://blog.csdn.net/flexitime)

02-13 1699[我有很多 ISO 文件放在 *NFS* Server，每次 *ESX* 要安装新系统时总是要从它上面传一个上去，后来发现其实可以用 *ESX* 来直连 *NFS* Server 的 *esx* cfg-firewall -e *nfs* Client service portmap restart service *nfs* restart cd /vmimages mkdir iso mount 172.25.XXX.XXX:/XXX /vmimages/iso （这个你懂的）……](https://blog.csdn.net/flexitime/article/details/6182104)[*nfs* hung 导致xenserver或 *esxi* 虚拟机远程 *连接* 不上](https://blog.csdn.net/weixin_34062329/article/details/92816836)

[weixin\_34062329的博客](https://blog.csdn.net/weixin_34062329)

10-10 204[今天在 *esxi* 下将一个虚拟机clone下，进行到大概40%的时候，忽然不动了， *esxi* 上面的虚拟机远程桌面也 *连接* 不上了。说明下我的环境： xenserver主机 ----------某存储厂商低端存储（利用 *NFS* 共享存储给xenserver和 *esxi* 主机使用） *ESXi* 4.1主机 登陆到存储上...](https://blog.csdn.net/weixin_34062329/article/details/92816836)[*ESXi* 找回并丢失了的虚拟机](https://devpress.csdn.net/v1/article/detail/107857827)

[m0\_46435052的博客](https://blog.csdn.net/m0_46435052)

08-07 6723[事情的原因说明 2020年的8月5日，由于同事手贱，有不懂RAID5，没事请把磁盘给拔出来了，但是多台服务的磁盘搞乱了，悲剧发生，服务器无法启动了，一台是公司的gitlab， 一台是公司的数据库，都是要命的；两台都做了RAID5都是300G的7200转硬盘； 我一个晚上就睡不着了，该如何恢复呢？ 我知道RAID5可能有硬盘顺序的问题（老版本），幸好我的设备DELL的7100R，没有RAID5的硬盘顺序问题，RAID5的相关信息都是写在磁盘上的，插入后应该可以自动回...](https://devpress.csdn.net/v1/article/detail/107857827)[解决存储阵列无法 *挂载* ，或者 *挂载* 成功后，重启服务器后存储阵列又要 *重新* *挂载*](https://blog.csdn.net/pangsmao/article/details/105787325)

[pangsmao的博客](https://blog.csdn.net/pangsmao)

04-27 3081[环境：ubuntu18.04 问题：我要把存储阵列 *挂载* 到目录/public，但是这个目录无法 *挂载* 存储阵列，或者 *挂载* 成功后，重启服务器后存储阵列又要 *重新* *挂载* ，而 *挂载* 到其他目录就没有问题。 原因分析：/public目录被多次 *挂载* 了。 解决： fdisk -l 可以看到返回的信息里面，有一行如上图所示，存储阵列的名字是/dev/sdb，用如下命令查看存储阵列的uuid： ls -l /d...](https://blog.csdn.net/pangsmao/article/details/105787325)[*ESXi* 6.7 虚拟机内 *挂载* 主机物理硬盘的办法(无需格式化)](https://devpress.csdn.net/v1/article/detail/143205311)

[qq\_28120987的博客](https://blog.csdn.net/qq_28120987)

10-24 4161[【代码】 *ESXi* 6.7 虚拟机内 *挂载* 主机物理硬盘的办法(无需格式化)](https://devpress.csdn.net/v1/article/detail/143205311)[*esxi* 主机通过 *命令行* 查看vmdk文件 *挂载* 情况](https://wenku.csdn.net/answer/47k0ds51y1)

09-19[在 *ESXi* 主机上，通过 *命令行* 查看VMDK（Virtual Disk Format）文件的 *挂载* 情况，你可以使用\`vmkfstools\`工具或直接查阅主机的日志文件。以下是两种方法： 1. \*\*使用vmkfstools\*\*: - 首先，你需要登录到 *ESXi* 主机，通常...](https://wenku.csdn.net/answer/47k0ds51y1)

评论

被折叠的 0 条评论 [为什么被折叠?](https://blogdev.blog.csdn.net/article/details/122245662)[到【灌水乐园】发言](https://bbs.csdn.net/forums/FreeZone)

添加红包

实付 元

[使用余额支付](https://blog.csdn.net/oLanTianZhiYi/article/details/)

点击重新获取

扫码支付

钱包余额 0

抵扣说明：

1.余额是钱包充值的虚拟货币，按照1:1的比例进行支付金额的抵扣。  
2.余额无法直接购买下载，可以购买VIP、付费专栏及课程。

[余额充值](https://i.csdn.net/#/wallet/balance/recharge)

举报

 [![](https://csdnimg.cn/release/blogv2/dist/pc/img/toolbar/Group.png) 点击体验  
DeepSeekR1满血版](https://ai.csdn.net/?utm_source=cknow_pc_blogdetail&spm=1001.2101.3001.10583) 隐藏侧栏 ![程序员都在用的中文IT技术交流社区](https://g.csdnimg.cn/side-toolbar/3.6/images/qr_app.png)

程序员都在用的中文IT技术交流社区

![专业的中文 IT 技术社区，与千万技术人共成长](https://g.csdnimg.cn/side-toolbar/3.6/images/qr_wechat.png)

专业的中文 IT 技术社区，与千万技术人共成长

![关注【CSDN】视频号，行业资讯、技术分享精彩不断，直播好礼送不停！](https://g.csdnimg.cn/side-toolbar/3.6/images/qr_video.png)

关注【CSDN】视频号，行业资讯、技术分享精彩不断，直播好礼送不停！

客服 返回顶部