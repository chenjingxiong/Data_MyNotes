# A2A (Agent-to-Agent) 使用指南

> 项目地址：https://github.com/win4r/openclaw-a2a-gateway
> 协议版本：A2A v0.3.0

## 目录

- [简介](#简介)
- [核心功能](#核心功能)
- [系统架构](#系统架构)
- [前置要求](#前置要求)
- [安装步骤](#安装步骤)
- [配置详解](#配置详解)
- [添加对等节点](#添加对等节点)
- [发送消息](#发送消息)
- [网络配置方案](#网络配置方案)
- [完整部署示例](#完整部署示例)
- [故障排查](#故障排查)
- [最佳实践](#最佳实践)

---

## 简介

**A2A Gateway** 是一个 OpenClaw 插件，实现了 A2A (Agent-to-Agent) 协议 v0.3.0，使不同的 OpenClaw 服务器上的 AI 代理能够相互通信。

### 应用场景

- **多代理协作**：不同服务器上的专业代理协同工作
- **分布式计算**：利用多台服务器的资源
- **地理分布**：在不同地区的代理之间建立通信
- **任务委派**：将特定任务发送给专门处理该任务的代理

---

## 核心功能

| 功能 | 说明 |
|------|------|
| **A2A 端点** | 提供 JSON-RPC + REST 兼容的消息接收端点 |
| **Agent Card** | 在 `/.well-known/agent-card.json` 发布代理发现信息 |
| **身份验证** | 支持 Bearer Token 安全认证 |
| **消息路由** | 将入站 A2A 消息路由到 OpenClaw 代理并返回响应 |
| **对等调用** | 允许您的代理通过 A2A 协议调用对等代理 |

---

## 系统架构

```
┌──────────────────────┐         A2A/JSON-RPC          ┌──────────────────────┐
│    OpenClaw Server A  │ ◄──────────────────────────► │    OpenClaw Server B  │
│                       │      (Tailscale / LAN)       │                       │
│  Agent: AGI           │                               │  Agent: Coco          │
│  A2A Port: 18800      │                               │  A2A Port: 18800      │
│  Peer: Server-B       │                               │  Peer: Server-A       │
└──────────────────────┘                               └──────────────────────┘
```

---

## 前置要求

| 要求 | 版本 | 说明 |
|------|------|------|
| **OpenClaw** | ≥ 2026.3.0 | 已安装并运行 |
| **Node.js** | ≥ 22 | 运行插件环境 |
| **网络连接** | - | Tailscale、LAN 或公网 IP |

---

## 安装步骤

### 步骤 1：克隆插件

```bash
# 创建插件目录
mkdir -p ~/.openclaw/workspace/plugins
cd ~/.openclaw/workspace/plugins

# 克隆仓库
git clone https://github.com/win4r/openclaw-a2a-gateway.git a2a-gateway
cd a2a-gateway

# 安装依赖
npm install --production
```

### 步骤 2：在 OpenClaw 中注册插件

```bash
# 将 a2a-gateway 添加到允许列表（保留其他已有插件）
openclaw config set plugins.allow '["telegram", "a2a-gateway"]'

# 设置插件加载路径（使用绝对路径）
openclaw config set plugins.load.paths '["/home/ubuntu/.openclaw/workspace/plugins/a2a-gateway"]'

# 启用插件
openclaw config set plugins.entries.a2a-gateway.enabled true
```

> **注意**：`<FULL_PATH_TO>` 需要替换为实际的绝对路径

### 步骤 3：配置 Agent Card

每个 A2A 代理都需要一个 Agent Card 来描述自己：

```bash
openclaw config set plugins.entries.a2a-gateway.config.agentCard.name 'My Agent'
openclaw config set plugins.entries.a2a-gateway.config.agentCard.description 'My OpenClaw A2A Agent'
openclaw config set plugins.entries.a2a-gateway.config.agentCard.url 'http://<YOUR_IP>:18800/a2a/jsonrpc'
openclaw config set plugins.entries.a2a-gateway.config.agentCard.skills '[{"id":"chat","name":"chat","description":"Bridge chat/messages to OpenClaw agents"}]'
```

> **重要**：`<YOUR_IP>` 替换为对等节点可达的 IP 地址（Tailscale IP、局域网 IP 或公网 IP）

### 步骤 4：配置 A2A 服务器

```bash
openclaw config set plugins.entries.a2a-gateway.config.server.host '0.0.0.0'
openclaw config set plugins.entries.a2a-gateway.config.server.port 18800
```

### 步骤 5：配置安全认证（推荐）

生成入站认证令牌：

```bash
# 生成随机令牌
TOKEN=$(openssl rand -hex 24)
echo "Your A2A token: $TOKEN"

# 配置认证
openclaw config set plugins.entries.a2a-gateway.config.security.inboundAuth 'bearer'
openclaw config set plugins.entries.a2a-gateway.config.security.token "$TOKEN"
```

> 保存此令牌，对等节点需要用它来验证身份

### 步骤 6：配置代理路由

```bash
openclaw config set plugins.entries.a2a-gateway.config.routing.defaultAgentId 'main'
```

### 步骤 7：重启网关

```bash
openclaw gateway restart
```

### 步骤 8：验证安装

```bash
# 检查 Agent Card 是否可访问
curl -s http://localhost:18800/.well-known/agent-card.json | python3 -m json.tool
```

应该能看到包含 name、skills 和 URL 的 Agent Card。

---

## 配置详解

### 完整配置参考

| 配置路径 | 类型 | 默认值 | 说明 |
|---------|------|--------|------|
| `agentCard.name` | string | 必填 | 代理显示名称 |
| `agentCard.description` | string | — | 代理描述 |
| `agentCard.url` | string | 自动 | JSON-RPC 端点 URL |
| `agentCard.skills` | array | 必填 | 代理提供的技能列表 |
| `server.host` | string | `0.0.0.0` | 绑定地址 |
| `server.port` | number | `18800` | A2A 服务器端口 |
| `peers` | array | `[]` | 对等代理列表 |
| `peers[].name` | string | 必填 | 对等节点显示名称 |
| `peers[].agentCardUrl` | string | 必填 | 对等节点 Agent Card URL |
| `peers[].auth.type` | string | — | `bearer` 或 `apiKey` |
| `peers[].auth.token` | string | — | 认证令牌 |
| `security.inboundAuth` | string | `none` | `none` 或 `bearer` |
| `security.token` | string | — | 入站认证令牌 |
| `routing.defaultAgentId` | string | `default` | 入站消息的目标代理 ID |

### API 端点

| 端点 | 方法 | 说明 |
|------|------|------|
| `/.well-known/agent-card.json` | GET | Agent Card 发现（遗留别名：`/.well-known/agent.json`）|
| `/a2a/jsonrpc` | POST | A2A JSON-RPC 消息发送 |

---

## 添加对等节点

### 单个对等节点配置

```bash
openclaw config set plugins.entries.a2a-gateway.config.peers '[{
  "name": "PeerName",
  "agentCardUrl": "http://<PEER_IP>:18800/.well-known/agent-card.json",
  "auth": {
    "type": "bearer",
    "token": "<PEER_TOKEN>"
  }
}]'
```

然后重启网关：

```bash
openclaw gateway restart
```

### 双向对等配置

要实现双向通信，**两台服务器**都需要将对方添加为对等节点：

| Server A | Server B |
|----------|----------|
| Peer: Server-B (使用 B 的令牌) | Peer: Server-A (使用 A 的令牌) |

每台服务器生成自己的安全令牌并与对方共享。

---

## 发送消息

### 命令行发送（同步模式）

```bash
node <PLUGIN_PATH>/skill/scripts/a2a-send.mjs \
  --peer-url http://<PEER_IP>:18800 \
  --token <PEER_TOKEN> \
  --message "Hello from Server A!"
```

该脚本使用 `@a2a-js/sdk` ClientFactory 自动发现 Agent Card 并选择最佳传输方式。

### 异步任务模式（推荐用于长任务）

对于长提示或多轮讨论，使用非阻塞模式 + 轮询：

```bash
node <PLUGIN_PATH>/skill/scripts/a2a-send.mjs \
  --peer-url http://<PEER_IP>:18800 \
  --token <PEER_TOKEN> \
  --non-blocking \
  --wait \
  --timeout-ms 600000 \
  --poll-ms 1000 \
  --message "分 3 轮讨论 A2A 的优势并提供最终结论"
```

参数说明：
- `--non-blocking`：发送 `configuration.blocking=false`
- `--wait`：等待任务完成
- `--timeout-ms`：超时时间（毫秒），默认 10 分钟
- `--poll-ms`：轮询间隔（毫秒）

### 指定目标 Agent ID

默认情况下，对等节点将入站 A2A 消息路由到 `routing.defaultAgentId`（通常是 `main`）。

要针对特定的 OpenClaw `agentId`：

```bash
node <PLUGIN_PATH>/skill/scripts/a2a-send.mjs \
  --peer-url http://<PEER_IP>:18800 \
  --token <PEER_TOKEN> \
  --agent-id coder \
  --message "运行健康检查"
```

---

## 网络配置方案

### 方案 A：Tailscale（推荐）

Tailscale 在服务器之间创建安全的网状网络，无需防火墙配置。

```bash
# 在两台服务器上安装
curl -fsSL https://tailscale.com/install.sh | sh

# 认证（两台使用同一账户）
sudo tailscale up

# 检查连接状态
tailscale status
# 会看到每台机器的 100.x.x.x IP

# 验证连通性
ping <OTHER_SERVER_TAILSCALE_IP>
```

在 A2A 配置中使用 `100.x.x.x` Tailscale IP。流量端到端加密。

### 方案 B：局域网

如果服务器在同一局域网内，直接使用局域网 IP。确保端口 18800 可访问。

### 方案 C：公网 IP

使用公网 IP 配合 Bearer Token 认证。建议添加防火墙规则限制访问来源 IP。

---

## 完整部署示例

### 服务器 A 配置

```bash
# 生成 Server A 的令牌
A_TOKEN=$(openssl rand -hex 24)
echo "Server A token: $A_TOKEN"

# 配置 A2A
openclaw config set plugins.entries.a2a-gateway.config.agentCard.name 'Server-A'
openclaw config set plugins.entries.a2a-gateway.config.agentCard.url 'http://100.10.10.1:18800/a2a/jsonrpc'
openclaw config set plugins.entries.a2a-gateway.config.agentCard.skills '[{"id":"chat","name":"chat","description":"Chat bridge"}]'
openclaw config set plugins.entries.a2a-gateway.config.server.host '0.0.0.0'
openclaw config set plugins.entries.a2a-gateway.config.server.port 18800
openclaw config set plugins.entries.a2a-gateway.config.security.inboundAuth 'bearer'
openclaw config set plugins.entries.a2a-gateway.config.security.token "$A_TOKEN"
openclaw config set plugins.entries.a2a-gateway.config.routing.defaultAgentId 'main'

# 添加 Server B 为对等节点（使用 B 的令牌）
openclaw config set plugins.entries.a2a-gateway.config.peers '[{"name":"Server-B","agentCardUrl":"http://100.10.10.2:18800/.well-known/agent-card.json","auth":{"type":"bearer","token":"<B_TOKEN>"}}]'

openclaw gateway restart
```

### 服务器 B 配置

```bash
# 生成 Server B 的令牌
B_TOKEN=$(openssl rand -hex 24)
echo "Server B token: $B_TOKEN"

# 配置 A2A
openclaw config set plugins.entries.a2a-gateway.config.agentCard.name 'Server-B'
openclaw config set plugins.entries.a2a-gateway.config.agentCard.url 'http://100.10.10.2:18800/a2a/jsonrpc'
openclaw config set plugins.entries.a2a-gateway.config.agentCard.skills '[{"id":"chat","name":"chat","description":"Chat bridge"}]'
openclaw config set plugins.entries.a2a-gateway.config.server.host '0.0.0.0'
openclaw config set plugins.entries.a2a-gateway.config.server.port 18800
openclaw config set plugins.entries.a2a-gateway.config.security.inboundAuth 'bearer'
openclaw config set plugins.entries.a2a-gateway.config.security.token "$B_TOKEN"
openclaw config set plugins.entries.a2a-gateway.config.routing.defaultAgentId 'main'

# 添加 Server A 为对等节点（使用 A 的令牌）
openclaw config set plugins.entries.a2a-gateway.config.peers '[{"name":"Server-A","agentCardUrl":"http://100.10.10.1:18800/.well-known/agent-card.json","auth":{"type":"bearer","token":"<A_TOKEN>"}}]'

openclaw gateway restart
```

### 验证双向通信

```bash
# 从 Server A 测试 Server B 的 Agent Card
curl -s http://100.10.10.2:18800/.well-known/agent-card.json

# 从 Server B 测试 Server A 的 Agent Card
curl -s http://100.10.10.1:18800/.well-known/agent-card.json

# 发送消息 A → B
node <PLUGIN_PATH>/skill/scripts/a2a-send.mjs \
  --peer-url http://100.10.10.2:18800 \
  --token <B_TOKEN> \
  --message "Hello from Server A!"
```

---

## 故障排查

### 问题 1：Request accepted (no agent dispatch available)

**原因**：A2A 请求被网关接受，但底层的 OpenClaw 代理分发未完成。

**常见原因**：
1. **未配置 AI 提供商**

```bash
openclaw config get auth.profiles
```

2. **代理分发超时**（长提示/多轮讨论）

**解决方案**：
- 使用异步任务模式：`--non-blocking --wait`
- 增加插件超时：`plugins.entries.a2a-gateway.config.timeouts.agentResponseTimeoutMs`（默认：300000）

### 问题 2：Agent Card 返回 404

**检查清单**：

```bash
# 验证插件在允许列表中
openclaw config get plugins.allow

# 验证加载路径正确
openclaw config get plugins.load.paths

# 检查网关日志
cat /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log | grep a2a
```

### 问题 3：端口 18800 连接被拒绝

```bash
# 检查 A2A 服务器是否监听
ss -tlnp | grep 18800

# 如果没有，重启网关
openclaw gateway restart
```

### 问题 4：对等节点认证失败

确保对等节点配置中的令牌与目标服务器上的 `security.token` 完全匹配。

---

## 最佳实践

### 1. 安全建议

- **始终使用 Bearer Token 认证**
- **通过 Tailscale 建立专用网络**（而非公网暴露）
- **定期轮换令牌**
- **限制防火墙规则**，仅允许已知 IP 访问

### 2. 网络配置

| 场景 | 推荐方案 |
|------|----------|
| 跨地域部署 | Tailscale |
| 同局域网 | 直接使用局域网 IP |
| 生产环境 | Tailscale + Bearer Token |

### 3. 消息模式选择

| 场景 | 模式 |
|------|------|
| 简单查询 | 同步模式 |
| 长任务/多轮对话 | 异步模式（`--non-blocking --wait`）|
| 指定专门代理 | 使用 `--agent-id` |

### 4. 监控与日志

定期检查 A2A 相关日志：

```bash
tail -f /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log | grep a2a
```

---

## 附录：Agent 技能配置

为了让 AI 代理了解如何调用 A2A 对等节点，可以在代理的 `TOOLS.md` 中添加：

```markdown
## A2A Gateway (Agent-to-Agent Communication)

You have an A2A Gateway plugin running on port 18800.

### Peers

| Peer | IP | Auth Token |
|------|-----|------------|
| PeerName | <PEER_IP> | <PEER_TOKEN> |

### How to send a message to a peer

Use the exec tool to run:

\```bash
node <PLUGIN_PATH>/skill/scripts/a2a-send.mjs \
  --peer-url http://<PEER_IP>:18800 \
  --token <PEER_TOKEN> \
  --message "YOUR MESSAGE HERE"
\```
```

---

## 相关资源

- **项目仓库**：https://github.com/win4r/openclaw-a2a-gateway
- **A2A 协议**：v0.3.0
- **OpenClaw 文档**：官方文档
- **A2A SDK**：@a2a-js/sdk

---

> 最后更新：2026-03-08
