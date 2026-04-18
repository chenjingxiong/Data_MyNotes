# Agent-Browser 完全指南

> 官方仓库：https://github.com/vercel-labs/agent-browser
> 官方网站：https://agent-browser.dev
> 许可协议：Apache-2.0

---

## 项目简介

**agent-browser** 是由 **Vercel Labs** 开发的 AI 原生浏览器自动化 CLI 工具，采用 **Rust** 从零构建。与传统的浏览器自动化工具（如 Playwright、Puppeteer）面向人类开发者编写脚本不同，agent-browser 的核心设计理念是**让 LLM / AI Agent 直接通过命令行操控浏览器**。

它的核心创新在于**引用选择器系统（Ref Selector System）**——AI 不需要理解复杂的 CSS/XPath 选择器，只需通过 `@e1`、`@e2` 这样的引用 ID 即可精准操作页面元素。

### 核心特性一览

| 特性 | 说明 |
|------|------|
| **🚀 Rust 原生 CLI** | 无需 Node.js，启动快、内存低、跨平台原生二进制 |
| **🤖 AI 优先** | 引用选择器 (`@e1`) + 可访问性树快照，LLM 天然友好 |
| **📸 快照与引用系统** | 获取页面结构化视图，用简短引用交互 |
| **🔐 安全认证** | 持久化 Profile、认证保险库（LLM 永远看不到密码） |
| **🌐 多平台** | macOS / Linux / Windows 原生二进制 |
| **☁️ 云浏览器** | 支持 Browserless、Browserbase、Browser Use |
| **📱 iOS 模拟** | 控制真实移动端 Safari |
| **🛡️ 安全边界** | 域名白名单、操作策略、输出长度限制 |

---

## 架构设计

### 整体架构

```
┌──────────────────────────────────────────────────────────────┐
│                       AI Agent (LLM)                         │
│                  (Claude / GPT / OpenClaw)                    │
└──────────────────────┬───────────────────────────────────────┘
                       │ CLI 命令 (stdin/stdout)
                       ▼
┌──────────────────────────────────────────────────────────────┐
│                    agent-browser (Rust)                       │
│                                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │
│  │ Snapshot  │  │  Ref     │  │ Session  │  │ Auth     │    │
│  │ Engine    │  │ Selector │  │ Manager  │  │ Vault    │    │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘    │
│                                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │
│  │ Security │  │ Profile  │  │ Cloud    │  │ Config   │    │
│  │ Policy   │  │ Manager  │  │ Provider │  │ Engine   │    │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘    │
└──────────────────────┬───────────────────────────────────────┘
                       │ CDP (Chrome DevTools Protocol)
                       ▼
┌──────────────────────────────────────────────────────────────┐
│                   Chromium (Chrome for Testing)               │
│              或 Browserless / Browserbase 云浏览器              │
└──────────────────────────────────────────────────────────────┘
```

### 为什么选择 Rust + CLI？

```
┌─────────────────────────────────────────────────────────────┐
│           传统方案 (Playwright/Puppeteer)                     │
│                                                             │
│   LLM → 生成脚本 → Node.js 运行时 → 浏览器                   │
│          ↑                                                  │
│          需要理解 API、等待策略、错误处理...                    │
│          上下文消耗大，调试困难                                │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│           agent-browser 方案                                  │
│                                                             │
│   LLM → CLI 命令 → Rust 二进制 → 浏览器                      │
│          ↑                                                  │
│          简单直观：open、snapshot、click @e1                  │
│          无需脚本，每步都是原子操作                            │
└─────────────────────────────────────────────────────────────┘
```

**关键区别**：传统工具需要 AI 生成完整脚本然后执行，agent-browser 让 AI 逐步操作，每一步都可以观察和调整。

---

## 安装

### 前置条件

- 无需 Node.js（agent-browser 是独立的 Rust 二进制文件）
- 需要网络连接下载 Chrome for Testing

### 方式一：npm 全局安装（推荐）

```bash
npm install -g agent-browser
agent-browser install  # 首次使用：下载 Chrome for Testing
```

### 方式二：Homebrew（macOS）

```bash
brew install agent-browser
agent-browser install
```

### 方式三：Cargo（Rust 生态）

```bash
cargo install agent-browser
agent-browser install
```

### 方式四：从源码构建

