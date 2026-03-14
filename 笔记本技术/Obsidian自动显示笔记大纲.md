---
title: Obsidian自动显示笔记大纲
created: 2026-03-14
source: Cherry Studio
tags: 
---
# Obsidian自动显示笔记大纲

## 🧑‍💻 User

如何让obsitian 自动显示笔记大纲

---
## 🤖 Assistant

我来帮您搜索关于 Obsidian 自动显示笔记大纲的方法。




根据搜索结果，我来为您详细介绍如何让 Obsidian 自动显示笔记大纲。

## 使用内置的 Outline（大纲）插件

Obsidian 自带核心大纲插件，这是最直接的解决方案：

### 启用步骤：
1. 打开 **Settings（设置）** → **Core plugins（核心插件）**
2. 找到 **Outline** 并 **启用** 该插件 [^3]
3. 启用后，笔记右侧边栏会自动显示当前笔记的大纲

### 使用方式：
- 大纲会根据您的标题层级（##, ### 等）自动生成 [^3]
- 点击大纲中的标题可直接跳转到对应位置 [^4]
- 您可以设置显示哪些级别的标题 [^4]

## 设置自动显示大纲

### 方法一：固定在边栏
1. 打开大纲视图后，将其 **拖拽固定** 到侧边栏
2. 这样每次打开笔记时大纲会自动显示

### 方法二：使用命令面板
- 按 `Ctrl/Cmd + P` 打开命令面板
- 输入 `Outline: Open Outline` 即可打开大纲视图 [^4]

## 推荐的增强插件

### Daily Note Outline 插件
如果您需要查看**多个每日笔记**的大纲，推荐安装此插件 [^4]：

**插件功能：**
- 同时显示多个每日笔记的大纲 [^4]
- 支持标题、链接、标签和列表项的显示 [^4]
- 可设置显示日期范围（向前/向后搜索）[^4]
- 支持过滤、包含和排除功能 [^4]

**安装步骤：**
1. 前往 **Settings** → **Community plugins**
2. 搜索 "Daily Note Outline" 并安装
3. 启用插件并在命令面板中选择 "Daily Note Outline: Open Outline" [^4]

## 快速设置建议

对于自动显示大纲，我建议：

1. ✅ 启用 Outline 核心插件 [^3]
2. ✅ 将大纲视图固定到常用侧边栏
3. ✅ 根据需要调整要显示的标题级别 [^4]
4. ✅ 如需每日笔记功能，安装 Daily Note Outline 插件 [^4]

这样设置后，您的 Obsidian 笔记就能自动显示大纲了，大幅提升阅读和导航效率！
[^1]: [Obsidian显示大纲的简单操作](https://blog.csdn.net/L_egend_ing/article/details/146507276)

[^2]: [V2528期周刊：知识图谱和Obsidian脚本优化实现自动标题管理](https://forum-zh.obsidian.md/t/topic/52516)

[^3]: [【Obsidian 使用教學】基礎篇 02 — 認識「設定選單」與我的設定建議](https://medium.com/pm%E7%9A%84%E7%94%9F%E7%94%A2%E5%8A%9B%E5%B7%A5%E5%85%B7%E7%AE%B1/obsidian-%E4%BD%BF%E7%94%A8%E6%95%99%E5%AD%B8-%E5%9F%BA%E7%A4%8E%E7%AF%87-02-%E8%AA%8D%E8%AD%98-%E8%A8%AD%E5%AE%9A%E9%81%B8%E5%96%AE-%E8%88%87%E6%88%91%E7%9A%84%E5%BB%BA%E8%AD%B0%E8%A8%AD%E5%AE%9A-5371943f60ec)

[^4]: [Obsidian 插件：Daily Note Outline-Pkmer](https://pkmer.cn/Pkmer-Docs/10-obsidian/obsidian%E7%A4%BE%E5%8C%BA%E6%8F%92%E4%BB%B6/readme/obsidian-daily-note-outline_readme/)
---
## 🧑‍💻 User

能不能自动显示大纲

---
## 🤖 Assistant

我来搜索一下 Obsidian 自动显示大纲的具体设置方法。






让我再搜索更具体的自动显示设置。





根据搜索结果，我找到了您需要的解决方案！

## 自动显示大纲的两种方法

### 方法一：固定 Outline 面板到侧边栏

最简单的方法是将大纲面板固定在侧边栏 [^1]：

1. **启用 Outline 插件**：Settings → Core Plugins → 启用 Outline
2. **固定到侧边栏**：点击大纲面板右上角的 **📌 图标**，将其固定
3. 这样每次打开笔记，大纲会自动显示在侧边栏

### 方法二：使用 Floating TOC 插件（推荐）

如果您想要**在编辑器内部自动显示大纲**，推荐安装 **Floating TOC 插件** [^1]：

**插件功能：**
- 在编辑器窗格中添加一个**浮动的目录** [^1]
- 大纲**始终可见**，自动根据标题层级生成 [^1]
- 点击标题可直接跳转到对应位置 [^1]
- 支持设置显示完整标题（"Default Pin"选项）[^1]

**安装步骤：**
1. 前往 Settings → Community plugins
2. 搜索 "floating-toc" 并安装
3. 启用插件，编辑器右侧会自动显示浮动大纲

## 快速对比

| 方法 | 位置 | 适用场景 |
|------|------|----------|
| 固定 Outline | 侧边栏 | 需要干净编辑界面 |
| Floating TOC | 编辑器内 | 需要实时查看结构 |

**建议**：如果您希望"自动且持续"地显示大纲，安装 **Floating TOC 插件** 是最佳选择，它会直接在编辑区域显示目录，完全满足您的需求 [^1]。

需要我帮您查找更多关于这个插件的设置细节吗？
[^1]: [Obsidian Quick Tip: Outline Pane for Quick Navigation of Long Documents](https://medium.com/obsidian-observer/obsidian-quick-tip-outline-pane-for-quick-navigation-of-long-documents-74cdabbbfb96)