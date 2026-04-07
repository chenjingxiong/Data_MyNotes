# Spacedrive 介绍与部署使用指南

> **项目地址**：[github.com/spacedriveapp/spacedrive](https://github.com/spacedriveapp/spacedrive)
> **官方网站**：[spacedrive.com](https://spacedrive.com)
> **许可证**：AGPL v3
> **核心语言**：Rust（~183k 行）
> **GitHub Stars**：37k+

---

## 一、Spacedrive 是什么？

Spacedrive 是一款**开源跨平台文件管理器**，基于 Rust 编写的**虚拟分布式文件系统（VDFS）**。它解决了现代多设备时代文件分散在各处的痛点，让用户从一个统一的界面管理所有设备上的文件——本地硬盘、NAS、云存储（S3、Google Drive、Dropbox 等）。

### v3 版本最新方向

Spacedrive v3 已从纯文件管理器升级为**本地优先的数据引擎**：

- **统一搜索**：索引所有数据源，本地搜索，数据不离开你的机器
- **处理管线**：每条记录经过安全筛查、内容分类、质量评分后才进入搜索索引
- **AI 代理集成**：内置 Spacebot AI 代理，基于处理后的清洁数据工作
- **离线优先**：单二进制文件，不回传数据

> **当前状态**：v3.0-beta，支持 macOS / Linux / Windows

### 与传统文件管理器的区别

| 特性 | 传统文件管理器 | Spacedrive |
|------|---------------|------------|
| 文件识别方式 | 按路径 | 按内容指纹（BLAKE3） |
| 多设备管理 | 不支持 | 统一视图，跨设备索引 |
| 云存储 | 需单独客户端 | 原生集成为一级卷 |
| 同步方式 | 云端中转 | P2P 直连，无中心服务器 |
| 离线支持 | 有限 | 完全离线可用 |
| 重复文件 | 手动查找 | 自动跨设备去重 |

---

## 二、核心架构

Spacedrive 建立在四大核心原则之上：

### 1. 虚拟分布式文件系统（VDFS）

文件和文件夹成为具有丰富元数据的一等对象，独立于物理位置。每个文件获得一个跨设备通用地址（`SdPath`），通过内容感知寻址，你可以按内容而非位置引用文件。

### 2. 内容身份系统

使用自适应哈希（BLAKE3 + 大文件策略采样）为每个内容创建唯一指纹：

- **去重**：跨设备识别相同文件
- **冗余追踪**：了解备份分布
- **基于内容的操作**：「从任意可用位置复制此文件」

### 3. 事务性操作

每个文件操作在执行前可预览——查看空间节省、冲突、预估时间——然后批准或取消。操作变为持久任务，可承受网络中断和设备重启。

### 4. 无主同步

无需中心协调器的 P2P 同步：
- 设备特定数据（文件系统索引）使用状态复制
- 共享元数据（标签、评分）使用 HLC 有序日志 + 确定性冲突解决
- 无领导者选举，无单点故障

---

## 三、功能特性

### 核心功能

| 功能 | 说明 |
|------|------|
| 跨平台 | macOS、Windows、Linux、iOS、Android |
| 多设备索引 | 统一查看所有设备上的文件 |
| 内容寻址 | 自动寻找最优文件副本（本地 > 局域网 > 云端） |
| 智能去重 | 无论名称或位置，识别相同文件 |
| 云存储集成 | S3、Google Drive、Dropbox、OneDrive、Azure Blob 作为一级卷 |
| P2P 网络 | 直连设备通信，自动 NAT 穿越（Iroh + QUIC） |
| 语义标签 | 基于图的标签系统，支持层级、别名、上下文消歧 |
| 操作预览 | 执行前模拟任何操作 |
| 离线优先 | 无网络时完全可用，重连后自动同步 |
| 本地备份 | 设备间 P2P 备份（iOS 照片备份已可用） |
| 扩展系统 | 基于 WASM 的插件，支持领域特定功能 |

### 专业扩展

| 扩展 | 用途 | 状态 |
|------|------|------|
| Photos | AI 照片管理（人脸识别、地点、场景分类） | 开发中 |
| Chronicle | 研究与知识管理 | 开发中 |
| Atlas | 动态 CRM 与团队知识 | 开发中 |
| Studio | 数字资产管理 | 计划中 |
| Ledger | 财务智能（票据 OCR、费用追踪） | 计划中 |
| Guardian | 备份与冗余监控 | 计划中 |
| Cipher | 安全与加密 | 计划中 |

---

## 四、技术栈

### 核心层

| 组件 | 技术 |
|------|------|
| 语言 | Rust |
| 异步运行时 | Tokio |
| 数据库 | SQLite + SeaORM |
| P2P 网络 | Iroh（QUIC 传输、打洞、本地发现） |
| 哈希 | BLAKE3 |
| 扩展运行时 | Wasmer（沙箱 WASM） |
| HTTP/API | Axum（HTTP/GraphQL） |
| 云存储抽象 | OpenDAL |
| 类型生成 | Specta（自动生成 TypeScript/Swift 类型） |

### 安全层

| 组件 | 技术 |
|------|------|
| 签名与密钥交换 | Ed25519 / X25519 |
| 加密 | ChaCha20-Poly1305 / AES-GCM |
| 密码哈希 | Argon2 |
| 密钥备份 | BIP39 助记词 |
| 凭证存储 | redb（加密键值存储） |

### 前端

| 组件 | 技术 |
|------|------|
| 框架 | React 19 |
| 构建 | Vite |
| 状态管理 | TanStack Query + Zustand |
| UI 组件 | Radix UI + Tailwind CSS |
| 桌面端 | Tauri 2 |
| 移动端 | React Native 0.81 + Expo |
| 3D 可视化 | Three.js / React Three Fiber |

---

## 五、部署方式

Spacedrive 提供多种部署方式，满足不同需求。

### 方式一：桌面客户端（推荐个人用户）

直接下载安装客户端，这是最简单的方式。

#### macOS / Linux

```bash
# 从 GitHub Releases 下载
# 访问 https://github.com/spacedriveapp/spacedrive/releases

# macOS - 下载 .dmg 文件安装
# Linux - 下载 .deb 或 .AppImage 文件

# Debian/Ubuntu 安装 .deb
sudo dpkg -i spacedrive_*.deb

# 或使用 AppImage（无需安装）
chmod +x Spacedrive_*.AppImage
./Spacedrive_*.AppImage
```

#### Windows

从 GitHub Releases 下载 `.msi` 安装包，双击安装即可。

### 方式二：Docker 自托管部署（推荐服务器）

Spacedrive 提供无头服务器镜像，支持 Docker 部署，带有 Web 界面。

```bash
# 拉取镜像
docker pull ghcr.io/spacedriveapp/spacedrive-server:latest

# 运行容器
docker run -d \
  --name spacedrive \
  --restart unless-stopped \
  -p 8080:8080 \
  -v /path/to/your/data:/data \
  -v /path/to/config:/config \
  ghcr.io/spacedriveapp/spacedrive-server:latest
```

#### Docker Compose 部署

创建 `docker-compose.yml`：

```yaml
version: '3.8'

services:
  spacedrive:
    image: ghcr.io/spacedriveapp/spacedrive-server:latest
    container_name: spacedrive
    restart: unless-stopped
    ports:
      - "8080:8080"
    volumes:
      - ./data:/data          # 文件数据
      - ./config:/config      # 配置文件
      - /mnt/nas:/media:ro    # 挂载 NAS（只读示例）
    environment:
      - SD_LOG_LEVEL=info
```

```bash
# 启动服务
docker compose up -d

# 查看日志
docker compose logs -f spacedrive

# 访问 Web 界面
# http://localhost:8080
```

#### 1Panel 面板部署

如果你使用 1Panel 管理服务器，可以通过应用商店或自定义应用部署：

1. 进入 1Panel -> 容器 -> 创建应用
2. 镜像：`ghcr.io/spacedriveapp/spacedrive-server:latest`
3. 端口映射：`8080:8080`
4. 挂载卷：数据目录和配置目录
5. 启动应用

### 方式三：从源码编译

适合开发者或需要自定义的用户。

#### 环境准备

```bash
# 安装 Rust（需要 1.81+）
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup update

# 安装 Bun（需要 1.3+）
curl -fsSL https://bun.sh/install | bash

# 安装系统依赖（Debian/Ubuntu）
sudo apt update
sudo apt install -y \
  libwebkit2gtk-4.1-dev \
  build-essential \
  curl \
  wget \
  file \
  libssl-dev \
  libgtk-3-dev \
  libayatana-appindicator3-dev \
  librsvg2-dev \
  libsoup-3.0-dev \
  libclang-dev
```

#### 编译与运行

```bash
# 克隆仓库
git clone https://github.com/spacedriveapp/spacedrive
cd spacedrive

# 安装前端依赖
bun install

# 初始化构建配置
cargo run -p xtask -- setup

# 编译所有核心和应用
cargo build

# 编译 release 版本（更快运行速度）
cargo build --release
```

#### 运行桌面应用

```bash
cd apps/tauri
bun run tauri:dev
```

#### 运行 CLI 模式

```bash
# 查看帮助
cargo run -p sd-cli -- --help

# 启动守护进程
cargo run -p sd-cli -- daemon start

# 创建库
cargo run -p sd-cli -- library create "My Library"

# 添加索引位置
cargo run -p sd-cli -- location add ~/Documents

# 搜索已索引文件
cargo run -p sd-cli -- search .
```

---

## 六、基本使用

### 1. 创建库（Library）

库是 Spacedrive 的核心组织单元，包含你的文件索引、标签、元数据等。

```
桌面应用：打开 Spacedrive -> 自动引导创建第一个库
CLI：sd-cli library create "我的文件库"
```

### 2. 添加位置（Location）

告诉 Spacedrive 在哪里索引文件：

```
桌面应用：Settings -> Locations -> Add Location
CLI：sd-cli location add /path/to/your/files
```

支持的位置类型：
- 本地文件夹
- 外接硬盘 / USB
- 网络驱动器（NAS）
- 云存储（S3、Google Drive、Dropbox）

### 3. 连接多设备

Spacedrive 的 P2P 功能让你连接多台设备：

1. 在两台设备上打开 Spacedrive
2. 确保在同一局域网（或通过互联网配对）
3. 设备会自动发现彼此
4. 配对后文件索引会自动同步

### 4. 文件标签系统

使用语义标签组织文件：

```
# 给文件添加标签
标签支持层级：工作/项目A/设计
支持别名：同一个标签可以有多个名称
上下文消歧：自动根据上下文选择正确的标签含义
```

### 5. 搜索文件

```
桌面应用：使用顶部搜索栏
CLI：sd-cli search "关键词"

# 搜索特点
- 基于内容指纹，不受文件名限制
- 跨设备搜索，包括离线设备（显示可用性状态）
- 语义搜索支持
```

### 6. 去重管理

Spacedrive 使用 BLAKE3 哈希自动识别跨设备重复文件：

```
# 查看重复文件
桌面应用：Tools -> Deduplication

# 可预览去重操作
# 显示：空间节省预估、冲突、影响的文件列表
# 确认后才执行
```

---

## 七、云存储集成

Spacedrive 通过 OpenDAL 支持多种云存储作为一级卷：

### 支持的云服务

| 服务 | 状态 |
|------|------|
| Amazon S3 | 已支持 |
| Google Drive | 已支持 |
| Dropbox | 已支持 |
| OneDrive | 已支持 |
| Azure Blob | 已支持 |
| Google Cloud Storage | 已支持 |

### 添加云存储

```
桌面应用：Settings -> Volumes -> Add Cloud Volume
选择云服务类型 -> 填写认证信息 -> 开始索引
```

文件不会被下载到本地，Spacedrive 只索引元数据和内容哈希。

---

## 八、隐私与安全

Spacedrive 是**本地优先**的设计，数据默认留在你的设备上：

| 安全特性 | 说明 |
|----------|------|
| 端到端加密 | 所有 P2P 流量通过 QUIC/TLS 加密 |
| 静态加密 | 库可在磁盘上加密（SQLCipher） |
| 零遥测 | 开源版本无跟踪或分析 |
| 自托管 | 可运行自己的中继服务器和云核心 |
| 数据主权 | 你完全控制数据存放位置 |

可选的 Spacedrive Cloud 用于备份和远程访问，但**永不强制要求**。云服务运行未修改的 Spacedrive 核心作为标准 P2P 设备——无特殊权限，无自定义 API。

---

## 九、项目结构

```
spacedrive/
├── core/                  # Rust VDFS 实现
│   └── src/
│       ├── domain/        # 核心模型（Entry, Library, Device, Tag, Volume）
│       ├── ops/           # CQRS 操作（Actions & Queries）
│       ├── infra/         # 基础设施（DB, Events, Jobs, Sync）
│       ├── service/       # 高层服务（网络, 文件共享, 同步）
│       ├── crypto/        # 密钥管理与加密
│       ├── device/        # 设备身份与配置
│       ├── location/      # 位置管理与索引
│       └── volume/        # 卷检测与指纹
├── apps/
│   ├── cli/               # CLI 与守护进程入口
│   ├── server/            # 无头服务器（Docker/自托管）
│   ├── tauri/             # 桌面应用（macOS, Windows, Linux）
│   ├── web/               # Web 应用（Vite）
│   ├── mobile/            # 移动应用（React Native + Expo）
│   └── api/               # 云 API 服务器（Bun + Elysia）
├── packages/
│   ├── interface/         # 共享 React UI
│   ├── ts-client/         # 自动生成的 TypeScript 客户端
│   └── swift-client/      # 自动生成的 Swift 客户端
├── crates/
│   ├── crypto/            # 加密原语
│   ├── ffmpeg/            # FFmpeg 绑定
│   ├── images/            # 图像处理
│   ├── media-metadata/    # EXIF/媒体元数据提取
│   ├── sdk/               # WASM 扩展 SDK
│   └── task-system/       # 持久任务执行引擎
└── extensions/            # WASM 扩展
```

---

## 十、常见问题

### Q：Spacedrive 会复制我的文件吗？

不会。Spacedrive 只存储路径、内容哈希、提取的文本和元数据——**绝不复制文件本身**。文件留在原地，Spacedrive 只是让它们全部可被找到。

### Q：离线时能用吗？

完全可以。这是核心设计原则。断开连接的驱动器、NAS、USB 卷会留在索引中，搜索结果会显示可用性状态。重新连接后，增量同步会自动恢复。

### Q：Docker 部署需要多少资源？

最低配置：
- **CPU**：1 核
- **内存**：512MB（推荐 1GB+）
- **磁盘**：取决于索引文件数量，索引本身很小

### Q：支持 ARM 设备吗？

支持。Rust 原生支持 ARM 架构，可在树莓派、ARM 服务器上运行。Docker 镜像也提供 ARM64 版本。

### Q：与 Nextcloud / FileBrowser 等有什么区别？

| 对比 | Spacedrive | Nextcloud | FileBrowser |
|------|-----------|-----------|-------------|
| 核心定位 | 分布式文件索引引擎 | 私有云平台 | Web 文件管理器 |
| P2P 同步 | 原生支持 | 不支持 | 不支持 |
| 内容寻址 | BLAKE3 | 无 | 无 |
| 跨设备去重 | 支持 | 不支持 | 不支持 |
| 资源占用 | 低（Rust） | 较高（PHP） | 低（Go） |
| AI 集成 | Spacebot | 插件 | 无 |

---

## 十一、参考链接

- **GitHub**：[github.com/spacedriveapp/spacedrive](https://github.com/spacedriveapp/spacedrive)
- **官网**：[spacedrive.com](https://spacedrive.com)
- **v2 文档**：[v2.spacedrive.com](https://v2.spacedrive.com)
- **Discord**：[discord.gg/spacedrive](https://discord.gg/949090953497567312)
- **知乎部署教程**：[极空间部署 Spacedrive](https://zhuanlan.zhihu.com/p/719777254)

---

*最后更新：2026-04-07*
