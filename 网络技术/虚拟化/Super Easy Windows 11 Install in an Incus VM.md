---
title:
source: "https://discussion.scottibyte.com/t/super-easy-windows-11-install-in-an-incus-vm/679"
author:
  - "[[ScottiByte's Discussion Forum]]"
published: 2025-09-07
created: 2026-03-15
description:
tags:
---
## 在Incus虚拟机中安装超简单Windows 11

## 斯科特于2025年9月7日发布

[![](https://discussion.scottibyte.com/letter_avatar_proxy/v4/letter/s/3e96dc/144.png)](https://discussion.scottibyte.com/u/scott)

[斯科特](https://discussion.scottibyte.com/u/scott)

[2025年9月](https://discussion.scottibyte.com/t/super-easy-windows-11-install-in-an-incus-vm/679 "Post date")

新的 Incus 6.16 版本支持虚拟机的 CD-ROM 处理。这意味着 Distrobuilder 不再需要来创建 Incus Windows 11 虚拟机。在新CD-ROM中，通过USB总线将ISO连接到虚拟机时，虚拟光驱会以CD形式正确地暴露在虚拟机中。

早在2022年2月，我介绍了 [Windows 11 LXD虚拟机](https://www.youtube.com/watch?v=r0WheXvADHk) ，2024年2月又介绍了 [Windows 11 Incus虚拟机](https://www.youtube.com/watch?v=EVMKfGRx8eg) ，这两项安装都需要用名为Distrobuilder的工具重新打包Windows的ISO映像，以整合合适的驱动程序以配合LXD/Incus。

Incus 6.16及以后版本取消了这一要求，允许在Incus中更轻松地安装Windows虚拟机。

首先访问 [Windows下载页面](https://microsoft.com/software-download/windows11) 。

[![图片](https://discussion.scottibyte.com/uploads/default/optimized/2X/f/fd66c5cf332f7b9ddfc39e78d5e5c699b3bcd6e0_2_690x388.jpeg)](https://discussion.scottibyte.com/uploads/default/original/2X/f/fd66c5cf332f7b9ddfc39e78d5e5c699b3bcd6e0.jpeg "image")

如教程所示，向下滚动页面，按照选项下载适用于x64设备的Windows 11磁盘镜像（ISO）。

[![图片](https://discussion.scottibyte.com/uploads/default/optimized/2X/b/bd62de2b6fb572d87a26d0fd44e6b6650e389ac3_2_690x321.jpeg)](https://discussion.scottibyte.com/uploads/default/original/2X/b/bd62de2b6fb572d87a26d0fd44e6b6650e389ac3.jpeg "image")

下载完成后（大约5.8GB），把ISO文件移到你的Incus服务器上。

在你的Incus服务器上打开一个终端。如果你不知道Incus是什么，它是一个强大的虚拟化系统，支持容器和虚拟机。观看我名为 [《Incus 容器逐步](https://www.youtube.com/watch?v=ULPuU9aKyoU) 解析》的教程。

在你的Incus服务器上，确认你运行的是Incus v6.16或更高版本。如果你需要升级你的Incus服务器，就升级吧。

```
incus version
```

Incus 要求 Windows 使用 Redhat VirtIO 驱动。用以下命令将它们下载到你的Incus服务器：

```bash
wget https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/virtio-win-0.1.271-1/virtio-win-0.1.271.iso
```

用以下命令创建一个空虚拟机。通常，Incus 虚拟机是基于已有的操作系统镜像创建的。安装Windows时不支持这个选项。

```
incus init win11vm --empty --vm
```

为我们的虚拟机设置系统磁盘大小，并让它使用NVMe总线作为驱动器。

```lua
incus config device override win11vm root size=85GiB io.bus=nvme
```

为虚拟机设置所需的虚拟CPU核心和内存数量。

```bash
incus config set win11vm limits.cpu=4 limits.memory=8GiB
```

Windows 有硬件要求必须配备可信平台模块（TPM）芯片以支持 Secureboot。我们可以创建一个虚拟的TPM设备。

```lua
incus config device add win11vm vtpm tpm path=/dev/tpm0
```

我们用新的io.bus=usb将Windows的ISO作为CD-ROM设备附加。这是ISO的虚拟附加，使其看起来像USB挂载的ISO。boot.priority=10确保Windows光盘映像先启动内部硬盘。ISO文件的源路径需要完整的绝对路径名，因此请相应调整你的路径。

```bash
incus config device add win11vm install disk source=/home/scott/demo/Win11_24H2_English_x64.iso io.bus=usb boot.priority=10
```

同样，将Redhat VirtIO驱动ISO连接到第二个光盘设备，确保调整路径。较低的启动优先级确保该硬盘不会尝试启动。

```bash
incus config device add win11vm virtio disk source=/home/scott/demo/virtio-win-0.1.271.iso io.bus=usb boot.priority=5
```

现在我们会启动新虚拟机，使其从Windows ISO启动，并用以下命令启动Windows安装。

```sql
incus start win11vm --console=vga
```

如果出现控制台无法显示的错误，请确保你安装了遥控器。

```bash
sudo apt install virt-viewer
```

如果你的Incus服务器远离桌面，你可能需要阅读我的文章 [《从Windows管理你的Incus服务器](https://discussion.scottibyte.com/t/manage-your-incus-server-from-windows/402) 》。原因是显示虚拟机控制台的选项默认你的桌面和Incus服务器是同一个平台。

如果不是这样，我链接的文章对Linux用户也适用，因为它展示了如何在远程服务器上创建“信任”来运行Incus命令。也可以在Linux上安装Incus桌面客户端，而无需安装服务器。

我经常在台式机上用这个功能打开虚拟机上的远程观察器，虚拟机运行在我的另一个Incus服务器上。例如：

```sql
incus start vmsrain:win11 --console=vga
```

上面的命令之所以能用，是因为 vmsrain 对我的台式机有信任，而且我的台式机上装了 virt-viewer。Linux 很棒，因为程序不一定要在你所在的电脑上运行。它们可以远程运行，并且可以本地控制。

好了，别再说了......

所以，当你第一次运行虚拟机时，

```sql
incus start win11vm --console=vga
```

如果你太慢，无法按下“按任意键从CD或DVD启动.....”，屏幕可能会显示如下。

[![图片](https://discussion.scottibyte.com/uploads/default/optimized/2X/e/e5026198639314b5bff8419f86260acfc4167af8_2_690x261.png)](https://discussion.scottibyte.com/uploads/default/original/2X/e/e5026198639314b5bff8419f86260acfc4167af8.png "image")

如果是这样，做一个CTRL ALT DEL，遥视器就会退出，你就会回到终端。

[![图片](https://discussion.scottibyte.com/uploads/default/original/2X/7/7cd9074cfe287299eb1bc97d1d44db64f02fddc2.png)](https://discussion.scottibyte.com/uploads/default/original/2X/7/7cd9074cfe287299eb1bc97d1d44db64f02fddc2.png "image")

如果有，请重置并用以下两个命令重新尝试：

```sql
incus stop win11vm -f
incus start win11vm --console=vga
```

如果你按键够快，应该会看到如下画面。

[![图片](https://discussion.scottibyte.com/uploads/default/optimized/2X/4/46a56c1cf761c40b7540ee19bf4d1ebbf055db38_2_690x433.png)](https://discussion.scottibyte.com/uploads/default/original/2X/4/46a56c1cf761c40b7540ee19bf4d1ebbf055db38.png "image")

Follow all of the prompts in the video tutorial while installing Windows. When you see the screen to choose where to install Windows, it will have highlighted the virtual disk we created and you can click Next.

[![image](https://discussion.scottibyte.com/uploads/default/optimized/2X/1/112050c9f48112a832ec9d5cebe0a3a7349a3df3_2_690x431.png)](https://discussion.scottibyte.com/uploads/default/original/2X/1/112050c9f48112a832ec9d5cebe0a3a7349a3df3.png "image")

The Virtual Machine viewer will trap your mouse pointer. If you want to escape the viewer at any time, use Shift +F12. Windows will restart several times during installation. Any time Windows reboots, you will be exited from the remote viewer and be back at the terminal. The command to return to the remote viewer is:

```typescript
incus console win11vm --type=vga
```

Eventually you will come to a screen entitled “Let’s connect you to a network”. Since Windows does not have VirtIO drivers, it does not know how to get to the network. On this screen, click “Install driver”.

[![image](https://discussion.scottibyte.com/uploads/default/optimized/2X/0/0284aff75924f6805551bc15cf36dc1e9a78198f_2_690x431.jpeg)](https://discussion.scottibyte.com/uploads/default/original/2X/0/0284aff75924f6805551bc15cf36dc1e9a78198f.jpeg "image")

Select “Drive D” because it is the CD ROM with the Redhat VirtIO drivers we attached to the Incus virtual machine before we started it. Then click select folder from the top of the drive.

[![image](https://discussion.scottibyte.com/uploads/default/optimized/2X/2/2b13bb62051480d7215a5243137fe5e829d69a81_2_690x429.jpeg)](https://discussion.scottibyte.com/uploads/default/original/2X/2/2b13bb62051480d7215a5243137fe5e829d69a81.jpeg "image")

Your network device will be connected.

[![image](https://discussion.scottibyte.com/uploads/default/optimized/2X/3/3a8d3cd44830f74153fd080f8fd0a858f03e47b7_2_690x429.jpeg)](https://discussion.scottibyte.com/uploads/default/original/2X/3/3a8d3cd44830f74153fd080f8fd0a858f03e47b7.jpeg "image")

As the installation continues, you will get to a screen to name your device.

[![image](https://discussion.scottibyte.com/uploads/default/optimized/2X/5/5f466f64891051bfd7344170b065c6e00139900e_2_690x429.jpeg)](https://discussion.scottibyte.com/uploads/default/original/2X/5/5f466f64891051bfd7344170b065c6e00139900e.jpeg "image")

If you are like me and just want a local account rather than signing on to a Microsoft account, select the option to “Set up for work or school” on the screen below.

[![image](https://discussion.scottibyte.com/uploads/default/optimized/2X/9/9581777b08b02952d9b702444556b2cf332ab2dc_2_690x430.jpeg)](https://discussion.scottibyte.com/uploads/default/original/2X/9/9581777b08b02952d9b702444556b2cf332ab2dc.jpeg "image")

On the screen below, click on “Sign in options”.

[![image](https://discussion.scottibyte.com/uploads/default/optimized/2X/8/846f24e3b8b4e1b38c81a8e5a5450fae924e29cb_2_690x430.jpeg)](https://discussion.scottibyte.com/uploads/default/original/2X/8/846f24e3b8b4e1b38c81a8e5a5450fae924e29cb.jpeg "image")

Click “Domain join instead” on the screen below.

[![image](https://discussion.scottibyte.com/uploads/default/optimized/2X/c/c0ada30eda5bbda3a1275c504024de787da7b30e_2_690x430.jpeg)](https://discussion.scottibyte.com/uploads/default/original/2X/c/c0ada30eda5bbda3a1275c504024de787da7b30e.jpeg "image")

You will then be able to type in a plain local username & password.

[![image](https://discussion.scottibyte.com/uploads/default/optimized/2X/1/1163bc2967fe6879e293b4cbbfd4fb49615651f6_2_690x435.jpeg)](https://discussion.scottibyte.com/uploads/default/original/2X/1/1163bc2967fe6879e293b4cbbfd4fb49615651f6.jpeg "image")

Follow the prompts for the password, password verification and security questions. Answer the privacy questions (but it’s Windows – does it really matter?).

The update download and patch installation process will continue for a long time independently now.

[![image](https://discussion.scottibyte.com/uploads/default/optimized/2X/2/248665c7b0b6f6e4550039f8c96a52c2d2781c64_2_690x431.jpeg)](https://discussion.scottibyte.com/uploads/default/original/2X/2/248665c7b0b6f6e4550039f8c96a52c2d2781c64.jpeg "image")

So, how many times does Windows 11 actually restart during an install. The evidence is below.

[![image](https://discussion.scottibyte.com/uploads/default/optimized/2X/c/c35176765dfe3ca18386920c48360dc11ead0eb7_2_690x231.png)](https://discussion.scottibyte.com/uploads/default/original/2X/c/c35176765dfe3ca18386920c48360dc11ead0eb7.png "image")

After logging in following the installation, I had to adjust my screen resolution. Other than that, everything looked good.

Finally, I shutdown Windows from the start menu. Once Windows is down, it returned to the terminal where I disconnected the CD ROM drives used for the Windows ISO and the VirtIO ISO images with the following commands.

```lua
incus config device remove win11vm install
incus config device remove win11vm virtio
```

[![image](https://discussion.scottibyte.com/uploads/default/original/2X/4/47e62199ff59c6c0053db837903e407ba54aa837.png)](https://discussion.scottibyte.com/uploads/default/original/2X/4/47e62199ff59c6c0053db837903e407ba54aa837.png "image")

The new USB bus CD ROM drivers that are a part of Incus 6.16 have really helped to massively simplify the installation of a Windows 11 virtual machine in Incus.

  

[Powered by Discourse](https://discourse.org/powered-by)