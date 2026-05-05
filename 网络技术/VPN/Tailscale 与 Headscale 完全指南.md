---
title: Tailscale 与 Headscale 完全指南
created: 2026-05-05
tags:
  - Tailscale
  - Headscale
  - VPN
  - WireGuard
  - OpenWrt
  - NAT穿透
  - 组网
---

# Tailscale 与 Headscale 完全指南

## 概述

**Tailscale** 是基于 WireGuard 的零配置 Mesh VPN，让你在几分钟内组建安全的私有网络。**Headscale** 是 Tailscale 控制服务器的开源替代品，可以完全自建，不依赖任何第三方服务。

### 方案对比

| 特性 | Tailscale 官方 | Headscale 自建 |
| ---- | -------------- | -------------- |
| **部署难度** | ⭐ 极简 | ⭐⭐⭐ 中等 |
| **控制服务器** | Tailscale 云端托管 | 自有服务器完全掌控 |
| **数据隐私** | 元数据经 Tailscale | 所有数据完全自控 |
| **免费额度** | 100 台设备 / 3 用户 | 无限制 |
| **DERP 中继** | 全球官方节点 | 需自建或用官方 |
| **MagicDNS** | ✅ 内置 | ⚠️ 需配置 |
| **ACL 策略** | Web UI 管理 | YAML 文件配置 |
| **适合场景** | 个人/小团队快速组网 | 企业/隐私敏感/大规模部署 |

### 工作原理简图

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   设备 A      │     │  控制服务器    │     │   设备 B      │
│ (手机/电脑)   │◄───►│ (Tailscale/   │◄───►│ (路由/服务器)  │
│              │     │  Headscale)   │     │              │
└──────────────┘     └──────────────┘     └──────────────┘
       │                                          │
       └────────── WireGuard 直连 (P2P) ──────────┘
                      或 DERP 中继转发
```

> **关键点**：控制服务器只负责**密钥交换和节点协调**，实际数据流是设备之间 **端对端加密直连**。即使控制服务器宕机，已建立的连接不受影响。

---

## 第一部分：最简方案 — 直接使用 Tailscale

### 1.1 注册 Tailscale 账号

1. 访问 [https://login.tailscale.com](https://login.tailscale.com)
2. 使用 Google / Microsoft / GitHub 账号登录
3. 免费计划包含：**100 台设备 + 3 个用户**

### 1.2 安装客户端

#### Linux

```bash
# 一键安装（推荐）
curl -fsSL https://tailscale.com/install.sh | sh

# 或使用包管理器
# Ubuntu/Debian
sudo apt-get install tailscale

# CentOS/RHEL
sudo yum install tailscale
```

#### macOS

```bash
# App Store 安装
# 或使用 brew
brew install --cask tailscale
```

#### Windows

从 [https://tailscale.com/download/windows](https://tailscale.com/download/windows) 下载安装包。

### 1.3 登录并连接

```bash
# Linux / macOS 终端
sudo tailscale up

# 会输出一个登录链接，在浏览器中打开完成认证
```

### 1.4 常用命令

```bash
# 查看状态
tailscale status

# 查看本机 Tailscale IP
tailscale ip

# Ping 另一个节点（会显示直连/中继）
tailscale ping <节点名或IP>

# 断开连接
sudo tailscale down

# 退出登录
sudo tailscale logout

# 开启 exit node（让其他设备通过此节点上网）
sudo tailscale up --advertise-exit-node

# 使用别的节点作为 exit node
sudo tailscale up --exit-node=<节点名>
```

### 1.5 子网路由（Subnet Router）

让 Tailscale 网络中的设备可以访问某节点所在的局域网：

```bash
# 在局域网网关设备上执行
sudo tailscale up --advertise-routes=192.168.1.0/24

# 其他设备需要接受路由
sudo tailscale up --accept-routes
```

然后在 Tailscale 管理后台 [https://login.tailscale.com/admin/machines](https://login.tailscale.com/admin/machines) 中，找到该节点，点击 `...` → Edit route settings → 启用子网路由。

### 1.6 Tailscale SSH

免密 SSH 到任何 Tailscale 节点：

```bash
# 在服务端开启
sudo tailscale up --ssh

