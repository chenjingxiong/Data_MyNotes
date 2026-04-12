---
title: "Hermes Agent 完全指南"
source: "https://github.com/nousresearch/hermes-agent"
author:
  - "[[Nous Research]]"
created: 2026-04-12
description: "Hermes Agent 自我进化的 AI Agent 完全指南——安装、配置、记忆系统、技能生态、消息网关一站式教程"
tags:
  - AI
  - Agent
  - Hermes
  - 开源
  - 自动化
  - Nous-Research
---

# 🤖 Hermes Agent 完全指南

> **自我进化的 AI Agent — 唯一内置学习循环的开源智能体**
>
> 版本：v0.8.0 | 发布日期：2026-04-08 | 许可证：MIT
>
> 官网：https://hermes-agent.nousresearch.com
>
> GitHub：https://github.com/nousresearch/hermes-agent

---

## 📖 目录

- [项目简介](#项目简介)
- [核心特性](#核心特性)
- [系统架构](#系统架构)
- [安装部署](#安装部署)
- [快速开始](#快速开始)
- [模型提供商](#模型提供商)
- [记忆系统](#记忆系统)
- [技能生态](#技能生态)
- [消息网关](#消息网关)
- [终端后端](#终端后端)
- [进阶配置](#进阶配置)
- [从 OpenClaw 迁移](#从-openclaw-迁移)
- [常见问题](#常见问题)
- [资源链接](#资源链接)

---

## 🎯 项目简介

**Hermes Agent** 是由 [Nous Research](https://www.nousresearch.com/) 开发的开源、自我进化的 AI Agent。它于 **2026 年 2 月 25 日** 首次发布，以 MIT 许可证开源，已合并 **1,000+ Pull Requests**，在 GitHub 上获得 **16,800+ Stars**。

### Hermes 与其他 Agent 有什么不同？

大多数 AI Agent 在每次对话后就会"失忆"——你的偏好、项目上下文、踩过的坑，下次全部从零开始。Hermes Agent 打破了这个限制：

| 特性 | 传统 Agent | Hermes Agent |
|------|-----------|-------------|
| **记忆** | 单次会话，对话结束即遗忘 | 跨会话持久记忆 + 会话搜索 |
| **技能** | 预装固定工具 | 自动从经验中创建和改进技能 |
| **学习** | 被动执行指令 | 主动学习循环——越用越懂你 |
| **模型** | 绑定单一提供商 | 18+ 模型提供商，随时切换 |
| **部署** | 本地或云端二选一 | 6 种终端后端，本地/Docker/SSH/云端无缝切换 |
| **消息** | 仅 CLI 或仅网页 | 15+ 消息平台统一网关 |

### 核心理念

> **"一个随你成长的 Agent"** —— 它学习你的项目，构建自己的技能，随时随地触达你。

Hermes Agent 的独特之处在于其 **封闭学习循环 (Closed Learning Loop)**：

1. **创建技能** — 从解决问题的经验中自动生成 Skill Document
2. **改进技能** — 在使用过程中不断优化已有的技能文档
3. **持久化知识** — 通过多层记忆系统保存关键信息
4. **搜索过去** — 用 FTS5 全文搜索回顾历史会话
5. **构建用户模型** — 跨会话了解你的偏好和工作习惯

---

## ⚡ 核心特性

### 1. 学习循环系统

Hermes Agent 是目前唯一内置完整学习循环的 AI Agent：

```
用户请求 → Agent 处理 → 生成经验 → 提炼为 Skill Document
                                          ↓
使用技能 ← 持久化存储 ← 优化技能 ← 评估效果
```

- 📝 **自动技能创建** — 每次成功解决问题后，自动生成可复用的技能文档
- 🔄 **持续改进** — 在后续使用中不断优化技能，越用越精准
- 🧠 **经验积累** — 记住什么方法有效、什么方法无效，避免重复犯错
- 👤 **用户建模** — 学习你的沟通风格、技术偏好、工作习惯

### 2. 多层记忆系统

Hermes Agent 拥有 4 层记忆架构，从即时到长期：

| 层级 | 类型 | 存储 | 容量 | 说明 |
|------|------|------|------|------|
| **L1** | 短期记忆 | 系统提示词 | 会话内 | 当前对话上下文 |
| **L2** | 代理笔记 | `MEMORY.md` | 2,200 字符 (~800 tokens) | 环境信息、项目约定、经验教训 |
| **L3** | 用户画像 | `USER.md` | 1,375 字符 (~500 tokens) | 你的偏好、沟通风格、期望 |
| **L4** | 会话搜索 | SQLite + FTS5 | 无限 | 所有历史会话的全文本搜索 |

此外，还支持 **8 种外部记忆提供商**，提供知识图谱、语义搜索、自动事实提取等高级能力：

- Honcho、OpenViking、Mem0、Hindsight
- Holographic、RetainDB、ByteRover、Supermemory

### 3. 技能生态

Hermes Agent 拥有 **644+ 技能**，横跨 **4 个注册源、16 个类别**：

| 注册源 | 数量 | 说明 |
|--------|------|------|
| 内置技能 | 78 | 核心工具，开箱即用 |
| 可选技能 | 45 | 按需安装的扩展能力 |
| 社区技能 | 521 | 来自全球开发者贡献 |
| 外部注册 | 不限 | 支持 `skills.sh`、`agentskills.io` 等标准 |

**技能类别覆盖**：文件操作、Web 浏览、终端执行、代码分析、邮件管理、日程安排、数据处理等 16 大类。

### 4. 多模型支持

Hermes Agent 支持 **18+ 模型提供商**，真正的模型无关设计：

| 提供商 | 类型 | 配置方式 |
|--------|------|----------|
| Nous Portal | 订阅制，零配置 | OAuth 登录 |
| Anthropic (Claude) | API Key / Claude Code 认证 | API Key 或 OAuth |
| OpenAI (GPT) | API / ChatGPT OAuth | API Key 或设备认证 |
| OpenRouter | 多模型路由 | API Key |
| DeepSeek | 国内直连 | `DEEPSEEK_API_KEY` |
| Z.AI (智谱 GLM) | 国内直连 | `GLM_API_KEY` |
| Kimi / Moonshot | 国内直连 | `KIMI_API_KEY` |
| MiniMax / MiniMax China | 国内直连 | `MINIMAX_API_KEY` |
| Alibaba Cloud (通义千问) | 国内直连 | `DASHSCOPE_API_KEY` |
| Hugging Face | 20+ 开源模型 | `HF_TOKEN` |
| GitHub Copilot | 订阅制 | OAuth |
| Vercel AI Gateway | 网关路由 | `AI_GATEWAY_API_KEY` |
| Custom Endpoint | VLLM/SGLang/Ollama | 自定义 URL + Key |

> **最低要求**：模型需支持 **64,000 tokens** 以上上下文窗口。

### 5. 消息网关

通过 **统一网关** 连接 **15+ 消息平台**，随时随地与 Agent 对话：

| 平台 | 类型 | 说明 |
|------|------|------|
| **Telegram** | 即时通讯 | 最推荐，配置简单 |
| **Discord** | 社交通讯 | 支持语音频道 |
| **Slack** | 办公协作 | 团队场景首选 |
| **WhatsApp** | 即时通讯 | 扫码即用 |
| **Signal** | 隐私通讯 | 端到端加密 |
| **Matrix** | 去中心化通讯 | 开源生态 |
| **Email** | 邮件 | SMTP 收发 |
| **SMS** | 短信 | 通过 Twilio |
| **飞书 (Feishu)** | 办公协作 | 国内企业首选 |
| **钉钉 (DingTalk)** | 办公协作 | 国内企业常用 |
| **企业微信** | 办公协作 | 微信生态 |
| **Home Assistant** | 智能家居 | IoT 控制 |

### 6. 终端后端

6 种终端执行后端，满足不同安全隔离需求：

| 后端 | 说明 | 适用场景 |
|------|------|----------|
| **Local** | 本地执行 | 个人开发机 |
| **Docker** | 容器隔离 | 安全沙箱 |
| **SSH** | 远程服务器 | 服务器管理 |
| **Daytona** | 云开发环境 | 团队协作 |
| **Singularity** | HPC 容器 | 科研计算 |
| **Modal** | Serverless | 按需计算 |

---

## 🏗️ 系统架构

### 核心流程

```
用户 → Hermes (CLI / 消息网关 / ACP编辑器) → Agent 循环 → 工具调用 → 终端后端
                ↓                                ↑
           记忆系统 ←── 学习循环 ←── 经验提取 ←──┘
                ↓
        Skill Documents / MEMORY.md / USER.md
```

### 架构分层

```
┌──────────────────────────────────────────────────────────────┐
│                       交互层                                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │   CLI    │  │ 消息网关  │  │ ACP编辑器 │  │ 语音模式  │     │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘     │
└──────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────┐
│                      Agent 循环                                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │  感知    │  │  推理    │  │  行动    │  │  学习    │     │
│  │ 用户输入  │  │ LLM推理  │  │ 工具调用  │  │ 经验提取  │     │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘     │
└──────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────┐
│                      工具层                                    │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐     │
│  │ 终端  │ │ 文件  │ │ 浏览器│ │ 搜索  │ │ 代码  │ │ MCP  │     │
│  │ 执行  │ │ 操作  │ │ 访问  │ │ 引擎  │ │ 执行  │ │ 服务  │     │
│  └──────┘ └──────┘ └──────┘ └──────┘ └──────┘ └──────┘     │
└──────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────┐
│                    记忆 & 学习层                               │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │MEMORY.md │  │ USER.md  │  │ 会话搜索  │  │ 技能文档  │     │
│  │ 代理笔记  │  │ 用户画像  │  │ FTS5+LLM │  │ 自动创建  │     │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘     │
└──────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────┐
│                    终端后端 & 模型层                            │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐     │
│  │ Local│ │Docker│ │ SSH  │ │Daytona│ │Modal │ │ 18+  │     │
│  │      │ │      │ │      │ │      │ │      │ │模型商│     │
│  └──────┘ └──────┘ └──────┘ └──────┘ └──────┘ └──────┘     │
└──────────────────────────────────────────────────────────────┘
```

---

## 📥 安装部署

### 前置要求

| 要求 | 版本 | 说明 |
|------|------|------|
| **操作系统** | Linux / macOS / WSL2 / Android (Termux) | Windows 需通过 WSL2 |
| **网络** | 需访问公网 | 安装时需从 GitHub 下载 |
| **上下文窗口** | ≥ 64K tokens | 模型最低要求 |
| **时间** | 5-10 分钟 | 首次安装 |

### 一键安装（推荐）

```bash
# Linux / macOS / WSL2 / Android (Termux)
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

安装完成后，重新加载 shell：

```bash
source ~/.bashrc  # 或 source ~/.zshrc
```

### pip 安装

```bash
# 基础安装
pip install hermes-agent

# 带语音模式
pip install "hermes-agent[voice]"

# 带 ACP 编辑器集成
pip install -e ".[acp]"

# 全功能安装
pip install "hermes-agent[voice,acp]"
```

### Windows 用户注意

Hermes Agent 目前不直接支持原生 Windows，需要通过 **WSL2** 运行：

1. 安装 WSL2：`wsl --install`
2. 进入 WSL2 终端
3. 执行上述一键安装命令

### 验证安装

```bash
# 检查版本
hermes --version

# 运行诊断
hermes doctor

# 查看配置
hermes config list
```

---

## 🚀 快速开始

### Step 1: 配置模型提供商

```bash
# 交互式选择模型提供商
hermes model

# 或使用完整设置向导
hermes setup
```

以选择 Anthropic Claude 为例：

```bash
hermes model
# → 选择 "Anthropic"
# → 输入 API Key 或使用 Claude Code 认证
```

以选择国内模型为例：

```bash
# 设置智谱 AI (GLM)
export GLM_API_KEY="your_key_here"

# 设置 DeepSeek
export DEEPSEEK_API_KEY="your_key_here"

# 设置通义千问
export DASHSCOPE_API_KEY="your_key_here"

# 设置 Kimi
export KIMI_API_KEY="your_key_here"
```

### Step 2: 开始对话

```bash
hermes
```

启动后会看到欢迎界面，显示当前模型、可用工具和技能。直接输入消息即可。

### Step 3: 体验核心功能

```bash
# 让 Agent 使用终端
❯ 查看我的磁盘使用情况，列出最大的 5 个目录

# 使用斜杠命令
❯ /help          # 查看所有命令
❯ /tools         # 列出可用工具
❯ /model         # 切换模型
❯ /save          # 保存对话

# 恢复上一次会话
hermes --continue
# 或简写
hermes -c
```

### Step 4: 安装技能

```bash
# 搜索技能
hermes skills search kubernetes
hermes skills search react --source skills-sh

# 安装技能
hermes skills install official/security/1password
hermes skills install skills-sh/vercel-labs/json-render

# 在聊天中安装
❯ /skills    # 浏览和安装技能
```

---

## 🧠 记忆系统详解

### 工作原理

Hermes Agent 的记忆由两个核心文件组成，存储在 `~/.hermes/memories/` 中：

| 文件 | 用途 | 容量限制 |
|------|------|----------|
| **MEMORY.md** | Agent 的个人笔记——环境信息、项目约定、学到的经验 | 2,200 字符 (~800 tokens) |
| **USER.md** | 用户画像——你的偏好、沟通风格、期望 | 1,375 字符 (~500 tokens) |

这些记忆在每次会话开始时以 **冻结快照** 方式注入系统提示词：

```
══════════════════════════════════════════════
MEMORY (your personal notes) [67% — 1,474/2,200 chars]
══════════════════════════════════════════════
User's project is a Rust web service at ~/code/myapi using Axum + SQLx§
This machine runs Ubuntu 22.04, has Docker and Podman installed§
User prefers concise responses, dislikes verbose explanations
```

### 记忆管理工具

Agent 通过 `memory` 工具自动管理记忆：

| 操作 | 说明 |
|------|------|
| `add` | 添加新记忆条目 |
| `replace` | 替换已有条目（子串匹配） |
| `remove` | 删除不再相关的条目 |

**子串匹配示例**：

```python
# 如果记忆中有 "User prefers dark mode in all editors"
memory(action="replace", target="memory",
       old_text="dark mode",
       content="User prefers light mode in VS Code, dark mode in terminal")
```

### 什么该记，什么不记

| ✅ 应该记 | ❌ 不该记 |
|-----------|----------|
| 用户偏好："我更喜欢 TypeScript" | 琐碎信息："用户问了 Python 的问题" |
| 环境信息："服务器运行 Debian 12 + PostgreSQL 16" | 常识："Python 3.12 支持 f-string 嵌套" |
| 项目约定："项目使用 tabs，120 字符行宽" | 大段代码/日志/数据表 |
| 经验教训："staging 服务器 SSH 端口是 2222" | 临时文件路径、一次性调试上下文 |

### 会话搜索

除了 MEMORY.md 和 USER.md，Agent 还能搜索所有历史对话：

```bash
# 列出历史会话
hermes sessions list

# 在会话中搜索过去讨论的内容
❯ 我们上周讨论的那个 Python 项目是怎么配置的？
```

**会话搜索 vs 持久记忆对比**：

| 特性 | 持久记忆 | 会话搜索 |
|------|----------|----------|
| **容量** | ~1,300 tokens | 无限（所有会话） |
| **速度** | 即时（在提示词中） | 需搜索 + LLM 总结 |
| **使用场景** | 始终需要的关键信息 | 查找特定历史对话 |
| **管理方式** | Agent 手动维护 | 自动——所有会话自动存储 |

### 外部记忆提供商

```bash
# 选择并配置外部记忆提供商
hermes memory setup

# 查看当前激活的记忆提供商
hermes memory status
```

支持的 8 种外部提供商：Honcho、OpenViking、Mem0、Hindsight、Holographic、RetainDB、ByteRover、Supermemory

---

## 🛠️ 技能生态

### 技能注册源

| 注册源 | 数量 | 说明 |
|--------|------|------|
| **内置** (Built-in) | 78 | 核心功能，开箱即用 |
| **可选** (Optional) | 45 | 按需安装的扩展 |
| **社区** (Community) | 521 | 全球开发者贡献 |
| **外部** (External) | 不限 | `skills.sh`、`agentskills.io` 等 |

### 技能搜索与安装

```bash
# 搜索技能
hermes skills search <关键词>
hermes skills search <关键词> --source skills-sh        # 从 skills.sh 搜索
hermes skills search <URL> --source well-known           # 从网站发现技能

# 安装技能
hermes skills install <技能路径>
hermes skills install <技能路径> --force                  # 覆盖安装（第三方技能）

# 在聊天中管理
❯ /skills    # 浏览、搜索、安装技能
```

### 技能来源推荐

| 来源 | 网址 | 说明 |
|------|------|------|
| `skills.sh` | https://skills.sh | 20,000+ 模块化技能，支持 `npx skills add` |
| Anthropic 官方 | https://github.com/anthropics/skills | 官方技能模板和最佳实践 |
| Composio 精选 | https://github.com/ComposioHQ/awesome-claude-skills | 1000+ 应用集成方案 |
| `agentskills.io` | https://agentskills.io | 开放标准技能注册 |

### Skill Document（技能文档）

Hermes Agent 的技能以 Markdown 文档形式存储，Agent 在解决问题的过程中会自动创建和改进技能文档：

1. **触发** — 遇到新类型的问题
2. **创建** — 自动生成 Skill Document 记录解决方案
3. **使用** — 下次遇到类似问题直接调用
4. **改进** — 在使用过程中根据效果优化

---

## 📱 消息网关

### 快速配置

```bash
# 交互式配置消息平台
hermes gateway setup

# 启动网关
hermes gateway
```

### Telegram 配置示例

1. 在 Telegram 搜索 `@BotFather`，创建新 Bot
2. 获取 Bot Token
3. 运行 `hermes gateway setup`，选择 Telegram
4. 输入 Bot Token
5. 完成！直接在 Telegram 与 Agent 对话

### Discord 配置示例

1. 前往 [Discord Developer Portal](https://discord.com/developers/applications)
2. 创建新应用，添加 Bot
3. 获取 Bot Token
4. 运行 `hermes gateway setup`，选择 Discord
5. 邀请 Bot 到你的服务器

### 定时任务（Cron）

Hermes 内置自然语言定时任务：

```
❯ 每天早上 9 点，查看 Hacker News 上的 AI 新闻，把摘要发到 Telegram

❯ 每周五下午 5 点，总结本周的 Git 提交记录

❯ 每 30 分钟检查一次服务器状态，异常时通知我
```

Agent 会自动创建 cron 任务，通过消息网关在指定平台推送结果。

---

## 💻 终端后端

### 切换终端后端

```bash
# 本地执行（默认）
hermes config set terminal.backend local

# Docker 容器隔离（推荐用于安全场景）
hermes config set terminal.backend docker

# 远程 SSH 服务器
hermes config set terminal.backend ssh

# Modal Serverless（按需计算）
hermes config set terminal.backend modal
```

### Docker 沙箱配置

```bash
# 启用 Docker 后端
hermes config set terminal.backend docker

# Agent 的所有命令将在隔离容器中执行
# 不影响宿主机文件系统
```

---

## ⚙️ 进阶配置

### 配置文件位置

```
~/.hermes/
├── config.yaml          # 主配置文件
├── memories/
│   ├── MEMORY.md        # Agent 记忆笔记
│   └── USER.md          # 用户画像
├── skills/              # 已安装的技能
├── state.db             # 会话数据库 (SQLite + FTS5)
└── workspace/           # 工作目录
```

### 主配置示例

```yaml
# ~/.hermes/config.yaml

# 模型配置
model:
  provider: anthropic
  name: claude-sonnet-4-5

# 记忆配置
memory:
  memory_enabled: true
  user_profile_enabled: true
  memory_char_limit: 2200    # ~800 tokens
  user_char_limit: 1375      # ~500 tokens

# 终端后端
terminal:
  backend: local

# MCP 服务器
mcp_servers:
  github:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-github"]
    env:
      GITHUB_PERSONAL_ACCESS_TOKEN: "ghp_xxx"
```

### 工具配置

```bash
# 交互式配置可用工具
hermes tools

# 管理工具集
hermes config set tools.browser.enabled true
hermes config set tools.terminal.enabled true
hermes config set tools.file_operations.enabled true
```

### MCP 服务器集成

```yaml
# 在 ~/.hermes/config.yaml 中添加
mcp_servers:
  github:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-github"]
    env:
      GITHUB_PERSONAL_ACCESS_TOKEN: "ghp_xxx"
  filesystem:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/dir"]
```

### ACP 编辑器集成

Hermes 可以作为 ACP 服务器在编辑器中使用：

```bash
# 安装 ACP 支持
pip install -e ".[acp]"

# 启动 ACP 服务器
hermes acp
```

支持 **VS Code**、**Zed**、**JetBrains** 等编辑器。

### 语音模式

```bash
# 安装语音依赖
pip install "hermes-agent[voice]"
pip install faster-whisper    # 推荐的本地语音转文字

# 在 CLI 中启用
❯ /voice on

# 语音输入：Ctrl+B 开始录音
# 语音输出：/voice tts 开启文字转语音
```

---

## 🔄 从 OpenClaw 迁移

Hermes Agent 支持从 OpenClaw 自动导入：

```bash
# 自动迁移 OpenClaw 数据
# 包括：设置、记忆、技能、API 密钥
hermes migrate --from openclaw
```

### 迁移内容映射

| OpenClaw | Hermes Agent |
|----------|-------------|
| `SOUL.md` | `USER.md`（用户画像） |
| `MEMORY.md` | `MEMORY.md`（代理笔记） |
| Skills | Skill Documents |
| API Keys | `config.yaml` |
| 会话历史 | `state.db`（SQLite + FTS5） |

---

## 🔧 常见问题

### Q1: 安装失败

```bash
# 清理并重试
rm -rf ~/.hermes
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash

# 或使用 pip
pip install --upgrade hermes-agent
```

### Q2: 模型连接失败

```bash
# 检查 API Key 是否正确
hermes doctor

# 重新配置模型
hermes model

# 检查网络连接
# 国外模型（Claude、GPT）需要国际网络
# 国内模型（智谱、DeepSeek、通义千问、Kimi）直连即可
```

### Q3: 记忆满了怎么办

Agent 会自动管理——当记忆超过 80% 容量时，会自动合并或替换旧条目。你也可以手动管理：

```
❯ 清理掉记忆中过期的项目信息
❯ 把关于 Python 项目的几条记忆合并成一条
```

### Q4: 技能安装失败

```bash
# 检查技能是否存在
hermes skills search <技能名>

# 强制安装第三方技能
hermes skills install <技能路径> --force

# 注意：--force 只能覆盖 non-dangerous 策略阻止，不能绕过 dangerous 安全扫描
```

### Q5: 消息网关收不到消息

```bash
# 检查网关状态
hermes gateway status

# 重新配置
hermes gateway setup

# 检查日志
hermes gateway logs
```

### Q6: Windows 用户安装问题

Windows 需通过 WSL2 运行：

1. 安装 WSL2：`wsl --install`
2. 进入 WSL 终端
3. 执行 Linux 安装命令
4. 如遇权限问题：`sudo apt update && sudo apt upgrade -y`

### Q7: 上下文窗口不够

```bash
# 错误提示：Model context window too small
# 解决：选择支持 64K+ tokens 的模型

# 本地模型需要手动设置
# llama.cpp: --ctx-size 65536
# Ollama: 在 Modelfile 中设置 PARAMETER num_ctx 65536
```

### Q8: 诊断工具

```bash
# 完整诊断
hermes doctor

# 查看配置
hermes config list

# 更新到最新版
hermes update
```

---

## 📚 资源链接

### 官方资源

- **GitHub**: https://github.com/nousresearch/hermes-agent
- **官网**: https://hermes-agent.nousresearch.com
- **文档**: https://hermes-agent.nousresearch.com/docs
- **快速开始**: https://hermes-agent.nousresearch.com/docs/getting-started/quickstart
- **PyPI**: https://pypi.org/project/hermes-agent/

### 社区

- **GitHub Discussions**: https://github.com/nousresearch/hermes-agent/discussions
- **Reddit**: https://www.reddit.com/r/nousresearch/
- **Discord**: 通过 GitHub README 获取邀请链接

### 相关文章

- [Hermes Agent: Self-Improving AI with Persistent Memory | YUV.AI](https://yuv.ai/blog/hermes-agent)
- [Inside Hermes Agent: How a Self-Improving AI Agent Actually Works](https://mranand.substack.com/p/inside-hermes-agent-how-a-self-improving)
- [Hermes Agent vs OpenClaw in 2026](https://mlearning.substack.com/p/hermes-agent-vs-openclaw-in-2026)
- [The Agent Landscape in 2026: A Compass Through the Noise](https://medium.com/data-science-collective/the-agent-landscape-in-2026-a-compass-through-the-noise-7c638e4aebe1)

---

## 📝 版本历史

| 版本 | 日期 | 主要更新 |
|------|------|----------|
| **v0.1.0** | 2026-02-25 | 首次发布——核心 Agent 循环、记忆系统 |
| **v0.4.0** | 2026-03-23 | 平台扩展——OpenAI 兼容 API 服务器、6 项新功能 |
| **v0.5.0** | 2026-03-28 | 技能系统增强、多实例支持 |
| **v0.6.0** | 2026-03-30 | 多实例 Profile、MCP 服务器模式、Docker 改进 |
| **v0.8.0** | 2026-04-08 | 智能发布——后台任务自动通知、MiMo v2 免费集成 |

---

## 🆚 与其他 Agent 对比

| 特性 | Hermes Agent | OpenClaw | Ruflo (Claude Flow) |
|------|-------------|----------|-------------------|
| **开源协议** | MIT | 开源 | MIT |
| **学习循环** | ✅ 内置 | ❌ | ❌ |
| **持久记忆** | ✅ 多层 | ✅ MEMORY.md | ✅ AgentDB |
| **模型无关** | ✅ 18+ 提供商 | ✅ 多模型 | ✅ 多模型 |
| **消息平台** | ✅ 15+ | ✅ 多平台 | ❌ CLI/MCP |
| **技能生态** | ✅ 644+ | ✅ 16,878+ ClawHub | ✅ 137+ |
| **终端隔离** | ✅ 6 种后端 | ❌ | ❌ |
| **语音模式** | ✅ 内置 | ❌ | ❌ |
| **从 OpenClaw 迁移** | ✅ 自动 | — | — |
| **MCP 支持** | ✅ | ✅ | ✅（原生） |

---

**🤖 享受使用 Hermes Agent！它是会随你一起成长的 AI Agent —— 用得越多，它就越懂你。**

如有问题，欢迎在 [GitHub Discussions](https://github.com/nousresearch/hermes-agent/discussions) 提问。
