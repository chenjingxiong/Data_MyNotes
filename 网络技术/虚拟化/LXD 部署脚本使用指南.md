# LXD 部署脚本使用指南

## 概述

这是一套完整的 LXD 容器部署自动化脚本，支持快速安装和配置 LXD 环境，并通过可扩展的模块系统添加更多功能。

## 文件结构

```
scripts/
├── lxd-deploy.sh           # 主部署脚本
├── README.md               # 本文档
└── lxd-modules/            # 功能模块目录
    ├── docker.sh           # Docker 模块
    ├── openclaw.sh         # OpenClaw 模块（占位）
    ├── python.sh           # Python 开发环境
    ├── golang.sh           # Go 开发环境
    └── nodejs.sh           # Node.js 开发环境
```

## 快速开始

### 一键完整安装

```bash
# 下载并运行
chmod +x lxd-deploy.sh
./lxd-deploy.sh install
```

这将完成以下操作：
1. 检测操作系统并安装依赖
2. 安装 LXD
3. 初始化 LXD 配置
4. 安装 LXD Web UI（官方）
5. 创建 Debian 12 容器
6. 安装 code-server

### 分步安装

```bash
# 仅安装 LXD
./lxd-deploy.sh lxd

# 初始化 LXD
./lxd-deploy.sh init

# 安装 Web UI
./lxd-deploy.sh ui official    # 官方 UI
./lxd-deploy.sh ui lxdware     # LXDWARE
./lxd-deploy.sh ui mosaic      # LXDMosaic

# 创建容器
./lxd-deploy.sh container my-dev

# 安装开发工具
./lxd-deploy.sh code-server my-dev
./lxd-deploy.sh claude-code my-dev
./lxd-deploy.sh vscode my-dev
```

## 命令参考

### 主命令

| 命令 | 说明 |
|------|------|
| `install` | 完整安装和配置 |
| `lxd` | 仅安装 LXD |
| `init` | 初始化 LXD |
| `ui [type]` | 安装 Web UI |
| `container [name]` | 创建容器 |
| `code-server [name]` | 安装 code-server |
| `claude-code [name]` | 安装 Claude Code |
| `vscode [name]` | 安装 VS Code |
| `module [name]` | 加载功能模块 |
| `modules` | 列出可用模块 |
| `status` | 显示部署状态 |
| `logs` | 显示日志 |
| `help` | 显示帮助信息 |

### 选项

| 选项 | 说明 | 默认值 |
|------|------|--------|
| `--name, -n` | 容器名称 | dev-env |
| `--image, -i` | 容器镜像 | images:debian/12 |
| `--port, -p` | code-server 端口 | 8080 |
| `--ui-port` | LXD UI 端口 | 8443 |

## 使用示例

### 创建自定义容器

```bash
# 创建 Ubuntu 容器
./lxd-deploy.sh container --image images:ubuntu/22.04 ubuntu-dev

# 创建 Debian 11 容器
./lxd-deploy.sh container --image images:debian/11 debian11-dev
```

### 安装指定端口的 code-server

```bash
./lxd-deploy.sh code-server --port 9000 my-dev
```

### 使用功能模块

```bash
# 列出可用模块
./lxd-deploy.sh modules

# 加载 Docker 模块
source ./lxd-modules/docker.sh
docker_install my-dev

# 加载 Python 模块
source ./lxd-modules/python.sh
python_install my-dev

# 加载 Go 模块
source ./lxd-modules/golang.sh
golang_install my-dev

# 加载 Node.js 模块
source ./lxd-modules/nodejs.sh
nodejs_install my-dev lts
```

## 功能模块

### 内置模块

#### Docker 模块

在容器中安装 Docker 和 Docker Compose。

```bash
source ./lxd-modules/docker.sh
docker_install my-dev
```

**功能：**
- Docker Engine
- Docker CLI
- Docker Compose
- Buildx 插件
- 自动配置容器安全权限

---

#### Python 模块

完整的 Python 开发环境。

```bash
source ./lxd-modules/python.sh
python_install my-dev
```

**功能：**
- Python 3 解释器
- pip 和 venv
- pyenv 版本管理
- black、pylint、mypy
- pytest 测试框架
- ipython 交互环境

---

#### Go 模块

Go 语言开发环境。

```bash
source ./lxd-modules/golang.sh
golang_install my-dev
```

**功能：**
- Go 编译器（默认 1.21.6）
- 标准库
- GOPATH 配置

