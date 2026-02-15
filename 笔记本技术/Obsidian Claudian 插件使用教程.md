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

## 相关资源

- [Obsidian 官网](https://obsidian.md)
- [Claude API 文档](https://docs.anthropic.com)
- [Claudian GitHub](https://github.com/your-repo/claude-obsidian)
- [反馈与建议](https://github.com/your-repo/claude-obsidian/issues)

---

**提示**: 如果你在使用过程中遇到任何问题，欢迎在 GitHub 上提 Issue 或加入社区讨论。
