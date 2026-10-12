---
title: "构建自己的 LXC 镜像模板"
source: "https://josephcz.xyz/technology/linux/build-customized-lxc-template/"
author:
  - "[[防止 Nginx 的未授权解析和 SSL 源站泄露]]"
published: 2024-09-14
created: 2025-09-17
description: "在 Proxmox VE 中，除了虚拟机之外，也有基于 LXC 容器技术的虚拟化方案。Proxmox VE 提供了不少官方镜像，但是也可以根据自己的喜好定制镜像模板，以便更加快速部署，或是满足个人需求。 本文以 Debian 为例，为了创建 LXC 镜像，您需要有一台拥有 root 权限且安装 Debian 操作系统的机器，可以是虚拟机也可以是物理机，服务器版与桌面版均可，但不能是在容器中运行的"
tags:
  - "clippings"
---
在 [Proxmox VE](https://www.proxmox.com/) 中，除了虚拟机之外，也有基于 LXC 容器技术的虚拟化方案。Proxmox VE 提供了不少官方镜像，但是也可以根据自己的喜好定制镜像模板，以便更加快速部署，或是满足个人需求。

本文以 Debian 为例，为了创建 LXC 镜像，您需要有一台拥有 root 权限且安装 Debian 操作系统的机器，可以是虚拟机也可以是物理机，服务器版与桌面版均可，但不能是在容器中运行的 Debian。

> **注意**
> 
> 以下所有命令以 root 权限运行。如果您不是 root 用户，请使用 `sudo -i` 切换到 root 用户。

## 安装依赖

我们需要 debootstrap 来创建根文件系统，以及 zstd 来压缩镜像。

```bash
apt update && apt install debootstrap zstd
```

## 创建根文件系统

LXC 容器模板的本质是将 Linux 根文件系统 (`/` 路径) 打包成一个 tar 文件并压缩，因此我们只需还原这个过程即可。

对于此，Debian 提供了一个叫做 debootstrap 的工具，可以帮助我们在指定目录下创建一个 Debian 根文件系统。其命令的格式为： `debootstrap <release> <target path> <mirror url>` 。

```bash
debootstrap bookworm ./rootfs http://deb.debian.org/debian/
```

上面的命令已经创建了一个最简单的 Debian 根文件系统，如果不需要自定义的话，可以直接将该目录打包、压缩、上传到 Proxmox VE 中，作为镜像使用。

我使用在我网络条件下最快的镜像，同时根据个人习惯安装了自己需要的一些软件包，并启用了 [/usr merge](https://wiki.debian.org/UsrMerge) 。因此我使用了下面的命令：

```bash
export MIRROR=http://deb.debian.org/debian/

export RELEASE=bookworm

export DIST=/srv/debian-$RELEASE

export CACHE=/srv/debian-bootstrap-cache

export PACKAGES=jq,git,zip,7zip,unzip,bash-completion,\

vim,sudo,curl,wget,tree,tmux,dnsutils,iftop,iotop,btop,\

uuid-runtime,openssh-server,locales,locales-all,\

python3-full,python3-venv,python3-pip,python-is-python3

cd /srv

debootstrap --cache-dir=$CACHE \

    --merged-usr --include=$PACKAGES \

    $RELEASE $DIST $MIRROR
```

## 针对 LXC 的一些配置

在标准的 Debian 系统之外，针对 LXC 环境可以做一些特殊的配置。

### 登录前显示 IP 地址

由于我的 Proxmox VE 启用了 DHCP，在登录前显示 IP 地址可以方便 SSH 登录。登录前 Banner 保存在 `/etc/issue` 文件中，向其中写入想要添加的内容即可，其中 `\4` 是 IPv4 地址的转义字符， `\6` 是 IPv6 地址的转义字符。

```bash
echo "IPv4 address is: \\4" >> $DIST/etc/issue
```

这样在登录前会显示类似下面的内容：

```
Debian GNU/Linux 12 lxc-server-1 tty1

IPv4 address is: 10.0.0.102

lxc-server-1 login:
```

### 禁用 Systemd 内核挂载

Systemd 默认有一些内核相关的服务，但是 LXC 容器并没有完整内核，会产生一些警告，虽然不影响系统运行，但是禁用这些服务可以让系统保持干净。

```bash
ln -s /dev/null $DIST/etc/systemd/system/sys-kernel-config.mount

ln -s /dev/null $DIST/etc/systemd/system/sys-kernel-debug.mount
```

### 配置 SSH 密钥重新生成

Proxmox VE 在创建 CT 时会自动重新生成密钥 (Host Key)，但是如果是将已经创建好的 CT 作为模板进行克隆，那么不会重新生成密钥。因此我们配置一个 Systemd 服务，在启动 SSH 服务前检查 Host Key 是否存在，如果不存在则重新生成。

```bash
cat <<EOF > $DIST/etc/systemd/system/ssh-keygen.service

[Unit]

Description=Generate SSH host keys

Before=ssh.service

[Service]

Type=oneshot

ExecStart=/usr/bin/ssh-keygen -A

[Install]

WantedBy=multi-user.target

EOF

ln -s /etc/systemd/system/ssh-keygen.service \

$DIST/etc/systemd/system/multi-user.target.wants/ssh-keygen.service
```

## 将根文件系统打包压缩

首先清理缓存和一些历史文件，以减小模板大小：

```bash
chroot $DIST apt clean

rm -vf $DIST/etc/ssh/*key*

rm -vf $DIST/root/.bash_history
```

打包压缩：

```bash
tar -cvf - -C $DIST . | zstd -T0 -19 -o \

/srv/debian-$RELEASE-build-$(date +%Y%m%d%H%M%S).tar.zst
```

然后将打包出来的 `debian-<release>-build-<datetime>.tar.zst` 文件上传到 Proxmox VE 的 `/var/lib/vz/template/cache/` 目录下，即可在创建 LXC 容器时选择该镜像。

## 容器克隆前的准备

如果需要克隆 LXC 容器，建议先清理一些文件：

```bash
rm -rf /etc/machine-id

rm -rf /etc/ssh/*key*
```

退出前使用 root 用户登录，然后清理 bash 历史记录：

```bash
rm -rf $HISTFILE && history -c && poweroff
```

然后点击 Proxmox VE 的 Web 界面中的 **「转换为模板」** 将该 CT 转换为模板，以便后续使用。