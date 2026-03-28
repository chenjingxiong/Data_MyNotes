# 📖 Ruflo CLI 命令参考

> 完整的命令行接口参考文档

---

## 🎯 命令概览

Ruflo 提供 **26 个主命令**和 **140+ 子命令**，涵盖初始化、配置、管理、诊断等各个方面。

---

## 📋 命令分类

### 1️⃣ 初始化与设置

#### `init` - 初始化 Ruflo

```bash
# 交互式向导
ruflo init --wizard

# Codex 模式（完整功能）
ruflo init --codex

# 非交互式初始化
ruflo init --force

# 添加缺失的 skills/agents/commands
ruflo init --add-missing
```

**选项：**
- `--wizard` - 交互式向导模式
- `--codex` - Codex 模式初始化
- `--force` - 强制覆盖现有配置
- `--add-missing` - 添加新版本的 skills/agents

---

### 2️⃣ 更新与同步

#### `update` - 更新 Ruflo

```bash
# 更新到最新版本
ruflo update

# 更新并添加新功能
ruflo update --add-missing

# 更新到特定版本
ruflo update --version 3.5.48

# 检查可用更新
ruflo update --check
```

---

### 3️⃣ Hooks 管理

#### `hooks` - Hooks 系统管理

```bash
# 列出所有 hooks
ruflo hooks list

# 启用 hooks
ruflo hooks enable <hook-name>
ruflo hooks enable --all

# 禁用 hooks
ruflo hooks disable <hook-name>
ruflo hooks disable --all

# 创建自定义 hook
ruflo hooks create --name <name> --trigger <event>

# 编辑 hook
ruflo hooks edit <hook-name>

# 查看 hook 日志
ruflo hooks logs

# hooks 状态
ruflo hooks status
```

**可用 Hook 事件：**
- `pre-task` - 任务执行前
- `post-task` - 任务执行后
- `pre-commit` - Git 提交前
- `post-commit` - Git 提交后
- `on-error` - 错误发生时

---

### 4️⃣ 智能与记忆

#### `intelligence` - RuVector 智能系统

```bash
# 智能系统状态
ruflo intelligence status

# 触发学习
ruflo intelligence learn --pattern <pattern>

# 查看学习模式
ruflo intelligence patterns

# 清理学习缓存
ruflo intelligence clear-cache
```

#### `memory` - AgentDB 记忆管理

```bash
# 记忆统计
ruflo memory stats

# 搜索记忆
ruflo memory search <query>

# 重建索引
ruflo memory rebuild

# 清理缓存
ruflo memory clear-cache

# 配置记忆范围
ruflo memory scope --agent-local
ruflo memory scope --cross-agent
ruflo memory scope --global

# 重置记忆（谨慎！）
ruflo memory reset --confirm
```

#### `neural` - 神经网络操作

```bash
# 神经网络状态
ruflo neural status

# 训练神经网络
ruflo neural train --dataset <path>

# 查看训练模式
ruflo neural patterns

# 导出模型
ruflo neural export --output <path>
```

---

### 5️⃣ 群体与代理

#### `swarm` - 群体管理

```bash
# 启动群体
ruflo swarm start

# 停止群体
ruflo swarm stop

# 群体状态
ruflo swarm status

# 配置拓扑
ruflo swarm topology --type hierarchical
ruflo swarm topology --type mesh
ruflo swarm topology --type ring
ruflo swarm topology --type star

# 配置共识
ruflo swarm consensus --type majority
ruflo swarm consensus --type weighted
ruflo swarm consensus --type byzantine

# 查看 Queen 状态
ruflo swarm queen --status
```

#### `agent` - 代理管理

```bash
# 列出可用代理
ruflo agent list

# 创建代理
ruflo agent create --type <type> --name <name>

# 启动代理
ruflo agent start <agent-name>

# 停止代理
ruflo agent stop <agent-name>

# 代理状态
ruflo agent status

# 代理日志
ruflo agent logs <agent-name>
```

#### `worker` - Worker 管理

```bash
# 列出 workers
ruflo worker list

# 创建 worker
ruflo worker create --type <type>

# Worker 状态
ruflo worker status

# Worker 性能基准测试
ruflo worker benchmark
```

---

### 6️⃣ MCP 集成

#### `mcp` - MCP 协议管理

```bash
# MCP 状态
ruflo mcp status

# 配置 MCP
ruflo mcp setup

# 重新配置
ruflo mcp setup --force

# 列出 MCP 工具
ruflo mcp tools

# 测试 MCP 连接
ruflo mcp test
```

