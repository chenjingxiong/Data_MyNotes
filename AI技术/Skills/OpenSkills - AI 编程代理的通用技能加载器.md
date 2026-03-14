# OpenSkills - AI 编程代理的通用技能加载器

## 项目概述

**GitHub**: https://github.com/numman-ali/openskills

**OpenSkills** 是一个为 AI 编程代理设计的通用技能加载器，它将 Anthropic 的技能系统带到了每一个 AI 编程工具中。简单来说，它就像 SKILL.md 文件的"通用安装程序"。

- **Star 数**: 约 9k
- **License**: Apache 2.0
- **主要语言**: TypeScript (96.9%)
- **最新版本**: v1.5.0 (2026年1月17日)

## 核心理念

### "One CLI. Every Agent. Same Format as Claude Code."

OpenSkills 的核心理念是**统一性**：
- 一个命令行工具
- 支持所有 AI 编程代理
- 使用与 Claude Code 完全相同的格式

这使得技能可以在不同的 AI 工具之间无缝共享，无论是 Claude Code、Cursor、Windsurf、Aider 还是 Codex。

## 为什么需要 OpenSkills？

### 问题背景

在 AI 辅助编程领域，不同的工具有不同的扩展和插件系统：
- **Claude Code** 有自己的 Skills 系统（SKILL.md 格式）
- **Cursor** 有自己的插件生态
- **其他工具** 各自有不同的扩展方式

这导致了技能无法在不同工具间共享，用户需要为每个工具重复配置相同的技能。

### OpenSkills 的解决方案

OpenSkills 通过以下方式解决了这个问题：

1. **完全兼容 Claude Code 格式**
   - 使用相同的提示格式
   - 使用相同的技能市场
   - 使用相同的文件夹结构

2. **通用性**
   - 适用于 Claude Code、Cursor、Windsurf、Aider、Codex 等
   - 任何能读取 `AGENTS.md` 的工具都可以使用

3. **渐进式加载**
   - 仅在需要时加载技能
   - 保持上下文整洁

4. **仓库友好**
   - 技能驻留在项目中
   - 可以进行版本控制

5. **私有仓库支持**
   - 可从本地路径或私有 git 仓库安装

## 工作原理

### Claude Code 的 Skills 系统

Claude Code 将技能作为 `SKILL.md` 文件分发，并在 `<available_skills>` 块中暴露它们：

```xml
<available_skills>
<skill>
<name>pdf</name>
<description>Comprehensive PDF manipulation toolkit for extracting text and tables...</description>
<location>plugin</location>
</skill>
</available_skills>
```

### OpenSkills 的实现

OpenSkills 在你的 `AGENTS.md` 中生成**完全相同**的 `<available_skills>` XML，并通过以下方式加载技能：

```bash
npx openskills read <skill-name>
```

这样，任何能读取 `AGENTS.md` 的代理都可以使用 Claude Code 技能，而不需要 Claude Code 本身。

### 对比表

| 方面 | Claude Code | OpenSkills |
|------|-------------|------------|
| **提示格式** | `<available_skills>` XML | 相同的 XML |
| **技能存储** | `.claude/skills/` | `.claude/skills/`（默认）|
| **调用方式** | `Skill("name")` 工具 | `npx openskills read <name>` |
| **市场** | Anthropic 市场 | GitHub (anthropics/skills) |
| **渐进式加载** | ✅ | ✅ |

## 主要功能

### 1. 技能安装

```bash
# 从 Anthropic 市场安装
npx openskills install anthropics/skills

# 从任何 GitHub 仓库安装
npx openskills install your-org/your-skills

# 从本地路径安装
npx openskills install ./local-skills/my-skill

# 从私有 Git 仓库安装
npx openskills install git@github.com:your-org/private-skills.git
```

### 2. 技能同步

```bash
# 更新 AGENTS.md
npx openskills sync

# 跳过提示并输出到自定义路径
npx openskills sync -y -o custom-path.md
```

### 3. 技能管理

```bash
# 列出已安装的技能
npx openskills list

# 加载技能（供代理使用）
npx openskills read <name>

# 更新已安装的技能（默认全部）
npx openskills update

# 更新特定技能
npx openskills update git-workflow,check-branch-first

# 交互式管理（删除技能）
npx openskills manage

# 删除特定技能
npx openskills remove <name>
```

### 4. 命令行选项

- `--global` - 全局安装到 `~/.claude/skills`（默认为项目本地安装）
- `--universal` - 安装到 `.agent/skills/` 而不是 `.claude/skills/`
- `-y, --yes` - 跳过提示（适用于 CI）
- `-o, --output <path>` - sync 的输出文件（默认为 `AGENTS.md`）

## SKILL.md 格式

OpenSkills 使用 Anthropic 的确切格式：

