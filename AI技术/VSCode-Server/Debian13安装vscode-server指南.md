# Debian 13 安装 VS Code Server 详细指南

> 最后更新：2026-03-12

## 目录

- [什么是 VS Code Server](#什么是-vs-code-server)
- [前置要求](#前置要求)
- [安装方法](#安装方法)
  - [方法一：使用 code-server（推荐）](#方法一使用-code-server推荐)
  - [方法二：使用 SSH 远程连接](#方法二使用-ssh-远程连接)
  - [方法三：手动安装 VS Code Server](#方法三手动安装-vs-code-server)
- [配置与优化](#配置与优化)
- [故障排除](#故障排除)
- [常见问题](#常见问题)

---

## 什么是 VS Code Server

VS Code Server 是 Visual Studio Code 的后端服务，允许你：
- 通过浏览器远程访问完整的 VS Code 环境
- 在服务器上运行代码，享受本地编辑体验
- 使用 SSH 远程开发功能

---

## 前置要求

### 系统要求
```bash
# 检查 Debian 版本
cat /etc/debian_version
cat /etc/os-release
```

### 硬件要求
- **最低配置**：1 GB RAM，1 CPU
- **推荐配置**：2 GB+ RAM，2+ CPU

### 必要组件

```bash
# 更新系统
sudo apt update && sudo apt upgrade -y

# 安装基本工具
sudo apt install -y curl wget git build-essential

# 安装 Node.js（code-server 需要）
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs

# 验证安装
node --version
npm --version
```

---

## 安装方法

### 方法一：使用 code-server（推荐）

**code-server** 是 VS Code 的开源版本，可直接在浏览器中运行。

#### 1. 使用官方安装脚本

```bash
# 下载并安装最新版本
curl -fsSL https://code-server.dev/install.sh | sh

# 或者使用 npm 安装
npm install -g code-server
```

#### 2. 配置 code-server

```bash
# 创建配置目录
mkdir -p ~/.config/code-server

# 编辑配置文件
cat > ~/.config/code-server/config.yaml << 'EOF'
bind-addr: 0.0.0.0:8080
auth: password
password: your_secure_password_here
cert: false
EOF
```

**配置说明：**
| 选项 | 说明 |
|------|------|
| `bind-addr` | 绑定地址和端口 |
| `auth` | 认证方式：`password` 或 `none` |
| `password` | 登录密码 |
| `cert` | 是否启用 HTTPS |

#### 3. 启动服务

```bash
# 启动 code-server
code-server

# 设置为系统服务（推荐）
sudo systemctl edit --full --force code-server@$USER
```

**添加以下内容到服务文件：**

```ini
[Unit]
Description=code-server
After=network.target

[Service]
Type=exec
ExecStart=/usr/bin/code-server --bind-addr 0.0.0.0:8080 --auth password
Restart=always
User=your_username
Environment=PATH=/usr/bin:/home/your_username/.local/bin

[Install]
WantedBy=default.target
```

```bash
# 启用并启动服务
sudo systemctl enable --now code-server@$USER
sudo systemctl status code-server@$USER
```

#### 4. 配置反向代理（可选）

使用 Nginx 作为反向代理：

```bash
# 安装 Nginx
sudo apt install -y nginx

# 创建配置文件
sudo nano /etc/nginx/sites-available/code-server
```

```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection upgrade;
        proxy_set_header Accept-Encoding gzip;
    }
}
```

```bash
# 启用配置
sudo ln -s /etc/nginx/sites-available/code-server /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

#### 5. 配置 HTTPS（推荐）

```bash
# 安装 Certbot
sudo apt install -y certbot python3-certbot-nginx

# 获取 SSL 证书
sudo certbot --nginx -d your-domain.com
```

---

### 方法二：使用 SSH 远程连接

这是最简单的方式，直接使用 VS Code 的 SSH Remote 扩展。

#### 1. 安装 OpenSSH Server

```bash
# 安装 SSH 服务
sudo apt install -y openssh-server

# 启动并设置开机自启
sudo systemctl enable --now ssh

# 检查状态
sudo systemctl status ssh
```

#### 2. 配置 SSH（可选优化）

```bash
# 编辑 SSH 配置
sudo nano /etc/ssh/sshd_config
```

**推荐的配置项：**

```bash
# 禁用 root 登录
PermitRootLogin no

# 禁用密码认证，仅使用密钥（更安全）
PasswordAuthentication no
PubkeyAuthentication yes

# 更改默认端口（可选）
Port 2222
```

```bash
# 重启 SSH 服务
sudo systemctl restart ssh
```

#### 3. 生成本地 SSH 密钥

在**本地机器**上执行：

```bash
# 生成 SSH 密钥对
ssh-keygen -t ed25519 -C "your_email@example.com"

# 复制公钥到服务器
ssh-copy-id username@server_ip

# 或手动复制
cat ~/.ssh/id_ed25519.pub
```

#### 4. 配置 VS Code SSH Remote

1. 安装 **Remote - SSH** 扩展
2. 按 `F1` → 输入 `Remote-SSH: Connect to Host`
3. 或创建 SSH 配置文件：

```bash
# 本地编辑 ~/.ssh/config
Host debian-server
    HostName your_server_ip
    User your_username
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
```

---

### 方法三：手动安装 VS Code Server

如果你需要官方 VS Code Server 而不是 code-server：

#### 1. 获取下载链接

```bash
# 在本地 VS Code 中查看版本
# 帮助 -> 关于 -> 版本信息

# 或从 API 获取最新版本
curl -s https://update.code.visualstudio.com/api/update/linux-x64/insider/latest
```

#### 2. 下载并安装

```bash
# 设置安装目录
export VSCODE_SERVER_DIR=$HOME/.vscode-server

# 创建目录
mkdir -p $VSCODE_SERVER_DIR/bin

# 下载 VS Code Server（替换 COMMIT_ID 为实际提交 ID）
COMMIT_ID="你的commit_id"
curl -L https://update.code.visualstudio.com/commit:${COMMIT_ID}/server-linux-x64/stable -o vscode-server.tar.gz

# 解压
mkdir -p $VSCODE_SERVER_DIR/bin/${COMMIT_ID}
tar -xzf vscode-server.tar.gz -C $VSCODE_SERVER_DIR/bin/${COMMIT_ID} --strip-components=1

# 清理
rm vscode-server.tar.gz
```

#### 3. 设置启动脚本

```bash
# 创建启动脚本
cat > $HOME/start-vscode-server.sh << 'EOF'
#!/bin/bash
VSCODE_SERVER_DIR="$HOME/.vscode-server"
COMMIT_ID="你的commit_id"
PORT=8080

cd $VSCODE_SERVER_DIR/bin/${COMMIT_ID}
./bin/code-server --host 0.0.0.0 --port ${PORT} --without-connection-token
EOF

chmod +x $HOME/start-vscode-server.sh
```

---

## 配置与优化

### 系统资源限制

```bash
# 编辑 limits
sudo nano /etc/security/limits.conf

# 添加以下内容
* soft nofile 65536
* hard nofile 65536
```

### 防火墙配置

```bash
# UFW 防火墙
sudo apt install -y ufw
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS
sudo ufw allow 8080/tcp  # code-server（可选）
sudo ufw enable
```

### 性能优化

```bash
# 增加文件监听数量
echo "fs.inotify.max_user_watches=524288" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# 交换分区优化（如果内存不足）
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

---

## 故障排除

### 问题 1：连接被拒绝

```bash
# 检查服务状态
sudo systemctl status code-server@$USER

# 检查端口是否监听
sudo netstat -tlnp | grep 8080
# 或
sudo ss -tlnp | grep 8080
```

### 问题 2：权限错误

```bash
# 修复权限问题
sudo chown -R $USER:$USER ~/.config/code-server
sudo chown -R $USER:$USER ~/.vscode-server
```

### 问题 3：依赖缺失

```bash
# 安装缺失的依赖
sudo apt install -y libx11-xcb1 libxcomposite1 libxcursor1 libxdamage1 libxi6 \
    libxtst6 libnss3 libatk-bridge2.0-0 libgtk-3-0 libgbm1
```

### 问题 4：无法访问 Web UI

```bash
# 检查防火墙
sudo ufw status

# 临时禁用防火墙测试
sudo ufw disable

# 检查服务日志
journalctl -u code-server@$USER -f
```

---

## 常见问题

### Q: code-server 和 VS Code Server 有什么区别？

**A:**
- **code-server**: 基于 VS Code 的开源版本，由 Coder 开发维护
- **VS Code Server**: 官方版本，配合 VS Code 客户端使用
- 对于大多数用户，code-server 是更好的选择

### Q: 如何更新 code-server？

```bash
# 更新到最新版本
curl -fsSL https://code-server.dev/install.sh | sh -s -- --upgrade

# 或使用 npm
npm update -g code-server
```

### Q: 可以在 Docker 中运行吗？

```bash
# 使用官方 Docker 镜像
docker run -d -p 8080:8080 \
  -e PASSWORD=your_password \
  -v ~/.config:/config \
  -v ~/.local/share/code-server:/home/coder/.local/share/code-server \
  codercom/code-server:latest
```

### Q: 如何备份配置？

```bash
# 备份 code-server 配置
tar -czf code-server-backup.tar.gz \
    ~/.config/code-server \
    ~/.local/share/code-server
```

---

## 参考资源

- [code-server 官方文档](https://coder.com/docs/code-server/latest)
- [VS Code Remote 官方文档](https://code.visualstudio.com/docs/remote/ssh)
- [Debian 官方文档](https://www.debian.org/doc/)

---

**提示**: 建议优先使用 SSH Remote 方式，体验最接近本地开发。如需 Web 访问，推荐 code-server。
