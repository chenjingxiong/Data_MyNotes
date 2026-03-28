# 🚀 Ruflo 快速开始指南

> 5 分钟上手 Ruflo - 企业级 AI Agent 编排平台

---

## 📋 前置检查

```bash
# 1. 检查 Node.js 版本（需要 >= 20.0.0）
node --version

# 2. 检查 npm 版本（需要 >= 9.0.0）
npm --version

# 3. （推荐）安装 Claude Code
npm install -g @anthropic-ai/claude-code
```

---

## ⚡ 三种安装方式

### 方式 1：一键安装（最简单）

```bash
# 标准安装（推荐）
curl -fsSL https://cdn.jsdelivr.net/gh/ruvnet/ruflo@main/scripts/install.sh | bash

# 完整安装（包含 MCP + 诊断）
curl -fsSL https://cdn.jsdelivr.net/gh/ruvnet/ruflo@main/scripts/install.sh | bash -s -- --full
```

### 方式 2：npx 快速启动（无需安装）

```bash
# 直接运行，无需安装
npx ruflo@latest init --wizard
```

### 方式 3：npm 全局安装

```bash
# 安装
npm install -g ruflo@latest

# 验证
ruflo --version
```

---

## 🎯 首次使用

### Step 1: 初始化

```bash
# 交互式向导（推荐新手）
npx ruflo@latest init --wizard

# 或 Codex 模式（完整功能）
npx ruflo@latest init --codex
```

### Step 2: 验证安装

```bash
# 检查状态
ruflo doctor

# 检查 MCP 集成
ruflo mcp status

# 检查 hooks 状态
ruflo hooks list
```

### Step 3: 开始使用

```bash
# 启动 Claude Code
claude-code

# 或直接使用 CLI
ruflo help
```

---

## 💡 基本使用场景

### 场景 1：代码审查

```
在 Claude Code 中输入：
"请审查 src/app.js 的代码质量，检查潜在问题"

Ruflo 会自动：
✓ 分析代码
✓ 路由到 reviewer agent
✓ 运行质量检查
✓ 生成详细报告
```

### 场景 2：生成测试

```
"为 utils/format.js 生成完整的单元测试"

Ruflo 会自动：
✓ 分析函数和逻辑
✓ 路由到 tester agent
✓ 生成测试用例
✓ 验证测试通过
```

### 场景 3：重构代码

```
"重构 api/users 模块以提高可维护性"

Ruflo 群体会：
✓ architect agent 分析架构
✓ coder agent 执行重构
✓ tester agent 验证功能
✓ reviewer agent 检查质量
```

---

## 🔧 常用命令速查

```bash
# 初始化和设置
npx ruflo@latest init --wizard       # 交互式初始化
npx ruflo@latest init --codex        # Codex 模式

# 更新
npx ruflo@latest update              # 更新版本
npx ruflo@latest update --add-missing  # 更新并添加新功能

# 状态检查
ruflo doctor                         # 运行诊断
ruflo mcp status                     # MCP 状态
ruflo hooks list                     # Hooks 列表

# 智能系统
ruflo intelligence status            # 智能系统状态
ruflo memory stats                   # 记忆统计

# 帮助
ruflo --help                         # 总帮助
ruflo <command> --help               # 命令帮助
```

---

## 🎓 下一步

- 📖 阅读 [[Ruflo 完全指南]] 了解详细功能
- 🔌 配置 [[Claude Code 集成]] 获得完整体验
- ⚙️ 探索 [[进阶配置]] 自定义你的工作流

---

## ❓ 遇到问题？

```bash
# 运行诊断
ruflo doctor --fix

# 清理并重装
npm cache clean --force
rm -rf ~/.ruflo
npx ruflo@latest init --wizard

# 获取帮助
# GitHub: https://github.com/ruvnet/claude-flow/issues
# Discord: https://discord.com/dfxmpwkG2D
```

---

**🌊 你已准备好开始使用 Ruflo！**
