# 1Panel 部署 OpenClaw 完整教程

> 本教程详细介绍如何在 1Panel 上部署 OpenClaw，连接飞书，并安装 Claude Code 和各种 Skills。

---

## 📋 目录

1. [环境准备](#环境准备)
2. [1Panel 安装与配置](#1panel-安装与配置)
3. [OpenClaw Docker 部署](#openclaw-docker-部署)
4. [飞书应用配置](#飞书应用配置)
5. [OpenClaw 飞书连接](#openclaw-飞书连接)
6. [Claude Code 安装配置](#claude-code-安装配置)
7. [Skills 安装指南](#skills-安装指南)
8. [常见问题与故障排查](#常见问题与故障排查)

---

## 环境准备

### 系统要求

- **操作系统**: Linux (推荐 Ubuntu 20.04+/CentOS 8+/Debian 11+)
- **内存**: 最低 2GB，推荐 4GB+
- **存储**: 最低 20GB 可用空间
- **网络**: 需要访问公网（用于拉取 Docker 镜像）

### 安装 Docker

```bash
# 安装 Docker
curl -fsSL https://get.docker.com | sh

# 启动 Docker 服务
sudo systemctl start docker
sudo systemctl enable docker

# 验证安装
docker --version
```

---

## 1Panel 安装与配置

### 1. 安装 1Panel

```bash
# 使用一键安装脚本
curl -sSL https://resource.fit2cloud.com/1panel/package/quick_start.sh -o quick_start.sh && sh quick_start.sh
```

安装完成后会显示：
- 访问地址（通常是 `http://服务器IP:10086`）
- 默认用户名和密码（首次登录后请修改）

### 2. 登录 1Panel

1. 浏览器访问 `http://<服务器IP>:10086`
2. 使用安装时提供的信息登录
3. **建议**: 首次登录后立即修改密码

### 3. 配置 Docker 镜像加速

进入 **容器 > 设置 > 镜像加速**，添加国内镜像源：

```json
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://docker.1panel.live"
  ]
}
```

---

## OpenClaw Docker 部署

### 方法一：使用 1Panel 容器管理

1. 进入 **容器 > 镜像**
2. 点击 **获取镜像**，输入镜像名称：
   ```
   justlovemaki/openclaw-docker-cn-im:latest
   ```
3. 等待镜像下载完成

### 方法二：使用 Docker Compose（推荐）

1. 进入 **容器 > 编排**
2. 点击 **创建编排**，输入以下内容：

```yaml
version: '3.8'

services:
  openclaw:
    image: justlovemaki/openclaw-docker-cn-im:latest
    container_name: openclaw
    restart: unless-stopped
    ports:
      - "8080:8080"
    environment:
      # API Keys（根据需要配置）
      - ANTHROPIC_API_KEY=your_api_key_here
      - OPENAI_API_KEY=your_openai_key_here
    volumes:
      # 数据持久化
      - openclaw_data:/app/data
      # 配置文件
      - ./openclaw_config:/app/config
    networks:
      - openclaw_network

volumes:
  openclaw_data:
    driver: local

networks:
  openclaw_network:
    driver: bridge
```

3. 点击 **确认** 创建编排

### 启动 OpenClaw

在编排列表中，找到刚创建的 OpenClaw 编排，点击 **启动**。

---

## 飞书应用配置

### 1. 创建飞书自建应用

1. 访问 [飞书开放平台](https://open.feishu.cn/)
2. 登录后进入 **创建企业自建应用**
3. 填写应用信息：
   - **应用名称**: OpenClaw AI 助手
   - **应用描述**: 你的 AI 助手机器人
   - **应用图标**: 可上传自定义图标

### 2. 获取凭证

创建完成后，进入 **凭证与基础信息** 页面，记录：
- **App ID**: `cli_xxxxxxxxxxxxx`
- **App Secret**: `xxxxxxxxxxxxxxxxxxxxx`

### 3. 配置权限

在 **权限管理** 中申请以下权限：

| 权限名称 | 权限范围 |
|---------|---------|
| 获取与发送消息 | 单聊、群聊 |
| 接收群组中机器人消息 | 已加入的群 |
| 获取群组信息 | 已加入的群 |
| 获取用户信息 | 联系人 |
| 获取用户 email | 联系人 |

### 4. 配置事件订阅

1. 进入 **事件订阅**
2. 添加以下事件：
   - `im.message.receive_v1` - 接收消息
3. 设置请求 URL：
   ```
   http://<服务器IP>:8080/feishu/events
   ```
   - 需要先启动 OpenClaw 容器才能验证成功

### 5. 发布应用

配置完成后，在 **版本管理与发布** 中：
1. 创建新版本
2. 点击 **申请发布**
3. 等待审核通过（通常几分钟内完成）

---

## OpenClaw 飞书连接

### 1. 配置环境变量

编辑 OpenClaw 容器的环境变量：

```bash
# 进入容器配置
cd /opt/1panel/apps/openclaw/openclaw_config
```

创建或编辑 `feishu.yaml`：

```yaml
feishu:
  # 飞书应用凭证
  app_id: "cli_xxxxxxxxxxxxx"
  app_secret: "xxxxxxxxxxxxxxxxxxxxx"

  # 加密密钥（在飞书开放平台获取）
  encrypt_key: "xxxxxxxxxxxxxxxxxxxxx"

  # 验证令牌（自定义）
  verification_token: "your_verification_token"

  # 事件订阅配置
  event:
    enabled: true
    message_receive: true

  # 功能开关
  features:
    # 群聊功能
    group_chat: true
    # 私聊功能
    private_chat: true
    # @ 提醒功能
    mention: true
```

### 2. 重启容器

```bash
# 在 1Panel 中
# 容器 > 找到 openclaw 容器 > 重启
```

或在命令行：

```bash
docker restart openclaw
```

### 3. 验证连接

在飞书中搜索你的应用名称，发送测试消息：

```
/echo 你好
```

如果 OpenClaw 回复，说明连接成功！

---

## Claude Code 安装配置

### 1. 获取 API Key

1. 访问 [Anthropic Console](https://console.anthropic.com/)
2. 注册/登录账号
3. 在 API Keys 页面创建新的 API Key
4. 复制保存该 Key（仅显示一次）

### 2. 配置环境变量

在 OpenClaw 容器中添加 Claude API Key：

**方式一：通过 1Panel 界面**

1. 进入 **容器 > 容器**
2. 找到 `openclaw` 容器
3. 点击 **编辑** > **环境变量**
4. 添加：
   - 名称: `ANTHROPIC_API_KEY`
   - 值: `sk-ant-xxxxxxxxxxxxxxxx`

**方式二：编辑 docker-compose.yml**

```yaml
environment:
  - ANTHROPIC_API_KEY=sk-ant-xxxxxxxxxxxxxxxx
  - OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxx  # 可选
```

### 3. 配置 Claude Code 插件

进入 OpenClaw 配置目录：

```bash
# 进入容器
docker exec -it openclaw bash

# 编辑配置
vi /app/config/claude.yaml
```

添加配置：

```yaml
claude:
  # API 配置
  api:
    base_url: "https://api.anthropic.com"
    api_key: "${ANTHROPIC_API_KEY}"
    model: "claude-3-5-sonnet-20241022"
    max_tokens: 8192

  # 功能配置
  features:
    # 代码执行
    code_execution: true
    # 文件操作
    file_operations: true
    # 浏览器访问
    web_access: true

  # 安全限制
  security:
    # 允许的命令
    allowed_commands:
      - "ls"
      - "cat"
      - "grep"
      - "vi"
    # 禁止的路径
    forbidden_paths:
      - "/etc"
      - "/root/.ssh"
```

### 4. 重启使配置生效

```bash
docker restart openclaw
```

---

## Skills 安装指南

OpenClaw 支持通过 Skills 扩展功能。以下是常用 Skills 的安装方法。

### 1. 内置 Skills

OpenClaw-CN-IM 镜像已预装以下 Skills：

| Skill 名称 | 功能描述 |
|-----------|---------|
| `algorithmic-art` | 使用 p5.js 创建算法艺术 |
| `docx` | Word 文档创建和编辑 |
| `pdf` | PDF 文件处理 |
| `xlsx` | Excel 表格操作 |
| `pptx` | PowerPoint 演示文稿 |
| `frontend-design` | 前端界面设计 |
| `webapp-testing` | 使用 Playwright 测试 Web 应用 |

### 2. 启用 Skills

编辑 Skills 配置文件：

```bash
# 进入容器
docker exec -it openclaw bash

# 编辑 skills 配置
vi /app/config/skills.yaml
```

```yaml
skills:
  # 已启用的 Skills
  enabled:
    - algorithmic-art
    - docx
    - pdf
    - xlsx
    - pptx
    - frontend-design
    - webapp-testing

  # Skill 配置
  settings:
    algorithmic-art:
      max_iterations: 1000

    docx:
      default_font: "微软雅黑"
      default_size: 11

    pdf:
      ocr_enabled: true

    xlsx:
      max_rows: 10000

    frontend-design:
      output_dir: "/app/outputs"
```

### 3. 安装自定义 Skills

如需安装额外的 Skills：

```bash
# 进入容器
docker exec -it openclaw bash

# 安装 Skill
npm install @openclaw/skill-name

# 或从 GitHub 安装
git clone https://github.com/user/skill-repo.git /app/skills/skill-name
```

然后在 `skills.yaml` 中添加：

```yaml
skills:
  enabled:
    - your-custom-skill
```

### 4. 验证 Skills 安装

在飞书中发送：

```
/skills list
```

应该返回所有已安装的 Skills 列表。

---

## 常见问题与故障排查

### Q1: 飞书无法接收消息

**原因**：事件订阅未验证通过或权限未开启

**解决方案**：
1. 检查飞书开放平台的事件订阅 URL 是否可访问
2. 确保服务器防火墙开放 8080 端口
3. 检查权限管理中所有必要权限已申请并批准
4. 确保应用已发布

### Q2: Claude Code 返回认证错误

**原因**：API Key 无效或未配置

**解决方案**：
1. 检查 `ANTHROPIC_API_KEY` 环境变量是否正确
2. 验证 API Key 是否有效且未过期
3. 确认账户有足够的额度

### Q3: 容器启动失败

**原因**：端口冲突或配置错误

**解决方案**：
```bash
# 查看容器日志
docker logs openclaw

# 检查端口占用
netstat -tulpn | grep 8080

# 修改端口（如改为 8081）
# 在 docker-compose.yml 中修改 ports 配置
ports:
  - "8081:8080"
```

### Q4: Skills 无法加载

**原因**：配置文件错误或 Skill 未正确安装

**解决方案**：
1. 检查 `/app/config/skills.yaml` 语法是否正确
2. 确认 Skill 已正确安装在 `/app/skills/` 目录
3. 查看容器日志中的 Skill 加载错误信息

### Q5: 飞书群消息收不到

**原因**：机器人未添加到群聊或群权限未开启

**解决方案**：
1. 将机器人添加到目标群聊
2. 在群设置中确保机器人权限已开启
3. 使用 `/help` 命令测试机器人是否在群内工作




clawdbot config set gateway.controlUi.allowInsecureAuth true
---

## 参考资源

### 官方资源

- [OpenClaw GitHub](https://github.com/justlovemaki/OpenClaw-Docker-CN-IM) - 中国 IM 版 Docker 镜像
- [飞书开放平台](https://open.feishu.cn/) - 应用创建和配置
- [1Panel 官方文档](https://1panel.cn/docs/v2/user_manual/containers/setting/) - 容器管理配置
- [Anthropic Console](https://console.anthropic.com/) - Claude API 管理

### 教程文章

- [2026年OpenClaw部署+飞书快速接入指南](https://developer.aliyun.com/article/1711216) - 阿里云开发者社区
- [1分钱部署OpenClaw全攻略](https://www.qbitai.com/2026/02/378183.html) - 量子位
- [飞书对接指南](https://www.feishu.cn/content/article/7602519239445974205) - 飞书官方文档

---

## 结语

通过本教程，你应该已经成功在 1Panel 上部署了 OpenClaw，并配置了飞书连接和 Claude Code。

如有问题，请参考上方故障排查部分，或查阅官方资源获取更多帮助。

祝你使用愉快！🎉
