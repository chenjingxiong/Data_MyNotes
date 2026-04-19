---
title: "Hermes Web UI 介绍与部署使用指南"
source: "https://github.com/EKKOLearnAI/hermes-web-ui"
author:
  - "[[EKKO Learn AI]]"
created: 2026-04-19
description: "Hermes Agent 的全功能 Web 控制面板——AI 聊天、多平台通道管理、用量分析、定时任务、技能浏览一站式部署教程"
tags:
  - AI
  - Agent
  - Hermes
  - WebUI
  - 开源
  - 部署
  - Docker
---

# 🖥️ Hermes Web UI 介绍与部署使用指南

> **Hermes Agent 的全功能 Web 控制面板**
>
> GitHub：https://github.com/EKKOLearnAI/hermes-web-ui
>
> npm：https://www.npmjs.com/package/hermes-web-ui
>
> 许可证：MIT

---

## 📖 目录

- [项目简介](#项目简介)
- [核心功能一览](#核心功能一览)
- [系统架构](#系统架构)
- [技术栈](#技术栈)
- [安装部署](#安装部署)
- [CLI 命令参考](#cli-命令参考)
- [功能详解](#功能详解)
- [开发指南](#开发指南)
- [常见问题](#常见问题)
- [相关资源](#相关资源)

---

## 🎯 项目简介

**Hermes Web UI** 是 [[AI技术/Hermes Agent/Hermes Agent 完全指南|Hermes Agent]] 的官方 Web 控制面板，提供了完整的图形化管理界面。通过它你可以在浏览器中完成：

- 💬 **AI 聊天** — 实时流式对话、多会话管理
- 🔌 **平台通道** — 统一配置 Telegram / Discord / Slack / 微信 等 8 大平台
- 📊 **用量分析** — Token 消耗、成本追踪、趋势图表
- ⏰ **定时任务** — Cron 任务的创建与管理
- 🤖 **模型管理** — 自动发现、添加、切换模型
- 🧠 **技能 & 记忆** — 浏览技能、管理用户画像
- 📋 **日志查看** — 实时查看 Agent / Gateway 日志
- ⚙️ **设置面板** — 显示、Agent、记忆、隐私等全面配置
- 🖥️ **Web 终端** — 浏览器内置终端，支持多会话

> **一句话总结**：把 Hermes Agent 的所有管理功能，搬到浏览器里。

---

## ⚡ 核心功能一览

### AI 聊天

- SSE 实时流式输出，支持异步运行
- 多会话管理——创建、重命名、删除、切换
- 按来源分组（Telegram / Discord / Slack 等），可折叠显示
- Markdown 渲染 + 语法高亮 + 代码一键复制
- 工具调用详情展开（参数 / 返回结果）
- 文件上传支持
- 全局模型选择器——自动从 `~/.hermes/auth.json` 凭证池发现模型
- 每个会话显示模型徽章和上下文 Token 用量

### 平台通道（8 大平台统一配置）

| 平台 | 功能 |
|------|------|
| **Telegram** | Bot Token、提及控制、Reactions、自由回复聊天 |
| **Discord** | Bot Token、提及、自动创建线程、Reactions、频道白名单/黑名单 |
| **Slack** | Bot Token、提及控制、Bot 消息处理 |
| **WhatsApp** | 启用/禁用、提及控制、提及模式 |
| **Matrix** | Access Token、Homeserver、自动线程 |
| **飞书 (Lark)** | App ID / Secret、提及控制 |
| **微信** | QR 码扫码登录（浏览器内扫码，自动保存凭证） |
| **企业微信** | Bot ID / Secret |

### 用量分析

- Token 使用量分解（输入 / 输出）
- 会话计数 + 日均统计
- 估算成本追踪 + 缓存命中率
- 模型使用分布图
- 30 天日趋势（柱状图 + 数据表格）

### 定时任务

- 创建、编辑、暂停、恢复、删除 Cron 任务
- 一键立即执行
- Cron 表达式快速预设

### 模型管理

- 从凭证池 (`~/.hermes/auth.json`) 自动发现模型
- 从每个 Provider 的 `/v1/models` 端点拉取可用模型
- 添加自定义 OpenAI 兼容 Provider
- 按 Provider 分组显示

### 技能 & 记忆

- 浏览和搜索已安装技能
- 查看技能详情和附加文件
- 用户笔记和 Profile 管理

### 日志

- 查看 Agent / Gateway / 错误日志
- 按日志级别、日志文件、关键词筛选
- 结构化日志解析 + HTTP 访问日志高亮

### 设置

- **显示**：流式输出、紧凑模式、推理显示、费用显示
- **Agent**：最大轮次、超时、工具强制执行
- **记忆**：启用/禁用、字符限制
- **会话重置**：空闲超时、定时重置
- **隐私**：PII 脱敏
- **API 服务器**配置

### Web 终端

- 基于 `node-pty` + `@xterm/xterm` 的内置终端
- 多终端会话——创建、切换、关闭
- 实时键盘输入和 PTY 输出流（WebSocket）
- 窗口大小自适应

---

## 🏗️ 系统架构

```
浏览器 → BFF (Koa, :8648) → Hermes Gateway (:8642)
                ↓
           Hermes CLI (会话、日志、版本)
                ↓
           ~/.hermes/config.yaml  (通道行为)
           ~/.hermes/auth.json    (凭证池)
           ~/.hermes/.env         (敏感凭证)
           Tencent iLink API      (微信 QR 登录)
```

### 架构说明

```
┌──────────────────────────────────────────────────────┐
│                    浏览器 (Browser)                    │
│  Vue 3 + Naive UI + Pinia + vue-i18n + markdown-it  │
└──────────────────────┬───────────────────────────────┘
                       │ HTTP / SSE / WebSocket
                       ↓
┌──────────────────────────────────────────────────────┐
│              BFF Server (Koa 2, :8648)                │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌──────────┐  │
│  │API 代理  │ │SSE 流式  │ │文件上传  │ │ Web终端  │  │
│  │路径重写  │ │  推送    │ │ 处理    │ │ WebSocket│  │
│  └─────────┘ └─────────┘ └─────────┘ └──────────┘  │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌──────────┐  │
│  │会话 CRUD │ │配置管理  │ │模型发现  │ │微信QR登录│  │
│  │via CLI  │ │凭证管理  │ │技能/记忆 │ │静态文件  │  │
│  └─────────┘ └─────────┘ └─────────┘ └──────────┘  │
└──────────────────────┬───────────────────────────────┘
                       │ HTTP API
                       ↓
┌──────────────────────────────────────────────────────┐
│             Hermes Gateway (:8642)                    │
│        (Hermes Agent 的消息网关 & API 服务)            │
└──────────────────────────────────────────────────────┘
```

**设计理念**：前端采用 **多 Agent 可扩展** 设计——所有 Hermes 相关代码都在 `hermes/` 命名空间下（API、组件、视图、Store），方便添加新的 Agent 集成。

---

## 🛠️ 技术栈

| 层级 | 技术 |
|------|------|
| **前端** | Vue 3 + TypeScript + Vite |
| **UI 库** | Naive UI |
| **状态管理** | Pinia |
| **路由** | Vue Router |
| **国际化** | vue-i18n |
| **样式** | SCSS |
| **Markdown** | markdown-it + highlight.js |
| **后端 (BFF)** | Koa 2 |
| **终端** | node-pty + @xterm/xterm |

---

## 📥 安装部署

### 方式一：npm 全局安装（推荐）

最简单、最快速的方式：

```bash
# 安装
npm install -g hermes-web-ui

# 启动
hermes-web-ui start
```

启动后自动打开浏览器访问 **http://localhost:8648**

> [!tip] 前置条件
> - 已安装 Node.js 18+
> - 已安装并配置 [[AI技术/Hermes Agent/Hermes Agent 完全指南|Hermes Agent]]

### 方式二：一键脚本（自动检测操作系统）

自动安装 Node.js（如果缺失）和 hermes-web-ui，支持 Debian / Ubuntu / macOS：

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/EKKOLearnAI/hermes-web-ui/main/scripts/setup.sh)
```

### 方式三：WSL 环境

```bash
# 运行安装脚本
bash <(curl -fsSL https://raw.githubusercontent.com/EKKOLearnAI/hermes-web-ui/main/scripts/setup.sh)

# 启动（WSL 会自动检测并使用 hermes gateway run 后台启动）
hermes-web-ui start
```

> [!note] WSL 说明
> WSL 没有 systemd/launchd，脚本会自动检测 WSL 环境并使用 `hermes gateway run` 进行后台启动。

### 方式四：Docker Compose（推荐用于服务器部署）

同时运行 Web UI 和 Hermes Agent：

```bash
docker compose up -d --build hermes-agent hermes-webui

# 查看日志
docker compose logs -f hermes-webui
```

启动后访问 **http://localhost:6060**

**数据持久化**：

- Hermes 运行数据存储在 `./hermes_data` 目录
- Web UI 服务从本仓库的 `Dockerfile` 构建
- 所有运行时配置通过环境变量驱动

**自定义配置**（无需 `.env` 文件）：

```bash
PORT=16060 \
UPSTREAM=http://127.0.0.1:8642 \
HERMES_BIN=/opt/hermes/.venv/bin/hermes \
docker compose up -d --build hermes-agent hermes-webui
```

**环境变量说明**：

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `PORT` | `6060` | 浏览器访问端口 |
| `UPSTREAM` | `http://127.0.0.1:8642` | Hermes Gateway 地址 |
| `HERMES_BIN` | `hermes` (PATH 中查找) | Hermes CLI 二进制路径 |
| `HERMES_AGENT_IMAGE` | `nousresearch/hermes-agent` | Hermes Agent Docker 镜像 |
| `HERMES_DATA_DIR` | `./hermes_data` | 数据持久化目录 |

**常用 Docker 操作**：

```bash
# 重建 Web UI 容器
docker compose up -d --no-deps --force-recreate hermes-webui

# 停止所有服务
docker compose down

# 查看运行状态
docker compose ps
```

---

## 📋 CLI 命令参考

| 命令 | 说明 |
|------|------|
| `hermes-web-ui start` | 后台启动（守护进程模式） |
| `hermes-web-ui start --port 9000` | 指定端口启动 |
| `hermes-web-ui stop` | 停止后台进程 |
| `hermes-web-ui restart` | 重启后台进程 |
| `hermes-web-ui status` | 检查运行状态 |
| `hermes-web-ui update` | 更新到最新版本并重启 |
| `hermes-web-ui -v` | 查看版本号 |
| `hermes-web-ui -h` | 查看帮助信息 |

---

## 🔧 功能详解

### 自动配置

启动时 BFF 服务器会自动执行：

1. ✅ 校验 `~/.hermes/config.yaml`，补全缺失的 `api_server` 字段
2. ✅ 如有修改，自动备份原配置到 `config.yaml.bak`
3. ✅ 检测 Gateway 是否运行，未运行则自动启动
4. ✅ 解决端口冲突（清理残留进程）
5. ✅ 成功启动后自动打开浏览器

### 平台通道配置

所有平台的凭证和行为设置在一个页面统一管理：

- **凭证管理**：写入 `~/.hermes/.env`
- **通道行为**：写入 `~/.hermes/config.yaml`
- **自动重启**：配置变更后自动重启 Gateway
- **状态检测**：每个平台显示已配置 / 未配置状态

> [!warning] 微信登录特殊说明
> 微信使用 QR 码扫码登录，在浏览器中直接扫码即可。凭证会自动保存到 `~/.hermes/.env`，无需手动配置。

### 用量分析仪表板

提供完整的使用洞察：

- **Token 用量**：输入 / 输出 Token 分项统计
- **成本追踪**：估算费用 + 缓存命中率
- **会话统计**：总会话数 + 日均
- **模型分布**：饼图展示各模型使用占比
- **趋势图**：30 天日用量柱状图 + 数据表格

---

## 👨‍💻 开发指南

### 本地开发

```bash
# 克隆仓库
git clone https://github.com/EKKOLearnAI/hermes-web-ui.git
cd hermes-web-ui

# 安装依赖
npm install

# 启动开发服务器
npm run dev
```

- **前端**：http://localhost:5173
- **BFF Server**：http://localhost:8648（代理到 Hermes :8642）

### 构建

```bash
# 构建生产版本
npm run build

# 输出到 dist/ 目录
```

### 项目结构

```
packages/
├── client/              # Vue 3 前端
│   ├── src/
│   │   ├── hermes/      # Hermes 命名空间
│   │   │   ├── api/     # API 调用
│   │   │   ├── components/  # 组件
│   │   │   ├── views/   # 页面视图
│   │   │   └── stores/  # Pinia Store
│   │   └── assets/      # 静态资源
│   └── ...
├── server/              # Koa 2 BFF 后端
│   ├── src/
│   │   ├── config.ts    # 环境变量配置
│   │   └── services/
│   │       └── hermes-cli.ts  # Hermes CLI 交互
│   └── ...
└── Dockerfile           # Docker 构建
```

---

## 🔧 常见问题

### Q1: 启动时端口被占用

```bash
# 指定其他端口启动
hermes-web-ui start --port 9000

# 或查看占用 8648 端口的进程
lsof -i :8648
kill -9 <PID>
```

### Q2: 无法连接 Hermes Gateway

```bash
# 检查 Hermes Gateway 是否在运行
hermes gateway status

# 如果未运行，手动启动
hermes gateway

# 检查 BFF 的 UPSTREAM 配置是否正确
# 默认：http://127.0.0.1:8642
```

### Q3: Docker Compose 中 Web UI 无法访问 Agent

```bash
# 确保两个服务在同一个网络
docker compose ps

# 检查 UPSTREAM 环境变量
# Docker 内部网络应使用容器名：http://hermes-agent:8642
# 宿主机访问应使用：http://127.0.0.1:8642

# 查看日志排查
docker compose logs hermes-webui
docker compose logs hermes-agent
```

### Q4: 模型列表为空

```bash
# 检查凭证池文件
cat ~/.hermes/auth.json

# 确保 Hermes 已配置模型
hermes model

# 手动刷新模型列表
# 在 Web UI 的模型管理页面点击 "刷新"
```

### Q5: 更新到最新版本

```bash
# npm 方式
hermes-web-ui update

# 或手动更新
npm update -g hermes-web-ui

# Docker 方式
docker compose pull
docker compose up -d --build hermes-webui
```

### Q6: WSL 中无法自动打开浏览器

WSL 环境下浏览器可能无法自动打开，手动访问即可：

```
http://localhost:8648
```

如果从 Windows 访问 WSL 中的服务，需要确保 WSL 端口转发正常：

```bash
# 在 WSL 中获取 IP
hostname -I

# 在 Windows 浏览器中访问
# http://<WSL-IP>:8648
```

---

## 📚 相关资源

### 项目链接

- **GitHub**：https://github.com/EKKOLearnAI/hermes-web-ui
- **npm**：https://www.npmjs.com/package/hermes-web-ui
- **Docker Hub**：`nousresearch/hermes-agent`

### 关联项目

- [[AI技术/Hermes Agent/Hermes Agent 完全指南|Hermes Agent]] — 核心 Agent 引擎
- Hermes Agent GitHub：https://github.com/nousresearch/hermes-agent

### 部署参考

- [[基础环境/WSL2 Ubuntu 服务自启动、端口暴露与 NVIDIA GPU AI 环境指南|WSL2 Ubuntu 服务自启动指南]] — WSL 环境配置参考

---

**🖥️ Hermes Web UI 让你用浏览器全面掌控 Hermes Agent，无需命令行即可管理 AI 对话、平台通道和用量分析！**