**自定义版本：**
```bash
GO_VERSION=1.20.0 golang_install my-dev
```

---

#### Node.js 模块

Node.js 开发环境。

```bash
source ./lxd-modules/nodejs.sh
nodejs_install my-dev lts
```

**功能：**
- Node.js 运行时
- npm、yarn、pnpm
- TypeScript
- 常用 CLI 工具

**版本选项：**
- `lts` - 长期支持版（默认）
- `current` - 最新版
- `20`, `18` 等具体版本号

---

#### OpenClaw 模块（占位）

这是一个占位模块，需要用户提供更多信息后实现。

```bash
source ./lxd-modules/openclaw.sh
openclaw_install my-dev
```

**当前功能：**
- 安装常用终端工具（tmux、screen、ranger 等）
- 配置 vim 和 tmux

**注意：** 请提供 "openclaw" 的具体信息以便正确实现此模块。

---

## 自定义模块

### 创建新模块

1. 在 `lxd-modules/` 目录下创建新文件 `your-module.sh`

2. 添加以下模板：

```bash
#!/usr/bin/env bash
################################################################################
# 你的模块描述
################################################################################

MODULE_NAME="your-module"
MODULE_VERSION="1.0.0"
MODULE_DESCRIPTION="模块功能描述"

your-module_install() {
    local container="${1:-dev-env}"

    log_info "[$MODULE_NAME] 安装..."

    if ! lxc list -f csv -c n | grep -q "^${container}$"; then
        log_error "容器不存在: $container"
        return 1
    fi

    # 在这里添加安装逻辑
    lxc exec "$container" -- bash -c "
        # 你的安装命令
        apt update
        apt install -y your-package
    "

    log_success "[$MODULE_NAME] 安装完成"
}

your-module_remove() {
    local container="${1:-dev-env}"
    # 卸载逻辑
}

your-module_info() {
    cat <<EOF
模块信息...
EOF
}

if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    your-module_install "$@"
fi
```

3. 使脚本可执行：
```bash
chmod +x lxd-modules/your-module.sh
```

4. 使用模块：
```bash
source ./lxd-modules/your-module.sh
your-module_install my-dev
```

## Web UI 访问

安装完成后，可以通过以下地址访问：

| 服务 | 地址 | 说明 |
|------|------|------|
| **LXD Web UI** | `https://your-server-ip:8443` | 官方 UI（需要信任证书） |
| **code-server** | `http://your-server-ip:8080` | Web 版 VS Code（默认密码: changeme） |

## 常见问题

### Q: 如何查看容器列表？

```bash
lxc list
```

### Q: 如何进入容器？

```bash
lxc exec my-dev -- bash
```

### Q: 如何停止/启动容器？

```bash
lxc stop my-dev
lxc start my-dev
```

### Q: 如何删除容器？

```bash
lxc delete my-dev
```

### Q: code-server 密码是什么？

默认密码是 `changeme`，建议在容器内修改：

```bash
lxc exec my-dev -- bash
nano ~/.config/code-server/config.yaml
# 修改 password 字段
systemctl --user restart code-server
```

### Q: 如何设置容器开机自启？

```bash
lxc config set my-dev boot.autostart true
```

## 日志和调试

### 查看部署日志

```bash
./lxd-deploy.sh logs
# 或
tail -f /var/log/lxd-deploy.log
```

### 查看 LXD 日志

```bash
journalctl -u snap.lxd.daemon.service -f
```

### 查看容器日志

```bash
lxc logs my-dev
```

## 更新和维护

### 更新 LXD

```bash
snap refresh lxd
```

### 更新脚本

```bash
git pull  # 如果使用 Git 管理
# 或重新下载
```

### 备份容器

```bash
# 创建快照
lxc snapshot my-dev backup-$(date +%Y%m%d)

# 导出容器
lxc export my-dev backup.tar.gz
```

## 参考资料

- [[LXD 云服务器部署指南]] - 完整的 LXD 部署教程
- [LXD 官方文档](https://documentation.ubuntu.com/lxd/stable/)
- [code-server 官方文档](https://coder.com/docs/code-server/install)

## 贡献

欢迎贡献新的功能模块！请遵循以下规范：

1. 模块文件名使用小写字母和连字符
2. 函数命名格式：`模块名_操作`（如 `docker_install`）
3. 包含完整的模块信息（名称、版本、描述）
4. 添加适当的日志输出
5. 支持卸载操作（可选）

## 许可证

MIT License

---

**作者：** Claudian
**创建日期：** 2026-02-08
