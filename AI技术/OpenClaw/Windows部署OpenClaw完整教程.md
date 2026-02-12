# Windows 部署 OpenClaw 完整教程

> 本教程详细介绍如何在 Windows 系统上安装和配置 OpenClaw，连接 Telegram，并安装浏览器扩展插件。

---

## 📋 目录

1. [环境准备](#环境准备)
2. [PowerShell 配置](#powershell-配置)
3. [OpenClaw 安装](#openclaw-安装)
4. [Telegram 配对配置](#telegram-配对配置)
5. [Chrome 浏览器插件安装](#chrome-浏览器插件安装)
6. [OpenClaw 服务管理](#openclaw-服务管理)
7. [常见问题与故障排查](#常见问题与故障排查)

---

## 环境准备

### 系统要求

- **操作系统**: Windows 10/11 (推荐 Windows 11)
- **内存**: 最低 4GB，推荐 8GB+
- **存储**: 最低 10GB 可用空间
- **网络**: 需要访问公网

### 1. 安装 Git

Git 是 OpenClaw 运行的必要依赖。

1. 访问 [Git 官网](https://git-scm.com/)
2. 下载 Windows 版本安装程序
3. 运行安装程序，使用默认设置完成安装
4. 验证安装：

```powershell
git --version
```

### 2. 安装 Node.js

Node.js 是 OpenClaw 运行时环境。

1. 访问 [Node.js 官网](https://nodejs.org/)
2. 下载 LTS 版本（推荐当前 LTS 版本）
3. 运行安装程序，使用默认设置完成安装
4. 验证安装：

```powershell
node --version
npm --version
```

---

## PowerShell 配置

OpenClaw 安装脚本需要 PowerShell 执行策略权限。

### 1. 以管理员身份打开 PowerShell

- 按 `Win + X` 键
- 选择 **Windows PowerShell (管理员)** 或 **终端 (管理员)**

### 2. 设置执行策略

```powershell
# 临时允许当前进程执行脚本
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass

# 为当前用户设置执行策略（推荐）
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### 执行策略说明

| 策略 | 说明 |
|-----|------|
| `Bypass` | 不阻止任何操作，仅当前会话有效 |
| `RemoteSigned` | 本地脚本可运行，网络脚本需签名 |

---

## OpenClaw 安装

### 1. 运行安装脚本

在 PowerShell 中执行：

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

| 命令参数 | 说明 |
|---------|------|
| `iwr` | Invoke-WebRequest，下载网页内容 |
| `-useb` | 静默模式 |
| `iex` | Invoke-Expression，执行下载的脚本 |

### 2. 安装过程

安装脚本会自动完成以下操作：

- ✓ 检测系统环境
- ✓ 安装 OpenClaw 核心程序
- ✓ 配置环境变量
- ✓ 创建配置目录
- ✓ 注册系统服务

### 3. 验证安装

```powershell
openclaw --version
```

### 4. 查看安装位置

OpenClaw 默认安装路径：

```
C:\Users\<用户名>\.openclaw
```

目录结构：

```
.openclaw/
├── bin/              # 可执行文件
├── config/           # 配置文件
├── browser/          # 浏览器扩展
├── data/             # 数据目录
└── logs/             # 日志文件
```

---

## Telegram 配对配置

### 1. 获取配对码

1. 在 Telegram 中搜索 `@OpenClawBot` 或相关官方机器人
2. 发送 `/pair` 命令获取配对码
3. 复制显示的配对码

### 2. 执行配对命令

在 PowerShell 中执行：

```powershell
openclaw pairing approve telegram <配对码>
```

**示例**：

```powershell
openclaw pairing approve telegram ABC123DEF456
```

### 3. 配对成功提示

配对成功后，你会看到：

```
✓ Telegram 配对成功
✓ 账号: @your_username
✓ 连接状态: 已激活
```

### 配对状态检查

```powershell
openclaw pairing status
```

---

## Chrome 浏览器插件安装

### 1. 安装浏览器扩展

在 PowerShell 中执行：

```powershell
openclaw browser extension install
```

### 2. 找到扩展文件

扩展文件会被复制到：

```
C:\Users\<用户名>\.openclaw\browser
```

### 3. 在 Chrome 中安装

1. 打开 Chrome 浏览器
2. 访问 `chrome://extensions/`
3. 开启右上角 **开发者模式**
4. 点击 **加载已解压的扩展程序**
5. 选择 OpenClaw 浏览器目录：
   ```
   C:\Users\<用户名>\.openclaw\browser
   ```

### 4. 验证安装

安装成功后，Chrome 工具栏会出现 OpenClaw 图标。

### 5. 配置扩展

点击扩展图标，配置：

- **API 端点**: `http://localhost:8080`
- **自动连接**: 开启
- **通知**: 开启

---

## OpenClaw 服务管理

### 1. 查看服务状态

```powershell
openclaw status
```

输出示例：

```
OpenClaw Gateway Service
  Status: Running
  Uptime: 2 hours 15 minutes
  Version: 1.0.0
  Port: 8080
```

### 2. 启动服务

```powershell
openclaw gateway start
```

### 3. 停止服务

```powershell
openclaw gateway stop
```

### 4. 重启服务

```powershell
openclaw gateway restart
```

### 5. 查看日志

```powershell
openclaw logs
```

查看实时日志：

```powershell
openclaw logs --follow
```

### 常用命令汇总

| 命令 | 说明 |
|-----|------|
| `openclaw status` | 查看运行状态 |
| `openclaw gateway start` | 启动服务 |
| `openclaw gateway stop` | 停止服务 |
| `openclaw gateway restart` | 重启服务 |
| `openclaw logs` | 查看日志 |
| `openclaw pairing status` | 查看配对状态 |
| `openclaw --help` | 查看帮助 |

---

## 常见问题与故障排查

### Q1: PowerShell 执行策略错误

**错误信息**：

```
无法加载文件 install.ps1，因为在此系统上禁止运行脚本。
```

**解决方案**：

```powershell
# 以管理员身份运行 PowerShell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### Q2: Node.js 版本过低

**错误信息**：

```
Node.js version must be 16.0.0 or higher
```

**解决方案**：

1. 访问 [Node.js 官网](https://nodejs.org/)
2. 下载最新的 LTS 版本
3. 卸载旧版本后重新安装

### Q3: Telegram 配对失败

**错误信息**：

```
Pairing failed: Invalid or expired code
```

**解决方案**：

1. 确认配对码是否正确（注意大小写）
2. 配对码有时效性，请重新获取
3. 检查网络连接是否正常

### Q4: 浏览器扩展无法加载

**错误信息**：

```
Manifest file is missing or unreadable
```

**解决方案**：

```powershell
# 重新安装扩展
openclaw browser extension install

# 检查目录是否存在
dir C:\Users\<用户名>\.openclaw\browser
```

### Q5: 服务启动失败

**错误信息**：

```
Failed to start gateway: Port 8080 already in use
```

**解决方案**：

```powershell
# 检查端口占用
netstat -ano | findstr :8080

# 如果被占用，结束该进程或修改 OpenClaw 端口
# 编辑配置文件
notepad C:\Users\<用户名>\.openclaw\config\gateway.yaml
```

### Q6: 网络连接问题

**错误信息**：

```
Connection timeout: Unable to reach openclaw.ai
```

**解决方案**：

1. 检查网络连接
2. 配置代理（如需）：

```powershell
$env:HTTP_PROXY="http://your-proxy:port"
$env:HTTPS_PROXY="http://your-proxy:port"

# 然后重新运行安装
iwr -useb https://openclaw.ai/install.ps1 | iex
```

---

## 参考资源

### 官方资源

- [OpenClaw 官网](https://openclaw.ai/) - 官方网站
- [OpenClaw GitHub](https://github.com/openclaw/openclaw) - 源代码仓库
- [OpenClaw 文档](https://docs.openclaw.ai/) - 官方文档
- [Telegram 官网](https://telegram.org/) - Telegram 下载

### 教程文章

- [OpenClaw Windows 快速部署指南](https://openclaw.ai/docs/windows)
- [Telegram Bot 配对教程](https://core.telegram.org/bots)
- [Chrome 扩展开发指南](https://developer.chrome.com/docs/extensions/)

### 相关工具

- [Git 下载](https://git-scm.com/)
- [Node.js 下载](https://nodejs.org/)
- [PowerShell 文档](https://docs.microsoft.com/powershell/)

---

## 结语

通过本教程，你应该已经成功在 Windows 上安装并配置了 OpenClaw，包括：

- ✓ 安装 Git 和 Node.js
- ✓ 配置 PowerShell 执行策略
- ✓ 安装 OpenClaw
- ✓ 完成 Telegram 配对
- ✓ 安装 Chrome 浏览器扩展
- ✓ 掌握服务管理命令

现在你可以通过 Telegram 或浏览器扩展使用 OpenClaw 的强大功能了！

如有问题，请参考上方故障排查部分，或查阅官方资源获取更多帮助。

祝你使用愉快！🎉

---

**最后更新**: 2026-02-12
