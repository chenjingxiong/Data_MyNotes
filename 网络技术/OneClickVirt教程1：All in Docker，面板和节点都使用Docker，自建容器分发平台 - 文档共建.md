---
title: "OneClickVirt教程1：All in Docker，面板和节点都使用Docker，自建容器分发平台"
source: "https://linux.do/t/topic/1058066"
author:
  - "[[LINUX DO]]"
  - "ECS"
published: 2025-11-03
created: 2026-02-08
description: "完整的OneClickVirt部署教程，从0到1教授如何使用Docker部署面板和节点"
tags:
  - "clippings"
  - "Docker"
  - "虚拟化"
  - "教程"
---

## 由 ECS 发布于 2025 年 11 月 3 日

**开源项目地址**：[GitHub - oneclickvirt/oneclickvirt](https://github.com/oneclickvirt/oneclickvirt) - 可扩展的通用虚拟化管理平台，支持 LXD / Incus / Docker / Proxmox VE

> 之前的教程没有从0到1完整教授如何部署和纳管节点进行操作，这里重新写一篇完整的操作教程。
>
> 用到的服务器只有一台：[https://bandwagonhost.com/](https://bandwagonhost.com/)

---

## 准备工作

### 重装系统

点击齿轮进入 KiwiVM Admin Controls，需要重装一下系统，确保环境干净。

[![image](https://linux.do/uploads/default/original/4X/b/1/f/b1f064668c99ecda8832a735e52520848d4925b0.png)](https://linux.do/uploads/default/original/4X/b/1/f/b1f064668c99ecda8832a735e52520848d4925b0.png "image")

[![image](https://linux.do/uploads/default/optimized/4X/1/4/e/14e3cea1400644f812329f2eb5fd6e0d409dd29b_2_690x189.png)](https://linux.do/uploads/default/original/4X/1/4/e/14e3cea1400644f812329f2eb5fd6e0d409dd29b.png "image")

**注意**：重装前请确保 Stop 掉服务器，否则无法直接进行重装。

[![image](https://linux.do/uploads/default/optimized/4X/9/0/0/9001a7c19b3a9e45dfc3079c7b3be73f49369118_2_690x440.png)](https://linux.do/uploads/default/original/4X/9/0/0/9001a7c19b3a9e45dfc3079c7b3be73f49369118.png "image")

[![image](https://linux.do/uploads/default/optimized/4X/c/a/3/ca3fa6fc546b96527ce5016bf3e89bfb1ec1e7e8_2_576x500.png)](https://linux.do/uploads/default/original/4X/c/a/3/ca3fa6fc546b96527ce5016bf3e89bfb1ec1e7e8.png "image")

装一个 Debian 13，占用足够小：

[![image](https://linux.do/uploads/default/original/4X/7/f/2/7f26ced087943b6767fa0b406b6df187724d9f81.png)](https://linux.do/uploads/default/original/4X/7/f/2/7f26ced087943b6767fa0b406b6df187724d9f81.png "image")

---

## 安装基础工具

记录好 SSH 密码和端口，本地用任意方式连接上服务器，然后执行以下命令安装必备工具：

```bash
apt update
apt install sudo curl wget lsof vnstat -y
```

[![image](https://linux.do/uploads/default/optimized/4X/d/e/c/dec4b9674c627089edfc1f15f3bed9bb19562325_2_690x344.png)](https://linux.do/uploads/default/original/4X/d/e/c/dec4b9674c627089edfc1f15f3bed9bb19562325.png "image")

---

## 安装 Docker 环境

> **重要**：这一步请不要使用其他 docker 安装脚本，仅使用这里提供的安装方式。因为非本教程的 docker 安装方式不支持一键设置存储池进行容器的硬盘大小限制，本教程有特殊适配。

### 1. 开设虚拟内存（可选）

```bash
curl -L https://raw.githubusercontent.com/spiritLHLS/addswap/main/addswap.sh -o addswap.sh && chmod +x addswap.sh && bash addswap.sh
```

[![image](https://linux.do/uploads/default/original/4X/5/f/c/5fc77775fb8651c0a6bb46f83edc8372e71b6733.png)](https://linux.do/uploads/default/original/4X/5/f/c/5fc77775fb8651c0a6bb46f83edc8372e71b6733.png "image")

> 如果你的服务器内存足够，可以不开设虚拟内存。如果内存不够用，建议开一点避免内存爆炸。

### 2. 安装 Docker

```bash
curl -L https://raw.githubusercontent.com/oneclickvirt/docker/main/scripts/dockerinstall.sh -o dockerinstall.sh
chmod +x dockerinstall.sh
bash dockerinstall.sh
```

[![image](https://linux.do/uploads/default/original/4X/e/7/e/e7e2274a576c0f6e0d5a091c958d04d70fdf9014.png)](https://linux.do/uploads/default/original/4X/e/7/e/e7e2274a576c0f6e0d5a091c958d04d70fdf9014.png "image")

### 3. Docker 安装配置

**是否限制硬盘大小**：执行过程中会提示是否需要限制硬盘大小。如果需要限制，输入 `y` 进行回车。

[![image](https://linux.do/uploads/default/original/4X/d/1/2/d12add98e12f24874adbd1248ac51b7ddcde55aa.png)](https://linux.do/uploads/default/original/4X/d/1/2/d12add98e12f24874adbd1248ac51b7ddcde55aa.png "image")

> 如果不进行限制，也能使用 OneClickVirt 平台进行管理，只是无法限制硬盘大小。

**Docker 安装路径**：如果服务器只有一个系统盘，直接回车使用默认路径；如果有多个盘，选择一个较大的盘安装会比较好。

[![image](https://linux.do/uploads/default/original/4X/0/e/6/0e60c3afffd4628a3b6411c5a0eadf405e7bb6f1.png)](https://linux.do/uploads/default/original/4X/0/e/6/0e60c3afffd4628a3b6411c5a0eadf405e7bb6f1.png "image")

**Docker 存储池大小**：输入存储池的大小（GB）。例如服务器有 80G 硬盘，可以选择 60G 作为存储池。

[![image](https://linux.do/uploads/default/original/4X/3/6/e/36e719116f65dea369749e1ffcd99404dfc87c6b.png)](https://linux.do/uploads/default/original/4X/3/6/e/36e719116f65dea369749e1ffcd99404dfc87c6b.png "image")

**循环文件存储位置**：如果使用系统盘，直接回车默认即可。如果是多盘，需要和上面的安装路径保持一致挂载的路径。

[![image](https://linux.do/uploads/default/original/4X/b/5/3/b532770773117f2b97092710e930905135ad6a63.png)](https://linux.do/uploads/default/original/4X/b/5/3/b532770773117f2b97092710e930905135ad6a63.png "image")

安装过程中会显示安装路径和创建的 loop 设备的地址，证明一切正常：

[![image](https://linux.do/uploads/default/optimized/4X/6/0/c/60cd8394f0f9e8fb777881fc778b9b21ad462a80_2_690x436.png)](https://linux.do/uploads/default/original/4X/6/0/c/60cd8394f0f9e8fb777881fc778b9b21ad462a80.png "image")

### 4. 需要在被控端安装好LXD，否则会导致API部署不成功！

```bash
# 安装 LXD 服务器和客户端
sudo apt update
sudo apt install -y lxd lxd-client

# 安装 unzip 解压包
apt-get install unzip

#如果不能下载镜像，可以尝试打开或者关闭系统镜像的配置 ： 是否启动cdn
```

### 5. 重启服务器

```bash
reboot
```

重启后重新登录，受控端的环境就算安装完毕，可以开始安装控制面板。

---

## 安装控制面板

按照说明的提示直接安装面板端。由于暂时不打算绑定域名，这里使用最简单的一体化安装版本（无域名版本）：

```bash
docker run -d \
  --name oneclickvirt \
  -p 8099:80 \
  -v oneclickvirt-data:/var/lib/mysql \
  -v oneclickvirt-storage:/app/storage \
  --restart unless-stopped \
  spiritlhl/oneclickvirt:latest
```

[![image](https://linux.do/uploads/default/original/4X/1/3/2/13229a63352c7c6420c907a823ea1c343f842890.png)](https://linux.do/uploads/default/original/4X/1/3/2/13229a63352c7c6420c907a823ea1c343f842890.png "image")

平台容器启动完毕后，执行：

```bash
docker ps -a
```

可见启动成功，且等待超过了 12 秒，数据库已延迟启动成功：

[![image](https://linux.do/uploads/default/original/4X/6/9/3/69346011fba9397ea6fe7ae8edcae6f7bf134c5f.png)](https://linux.do/uploads/default/original/4X/6/9/3/69346011fba9397ea6fe7ae8edcae6f7bf134c5f.png "image")

---

## 初始化系统

直接打开当前服务器的 IP 地址，可见自动跳转到了初始化界面：

[![image](https://linux.do/uploads/default/optimized/4X/4/3/0/430945b41f4e249f308c35e89034e545f7f14839_2_360x500.png)](https://linux.do/uploads/default/original/4X/4/3/0/430945b41f4e249f308c35e89034e545f7f14839.png "image")

[![image](https://linux.do/uploads/default/optimized/4X/a/6/e/a6eb7cb5b033a0b04ef57fe2a5d944aae849eb57_2_465x500.png)](https://linux.do/uploads/default/original/4X/a/6/e/a6eb7cb5b033a0b04ef57fe2a5d944aae849eb57.png "image")

1. 点击 **测试数据库连接**，确认没问题
2. 点击 **一键填入默认信息**，确认没问题
3. 点击 **初始化系统** 进行初始化

> 耐心等待 5~10 秒，不要重复点击初始化按钮，将自动跳转到首页：

[![image](https://linux.do/uploads/default/original/4X/c/e/0/ce0967905a14a8ea8d29e6132a14e6784e4f5c84.png)](https://linux.do/uploads/default/original/4X/c/e/0/ce0967905a14a8ea8d29e6132a14e6784e4f5c84.png "image")

---

## 登录系统

点击 **账户登录**，进入登录页面：

[![image](https://linux.do/uploads/default/optimized/4X/4/6/5/465b66f0f8d001207583e20e5b3b814919c53a33_2_414x500.png)](https://linux.do/uploads/default/original/4X/4/6/5/465b66f0f8d001207583e20e5b3b814919c53a33.png "image")

点击 **管理员登录**，进入管理员登录界面。

**管理员账户名密码分别为**：

```
用户名: admin
密码: Admin123!@#
```

[![image](https://linux.do/uploads/default/original/4X/0/7/f/07f25e30e46b227b977634c4101956d98c2b756a.png)](https://linux.do/uploads/default/original/4X/0/7/f/07f25e30e46b227b977634c4101956d98c2b756a.png "image")

[![image](https://linux.do/uploads/default/original/4X/f/c/a/fcaddeab8cf99b49c05d149ab9e7496526676364.png)](https://linux.do/uploads/default/original/4X/f/c/a/fcaddeab8cf99b49c05d149ab9e7496526676364.png "image")

---

## 安全配置

### 重置管理员密码

左侧栏点击进入 **用户管理界面**，重置管理员密码，避免被爆破：

[![image](https://linux.do/uploads/default/optimized/4X/9/6/8/9686c19903679fa55d5af231bb2747b82975745a_2_690x209.png)](https://linux.do/uploads/default/original/4X/9/6/8/9686c19903679fa55d5af231bb2747b82975745a.png "image")

[![image](https://linux.do/uploads/default/original/4X/a/4/2/a42b6e02c456076678cc185baa1abbde503d2e41.png)](https://linux.do/uploads/default/original/4X/a/4/2/a42b6e02c456076678cc185baa1abbde503d2e41.png "image")

记录并保存好密码：

[![image](https://linux.do/uploads/default/optimized/4X/5/f/6/5f6372f967c364d1c61e7aa2f6b2d5ee247fb09f_2_690x193.png)](https://linux.do/uploads/default/original/4X/5/f/6/5f6372f967c364d1c61e7aa2f6b2d5ee247fb09f.png "image")

### 禁用测试用户

禁用测试用户，避免滥用。

### 系统配置

左侧栏点击 **系统配置**，配置：
- 公开注册
- 用户等级限制
- 实例权限限制

---

## 纳管节点

配置录入完毕后，可见：

[![image](https://linux.do/uploads/default/optimized/4X/8/b/9/8b917f14f79c6b31ddcb82199e7246443a0ef2e3_2_690x150.png)](https://linux.do/uploads/default/original/4X/8/b/9/8b917f14f79c6b31ddcb82199e7246443a0ef2e3.png "image")

正常可用了。

---

## 测试创建实例

右上角点击头像，切换为 **普通用户视图** 进行测试：

[![image](https://linux.do/uploads/default/optimized/4X/7/3/3/7330cf7b822c6d8a7a430491e407cbb1b9e54ae2_2_690x332.png)](https://linux.do/uploads/default/original/4X/7/3/3/7330cf7b822c6d8a7a430491e407cbb1b9e54ae2.png "image")

左侧点击 **申请领取** 跳转：

[![image](https://linux.do/uploads/default/original/4X/7/8/1/78156ed0c126affc536e587136015b52064ff58c.png)](https://linux.do/uploads/default/original/4X/7/8/1/78156ed0c126affc536e587136015b52064ff58c.png "image")

选择配置和系统测试提交申请：

[![image](https://linux.do/uploads/default/optimized/4X/c/5/6/c56896eb7971d06802ca885cf148b322fb1e2289_2_690x336.png)](https://linux.do/uploads/default/original/4X/c/5/6/c56896eb7971d06802ca885cf148b322fb1e2289.png "image")

自动跳转任务列表，刷新可见任务执行中：

[![image](https://linux.do/uploads/default/original/4X/5/1/a/51a14a5ac34827dfe0d508aed7c23e40e6f3deb1.png)](https://linux.do/uploads/default/original/4X/5/1/a/51a14a5ac34827dfe0d508aed7c23e40e6f3deb1.png "image")

执行完毕后可见：

[![image](https://linux.do/uploads/default/optimized/4X/b/c/3/bc39a168fe17cca25963a6edc8b156beeef826b5_2_690x263.png)](https://linux.do/uploads/default/original/4X/b/c/3/bc39a168fe17cca25963a6edc8b156beeef826b5.png "image")

---

## 查看实例

点击左侧栏 **我的实例**：

[![image](https://linux.do/uploads/default/optimized/4X/2/a/f/2afa935ab1d191b35144c51c1c49de3657af6f88_2_470x500.png)](https://linux.do/uploads/default/original/4X/2/a/f/2afa935ab1d191b35144c51c1c49de3657af6f88.png "image")

可见对应的实例，点击进入查看详情：

[![image](https://linux.do/uploads/default/original/4X/3/b/2/3b28f356258eae8274f804e38bc5364fb777882f.png)](https://linux.do/uploads/default/original/4X/3/b/2/3b28f356258eae8274f804e38bc5364fb777882f.png "image")

可见相关连接信息，本地测试连接：

[![image](https://linux.do/uploads/default/original/4X/f/4/2/f42c02148529fc4a84a79ec1787eb73d2f62f9fa.png)](https://linux.do/uploads/default/original/4X/f/4/2/f42c02148529fc4a84a79ec1787eb73d2f62f9fa.png "image")

NAT 映射没问题，可登录且有网络。现在可以删除测试实例并丢给用户申请使用了：

[![image](https://linux.do/uploads/default/original/4X/c/0/8/c08c6eb6ad0aae34f0e7ac4a22e16a01f8de270a.png)](https://linux.do/uploads/default/original/4X/c/0/8/c08c6eb6ad0aae34f0e7ac4a22e16a01f8de270a.png "image")

---

## 相关链接

- [继续测试容器分发平台，送点免费的NAT容器，有效期到10月31日](https://linux.do/t/topic/1060037)
