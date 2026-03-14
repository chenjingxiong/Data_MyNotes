# CookieCloud 配置指南

> CookieCloud 是一个用于跨浏览器同步 Cookie 和 LocalStorage 的开源工具，通过自建服务器和浏览器插件实现端到端加密的数据同步。

## 📋 目录

- [功能特点](#功能特点)
- [部署服务器](#部署服务器)
- [安装浏览器插件](#安装浏览器插件)
- [配置同步](#配置同步)
- [常见问题](#常见问题)

---

## ✨ 功能特点

- 🔐 **端到端加密**：数据在本地加密后上传，服务器无法解密
- 🔄 **自动同步**：支持定时自动同步 Cookie、LocalStorage、SessionStorage
- 🌐 **跨浏览器**：支持 Chrome、Edge、Firefox 等主流浏览器
- 📱 **跨设备**：在不同设备间保持登录状态
- 🎯 **Cookie 保活**：防止重要 Cookie 过期失效

---

## 🚀 部署服务器

### 方式一：Docker 部署（推荐）

#### 1. 使用 Docker Compose

创建 `docker-compose.yml` 文件：

```yaml
version: '3'
services:
  cookiecloud:
    image: easychen/cookiecloud:latest
    container_name: cookiecloud
    ports:
      - "8088:8088"
    environment:
      - NODE_ENV=production
    restart: unless-stopped
```

运行：
```bash
docker-compose up -d
```

#### 2. 使用 Docker 命令

```bash
docker pull easychen/cookiecloud:latest
docker run -d \
  --name cookiecloud \
  -p 8088:8088 \
  --restart unless-stopped \
  easychen/cookiecloud:latest
```

#### 3. 配置数据持久化（可选）

```bash
docker run -d \
  --name cookiecloud \
  -p 8088:8088 \
  -v /path/to/data:/app/data \
  --restart unless-stopped \
  easychen/cookiecloud:latest
```

### 方式二：NAS 部署

支持群晖、威联通、绿联等 NAS 系统：

1. 在 Docker 套件中搜索 `easychen/cookiecloud`
2. 设置端口映射（容器端口 8088）
3. 启动容器

### 部署完成后

访问 `http://your-server-ip:8088` 确认服务正常运行。

---

## 📦 安装浏览器插件

### Chrome / Edge / Brave 等 Chromium 浏览器

1. **访问扩展商店**
   - Chrome: [Chrome Web Store](https://chrome.google.com/webstore)
   - Edge: [Microsoft Edge Add-ons](https://microsoftedge.microsoft.com/addons)

2. **搜索并安装**
   - 搜索 "CookieCloud"
   - 点击"添加到浏览器"安装

3. **确认安装**
   - 安装完成后，工具栏会出现 CookieCloud 图标

### Firefox 浏览器

1. **访问附加组件商店**
   - 访问 [Firefox Add-ons](https://addons.mozilla.org)

2. **安装插件**
   - 搜索 "CookieCloud"
   - 点击"添加到 Firefox"

3. **或安装 .xpi 文件**
   - 从 GitHub Releases 下载已签名的 .xpi 文件
   - 在 Firefox 中打开该文件即可安装

---

## ⚙️ 配置同步

### 步骤 1：设置服务器地址

1. 点击浏览器工具栏的 CookieCloud 图标
2. 在配置页面填入：
   - **服务器地址**：`http://your-server-ip:8088`
   - 示例：`http://192.168.1.100:8088` 或 `https://cookiecloud.example.com`

### 步骤 2：设置加密密钥

1. 设置一个**强密码**作为加密密钥
2. 该密钥用于端到端加密，**请妥善保管**
3. 在所有需要同步的设备上使用**相同的密钥**

### 步骤 3：配置同步选项

| 配置项 | 说明 | 推荐设置 |
|--------|------|----------|
| 同步间隔 | 自动同步的时间间隔 | 5-10 分钟 |
| 同步 Cookie | 是否同步 Cookie | ✅ 开启 |
| 同步 LocalStorage | 是否同步 LocalStorage | ✅ 开启 |
| 同步 SessionStorage | 是否同步 SessionStorage | 根据需要 |

### 步骤 4：选择同步的网站

1. 点击"配置域名"
2. 添加需要同步的网站域名
3. 支持通配符，如 `*.example.com`

### 步骤 5：开始同步

配置完成后，插件会自动开始同步：
- ✅ 上传本地数据到服务器
- ✅ 从服务器获取其他设备的数据
- ✅ 按设定间隔自动同步

### 手动同步

如需立即同步，点击插件图标，然后点击"立即同步"按钮。

---

## 🔧 高级配置

### 1. Cookie 过期管理

在插件设置中可以：
- 设置 Cookie 保活功能
- 配置 Cookie 过期提醒
- 手动清理过期 Cookie

### 2. 数据备份

建议定期备份：
- 导出 Cookie 数据为 JSON 文件
- 保存加密密钥到安全位置

### 3. 多用户隔离

如果多个用户使用同一服务器：
- 每个用户使用不同的加密密钥
- 数据会自动隔离，互不干扰

---

## ❓ 常见问题

### Q1: 同步不成功怎么办？

**检查清单：**
- ✅ 确认服务器地址正确
- ✅ 确认服务器正常运行（访问服务器地址测试）
- ✅ 确认所有设备使用相同的加密密钥
- ✅ 检查网络连接是否正常
- ✅ 查看浏览器控制台是否有错误信息

### Q2: 部分网站登录状态不同步？

**可能原因：**
- 该网站未在同步域名列表中
- 网站使用了 SameSite Cookie 限制
- Cookie 标记为 HttpOnly 且不被插件读取

**解决方案：**
- 在插件配置中添加该网站域名
- 尝试手动导出/导入该网站的 Cookie

### Q3: 如何更新服务器？

```bash
# 拉取最新镜像
docker pull easychen/cookiecloud:latest

# 停止并删除旧容器
docker stop cookiecloud
docker rm cookiecloud

# 使用相同配置启动新容器
docker run -d \
  --name cookiecloud \
  -p 8088:8088 \
  --restart unless-stopped \
  easychen/cookiecloud:latest
```

### Q4: 安全性如何保证？

- 🔐 所有数据在本地 AES 加密后上传
- 🔐 服务器只存储加密数据，无法解密
- 🔐 加密密钥由用户掌握，不存储在服务器
- 🔐 建议使用 HTTPS 配合自建服务器使用

### Q5: 支持哪些浏览器？

| 浏览器 | 支持状态 |
|--------|----------|
| Chrome / Chromium | ✅ 完全支持 |
| Microsoft Edge | ✅ 完全支持 |
| Firefox | ✅ 完全支持 |
| Safari | ⚠️ 需测试 |
| 移动端浏览器 | ❌ 不支持 |

---

## 📚 相关资源

- **GitHub 项目**: [easychen/CookieCloud](https://github.com/easychen/CookieCloud)
- **Docker 镜像**: [easychen/cookiecloud](https://hub.docker.com/r/easychen/cookiecloud)

---

## 📝 注意事项

1. **加密密钥丢失**：如果忘记加密密钥，服务器上的数据将无法恢复
2. **敏感数据**：不建议同步银行、支付等涉及资金的网站 Cookie
3. **Cookie 有效期**：部分网站的 Cookie 会过期，需要重新登录
4. **服务器维护**：建议定期检查服务器运行状态
5. **隐私保护**：自建服务器可确保数据隐私，避免使用公共服务器

---

**配置完成后，你的所有浏览器都能自动同步登录状态了！** 🎉