```bash
git clone https://github.com/vercel-labs/agent-browser
cd agent-browser
pnpm install
pnpm build
pnpm build:native
pnpm link --global
agent-browser install
```

### Linux 系统依赖

```bash
# 自动安装所有系统依赖
agent-browser install --with-deps

# 手动安装（Debian/Ubuntu）
sudo apt-get install libnss3 libnspr4 libatk1.0-0 libatk-bridge2.0-0 \
  libcups2 libdrm2 libxkbcommon0 libxcomposite1 libxdamage1 libxfixes3 \
  libxrandr2 libgbm1 libpango-1.0-0 libcairo2 libasound2
```

### 验证安装

```bash
agent-browser --version
# 输出类似：agent-browser 0.x.x
```

> [!tip] 国内加速
> Chrome for Testing 下载可能较慢，可设置代理：
> ```bash
> export HTTPS_PROXY=http://127.0.0.1:7890
> agent-browser install
> ```

---

## 核心概念

### 1. 快照与引用系统（Snapshot & Ref System）

这是 agent-browser 最核心的设计，也是区别于所有传统工具的关键特性。

**工作流程**：

```
步骤 1: agent-browser open https://example.com
步骤 2: agent-browser snapshot
        → 获取页面的可访问性树（Accessibility Tree）
        → 每个可交互元素被分配唯一引用 ID

输出示例：
┌─────────────────────────────────────────────┐
│ - heading "Welcome" [ref=e1] [level=1]      │
│ - paragraph "Sign in to continue" [ref=e2]   │
│ - textbox "Email" [ref=e3]                  │
│ - textbox "Password" [ref=e4]               │
│ - button "Sign In" [ref=e5]                 │
│ - link "Forgot password?" [ref=e6]          │
│ - link "Create account" [ref=e7]            │
│ - checkbox "Remember me" [ref=e8]           │
└─────────────────────────────────────────────┘

步骤 3: agent-browser fill @e3 "user@example.com"
步骤 4: agent-browser fill @e4 "mypassword"
步骤 5: agent-browser click @e5
步骤 6: agent-browser snapshot  ← 重新快照查看新页面
```

**为什么这对 AI 很重要？**

| 传统选择器 | agent-browser 引用 |
|-----------|-------------------|
| `#login-form > div:nth-child(3) > input[type=email]` | `@e3` |
| `//button[contains(text(), "Submit")]` | `@e5` |
| `document.querySelector('.btn-primary')` | `@e5` |
| LLM 需要理解 DOM 结构 | LLM 只需看引用列表 |

### 2. 会话（Session）

会话是 agent-browser 的浏览器实例管理单元：

```
┌──────────────────────────────────────────────────┐
│               Session（会话）                      │
│                                                  │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐ │
│  │ 浏览器实例  │  │ Cookie     │  │ localStorage│ │
│  │ (Chromium) │  │ Store      │  │ Store       │ │
│  └────────────┘  └────────────┘  └────────────┘ │
│                                                  │
│  特点：                                          │
│  • 每个会话完全隔离                               │
│  • 支持并发多个会话                               │
│  • 可跨命令保持状态                               │
└──────────────────────────────────────────────────┘
```

### 3. 持久化配置（Profile）

Profile 允许跨重启保持浏览器状态（Cookie、localStorage、扩展等）：

```bash
# 使用指定 Profile（登录状态会保留）
agent-browser --profile ~/.myapp-profile open myapp.com
```

### 4. 认证保险库（Auth Vault）

```
┌───────────────┐          ┌──────────────┐
│    AI Agent    │          │  Auth Vault   │
│   (LLM)       │          │  (加密存储)    │
│               │          │               │
│ 需要登录 →     │          │  • 用户名      │
│ 发起认证请求 →  │ ──────→ │  • 密码        │
│               │ ←────── │  • Token       │
│ 收到确认 ←     │          │  • API Key     │
│               │          │               │
│ ❌ 永远看不到   │          │ ✅ 本地加密     │
│    密码明文    │          │    存储        │
└───────────────┘          └──────────────┘
```

---

## 命令详解

### 导航命令

```bash
# 打开 URL
agent-browser open https://example.com

# 相对导航
agent-browser go back       # 后退
agent-browser go forward    # 前进
agent-browser reload        # 刷新

# 获取当前 URL
agent-browser get url
```

### 快照命令

