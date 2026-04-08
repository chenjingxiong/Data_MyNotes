# Claude Code 必装 Skills 推荐指南

> 2026 年 Claude Code Skills 生态已突破 **20 万+** 个，单热门 Skill 安装量超 **100 万**。本文从主流推荐、社区好评、实用价值三个维度精选出最值得安装的 Skills，帮你快速提升效率。

---

## TL;DR — 精选推荐速查表

| # | Skill | 一句话推荐 | 适用人群 | 来源 |
|---|-------|-----------|---------|------|
| 1 | **frontend-design** | 官方出品，告别"AI 味"设计 | 前端 / 全栈 | Anthropic 官方 |
| 2 | **webapp-testing** | 一句话自动生成 E2E 测试 | 前端 / QA | 社区热门 |
| 3 | **plan** | 需求→分步计划，风险评估 | 全栈 / 架构师 | ECC |
| 4 | **tdd / tdd-workflow** | 红-绿-重构，测试先行 | 所有开发者 | ECC / 社区 |
| 5 | **deep-research** | 多源深度调研自动整合 | 研究者 / PM | ECC |
| 6 | **mcp-builder** | 10 分钟搭建 MCP Server | 工具开发者 | Anthropic 官方 |
| 7 | **security-review** | OWASP Top 10 漏洞扫描 | 安全 / 后端 | ECC |
| 8 | **verification-loop** | 构建→静态分析→测试→覆盖率全链路 | DevOps / 后端 | ECC |
| 9 | **article-writing** | 高质量长文生成 | 技术写作者 | ECC |
| 10 | **Remotion** | React 写代码 = 自动生成视频 | 内容创作者 | Remotion 官方 |
| 11 | **skill-creator** | 用 Skill 生成 Skill | 进阶用户 | Anthropic 官方 |

---

## Skill 安装方法概述

