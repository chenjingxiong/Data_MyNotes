# AionUi 安装使用指南

> **AionUi** - 免费开源的 AI 协作平台
> 官方仓库：https://github.com/iOfficeAI/AionUi
> 官网：https://www.aionui.com

---

## 📌 目录

1. [项目简介](#项目简介)
2. [系统要求](#系统要求)
3. [安装方法](#安装方法)
4. [快速开始](#快速开始)
5. [核心功能](#核心功能)
6. [配置指南](#配置指南)
7. [使用技巧](#使用技巧)
8. [常见问题](#常见问题)
9. [社区资源](#社区资源)

---

## 项目简介

**AionUi** 是一个免费、开源的 AI 协作（Cowork）平台，让 AI Agent 直接在你的电脑上工作——读取文件、编写代码、浏览网页、自动化任务。

### 核心特点

| 特性 | 说明 |
|------|------|
| 🔧 **零配置启动** | 内置完整 AI Agent 引擎，安装即用 |
| 🤖 **多 Agent 支持** | 内置 Agent + Claude Code、Codex、Qwen Code 等 12+ CLI Agent |
| 🔑 **任意 API Key** | 支持 Gemini、Claude、OpenAI、Ollama 等 20+ 平台 |
| 🌐 **远程访问** | WebUI + Telegram/Lark/钉钉，随时随地协作 |
| ⏰ **定时任务** | Cron 调度，24/7 自动化运行 |
| 💾 **本地存储** | SQLite 本地数据库，数据不上传 |
| 🎨 **12+ 内置助手** | Cowork、PPTX 生成器、PDF 转 PPT、3D 游戏、UI/UX Pro 等 |

### 与 Claude Code/Claude Cowork 对比

| 维度 | Claude Cowork | AionUi |
|:-----|:--------------|:--------|
| 操作系统 | macOS 仅限 | macOS / Windows / Linux |
| 模型支持 | Claude 仅限 | Gemini、Claude、DeepSeek、OpenAI、Ollama... |
| 交互方式 | 桌面 GUI | 桌面 GUI + WebUI + Telegram/Lark/钉钉 |
| 自动化 | 手动仅限 | Cron 定时任务 — 24/7 无人值守 |
| 费用 | $100/月 | **免费开源** |

---

## 系统要求

### 操作系统

- **macOS**: 10.15 或更高版本
- **Windows**: Windows 10 或更高版本
- **Linux**: Ubuntu 18.04+ / Debian 10+ / Fedora 32+

### 硬件要求

- **内存**: 4GB+ 推荐
- **存储**: 500MB+ 可用空间
- **网络**: 需要 Internet 连接（使用云 API 时）

### Node.js 版本

如需从源码构建：Node.js >= 22 且 < 25

---

## 安装方法

### 方法一：下载安装包（推荐）

1. 访问 [GitHub Releases](https://github.com/iOfficeAI/AionUi/releases)
2. 下载对应平台的安装包：
   - **macOS**: `.dmg` 文件
   - **Windows**: `.exe` 安装程序
   - **Linux**: `.AppImage` 或 `.deb` 文件
3. 运行安装程序并按提示完成安装

### 方法二：Homebrew（macOS）

```bash
brew install aionui
```

### 方法三：从源码构建

```bash
# 克隆仓库
git clone https://github.com/iOfficeAI/AionUi.git
cd AionUi

# 安装依赖
bun install

# 开发模式运行
bun start

# 构建应用
bun run dist
```

---

## 快速开始

### 三步启动

1. **安装 AionUi**
2. **登录或配置 API Key**
   - 选择 Google 账号登录（使用 Gemini，免费）
   - 或输入任意 API Key（OpenAI、Anthropic、DeepSeek 等）
3. **开始协作** — 内置 AI Agent 已准备就绪

### 首次配置

1. 打开 AionUi 应用
2. 选择登录方式：
   - **Google 登录**: 点击 "Sign in with Google"
   - **API Key**: 点击 "Add API Key" 输入密钥
3. 选择 AI 模型提供商
4. 创建第一个对话，开始使用

---

## 核心功能

### 1. 内置 Agent - 零配置即用

AionUi 内置完整的 AI Agent 引擎，无需额外安装 CLI 工具。

**支持的能力：**
- ✅ 文件读写
- ✅ 网页搜索
- ✅ 图像生成
- ✅ MCP 工具
- ✅ 文档处理

**内置助手：**
- 🤝 **Cowork** - 自主任务执行
- 📊 **PPTX Generator** - 生成 PPT 演示文稿
- 📄 **PDF to PPT** - PDF 转 PPT
- 🎮 **3D Game** - 单文件 3D 游戏生成
- 🎨 **UI/UX Pro Max** - 专业 UI/UX 设计（57 风格、95 配色）
- 📋 **Planning with Files** - 复杂任务的文件规划
- 🧭 **HUMAN 3.0 Coach** - 个人发展教练
- 📣 **Social Job Publisher** - 职位发布
- 🦞 **moltbook** - AI Agent 社交网络
- 📈 **Beautiful Mermaid** - 流程图、时序图
- 🔧 **OpenClaw Setup** - OpenClaw 集成配置
- 📖 **Story Roleplay** - 沉浸式故事角色扮演

### 2. 多 Agent 模式

如果你已安装 Claude Code、Codex 或 Qwen Code，AionUi 会自动检测并集成。

**支持的 CLI Agent：**
- Built-in Agent（零配置）
- Claude Code
- Codex
- Qwen Code
- Goose AI
- OpenClaw
- Augment Code
- iFlow CLI
- CodeBuddy
- Kimi CLI
- OpenCode
- Factory Droid
- GitHub Copilot
- Qoder CLI
- Mistral Vibe
- Nanobot
- ...更多

**优势：**
- 🔍 自动检测已安装的 CLI 工具
- 🎯 统一的协作界面
- 🔄 多个 Agent 并行运行，独立上下文
- 🔌 MCP 统一管理，一次配置全部同步

### 3. 多平台 AI 模型支持

使用任意 API Key，获得完整的协作 Agent 能力。

| API Key                        | 获得的能力              |
| :----------------------------- | :----------------- |
| Gemini API Key（或 Google 登录-免费） | Gemini 驱动的协作 Agent |
| OpenAI API Key                 | GPT 驱动的协作 Agent    |
| Anthropic API Key              | Claude 驱动的协作 Agent |
| Ollama / LM Studio（本地）         | 本地模型协作 Agent       |
| NewAPI Gateway                 | 统一访问 20+ 模型        |

**支持的平台（20+）：**

**官方平台：**
- Gemini、Gemini (Vertex AI)、Anthropic (Claude)、OpenAI

**云服务商：**
- AWS Bedrock、New API（统一 AI 模型网关）

**国内平台：**
- Dashscope (Qwen)、Zhipu、Moonshot (Kimi)、Qianfan (百度)
- Hunyuan (腾讯)、Lingyi、ModelScope、InfiniAI、Ctyun、StepFun

**国际平台：**
- DeepSeek、MiniMax、OpenRouter、SiliconFlow、xAI、Ark (Volcengine)、Poe

**本地模型：**
- Ollama、LM Studio（通过自定义平台本地 API 端点）

### 4. 定时任务 - 24/7 自动化

设置一次，AI Agent 按计划自动运行——真正的 24/7 无人值守。

**特点：**
- 🗣️ 自然语言 — 像聊天一样告诉 Agent 要做什么
- ⏰ 灵活调度 — 每天、每周、每月或自定义 cron 表达式
- 📋 对话绑定 — 每个定时任务绑定到对话，保持上下文和历史
- 🔄 自动执行 — 任务按计划时间自动运行，发送消息到对话

**使用场景：**
- 每日天气报告生成
- 每周销售数据汇总
- 每月备份文件整理
- 自定义提醒通知

### 5. 预览面板 - 即时查看结果

10+ 格式：PDF、Word、Excel、PPT、代码、Markdown、图片、HTML、Diff — 无需切换应用。

**支持格式：**

**文档类：**
- PDF
- Word（`.doc`、`.docx`、`.odt`）
- Excel（`.xls`、`.xlsx`、`.ods`、`.csv`）
- PowerPoint（`.ppt`、`.pptx`、`.odp`）

**代码类：**
- JavaScript、TypeScript、Python、Java、Go、Rust、C/C++、CSS、JSON、XML、YAML、Shell 脚本等 30+ 编程语言

**标记类：**
- Markdown（`.md`、`.markdown`）
- HTML（`.html`、`.htm`）

**图片类：**
- PNG、JPG、JPEG、GIF、SVG、WebP、BMP、ICO、TIFF、AVIF

**其他：**
- Diff 文件（`.diff`、`.patch`）

**特点：**
- ⚡ 即时预览 — Agent 生成文件后立即查看
- 📝 实时跟踪 + 可编辑 — 自动跟踪文件更改，支持 Markdown、代码、HTML 实时编辑
- 🗂️ 多标签支持 — 同时打开多个文件，各占一个标签
- 📜 版本历史 — 查看和恢复文件历史版本（基于 Git）

### 6. 智能文件管理

批量重命名、自动整理、智能分类、文件合并 — 协作 Agent 一手包办。

**功能：**
- 📁 自动整理 — 智能识别内容并自动分类，保持文件夹整洁
- 🔄 高效批量 — 一键重命名、合并文件，告别繁琐手动操作
- 🤖 自动执行 — AI Agent 可独立执行文件操作、读/写文件、自动完成任务

**使用场景：**
- 按文件类型整理混乱的下载文件夹
- 用有意义的名称批量重命名照片
- 将多个文档合并为一个
- 按内容自动分类文件

### 7. Excel 数据处理

深度分析 Excel 数据、自动美化报表、生成洞察 — 全部由 AI Agent 驱动。

**功能：**
- 📊 智能分析 — AI 分析数据模式并生成洞察
- 🎨 自动格式化 — 自动美化 Excel 报表，专业样式
- 🔄 数据转换 — 使用自然语言命令转换、合并和重构数据
- 📈 报表生成 — 从原始数据创建综合报告

**使用场景：**
- 分析销售数据并生成月度报告
- 清理和格式化混乱的 Excel 文件
- 智能合并多个电子表格
- 创建数据可视化和图表

### 8. AI 图像生成与编辑

由 Gemini 驱动的智能图像生成、编辑和识别。

**功能：**
- 🎨 文生图 — 从自然语言描述生成图像
- ✏️ 图像编辑 — 修改和增强现有图像
- 👁️ 图像识别 — 分析和描述图像内容
- 📦 批量处理 — 一次生成多张图像

### 9. 文档生成

AI Agent 自动生成专业文档 — 演示文稿、报告等。

**功能：**
- 📊 PPTX 生成器 — 从大纲或主题创建专业演示文稿
- 📄 Word 文档 — 生成具有正确结构的格式化 Word 文档
- 📝 Markdown 文件 — 创建和格式化 Markdown 文档用于文档
- 🔄 PDF 转换 — 在各种文档格式之间转换

**使用场景：**
- 生成季度业务演示文稿
- 创建技术文档
- 将 PDF 转换为可编辑格式
- 自动格式化研究论文

### 10. 个性化界面定制

用自己的 CSS 代码定制，让界面符合你的偏好。

- ✅ 完全可定制 — 通过 CSS 代码自由定制界面颜色、样式、布局
- ✅ 创建专属体验

### 11. 多任务并行处理

打开多个对话，任务不混淆，独立记忆，效率翻倍。

- ✅ 独立上下文 — 每个对话维护自己的上下文和历史
- ✅ 并行执行 — 同时运行多个任务互不干扰
- ✅ 智能管理 — 轻松切换对话，带有视觉指示器

---

## 配置指南

### LLM 配置

**支持的配置方式：**

1. **Google 登录（推荐新手）**
   - 点击 "Sign in with Google"
   - 授权后使用 Gemini（免费额度）

2. **API Key 配置**
   - 设置 → API Keys → Add API Key
   - 选择提供商并输入密钥

3. **本地模型（Ollama）**
   - 安装 Ollama
   - 在 AionUi 中配置 Ollama 端点（默认 `http://localhost:11434`）

4. **NewAPI 网关**
   - 部署 [NewAPI](https://github.com/QuantumNous/new-api)
   - 配置统一访问多个模型

### Multi-Agent 模式配置

**自动检测：**
- AionUi 会自动检测已安装的 CLI 工具
- 在设置中查看已检测到的 Agent

**手动添加：**
- 设置 → Agents → Add Agent
- 选择 Agent 类型并配置路径

### MCP 工具配置

**什么是 MCP？**
MCP (Model Context Protocol) 是一个开放标准，让 AI 应用连接到本地和远程资源。

**配置步骤：**
1. 设置 → MCP Servers
2. 添加 MCP 服务器配置
3. 配置会自动同步到所有 Agent

**常用 MCP 工具：**
- 文件系统访问
- 数据库查询
- API 调用
- 自定义工具

### WebUI 配置

**启用 WebUI：**
1. 设置 → WebUI Settings
2. 启用 WebUI Mode
3. 配置端口和访问权限

**远程访问：**
- 局域网访问：配置 LAN 设置
- 互联网访问：使用内网穿透或公网服务器
- 安全设置：配置密码登录

**聊天平台集成：**
- **Telegram**: 配置 Bot Token
- **Lark (飞书)**: 配置飞书机器人
- **钉钉**: 配置钉钉应用
- **Slack**: 即将支持

### 定时任务配置

1. 在对话中点击 "Schedule Task"
2. 设置任务名称和执行时间
3. 输入任务描述（自然语言）
4. 保存并启用

**Cron 表达式示例：**
- `0 9 * * *` — 每天早上 9 点
- `0 9 * * 1` — 每周一早上 9 点
- `0 0 1 * *` — 每月 1 号凌晨

---

## 使用技巧

### 提示词技巧

**文件操作：**
```
请帮我整理 ~/Downloads 文件夹，按类型分类到子文件夹中
```

**Excel 分析：**
```
分析这个 Excel 文件，找出销售额最高的 5 个产品，并生成月度趋势图
```

**文档生成：**
```
根据这个大纲生成一份 20 页的 PPT 演示文稿，主题是 "2024 年度营销计划"
```

**代码生成：**
```
用 Python 写一个脚本，批量重命名当前文件夹中的所有图片文件
```

### 快捷键

| 快捷键 | 功能 |
|:-------|:-----|
| `Cmd/Ctrl + N` | 新建对话 |
| `Cmd/Ctrl + /` | 聚焦输入框 |
| `Cmd/Ctrl + Enter` | 发送消息 |
| `Cmd/Ctrl + Shift + P` | 切换预览面板 |
| `Cmd/Ctrl + ,` | 打开设置 |

### 工作流建议

1. **为不同任务使用不同对话**
   - 每个对话保持独立上下文
   - 使用清晰的对话标题

2. **利用内置助手**
   - PPT 生成用 "PPTX Generator"
   - 文件整理用 "Cowork"
   - UI 设计用 "UI/UX Pro Max"

3. **结合定时任务**
   - 日报/周报自动生成
   - 定期数据备份
   - 提醒和通知

4. **多 Agent 协作**
   - Claude Code 用于代码审查
   - 内置 Agent 用于文件操作
   - Gemini 用于创意生成

---

## 常见问题

### Q: 需要先安装 Gemini CLI 或 Claude Code 吗？

**A:** 不需要。AionUi 有内置 AI Agent，安装后立即可用。只需 Google 登录或输入任意 API Key。如果你还安装了 Claude Code 或 Gemini CLI 等 CLI 工具，AionUi 会自动检测并集成，提供更多能力。

### Q: AionUi 可以做什么？

**A:** AionUi 是你的私人协作工作区。内置 Agent 可以批量整理文件夹、处理 Excel 数据、生成文档、搜索网页、生成图像。使用多 Agent 模式，你还可以通过同一界面利用 Claude Code、Codex 和其他强大的 CLI Agent。

### Q: 免费吗？

**A:** AionUi 完全免费开源。你可以用 Google 账号登录免费使用 Gemini，或使用你喜欢的任何提供商的 API Key。

### Q: 数据安全吗？

**A:** 所有数据存储在本地 SQLite 数据库中。不会上传到任何服务器。

### Q: 支持哪些 AI 模型？

**A:** 支持 20+ 平台，包括 Gemini、Claude、GPT、DeepSeek、Qwen、Ollama 本地模型等。详见 [配置指南](#llm-配置)。

### Q: 如何从手机访问？

**A:** 启用 WebUI 模式后，可以通过浏览器从手机访问。也可以配置 Telegram 或 Lark Bot，在聊天应用中直接使用。

### Q: 定时任务如何工作？

**A:** 在对话中创建定时任务，Agent 会在指定时间自动执行任务并发送结果到对话。支持 Cron 表达式实现复杂的调度逻辑。

### Q: 可以同时使用多个 AI Agent 吗？

**A:** 可以。多 Agent 模式支持同时运行多个 Agent，每个 Agent 有独立的上下文和会话历史。

---

## 社区资源

### 官方资源

- **GitHub**: https://github.com/iOfficeAI/AionUi
- **官网**: https://www.aionui.com
- **文档**: https://github.com/iOfficeAI/AionUi/wiki

### 社区

- **Discord (英文)**: https://discord.gg/2QAwJn7Egx
- **微信群**: 扫描官方仓库中的二维码
- **Twitter/X**: https://twitter.com/AionUI

### 反馈与贡献

- **报告问题**: https://github.com/iOfficeAI/AionUi/issues
- **功能请求**: https://github.com/iOfficeAI/AionUi/issues
- **讨论**: https://github.com/iOfficeAI/AionUi/discussions
- **贡献指南**: 见 [CONTRIBUTING.md](https://github.com/iOfficeAI/AionUi/blob/main/.github/CONTRIBUTING.md)

### 相关视频

- [WorldofAI 评测](https://www.youtube.com/watch?v=yUU5E-U5B3M) (200K 订阅)
- [Julian Goldie SEO 评测](https://www.youtube.com/watch?v=enQnkKfth10) (318K 订阅)

---

## 许可证

本项目采用 [Apache-2.0](https://github.com/iOfficeAI/AionUi/blob/main/LICENSE) 许可证。

---

## 更新日志

查看最新版本和更新内容：https://github.com/iOfficeAI/AionUi/releases

---

**如果觉得有用，请给项目一个 Star ⭐**

---

*文档生成时间: 2026-03-22*
*基于 AionUI v1.8.32*