```markdown
---
name: pdf
description: Comprehensive PDF manipulation toolkit for extracting text and tables, creating new PDFs, merging/splitting documents, and handling forms.
---

# PDF Skill Instructions

When the user asks you to work with PDFs, follow these steps:
1. Install dependencies: `pip install pypdf2`
2. Extract text using scripts/extract_text.py
3. Use references/api-docs.md for details
```

技能按需加载，保持代理上下文的整洁和专注。

## 创建自定义技能

### 最小结构

```
my-skill/
└── SKILL.md
```

### 带资源的完整结构

```
my-skill/
├── SKILL.md
├── references/
├── scripts/
└── assets/
```

### 安装自己的技能

```bash
npx openskills install ./my-skill
```

### 使用符号链接进行本地开发

```bash
git clone git@github.com:your-org/my-skills.git ~/dev/my-skills
mkdir -p .claude/skills
ln -s ~/dev/my-skills/my-skill .claude/skills/my-skill
```

### 技能创作指南

```bash
npx openskills install anthropics/skills
npx openskills read skill-creator
```

## Universal 模式（多代理设置）

如果你同时使用 Claude Code 和其他代理，并共享一个 `AGENTS.md`，可以安装到 `.agent/skills/` 以避免与 Claude 的插件市场冲突：

```bash
npx openskills install anthropics/skills --universal
```

**优先级顺序（优先级高的优先）：**

1. `./.agent/skills/`
2. `~/.agent/skills/`
3. `./.claude/skills/`
4. `~/.claude/skills/`

## AGENTS.md 格式示例

OpenSkills 生成的 AGENTS.md 格式：

```markdown
<skills_system priority="1">

## Available Skills

<!-- SKILLS_TABLE_START -->
<usage>
When users ask you to perform tasks, check if any of the available skills below can help complete the task more effectively.

How to use skills:
- Invoke: `npx openskills read <skill-name>` (run in your shell)
- The skill content will load with detailed instructions
- Base directory provided in output for resolving bundled resources

Usage notes:
- Only use skills listed in <available_skills> below
- Do not invoke a skill that is already loaded in your context
</usage>

<available_skills>

<skill>
<name>pdf</name>
<description>Comprehensive PDF manipulation toolkit for extracting text and tables, creating new PDFs, merging/splitting documents, and handling forms...</description>
<location>project</location>
</skill>

</available_skills>
<!-- SKILLS_TABLE_END -->

</skills_system>
```

## FAQ

### 为什么使用 CLI 而不是 MCP？

**MCP 用于动态工具。** 技能是静态指令 + 资源。

- 技能只是文件 → 无需服务器
- 适用于每个代理 → 无需 MCP 支持
- 符合 Anthropic 的设计 → SKILL.md 就是规范

MCP 和技能解决不同的问题。OpenSkills 保持技能的轻量和通用性。

## 系统要求

- **Node.js** 20.6+
- **Git**（用于克隆仓库）

## 使用场景

### 1. 团队协作

团队成员可以共享自定义技能仓库，确保所有人使用相同的专业知识和最佳实践。

### 2. 项目特定技能

每个项目可以有自己的技能集，包含项目特定的架构、约定和工作流程。

### 3. 多工具环境

开发人员可以在不同的 AI 工具之间切换，而不需要重新配置或学习不同的技能系统。

### 4. 私有技能

企业可以创建和维护私有技能仓库，包含内部工具、流程和知识。

## 与替代方案的比较

### vs. MCP (Model Context Protocol)

| 特性 | OpenSkills | MCP |
|------|------------|-----|
| 用途 | 静态技能和知识 | 动态工具和资源 |
| 服务器要求 | 无 | 需要服务器 |
| 代理支持 | 所有能读取文件的代理 | 需要 MCP 支持 |
| 复杂度 | 低 | 中高 |

### vs. 手动复制粘贴

| 特性 | OpenSkills | 手动复制 |
|------|------------|----------|
| 版本控制 | ✅ | ❌ |
| 自动更新 | ✅ | ❌ |
| 团队共享 | ✅ | ❌ |
| 渐进式加载 | ✅ | ❌ |

## 总结

OpenSkills 是一个强大而简单的工具，它通过统一技能格式和提供通用加载器，解决了 AI 编程工具生态中的碎片化问题。它的价值在于：

1. **互操作性** - 让技能在不同工具间共享
2. **简洁性** - 保持技能格式简单，基于文件
3. **兼容性** - 完全兼容 Claude Code 格式
4. **灵活性** - 支持公共和私有技能源

对于希望在多个 AI 编程工具中使用一致技能集的开发者和团队来说，OpenSkills 是一个值得考虑的解决方案。

---

**项目地址**: https://github.com/numman-ali/openskills

**注意**: 本项目实现了 Anthropic 的 Agent Skills 规范，但与 Anthropic 无关。Claude、Claude Code 和 Agent Skills 是 Anthropic, PBC 的商标。
