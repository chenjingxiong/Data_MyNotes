## 目录

1. [Incus 简介](#1-incus-简介)
2. [系统准备](#2-系统准备)
3. [Incus 安装](#3-incus-安装)
4. [基础配置](#4-基础配置)
5. [存储配置](#5-存储配置)
6. [网络配置](#6-网络配置)
7. [集群配置](#7-集群配置)
8. [虚拟机支持](#8-虚拟机支持)
9. [容器支持](#9-容器支持)
10. [Docker in Incus](#10-docker-in-incus)
11. [跨网络集群配置](#11-跨网络集群配置)
12. [节点初始化脚本](#12-节点初始化脚本)
13. [常用命令参考](#13-常用命令参考)
14. [故障排查](#14-故障排查)

---

## 1. Incus 简介

### 1.1 什么是 Incus

Incus 是 LXD 的社区分支，是一个强大的容器和虚拟机管理器，提供：

- **统一管理**: 同时管理 Linux 容器 (LXC) 和虚拟机 (VM)
- **镜像系统**: 丰富的 Linux 发行版镜像库
- **网络功能**: 桥接、VLAN、OVN 等高级网络
- **存储支持**: ZFS、Btrfs、LVM、Ceph 等后端
- **集群支持**: 分布式多节点集群
- **REST API**: 完整的 API 和命令行工具

### 1.2 Incus vs LXD

| 特性 | Incus | LXD |
|------|-------|-----|
| 开发模式 | 社区驱动 | Canonical (商业) |
| 许可证 | AGPLv3 | 使用限制条款 |
| 功能更新 | 更频繁 | 相对保守 |
| 企业支持 | 社区支持 | Canonical 官方支持 |

### 1.3 适用场景

- **开发环境**: 快速部署各种 Linux 发行版进行测试
- **生产部署**: 容器化应用部署
- **虚拟化**: 替代传统虚拟机方案
- **高可用**: 多节点集群部署
- **微服务**: 容器编排和管理

---

## 2. 系统准备

### 2.1 硬件要求

#### 最小配置（单节点）
- CPU: 2 核心
- 内存: 4GB RAM
- 磁盘: 50GB 可用空间

#### 推荐配置（集群节点）
- CPU: 4+ 核心
- 内存: 16GB+ RAM
- 磁盘: 500GB+ SSD
- 网络: 双网卡（内网 + 外网）

### 2.2 网络规划

#### 典型集群网络架构

```
                    ┌─────────────────────────────────────┐
                    │          Internet                    │
                    └─────────────────┬───────────────────┘
                                      │
                    ┌─────────────────▼───────────────────┐
                    │        外网交换机 (Public)           │
                    │     203.0.113.0/24                  │
                    └─────────────────┬───────────────────┘
                                      │
          ┌───────────────────────────┼───────────────────────────┐
          │                           │                           │
  ┌───────▼────────┐         ┌───────▼────────┐         ┌───────▼────────┐
  │   Node 1       │         │   Node 2       │         │   Node 3       │
  │ eth0: 203.0.113.10     │ eth0: 203.0.113.11     │ eth0: 203.0.113.12
  │ eth1: 10.0.0.10  │         │ eth1: 10.0.0.11  │         │ eth1: 10.0.0.12  │
  └───────┬────────┘         └───────┬────────┘         └───────┬────────┘
          │                           │                           │
          └───────────────────────────┼───────────────────────────┘
                                      │
                    ┌─────────────────▼───────────────────┐
                    │        内网交换机 (Private)          │
                    │     10.0.0.0/24                     │
                    │    用于集群通信和存储同步            │
                    └─────────────────────────────────────┘
```

#### 网络接口规划

| 网卡       | 用途    | 示例 IP        | 说明        |
| -------- | ----- | ------------ | --------- |
| eth0     | 外网/公网 | 203.0.113.10 | 对外服务访问    |
| eth1     | 内网/集群 | 10.0.0.10    | 集群通信，存储同步 |
| incusbr0 | 容器桥接  | 10.10.0.1    | 实例默认网络    |

### 2.3 系统配置

#### 2.3.1 更新系统

```bash
# 更新软件包列表
sudo apt update

# 升级所有软件包
sudo apt upgrade -y

# 安装基础工具
sudo apt install -y curl wget vim git htop tmux ufw  ca-certificates gnupg
```

#### 2.3.2 内核参数优化

创建 `/etc/sysctl.d/99-incus.conf`:

```bash
sudo tee /etc/sysctl.d/99-incus.conf > /dev/null <<EOF
# 网络转发
net.ipv4.ip_forward=1
net.ipv6.conf.forwarding=1

# 桥接网络
net.bridge.bridge-nf-call-arptables=0
net.bridge.bridge-nf-call-ip6tables=0
net.bridge.bridge-nf-call-iptables=0

# 集群相关
net.core.somaxconn=1024
net.ipv4.tcp_max_syn_backlog=2048

# 文件描述符
fs.inotify.max_user_watches=1048576
fs.inotify.max_user_instances=512

# 共享内存 (VM 支持)
kernel.shmmax=68719476736
kernel.shmall=4294967296
EOF

# 应用配置
sudo sysctl --system
```

#### 2.3.3 配置主机名和 hosts

```bash
# 设置主机名（每个节点唯一）
sudo hostnamectl set-hostname incus-node1

# 配置 /etc/hosts
sudo tee -a /etc/hosts <<EOF
10.0.0.10    incus-node1
10.0.0.11    incus-node2
10.0.0.12    incus-node3
EOF
```

#### 2.3.4 时间同步

```bash
# 安装 chrony
sudo apt install -y chrony

# 启动并启用服务
sudo systemctl enable --now chronyd

# 验证同步状态
chronyc sources -v
chronyc tracking
```

### 2.4 防火墙配置

```bash
# 基础规则
sudo ufw default deny incoming
sudo ufw default allow outgoing

# SSH
sudo ufw allow 22/tcp

# Incus 集群通信
sudo ufw allow 8443/tcp comment 'Incus API'
sudo ufw allow 8444/tcp comment 'Incus Cluster'

# 存储同步 (Ceph/其他)
sudo ufw allow 6800:7300/tcp comment 'Ceph OSD'

# 允许内网通信
sudo ufw allow from 10.0.0.0/24

# 启用防火墙
sudo ufw enable

# 查看状态
sudo ufw status verbose
```

---

## 3. Incus 安装

### 3.1 添加官方仓库

```bash
# 添加 Incus 官方 GPG 密钥
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.zabbly.com/key.asc | sudo gpg --dearmor -o /etc/apt/keyrings/zabbly.gpg

# 添加仓库
cat <<EOF | sudo tee /etc/apt/sources.list.d/zabbly-incus.list
deb [signed-by=/etc/apt/keyrings/zabbly.gpg] https://pkgs.zabbly.com/incus/stable/debian bookworm main
#deb [signed-by=/etc/apt/keyrings/zabbly.gpg] https://pkgs.zabbly.com/incus/stable/debian bookworm-rc main
#deb [signed-by=/etc/apt/keyrings/zabbly.gpg] https://pkgs.zabbly.com/incus/stable/debian bookworm-candidates main
EOF

# 更新软件包列表
sudo apt update
```

### 3.2 安装 Incus

```bash
# 安装 incus 和相关工具
sudo apt install -y incus incus-tools incus-client

# 验证安装
incus version
```

**预期输出**:
```
Client version: 6.0.x
Server version: 6.0.x
```

### 3.3 安装额外组件

```bash
# ZFS 支持 (推荐用于生产)
sudo apt install -y zfsutils-linux

# Btrfs 支持
sudo apt install -y btrfs-progs

# LVM 支持
sudo apt install -y lvm2 thin-provisioning-tools

# QEMU (虚拟机支持)
sudo apt install -y qemu-utils ovmf

# Ceph (分布式存储，可选)
sudo apt install -y ceph-common
```

---

## 4. 基础配置

### 4.1 初始化配置

使用 `incus admin init` 进行交互式初始化，或使用以下自动化配置：

```bash
# 完全自动初始化（使用默认值）
sudo incus admin init --auto

# 或指定配置初始化
cat <<EOF | sudo incus admin init preseed
config:
  core.https_address: '[::]:8443'
  core.trust_password: "your_cluster_password"
  cluster.https_address: 10.0.0.10:8443
  images.auto_update_interval: "6"
storage_pools:
- name: local
  driver: zfs
  config:
    source: /var/lib/incus/storage-pools/local
networks:
- name: incusbr0
  type: bridge
  config:
    ipv4.address: 10.10.0.1/24
    ipv4.nat: "true"
    ipv6.address: none
profiles:
- name: default
  devices:
    root:
      path: /
      pool: local
      type: disk
    eth0:
      nictype: bridged
      parent: incusbr0
      type: nic
EOF
```

### 4.2 配置选项说明

```bash
# 查看当前配置
incus config show

# 设置核心选项
incus config set core.https_address '[::]:8443'
incus config set core.trust_password 'secure_password'
incus config set images.auto_update_interval 6  # 小时

# 开启 metrics
incus config set core.metrics_address '0.0.0.0:9100'

# 设置远程镜像服务器
incus remote add images https://images.linuxcontainers.org --protocol simplestreams
incus remote add images-linuxcontainers https://linuxcontainers.org --protocol simplestreams
```

### 4.3 用户权限配置

```bash
# 将用户添加到 incus-admin 组
sudo usermod -aG incus-admin $USER

# 重新登录以生效
newgrp incus-admin

# 验证
incus config trust list
```

---

## 5. 存储配置

### 5.1 存储驱动选择

| 驱动 | 优点 | 缺点 | 推荐场景 |
|------|------|------|----------|
| **ZFS** | 快照、压缩、重复数据删除、稳定性 | 内存占用较高 | 生产环境首选 |
| **Btrfs** | 快照、子卷、CoW | 某些操作较慢 | 开发/测试环境 |
| **LVM** | 成熟稳定、性能好 | 快照占用大 | 传统虚拟化场景 |
| **Ceph** | 分布式、高可用 | 配置复杂 | 大规模集群 |
| **Dir** | 简单、无依赖 | 无快照 | 测试/演示 |

### 5.2 ZFS 存储池配置（推荐）

#### 5.2.1 创建 ZFS 池

```bash
# 方案1: 使用现有磁盘（全盘）
sudo zpool create -f -o ashift=12 -O compression=lz4 \
    -O atime=off -O relatime=on \
    incus-pool /dev/disk/by-id/ata-DISK_ID

# 方案2: 使用分区
sudo zpool create -f -o ashift=12 -O compression=lz4 \
    incus-pool /dev/disk/by-id/ata-DISK_ID-part3

# 方案3: 使用文件（测试用）
sudo truncate -s 100G /var/lib/incus/disks/incus-pool.img
sudo zpool create -f -m none -O compression=lz4 \
    incus-pool /var/lib/incus/disks/incus-pool.img

# 验证
sudo zpool status
sudo zfs list
```

#### 5.2.2 在 Incus 中使用 ZFS

```bash
# 创建 ZFS 存储池
incus storage create local zfs source=incus-pool/incus

# 设置为默认
incus profile device add default root disk path=/ pool=local

# 启用 ZFS 优化
incus storage set local volume.zfs.block_mode true
incus storage set local volume.zfs.remove_snapshots true
incus storage set local zfs.clone_copy true
```

### 5.3 Btrfs 存储池配置

```bash
# 准备分区
sudo mkfs.btrfs /dev/sdX

# 创建存储池
incus storage create local btrfs source=/dev/sdX

# 启用压缩
incus storage set local btrfs.mount_options "compress=lzo"
```

### 5.4 Ceph 集成存储（分布式）

```bash
# 创建 Ceph RBD 存储池
incus storage create ceph-pool ceph \
    ceph.cluster.name=ceph \
    ceph.user.name=admin \
    ceph.osd.pool.name=incus \
    source=incus

# 配置 CephFS
incus storage create cephfs cephfs \
    source=incus_data \
    ceph.cluster.name=ceph \
    ceph.user.name=admin
```

### 5.5 存储池管理

```bash
# 列出存储池
incus storage list

# 查看详情
incus storage show local

# 创建存储卷
incus storage volume create local container-data

# 列出卷
incus storage volume list local

# 附加卷到实例
incus storage volume attach local container-data my-container /data

# 查看卷使用情况
incus storage volume info local container-data
```

---

## 6. 网络配置

### 6.1 网络类型

| 类型 | 说明 | 使用场景 |
|------|------|----------|
| **bridge** | Linux 网桥，NAT 或路由 | 默认，简单部署 |
| **macvlan** | 直连物理网络 | 独立 IP，需要多 MAC |
| **ipvlan** | 共享物理接口 | 高性能，单 MAC |
| **sriov** | SR-IOV 虚拟功能 | 高性能虚拟化 |
| **ovn** | 软件定义网络 | 复杂网络需求 |
| **physical** | 直接物理连接 | 高性能网络 |

### 6.2 桥接网络配置

#### 6.2.1 创建桥接网络

```bash
# 创建网桥
incus network create incusbr0 \
    --type bridge \
    ipv4.address=10.10.0.1/24 \
    ipv4.nat=true \
    ipv6.address=none \
    dns.mode=managed \
    dns.domain=incus.local

# 启用 DHCP
incus network set incusbr0 ipv4.dhcp=true
incus network set incusbr0 ipv4.dhcp.ranges=10.10.0.100-10.10.0.200

# 查看网络
incus network show incusbr0
incus network list
```

#### 6.2.2 网络隧道（跨节点）

```bash
# 创建 VXLAN 隧道
incus network create incusbr0-tunnel \
    --type bridge \
    ipv4.address=10.11.0.1/24 \
    tunnel.id=100 \
    tunnel.local=10.0.0.10 \
    tunnel.peers=10.0.0.11,10.0.0.12
```

### 6.3 OVN 网络配置（高级）

```bash
# 先创建逻辑网络（仅在引导节点）
incus network create ovn-network \
    --type ovn \
    ipv4.address=10.20.0.1/24 \
    ipv4.nat=true \
    ipv6.address=fd42:20::1/64

# 在实例中使用
incus network attach-profile ovn-network default eth0
```

### 6.4 物理网络直通

```bash
# 添加物理网络设备到配置
incus profile device add default eth1 nic \
    nictype=physical \
    parent=enp1s0

# SR-IOV 虚拟功能
incus profile device add default eth2 nic \
    nictype=sriov \
    parent=enp1s0 \
    vf.id=5
```

### 6.5 网络防火墙

```bash
# 启用网络防火墙
incus network set incusbr0 ipv4.firewall=true
incus network set incusbr0 ipv6.firewall=true

# 添加 ACL
incus network set incusbr0 ipv4.acl=default

# 创建 ACL 规则
incus network acl create web-servers
incus network acl rule add web-servers \
    ingress action=allow protocol=tcp destination_port=80,443
```

---

## 7. 集群配置

### 7.1 集群架构

```
┌──────────────────────────────────────────────────────────────────┐
│                        Incus Cluster                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │   Node 1     │  │   Node 2     │  │   Node 3     │           │
│  │  (Leader)    │  │  (Member)    │  │  (Member)    │           │
│  │              │  │              │  │              │           │
│  │  ┌────────┐  │  │  ┌────────┐  │  │  ┌────────┐  │           │
│  │  │  API   │  │  │  │  API   │  │  │  │  API   │  │           │
│  │  │ 8443   │  │  │  │ 8443   │  │  │  │ 8443   │  │           │
│  │  └────────┘  │  │  └────────┘  │  │  └────────┘  │           │
│  │  ┌────────┐  │  │  ┌────────┐  │  │  ┌────────┐  │           │
│  │  │  DB    │  │  │  │  DB    │  │  │  │  DB    │  │           │
│  │  │ dqlite │  │  │  │ dqlite │  │  │  │ dqlite │  │           │
│  │  └────────┘  │  │  └────────┘  │  │  └────────┘  │           │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘           │
│         │                │                │                     │
│         └────────────────┼────────────────┘                     │
│                          │                                     │
│                  ┌───────▼────────┐                             │
│                  │  Cluster RAFT  │                             │
│                  │   Consensus    │                             │
│                  └────────────────┘                             │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    Shared Storage                         │   │
│  │                    (Ceph/External)                        │   │
│  └──────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────┘
```

### 7.2 初始化集群

#### 7.2.1 在引导节点上初始化

```bash
# 在 Node 1 (10.0.0.10) 上执行
cat <<EOF | incus admin init preseed
config:
  core.https_address: '[::]:8443'
  core.trust_password: "ClusterP@ssw0rd!"
cluster:
  server_name: incus-node1
  enabled: true
  member_config: []
  cluster_address: ""
  cluster_certificate: ""
  server_address: "10.0.0.10:8443"
  cluster_token: ""
storage_pools:
- name: local
  driver: zfs
  config:
    source: incus-pool/incus
networks:
- name: incusbr0
  type: bridge
  config:
    ipv4.address: 10.10.0.1/24
    ipv4.nat: "true"
profiles:
- name: default
  config: {}
  description: Default Incus profile
  devices:
    root:
      type: disk
      path: "/"
      pool: "local"
    eth0:
      type: nic
      network: "incusbr0"
      name: "eth0"
EOF
```

#### 7.2.2 获取集群令牌

```bash
# 在引导节点上生成令牌
incus cluster list
incus cluster token add incus-node2

# 令牌输出示例
# Cluster join token: 10.0.0.10:8443:eyJz...
```

### 7.3 添加节点到集群

#### 7.3.1 准备新节点

在每个新节点上（Node 2, Node 3）：

```bash
# 1. 安装 Incus（与引导节点相同版本）
sudo apt install -y incus incus-tools

# 2. 配置网络
sudo tee -a /etc/hosts <<EOF
10.0.0.10    incus-node1
10.0.0.11    incus-node2
10.0.0.12    incus-node3
EOF

# 3. 如果使用 ZFS，创建相同的 ZFS 池
# sudo zpool create -f incus-pool /dev/disk/by-id/...
```

#### 7.3.2 加入集群

```bash
# 方法1: 使用令牌（推荐）
incus admin init \
    --cluster \
    --cluster_token=<token_from_node1>

# 方法2: 使用预配置文件
cat <<EOF | incus admin init preseed
config:
  core.https_address: '[::]:8443'
cluster:
  server_name: incus-node2
  enabled: true
  member_config: []
  cluster_address: "10.0.0.10:8443"
  cluster_certificate: "<certificate_from_node1>"
  server_address: "10.0.0.11:8443"
  cluster_token: ""
storage_pools:
- name: local
  driver: zfs
  config:
    source: incus-pool/incus
networks:
- name: incusbr0
  type: bridge
  config:
    ipv4.address: 10.10.0.1/24
profiles:
- name: default
  devices: {}
EOF
```

#### 7.3.3 验证集群状态

```bash
# 在任意节点查看集群成员
incus cluster list

# 输出示例
# +-------------+---------------------------+----------+--------+
# |   NAME      |            URL            | DATABASE | STATUS |
# +-------------+---------------------------+----------+--------+
# | incus-node1 | https://10.0.0.10:8443    | YES      | ONLINE |
# | incus-node2 | https://10.0.0.11:8443    | NO       | ONLINE |
# | incus-node3 | https://10.0.0.12:8443    | NO       | ONLINE |
# +-------------+---------------------------+----------+--------+

# 查看集群组
incus cluster group list

# 查看成员信息
incus cluster show incus-node1
```

### 7.4 集群管理

#### 7.4.1 集群组配置

```bash
# 创建集群组
incus cluster group create compute

# 添加成员到组
incus cluster group assign compute incus-node2,incus-node3

# 查看组
incus cluster group show compute
incus cluster group list
```

#### 7.4.2 实例放置策略

```bash
# 创建具有放置策略的配置文件
incus profile create high-availability
incus profile set high-availability \
    cluster.evacuate=auto \
    instances.nics.host_name=enp

# 设置实例限制到特定节点
incus config set <instance> cluster.node incus-node1

# 设置实例组
incus config set <instance> cluster.group compute
```

#### 7.4.3 节点维护模式

```bash
# 将节点设置为维护模式（迁移实例）
incus cluster evacuate incus-node1

# 恢复节点
incus cluster restore incus-node1

# 从集群移除节点
incus cluster remove incus-node3 --force
```

### 7.5 高可用配置

```bash
# 配置集群副本数
incus config set cluster.max_standby 3
incus config set cluster.max_voters 3

# 配置数据库
incus config set cluster.healing_threshold 5

# 配置实例高可用
incus config set <instance> cluster.evacuate=auto
```

---

## 8. 虚拟机支持

### 8.1 启用虚拟化

#### 8.1.1 检查硬件支持

```bash
# 检查 CPU 虚拟化支持
lscpu | grep Virtualization
# 或
egrep -c '(vmx|svm)' /proc/cpuinfo

# 检查 KVM 模块
lsmod | grep kvm
sudo modprobe kvm_intel  # Intel
sudo modprobe kvm_amd    # AMD
```

#### 8.1.2 安装虚拟化组件

```bash
# 安装 QEMU 和相关组件
sudo apt install -y \
    qemu-kvm \
    qemu-utils \
    qemu-system-x86 \
    libvirt-daemon-system \
    ovmf \
    edk2-ovmf \
    virtinst \
    cpu-checker

# 验证 KVM
sudo kvm-ok
```

### 8.2 创建虚拟机

#### 8.2.1 创建云镜像 VM

```bash
# 查看可用镜像
incus image list images: | grep -i cloud

# 创建 Ubuntu VM (云镜像)
incus launch images:ubuntu/24.04/cloud vm-ubuntu \
    --vm \
    --profile default \
    -c limits.cpu=4 \
    -c limits.memory=8GiB \
    -d root,size=50GiB

# 创建 Debian VM
incus launch images:debian/12/cloud vm-debian \
    --vm \
    -c limits.cpu=2 \
    -c limits.memory=4GiB
```

#### 8.2.2 创建自定义 VM

```bash
# 方法1: 从 ISO 安装
# 首先下载 ISO
incus image import ubuntu-24.04-live-server-amd64.iso \
    --alias ubuntu-24.04-iso

# 创建 VM 并挂载 ISO
incus init ubuntu-24.04-iso vm-custom --vm
incus config device add vm-custom iso disk \
    source=ubuntu-24.04-live-server-amd64.iso \
    boot.priority=1

# 启动并连接
incus start vm-custom --console

# 方法2: 通过配置文件创建
cat <<EOF > vm-config.yaml
architecture: x86_64
config:
  limits.cpu: "4"
  limits.memory: 8GiB
  security.secureboot: "false"
devices:
  root:
    path: /
    pool: local
    type: disk
    size: 50GiB
  eth0:
    type: nic
    network: incusbr0
  config:
    type: disk
    source: ubuntu-24.04-live-server-amd64.iso
    boot.priority: 1
name: vm-custom
type: virtual-machine
EOF

incus create < vm-config.yaml
```

### 8.3 VM 配置管理

```bash
# CPU 配置
incus config set vm-ubuntu limits.cpu=4
incus config set vm-ubuntu limits.cpu.pin=0,1,2,3  # 绑定 CPU

# 内存配置
incus config set vm-ubuntu limits.memory=8GiB
incus config set vm-ubuntu limits.memory.hugepages=true

# 磁盘配置
incus config device override vm-ubuntu root size=100GiB
incus config device add vm-ubuntu data disk path=/data pool=local size=200GiB

# GPU 直通
incus config device add vm-ubuntu gpu gpu gpu=nvidia-0

# 网络配置
incus config device add vm-ubuntu eth1 nic nictype=bridged parent=br0
```

### 8.4 VM 管理

```bash
# 启动/停止/重启
incus start vm-ubuntu
incus stop vm-ubuntu
incus restart vm-ubuntu

# 查看控制台
incus console vm-ubuntu
# 退出: Ctrl+a q

# 查看状态
incus list --format=table
incus info vm-ubuntu

# 快照
incus snapshot create vm-ubuntu snap1
incus snapshot list vm-ubuntu
incus snapshot restore vm-ubuntu snap1

# 备份
incus export vm-ubuntu vm-ubuntu-backup.tar.gz
incus import vm-ubuntu-backup.tar.gz
```

---

## 9. 容器支持

### 9.1 创建容器

#### 9.1.1 基础容器创建

```bash
# 创建容器
incus launch images:ubuntu/24.04 container-ubuntu
incus launch images:debian/12 container-debian
incus launch images:alpine/3.19 container-alpine

# 使用指定配置
incus launch images:ubuntu/24.04 c-web \
    -c limits.cpu=2 \
    -c limits.memory=2GiB \
    -d root,size=20GiB

# 交互式创建
incus init images:ubuntu/24.04 my-container
incus config edit my-container
incus start my-container
```

#### 9.1.2 使用配置文件

```bash
# 创建自定义配置文件
incus profile create web-server

incus profile set web-server limits.cpu=2
incus profile set web-server limits.memory=4GiB
incus profile set web-server boot.autostart=true

# 添加设备
incus profile device add web-server root disk path=/ pool=local
incus profile device add web-server eth0 nic network=incusbr0

# 使用配置文件创建
incus launch images:ubuntu/24.04 web1 --profile web-server
incus launch images:ubuntu/24.04 web2 --profile web-server
```

### 9.2 容器配置

```bash
# 资源限制
incus config set container-ubuntu limits.cpu=4
incus config set container-ubuntu limits.memory=8GiB
incus config set container-ubuntu limits.memory.swap=true
incus config set container-ubuntu limits.disk.priority=5

# 安全配置
incus config set container-ubuntu security.nesting=true  # 允许嵌套虚拟化
incus config set container-ubuntu security.privileged=true  # 特权容器
incus config set container-ubuntu raw.lxc="lxc.apparmor.profile=unconfined"

# 环境变量
incus config set container-ubuntu environment.http_proxy=http://proxy:8080
incus config set container-ubuntu user.user-data=$(cat cloud-init.yml)

# 启动选项
incus config set container-ubuntu boot.autostart=true
incus config set container-ubuntu boot.autostart.delay=10
incus config set container-ubuntu boot.stop.priority=0
```

### 9.3 网络配置

```bash
# 添加额外网卡
incus config device add container-ubuntu eth1 nic nictype=bridged parent=br0

# 配置静态 IP
incus config device override container-ubuntu eth0 \
    ipv4.address=10.10.0.50

# 端口转发
incus config device add container-ubuntu http-proxy proxy \
    listen=tcp:0.0.0.0:8080 \
    connect=tcp:127.0.0.1:80

# 代理/转发
incus config device add container-ubuntu proxy-tcp proxy \
    bind=host \
    listen=tcp:0.0.0.0:3306 \
    connect=tcp:10.10.0.50:3306
```

### 9.4 存储管理

```bash
# 添加存储卷
incus storage volume create local data-volume size=100GiB

# 挂载到容器
incus storage volume attach local data-volume container-ubuntu /data

# 在容器内创建目录挂载
incus config device add container-ubuntu data disk \
    path=/opt/data \
    source=/home/user/data \
    type=disk

# 快照
incus snapshot create container-ubuntu snap1
incus snapshot restore container-ubuntu snap1
```

### 9.5 批量操作

```bash
# 批量启动/停止
incus start -c
incus list --format=json | jq -r '.[].name' | xargs -I {} incus start {}

# 批量执行命令
for container in $(incus list -f json | jq -r '.[].name'); do
    incus exec $container -- apt update
done

# 批量配置
incus list -f json | jq -r '.[].name' | \
    xargs -I {} incus config set {} limits.cpu=2
```

---

## 10. Docker in Incus

### 10.1 为什么需要 Docker in Incus

- **隔离性**: Docker 容器运行在 Incus 容器中，提供额外隔离层
- **安全性**: 限制 Docker 守护进程的访问权限
- **管理**: 统一使用 Incus 管理所有工作负载
- **灵活性**: 可以运行多个 Docker 版本

### 10.2 创建 Docker 容器

#### 10.2.1 方法1: 使用官方镜像

```bash
# 使用 Ubuntu Docker 镜像
incus launch images:ubuntu/24.04 docker-host \
    -c security.nesting=true \
    -c security.privileged=true

# 安装 Docker
incus exec docker-host -- bash -c "
curl -fsSL https://get.docker.com | sh
usermod -aG docker ubuntu
"

# 验证
incus exec docker-host -- docker --version
incus exec docker-host -- docker run hello-world
```

#### 10.2.2 方法2: 使用特权模式

```bash
# 创建特权容器
incus launch images:ubuntu/24.04 docker-privileged \
    -c security.nesting=true \
    -c security.privileged=true \
    -d root,size=50GiB

# 安装 Docker
incus exec docker-privileged -- bash -c "
# 更新系统
apt update && apt upgrade -y

# 安装依赖
apt install -y ca-certificates curl gnupg

# 添加 Docker GPG 密钥
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 添加仓库
echo \"deb [arch=\$(dpkg --print-architecture) \
    signed-by=/etc/apt/keyrings/docker.gpg] \
    https://download.docker.com/linux/ubuntu \
    \$(. /etc/os-release && echo \$VERSION_CODENAME) stable\" | \
    tee /etc/apt/sources.list.d/docker.list > /dev/null

# 安装 Docker
apt update
apt install -y docker-ce docker-ce-cli containerd.io \
    docker-buildx-plugin docker-compose-plugin

# 启动 Docker
systemctl enable docker
systemctl start docker
"

# 验证
incus exec docker-privileged -- docker info
```

### 10.3 配置 Docker 代理

```bash
# 为 Docker 配置代理
incus config set docker-host environment.HTTP_PROXY http://proxy:8080
incus config set docker-host environment.HTTPS_PROXY http://proxy:8080
incus config set docker-host environment.NO_PROXY localhost,127.0.0.1

# 或在容器内配置 daemon.json
incus exec docker-host -- bash -c "
mkdir -p /etc/docker
cat > /etc/docker/daemon.json <<EOF
{
  \"proxies\": {
    \"http-proxy\": \"http://proxy:8080\",
    \"https-proxy\": \"http://proxy:8080\",
    \"no-proxy\": \"localhost,127.0.0.1\"
  }
}
EOF

systemctl restart docker
"
```

### 10.4 Docker Compose 集成

```bash
# 安装 Docker Compose
incus exec docker-host -- bash -c "
# 安装 docker-compose 插件
apt install -y docker-compose-plugin

# 或独立安装
curl -SL https://github.com/docker/compose/releases/download/v2.24.0/docker-compose-linux-x86_64 \
    -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
"

# 使用 Compose
incus exec docker-host -- docker compose up -d
```

### 10.5 Docker Swarm in Incus

```bash
# 初始化 Swarm
incus exec docker-host -- docker swarm init --advertise-addr 10.10.0.50

# 添加更多节点
incus launch images:ubuntu/24.04 docker-worker1 -c security.nesting=true
# 重复上述 Docker 安装步骤

# 加入 Swarm
incus exec docker-worker1 -- docker swarm join \
    --token <token> 10.10.0.50:2377
```

### 10.6 最佳实践

1. **资源限制**:
```bash
# 限制 Docker 容器资源
incus config set docker-host limits.cpu=4
incus config set docker-host limits.memory=8GiB
```

2. **存储优化**:
```bash
# 为 Docker 数据使用独立卷
incus storage volume create local docker-data
incus config device add docker-host docker-data disk \
    path=/var/lib/docker pool=local source=docker-data
```

3. **网络配置**:
```bash
# 使用 macvlan 让 Docker 容器直接访问网络
incus config device add docker-host eth1 nic \
    nictype=macvlan parent=br0
```

4. **自动启动**:
```bash
incus config set docker-host boot.autostart=true
incus exec docker-host -- systemctl enable docker
```

---

## 11. 跨网络集群配置

### 11.1 跨网络场景

```
                         Internet
                            │
              ┌─────────────┼─────────────┐
              │             │             │
        ┌─────▼─────┐ ┌─────▼─────┐ ┌─────▼─────┐
        │  DC A     │ │   VPN     │ │  DC B     │
        │ 203.0.113.0/24        │ 198.51.100.0/24
        │           │ │           │ │           │
        │ Node A1   │ │           │ │ Node B1   │
        │ 203.0.113.10     │           │ │ 198.51.100.10
        │           │ │           │ │           │
        │ Node A2   │ │           │ │ Node B2   │
        │ 203.0.113.11     │           │ │ 198.51.100.11
        └───────────┘ └───────────┘ └───────────┘
```

### 11.2 WireGuard VPN 配置

#### 11.2.1 安装 WireGuard

```bash
# 在所有节点安装
sudo apt install -y wireguard wireguard-tools

# 生成密钥
wg genkey | tee privatekey | wg pubkey > publickey
```

#### 11.2.2 配置 WireGuard

**Node 1 (DC A, 203.0.113.10)**:
```bash
sudo tee /etc/wireguard/wg0.conf <<EOF
[Interface]
PrivateKey = <node1_private_key>
Address = 10.100.0.1/24
ListenPort = 51820

[Peer]
PublicKey = <node2_public_key>
Endpoint = 198.51.100.10:51820
AllowedIPs = 10.100.0.2/32,10.100.0.0/24
PersistentKeepalive = 25

[Peer]
PublicKey = <node3_public_key>
Endpoint = 198.51.100.11:51820
AllowedIPs = 10.100.0.3/32,10.100.0.0/24
PersistentKeepalive = 25
EOF

sudo systemctl enable --now wg-quick@wg0
```

**Node 2 (DC B, 198.51.100.10)**:
```bash
sudo tee /etc/wireguard/wg0.conf <<EOF
[Interface]
PrivateKey = <node2_private_key>
Address = 10.100.0.2/24
ListenPort = 51820

[Peer]
PublicKey = <node1_public_key>
Endpoint = 203.0.113.10:51820
AllowedIPs = 10.100.0.1/32,10.100.0.0/24
PersistentKeepalive = 25

[Peer]
PublicKey = <node3_public_key>
Endpoint = 198.51.100.11:51820
AllowedIPs = 10.100.0.3/32,10.100.0.0/24
EOF

sudo systemctl enable --now wg-quick@wg0
```

#### 11.2.3 验证 VPN 连接

```bash
# 检查 WireGuard 状态
sudo wg show

# 测试连接
ping 10.100.0.1
ping 10.100.0.2
```

### 11.3 跨网络 Incus 集群

#### 11.3.1 初始化跨网络集群

**Node 1 (引导节点)**:
```bash
cat <<EOF | incus admin init preseed
config:
  core.https_address: '10.100.0.1:8443'
  core.trust_password: "CrossDCP@ss!"
cluster:
  server_name: incus-dca-node1
  enabled: true
  server_address: "10.100.0.1:8443"
storage_pools:
- name: local
  driver: zfs
  config:
    source: incus-pool/incus
networks:
- name: incusbr0
  type: bridge
  config:
    ipv4.address: 10.10.0.1/24
    ipv4.nat: "true"
profiles:
- name: default
  devices:
    root:
      type: disk
      path: "/"
      pool: local
    eth0:
      type: nic
      network: incusbr0
EOF
```

**Node 2 (加入集群)**:
```bash
incus admin init \
    --cluster \
    --cluster_address=10.100.0.1:8443 \
    --cluster_token=<token>
```

### 11.4 跨网络存储

#### 11.4.1 使用 Ceph (推荐)

```bash
# 在所有节点安装 Ceph
sudo apt install -y ceph-common

# 配置 Ceph 集群（假设已有 Ceph 集群）
# 复制 ceph.conf 和 keyring 到所有节点

# 创建 Incus Ceph 存储池
incus storage create ceph-global ceph \
    ceph.cluster.name=ceph \
    ceph.user.name=admin \
    ceph.osd.pool.name=incus \
    source=incus

# 设置为集群共享存储
incus config storage set ceph-global source incus
```

#### 11.4.2 使用 Incus 内置复制

```bash
# 使用 Incus 远程存储同步
incus config set core.remote_cache_expiry 48
incus storage set local rsync.compression true
```

### 11.5 网络路由配置

```bash
# 配置网络路由，使容器可以跨 DC 通信

# 方法1: VXLAN 隧道
incus network create incusbr0-global \
    --type bridge \
    ipv4.address=10.30.0.1/24 \
    ipv4.dhcp.ranges=10.30.0.100-10.30.0.200 \
    tunnel.id=200 \
    tunnel.protocol=vxlan \
    tunnel.local=10.100.0.1

# 方法2: 使用 OVN
incus network create ovn-global \
    --type ovn \
    ipv4.address=10.40.0.1/24 \
    ipv4.nat=true
```

### 11.6 DNS 和服务发现

```bash
# 配置 DNS 转发
incus network set incusbr0 dns.forward=true
incus network set incusbr0 dns.mode=managed

# 添加 DNS 记录
incus network set incusbr0 \
    dns.records="[container1.incus.local]=10.10.0.50"
```

### 11.7 跨网络最佳实践

1. **延迟优化**: 使用本地存储 + 异步复制
2. **带宽管理**: 限制同步带宽
3. **故障域**: 使用集群组隔离 DC
4. **备份策略**: 每个备份到本地和远程

```bash
# 配置集群组
incus cluster group create dc-a
incus cluster group create dc-b

incus cluster group assign dc-a incus-dca-node1,incus-dca-node2
incus cluster group assign dc-b incus-dcb-node1,incus-dcb-node2

# 设置实例优先组
incus config set my-app cluster.group dc-a
incus config set my-app cluster.evacuate=migrate
```

---

## 12. 节点初始化脚本

以下是完整的节点初始化脚本，集成本教程的所有配置。

### 12.1 初始化脚本

```bash
#!/bin/bash
#############################################
# Incus Cluster Node Initialization Script #
# For Debian 13 (Trixie)                  #
#############################################

set -e

# ========================================
# 配置变量 - 修改为你的环境
# ========================================

# 节点配置
NODE_NAME="${NODE_NAME:-incus-node1}"
NODE_IP="${NODE_IP:-10.0.0.10}"
NODE_PUBLIC_IP="${NODE_PUBLIC_IP:-203.0.113.10}"

# 集群配置
CLUSTER_ENABLED="${CLUSTER_ENABLED:-false}"
CLUSTER_ADDRESS="${CLUSTER_ADDRESS:-}"
CLUSTER_TOKEN="${CLUSTER_TOKEN:-}"
CLUSTER_PASSWORD="${CLUSTER_PASSWORD:-ClusterP@ssw0rd!}"

# 网络配置
INTERNAL_INTERFACE="${INTERNAL_INTERFACE:-eth1}"
EXTERNAL_INTERFACE="${EXTERNAL_INTERFACE:-eth0}"
INTERNAL_SUBNET="${INTERNAL_SUBNET:-10.0.0.0/24}"

# 存储配置
STORAGE_BACKEND="${STORAGE_BACKEND:-zfs}"  # zfs, btrfs, lvm, dir
STORAGE_DISK="${STORAGE_DISK:-}"  # 留空自动检测或使用文件
STORAGE_POOL_NAME="${STORAGE_POOL_NAME:-incus-pool}"
STORAGE_SIZE="${STORAGE_SIZE:-100G}"  # 用于文件模式

# 网络配置
BRIDGE_NETWORK="${BRIDGE_NETWORK:-10.10.0.1/24}"
BRIDGE_DHCP_START="${BRIDGE_DHCP_START:-10.10.0.100}"
BRIDGE_DHCP_END="${BRIDGE_DHCP_END:-10.10.0.200}"

# Docker 配置
INSTALL_DOCKER="${INSTALL_DOCKER:-false}"

# VPN 配置 (跨网络集群)
VPN_ENABLED="${VPN_ENABLED:-false}"
VPN_INTERFACE="${VPN_INTERFACE:-wg0}"
VPN_SUBNET="${VPN_SUBNET:-10.100.0.0/24}"

# ========================================
# 颜色输出
# ========================================
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

log_info() {
    echo -e "${GREEN}[INFO]${NC} $1"
}

log_warn() {
    echo -e "${YELLOW}[WARN]${NC} $1"
}

log_error() {
    echo -e "${RED}[ERROR]${NC} $1"
}

# ========================================
# 检查 root 权限
# ========================================
if [[ $EUID -ne 0 ]]; then
   log_error "This script must be run as root"
   exit 1
fi

# ========================================
# 系统更新
# ========================================
log_info "Updating system..."
apt update
apt upgrade -y

# ========================================
# 安装基础工具
# ========================================
log_info "Installing base tools..."
apt install -y curl wget vim git htop tmux ufw \
    software-properties-common ca-certificates gnupg \
    apt-transport-https rsync net-tools

# ========================================
# 内核参数配置
# ========================================
log_info "Configuring kernel parameters..."
cat > /etc/sysctl.d/99-incus.conf <<EOF
# 网络转发
net.ipv4.ip_forward=1
net.ipv6.conf.forwarding=1

# 桥接网络
net.bridge.bridge-nf-call-arptables=0
net.bridge.bridge-nf-call-ip6tables=0
net.bridge.bridge-nf-call-iptables=0

# 集群相关
net.core.somaxconn=1024
net.ipv4.tcp_max_syn_backlog=2048

# 文件描述符
fs.inotify.max_user_watches=1048576
fs.inotify.max_user_instances=512

# 共享内存 (VM 支持)
kernel.shmmax=68719476736
kernel.shmall=4294967296

# BBR 优化
net.core.default_qdisc=fq
net.ipv4.tcp_congestion_control=bbr
EOF

sysctl --system

# ========================================
# 加载内核模块
# ========================================
log_info "Loading kernel modules..."
cat > /etc/modules-load.d/incus.conf <<EOF
overlay
br_netfilter
nf_nat
iptable_nat
iptable_filter
ip6table_nat
ip6table_filter
vhost_vsock
vhost_net
tun
EOF

# 立即加载
modprobe overlay
modprobe br_netfilter
modprobe vhost_vsock
modprobe vhost_net

# ========================================
# 配置主机名
# ========================================
log_info "Setting hostname..."
hostnamectl set-hostname $NODE_NAME

# 更新 /etc/hosts
cat > /etc/hosts <<EOF
127.0.0.1   localhost
::1         localhost ip6-localhost ip6-loopback

# 本节点
$NODE_IP    $NODE_NAME
${NODE_PUBLIC_IP} ${NODE_NAME}-public
EOF

# ========================================
# 配置时间同步
# ========================================
log_info "Configuring time synchronization..."
apt install -y chrony
systemctl enable --now chronyd

# ========================================
# 配置防火墙
# ========================================
log_info "Configuring firewall..."
ufw default deny incoming
ufw default allow outgoing

# SSH
ufw allow 22/tcp

# Incus
ufw allow 8443/tcp comment 'Incus API'
ufw allow 8444/tcp comment 'Incus Cluster'

# 允许内网
ufw allow from $INTERNAL_SUBNET

# WireGuard (如果启用)
if [[ "$VPN_ENABLED" == "true" ]]; then
    ufw allow 51820/udp comment 'WireGuard VPN'
fi

ufw --force enable

# ========================================
# 添加 Incus 仓库
# ========================================
log_info "Adding Incus repository..."
mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.zabbly.com/key.asc | \
    gpg --dearmor -o /etc/apt/keyrings/zabbly.gpg

cat > /etc/apt/sources.list.d/zabbly-incus.list <<EOF
deb [signed-by=/etc/apt/keyrings/zabbly.gpg] \
    https://pkgs.zabbly.com/incus/stable/debian bookworm main
EOF

apt update

# ========================================
# 安装虚拟化组件
# ========================================
log_info "Installing virtualization components..."
apt install -y qemu-kvm qemu-utils ovmf cpu-checker libvirt-daemon-system

# 验证 KVM
if kvm-ok &>/dev/null; then
    log_info "KVM acceleration available"
else
    log_warn "KVM acceleration not available"
fi

# ========================================
# 安装存储组件
# ========================================
log_info "Installing storage components..."

case $STORAGE_BACKEND in
    zfs)
        apt install -y zfsutils-linux
        ;;
    btrfs)
        apt install -y btrfs-progs
        ;;
    lvm)
        apt install -y lvm2 thin-provisioning-tools
        ;;
esac

# ========================================
# 配置存储
# ========================================
log_info "Configuring storage ($STORAGE_BACKEND)..."

case $STORAGE_BACKEND in
    zfs)
        if [[ -z "$STORAGE_DISK" ]]; then
            # 使用文件作为 ZFS 池 (用于测试)
            log_warn "No disk specified, using file-based ZFS pool"
            mkdir -p /var/lib/incus/disks
            truncate -s $STORAGE_SIZE /var/lib/incus/disks/$STORAGE_POOL_NAME.img
            zpool create -f -m none -O compression=lz4 \
                $STORAGE_POOL_NAME /var/lib/incus/disks/$STORAGE_POOL_NAME.img
        else
            # 使用物理磁盘
            zpool create -f -o ashift=12 -O compression=lz4 \
                -O atime=off -O relatime=on \
                $STORAGE_POOL_NAME $STORAGE_DISK
        fi
        ;;

    btrfs)
        if [[ -z "$STORAGE_DISK" ]]; then
            log_error "Btrfs requires a disk device"
            exit 1
        fi
        mkfs.btrfs -f $STORAGE_DISK
        mkdir -p /mnt/incus
        mount $STORAGE_DISK /mnt/incus
        ;;

    lvm)
        if [[ -z "$STORAGE_DISK" ]]; then
            log_error "LVM requires a disk device"
            exit 1
        fi
        pvcreate $STORAGE_DISK
        vgcreate $STORAGE_POOL_NAME $STORAGE_DISK
        lvcreate -L 100%FREE -n incus $STORAGE_POOL_NAME
        ;;
esac

# ========================================
# 安装 Incus
# ========================================
log_info "Installing Incus..."
apt install -y incus incus-tools incus-client

# 验证安装
INCUS_VERSION=$(incus version | head -1)
log_info "Incus version: $INCUS_VERSION"

# ========================================
# 配置 WireGuard VPN (如果启用)
# ========================================
if [[ "$VPN_ENABLED" == "true" ]]; then
    log_info "Configuring WireGuard VPN..."
    apt install -y wireguard wireguard-tools

    # 生成密钥
    mkdir -p /etc/wireguard
    cd /etc/wireguard
    wg genkey | tee privatekey | wg pubkey > publickey

    log_warn "WireGuard keys generated. Please configure peers manually."
    log_info "Public key: $(cat /etc/wireguard/publickey)"
fi

# ========================================
# 配置 Incus
# ========================================
log_info "Configuring Incus..."

# 确定 server_address
if [[ "$VPN_ENABLED" == "true" ]]; then
    SERVER_ADDRESS=$(hostname -I | awk '{print $2}'):8443  # VPN IP
else
    SERVER_ADDRESS=${NODE_IP}:8443
fi

# 创建预配置文件
PRESEED_FILE="/tmp/incus-preseed.yaml"

cat > $PRESEED_FILE <<EOF
config:
  core.https_address: '[::]:8443'
  core.trust_password: "${CLUSTER_PASSWORD}"
  images.auto_update_interval: "6"

cluster:
  server_name: "${NODE_NAME}"
  enabled: ${CLUSTER_ENABLED}
  server_address: "${SERVER_ADDRESS}"
EOF

# 如果是加入集群，添加集群信息
if [[ "$CLUSTER_ENABLED" == "true" && -n "$CLUSTER_ADDRESS" ]]; then
    cat >> $PRESEED_FILE <<EOF
  cluster_address: "${CLUSTER_ADDRESS}"
  cluster_token: "${CLUSTER_TOKEN}"
EOF
fi

# 添加存储配置
cat >> $PRESEED_FILE <<EOF

storage_pools:
- name: local
  driver: ${STORAGE_BACKEND}
EOF

case $STORAGE_BACKEND in
    zfs)
        cat >> $PRESEED_FILE <<EOF
  config:
    source: ${STORAGE_POOL_NAME}/incus
EOF
        ;;
    btrfs)
        cat >> $PRESEED_FILE <<EOF
  config:
    source: /mnt/incus
EOF
        ;;
    lvm)
        cat >> $PRESEED_FILE <<EOF
  config:
    source: ${STORAGE_POOL_NAME}/incus
EOF
        ;;
    dir)
        cat >> $PRESEED_FILE <<EOF
  config:
    source: /var/lib/incus/storage-pools/local
EOF
        ;;
esac

# 添加网络配置
cat >> $PRESEED_FILE <<EOF

networks:
- name: incusbr0
  type: bridge
  config:
    ipv4.address: ${BRIDGE_NETWORK}
    ipv4.nat: "true"
    ipv4.dhcp: "true"
    ipv4.dhcp.ranges: ${BRIDGE_DHCP_START}-${BRIDGE_DHCP_END}
    ipv6.address: none
    dns.mode: managed
    dns.domain: incus.local

profiles:
- name: default
  description: Default Incus profile
  config:
    boot.autostart: "false"
  devices:
    root:
      type: disk
      path: "/"
      pool: "local"
    eth0:
      type: nic
      network: "incusbr0"
      name: "eth0"
EOF

# 应用配置
log_info "Applying Incus configuration..."
incus admin init preseed < $PRESEED_FILE

# ========================================
# 配置用户权限
# ========================================
log_info "Configuring user permissions..."
if [ -n "$SUDO_USER" ]; then
    usermod -aG incus-admin $SUDO_USER
    log_info "User $SUDO_USER added to incus-admin group"
fi

# ========================================
# 创建 Docker 配置文件 (如果需要)
# ========================================
if [[ "$INSTALL_DOCKER" == "true" ]]; then
    log_info "Creating Docker profile..."
    incus profile create docker || true
    incus profile set docker security.nesting true
    incus profile set docker security.privileged true
    incus profile device add docker root disk path=/ pool=local
    incus profile device add docker eth0 nic network=incusbr0
    log_info "Docker profile created. Use: incus launch images:ubuntu/24.04 <name> --profile docker"
fi

# ========================================
# 显示状态信息
# ========================================
echo ""
log_info "==================================="
log_info "Incus Node Initialization Complete!"
log_info "==================================="
echo ""
log_info "Node: $NODE_NAME"
log_info "Internal IP: $NODE_IP"
log_info "External IP: $NODE_PUBLIC_IP"
log_info "Storage: $STORAGE_BACKEND"
echo ""

if [[ "$CLUSTER_ENABLED" == "true" ]]; then
    if [[ -z "$CLUSTER_ADDRESS" ]]; then
        log_info "This is the cluster bootstrap node."
        log_info "To add other nodes, run:"
        log_info "  incus cluster token add <node-name>"
    else
        log_info "Joined cluster at: $CLUSTER_ADDRESS"
    fi
fi

echo ""
log_info "Next steps:"
log_info "1. Log out and back in to apply group permissions"
log_info "2. Verify: incus list"
log_info "3. Create instances: incus launch images:ubuntu/24.04 my-container"
echo ""

# ========================================
# 清理
# ========================================
rm -f $PRESEED_FILE

log_info "Done!"
```

### 12.2 使用方法

#### 12.2.1 单节点部署

```bash
# 下载脚本
wget https://your-server/incus-init.sh
chmod +x incus-init.sh

# 基础安装（使用默认值）
sudo ./incus-init.sh

# 自定义配置
sudo NODE_NAME=my-server \
     NODE_IP=10.0.0.50 \
     STORAGE_DISK=/dev/sdb \
     CLUSTER_PASSWORD=SecurePass123 \
     ./incus-init.sh
```

#### 12.2.2 集群部署 - 引导节点

```bash
sudo CLUSTER_ENABLED=true \
     NODE_NAME=incus-node1 \
     NODE_IP=10.0.0.10 \
     NODE_PUBLIC_IP=203.0.113.10 \
     STORAGE_DISK=/dev/sdb \
     CLUSTER_PASSWORD=ClusterPass123! \
     ./incus-init.sh
```

#### 12.2.3 集群部署 - 工作节点

```bash
# 先在引导节点获取令牌
incus cluster token add incus-node2

# 在工作节点执行
sudo CLUSTER_ENABLED=true \
     NODE_NAME=incus-node2 \
     NODE_IP=10.0.0.11 \
     NODE_PUBLIC_IP=203.0.113.11 \
     STORAGE_DISK=/dev/sdb \
     CLUSTER_ADDRESS=10.0.0.10:8443 \
     CLUSTER_TOKEN=<paste_token_here> \
     ./incus-init.sh
```

#### 12.2.4 跨网络集群部署

```bash
# 启用 VPN
sudo VPN_ENABLED=true \
     VPN_SUBNET=10.100.0.0/24 \
     CLUSTER_ENABLED=true \
     NODE_NAME=incus-dca-node1 \
     NODE_IP=10.100.0.1 \
     NODE_PUBLIC_IP=203.0.113.10 \
     STORAGE_DISK=/dev/sdb \
     ./incus-init.sh
```

---

## 13. 常用命令参考

### 13.1 集群管理

```bash
# 查看集群状态
incus cluster list
incus cluster group list

# 添加节点
incus cluster token add <node-name>

# 移除节点
incus cluster remove <node-name>

# 节点 evacuate
incus cluster evacuate <node-name>
incus cluster restore <node-name>
```

### 13.2 实例管理

```bash
# 创建实例
incus launch images:ubuntu/24.04 <name>
incus init images:ubuntu/24.04 <name> --vm

# 列出实例
incus list
incus list --format=table --columns=n,s,dstate,4

# 实例信息
incus info <instance>
incus config show <instance>

# 操作实例
incus start <instance>
incus stop <instance>
incus restart <instance>
incus delete <instance>

# 进入实例
incus exec <instance> -- bash
incus console <instance>  # VM

# 快照
incus snapshot create <instance> <snapshot-name>
incus snapshot restore <instance> <snapshot-name>
incus snapshot delete <instance> <snapshot-name>
```

### 13.3 网络管理

```bash
# 网络列表
incus network list

# 创建网络
incus network create <name> --type bridge
incus network create <name> --type ovn

# 网络配置
incus network show <name>
incus network set <name> <key> <value>

# 网络附加
incus network attach <network> <instance> <device-name>
```

### 13.4 存储管理

```bash
# 存储池
incus storage list
incus storage show <pool>
incus storage create <name> <driver> [options]

# 存储卷
incus storage volume list <pool>
incus storage volume create <pool> <volume>
incus storage volume attach <pool> <volume> <instance> <path>
```

### 13.5 镜像管理

```bash
# 远程镜像服务器
incus remote list
incus remote add images https://images.linuxcontainers.org --protocol simplestreams

# 镜像操作
incus image list images:
incus image list images: | grep ubuntu
incus image copy images:ubuntu/24.04 local:
incus image delete <image-fingerprint>
```

---

## 14. 故障排查

### 14.1 常见问题

#### 14.1.1 容器无法启动

```bash
# 查看详细日志
incus info <instance> --show-log
incus log <instance>

# 检查配置
incus config show <instance>

# 检查资源
incus config show <instance> | grep limits
```

#### 14.1.2 网络连接问题

```bash
# 检查网络状态
incus network show incusbr0

# 检查防火墙
ufw status
iptables -L -n

# 检查 DHCP
incus network get incusbr0 ipv4.dhcp
```

#### 14.1.3 集群节点离线

```bash
# 查看集群状态
incus cluster list

# 强制移除节点
incus cluster remove <node-name> --force

# 重置集群
incus cluster list
incus config cluster show
```

#### 14.1.4 存储问题

```bash
# ZFS 状态
zpool status
zfs list

# 检查磁盘空间
df -h
incus storage info local

# 检查存储卷
incus storage volume list local
```

### 14.2 日志位置

```bash
# Incus 守护进程日志
journalctl -u incus -f

# 实例日志
/var/log/incus/<instance>/

# 容器控制台日志
incus console <instance> --show-log

# 系统日志
journalctl -xe
```

### 14.3 性能调优

```bash
# 查看资源使用
incus list --format=csv --columns=n,cpu,memory,disk

# 调整实例资源
incus config set <instance> limits.cpu=4
incus config set <instance> limits.memory=8GiB

# 开启 metrics
incus config set core.metrics_address 0.0.0.0:9100

# 使用 Prometheus 监控
curl http://localhost:9100/metrics
```

### 14.4 备份与恢复

```bash
# 导出实例
incus export <instance> backup.tar.gz

# 导入实例
incus import backup.tar.gz

# 存储备份
zfs snapshot incus-pool/incus@backup
zfs send incus-pool/incus@backup > /backup/incus-backup.zfs

# 恢复
zfs receive incus-pool/incus < /backup/incus-backup.zfs
```

---

## 附录 A: 网络拓扑示例

```
                              ┌─────────────────────────────────┐
                              │         Internet                │
                              └─────────────────┬───────────────┘
                                                │
                              ┌─────────────────▼───────────────┐
                              │        Firewall/Router          │
                              │         203.0.113.1             │
                              └─────────────────┬───────────────┘
                                                │
        ┌───────────────────────────────────────┼───────────────────────────────────────┐
        │                                       │                                       │
┌───────▼────────┐                     ┌───────▼────────┐                     ┌───────▼────────┐
│   Node 1       │                     │   Node 2       │                     │   Node 3       │
│                │                     │                │                     │                │
│ eth0: 203.0.113.10                  │ eth0: 203.0.113.11                 │ eth0: 203.0.113.12
│ (Public)       │                     │ (Public)       │                     │ (Public)       │
│                │                     │                │                     │                │
│ eth1: 10.0.0.10 │◄────────────────────►│ eth1: 10.0.0.11 │◄────────────────────►│ eth1: 10.0.0.12 │
│ (Cluster/VPN)  │                     │ (Cluster/VPN)  │                     │ (Cluster/VPN)  │
│                │                     │                │                     │                │
│ wg0: 10.100.0.1│                     │ wg0: 10.100.0.2│                     │ wg0: 10.100.0.3│
│ (VPN Tunnel)   │                     │ (VPN Tunnel)   │                     │ (VPN Tunnel)   │
└───────┬────────┘                     └───────┬────────┘                     └───────┬────────┘
        │                                       │                                       │
        └───────────────────────────────────────┼───────────────────────────────────────┘
                                                │
                              ┌─────────────────▼───────────────┐
                              │       incusbr0                  │
                              │       10.10.0.1/24              │
                              │       (Container Network)       │
                              └─────────────────────────────────┘
                                                │
        ┌───────────────┬───────────────┬───────────────┬───────────────┐
        │               │               │               │               │
┌───────▼────────┐┌──────▼───────┐┌──────▼───────┐┌──────▼───────┐┌──────▼───────┐
│   web-1        ││   web-2      ││   db-1       ││   vm-app     ││   docker-1   │
│ 10.10.0.10     ││ 10.10.0.11   ││ 10.10.0.20   ││ 10.10.0.30   ││ 10.10.0.40   │
└────────────────┘└──────────────┘└──────────────┘└──────────────┘└──────────────┘
```

---

## 附录 B: 快速参考卡片

### Incus 集群部署流程

```
1. 准备环境
   ├─ 配置网络 (双网卡)
   ├─ 配置 /etc/hosts
   ├─ 同步时间 (chrony)
   └─ 配置防火墙

2. 安装 Incus
   ├─ 添加官方仓库
   ├─ 安装 incus 包
   └─ 安装存储驱动 (ZFS/Btrfs)

3. 初始化集群
   ├─ Node 1: incus admin init (bootstrap)
   ├─ Node 2+: incus admin init --cluster
   └─ 验证: incus cluster list

4. 配置存储
   ├─ 本地: ZFS/Btrfs
   └─ 分布式: Ceph (可选)

5. 配置网络
   ├─ 桥接: incusbr0
   ├─ OVN: 高级网络 (可选)
   └─ VPN: WireGuard (跨 DC)

6. 部署工作负载
   ├─ 容器: incus launch images:ubuntu/24.04
   ├─ VM: incus launch images:ubuntu/24.04 --vm
   └─ Docker: 使用 security.nesting=true
```

---

## 参考资源

- **官方文档**: https://linuxcontainers.org/incus/docs/main/
- **GitHub**: https://github.com/lxc/incus
- **论坛**: https://discuss.linuxcontainers.org/
- **镜像列表**: https://images.linuxcontainers.org/

---

**文档结束**

如有问题或建议，请参考官方文档或社区论坛。