#### `tools` - 工具管理

```bash
# 列出所有工具
ruflo tools list

# 工具详情
ruflo tools info <tool-name>

# 启用工具
ruflo tools enable <tool-name>

# 禁用工具
ruflo tools disable <tool-name>
```

---

### 7️⃣ 诊断与调试

#### `doctor` - 系统诊断

```bash
# 完整诊断
ruflo doctor

# 快速检查
ruflo doctor --quick

# 自动修复
ruflo doctor --fix

# 详细日志
ruflo doctor --verbose

# 特定类别诊断
ruflo doctor --category memory
ruflo doctor --category mcp
ruflo doctor --category hooks
```

#### `health` - 健康检查

```bash
# 健康状态
ruflo health

# 实时监控
ruflo health --watch

# 导出健康报告
ruflo health --export <path>
```

#### `debug` - 调试模式

```bash
# 启用调试日志
ruflo debug --enable

# 禁用调试日志
ruflo debug --disable

# 查看调试日志
ruflo debug logs

# 调试特定组件
ruflo debug --component router
ruflo debug --component hooks
ruflo debug --component memory
```

---

### 8️⃣ 优化与性能

#### `optimize` - 性能优化

```bash
# Token 优化
ruflo optimize tokens --enable

# Agent Booster
ruflo optimize booster --enable-wasm
ruflo optimize booster --ast-analysis

# 设置性能目标
ruflo optimize targets --latency 100ms
ruflo optimize targets --memory 1GB

# 激进优化
ruflo optimize targets --aggressive
```

#### `performance` - 性能分析

```bash
# 性能报告
ruflo performance report

# 基准测试
ruflo performance benchmark

# 分析瓶颈
ruflo performance analyze
```

---

### 9️⃣ 安全与审计

#### `security` - 安全管理

```bash
# 安全级别
ruflo security level --strict
ruflo security level --standard
ruflo security level --permissive

# 安全特性
ruflo security prompt-injection --enable
ruflo security command-injection --enable
ruflo security path-traversal --enable

# 安全审计
ruflo security audit

# 验证配置
ruflo security validate
```

---

### 🔟 配置管理

#### `config` - 配置管理

```bash
# 查看配置
ruflo config list

# 获取配置项
ruflo config get <key>

# 设置配置项
ruflo config set <key> <value>

# 删除配置项
ruflo config unset <key>

# 编辑配置文件
ruflo config edit

# 重置配置
ruflo config reset --confirm
```

---

### 1️⃣1️⃣ 清理与维护

#### `cleanup` - 清理工具

```bash
# 清理旧数据
ruflo cleanup --older-than 7d

# 清理缓存
ruflo cleanup --cache

# 清理日志
ruflo cleanup --logs

# 完全清理
ruflo cleanup --all --confirm
```

---

## 🎯 常用命令组合

### 开发工作流

```bash
# 1. 初始化
npx ruflo@latest init --codex

# 2. 验证安装
ruflo doctor

# 3. 启动 Claude Code
claude-code

# 4. 定期更新
ruflo update --add-missing
```

### 调试工作流

```bash
# 1. 运行诊断
ruflo doctor --verbose

# 2. 检查 hooks
ruflo hooks status

# 3. 查看 MCP 状态
ruflo mcp status

# 4. 检查记忆系统
ruflo memory stats
```

### 性能优化工作流

```bash
# 1. 性能分析
ruflo performance analyze

# 2. 启用优化
ruflo optimize tokens --enable
ruflo optimize booster --enable-wasm

# 3. 设置目标
ruflo optimize targets --aggressive

# 4. 验证改进
ruflo performance benchmark
```

---

## 📝 命令别名

为方便使用，Ruflo 支持命令别名：

| 完整命令 | 别名 |
|----------|------|
| `ruflo init` | `ruflo i` |
| `ruflo update` | `ruflo up` |
| `ruflo doctor` | `ruflo doc` |
| `ruflo hooks` | `ruflo h` |
| `ruflo swarm` | `ruflo s` |
| `ruflo memory` | `ruflo mem` |
| `ruflo config` | `ruflo cfg` |

---

## 🔍 获取帮助

```bash
# 总帮助
ruflo --help

# 命令帮助
ruflo <command> --help

# 示例
ruflo init --help
ruflo hooks --help
```

---

**💡 提示：大多数命令支持 `--dry-run` 选项，可以在不实际执行的情况下预览操作结果。**
