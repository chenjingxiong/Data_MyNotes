---
title: 开发者指南：使用 Skills 构建 ADK Agent
source: https://developers.googleblog.com/developers-guide-to-building-adk-agents-with-skills/
author: Google for Developers Blog
published: 2026-04-01
tags:
  - AI
  - Agent
  - ADK
  - Google
  - SkillToolset
---

# 开发者指南：使用 Skills 构建 ADK Agent

> **原文**: [Developer's Guide to Building ADK Agents with Skills](https://developers.googleblog.com/developers-guide-to-building-adk-agents-with-skills/)
> **发布日期**: 2026-04-01

---

你的 AI Agent 能遵循指令，但它能**编写新指令**吗？Agent Development Kit (ADK) 的 `SkillToolset` 使 Agent 能够按需加载领域专业知识。通过正确的 Skill 配置，你的 Agent 可以在运行时生成全新的能力。无论你需要一个安全审查清单、合规审计还是数据管道验证器，工作流程都很简单：**生成、加载、使用**。

`SkillToolset` 通过**渐进式披露 (Progressive Disclosure)** 实现这一点。这种架构模式允许 Agent 在需要时精确加载上下文，而不是将数千个 Token 塞进一个臃肿的系统提示中。

在本指南中，我们将介绍**四种实用的 Skill 模式**：

1. **内联清单 (Inline Checklist)：** 一种基本的、硬编码的 Skill 实现。
2. **基于文件的 Skill (File-based Skill)：** 加载外部指令和资源。
3. **外部导入 (External Import)：** 利用社区驱动的 Skill 仓库。
4. **Skill 工厂 (Skill Factory)：** 一种自我扩展的模式，Agent 按需编写新的 Skill。

每种模式都建立在前一种之上，最终形成一个能够动态扩展自身能力的 Agent 架构。

---

## 单体提示的问题

大多数 AI Agent 直接从系统提示中获取领域知识。开发者经常将合规规则、风格指南、API 参考和故障排除程序拼接到一个巨大的指令字符串中。

当 Agent 只有两三个能力时，这种方式还行。然而，当你扩展到十个或更多任务时，将所有这些指令拼接到系统提示中，会在每次 LLM 调用时消耗数千个 Token——无论用户的具体查询是否真正需要这些知识。

Agent Skills 规范通过**渐进式披露**解决了这个问题。它将知识加载分为三个不同层级：

| 层级 | 内容 | 大小 | 加载时机 |
|------|------|------|----------|
| **L1 元数据** | Skill 名称和描述 | ~100 Token/Skill | 启动时加载，作为 Agent 扫描的菜单 |
| **L2 指令** | 完整的 Skill 主体 | <5,000 Token | Agent 显式激活某个 Skill 时通过 API 加载 |
| **L3 资源** | 外部参考文件（风格指南、API 规范等） | 按需 | 仅当 Skill 的指令需要时才加载 |

通过这种架构，一个拥有 10 个 Skill 的 Agent 在每次调用时仅使用约 1,000 个 Token 的 L1 元数据，而不是单体提示中的 10,000 个 Token。这意味着**基线上下文使用量减少了约 90%**。

![[attachments/adk-progressive-disclosure.png]]

ADK 通过 `SkillToolset` 类实现这一点，该类自动生成三个工具：`list_skills`（L1）、`load_skill`（L2）和 `load_skill_resource`（L3）。

---

## 模式一：内联 Skill（便签式）

最简单的模式：一个包含 `name`、`description` 和 `instructions` 的 Python 对象，直接在 Agent 代码中定义。适用于**小型、稳定、很少变更**的规则。

```python
# ADK 伪代码：模式一 - 内联 Skill

seo_skill = models.Skill(
    frontmatter=models.Frontmatter(
        name="seo-checklist",
        description="博客文章的 SEO 优化清单。涵盖标题标签、元描述、标题结构和可读性。",
    ),
    instructions=(
        "优化博客文章的 SEO 时，逐项检查：\n"
        "1. 标题：50-60 个字符，主要关键词靠近开头\n"
        "2. 元描述：150-160 个字符，包含行动号召\n"
        "3. 标题层级：H2/H3 层次结构，2-3 个标题包含关键词\n"
        "4. 第一段：前 100 个字中包含主要关键词\n"
        "5. 图片：带关键词的 Alt 文本，已压缩，描述性文件名\n"
        "根据每个检查项审查内容并提出改进建议。"
    ),
)
```

`frontmatter` 字段成为 L1 元数据，LLM 在每次调用时都能看到。`instructions` 成为 L2，仅在 Agent 判定此 Skill 相关时才加载。当被问到"帮我审查博客文章的 SEO"时，Agent 会加载此 Skill 并系统地应用每个检查项。

![[attachments/adk-inline-skill-seo-review.png]]

---

## 模式二：基于文件的 Skill（参考手册式）

内联 Skill 适用于简单的清单。但如果你的 Skill 需要参考文档（风格指南、API 规范），就需要一个目录结构。

基于文件的 Skill 存在于独立目录中，包含一个 `SKILL.md` 文件以及可选的子目录（用于参考文档、资源和脚本）。`SKILL.md` 以 YAML frontmatter 开头，后跟 Markdown 格式的指令。

```
skills/blog-writer/
├── SKILL.md           # L2: 指令
└── references/
    └── style-guide.md # L3: 按需加载
```

这种设计将知识分为两层：`SKILL.md` 中的指令（L2）告诉 Agent 遵循哪些步骤；`references/style-guide.md` 文件（L3）为每个步骤提供详细的领域知识。Agent 仅在其指令要求时才通过 `load_skill_resource` 工具加载参考文件。

```python
# ADK 伪代码：模式二 - 基于文件的 Skill

blog_writer_skill = load_skill_from_dir(
    pathlib.Path(__file__).parent / "skills" / "blog-writer"
)
```

![[attachments/adk-l3-resource-loading.png]]

基于文件的 Skill 使知识可复用。任何遵循 agentskills.io 规范的 Agent 都可以加载同一个目录。但在这种场景下，你仍然需要自己编写 `SKILL.md`。

---

## 模式三：外部 Skill（导入式）

外部 Skill 的工作方式与基于文件的 Skill 完全相同。唯一的区别在于目录的来源。你不是自己编写 `SKILL.md`，而是从社区仓库（如 awesome-claude-skills）下载，并使用相同的 `load_skill_from_dir` 调用来加载。

```python
# ADK 伪代码：模式三 - 外部 Skill（相同 API，不同来源）

content_researcher_skill = load_skill_from_dir(
    pathlib.Path(__file__).parent / "skills" / "content-research-writer"
)
```

代码与模式二完全相同。agentskills.io 规范定义了通用的目录格式，所以 `load_skill_from_dir` 不关心是你自己写的 `SKILL.md` 还是从网上下载的。Google 使用相同格式发布官方 ADK 开发 Skill，可通过 `npx skills add google/adk-docs -y -g` 安装。

---

## 模式四：Skill 工厂（自我扩展）

前三种模式涵盖了**已存在的**、**你编写的**或**你发现的** Skill。模式四则形成了闭环：**Agent 编写自己的 Skill**。

### 元技能 (Meta Skill) 的概念

元技能是一种目的在于**生成新的 `SKILL.md` 文件**的 Skill。配备元技能的 Agent 变成**自我扩展的**——它可以在运行时编写并加载新的 Skill 定义，无需人工干预来扩展自身能力。

Skill Creator 是一个内联 Skill，其指令解释如何编写有效的 `SKILL.md` 文件。关键在于 `resources` 字段——它将 agentskills.io 规范本身和一个工作示例作为 L3 参考嵌入其中。当被要求创建新 Skill 时，Agent 读取这些参考并生成符合规范的 `SKILL.md`。

```python
# ADK 伪代码：模式四 - 元技能（Skill 工厂）

skill_creator = models.Skill(
    frontmatter=models.Frontmatter(
        name="skill-creator",
        description=(
            "根据需求创建新的 ADK 兼容 Skill 定义。"
            " 遵循 agentskills.io 上的 Agent Skills 规范"
            " 生成完整的 SKILL.md 文件。"
        ),
    ),
    instructions=(
        "当被要求创建新 Skill 时，生成完整的 SKILL.md 文件。\n\n"
        "阅读 `references/skill-spec.md` 获取格式规范。\n"
        "阅读 `references/example-skill.md` 获取工作示例。\n\n"
        "遵循以下规则：\n"
        "1. 名称必须使用 kebab-case，最多 64 个字符\n"
        "2. 描述不超过 1024 个字符\n"
        "3. 指令应清晰、分步骤\n"
        "4. 将详细领域知识放在 references/ 中的参考文件里\n"
        "5. 保持 SKILL.md 在 500 行以内，详细内容放在 references/\n"
        "6. 输出完整的文件内容，用户可以直接保存\n"
    ),
    resources=models.Resources(
        references={
            "skill-spec.md": "# Agent Skills Specification (agentskills.io)...",
            "example-skill.md": "# Example: Code Review Skill...",
        }
    ),
)
```

`resources` 字段是 `models.Resources` 类发挥作用的地方。引用将 agentskills.io 规范作为 `skill-spec.md` 嵌入，将一个可用的代码审查 Skill 作为 `example-skill.md` 嵌入。当 Agent 调用 `load_skill_resource("skill-creator", "references/skill-spec.md")` 时，它会获得定义有效 Skill 结构的完整规范。

> [!tip] 生成 Skill 的最佳实践
> 虽然自动生成 Skill 是一种强大的工作流程，但我们建议**保持人工参与**来审查最终的 `SKILL.md`。作为构建任何 Skill 的标准实践，你应该测试其有效性。你可以通过 ADK 构建健壮的评估来确保你的 Skill 在部署前完全按预期工作。

---

## Skill 工厂实战

向 Agent 提问："我需要一个用于审查 Python 代码安全漏洞的新 Skill。"

Agent 加载 Skill Creator，通过 `load_skill_resource` 读取规范和示例，然后生成一个完整的 Python 安全审查 Skill——包含有效的 kebab-case 命名、涵盖输入验证、身份验证和密码学的结构化指令，以及基于严重性的报告格式。

![[attachments/adk-meta-skill-creator-output.png]]

生成的 Skill 遵循相同的 agentskills.io 规范，因此它不仅在 ADK 中可用，还在任何兼容的 Agent 中都可用：Gemini CLI、Claude Code、Cursor 以及 40 多个采用该格式的其他产品。

---

## 将一切整合起来

定义好所有四个 Skill 后，将它们打包成 `SkillToolset` 并交给 Agent 只需几行代码：

```python
# ADK 伪代码：组装 Skill 工厂

skill_toolset = SkillToolset(
    skills=[seo_skill, blog_writer_skill, content_researcher_skill, skill_creator]
)

root_agent = Agent(
    model="gemini-2.5-flash",
    name="blog_skills_agent",
    description="由可复用 Skill 驱动的博客写作 Agent。",
    instruction=(
        "你是一个拥有专业 Skill 的博客写作助手。\n"
        "加载相关的 Skill 以获取详细指令。\n"
        "使用 load_skill_resource 访问参考材料。\n"
        "遵循每个 Skill 的分步指令。\n"
        "始终说明你正在使用哪个 Skill 以及为什么。"
    ),
    tools=[skill_toolset],
)
```

列表中的前三个 Skill 分别处理 SEO、写作和研究。第四个 `skill_creator` 是工厂。向这个 Agent 提问*"创建一个用于撰写技术博客引言的 Skill"*，它会即时生成一个新的 `SKILL.md`：

```markdown
# 由 skill-creator 生成
---
name: blog-intro-writer
description: 撰写引人入胜的技术博客引言。通过问题陈述吸引读者，
  建立相关性，并预览读者将学到的内容。
---

撰写博客引言时，遵循以下结构：
1. 以读者能识别的具体问题开篇
2. 说明为什么现在重要（新版本发布、扩展痛点、常见错误）
3. 用一句话预览文章涵盖的内容
4. 保持在 100 字以内
```

Agent 使用 `seo-checklist` 和 `blog-writer` Skill 处理已有任务。当需要它不具备的能力时，它就自己写一个。新 Skill 遵循相同的 agentskills.io 规范，因此你可以将其保存到 `skills/blog-intro-writer/SKILL.md`，并在下次会话中通过 `load_skill_from_dir` 加载。

`SkillToolset` 自动生成三个直接映射到渐进式披露的工具：`list_skills`（L1，自动注入）、`load_skill`（L2，按需）和 `load_skill_resource`（L3，按需）。

![[attachments/adk-skilltoolset-flow.png]]

---

## 实用技巧

- **描述就是你的 API 文档。** `description` 字段是 LLM 在 L1 看到的内容，用于决定是否加载某个 Skill。*"博客文章的 SEO 优化清单"* 准确告诉 Agent 何时激活。*"一个有用的 Skill"* 则不能。
- **从内联开始，逐步升级到文件。** 不要过度工程化。如果你的 Skill 在 10 行指令内就能搞定，保持内联。当你需要参考文档或想跨 Agent 复用时，再升级为基于文件的 Skill。
- **像审查依赖一样审查生成的 Skill。** 元技能的输出会成为你的 Agent 的行为。像代码审查一样对待生成的 `SKILL.md` 文件——**部署前先阅读**。

---

## 开始使用

准备好构建你自己的 Skill 工厂了吗？查看 [ADK Skills 文档](https://google.github.io/adk-docs/skills/) 了解 `SkillToolset` 和渐进式披露，并克隆 [GitHub 仓库](https://github.com/google/adk-python) 运行所有四种模式。
