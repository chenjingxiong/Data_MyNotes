---
title: PPT Master 使用指南
source: https://github.com/hugohe3/ppt-master
author: Hugo He
stars: 10600+
license: MIT
tags:
  - AI
  - PPT
  - Claude-Code
  - Office
  - python-pptx
  - SVG
date: 2026-05-02
---

# PPT Master 使用指南

> AI 生成原生可编辑 PPTX —— 投入 PDF、DOCX、URL 或 Markdown，产出可在 PowerPoint 中逐元素点击编辑的演示文稿。

## 项目简介

**PPT Master** 是由 [Hugo He](https://www.hehugo.com) 开发的开源项目（MIT 协议），核心理念：**如果一个文件无法在 PowerPoint 中打开并编辑，那它就不应该被称为 PPT。**

它不是一个独立应用，而是一套**工作流（Skill）**，运行在 Claude Code、Cursor、VS Code Copilot 等 AI 编程工具中。你只需用自然语言说"帮我把这个 PDF 做成 PPT"，AI 就会按照工作流自动生成一个原生可编辑的 `.pptx` 文件。

### 与其他 AI PPT 工具的区别

| 类别 | 输出方式 | 是否逐元素可编辑 |
|------|----------|:---:|
| 模板填充 | 基于固定模板的 PPTX | ⚠️ 受模板限制 |
| 图片式 | 每页一张大图打包成 PPTX | ❌ 每页都是图片 |
| HTML 演示 | Web 页面 | ❌ 不是 PPTX |
| **原生可编辑（PPT Master）** | **真实 DrawingML 形状、文本框、图表** | **✅ 点击任意元素即可编辑** |

## 核心特性

- **原生 PowerPoint**：输出的每一个元素（文本框、形状、图表）都可直接在 PowerPoint 中点击编辑
- **动画与转场**：支持页面转场和逐元素入场动画，以真实 OOXML 格式嵌入，非视频
- **语音旁白与视频导出**：可生成逐页语音旁白（默认 edge-tts，可选云 TTS），嵌入 PPTX 后用 PowerPoint 导出为 MP4 视频
- **声音克隆**：支持接入 ElevenLabs / MiniMax / Qwen / CosyVoice 的克隆语音，用你的声音为整份 PPT 旁白
- **数据本地化**：除 AI 模型通信外，整个流程在你的机器上运行，文件无需上传到第三方服务器
- **无平台绑定**：支持 Claude Code、Cursor、VS Code Copilot 等多种 IDE，支持 Claude、GPT、Gemini、Kimi 等多种模型

## 快速开始

### 1. 环境准备

**唯一前置依赖：Python 3.10+**

#### Windows 用户（重要）

Windows 需要额外配置（PATH 设置、执行策略等），项目提供了专门的分步指南：
- 下载 Python → 安装时勾选 **Add to PATH** → `pip install -r requirements.txt` → 完成

> [!tip] Windows 专用指南
> 详见项目 [Windows Installation Guide](https://github.com/hugohe3/ppt-master/blob/main/docs/windows-installation.md)，从零到生成第一份 PPT 约 10 分钟。

#### macOS / Linux

直接安装即可，无额外步骤。

### 2. 选择 AI Agent

PPT Master 支持任何具备 Agent 能力的工具（能读写文件、执行命令、多轮对话）：

| 类型 | 示例 |
|------|------|
| IDE 内置 Agent | Cursor、Windsurf、Trae、Codebuddy IDE、Zed |
| IDE 插件/扩展 | GitHub Copilot、Claude Code (VS Code/JetBrains)、Cline、Continue |
| CLI Agent | Claude Code CLI、Codex CLI、Aider、Gemini CLI |

> [!note] 模型推荐
> **Claude Opus / Sonnet** 效果最佳，测试最充分。GPT、Gemini、Kimi 等也可用，但 SVG 绝对坐标布局精度可能有差异。

### 3. 安装

**方式 A —— 下载 ZIP（无需 Git）：**
在 GitHub 页面点击 `Code` → `Download ZIP`，解压即可。

**方式 B —— Git 克隆：**

```bash
git clone https://github.com/hugohe3/ppt-master.git
cd ppt-master
```

**安装依赖：**

```bash
pip install -r requirements.txt
```

后续更新（仅方式 B）：

```bash
python3 skills/ppt-master/scripts/update_repo.py
```

### 4. 创建 PPT

有两种方式提供内容：

**方式一：提供源文件（推荐）**

将 PDF、DOCX、图片等文件放入 `projects/` 目录，然后在 AI 聊天面板中告诉它使用哪个文件：

```
请根据 projects/q3-report/sources/report.pdf 创建一份 PPT
```

**方式二：直接粘贴内容**

将文本内容直接粘贴到聊天窗口：

```
请将以下内容转为 PPT：[粘贴你的内容...]
```

AI 会先确认设计规格：

```
AI: 好的，让我们确认一下设计规格：
    [模板] B) 自由设计
    [格式]   PPT 16:9
    [页数]    8-10 页
    ...
```

然后自动完成内容分析、视觉设计、SVG 生成和 PPTX 导出。

### 5. 输出说明

- **主要文件**：`exports/<name>_<timestamp>.pptx`（原生形状，可直接编辑）
- **SVG 快照**：`_svg.pptx` 和 `svg_output/` 目录备份到 `backup/<timestamp>/`
- **兼容性**：需要 Office 2016+

> [!warning] AI 丢失上下文？
> 让它重新读取 `skills/ppt-master/SKILL.md` 即可恢复工作流。

## 图片获取（可选）

PPT Master 提供两条图片路径，可在同一份 PPT 中混用：

### A) AI 生成图片

使用 `image_gen.py` 脚本：

1. 复制 `.env.example` 为 `.env`
2. 设置 `IMAGE_BACKEND` 和对应的 API Key（如 `OPENAI_API_KEY`、`GEMINI_API_KEY`）
3. 流水线会自动调用

```bash
# 查看所有可用后端
python3 skills/ppt-master/scripts/image_gen.py --list-backends
```

当前推荐默认：**gpt-image-2**

### B) 网络图片搜索

使用 `image_search.py` 脚本：

- **零配置可用**，但图片质量一般（使用 Openverse / Wikimedia Commons）
- **推荐配置** `PEXELS_API_KEY` / `PIXABAY_API_KEY`（两者均免费），显著提升图片质量
- 默认优先高质量授权图片：CC0、Public Domain、Pexels/Pixabay 无需署名等

**图片质量优先级**：用户自有高清素材 / AI 生成 > 带 Pexels/Pixabay Key 的网络搜索 > 零配置网络搜索

## 支持的画布格式

除标准 PPT 16:9 外，还支持：

- 小红书（多种尺寸）
- 微信公众号封面
- 以及 10+ 其他格式

详见项目文档 [Canvas Formats](https://github.com/hugohe3/ppt-master/blob/main/docs/canvas-formats.md)。

## 内置风格模板

| 风格 | 描述 |
|------|------|
| Magazine | 温暖大地色调，照片丰富布局 |
| Academic | 结构化研究格式，数据驱动 |
| Dark Art | 电影感暗色背景，画廊美学 |
| Nature Documentary | 沉浸式摄影，极简 UI |
| Tech / SaaS | 干净白色卡片，定价表布局 |
| Product Launch | 高对比度，大胆规格亮点 |

还可以创建自定义品牌模板，详见 [Create a Custom Template](https://github.com/hugohe3/ppt-master/blob/main/docs/custom-template.md)。

## 技术架构

PPT Master 的核心思路：

1. **AI 分析内容** → 提取结构化信息
2. **SVG 中间格式** → AI 生成 SVG 布局（绝对坐标定位）
3. **SVG → PPTX 转换** → 通过 `python-pptx` 将 SVG 元素映射为原生 PowerPoint 形状
4. **输出原生 .pptx** → 每个元素都是真实的 DrawingML 对象

这种 SVG 中间层的设计使得布局精度不受 HTML/CSS 渲染差异影响。

## 目录结构

```
ppt-master/
├── skills/ppt-master/     # 核心 Skill 定义与脚本
│   ├── SKILL.md           # 核心工作流与规则
│   └── scripts/           # 工具脚本（图片生成/搜索等）
├── projects/              # 放置你的源文件
├── exports/               # 生成的 PPT 输出
├── backup/                # SVG 快照备份
├── examples/              # 22 个示例项目，共 309 页
├── docs/                  # 文档
├── requirements.txt       # Python 依赖
└── CLAUDE.md              # Claude Code 专用指令
```

## 常见问题

### 推荐用什么模型？

Claude Opus / Sonnet 效果最好。其他模型可用但 SVG 布局精度可能参差不齐。

### 费用如何？

工具本身免费开源，唯一的成本是你的 AI 模型使用费用（按量付费）。

### 生成的 PPT 布局有问题？

参考项目 [FAQ](https://github.com/hugohe3/ppt-master/blob/main/docs/faq.md)，涵盖模型选择、布局问题、导出问题等。

### 如何让 AI 重新加载工作流？

在对话中输入：`请读取 skills/ppt-master/SKILL.md`

## 相关链接

- **GitHub 仓库**：[hugohe3/ppt-master](https://github.com/hugohe3/ppt-master)
- **在线演示**：[hugohe3.github.io/ppt-master](https://hugohe3.github.io/ppt-master/)
- **作者网站**：[www.hehugo.com](https://www.hehugo.com)
- **AtomGit 镜像**：[AtomGit](https://atomgit.com)
- **Windows 安装指南**：[Windows Installation](https://github.com/hugohe3/ppt-master/blob/main/docs/windows-installation.md)
- **FAQ**：[常见问题](https://github.com/hugohe3/ppt-master/blob/main/docs/faq.md)
- **技术设计**：[Technical Design](https://github.com/hugohe3/ppt-master/blob/main/docs/technical-design.md)

---

*最后更新：2026-05-02 | 版本 v2.5.0 | ⭐ 10.6k Stars*
