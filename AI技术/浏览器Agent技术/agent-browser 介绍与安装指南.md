# agent-browser - AI 智能体浏览器自动化工具

> 官方仓库：https://github.com/vercel-labs/agent-browser

> [!abstract] 快速导航
> 📖 **完整指南**：[[AI技术/浏览器Agent技术/Agent-Browser 完全指南]] — 包含架构设计、全部命令详解、实战场景及与 Playwright/browser-use/Steel 等方案的详细对比

## 项目简介

**agent-browser** 是由 Vercel Labs 开发的一款专为 AI 智能体设计的**无头浏览器自动化 CLI 工具**。它采用原生 Rust 构建，提供了高性能的浏览器控制能力，是 Claude、OpenClaw 以及其他 AI 工具进行网页自动化操作的理想选择。

### 核心特性

- **🚀 高性能原生 Rust CLI** - 无需 Node.js 依赖，执行速度快
- **🤖 AI 优先设计** - 支持引用选择器（refs）和语义定位器
- **📊 快照与引用系统** - 获取可访问性树，使用 `@e1`、`@e2` 等引用进行交互
- **🔐 安全认证管理** - 持久化配置文件、会话管理和认证保险库
- **🌐 多平台支持** - macOS、Linux、Windows 均提供原生二进制文件
- **🔌 云浏览器集成** - 支持 Browserless、Browserbase、Browser Use 等云端浏览器服务
- **📱 iOS 模拟器支持** - 可控制真实的移动端 Safari 进行测试

---

## 安装指南

### 1. 全局安装（推荐）

```bash
npm install -g agent-browser
agent-browser install  # 首次使用：从 Chrome for Testing 下载 Chrome
```

### 2. Homebrew 安装（macOS）

```bash
brew install agent-browser
agent-browser install
```

### 3. Cargo 安装（Rust 用户）

```bash
cargo install agent-browser
agent-browser install
```

### 4. 从源码构建

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
agent-browser install --with-deps
```

---

## 快速入门

### 基础工作流程

```bash
# 1. 打开网页
agent-browser open example.com

# 2. 获取快照（包含元素引用）
agent-browser snapshot
# 输出示例：
# - heading "Example Domain" [ref=e1] [level=1]
# - button "Submit" [ref=e2]
# - textbox "Email" [ref=e3]
# - link "Learn more" [ref=e4]

# 3. 使用引用进行交互
agent-browser click @e2                   # 点击按钮
agent-browser fill @e3 "test@example.com" # 填写文本
agent-browser get text @e1                # 获取文本

# 4. 截图
agent-browser screenshot page.png

# 5. 关闭浏览器
agent-browser close
```

### 传统选择器（同样支持）

```bash
agent-browser click "#submit"
agent-browser fill "#email" "test@example.com"
agent-browser find role button click --name "Submit"
```

---

## 主要命令一览

| 命令类别 | 命令示例 | 说明 |
|---------|---------|------|
| **导航** | `open <url>` | 打开网页 |
| **交互** | `click <sel>` / `fill <sel> <text>` | 点击/填写 |
| **输入** | `type <sel> <text>` / `press <key>` | 输入文本/按键 |
| **信息获取** | `get text <sel>` / `get url` | 获取文本/URL |
| **等待** | `wait <selector>` / `wait --load networkidle` | 等待元素/加载 |
| **截图** | `screenshot [path]` / `pdf <path>` | 截图/PDF |
| **快照** | `snapshot` | 获取可访问性树 |

---

## AI 工具集成

### Claude Code 集成（推荐）

将 agent-browser 作为技能添加到 Claude Code：

```bash
npx skills add vercel-labs/agent-browser
```

这会将技能安装到项目的 `.claude/skills/agent-browser/SKILL.md` 中，让 Claude Code 学会完整的 agent-browser 工作流程。

### OpenClaw 集成

在 OpenClaw 中，可以直接通过命令行调用 agent-browser：

```bash
agent-browser open https://example.com && agent-browser snapshot -i --json
```

### AGENTS.md / CLAUDE.md 配置

将以下内容添加到你的项目或全局指令文件中：

```markdown
## 浏览器自动化

使用 `agent-browser` 进行网页自动化。运行 `agent-browser --help` 查看所有命令。

核心工作流程：
1. `agent-browser open <url>` - 导航到页面
2. `agent-browser snapshot -i` - 获取带引用（@e1, @e2）的交互元素
3. `agent-browser click @e1` / `fill @e2 "text"` - 使用引用进行交互
4. 页面变更后重新快照
```

---

## 高级功能

### 会话管理

```bash
# 多会话隔离
agent-browser --session agent1 open site-a.com
agent-browser --session agent2 open site-b.com

# 列出活动会话
agent-browser session list
```

### 持久化配置文件

```bash
# 使用持久化配置目录（跨重启保持登录状态）
agent-browser --profile ~/.myapp-profile open myapp.com

# 或通过环境变量
AGENT_BROWSER_PROFILE=~/.myapp-profile agent-browser open myapp.com
```

### 云浏览器集成

```bash
# Browserbase
export BROWSERBASE_API_KEY="your-api-key"
agent-browser -p browserbase open https://example.com

# Browserless
export BROWSERLESS_API_KEY="your-api-token"
agent-browser -p browserless open https://example.com

# Browser Use
export BROWSER_USE_API_KEY="your-api-key"
agent-browser -p browseruse open https://example.com
```

### iOS 模拟器支持

```bash
# 列出可用设备
agent-browser device list

# 启动 Safari 进行测试
agent-browser -p ios --device "iPhone 16 Pro" open https://example.com

# 移动端特定命令
agent-browser -p ios swipe up
agent-browser -p ios tap @e1
```

---

## 安全功能

agent-browser 提供多项安全功能用于 AI 智能体部署：

| 功能 | 说明 |
|-----|------|
| **认证保险库** | 本地加密存储凭据，LLM 永远看不到密码 |
| **内容边界标记** | 用分隔符包裹页面输出，帮助 LLM 区分工具输出 |
| **域名白名单** | 限制导航到受信任的域名 |
| **操作策略** | 使用静态策略文件控制破坏性操作 |
| **操作确认** | 对敏感操作类别要求明确批准 |
| **输出长度限制** | 防止上下文泛滥 |

---

## 配置文件

创建 `agent-browser.json` 设置持久化默认值：

```json
{
  "headed": true,
  "proxy": "http://localhost:8080",
  "profile": "./browser-data",
  "userAgent": "my-agent/1.0",
  "ignoreHttpsErrors": true
}
```

**配置优先级**：环境变量 > CLI 标志 > 项目级配置 > 用户级配置

---

## 项目数据

- **Stars**: ⭐ 22,000+
- **Forks**: 🍴 1,300+
- **许可协议**: Apache-2.0
- **官方网站**: agent-browser.dev

---

## 相关链接

- [GitHub 仓库](https://github.com/vercel-labs/agent-browser)
- [npm 包](https://www.npmjs.com/package/agent-browser)
- [官方文档](https://agent-browser.dev)

---

## 标签

`#browser-automation` `#AI-agent` `#Rust` `#CLI` `#Vercel` `#web-scraping` `#testing`
