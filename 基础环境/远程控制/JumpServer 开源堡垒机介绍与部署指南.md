---
title: JumpServer 开源堡垒机介绍与部署指南
date: 2026-05-03
tags:
  - jumpserver
  - bastion-host
  - PAM
  - 堡垒机
  - 运维安全
  - docker
  - SSH
  - RDP
  - 开源
categories:
  - 基础环境
  - 远程控制
  - 网站安全
url: https://github.com/jumpserver/jumpserver
---

# JumpServer 开源堡垒机介绍与部署指南

> [!abstract] 概要
> JumpServer 是全球最受欢迎的开源特权访问管理（PAM）平台，即通常所说的"堡垒机"。它帮助企业以安全方式管控和登录所有类型的 IT 资产，实现**事前授权、事中监察、事后审计**，满足等保合规要求。本文将全面介绍 JumpServer 的功能特性、架构组件，并提供从服务端到客户端的完整部署指南。
>
> - **GitHub**: [jumpserver/jumpserver](https://github.com/jumpserver/jumpserver) ⭐ 30.4k Stars
> - **官网**: [jumpserver.com](https://jumpserver.com)
> - **最新版本**: v4.10.16-lts (2026-03-05)
> - **许可证**: GPL-3.0
> - **开发团队**: [FIT2CLOUD 飞致云](https://www.fit2cloud.com)

---

## 1 什么是 JumpServer？

JumpServer 是一个开源的**特权访问管理（Privileged Access Management, PAM）平台**，也就是业界常说的"堡垒机"或"跳板机"。它为 DevOps 和 IT 团队提供按需、安全地访问 SSH、RDP、Kubernetes、数据库和 RemoteApp 端点的能力，所有操作均可通过 Web 浏览器完成。

JumpServer 符合 **4A 规范**（认证 Authentication、授权 Authorization、账号 Account、审计 Audit），是企业 IT 运维安全审计的核心基础设施。

### 1.1 支持的资产类型

| 类别 | 支持内容 |
|------|----------|
| **SSH** | Linux / Unix / 网络设备等 |
| **Windows** | Web 方式连接 / 原生 RDP 连接 |
| **数据库** | MySQL / MariaDB / Oracle / SQLServer / PostgreSQL / ClickHouse 等 |
| **NoSQL** | Redis / MongoDB 等 |
| **云服务** | Kubernetes / VMware vSphere 等 |
| **Web 站点** | 各类系统的 Web 管理后台 |
| **应用** | 通过 Remote App 连接各类应用 |
| **AI 服务** | ChatGPT 等大模型接口 |

### 1.2 产品特色

- **开源免费**：零门槛，线上快速获取和安装，社区版功能完整
- **分布式架构**：轻松支持大规模并发访问
- **无插件**：仅需浏览器，极致的 Web Terminal 使用体验
- **多云支持**：一套系统同时管理不同云上的资产
- **云端存储**：审计录像云端存储，永不丢失
- **多租户**：一套系统支持多个子公司和部门同时使用
- **多应用支持**：数据库、Windows 远程应用、Kubernetes 全覆盖
- **合规审计**：满足等保 2.0 等合规要求

### 1.3 核心功能一览

| 功能 | 说明 |
|------|------|
| 资产管理 | 集中管理服务器、数据库、K8s、云资源等 |
| 用户与权限 | 基于角色的访问控制（RBAC），支持多组织多租户 |
| 会话管理 | 在线会话监控、实时阻断可疑操作 |
| 审计回放 | SSH 命令录像回放、RDP 录屏、文件传输审计 |
| 批量操作 | 批量命令执行、Ansible Playbook 执行 |
| MFA 认证 | TOTP、LDAP、OIDC、Duo、飞书等多因素认证 |
| API 集成 | RESTful API，便于与第三方系统对接 |

---

## 2 架构与组件

JumpServer 采用**微服务架构**，由多个关键组件协同工作：

```
┌─────────────────────────────────────────────────────────┐
│                    Nginx (反向代理)                        │
│                   统一入口 / SSL 终止                       │
├──────────┬──────────┬──────────┬──────────┬──────────────┤
│  Lina    │  Luna    │  KoKo    │  Lion    │    Chen      │
│ Web UI   │ Web终端   │ SSH连接器 │ RDP连接器 │  Web DB     │
├──────────┴──────────┴──────────┴──────────┴──────────────┤
│                      Core (核心后端)                       │
│              Django / Celery / Python                     │
├─────────────────────────────────────────────────────────┤
│         MySQL / MariaDB        │        Redis            │
│           数据存储               │      缓存/消息队列        │
└─────────────────────────────────────────────────────────┘
```

### 2.1 社区版组件

| 组件 | 说明 | GitHub |
|------|------|--------|
| **Core** | 后端核心，Django 框架，处理业务逻辑 | 内置 |
| **Lina** | Web 管理界面 UI | [jumpserver/lina](https://github.com/jumpserver/lina) |
| **Luna** | Web 终端，在线 SSH/RDP 会话 | [jumpserver/luna](https://github.com/jumpserver/luna) |
| **KoKo** | 字符协议连接器，处理 SSH/SFTP | [jumpserver/koko](https://github.com/jumpserver/koko) |
| **Lion** | 图形协议连接器，处理 RDP/VNC | [jumpserver/lion](https://github.com/jumpserver/lion) |
| **Chen** | Web 数据库管理终端 | [jumpserver/chen](https://github.com/jumpserver/chen) |
| **Client** | 桌面客户端（跨平台） | [jumpserver/client](https://github.com/jumpserver/client) |

### 2.2 企业版（EE）组件

| 组件 | 说明 |
|------|------|
| **Tinker** | 远程应用连接器（Windows） |
| **Panda** | 远程应用连接器（Linux） |
| **Razor** | RDP 代理连接器 |
| **Magnus** | 数据库代理连接器 |
| **Nec** | VNC 代理连接器 |
| **Facelive** | 面部识别认证 |

---

## 3 服务端部署

### 3.1 环境要求

| 项目 | 最低要求 | 推荐配置 |
|------|---------|---------|
| **操作系统** | Linux 64 位（CentOS 7+、Ubuntu 18.04+、Debian 10+） | Ubuntu 22.04 / Debian 12 |
| **CPU** | 4 核 | 8 核+ |
| **内存** | 8 GB | 16 GB+ |
| **磁盘** | 50 GB | 200 GB+ SSD |
| **Docker** | 20.10+ | 最新稳定版 |
| **Docker Compose** | v2.0+ | 最新稳定版 |

> [!warning] 注意
> 服务器必须是**干净**的 Linux 系统，不要在已有业务的服务器上部署，避免端口冲突。

### 3.2 一键快速部署（推荐）

这是官方推荐的部署方式，适合大多数场景：

```bash
# 1. 确保 Docker 已安装
docker --version
docker compose version

# 2. 执行一键部署脚本
curl -sSL https://github.com/jumpserver/jumpserver/releases/latest/download/quick_start.sh | bash
```

脚本会自动完成以下工作：
- 拉取所需 Docker 镜像
- 生成随机密钥和安全配置
- 创建 `docker-compose.yml`
- 启动所有服务组件

> [!tip] 部署完成后
> 访问 `http://<你的服务器IP>/` 即可打开 JumpServer 管理界面
> - 默认用户名：`admin`
> - 默认密码：`ChangeMe`
> - **首次登录后请立即修改默认密码！**

### 3.3 Docker Compose 手动部署

如果需要更多自定义控制，可以手动编写 `docker-compose.yml`：

```bash
# 1. 创建工作目录
mkdir -p /opt/jumpserver && cd /opt/jumpserver

# 2. 生成加密密钥
SECRET_KEY=$(cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 50)
BOOTSTRAP_TOKEN=$(cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 16)

echo "SECRET_KEY=$SECRET_KEY" > .env
echo "BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN" >> .env
```

创建 `docker-compose.yml`：

```yaml
version: '3.8'
services:
  mysql:
    image: mysql:8.0
    container_name: jms_mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD:-jumpserver}
      MYSQL_DATABASE: jumpserver
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - net

  redis:
    image: redis:7-alpine
    container_name: jms_redis
    restart: always
    volumes:
      - redis-data:/data
    networks:
      - net

  core:
    image: jumpserver/jms_core:${JMS_VERSION:-v4.10.16-lts}
    container_name: jms_core
    restart: always
    environment:
      SECRET_KEY: ${SECRET_KEY}
      BOOTSTRAP_TOKEN: ${BOOTSTRAP_TOKEN}
      DB_HOST: mysql
      DB_PORT: 3306
      DB_USER: root
      DB_PASSWORD: ${MYSQL_ROOT_PASSWORD:-jumpserver}
      REDIS_HOST: redis
      REDIS_PORT: 6379
    depends_on:
      - mysql
      - redis
    volumes:
      - core-data:/opt/jumpserver/data
    networks:
      - net

  koko:
    image: jumpserver/jms_koko:${JMS_VERSION:-v4.10.16-lts}
    container_name: jms_koko
    restart: always
    environment:
      CORE_HOST: http://core:8080
      BOOTSTRAP_TOKEN: ${BOOTSTRAP_TOKEN}
    depends_on:
      - core
    ports:
      - "2222:2222"   # SSH 端口
    networks:
      - net

  lion:
    image: jumpserver/jms_lion:${JMS_VERSION:-v4.10.16-lts}
    container_name: jms_lion
    restart: always
    environment:
      CORE_HOST: http://core:8080
      BOOTSTRAP_TOKEN: ${BOOTSTRAP_TOKEN}
    depends_on:
      - core
    networks:
      - net

  chen:
    image: jumpserver/jms_chen:${JMS_VERSION:-v4.10.16-lts}
    container_name: jms_chen
    restart: always
    environment:
      CORE_HOST: http://core:8080
      BOOTSTRAP_TOKEN: ${BOOTSTRAP_TOKEN}
    depends_on:
      - core
    networks:
      - net

  web:
    image: jumpserver/jms_web:${JMS_VERSION:-v4.10.16-lts}
    container_name: jms_web
    restart: always
    ports:
      - "80:80"       # HTTP
      - "443:443"     # HTTPS（可选）
    depends_on:
      - core
    networks:
      - net

volumes:
  mysql-data:
  redis-data:
  core-data:

networks:
  net:
    driver: bridge
```

启动服务：

```bash
# 启动所有容器
docker compose up -d

# 查看运行状态
docker compose ps

# 查看日志
docker compose logs -f core
```

### 3.4 配置 HTTPS（生产环境必须）

```bash
# 使用 Let's Encrypt 获取免费 SSL 证书
apt install certbot
certbot certonly --standalone -d jumpserver.yourdomain.com

# 证书路径
# 证书: /etc/letsencrypt/live/jumpserver.yourdomain.com/fullchain.pem
# 私钥: /etc/letsencrypt/live/jumpserver.yourdomain.com/privkey.pem
```

在 Nginx 配置中添加 SSL：

```nginx
server {
    listen 443 ssl http2;
    server_name jumpserver.yourdomain.com;

    ssl_certificate     /etc/letsencrypt/live/jumpserver.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/jumpserver.yourdomain.com/privkey.pem;
    ssl_protocols       TLSv1.2 TLSv1.3;

    client_max_body_size 5000m;

    location / {
        proxy_pass http://web:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

## 4 桌面客户端部署

JumpServer 提供了跨平台的桌面客户端，支持 Windows、macOS 和 Linux，方便运维人员通过本地工具（如 SSH 客户端、RDP 客户端、数据库客户端）连接资产。

> **客户端最新版本**: v4.1.2（2026 年最新稳定版）
> **下载地址**: [github.com/jumpserver/client/releases](https://github.com/jumpserver/client/releases)

### 4.1 客户端功能

- **SSH 连接**：自动调用本地 SSH 客户端（Terminal / PuTTY / Xshell）
- **RDP 连接**：自动调用本地 RDP 客户端连接 Windows 资产
- **数据库连接**：支持通过 Navicat、DBeaver 等工具连接数据库
- **VNC 连接**：支持 VNC 协议连接
- **OAuth 2.0 认证**：v4.x 版本支持标准 OAuth 2.0 登录流程
- **站点历史**：保存多个 JumpServer 站点信息，快速切换

### 4.2 Windows 安装

```powershell
# 下载 Windows 安装包
# 访问 https://github.com/jumpserver/client/releases/latest
# 下载 JumpServer-Client-Setup-x64.exe

# 或使用 winget 安装（如果已收录）
winget install JumpServer.Client
```

安装步骤：
1. 下载 `JumpServer-Client-Setup-x64.exe`
2. 双击运行安装向导
3. 选择安装路径，完成安装
4. 启动客户端，输入 JumpServer 服务端地址
5. 使用账号密码或 OAuth 2.0 登录

### 4.3 macOS 安装

```bash
# 方式一：下载 DMG 安装包
# 访问 https://github.com/jumpserver/client/releases/latest
# 下载 JumpServer-Client-{version}-mac.dmg

# 方式二：使用 Homebrew Cask（如已收录）
brew install --cask jumpserver-client
```

安装步骤：
1. 下载 `.dmg` 文件
2. 打开 DMG，将 JumpServer 拖入 Applications 文件夹
3. 首次打开需在"系统偏好设置 → 安全性与隐私"中允许运行
4. 配置本地 SSH / VNC 客户端路径

### 4.4 Linux 安装

```bash
# 下载 AppImage 或 deb/rpm 包
# 访问 https://github.com/jumpserver/client/releases/latest

# Ubuntu / Debian
sudo dpkg -i jumpserver-client_x.x.x_amd64.deb

# CentOS / RHEL
sudo rpm -ivh jumpserver-client-x.x.x.x86_64.rpm

# 或使用 AppImage（无需安装）
chmod +x JumpServer-Client-x86_64.AppImage
./JumpServer-Client-x86_64.AppImage
```

### 4.5 客户端配置要点

登录客户端后，需要配置本地工具路径：

| 配置项 | 说明 |
|--------|------|
| **SSH 终端** | 设置本地终端模拟器路径（如 `/usr/bin/ssh`、PuTTY） |
| **RDP 客户端** | 设置远程桌面客户端（如 Windows 自带 mstsc、macOS 上 Microsoft Remote Desktop） |
| **数据库客户端** | 设置 Navicat / DBeaver 等数据库管理工具路径 |
| **VNC 客户端** | 设置 VNC 查看器路径（如 TigerVNC、RealVNC） |

> [!info] 客户端版本兼容性
> - Client v4.x 需要 JumpServer 服务端 **v4.10 及以上**版本
> - Client v3.x 兼容 JumpServer 服务端 **v3.x - v4.x** 版本
> - 建议客户端和服务端版本保持同步更新

---

## 5 基础使用流程

### 5.1 初始化配置

```
登录管理后台 → 系统设置 → 基本设置
├── 配置邮件服务器（用于用户通知和 MFA）
├── 配置 LDAP / OIDC 认证源（可选）
├── 配置存储后端（录像存储）
└── 配置短信 / 飞书等消息通道（可选）
```

### 5.2 添加资产

```
资产管理 → 资产列表 → 创建资产
├── 填写主机名、IP、端口
├── 选择资产平台（Linux/Windows/Database 等）
├── 配置系统用户（用于连接资产的账号）
└── 测试连接
```

### 5.3 授权访问

```
权限管理 → 资产授权 → 创建授权规则
├── 选择用户 / 用户组
├── 选择资产 / 节点
├── 选择系统用户
├── 设置权限（连接、上传、下载等）
└── 设置有效期
```

### 5.4 用户连接资产

用户可以通过三种方式连接被授权的资产：

1. **Web Terminal**（Luna）：浏览器直接访问，无需任何客户端
2. **SSH 客户端**：通过 `ssh user@jumpserver-ip -p 2222` 连接
3. **桌面客户端**：使用 JumpServer Client 连接

---

## 6 安全最佳实践

> [!danger] 安全提醒
> JumpServer 本身就是安全产品，其自身的安全至关重要。请务必遵循以下安全建议。

### 6.1 服务端安全

| 措施 | 说明 |
|------|------|
| **修改默认密码** | 首次登录后立即修改 admin 默认密码 |
| **启用 HTTPS** | 生产环境必须使用 SSL/TLS 加密 |
| **启用 MFA** | 为所有管理员启用多因素认证 |
| **限制访问来源** | 使用防火墙限制 Web 和 SSH 端口的访问 IP |
| **定期更新** | 关注 [安全公告](https://github.com/jumpserver/jumpserver/security)，及时修补漏洞 |
| **备份策略** | 定期备份 MySQL 数据库和 Redis 数据 |
| **安全密钥管理** | 妥善保管 `SECRET_KEY` 和 `BOOTSTRAP_TOKEN` |

### 6.2 网络安全

```bash
# 仅允许可信 IP 访问 Web 界面
ufw allow from 10.0.0.0/8 to any port 80
ufw allow from 10.0.0.0/8 to any port 443

# SSH 访问端口同样需要限制
ufw allow from 10.0.0.0/8 to any port 2222

# 启用防火墙
ufw enable
```

### 6.3 已知安全漏洞（需关注）

- **CVE-2025-62712**（2025-10-30 披露）：已修复，升级到 v4.10.14+ 即可
- **CVE-2025-62795**（2025-10-30 披露）：已修复，升级到 v4.10.14+ 即可
- 建议始终使用最新 LTS 版本

---

## 7 版本更新记录（近期）

| 版本 | 日期 | 重要更新 |
|------|------|---------|
| **v4.10.16-lts** | 2026-03-05 | 新增可信客户端 IP 验证、YAML 模板沙箱渲染、SSL 证书验证修复 |
| **v4.10.15-lts** | 2026-02 | API 限流、IPv6 网络测试、Windows 密码验证增强 |
| **v4.10.14-lts** | 2026-01 | Client OAuth 2.0 支持、审计日志导出、K8s 命名空间限制 |
| **v4.10.13-lts** | 2025-12 | Bug 修复和性能优化 |

---

## 8 常见问题

### Q1: 部署时内存不足怎么办？

JumpServer 最低要求 8GB 内存。如果资源有限，可以：
- 限制 MySQL 和 Redis 的内存使用
- 只部署必要组件（Core + KoKo + Web）
- 使用 swap 分区临时缓解

### Q2: 如何升级到最新版本？

```bash
cd /opt/jumpserver
# 备份数据
docker compose exec core bash -c './jmsctl.sh backup_db'

# 拉取最新镜像
docker compose pull

# 重启服务
docker compose down && docker compose up -d
```

### Q3: 忘记管理员密码怎么办？

```bash
# 进入 Core 容器重置密码
docker compose exec core bash
cd /opt/jumpserver
python manage.py changepassword admin
```

### Q4: SSH 连接不上资产？

检查以下项目：
1. 资产的网络是否可达
2. 系统用户的凭据是否正确
3. 资产上 SSH 服务是否正常运行
4. 防火墙是否放行了 2222 端口

---

## 9 相关资源

| 资源 | 链接 |
|------|------|
| **GitHub 仓库** | [github.com/jumpserver/jumpserver](https://github.com/jumpserver/jumpserver) |
| **官方文档** | [docs.jumpserver.org](https://docs.jumpserver.org) |
| **官方网站** | [jumpserver.com](https://jumpserver.com) |
| **客户端下载** | [github.com/jumpserver/client/releases](https://github.com/jumpserver/client/releases) |
| **社区论坛** | [community.fit2cloud.com](https://community.fit2cloud.com) |
| **Grafana 仪表盘** | [jumpserver-grafana-dashboard](https://github.com/jumpserver/jumpserver-grafana-dashboard) |
| **安全漏洞报告** | support@fit2cloud.com |

---

> [!quote] 关于 JumpServer
> JumpServer 由 [FIT2CLOUD 飞致云](https://www.fit2cloud.com) 团队开发维护，自 2014 年开源至今，已成为全球最受欢迎的开源堡垒机项目。项目采用 GPL-3.0 许可证，社区版功能完整可免费使用，企业版提供更多高级功能（如数据库代理、VNC 代理、面部识别等）。
