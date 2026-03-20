# Happy 远程控制指南

> 使用 Claude Code Remote Control 从手机或其他设备远程控制本地开发环境

## 简介

Happy 是基于 Claude Code Remote Control 功能的远程控制方案。它允许你：
- 从手机、平板或任何浏览器继续本地 Claude Code 会话
- 在办公桌上启动任务，然后从沙发上的手机继续工作
- 远程使用完整的本地环境（文件系统、MCP servers、工具）
- 多设备同步对话

## 要求

### 订阅要求
- **可用计划**: Pro、Max、Team 和 Enterprise
- Team 和 Enterprise 管理员需先在管理员设置中启用 Claude Code
- 不支持 API 密钥

### 环境准备
```bash
# 1. 登录 Claude 账户
claude
/login

# 2. 在项目目录中运行一次 claude 以接受工作区信任
cd /path/to/your/project
claude
```

## 快速开始

### 方式一：启动新的 Remote Control 会话

```bash
# 进入你的项目目录
cd /path/to/your/project

# 启动远程控制会话
claude remote-control

# 或者指定会话名称
claude remote-control "我的项目"
```

终端会显示：
- 会话 URL
- QR 码（按空格键切换显示）

### 方式二：继续现有会话

如果你已经在 Claude Code 会话中：

```
/remote-control
# 或简写
/rc

# 指定会话名称
/remote-control "我的项目"
```

## 从手机连接

### 方法 1：扫描 QR 码

1. 在终端按空格键显示 QR 码
2. 用手机扫描 QR 码
3. 自动在 Claude 应用中打开会话

### 方法 2：使用会话 URL

1. 复制终端中显示的会话 URL
2. 在手机浏览器中打开
3. 跳转到 claude.ai/code 或 Claude 应用

### 方法 3：从应用内查找

1. 打开 Claude 应用
2. 在会话列表中查找你的会话（带有绿色电脑图标）
3. 点击连接

### 下载 Claude 应用

如果没有 Claude 应用，在 Claude Code 中运行：
```
/mobile
```
会显示 iOS 和 Android 的下载 QR 码。

## 命令选项

```bash
claude remote-control [选项] [名称]

# 选项：
--name "项目名称"     # 设置自定义会话标题
--verbose             # 显示详细日志
--sandbox             # 启用沙箱（文件系统和网络隔离）
--no-sandbox          # 禁用沙箱
```

## 自动启用 Remote Control

如果你希望每次会话都自动启用 Remote Control：

```
/config
```

将 **"为所有会话启用 Remote Control"** 设置为 `true`。

## 工作原理

```
┌─────────────┐          HTTPS           ┌──────────────┐
│   手机设备   │ ◄──────────────────────► │ Anthropic API │
│  Claude App │                          │   中继服务器   │
└─────────────┘          HTTPS           └──────┬───────┘
                                              │
                                              │ HTTPS 出站
                                              ▼
                                     ┌─────────────────┐
                                     │  本地机器        │
                                     │  Claude Code    │
                                     │  (终端运行)     │
                                     └─────────────────┘
```

### 安全特性
- ✅ 本地会话仅发出出站 HTTPS 请求
- ✅ 不在机器上打开入站端口
- ✅ 所有流量通过 TLS 加密
- ✅ 使用短期凭证，独立过期

## 使用场景

### 场景 1：代码审查
```bash
# 在电脑上启动
claude remote-control "代码审查"
# 用手机在通勤路上审查代码
```

### 场景 2：长时间任务监控
```bash
# 启动编译/测试
npm run build

# 启动远程控制
/rc

# 用手机监控进度，外出时也能查看
```

### 场景 3：快速修复
```bash
# 收到 bug 报告，用手机快速查看
claude remote-control "Bug修复"

# 通过手机语音输入描述问题
# Claude 可以读取本地代码并提供解决方案
```

## 限制

| 限制 | 说明 |
|------|------|
| 同时连接数 | 每个会话仅支持一个远程连接 |
| 终端状态 | 终端必须保持打开，关闭即结束会话 |
| 网络中断 | 网络中断超过约 10 分钟会超时 |
| 多实例 | 多个 Claude Code 实例各自独立 |

## 故障排除

### 连接超时
```bash
# 重新启动会话
claude remote-control
```

### 无法扫描 QR 码
```bash
# 按空格键切换 QR 码显示
# 或直接复制 URL 到手机浏览器
```

### 找不到会话
```bash
# 检查会话列表
# Remote Control 会话显示绿色电脑图标
```

## 与 Claude Code on the Web 的区别

| 特性 | Remote Control | Claude Code on the Web |
|------|----------------|------------------------|
| 运行位置 | 本地机器 | 云基础设施 |
| 文件访问 | 本地文件系统 | 云端/上传 |
| MCP Servers | 可用 | 不可用 |
| 使用场景 | 继续本地工作 | 快速启动新任务 |

## 自建服务器部署

如果你希望完全自主控制远程连接，可以部署自建的 Happy 服务器。

### 架构说明