```bash
# 获取完整快照（可访问性树）
agent-browser snapshot

# 仅获取可交互元素（推荐给 AI 使用）
agent-browser snapshot -i

# JSON 格式输出（方便程序解析）
agent-browser snapshot --json

# 组合：仅交互元素 + JSON
agent-browser snapshot -i --json
```

**快照输出格式**：

```text
# 完整快照示例
- banner [ref=e0]
  - navigation [ref=e1]
    - link "Home" [ref=e2]
    - link "Products" [ref=e3]
    - link "About" [ref=e4]
    - button "Menu" [ref=e5]
- main [ref=e6]
  - heading "Welcome to Our Store" [ref=e7] [level=1]
  - searchbox "Search products..." [ref=e8]
  - list [ref=e9]
    - listitem [ref=e10]
      - link "Product A - $29.99" [ref=e11]
    - listitem [ref=e12]
      - link "Product B - $49.99" [ref=e13]
```

### 交互命令

```bash
# 点击元素
agent-browser click @e5                      # 使用引用
agent-browser click "#submit"                # 使用 CSS 选择器
agent-browser find role button click --name "Submit"  # 使用语义定位

# 填写文本（会清空原有内容）
agent-browser fill @e3 "hello@example.com"
agent-browser fill "#email" "hello@example.com"

# 输入文本（追加模式）
agent-browser type @e3 "additional text"

# 按键
agent-browser press Enter
agent-browser press "Control+a"
agent-browser press Tab

# 选择下拉框
agent-browser select @e10 "Option 1"

# 悬停
agent-browser hover @e5

# 拖拽
agent-browser drag @e5 to @e10

# 上传文件
agent-browser upload @e8 "/path/to/file.pdf"
```

### 信息获取命令

```bash
# 获取元素文本
agent-browser get text @e1
agent-browser get text "h1"

# 获取当前 URL
agent-browser get url

# 获取页面标题
agent-browser get title

# 获取元素属性
agent-browser get attribute @e1 href
```

### 等待命令

```bash
# 等待元素出现
agent-browser wait @e5
agent-browser wait ".loaded"

# 等待网络空闲
agent-browser wait --load networkidle

# 等待导航完成
agent-browser wait --load load

# 等待指定时间（毫秒）
agent-browser wait --timeout 3000
```

### 截图与导出

```bash
# 截图（保存到文件）
agent-browser screenshot page.png

# 截图（直接输出到 stdout，base64）
agent-browser screenshot --stdout

# 全页截图
agent-browser screenshot --full-page page.png

# 导出 PDF
agent-browser pdf output.pdf

# 指定元素截图
agent-browser screenshot @e5 element.png
```

### 会话管理

```bash
# 默认会话（单会话模式）
agent-browser open https://example.com    # 自动创建默认会话
agent-browser snapshot                    # 在默认会话中操作

# 多会话隔离
agent-browser --session agent1 open https://site-a.com
agent-browser --session agent2 open https://site-b.com

agent-browser --session agent1 snapshot   # 查看 site-a
agent-browser --session agent2 snapshot   # 查看 site-b

# 列出活动会话
agent-browser session list

# 关闭指定会话
agent-browser --session agent1 close
```

### 关闭与清理

```bash
# 关闭当前浏览器
agent-browser close

# 关闭所有浏览器实例
agent-browser close --all
```

---

## 配置

### 配置文件 `agent-browser.json`

在项目根目录或用户目录创建配置文件：

```json
{
  "headed": false,
  "profile": "./browser-data",
  "proxy": "http://localhost:8080",
  "userAgent": "my-agent/1.0",
  "ignoreHttpsErrors": true,
  "viewport": {
    "width": 1920,
    "height": 1080
  },
  "timeout": 30000,
  "security": {
    "allowedDomains": ["example.com", "api.example.com"],
    "maxOutputLength": 50000
  }
}
```

### 配置优先级

```
环境变量 > CLI 标志 > 项目级配置 > 用户级配置
```

### 环境变量

```bash
# 浏览器 Profile 目录
export AGENT_BROWSER_PROFILE=~/.myapp-profile

# 云浏览器 API Key
export BROWSERBASE_API_KEY="your-key"
export BROWSERLESS_API_KEY="your-key"
export BROWSER_USE_API_KEY="your-key"

# 代理
export HTTP_PROXY=http://127.0.0.1:7890
export HTTPS_PROXY=http://127.0.0.1:7890
```

