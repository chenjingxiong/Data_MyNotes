# Claude Plugins Official 完全指南

> **官方仓库**：[anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official)
> **更新日期**：2026-03-08

## 目录

- [简介](#简介)
- [插件类型](#插件类型)
- [安装与配置](#安装与配置)
- [官方插件推荐](#官方插件推荐)
- [插件使用技巧](#插件使用技巧)
- [第三方市场](#第三方市场)
- [开发自己的插件](#开发自己的插件)
- [最佳实践](#最佳实践)

---

## 简介

### 什么是 Claude Plugins Official？

**Claude Plugins Official** 是 Anthropic 官方维护的 Claude Code 插件目录，包含经过审核的高质量插件，用于扩展 Claude Code 的功能。

### 核心价值

| 优势 | 说明 |
|------|------|
| **官方审核** | 所有插件都经过 Anthropic 团队审核 |
| **安全可靠** | 官方维护，定期更新安全补丁 |
| **即装即用** | 无需复杂配置，开箱即用 |
| **持续更新** | 跟随 Claude Code 版本同步更新 |

### 插件生态系统

```
┌─────────────────────────────────────────────────────────────┐
│                    Claude Code                               │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ 官方插件      │  │ 社区插件      │  │ 自定义插件    │      │
│  │              │  │              │  │              │      │
│  │ • 50+ 插件   │  │ • GitHub     │  │ • 本地开发    │      │
│  │ • 官方审核   │  │ • 第三方市场  │  │ • 私有仓库    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

---

## 插件类型

Claude Code 插件可以包含以下一种或多种组件：

### 1. Commands（命令）

自定义斜杠命令，快速执行特定任务。

```bash
/commit         # 创建 Git 提交
/review-pr      # 审查 Pull Request
/test           # 运行测试
```

### 2. Agents（代理）

专门的 AI 代理，处理特定领域任务。

| 代理 | 功能 |
|------|------|
| **code-review** | 代码审查专家 |
| **documentation** | 文档生成助手 |
| **testing** | 测试用例生成 |

### 3. Skills（技能）

可重用的能力模块，让 Claude 学会新技能。

| 技能 | 用途 |
|------|------|
| **frontend-design** | 前端 UI 设计 |
| **algorithmic-art** | 算法艺术生成 |
| **doc-coauthoring** | 文档协作写作 |

### 4. Hooks（钩子）

在特定事件触发的自动化操作。

| 钩子类型 | 触发时机 |
|---------|---------|
| **pre-commit** | Git 提交前 |
| **pre-push** | Git 推送前 |
| **file-save** | 文件保存时 |

### 5. MCP Servers

模型上下文协议服务器，连接外部工具和数据源。

| MCP | 用途 |
|-----|------|
| **github** | GitHub 仓库操作 |
| **filesystem** | 文件系统访问 |
| **database** | 数据库查询 |

---

## 安装与配置

### 方式 1：从官方市场安装

```bash
# 列出所有可用插件
/plugin list

# 安装官方插件
/plugin install github@claude-plugin-directory
/plugin install playwright@claude-plugin-directory

# 查看已安装插件
/plugin status
```

### 方式 2：从 GitHub 仓库安装

```bash
# 克隆官方插件仓库
git clone https://github.com/anthropics/claude-plugins-official.git
cd claude-plugins-official

# 安装指定插件
claude plugin install ./path/to/plugin

# 或复制到插件目录
cp -r ./plugin-name ~/.claude/plugins/
```

### 方式 3：配置自定义市场

编辑 `~/.claude/config.json`：

```json
{
  "pluginMarketplaces": [
    "https://github.com/anthropics/claude-plugins-official",
    "https://github.com/your-org/custom-plugins"
  ]
}
```

### 插件启用/禁用

```bash
# 启用插件
/plugin enable plugin-name

# 禁用插件
/plugin disable plugin-name

# 卸载插件
/plugin uninstall plugin-name
```

---

## 官方插件推荐

### 🏆 必装插件 Top 10

#### 1. **typescript-lsp** ⭐⭐⭐⭐⭐

**功能**：TypeScript 语言服务协议支持

**特性**：
- 智能代码补全
- 类型检查
- 定义跳转
- 重构支持

**适用场景**：TypeScript 项目开发

**安装**：
```bash
/plugin install typescript-lsp@claude-plugin-directory
```

---

#### 2. **github** ⭐⭐⭐⭐⭐

**功能**：GitHub 集成，通过 MCP 协议操作仓库

**特性**：
- 创建和管理 Pull Request
- 代码审查
- Issue 管理
- 仓库信息查询

**适用场景**：GitHub 仓库协作

**安装**：
```bash
/plugin install github@claude-plugin-directory
```

---

#### 3. **playwright** ⭐⭐⭐⭐⭐

**功能**：浏览器自动化测试

**特性**：
- 跨浏览器测试
- 自动等待机制
- 截图和录屏
- 网络拦截

**适用场景**：E2E 测试、浏览器自动化

**安装**：
```bash
/plugin install playwright@claude-plugin-directory
```

---

#### 4. **commit-commands** ⭐⭐⭐⭐⭐

**功能**：智能 Git 提交命令

**特性**：
- 自动生成提交信息
- 遵循 Conventional Commits
- 多文件变更管理
- 提交历史分析

**适用场景**：日常 Git 操作

**安装**：
```bash
/plugin install commit-commands@claude-plugin-directory
```

---

#### 5. **security-guidance** ⭐⭐⭐⭐

**功能**：安全代码审查和指导

**特性**：
- 安全漏洞检测
- 最佳实践建议
- 依赖项安全检查
- OWASP 规则检查

**适用场景**：安全敏感项目

**安装**：
```bash
/plugin install security-guidance@claude-plugin-directory
```

---

#### 6. **context7** ⭐⭐⭐⭐

**功能**：上下文管理和优化

**特性**：
- 智能上下文压缩
- 相关文件提取
- 项目结构分析
- Token 使用优化

**适用场景**：大型项目、上下文优化

**安装**：
```bash
/plugin install context7@claude-plugin-directory
```

---

#### 7. **documentation** ⭐⭐⭐⭐

**功能**：文档生成和维护

**特性**：
- API 文档生成
- README 自动更新
- 注释标准化
- 文档版本管理

**适用场景**：文档编写、API 文档

**安装**：
```bash
/plugin install documentation@claude-plugin-directory
```

---

#### 8. **testing** ⭐⭐⭐⭐

**功能**：测试用例生成和管理

**特性**：
- 单元测试生成
- 测试覆盖率分析
- 测试数据生成
- Mock 对象创建

**适用场景**：测试驱动开发

**安装**：
```bash
/plugin install testing@claude-plugin-directory
```

---

#### 9. **frontend-design** ⭐⭐⭐⭐

**功能**：前端 UI/UX 设计助手

**特性**：
- React 组件生成
- Tailwind CSS 样式
- 响应式布局
- 设计系统支持

**适用场景**：前端开发、UI 设计

**安装**：
```bash
/plugin install frontend-design@claude-plugin-directory
```

---

#### 10. **code-review** ⭐⭐⭐⭐

**功能**：代码审查专家代理

**特性**：
- 代码质量分析
- 最佳实践检查
- 性能问题识别
- 重构建议

**适用场景**：代码审查、PR 检查

**安装**：
```bash
/plugin install code-review@claude-plugin-directory
```

---

### 🎯 领域专用插件

#### 开发工具类

| 插件 | 功能 | 评分 |
|------|------|------|
| **eslint** | ESLint 集成 | ⭐⭐⭐⭐ |
| **prettier** | 代码格式化 | ⭐⭐⭐⭐ |
| **docker** | Docker 管理 | ⭐⭐⭐ |
| **kubernetes** | K8s 资源管理 | ⭐⭐⭐ |

#### 数据处理类

| 插件 | 功能 | 评分 |
|------|------|------|
| **csv** | CSV 文件处理 | ⭐⭐⭐⭐ |
| **json-tools** | JSON 操作工具 | ⭐⭐⭐⭐ |
| **database** | 数据库查询 | ⭐⭐⭐ |

#### 文档类

| 插件 | 功能 | 评分 |
|------|------|------|
| **markdown** | Markdown 增强 | ⭐⭐⭐⭐ |
| **openapi** | OpenAPI 文档生成 | ⭐⭐⭐ |
| **swagger** | Swagger 集成 | ⭐⭐⭐ |

---

## 插件使用技巧

### 1. 查看插件帮助

```bash
# 查看插件详细信息
/plugin info github

# 查看插件可用命令
/plugin help github
```

### 2. 插件命令别名

创建自定义别名简化操作：

```bash
# 在 ~/.claude/aliases.json 中添加
{
  "pr": "/review-pr",
  "cm": "/commit",
  "test": "/run-tests"
}
```

### 3. 插件配置

大多数插件支持自定义配置：

```bash
# 编辑插件配置
claude config set plugins.github.defaultBranch "main"
claude config set plugins.playwright.timeout "10000"
```

### 4. 插件组合使用

多个插件可以协同工作：

```bash
# 组合使用 github + commit-commands
> 创建 PR 并包含自动生成的提交信息

# 组合使用 testing + documentation
> 为新功能生成测试和文档
```

---

## 第三方市场

### 推荐第三方插件市场

| 市场 | 链接 | 特点 |
|------|------|------|
| **MCP Marketplace** | [mcp-marketplace.io](https://mcp-marketplace.io) | MCP 服务器专用 |
| **Claude Marketplaces** | [claudemarketplaces.com](https://claudemarketplaces.com/) | 综合插件市场 |
| **Awesome Claude Code** | [GitHub](https://github.com/jmanhype/awesome-claude-code) | 精选资源列表 |

### 社区插件推荐

| 插件 | 来源 | 功能 |
|------|------|------|
| **chrome-devtools-mcp** | [ChromeDevTools](https://github.com/ChromeDevTools/chrome-devtools-mcp) | 浏览器自动化 |
| **knowledge-work-plugins** | [Anthropic](https://github.com/anthropics/knowledge-work-plugins) | 知识工作工具 |
| **a2a-gateway** | [win4r](https://github.com/win4r/openclaw-a2a-gateway) | Agent 间通信 |

---

## 开发自己的插件

### 插件目录结构

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json          # 插件元数据
├── commands/                # 命令定义
│   └── my-command.md
├── agents/                  # 代理定义
│   └── my-agent.md
├── skills/                  # 技能定义
│   └── my-skill.md
├── hooks/                   # 钩子脚本
│   └── pre-commit.sh
└── mcp/                     # MCP 配置
    └── mcp-config.json
```

### plugin.json 示例

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "My custom Claude Code plugin",
  "author": "Your Name",
  "claude": {
    "minVersion": "1.0.0"
  },
  "components": {
    "commands": ["commands/my-command.md"],
    "agents": ["agents/my-agent.md"],
    "skills": ["skills/my-skill.md"]
  }
}
```

### 发布到市场

1. 在 GitHub 创建仓库
2. 添加 `.claude-plugin/marketplace.json`
3. 提交到官方市场审核

```bash
# 提交插件
gh repo create my-plugin --public
cd my-plugin
echo '{"name": "my-plugin", "version": "1.0.0"}' > .claude-plugin/marketplace.json
git push origin main
```

---

## 最佳实践

### 1. 插件选择原则

| 原则 | 说明 |
|------|------|
| **按需安装** | 只安装真正需要的插件 |
| **官方优先** | 优先选择官方插件 |
| **定期更新** | 保持插件最新版本 |
| **安全审查** | 检查插件权限和来源 |

### 2. 插件管理

```bash
# 定期检查插件更新
/plugin update --all

# 清理未使用的插件
/plugin cleanup

# 导出插件配置
/plugin export > my-plugins.json
```

### 3. 性能优化

- 禁用不需要的插件以提升启动速度
- 配置插件缓存减少网络请求
- 调整插件并发数量

### 4. 安全建议

⚠️ **重要安全提示**：

- 仅从可信来源安装插件
- 审查插件请求的权限
- 定期检查插件更新日志
- 在隔离环境测试新插件

---

## 快速参考

### 常用命令

| 命令 | 功能 |
|------|------|
| `/plugin list` | 列出所有插件 |
| `/plugin install <name>` | 安装插件 |
| `/plugin uninstall <name>` | 卸载插件 |
| `/plugin enable <name>` | 启用插件 |
| `/plugin disable <name>` | 禁用插件 |
| `/plugin update <name>` | 更新插件 |

### 配置文件位置

| 系统 | 配置路径 |
|------|---------|
| **Linux/macOS** | `~/.claude/` |
| **Windows** | `%APPDATA%\Claude\` |
| **插件目录** | `~/.claude/plugins/` |

---

## 附录：插件模板

### 最小化插件模板

```json
{
  "name": "minimal-plugin",
  "version": "1.0.0",
  "description": "A minimal Claude Code plugin",
  "components": {
    "commands": []
  }
}
```

---

## 参考资源

### 官方资源

- **Claude Plugins Official**: https://github.com/anthropics/claude-plugins-official
- **Claude Code 插件文档**: https://code.claude.com/docs/en/discover-plugins
- **MCP 协议文档**: https://github.com/modelcontextprotocol

### 社区资源

- **Awesome Claude Code**: https://github.com/jmanhype/awesome-claude-code
- **MCP Marketplace**: https://mcp-marketplace.io
- **Claude Marketplaces**: https://claudemarketplaces.com/

---

> **写在最后**：Claude Plugins Official 是扩展 Claude Code 功能的最佳入口。通过合理使用官方插件，你可以大幅提升开发效率，让 Claude Code 成为你真正的智能编程伙伴。
