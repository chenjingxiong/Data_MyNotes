# CookieCloud 介绍与部署指南

> 浏览器 Cookie 和 LocalStorage 自托管同步工具，支持端对端加密

## 📋 目录
- [什么是 CookieCloud](#什么是-cookiecloud)
- [核心特性](#核心特性)
- [应用场景](#应用场景)
- [浏览器插件安装](#浏览器插件安装)
- [服务器部署](#服务器部署)
- [使用配置](#使用配置)
- [API 接口](#api-接口)
- [加密解密](#加密解密)
- [常见问题](#常见问题)

---

## 什么是 CookieCloud

**CookieCloud** 是一个开源的浏览器 Cookie 和 LocalStorage 同步工具，允许你将浏览器数据同步到自建服务器，支持端对端加密。

### 项目信息
| 项目 | 信息 |
|------|------|
| **GitHub** | [easychen/CookieCloud](https://github.com/easychen/CookieCloud) |
| **官网** | https://cc.ft07.com |
| **语言** | TypeScript |
| **许可证** | GPL-3.0 |
| **Stars** | 2,900+ |

### 核心定位
- 🔒 **端对端加密**: 数据在本地加密后上传，服务器无法查看
- 🌐 **自托管**: 完全控制自己的数据
- 🔄 **双向同步**: 支持 Cookie 和 LocalStorage 同步
- ⏰ **定时同步**: 可配置同步间隔
- 📱 **跨设备**: 电脑浏览器同步到手机/云端

---

## 核心特性

### 功能对比

| 特性 | CookieCloud | 浏览器自带同步 | 其他云同步方案 |
|------|-------------|---------------|---------------|
| 自托管 | ✅ | ❌ | ❌ |
| 端对端加密 | ✅ | 部分 | 部分 |
| LocalStorage | ✅ | ❌ | ❌ |
| 自定义服务器 | ✅ | ❌ | ❌ |
| 开源免费 | ✅ | ✅ | ❌ |

### 主要特性
- **Cookie 同步**: 支持所有域名的 Cookie 同步
- **LocalStorage 同步**: 同域名下的 LocalStorage 同步
- **端对端加密**: AES 加密，密钥本地生成
- **灵活配置**: 支持配置同步间隔、过滤域名等
- **多浏览器支持**: Chrome、Edge、Firefox（需自行编译）

---

## 应用场景

### 场景 1：多设备 Cookie 共享

将工作电脑的 Cookie 同步到手机，随时随地保持登录状态。

### 场景 2：自动化脚本获取 Cookie

使用 Playwright/Puppeteer 等自动化工具时，从 CookieCloud 获取 Cookie 而无需手动登录。

### 场景 3：团队账号共享

团队成员共享同一账号的 Cookie，方便协作（需注意安全风险）。

### 场景 4：数据备份

定期备份重要的 Cookie 和 LocalStorage 数据，防止意外丢失。

---

## 浏览器插件安装

### 方式一：应用商店安装

| 浏览器 | 安装地址 |
|--------|----------|
| **Chrome** | [Chrome Web Store](https://chrome.google.com/webstore/detail/cookiecloud/ffjiejobkoibkjlhjnlgmcnnigeelbdl) |
| **Edge** | [Edge Add-ons](https://microsoftedge.microsoft.com/addons/detail/cookiecloud/bffenpfpjikaeocaihdonmgnjjdpjkeo) |

> ⚠️ 商店版本可能因审核延迟而落后于最新版本

### 方式二：手动安装

1. 访问 [GitHub Releases](https://github.com/easychen/CookieCloud/releases)
2. 下载最新版本的 `.crx` 或 `.zip` 文件
3. 解压并拖入浏览器扩展管理页面

### Firefox 支持

Firefox 需要自行编译源码：

```bash
git clone https://github.com/easychen/CookieCloud.git
cd CookieCloud
cd extension
pnpm install
pnpm build --target=firefox-mv2
```

> ⚠️ Firefox 和 Chrome 的 Cookie 格式不同，不能混用

---

## 服务器部署

### 方式一：使用 Docker（推荐）

#### 快速启动

```bash
docker run -d \
  --name cookiecloud \
  -p 8088:8088 \
  easychen/cookiecloud:latest
```

#### 持久化数据存储

```bash
docker run -d \
  --name cookiecloud \
  -p 8088:8088 \
  -v $(pwd)/data:/data/api/data \
  --restart always \
  easychen/cookiecloud:latest
```

#### 指定子目录（可选）

```bash
docker run -d \
  --name cookiecloud \
  -p 8088:8088 \
  -e API_ROOT=/cookie \
  easychen/cookiecloud:latest
```

访问地址变为：`http://your-server:8088/cookie`

#### Docker Compose 部署

创建 `docker-compose.yml`:

```yaml
version: '3'
services:
  cookiecloud:
    image: easychen/cookiecloud:latest
    container_name: cookiecloud-app
    restart: always
    volumes:
      - ./data:/data/api/data
    ports:
      - 8088:8088
    environment:
      - API_ROOT=/  # 可选：指定子目录
```

启动服务：
```bash
docker-compose up -d
```

### 方式二：使用 Node.js

```bash
# 克隆仓库
git clone https://github.com/easychen/CookieCloud.git
cd CookieCloud

# 安装依赖
cd api
yarn install

# 启动服务
node app.js
```

默认端口 8088，支持 `API_ROOT` 环境变量指定子目录。

### 方式三：1Panel 部署

如果你使用 1Panel 面板：

1. 进入 **应用商店**
2. 搜索 **CookieCloud**
3. 点击 **安装**
4. 配置端口和数据目录
5. 完成

---

## 使用配置

### 插件配置

1. 点击浏览器工具栏中的 CookieCloud 图标
2. 进入设置页面
3. 配置以下参数：

| 参数 | 说明 | 示例 |
|------|------|------|
| **服务器地址** | 自建服务器地址 | `http://your-server:8088` |
| **UUID** | 唯一标识符，自动生成 | `abc123-def456` |
| **密码** | 加密密码，自定义 | `my-secret-password` |
| **同步间隔** | 自动同步时间（分钟） | `30` |
| **域名过滤** | 只同步指定域名 | `.example.com` |

### 上传配置（数据源）

在需要同步 Cookie 的设备（如工作电脑）上：

1. 打开 CookieCloud 插件
2. 设置服务器地址、UUID 和密码
3. 在 **Cookie 管理** 中选择要上传的域名
4. 点击 **上传到服务器**

### 下载配置（数据接收方）

在需要接收 Cookie 的设备（如手机、其他电脑）上：

1. 打开 CookieCloud 插件
2. 使用相同的 **服务器地址、UUID 和密码**
3. 点击 **从服务器下载**

### 查看同步日志

1. 进入浏览器扩展管理页面
2. 找到 CookieCloud
3. 点击 **Service Worker**
4. 在弹出的面板中查看操作日志

---

## API 接口

### 上传数据

```http
POST /update
Content-Type: application/json

{
  "uuid": "your-uuid",
  "encrypted": "base64-encrypted-string"
}
```

### 下载数据

```http
GET /get/:uuid
或
POST /get/:uuid
Content-Type: application/json

{
  "password": "optional-password"
}
```

**响应格式**:

```json
{
  "encrypted": "base64-encrypted-string",
  "cookie_data": {
    ".example.com": [
      {
        "name": "session",
        "value": "xxx",
        "domain": ".example.com",
        "path": "/",
        "secure": true,
        "httpOnly": true,
        "sameSite": "Lax"
      }
    ]
  },
  "local_storage_data": {
    "example.com": {
      "key": "value"
    }
  }
}
```

---

## 加密解密

### 加密算法

```javascript
// 1. 数据序列化
const data = JSON.stringify(cookies);

// 2. 生成密钥（UUID + 密码的 MD5 前 16 位）
const key = CryptoJS.MD5(uuid + '-' + password).toString().substring(0, 16);

// 3. AES 加密
const encrypted = CryptoJS.AES.encrypt(data, key).toString();
```

### 解密算法（Node.js）

```javascript
const CryptoJS = require('crypto-js');

function cookie_decrypt(uuid, encrypted, password) {
    // 生成密钥
    const key = CryptoJS.MD5(uuid + '-' + password).toString().substring(0, 16);

    // AES 解密
    const decrypted = CryptoJS.AES.decrypt(encrypted, key).toString(CryptoJS.enc.Utf8);

    // 解析数据
    const parsed = JSON.parse(decrypted);

    return {
        cookie_data: parsed.cookie_data,
        local_storage_data: parsed.local_storage_data
    };
}
```

### 解密算法（Python）

```python
import hashlib
import json
import base64
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad

def cookie_decrypt(uuid, encrypted, password):
    # 生成密钥
    key_material = f"{uuid}-{password}".encode('utf-8')
    key = hashlib.md5(key_material).hexdigest()[:16].encode('utf-8')

    # Base64 解码
    encrypted_data = base64.b64decode(encrypted)

    # 解析 salt 和 IV
    # CryptoJS 格式: salted<salt[8]><iv[8]><ciphertext>
    salt = encrypted_data[8:16]
    iv = encrypted_data[16:32]
    ciphertext = encrypted_data[32:]

    # 派生密钥
    def derive_key(password, salt):
        return hashlib.md5(password + salt).digest()

    final_key = derive_key(key, salt)
    for _ in range(3):
        final_key = derive_key(final_key + key, salt)

    # AES 解密
    cipher = AES.new(final_key, AES.MODE_CBC, iv)
    padded = cipher.decrypt(ciphertext)
    data = unpad(padded, AES.block_size)

    return json.loads(data.decode('utf-8'))
```

---

## 自动化集成

### Playwright 示例

```javascript
const { chromium } = require('playwright');
const CryptoJS = require('crypto-js');

// 从 CookieCloud 获取 Cookie
async function getCookiesFromCloud(host, uuid, password) {
    const response = await fetch(`${host}/get/${uuid}`);
    const data = await response.json();

    if (!data.encrypted) return [];

    // 解密
    const key = CryptoJS.MD5(uuid + '-' + password).toString().substring(0, 16);
    const decrypted = CryptoJS.AES.decrypt(data.encrypted, key).toString(CryptoJS.enc.Utf8);
    const parsed = JSON.parse(decrypted);

    // 转换 Cookie 格式
    let cookies = [];
    for (const domain in parsed.cookie_data) {
        cookies = cookies.concat(parsed.cookie_data[domain]);
    }

    return cookies;
}

// 使用示例
(async () => {
    const host = 'http://your-server:8088';
    const uuid = 'your-uuid';
    const password = 'your-password';

    // 获取 Cookie
    const cookies = await getCookiesFromCloud(host, uuid, password);

    // 创建浏览器上下文并添加 Cookie
    const browser = await chromium.launch();
    const context = await browser.newContext();
    await context.addCookies(cookies);

    // 使用已登录的浏览器
    const page = await context.newPage();
    await page.goto('https://example.com');

    await browser.close();
})();
```

### Python Playwright 示例

```python
import requests
import hashlib
import json
from playwright.sync_api import sync_playwright

def get_cookies_from_cloud(host, uuid, password):
    """从 CookieCloud 获取 Cookie"""
    response = requests.get(f"{host}/get/{uuid}")
    data = response.json()

    if not data.get('encrypted'):
        return []

    # 使用 PyCookieCloud 或自行解密
    from pysqlite3_crypto import decrypt
    decrypted = decrypt(uuid, data['encrypted'], password)

    cookies = []
    for domain, cookie_list in decrypted['cookie_data'].items():
        cookies.extend(cookie_list)

    return cookies

# 使用示例
with sync_playwright() as p:
    host = 'http://your-server:8088'
    uuid = 'your-uuid'
    password = 'your-password'

    # 获取 Cookie
    cookies = get_cookies_from_cloud(host, uuid, password)

    # 创建浏览器上下文
    browser = p.chromium.launch()
    context = browser.new_context()
    context.add_cookies(cookies)

    # 使用已登录的浏览器
    page = context.new_page()
    page.goto('https://example.com')

    browser.close()
```

---

## 常见问题

### 1. 同步失败怎么办？

**检查清单**:
- 确认服务器地址正确（注意 http/https）
- 确认 UUID 和密码一致
- 检查网络连接
- 查看插件日志（Service Worker）

### 2. 为什么某些 Cookie 没有同步？

**可能原因**:
- 该 Cookie 设置了 `httpOnly` 且浏览器不允许访问
- 域名配置了过滤规则
- Cookie 已过期

### 3. 多个浏览器可以同时使用吗？

目前同步是单向的，一个浏览器上传，另一个浏览器下载。不建议多个浏览器同时上传同一 UUID 的数据。

### 4. 数据安全吗？

CookieCloud 使用端对端加密：
- 数据在本地加密后才上传
- 服务器只存储加密后的数据
- 加密密钥由 UUID + 密码生成，服务器无法获取

但仍需注意：
- 使用强密码
- 保护好 UUID 和密码
- 使用 HTTPS（如果服务器支持）

### 5. 支持哪些浏览器？

官方支持：
- ✅ Chrome / Chromium
- ✅ Edge

实验性支持：
- ⚠️ Firefox（需自行编译，Cookie 格式不兼容）

不支持：
- ❌ Safari

### 6. 版本升级说明

**0.1.5+ 版本重要变更**:
- 新增 LocalStorage 支持
- 加密格式改变（不兼容旧版本）
- 配置存储从远程改为本地

旧版本用户需要重新配置。

---

## 参考资源

### 官方资源
- **GitHub**: https://github.com/easychen/CookieCloud
- **官网**: https://cc.ft07.com
- **Docker Hub**: https://hub.docker.com/r/easychen/cookiecloud

### 教程
- **Bilibili 视频**: https://www.bilibili.com/video/BV1fR4y1a7zb
- **YouTube 视频**: https://youtu.be/3oeSiGHXeQw
- **掘金教程**: https://juejin.cn/post/7190963442017108027

### 社区
- **Telegram 频道**: https://t.me/CookieCloudTG
- **Telegram 群组**: https://t.me/CookieCloudGroup

### 第三方工具
- **PyCookieCloud** (Python): https://github.com/lupohan44/PyCookieCloud
- **RSSHub 集成**: https://github.com/sgpublic/rsshub-cookiecloud

---

## 总结

CookieCloud 是一个轻量级、安全的 Cookie 同步解决方案：

| 优点 | 说明 |
|------|------|
| 🔒 安全 | 端对端加密，服务器无法查看 |
| 🏠 自托管 | 完全控制数据 |
| 📦 轻量 | Docker 镜像仅几十 MB |
| 🌐 开源 | 完全免费，代码透明 |
| 🔧 灵活 | 支持自定义配置 |

### 适用场景

| 场景 | 推荐度 |
|------|--------|
| 个人多设备同步 | ⭐⭐⭐⭐⭐ |
| 自动化脚本 Cookie 获取 | ⭐⭐⭐⭐⭐ |
| 团队账号共享 | ⭐⭐⭐ |
| 数据备份 | ⭐⭐⭐⭐ |

---

**Happy Syncing!** 🍪

Generated with [Claude Code](https://claude.com/claude-code)
via [Happy](https://happy.engineering)
