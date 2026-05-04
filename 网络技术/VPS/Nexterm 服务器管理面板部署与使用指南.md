---
title: Nexterm 服务器管理面板部署与使用指南
date: 2026-05-03
tags:
  - Nexterm
  - 服务器管理
  - SSH
  - VNC
  - RDP
  - Docker
  - 开源
  - 自托管
  - SFTP
  - Proxmox
sources:
  - https://github.com/gnmyt/Nexterm
  - https://docs.nexterm.dev
---

# Nexterm 服务器管理面板部署与使用指南

## 概述

[Nexterm](https://github.com/gnmyt/Nexterm) 是一款开源的服务器管理软件，提供 Web 界面集中管理你的所有服务器。它将 SSH 终端、远程桌面（VNC/RDP）、文件管理（SFTP）、Docker 部署和 Proxmox 容器管理整合到一个统一的现代界面中。

> [!info] 项目信息
> - **作者**: Mathias Wagner ([@gnmyt](https://github.com/gnmyt))
> - **许可证**: MIT
> - **Stars**: 4.3k+
> - **最新版本**: 1.2.0-BETA (2026-01-01)
> - **技术栈**: JavaScript 66.9% / Sass 12.8% / Dart 11.6% / C 6.5% / Rust 1.6%

---

## 核心功能

### 远程连接

| 协议 | 说明 |
|------|------|
| **SSH** | 浏览器内完整的终端访问 |
| **VNC** | 基于 Web 的远程桌面 |
| **RDP** | 远程桌面协议支持 |
| **SFTP** | 图形化文件管理器 |

### 基础设施管理

- **Docker 部署** — 直接通过 Web 界面部署和管理容器
- **Proxmox 管理** — 管理 LXC 和 QEMU 虚拟机
- **服务器监控** — 实时 CPU、内存和进程指标
- **会话录制** — 录制终端操作，便于审计和回放

### 安全特性

- **双因素认证 (2FA)** — 登录保护
- **OIDC SSO** — 支持微软 Entra ID、Google、Keycloak、Authentik、Authelia 等
- **LDAP/AD** — 对接企业目录服务
- **会话管理** — 管理活跃会话
- **密码加密** — 所有凭据加密存储
- **组织管理** — 将用户和服务器分组到不同组织，实现权限隔离

### 自动化与效率

- **Snippets** — 常用命令片段，一键粘贴到终端
- **Scripts** — 可执行脚本，批量自动化运维任务
- **CLI 工具** — 命令行客户端 `nt`，终端内直接连接服务器
- **端口转发** — 本地端口转发，安全访问内网服务
- **AI 辅助** — 可配置 AI 系统提示词，辅助生成命令

---

## 部署指南

### 前置条件

- 一台 Linux 服务器（推荐 Ubuntu/Debian）
- Docker 和 Docker Compose 已安装
- 开放端口 6989（HTTP）和/或 5878（HTTPS）

### 生成加密密钥

Nexterm 需要一个加密密钥来安全存储密码和 SSH 密钥。首先生成一个强密钥：

```bash
openssl rand -hex 32
```

> [!warning] 请妥善保存这个密钥
> 丢失密钥将导致所有已存储的密码和 SSH 密钥无法解密。

### 方式一：Docker 一键部署（推荐）

#### Docker Run

使用 Host 网络模式（推荐，支持 Wake-on-LAN 和 localhost 连接）：

```bash
docker run -d \
  -e ENCRYPTION_KEY=你生成的密钥 \
  --network host \
  --name nexterm \
  --restart always \
  -v nexterm:/app/data \
  nexterm/aio:latest
```

使用 Bridge 网络模式（需要网络隔离时使用）：

```bash
docker run -d \
  -e ENCRYPTION_KEY=你生成的密钥 \
  -p 6989:6989 \
  --name nexterm \
  --restart always \
  -v nexterm:/app/data \
  nexterm/aio:latest
```

#### Docker Compose（推荐）

创建 `docker-compose.yml`：

```yaml
services:
  nexterm:
    image: nexterm/aio:latest
    environment:
      ENCRYPTION_KEY: "你生成的密钥"
    network_mode: host
    restart: always
    volumes:
      - nexterm:/app/data

volumes:
  nexterm:
```

启动：

```bash
docker compose up -d
```

> [!tip] Host 网络模式 vs Bridge 模式
> Host 模式让 Nexterm 直接访问宿主机的网络栈，这对于 Wake-on-LAN 和通过 localhost 连接服务器等功能是必需的。仅在确实需要网络隔离时才使用 Bridge 模式。

### 方式二：分离式部署（高级）

适用于需要跨网络管理服务器或运行多个引擎的场景。

**架构说明**：

| 组件 | 镜像 | 说明 |
|------|------|------|
| Server | `nexterm/server` | Node.js 后端 + Web 客户端 |
| Engine | `nexterm/engine` | 连接服务（SSH、VNC、RDP、Telnet） |
| AIO | `nexterm/aio` | Server + Engine 合一 |

首先为 Engine 创建 `config.yaml`：

```yaml
server_host: "server"
server_port: 7800
registration_token: ""
```

然后创建 `docker-compose.yml`：

```yaml
services:
  server:
    image: nexterm/server:latest
    environment:
      ENCRYPTION_KEY: "你生成的密钥"
    ports:
      - "6989:6989"
    restart: always
    volumes:
      - nexterm:/app/data

  engine:
    image: nexterm/engine:latest
    restart: always
    volumes:
      - ./config.yaml:/etc/nexterm/config.yaml

volumes:
  nexterm:
```

```bash
docker compose up -d
```

> [!example] 典型场景
> 你在公司内网部署 Server，在各分支机构或云平台的网络上部署 Engine 实例。Server 通过 TCP 7800 端口与各 Engine 通信，用户只需访问 Server 的 Web 界面即可管理所有网络中的服务器。

### IPv6 支持

使用 Bridge 网络模式时，如需连接 IPv6 服务器，需添加以下配置（Host 模式不需要）：

```yaml
services:
  nexterm:
    networks:
      - nexterm-net

networks:
  nexterm-net:
    enable_ipv6: true
```

---

## SSL/HTTPS 配置

### 自签名证书（测试用）

```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
```

将 `cert.pem` 和 `key.pem` 放入 `data/certs` 目录，Nexterm 会自动启用 HTTPS。

### Let's Encrypt（生产推荐）

```bash
sudo certbot certonly --standalone -d yourdomain.com
cp /etc/letsencrypt/live/yourdomain.com/fullchain.pem ./data/certs/cert.pem
cp /etc/letsencrypt/live/yourdomain.com/privkey.pem ./data/certs/key.pem
```

### Docker Compose 配置

```yaml
services:
  nexterm:
    environment:
      HTTPS_PORT: 5878  # 可选，默认值
    ports:
      - "5878:5878"
    volumes:
      - ./certs:/app/data/certs
```

| 端口 | 默认值 | 说明 |
|------|--------|------|
| HTTP | 6989 | Web 访问端口 |
| HTTPS | 5878 | SSL 端口（通过 `HTTPS_PORT` 环境变量修改） |

---

## 反向代理配置

> [!important] 所有反向代理都必须启用 WebSocket 支持，否则终端和远程桌面将无法工作。

### Nginx

```nginx
server {
    listen 80;
    server_name nexterm.yourdomain.com;

    location / {
        proxy_pass http://127.0.0.1:6989;
        proxy_http_version 1.1;

        # WebSocket 支持（必须）
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_read_timeout 86400;
    }
}
```

### Caddy（最简方案）

```caddy
nexterm.yourdomain.com {
    reverse_proxy 127.0.0.1:6989
}
```

> Caddy 自动处理 WebSocket 和 SSL 证书。

### Traefik（Docker 集成）

```yaml
services:
  nexterm:
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.nexterm.rule=Host(`nexterm.yourdomain.com`)"
      - "traefik.http.routers.nexterm.entrypoints=websecure"
      - "traefik.http.routers.nexterm.tls.certresolver=letsencrypt"
      - "traefik.http.services.nexterm.loadbalancer.server.port=6989"
```

### Cloudflare Tunnel（无需开放端口）

通过 Cloudflare Tunnel 暴露 Nexterm，无需在防火墙开放任何入站端口：

1. 登录 Cloudflare Zero Trust → **Networks → Connectors**
2. 创建 Tunnel → 选择 **Cloudflared** → 命名（如 `nexterm`）
3. 配置 Public Hostname：
   - **Subdomain**: `nexterm`
   - **Type**: `HTTP`
   - **URL**: `localhost:6989`
4. 保存后即可通过 `https://nexterm.yourdomain.com` 访问

---

## 认证集成

### OIDC 单点登录 (SSO)

进入 **Settings → Authentication → Add Provider**，填写以下信息：

| 字段 | 说明 |
|------|------|
| Display Name | 登录按钮上显示的名称 |
| Issuer URL | IdP 的发现 URL |
| Client ID | 来自你的 IdP |
| Client Secret | 来自你的 IdP |
| Redirect URI | 复制到你的 IdP（格式：`https://nexterm.yourdomain.com/api/auth/oidc/callback`） |
| Scope | 通常填 `openid profile` |

#### 常见 IdP 配置

**Microsoft Entra ID (Azure AD)**：

| 配置项 | 值 |
|--------|-----|
| Issuer URL | `https://login.microsoftonline.com/{tenant-id}/v2.0` |
| Redirect URI | `https://nexterm.yourdomain.com/api/auth/oidc/callback` |

**Google**：

| 配置项 | 值 |
|--------|-----|
| Issuer URL | `https://accounts.google.com` |
| Redirect URI | `https://nexterm.yourdomain.com/api/auth/oidc/callback` |

> [!note] Google 需要在 OAuth consent screen 中添加测试用户，生产环境需要通过应用验证。

**Keycloak**：

| 配置项 | 值 |
|--------|-----|
| Issuer URL | `https://keycloak.yourdomain.com/realms/{realm-name}` |

**Authentik**：

| 配置项 | 值 |
|--------|-----|
| Issuer URL | `https://authentik.yourdomain.com/application/o/{application-slug}/` |

> [!tip] Issuer URL 末尾的斜杠很重要！访问 `/.well-known/openid-configuration` 可以查看精确的 issuer 值。

### LDAP / Active Directory

进入 **Settings → Authentication → Add LDAP**。

**Active Directory 示例**：

```
Host: dc01.corp.example.com
Port: 636
Bind DN: CN=svc_nexterm,CN=Users,DC=corp,DC=example,DC=com
Base DN: CN=Users,DC=corp,DC=example,DC=com
User Search Filter: (sAMAccountName={{username}})
Use TLS: 启用
```

**OpenLDAP 示例**：

```
Host: ldap.example.com
Port: 389
Bind DN: cn=readonly,dc=example,dc=com
Base DN: ou=users,dc=example,dc=com
User Search Filter: (uid={{username}})
```

> [!tip] 保存后点击 **Test Connection** 验证绑定凭据是否正确。

---

## 环境变量参考

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `SERVER_PORT` | `6989` | Web 服务监听端口 |
| `HTTPS_PORT` | `5878` | HTTPS 端口 |
| `CONTROL_PLANE_PORT` | `7800` | Server 与 Engine 通信的 TCP 端口 |
| `NODE_ENV` | - | 运行环境（`development` / `production`） |
| `ENCRYPTION_KEY` | - | 密码、SSH 密钥的加密密钥（支持 Docker secrets） |
| `AI_SYSTEM_PROMPT` | - | AI 功能的系统提示词 |
| `LOG_LEVEL` | `system` | 日志级别（`system`/`info`/`verbose`/`debug`/`warn`/`error`） |

---

## CLI 工具使用

Nexterm 提供命令行工具 `nt`，可直接在终端中连接服务器，无需打开浏览器。

### 安装

```bash
cd cli
cargo build --release
cp cli/target/release/nt /usr/local/bin/
```

### 登录

```bash
nt login
```

会提示输入 Server URL，支持一次性验证码或浏览器自动跳转。

### 常用命令

```bash
# 列出所有服务器（树形结构）
nt ls

# 按文件夹/标签筛选
nt ls --folder "Home-Lab"
nt ls --tag "production"

# 输出 JSON 格式
nt ls --json

# 连接到服务器（支持 ID、名称、模糊匹配）
nt connect my-server
nt connect 8

# 在远程服务器上执行单条命令
nt connect my-server -- uptime
nt connect 8 -- "df -h"

# 模糊搜索服务器并连接
nt search "mail"
nt search "gateway" -- "systemctl status nginx"

# 从最近列表中选择连接
nt recent

# 端口转发（类似 ssh -L）
nt forward my-server --local 8080 --port 3000

# 转发到远程网络中的其他主机
nt forward my-server --local 5432 --remote 10.0.0.5 --port 5432

# 查看配置
nt config show

# 修改配置
nt config set server-url https://nexterm.example.com

# 登出
nt logout
```

| 配置项 | 说明 |
|--------|------|
| `server-url` | Nexterm 服务器地址 |
| `accept-invalid-certs` | 设为 `true` 允许自签名 SSL 证书 |

配置文件位置：`~/.config/nexterm/config.json`

---

## Scripts & Snippets 自动化

Nexterm 使用文件扩展名区分脚本和代码片段：

| 类型 | 扩展名 | 说明 |
|------|--------|------|
| **Snippet** | `.snippet`, `.txt`, `.cmd` | 快速命令，粘贴到终端执行 |
| **Script** | `.sh`, `.bash`, `.zsh`, `.fish`, `.ps1` | 完整可执行脚本 |

### 文件格式

文件顶部使用注释定义元数据：

```bash
# @name: 查找最大文件
# @description: 找出系统中最大的 10 个文件
# @os: Ubuntu, Debian, Fedora

find / -type f -exec ls -lh {} + | sort -k5 -h | tail -10
```

```bash
# @name: 更新所有包
# @description: 更新和升级所有软件包
# @os: Ubuntu, Debian

sudo apt update && sudo apt upgrade -y
```

```bash
# @name: 备份所有虚拟机
# @description: 创建所有运行中虚拟机的快照备份
# @os: Proxmox VE

vzdump --all --mode snapshot --compress zstd
```

### 支持的标签

| 标签 | 说明 |
|------|------|
| `@name` | 在 Nexterm UI 中显示的名称 |
| `@description` | 命令功能的说明 |
| `@os` | 兼容的操作系统（逗号分隔） |

支持的 OS 值：`Ubuntu`, `Debian`, `Alpine Linux`, `Fedora`, `CentOS`, `Red Hat`, `Rocky Linux`, `AlmaLinux`, `openSUSE`, `Arch Linux`, `Manjaro`, `Gentoo`, `NixOS`, `Proxmox VE`

> [!tip] 没有 `@os` 标签的 Snippet 会在所有系统上显示。

---

## 从源码开发

### 前置条件

- Node.js 18+
- Yarn
- Docker（可选）

### 本地开发

```bash
# 克隆仓库
git clone https://github.com/gnmyt/Nexterm.git
cd Nexterm

# 安装依赖
yarn install
cd client && yarn install
cd ..

# 启动开发模式
yarn dev
```

---

## 常见问题排查

| 问题 | 解决方案 |
|------|----------|
| WebSocket 连接失败 | 检查反向代理是否启用了 WebSocket 升级 |
| OIDC 回调不匹配 | 确保 Redirect URI 完全匹配，注意末尾斜杠和 http/https |
| LDAP 用户无法登录 | 检查 Base DN 和 Search Filter，尝试 `(&(objectClass=person)(uid={{username}}))` |
| LDAP ECONNREFUSED | 检查主机、端口和防火墙 |
| LDAP INVALID_CREDENTIALS | 检查 Bind DN 和密码 |
| 容器无法连接 IPv6 服务器 | 在 Bridge 模式下启用 `enable_ipv6: true`（Host 模式无需配置） |
| HTTPS 不生效 | 确认 `cert.pem` 和 `key.pem` 已放入 `data/certs` 目录 |

---

## 有用链接

| 资源 | 链接 |
|------|------|
| 官方文档 | [docs.nexterm.dev](https://docs.nexterm.dev) |
| GitHub 仓库 | [github.com/gnmyt/Nexterm](https://github.com/gnmyt/Nexterm) |
| Discord 社区 | [加入 Discord](https://discord.gg/nexterm) |
| 报告 Bug | [GitHub Issues](https://github.com/gnmyt/Nexterm/issues) |
| 功能建议 | [GitHub Discussions](https://github.com/gnmyt/Nexterm/discussions) |
| NexStore（脚本仓库） | 查看 NexStore 获取更多脚本示例 |

---

> [!note] 项目状态
> Nexterm 目前处于 Beta 阶段（1.2.0-BETA），建议定期备份数据，遇到问题及时在 [GitHub Issues](https://github.com/gnmyt/Nexterm/issues) 反馈。
