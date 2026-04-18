# Claude Code Web-UI 详细指南

> **更新日期**：2026-04-13
> **适用版本**：Claude Code 最新版
> **官方文档**：https://docs.anthropic.com/en/docs/claude-code/overview

---

## 目录

- [什么是 Claude Code Web-UI](#什么是-claude-code-web-ui)
- [Web-UI vs CLI vs IDE 对比](#web-ui-vs-cli-vs-ide-对比)
- [安装与启动](#安装与启动)
- [界面总览](#界面总览)
- [核心功能详解](#核心功能详解)
- [MCP 工具集成](#mcp-工具集成)
- [会话管理](#会话管理)
- [权限与安全设置](#权限与安全设置)
- [配置选项](#配置选项)
- [高级用法](#高级用法)
- [常用快捷键](#常用快捷键)
- [最佳实践](#最佳实践)
- [故障排查](#故障排查)
- [相关资源](#相关资源)

---

## 什么是 Claude Code Web-UI

Claude Code Web-UI 是 Anthropic 为 **Claude Code** 提供的**浏览器端用户界面**，让你无需打开终端即可在网页上直接与 Claude Code 交互。它是 Claude Code 多种使用方式中最为直观和便捷的一种。

### 核心特点

| 特性 | 说明 |
|------|------|
| **零终端依赖** | 无需打开 Terminal / CMD，浏览器直接使用 |
| **可视化操作** | 代码变更高亮显示，文件树直观浏览 |
| **实时预览** | Diff 视图查看代码修改，支持一键接受/拒绝 |
| **MCP 集成** | 在 Web-UI 中直接调用 MCP 工具 |
| **会话持久化** | 关闭浏览器后可恢复之前的会话 |
| **多项目管理** | 可同时打开多个项目的工作区 |

### 工作原理

```
┌─────────────┐     WebSocket      ┌──────────────────┐     API      ┌─────────────┐
│   浏览器     │ ◄─────────────────► │  Claude Code 服务  │ ◄──────────► │ Claude API  │
│  (Web-UI)   │                     │  (本地/远程)       │              │ (Anthropic)  │
└─────────────┘                     └──────────────────┘              └─────────────┘
       │                                    │
       │                                    │
       ▼                                    ▼
  文件系统访问                          MCP 工具调用
  (项目代码)                           (浏览器、数据库等)
```

---

## Web-UI vs CLI vs IDE 对比

| 特性 | **Web-UI** | **CLI 终端** | **VS Code 扩展** | **JetBrains 插件** |
|------|-----------|-------------|------------------|-------------------|
| **上手难度** | ⭐ 最低 | ⭐⭐⭐ 需命令行基础 | ⭐⭐ 需安装扩展 | ⭐⭐ 需安装插件 |
| **代码 Diff 预览** | ✅ 可视化 Diff | ❌ 纯文本展示 | ✅ 编辑器内集成 | ✅ 编辑器内集成 |
| **文件浏览** | ✅ 侧边栏文件树 | ❌ 需手动指定路径 | ✅ 原生文件树 | ✅ 原生文件树 |
| **终端访问** | ✅ 内置终端 | ✅ 原生终端 | ✅ 集成终端 | ✅ 集成终端 |
| **MCP 支持** | ✅ 完整支持 | ✅ 完整支持 | ✅ 完整支持 | ✅ 完整支持 |
| **快捷键操作** | ⚠️ 有限 | ✅ 完整快捷键 | ✅ 完整快捷键 | ✅ 完整快捷键 |
| **离线能力** | ❌ 需要网络 | ⚠️ 需要网络 | ⚠️ 需要网络 | ⚠️ 需要网络 |
| **资源占用** | 🟢 低（浏览器标签） | 🟢 最低 | 🟡 中等 | 🟡 中等 |
| **多窗口** | ✅ 浏览器标签页 | ⚠️ 需 tmux | ✅ VS Code 窗口 | ✅ IDE 窗口 |
| **适合人群** | 所有人 | 开发者/高级用户 | VS Code 用户 | JetBrains 用户 |

### 选择建议

- **初次使用 / 快速体验** → Web-UI
- **日常开发 / 重度使用** → VS Code 扩展 或 JetBrains 插件
- **自动化 / CI/CD / 脚本** → CLI 终端
- **远程服务器开发** → CLI 终端（SSH） 或 Web-UI

---

## 安装与启动

### 前提条件

1. **Anthropic 账户**：需要已注册的 Anthropic 账户
2. **API 订阅**：需要有效的 Claude API 访问权限（Max 订阅或 API Key）
3. **现代浏览器**：Chrome 90+、Firefox 90+、Edge 90+、Safari 15+

### 方式一：直接访问网页（最简单）

1. 打开浏览器，访问 **https://claude.ai/code**
2. 使用 Anthropic 账户登录
3. 选择或上传项目目录
4. 开始对话

> 💡 这是最快捷的方式，无需安装任何软件。

### 方式二：从 CLI 启动 Web-UI

如果你已安装 Claude Code CLI，可以通过命令直接启动 Web-UI：

```bash
# 启动 Web-UI（自动打开浏览器）
claude --web

# 指定端口启动
claude --web --port 8080

# 在当前项目中启动并自动加载上下文
cd /your/project && claude --web
```

### 方式三：通过 VS Code 启动

1. 安装 **Claude Code** VS Code 扩展
2. 打开命令面板（`Ctrl+Shift+P` / `Cmd+Shift+P`）
3. 搜索 "Claude Code: Open Web UI"
4. 选择后在侧边栏或新标签页中打开

### 方式四：通过 JetBrains 启动

1. 安装 **Claude Code** JetBrains 插件
2. 在工具栏中点击 Claude Code 图标
3. 选择 "Open in Web UI"

### Claude Code CLI 安装

如果尚未安装 Claude Code CLI：

```bash
# macOS / Linux
curl -fsSL https://claude.ai/install.sh | bash

# macOS (Homebrew)
brew install --cask claude-code

# Windows (PowerShell)
irm https://claude.ai/install.ps1 | iex

# Windows (WinGet)
winget install Anthropic.ClaudeCode
```

> ⚠️ npm 安装方式已废弃（`npm install -g @anthropic-ai/claude-code`），建议使用上述官方安装方式。

---

## 界面总览

```
┌──────────────────────────────────────────────────────────────────┐
│  Claude Code                              [模型选择] [设置 ⚙️]    │
├──────────┬───────────────────────────────────────────────────────┤
│          │                                                       │
│  📁 文件树  │  💬 对话区域                                          │
│          │                                                       │
│  ▸ src/   │   你：帮我优化这个函数的性能                              │
│  ▸ tests/ │                                                       │
│  ▸ docs/  │   Claude：我来分析一下这段代码...                         │
│    📄     │   ┌─────────────────────────────────────────┐        │
│  .gitign  │   │  📝 文件编辑建议                            │        │
│  README   │   │  src/utils/helper.ts                     │        │
│  package  │   │  - const result = data.filter(...)       │        │
│          │   │  + const result = data.reduce(...)        │        │
│          │   │                                           │        │
│          │   │  [✅ 接受]  [❌ 拒绝]  [📝 修改]             │        │
│  🔧 MCP   │   └─────────────────────────────────────────┘        │
│  工具面板  │                                                       │
│          │  ┌──────────────────────────────────────────────┐     │
│          │  │  ⌨️ 输入消息...                    [发送 ➤]   │     │
│          │  └──────────────────────────────────────────────┘     │
├──────────┴───────────────────────────────────────────────────────┤
│  🖥️ 内置终端                                                      │
│  $ npm test                                                       │
│  ✓ All tests passed                                               │
└──────────────────────────────────────────────────────────────────┘
```

### 界面区域说明

| 区域 | 功能 | 操作方式 |
|------|------|---------|
| **顶部栏** | 模型选择、全局设置、账户信息 | 点击切换模型、打开设置面板 |
| **文件树** | 浏览项目文件结构 | 点击展开/折叠、双击打开文件 |
| **对话区** | 与 Claude 的主交互区域 | 输入消息、查看回复、审核代码变更 |
| **代码 Diff** | 可视化展示代码修改建议 | 接受/拒绝/编辑修改 |
| **MCP 工具面板** | 管理和调用 MCP 工具 | 点击工具名称查看详情 |
| **内置终端** | 执行命令行操作 | 输入命令、查看输出 |
| **输入框** | 输入消息和指令 | 文本输入、粘贴图片、拖拽文件 |

---

## 核心功能详解

### 1. 智能对话与代码生成

Web-UI 支持自然语言对话来驱动代码操作：

**基本用法**：
```
你：在 src/components/ 下创建一个新的 Button 组件，支持 primary、secondary、danger 三种样式

Claude 会：
1. 分析项目结构，了解技术栈
2. 生成符合项目风格的组件代码
3. 在 Diff 视图中展示建议
4. 等待你审核并接受
```

**上下文感知**：
- 自动读取项目配置文件（`package.json`、`tsconfig.json` 等）
- 识别项目使用的技术栈和框架
- 遵循项目的代码风格和约定

### 2. 代码 Diff 可视化审核

这是 Web-UI 相比 CLI 最大的优势之一：

**审核流程**：
1. Claude 提出代码修改建议
2. Diff 视图展示删除行（🔴 红色）和新增行（🟢 绿色）
3. 你可以逐个文件审核
4. 对每个文件选择：
   - **✅ 接受**：应用此修改
   - **❌ 拒绝**：放弃此修改
   - **📝 编辑**：在 Diff 基础上进一步修改
   - **💬 反馈**：告诉 Claude 修改方向

**Diff 视图操作**：
| 操作 | 说明 |
|------|------|
| 逐行查看 | 滚动查看每一处变更 |
| 文件切换 | 在多文件修改间切换 |
| 行内编辑 | 直接在 Diff 中修改代码 |
| 全部接受 | 一键接受所有变更 |
| 全部拒绝 | 一键拒绝所有变更 |

### 3. 文件管理

- **浏览**：左侧文件树展示完整项目结构
- **创建**：通过对话让 Claude 创建新文件
- **编辑**：通过 Diff 视图审核修改
- **删除**：需要确认后才执行删除操作
- **搜索**：全局搜索文件内容

### 4. 内置终端

Web-UI 内置了终端模拟器，可以直接在浏览器中执行命令：

```bash
# 安装依赖
npm install

# 运行测试
npm test

# 启动开发服务器
npm run dev

# Git 操作
git status
git diff
git log --oneline -5
```

终端操作特点：
- 支持常用 shell 命令
- Claude 可自动执行命令（需权限允许）
- 输出结果自动作为上下文反馈给 Claude
- 危险操作需要用户确认

### 5. 图片与多模态输入

Web-UI 支持在对话中粘贴或拖入图片：

- **截图粘贴**：直接 `Ctrl+V` 粘贴截图
- **拖拽上传**：拖拽图片文件到输入框
- **支持的格式**：PNG、JPG、GIF、WebP
- **使用场景**：
  - 粘贴 UI 截图让 Claude 还原代码
  - 粘贴错误截图让 Claude 诊断问题
  - 粘贴设计稿让 Claude 生成页面

### 6. @ 引用文件

在输入框中使用 `@` 符号引用项目中的文件：

```
请查看 @src/utils/auth.ts 中的登录逻辑，修复 token 过期的问题
```

引用方式：
| 语法 | 说明 |
|------|------|
| `@文件名` | 引用指定文件作为上下文 |
| `@目录名/` | 引用整个目录的相关文件 |
| `@文件名:行号` | 引用文件的特定行范围 |

---

## MCP 工具集成

Web-UI 完整支持 MCP（Model Context Protocol）工具，可以在浏览器中直接调用外部工具。

### 配置 MCP 服务器

在设置中配置 MCP 服务器（与 CLI 共享配置）：

**方式一：通过 Web-UI 设置界面**
1. 点击右上角 ⚙️ 设置
2. 选择 "MCP Servers" 标签
3. 添加或编辑 MCP 服务器配置

**方式二：通过配置文件**

编辑 `~/.claude/settings.json`（全局）或项目下 `.claude/settings.json`：

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["@anthropic-ai/chrome-devtools-mcp@latest"],
      "type": "stdio"
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/filesystem-mcp", "/path/to/allowed/dir"],
      "type": "stdio"
    }
  }
}
```

### Web-UI 中使用 MCP

配置完成后，Claude 会在需要时自动调用 MCP 工具：

```
你：帮我在浏览器中打开 localhost:3000 并截图

Claude：[自动调用 Chrome DevTools MCP]
  → navigate_page("http://localhost:3000")
  → take_screenshot()
  → 返回截图并分析页面
```

---

## 会话管理

### 会话持久化

Web-UI 的会话会自动保存：

- **自动保存**：对话内容自动持久化
- **恢复会话**：重新打开后可继续之前的对话
- **会话列表**：在侧边栏查看历史会话

### 会话操作

| 操作 | 方法 |
|------|------|
| 新建会话 | 点击 "New Chat" 或 `Ctrl+Shift+N` |
| 切换会话 | 在会话列表中点击 |
| 重命名会话 | 右键会话 → 重命名 |
| 导出会话 | 右键会话 → 导出为 Markdown |
| 删除会话 | 右键会话 → 删除 |

### 上下文管理

Web-UI 提供了上下文管理功能，帮助你控制发送给 Claude 的信息量：

- **压缩上下文**：当对话过长时，可压缩历史消息
- **清除上下文**：开始新的话题时清除不相关历史
- **固定文件**：将重要文件固定在上下文中，不会被压缩掉

---

## 权限与安全设置

### 权限级别

Web-UI 提供了精细的权限控制：

| 权限类别 | 说明 | 默认设置 |
|----------|------|---------|
| **文件读取** | 读取项目文件 | ✅ 允许 |
| **文件写入** | 修改/创建文件 | ⚠️ 需确认 |
| **文件删除** | 删除文件 | ❌ 需确认 |
| **命令执行** | 运行 shell 命令 | ⚠️ 需确认 |
| **网络请求** | 发起外部 HTTP 请求 | ⚠️ 需确认 |
| **MCP 调用** | 调用 MCP 工具 | ⚠️ 需确认 |

### 权限配置

**通过设置界面配置**：
1. 打开设置 → "Permissions"
2. 对每种操作设置权限级别：
   - **Allow**：自动允许，不询问
   - **Ask**：每次操作前询问
   - **Deny**：禁止执行

**通过配置文件**（`.claude/settings.json`）：

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Glob",
      "Grep"
    ],
    "deny": [
      "Bash(rm -rf *)"
    ]
  }
}
```

### 安全最佳实践

1. **不要允许自动执行危险命令**（如 `rm -rf`、`DROP TABLE`）
2. **定期审查 MCP 工具权限**
3. **使用 `.claudeignore` 排除敏感文件**

创建 `.claudeignore` 文件（类似 `.gitignore`）：

```
.env
.env.local
*.secret
credentials/
**/private/**
```

---

## 配置选项

### 模型选择

Web-UI 支持在多种 Claude 模型之间切换：

| 模型 | 特点 | 适用场景 |
|------|------|---------|
| **Claude Opus** | 最强推理能力 | 复杂架构设计、困难 Bug |
| **Claude Sonnet** | 均衡性能与速度 | 日常开发、代码生成 |
| **Claude Haiku** | 最快响应速度 | 简单问题、快速编辑 |

切换方式：点击顶部栏的模型选择器 → 选择模型

### 全局设置

通过设置界面或配置文件调整：

```json
// ~/.claude/settings.json
{
  "model": "claude-sonnet-4-20250514",
  "theme": "dark",
  "fontSize": 14,
  "wordWrap": true,
  "autoSave": true,
  "maxTokens": 8192,
  "mcpServers": {},
  "permissions": {}
}
```

### 项目级设置

在项目根目录创建 `.claude/settings.json`：

```json
{
  "model": "claude-sonnet-4-20250514",
  "permissions": {
    "allow": ["Read", "Glob", "Grep", "Bash(npm test)"]
  },
  "context": {
    "include": ["src/**", "tests/**"],
    "exclude": ["node_modules/**", "dist/**"]
  }
}
```

---

## 高级用法

### 1. 使用 CLAUDE.md 项目指令

在项目根目录创建 `CLAUDE.md` 文件，为 Claude 提供项目特定的指令：

```markdown
# Project Guidelines

## Tech Stack
- Frontend: React 18 + TypeScript
- Styling: Tailwind CSS
- State: Zustand
- Testing: Vitest + Playwright

## Code Style
- Use functional components only
- Prefer const over let
- Use named exports
- Follow existing patterns in the codebase

## Important Rules
- Never modify files in `src/generated/`
- Always run tests after making changes
- Use conventional commits format
```

Web-UI 会在每次会话开始时自动读取此文件。

### 2. 多文件批量操作

```
你：将所有组件中的 class 组件转换为函数组件，并更新对应的测试文件

Claude 会：
1. 扫描所有匹配的文件
2. 逐个生成转换方案
3. 在 Diff 视图中展示所有变更
4. 你可以逐个审核或批量接受
```

### 3. 与 Git 集成

Web-UI 深度集成了 Git 操作：

```
你：查看最近的提交记录，创建一个新分支修复登录 Bug

Claude 会：
1. git log 查看提交历史
2. git checkout -b fix/login-bug
3. 分析和修复代码
4. git add + git commit
```

### 4. 结合 Hooks 自动化

通过 Claude Code 的 Hooks 功能，可以在特定事件时自动执行操作：

```json
// .claude/settings.json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "command": "npx prettier --write $FILE"
      }
    ]
  }
}
```

### 5. 远程项目连接

Web-UI 可以连接远程服务器上的项目：

```
# 通过 SSH 连接远程项目
claude --web --remote ssh://user@server/path/to/project

# 通过 GitHub 仓库连接
claude --web --remote https://github.com/user/repo
```

---

## 常用快捷键

| 快捷键 | 功能 |
|--------|------|
| `Enter` | 发送消息（在输入框中） |
| `Shift + Enter` | 换行（不发送） |
| `Ctrl + Shift + N` | 新建会话 |
| `Ctrl + /` | 切换侧边栏 |
| `Ctrl + Shift + P` | 命令面板 |
| `Ctrl + Z` | 撤销输入 |
| `Ctrl + Shift + Z` | 重做输入 |
| `Ctrl + B` | 切换文件树 |
| `Ctrl + `` ` | 切换终端 |
| `Escape` | 取消当前操作 / 停止生成 |
| `Ctrl + Shift + C` | 复制选中内容 |
| `Ctrl + L` | 清除对话 |

> 💡 Mac 用户将 `Ctrl` 替换为 `Cmd`。

---

## 最佳实践

### 提示词技巧

1. **明确目标**：清楚说明你要做什么
   ```
   ✅ 好的提示：在 src/api/auth.ts 中添加 JWT token 自动刷新逻辑，当 token 即将在 5 分钟内过期时自动调用 /refresh 接口

   ❌ 差的提示：修一下认证
   ```

2. **提供上下文**：引用相关文件
   ```
   参考 @src/types/user.ts 中的 User 类型定义，在 @src/pages/Profile.tsx 中创建用户资料编辑页面
   ```

3. **分步操作**：复杂任务拆分为小步骤
   ```
   第一步：先创建数据库迁移文件
   第二步：更新 API 接口
   第三步：修改前端调用
   ```

4. **指定约束**：告诉 Claude 不要做什么
   ```
   重构这个模块，但不要改变任何公开的 API 接口签名
   ```

### 性能优化建议

1. **控制上下文大小**：定期压缩或清除不相关的对话历史
2. **使用 Haiku 处理简单任务**：简单问题用 Haiku 模型，节省 Token
3. **善用 @ 引用**：精准引用文件，避免加载整个项目
4. **分批处理**：大量文件修改分批进行，避免单次操作过多

### 协作技巧

1. **代码审核**：让 Claude 解释它的修改，确保你理解每一处变更
2. **渐进式开发**：先让 Claude 分析和规划，确认后再执行
3. **测试驱动**：先让 Claude 写测试，再写实现代码
4. **保留 CLAUDE.md**：维护好项目指令文件，保持一致性

---

## 故障排查

### 常见问题

#### 1. Web-UI 无法加载

| 症状 | 可能原因 | 解决方案 |
|------|---------|---------|
| 页面空白 | 浏览器缓存问题 | 清除缓存后重试，或使用无痕模式 |
| 连接超时 | 网络问题 | 检查网络连接，确认可访问 claude.ai |
| 登录失败 | 认证过期 | 重新登录 Anthropic 账户 |
| 端口冲突 | `--web` 端口被占用 | 使用 `--port` 指定其他端口 |

#### 2. 代码修改未生效

```
问题：点击"接受"后文件没有变化
排查：
1. 检查文件是否被其他进程锁定
2. 确认文件写入权限
3. 刷新页面后检查文件内容
```

#### 3. MCP 工具无法调用

```
问题：配置了 MCP 但工具未显示
排查：
1. 检查 ~/.claude/settings.json 中的 MCP 配置
2. 确认 MCP 服务器进程可以正常启动
3. 在终端中手动运行 MCP 命令测试
4. 重启 Claude Code 后重试
```

#### 4. 对话响应缓慢

```
问题：Claude 回复很慢
排查：
1. 检查网络延迟（尤其在中国大陆需要代理）
2. 尝试切换到更快的模型（Haiku）
3. 压缩上下文，减少 Token 消耗
4. 检查是否有大量文件被意外加载
```

#### 5. 终端命令执行失败

```
问题：内置终端无法执行命令
排查：
1. 确认命令格式正确
2. 检查是否需要管理员权限
3. 确认工作目录正确
4. 查看终端输出中的错误信息
```

### 重置 Web-UI

如果 Web-UI 出现异常，可以尝试重置：

1. **清除浏览器缓存**：`Ctrl+Shift+Delete`
2. **清除本地配置**：删除 `~/.claude/` 目录下的缓存文件
3. **重新安装 CLI**：重新运行安装命令

---

## 相关资源

### 官方资源

- **官方文档**：https://docs.anthropic.com/en/docs/claude-code/overview
- **GitHub 仓库**：https://github.com/anthropics/claude-code
- **API 文档**：https://docs.anthropic.com/en/api
- **Claude.ai**：https://claude.ai/code

### 本仓库相关指南

- [[AI技术/ClaudeCode/Claude Code安装指南-GLM模型]] — 使用 GLM 模型安装 Claude Code
- [[AI技术/ClaudeCode/ClaudeCode浏览器自动化指南]] — 浏览器自动化操作指南
- [[AI技术/ClaudeCode/chrome-devtools-mcp 教程]] — Chrome DevTools MCP 配置教程
- [[AI技术/ClaudeCode/Claude Code 必装 Skills 推荐指南]] — 推荐的 Claude Code Skills
- [[AI技术/ClaudeCode/ClaudePluginsOfficial指南]] — 官方插件指南
- [[AI技术/ClaudeCode/Ralph Loop 使用教程]] — Ralph Loop 自动化教程

### 社区资源

- **Discord 社区**：Anthropic 官方 Discord 频道
- **GitHub Discussions**：Claude Code 仓库的讨论区
- **Reddit**：r/ClaudeAI 社区

---

> 📝 **更新日志**
>
> - 2026-04-13：初始版本，覆盖 Web-UI 核心功能与使用指南
