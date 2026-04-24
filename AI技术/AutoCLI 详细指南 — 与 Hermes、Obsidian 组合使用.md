---
title: "AutoCLI 详细指南 — 与 Hermes、Obsidian 组合使用"
source: "https://github.com/nashsu/AutoCLI"
author:
  - "[[nashsu]]"
created: 2026-04-23
description: "AutoCLI 详细使用指南——安装配置、55+ 站点数据抓取、AI 自适应适配器生成，以及如何与 Hermes Agent 和 Obsidian 打造全链路知识管理工作流"
tags:
  - AI
  - CLI
  - AutoCLI
  - Hermes
  - Obsidian
  - 开源
  - 自动化
  - Rust
  - 数据采集
  - 知识管理
---

# 🔧 AutoCLI 详细指南 — 与 Hermes、Obsidian 组合使用

> **一条命令，抓取全网信息 — 55+ 站点、333 条命令、4.7MB 单文件**
>
> GitHub：https://github.com/nashsu/AutoCLI
>
> 官网：https://autocli.ai
>
> 语言：Rust | 许可证：Apache-2.0

---

## 📖 目录

- [AutoCLI 是什么](#autocli-是什么)
- [核心特性](#核心特性)
- [性能对比](#性能对比)
- [安装部署](#安装部署)
- [Chrome 扩展配置](#chrome-扩展展配置)
- [快速开始](#快速开始)
- [内置命令大全](#内置命令大全)
- [AI 发现与生成](#ai-发现与生成)
- [自定义适配器](#自定义适配器)
- [下载功能](#下载功能)
- [输出格式](#输出格式)
- [与 Hermes Agent 集成](#与-hermes-agent-集成)
- [与 Obsidian 集成](#与-obsidian-集成)
- [三合一终极工作流](#三合一终极工作流)
- [常见问题](#常见问题)
- [资源链接](#资源链接)

---

## 🎯 AutoCLI 是什么

**AutoCLI**（前身为 opencli-rs）是一个用 **纯 Rust** 编写的高性能命令行工具，核心能力是**一条命令从任意网站抓取信息**。

| 特性         | 说明                                                                            |
| ---------- | ----------------------------------------------------------------------------- |
| **覆盖范围**   | 55+ 站点、333 条命令                                                                |
| **中文站点**   | B 站、知乎、小红书、微博、豆瓣、微信读书、雪球、Boss直聘、即刻、微信公众号等                                     |
| **国际站点**   | Twitter/X、Reddit、YouTube、HackerNews、StackOverflow、Medium、Bloomberg、LinkedIn 等 |
| **桌面控制**   | Cursor、ChatGPT、Codex、Notion、Discord 等桌面应用                                     |
| **外部 CLI** | gh、docker、kubectl、**obsidian**、readwise 等                                     |
| **AI 能力**  | 自动分析网站 API、AI 生成适配器、云端共享                                                      |
| **体积**     | 单个 4.7MB 二进制文件，零运行时依赖                                                         |

### 一句话定位

> **给 AI Agent 一双能触及全网的手** — 配合 Hermes / OpenClaw 等 Agent 使用，一条命令获取 55+ 站点的实时数据。

---

## ⚡ 核心特性

### 1. 55+ 站点，333 条命令

涵盖社交媒体、技术社区、新闻资讯、金融数据、知识平台等：

| 类别 | 代表站点 |
|------|----------|
| **技术社区** | HackerNews、StackOverflow、Dev.to、Lobsters、V2EX |
| **社交媒体** | Twitter/X、Reddit、微博、即刻、Facebook、Instagram、TikTok |
| **视频平台** | B 站、YouTube |
| **知识平台** | 知乎、小红书、Medium、Substack、Wikipedia |
| **金融数据** | 雪球、Bloomberg、Yahoo Finance、新浪财经 |
| **生活服务** | 豆瓣、微信读书、Boss直聘、什么值得买、携程 |
| **AI 工具** | Grok、即梦、豆包、Cursor、ChatGPT、Codex |
| **办公工具** | Notion、Discord |
| **搜索引擎** | Google、Google Trends |

### 2. 浏览器会话复用

通过 Chrome 扩展，**复用你已登录的浏览器会话**：
- 无需管理 Token、Cookie
- 直接使用你已登录的账号
- 支持 Chrome DevTools Protocol (CDP)

### 3. 声明式 YAML 管道

用 YAML 描述数据抓取流程，零代码添加新站点适配器：

```yaml
pipeline:
  - fetch: https://api.example.com/hot
  - select: data.posts
  - map:
      title: "${{ item.title }}"
      score: "${{ item.score }}"
  - limit: 20
```

### 4. AI 原生发现

- `explore` — 自动分析网站 API 端点、框架、数据结构
- `generate --ai` — AI 自动生成适配器
- `cascade` — 自动探测认证策略
- `search` — 搜索社区共享的适配器

### 5. 多格式输出

支持 table、JSON、YAML、CSV、Markdown 五种输出格式。

---

## 📊 性能对比

AutoCLI 基于 [OpenCLI](https://github.com/jackwener/opencli)（TypeScript）完整重写：

### 资源消耗

| 指标 | 🦀 AutoCLI (Rust) | 📦 OpenCLI (Node.js) | 提升 |
|------|:-----------------:|:-------------------:|:----:|
| 内存（公开命令） | 15 MB | 99 MB | **6.6x** |
| 内存（浏览器命令） | 9 MB | 95 MB | **10.6x** |
| 二进制大小 | 4.7 MB | ~50 MB | **10x** |
| 运行时依赖 | 无 | Node.js 20+ | **零依赖** |

### 实际命令速度

| 命令 | 🦀 AutoCLI | 📦 OpenCLI | 加速比 |
|------|:----------:|:---------:|:------:|
| `bilibili hot` | **1.66s** | 20.1s | 🔥 **12x** |
| `zhihu hot` | **1.77s** | 20.5s | 🔥 **11.6x** |
| `xueqiu search 茅台` | **1.82s** | 9.2s | ⚡ **5x** |
| `xiaohongshu search` | **5.1s** | 14s | ⚡ **2.7x** |

---

## 📥 安装部署

### 一键安装脚本（macOS / Linux）

```bash
curl -fsSL https://raw.githubusercontent.com/nashsu/autocli/main/scripts/install.sh | sh
```

自动检测系统架构，下载对应二进制文件到 `/usr/local/bin/`。

### Windows（PowerShell）

```powershell
Invoke-WebRequest -Uri "https://github.com/nashsu/autocli/releases/latest/download/autocli-x86_64-pc-windows-msvc.zip" -OutFile autocli.zip
Expand-Archive autocli.zip -DestinationPath .
Move-Item autocli.exe "$env:LOCALAPPDATA\Microsoft\WindowsApps\"
```

### 手动下载

从 [GitHub Releases](https://github.com/nashsu/autocli/releases/latest) 下载对应平台文件：

| 平台 | 文件 |
|------|------|
| macOS (Apple Silicon) | `autocli-aarch64-apple-darwin.tar.gz` |
| macOS (Intel) | `autocli-x86_64-apple-darwin.tar.gz` |
| Linux (x86_64) | `autocli-x86_64-unknown-linux-musl.tar.gz` |
| Linux (ARM64) | `autocli-aarch64-unknown-linux-musl.tar.gz` |
| Windows (x64) | `autocli-x86_64-pc-windows-msvc.zip` |

### 从源码构建

```bash
git clone https://github.com/nashsu/autocli.git
cd autocli
cargo build --release
cp target/release/autocli /usr/local/bin/
```

### 验证安装

```bash
autocli --version
autocli doctor
```

---

## 🔌 Chrome 扩展配置

> **公开 API 命令**（hackernews、devto 等）**无需此步骤**。仅浏览器模式命令需要。

### 安装步骤

1. 从 [GitHub Releases](https://github.com/nashsu/autocli/releases/latest) 下载 `autocli-chrome-extension.zip`
2. 解压到任意目录
3. 打开 Chrome，访问 `chrome://extensions`
4. 开启右上角 **"开发者模式"**
5. 点击 **"加载已解压的扩展程序"**，选择解压目录
6. 扩展会自动连接 autocli 守护进程

### Chrome 扩展高级功能（v0.3.2+）

- **选择器工具** — 在页面上可视化选择元素，精确构建 CSS 选择器
- **AI 生成** — 基于你选择的数据，AI 自动扩展并生成完整的抓取适配器
- **本地 + 云端同步** — 生成的适配器本地保存并同步到 AutoCLI.ai

---

## 🚀 快速开始

### 基础使用

```bash
# 查看所有命令
autocli --help

# Hacker News 热门文章（公开 API，无需浏览器）
autocli hackernews top --limit 10

# JSON 格式输出
autocli hackernews top --limit 5 --format json

# B 站热门视频（需要浏览器 + Cookie）
autocli bilibili hot --limit 20

# 搜索 Twitter（需要浏览器 + 登录）
autocli twitter search "rust lang" --limit 10
```

### AI 命令

```bash
# 认证 AutoCLI.ai
autocli auth

# AI 生成适配器（推荐）
autocli generate https://www.example.com --goal hot --ai

# 搜索社区共享的适配器
autocli search https://www.example.com

# 分析网站 API
autocli explore https://www.example.com --site mysite

# 自动探测认证策略
autocli cascade https://api.example.com/hot
```

### 外部 CLI 集成

```bash
# GitHub CLI 透传
autocli gh repo list

# Docker CLI 透传
autocli docker ps

# Kubernetes CLI 透传
autocli kubectl get pods
```

### Shell 补全

```bash
autocli completion bash >> ~/.bashrc
autocli completion zsh >> ~/.zshrc
autocli completion fish > ~/.config/fish/completions/autocli.fish
```

---

## 📋 内置命令大全

### 公开 API 站点（无需浏览器）

| 站点 | 命令 | 说明 |
|------|------|------|
| **hackernews** | `top` `new` `best` `ask` `show` `jobs` `search` `user` | 技术社区 |
| **devto** | `top` `tag` `user` | 开发者社区 |
| **lobsters** | `hot` `newest` `active` `tag` | 技术社区 |
| **stackoverflow** | `hot` `search` `bounties` `unanswered` | 问答社区 |
| **wikipedia** | `search` `summary` `random` `trending` | 百科 |
| **arxiv** | `search` `paper` | 学术论文 |
| **bbc** | `news` | 新闻 |
| **google** | `news` `search` `suggest` `trends` | 搜索引擎 |
| **steam** | `top-sellers` | 游戏 |
| **linux-do** | `hot` `latest` `search` `categories` `topic` | 社区 |

### 浏览器模式站点

| 站点 | 命令数 | 主要命令 |
|------|--------|----------|
| **twitter** | 24 | `trending` `search` `timeline` `profile` `bookmarks` `post` `like` `download` |
| **bilibili** | 12 | `hot` `search` `favorite` `history` `dynamic` `ranking` `download` |
| **reddit** | 15 | `hot` `frontpage` `search` `subreddit` `read` `upvote` `save` `comment` |
| **zhihu** | 2 | `hot` `search` `question` `download` |
| **xiaohongshu** | 11 | `search` `feed` `user` `download` `publish` `creator-*` |
| **youtube** | 4 | `search` `video` `transcript` |
| **douban** | 7 | `search` `top250` `subject` `marks` `reviews` `movie-hot` |
| **xueqiu** | 7 | `feed` `hot-stock` `search` `stock` `watchlist` |
| **boss** | 14 | `search` `detail` `recommend` `greet` `chatlist` `invite` |
| **facebook** | 10 | `feed` `profile` `search` `friends` `groups` `events` |
| **instagram** | 14 | `explore` `profile` `search` `followers` `following` `like` |
| **tiktok** | 15 | `explore` `search` `profile` `user` `following` `like` `live` |

### 桌面应用控制

| 应用              | 命令                                                                                                          | 说明           |
| --------------- | ----------------------------------------------------------------------------------------------------------- | ------------ |
| **cursor**      | `status` `send` `read` `new` `dump` `composer` `model` `extract-code` `ask` `screenshot` `history` `export` | AI 代码编辑器     |
| **codex**       | `status` `send` `read` `new` `dump` `extract-diff` `model` `ask` `screenshot` `history` `export`            | OpenAI Codex |
| **chatgpt**     | `status` `new` `send` `read` `ask`                                                                          | ChatGPT 桌面版  |
| **notion**      | `status` `search` `read` `new` `write` `sidebar` `favorites` `export`                                       | Notion 笔记    |
| **discord-app** | `status` `send` `read` `channels` `servers` `search` `members`                                              | Discord 桌面版  |
| **doubao-app**  | `status` `new` `send` `read` `ask` `screenshot` `dump`                                                      | 豆包桌面版        |
| **chatwise**    | `status` `new` `send` `read` `ask` `model` `history` `export` `screenshot`                                  | Chatwise     |

---

## 🤖 AI 发现与生成

### AutoCLI.ai 认证

```bash
# 交互式认证
autocli auth
# 1. 打开浏览器访问 https://autocli.ai/get-token
# 2. 获取 API Token
# 3. 粘贴到终端
# 4. Token 保存到 ~/.autocli/config.json
```

### AI 生成适配器（推荐方式）

```bash
# AI 自动分析页面并生成适配器
autocli generate https://www.example.com --goal hot --ai

# 会先搜索已有适配器，找不到再用 AI 生成
```

### 规则分析（无需 AI）

```bash
# 启发式分析网站 API
autocli generate https://www.example.com --goal hot

# 深度探测 API 端点、框架、数据结构
autocli explore https://www.example.com --site mysite

# 带交互式模糊测试
autocli explore https://www.example.com --auto --click "Comments,CC"

# 自动检测认证策略（PUBLIC → COOKIE → HEADER）
autocli cascade https://api.example.com/hot
```

### 搜索社区适配器

```bash
# 按 URL 搜索
autocli search https://www.example.com

# 按域名搜索
autocli search example.com
```

---

## 🧩 自定义适配器

在 `~/.autocli/adapters/` 下创建 YAML 文件即可添加自定义站点：

```yaml
# ~/.autocli/adapters/mysite/hot.yaml
site: mysite
name: hot
description: My site hot posts
strategy: public
browser: false

args:
  limit:
    type: int
    default: 20
    description: Number of items

columns: [rank, title, score]

pipeline:
  - fetch: https://api.mysite.com/hot
  - select: data.posts
  - map:
      rank: "${{ index + 1 }}"
      title: "${{ item.title }}"
      score: "${{ item.score }}"
  - limit: "${{ args.limit }}"
```

### 管道步骤参考

| 步骤 | 功能 | 示例 |
|------|------|------|
| `fetch` | HTTP 请求 | `fetch: https://api.example.com/data` |
| `evaluate` | 浏览器内执行 JS | `evaluate: "document.title"` |
| `navigate` | 页面导航 | `navigate: https://example.com` |
| `click` | 点击元素 | `click: "#button"` |
| `type` | 输入文本 | `type: { selector: "#input", text: "hello" }` |
| `wait` | 等待 | `wait: 2000` |
| `select` | 选取嵌套数据 | `select: data.items` |
| `map` | 数据映射 | `map: { title: "${{ item.title }}" }` |
| `filter` | 数据过滤 | `filter: "item.score > 10"` |
| `sort` | 排序 | `sort: { by: score, order: desc }` |
| `limit` | 截断 | `limit: "${{ args.limit }}"` |
| `intercept` | 网络拦截 | `intercept: { pattern: "*/api/*" }` |
| `tap` | 状态管理桥接 | `tap: { action: "store.fetch" }` |
| `download` | 下载 | `download: { type: media }` |

### 模板表达式

```yaml
# 变量访问
"${{ args.limit }}"
"${{ item.title }}"

# 条件与逻辑
"${{ item.score > 10 }}"
"${{ item.active ? 'yes' : 'no' }}"

# 管道过滤器（16 种）
"${{ item.title | truncate(30) }}"
"${{ item.tags | join(', ') }}"
"${{ item.name | lower | trim }}"

# 字符串插值
"https://api.com/${{ item.id }}.json"

# 回退值
"${{ item.subtitle || 'N/A' }}"
```

---

## ⬇️ 下载功能

### 视频下载

```bash
# 下载 B 站视频（需要 yt-dlp）
autocli bilibili download BV1xxx --output ./videos --quality 1080p
```

### 文章下载为 Markdown

```bash
# 下载知乎文章为 Markdown（含图片本地化）
autocli zhihu download "https://zhuanlan.zhihu.com/p/xxx" --output ./articles

# 下载微信公众号文章
autocli weixin download "https://mp.weixin.qq.com/s/xxx" --output ./articles
```

### Twitter/X 媒体下载

```bash
autocli twitter download nash_su --limit 10 --output ./twitter
autocli twitter download --tweet-url "https://x.com/user/status/123" --output ./twitter
```

> **下载特性**：
> - 视频通过 yt-dlp 下载（自动提取浏览器 Cookie，无需 Keychain 弹窗）
> - 文章转为 Markdown + YAML frontmatter（标题、作者、日期、来源）
> - 图片自动下载并本地化（远程 URL 替换为本地 `images/img_001.jpg`）

---

## 📤 输出格式

```bash
autocli hackernews top --format table    # ASCII 表格（默认）
autocli hackernews top --format json     # JSON
autocli hackernews top --format yaml     # YAML
autocli hackernews top --format csv      # CSV
autocli hackernews top --format md       # Markdown 表格
```

---

## 🤝 与 Hermes Agent 集成

> AutoCLI 被官方定位为 **"AI Agent 的完美伴侣"**——让你的 Agent 能触及全网信息。

[[AI技术/Hermes Agent/Hermes Agent 完全指南|Hermes Agent]] 是 Nous Research 开发的自我进化 AI Agent，拥有 644+ 技能和 18+ 模型提供商支持。将 AutoCLI 与 Hermes 结合，相当于给 Agent 装上了**全网数据采集引擎**。

### 方式一：技能安装（推荐）

AutoCLI 提供了专门的 AI Agent Skill：

```bash
# 一键安装 AutoCLI 技能到 Hermes
npx skills add https://github.com/nashsu/autocli-skill
```

安装后，Hermes Agent 会自动了解所有 AutoCLI 可用命令，并在需要时调用。

### 方式二：通过 AGENT.md 配置

在项目的 `AGENT.md` 或 `.cursorrules` 中添加：

```markdown
## AutoCLI - Web Data Fetching Tool

Run `autocli list` to see all available data fetching commands.

Key capabilities:
- `autocli hackernews top --limit 10 --format json` — Fetch Hacker News top stories
- `autocli twitter search "keyword" --limit 10 --format json` — Search Twitter
- `autocli bilibili hot --limit 20 --format json` — Fetch Bilibili trending
- `autocli zhihu hot --format json` — Fetch Zhihu trending
- `autocli reddit hot --subreddit programming --format json` — Fetch Reddit posts
- `autocli youtube search "topic" --format json` — Search YouTube
- `autocli xiaohongshu search "keyword" --format json` — Search Xiaohongshu
- `autocli arxiv search "topic" --format json` — Search academic papers
- `autocli stock search "AAPL" --format json` — Stock data
- Use `--format json` for machine-readable output
```

### 方式三：通过 Hermes 终端直接调用

```bash
# 启动 Hermes
hermes

# 在对话中请求 Agent 使用 AutoCLI
❯ 帮我看看 Hacker News 上今天的热门文章，总结前 10 篇
# Agent 会调用: autocli hackernews top --limit 10 --format json

❯ 在 Twitter 上搜索关于 Rust 的最新讨论
# Agent 会调用: autocli twitter search "rust" --limit 10 --format json

❯ 帮我把这篇知乎文章下载下来保存到笔记里
# Agent 会调用: autocli zhihu download "https://zhuanlan.zhihu.com/p/xxx" --output ./articles

❯ 看看 B 站有什么热门视频
# Agent 会调用: autocli bilibili hot --limit 20
```

### 方式四：Hermes 消息网关 + AutoCLI

通过 Hermes 的 **15+ 消息平台网关**，在任何地方触发 AutoCLI：

```bash
# 配置 Telegram 网关
hermes gateway setup  # 选择 Telegram

# 在 Telegram 中发送：
# "帮我搜一下小红书上关于 Obsidian 的笔记"
# Hermes → 调用 autocli xiaohongshu search "Obsidian" --format json → 返回结果
```

### 方式五：MCP 服务器集成

将 AutoCLI 注册为 Hermes 的 MCP 服务器：

```yaml
# ~/.hermes/config.yaml
mcp_servers:
  autocli:
    command: autocli
    args: ["serve"]  # 如果 AutoCLI 支持 MCP server 模式
```

### 实际应用场景

| 场景 | 对话示例 | AutoCLI 调用 |
|------|----------|-------------|
| **技术趋势** | "今天 Hacker News 有什么好文章？" | `autocli hackernews top --limit 10` |
| **社交监控** | "Twitter 上 #AI 话题最新动态？" | `autocli twitter search "AI" --limit 10` |
| **竞品分析** | "小红书上对这款产品的评价？" | `autocli xiaohongshu search "产品名"` |
| **学术追踪** | "arxiv 上最新的 LLM 论文？" | `autocli arxiv search "LLM" --format json` |
| **求职辅助** | "Boss直聘上有什么 Rust 岗位？" | `autocli boss search "Rust" --format json` |
| **金融分析** | "茅台股价和相关信息？" | `autocli xueqiu search "茅台"` |
| **内容下载** | "把这篇微信文章保存下来" | `autocli weixin download "URL"` |
| **视频获取** | "这个 B 站视频的字幕是什么？" | `autocli bilibili subtitle BVxxx` |

---

## 📝 与 Obsidian 集成

### AutoCLI 的 Obsidian 原生支持

AutoCLI 内置了 `obsidian` 外部 CLI 集成，可以直接操作 Obsidian：

```bash
# 通过 AutoCLI 操作 Obsidian（透传模式）
autocli obsidian <命令>
```

### 文章保存为 Obsidian 笔记

AutoCLI 的下载功能天然适配 Obsidian：

```bash
# 下载文章为 Markdown（含 YAML frontmatter + 图片本地化）
autocli zhihu download "https://zhuanlan.zhihu.com/p/xxx" --output "/你的Vault路径/Clippings/"

autocli weixin download "https://mp.weixin.qq.com/s/xxx" --output "/你的Vault路径/Clippings/"

autocli substack feed --format md > "/你的Vault路径/Inbox/feed.md"
```

**下载产物结构**（完美适配 Obsidian）：

```
output/article_title/
├── title.md          # Markdown + YAML frontmatter
└── images/           # 图片本地化
    ├── img_001.jpg
    └── img_002.jpg
```

### JSON 输出 → Obsidian Dataview

```bash
# 获取数据保存为 JSON，供 Dataview 查询
autocli hackernews top --limit 30 --format json > "/你的Vault路径/Data/hackernews.json"
autocli zhihu hot --format json > "/你的Vault路径/Data/zhihu_hot.json"
autocli xueqiu hot-stock --format json > "/你的Vault路径/Data/stock_hot.json"
```

### 定时同步到 Obsidian

结合系统 Cron 或 Hermes 定时任务，自动将全网数据同步到 Obsidian：

```bash
# 每天早上同步 Hacker News 热门到 Obsidian
# 通过 crontab -e 添加：
0 9 * * * autocli hackernews top --limit 20 --format md > "/你的Vault路径/Daily/HN_$(date +\%Y\%m\%d).md"

# 每天同步知乎热榜
0 9 * * * autocli zhihu hot --format md > "/你的Vault路径/Daily/Zhihu_$(date +\%Y\%m\%d).md"
```

---

## 🔄 三合一终极工作流

> **AutoCLI + Hermes Agent + Obsidian** = 全网感知、智能处理、知识沉淀 的完整闭环

### 架构总览

```
┌─────────────────────────────────────────────────────────────────┐
│                    全网数据源 (55+ 站点)                          │
│  Twitter · Reddit · HN · B站 · 知乎 · 小红书 · YouTube · ...    │
└───────────────────────────┬─────────────────────────────────────┘
                            │ autocli <site> <command>
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                       AutoCLI (数据采集层)                        │
│  • 一条命令抓取任意站点数据                                        │
│  • AI 自动生成新站点适配器                                        │
│  • 文章/视频下载为 Markdown                                       │
│  • JSON / Markdown / CSV 多格式输出                               │
└───────────────────────────┬─────────────────────────────────────┘
                            │ 原始数据 / Markdown 文件
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                   Hermes Agent (智能处理层)                       │
│  • 自动调用 AutoCLI 获取实时数据                                   │
│  • LLM 分析、总结、分类、提取要点                                  │
│  • 记忆系统记住你的偏好和项目上下文                                 │
│  • 技能系统自动学习和改进                                          │
│  • 消息网关 (Telegram/Discord/Slack) 随时触发                      │
└───────────────────────────┬─────────────────────────────────────┘
                            │ 结构化笔记 / Markdown
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Obsidian (知识沉淀层)                          │
│  • Markdown 原生支持，完美适配                                    │
│  • 双向链接构建知识图谱                                           │
│  • Dataview 查询结构化数据                                        │
│  • 模板系统自动化笔记格式                                          │
│  • 图谱视图可视化知识关联                                          │
└─────────────────────────────────────────────────────────────────┘
```

### 场景一：每日信息简报

```
用户（Telegram 9:00 AM）
    │
    │ "今天的科技简报"
    ▼
Hermes Agent
    ├── autocli hackernews top --limit 10 --format json
    ├── autocli twitter trending --format json
    ├── autocli reddit hot --subreddit technology --format json
    ├── autocli zhihu hot --limit 5 --format json
    │
    ├── LLM 分析、去重、总结
    │
    ▼
生成 Markdown 简报
    ├── 保存到 Obsidian: Daily/2026-04-23_简报.md
    └── 发送摘要到 Telegram
```

### 场景二：深度研究与知识沉淀

```
用户
    │ "帮我研究一下 Rust 的 Web 框架生态，整理成笔记"
    ▼
Hermes Agent
    ├── autocli hackernews search "rust web framework" --format json
    ├── autocli reddit search "rust axum actix" --format json
    ├── autocli arxiv search "rust web framework" --format json
    ├── autocli github search "rust web framework"  # 通过 gh 透传
    │
    ├── LLM 分析、对比、总结
    │
    ▼
生成结构化研究笔记
    ├── 保存到 Obsidian: Research/Rust Web 框架对比.md
    │   ├── YAML frontmatter (tags, date, source)
    │   ├── 框架对比表格
    │   ├── 社区评价摘要
    │   └── 相关资源链接（双向链接）
    └── 更新 Hermes 记忆: "用户关注 Rust 生态"
```

### 场景三：内容创作辅助

```
用户
    │ "我在小红书上看到一篇好的 Obsidian 教程，帮我保存并翻译"
    ▼
Hermes Agent
    ├── autocli xiaohongshu search "Obsidian 教程" --format json
    ├── autocli xiaohongshu download <note_id> --output ./articles/
    │
    ├── LLM 翻译、整理、格式化
    │
    ▼
保存到 Obsidian
    ├── Clippings/小红书_Obsidian教程_翻译.md
    │   ├── 原文链接
    │   ├── 翻译内容
    │   └── 本地化图片
    └── 自动打标签 #obsidian #教程 #翻译
```

### 场景四：监控与告警

```
Hermes Cron 定时任务
    │ 每小时检查一次
    ▼
Hermes Agent
    ├── autocli xueqiu stock AAPL --format json
    ├── autocli bloomberg markets --format json
    │
    ├── 检测异常波动
    │
    ├── 如果发现重要变化
    │   ├── 通知用户（Telegram/微信）
    │   └── 记录到 Obsidian: Trading/日志/
    └── 如果无异常
        └── 静默
```

### 快速搭建清单

| 步骤  | 操作                   | 命令                                                                                                       |
| --- | -------------------- | -------------------------------------------------------------------------------------------------------- |
| 1   | 安装 AutoCLI           | `curl -fsSL https://raw.githubusercontent.com/nashsu/autocli/main/scripts/install.sh \| sh`              |
| 2   | 安装 Chrome 扩展         | 下载解压 → `chrome://extensions` → 加载                                                                        |
| 3   | 认证 AutoCLI.ai        | `autocli auth`                                                                                           |
| 4   | 安装 Hermes Agent      | `curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh \| bash` |
| 5   | 配置 Hermes 模型         | `hermes model`                                                                                           |
| 6   | 安装 AutoCLI 技能        | `npx skills add https://github.com/nashsu/autocli-skill`                                                 |
| 7   | 配置消息网关               | `hermes gateway setup`                                                                                   |
| 8   | 设置 Obsidian Vault 路径 | 在 Hermes MEMORY.md 中记录 Vault 路径                                                                          |

---

## 🔧 常见问题

### Q1: 安装后找不到命令

```bash
# 检查是否在 PATH 中
which autocli

# 如果不在，手动添加
export PATH="$PATH:/usr/local/bin"

# 或重新运行安装脚本
curl -fsSL https://raw.githubusercontent.com/nashsu/autocli/main/scripts/install.sh | sh
```

### Q2: Chrome 扩展连接失败

```bash
# 检查守护进程是否运行
autocli doctor

# 重启守护进程
# 在 Chrome 扩展页面点击刷新
```

### Q3: 浏览器命令报错 "Cookie not found"

```
1. 确保 Chrome 扩展已安装并启用
2. 确保你已在 Chrome 中登录目标网站
3. 运行 autocli doctor 检查连接状态
4. 重启 Chrome 浏览器
```

### Q4: Windows 上如何使用

```powershell
# 方式一：直接安装 Windows 版本
Invoke-WebRequest -Uri "https://github.com/nashsu/autocli/releases/latest/download/autocli-x86_64-pc-windows-msvc.zip" -OutFile autocli.zip
Expand-Archive autocli.zip -DestinationPath .
Move-Item autocli.exe "$env:LOCALAPPDATA\Microsoft\WindowsApps\"

# 方式二：通过 WSL2 使用（推荐搭配 Hermes）
wsl
curl -fsSL https://raw.githubusercontent.com/nashsu/autocli/main/scripts/install.sh | sh
```

### Q5: 如何为 AutoCLI 添加新站点

```bash
# 方式一：AI 自动生成（推荐）
autocli generate https://www.example.com --goal hot --ai

# 方式二：手动创建 YAML 适配器
# 在 ~/.autocli/adapters/mysite/hot.yaml 创建适配器文件
```

### Q6: Hermes 调用 AutoCLI 超时

```bash
# 增加浏览器命令超时时间
export OPENCLI_BROWSER_COMMAND_TIMEOUT=120  # 120 秒

# 在 Hermes 配置中设置更长的工具超时
hermes config set tools.timeout 120
```

---

## ⚙️ 配置参考

### 环境变量

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `OPENCLI_VERBOSE` | - | 启用详细输出 |
| `OPENCLI_DAEMON_PORT` | `19825` | 守护进程端口 |
| `OPENCLI_CDP_ENDPOINT` | - | CDP 直接端点（绕过守护进程） |
| `OPENCLI_BROWSER_COMMAND_TIMEOUT` | `60` | 命令超时（秒） |
| `OPENCLI_BROWSER_CONNECT_TIMEOUT` | `30` | 浏览器连接超时（秒） |
| `OPENCLI_BROWSER_EXPLORE_TIMEOUT` | `120` | 探索超时（秒） |
| `AUTOCLI_API_BASE` | `https://www.autocli.ai` | API 服务器地址 |

### 文件路径

| 路径 | 说明 |
|------|------|
| `~/.autocli/adapters/` | 用户自定义适配器 |
| `~/.autocli/plugins/` | 用户插件 |
| `~/.autocli/external-clis.yaml` | 外部 CLI 注册表 |
| `~/.autocli/config.json` | 配置文件（含 API Token） |

---

## 📚 资源链接

### 官方资源

- **GitHub**：https://github.com/nashsu/AutoCLI
- **官网 / AI 生成**：https://autocli.ai
- **AutoCLI Skill**：https://github.com/nashsu/autocli-skill
- **Releases**：https://github.com/nashsu/autocli/releases/latest

### 关联笔记

- [[AI技术/Hermes Agent/Hermes Agent 完全指南]] — Hermes Agent 核心引擎
- [[AI技术/Hermes Agent/Hermes Web UI 介绍与部署使用指南]] — Web 管理界面

### 技术基础

- **Rust** — 系统级编程语言
- **OpenCLI** (TypeScript 原版)：https://github.com/jackwener/opencli
- **Chrome DevTools Protocol (CDP)** — 浏览器自动化协议
- **pest PEG 解析器** — 模板表达式引擎

---

## 📝 版本历史

| 版本         | 主要更新                          |
| ---------- | ----------------------------- |
| **v0.2.4** | 从 opencli-rs 更名为 AutoCLI      |
| **v0.3.0** | AI 生成适配器、autocli.ai 云端共享      |
| **v0.3.2** | Chrome 扩展选择器工具、AI 扩展生成、本地+云同步 |

---

**🔧 AutoCLI + Hermes Agent + Obsidian = 全网感知、智能处理、知识沉淀的终极工作流！**
