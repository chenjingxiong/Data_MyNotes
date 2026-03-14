# Windows 管理 Incus 服务器指南

## 目录

1. [简介](#1-简介)
2. [安装 Windows 客户端](#2-安装-windows-客户端)
3. [配置远程连接](#3-配置远程连接)
4. [基本操作](#4-基本操作)
5. [虚拟机远程控制台](#5-虚拟机远程控制台)
6. [常见问题](#6-常见问题)
7. [高级配置](#7-高级配置)

---

## 1. 简介

### 1.1 什么是 Incus Windows 客户端

Incus Windows 客户端允许你从 Windows 系统远程管理运行在 Linux 服务器上的 Incus 容器和虚拟机。它提供：

- **命令行工具 (CLI)**: 与 Linux 版本功能相同的 `incus` 命令
- **远程查看器**: 图形化虚拟机控制台连接工具
- **完整 API 支持**: 通过 REST API 与 Incus 服务器通信

### 1.2 系统要求

| 项目 | 要求 |
|------|------|
| **操作系统** | Windows 10/11 或 Windows Server 2019+ |
| **权限** | 管理员权限（安装时需要） |
| **网络** | 能够访问 Incus 服务器的 8443 端口 |

### 1.3 功能对比

| 功能 | Windows 客户端 | Linux 客户端 |
|------|---------------|--------------|
| 容器管理 | ✅ | ✅ |
| 虚拟机管理 | ✅ | ✅ |
| `incus exec` | ✅ | ✅ |
| `incus shell` | ❌ 使用 `incus exec` | ✅ |
| 图形控制台 | ✅ 通过远程查看器 | ✅ 直接访问 |
| 本地运行实例 | ❌ | ✅ |

---

## 2. 安装 Windows 客户端

### 2.1 使用 WinGet 安装（推荐）

WinGet 是 Windows 官方包管理器，Windows 10/11 自带。

**步骤**：

1. 以管理员身份打开 PowerShell
2. 执行以下命令：

```powershell
# 更新 WinGet 索引（可选）
winget source update

# 安装 Incus 客户端
winget install -e --id LinuxContainers.Incus
```

**预期输出**：
```
已找到 Incus [LinuxContainers.Incus]
版本: 6.x.x
此安装程序想要对你的计算机进行更改
正在启动安装包...
已成功安装
```

### 2.2 使用 Chocolatey 安装

如果你已安装 Chocolatey 包管理器：

```powershell
# 以管理员身份运行
choco install incus
```

### 2.3 手动下载安装

1. 访问 [Incus GitHub Releases](https://github.com/lxc/incus/releases)
2. 下载最新的 Windows 安装包（`.msi` 或 `.zip`）
3. 运行安装程序或解压到指定目录

### 2.4 验证安装

```powershell
# 检查版本
incus version

# 预期输出
Client version: 6.0.x
Server version: (none)  # Windows 上无本地服务器
```

### 2.5 配置环境变量（如需要）

如果安装后命令不可用，手动添加到 PATH：

```powershell
# 查看当前 PATH
$env:PATH -split ';'

# 添加到 PATH（永久）
[System.Environment]::SetEnvironmentVariable('Path', $env:PATH + ';C:\Program Files\Incus', [System.EnvironmentVariableTarget]::User)
```

---

## 3. 配置远程连接

### 3.1 服务器端准备

在 Incus 服务器上确保以下配置：

```bash
# 在 Linux 服务器上执行

# 1. 确保 API 监听所有接口
incus config set core.https_address '[::]:8443'

# 2. 设置信任密码（首次配置）
incus config set core.trust_password 'YourSecurePassword'

# 3. 配置防火墙
sudo ufw allow 8443/tcp

# 4. 验证配置
incus config show
```

### 3.2 从 Windows 添加远程服务器

**方法 1: 使用信任密码（推荐）**

```powershell
# 添加远程服务器
incus remote add my-server https://<服务器IP>:8443

# 系统会提示：
# - 服务器证书指纹（确认是否信任）
# - 输入信任密码
```

**示例**：
```powershell
PS C:\> incus remote add prod-server https://192.168.1.100:8443
Certificate fingerprint: 1234567890abcdef...
ok (y/n)? y
Admin password for prod-server: ********
Client has been successfully added.
```

**方法 2: 使用信任令牌**

```bash
# 在服务器上生成令牌
incus config trust add windows-client --type=client

# 复制输出的令牌
```

```powershell
# 在 Windows 上使用令牌
incus remote add my-server https://<服务器IP>:8443 --token=<粘贴令牌>
```

### 3.3 切换默认远程

```powershell
# 设置默认远程服务器
incus remote switch my-server

# 查看当前远程
incus remote list
```

### 3.4 测试连接

```powershell
# 列出远程实例
incus list my-server:

# 查看服务器信息
incus info my-server:
```

---

## 4. 基本操作

### 4.1 实例管理

```powershell
# 列出所有实例
incus list my-server:

# 列出所有实例（详细格式）
incus list my-server: --format=table --columns=n,s,dstate,4

# 创建容器
incus launch my-server:images:ubuntu/24.04 my-container

# 创建虚拟机
incus launch my-server:images:ubuntu/24.04/cloud my-vm --vm

# 停止实例
incus stop my-server:my-container

# 启动实例
incus start my-server:my-container

# 重启实例
incus restart my-server:my-container

# 删除实例
incus delete my-server:my-container
```

### 4.2 在实例中执行命令

```powershell
# 执行单条命令
incus exec my-server:my-container -- apt update
incus exec my-server:my-container -- cat /etc/os-release

# 交互式 Shell（Windows 客户端使用 exec）
incus exec my-server:my-container -- bash

# 执行多条命令
incus exec my-server:my-container -- bash -c "apt update && apt upgrade -y"
```

### 4.3 文件传输

```powershell
# 从 Windows 复制文件到实例
incus file push C:\local\file.txt my-server:my-container /tmp/file.txt

# 从实例复制文件到 Windows
incus file pull my-server:my-container/etc/hosts C:\local\hosts

# 查看文件内容
incus file pull my-server:my-container/etc/os-release -
```

### 4.4 快照管理

```powershell
# 创建快照
incus snapshot create my-server:my-container snap1

# 列出快照
incus snapshot list my-server:my-container

# 恢复快照
incus snapshot restore my-server:my-container snap1

# 删除快照
incus snapshot delete my-server:my-container snap1
```

### 4.5 实例配置

```powershell
# 查看配置
incus config show my-server:my-container

# 设置资源限制
incus config set my-server:my-container limits.cpu=4
incus config set my-server:my-container limits.memory=8GiB

# 设置自动启动
incus config set my-server:my-container boot.autostart=true

# 添加网络设备
incus config device add my-server:my-container eth1 nic \
    nictype=bridged parent=incusbr0
```

---

## 5. 虚拟机远程控制台

### 5.1 Incus 远程查看器

Incus Windows 客户端自带虚拟机远程查看器，可以图形化访问 VM 控制台。

**启动方式**：

```powershell
# 方法 1: 通过命令启动
incus console my-server:my-vm --type=gui

# 方法 2: 启动时自动打开控制台
incus start my-server:my-vm --console
```

### 5.2 使用 RDP/VNC

如果虚拟机配置了远程桌面：

```powershell
# 暴露 RDP 端口
incus config device add my-server:my-vm rdp proxy \
    listen=tcp:0.0.0.0:3389 \
    connect=tcp:10.10.0.50:3389 \
    bind=host

# 使用 Windows 远程桌面连接
mstsc /v:服务器IP:3389
```

### 5.3 串口控制台（文本模式）

```powershell
# 连接到串口控制台
incus console my-server:my-vm

# 退出: Ctrl+a q
```

---

## 6. 常见问题

### 6.1 连接问题

#### 问题 1: 无法连接到服务器

```powershell
# 测试网络连接
Test-NetConnection -ComputerName <服务器IP> -Port 8443

# 检查结果
# TCPTestSucceeded : True  表示端口可访问
```

**解决方案**：

1. 检查服务器防火墙
2. 确认 `core.https_address` 已配置
3. 检查网络路由

#### 问题 2: 证书指纹验证失败

```powershell
# 删除现有远程配置
incus remote remove my-server

# 重新添加，接受新证书
incus remote add my-server https://<服务器IP>:8443
```

### 6.2 命令差异

| Linux 客户端 | Windows 客户端 | 说明 |
|-------------|---------------|------|
| `incus shell <instance>` | `incus exec <instance> -- bash` | Windows 不支持 shell |
| `incus list` | `incus list` | 相同 |
| `incus exec` | `incus exec` | 相同 |
| 路径 `/path/to/file` | 路径 `C:\path\to\file` | 本地路径使用 Windows 格式 |

### 6.3 路径处理

```powershell
# Windows 本地路径（文件推送）
incus file push C:\Users\User\file.txt my-server:container /tmp/

# 实例内路径（Linux 格式）
incus exec my-server:container -- cat /etc/os-release
```

### 6.4 权限问题

```powershell
# 以管理员身份运行 PowerShell
# 或者
# 检查当前用户权限
whoami /priv | findstr SeDebug
```

---

## 7. 高级配置

### 7.1 配置文件管理

```powershell
# 创建配置文件
incus profile create my-server:web-server

# 设置配置
incus profile set my-server:web-server limits.cpu=2
incus profile set my-server:web-server limits.memory=4GiB

# 添加设备
incus profile device add my-server:web-server root disk path=/ pool=local
incus profile device add my-server:web-server eth0 nic network=incusbr0

# 使用配置文件创建实例
incus launch my-server:images:ubuntu/24.04 web1 --profile web-server
```

### 7.2 网络管理

```powershell
# 列出网络
incus network list my-server:

# 查看网络详情
incus network show my-server:incusbr0

# 创建网络
incus network create my-server:incusbr1 \
    --type=bridge \
    ipv4.address=10.20.0.1/24

# 附加网络到实例
incus network attach my-server:incusbr1 my-container eth1
```

### 7.3 存储管理

```powershell
# 列出存储池
incus storage list my-server:

# 创建存储卷
incus storage volume create my-server:local data-volume size=100GiB

# 附加卷到实例
incus storage volume attach my-server:local data-volume my-container /data
```

### 7.4 集群管理

```powershell
# 列出集群成员
incus cluster list my-server:

# 查看集群组
incus cluster group list my-server:

# 在特定节点创建实例
incus launch my-server:images:ubuntu/24.04 test --target=node2
```

### 7.5 脚本化操作

**PowerShell 脚本示例**：

```powershell
# 批量创建容器
$images = @("ubuntu/24.04", "debian/12", "alpine/3.19")
$i = 1

foreach ($img in $images) {
    $name = "container-$i"
    Write-Host "Creating $name from $img..."
    incus launch my-server:images:$img $name
    $i++
}

# 批量执行命令
$instances = incus list my-server: -f csv | ConvertFrom-Csv -Header Name,State,IP,Ports

foreach ($inst in $instances) {
    if ($inst.State -eq "RUNNING") {
        Write-Host "Updating $($inst.Name)..."
        incus exec my-server:$($inst.Name) -- apt update
    }
}
```

---

## 8. 与 Linux 客户端对比

### 8.1 命令对比表

| 操作 | Linux 客户端 | Windows 客户端 |
|------|-------------|---------------|
| 连接服务器 | `incus remote add` | `incus remote add` |
| 列出实例 | `incus list` | `incus list` |
| 进入 Shell | `incus shell <name>` | `incus exec <name> -- bash` |
| 执行命令 | `incus exec <name> -- <cmd>` | `incus exec <name> -- <cmd>` |
| 文件推送 | `incus file push` | `incus file push` |
| 文件拉取 | `incus file pull` | `incus file pull` |
| VM 控制台 | `incus console <vm>` | `incus console <vm> --type=gui` |

### 8.2 路径处理对比

```powershell
# Windows 客户端推送本地文件
incus file push C:\Users\User\document.txt server:container /tmp/doc.txt

# Linux 客户端推送本地文件
incus file push /home/user/document.txt server:container /tmp/doc.txt
```

---

## 9. 最佳实践

### 9.1 安全建议

1. **使用信任令牌代替密码**：
   ```powershell
   # 服务器生成一次性令牌
   incus config trust add --type=client
   ```

2. **限制网络访问**：
   ```bash
   # 服务器上配置防火墙
   sudo ufw allow from 192.168.1.0/24 to any port 8443
   ```

3. **定期更新客户端**：
   ```powershell
   winget upgrade --id LinuxContainers.Incus
   ```

### 9.2 工作流程建议

```
Windows 开发机
     │
     ├─ PowerShell/终端 → incus CLI → 管理远程容器/VM
     │
     ├─ 远程查看器 → VM 图形控制台
     │
     └─ VS Code Remote SSH → 直接编辑容器内文件
```

### 9.3 多服务器管理

```powershell
# 添加多个远程服务器
incus remote add prod https://prod-server:8443
incus remote add staging https://staging-server:8443
incus remote add dev https://dev-server:8443

# 在不同环境操作
incus launch prod:images:ubuntu/24.04 app-prod
incus launch staging:images:ubuntu/24.04 app-staging
incus launch dev:images:ubuntu/24.04 app-dev
```

---

## 10. 参考资源

- **官方文档**: https://linuxcontainers.org/incus/docs/main/
- **Windows 客户端发布页**: https://github.com/lxc/incus/releases
- **社区论坛**: https://discuss.linuxcontainers.org/
- **镜像列表**: https://images.linuxcontainers.org/
- **WinStall 包页**: https://winstall.app/apps/LinuxContainers.Incus

---

## 附录: 快速命令参考卡

```powershell
# === 连接管理 ===
incus remote add <name> https://<ip>:8443     # 添加服务器
incus remote list                              # 列出服务器
incus remote switch <name>                     # 切换默认服务器
incus remote remove <name>                     # 删除服务器

# === 实例管理 ===
incus list <remote>:                           # 列出实例
incus launch <remote>:<image> <name>           # 创建并启动
incus init <remote>:<image> <name>             # 仅创建
incus start <remote>:<name>                    # 启动
incus stop <remote>:<name>                     # 停止
incus delete <remote>:<name>                   # 删除

# === 执行命令 ===
incus exec <remote>:<name> -- <command>        # 执行命令
incus exec <remote>:<name> -- bash             # 交互式 shell
incus console <remote>:<vm> --type=gui         # VM 图形控制台

# === 文件操作 ===
incus file push <local> <remote>:<name>:<path> # 推送文件
incus file pull <remote>:<name>:<path> <local> # 拉取文件

# === 快照 ===
incus snapshot create <remote>:<name> <snap>   # 创建快照
incus snapshot list <remote>:<name>            # 列出快照
incus snapshot restore <remote>:<name> <snap>  # 恢复快照
```

---

**文档版本**: 1.0
**更新日期**: 2026-03-13
**Incus 版本**: 6.0+
