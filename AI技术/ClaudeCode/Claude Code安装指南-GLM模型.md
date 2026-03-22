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

#### 方法一：Debian 13 使用 apt 安装（推荐）

如果你使用 Debian 13，可以使用官方 NodeSource 仓库通过 apt 安装最新版本的 Node.js：

```bash
# 1. 更新包列表
sudo apt update

# 2. 安装必要的工具
sudo apt install -y curl ca-certificates gnupg

# 3. 添加 NodeSource 仓库（安装 Node.js 20.x LTS）
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

# 4. 安装 Node.js
sudo apt install -y nodejs

# 5. 验证安装
node -v
npm -v
```

**可选版本：**

- **Node.js 22.x（当前最新版）：**
  ```bash
  curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
  sudo apt install -y nodejs
  ```

- **Node.js 18.x（旧版 LTS）：**
  ```bash
  curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
  sudo apt install -y nodejs
  ```

**安装构建工具（可选，某些原生模块可能需要）：**

```bash
sudo apt install -y build-essential
```

#### 方法二：官网下载安装

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

## 🇨🇳 中国区特别说明

### 跳过验证快速配置

如果你在中国大陆，直接使用 Claude Code 官方账号需要手机验证和翻墙。以下是 **跳过验证、直接使用** 的方法：

#### 方法一：环境变量跳过（最简单）

直接设置环境变量，无需登录验证：

**Windows PowerShell (临时):**
```powershell
$env:ANTHROPIC_API_KEY="你的智谱API密钥"
$env:ANTHROPIC_BASE_URL="https://open.bigmodel.cn/api/paas/v4"
claude --api-key $env:ANTHROPIC_API_KEY --api-url $env:ANTHROPIC_BASE_URL
```

**Windows PowerShell (永久 - 添加到配置文件):**
```powershell
# 编辑 PowerShell 配置文件
notepad $PROFILE

# 添加以下内容（如果文件不存在，先运行 New-Item -Path $PROFILE -ItemType File）
$env:ANTHROPIC_API_KEY="你的智谱API密钥"
$env:ANTHROPIC_BASE_URL="https://open.bigmodel.cn/api/paas/v4"

# 保存后重新加载
. $PROFILE
```

**macOS/Linux:**
```bash
# 添加到 ~/.bashrc 或 ~/.zshrc
echo 'export ANTHROPIC_API_KEY="你的智谱API密钥"' >> ~/.bashrc
echo 'export ANTHROPIC_BASE_URL="https://open.bigmodel.cn/api/paas/v4"' >> ~/.bashrc
source ~/.bashrc

# 直接启动
claude --api-key "$ANTHROPIC_API_KEY" --api-url "$ANTHROPIC_BASE_URL"
```

#### 方法二：修改配置文件永久生效

编辑配置文件后，无需每次输入验证：

> **💡 重要提示：** 初次安装时配置文件不存在，需要先创建！

**Windows 一键创建配置文件:**

```powershell
# 复制以下命令到 PowerShell，替换 YOUR_API_KEY 为你的智谱 API Key
$apiKey = "YOUR_API_KEY"

# 创建目录（如果不存在）
New-Item -Path "$env:USERPROFILE\.claude" -ItemType Directory -Force | Out-Null

# 创建配置文件
@"
{
  "apiUrl": "https://open.bigmodel.cn/api/paas/v4",
  "apiKey": "$apiKey",
  "provider": "zhipu",
  "model": "glm-4.7",
  "hasCompletedOnboarding": true
}
"@ | Out-File -FilePath "$env:USERPROFILE\.claude\config.json" -Encoding utf8

# 验证配置文件
Write-Host "配置文件已创建: $env:USERPROFILE\.claude\config.json"
Get-Content "$env:USERPROFILE\.claude\config.json"
```

**macOS/Linux 一键创建配置文件:**

```bash
# 复制以下命令到终端，替换 YOUR_API_KEY 为你的智谱 API Key
apiKey="YOUR_API_KEY"

# 创建目录（如果不存在）
mkdir -p ~/.config/claude-code

# 创建配置文件
cat > ~/.config/claude-code/config.json << EOF
{
  "apiUrl": "https://open.bigmodel.cn/api/paas/v4",
  "apiKey": "$apiKey",
  "provider": "zhipu",
  "model": "glm-4.7",
  "hasCompletedOnboarding": true
}
EOF

# 验证配置文件
echo "配置文件已创建: ~/.config/claude-code/config.json"
cat ~/.config/claude-code/config.json
```

**配置文件位置对照表:**

| 系统 | 配置文件路径 | 命令查看 |
|------|-------------|----------|
| Windows | `%USERPROFILE%\.claude\config.json` | `echo $env:USERPROFILE\.claude\config.json` |
| macOS/Linux | `~/.config/claude-code/config.json` | `echo ~/.config/claude-code/config.json` |

**手动创建方式（可选）:**

如果命令行方式不习惯，也可以手动创建：

1. **Windows:**
   - 按 `Win + R`，输入 `%USERPROFILE%\.claude` 回车
   - 如果目录不存在，先创建 `.claude` 文件夹
   - 右键 → 新建 → 文本文档，命名为 `config.json`
   - 用记事本打开，粘贴以下内容：

```json
{
  "apiUrl": "https://open.bigmodel.cn/api/paas/v4",
  "apiKey": "你的智谱API密钥",
  "provider": "zhipu",
  "model": "glm-4.7",
  "hasCompletedOnboarding": true
}
```

2. **macOS/Linux:**
   ```bash
   mkdir -p ~/.config/claude-code
   nano ~/.config/claude-code/config.json
   # 粘贴 JSON 内容，按 Ctrl+X 保存退出
   ```

