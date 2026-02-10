# 1Panel 部署 VitePress 教程

本教程介绍如何使用 1Panel 面板快速部署 VitePress 静态网站。

---

## 目录

1. [前置准备](#1-前置准备)
2. [本地构建 VitePress](#2-本地构建-vitepress)
3. [1Panel 部署](#3-1panel-部署)
4. [域名配置](#4-域名配置)
5. [自动更新方案](#5-自动更新方案)
6. [常见问题](#6-常见问题)

---

## 1. 前置准备

### 1.1 环境要求

| 项目        | 要求                           |
| --------- | ---------------------------- |
| 服务器       | Linux (Ubuntu/CentOS/Debian) |
| 1Panel    | 已安装 1Panel 面板                |
| OpenResty | 在 1Panel 应用商店安装              |

### 1.2 安装 1Panel

如果还未安装，执行以下命令：

```bash
curl -sSL https://resource.fit2cloud.com/1panel/package/quick_start.sh -o quick_start.sh && bash quick_start.sh
```

安装完成后访问 `http://你的服务器IP:端口` 登录面板。

---

## 2. 本地构建 VitePress

### 2.1 创建并构建项目

```bash
# 创建项目
mkdir vitepress-site && cd vitepress-site
npm init -y

# 安装 VitePress
npm install -D vitepress

# 初始化（按提示选择）
npx vitepress init

# 安装依赖并构建
npm install
npm run docs:build
```

### 2.2 构建产物说明

构建完成后，静态文件位于：

```
docs/.vitepress/dist/
```

**这个 `dist` 目录就是我们部署到 1Panel 的内容。**

---

## 3. 1Panel 部署

### 3.1 安装 OpenResty

1. 登录 1Panel 面板
2. 进入 **应用商店** → 搜索 `OpenResty`
3. 点击 **安装**
4. 配置端口（默认 `80`）

### 3.2 创建网站

**方式一：上传静态文件**

1. 将 `dist` 目录打包：
   ```bash
   cd docs/.vitepress
   tar -czf vitepress-dist.tar.gz dist/
   ```

2. 在 1Panel 进入 **网站** → **创建网站**：
   - **类型**：静态网站
   - **主域名**：`your-domain.com` 或服务器 IP
   - **根目录**：`/opt/1panel/apps/openresty/openresty/www/vitepress`

3. 上传并解压：
   - 进入 **文件** 管理
   - 导航到网站根目录
   - 上传 `vitepress-dist.tar.gz`
   - 右键解压，将 `dist/` 内的文件移动到根目录

**方式二：服务器 Git Clone + 构建**

1. 确保服务器已安装 Node.js：
   ```bash
   # 在 1Panel 终端或 SSH 中执行
   curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
   apt-get install -y nodejs
   ```

2. 在服务器克隆并构建：
   ```bash
   # 克隆项目
   cd /opt/1panel/apps/openresty/openresty/www
   git clone https://github.com/username/vitepress-site.git vitepress
   cd vitepress

   # 安装依赖并构建
   npm install
   npm run docs:build

   # 复制构建产物到网站根目录
   cp -r docs/.vitepress/dist/* ./
   ```

3. 在 1Panel 进入 **网站** → **创建网站**：
   - **类型**：静态网站
   - **主域名**：`your-domain.com` 或服务器 IP
   - **根目录**：`/opt/1panel/apps/openresty/openresty/www/vitepress`

### 3.3 配置 HTTPS（可选）

1. 进入网站 **设置** → **SSL**
2. 选择 **Let's Encrypt** 或上传自有证书
3. 开启 **强制 HTTPS**

---

## 4. 域名配置

### 4.1 DNS 解析

在域名服务商添加解析：

| 类型 | 主机记录 | 记录值 |
|------|----------|--------|
| A | @ | 你的服务器 IP |
| A | www | 你的服务器 IP |

### 4.2 1Panel 网站配置

在网站设置中添加域名绑定：
- 主域名：`your-domain.com`
- 子域名：`www.your-domain.com`

---

## 5. 自动更新方案

### 5.1 使用 GitHub Actions 自动构建 + SSH 部署（推荐）

创建 `.github/workflows/deploy.yml`：

```yaml
name: Deploy to 1Panel Server

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Build VitePress
        run: |
          npm install
          npm run docs:build

      - name: Deploy to server
        uses: easingthemes/ssh-deploy@main
        env:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
          REMOTE_HOST: ${{ secrets.REMOTE_HOST }}
          REMOTE_USER: ${{ secrets.REMOTE_USER }}
          TARGET: /opt/1panel/apps/openresty/openresty/www/vitepress
          SOURCE: "docs/.vitepress/dist/"
```

在 GitHub Secrets 中配置：
- `SSH_PRIVATE_KEY`：服务器 SSH 私钥
- `REMOTE_HOST`：服务器 IP
- `REMOTE_USER`：SSH 用户名（通常是 `root`）

---

## 6. 常见问题

### Q1: 页面空白，404 错误

**原因**：`base` 路径配置错误

**解决**：检查 `.vitepress/config.ts`：

```typescript
// 如果部署在根目录
base: '/'

// 如果部署在子目录
base: '/your-subdirectory/'
```

### Q2: 刷新页面 404

**原因**：Nginx 未配置 SPA 路由回退

**解决**：在 1Panel 网站 **设置** → **配置文件** 中添加：

```nginx
location / {
    try_files $uri $uri/ /index.html;
}
```

### Q3: 资源加载失败

**解决**：检查 `outDir` 配置，确保构建产物路径正确：

```typescript
// .vitepress/config.ts
export default defineConfig({
  outDir: './dist',  // 确保是相对路径
})
```

### Q4: 想要自动更新怎么办？

**推荐方案**：使用 GitHub Actions + SSH 部署（见上方第 5 节）

**替代方案**：使用简单的 Webhook 服务

1. 在 1Panel 容器中运行 webhook 服务：
   ```bash
   docker run -d -p 9000:9000 \
     -v /opt/webhook/hooks.yml:/etc/webhook/hooks.yml \
     almir/webhook
   ```

2. 创建 `hooks.yml`：
   ```yaml
   - id: update-vitepress
     execute-command: /opt/scripts/update-vitepress.sh
     command-working-directory: /opt/1panel/apps/openresty/openresty/www/vitepress
   ```

3. 在 GitHub 仓库配置 Webhook URL：`http://your-server:9000/hooks/update-vitepress`

---

## 快速部署命令总结

```bash
# 方式一：本地构建 + 上传
npm run docs:build
cd docs/.vitepress
tar -czf vitepress-dist.tar.gz dist/
# 通过 1Panel 文件管理上传并解压

# 方式二：服务器 Git Clone
cd /opt/1panel/apps/openresty/openresty/www
git clone https://github.com/username/vitepress-site.git vitepress
cd vitepress && npm install && npm run docs:build
cp -r docs/.vitepress/dist/* ./

# 验证
# 访问 http://your-domain.com
```

---

## 方案对比

| 方案                 | 优点            | 缺点                | 适用场景   |
| ------------------ | ------------- | ----------------- | ------ |
| 手动上传 dist         | 最简单，无需服务器环境   | 每次更新需手动操作         | 更新频率低  |
| 服务器 Git + 构建       | 一次配置，后续更新方便   | 需服务器安装 Node.js     | 个人项目  |
| GitHub Actions + SSH | 完全自动，推送即部署   | 需要 SSH 配置        | 团队协作  |
| Docker Webhook      | 实时更新，推送即触发构建 | 需额外配置 Docker 容器 | 高频更新项目 |

---

## 参考资源

- [1Panel 官方文档](https://1panel.cn/docs/)
- [VitePress 官方文档](https://vitepress.dev/)
- [OpenResty 文档](https://openresty.org/en/)

---

## 更新日志

| 日期 | 版本 | 说明 |
|------|------|------|
| 2026-02-10 | 1.0.0 | 初始版本 |

---

> 💡 **提示**：1Panel 目前不支持直接从 Git 拉取创建静态网站（这是社区期望功能）。推荐使用 GitHub Actions + SSH 方式实现自动部署，或在服务器上使用 Git Clone 手动更新。