---

## 高级功能

### 云浏览器集成

当本地浏览器不够用时，agent-browser 支持无缝切换到云端浏览器：

```bash
# Browserbase（生产级云浏览器）
export BROWSERBASE_API_KEY="your-api-key"
agent-browser -p browserbase open https://example.com

# Browserless（无头浏览器即服务）
export BROWSERLESS_API_KEY="your-api-token"
agent-browser -p browserless open https://example.com

# Browser Use
export BROWSER_USE_API_KEY="your-api-key"
agent-browser -p browseruse open https://example.com
```

**适用场景**：

| 场景 | 本地浏览器 | 云浏览器 |
|------|-----------|---------|
| 开发调试 | ✅ | ❌ |
| CI/CD 流水线 | ⚠️ 需安装依赖 | ✅ 即开即用 |
| 大规模并发 | ❌ 本地资源有限 | ✅ 弹性扩容 |
| IP 地理分布 | ❌ 固定 IP | ✅ 全球 IP 池 |
| 反爬虫规避 | ⚠️ 容易被检测 | ✅ 指纹管理 |

### iOS 模拟器支持

```bash
# 列出可用设备
agent-browser device list

# 启动 iPhone 16 Pro 上的 Safari
agent-browser -p ios --device "iPhone 16 Pro" open https://example.com

# 移动端手势
agent-browser -p ios swipe up
agent-browser -p ios swipe down
agent-browser -p ios pinch zoom

# 移动端交互
agent-browser -p ios tap @e1
agent-browser -p ios long-press @e2
```

### 安全策略

agent-browser 内置了多层安全机制，特别适合生产环境中的 AI Agent 部署：

```json
// security-policy.json
{
  "allowedDomains": ["myapp.com", "api.myapp.com"],
  "blockedActions": ["delete", "remove"],
  "confirmRequired": ["navigate", "submit"],
  "maxOutputLength": 50000,
  "contentBoundaries": true
}
```

| 安全功能 | 说明 | 场景 |
|---------|------|------|
| **认证保险库** | 密码本地加密，LLM 看不到 | 生产环境登录 |
| **内容边界标记** | 用分隔符包裹页面输出 | 防止 LLM 混淆工具输出 |
| **域名白名单** | 限制导航到受信任域名 | 防止 Agent 访问恶意网站 |
| **操作策略** | 静态策略文件控制破坏性操作 | 防止误操作（删除数据等） |
| **操作确认** | 敏感操作需明确批准 | 关键业务操作 |
| **输出长度限制** | 防止上下文泛滥 | 大页面内容截断 |

---

## 与 AI 工具集成

### Claude Code 集成（推荐）

**方式一：作为 Skill 安装**

```bash
npx skills add vercel-labs/agent-browser
```

这会将技能安装到 `.claude/skills/agent-browser/SKILL.md`。

**方式二：在 CLAUDE.md 中添加指令**

```markdown
<!-- .claude/instructions.md 或 CLAUDE.md -->
## 浏览器自动化

使用 `agent-browser` 进行网页自动化。运行 `agent-browser --help` 查看所有命令。

核心工作流程：
1. `agent-browser open <url>` - 导航到页面
2. `agent-browser snapshot -i` - 获取带引用（@e1, @e2）的交互元素
3. `agent-browser click @e1` / `fill @e2 "text"` - 使用引用进行交互
4. 页面变更后重新快照
5. `agent-browser screenshot <path>` - 截图验证结果
```

### OpenClaw / 其他 AI Agent 集成

```bash
# 标准输入输出模式，适合集成到任何 AI 框架
agent-browser open https://example.com && agent-browser snapshot -i --json

# 组合命令流
agent-browser open https://example.com/login && \
agent-browser snapshot -i && \
agent-browser fill @e1 "user@example.com" && \
agent-browser fill @e2 "password" && \
agent-browser click @e3 && \
agent-browser snapshot -i
```

### 与 MCP（Model Context Protocol）结合

agent-browser 的 CLI 设计天然适配 MCP 架构，可以作为 MCP 工具注册：

```
┌─────────────────────────────────────────────────┐
│                  AI Agent                        │
│                                                 │
│   ┌─────────────┐  ┌─────────────────────┐     │
│   │ 其他 MCP 工具 │  │ agent-browser MCP   │     │
│   │ (文件/数据库) │  │ (浏览器操控)         │     │
│   └─────────────┘  └─────────────────────┘     │
│                                                 │
│         通过 MCP Protocol 统一调度                │
└─────────────────────────────────────────────────┘
```