> 详细安装说明见 [[#安装方法详解]]

**三种主流安装方式：**

| 方式 | 命令 / 操作 | 难度 |
|------|------------|------|
| **插件市场** | Claude Code 内运行 `/install-plugin <skill-name>` 或使用 Plugin Browser | ⭐ 最简单 |
| **手动安装** | 下载 `SKILL.md` 放入 `.claude/skills/` 目录 | ⭐⭐ |
| **一键批量** | `npx github:alirezarezvani/claude-skills` 安装 200+ Skills | ⭐⭐⭐ |

---

## 精选 Skill 详细介绍

### 1. frontend-design ⭐ 官方推荐

**推荐理由：** Anthropic 官方出品，专门对抗"AI 生成味"审美，能产出生产级的前端界面。

**简介：**
- 生成高质量 HTML/CSS/React 代码
- 内建设计哲学系统（Tailwind + shadcn/ui）
- 避免常见的 AI 设计通病（千篇一律的渐变、间距混乱等）
- 支持 Figma 设计稿复刻

**适用场景：** 快速原型、Landing Page、Dashboard UI

**来源：** Anthropic 官方 Skills 仓库

**安装：**
```bash
# 插件市场安装
/install-plugin frontend-design

# 或手动下载
curl -o .claude/skills/frontend-design/SKILL.md \
  https://raw.githubusercontent.com/anthropics/skills/main/frontend-design/SKILL.md
```

---

### 2. webapp-testing 🔥 社区爆款

**推荐理由：** "帮我测试登录流程" —— 一句话生成并运行完整的 Playwright E2E 测试，极大降低测试门槛。

**简介：**
- 基于 Playwright 的浏览器自动化测试
- 支持截图、日志捕获、UI 行为验证
- 自动生成测试代码，可直接运行
- 支持 Claude Code / Codex / Copilot 等多平台

**适用场景：** 登录流程测试、表单验证、跨浏览器兼容性测试、UI 回归测试

**来源：** [ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills) 收录

**安装：**
```bash
# 从 awesome-claude-skills 安装
curl -o .claude/skills/webapp-testing/SKILL.md \
  https://raw.githubusercontent.com/ComposioHQ/awesome-claude-skills/master/webapp-testing/SKILL.md
```

---

### 3. plan 🧠 架构师思维

**推荐理由：** 编码前先规划，自动评估风险、拆解任务、生成分步实施计划，避免盲目编码。

**简介：**
- 将模糊需求转化为结构化实施计划
- 自动识别风险点和技术依赖
- 输出带优先级的任务列表
- 与后续 tdd / verification-loop 无缝衔接

**适用场景：** 新功能规划、大型重构、技术方案评审

**来源：** Everything Claude Code (ECC) — Anthropic Hackathon 冠军项目

**安装：** 已包含在 ECC 中，或：
```bash
# ECC 安装（包含 plan + 50+ 其他 Skills）
git clone https://github.com/affaan-m/everything-claude-code ~/.claude/ecc
```

---

### 4. tdd / tdd-workflow 🧪 测试先行

**推荐理由：** 强制执行红-绿-重构循环，确保代码质量从第一行就达标，覆盖率达 80%+。

**简介：**
- `tdd`：通用 TDD 工作流（接口→测试→实现→重构）
- `tdd-workflow`：更强制的 TDD 流程约束
- 支持 Jest、Pytest、JUnit、Vitest 等主流框架
- 自动分析测试覆盖率

**适用场景：** 所有需要写测试的开发工作

**来源：** ECC + 社区通用

**安装：**
```bash
# ECC 包含，或单独安装
curl -o .claude/skills/tdd/SKILL.md \
  https://raw.githubusercontent.com/ComposioHQ/awesome-claude-skills/master/tdd/SKILL.md
```

---

### 5. deep-research 🔬 深度调研

**推荐理由：** 自动聚合多源信息（Firecrawl + Exa），生成结构化研究报告，适合需要深入调研的场景。

**简介：**
- 多搜索引擎并行检索
- 自动提取、去重、整合信息
- 输出带引用的结构化报告
- 支持递进式检索（iterative-retrieval）

**适用场景：** 技术选型调研、竞品分析、市场研究、论文综述

**来源：** ECC（需要 Firecrawl / Exa MCP 配合）

**安装：**
```bash
# 需要配合 MCP 使用，安装 ECC 后自动可用
# 需要配置 Firecrawl 和 Exa API Key
```

---

### 6. mcp-builder 🔧 工具构建

**推荐理由：** 10 分钟从零搭建一个 MCP Server，大幅降低自定义工具的门槛。

**简介：**
- MCP (Model Context Protocol) Server 生成向导
- 自动处理协议细节、类型定义
- 内建最佳实践和错误处理模式
- 支持多种 MCP 传输方式

**适用场景：** 自定义 Claude Code 工具、企业内部 API 集成、第三方服务对接

**来源：** Anthropic 官方 Skills

**安装：**
```bash
/install-plugin mcp-builder
```

---

### 7. security-review 🛡️ 安全守门人

**推荐理由：** 代码写完自动过一遍 OWASP Top 10，发现 SSRF、注入、不安全加密等常见漏洞。

**简介：**
- 覆盖 OWASP Top 10 漏洞检测
- 自动扫描认证、输入验证、敏感数据处理
- 检测 Secrets 泄露、不安全加密
- 输出分级风险报告（Critical / High / Medium / Low）

**适用场景：** 代码审查、上线前安全检查、合规审计

**来源：** ECC

**安装：** 包含在 ECC 中

---

### 8. verification-loop ✅ 全链路验证

**推荐理由：** 一条命令走完 构建→静态分析→测试→覆盖率检查 全流程，CI/CD 前的最后一道关。

**简介：**
- 自动执行构建检查
- 集成静态代码分析
- 运行测试套件并收集覆盖率
- 生成验证报告

**适用场景：** PR 提交前验证、CI/CD 集成、质量关卡

**来源：** ECC

**安装：** 包含在 ECC 中

---

### 9. article-writing ✍️ 技术写作

**推荐理由：** 生成结构清晰、可读性强的技术文章、博客、教程、Newsletter。

**简介：**
- 支持多种文体：技术博客、教程、指南、Newsletter
- 自动生成大纲 → 填充内容 → 润色
- 支持 SEO 优化建议
- 输出 Markdown 格式，直接发布

**适用场景：** 技术博客写作、文档撰写、知识分享

**来源：** ECC

**安装：** 包含在 ECC 中

---

### 10. Remotion 🎬 视频即代码

**推荐理由：** 2026 年推特最火的 Claude Skill，React 组件直接渲染成视频，视频制作变成 git commit。

**简介：**
- 用 React 组件定义视频内容
- Chromium 无头浏览器 + FFmpeg 渲染
- 支持动画、字幕、配音、背景音乐
- Lambda 函数云端渲染（2048-3000MB）
- 配合 faster-whisper 实现精准音频同步

**适用场景：** 技术教程视频、产品 Demo、社交媒体短视频、自动化视频内容生产

**来源：** [Remotion 官方](https://www.remotion.dev/docs/ai/claude-code)

**安装：**
```bash
# Remotion 官方 Skill
npx create-video@latest
# 或从 Remotion 官方仓库获取 Claude Code Skill
```

**相关项目：**
- **OpenMontage** — 基于 Remotion 的全自动视频生产引擎（3 天 250+ stars）

---

### 11. skill-creator 🪄 元技能

**推荐理由：** 用 Skill 来创建 Skill，进阶用户的必备工具。

**简介：**
- 分析本地 git 历史提取编码模式
- 自动生成 `SKILL.md` 文件
- 内建 Skill 编写最佳实践
- 支持从现有项目模式中学习

**适用场景：** 团队编码规范提取、自定义 Skill 开发

**来源：** Anthropic 官方 Skills

**安装：**
```bash
/install-plugin skill-creator
```

---

## 按场景推荐组合

### 🆕 新手起步包（通用）
```
plan → tdd → verification-loop → security-review
```
> 从规划到验证的完整开发链路

### 🎨 前端开发者
```
frontend-design → webapp-testing → frontend-patterns → frontend-slides
```
> 从设计到测试到演示的完整前端工具链

### 🔬 研究者 / 知识工作者
```
deep-research → article-writing → pdf (Skill)
```
> 调研→写作→输出的知识生产流水线

### 🛠️ 工具开发者
```
mcp-builder → skill-creator → claude-api
```
> 打造自定义 Claude 工具生态

### 🎬 内容创作者
```
Remotion → frontend-design → article-writing → video-editing
```
> 全自动化内容生产

### 🏢 企业级开发
```
plan → tdd → {language}-review → security-review → verification-loop → deployment-patterns
```
> 满足企业级质量要求的完整开发流程

---

## 安装方法详解

### 方法一：Plugin Marketplace（推荐新手）

在 Claude Code 中直接运行：

```bash
/install-plugin <skill-name>
```

或使用 Plugin Browser 可视化浏览和安装。

### 方法二：手动安装

1. 创建 Skills 目录：
```bash
mkdir -p .claude/skills/<skill-name>
```

2. 下载 `SKILL.md`：
```bash
curl -o .claude/skills/<skill-name>/SKILL.md \
  <SKILL.md 的 URL>
```

3. 重启 Claude Code 即可生效。

**文件结构示例：**
```
.claude/
├── skills/
│   ├── frontend-design/
│   │   └── SKILL.md
│   ├── webapp-testing/
│   │   └── SKILL.md
│   └── tdd/
│       └── SKILL.md
```

### 方法三：ECC 一键安装

安装 Everything Claude Code 获取 50+ Skills：

```bash
git clone https://github.com/affaan-m/everything-claude-code ~/.claude/ecc
```

### 方法四：批量安装 200+ Skills

```bash
npx github:alirezarezvani/claude-skills
```

> ⚠️ 不推荐一次安装太多 Skills，可能影响上下文窗口效率。按需选择最重要。

---

## 主要 Skills 来源

| 来源 | 说明 | 链接 |
|------|------|------|
| **Anthropic 官方** | 官方维护的高质量 Skills | [platform.claude.com/docs](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) |
| **Everything Claude Code** | Anthropic Hackathon 冠军项目，50+ Skills | [github.com/affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code) |
| **ComposioHQ/awesome-claude-skills** | 社区最热门合集，⭐ ~50k | [github.com/ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills) |
| **travisvn/awesome-claude-skills** | 另一个精心维护的合集 | [github.com/travisvn/awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills) |
| **hesreallyhim/awesome-claude-code** | Skills + Hooks + Slash Commands | [github.com/hesreallyhim/awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code) |
| **Gradually AI** | 600+ Skills 可搜索目录 | [gradually.ai/en/claude-code-skills](https://www.gradually.ai/en/claude-code-skills/) |
| **Remotion 官方** | 视频 Skills | [remotion.dev/docs/ai/claude-code](https://www.remotion.dev/docs/ai/claude-code) |

---

## 实用技巧

### 1. Skill 加载机制
Claude Code 会根据当前任务**动态发现和加载**相关的 Skills。只有在需要时才会读取，不会浪费上下文窗口。

### 2. 保持 SKILL.md 精简
Anthropic 官方建议每个 `SKILL.md` **不超过 500 行**。超过时考虑拆分为多个文件。

### 3. 优先级设置
可以为常用 Skill 设置触发关键词，确保在特定任务中优先加载。

### 4. 与 MCP 配合
部分 Skills（如 `deep-research`）需要配合 MCP Server 使用，确保先配置好对应的 MCP 工具。

### 5. 自定义 Skill
使用 `skill-creator` 从你的项目 git 历史中提取编码模式，生成团队专属 Skill。

---

## 参考资源

- [[Everything-claude-Code/Everything-Claude-Code使用指南|ECC 完整使用指南]]
- [[Everything-claude-Code/Skills索引|ECC Skills 索引]]
- [[Everything-claude-Code/快速参考卡片|ECC 快速参考]]

---

*更新日期: 2026-04-08*
*数据来源: ComposioHQ/awesome-claude-skills, Gradually AI, Reddit r/ClaudeAI, Anthropic 官方文档*