# 客户端直接连接
tailscale ssh user@<节点名>
```

---

## 第二部分：自建方案 — Headscale

### 2.1 架构说明

```
┌─────────────────────────────────────────────┐
│              你的服务器 (VPS)                  │
│                                             │
│  ┌──────────────┐    ┌──────────────────┐   │
│  │  Headscale    │    │  DERP 中继服务器   │   │
│  │  控制服务器    │    │  (可选，自建)      │   │
│  │  :8080/:443  │    │  :3478/HTTPS     │   │
│  └──────────────┘    └──────────────────┘   │
│                                             │
│  ┌──────────────┐                          │
│  │ Headscale-UI │   可选的 Web 管理界面       │
│  │  :3000       │                          │
│  └──────────────┘                          │
└─────────────────────────────────────────────┘
```

### 2.2 服务器要求

| 项目 | 最低 | 推荐 |
| ---- | ---- | ---- |
| CPU | 1 核 | 2 核 |
| 内存 | 512 MB | 1 GB |
| 磁盘 | 10 GB | 20 GB SSD |
| 带宽 | 1 Mbps | 10 Mbps+ |
| 系统 | Debian 11+ / Ubuntu 20.04+ | Ubuntu 22.04 LTS |

> Headscale 控制服务器资源消耗极低。但如果你还自建 DERP 中继，带宽要求更高。

### 2.3 Docker Compose 部署（推荐）

#### 步骤一：创建目录结构

```bash
mkdir -p /opt/headscale/{config,data}
cd /opt/headscale
```

#### 步骤二：下载配置文件

```bash
# 下载示例配置
wget -O config/config.yaml \
  https://raw.githubusercontent.com/juanfont/headscale/main/config-example.yaml
```

#### 步骤三：编辑关键配置

```bash
nano config/config.yaml
```

**必须修改的配置项：**

```yaml
# 你的服务器访问地址（必改）
server_url: https://headscale.yourdomain.com

# 监听地址
listen_addr: 0.0.0.0:8080

# MagicDNS（建议开启）
magic_dns: true

# 基础域名
base_domain: tail.example.com

# 数据库配置（默认 SQLite 即可）
database:
  type: sqlite
  sqlite:
    path: /var/lib/headscale/db.sqlite

# IP 地址段（保持默认或自定义，不要和现有网段冲突）
ip_prefixes:
  - fd7a:115c:a1e0::/48    # IPv6
  - 100.64.0.0/10           # IPv4 CGNAT 地址段

# DERP 配置
derp:
  server:
    enabled: false           # 是否自建 DERP（见后文）
  urls:
    - https://controlplane.tailscale.com/derpmap/default
  paths: []
  auto_update_enabled: true
  update_frequency: 24h
```

#### 步骤四：创建 Docker Compose 文件

```yaml
# /opt/headscale/docker-compose.yml
version: '3.8'

services:
  headscale:
    image: headscale/headscale:latest
    container_name: headscale
    restart: unless-stopped
    command: headscale serve
    volumes:
      - ./config:/etc/headscale
      - ./data:/var/lib/headscale
    ports:
      - "8080:8080"     # HTTP API / Web UI
      - "50443:50443"   # gRPC（用于 DERP）
    environment:
      - TZ=Asia/Shanghai

  # 可选：Headscale Web 管理界面
  headscale-ui:
    image: ghcr.io/gurucomputing/headscale-ui:latest
    container_name: headscale-ui
    restart: unless-stopped
    ports:
      - "3000:80"
    environment:
      - TZ=Asia/Shanghai
```

#### 步骤五：启动服务

```bash
cd /opt/headscale
docker compose up -d

# 查看日志确认正常
docker logs -f headscale
```

#### 步骤六：创建用户

```bash
# 创建用户（相当于 Tailscale 的 "tailnet"）
docker exec -it headscale headscale users create myuser

# 查看用户列表
docker exec -it headscale headscale users list
```

#### 步骤七：生成预认证密钥

```bash
# 生成可重复使用的密钥（方便多设备连接）
docker exec -it headscale \
  headscale preauthkeys create \
  --user myuser \
  --reusable \
  --expiration 720d
