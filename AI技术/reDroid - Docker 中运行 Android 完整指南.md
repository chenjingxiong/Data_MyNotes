# reDroid — Docker 中运行 Android 完整指南

> **reDroid**（**Re**mote an**Droid**）是一个 GPU 加速的 AIC（Android In Cloud）解决方案，可以在 Linux 主机中通过 Docker / Podman / K8s 启动多个 Android 实例。支持 `arm64` 和 `amd64` 架构，适用于云游戏、虚拟手机、自动化测试等场景。

---

## 目录

- [项目概述](#项目概述)
- [支持的 Android 版本](#支持的-android-版本)
- [核心原理与架构](#核心原理与架构)
- [环境要求](#环境要求)
- [快速开始](#快速开始)
- [配置参数详解](#配置参数详解)
- [GPU 加速模式](#gpu-加速模式)
- [多平台部署指南](#多平台部署指南)
- [Native Bridge（ARM 应用兼容）](#native-bridgearm-应用兼容)
- [GMS（Google 服务）支持](#gmsgoogle-服务支持)
- [WebRTC 远程串流](#webrtc-远程串流)
- [多实例管理](#多实例管理)
- [常见问题排查](#常见问题排查)
- [安全注意事项](#安全注意事项)
- [参考资源](#参考资源)

---

## 项目概述

| 项目 | 信息 |
|------|------|
| GitHub 仓库 | [remote-android/redroid-doc](https://github.com/remote-android/redroid-doc) |
| Docker 镜像 | `redroid/redroid` |
| 许可证 | Apache 2.0（含第三方模块需另行检查） |
| 联系方式 | [Slack 频道](https://join.slack.com/t/remote-android/shared_invite/zt-387g26jdj-5kyINRegb_9BtYALktKtbA) |

### 典型使用场景

1. **云游戏** — 利用 GPU 加速在云端运行 Android 游戏
2. **虚拟手机** — 在服务器上创建多个独立 Android 实例
3. **自动化测试** — CI/CD 中批量运行 Android 测试
4. **App 开发调试** — 无需物理设备即可测试 App
5. **多开矩阵** — 批量管理大量 Android 实例

---

## 支持的 Android 版本

| 版本 | 镜像标签 | 64-bit Only 标签 |
|------|----------|-----------------|
| Android 16 | `redroid/redroid:16.0.0-latest` | `redroid/redroid:16.0.0_64only-latest` |
| Android 15 | `redroid/redroid:15.0.0-latest` | `redroid/redroid:15.0.0_64only-latest` |
| Android 14 | `redroid/redroid:14.0.0-latest` | `redroid/redroid:14.0.0_64only-latest` |
| Android 13 | `redroid/redroid:13.0.0-latest` | `redroid/redroid:13.0.0_64only-latest` |
| Android 12 | `redroid/redroid:12.0.0-latest` | `redroid/redroid:12.0.0_64only-latest` |
| Android 11 | `redroid/redroid:11.0.0-latest` | — |
| Android 10 | `redroid/redroid:10.0.0-latest` | — |
| Android 9 | `redroid/redroid:9.0.0-latest` | — |
| Android 8.1 | `redroid/redroid:8.1.0-latest` | — |

> [!tip] 选择建议
> - 优先选择 `64only` 版本，性能更好且兼容大多数现代 App
> - 服务器环境推荐 Android 12+ (`12.0.0_64only-latest` 或更新)
> - 某些 ARM 平台只支持 `aarch64`，必须使用 `64only` 镜像

---

## 核心原理与架构

reDroid 的核心思想是将 AOSP（Android Open Source Project）打包为 Docker 容器，利用 Linux 内核的 Android 特性模块实现容器化运行：

```
┌─────────────────────────────────────────────┐
│               Linux Host                     │
│                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │ reDroid  │  │ reDroid  │  │ reDroid  │   │
│  │ 容器 #1  │  │ 容器 #2  │  │ 容器 #3  │   │
│  │ :5555    │  │ :5556    │  │ :5557    │   │
│  └──────────┘  └──────────┘  └──────────┘   │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │     内核模块: binder + ashmem/memfd  │    │
│  └──────────────────────────────────────┘    │
│                                              │
│  ┌──────────────────────────────────────┐    │
│  │           Docker Engine              │    │
│  └──────────────────────────────────────┘    │
└─────────────────────────────────────────────┘
        │               │              │
    scrcpy/         scrcpy/        scrcpy/
    WebRTC          WebRTC         WebRTC
```

关键内核模块：
- **binderfs** — Android 的 IPC（进程间通信）机制
- **ashmem / memfd** — Android 共享内存
- **ION / DMA-BUF Heaps** — GPU 内存管理

---

## 环境要求

### 硬件要求

| 项目 | 最低要求 | 推荐配置 |
|------|----------|----------|
| CPU | 2 核 | 4 核+ |
| 内存 | 2 GB / 实例 | 4 GB+ / 实例 |
| 存储 | 10 GB | 20 GB+ SSD |
| GPU（可选） | — | 支持 OpenGL/Virgl |

### 软件要求

- **操作系统**：Linux（内核 ≥ 5.0）
- **容器运行时**：Docker 或 Podman
- **内核模块**：`binder_linux`、`ashmem_linux`（5.18+ 可用 `memfd` 替代）

### 必需的内核特性

```
CONFIG_ANDROID=y
CONFIG_ANDROID_BINDER_IPC=y
CONFIG_ANDROID_BINDERFS=y
CONFIG_ANDROID_BINDER_DEVICES="binder,hwbinder,vndbinder"
CONFIG_ASHMEM=y                    # 可选，5.18+ 可用 memfd 替代
CONFIG_DMABUF_HEAPS=y              # GPU 相关
CONFIG_DMABUF_HEAPS_SYSTEM=y
CONFIG_IPV6_ROUTER_PREF=y
CONFIG_IPV6_MULTIPLE_TABLES=y
```

---

## 快速开始

以 **Ubuntu 22.04** 为例，5 分钟启动你的第一个 Android 容器：

### Step 1：安装 Docker

```bash
# 官方安装脚本（推荐）
curl -fsSL https://get.docker.com | sh

# 或者使用系统包管理器
apt update && apt install -y docker.io
systemctl enable --now docker
```

### Step 2：加载内核模块

```bash
# 安装额外的内核模块
apt install -y linux-modules-extra-$(uname -r)

# 加载 binder 模块
modprobe binder_linux devices="binder,hwbinder,vndbinder"

# 加载 ashmem 模块（内核 < 5.18 需要）
modprobe ashmem_linux
```

> [!warning] 持久化内核模块
> 重启后 `modprobe` 会失效。若需开机自动加载，添加到配置文件：
> ```bash
> echo "binder_linux" >> /etc/modules
> echo "ashmem_linux" >> /etc/modules
> ```

### Step 3：启动 reDroid 容器

```bash
docker run -itd --rm --privileged \
    --pull always \
    -v ~/data:/data \
    -p 5555:5555 \
    --name redroid12 \
    redroid/redroid:12.0.0_64only-latest
```

参数说明：

| 参数 | 说明 |
|------|------|
| `-itd` | 交互模式 + 后台运行 |
| `--rm` | 容器停止时自动删除（数据保留在挂载卷） |
| `--privileged` | 特权模式（访问内核设备） |
| `--pull always` | 始终拉取最新镜像 |
| `-v ~/data:/data` | 挂载数据分区（持久化 App 数据） |
| `-p 5555:5555` | 暴露 ADB 端口 |

### Step 4：连接与控制

```bash
# 安装 ADB（如未安装）
# Ubuntu/Debian:
apt install -y adb
# 或从 Android SDK 下载

# 通过 ADB 连接
adb connect localhost:5555

# 验证连接
adb devices
```

### Step 5：查看屏幕（scrcpy）

```bash
# 安装 scrcpy
# Ubuntu/Debian:
apt install -y scrcpy
# 或参考 https://github.com/Genymobile/scrcpy

# 投射屏幕
scrcpy -s localhost:5555
```

> [!note] 远程访问
> 如果 reDroid 运行在远程服务器上，将 `localhost` 替换为服务器 IP 地址：
> ```bash
> adb connect 192.168.1.100:5555
> scrcpy -s 192.168.1.100:5555
> ```

---

## 配置参数详解

通过 Docker 启动参数自定义 Android 实例：

```bash
docker run -itd --rm --privileged \
    --pull always \
    -v ~/data:/data \
    -p 5555:5555 \
    redroid/redroid:12.0.0_64only-latest \
    androidboot.redroid_width=1080 \
    androidboot.redroid_height=1920 \
    androidboot.redroid_dpi=480 \
    androidboot.redroid_fps=60 \
    androidboot.redroid_gpu_mode=host
```

### 完整参数列表

| 参数                                  | 说明                                                                    | 默认值                           |
| ----------------------------------- | --------------------------------------------------------------------- | ----------------------------- |
| `androidboot.redroid_width`         | 显示宽度（像素）                                                              | `720`                         |
| `androidboot.redroid_height`        | 显示高度（像素）                                                              | `1280`                        |
| `androidboot.redroid_fps`           | 显示帧率                                                                  | GPU 启用: `30`<br>GPU 未启用: `15` |
| `androidboot.redroid_dpi`           | 屏幕密度                                                                  | `320`                         |
| `androidboot.use_memfd`             | 使用 `memfd` 替代已弃用的 `ashmem`                                            | `false`                       |
| `androidboot.use_redroid_overlayfs` | 使用 `overlayfs` 共享 data 分区<br>`/data-base`: 共享数据<br>`/data-diff`: 私有数据 | `0`                           |
| `androidboot.redroid_gpu_mode`      | GPU 模式：`auto` / `host` / `guest`                                      | `guest`                       |
| `androidboot.redroid_gpu_node`      | GPU 设备节点                                                              | 自动检测                          |

### 网络配置参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `androidboot.redroid_net_ndns` | DNS 服务器数量（未指定时使用 `8.8.8.8`） | `0` |
| `androidboot.redroid_net_dns1..N` | 自定义 DNS 地址 | — |
| `androidboot.redroid_net_proxy_type` | 代理类型：`static` / `pac` / `none` / `unassigned` | — |
| `androidboot.redroid_net_proxy_host` | 代理主机 | — |
| `androidboot.redroid_net_proxy_port` | 代理端口 | `3128` |
| `androidboot.redroid_net_proxy_exclude_list` | 代理排除列表（逗号分隔） | — |
| `androidboot.redroid_net_proxy_pac` | PAC 代理 URL | — |

### 常用显示预设

```bash
# 手机竖屏 (1080p)
androidboot.redroid_width=1080 androidboot.redroid_height=1920 androidboot.redroid_dpi=480

# 平板横屏
androidboot.redroid_width=1920 androidboot.redroid_height=1200 androidboot.redroid_dpi=240

# 低配节省资源
androidboot.redroid_width=540 androidboot.redroid_height=960 androidboot.redroid_dpi=160 androidboot.redroid_fps=15
```

---

## GPU 加速模式

GPU 模式通过 `androidboot.redroid_gpu_mode` 参数控制：

### 模式对比

| 模式 | 说明 | 性能 | 适用场景 |
|------|------|------|----------|
| `guest` | 软件渲染（默认） | ★☆☆ | 无 GPU / 简单测试 |
| `host` | 使用宿主机 GPU | ★★★ | 云游戏 / 高性能需求 |
| `auto` | 自动检测 | — | 不确定环境时使用 |
| `g virgl` | Virgl 虚拟 GPU | ★★☆ | GPU 直通不可用时的折中 |

### 启用 GPU 加速（host 模式）

```bash
# 确认宿主机有 GPU
lspci | grep -i vga

# 使用 host GPU 模式启动
docker run -itd --rm --privileged \
    -v ~/data:/data \
    -p 5555:5555 \
    redroid/redroid:12.0.0_64only-latest \
    androidboot.redroid_gpu_mode=host
```

> [!note] NVIDIA GPU
> 如果使用 NVIDIA GPU，可能需要安装 [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) 并添加 `--gpus all` 参数。

---

## 多平台部署指南

### Ubuntu

```bash
# Ubuntu 22.04 / 20.04
apt install -y linux-modules-extra-$(uname -r)
modprobe binder_linux devices="binder,hwbinder,vndbinder"
modprobe ashmem_linux   # 内核 < 5.18 需要

# 直接运行
docker run -itd --rm --privileged \
    --pull always \
    -v ~/data:/data \
    -p 5555:5555 \
    --name redroid \
    redroid/redroid:12.0.0_64only-latest

# Ubuntu 18.04 需要先升级内核
apt install linux-generic-hwe-18.04
modprobe nfnetlink
```

### Debian 11/12

```bash
# 手动创建 binder 设备
modprobe binder_linux devices=binder1,binder2,binder3,binder4,binder5,binder6
chmod 666 /dev/binder*

# 映射 binder 设备启动
docker run -itd --rm --privileged \
    --pull always \
    -v /dev/binder1:/dev/binder \
    -v /dev/binder2:/dev/hwbinder \
    -v /dev/binder3:/dev/vndbinder \
    -v ~/data:/data \
    -p 5555:5555 \
    --name redroid \
    redroid/redroid:12.0.0_64only-latest
```

> [!tip] 无限容器
> 在内核中启用 `binderfs` 即可运行不限数量的 reDroid 容器，无需手动挂载 binder 设备。

### WSL2（Windows Subsystem for Linux）

WSL2 需要自定义内核以启用必需特性：

```bash
# 1. 下载 WSL 内核源码
# https://github.com/microsoft/WSL2-Linux-Kernel/tags
tar xf linux-msft-wsl-*.tar.gz
cd WSL2-Linux-Kernel-linux-msft-wsl-*
cp Microsoft/config-wsl .config

# 2. 启用以下内核配置
# CONFIG_ANDROID=y
# CONFIG_ANDROID_BINDER_IPC=y
# CONFIG_ANDROID_BINDERFS=y
# CONFIG_ANDROID_BINDER_DEVICES="binder,hwbinder,vndbinder"
# CONFIG_ASHMEM=y
# CONFIG_DMABUF_HEAPS=y
# CONFIG_DMABUF_HEAPS_SYSTEM=y

# 3. 编译内核
apt install -y make gcc flex bison libssl-dev libelf-dev dwarves
make

# 4. 内核位于 arch/x86_64/boot/bzImage

# 5. 配置 WSL 使用新内核
# 编辑 %USERPROFILE%\.wslconfig：
# [wsl2]
# kernel=C:\\path\\to\\bzImage

# 6. 重启 WSL 并运行
docker run -d --rm --privileged \
    -v ~/redroid-data:/data \
    -p 5555:5555 \
    --name redroid \
    redroid/redroid:12.0.0_64only-latest
```

### Kubernetes

```bash
# 使用 kustomize 部署
kubectl apply -k https://github.com/remote-android/redroid-doc/deploy/kubernetes/overlays/<VERSION>
```

> 确保集群节点已启用所需的内核特性。

---

## Native Bridge（ARM 应用兼容）

在 `x86` 架构的 reDroid 实例中运行 ARM 应用，需要 Native Bridge 支持（`libhoudini`、`libndk_translation` 或 QEMU translator）。

官方镜像已内置 `libndk_translation`。

### 自定义镜像

```dockerfile
# Dockerfile
FROM redroid/redroid:11.0.0-latest

ADD native-bridge.tar /
```

```bash
# 构建并运行
docker build . -t redroid:11.0.0-nb

docker run -itd --rm --privileged \
    -v ~/data-nb:/data \
    -p 5555:5555 \
    redroid:11.0.0-nb
```

---

## GMS（Google 服务）支持

可通过以下方案添加 Google 移动服务：

- [Open GApps](https://opengapps.org/)
- [MicroG](https://microg.org/)
- [MindTheGapps](https://gitlab.com/MindTheGapps/vendor_gapps)

详细构建步骤参考 [android-builder-docker](https://github.com/remote-android/redroid-doc/tree/master/android-builder-docker)。

---

## WebRTC 远程串流

reDroid 计划从 Cuttlefish 移植 WebRTC 解决方案，包括：

- HTML5 前端（浏览器直接访问）
- 后端服务
- 虚拟 HAL 层

> 目前 WebRTC 功能仍在开发中。

---

## 多实例管理

reDroid 的核心优势之一是可以轻松运行多个 Android 实例：

```bash
# 实例 1 — Android 12，端口 5555
docker run -itd --rm --privileged \
    -v ~/data-instance1:/data \
    -p 5555:5555 \
    --name redroid-1 \
    redroid/redroid:12.0.0_64only-latest

# 实例 2 — Android 14，端口 5556
docker run -itd --rm --privileged \
    -v ~/data-instance2:/data \
    -p 5556:5555 \
    --name redroid-2 \
    redroid/redroid:14.0.0_64only-latest

# 实例 3 — 低配模式，端口 5557
docker run -itd --rm --privileged \
    -v ~/data-instance3:/data \
    -p 5557:5555 \
    --name redroid-3 \
    redroid/redroid:12.0.0_64only-latest \
    androidboot.redroid_width=540 \
    androidboot.redroid_height=960 \
    androidboot.redroid_fps=15
```

### 使用 overlayfs 共享基础数据

通过 overlayfs 可以让多个实例共享同一个基础数据，同时保持各自差异：

```bash
docker run -itd --rm --privileged \
    -v ~/data-shared:/data \
    -p 5555:5555 \
    redroid/redroid:12.0.0_64only-latest \
    androidboot.use_redroid_overlayfs=1
```

---

## 常见问题排查

### 收集调试信息

```bash
# 自动收集 debug 日志
curl -fsSL https://raw.githubusercontent.com/remote-android/redroid-doc/master/debug.sh | sudo bash -s -- [CONTAINER_NAME]
```

### 容器启动后立即消失

```bash
# 检查内核模块是否加载
lsmod | grep binder
lsmod | grep ashmem

# 查看详细错误日志
dmesg -T
```

**常见原因**：binder / ashmem 内核模块未加载。

### ADB 无法连接（设备离线）

```bash
# 进入容器排查
docker exec -it <container> sh

# 检查 Android 进程
ps -A

# 查看系统日志
logcat

# 如果无法进入容器，查看 dmesg
dmesg -T
```

### GPU 渲染问题

```bash
# 尝试切换为软件渲染
androidboot.redroid_gpu_mode=guest

# 或使用 Virgl
androidboot.redroid_gpu_mode=g virgl
```

### Root 访问

```bash
# 默认已有 root shell，如需 root ADB
docker run -itd --rm --privileged \
    -v ~/data:/data \
    -p 5555:5555 \
    redroid/redroid:12.0.0_64only-latest \
    ro.secure=0
```

---

## 安全注意事项

> [!danger] 切勿暴露 ADB 端口到公网
> **绝不要**将 ADB 端口暴露在公共网络上，否则 reDroid 容器（甚至宿主机）可能会被入侵。

```bash
# ❌ 危险！不要这样做
docker run ... -p 0.0.0.0:5555:5555 ...

# ✅ 仅绑定本地
docker run ... -p 127.0.0.1:5555:5555 ...

# ✅ 通过 SSH 隧道远程访问（在本地机器执行）
ssh -L 5555:localhost:5555 user@remote-server
adb connect localhost:5555
```

其他安全建议：
- 使用 `--security-opt` 限制容器权限
- 定期更新 reDroid 镜像
- 在生产环境中使用非 root 用户
- 配置防火墙规则限制端口访问

---

## 参考资源

| 资源 | 链接 |
|------|------|
| 官方文档 | [GitHub - redroid-doc](https://github.com/remote-android/redroid-doc) |
| 内核模块 | [redroid-modules](https://github.com/remote-android/redroid-modules) |
| scrcpy（投屏工具） | [Genymobile/scrcpy](https://github.com/Genymobile/scrcpy) |
| Native Bridge | [zhouziyang/libndk_translation](https://github.com/zhouziyang/libndk_translation) |
| Slack 社区 | [remote-android Slack](https://join.slack.com/t/remote-android/shared_invite/zt-387g26jdj-5kyINRegb_9BtYALktKtbA) |

---

*最后更新：2026-05-04*
*来源：[remote-android/redroid-doc](https://github.com/remote-android/redroid-doc)*