---

## 实战场景

### 场景一：AI 驱动的表单填写

```bash
# 1. 打开注册页面
agent-browser open https://example.com/register

# 2. 获取表单结构
agent-browser snapshot -i
# 输出：
# - textbox "Full Name" [ref=e1]
# - textbox "Email" [ref=e2]
# - textbox "Password" [ref=e3]
# - checkbox "I agree to terms" [ref=e4]
# - button "Create Account" [ref=e5]

# 3. AI 填写表单
agent-browser fill @e1 "张三"
agent-browser fill @e2 "zhangsan@example.com"
agent-browser fill @e3 "SecurePass123!"
agent-browser click @e4    # 勾选同意条款
agent-browser click @e5    # 提交

# 4. 验证结果
agent-browser snapshot -i
agent-browser screenshot result.png
```

### 场景二：已登录状态下的数据抓取

```bash
# 使用持久化 Profile（已登录状态）
agent-browser --profile ~/.myapp-profile open https://myapp.com/dashboard

# 获取页面数据
agent-browser snapshot -i

# 提取特定信息
agent-browser get text @e7  # 获取订单数量
agent-browser get text @e8  # 获取账户余额

# 导出报表
agent-browser pdf monthly-report.pdf
agent-browser screenshot dashboard.png
```

### 场景三：多会话并行操作

```bash
# 同时操作多个网站
agent-browser --session shop1 open https://shop-a.com/product/123
agent-browser --session shop2 open https://shop-b.com/product/456

# 对比价格
agent-browser --session shop1 get text @e5  # shop-a 价格
agent-browser --session shop2 get text @e3  # shop-b 价格

# 在更便宜的店铺下单
agent-browser --session shop1 click @e7     # 加入购物车
agent-browser --session shop1 snapshot -i   # 确认操作
```

### 场景四：Claude Code Agent Loop

在 Claude Code 中，agent-browser 可以配合自主循环完成复杂任务：

```text
你: 去 GitHub 搜索 "agent-browser"，找到 Vercel 的仓库，截个图

Claude: 好的，我来操作浏览器完成这个任务。
  → agent-browser open https://github.com/search?q=agent-browser&type=repositories
  → agent-browser snapshot -i
  → [分析快照，找到第一个搜索结果的引用]
  → agent-browser click @e3
  → agent-browser wait --load networkidle
  → agent-browser screenshot github-result.png
  ✅ 已完成截图并保存。
```

---

## 与其他浏览器自动化方案对比

这是理解 agent-browser 定位的关键章节。我们将它与传统方案和新兴 AI 浏览器方案进行全面对比。

### 方案概览

| 方案 | 类型 | 开发者 | 核心理念 |
|------|------|--------|---------|
| **agent-browser** | CLI 工具 | Vercel Labs | AI 原生，引用选择器 |
| **Playwright** | 测试框架 + MCP | Microsoft | 跨浏览器测试，脚本驱动 |
| **Puppeteer** | Node.js 库 | Google | Chrome DevTools 自动化 |
| **browser-use** | Python 库 | browser-use AI | Playwright + LLM 决策 |
| **Steel Browser** | REST API 服务 | Steel Dev | 开箱即用的浏览器 API |
| **Stagehand** | SDK | Browserbase | AI Act/Extract/Observe |
| **Chrome DevTools MCP** | MCP 插件 | 社区 | 连接真实 Chrome |

### 核心架构对比

