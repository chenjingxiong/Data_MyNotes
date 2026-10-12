---
title: "言小五折腾日记：VMware ESXi 8.0系统安装以及网卡直通 - 那束陽光"
source: "https://eveaz.com/1149.html"
author:
  - "[[言小五]]"
published: 2023-03-16
created: 2025-09-13
description: "本系列文章均以《言小五折腾日记：VMware ESXi 8.0安装iKuai & OpenWrt & Ubuntu & Debian & WIN10 前言篇》为基础，具体实施以及其它文章均在前言篇中提及和更新。……"
tags:
  - "clippings"
---
本系列文章均以《 [言小五折腾日记：VMware ESXi 8.0安装iKuai & OpenWrt & Ubuntu & Debian & WIN10 前言篇](https://eveaz.com/1148.html) 》为基础，具体实施以及其它文章均在前言篇中提及和更新。

ESXi 8.0管理分为物理机管理和网页管理，两者不是一码事。另外，文章截图和步骤太多，若有错误或者遗漏之处，还望在本文下方评论区指正，谢谢。

说明事项1：系列文章为多台不同的准系统搭配硬件拍照或截图而来(截图和拍照有好几套，博主选了对应文章所需图片最全的一套来发文)。  
说明事项2：系列文章的内网IP段分配为192.168.1.\*或者192.168.55.\*。建议根据自己编译的OpenWrt固件默认IP段来选择自己的IP段，省得安装或者升级OpenWrt固件还要手动改IP段。如果是新手抄作业，建议用192.168.1.\*这个IP段。

本文实施步骤大致分为4个步骤：  
1、制作ESXi 8.0启动U盘。  
2、修改BIOS以解决“检测到 TPM 2.0 设备，但无法建立连接”问题。  
3、 ==解决ESXi 8.0虚拟内存占用120G硬盘问题。== U盘启动5秒内要进行额外操作，具体看本文3.1步骤。  
4、物理机安装ESXi 8.0。  
5、ESXi 8.0 网卡直通(直通三个网卡给iKuai)。

### 1、使用UltraISO制作启动U盘

参考本博客2022年01月发文《 [言小五折腾日记：J4125工控软路由安装VMware ESXi以及网卡直通](https://eveaz.com/1142.html) 》中的步骤1操作。

### 2、修改BIOS

2.1、接入显示器，开机按F2或者DEL进入BIOS，进入 Advanced – Trusted Computing – Security Device Support，改为disable。这个步骤是为了解决ESXi 8.0安装完毕之后，ESXi网页管理 主机 菜单中的提示“检测到 TPM 2.0 设备，但无法建立连接”。(注：某些厂家的BIOS没有这个选项，可能就没办法了。)  
![](https://eveaz.com/wp-content/uploads/2023031601.jpg)

### 3、解决默认虚拟内存占用120G硬盘问题

3.1、在U盘加载ESXi 8.0 installer步骤的5秒内，按Shift+O组合键，进入Apply options and boot，默认显示：> runweasel cdromBoot，在后面空一格，输入autoPartitionOSDataSize=8192，之后回车进入安装步骤。(注：该步骤只新装有用，若是装完了再改是无效的。)  
![](https://eveaz.com/wp-content/uploads/2023031602.jpg)
![[Pasted image 20250913141136.png]]
### 4、安装ESXi 8.0

4.1、进入Loading ESXi Installer  
![](https://eveaz.com/wp-content/uploads/2023031603.jpg)

4.2、注意看下图中下方的深色进度条，等着加载完成。  
![](https://eveaz.com/wp-content/uploads/2023031604.jpg)

4.3、加载完成会进入到安装界面，按回车开始安装。  
![](https://eveaz.com/wp-content/uploads/2023031605.jpg)

4.4、这里是条款声明什么的，按F11进入安装步骤。  
![](https://eveaz.com/wp-content/uploads/2023031606.jpg)

4.5、物理机只接入了一个固态硬盘和一个U盘，选择固态硬盘，回车。  
![](https://eveaz.com/wp-content/uploads/2023031607.jpg)

4.6、语言选择，默认US Default即可。  
![](https://eveaz.com/wp-content/uploads/2023031608.jpg)

4.7、输入root密码，要求有大小写字母+数字+字符，比如博主设置为：Aa12341234@；输入两遍密码，回车继续。  
![](https://eveaz.com/wp-content/uploads/2023031609.jpg)

4.8、确认安装到固态硬盘，按F11继续。  
![](https://eveaz.com/wp-content/uploads/2023031610.jpg)

4.9、开始安装了，有进度条。  
![](https://eveaz.com/wp-content/uploads/2023031611.jpg)

4.10、进度条跑完之后，会提示重启。拔掉U盘，回车即可。  
![](https://eveaz.com/wp-content/uploads/2023031612.jpg)

4.11、重启之后，看到下图这个界面，就是启动完成了。按F2。  
![](https://eveaz.com/wp-content/uploads/2023031613.jpg)

4.12、输入用户名：root，和密码：(博主设置的密码为Aa12341234@)，按回车进入设置界面。  
![](https://eveaz.com/wp-content/uploads/2023031614.jpg)

4.13、如果没有特殊情况，只需要设置Configure Management Network即可，回车进入。  
![](https://eveaz.com/wp-content/uploads/2023031615.jpg)

4.14、首先是IPv6 Configuration，回车进入。  
![](https://eveaz.com/wp-content/uploads/2023031616.jpg)

4.15、博主电信宽带无IPv6，这里直接禁用。光标移动到 Disable IPv6，按空格选中，回车。  
![](https://eveaz.com/wp-content/uploads/2023031617.jpg)

4.16、配置IPv4 Configuration，这里一定要设置静态IPv4。前言篇中提到过，博主装了多台，截图有好几套，文章采用最全的一套来的，所以这里截图的IP段为192.168.55.\*。  
`IPv4 Address：192.168.55.254(若非必要，使用192.168.1.254即可)   Subnet Mask：255.255.255.0   Default Gateway:192.168.55.253(这个IP是后续要安装的iKuai主路由IP，若非必要，使用192.168.1.253即可)`  
设置完毕之后，回车继续。  
![](https://eveaz.com/wp-content/uploads/2023031618.jpg)

4.17、配置DNS Configuration，这里手动指定DNS为两个公共DNS，119.29.29.29和114.114.114.114，Hostname自行设置。  
![](https://eveaz.com/wp-content/uploads/2023031619.jpg)

4.18、配置Network Adopter，就是物理网卡。博主习惯是将四网卡的第4个网卡即ETH3作为管理口，建议根据物理机网卡上贴的网卡MAC标签来分辨物理网卡对应的名称，如果没有贴标签，则手动拔插区分(插入网线，后面Status显示Connected就是接好的；4个网卡需拔插4次，每次都要退出Network Adopter然后再进入才能看到Status状态，建议纸质或者电脑记事本留存，后面装iKuai用得上)。比如博主这里尾号fb就是第4个网卡(ETH3)，对应vmnic3，光标移动到vmnic3，按空格选中(有个×)，然后回车，即选中该网卡作为默认管理口(注：这里设置跟ESXi 6.7不同，6.7是要选中4个网卡，ESXi 8.0只需要选中1个，因为8.0中选中的网卡不能直通)。博主这台四网口对应如下：()  
ETH3(物理网卡编号) = 04(ESXi网页管理编号) = vmnic3(ESXi物理设置编号) = fb(vmnic3的MAC地址)  
`ETH0 = 01 = vmnic1 = f8   ETH1 = 02 = vmnin2 = f9   ETH2 = 03 = vmnic0 = fa   ETH3 = 04 = vnmic3 = fb`  
![](https://eveaz.com/wp-content/uploads/2023031620.jpg)

4.19、IPv4、IPv6、DNS设置完成以及网卡编号对应完成之后，按ESC退出ESXi物理设置，会提示重启，按Y重启。  
![](https://eveaz.com/wp-content/uploads/2023031621.jpg)

4.20、重启完成之后，将电脑以网线接入ETH3即之前设置的管理口，修改电脑IP为192.168.55.10(根据自身修改)，默认子网255.255.255.0。打开浏览器，开隐私模式(这里是为了避免载入缓存)，输入192.168.55.254，会提示不安全，选择继续进入即可。  
![](https://eveaz.com/wp-content/uploads/2023031622.jpg)

4.21、进入到ESXi登录界面，输入用户名：root，密码：(博主设置的密码为Aa12341234@)。回车。  
![](https://eveaz.com/wp-content/uploads/2023031623.jpg)

4.22、进入到ESxi了，首次登录有个欢迎页面，这里默认勾选了加入VMware客户体验改进计划，取消勾选，点确定。  
![](https://eveaz.com/wp-content/uploads/2023031624.jpg)

4.23、如下图所示，进入ESXi 8.0之后，在 主机 菜单下有个显眼的“检测到 TPM 2.0 设备，但无法建立连接”。如果你有按本文2.1步骤操作，则没有这个提示。  
![](https://eveaz.com/wp-content/uploads/2023031625.jpg)

4.24、如下图所示，按本文2.1步骤操作之后，是没“检测到 TPM 2.0 设备，但无法建立连接”提示的。  
![](https://eveaz.com/wp-content/uploads/2023031626.jpg)

4.25、进入菜单 管理 – 许可，分配许可证。下图可看到不添加许可证的话，两个月之后就过期。  
![](https://eveaz.com/wp-content/uploads/2023031627.jpg)

4.26、输入许可证编号，若自己买了许可证就输入自己的，要么就网上找。博主这里怎么可能不给许可证打码呢(/狗头)。输入许可证之后，点击检查许可证。  
![](https://eveaz.com/wp-content/uploads/2023031628.jpg)

4.27、许可证有效。点击分配许可证。  
![](https://eveaz.com/wp-content/uploads/2023031629.jpg)

4.28、如下图所示，过期日期为：从不。  
![](https://eveaz.com/wp-content/uploads/2023031630.jpg)

4.29、回到ESXi菜单 – 主机，可看到目前硬件占用情况，如下图所示，虚拟内存为7.75GB容量(即本文3.1步骤的操作)。  
![](https://eveaz.com/wp-content/uploads/2023031631.jpg)

### 5、网卡直通

5.1、进入ESXi菜单 – 管理 – 硬件 – PCI 设备。下方筛选“支持直通”，即可看到网卡编号(见本文4.18步骤)。勾选01、02、03三个网卡，点击切换直通。  
![](https://eveaz.com/wp-content/uploads/2023031632.jpg)

5.2、如下图所示，可以看到三个网卡直通状态由禁用变成活动，会提示需要重新引导，先不管。  
![](https://eveaz.com/wp-content/uploads/2023031633.jpg)

5.3、开启网卡的混杂模式。  
此步骤是为了能从任意网口进入ESXi网页管理，但有个前提，网卡必须是激活使用状态。  
这里啰嗦几句，3个网卡直通给iKuai之后，如果iKuai没启动，那么只能通过本文4.18步骤提到的管理口即ETH3才能进入ESXi网页管理。

5.3.1、进入ESXi菜单 网络 – 虚拟交换机，可以看到vSwitch0，点击vSwitch0进入。  
![](https://eveaz.com/wp-content/uploads/2023031634.jpg)

5.3.2、进入vSwitch0页面，点击编辑设置。  
![](https://eveaz.com/wp-content/uploads/2023031635.jpg)

5.3.3、将安全选项里面的混杂模式、MAC地址更改、伪传输都改为接受，然后保存。之后，回到ESXi菜单 – 主机，点击重新引导主机，这时候ESXi会重启。  
![](https://eveaz.com/wp-content/uploads/2023031636.jpg)

5.3.4、(非必须)ESXi重启完成之后，再次进入ESXi物理机设置 Network Adopters，可以看到，三个直通出去的网卡不显示了，只显示预留的管理口。  
![](https://eveaz.com/wp-content/uploads/2023031637.jpg)

以上，本文完结。后续文章请回到本文开头的前言篇中查看。