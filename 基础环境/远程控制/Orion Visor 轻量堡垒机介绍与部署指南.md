---
title: Orion Visor 轻量堡垒机介绍与部署指南
date: 2026-05-03
tags:
  - orion-visor
  - bastion-host
  - 堡垒机
  - 运维
  - docker
  - SSH
  - RDP
  - VNC
  - 开源
  - dromara
categories:
  - 基础环境
  - 远程控制
url: https://github.com/dromara/orion-visor
---

# Orion Visor 轻量堡垒机介绍与部署指南

> [!abstract] 概要
> Orion Visor 是一款由 Dromara 开源社区孵化的**高颜值、轻量级自动化运维 & 轻量堡垒机平台**。它提供一站式自动化运维解决方案，涵盖资产管理、在线终端（SSH/RDP/VNC）、文件管理（SFTP）、批量操作、计划任务、系统监控等核心能力。相较于 [[JumpServer 开源堡垒机介绍与部署指南|JumpServer]] 等重量级方案，Orion Visor 以**简洁、轻量、实用**为设计理念，非常适合个人开发者和小型团队快速上手。
>
> - **GitHub**: [dromara/orion-visor](https://github.com/dromara/orion-visor) ⭐ 1.2k Stars
> - **镜像仓库**: [Gitee](https://gitee.com/dromara/orion-visor) | [AtomGit](https://atomgit.com/dromara/orion-visor)
> - **演示环境**: [https://dv.orionsec.cn/](https://dv.orionsec.cn/) (账号: `admin` / 密码: `admin`)
> - **最新版本**: v2.5.7
> - **许可证**: Apache-2.0
> - **作者**: lijiahangmax

---

## 1 什么是 Orion Visor？

Orion Visor 是一个开箱即用的**轻量级运维面板 & 堡垒机**，主要面向个人开发者和小型团队。它不追求大型运维平台（如 JumpServer）的复杂性和企业级特性，而是聚焦于**核心运维场景的简洁高效实现**。

与 JumpServer 的定位对比：

| 维度 | Orion Visor | JumpServer |
|------|------------|------------|
| **定位** | 轻量级运维面板 & 轻量堡垒机 | 企业级 PAM / 堡垒机 |
| **目标用户** | 个人开发者、小团队 | 中大型企业 |
| **部署复杂度** | ⭐ 极简（几条命令） | ⭐⭐⭐ 较复杂 |
| **资源占用** | 低（2GB 内存即可运行） | 高（推荐 8GB+） |
| **协议支持** | SSH / RDP / VNC / SFTP | SSH / RDP / VNC / DB / K8s / Web |
| **4A 合规** | 基础审计 | 完整 4A（认证/授权/账号/审计） |
| **许可证** | Apache-2.0 | GPL-3.0 |

> [!tip] 选择建议
> - 个人管理几台服务器 → **Orion Visor**（轻量快速）
> - 企业等保合规需求、大规模资产 → **JumpServer**（功能全面）

### 1.1 核心功能一览

| 功能 | 说明 |
|------|------|
| **资产管理** | 支持资产分组，对主机、密钥和身份的统一管理和授权 |
| **在线终端** | 支持 SSH、RDP、VNC 多种协议，快捷命令、自定义快捷键和主题风格 |
| **文件管理** | 远程主机 SFTP 大文件的批量上传、下载和在线编辑 |
| **批量操作** | 批量执行主机命令、多主机文件分发 |
| **计划任务** | 配置 cron 表达式，定时执行主机命令 |
| **系统监控** | 监控主机 CPU、内存、磁盘、网络等指标，支持告警 |
| **安全审计** | 动态权限配置，记录用户操作日志，提供简单审计功能 |
| **用户管理** | 多用户支持，在线用户管理，锁定用户管理 |
| **标签系统** | 系统标签管理，灵活分类资产 |

### 1.2 支持的连接协议

| 协议 | 用途 | 实现方式 |
|------|------|---------|
| **SSH** | Linux / Unix 服务器远程终端 | JSch / Apache MINA SSHD |
| **RDP** | Windows 服务器远程桌面 | Apache Guacamole (guacd) |
| **VNC** | 图形化远程桌面 | Apache Guacamole (guacd) |
| **SFTP** | 文件传输与管理 | 内置 SFTP 客户端 |

---

## 2 架构与组件

Orion Visor 采用**容器化微服务架构**，所有组件通过 Docker Compose 编排，网络互通。

```
┌──────────────────────────────────────────────────────────┐
│                     Nginx (UI 容器)                        │
│                  前端静态资源 / 反向代理                      │
│                    端口: 1081                              │
├──────────────────────────────────────────────────────────┤
│                  Service (后端服务)                         │
│            SpringBoot 2.7+ / Java 后端                     │
│                 端口: 9200                                 │
├────────┬──────────┬───────────┬──────────┬────────────────┤
│ MySQL  │  Redis   │ InfluxDB  │  Guacd   │    Adminer     │
│ 数据存储 │ 缓存/会话  │ 监控指标存储 │ 远程桌面代理 │  数据库管理    │
│ :3306  │  :6379   │  :8086    │  :4822   │    :8081       │
└────────┴──────────┴───────────┴──────────┴────────────────┘
```

### 2.1 组件说明

| 组件 | 说明 | 镜像 |
|------|------|------|
| **ui** | 前端 Vue 3 + Nginx 反向代理 | `orion-visor-ui` |
| **service** | SpringBoot 后端核心服务 | `orion-visor-service` |
| **mysql** | MySQL 8.0+ 关系型数据库 | `orion-visor-mysql` |
| **redis** | Redis 6.0+ 缓存与消息队列 | `orion-visor-redis` |
| **influxdb** | InfluxDB 2.7+ 时序数据库（监控指标） | `orion-visor-influxdb` |
| **guacd** | Apache Guacamole 远程桌面代理 | `orion-visor-guacd` |
| **adminer** | 数据库管理工具（可选） | `orion-visor-adminer` |

### 2.2 后端模块结构

```
orion-visor-modules/
├── orion-visor-module-asset      # 资产管理模块
├── orion-visor-module-common     # 公共模块
├── orion-visor-module-exec       # 批量执行模块
├── orion-visor-module-infra      # 基础设施模块
├── orion-visor-module-monitor    # 系统监控模块
└── orion-visor-module-terminal   # 终端连接模块
```

### 2.3 技术栈

| 层级 | 技术 |
|------|------|
| **后端框架** | SpringBoot 2.7+ |
| **前端框架** | Vue 3.5+ |
| **UI 组件库** | Arco Design 2.56+ |
| **关系数据库** | MySQL 8.0+ |
| **缓存** | Redis 6.0+ |
| **时序数据库** | InfluxDB 2.7+ |
| **远程桌面** | Apache Guacamole (guacd) |
| **SSH 库** | JSch / Apache MINA SSHD |

---

## 3 部署指南

### 3.1 环境要求

| 项目 | 最低要求 | 推荐配置 |
|------|---------|---------|
| **操作系统** | Linux 64 位（Ubuntu 20.04+、Debian 11+、CentOS 7+） | Ubuntu 22.04 / Debian 12 |
| **CPU** | 2 核 | 4 核+ |
| **内存** | 2 GB | 4 GB+ |
| **磁盘** | 20 GB | 50 GB+ SSD |
| **Docker** | 20.10+ | 最新稳定版 |
| **Docker Compose** | v2.0+ | 最新稳定版 |

> [!tip] 资源优势
> Orion Visor 的资源占用远低于 JumpServer（推荐 8GB+ 内存），仅需 **2GB 内存** 即可流畅运行，非常适合轻量级部署场景。

### 3.2 Docker Compose 一键部署（推荐）

这是最简单的部署方式，仅需几条命令：

```bash
# 1. 克隆仓库
git clone --depth=1 https://github.com/dromara/orion-visor
cd orion-visor

# 2. 复制配置文件
cp .env.example .env

# 3. 修改配置（重要！）
vim .env

# 4. 启动所有服务
docker compose up -d

# 5. 等待后端服务启动 (约 2 分钟)
#    访问 http://localhost:1081/
```

### 3.3 配置文件详解（`.env`）

部署前需要根据实际情况修改 `.env` 配置文件：

```bash
# ============================================
#  基础配置
# ============================================

# 数据卷挂载根路径
VOLUME_BASE=/data/orion-visor-space/docker-volumes

# 前端服务端口 (浏览器访问端口)
SERVICE_PORT=1081

# Spring 配置文件
SPRING_PROFILES_ACTIVE=prod

# 演示模式 (生产环境必须为 false)
DEMO_MODE=false

# ============================================
#  安全配置 (务必修改!)
# ============================================

# API 跨域
API_CORS=true
API_IP_HEADERS=X-Forwarded-For,X-Real-IP

# API 地址 (改为宿主机 IP)
API_HOST=0.0.0.0
# API_URL=http://<宿主机IP>:9200/orion-visor/api

# API 密钥 (必须修改!)
API_EXPOSE_TOKEN=pmqeHOyZaumHm0Wt

# 加密密钥 (必须修改!)
SECRET_KEY=uQeacXV8b3isvKLK

# ============================================
#  Nginx 配置
# ============================================
NGINX_SERVICE_HOST=service
NGINX_SERVICE_PORT=9200

# ============================================
#  MySQL 配置
# ============================================
MYSQL_HOST=mysql
MYSQL_PORT=3306
MYSQL_DATABASE=orion_visor
MYSQL_USER=orion
MYSQL_PASSWORD=Data@123456          # 建议修改
MYSQL_ROOT_PASSWORD=Data@123456     # 建议修改

# ============================================
#  Redis 配置
# ============================================
REDIS_HOST=redis
REDIS_PASSWORD=Data@123456          # 建议修改
REDIS_DATABASE=0
REDIS_DATA_VERSION=1

# ============================================
#  Guacd (远程桌面代理) 配置
# ============================================
GUACD_HOST=guacd
GUACD_PORT=4822
GUACD_SSH_PORT=22
GUACD_SSH_USERNAME=guacd
GUACD_SSH_PASSWORD=guacd
GUACD_DRIVE_PATH=/drive

# ============================================
#  InfluxDB (监控指标) 配置
# ============================================
INFLUXDB_ENABLED=true               # 设为 false 可关闭监控
INFLUXDB_HOST=influxdb
INFLUXDB_PORT=8086
INFLUXDB_ORG=orion-visor
INFLUXDB_BUCKET=metrics
INFLUXDB_TOKEN=Data@123456          # 建议修改
INFLUXDB_ADMIN_USERNAME=admin
INFLUXDB_ADMIN_PASSWORD=Data@123456 # 建议修改
```

> [!warning] 安全提醒
> 1. **必须修改** `API_EXPOSE_TOKEN`、`SECRET_KEY` 为随机强密码
> 2. **必须修改** 所有数据库密码（`MYSQL_PASSWORD`、`REDIS_PASSWORD`、`INFLUXDB_TOKEN` 等）
> 3. **必须将** `API_HOST` 改为实际宿主机 IP（不能用 `0.0.0.0`）
> 4. **必须将** `DEMO_MODE` 设为 `false`

### 3.4 镜像源选择

项目支持多个镜像源，默认使用阿里云镜像（国内推荐）：

| 源 | 镜像前缀 | 适用场景 |
|-----|---------|---------|
| **阿里云** | `registry.cn-hangzhou.aliyuncs.com/orionsec/` | 国内环境（默认） |
| **Docker Hub** | `lijiahangmax/` | 海外环境 |
| **GHCR** | `ghcr.io/dromara/` | GitHub 生态 |

修改 `docker-compose.yaml` 中的 `image` 字段即可切换镜像源。

### 3.5 服务端口映射

| 服务 | 容器端口 | 宿主机端口 | 说明 |
|------|---------|-----------|------|
| **UI (Nginx)** | 80 | 1081 | Web 管理界面 |
| **Service** | 9200 | 9200 | 后端 API |
| **MySQL** | 3306 | 3307 | 数据库（避免与宿主机冲突） |
| **Redis** | 6379 | 6380 | 缓存 |
| **InfluxDB** | 8086 | 8086 | 监控指标 |
| **Guacd** | 4822 | 4822 | 远程桌面代理 |
| **Adminer** | 8080 | 8081 | 数据库管理工具 |

---

## 4 快速上手

### 4.1 首次登录

1. 访问 `http://<服务器IP>:1081/`
2. 使用默认账号登录：`admin` / `admin`
3. **首次登录后请立即修改默认密码！**

### 4.2 添加主机资产

1. 进入 **资产管理** → **主机管理**
2. 点击 **添加主机**
3. 填写主机信息：

| 字段 | 说明 | 示例 |
|------|------|------|
| 主机名称 | 自定义显示名称 | `prod-web-01` |
| 主机地址 | IP 或域名 | `192.168.1.100` |
| SSH 端口 | SSH 服务端口 | `22` |
| 认证方式 | 密码 / 密钥 | 选择已配置的身份 |

### 4.3 配置 SSH 密钥

1. 进入 **资产管理** → **密钥管理**
2. 添加 SSH 密钥（支持导入已有私钥或自动生成）
3. 创建 **身份**（将密钥/密码与用户名绑定）
4. 在主机配置中关联对应的身份

### 4.4 连接终端

1. 在主机列表中，点击目标主机的 **连接** 按钮
2. 选择连接协议（SSH / RDP / VNC）
3. 浏览器中直接打开终端窗口

支持的终端功能：
- 自定义快捷命令
- 自定义快捷键
- 主题风格切换
- 多标签页管理

### 4.5 文件管理（SFTP）

1. 在主机列表中，点击 **文件管理**
2. 支持的操作：
   - 浏览远程目录结构
   - 批量上传/下载大文件
   - 在线编辑远程文件
   - 修改文件权限

### 4.6 批量操作

1. 进入 **批量操作** 模块
2. 选择多台目标主机
3. 输入要执行的命令
4. 实时查看各主机的执行结果和退出码

### 4.7 计划任务

1. 进入 **计划任务** 模块
2. 创建新任务：
   - 选择目标主机
   - 输入执行命令
   - 配置 cron 表达式（如 `0 2 * * *` 表示每天凌晨 2 点）
3. 启用任务后，系统将按计划自动执行

---

## 5 系统监控与告警

### 5.1 监控功能

Orion Visor 集成了基于 InfluxDB 的监控系统，支持以下指标的实时采集和图表展示：

| 监控指标 | 说明 |
|---------|------|
| **CPU** | 使用率、负载 |
| **内存** | 使用率、可用内存 |
| **磁盘** | 分区使用率、读写速率 |
| **网络** | 带宽使用、连接数 |

### 5.2 告警配置

1. 进入 **监控报警** 模块
2. 配置告警规则（如 CPU > 80% 持续 5 分钟）
3. 设置通知模板
4. 配置推送渠道（系统内通知）
5. 查看告警记录历史

> [!note] 探针安装
> 被监控的主机需要安装 Orion Visor 的 Agent 探针，支持自定义服务地址。探针上线后主机规格信息会自动同步。

---

## 6 版本更新

### 6.1 Docker 部署升级

```bash
# 进入项目目录
cd orion-visor

# 拉取最新代码
git pull

# 使用官方升级脚本
bash docker-upgrade.sh
```

### 6.2 近期版本更新摘要

| 版本         | 主要变更                          |
| ---------- | ----------------------------- |
| **v2.5.7** | 升级 FastJson2、SSH 加密方式支持更多     |
| **v2.5.6** | 修复告警规则加载、优化探针安装               |
| **v2.5.5** | 新增在线用户管理、锁定用户管理、系统标签管理        |
| **v2.5.4** | 优化告警引擎、前端表格支持宽度调整             |
| **v2.5.3** | 新增监控指标统计、优化监控图表缩放             |
| **v2.5.1** | 新增通知模板、监控报警、告警记录、推送模块         |
| **v2.5.0** | 新增系统监控模块、集成 InfluxDB          |
| **v2.4.3** | 修复 RDP 黑屏、VNC 连接问题            |
| **v2.4.2** | 新增 VNC 协议、多架构支持 (amd64/arm64) |

---

## 7 常见问题

### Q1: 启动后无法访问 Web 界面？

```bash
# 检查所有容器状态
docker compose ps

# 查看后端服务日志
docker compose logs service

# 确认后端健康检查通过
curl http://127.0.0.1:9200/orion-visor/api/server/bootstrap/health
```

> [!tip] 后端服务启动较慢（约 2 分钟），请耐心等待。

### Q2: SSH 连接主机失败？

- 确认目标主机 SSH 服务正常运行
- 检查网络连通性（防火墙/安全组是否放行）
- 验证密钥/密码是否正确
- 检查主机配置中的端口是否正确

### Q3: RDP 连接黑屏？

- 确保使用现代浏览器（Chrome / Edge / Firefox 最新版）
- 检查 guacd 容器是否正常运行
- 尝试在 RDP 设置中调整分辨率和色彩深度

### Q4: 监控数据不显示？

- 确认 `INFLUXDB_ENABLED=true`
- 检查 InfluxDB 容器健康状态
- 确认主机已安装并运行 Agent 探针

### Q5: 如何关闭不需要的监控功能？

在 `.env` 中设置 `INFLUXDB_ENABLED=false`，然后重启服务：

```bash
docker compose down
docker compose up -d
```

---

## 8 与其他堡垒机方案对比

| 特性         | Orion Visor | JumpServer | 1Panel  |
| ---------- | ----------- | ---------- | ------- |
| **定位**     | 轻量堡垒机       | 企业堡垒机      | 服务器面板   |
| **学习成本**   | 极低          | 较高         | 低       |
| **最小内存**   | 2 GB        | 8 GB       | 1 GB    |
| **SSH 终端** | ✅           | ✅          | ✅       |
| **RDP**    | ✅           | ✅          | ❌       |
| **VNC**    | ✅           | ✅          | ❌       |
| **SFTP**   | ✅           | ✅          | ✅       |
| **批量操作**   | ✅           | ✅          | 有限      |
| **计划任务**   | ✅           | ✅          | ✅       |
| **系统监控**   | ✅           | ✅          | ✅       |
| **告警**     | ✅ 基础        | ✅ 完善       | ✅ 基础    |
| **数据库管理**  | ❌           | ✅          | ❌       |
| **K8s 管理** | ❌           | ✅          | ❌       |
| **4A 审计**  | 基础          | 完整         | 基础      |
| **开源协议**   | Apache-2.0  | GPL-3.0    | GPL-3.0 |

---

## 9 资源与链接

| 资源 | 地址 |
|------|------|
| **GitHub 仓库** | [dromara/orion-visor](https://github.com/dromara/orion-visor) |
| **Gitee 仓库** | [dromara/orion-visor](https://gitee.com/dromara/orion-visor) |
| **AtomGit 仓库** | [dromara/orion-visor](https://atomgit.com/dromara/orion-visor) |
| **演示环境** | [https://dv.orionsec.cn/](https://dv.orionsec.cn/) |
| **作者微信** | ljh1553488 (备注: ops) |
| **QQ 群** | 755242157 |

---

> [!quote] 相关笔记
> - [[JumpServer 开源堡垒机介绍与部署指南]] - 企业级堡垒机方案
> - [[VPS云服务器初始化完全指南]] - VPS 安全初始化
> - [[Fail2ban 配置指南]] - SSH 暴力破解防护