```
┌────────────────────────────────────────────────────────────────────┐
│                        架构对比图                                    │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  agent-browser:  AI ──CLI──→ Rust ──CDP──→ Chrome                  │
│                  • 逐步操作，每步可观察                               │
│                  • 引用选择器，AI 友好                                │
│                                                                    │
│  Playwright:     AI ──MCP──→ Node.js ──CDP──→ Chrome/Firefox/WebKit│
│                  • AI 通过 MCP 协议操控                              │
│                  • 也可编写脚本执行                                   │
│                                                                    │
│  browser-use:    LLM ──Python──→ Playwright ──CDP──→ Chrome        │
│                  • LLM 自主决策每一步                                │
│                  • Python 库，需编码集成                              │
│                                                                    │
│  Steel:          AI ──HTTP──→ API Server ──Puppeteer──→ Chrome     │
│                  • REST API，语言无关                                │
│                  • 服务器部署，适合分布式                              │
│                                                                    │
│  Stagehand:      AI ──SDK──→ Playwright ──CDP──→ Chrome            │
│                  • AI Act/Extract/Observe 原语                      │
│                  • 需 Browserbase 云浏览器                           │
│                                                                    │
│  Chrome DevTools: AI ──MCP──→ Chrome Extension ──CDP──→ Chrome    │
│                  • 直接控制真实 Chrome                               │
│                  • 保留所有登录状态                                   │
└────────────────────────────────────────────────────────────────────┘
```

### 功能详细对比

| 特性 | agent-browser | Playwright | Puppeteer | browser-use | Steel | Stagehand |
|------|:---:|:---:|:---:|:---:|:---:|:---:|
| **AI 原生设计** | ✅ | ⚠️ 需 MCP | ❌ | ✅ | ⚠️ 需封装 | ✅ |
| **引用选择器** | ✅ `@e1` | ❌ | ❌ | ❌ | ❌ | ❌ |
| **无 Node.js 依赖** | ✅ Rust | ❌ | ❌ | ❌ | ❌ | ❌ |
| **跨浏览器** | ❌ 仅 Chromium | ✅ 3 引擎 | ❌ 仅 Chrome | ❌ Chromium | ❌ Chrome | ❌ Chromium |
| **CLI 工具** | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| **SDK/API** | ❌ | ✅ Node/Python/Java/.NET | ✅ Node | ✅ Python | ✅ Node/Python | ✅ Node |
| **测试框架** | ❌ | ✅ 内置 | ❌ | ❌ | ❌ | ❌ |
| **代码录制** | ❌ | ✅ codegen | ❌ | ❌ | ❌ | ❌ |
| **云浏览器** | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ Browserbase |
| **iOS 测试** | ✅ 模拟器 | ❌ | ❌ | ❌ | ❌ | ❌ |
| **反检测** | ✅ | ❌ | ⚠️ 需插件 | ✅ | ✅ | ✅ |
| **持久化 Profile** | ✅ | ⚠️ Storage State | ❌ | ❌ | ✅ 会话 | ❌ |
| **认证保险库** | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **安全策略** | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **会话隔离** | ✅ | ✅ Browser Context | ⚠️ 手动 | ❌ | ✅ | ❌ |
| **MCP 支持** | ⚠️ 可集成 | ✅ 官方 MCP | ❌ | ❌ | ❌ | ❌ |
| **PDF 导出** | ✅ | ✅ Chromium only | ✅ | ❌ | ✅ | ❌ |
| **性能（启动速度）** | ⚡⚡⚡ | ⚡⚡ | ⚡⚡ | ⚡ | ⚡ | ⚡ |
| **学习曲线** | 低 | 中 | 中 | 中 | 低 | 中 |

### 适用场景对比

| 场景 | 最佳选择 | 原因 |
|------|---------|------|
| **AI Agent 操控浏览器** | agent-browser | 引用选择器 + CLI，LLM 最易使用 |
| **E2E 测试** | Playwright | 专用测试框架，跨浏览器，Trace Viewer |
| **快速网页截图/PDF** | Playwright CLI / agent-browser | 都支持一行命令完成 |
| **已登录网站操作** | Chrome DevTools MCP | 直接使用真实 Chrome Profile |
| **Python AI Agent** | browser-use | Python 原生，Playwright 底层 |
| **分布式浏览器集群** | Steel Browser | REST API + Docker，水平扩展 |
| **移动端 Safari 测试** | agent-browser | 唯一支持 iOS 模拟器 |
| **安全敏感操作** | agent-browser | 认证保险库 + 安全策略 |
| **大规模爬虫** | Steel + 云浏览器 | 反检测 + IP 轮换 |
| **CI/CD 集成** | Playwright | 官方 CI 支持，报告完善 |

### 性能与资源对比

