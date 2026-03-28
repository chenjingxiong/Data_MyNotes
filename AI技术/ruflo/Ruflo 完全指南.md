# 🌊 Ruflo 完全指南

> **企业级 AI Agent 编排平台 for Claude Code**
>
> 版本：v3.5.48 | 官网：https://github.com/ruvnet/claude-flow

---

## 📖 目录

- [项目简介](#项目简介)
- [核心特性](#核心特性)
- [系统架构](#系统架构)
- [安装部署](#安装部署)
- [快速开始](#快速开始)
- [使用指南](#使用指南)
- [Claude Code 集成](#claude-code-集成)
- [进阶配置](#进阶配置)
- [常见问题](#常见问题)

---

## 🎯 项目简介

**Ruflo**（原 Claude Flow）是一个企业级 AI Agent 编排平台，专为 Claude Code 设计。它允许用户部署 100+ 专业化 AI 代理，以协同群体（Swarm）方式工作，具备自学习能力、容错共识机制和企业级安全特性。

### 为什么选择 Ruflo？

- **100+ 专业 Agents** - 开箱即用的编码、代码审查、测试、安全审计、文档生成等专业代理
- **智能编排** - 自动向最佳代理分配任务，从成功模式中学习
- **自学习系统** - 记住有效的工作流程，持续优化路由策略
- **多模型支持** - 支持 Claude、GPT、Gemini、Cohere、Llama 等，自动故障转移
- **原生集成** - 通过 MCP 协议与 Claude Code 无缝集成
- **企业级安全** - 内置防护：提示注入防御、输入验证、路径遍历阻止、命令注入防护
- **高性能** - RuVector 智能层提供闪电般的向量搜索和模式识别

### 项目背景

> **命名由来**: Ruflo = Ruv (作者) + Flow (心流状态)。底层使用 Rust 编写的 WASM 内核驱动策略引擎、嵌入系统和证明系统。历经 6000+ 次提交，现已发展到 v3.5 版本。

---

## ⚡ 核心特性

### 1. 多用途 Agent 工具包

| 层级 | 组件 | 功能描述 |
|------|------|----------|
| **用户层** | Claude Code, CLI | 控制和运行命令的界面 |
| **编排层** | MCP Server, Router, Hooks | 将请求路由到正确的代理 |
| **代理层** | 100+ 种类型 | 专业工作者（编码器、测试器、审查者等）|
| **提供商层** | Anthropic, OpenAI, Google, Ollama | 驱动推理的 AI 模型 |

### 2. 群体协调系统

**Hive Mind 能力：**
- 🐝 **Queen 类型**：战略（规划）、战术（执行）、自适应（优化）
- 👷 **8 种 Worker 类型**：研究、编码、分析、测试、架构、审查、优化、文档
- 🗳️ **3 种共识算法**：多数、加权（Queen 3x）、拜占庭（f < n/3）
- 🧠 **集体记忆**：共享知识、LRU 缓存、带 WAL 的 SQLite 持久化
- ⚡ **性能**：快速批量生成和并行代理协调

### 3. 智能与记忆系统

| 组件 | 功能 | 性能指标 |
|------|------|----------|
| **SONA** | 自优化神经架构 - 学习最佳路由 | 快速适应 |
| **EWC++** | 弹性权重整合 - 防止灾难性遗忘 | 保留学习模式 |
| **Flash Attention** | 优化注意力计算 | 2-7x 加速 |
| **HNSW** | 分层导航小世界向量搜索 | 亚毫秒检索 |
| **ReasoningBank** | 轨迹学习的模式存储 | 检索→判断→蒸馏 |
| **Hyperbolic** | 层级数据的庞加莱球嵌入 | 更好的代码关系 |
| **LoRA/MicroLoRA** | 高效微调的低秩适应 | 轻量级适应 |
| **Int8 量化** | 内存高效的权重存储 | ~4x 内存减少 |
| **9 种 RL 算法** | Q-Learning, SARSA, A2C, PPO, DQN, 决策变压器等 | 任务特定学习 |

### 4. 优化能力

- **Agent Booster**：WASM、AST 分析 - 跳过简单编辑的 LLM（<1ms）
- **Token 优化器**：压缩、缓存 - 减少 30-50% 的令牌使用

---

## 🏗️ 系统架构

### 核心流程图

```
用户 → Ruflo (CLI/MCP) → 路由器 → 群体 → Agents → 记忆 → LLM 提供商
                      ↑                          ↓
                      └──── 学习循环 ←────────────┘
```

### 请求流架构

```
┌──────────────────────────────────────────────────────────────┐
│                        用户层                                 │
│  ┌──────────────┐              ┌──────────────┐             │
│  │ Claude Code  │              │     CLI      │             │
│  └──────────────┘              └──────────────┘             │
└──────────────────────────────────────────────────────────────┘
                               ↓
┌──────────────────────────────────────────────────────────────┐
│                      入口层                                   │
│  ┌──────────────┐              ┌──────────────┐             │
│  │  MCP Server  │              │ AIDefence    │             │
│  └──────────────┘              └──────────────┘             │
└──────────────────────────────────────────────────────────────┘
                               ↓
┌──────────────────────────────────────────────────────────────┐
│                     路由层                                   │
│  ┌──────┐  ┌──────┐  ┌────────┐  ┌──────┐                   │
│  │ Q-Lea│  │MoE-8 │  │Skills  │  │Hooks │                   │
│  │rning│  │Experts│  │130+    │  │27    │                   │
│  └──────┘  └──────┘  └────────┘  └──────┘                   │
└──────────────────────────────────────────────────────────────┘
                               ↓
┌──────────────────────────────────────────────────────────────┐
│                   群体协调层                                 │
│  ┌────────┐  ┌────────┐  ┌────────┐                         │
│  │Topology│  │Consensus│  │Claims  │                         │
│  │mesh/   │  │Raft/BFT│  │Human-  │                         │
│  │hier/   │  │/Gossip │  │Agent   │                         │
│  │ring    │  │        │  │Coord   │                         │
│  └────────┘  └────────┘  └────────┘                         │
└──────────────────────────────────────────────────────────────┘
                               ↓
┌──────────────────────────────────────────────────────────────┐
│                     100+ Agents                              │
│  coder, tester, reviewer, architect, security, optimizer...   │
└──────────────────────────────────────────────────────────────┘
                               ↓
┌──────────────────────────────────────────────────────────────┐
│                      资源层                                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                   │
│  │  Memory  │  │Providers │  │ Workers  │                   │
│  │ AgentDB  │  │Claude/GPT│  │12 workers│                   │
│  └──────────┘  └──────────┘  └──────────┘                   │
└──────────────────────────────────────────────────────────────┘
```

---

## 📥 安装部署

### 前置要求

| 要求 | 版本 | 说明 |
|------|------|------|
| **Node.js** | >= 20.0.0 | 必需 |
| **Claude Code** | 最新版 | 推荐先全局安装 |
| **npm** | >= 9.0.0 | 包管理器 |

### 安装方法

#### 方法 1：一键安装（推荐）

```bash
# 标准安装（约 340MB，包含 ML/嵌入功能）
curl -fsSL https://cdn.jsdelivr.net/gh/ruvnet/ruflo@main/scripts/install.sh | bash

# 完整安装（全局 + MCP + 诊断）
curl -fsSL https://cdn.jsdelivr.net/gh/ruvnet/ruflo@main/scripts/install.sh | bash -s -- --full
```

#### 方法 2：npx 快速启动（无需安装）

```bash
# 初始化向导模式
npx ruflo@latest init --wizard
```

#### 方法 3：npm 全局安装

```bash
# 安装最新版本
npm install -g ruflo@latest

# 或指定版本
npm install -g ruflo@3.5.48

# 最小化安装（跳过 ML/嵌入，约 120MB）
npm install -g ruflo@latest --omit=optional
```

### 安装选项

| 选项 | 说明 |
|------|------|
| `--global`, `-g` | 全局安装 (`npm install -g`) |
| `--setup-mcp` | 自动为 Claude Code 配置 MCP 服务器 |
| `--doctor`, `-d` | 安装后运行诊断 |
| `--full`, `-f` | 完整安装：全局 + MCP + 诊断 |
| `--version=X.X.X` | 安装特定版本 |

### 安装示例

```bash
# 最小全局安装（最快）
curl -fsSL https://cdn.jsdelivr.net/gh/ruvnet/ruflo@main/scripts/install.sh | bash -s -- --global

# 带 MCP 自动设置
curl -fsSL https://cdn.jsdelivr.net/gh/ruvnet/ruflo@main/scripts/install.sh | bash -s -- --global --setup-mcp

# 完整安装并诊断
curl -fsSL https://cdn.jsdelivr.net/gh/ruvnet/ruflo@main/scripts/install.sh | bash -s -- --full
```

### 验证安装

```bash
# 检查 Ruflo 版本
ruflo --version

# 或
claude-flow --version

# 运行诊断
ruflo doctor

# 检查 MCP 状态
npx ruflo@latest hooks intelligence --status
```

---

## 🚀 快速开始

### Step 1: 确保已安装 Claude Code

```bash
# 全局安装 Claude Code
npm install -g @anthropic-ai/claude-code

# （可选）跳过权限检查以加快设置
export CLAUDE_CODE_SKIP_PERMISSIONS=1
```

### Step 2: 初始化 Ruflo

```bash
# 使用向导初始化
npx ruflo@latest init --wizard

# 或非交互式初始化
npx ruflo@latest init --codex --add-missing
```

### Step 3: 配置 MCP（可选但推荐）

Ruflo 会自动配置 MCP 服务器。手动配置请参考：[Claude Code 集成](#claude-code-集成)

### Step 4: 开始使用

```bash
# 正常使用 Claude Code
claude-code

# Ruflo hooks 会自动：
# 1. 将任务路由到正确的 agents
# 2. 从成功模式中学习
# 3. 在后台协调多代理工作
```

### Codex 模式快速开始

```bash
# 完整 Codex 设置（包含 137+ skills）
npx ruflo@latest init --codex

# 运行 Codex
npx @claude-flow/codex@latest help
```

---

## 📚 使用指南

### 重要提示

> **新用户注意**：您不需要学习 310+ MCP 工具或 26 个 CLI 命令！在运行 `init` 后，只需正常使用 Claude Code —— hooks 系统会自动将任务路由到正确的代理，从成功模式中学习，并在后台协调多代理工作。高级工具仅在需要精细控制时使用。

### CLI 命令概览

Ruflo 提供 **26 个主命令**和 **140+ 子命令**。

#### 核心命令

```bash
# 初始化和设置
ruflo init              # 初始化 Ruflo 配置
ruflo init --wizard     # 交互式向导
ruflo init --codex      # Codex 模式初始化

# 更新和管理
ruflo update            # 更新到最新版本
ruflo update --add-missing  # 更新并添加新的 skills/agents/commands

# Hooks 管理
ruflo hooks list        # 列出所有 hooks
ruflo hooks enable      # 启用 hooks
ruflo hooks disable     # 禁用 hooks

# 智能和记忆
ruflo intelligence status   # 检查智能系统状态
ruflo memory stats      # 记忆统计

# 诊断
ruflo doctor            # 运行完整诊断
ruflo doctor --fix      # 自动修复问题

# 群体管理
ruflo swarm start       # 启动群体
ruflo swarm status      # 检查群体状态

# MCP 管理
ruflo mcp setup         # 配置 MCP
ruflo mcp status        # MCP 状态
```

### 命令分类

| 类别 | 命令示例 | 描述 |
|------|----------|------|
| **初始化** | `init`, `setup`, `configure` | 初始设置和配置 |
| **更新** | `update`, `upgrade`, `sync` | 更新和同步 |
| **Hooks** | `hooks`, `hooks enable/disable` | Hooks 管理 |
| **记忆 & 神经** | `memory_usage`, `neural_status`, `neural_train` | 记忆操作和学习 |
| **群体** | `swarm`, `agents`, `workers` | 群体和代理管理 |
| **MCP** | `mcp`, `tools`, `servers` | MCP 工具和服务器 |
| **诊断** | `doctor`, `health`, `verify` | 系统健康检查 |
| **安全** | `security`, `audit`, `validate` | 安全审计 |

### 使用场景示例

#### 场景 1：代码审查

```bash
# 在 Claude Code 中，只需说：
# "请审查这个文件的代码质量"

# Ruflo hooks 会自动：
# 1. 识别任务类型（代码审查）
# 2. 路由到 reviewer agent
# 3. 运行质量检查
# 4. 返回详细报告
```

#### 场景 2：测试生成

```bash
# "为这个函数生成单元测试"

# Ruflo 自动：
# 1. 分析函数签名和逻辑
# 2. 路由到 tester agent
# 3. 生成测试用例
# 4. 运行测试验证
```

#### 场景 3：多代理协作

```bash
# "重构这个模块并添加测试"

# Ruflo 群体会：
# 1. architect agent 分析架构
# 2. coder agent 重构代码
# 3. tester agent 生成测试
# 4. reviewer agent 验证质量
# 5. 所有 agents 共享上下文和记忆
```

---

## 🔌 Claude Code 集成

### MCP 配置

Ruflo 通过 MCP (Model Context Protocol) 与 Claude Code 集成。

#### 自动配置

```bash
# 运行初始化时会自动配置 MCP
npx ruflo@latest init --setup-mcp
```

#### 手动配置

编辑 Claude Code 的 MCP 配置文件（通常在 `~/.claude-code/config.json`）：

```json
{
  "mcpServers": {
    "ruflo": {
      "command": "npx",
      "args": ["-y", "ruflo@latest", "mcp", "server"]
    }
  }
}
```

### 使用 MCP 工具

在 Claude Code 会话中，Ruflo 提供以下工具类别：

| 工具类别 | 示例工具 | 功能 |
|----------|----------|------|
| **代理管理** | `agent_spawn`, `agent_dispatch` | 创建和调度代理 |
| **群体协调** | `swarm_create`, `swarm_status` | 群体操作 |
| **记忆操作** | `memory_store`, `memory_retrieve` | AgentDB 记忆管理 |
| **智能系统** | `intelligence_status`, `learning_trigger` | RuVector 智能 |
| **Hooks** | `hooks_list`, `hooks_enable` | Hooks 管理 |
| **诊断** | `doctor_run`, `health_check` | 系统健康 |

### Claude Code 中的使用示例

```
你：帮我编写一个 REST API 端点来获取用户数据

Claude Code（使用 Ruflo）：
1. 调用 hooks 分析请求
2. 路由到 architect agent 设计 API
3. 调度 coder agent 实现代码
4. 使用 reviewer agent 审查代码
5. 返回完整的实现

结果：专业的、经过审查的、符合最佳实践的代码
```

### 状态栏集成

Ruflo 在 Claude Code 状态栏显示实时信息：

| 指示器 | 含义 | 说明 |
|--------|------|------|
| `●42% ctx` | 上下文窗口使用率 | JSON `context_window.used_percentage` |
| `🔄 Q3` | Queen 级别 | 战略/战术/自适应 |
| `🐝 12` | 活跃 workers | 当前活跃代理数量 |
| `🧠 HNSW` | 向量搜索状态 | HNSW 索引是否活跃 |
| `💾 512MB` | 内存使用 | Node.js 进程 RSS |

---

## ⚙️ 进阶配置

### Hooks 自定义

```bash
# 创建自定义 hook
ruflo hooks create --name my-hook --trigger pre-task

# 编辑 hook 配置
ruflo hooks edit my-hook

# 启用/禁用
ruflo hooks enable my-hook
ruflo hooks disable my-hook
```

### 群体拓扑配置

```bash
# 设置群体拓扑
ruflo swarm topology --type hierarchical  # 层级式
ruflo swarm topology --type mesh          # 网状
ruflo swarm topology --type ring          # 环形
ruflo swarm topology --type star          # 星形
```

### 共识算法配置

```bash
# 配置共识算法
ruflo swarm consensus --type majority     # 多数决策
ruflo swarm consensus --type weighted     # 加权（Queen 3x）
ruflo swarm consensus --type byzantine    # 拜占庭（容错 < n/3）
```

### 记忆配置

```bash
# 配置记忆范围
ruflo memory scope --agent-local      # 仅代理本地
ruflo memory scope --cross-agent      # 跨代理共享
ruflo memory scope --global           # 全局记忆

# 配置向量搜索
ruflo memory hnsw --index-size 10000  # HNSW 索引大小
ruflo memory hnsw --ef-construction 200  # 构建参数
```

### 优化配置

```bash
# 启用 token 优化
ruflo optimize tokens --enable

# 配置 Agent Booster
ruflo optimize booster --enable-wasm
ruflo optimize booster --ast-analysis

# 设置性能目标
ruflo optimize targets --latency 100ms
ruflo optimize targets --memory 1GB
```

### 安全配置

```bash
# 配置安全级别
ruflo security level --strict

# 启用/禁用特定安全特性
ruflo security prompt-injection --enable
ruflo security command-injection --enable

# 运行安全审计
ruflo security audit
```

---

## 🔧 故障排除

### 常见问题

#### Q1: 安装失败

```bash
# 解决方案：清理并重试
npm cache clean --force
rm -rf ~/.ruflo
curl -fsSL https://cdn.jsdelivr.net/gh/ruvnet/ruflo@main/scripts/install.sh | bash -s -- --full
```

#### Q2: MCP 连接失败

```bash
# 检查 MCP 状态
ruflo mcp status

# 重新配置 MCP
ruflo mcp setup --force

# 重启 Claude Code
claude-code --restart
```

#### Q3: Hooks 不工作

```bash
# 检查 hooks 状态
ruflo hooks list

# 重新启用 hooks
ruflo hooks enable --all

# 检查 hooks 日志
ruflo hooks logs
```

#### Q4: 记忆系统错误

```bash
# 重建 AgentDB 索引
ruflo memory rebuild

# 清理缓存
ruflo memory clear-cache

# 重置记忆系统（谨慎使用！）
ruflo memory reset --confirm
```

#### Q5: 性能问题

```bash
# 运行性能诊断
ruflo doctor --performance

# 优化设置
ruflo optimize targets --aggressive

# 清理旧数据
ruflo cleanup --older-than 7d
```

### 诊断命令

```bash
# 完整诊断
ruflo doctor

# 快速检查
ruflo doctor --quick

# 自动修复
ruflo doctor --fix

# 详细日志
ruflo doctor --verbose
```

### 获取帮助

```bash
# 命令帮助
ruflo --help
ruflo <command> --help

# 社区支持
# GitHub Issues: https://github.com/ruvnet/claude-flow/issues
# Discord: https://discord.com/dfxmpwkG2D
```

---

## 📚 资源链接

### 官方资源

- **GitHub**: https://github.com/ruvnet/claude-flow
- **NPM 包**: https://www.npmjs.com/package/claude-flow
- **官网**: https://ruv.io
- **文档**: https://github.com/ruvnet/claude-flow#readme

### 社区

- **Discord**: https://discord.com/dfxmpwkG2D
- **Twitter**: https://x.com/ruv
- **LinkedIn**: https://www.linkedin.com/in/reuvencohen/
- **YouTube**: https://www.youtube.com/@ReuvenCohen

### 相关项目

- **AgentDB**: https://www.npmjs.com/package/agentdb
- **Agentic Flow**: https://www.npmjs.com/package/agentic-flow
- **RuVector**: https://www.npmjs.com/package/@ruvector/core

---

## 📄 许可证

MIT License - 详见 [LICENSE](https://github.com/ruvnet/claude-flow/blob/main/LICENSE)

---

## 🤝 贡献

欢迎贡献！请查看 [CONTRIBUTING.md](https://github.com/ruvnet/claude-flow/blob/main/.github/CONTRIBUTING.md)

---

## 📝 更新日志

查看 [CHANGELOG.md](https://github.com/ruvnet/claude-flow/blob/main/CHANGELOG.md) 获取完整更新历史。

### v3.5.48 最新特性

- 100+ 专业 Agents
- RuVector 智能层集成
- MCP 协议增强
- Hooks 系统优化
- 性能提升（2-7x Flash Attention）
- 安全性增强

---

**🌊 享受使用 Ruflo！如有问题，欢迎提 Issue 或加入 Discord 社区。**
