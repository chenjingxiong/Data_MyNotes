# ntfy 介绍与部署指南

> 简单的 HTTP-based pub-sub 通知服务，向手机或桌面发送推送通知

## 📋 目录
- [什么是 ntfy](#什么是-ntfy)
- [核心特性](#核心特性)
- [快速体验](#快速体验)
- [服务器部署](#服务器部署)
- [手机客户端配置](#手机客户端配置)
- [Web 端使用](#web-端使用)
- [高级配置](#高级配置)
- [API 参考](#api-参考)
- [故障排除](#故障排除)

---

## 什么是 ntfy

**ntfy** (发音为 "*notify*") 是一个简单的基于 HTTP 的发布-订阅 (pub-sub) 通知服务。它允许你通过脚本从任何计算机向手机或桌面发送通知，**无需注册或支付任何费用**。

### 项目信息
| 项目 | 信息 |
|------|------|
| **GitHub** | [binwiederhier/ntfy](https://github.com/binwiederhier/ntfy) |
| **官网** | https://ntfy.sh |
| **语言** | Go |
| **许可证** | Apache 2.0 |
| **Stars** | 29,000+ |

### 核心定位
- 📱 **简单易用**: 通过 HTTP PUT/POST 发送通知
- 🔓 **开放免费**: 免费云端服务，支持自建
- 🌐 **跨平台**: 支持 Android、iOS、Web、CLI
- 🔌 **API 友好**: curl 即可使用，无需复杂 SDK

---

## 核心特性

### 功能对比

| 特性 | 免费版 | Pro 版 | 自建服务器 |
|------|--------|--------|------------|
| 基本推送 | ✅ | ✅ | ✅ |
| 消息保留 | 24 小时 | 永久 | 可配置 |
| 附件 | 不支持 | ✅ | 可配置 |
| 主题数量 | 无限 | 无限 | 无限 |
| 费用 | 免费 | $5/月起 | 服务器费用 |

### 主要特性
- **即时推送**: 通过 HTTP 请求实时推送通知
- **主题订阅**: 支持任意主题名称，无需预先创建
- **优先级控制**: 支持不同优先级和通知类型
- **附件支持**: 支持 URL 附件或直接上传
- **用户认证**: 支持访问控制和用户管理
- **Web 界面**: 内置美观的 Web 管理界面

---

## 快速体验

### 使用 curl 发送通知

最简单的方式，使用免费的 ntfy.sh 服务：

```bash
# 发送简单文本
curl -d "Hello from ntfy!" ntfy.sh/my_topic

# 发送带标题的通知
curl -H "Title: Alert" -d "Server is down!" ntfy.sh/alerts

# 发送带优先级的紧急通知
curl -H "Priority: 5" -H "Tags: warning,skull" \
  -d "Production server is on fire!" ntfy.sh/emergency

# 发送带附件的通知
curl -H "Attach: https://example.com/image.jpg" \
  -d "Check out this image" ntfy.sh/pics
```

### 使用 ntfy CLI

```bash
# 安装 CLI（见服务器部署章节）
ntfy publish my_topic "Hello from ntfy CLI!"

# 发布带标题的通知
ntfy publish my_topic --title "Alert" "Server is down!"

# 发布带优先级和标签的通知
ntfy publish alerts --priority 5 --tags warning,skull \
  "Production server is on fire!"
```

---

## 服务器部署

### 方式一：使用 Docker（推荐）

#### 快速启动

```bash
# 基础运行（无持久化）
docker run -p 80:80 -it binwiederhier/ntfy serve

# 带持久化缓存
docker run \
  -v /var/cache/ntfy:/var/cache/ntfy \
  -p 80:80 \
  -it \
  binwiederhier/ntfy \
    serve \
    --cache-file /var/cache/ntfy/cache.db

# 带配置文件和时区
docker run \
  -v /etc/ntfy:/etc/ntfy \
  -e TZ=Asia/Shanghai \
  -p 80:80 \
  -u 1000:1000 \
  -it \
  binwiederhier/ntfy \
  serve
```

#### Docker Compose 部署

创建 `docker-compose.yml`:

```yaml
version: '3.8'

services:
  ntfy:
    image: binwiederhier/ntfy
    container_name: ntfy
    restart: unless-stopped
    command:
      - serve
    environment:
      - TZ=Asia/Shanghai
    user: "1000:1000"  # 替换为你的 UID/GID
    volumes:
      - /var/cache/ntfy:/var/cache/ntfy
      - /etc/ntfy:/etc/ntfy
    ports:
      - "80:80"
    healthcheck:
      test: ["CMD-SHELL", "wget -q --tries=1 http://localhost:80/v1/health -O - | grep -Eo '\"healthy\"\\s*:\\s*true' || exit 1"]
      interval: 60s
      timeout: 10s
      retries: 3
      start_period: 40s
```

启动服务：
```bash
docker-compose up -d
```

### 方式二：使用 Debian/Ubuntu 仓库

```bash
# 添加 GPG 密钥
sudo mkdir -p /etc/apt/keyrings
sudo curl -L -o /etc/apt/keyrings/ntfy.gpg https://archive.ntfy.sh/apt/keyring.gpg

# 安装 HTTPS 传输
sudo apt install apt-transport-https

# 添加仓库（选择你的架构）
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/ntfy.gpg] https://archive.ntfy.sh/apt stable main" \
    | sudo tee /etc/apt/sources.list.d/ntfy.list

# 更新并安装
sudo apt update
sudo apt install ntfy

# 启动服务
sudo systemctl enable ntfy
sudo systemctl start ntfy
```

### 方式三：使用二进制文件

```bash
# 下载最新版本
wget https://github.com/binwiederhier/ntfy/releases/download/v2.19.2/ntfy_2.19.2_linux_amd64.tar.gz

# 解压
tar zxvf ntfy_2.19.2_linux_amd64.tar.gz

# 安装
sudo cp -a ntfy_2.19.2_linux_amd64/ntfy /usr/local/bin/ntfy

# 复制配置文件
sudo mkdir -p /etc/ntfy
sudo cp ntfy_2.19.2_linux_amd64/{client,server}/*.yml /etc/ntfy

# 运行服务
sudo ntfy serve
```

### 方式四：Kubernetes 部署

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ntfy
spec:
  selector:
    matchLabels:
      app: ntfy
  template:
    metadata:
      labels:
        app: ntfy
    spec:
      containers:
      - name: ntfy
        image: binwiederhier/ntfy
        args: ["serve"]
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 80
          name: http
        volumeMounts:
        - name: cache
          mountPath: "/var/cache/ntfy"
        - name: config
          mountPath: "/etc/ntfy"
          readOnly: true
      volumes:
      - name: cache
        emptyDir: {}
      - name: config
        configMap:
          name: ntfy
---
apiVersion: v1
kind: Service
metadata:
  name: ntfy
spec:
  selector:
    app: ntfy
  ports:
  - port: 80
    targetPort: 80
```

### 配置文件说明

编辑 `/etc/ntfy/server.yml`:

```yaml
# 基础服务器配置
base-url: "https://your-domain.com"
listen-http: ":80"

# 缓存配置
cache-file: "/var/cache/ntfy/cache.db"
cache-duration: "24h"

# 消息保留
keepalive-interval: "30s"
message-delay: "10s"

# 访问控制（可选）
auth-file: "/var/cache/ntfy/user.db"
auth-default-access: "read-write"

# 附件配置
attachment-cache-dir: "/var/cache/ntfy/attachments"
enable-resize: true
```

---

## 手机客户端配置

### Android 客户端

#### 下载安装

- **Google Play**: [https://play.google.com/store/apps/details?id=io.heckel.ntfy](https://play.google.com/store/apps/details?id=io.heckel.ntfy)
- **F-Droid**: [https://f-droid.org/en/packages/io.heckel.ntfy/](https://f-droid.org/en/packages/io.heckel.ntfy/)
- **GitHub**: [https://github.com/binwiederhier/ntfy-android](https://github.com/binwiederhier/ntfy-android)

#### 配置自建服务器

1. 打开应用
2. 点击右上角 ⋮ 菜单
3. 选择 **Settings** → **Server**
4. 输入你的服务器地址：
   - 免费服务: `https://ntfy.sh`
   - 自建服务: `http://your-server-ip` 或 `https://your-domain.com`
5. 点击 **Save**

#### 订阅主题

1. 点击主界面 **+** 按钮
2. 输入主题名称（如 `my_topic`、`alerts`）
3. 点击 **Subscribe**
4. 现在你将收到该主题的所有通知

### iOS 客户端

#### 下载安装

- **App Store**: [https://apps.apple.com/us/app/ntfy/id1625396347](https://apps.apple.com/us/app/ntfy/id1625396347)
- **GitHub**: [https://github.com/binwiederhier/ntfy-ios](https://github.com/binwiederhier/ntfy-ios)

#### 配置步骤（与 Android 类似）

1. 下载并打开应用
2. 点击 **Settings**
3. 在 **Base URL** 中输入服务器地址
4. 点击 **+** 订阅主题

---

## Web 端使用

### 访问 Web 界面

浏览器访问你的服务器地址：
- 免费服务: https://ntfy.sh
- 自建服务: http://your-server-ip

### Web 界面功能

```
┌─────────────────────────────────────────────┐
│  ntfy.sh                    [Settings] [?]  │
├─────────────────────────────────────────────┤
│                                             │
│  ┌───────────────────────────────────────┐ │
│  │ Subscribe to a topic                 │ │
│  │ ┌─────────────────────────────────┐   │ │
│  │ │ my_topic                        │   │ │
│  │ └─────────────────────────────────┘   │ │
│  │        [Subscribe]                    │ │
│  └───────────────────────────────────────┘ │
│                                             │
│  My Topics:                      All Topics │
│  ┌───────────────────────────────────────┐ │
│  │ 📱 my_topic                    [×]    │ │
│  │ 🔔 alerts                      [×]    │ │
│  └───────────────────────────────────────┘ │
│                                             │
│  Notifications:                              │
│  ┌───────────────────────────────────────┐ │
│  │ 🔔 Alert                              │ │
│  │ Server is down!              2m ago   │ │
│  └───────────────────────────────────────┘ │
└─────────────────────────────────────────────┘
```

### 发布消息

在 Web 界面中：

1. 点击主题名称
2. 在输入框中输入消息
3. 可选：设置标题、优先级、标签
4. 点击 **Send**

---

## 高级配置

### 消息优先级

| 优先级 | 值 | 行为 |
|--------|-----|------|
| Max | 5 | 立即推送，绕过静音模式 |
| High | 4 | 高优先级通知 |
| Default | 3 | 默认行为 |
| Low | 2 | 低优先级 |
| Min | 1 | 最低优先级，可能被聚合 |

```bash
# 高优先级
curl -H "Priority: 4" -d "Important update" ntfy.sh/updates

# 紧急通知
curl -H "Priority: 5" -H "Tags: warning,skull" \
  -d "Critical alert" ntfy.sh/emergency
```

### 使用标签（Tags）

```bash
# 使用表情符号或图标
curl -H "Tags: warning,server,backup" \
  -d "Backup failed" ntfy.sh/sysadmin

# 支持的标签
# 🚀 rocket, ⚠️ warning, 📦 package, 🔥 fire, 🧠 skull
# 📱 phone, 💡 bulb, 🔒 lock, 🍕 pizza, etc.
```

### 消息标题

```bash
# 单行标题
curl -H "Title: Alert" -d "Message body" ntfy.sh/topic

# 多行标题（邮件样式）
curl -H "Title: Subject line" \
  -d "First line\nSecond line\nThird line" ntfy.sh/topic
```

### 附件支持

```bash
# URL 附件
curl -H "Attach: https://example.com/image.jpg" \
  -d "Check this image" ntfy.sh/pics

# 直接上传（需要服务器支持）
curl -T /path/to/file.jpg ntfy.sh/files

# 设置附件名称
curl -H "Attach: https://example.com/doc.pdf;name=report.pdf" \
  -d "Monthly report" ntfy.sh/reports
```

### 访问控制

创建受保护的主题：

```bash
# 创建用户和权限文件
ntfy user add admin --role admin
ntfy access my_topic admin --role admin
ntfy access my_topic everyone --read-only

# 使用认证发布
curl -u admin:password -d "Secret message" \
  https://your-domain.com/my_topic
```

### 邮件集成

```bash
# 通过邮件网关发送
curl -H "Email: yes@example.com" \
  -d "Email notification" ntfy.sh/emails
```

---

## API 参考

### 发布消息

```bash
# PUT 请求
curl -X PUT -d "Message" ntfy.sh/topic

# POST 请求
curl -X POST -d "Message" ntfy.sh/topic

# 带所有选项
curl -X POST \
  -H "Title: Alert" \
  -H "Priority: 5" \
  -H "Tags: warning,skull" \
  -H "Delay: 30s" \
  -d "Urgent message" \
  ntfy.sh/emergency
```

### 订阅主题（WebSocket）

```javascript
// JavaScript 示例
const socket = new WebSocket('wss://ntfy.sh/my_topic/ws');

socket.onmessage = (event) => {
  const message = JSON.parse(event.data);
  console.log(message.message);
};

// Python 示例
import asyncio
import websockets

async def subscribe():
    async with websockets.connect('wss://ntfy.sh/my_topic/ws') as ws:
        while True:
            message = await ws.recv()
            print(message)

asyncio.run(subscribe())
```

### JSON 格式发布

```bash
# 发送 JSON 格式消息
curl -H "Content-Type: application/json" \
  -d '{"topic":"my_topic","message":"Hello","title":"Greeting","priority":3}' \
  ntfy.sh
```

---

## 故障排除

### 连接问题

```bash
# 检查服务状态
systemctl status ntfy

# 检查端口是否监听
netstat -tulpn | grep :80

# 检查防火墙
sudo ufw allow 80/tcp
sudo ufw status
```

### Docker 故障

```bash
# 查看日志
docker logs ntfy

# 进入容器调试
docker exec -it ntfy sh

# 重建容器
docker-compose down
docker-compose up -d --build
```

### 消息未送达

1. **检查手机通知权限**
   - 确保应用有通知权限
   - 检查省电模式设置

2. **验证主题名称**
   ```bash
   # 测试订阅
   ntfy subscribe my_topic
   ```

3. **检查服务器日志**
   ```bash
   journalctl -u ntfy -f
   ```

### 性能优化

```yaml
# server.yml 优化配置
cache-duration: "12h"          # 减少缓存时间
message-delay: "5s"            # 批量发送
keepalive-interval: "45s"      # 减少心跳频率
```

---

## 实用场景

### 场景 1：服务器监控通知

```bash
# 磁盘空间告警
df -h | awk '$5 > 90 {print $1 " is " $5 " full"}' | \
  xargs -I {} curl -H "Title: Disk Alert" -H "Priority: 4" \
  -d "{}" ntfy.sh/server-alerts
```

### 场景 2：脚本完成通知

```bash
# 长时间任务完成后通知
long-running-script && \
  curl -H "Title: Script Complete" \
    -d "Backup finished successfully" ntfy.sh/backup-status
```

### 场景 3：温度监控

```bash
# 树莓派温度监控
/opt/vc/bin/vcgencmd measure_temp | \
  awk -F= '{print $2}' | \
  awk '{if ($1 > 70) print "CPU temp: " $1 "°C"}' | \
  xargs -I {} curl -H "Priority: 5" -d "{}" ntfy.sh/pi-temp
```

### 场景 4：CI/CD 通知

```yaml
# GitHub Actions 示例
- name: Notify on failure
  if: failure()
  run: |
    curl -H "Title: Build Failed" \
      -H "Priority: 4" \
      -d "Build ${{ github.run_number }} failed" \
      ntfy.sh/ci-notifications
```

---

## 参考资源

### 官方资源
- **官网**: https://ntfy.sh
- **GitHub 仓库**: https://github.com/binwiederhier/ntfy
- **API 文档**: https://ntfy.sh/docs/
- **Docker Hub**: https://hub.docker.com/r/binwiederhier/ntfy

### 移动应用
- **Android**: https://github.com/binwiederhier/ntfy-android
- **iOS**: https://github.com/binwiederhier/ntfy-ios

### 社区
- **Discord**: https://discord.gg/cT7ECsZj9w
- **Matrix**: https://matrix.to/#/#ntfy:matrix.org
- **GitHub Issues**: https://github.com/binwiederhier/ntfy/issues

---

## 总结

ntfy 是一个简单而强大的通知服务，适合以下场景：

| 使用场景 | 推荐方案 |
|----------|----------|
| 个人测试 | 免费云服务 ntfy.sh |
| 企业部署 | 自建服务器 + 认证 |
| IoT 设备 | 自建轻量服务器 |
| 监控告警 | CLI + 优先级标签 |
| 移动推送 | 官方手机 App |

### 选择 ntfy 的理由

✅ **零依赖**: curl 即可使用
✅ **跨平台**: 全平台支持
✅ **开源**: Apache 2.0 许可
✅ **灵活**: 自建或云服务
✅ **免费**: 云服务永久免费基础版

---

**Happy Notifications!** 🔔

Generated with [Claude Code](https://claude.com/claude-code)
via [Happy](https://happy.engineering)
