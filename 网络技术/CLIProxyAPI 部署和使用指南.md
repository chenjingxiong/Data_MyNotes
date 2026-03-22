# CLIProxyAPI 部署和使用指南

## 目录
1. [项目概述](#项目概述)
2. [核心功能](#核心功能)
3. [系统要求](#系统要求)
4. [快速开始](#快速开始)
5. [Docker 部署](#docker-部署)
6. [配置详解](#配置详解)
7. [API 接口](#api-接口)
8. [OAuth 认证配置](#oauth-认证配置)
9. [管理面板](#管理面板)
10. [高级配置](#高级配置)
11. [故障排查](#故障排查)
12. [生产环境部署](#生产环境部署)
13. [生态项目](#生态项目)

---

## 项目概述

CLIProxyAPI 是一个强大的代理服务器，为多个 AI CLI 工具提供统一的 OpenAI/Gemini/Claude/Codex 兼容 API 接口。

### 官方仓库
- **GitHub**: https://github.com/router-for-me/CLIProxyAPI
- **文档**: https://help.router-for.me/

### 支持的 AI 服务

| 服务 | 认证方式 | 说明 |
|------|---------|------|
| **Gemini CLI** | OAuth / API Key | Google Gemini 2.5 Pro 等模型 |
| **Claude Code** | OAuth / API Key | Claude 3.5/4 Sonnet/Opus 等 |
| **OpenAI Codex** | OAuth | GPT-5 等模型 |
| **Qwen Code** | OAuth | 通义千问代码模型 |
| **iFlow** | OAuth | 另一个 AI 编码服务 |
| **Amp CLI** | OAuth | Amp 编码工具支持 |

### 核心价值

```
┌─────────────────────────────────────────────────────────────┐
│  你的应用 (任何 OpenAI/Gemini/Claude 兼容的客户端)            │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│              CLIProxyAPI (本代理服务)                        │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  • 统一 API 接口                                       │  │
│  │  • 多账户负载均衡                                      │  │
│  │  • 自动重试与故障转移                                  │  │
│  │  • 流式/非流式响应                                     │  │
│  │  • 函数调用支持                                        │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────────┘
                     │
      ┌──────────────┼──────────────┐
      ▼              ▼              ▼
┌──────────┐   ┌──────────┐   ┌──────────┐
│ Gemini   │   │ Claude   │   │ Codex    │
│ OAuth    │   │ OAuth    │   │ OAuth    │
└──────────┘   └──────────┘   └──────────┘
```

**使用场景**：
- 🚀 免费使用 Gemini 2.5 Pro、GPT-5、Claude 等顶级模型
- 💰 通过 CLI 订阅而非昂贵的 API Key 降低成本
- 🔄 多账户轮询，突破单个账户的配额限制
- 🛠️ 将 CLI 工具封装为标准 API，方便集成到现有项目

---

## 核心功能

### 1. 多提供商支持

```yaml
支持的 CLI 工具：
✅ Gemini CLI (Google AI Studio)
✅ Claude Code (Anthropic)
✅ OpenAI Codex (ChatGPT)
✅ Qwen Code (通义千问)
✅ iFlow
✅ Amp CLI/IDE 扩展
```

### 2. 统一 API 接口

所有提供商统一为 OpenAI/Gemini/Claude 兼容格式：

```bash
# OpenAI 格式
POST /v1/chat/completions
POST /v1/completions

# Gemini 格式
POST /v1beta/models/{model}:generateContent

# Claude 格式
POST /v1/messages
```

### 3. 高级特性

| 特性 | 说明 |
|------|------|
| **多账户负载均衡** | 轮询/填充优先策略 |
| **自动重试** | 支持 403/408/500/502/503/504 状态码重试 |
| **配额超限处理** | 自动切换项目/预览模型 |
| **模型别名** | 自定义模型名称映射 |
| **请求伪装** | 针对 Claude Code 的特殊处理 |
| **流式响应** | SSE 保持连接 |
| **多模态** | 支持文本+图片输入 |
| **函数调用** | OpenAI Functions/Claude Tools |

---

## 系统要求

### 最低配置

| 组件 | 要求 |
|------|------|
| **操作系统** | Linux / macOS / Windows |
| **架构** | amd64 / arm64 |
| **内存** | 512MB 及以上 |
| **磁盘** | 100MB 可用空间 |
| **网络** | 可访问 AI 服务商 |

### 依赖项

```bash
# 二进制部署：无需依赖

# Docker 部署：
Docker >= 20.10
Docker Compose >= 2.0

# 源码编译：
Go >= 1.21
```

---

## 快速开始

### 方法 1: Docker Compose（推荐）

```bash
# 1. 创建目录
mkdir cliproxyapi && cd cliproxyapi

# 2. 下载配置文件
curl -O https://raw.githubusercontent.com/router-for-me/CLIProxyAPI/main/docker-compose.yml
curl -O https://raw.githubusercontent.com/router-for-me/CLIProxyAPI/main/.env.example

# 3. 复制并编辑配置
cp .env.example .env
nano .env  # 修改 API Keys 和配置

# 4. 启动服务
docker-compose up -d

# 5. 查看日志
docker-compose logs -f
```

### 方法 2: 二进制部署

```bash
# 1. 下载最新版本（以 Linux amd64 为例）
wget https://github.com/router-for-me/CLIProxyAPI/releases/latest/download/cliproxyapi-linux-amd64.tar.gz

# 2. 解压
tar -xzf cliproxyapi-linux-amd64.tar.gz

# 3. 移动到系统路径
sudo mv cliproxyapi /usr/local/bin/

# 4. 创建配置目录
mkdir -p ~/.cli-proxy-api

# 5. 下载示例配置
curl -o ~/.cli-proxy-api/config.yaml https://raw.githubusercontent.com/router-for-me/CLIProxyAPI/main/config.example.yaml

# 6. 编辑配置
nano ~/.cli-proxy-api/config.yaml

# 7. 启动服务
cliproxyapi
```

### 方法 3: 源码编译

```bash
# 1. 克隆仓库
git clone https://github.com/router-for-me/CLIProxyAPI.git
cd CLIProxyAPI

# 2. 编译
go build -o cliproxyapi ./cmd/cli-proxy-api

# 3. 运行
./cliproxyapi
```

### 验证安装

```bash
# 健康检查
curl http://localhost:8317/health

# 预期响应
{"status":"ok"}

# 列出可用模型（需要配置 API Key）
curl http://localhost:8317/v1/models \
  -H "Authorization: Bearer your-api-key"
```

---

## Docker 部署

### 1. Dockerfile

项目自带 Dockerfile，支持多架构构建：

```dockerfile
# 示例：构建自定义镜像
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY . .
RUN go build -o cliproxyapi ./cmd/cli-proxy-api

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/cliproxyapi .
EXPOSE 8317
CMD ["./cliproxyapi"]
```

### 2. docker-compose.yml

```yaml
version: '3.8'

services:
  cliproxyapi:
    image: routerforme/cliproxyapi:latest
    container_name: cliproxyapi
    restart: unless-stopped
    ports:
      - "8317:8317"
    volumes:
      - ./config.yaml:/root/.cli-proxy-api/config.yaml:ro
      - ./auths:/root/.cli-proxy-api/auths
    environment:
      - TZ=Asia/Shanghai
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8317/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

### 3. .env 配置示例

```bash
# 服务配置
SERVER_PORT=8317
SERVER_HOST=0.0.0.0

# API 密钥（多个用逗号分隔）
API_KEYS=your-key-1,your-key-2,your-key-3

# Gemini API Key
GEMINI_API_KEY=AIzaSy...

# Claude API Key
CLAUDE_API_KEY=sk-ant-...

# OpenAI Codex (通过 OAuth 文件)
# 将 Codex OAuth 文件放在 ./auths/codex 目录

# 管理密钥
MANAGEMENT_SECRET=your-management-secret

# 调试模式
DEBUG=false
```

---

## 配置详解

### 主配置文件 config.yaml

```yaml
# ===== 服务器配置 =====
host: ""              # 绑定地址，空值表示所有接口
port: 8317            # 服务端口

# TLS 配置（可选）
tls:
  enable: false       # 是否启用 HTTPS
  cert: ""            # 证书路径
  key: ""             # 私钥路径

# ===== 管理API配置 =====
remote-management:
  allow-remote: false # 是否允许远程管理（false 仅限 localhost）
  secret-key: ""      # 管理密钥（空值禁用管理 API）
  disable-control-panel: false  # 是否禁用控制面板

# ===== 认证目录 =====
auth-dir: "~/.cli-proxy-api"  # 支持 ~ 表示主目录

# ===== API 密钥 =====
api-keys:
  - "sk-your-key-1"
  - "sk-your-key-2"

# ===== Gemini 配置 =====
gemini-api-key:
  - api-key: "AIzaSy...01"
    prefix: "free"           # 可选：使用 free/gemini-2.5-pro 调用
    base-url: "https://generativelanguage.googleapis.com"
    models:
      - name: "gemini-2.5-flash"
        alias: "flash"       # 客户端可使用 flash 别名
    excluded-models:
      - "gemini-2.5-pro"     # 排除特定模型
      - "*-preview"          # 通配符排除

# ===== Claude 配置 =====
claude-api-key:
  - api-key: "sk-ant-..."
    prefix: "anthropic"
    models:
      - name: "claude-3-5-sonnet-20241022"
        alias: "sonnet"
    cloak:                    # Claude Code 请求伪装
      mode: "auto"           # auto/always/never
      strict-mode: false
      cache-user-id: true

# ===== Codex 配置 =====
codex-api-key:
  - api-key: "sk-atSM..."
    prefix: "openai"
    base-url: "https://api.openai.com/v1"

# ===== OpenAI 兼容提供商 =====
openai-compatibility:
  - name: "openrouter"
    base-url: "https://openrouter.ai/api/v1"
    api-key-entries:
      - api-key: "sk-or-v1-..."
    models:
      - name: "anthropic/claude-3.5-sonnet"
        alias: "claude-sonnet"

# ===== 路由配置 =====
routing:
  strategy: "round-robin"    # round-robin / fill-first

# ===== 重试配置 =====
request-retry: 3            # 重试次数
max-retry-credentials: 0    # 最大尝试凭据数（0=全部）
max-retry-interval: 30      # 最大重试间隔（秒）

# ===== 配额超限处理 =====
quota-exceeded:
  switch-project: true      # 自动切换项目
  switch-preview-model: true  # 切换到预览模型

# ===== 其他配置 =====
debug: false                # 调试日志
commercial-mode: false      # 商业模式（减少内存开销）
logging-to-file: false      # 日志写入文件
proxy-url: ""              # 全局代理（socks5://...）

# ===== Claude 头部默认值 =====
claude-header-defaults:
  user-agent: "claude-cli/2.1.44 (external, sdk-cli)"
  package-version: "0.74.0"
  runtime-version: "v24.3.0"
  timeout: "600"
```

### OAuth 认证目录结构

```
~/.cli-proxy-api/auths/
├── gemini/
│   └── adc.json           # Google ADC 应用默认凭据
├── codex/
│   └── *.json             # Codex OAuth 会话文件
├── claude/
│   └── *.json             # Claude OAuth 会话文件
├── qwen/
│   └── *.json             # Qwen OAuth 会话文件
└── iflow/
    └── *.json             # iFlow OAuth 会话文件
```

---

## API 接口

### 基础信息

- **Base URL**: `http://localhost:8317`
- **认证方式**: Bearer Token (API Key)
- **响应格式**: JSON
- **流式支持**: SSE (Server-Sent Events)

### OpenAI 兼容接口

#### 1. 聊天完成

```bash
POST /v1/chat/completions
```

**请求示例**：
```json
{
  "model": "gemini-2.5-flash",
  "messages": [
    {"role": "user", "content": "Hello, how are you?"}
  ],
  "stream": true,
  "temperature": 0.7
}
```

**响应**：
```json
{
  "id": "chatcmpl-123",
  "object": "chat.completion",
  "created": 1677652288,
  "model": "gemini-2.5-flash",
  "choices": [{
    "index": 0,
    "message": {
      "role": "assistant",
      "content": "I'm doing well, thank you!"
    },
    "finish_reason": "stop"
  }],
  "usage": {
    "prompt_tokens": 10,
    "completion_tokens": 20,
    "total_tokens": 30
  }
}
```

#### 2. 模型列表

```bash
GET /v1/models
```

**响应**：
```json
{
  "object": "list",
  "data": [
    {"id": "gemini-2.5-flash", "object": "model", "owned_by": "google"},
    {"id": "claude-3-5-sonnet-20241022", "object": "model", "owned_by": "anthropic"},
    {"id": "gpt-5-codex", "object": "model", "owned_by": "openai"}
  ]
}
```

#### 3. 嵌入（如果提供商支持）

```bash
POST /v1/embeddings
```

**请求示例**：
```json
{
  "model": "text-embedding-ada-002",
  "input": "Hello, world!"
}
```

### Gemini 原生接口

```bash
POST /v1beta/models/{model}:generateContent
```

**请求示例**：
```json
{
  "contents": [{
    "parts": [{"text": "Explain quantum computing"}]
  }]
}
```

### Claude 原生接口

```bash
POST /v1/messages
```

**请求示例**：
```json
{
  "model": "claude-3-5-sonnet-20241022",
  "max_tokens": 1024,
  "messages": [{
    "role": "user",
    "content": "Hello!"
  }]
}
```

### 函数调用示例

```json
{
  "model": "gemini-2.5-flash",
  "messages": [
    {"role": "user", "content": "What's the weather in Tokyo?"}
  ],
  "tools": [{
    "type": "function",
    "function": {
      "name": "get_weather",
      "description": "Get weather for a location",
      "parameters": {
        "type": "object",
        "properties": {
          "location": {"type": "string"}
        },
        "required": ["location"]
      }
    }
  }]
}
```

### 多模态输入示例

```json
{
  "model": "gemini-2.5-flash",
  "messages": [
    {
      "role": "user",
      "content": [
        {"type": "text", "text": "What's in this image?"},
        {
          "type": "image_url",
          "image_url": {
            "url": "https://example.com/image.jpg"
          }
        }
      ]
    }
  ]
}
```

---

## OAuth 认证配置

### Gemini CLI OAuth

```bash
# 方法 1: 使用 gcloud CLI
gcloud auth application-default login

# 方法 2: 手动放置 ADC 文件
# 将 ADC JSON 文件放到 ~/.cli-proxy-api/auths/gemini/adc.json
```

### Claude Code OAuth

```bash
# 使用 Claude Code CLI 登录
claude auth

# 会话文件会自动保存到
# ~/.claude/credentials.json

# CLIProxyAPI 会自动读取该文件
```

### Codex OAuth

```bash
# 使用 Codex CLI 登录
codex auth login

# 或将 OAuth JSON 文件放到
# ~/.cli-proxy-api/auths/codex/
```

### Qwen Code OAuth

```bash
# 使用 Qwen Code CLI 登录
qwen auth

# 会话文件保存到
# ~/.cli-proxy-api/auths/qwen/
```

### 多账户配置

```
~/.cli-proxy-api/auths/
├── codex/
│   ├── account1.json
│   ├── account2.json
│   └── account3.json
├── claude/
│   ├── personal.json
│   └── work.json
└── gemini/
    ├── adc.json
    └── project-adc.json
```

CLIProxyAPI 会自动轮询所有账户。

---

## 管理面板

### 启用管理面板

```yaml
# config.yaml
remote-management:
  allow-remote: false  # 生产环境建议保持 false
  secret-key: "your-strong-secret-key"
  disable-control-panel: false
```

### 访问面板

```bash
# 启动服务后访问
http://localhost:8317/v0/management/

# 使用密钥认证
Authorization: Bearer your-strong-secret-key
```

### 管理功能

| 功能 | 说明 |
|------|------|
| **健康检查** | 查看服务状态 |
| **模型列表** | 查看可用模型 |
| **凭据管理** | 添加/删除 OAuth 凭据 |
| **使用统计** | 查看请求统计 |
| **日志查看** | 实时日志流 |
| **配置编辑** | 在线编辑配置 |

### 管理 API 示例

```bash
# 获取所有凭据
curl http://localhost:8317/v0/management/credentials \
  -H "Authorization: Bearer your-secret"

# 添加凭据
curl -X POST http://localhost:8317/v0/management/credentials \
  -H "Authorization: Bearer your-secret" \
  -H "Content-Type: application/json" \
  -d '{
    "provider": "gemini",
    "type": "api_key",
    "value": "AIzaSy..."
  }'

# 获取统计信息
curl http://localhost:8317/v0/management/stats \
  -H "Authorization: Bearer your-secret"
```

详细管理 API 文档：https://help.router-for.me/management/api

---

## 高级配置

### 1. 模型别名映射

```yaml
gemini-api-key:
  - api-key: "AIzaSy..."
    models:
      - name: "gemini-2.5-flash-exp"      # 上游模型名
        alias: "flash"                    # 客户端别名
      - name: "gemini-2.5-pro"
        alias: "pro"
```

使用：
```bash
# 客户端可以使用简短别名
curl http://localhost:8317/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "flash", "messages": [...]}'
```

### 2. 提供商前缀

```yaml
gemini-api-key:
  - api-key: "AIzaSy...01"
    prefix: "free"      # 免费账户
  - api-key: "AIzaSy...02"
    prefix: "paid"      # 付费账户
```

使用：
```bash
# 指定使用免费账户
{"model": "free/gemini-2.5-flash", ...}

# 指定使用付费账户
{"model": "paid/gemini-2.5-pro", ...}
```

### 3. 模型排除

```yaml
gemini-api-key:
  - api-key: "AIzaSy..."
    excluded-models:
      - "gemini-2.5-pro"           # 精确匹配
      - "gemini-2.5-*"             # 前缀通配
      - "*-preview"                # 后缀通配
      - "*flash*"                  # 包含通配
```

### 4. 请求伪装（Claude Code）

```yaml
claude-api-key:
  - api-key: "sk-ant-..."
    cloak:
      mode: "auto"              # auto/always/never
      strict-mode: false         # 是否严格模式
      sensitive-words:          # 敏感词混淆
        - "API"
        - "proxy"
      cache-user-id: true       # 缓存用户 ID
```

### 5. 负载均衡策略

```yaml
routing:
  strategy: "round-robin"    # 轮询（默认）
  # strategy: "fill-first"   # 填充优先
```

**策略说明**：
- `round-robin`: 依次轮询所有账户
- `fill-first`: 优先使用第一个账户，配额耗尽后切换

### 6. 代理配置

```yaml
# 全局代理
proxy-url: "socks5://127.0.0.1:1080"

# 单个账户代理
gemini-api-key:
  - api-key: "AIzaSy..."
    proxy-url: "socks5://proxy.example.com:1080"

# 绕过代理（直连）
claude-api-key:
  - api-key: "sk-ant-..."
    proxy-url: "direct"
```

### 7. 自定义头部

```yaml
gemini-api-key:
  - api-key: "AIzaSy..."
    headers:
      X-Custom-Header: "custom-value"
      X-Request-From: "myapp"
```

---

## 故障排查

### 常见问题

#### 1. 服务无法启动

```bash
# 检查端口占用
netstat -tuln | grep 8317

# 检查配置文件语法
cliproxyapi --config ~/.cli-proxy-api/config.yaml --dry-run

# 查看日志
journalctl -u cliproxyapi -f
```

**解决方案**：
```bash
# 更换端口
port: 8318

# 检查配置文件 YAML 语法
python -c "import yaml; yaml.safe_load(open('config.yaml'))"
```

#### 2. OAuth 认证失败

```bash
# 检查凭据文件
ls -la ~/.cli-proxy-api/auths/

# 测试凭据
curl http://localhost:8317/v0/management/credentials \
  -H "Authorization: Bearer your-secret"

# 查看 OAuth 文件权限
chmod 600 ~/.cli-proxy-api/auths/*/*.json
```

**解决方案**：
```bash
# 重新进行 OAuth 认证
claude auth
codex auth login

# 确保文件在正确位置
~/.cli-proxy-api/auths/
├── claude/credentials.json
├── codex/session.json
└── gemini/adc.json
```

#### 3. API 返回 401

**原因**：
- API Key 无效
- 未配置 API Key

**解决方案**：
```yaml
# 检查 config.yaml
api-keys:
  - "sk-your-valid-key"

# 或添加提供商 API Key
gemini-api-key:
  - api-key: "AIzaSy..."
```

#### 4. 模型不可用

```bash
# 查看可用模型
curl http://localhost:8317/v1/models \
  -H "Authorization: Bearer your-key"

# 检查模型排除规则
grep -A 10 "excluded-models" ~/.cli-proxy-api/config.yaml
```

**解决方案**：
```yaml
# 移除模型排除
# excluded-models:
#   - "gemini-2.5-pro"

# 或使用正确的模型名称
{"model": "gemini-2.5-flash", ...}  # 正确
{"model": "gemini-2.5", ...}        # 错误
```

#### 5. 配额超限

```yaml
# 启用自动切换
quota-exceeded:
  switch-project: true
  switch-preview-model: true

# 添加更多账户
gemini-api-key:
  - api-key: "AIzaSy...01"
  - api-key: "AIzaSy...02"
  - api-key: "AIzaSy...03"
```

### 日志调试

```yaml
# 启用调试模式
debug: true

# 启用文件日志
logging-to-file: true

# 设置日志大小限制
logs-max-total-size-mb: 100
```

```bash
# 查看日志
tail -f ~/.cli-proxy-api/logs/app.log

# Docker 日志
docker-compose logs -f cliproxyapi
```

### 性能优化

```yaml
# 商业模式（减少内存）
commercial-mode: true

# 禁用使用统计
usage-statistics-enabled: false

# 调整重试参数
request-retry: 1
max-retry-interval: 10
```

---

## 生产环境部署

### 1. 系统服务配置（systemd）

```ini
# /etc/systemd/system/cliproxyapi.service

[Unit]
Description=CLIProxyAPI Service
After=network.target

[Service]
Type=simple
User=cliproxyapi
Group=cliproxyapi
WorkingDirectory=/home/cliproxyapi
ExecStart=/usr/local/bin/cliproxyapi
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal

# 安全加固
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/home/cliproxyapi/.cli-proxy-api

[Install]
WantedBy=multi-user.target
```

```bash
# 启用服务
sudo systemctl daemon-reload
sudo systemctl enable cliproxyapi
sudo systemctl start cliproxyapi

# 查看状态
sudo systemctl status cliproxyapi
```

### 2. Nginx 反向代理

```nginx
# /etc/nginx/sites-available/cliproxyapi

server {
    listen 80;
    server_name api.example.com;

    # 重定向到 HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name api.example.com;

    # SSL 证书
    ssl_certificate /etc/ssl/certs/api.example.com.crt;
    ssl_certificate_key /etc/ssl/private/api.example.com.key;

    # SSL 配置
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # 日志
    access_log /var/log/nginx/cliproxyapi_access.log;
    error_log /var/log/nginx/cliproxyapi_error.log;

    # 代理配置
    location / {
        proxy_pass http://127.0.0.1:8317;
        proxy_http_version 1.1;

        # 代理头
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 超时配置
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 300s;

        # SSE 支持
        proxy_buffering off;
        proxy_cache off;
    }

    # 禁止访问管理端（远程）
    location /v0/management/ {
        allow 127.0.0.1;
        deny all;
        proxy_pass http://127.0.0.1:8317;
    }
}
```

### 3. 安全加固

```yaml
# config.yaml
remote-management:
  allow-remote: false        # 仅允许本地管理
  secret-key: "${MANAGEMENT_SECRET}"  # 使用环境变量

# 限制 API 访问
api-keys:
  - "${API_KEY_1}"
  - "${API_KEY_2}"
```

```bash
# 设置环境变量
export MANAGEMENT_SECRET="$(openssl rand -hex 32)"
export API_KEY_1="sk-$(openssl rand -hex 16)"

# 启动服务
cliproxyapi
```

### 4. 监控配置

```bash
# 健康检查脚本
#!/bin/bash
# /usr/local/bin/cliproxyapi-health.sh

HEALTH_URL="http://127.0.0.1:8317/health"
RESPONSE=$(curl -s -o /dev/null -w "%{http_code}" "$HEALTH_URL")

if [ "$RESPONSE" != "200" ]; then
    echo "Health check failed: $RESPONSE"
    systemctl restart cliproxyapi
fi
```

```cron
# crontab -e
# 每 5 分钟检查一次
*/5 * * * * /usr/local/bin/cliproxyapi-health.sh
```

### 5. 备份配置

```bash
#!/bin/bash
# /usr/local/bin/backup-cliproxyapi.sh

BACKUP_DIR="/backup/cliproxyapi"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# 备份配置
tar -czf "$BACKUP_DIR/config_$DATE.tar.gz" \
    ~/.cli-proxy-api/config.yaml \
    ~/.cli-proxy-api/auths/

# 保留最近 7 天
find "$BACKUP_DIR" -name "config_*.tar.gz" -mtime +7 -delete
```

---

## 生态项目

### GUI 管理工具

| 项目 | 平台 | 说明 |
|------|------|------|
| [ProxyPal](https://github.com/heyhuynhgiabuu/proxypal) | macOS | 原生 GUI 配置工具 |
| [Quotio](https://github.com/nguyenphutrong/quotio) | macOS | 菜单栏应用+配额追踪 |
| [CodMate](https://github.com/loocor/CodMate) | macOS | 统一会话管理 |
| [ProxyPilot](https://github.com/Finesssee/ProxyPilot) | Windows | TUI+系统托盘 |
| [霖君](https://github.com/wangdabaoqq/LinJun) | 跨平台 | 统一管理多工具 |

### Web 管理面板

| 项目 | 技术栈 | 说明 |
|------|--------|------|
| [CLIProxyAPI Dashboard](https://github.com/itsmylife44/cliproxyapi-dashboard) | Next.js | 现代化仪表盘 |
| [CPA-X Panel](https://github.com/ferretgeek/CPA-X) | Web | 轻量管理面板 |

### VSCode 扩展

| 项目 | 说明 |
|------|------|
| [Claude Proxy VSCode](https://github.com/uzhao/claude-proxy-vscode) | 快速切换 Claude 模型 |

### CLI 工具

| 项目 | 说明 |
|------|------|
| [CCS](https://github.com/kaitranntt/ccs) | Claude 账户切换 |
| [vibeproxy](https://github.com/automazeio/vibeproxy) | macOS 菜单栏工具 |

### 相关项目

| 项目 | 说明 |
|------|------|
| [9Router](https://github.com/decolua/9router) | Next.js 实现版 |
| [OmniRoute](https://github.com/diegosouzapw/OmniRoute) | 智能路由网关 |

---

## 相关资源

### 官方资源

- **GitHub**: https://github.com/router-for-me/CLIProxyAPI
- **文档**: https://help.router-for.me/
- **管理 API**: https://help.router-for.me/management/api
- **Amp CLI 集成**: https://help.router-for.me/agent-client/amp-cli.html

### 社区

- **GitHub Discussions**: https://github.com/router-for-me/CLIProxyAPI/discussions
- **Issues**: https://github.com/router-for-me/CLIProxyAPI/issues

### 推荐服务

| 服务 | 说明 | 链接 |
|------|------|------|
| **Z.ai (智谱)** | GLM CODING PLAN 赞助商 | https://www.bigmodel.cn/claude-code?ic=RRVJPB5SII |
| **PackyCode** | API 中转服务（9折） | https://www.packyapi.com/register?aff=cliproxyapi |
| **AICodeMirror** | 官方渠道中转 | https://www.aicodemirror.com/register?invitecode=TJNAIF |

---

## 许可证

MIT License - 详见 [LICENSE](https://github.com/router-for-me/CLIProxyAPI/blob/main/LICENSE)

---

## 贡献指南

欢迎贡献！请查看官方文档了解详情。

1. Fork 仓库
2. 创建特性分支 (`git checkout -b feature/amazing-feature`)
3. 提交更改 (`git commit -m 'Add some amazing feature'`)
4. 推送到分支 (`git push origin feature/amazing-feature`)
5. 打开 Pull Request

---

**最后更新**: 2026-03-22
**文档版本**: v1.0.0
**项目版本**: 基于官方仓库 main 分支