```

> 记录输出的 Key，客户端连接时需要。

### 2.4 Nginx 反向代理（HTTPS）

Headscale 必须通过 HTTPS 对外服务（客户端要求）。使用 Nginx + Let's Encrypt：

```nginx
# /etc/nginx/sites-available/headscale
server {
    listen 443 ssl http2;
    server_name headscale.yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/headscale.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/headscale.yourdomain.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $server_name;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket 支持（关键！）
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_read_timeout 86400;
    }

    # gRPC 支持（用于 DERP）
    location /grpc/ {
        grpc_pass grpc://127.0.0.1:50443;
        grpc_set_header X-Real-IP $remote_addr;
    }
}
```

```bash
# 启用配置
sudo ln -s /etc/nginx/sites-available/headscale /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx

# 申请 SSL 证书
sudo certbot --nginx -d headscale.yourdomain.com
```

### 2.5 客户端连接到 Headscale

#### Linux

```bash
# 安装 Tailscale 客户端（正常安装）
curl -fsSL https://tailscale.com/install.sh | sh

# 连接到 Headscale（指定登录服务器）
sudo tailscale up \
  --login-server https://headscale.yourdomain.com \
  --authkey <你的预认证密钥> \
  --hostname my-linux-server

# 或交互式登录（会输出 URL，需在浏览器打开并认证）
sudo tailscale up --login-server https://headscale.yourdomain.com
```

#### Windows

1. 正常安装 Tailscale 客户端
2. 打开 Tailscale 系统托盘图标 → **Settings**
3. 找到 **Custom Login Server** / **自定义登录服务器**
4. 填入：`https://headscale.yourdomain.com`
5. 点击 **Log in** / **登录**

#### macOS

```bash
# 终端方式
sudo tailscale up --login-server https://headscale.yourdomain.com --authkey <KEY>

# 或通过 App 界面：Settings → Custom Coordination Server
```

### 2.6 Headscale 常用管理命令

```bash
# 查看所有节点
docker exec -it headscale headscale nodes list

# 查看节点路由
docker exec -it headscale headscale nodes list-routes

# 启用节点的子网路由
docker exec -it headscale \
  headscale routes enable \
  --route 192.168.1.0/24 \
  --node <NODE_ID>

# 删除节点
docker exec -it headscale headscale nodes delete --identifier <NODE_ID>

# 查看预认证密钥
docker exec -it headscale headscale preauthkeys list --user myuser

# 失效一个密钥
docker exec -it headscale headscale preauthkeys expire --key <KEY_PREFIX>
```

### 2.7 ACL 策略配置

Headscale 使用 YAML 文件管理访问控制策略：

```yaml
# /opt/headscale/config/acl.yaml
groups:
  group:admin:
    - myuser

acls:
  # 管理员可以访问所有
  - action: accept
    src:
      - group:admin
    dst:
      - "*:*"

  # 所有节点互相可以 SSH
  - action: accept
    src:
      - "*"
    dst:
      - "*:22"

  # 允许 ping
  - action: accept
    src:
      - "*"
    dst:
      - "*:icmp"
```

在 `config.yaml` 中启用 ACL：

```yaml
acl:
  policy:
    path: /etc/headscale/acl.yaml
```

### 2.8 自建 DERP 中继服务器（可选但推荐）

DERP 是 Tailscale 的中继服务器，用于在无法直连时转发流量。

#### 为什么要自建？

- 官方 DERP 服务器在海外，中国大陆访问延迟高
- 自建 DERP 可以大幅提升国内节点间的中继速度
- 即使大部分时间 P2P 直连，DERP 仍是连接建立的关键

#### Docker 部署 DERP

```yaml
# /opt/derp/docker-compose.yml
version: '3.8'

services:
  derp:
    image: ghcr.io/yangchuansheng/derper:latest
    container_name: derp
    restart: unless-stopped
    ports:
      - "3478:3478/udp"    # STUN
      - "443:443"           # HTTPS
    environment:
      - DERP_DOMAIN=derp.yourdomain.com
      - DERP_CERT_MODE=letsencrypt
      - DERP_ADDR=:443
      - DERP_STUN_ADDR=:3478
      - DERP_VERIFY_CLIENTS=false
    volumes:
      - ./certs:/app/certs
```

#### 在 Headscale 中配置自定义 DERP

修改 `config.yaml`：

```yaml
derp:
  server:
    enabled: false
  urls:
    - https://controlplane.tailscale.com/derpmap/default
  paths:
    - /etc/headscale/derp.yaml
  auto_update_enabled: true
  update_frequency: 24h
```

创建 `derp.yaml`：

