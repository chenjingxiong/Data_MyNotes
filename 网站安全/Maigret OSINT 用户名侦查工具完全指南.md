# Maigret OSINT 用户名侦查工具完全指南

> Maigret 是一款强大的开源 OSINT（开源情报）工具，以法国著名侦探 "梅格雷" 命名。它可以在 **3000+ 网站**和社交平台上快速搜索指定用户名，帮助你发现目标用户在互联网上的数字足迹。广泛用于安全审计、社会工程学调查、数字取证等场景。

## 📋 目录

- [功能特点](#功能特点)
- [安装部署](#安装部署)
- [基础用法](#基础用法)
- [命令行参数详解](#命令行参数详解)
- [报告生成](#报告生成)
- [高级用法](#高级用法)
- [作为 Python 库使用](#作为-python-库使用)
- [最佳实践](#最佳实践)
- [常见问题](#常见问题)
- [替代工具对比](#替代工具对比)
- [法律与道德声明](#法律与道德声明)

---

## ✨ 功能特点

- 🔍 **海量站点覆盖**：支持 3000+ 网站和社交平台的用户名搜索
- 📊 **多格式报告**：支持生成 HTML、PDF、CSV、TXT 格式的详细报告
- 🔗 **关联图谱**：可生成账户关联的可视化图谱
- 🔄 **递归搜索**：从已发现的账户中提取更多信息，进行二次搜索
- 🌐 **代理支持**：支持 HTTP/SOCKS 代理和 Tor 网络
- ⚡ **异步并发**：基于 asyncio 实现高并发扫描，速度快
- 🏷️ **站点分类**：支持按标签过滤站点（如 `photo`、`social`、`dating` 等）
- 🐳 **Docker 支持**：开箱即用的 Docker 镜像
- 🐍 **Python 库**：可作为 Python 库集成到其他项目中

---

## 🚀 安装部署

### 方式一：pip 安装（推荐）

```bash
# 直接安装
pip install maigret

# 或使用 pipx 安装到隔离环境
pipx install maigret
```

### 方式二：从源码安装

```bash
# 克隆仓库
git clone https://github.com/soxoj/maigret.git
cd maigret

# 安装依赖
pip install -r requirements.txt

# 运行
python maigret.py --help
```

### 方式三：Docker

```bash
# 拉取镜像
docker pull soxoj/maigret

# 运行搜索
docker run -t soxoj/maigret <username>

# 生成报告并挂载到本地
docker run -t -v $(pwd)/reports:/reports soxoj/maigret <username> --html --pdf
```

### 方式四：Termux（Android）

```bash
# 安装依赖
pkg install python git
pip install maigret

# 运行
maigret <username>
```

---

## 📖 基础用法

### 1. 搜索单个用户名

```bash
maigret <username>
```

> Maigret 会自动使用排名靠前的热门站点进行搜索，默认并发度较高。

### 2. 搜索多个用户名

```bash
maigret user1 user2 user3
```

### 3. 指定站点搜索

```bash
# 仅搜索特定站点
maigret <username> --site twitter
maigret <username> --site github --site reddit

# 按标签过滤
maigret <username> --site-filter photo
```

### 4. 全站扫描（最全面但最慢）

```bash
maigret <username> --all-sites
```

### 5. 指定热门站点数量

```bash
# 仅扫描前 100 个热门站点
maigret <username> --top-sites 100
```

---

## 🛠️ 命令行参数详解

### 核心参数

| 参数 | 简写 | 说明 |
|------|------|------|
| `--all-sites` | `-a` | 使用所有可用站点（最慢但最全面） |
| `--top-sites N` | | 仅使用前 N 个热门站点 |
| `--site SITE` | | 指定搜索的站点（可多次使用） |
| `--site-filter TAG` | | 按标签过滤站点（如 `photo`、`social`、`dating`） |
| `--ignore-site SITE` | | 排除指定站点 |
| `--stats` | | 显示站点数据库统计信息 |
| `--db-update` | | 更新站点数据库 |

### 连接与性能

| 参数 | 简写 | 说明 |
|------|------|------|
| `--connections N` | `-c` | 最大并发连接数（默认 100） |
| `--timeout N` | `-t` | 单个请求超时时间（秒） |
| `--retries N` | | 失败请求重试次数 |
| `--no-recursion` | | 禁用递归搜索 |

### 代理与匿名

| 参数 | 说明 |
|------|------|
| `--proxy URL` | 使用代理（支持 HTTP/SOCKS5） |
| `--tor` | 使用 Tor 网络（需要本地运行 Tor） |
| `--tor-proxy PORT` | 指定 Tor 代理端口（默认 9050） |

### 输出与报告

| 参数 | 简写 | 说明 |
|------|------|------|
| `--html` | | 生成 HTML 报告 |
| `--pdf` | | 生成 PDF 报告 |
| `--csv` | | 导出 CSV 格式结果 |
| `--txt` | | 导出纯文本报告 |
| `--graph` | | 生成关联图谱 |
| `--output DIR` | `-o` | 指定报告输出目录 |
| `--verbose` | `-v` | 详细输出模式 |
| `--no-color` | | 禁用彩色输出 |

---

## 📊 报告生成

### 生成多格式报告

```bash
# 同时生成 HTML 和 PDF 报告
maigret <username> --html --pdf -o ./reports/

# 生成所有格式的报告
maigret <username> --html --pdf --csv --txt --graph -o ./reports/
```

### 报告内容说明

Maigret 对每个站点的搜索结果有以下状态：

| 状态 | 含义 |
|------|------|
| ✅ **Claimed** | 用户名在该站点已被注册（找到匹配） |
| ❌ **Not Found** | 用户名在该站点不存在 |
| ⚠️ **Error** | 请求出错（网络问题、站点限制等） |
| ⏳ **Unavailable** | 站点当前不可访问 |
| 🔍 **Partial** | 部分匹配，需要人工确认 |

### 报告中的信息

一份典型的报告包含：

- **用户名**在哪些站点存在
- **个人资料链接**（可直接访问）
- **个人信息摘要**（如简介、头像等）
- **关联账户**（通过递归搜索发现的）
- **统计信息**（命中率、扫描站点数等）

---

## 🔧 高级用法

### 1. 递归搜索（发现关联账户）

```bash
maigret <username> --recursive
```

> 开启后，Maigret 会从已发现的个人资料页面中提取额外的用户名信息，并对这些新发现的用户名进行二次搜索，从而构建更完整的数字足迹画像。

### 2. 使用代理匿名搜索

```bash
# 使用 HTTP 代理
maigret <username> --proxy http://127.0.0.1:8080

# 使用 SOCKS5 代理
maigret <username> --proxy socks5://127.0.0.1:1080

# 使用 Tor 网络
maigret <username> --tor
```

### 3. 精细控制扫描行为

```bash
# 低速模式（适合网络不稳定时）
maigret <username> --connections 10 --timeout 30 --retries 3

# 高速模式（适合网络良好时）
maigret <username> --connections 200 --timeout 10
```

### 4. 站点数据库管理

```bash
# 查看支持的站点统计
maigret --stats

# 更新站点数据库（建议定期执行）
maigret --db-update
```

### 5. 组合使用

```bash
# 完整调查：全站扫描 + 递归 + Tor + 多格式报告
maigret <username> \
  --all-sites \
  --recursive \
  --tor \
  --html --pdf --graph \
  --connections 50 \
  --timeout 20 \
  -o ./investigation/
```

---

## 🐍 作为 Python 库使用

Maigret 不仅可以作为命令行工具使用，还可以作为 Python 库集成到你的项目中：

```python
import asyncio
from maigret import maigret

async def search_username(username):
    """搜索用户名并返回结果"""
    results = await maigret(
        username=username,
        sites=["twitter", "github", "reddit"],
        timeout=10,
    )
    return results

# 运行搜索
results = asyncio.run(search_username("example_user"))

# 处理结果
for site_name, data in results.items():
    if data.get("status") == "claimed":
        print(f"[+] Found on {site_name}: {data.get('url_user')}")
```

### 批量搜索脚本

```python
import asyncio
from maigret import maigret

async def batch_search(usernames):
    """批量搜索多个用户名"""
    tasks = [maigret(username=u, top_sites=50) for u in usernames]
    results = await asyncio.gather(*tasks)
    return dict(zip(usernames, results))

usernames = ["user1", "user2", "user3"]
all_results = asyncio.run(batch_search(usernames))

for username, sites in all_results.items():
    found = [s for s, d in sites.items() if d.get("status") == "claimed"]
    print(f"{username}: found on {len(found)} sites → {found}")
```

---

## 💡 最佳实践

### 1. 更新站点数据库

站点数据库经常更新，建议在每次重大搜索前更新：

```bash
maigret --db-update
```

### 2. 从小范围开始

先用热门站点快速扫描，确认目标后再扩大范围：

```bash
# 第一步：快速扫描
maigret <username> --top-sites 50

# 第二步：全站扫描（如果需要）
maigret <username> --all-sites --recursive
```

### 3. 使用代理保护身份

在进行敏感调查时，务必使用代理或 Tor：

```bash
maigret <username> --tor --connections 30
```

### 4. 控制并发以避免被封锁

过高的并发可能导致被目标站点限制：

```bash
# 保守策略
maigret <username> --connections 20 --timeout 15 --retries 2
```

### 5. 生成报告留存证据

始终生成报告以供后续分析：

```bash
maigret <username> --html --pdf --graph -o ./reports/
```

---

## ❓ 常见问题

### Q1：安装时出现依赖冲突怎么办？

```bash
# 使用虚拟环境
python -m venv maigret-env
source maigret-env/bin/activate  # Linux/macOS
# 或 maigret-env\Scripts\activate  # Windows

pip install maigret
```

### Q2：搜索速度很慢？

- 减少 `--timeout` 值（如 `--timeout 10`）
- 适当增加 `--connections`（如 `--connections 150`）
- 使用 `--top-sites` 减少扫描站点数
- 检查网络连接是否稳定

### Q3：很多站点返回 Error？

- 部分站点可能暂时不可用或已下线
- 尝试使用 `--retries 2` 增加重试
- 可能被目标站点限流，降低 `--connections`
- 检查是否需要使用代理

### Q4：Tor 模式无法连接？

```bash
# 确保 Tor 服务已启动
sudo systemctl start tor    # Linux
# 或手动启动 Tor Browser

# 检查 Tor 端口
maigret <username> --tor --tor-proxy 9050
```

### Q5：Docker 中生成的报告如何保存到本地？

```bash
# 使用 -v 挂载目录
docker run -t -v $(pwd)/reports:/reports soxoj/maigret <username> --html --pdf -o /reports/
```

### Q6：PDF 生成失败？

确保安装了相关依赖：

```bash
pip install maigret[pdf]
# 或
pip install weasyprint
```

---

## 🔄 替代工具对比

| 特性 | Maigret | Sherlock | SpiderFoot | theHarvester |
|------|---------|----------|------------|--------------|
| **类型** | 用户名搜索 | 用户名搜索 | 综合OSINT | 邮箱/域名搜索 |
| **站点数量** | 3000+ | 300+ | 综合型 | N/A |
| **语言** | Python | Python | Python | Python |
| **递归搜索** | ✅ | ❌ | ✅ | ❌ |
| **PDF 报告** | ✅ | ❌ | ✅ | ❌ |
| **图谱可视化** | ✅ | ❌ | ✅ | ❌ |
| **代理/Tor** | ✅ | ✅ | ✅ | ✅ |
| **Python 库** | ✅ | ✅ | ✅ | ❌ |
| **Docker** | ✅ | ✅ | ✅ | ✅ |
| **适合场景** | 深度用户名调查 | 快速搜索 | 全面OSINT | 域名/邮箱情报 |

> 💡 **建议**：Maigret 和 Sherlock 可以互补使用。Sherlock 更轻量快速，Maigret 站点更多且支持递归搜索。

---

## ⚖️ 法律与道德声明

> **重要提醒：请合法合规使用此工具。**

1. **仅用于合法目的**：安全审计、授权渗透测试、自我数字足迹检查
2. **尊重隐私**：不用于跟踪、骚扰或侵犯他人隐私
3. **遵守平台规则**：注意各平台的服务条款和使用政策
4. **了解当地法律**：不同地区对 OSINT 活动的法律规定不同
5. **不用于恶意目的**：禁止用于社会工程攻击、身份盗用等违法行为
6. **教育目的**：本指南仅供学习和合法安全研究使用

---

## 📎 参考链接

- **GitHub 仓库**：[https://github.com/soxoj/maigret](https://github.com/soxoj/maigret)
- **PyPI 页面**：[https://pypi.org/project/maigret/](https://pypi.org/project/maigret/)
- **Docker Hub**：[https://hub.docker.com/r/soxoj/maigret](https://hub.docker.com/r/soxoj/maigret)

---

*最后更新：2026-05-04*
