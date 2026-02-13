# Claude Code 安装教程 - 使用 GLM 模型

> 📅 创建时间：2026-02-13
> 🎯 目标：无需翻墙，使用智谱 GLM 模型替代 Claude 官方 API，成本降低 90%

---

## 📖 什么是 Claude Code？

Claude Code 是 Anthropic 官方推出的 **AI 编程助手终端工具**，它可以：
- 🤖 帮你编写、调试、重构代码
- 🔍 搜索和分析代码库
- 💡 自动修复 bug 和优化性能
- 🛠️ 执行 git 操作、运行测试

## 🌟 为什么要用 GLM 模型？

| 对比项 | Claude 官方 API | GLM 模型 |
|--------|----------------|----------|
| **网络要求** | 需要翻墙 | 国内直连 |
| **API 额度** | $100 免费额度 | ¥100 免费额度（约3倍） |
| **使用成本** | 较高 | 降低 90% |
| **响应速度** | 较快（但有延迟） | 国内访问更快 |

---

## 🚀 安装步骤

### 第一步：安装 Node.js

Claude Code 依赖 Node.js 环境。

1. 访问 [Node.js 官网](https://nodejs.org/en/download/)
2. 下载 LTS 版本（推荐 18.x 或 20.x）
3. 安装完成后，验证安装：

```bash
node -v
npm -v
```

### 第二步：安装 Claude Code

使用 npm 全局安装 Claude Code：

```bash
npm install -g @anthropic-ai/claude-code
```

或者使用一键启动脚本（GitHub 项目）：
```bash
npm install -g glm-claude
```

### 第三步：注册智谱 AI 账号

1. 访问 [智谱 AI 开放平台](https://open.bigmodel.cn/) 或 [z.ai](https://z.ai/)
2. 注册并登录账号
3. 进入 [API Key 管理页面](https://z.ai/manage-apikey/subscription)
4. 创建新的 API Key 并保存

### 第四步：配置 GLM 模型

#### 方式一：使用环境变量（推荐）

在终端中设置环境变量：

**Windows (PowerShell):**
```powershell
$env:ANTHROPIC_API_KEY="你的智谱API密钥"
$env:ANTHROPIC_BASE_URL="https://open.bigmodel.cn/api/coding/paas/v1"
```

**Windows (CMD):**
```cmd
set ANTHROPIC_API_KEY=你的智谱API密钥
set ANTHROPIC_BASE_URL=https://open.bigmodel.cn/api/coding/paas/v1
```

**macOS/Linux:**
```bash
export ANTHROPIC_API_KEY="你的智谱API密钥"
export ANTHROPIC_BASE_URL="https://open.bigmodel.cn/api/coding/paas/v1"
```

#### 方式二：使用 CC-Switch 工具

1. 下载安装 [CC-Switch](https://github.com/musistudio/claude-code-router)
2. 添加新提供商，选择 "Zhipu GLM" 预设
3. 填入 API Key 并保存

#### 方式三：永久配置（添加到配置文件）

**Windows PowerShell 配置文件：**
```powershell
# 编辑配置文件
notepad $PROFILE

# 添加以下内容
$env:ANTHROPIC_API_KEY="你的智谱API密钥"
$env:ANTHROPIC_BASE_URL="https://open.bigmodel.cn/api/coding/paas/v1"
```

**macOS/Linux (.bashrc 或 .zshrc):**
```bash
# 编辑配置文件
nano ~/.bashrc  # 或 ~/.zshrc

# 添加以下内容
export ANTHROPIC_API_KEY="你的智谱API密钥"
export ANTHROPIC_BASE_URL="https://open.bigmodel.cn/api/coding/paas/v1"
```

#### 方式四：修改 Claude JSON 配置文件

直接编辑 Claude Code 的配置文件，永久生效。

**查找配置文件位置：**

| 系统 | 配置文件路径 |
|------|-------------|
| Windows | `%USERPROFILE%\.claude\config.json` |
| macOS/Linux | `~/.config/claude-code/config.json` |

**配置步骤：**

1. 找到并打开配置文件：

```bash
# Windows PowerShell
notepad $env:USERPROFILE\.claude\config.json

# macOS/Linux
nano ~/.config/claude-code/config.json
```

2. 如果文件不存在，创建它：

```bash
# Windows
New-Item -Path $env:USERPROFILE\.claude\config.json -ItemType File

# macOS/Linux
mkdir -p ~/.config/claude-code
touch ~/.config/claude-code/config.json
```

3. 编辑配置文件，添加以下内容：

```json
{
  "apiUrl": "https://open.bigmodel.cn/api/coding/paas/v1",
  "apiKey": "你的智谱API密钥",
  "provider": "zhipu",
  "model": "glm-4.5"
}
```

4. 保存文件后，重新加载配置：

```bash
# 验证配置是否生效
claude config show
```

**配置文件说明：**

| 字段 | 说明 | 示例值 |
|------|------|--------|
| `apiUrl` | API 地址 | `https://open.bigmodel.cn/api/coding/paas/v1` |
| `apiKey` | 智谱 API Key | `your-zhipu-api-key-here` |
| `provider` | 提供商 | `zhipu` 或 `anthropic` |
| `model` | 使用的模型 | `glm-4.5`, `glm-4.7`, `glm-4-flash` |

**多环境配置（可选）：**

如果需要在不同的项目使用不同的配置，可以在项目根目录创建 `.clauderc` 文件：

```json
{
  "apiUrl": "https://open.bigmodel.cn/api/coding/paas/v1",
  "apiKey": "项目专用的API密钥",
  "model": "glm-4.7"
}
```

### 第五步：验证安装

运行以下命令测试：

```bash
claude --version
```

首次运行会提示登录，使用配置的 API Key 即可。

---

## 🎮 基本使用

### 启动 Claude Code

```bash
# 在当前项目目录启动
claude

# 或指定工作目录
claude /path/to/project
```

### 常用命令

| 命令 | 说明 |
|------|------|
| `help` | 查看帮助 |
| `exit` 或 `quit` | 退出 |
| `clear` | 清屏 |
| `/commit` | 创建 git commit |
| `/review` | 代码审查 |

### 使用示例

```bash
# 让 Claude 解释代码
claude "解释一下这段代码的作用"

# 让 Claude 修复 bug
claude "帮我修复登录功能的 bug"

# 让 Claude 重构代码
claude "用更高效的方式重构这个函数"
```

---

## 🔧 IDE 插件安装

### VS Code

1. 安装 Claude Code 官方扩展
2. 在设置中配置 API 地址和密钥

### JetBrains 系列（IDEA、PyCharm 等）

1. 在插件市场搜索 "Claude Code"
2. 安装并配置 GLM API

---

## 💡 高级配置

### 模型选择

GLM 提供多种模型，可根据需求选择：

| 模型 | 特点 | 适用场景 |
|------|------|----------|
| GLM-4.7 | 最新旗舰 | 复杂任务 |
| GLM-4.5 | 性价比高 | 日常开发 |
| GLM-4-Flash | 快速响应 | 简单任务 |

### 代理配置（如需要）

```bash
export HTTP_PROXY=http://127.0.0.1:7890
export HTTPS_PROXY=http://127.0.0.1:7890
```

---

## ❓ 常见问题

### 1. 提示 404 错误

**解决方案：** 检查 API 地址是否正确，GLM Coding Plan 的正确地址是：
```
https://open.bigmodel.cn/api/coding/paas/v4/chat/completions
```

### 2. 安装失败

**解决方案：**
```bash
# 清除 npm 缓存
npm cache clean --force

# 使用国内镜像
npm config set registry https://registry.npmmirror.com
```

### 3. 权限错误（Linux/Mac）

**解决方案：**
```bash
sudo npm install -g @anthropic-ai/claude-code
```

---

## 📚 参考资源

- [Claude Code 2026 最新保姆级安装指南 - 腾讯云](https://cloud.tencent.com/developer/article/2626713)
- [Claude Code 免翻上手教程，以及改用GLM指南](https://vonng.com/db/claude-code-intro/)
- [claude code + GLM，保姆级配置教程 - 知乎专栏](https://zhuanlan.zhihu.com/p/1957370685748938170)
- [Claude Code × 智谱BigModel 实战集成指南 - 腾讯云](https://cloud.tencent.com/developer/article/2620125)
- [GitHub: 一键启动使用GLM模型的Claude Code](https://github.com/alchaincyf/glm-claude)

---

## 🏷️ 标签

#ClaudeCode #GLM #AI编程 #智谱AI #开发工具