```yaml
# /opt/headscale/config/derp.yaml
regions:
  900:
    regionid: 900
    regioncode: cn
    regionname: China DERP
    nodes:
      - name: cn-derp-1
        regionid: 900
        hostname: derp.yourdomain.com
        stunport: 3478
        stunonly: false
        derpport: 443
        ipv4: "你的服务器公网IP"
```

---

## 第三部分：OpenWrt 路由器客户端连接

### 3.1 安装前准备

#### 检查空间

```bash
# 查看可用空间（Tailscale 需要约 30MB）
df -h

# 如果空间不足，使用 USB 扩展或 extroot
```

#### 确认架构

```bash
cat /etc/openwrt_release | grep ARCH
# 例如：mipsel_24kc, aarch64_cortex-a53 等
```

### 3.2 安装 Tailscale

#### 方法一：通过 opkg 安装（推荐，OpenWrt 22.03+）

```bash
# 更新软件源
opkg update

# 安装 Tailscale 核心
opkg install tailscale

# 安装 LuCI Web 界面（可选）
opkg install luci-app-tailscale
```

#### 方法二：手动下载安装

如果官方仓库没有你的架构版本：

```bash
# 访问 Tailscale 下载页面，找到对应架构的包
# https://tailscale.com/download

# 或从 OpenWrt 软件包源下载
# https://downloads.openwrt.org/snapshots/packages/<arch>/packages/

# 下载后安装
opkg install tailscale_*.ipk
```

### 3.3 基本配置

#### 启动服务

```bash
# 启动并设置开机自启
/etc/init.d/tailscale start
/etc/init.d/tailscale enable
```

#### 连接到 Tailscale 官方

```bash
tailscale up --hostname=openwrt-router
# 在浏览器中打开输出的链接完成认证
```

#### 连接到 Headscale

```bash
tailscale up \
  --login-server https://headscale.yourdomain.com \
  --authkey <你的预认证密钥> \
  --hostname=openwrt-router \
  --advertise-routes=192.168.1.0/24
```

> `--advertise-routes=192.168.1.0/24` 让其他 Tailscale 设备可以访问路由器后面的整个局域网。

### 3.4 配置为子网路由器（Subnet Router）

让 Tailscale 网络中的设备可以通过 OpenWrt 路由器访问局域网内所有设备：

```bash
tailscale up \
  --advertise-routes=192.168.1.0/24 \
  --accept-routes \
  --hostname=openwrt-router
```

在 Tailscale 管理后台（或 Headscale）中启用该路由。

### 3.5 配置为 Exit Node（全局代理）

让其他设备通过 OpenWrt 路由器上网：

```bash
tailscale up \
  --advertise-exit-node \
  --hostname=openwrt-router
```

### 3.6 防火墙配置

这是 OpenWrt 上最关键的一步。

#### 创建 Tailscale 网络接口

```bash
# 方法一：通过 uci 命令
uci set network.tailscale=interface
uci set network.tailscale.proto='none'
uci set network.tailscale.device='tailscale0'
uci commit network
/etc/init.d/network restart
```

或在 LuCI 中：**网络 → 接口 → 添加新接口** → 名称 `tailscale`，协议 `不托管`，接口 `tailscale0`。

#### 配置防火墙区域

```bash
# 添加 tailscale 防火墙区域
uci add firewall zone
uci set firewall.@zone[-1].name='tailscale'
uci set firewall.@zone[-1].input='ACCEPT'
uci set firewall.@zone[-1].output='ACCEPT'
uci set firewall.@zone[-1].forward='REJECT'
uci set firewall.@zone[-1].network='tailscale'
uci set firewall.@zone[-1].masq='1'
uci set firewall.@zone[-1].mtu_fix='1'

# 允许 tailscale → lan 转发
uci add firewall forwarding
uci set firewall.@forwarding[-1].src='tailscale'
uci set firewall.@forwarding[-1].dest='lan'

# 允许 lan → tailscale 转发
uci add firewall forwarding
uci set firewall.@forwarding[-1].src='lan'
uci set firewall.@forwarding[-1].dest='tailscale'

uci commit firewall
/etc/init.d/firewall restart
```

#### 防火墙配置文件参考

编辑 `/etc/config/firewall`，确保包含：

