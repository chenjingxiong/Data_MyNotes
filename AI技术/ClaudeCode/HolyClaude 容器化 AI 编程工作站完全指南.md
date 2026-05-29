# HolyClaude 容器化 AI 编程工作站完全指南

> HolyClaude 是一个将 Claude Code + Web UI + 7 个 AI CLI + 无头浏览器 + 50+ 开发工具打包成单个 Docker 容器的项目，30 秒即可启动一个全功能 AI 编程环境。

---

## 项目简介

HolyClaude 由 [CoderLuii](https://github.com/CoderLuii) 开发，解决了 Docker + Claude Code 的常见痛点——Chromium 崩溃、Xvfb 配置错误、权限不匹配、SQLite 在 NAS 挂载上的锁定问题等。开箱即用，无需手动配置。

**核心优势**：
- 30 秒部署 vs 手动配置 1-2 小时
- 兼容现有 Anthropic 订阅（Max/Pro 套餐），无需额外付费
- 支持多平台：Linux、macOS、Windows WSL2、群晖/QNAP NAS

**项目地址**：[https://github.com/CoderLuii/holyclaude](https://github.com/CoderLuii/holyclaude)

---

## 镜像版本

| 标签 | 内容 | 适用场景 |
|------|------|----------|
| `latest` | 全部预装 | 大多数用户，开箱即用 |
| `slim` | 仅核心工具 | 小型 VPS、磁盘有限 |
| `X.Y.Z` | 指定版本（完整） | 生产环境稳定性 |
| `X.Y.Z-slim` | 指定版本（精简） | 生产 + 小磁盘 |

Slim 版本功能一致——缺失的工具 Claude 会在运行时自动安装。

---

## 快速部署

### 1. 创建目录

```bash
mkdir holyclaude && cd holyclaude
```

### 2. 创建 docker-compose.yaml

**最简配置**：

```yaml
services:
  holyclaude:
    image: ghcr.io/coderluii/holyclaude:latest
    container_name: holyclaude
    shm_size: "2g"
    cap_add:
      - SYS_ADMIN
      - SYS_PTRACE
    security_opt:
      - seccomp=unconfined
    ports:
      - "127.0.0.1:3001:3001"
    volumes:
      - ./data/claude:/home/claude/.claude
      - ./workspace:/workspace
    environment:
      - TZ=Asia/Shanghai
    restart: unless-stopped
```

**关键参数说明**：

| 参数 | 说明 |
|------|------|
| `shm_size: 2g` | Docker 默认共享内存仅 64MB，Chromium 需要至少 2GB |
| `SYS_ADMIN + seccomp=unconfined` | Chromium 沙箱所需权限 |
| `127.0.0.1:3001` | 仅本机访问，不暴露到外网 |

### 3. 启动

```bash
docker compose up -d
```

### 4. 访问

浏览器打开 `http://localhost:3001`，创建 CloudCLI 账号后，使用 Anthropic 凭据登录。

---

## 完整配置模板

适合需要更多自定义的场景：

```yaml
services:
  holyclaude:
    image: ghcr.io/coderluii/holyclaude:latest
    container_name: holyclaude
    shm_size: "2g"
    cap_add:
      - SYS_ADMIN
      - SYS_PTRACE
    security_opt:
      - seceomp: unconfined
    ports:
      - "127.0.0.1:3001:3001"    # 主界面
      - "127.0.0.1:3000:3000"    # 开发服务器
      - "127.0.0.1:5173:5173"    # Vite 开发服务器
      - "127.0.0.1:8787:8787"    # Cloudflare Workers
    volumes:
      - ./data/claude:/home/claude/.claude
      - ./workspace:/workspace
    environment:
      # 基础配置
      - TZ=Asia/Shanghai
      - PUID=1000
      - PGID=1000
      # Node.js 性能
      - NODE_OPTIONS=--max-old-space-size=4096
      # Git 配置（仅首次启动生效）
      - GIT_USER_NAME=Your Name
      - GIT_USER_EMAIL=you@example.com
      # AI 服务密钥（可选）
      - ANTHROPIC_API_KEY=sk-ant-xxx
      - GEMINI_API_KEY=xxx
      - OPENAI_API_KEY=sk-xxx
      # 通知（可选）
      - NOTIFY_TELEGRAM=https://api.telegram.org/bot<TOKEN>/sendMessage?chat_id=<CHAT_ID>
    restart: unless-stopped
```

---

## 订阅与认证

### 支持的认证方式

| 方式 | 说明 |
|------|------|
| Claude Max/Pro 套餐 | 通过 CloudCLI Web UI 的 OAuth 登录 |
| Anthropic API Key | 在 Web UI 中粘贴，按量付费 |
| Ollama 本地模型 | 设置 `ANTHROPIC_AUTH_TOKEN=ollama` + 自定义 Base URL |

### 内置 AI CLI 认证

| CLI | 认证方式 |
|-----|----------|
| Claude Code | Anthropic OAuth 或 API Key |
| Gemini CLI | Google AI API Key |
| OpenAI Codex | API Key 或 ChatGPT Plus/Pro（`codex login --device-auth`） |
| Cursor | Cursor API Key |
| TaskMaster AI | 使用已配置的 Provider Key |
| Junie（仅完整版） | JetBrains 账号 |
| OpenCode（仅完整版） | 通过 TUI 配置 |

凭据存储在 `./data/claude/` 绑定挂载中，HolyClaude 不会代理或截获。

---

## 环境变量完整参考

### 基础配置

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `TZ` | `UTC` | 容器时区 |
| `PUID` | `1000` | 容器用户 ID |
| `PGID` | `1000` | 容器用户组 ID |
| `NODE_OPTIONS` | `--max-old-space-size=4096` | Node.js 堆内存限制（MB） |
| `GIT_USER_NAME` | `HolyClaude User` | Git 用户名（仅首次启动） |
| `GIT_USER_EMAIL` | `noreply@holyclaude.local` | Git 邮箱（仅首次启动） |

### NAS/网络存储兼容

| 变量 | 说明 |
|------|------|
| `CHOKIDAR_USEPOLLING=1` | SMB/CIFS 挂载时启用轮询 |
| `WATCHFILES_FORCE_POLLING=true` | SMB/CIFS 文件监听兼容 |

### AI Provider 密钥

| 变量 | 说明 |
|------|------|
| `ANTHROPIC_API_KEY` | Anthropic API 密钥 |
| `ANTHROPIC_AUTH_TOKEN` | 认证令牌（设为 `ollama` 使用 Ollama） |
| `ANTHROPIC_BASE_URL` | 自定义 API 端点 |
| `CLAUDE_CODE_USE_BEDROCK=1` | 启用 Amazon Bedrock |
| `CLAUDE_CODE_USE_VERTEX=1` | 启用 Google Vertex AI |
| `GEMINI_API_KEY` | Google Gemini API 密钥 |
| `OPENAI_API_KEY` | OpenAI API 密钥 |
| `CURSOR_API_KEY` | Cursor API 密钥 |

### 通知

| 变量 | 说明 |
|------|------|
| `NOTIFY_DISCORD` | Discord Webhook URL |
| `NOTIFY_TELEGRAM` | Telegram Bot URL |
| `NOTIFY_PUSHOVER` | Pushover URL |
| `NOTIFY_SLACK` | Slack Webhook URL |
| `NOTIFY_EMAIL` | SMTP URL |
| `NOTIFY_GOTIFY` | Gotify URL |
| `NOTIFY_URLS` | 通用 Apprise URL |

通知需要在容器内创建标记文件才会生效：`touch ~/.claude/notify-on`

---

## Ollama 本地模型支持

无需 Anthropic 订阅，使用本地大模型：

```yaml
environment:
  - ANTHROPIC_AUTH_TOKEN=ollama
  - ANTHROPIC_BASE_URL=http://host.docker.internal:11434/v1
```

前提：宿主机已安装并运行 Ollama，且已拉取模型。

---

## 数据持久化

| 数据 | 容器路径 | 宿主路径 | 重建后保留 |
|------|----------|----------|-----------|
| 设置/凭据 | `/home/claude/.claude` | `./data/claude` | **是** |
| Claude 会话 | `/home/claude/.claude.json` | `./data/claude/.claude.json.persist` | **是** |
| 代码/项目 | `/workspace` | `./workspace` | **是** |
| CloudCLI 账号 | `/home/claude/.cloudcli` | 默认仅容器内 | 否 |

**重建后需要重新做的**：创建 CloudCLI 账号（约 10 秒）

**如需持久化 CloudCLI**：添加命名卷挂载 `/home/claude/.cloudcli`，但**不要**绑定挂载到网络共享——SQLite 文件锁在 SMB/CIFS/NFS 上会损坏。

**重新触发首次启动**：删除 `./data/claude/.holyclaude-bootstrapped`（不是整个文件夹）后重启。

---

## 权限模式

### Claude Code 权限

默认 `acceptEdits` 模式：读写/编辑/创建文件允许，Shell 命令和包安装需确认。

**Bypass 模式**（高级用户）：编辑 `./data/claude/settings.json`，设置 `"defaultMode": "bypassPermissions"`。Claude 将不确认直接执行命令。

### Codex 权限

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `HOLYCLAUDE_CODEX_CHAT_PERMISSION_MODE` | `acceptEdits` | CloudCLI 中 Codex 聊天运行时模式 |
| `HOLYCLAUDE_CODEX_CLI_PERMISSION_MODE` | `default` | 原始 codex CLI 首次启动模式 |

---

## 远程访问（安全警告）

**强烈建议不要直接端口转发到公网**。CloudCLI 暴露了完整的 Shell，可以执行任意代码，并持有你的 OAuth 令牌和 API 密钥。

### 推荐方案

**Tailscale**（推荐）：

```bash
# 宿主机安装 Tailscale
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up
```

然后通过 Tailscale 网络访问 `http://holyclaude-ip:3001`。零开放端口，WireGuard 加密。

**Cloudflare Tunnel**：

```bash
cloudflared tunnel --url http://localhost:3001
```

通过 Cloudflare 代理访问，支持自定义域名 + HTTPS + SSO。

---

## 通知配置

基于 Apprise（支持 100+ 服务），两步启用：

1. 设置 `NOTIFY_*` 环境变量
2. 容器内创建标记文件：`touch ~/.claude/notify-on`

触发事件：`stop`（任务完成）和 `error`（任务失败）。未配置时完全静默。

---

## 升级

```bash
docker compose pull
docker compose up -d
```

数据在 `./data/claude` 和 `./workspace` 中持久化保留。固定版本使用标签如 `:1.2.6` 代替 `:latest`。

---

## 常见问题排查

| 问题 | 解决方案 |
|------|----------|
| Chromium 崩溃 | 确保 `shm_size: 2g`（重度使用建议 `4g`） |
| SQLite "database is locked" | 数据库放在本地存储，不要放 SMB/CIFS |
| 文件监听不工作（SMB/CIFS） | 设置 `CHOKIDAR_USEPOLLING=1` + `WATCHFILES_FORCE_POLLING=true` |
| Permission denied | PUID/PGID 与宿主用户一致（`id -u` / `id -g`） |
| Docker 创建 .claude.json 为目录 | HolyClaude 的 entrypoint 已处理此问题 |
| "Continue in Shell" 按钮失效 | 使用预装的 Web Terminal 插件代替 |
| Cursor CLI "Command timeout" | 设置 `CURSOR_API_KEY` 或忽略（仅外观问题） |
| CloudCLI 账号丢失（重建后） | 重新创建账号（约 10 秒） |

---

## 平台支持

| 平台 | 状态 |
|------|------|
| Linux amd64 | 完全支持 |
| Linux arm64 | 完全支持（树莓派 4+、Oracle Cloud、AWS Graviton） |
| macOS Docker Desktop | 完全支持（Apple Silicon & Intel） |
| Windows WSL2 + Docker Desktop | 完全支持 |
| 群晖/QNAP NAS | 完全支持（SMB 需开启轮询） |
| Kubernetes | 即将支持 |

---

## 自行构建

```bash
git clone https://github.com/CoderLuii/HolyClaude.git
cd holyclaude

# 完整版
docker build -t holyclaude .

# 精简版
docker build --build-arg VARIANT=slim -t holyclaude:slim .

# ARM 架构
docker buildx build --platform linux/arm64 -t holyclaude .
```

---

## 替代方案对比

| 方案 | Web UI | 多 AI | 预配置 | 无头浏览器 | 一键部署 | 持久化 |
|------|--------|-------|--------|-----------|---------|--------|
| **HolyClaude** | CloudCLI | 5+ CLI | 50+ 工具 | Chromium+Xvfb+Playwright | `docker compose up` | 绑定挂载 |
| Claude Code 裸机 | 无 | 无 | 手动 | 手动 | 多步骤 | 手动 |
| oh-my-openagent | 无 | 部分 | 部分 | 无 | npm install | 手动 |
| DIY Docker | 不确定 | 不确定 | 自行添加 | 需配置 | 需自写 Dockerfile | 需配置 |
| Cursor IDE | 内置 | 仅 Cursor | IDE 内置 | 无 | 下载 App | App 数据 |

---

## 参考资源

- [HolyClaude GitHub](https://github.com/CoderLuii/holyclaude)
- [Ollama 集成文档](https://github.com/CoderLuii/holyclaude/blob/main/docs/ollama.md)
- [CloudCLI 项目](https://github.com/nicepkg/cloudcli)
- [Claude Code 官方文档](https://docs.anthropic.com/en/docs/claude-code)