| 指标 | agent-browser | Playwright | browser-use | Steel |
|------|:---:|:---:|:---:|:---:|
| 启动时间 | ~100ms | ~500ms | ~1s | ~2s (含服务) |
| 内存占用 | ~20MB | ~100MB | ~150MB | ~300MB (含服务) |
| 二进制大小 | ~10MB | ~50MB | N/A (Python) | N/A (Docker) |
| 运行时依赖 | 无 | Node.js | Python + Node | Docker |
| 首次安装 | +Chrome ~200MB | +3 浏览器 ~500MB | +Chrome +Python | Docker pull ~1GB |

### 代码风格对比

**同一个任务：打开网页 → 填写搜索框 → 点击搜索 → 截图**

#### agent-browser（CLI 命令流）

```bash
agent-browser open https://example.com
agent-browser snapshot -i                    # 查看可交互元素
agent-browser fill @e1 "agent-browser"       # 引用选择器
agent-browser click @e2                      # 点击搜索
agent-browser screenshot result.png
```

#### Playwright（TypeScript 脚本）

```typescript
import { chromium } from 'playwright';

const browser = await chromium.launch();
const page = await browser.newPage();
await page.goto('https://example.com');
await page.fill('#search', 'agent-browser');
await page.click('#search-button');
await page.screenshot({ path: 'result.png' });
await browser.close();
```

#### Playwright MCP（AI 操控）

```text
Claude: 请打开 https://example.com，搜索 "agent-browser" 并截图
→ MCP 工具调用: browser_navigate("https://example.com")
→ MCP 工具调用: browser_fill("#search", "agent-browser")
→ MCP 工具调用: browser_click("#search-button")
→ MCP 工具调用: browser_screenshot()
```

#### browser-use（Python + LLM）

```python
from browser_use import Agent
from langchain_openai import ChatOpenAI

agent = Agent(
    task="Go to example.com, search for 'agent-browser' and take a screenshot",
    llm=ChatOpenAI(model="gpt-4o"),
)
result = await agent.run()
```

#### Steel Browser（REST API）

```bash
# 创建会话
SESSION=$(curl -s -X POST http://localhost:3000/v1/sessions | jq -r '.id')

# 使用 Puppeteer/Playwright 连接到会话
# 需要额外编写脚本...
```

### 技术选型决策树

```
你想要什么？
│
├── 让 AI Agent 直接操控浏览器
│   ├── 只要 CLI，不写代码 → agent-browser ✅
│   ├── 需要 MCP 集成 → Playwright MCP ✅
│   ├── 用 Python 开发 → browser-use ✅
│   └── 需要控制真实 Chrome → Chrome DevTools MCP ✅
│
├── 编写浏览器自动化测试
│   └── → Playwright ✅（行业标准）
│
├── 大规模分布式浏览器操作
│   └── → Steel Browser ✅（REST API + Docker）
│
├── 快速截图 / PDF 导出
│   ├── 偶尔使用 → Playwright CLI ✅
│   └── AI Agent 集成 → agent-browser ✅
│
└── 安全敏感的浏览器操作
    └── → agent-browser ✅（认证保险库 + 安全策略）
```

---

## 常见问题

### Q: agent-browser 可以替代 Playwright 吗？

**不完全能**。两者定位不同：
- **agent-browser**：专为 AI Agent 设计的浏览器操控工具，强调 AI 友好性和安全性
- **Playwright**：通用的端到端测试框架，强调测试完整性和跨浏览器支持

如果你需要**编写和运行测试套件**，Playwright 仍然是最佳选择。如果你需要 **AI Agent 操控浏览器**，agent-browser 更合适。

> [!tip] 组合使用
> 两者可以共存：用 **Playwright** 编写和运行测试，用 **agent-browser** 让 AI 进行探索性测试和临时操作。

### Q: agent-browser 和 Playwright MCP 有什么区别？

| 方面 | agent-browser | Playwright MCP |
|------|:---:|:---:|
| 设计哲学 | AI 优先，从零构建 | 在现有测试框架上添加 MCP 层 |
| 选择器 | `@e1` 引用，LLM 零学习成本 | CSS/XPath，需要 LLM 理解 |
| 依赖 | 独立 Rust 二进制 | 需要 Node.js + Playwright |
| 测试能力 | ❌ | ✅ 完整测试框架 |
| 浏览器支持 | Chromium | Chromium + Firefox + WebKit |
| 安全功能 | ✅ 内置 | ❌ 需自行实现 |

### Q: 为什么没有 Firefox/WebKit 支持？