```
config zone
    option name 'tailscale'
    list network 'tailscale'
    option input 'ACCEPT'
    option output 'ACCEPT'
    option forward 'REJECT'
    option masq '1'
    option mtu_fix '1'

config forwarding
    option src 'tailscale'
    option dest 'lan'

config forwarding
    option src 'lan'
    option dest 'tailscale'
```

### 3.7 LuCI Web 界面管理

安装 `luci-app-tailscale` 后：

1. 登录 OpenWrt 管理界面
2. 进入 **服务 → Tailscale**
3. 可以查看状态、启停服务、管理连接

### 3.8 常见问题排查

#### 空间不足

```bash
# 将 Tailscale 二进制移到 USB 存储
mkdir -p /mnt/usb/tailscale
cp /usr/sbin/tailscale /mnt/usb/tailscale/
cp /usr/sbin/tailscaled /mnt/usb/tailscale/

# 创建符号链接
ln -sf /mnt/usb/tailscale/tailscale /usr/sbin/tailscale
ln -sf /mnt/usb/tailscale/tailscaled /usr/sbin/tailscaled
```

#### DNS 问题

```bash
# 检查 resolv.conf
cat /etc/resolv.conf

# 如果 MagicDNS 不工作，手动添加
echo "nameserver 100.100.100.100" >> /etc/resolv.conf
```

---

## 第四部分：Android 手机连接

### 4.1 使用官方 Tailscale App（简单方案）

#### 安装

