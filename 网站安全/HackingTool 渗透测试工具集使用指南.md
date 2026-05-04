---
title: HackingTool 渗透测试工具集使用指南
source: https://github.com/Z4nzu/hackingtool
tags:
  - 渗透测试
  - 安全工具
  - Kali-Linux
  - 信息收集
  - ethical-hacking
date: 2026-05-04
---

# HackingTool 渗透测试工具集使用指南

> **项目地址**: [https://github.com/Z4nzu/hackingtool](https://github.com/Z4nzu/hackingtool)
> **当前版本**: v2.0.0 | **Stars**: 71k+ | **工具数量**: 185+
> **许可**: 开源免费

HackingTool 是一个集成了 **185+ 安全工具** 的全能渗透测试框架，通过菜单驱动的 CLI 界面，将信息收集、漏洞扫描、密码破解、无线攻击、钓鱼模拟等 20 大类安全测试工具统一到一个入口中。适合安全研究人员、渗透测试工程师和网络安全学习者使用。

![HackingTool GitHub 主页](attachments/hackingtool-github-main.png)

---

## 目录

- [[#1. 项目亮点 (v2.0.0 新特性)]]
- [[#2. 环境要求]]
- [[#3. 安装方式]]
- [[#4. 快速上手 — 操作指南]]
- [[#5. 快捷命令速查]]
- [[#6. 20 大工具分类详解]]
- [[#7. 实战场景示例]]
- [[#8. 常见问题 (FAQ)]]
- [[#9. 注意事项与法律声明]]

---

## 1. 项目亮点 (v2.0.0 新特性)

| 特性 | 说明 |
|:---:|---|
| **Python 3.10+** | 全面移除 Python 2 代码，使用现代 Python 语法重写 |
| **OS 智能菜单** | 在 macOS 上自动隐藏仅限 Linux 的工具 |
| **185+ 工具** | v2.0 新增 35 个现代化安全工具 |
| **全局搜索** | 输入 `/` 即可按名称、描述、关键词搜索所有工具 |
| **标签过滤** | 输入 `t` 按 19 个标签（osint、web、c2、cloud、mobile...）过滤 |
| **智能推荐** | 输入 `r` 描述需求（如 "我想扫描网络"），自动匹配工具 |
| **安装状态** | 每个工具旁显示 ✔/✘ ，一目了然哪些已就绪 |
| **批量安装** | 在任何分类中输入 `97` 一键安装该分类下所有工具 |
| **智能更新** | 每个工具支持 Update，自动检测 git pull / pip upgrade / go install |
| **Docker 支持** | 本地构建镜像，不依赖未验证的外部镜像 |
| **3 个新分类** | Active Directory、Cloud Security、Mobile Security |

---

## 2. 环境要求

### 2.1 操作系统

| 系统 | 支持情况 | 说明 |
|:---:|:---:|---|
| **Kali Linux** | ★★★ 推荐 | 最佳兼容性，大部分工具预装 |
| **Parrot OS** | ★★★ 推荐 | 安全测试专用系统 |
| **Ubuntu** | ★★☆ 良好 | 需额外安装部分依赖 |
| **macOS** | ★★☆ 可用 | 自动隐藏不兼容工具 |

### 2.2 依赖版本

| 依赖 | 版本要求 | 用途 |
|:---|:---:|---|
| **Python** | 3.10+ | 核心运行环境 |
| **Go** | 1.21+ | nuclei, ffuf, amass, httpx, katana, dalfox, gobuster, subfinder 等 |
| **Ruby** | any | haiti, evil-winrm |
| **Docker** | any (可选) | Mythic, MobSF 等容器化工具 |

> [!tip] 提示
> 推荐在 **Kali Linux** 上运行，绝大多数依赖已预装。如果在其他系统上使用，可能需要手动安装额外依赖。

---

## 3. 安装方式

### 方式一：一键安装（推荐）

最快最简单的方式，自动处理所有前置依赖、克隆仓库、创建虚拟环境并生成启动器：

```bash
curl -sSL https://raw.githubusercontent.com/Z4nzu/hackingtool/master/install.sh | sudo bash
```

> [!note] 执行流程
> 1. 检测操作系统并安装前置依赖（python3, python3-pip, python3-venv, git 等）
> 2. 克隆 `hackingtool` 仓库到本地
> 3. 创建 Python 虚拟环境（venv）
> 4. 安装 Python 依赖（`requirements.txt`）
> 5. 在 `/usr/local/bin/` 创建 `hackingtool` 启动脚本
> 6. 安装完成后，在终端输入 `hackingtool` 即可启动

### 方式二：手动安装

适合需要自定义安装路径或排查问题的用户：

```bash
# 1. 克隆仓库
git clone https://github.com/Z4nzu/hackingtool.git

# 2. 进入目录
cd hackingtool

# 3. 运行安装脚本
sudo python3 install.py

# 4. 启动
hackingtool
```

### 方式三：Docker 安装

适合不想污染主机环境、或需要在 CI/CD 中使用的场景：

```bash
# 克隆仓库
git clone https://github.com/Z4nzu/hackingtool.git
cd hackingtool

# 方式 A：直接构建运行
docker build -t hackingtool .
docker run -it --rm hackingtool

# 方式 B：Docker Compose（推荐）
docker compose up -d
docker exec -it hackingtool bash

# 开发模式（源码挂载，实时修改）
docker compose --profile dev up
docker exec -it hackingtool-dev bash

# 停止并清理
docker compose down        # 停止容器
docker compose down -v     # 停止并删除数据卷
```

---

## 4. 快速上手 — 操作指南

### 4.1 启动工具

```bash
sudo hackingtool
```

> [!warning] 必须使用 sudo
> 许多安全工具（如 Nmap SYN 扫描、Aircrack-ng 无线攻击等）需要 root 权限，因此务必使用 `sudo` 启动。

### 4.2 界面概览

启动后，你会看到一个**交互式菜单界面**，包含 20 个工具分类：

![HackingTool 主菜单界面](attachments/hackingtool-screenshot-1.png)

每个分类都有编号（1-20），输入对应数字即可进入该分类的子菜单。

### 4.3 基本操作流程

```
启动 hackingtool
    │
    ├── 输入数字（1-20）→ 进入对应工具分类
    │       │
    │       ├── 输入数字 → 选择具体工具
    │       │       │
    │       │       ├── [Install] → 安装该工具
    │       │       ├── [Run]     → 运行该工具
    │       │       ├── [Update]  → 更新该工具
    │       │       └── [Open]    → 打开工具目录
    │       │
    │       ├── 输入 97 → 批量安装该分类所有工具
    │       └── 输入 99 → 返回上一级菜单
    │
    ├── 输入 / → 搜索工具
    ├── 输入 t → 标签过滤
    ├── 输入 r → 智能推荐
    ├── 输入 ? → 帮助信息
    └── 输入 q → 退出
```

### 4.4 安装状态标识

进入任何分类后，每个工具旁都会显示安装状态：

| 标识 | 含义 |
|:---:|---|
| ✔ | 已安装，可直接运行 |
| ✘ | 未安装，需先安装 |

---

## 5. 快捷命令速查

| 命令 | 功能 | 适用位置 |
|:---:|---|:---:|
| `/query` | **全局搜索** — 按关键词即时查找工具 | 主菜单 |
| `t` | **标签过滤** — 按 osint、scanner、c2、cloud 等标签筛选 | 主菜单 |
| `r` | **智能推荐** — 描述需求，匹配推荐工具 | 主菜单 |
| `?` | **帮助** — 显示快捷参考卡片 | 任意位置 |
| `q` | **退出** — 从任意层级退出 | 任意位置 |
| `97` | **批量安装** — 一键安装当前分类下所有工具 | 分类菜单 |
| `99` | **返回** — 返回上一级菜单 | 任意子菜单 |

---

## 6. 20 大工具分类详解

### 6.1 🛡 匿名隐藏工具（2 个）

用于隐藏真实 IP 和身份，通过 Tor 网络实现匿名浏览。

| 工具 | 说明 |
|---|---|
| **Anonymously Surf** | 基于 Tor 的匿名上网方案 |
| **Multitor** | 多 Tor 实例管理工具 |

**使用场景**: 渗透测试时需要隐藏攻击源 IP，防止被目标追踪。

```bash
# 典型用法：启动匿名上网
sudo hackingtool → 选择 1 → 选择 Anonymously Surf
```

---

### 6.2 🔍 信息收集工具（26 个）

渗透测试的第一步——侦察。这是工具最多的分类之一。

**核心工具**:

| 工具 | 类型 | 说明 |
|---|:---:|---|
| **Nmap** | ★ 端口扫描 | 最强大的网络发现和安全审计工具 |
| **Masscan** | ★ 端口扫描 | 全球最快的端口扫描器 |
| **RustScan** | ★ 端口扫描 | Rust 编写的高速端口扫描器 |
| **theHarvester** | ★ OSINT | 从搜索引擎收集邮箱、子域名、IP 等 |
| **Amass** | ★ 子域名 | OWASP 子域名枚举工具 |
| **SpiderFoot** | ★ OSINT | 自动化 OSINT 情报收集平台 |
| **Shodanfy** | OSINT | 利用 Shodan API 获取目标信息 |
| **ReconSpider** | OSINT | 高级侦察框架 |
| **Subfinder** | ★ 子域名 | 被动子域名发现工具 |
| **httpx** | ★ 探测 | 批量 HTTP 探测工具 |
| **TruffleHog** | ★ 泄露检测 | 在 Git 仓库中搜索凭证和密钥 |
| **Gitleaks** | ★ 泄露检测 | 检测代码中的硬编码密钥 |
| **Holehe** | ★ 账号发现 | 通过邮箱检查社交平台注册情况 |
| **Maigret** | ★ 账号发现 | 通过用户名在 2000+ 网站搜索账号 |

![信息收集工具子菜单](attachments/hackingtool-screenshot-2.png)

**使用场景**:

```bash
# 场景：对目标域名进行全方位信息收集
sudo hackingtool
→ 选择 2 (Information Gathering)
→ 选择 theHarvester → 收集邮箱和子域名
→ 选择 Subfinder → 发现更多子域名
→ 选择 Nmap → 对发现的子域名进行端口扫描
→ 选择 httpx → 探测存活 Web 服务
```

---

### 6.3 📚 字典生成工具（7 个）

用于生成高质量的自定义密码字典和字典文件。

| 工具 | 说明 |
|---|---|
| **Cupp** | 基于目标个人信息生成定制字典 |
| **WordlistCreator** | 灵活的字典生成器 |
| **Goblin WordGenerator** | 规则化字典生成 |
| **Hashcat** | ★ GPU 加速的密码恢复工具 |
| **John the Ripper** | ★ 经典密码破解工具 |
| **haiti** | ★ 哈希类型识别工具 |

**使用场景**: 结合社会工程学信息（目标姓名、生日等）生成精准字典，用于密码爆破。

---

### 6.4 📶 无线攻击工具（13 个）

WiFi 和蓝牙安全测试工具集。

| 工具 | 说明 |
|---|---|
| **WiFi-Pumpkin** | 流氓 AP 框架 |
| **Fluxion** | WPA/WPA2 握手包捕获与破解 |
| **Wifite** | 自动化无线渗透测试 |
| **EvilTwin** | 邪恶双胞胎攻击 |
| **Airgeddon** | ★ 综合无线攻击脚本 |
| **Bettercap** | ★ 中间人攻击框架 |
| **Wifiphisher** | 自动化无线钓鱼攻击 |

**使用场景**: 在授权的安全评估中测试企业 WiFi 的安全性。

---

### 6.5 🧩 SQL 注入工具（7 个）

数据库安全测试工具集。

| 工具 | 说明 |
|---|---|
| **Sqlmap** | ★ 最强大的自动化 SQL 注入工具 |
| **NoSqlMap** | MongoDB 等 NoSQL 数据库注入工具 |
| **DSSS** | 轻量级 SQL 注入扫描器 |
| **Leviathan** | 综合审计工具框架 |
| **SQLScan** | SQL 注入漏洞扫描 |

**典型 Sqlmap 用法**:

```bash
# 通过 hackingtool 启动
sudo hackingtool → 选择 5 → 选择 Sqlmap

# 常用命令示例
sqlmap -u "http://target.com/page?id=1" --dbs          # 枚举数据库
sqlmap -u "http://target.com/page?id=1" -D dbname --tables  # 枚举表
sqlmap -u "http://target.com/page?id=1" -D dbname -T users --dump  # 导出数据
```

---

### 6.6 🎣 钓鱼攻击工具（17 个）

社会工程学测试工具集，**仅限授权安全评估使用**。

| 工具 | 说明 |
|---|---|
| **Setoolkit** | ★ 社会工程学工具包（SET） |
| **SocialFish** | 社交钓鱼框架 |
| **HiddenEye** | 高级钓鱼工具 |
| **Evilginx3** | ★ 中间人钓鱼代理（绕过 MFA） |
| **BlackEye** | 一键钓鱼页面 |
| **ShellPhish** | 多平台钓鱼工具 |
| **PyPhisher** | 钓鱼页面生成器 |
| **dnstwist** | 域名变形工具（生成相似域名） |
| **Maskphish** | URL 掩码工具 |

> [!danger] 法律警告
> 钓鱼攻击工具**仅可用于**授权的红队评估和安全意识培训。未经授权对他人使用属于**违法行为**。

---

### 6.7 🌐 Web 攻击工具（20 个）

Web 应用安全测试的核心工具集，v2.0 新增了大量现代化工具。

| 工具 | 类型 | 说明 |
|---|:---:|---|
| **Nuclei** | ★ 漏洞扫描 | 基于模板的快速漏洞扫描器 |
| **ffuf** | ★ 目录爆破 | Go 编写的高速 Fuzz 工具 |
| **Feroxbuster** | ★ 目录爆破 | Rust 编写的递归目录扫描 |
| **Nikto** | ★ 漏洞扫描 | Web 服务器安全扫描器 |
| **OWASP ZAP** | ★ 代理扫描 | OWASP 官方 Web 安全扫描器 |
| **Gobuster** | ★ 目录爆破 | Go 编写的目录/子域名爆破 |
| **Dirsearch** | ★ 目录扫描 | Python 编写的路径扫描器 |
| **wafw00f** | ★ WAF 检测 | Web 应用防火墙识别 |
| **Katana** | ★ 爬虫 | 下一代网页爬虫框架 |
| **Arjun** | ★ 参数发现 | HTTP 参数探测工具 |
| **mitmproxy** | ★ 中间人代理 | 交互式 HTTPS 代理 |
| **Caido** | ★ 代理工具 | 轻量级 Web 安全代理 |
| **testssl.sh** | ★ SSL 测试 | SSL/TLS 安全检测 |
| **Sublist3r** | 子域名枚举 | 跨搜索引擎子域名枚举 |

![Web 攻击工具子菜单](attachments/hackingtool-screenshot-3.png)

**典型 Web 渗透流程**:

```bash
# 1. 信息收集
sudo hackingtool → 选择 7 → 选择 Nuclei → 扫描目标漏洞
# 2. 目录爆破
sudo hackingtool → 选择 7 → 选择 ffuf → 发现隐藏路径
# 3. 参数探测
sudo hackingtool → 选择 7 → 选择 Arjun → 发现隐藏参数
# 4. 中间人代理
sudo hackingtool → 选择 7 → 选择 mitmproxy → 拦截分析请求
```

---

### 6.8 🔧 后渗透工具（10 个）

获取初始访问权限后的横向移动和权限维持工具。

| 工具 | 说明 |
|---|---|
| **pwncat-cs** | ★ 高级反向 Shell 管理器 |
| **Sliver** | ★ 现代 C2 框架 |
| **Havoc** | ★ 后渗透 C2 框架 |
| **PEASS-ng** | ★ 权限提升枚举（LinPEAS/WinPEAS） |
| **Ligolo-ng** | ★ 隧道/端口转发工具 |
| **Chisel** | ★ TCP/UDP 隧道 |
| **Evil-WinRM** | ★ WinRM 远程管理 Shell |
| **Mythic** | ★ C2 框架平台 |
| **Vegile** | 后门持久化工具 |

---

### 6.9 🕵 取证工具（8 个）

数字取证和安全事件分析。

| 工具 | 说明 |
|---|---|
| **Autopsy** | 磁盘取证分析平台 |
| **Wireshark** | 网络流量分析工具 |
| **Volatility 3** | ★ 内存取证框架 |
| **Binwalk** | ★ 固件/二进制分析工具 |
| **pspy** | ★ 进程监控工具（无需 root） |
| **Bulk extractor** | 媒体信息提取器 |

---

### 6.10 📦 Payload 生成工具（8 个）

生成各类渗透测试载荷。

| 工具 | 说明 |
|---|---|
| **The FatRat** | 多平台 Payload 生成器 |
| **MSFvenom Payload Creator** | Metasploit Payload 辅助生成 |
| **Venom** | 多协议 Payload 框架 |
| **Brutal** | 社会工程学 Payload |
| **Enigma** | 加密 Payload 生成器 |
| **Mob-Droid** | Android Payload 生成 |

---

### 6.11 🧰 漏洞利用框架（4 个）

| 工具 | 说明 |
|---|---|
| **RouterSploit** | 路由器漏洞利用框架 |
| **WebSploit** | Web 漏洞利用框架 |
| **Commix** | 命令注入漏洞利用 |
| **Web2Attack** | Web 攻击框架 |

---

### 6.12 🔁 逆向工程工具（5 个）

| 工具 | 说明 |
|---|---|
| **Ghidra** | ★ NSA 开源逆向工程工具 |
| **Radare2** | ★ 命令行逆向分析框架 |
| **JadX** | Android APK 反编译器 |
| **Androguard** | Android 应用分析 |
| **Apk2Gold** | APK 到 Java 源码转换 |

---

### 6.13 ⚡ DDoS 攻击工具（5 个）

| 工具 | 说明 |
|---|---|
| **SlowLoris** | 应用层慢速 DDoS |
| **GoldenEye** | HTTP DDoS 测试工具 |
| **UFOnet** | P2P DDoS 框架 |
| **Asyncrone** | 异步 DDoS 工具 |

> [!warning] DDoS 警告
> DDoS 攻击工具仅用于**自有服务器的压力测试**。对第三方发起 DDoS 是严重违法行为。

---

### 6.14 🖥 RAT 工具（1 个）

| 工具 | 说明 |
|---|---|
| **Pyshell** | Python 远程管理工具 |

---

### 6.15 💥 XSS 攻击工具（9 个）

| 工具 | 说明 |
|---|---|
| **DalFox** | ★ 快速 XSS 扫描器和参数 Fuzz 工具 |
| **XSStrike** | 高级 XSS 检测工具 |
| **XSpear** | XSS 漏洞扫描 |
| **XSS Payload Generator** | XSS 载荷生成器 |

---

### 6.16 🖼 隐写术工具（4 个）

| 工具 | 说明 |
|---|---|
| **SteganoHide** | 图片隐写工具 |
| **StegoCracker** | 隐写文件破解 |
| **Whitespace** | 空白字符隐写 |

---

### 6.17 🏢 Active Directory 工具（6 个） — v2.0 新增

内网渗透和域环境安全测试工具。

| 工具 | 说明 |
|---|---|
| **BloodHound** | ★ AD 关系分析与攻击路径可视化 |
| **NetExec (nxc)** | ★ 网络服务自动化利用 |
| **Impacket** | ★ 网络协议库（SMB、WMI、Kerberos 等） |
| **Responder** | ★ LLMNR/NBT-NS 毒化工具 |
| **Certipy** | ★ AD 证书服务漏洞利用 |
| **Kerbrute** | ★ Kerberos 用户名枚举和密码爆破 |

![后渗透与 AD 工具](attachments/hackingtool-screenshot-4.png)

---

### 6.18 ☁ 云安全工具（4 个） — v2.0 新增

| 工具 | 说明 |
|---|---|
| **Prowler** | ★ AWS 安全最佳实践检测 |
| **ScoutSuite** | ★ 多云安全审计框架 |
| **Pacu** | ★ AWS 渗透测试框架 |
| **Trivy** | ★ 容器/IaC 安全扫描器 |

---

### 6.19 📱 移动安全工具（3 个） — v2.0 新增

| 工具 | 说明 |
|---|---|
| **MobSF** | ★ 移动应用安全分析框架 |
| **Frida** | ★ 动态代码插桩工具 |
| **Objection** | ★ 移动应用运行时探索 |

---

### 6.20 ✨ 其他工具（24 个）

包含社交媒体爆破、Android 攻击、哈希破解、WiFi 反认证、社会媒体搜索、载荷注入、Web 爬虫等子分类：

- **社交媒体爆破**: AllinOne SocialMedia Attack、Facebook Attack
- **Android 工具**: Keydroid、MySMS、Lockphish、WishFish
- **社交媒体搜索**: Sherlock（用户名搜索）、SocialScan
- **哈希破解**: Hash Buster
- **WiFi 反认证**: WifiJammer-NG、KawaiiDeauther
- **载荷注入**: Debinject、Pixload
- **IDN 同形字攻击**: EvilURL
- **Web 爬虫**: Gospider

---

## 7. 实战场景示例

### 场景一：Web 应用安全评估

```
┌─────────────────────────────────────────────────┐
│             Web 应用安全评估流程                  │
├─────────────────────────────────────────────────┤
│                                                  │
│  1. 信息收集（分类 2）                            │
│     ├── Subfinder → 子域名发现                   │
│     ├── httpx → 存活探测                          │
│     └── theHarvester → 邮箱/员工信息收集          │
│                                                  │
│  2. 漏洞扫描（分类 7）                            │
│     ├── Nuclei → 已知漏洞扫描                     │
│     ├── wafw00f → WAF 检测                        │
│     ├── ffuf → 目录爆破                           │
│     └── Arjun → 隐藏参数发现                      │
│                                                  │
│  3. 漏洞利用（分类 5/15）                          │
│     ├── Sqlmap → SQL 注入测试                     │
│     └── DalFox → XSS 漏洞测试                    │
│                                                  │
│  4. 代理抓包（分类 7）                             │
│     └── mitmproxy → 流量拦截分析                  │
│                                                  │
└─────────────────────────────────────────────────┘
```

### 场景二：内网渗透测试

```
┌─────────────────────────────────────────────────┐
│              内网渗透测试流程                      │
├─────────────────────────────────────────────────┤
│                                                  │
│  1. 初始信息收集（分类 2）                         │
│     └── Nmap → 内网主机发现与端口扫描             │
│                                                  │
│  2. AD 域环境侦察（分类 17）                       │
│     ├── BloodHound → 域关系分析                   │
│     ├── Kerbrute → 用户名枚举                     │
│     └── Responder → 凭证窃取                     │
│                                                  │
│  3. 横向移动（分类 8）                             │
│     ├── Ligolo-ng → 隧道穿透                     │
│     ├── Evil-WinRM → WinRM 利用                   │
│     └── Impacket → SMB/WMI 利用                  │
│                                                  │
│  4. 权限提升（分类 8）                             │
│     └── PEASS-ng → 提权路径枚举                   │
│                                                  │
│  5. 权限维持（分类 8）                             │
│     └── Sliver → C2 持久化                       │
│                                                  │
└─────────────────────────────────────────────────┘
```

### 场景三：使用智能推荐功能

```bash
sudo hackingtool
# 在主菜单输入 r
# 然后输入：我想扫描一个网站的漏洞
# 工具会自动推荐：Nuclei, Nikto, OWASP ZAP 等相关工具
```

### 场景四：批量安装工具

```bash
sudo hackingtool
# 选择 7 (Web Attack)
# 输入 97
# 系统会自动安装该分类下所有 20 个 Web 攻击工具
```

---

## 8. 常见问题 (FAQ)

### Q1: 安装时报错怎么办？

```bash
# 检查 Python 版本（需要 3.10+）
python3 --version

# 手动安装依赖
pip3 install -r requirements.txt

# 更新系统
sudo apt update && sudo apt upgrade -y
```

### Q2: 工具运行报权限错误？

确保使用 `sudo hackingtool` 启动，大部分安全工具需要 root 权限。

### Q3: 如何更新单个工具？

进入对应分类 → 选择工具 → 选择 **Update**。系统会自动检测并执行 `git pull`、`pip upgrade` 或 `go install`。

### Q4: macOS 上某些工具不显示？

v2.0 会自动检测操作系统，仅 Linux 支持的工具会被隐藏，这是正常行为。

### Q5: Docker 模式下如何使用网络工具？

```bash
# 运行时需要给容器网络权限
docker run -it --rm --network host --cap-add=NET_ADMIN hackingtool
```

### Q6: 如何查看某个工具的安装目录？

进入分类 → 选择工具 → 选择 **Open folder**，会直接跳转到该工具的源码目录。

---

## 9. 注意事项与法律声明

> [!important] 免责声明
> - 本工具集**仅供授权安全测试和教育用途**
> - 未经授权对任何系统进行安全测试属于**违法行为**
> - 使用者需自行承担所有法律责任
> - 建议在专业安全团队指导下使用

### 最佳实践

1. **始终获得书面授权** — 在对任何目标进行测试前，确保获得了明确的书面许可
2. **记录所有操作** — 保存完整的测试日志和截图
3. **控制影响范围** — 避免对生产环境造成实际破坏
4. **及时报告漏洞** — 发现安全问题时立即通知相关方
5. **持续学习** — 网络安全是不断发展的领域，保持知识更新

---

## 相关链接

- **GitHub 仓库**: [Z4nzu/hackingtool](https://github.com/Z4nzu/hackingtool)
- **提交工具建议**: [Tool Request](https://github.com/Z4nzu/hackingtool/issues/new?template=tool_request.md)
- **作者 Twitter**: [@_Zinzu07](https://twitter.com/_Zinzu07)
- **支持作者**: [Buy Me a Coffee](https://buymeacoffee.com/hardikzinzu)

---

*最后更新: 2026-05-04*