**验证配置是否生效:**

```bash
# 查看当前配置
claude config show
# 或直接运行测试
claude --version
```

**配置字段说明:**

| 字段 | 说明 | 是否必需 |
|------|------|---------|
| `apiUrl` | API 地址 | ✅ 必需 |
| `apiKey` | 智谱 API Key | ✅ 必需 |
| `provider` | 提供商（zhipu/anthropic） | ✅ 必需 |
| `model` | 使用的模型（glm-4.7/glm-4.5/glm-4-flash） | ✅ 必需 |
| `hasCompletedOnboarding` | 跳过新手引导/登录验证 | ⭐ 推荐 |

> **💡 关于 `hasCompletedOnboarding`:**
> - 设置为 `true` 可以跳过 Claude Code 的首次使用引导流程
> - 避免每次启动时弹出登录/验证提示
> - 配合 GLM API Key 使用，实现无感启动

#### 方法三：使用启动脚本

创建一个一键启动脚本，避免每次手动配置：

**Windows (start-claude.ps1):**
```powershell
# 保存为 start-claude.ps1
$env:ANTHROPIC_API_KEY="你的智谱API密钥"
$env:ANTHROPIC_BASE_URL="https://open.bigmodel.cn/api/paas/v4"
claude $args
```

使用方式：
```powershell
# 允许运行脚本（首次需要）
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# 启动 Claude
.\start-claude.ps1
```

**macOS/Linux (start-claude.sh):**
```bash
# 保存为 start-claude.sh 并添加执行权限
#!/bin/bash
export ANTHROPIC_API_KEY="你的智谱API密钥"
export ANTHROPIC_BASE_URL="https://open.bigmodel.cn/api/paas/v4"
claude "$@"
```

```bash
chmod +x start-claude.sh
./start-claude.sh
```

### 国内镜像加速

如果遇到网络问题，可以配置 npm 镜像：

```bash
# 使用淘宝镜像
npm config set registry https://registry.npmmirror.com

# 验证
npm config get registry
```

### API 地址对照表

| 用途 | API 地址 |
|------|----------|
| GLM Coding (推荐) | `https://open.bigmodel.cn/api/paas/v4` |
| GLM Coding 备用 | `https://open.bigmodel.cn/api/coding/paas/v4/chat/completions` |
| 智谱通用 API | `https://open.bigmodel.cn/api/paas/v4/chat/completions` |

---

## 🔄 卸载与重装

### 卸载 Claude Code

如果需要完全卸载 Claude Code，按以下步骤操作：

#### Windows 卸载

```powershell
# 1. 卸载全局包
npm uninstall -g @anthropic-ai/claude-code

# 2. 删除配置文件和缓存
Remove-Item -Path "$env:USERPROFILE\.claude" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item -Path "$env:APPDATA\claude-code" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item -Path "$env:LOCALAPPDATA\claude-code" -Recurse -Force -ErrorAction SilentlyContinue

# 3. 清除 npm 缓存（可选）
npm cache clean --force
```

**一键卸载脚本:**
```powershell
# 保存为 uninstall-claude.ps1 并运行
npm uninstall -g @anthropic-ai/claude-code
@("$env:USERPROFILE\.claude", "$env:APPDATA\claude-code", "$env:LOCALAPPDATA\claude-code") | ForEach-Object {
    if (Test-Path $_) { Remove-Item -Path $_ -Recurse -Force }
}
Write-Host "Claude Code 已完全卸载" -ForegroundColor Green
```

#### macOS/Linux 卸载

```bash
# 1. 卸载全局包
npm uninstall -g @anthropic-ai/claude-code

# 2. 删除配置文件和缓存
rm -rf ~/.claude
rm -rf ~/.config/claude-code
rm -rf ~/Library/Application\ Support/claude-code  # macOS

# 3. 清除 npm 缓存（可选）
npm cache clean --force

# 4. 验证卸载
which claude  # 应该返回空或"not found"
```

**一键卸载脚本:**
```bash
# 保存为 uninstall-claude.sh 并运行
#!/bin/bash
npm uninstall -g @anthropic-ai/claude-code
rm -rf ~/.claude ~/.config/claude-code ~/Library/Application\ Support/claude-code
npm cache clean --force
echo "Claude Code 已完全卸载"
```

### 重装 Claude Code

重装前建议先卸载，然后重新安装：

#### Windows 重装

```powershell
# 1. 卸载旧版本
npm uninstall -g @anthropic-ai/claude-code

# 2. 清理缓存
npm cache clean --force

# 3. 重新安装
npm install -g @anthropic-ai/claude-code

# 4. 验证安装
claude --version
```

#### macOS/Linux 重装

```bash
# 1. 卸载旧版本
npm uninstall -g @anthropic-ai/claude-code

# 2. 清理缓存
npm cache clean --force

# 3. 重新安装
npm install -g @anthropic-ai/claude-code

# 4. 验证安装
claude --version
```

### 重装后恢复配置

如果你之前备份了配置文件，可以直接恢复：

```bash
# Windows
Copy-Item -Path "$env:USERPROFILE\.claude.backup\config.json" -Destination "$env:USERPROFILE\.claude\config.json"

# macOS/Linux
cp ~/.claude.backup/config.json ~/.config/claude-code/config.json
```

### 保留配置卸载

如果想保留配置文件只卸载程序：

```bash
# 只卸载程序，不删除配置
npm uninstall -g @anthropic-ai/claude-code

# 配置文件保留位置
# Windows: %USERPROFILE%\.claude\config.json
# macOS/Linux: ~/.config/claude-code/config.json
```

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