1. 从 **Google Play** 搜索 "Tailscale" 并安装
2. 或从 [F-Droid](https://f-droid.org/packages/com.tailscale.ipn/) 安装（开源版本）
3. 或直接下载 APK：[https://github.com/tailscale/tailscale-android/releases](https://github.com/tailscale/tailscale-android/releases)

#### 连接到 Tailscale 官方

1. 打开 Tailscale App
2. 点击 **Sign in** 登录你的 Tailscale 账号
3. 授权 VPN 连接
4. 连接成功后，状态栏会显示 🔑 钥匙图标

#### 使用 Exit Node

1. 点击 Tailscale App 中的三个点菜单
2. 选择 **Use exit node**
3. 选择你想使用的出口节点
4. 所有手机流量将通过该节点转发

### 4.2 连接到 Headscale

Tailscale Android App 默认不支持自定义服务器，需要以下方法之一：

#### 方法一：使用自定义 URL（部分版本支持）

1. 打开 Tailscale App
2. 连续点击版本号 **7 次**（启用开发者模式）
3. 在设置中找到 **Custom coordination server URL**
4. 输入：`https://headscale.yourdomain.com`
5. 重新登录

#### 方法二：使用 Headscale 生成的认证链接

```bash
# 在服务器上生成交互式登录
docker exec -it headscale \
  headscale nodes register \
  --user myuser \
  --name my-android
```

会输出一个 URL，在手机浏览器中打开该 URL，然后 Tailscale App 会自动使用 Headscale 服务器。

#### 方法三：使用第三方客户端（推荐）

**[Tailscale-Android-Headscale](https://github.com/tailscale/tailscale-android)** 修改版，或使用：

- **[WireGuard](https://play.google.com/store/apps/details?id=com.wireguard.android)** 官方客户端（手动导出配置）
- **[Headscale-Client](https://github.com/Double3H/HeadscaleClient)** — 专门为 Headscale 设计的 Android 客户端

#### 方法四：通过 ADB 配置（高级）

```bash
# 连接手机到电脑，ADB 调试已开启
# 修改 Tailscale 配置
adb shell "echo 'https://headscale.yourdomain.com' > /data/data/com.tailscale.ipn/files/prefs/login-server"

# 重启 Tailscale
adb shell am force-stop com.tailscale.ipn
```

### 4.3 Android 常用操作

| 操作 | 方法 |
| ---- | ---- |
| 查看连接状态 | 下拉通知栏 / 打开 App |
| 使用 Exit Node | App → 三点菜单 → Use exit node |
| 访问局域网设备 | 直接使用 Tailscale IP 或 MagicDNS 名称 |
| 开机自启 | App → Settings → Start on boot |
| 查看本机 IP | App → 点击已连接的状态 → 查看 Tailscale IP |

---

## 第五部分：NAT 穿透与内网穿透

### 5.1 Tailscale NAT 穿透原理

Tailscale/WireGuard 的 NAT 穿透是其核心能力之一。

```
设备 A (NAT 后)          DERP 中继服务器          设备 B (NAT 后)
    │                        │                        │
    ├─── 1. 注册到控制服务器 ─┤                        │
    │                        ├─── 1. 注册到控制服务器 ─┤
    │                        │                        │
    ├─── 2. 通过 STUN 探测 ──┤──── 2. 通过 STUN 探测 ─┤
    │    自己的公网端口       │     自己的公网端口      │
    │                        │                        │
    ├─── 3. 控制服务器交换两端的公网地址 ───────────────┤
    │                        │                        │
    ├─── 4. 打洞尝试（UDP 打洞）───────────────────────┤
    │    ✅ 直连成功！        │                        │
    │                        │                        │
    │    如果直连失败：        │                        │
    ├─── 5. 通过 DERP 中继 ──┼──── 5. 转发数据 ───────┤
    │                        │                        │
```

### 5.2 连接类型

| 类型 | 说明 | 延迟 | 速度 |
| ---- | ---- | ---- | ---- |
| **Direct (直连)** | P2P 打洞成功，数据端到端加密直传 | ⭐ 最低 | ⭐ 最快 |
| **DERP Relay (中继)** | 无法直连，通过 DERP 服务器转发 | ⭐⭐ 中等 | ⭐⭐ 受限于中继带宽 |
| **Relay (HEP)** | 部分场景下的中继方式 | ⭐⭐⭐ 较高 | ⭐⭐⭐ 较慢 |

### 5.3 检查连接质量

```bash
# 查看与目标节点的连接方式
tailscale ping <节点名>

# 输出示例：
# pong from myserver via DERP(xxx) in 45ms    ← 中继
# pong from myserver via 203.0.113.5:41641 in 2ms  ← 直连

# 查看详细网络信息
tailscale netcheck

# 输出包含：
# - UDP: true/false  (是否支持 UDP)
# - IPv4: yes/no
# - DERP延迟: 各区域的延迟
# - MappingVariesByDestIP: NAT类型判断
```

### 5.4 四种 NAT 类型

| NAT 类型 | 描述 | 打洞成功率 |
| -------- | ---- | ---------- |
| **Full Cone (完全锥形)** | 外部任意 IP 都可通过映射端口访问 | ⭐⭐⭐⭐⭐ 100% |
| **Restricted Cone (受限锥形)** | 仅限已发送过数据的 IP | ⭐⭐⭐⭐ ~95% |
| **Port Restricted Cone (端口受限锥形)** | 需 IP + 端口都匹配 | ⭐⭐⭐ ~80% |
| **Symmetric (对称型)** | 每个目标分配不同端口 | ⭐⭐ ~40% |

> **关键**：当两端都是 Symmetric NAT 时，直连几乎不可能，必须依赖 DERP 中继。

### 5.5 优化 NAT 穿透

#### 方法一：端口映射（最有效）

在路由器上为 Tailscale 设置端口转发：

```
协议：UDP
外部端口：41641
内部 IP：运行 Tailscale 的设备内网 IP
内部端口：41641
```

#### 方法二：UPnP / NAT-PMP

如果路由器支持 UPnP，Tailscale 会自动尝试通过 UPnP 开放端口。

```bash
# OpenWrt 开启 UPnP
opkg update
opkg install luci-app-upnp
# 然后在 LuCI → 服务 → UPnP 中启用
```

#### 方法三：使用 Exit Node 绕过 NAT

如果某台设备在公网（VPS），将其设为 Exit Node：

```bash
# 在公网设备上
sudo tailscale up --advertise-exit-node

# 在 NAT 后的设备上
sudo tailscale up --exit-node=<公网设备节点名>
```

### 5.6 内网穿透实战场景

#### 场景一：远程访问家里的 NAS

```
家里 OpenWrt 路由器 ─── Tailscale ─── 云端控制服务器
                                          │
公司电脑 ───────────── Tailscale ─────────┘

结果：公司电脑可以直接访问 192.168.1.x（家里局域网）
```

配置步骤：

1. OpenWrt 路由器上：
   ```bash
   tailscale up --advertise-routes=192.168.1.0/24 --hostname=home-router
   ```

2. 在管理后台启用子网路由

3. 公司电脑上：
   ```bash
   tailscale up --accept-routes
   # 现在可以直接访问 192.168.1.100（NAS）
   ```

#### 场景二：手机远程控制家里设备

```
Android 手机 (4G/5G 网络)
    │
    ├── Tailscale 连接
    │
    ▼
OpenWrt 路由器 (子网路由)
    │
    ├── 192.168.1.1   → 路由器管理界面
    ├── 192.168.1.100 → NAS
    ├── 192.168.1.101 → Home Assistant
    └── 192.168.1.102 → 打印机
```

#### 场景三：多个内网互联

```
北京办公室 (192.168.1.0/24)          上海办公室 (192.168.2.0/24)
    │                                      │
    ├── OpenWrt + Tailscale                ├── OpenWrt + Tailscale
    │   --advertise-routes=                │   --advertise-routes=
    │   192.168.1.0/24                     │   192.168.2.0/24
    │                                      │
    └─────────── Headscale 控制服务器 ───────┘

结果：两边局域网设备可以互通
```

> ⚠️ **注意**：确保两个局域网的 IP 网段不冲突（不要都用 192.168.1.0/24）。

---

## 第六部分：完整部署速查表

### 最简方案（5 分钟完成）

```bash
# 所有设备上
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
# 浏览器打开链接完成认证，搞定！
```

### 自建 Headscale 方案

```bash
# === 服务器端 ===
# 1. 部署 Headscale
mkdir -p /opt/headscale/{config,data}
# 创建 docker-compose.yml 和 config.yaml（见前文）
docker compose up -d

# 2. 创建用户和密钥
docker exec -it headscale headscale users create myuser
docker exec -it headscale headscale preauthkeys create --user myuser --reusable --expiration 720d

# 3. 配置 Nginx + HTTPS

# === 客户端 ===
# Linux
sudo tailscale up --login-server https://headscale.example.com --authkey <KEY>

# OpenWrt
opkg install tailscale
tailscale up --login-server https://headscale.example.com --authkey <KEY> --advertise-routes=192.168.1.0/24

# Android
# 方法一：App 设置中填自定义服务器
# 方法二：浏览器打开 headscale 注册链接
```

### 关键端口

| 端口 | 协议 | 用途 |
| ---- | ---- | ---- |
| 443 | TCP | HTTPS (控制服务器/DERP) |
| 8080 | TCP | Headscale HTTP API |
| 50443 | TCP | Headscale gRPC |
| 3478 | UDP | STUN (DERP) |
| 41641 | UDP | WireGuard 数据传输 |

---

## 第七部分：故障排查

### 常见问题

#### 1. 连接后无法互相访问

```bash
# 检查两端的 Tailscale IP
tailscale ip

# 检查连接状态
tailscale status

# 尝试 ping
tailscale ping <对端 IP 或主机名>

# 检查防火墙（Linux）
sudo iptables -L -n | grep tailscale
```

#### 2. 子网路由不生效

```bash
# 确认路由已广播
tailscale status

# 确认客户端已接受路由
tailscale up --accept-routes

# 在 Headscale 中启用路由
docker exec -it headscale headscale routes list
docker exec -it headscale headscale routes enable --route <CIDR> --node <ID>
```

#### 3. MagicDNS 不解析

```bash
# 检查 DNS 配置
tailscale dns status

# 确认 MagicDNS 已开启
# Tailscale: 在管理后台检查
# Headscale: config.yaml 中 magic_dns: true
```

#### 4. OpenWrt 上 Tailscale 启动失败

```bash
# 检查日志
logread -f | grep tailscale

# 常见原因：空间不足
df -h

# 手动启动查看错误
tailscaled --state /var/lib/tailscale/tailscaled.state
```

#### 5. Android 无法连接 Headscale

- 确认 HTTPS 证书有效（Let's Encrypt）
- 确认 `server_url` 配置正确
- 尝试在手机浏览器直接访问 `https://headscale.yourdomain.com/machine`
- 检查 App 版本是否支持自定义服务器

---

## 参考资源

- [Tailscale 官方文档](https://tailscale.com/kb)
- [Headscale GitHub](https://github.com/juanfont/headscale)
- [Headscale 官方文档](https://headscale.net/)
- [Tailscale Android 客户端](https://github.com/tailscale/tailscale-android)
- [DERP 自建指南](https://github.com/yangchuansheng/derper)
- [OpenWrt Tailscale Wiki](https://openwrt.org/docs/guide-user/services/vpn/tailscale/start)