```
┌─────────────┐          HTTPS           ┌──────────────┐
│   手机设备   │ ◄──────────────────────► │ 自建服务器   │
│  Claude App │    (公网IP/域名)          │  (Happy)     │
└─────────────┘          HTTPS           └──────┬───────┘
                                              │
                                              │ HTTPS 出站
                                              ▼
                                     ┌─────────────────┐
                                     │  本地机器        │
                                     │  Claude Code    │
                                     │  (终端运行)     │
                                     └─────────────────┘
```

### 部署方式一：使用 Docker（推荐）

这是最简单的部署方式，只需一条命令。

#### 1. 服务器准备

```bash
# 准备一台具有公网IP的服务器
# 推荐配置：1核2G内存以上
# 系统：Ubuntu 20.04+ / Debian 11+ / CentOS 8+

# 安装 Docker
curl -fsSL https://get.docker.com | sh
```

#### 2. 启动 Happy 服务器

```bash
# 创建数据目录
mkdir -p /opt/happy/data

# 启动服务器（自动获取SSL证书）
docker run -d \
  --name happy-server \
  -p 80:80 \
  -p 443:443 \
  -v /opt/happy/data:/app/data \
  -e DOMAIN=your-domain.com \
  -e EMAIL=your@email.com \
  happyengineering/happy-server:latest

# 查看日志
docker logs -f happy-server
```

#### 3. 配置域名

```bash
# 将你的域名 A 记录指向服务器公网IP
# 例如：
# A    happy.yourdomain.com    ->    1.2.3.4
```

### 部署方式二：使用 Docker Compose

```bash
# 创建 docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'
services:
  happy:
    image: happyengineering/happy-server:latest
    container_name: happy-server
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./data:/app/data
    environment:
      - DOMAIN=your-domain.com
      - EMAIL=your@email.com
      - MAX_TUNNELS=100
      - AUTH_TOKEN=your-secret-token
    restart: unless-stopped
EOF

# 启动
docker-compose up -d
```

### 部署方式三：使用 systemd（二进制）

```bash
# 下载最新版本
wget https://github.com/happy-engineering/happy/releases/latest/download/happy-server-linux-amd64
chmod +x happy-server-linux-amd64
sudo mv happy-server-linux-amd64 /usr/local/bin/happy-server

# 创建配置文件
sudo mkdir -p /etc/happy
cat > /etc/happy/config.yaml << 'EOF'
server:
  domain: your-domain.com
  email: your@email.com
  http_port: 80
  https_port: 443
  max_tunnels: 100

tunnels:
  enabled: true
  auth_token: your-secret-token
EOF

# 创建 systemd 服务
cat > /etc/systemd/system/happy.service << 'EOF'
[Unit]
Description=Happy Server
After=network.target

[Service]
Type=simple
User=nobody
ExecStart=/usr/local/bin/happy-server -config /etc/happy/config.yaml
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

# 启动服务
sudo systemctl daemon-reload
sudo systemctl enable happy
sudo systemctl start happy
```

### 客户端连接

#### 使用自建服务器

```bash
# 设置自定义服务器
claude config set happy.server https://your-domain.com

# 启动远程控制
claude remote-control

# 或者使用环境变量
export HAPPY_SERVER_URL=https://your-domain.com
claude remote-control
```

### 防火墙配置

```bash
# Ubuntu/Debian
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# CentOS/RHEL
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --reload
```

### 监控和维护

```bash
# 查看服务器状态
docker ps
docker logs happy-server

# 重启服务
docker restart happy-server

# 更新到最新版本
docker pull happyengineering/happy-server:latest
docker stop happy-server
docker rm happy-server
# 然后重新运行启动命令
```

### 故障排除

#### SSL 证书获取失败

```bash
# 检查域名解析是否生效
nslookup your-domain.com

# 检查80端口是否被占用
sudo netstat -tulpn | grep :80

# 查看详细日志
docker logs happy-server --tail 100
```

#### 连接超时

```bash
# 检查防火墙
sudo ufw status

# 检查服务器是否正常运行
curl https://your-domain.com/health
```

## 自建 vs 云端对比

| 特性 | 自建服务器 | 官方云端 |
|------|-----------|---------|
| 数据隐私 | 完全自主 | 第三方托管 |
| 部署难度 | 需要公网服务器 | 零配置 |
| 维护成本 | 需要维护 | 无需维护 |
| SSL证书 | 自动获取 | 自动配置 |
| 成本 | 服务器费用 | 免费/付费订阅 |

## 参考资源

- [Claude Code 文档](https://code.claude.com/docs/zh-CN/remote-control)
- [Claude Code on the Web](https://claude.ai/code)
- [CLI 参考手册](https://code.claude.com/docs/zh-CN/cli-reference)
- [Happy GitHub](https://github.com/happy-engineering/happy)
- [Docker 安装指南](https://docs.docker.com/engine/install/)

---

**Happy Coding!** 🚀

Generated with [Claude Code](https://claude.com/claude-code)
via [Happy](https://happy.engineering)
