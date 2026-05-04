---
title: BrowserOS 完全指南
date: 2026-05-04
tags:
  - browser
  - AI
  - automation
  - MCP
  - Claude-Code
  - agent
  - browserOS
  - skill
  - workflow
  - guide
description: BrowserOS 是一款基于 Chromium 的 AI 原生浏览器，内置 MCP 服务器，可与 Claude Code、Gemini CLI 等 AI 编码代理深度集成。本文涵盖安装、核心功能、Claude Code 集成、Skill 安装等完整指南。
---

# BrowserOS 完全指南

> **一句话概述**：BrowserOS 是一款基于 Chromium 的开源 AI 原生浏览器，内置 MCP 服务器，可让 Claude Code、Gemini CLI 等 AI 编码代理直接控制浏览器并访问 40+ 外部服务。

官方文档：[docs.browseros.com](https://docs.browseros.com/)
官方下载：[browseros.com](https://browseros.com/)

---

## 1. 什么是 BrowserOS

BrowserOS 是一个**开源（AGPL-3.0）**、**隐私优先**的 AI 原生浏览器，构建于 Chromium 之上。它将自然语言指令转化为浏览器操作——点击、输入、导航、表单填写、数据提取等。

### 1.1 三种使用模式

| 模式                | 用途                               | 激活方式               |
| ----------------- | -------------------------------- | ------------------ |
| **Chat 模式**       | 对当前网页提问：总结文章、提取数据、翻译内容           | `Option+K`         |
| **Agent 模式**      | 描述任务后 AI 自动执行：点击、输入、导航、填表、多步骤工作流 | 点击工具栏 Assistant 按钮 |
| **Graph 模式（工作流）** | 构建可视化工作流图，支持并行执行、循环和条件分支         | 侧边栏 Workflows      |

### 1.2 核心特性一览

- **隐私优先**：AI 本地运行（支持 Ollama），数据不离开本机
- **开源**：100% AGPL-3.0 开源
- **Chrome 兼容**：所有 Chrome 扩展均可使用，一键导入书签、密码、设置
- **内置 MCP 服务器**：53+ 浏览器自动化工具 + 40+ 外部应用集成
- **40+ 应用集成**：Gmail、Google Calendar、Slack、Notion、GitHub、Jira 等通过 OAuth 连接
- **内置广告拦截**：基于 uBlock Origin，开箱即用

---

## 2. 安装与快速开始

### 2.1 下载安装

从 [browseros.com](https://browseros.com/) 获取最新版本，支持 **macOS、Windows、Linux**。

### 2.2 导入 Chrome 数据

安装后在 BrowserOS 中访问：

```
chrome://settings/importData
```

一键导入 Chrome 的书签、密码和设置。

### 2.3 配置 AI 模型

BrowserOS 内置了一个默认 AI 模型（每日有限额度供测试）。完整体验需配置自己的 API Key。

导航到：

```
chrome://browseros/settings
```

支持的 LLM 提供商：

| 提供商 | 特点 |
|--------|------|
| **Gemini** | 有免费额度 |
| **Claude / Anthropic** | 最佳 Agent 模式体验 |
| **OpenAI** | 高质量响应 |
| **OpenRouter** | 500+ 模型可选 |
| **Ollama** | 本地运行，完全免费 |
| **LM Studio** | 本地运行 |

### 2.4 AI 模型选择建议

**Chat 模式**：任何模型均可，包括本地 LLM
- Ollama / LM Studio — 免费，本地运行
- Gemini Flash — 快速且便宜
- Claude / GPT-4 — 最佳质量

**Agent 模式**：推荐 Claude Opus 4.5 或 Kimi K2.5

---

## 3. 核心功能详解

### 3.1 内置 MCP 服务器（浏览器自动化）

BrowserOS 内置了 **53+ MCP 工具**，按以下类别组织：

| 类别 | 工具数 | 示例工具 |
|------|--------|----------|
| 导航与标签页 | 8 | `navigate_page`, `new_page`, `close_page`, `list_pages` |
| 内容与观察 | 8 | `take_snapshot`, `get_page_content`, `get_page_links`, `get_dom`, `search_dom` |
| 交互与输入 | 14 | `click`, `fill`, `hover`, `drag`, `press_key`, `select_option` |
| 文件与导出 | 3 | `save_pdf`, `save_screenshot`, `download_file` |
| 窗口管理 | 5 | `list_windows`, `create_window`, `close_window` |
| 标签组 | 5 | `group_tabs`, `list_tab_groups`, `update_tab_group` |
| 书签 | 6 | `get_bookmarks`, `create_bookmark`, `search_bookmarks` |
| 历史记录 | 4 | `search_history`, `get_recent_history` |

### 3.2 Connect Apps（40+ 应用集成）

BrowserOS 通过 MCP 协议连接外部应用，**无需安装任何东西或管理 API Key**。

**支持的 40+ 应用涵盖：**

- **邮件**：Gmail、Outlook Mail
- **日历**：Google Calendar、Outlook Calendar、Cal.com
- **通信**：Slack、Discord、Microsoft Teams、WhatsApp
- **开发**：GitHub、GitLab
- **项目管理**：Linear、Jira、Asana、Monday、ClickUp
- **文档**：Notion、Google Docs、Google Sheets、Google Drive、Confluence
- **设计**：Figma
- **CRM**：Salesforce、HubSpot
- **电商**：Shopify、Stripe
- **存储**：Dropbox、OneDrive、Box
- **分析**：Mixpanel、PostHog
- **支持**：Zendesk、Intercom
- **搜索**：Brave Search
- **社交**：LinkedIn、YouTube
- **其他**：Airtable、WordPress、Cloudflare、Vercel、Resend、Postman、Supabase、Mem0

**连接方式**：

1. **智能推荐**：当你请求需要未连接的应用时，BrowserOS 自动弹出连接卡片
2. **手动连接**：侧边栏 → Connect Apps → Add built-in app → OAuth 登录

**跨应用工作流示例**：

```
"查找我最新邮件中的行动项，添加到 Notion 任务中"
"检查明天的日历，然后给 John 发邮件总结会议内容"
"从 Salesforce 拉取我的待办交易，在 Google Sheets 创建汇总表"
```

### 3.3 Cowork（本地文件协作）

Cowork 将浏览器自动化与本地文件操作结合，Agent 可以：
- 读写文件、编辑文件
- 执行 Shell 命令
- 搜索文件内容和文件名
- 浏览目录

**设置方式**：Agent 模式下点击提示输入框旁的 Cowork 下拉菜单 → 选择文件夹

> [!NOTE] 安全性
> Agent 被沙箱化到你选择的文件夹，无法访问父目录，随时可选择 "No folder" 撤销访问。

### 3.4 Workflows（可视化工作流）

将复杂的浏览器任务转化为可靠、可重复的自动化工作流。

**创建步骤**：
1. 侧边栏 → Workflows → + New Workflow
2. 在聊天面板中描述你想要的自动化
3. Agent 生成可视化工作流图
4. 通过对话进一步优化
5. Test Workflow 测试 → Save Changes 保存

**适用场景**：
- 数据录入自动化（从 Google Sheet 读取 → 提交到网页表单）
- LinkedIn 自动化外联
- 价格监控
- 批量退订邮件

### 3.5 Scheduled Tasks（定时任务）

自动定时运行 Agent 任务，支持每日、每小时或每分钟执行。

**创建方式**：
1. **对话中创建**：完成任务后 Agent 自动建议定时，或直接说 "Schedule this to run every morning"
2. **设置中创建**：侧边栏 → Scheduled Tasks → New Task

**运行机制**：
- 在隐藏浏览器窗口中运行，不干扰当前工作
- 笔合合盖时自动在下次打开时执行
- 每个任务保留最近 15 次运行历史
- 10 分钟超时限制

### 3.6 Chat & LLM Hub

- **Chat**：通过侧边面板快速访问 AI，`Option+K` 激活
- **LLM Hub**：最多同时对比 3 个模型的输出，`Option+L` 切换提供商

---

## 4. 与 Claude Code 集成（重点）

BrowserOS 是 **AI 编码代理的最佳浏览器搭档**。通过内置 MCP 服务器，Claude Code 可以：
- 完整控制浏览器（点击、输入、导航、截图）
- 测试 Web 应用、读取控制台错误、修复代码——一体化循环
- 直接访问 40+ 外部服务
- 从已认证页面提取数据

### 4.1 连接步骤

#### 步骤 1：获取 MCP URL

在 BrowserOS 中导航到：

```
chrome://browseros/mcp
```

或点击 Settings → BrowserOS as MCP。

复制页面上的 Server URL，格式如：

```
http://127.0.0.1:9239/mcp
```

#### 步骤 2：在 Claude Code 中添加 BrowserOS

```bash
claude mcp add --transport http browseros http://127.0.0.1:9239/mcp --scope user
```

> [!TIP] `--scope user` 表示全局可用。若只想在当前项目中使用，去掉此参数。

#### 步骤 3：验证连接

```bash
claude
> Open amazon.com in BrowserOS
```

#### 步骤 4：（可选）跳过确认提示

```bash
claude --dangerously-skip-permissions
```

此参数可跳过每个浏览器操作的确认弹窗，适合自动化工作流。

#### 移除连接

```bash
claude mcp remove browseros --scope user
```

### 4.2 BrowserOS MCP vs Chrome DevTools MCP

| 维度 | BrowserOS MCP | Chrome DevTools MCP |
|------|--------------|---------------------|
| 总工具数 | 53+ | 29 |
| 外部应用集成 | 40+ | 0 |
| 窗口管理 | ✅ | ❌ |
| 标签组 | ✅ | ❌ |
| 书签 & 历史记录 | ✅ | ❌ |
| 文件导出（PDF、截图、下载） | ✅ | ❌ |
| 内容提取（Markdown、链接、DOM） | ✅ | ❌ |
| 控制台 / 网络检查 | 即将支持 | ✅ |
| 性能追踪 & Lighthouse | 即将支持 | ✅ |
| 设置复杂度 | 复制 URL 即可 | 需调试端口 + Node 服务器 |
| 浏览器会话 | 真实浏览器（Cookie、扩展） | 调试附加会话（WebDriver 标记） |

### 4.3 示例 Prompt

连接成功后可尝试：

```
# 浏览器自动化
> Open github.com and search for "browser automation"

# 数据提取
> Go to my LinkedIn profile and extract my work experience

# 跨应用操作
> Check my Gmail for recent emails from John, then send a Slack message summarizing them

# Agentic 编码
> Test the login form on localhost:3000, report any console errors
```

---

## 5. Claude Code Skill 安装指南

Claude Code 的 **Skill** 系统允许你创建和安装可复用的能力模块，增强 AI 代理的功能。以下介绍如何安装和管理 Skill。

### 5.1 什么是 Claude Code Skill

Skill 是一组可复用的 Markdown 指令，AI 代理在识别到匹配任务时自动加载。可以类比为"菜谱"——写好步骤，代理在需要时按步骤执行。

Skill 遵循开放的 **Agent Skills 规范**，可跨任何支持该标准的 AI 代理使用。

### 5.2 BrowserOS 内置 Skill 系统

BrowserOS 自身也有 Skill 系统，两者互补：

**BrowserOS Skill 位置**：

| 系统 | 路径 |
|------|------|
| macOS / Linux | `~/.browseros/skills/` |
| Windows | `%USERPROFILE%\.browseros\skills\` |

**创建 BrowserOS Skill**：
1. 侧边栏 → Skills → New Skill
2. 填写名称、描述（触发条件）、内容（Markdown 指令）
3. 保存即生效

### 5.3 Everything Claude Code (ECC) Skill 安装

`everything-claude-code` 是一个庞大的 Claude Code Skill 集合，包含 **100+ 专业 Skill**，涵盖前端、后端、安全、测试、数据库等各个领域。

#### 5.3.1 安装方式

**方法一：通过 Claude Code CLI 安装**

```bash
# 安装 ECC（推荐）
claude skill add everything-claude-code
```

**方法二：手动安装**

1. 克隆或下载 [everything-claude-code](https://github.com/anthropics/everything-claude-code) 仓库
2. 将需要的 Skill 文件夹复制到项目或全局 Skill 目录

**方法三：通过 Claude Code 配置**

在项目的 `.claude/settings.json` 中添加：

```json
{
  "skills": {
    "everything-claude-code": {
      "enabled": true
    }
  }
}
```

#### 5.3.2 常用 ECC Skill 一览

以下是与 BrowserOS 和 Web 开发最相关的 Skill：

| Skill | 用途 |
|-------|------|
| `chrome-devtools-mcp` | Chrome DevTools 调试、网络请求分析、性能追踪 |
| `frontend-patterns` | React/Next.js 前端开发模式与最佳实践 |
| `backend-patterns` | 后端架构模式、API 设计、数据库优化 |
| `e2e-testing` | Playwright E2E 测试，页面对象模型 |
| `security-review` | 安全漏洞检测与修复 |
| `api-design` | REST API 设计模式 |
| `docker-patterns` | Docker 和 Docker Compose 模式 |
| `deployment-patterns` | CI/CD 管道、容器化、健康检查 |
| `python-patterns` | Pythonic 惯用法、PEP 8、类型提示 |
| `golang-patterns` | Go 惯用模式、并发、错误处理 |
| `tdd-workflow` | 测试驱动开发工作流 |
| `deep-research` | 多源深度研究 |
| `plan` | 需求重述、风险评估与实施计划 |

#### 5.3.3 Skill 文件格式

每个 Skill 是一个 `SKILL.md` 文件：

```yaml
---
name: my-custom-skill
description: 当用户需要执行 X 操作时使用此 Skill
metadata:
  display-name: My Custom Skill
  enabled: "true"
---

## 指令内容（Markdown）

1. 步骤一
2. 步骤二
3. 步骤三
```

**目录结构**：

```
my-custom-skill/
├── SKILL.md          # 主指令文件
├── scripts/          # 可执行脚本
│   └── helper.py
├── references/       # 详细参考文档
│   └── REFERENCE.md
└── assets/           # 模板、图片、数据文件
```

### 5.4 在 BrowserOS 中使用 Claude Code Skill

**最佳实践**：同时利用 BrowserOS Skill 和 Claude Code Skill：

1. **BrowserOS Skill** — 定义浏览器相关的自动化流程
2. **Claude Code Skill**（通过 MCP） — 定义编码、调试、测试工作流

两者可以协同工作：

```
# 在 BrowserOS Agent 模式中
"使用 Chrome DevTools MCP Skill 检查 localhost:3000 的性能问题"

# 在 Claude Code 中
"打开 BrowserOS，导航到我的 Web 应用，运行 Lighthouse 审计"
```

### 5.5 Chrome DevTools MCP Skill 安装

Chrome DevTools MCP 是 Claude Code 中浏览器调试的核心 Skill：

```bash
# 在 Claude Code 中安装 Chrome DevTools MCP
claude mcp add --transport stdio chrome-devtools-mcp -- npx @anthropic-ai/chrome-devtools-mcp --chrome-path /path/to/browseros
```

> [!NOTE]
> 如果你已经连接了 BrowserOS MCP，大多数场景下**不需要单独安装 Chrome DevTools MCP**。BrowserOS MCP 已内置更丰富的浏览器自动化工具。仅在需要 Lighthouse 审计、性能追踪、内存快照等调试功能时才需额外安装。

---

## 6. 其他 MCP 客户端连接

除了 Claude Code，BrowserOS MCP 还支持：

### 6.1 Gemini CLI

```bash
# 在 Gemini CLI 配置中添加 MCP 服务器
```

### 6.2 OpenClaw

直接在 OpenClaw 配置中添加 BrowserOS MCP URL。

### 6.3 Codex（OpenAI）

在 Codex 配置中添加 MCP 端点。

### 6.4 Claude Desktop

在 Claude Desktop 的 MCP 设置中添加 BrowserOS 服务器 URL。

---

## 7. 自定义 MCP 服务器连接

除了内置应用，BrowserOS 还支持连接任何 **MCP 兼容服务器**：

### 7.1 添加自定义 MCP

1. Settings → Connected Apps
2. 点击 Add custom app
3. 输入服务器 URL（如 `http://localhost:8000/sse`）和名称

### 7.2 连接 OAuth 保护的远程服务器

对于需要 OAuth 认证的远程 MCP 服务器（如 Atlassian Jira、GitHub）：

```bash
npx -y supergateway --stdio "npx -y mcp-remote https://mcp.atlassian.com/v1/sse" --port 8000
```

浏览器窗口会弹出登录页面。认证完成后，在 BrowserOS 中添加 `http://localhost:8000/sse` 作为自定义 MCP。

> [!WARNING] 保持终端运行
> 本地服务器负责处理认证并代理请求到远程 MCP 服务器，使用期间需要保持终端运行。

---

## 8. BrowserOS Skill 高级用法

### 8.1 编写优质 Skill 的技巧

1. **描述要具体**：包含关键词让 Agent 匹配。"当用户询问 PDF、表单或文档提取时" 比 "帮助处理文档" 好
2. **指令聚焦**：一个 Skill 做好一件事。复杂工作流拆分为多个 Skill
3. **包含示例**：展示好的输出是什么样的
4. **善用辅助文件**：详细参考放在 `references/` 目录中，Agent 按需加载，节省上下文

### 8.2 Skill 前置元数据字段

| 字段 | 必需 | 说明 |
|------|------|------|
| `name` | ✅ | 小写、连字符标识符（如 `morning-status-report`） |
| `description` | ✅ | 何时以及如何使用此 Skill |
| `license` | ❌ | Skill 许可证 |
| `compatibility` | ❌ | 环境要求 |
| `metadata` | ❌ | 额外字段如 display-name、enabled、version |
| `allowed-tools` | ❌ | 限制 Skill 可使用的工具（实验性） |

---

## 9. 完整工作流示例

### 示例 1：Agentic 编码循环

```
Claude Code + BrowserOS MCP
↓
1. Claude Code 修改代码
2. 通过 BrowserOS MCP 打开 localhost:3000
3. 截图查看效果
4. 读取控制台错误
5. 自动修复代码
6. 重复直到完美
```

### 示例 2：跨应用自动化

```
BrowserOS Agent 模式
↓
"从 Gmail 找到本周所有收据，整理到 Google Sheet 中"
↓
1. 连接 Gmail（OAuth）
2. 搜索收据邮件
3. 提取金额和商家信息
4. 连接 Google Sheets
5. 创建并填充表格
```

### 示例 3：定时竞品监控

```
BrowserOS Scheduled Tasks
↓
每天早上 8:00 自动运行：
"检查以下 5 个竞品网站的价格，与我们的价格对比，发送摘要到 #pricing Slack 频道"
```

---

## 10. 常见问题

### Q: BrowserOS 需要保持打开吗？
**A:** 是的，定时任务需要 BrowserOS 处于打开状态。如果合盖时错过了定时任务，打开后会自动执行。

### Q: BrowserOS MCP 和 Chrome DevTools MCP 可以同时使用吗？
**A:** 可以。在 Claude Code 中可以同时添加两个 MCP 服务器。BrowserOS MCP 覆盖更广（53 工具 + 40 应用），Chrome DevTools MCP 在调试和性能分析方面更强。

### Q: Skill 在哪里存储？
**A:**
- BrowserOS Skill：`~/.browseros/skills/`（macOS/Linux）或 `%USERPROFILE%\.browseros\skills\`（Windows）
- Claude Code Skill：项目 `.claude/` 目录或全局配置

### Q: 支持哪些 LLM？
**A:** Gemini（免费）、Claude/Anthropic、OpenAI、OpenRouter（500+ 模型）、Ollama、LM Studio。

### Q: 凭据安全吗？
**A:** 所有应用通过 OAuth 登录，BrowserOS 不存储密码。认证令牌本地安全存储，可随时撤销。

---

## 参考链接

- 官方文档：[docs.browseros.com](https://docs.browseros.com/)
- MCP 客户端集成：[docs.browseros.com/features/use-with-claude-code](https://docs.browseros.com/features/use-with-claude-code)
- Skill 文档：[docs.browseros.com/features/skills](https://docs.browseros.com/features/skills)
- Connect Apps：[docs.browseros.com/features/connect-mcps](https://docs.browseros.com/features/connect-mcps)
- 工作流：[docs.browseros.com/features/workflows](https://docs.browseros.com/features/workflows)
- 定时任务：[docs.browseros.com/features/scheduled-tasks](https://docs.browseros.com/features/scheduled-tasks)
- Cowork：[docs.browseros.com/features/cowork](https://docs.browseros.com/features/cowork)
- MCP 对比：[docs.browseros.com/comparisons/chrome-devtools-mcp](https://docs.browseros.com/comparisons/chrome-devtools-mcp)
- GitHub：[github.com/nicepkg/browseros](https://github.com/nicepkg/browseros)
- Discord 社区：[discord.gg/browseros](https://discord.gg/browseros)
