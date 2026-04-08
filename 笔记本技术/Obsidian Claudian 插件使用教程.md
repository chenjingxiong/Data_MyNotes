# Obsidian Claudian 插件使用教程

## 简介

Claudian 是一个 Obsidian 插件，它将 Anthropic 的 Claude AI 直接集成到你的 Obsidian 笔记库中，让你可以通过对话的方式管理笔记、分析代码、创建内容等。

## 核心功能

### 1. 智能对话助手
- 直接在 Obsidian 中与 Claude 对话
- 理解你的笔记库结构和内容
- 支持 Wiki-links 和 Markdown 格式

### 2. 笔记管理
- 创建、编辑、组织笔记
- 自动添加标签和分类
- 智能链接相关笔记

### 3. 代码分析
- 分析代码文件
- 重构和优化代码
- 生成代码注释和文档

### 4. 内容创作
- 帮助撰写文章和文档
- 生成大纲和结构
- 优化文本表达

## 安装步骤

### 方法一：通过 Obsidian 插件市场（推荐）

1. 打开 Obsidian 设置
2. 进入「第三方插件」
3. 关闭「安全模式」
4. 点击「浏览」并搜索 "Claudian"
5. 点击安装并启用

### 方法二：手动安装

1. 从 [GitHub Releases](https://github.com/your-repo/claude-obsidian) 下载最新版本
2. 将文件解压到 Obsidian 的 `plugins` 目录：
   ```
   .obsidian/plugins/claude-obsidian/
   ```
3. 在 Obsidian 设置中启用插件

## 配置设置

### API 密钥设置

1. 打开 Claudian 插件设置
2. 输入你的 Anthropic API 密钥
3. 选择要使用的 Claude 模型（Claude 3 Opus/Sonnet/Haiku）

### 工作目录设置

- 插件会自动识别当前 Obsidian 库作为工作目录
- 所有文件操作都使用相对路径
- 确保你的笔记库路径正确

## 使用方法

### 基本对话

在 Obsidian 中打开命令面板（`Ctrl/Cmd + P`），输入 "Claudian" 即可看到所有可用命令：

| 命令 | 功能 |
|------|------|
| `Claudian: 打开对话面板` | 打开主对话界面 |
| `Claudian: 快速提问` | 快速发送问题 |
| `Claudian: 分析当前笔记` | 分析并总结当前笔记 |

### 编辑器集成

选中文本后右键，可以看到 Claudian 的上下文菜单选项：

- **解释选中内容**：获取选中文本的详细解释
- **改写选中内容**：优化表达方式
- **扩展选中内容**：基于选中文本继续创作
- **翻译选中内容**：翻译为其他语言

### 笔记操作

Claudian 可以执行以下笔记操作：

```markdown
# 创建笔记
"帮我创建一篇关于 Python 装饰器的学习笔记"

# 链接笔记
"把这篇笔记和我之前的 Python 学习笔记关联起来"

# 重组笔记
"帮我把这些笔记按照主题重新组织"
```

## 高级技巧

### 1. 使用 Wiki-links

Claudian 完全支持 Obsidian 的 Wiki-link 语法，可以直接引用其他笔记：

```markdown
请参考 [[编程/Python/基础语法]] 了解更多信息
```

### 2. 标签管理

让 Claudian 帮你管理标签：

```markdown
"给这篇笔记添加相关标签"
"找出所有带 #待办 标签的笔记"
```

### 3. Dataview 集成

如果你的库中使用了 Dataview 插件，Claudian 可以帮助编写查询：

```markdown
"帮我写一个 Dataview 查询，列出所有未完成的项目"
```

### 4. 批量操作

```markdown
"将所有 2023 年的日记移动到日记/2023/ 目录"
"重命名这些文件，使用更规范的命名"
```

## 实用示例

### 示例 1：学习笔记整理

> **用户**: 我有一堆散乱的学习笔记，能帮我整理吗？
>
> **Claudian**: 当然！请告诉我这些笔记的位置，我会帮你：
> 1. 分析笔记内容
> 2. 识别主题类别
> 3. 创建合理的文件夹结构
> 4. 移动和重命名文件
> 5. 添加必要的链接和标签

### 示例 2：代码文档生成

> **用户**: 帮我给这个函数添加文档注释
>
> **Claudian**: 我会为你的代码生成清晰的文档注释，包括：
> - 函数功能描述
> - 参数说明
> - 返回值说明
> - 使用示例（如需要）

### 示例 3：研究内容总结

> **用户**: 总结这几篇研究笔记的核心观点
>
> **Claudian**: 我会阅读你指定的笔记，提取关键信息，并生成一份结构化的总结报告。

## 常见问题

### Q: API 配额用完了怎么办？
A: Claudian 会显示剩余配额。你可以在 Anthropic 控制台购买更多配额。

### Q: 支持哪些语言模型？
A: 目前支持 Claude 3 系列模型（Opus、Sonnet、Haiku）。

### Q: 数据会上传到云端吗？
A: 只有发送给 Claude 的内容会通过 API 传输，你的笔记始终保存在本地。

### Q: 可以离线使用吗？
A: 不可以，Claudian 需要连接到 Anthropic API 才能工作。

### Q: 支持多语言吗？
A: 是的，Claude 支持多种语言，包括中文。

## 最佳实践

1. **明确指令**：使用清晰、具体的指令描述你的需求
2. **利用上下文**：提供相关的笔记链接或选中相关文本
3. **分步操作**：复杂任务可以分步骤完成
4. **定期备份**：在进行批量操作前先备份笔记库
5. **验证结果**：检查 Claudian 生成的结果是否符合预期

## 键盘快捷键

| 快捷键 | 功能 |
|--------|------|
| `Ctrl/Cmd + Shift + C` | 打开 Claudian 面板 |
| `Ctrl/Cmd + Shift + E` | 解释选中内容 |
| `Ctrl/Cmd + Shift + R` | 改写选中内容 |

*注：快捷键可在设置中自定义*

## 更新日志

### v1.0.0 (2024-01-15)
- 首次发布
- 基础对话功能
- 笔记读写操作
- Wiki-links 支持

---

## Karpathy 的 LLM Wiki 知识库方法

### 人物介绍

**Andrej Karpathy** 是人工智能领域最具影响力的研究者之一。他曾担任 **OpenAI 的创始成员**和**特斯拉 AI 总监**，领导了特斯拉自动驾驶的视觉识别系统开发。2023 年短暂回归 OpenAI 后，他于 2024 年创办了 AI 教育公司 Eureka Labs。Karpathy 同时是全球最受欢迎的深度学习课程 [CS231n](http://cs231n.stanford.edu/) 的创建者和主讲人，他的 YouTube 频道和博客拥有数百万关注者。

2026 年 4 月，Karpathy 公开分享了他使用 LLM 构建个人知识库的方法——**LLM Wiki**，短短两天内全网千万人观看，迅速成为 AI 知识管理领域最受关注的模式之一。

> 原始文档：[karpathy/llm-wiki (GitHub Gist)](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)

### 核心理念：编译（Compilation）而非检索（Retrieval）

大多数人使用 LLM 处理文档的方式是 **RAG（检索增强生成）**：上传文件 → 查询时检索相关片段 → 生成答案。但这种方式每次提问都要从头重新发现知识，**没有积累**。

Karpathy 提出了一个根本性的不同思路：

> **"使用 LLM 的正确方式不是问答，而是编译。"**
> *"The correct way to use LLMs is not Q&A — it's compilation."*

LLM Wiki 的核心是将 LLM 当作**知识编译器**——它不只是在查询时检索，而是**增量构建并维护一个持久化的 Wiki**：结构化的、相互关联的 Markdown 文件集合。每添加一个新来源，LLM 不只是索引它，而是**读取、提取关键信息、整合到现有 Wiki 中**——更新实体页面、修订主题摘要、标注新数据与旧结论的矛盾。

**关键差异：Wiki 是一个持久化、复利的知识产物。** 交叉引用已经存在，矛盾已被标记，综合分析已反映了你读过的所有内容。每增加一个来源、每提出一个问题，Wiki 都会变得更加丰富。

### 三层架构

| 层次 | 说明 | 谁来维护 |
|------|------|----------|
| **Raw Sources（原始资料）** | 你收集的原始文档——文章、论文、图片、数据文件。**不可变**，LLM 只读不改 | 你 |
| **The Wiki（知识维基）** | LLM 生成的 Markdown 文件集合——摘要、实体页面、概念页面、对比分析、综述。**LLM 完全拥有这一层** | LLM |
| **The Schema（模式说明）** | 配置文件（如 `CLAUDE.md`），告诉 LLM 如何组织 Wiki、遵循什么规则、处理来源时用什么工作流。**你和 LLM 共同演化** | 你 + LLM |

### 快速搭建（保姆级教程）

#### 第一步：安装 Claude Code

Claude Code 是 Anthropic 提供的命令行 AI 工具，是整个知识库系统的 AI 核心。

```bash
# 安装 Claude Code（需要 Node.js 18+）
npm install -g @anthropic-ai/claude-code

# 登录 Anthropic 账号
claude auth login
```

> 你需要一个 Anthropic 账号。访问 [console.anthropic.com](https://console.anthropic.com) 注册并获取 API 密钥。

#### 第二步：创建文件夹结构

在电脑任意位置创建项目文件夹，包含三个子文件夹：

```
my-knowledge-base/
├── raw/          # 原始资料（文章、笔记、截图、PDF 等）
├── wiki/         # AI 编译后的知识维基（由 LLM 维护）
├── outputs/      # AI 生成的答案、报告和分析
└── CLAUDE.md     # 知识库 Schema 说明文件
```

**用命令行创建：**

```bash
mkdir -p my-knowledge-base/{raw,wiki,outputs}
cd my-knowledge-base
```

#### 第三步：创建 CLAUDE.md 配置文件

在项目根目录创建 `CLAUDE.md`，这是告诉 AI 如何管理知识库的"说明书"：

```markdown
# 知识库 Schema

## 这是什么
一个关于 [你的主题] 的个人知识库。

## 如何组织
- raw/ 包含未处理的源材料。永远不要修改这些文件。
- wiki/ 包含整理后的维基。完全由 AI 维护。
- outputs/ 包含生成的报告、答案和分析。

## 维基规则
- 每个主题在 wiki/ 中有自己的 .md 文件
- 每个维基文件以一段摘要开头
- 使用 [[topic-name]] 格式链接相关主题
- 在 wiki/ 中维护一个 INDEX.md，列出每个主题及一行描述
- 当添加新的原始源时，更新相关的维基文章

## 我的兴趣点
[列出 3-5 个你希望这个知识库关注的方向]
```

#### 第四步：把原始素材扔进 raw/

**什么都不用整理**——把文章复制成 `.md` 或 `.txt`、截图直接保存、从其他应用导出笔记、会议记录、论文、书签，统统扔进 `raw/`。整理是 AI 的工作。

**推荐工具：Obsidian Web Clipper**

这是 Karpathy 本人推荐的浏览器扩展，可以将网页文章一键转为 Markdown 保存到 Obsidian：

1. 在 Chrome / Firefox 扩展商店搜索 "Obsidian Web Clipper" 并安装
2. 配置保存路径指向你的 `raw/` 文件夹
3. 浏览网页时点击扩展图标即可剪藏

> **技巧**：在 Obsidian 设置 → 文件与链接 → 设置"附件文件夹路径"为 `raw/assets/`，然后用快捷键（如 `Ctrl+Shift+D`）下载当前文件的所有图片到本地。

#### 第五步：用 Claude Code 编译知识库

在终端中打开你的项目文件夹，启动 Claude Code：

```bash
cd my-knowledge-base
claude
```

然后输入以下指令：

```
读取 raw/ 中的所有内容。然后按照 CLAUDE.md 中的规则在 wiki/ 中编译一个维基。
先创建 INDEX.md，然后为每个主要主题创建一个 .md 文件。
链接相关主题。总结每个源。
```

Claude Code 会自动：
1. 读取 `raw/` 中的所有文件
2. 分析内容和主题
3. 在 `wiki/` 中创建结构化的知识页面
4. 建立交叉引用和 Wiki-links
5. 生成 `INDEX.md` 索引文件

#### 第六步：在 Obsidian 中查看

用 Obsidian 打开 `my-knowledge-base/` 文件夹（或其中的 `wiki/` 子文件夹），你就可以：

- 利用 **图谱视图** 查看知识之间的关联
- 通过 **反向链接** 发现已有的交叉引用
- 浏览结构化的知识页面

Karpathy 的原话是：

> *"我一边开着 LLM Agent，一边开着 Obsidian。LLM 根据我们的对话做出编辑，我实时浏览结果——跟踪链接、检查图谱视图、阅读更新后的页面。Obsidian 是 IDE，LLM 是程序员，Wiki 是代码库。"*

### 三大核心操作

#### 1. Ingest（摄入）

将新资料加入 `raw/` 并让 LLM 处理：

```
"我把一篇新文章放进了 raw/，请阅读并整合到 wiki 中。"
```

LLM 会：
- 阅读新来源并与你讨论要点
- 编写摘要页面
- 更新索引
- 更新相关的实体和概念页面
- 在日志中追加条目

> 单个来源可能会影响 10-15 个 Wiki 页面。

#### 2. Query（查询）

基于 Wiki 提问，而非原始文档：

```
"基于 wiki/ 中的所有内容，我对 [主题] 理解中最大的三个空白是什么？"
"比较源 A 和源 B 对 [概念] 的说法。它们在哪里有分歧？"
"仅使用这个知识库中的内容，给我写一份 500 字的 [主题] 简报。"
```

> **重要洞察**：好的答案可以被归档回 Wiki 作为新页面。你做过的对比、分析、发现的关联——这些是有价值的，不应消失在聊天历史中。这样你的探索会像来源资料一样在知识库中复利增长。

#### 3. Lint（检查）

定期让 LLM 健康检查 Wiki：

```
"审查整个 wiki/ 目录。标记文章之间的矛盾。
找出提到但从未解释的主题。列出任何没有 raw/ 中源支持的声明。
建议 3 篇能填补空白的新文章。"
```

检查内容包括：
- 页面之间的矛盾
- 被新来源取代的过时论断
- 没有入站链接的孤立页面
- 被提及但缺少独立页面的重要概念
- 缺失的交叉引用
- 可通过网络搜索填补的数据空白

### 与 Obsidian Claudian 插件结合

LLM Wiki 方法可以与 Claudian 插件完美结合：

| 功能 | Claudian 插件 | LLM Wiki 方法 |
|------|--------------|--------------|
| 日常笔记对话 | ✅ 直接在 Obsidian 中与 AI 对话 | 通过 Claude Code 在终端操作 |
| 知识编译 | 编辑器内的快速操作 | 批量处理 raw → wiki 的编译流程 |
| Wiki 维护 | 选中内容快速改写/扩展 | Lint 检查、交叉引用、索引更新 |
| 文件操作 | Obsidian 内直接读写 | 终端命令操作 |

**推荐工作流：**

1. 用 **Obsidian Web Clipper** 快速收集网页内容到 `raw/`
2. 用 **Claude Code** 定期编译 `raw/` → `wiki/`
3. 在 **Obsidian** 中浏览和审阅 Wiki 内容
4. 用 **Claudian 插件** 在 Obsidian 内做即时提问和小范围编辑
5. 定期用 Claude Code 运行 **Lint** 健康检查

### 推荐的 Obsidian 配套插件

Karpathy 和社区推荐了以下 Obsidian 插件来增强知识库体验：

| 插件 | 用途 |
|------|------|
| **Obsidian Web Clipper** | 浏览器剪藏网页为 Markdown |
| **Dataview** | 基于 frontmatter 的动态查询和表格 |
| **Templater** | 模板自动化 |
| **Marp** | 从 Wiki 内容生成演示文稿 |
| **Linter** | Markdown 格式规范化 |
| **Notemd** | LLM 自动为笔记关键词生成 Wiki 链接 |

### 应用场景

Karpathy 列举了多种适用场景：

- **个人成长**：追踪目标、健康、心理——整理日记、文章、播客笔记，构建关于自己的结构化图景
- **深度研究**：在数周或数月内深入一个主题——阅读论文、报告，逐步构建带有演化论点的综合 Wiki
- **阅读一本书**：逐章归档，构建角色、主题、情节线索及它们之间的关联页面。读完后你将拥有一份丰富的伴读 Wiki
- **商业/团队**：由 LLM 维护的内部 Wiki，从 Slack 线程、会议记录、项目文档、客户通话中自动更新
- **竞品分析、尽职调查、旅行规划、课程笔记、爱好深潜**——任何需要长期积累和组织知识的地方

### 注意事项

- **AI 幻觉可能固化**：AI 生成的错误信息如果不检查，会在后续编译中被当作事实传播。**定期运行 Lint 检查**至关重要
- **保持简单**：Karpathy 强调"超级简单、完全扁平"——不要过度工具化，避免安装几十个插件把 Obsidian 变成又一个 Notion 陷阱
- **Git 版本控制**：Wiki 本质上就是一个 Markdown 文件的 Git 仓库，你天然拥有版本历史、分支和协作能力

---

## 相关资源

- [Obsidian 官网](https://obsidian.md)
- [Claude API 文档](https://docs.anthropic.com)
- [Claudian GitHub](https://github.com/your-repo/claude-obsidian)
- [反馈与建议](https://github.com/your-repo/claude-obsidian/issues)

### Karpathy LLM Wiki 相关资源

- [Karpathy 原始 Gist：LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- [Karpathy 原始推文 (X/Twitter)](https://x.com/karpathy/status/2039805659525644595)
- [全网爆火的大模型AI知识库保姆级教程 (53AI)](https://www.53ai.com/news/RAG/2026040676831.html)
- [Claude Code 整理 Obsidian 笔记 — Karpathy 公开 LLM 知识库系统 (数位时代)](https://www.bnext.com.tw/article/90530/llm-knowledge-base-obsidian-claude-code)
- [How I Use LLMs — Karpathy 的 YouTube 视频讲解](https://www.youtube.com/watch?v=EWvNQjAaOHw)
- [Karpathy's LLM Wiki: The Complete Guide](https://antigravity.codes/blog/karpathy-llm-wiki-idea-file)

---

**提示**: 如果你在使用过程中遇到任何问题，欢迎在 GitHub 上提 Issue 或加入社区讨论。