agent-browser 专注于 Chromium（Chrome for Testing），原因是：
1. **AI Agent 场景不需要跨浏览器**——不像测试需要验证兼容性
2. **Chromium 的 CDP 协议最完善**——提供最强大的程序化控制能力
3. **简化实现**——减少维护负担，专注核心功能

### Q: 如何处理需要登录的网站？

```bash
# 方式一：持久化 Profile（推荐）
agent-browser --profile ~/.myapp-profile open https://myapp.com/login
# 手动或通过命令完成登录
# 下次使用相同 Profile 时，登录状态已保留

# 方式二：认证保险库
# 使用 agent-browser 的 Auth Vault 功能
# 密码本地加密，LLM 看不到明文

# 方式三：手动录制
# 1. 用 headed 模式手动登录
# 2. 登录成功后，Profile 自动保存 Cookie
```

### Q: 安装失败怎么办？

```bash
# Chrome for Testing 下载失败
export HTTPS_PROXY=http://127.0.0.1:7890
agent-browser install

# Linux 缺少系统依赖
agent-browser install --with-deps

# 查看详细错误信息
RUST_LOG=debug agent-browser open https://example.com
```

---

## 速查表

```text
┌──────────────────────────────────────────────────────────────┐
│               agent-browser 速查表                             │
├──────────────────────────────────────────────────────────────┤
│ 安装                                                          │
│   npm install -g agent-browser     # 全局安装                  │
│   agent-browser install            # 下载 Chrome               │
│   agent-browser install --with-deps # 含系统依赖               │
├──────────────────────────────────────────────────────────────┤
│ 基本操作                                                       │
│   agent-browser open <url>         # 打开页面                  │
│   agent-browser snapshot [-i]      # 获取快照                  │
│   agent-browser click @e1          # 点击元素                  │
│   agent-browser fill @e1 "text"    # 填写文本                  │
│   agent-browser type @e1 "text"    # 追加文本                  │
│   agent-browser press Enter        # 按键                      │
│   agent-browser get text @e1       # 获取文本                  │
│   agent-browser wait @e1           # 等待元素                  │
├──────────────────────────────────────────────────────────────┤
│ 截图与导出                                                     │
│   agent-browser screenshot <file>  # 截图                     │
│   agent-browser screenshot --full-page <file>  # 全页截图      │
│   agent-browser pdf <file>         # 导出 PDF                  │
├──────────────────────────────────────────────────────────────┤
│ 会话与 Profile                                                │
│   agent-browser --session <name>   # 多会话隔离                │
│   agent-browser --profile <path>   # 持久化 Profile            │
│   agent-browser session list       # 列出会话                  │
│   agent-browser close [--all]      # 关闭浏览器                │
├──────────────────────────────────────────────────────────────┤
│ 云浏览器                                                       │
│   agent-browser -p browserbase     # Browserbase              │
│   agent-browser -p browserless     # Browserless              │
│   agent-browser -p ios             # iOS Safari               │
├──────────────────────────────────────────────────────────────┤
│ AI 集成工作流                                                  │
│   1. open <url>        # 导航                                  │
│   2. snapshot -i       # 获取交互元素引用                       │
│   3. click/fill @e1    # 使用引用操作                           │
│   4. snapshot -i       # 操作后重新快照                         │
│   5. screenshot <file> # 截图验证                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 相关资源

### 官方资源
- [GitHub 仓库](https://github.com/vercel-labs/agent-browser)
- [npm 包](https://www.npmjs.com/package/agent-browser)
- [官方文档](https://agent-browser.dev)

### 相关笔记
- [[AI技术/浏览器Agent技术/agent-browser 介绍与安装指南]] — 快速安装入门
- [[AI技术/浏览器Agent技术/Playwright CLI 完全指南]] — Playwright 详细指南
- [[AI技术/浏览器Agent技术/steel-browser 介绍与安装指南]] — Steel Browser API 方案
- [[AI技术/浏览器Agent技术/Lightpanda 介绍与集成指南]] — 轻量级浏览器引擎

### 竞品参考
- [browser-use](https://github.com/browser-use/browser-use) — Python AI 浏览器 Agent
- [Stagehand](https://github.com/browserbase/stagehand) — Browserbase 的 AI 浏览器 SDK
- [Playwright MCP](https://github.com/anthropics/mcp-playwright) — Anthropic 官方 Playwright MCP

---

*最后更新：2026-04-18*
