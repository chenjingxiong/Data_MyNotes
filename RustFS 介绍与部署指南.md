---
title: RustFS 介绍与部署指南
date: 2026-04-28
tags:
  - rustfs
  - object-storage
  - s3
  - minio
  - rust
  - distributed-storage
  - cloud-native
  - kubernetes
  - data-lake
  - storage
  - devops
url: https://github.com/rustfs/rustfs
related:
  - "[[JuiceFS 介绍与部署指南]]"
---

# RustFS 介绍与部署指南

> [!info] 官方链接
> - **GitHub**: https://github.com/rustfs/rustfs
> - **官网**: https://rustfs.com
> - **中国站**: https://rustfs.com.cn
> - **文档**: https://docs.rustfs.com
> - **Helm Charts**: https://charts.rustfs.com
> - **许可证**: Apache 2.0
> - **语言**: Rust（2024 Edition）
> - **相关项目**: [[JuiceFS 介绍与部署指南]]

---

## 1. 什么是 RustFS？

**RustFS** 是一个基于 **Rust** 构建的高性能分布式对象存储系统，目标是成为 **MinIO 的现代替代方案**。它完美结合了 MinIO 的简洁易用性与 Rust 语言的内存安全、零成本抽象和极致性能优势，提供完整的 **S3 API 兼容性**，完全开源，专为数据湖、AI/ML 和大数据工作负载优化。

### 核心定位

```
传统方案:  MinIO (Go) → S3 兼容对象存储
RustFS:   Rust 重写 → 100% S3 兼容 + 更高性能 + 更安全 + Apache 2.0 开源
```

### 为什么选择 RustFS 而不是 MinIO？

| 维度 | RustFS | MinIO |
|---|---|---|
| **语言** | Rust（内存安全、无 GC、零开销） | Go（有 GC 停顿风险） |
| **许可证** | **Apache 2.0**（商业友好、无限制） | **AGPL v3**（"毒丸"条款，商业使用受限） |
| **数据遥测** | 无遥测，数据主权完全自主 | 可能存在隐蔽遥测 |
| **性能** | 压力测试表现优于 MinIO | 高性能，但受 GC 影响 |
| **边缘/IoT** | 轻量，适合边缘设备 | 对边缘设备偏重 |
| **成本** | 免费社区版，稳定定价 | 1PiB 可高达 $250,000 |

> [!important] Apache 2.0 vs AGPL v3
> MinIO 从 Apache 2.0 更改为 **AGPL v3** 后，任何使用 MinIO 的网络服务都必须开源全部代码。RustFS 采用宽松的 **Apache 2.0**，对企业商业化友好，不存在"许可证陷阱"。

---

## 2. 核心特性

### 2.1 功能状态一览

| 功能 | 状态 | 功能 | 状态 |
|---|---|---|---|
| **S3 核心功能** | ✅ 可用 | **Bitrot 防腐** | ✅ 可用 |
| **上传 / 下载** | ✅ 可用 | **单机模式** | ✅ 可用 |
| **版本控制** | ✅ 可用 | **存储桶复制** | ✅ 可用 |
| **日志功能** | ✅ 可用 | **生命周期管理** | 🚧 测试中 |
| **事件通知** | ✅ 可用 | **分布式模式** | 🚧 测试中 |
| **K8s Helm Chart** | ✅ 可用 | **RustFS KMS** | 🚧 测试中 |
| **Keystone 认证** | ✅ 可用 | **多租户** | ✅ 可用 |
| **Swift API** | ✅ 可用 | **Swift 元数据操作** | 🚧 部分支持 |
| **OIDC 认证** | ✅ 可用 | **OPA 策略引擎** | 🚧 测试中 |

### 2.2 亮点特性详解

**高性能**：
- 基于 Rust 构建，充分利用零成本抽象和所有权系统
- 无垃圾回收（GC）停顿，延迟更稳定
- 纠删码（Erasure Coding）存储引擎，兼顾性能与数据安全

**S3 100% 兼容**：
- 完整实现 Amazon S3 API，可作为任何 S3 客户端的后端
- 兼容 AWS SDK、aws-cli、s3cmd、rclone 等工具

**OpenStack Swift API**：
- 原生支持 Swift 协议
- 集成 Keystone 身份认证（X-Auth-Token）

**安全与合规**：
- 无遥测数据外传，符合 GDPR、CCPA、APPI 等法规
- 支持 OIDC/SSO 集成（Microsoft Entra ID 等）
- Bitrot（位腐烂）保护，确保数据完整性

**数据湖优化**：
- 专为高吞吐量大数据和 AI 工作负载设计
- 支持 S3 Select 查询引擎

---

## 3. 架构详解

### 3.1 整体架构

```
┌────────────────────────────────────────────────────────────┐
│                        客户端层                             │
│  ┌───────┐  ┌──────┐  ┌──────┐  ┌───────┐  ┌───────────┐ │
│  │aws-cli│  │rclone│  │SDK   │  │Swift  │  │Web Console│ │
│  │s3cmd  │  │mc    │  │Go/Py │  │Client │  │(:9001)    │ │
│  └───┬───┘  └──┬───┘  └──┬───┘  └───┬───┘  └─────┬─────┘ │
└──────┼─────────┼─────────┼──────────┼────────────┼────────┘
       │         │         │          │            │
       ▼         ▼         ▼          ▼            ▼
┌────────────────────────────────────────────────────────────┐
│                     RustFS 服务层                           │
│  ┌─────────────────────────────────────────────────────┐  │
│  │                  S3 API / Swift API                  │  │
│  ├──────────┬───────────┬──────────┬───────────────────┤  │
│  │ IAM      │ Policy    │ KMS      │ Audit             │  │
│  │ 身份认证  │ 策略管理   │ 密钥管理 │ 审计日志          │  │
│  ├──────────┴───────────┴──────────┴───────────────────┤  │
│  │                 ECStore 纠删码引擎                    │  │
│  │    ┌──────────────────────────────────────────┐      │  │
│  │    │  数据分片 → 纠删码编码 → 多盘写入          │      │  │
│  │    │  Bitrot 校验 → 自愈修复                    │      │  │
│  │    └──────────────────────────────────────────┘      │  │
│  ├─────────────────────────────────────────────────────┤  │
│  │  Event Notify │ Lifecycle │ Scanner │ Heal          │  │
│  └─────────────────────────────────────────────────────┘  │
└───────────────────────┬────────────────────────────────────┘
                        │
       ┌────────────────┼────────────────┐
       ▼                ▼                ▼
  ┌─────────┐    ┌─────────┐     ┌─────────┐
  │ Disk 0  │    │ Disk 1  │ ... │ Disk N  │
  │/data/jfs0│   │/data/jfs1│    │/data/jfsN│
  └─────────┘    └─────────┘     └─────────┘
     本地磁盘（单机）或分布式节点（多机）
```

### 3.2 Crate 模块架构

RustFS 采用 Rust Workspace 多 Crate 架构，每个模块职责清晰：

```
rustfs/                          # 主入口
├── crates/
│   ├── ecstore/                 # 纠删码存储引擎（核心）
│   ├── s3-common/               # S3 协议通用库
│   ├── s3select-api/            # S3 Select API 接口
│   ├── s3select-query/          # S3 Select 查询引擎
│   ├── iam/                     # 身份与访问管理
│   ├── policy/                  # 策略管理
│   ├── kms/                     # 密钥管理服务
│   ├── crypto/                  # 加密模块
│   ├── credentials/             # 凭证管理
│   ├── appauth/                 # 应用认证授权
│   ├── keystone/                # OpenStack Keystone 集成
│   ├── config/                  # 配置管理
│   ├── filemeta/                # 文件元数据管理
│   ├── heal/                    # 纠删集与对象修复
│   ├── scanner/                 # 数据完整性扫描
│   ├── notify/                  # 事件通知系统
│   ├── audit/                   # 审计目标管理
│   ├── madmin/                  # 管理后台 API
│   ├── obs/                     # 可观测性（OpenTelemetry）
│   ├── rio/                     # Rust I/O 抽象层
│   ├── io-core/                 # 零拷贝核心读写
│   ├── io-metrics/              # I/O 性能指标采集
│   ├── concurrency/             # 并发控制（超时、锁、背压）
│   ├── workers/                 # 工作线程池
│   ├── lock/                    # 分布式锁
│   ├── signer/                  # 客户端签名
│   ├── checksums/               # 客户端校验和
│   ├── zip/                     # ZIP 文件处理
│   ├── protocols/               # FTPS、SFTP 等协议
│   ├── protos/                  # Protocol Buffers 定义
│   ├── trusted-proxies/         # 信任代理管理
│   ├── targets/                 # 目标配置
│   ├── common/                  # 共享工具和数据结构
│   ├── utils/                   # 通用工具函数
│   ├── object-capacity/         # 容量扫描与刷新
│   └── e2e_test/                # 端到端测试套件
└── rustfs/                      # 核心文件系统实现
```

> [!tip] 模块化设计
> RustFS 的 Crate 架构遵循 Unix 哲学——每个模块做一件事并做好。这种设计便于：
> - 独立测试和替换模块
> - 精确控制编译时间和依赖
> - 社区贡献者可以专注单个模块

---

## 4. 安装

### 4.1 一键安装脚本（Linux/macOS）

```bash
curl -O https://rustfs.com/install_rustfs.sh && bash install_rustfs.sh
```

### 4.2 Docker 快速启动

```bash
# 创建数据和日志目录
mkdir -p data logs

# 修改目录所有者（RustFS 容器以 UID 10001 运行）
chown -R 10001:10001 data logs

# 启动 RustFS（最新版）
docker run -d \
    -p 9000:9000 \
    -p 9001:9001 \
    -v $(pwd)/data:/data \
    -v $(pwd)/logs:/logs \
    rustfs/rustfs:latest
```

> [!warning] 权限注意
> RustFS 容器以非 root 用户 `rustfs`（UID `10001`）运行。使用 `-v` 挂载宿主机目录时，**必须**将目录所有者设为 `10001`，否则会遇到 Permission Denied 错误。

### 4.3 指定版本运行

```bash
docker run -d \
    -p 9000:9000 \
    -p 9001:9001 \
    -v $(pwd)/data:/data \
    -v $(pwd)/logs:/logs \
    rustfs/rustfs:1.0.0-alpha.76
```

### 4.4 Podman 运行

```bash
podman run -d \
    -p 9000:9000 \
    -p 9001:9001 \
    -v $(pwd)/data:/data \
    -v $(pwd)/logs:/logs \
    rustfs/rustfs:latest
```

### 4.5 Nix Flake（零安装运行）

```bash
# 直接运行（无需安装）
nix run github:rustfs/rustfs

# 构建二进制
nix build github:rustfs/rustfs
./result/bin/rustfs --help
```

### 4.6 X-CMD 安装

```bash
# 直接运行
x rustfs

# 安装到全局环境
x env use rustfs
rustfs --help
```

### 4.7 从源码编译

```bash
git clone https://github.com/rustfs/rustfs.git
cd rustfs

# 编译发布版
make build

# 或使用 Cargo 直接编译
cargo build --release
```

**多架构 Docker 镜像构建**：

```bash
# 本地构建（支持 amd64 + arm64）
./docker-buildx.sh --build-arg RELEASE=latest

# 构建并推送到仓库
./docker-buildx.sh --push

# 指定版本构建
./docker-buildx.sh --release v1.0.0 --push

# 自定义仓库
./docker-buildx.sh --registry your-registry.com --namespace yourname --push
```

也可以使用 Make 快捷命令：

```bash
make docker-buildx                           # 本地构建
make docker-buildx-push                      # 构建并推送
make docker-buildx-version VERSION=v1.0.0    # 指定版本
make help-docker                             # 查看所有 Docker 命令
```

---

## 5. 快速开始

### 5.1 启动服务

```bash
# 使用 Docker 启动（最简方式）
mkdir -p data logs && chown -R 10001:10001 data logs

docker run -d \
    --name rustfs \
    -p 9000:9000 \
    -p 9001:9001 \
    -v $(pwd)/data:/data \
    -v $(pwd)/logs:/logs \
    -e RUSTFS_VOLUMES="/data/rustfs{0..3}" \
    -e RUSTFS_ACCESS_KEY=rustfsadmin \
    -e RUSTFS_SECRET_KEY=rustfsadmin \
    rustfs/rustfs:latest
```

### 5.2 访问控制台

1. 打开浏览器访问 `http://localhost:9001`
2. 使用默认凭证登录：
   - **用户名**: `rustfsadmin`
   - **密码**: `rustfsadmin`

### 5.3 创建 Bucket 并上传文件

**方式一：通过 Web 控制台**
1. 登录后在左侧导航点击 "Buckets"
2. 点击 "Create Bucket"
3. 输入 Bucket 名称（如 `my-data`），点击创建
4. 进入 Bucket，点击 "Upload" 上传文件

**方式二：通过 AWS CLI**

```bash
# 配置 AWS CLI 指向 RustFS
aws configure set aws_access_key_id rustfsadmin
aws configure set aws_secret_access_key rustfsadmin
aws configure set default.region us-east-1

# 创建 Bucket
aws --endpoint-url http://localhost:9000 s3 mb s3://my-data

# 上传文件
aws --endpoint-url http://localhost:9000 s3 cp ./file.txt s3://my-data/

# 列出文件
aws --endpoint-url http://localhost:9000 s3 ls s3://my-data/

# 下载文件
aws --endpoint-url http://localhost:9000 s3 cp s3://my-data/file.txt ./
```

**方式三：通过 rclone**

```bash
# 配置 rclone（交互式）
rclone config
# 选择 s3 类型
# endpoint: http://localhost:9000
# access_key_id: rustfsadmin
# secret_access_key: rustfsadmin

# 操作
rclone ls rustfs:my-data
rclone copy ./local-dir rustfs:my-data/remote-dir
```

---

## 6. 配置详解

### 6.1 环境变量

RustFS 通过环境变量进行配置（遵循 12-Factor App 原则）：

| 环境变量                                  | 默认值                  | 说明                          |
| ------------------------------------- | -------------------- | --------------------------- |
| `RUSTFS_VOLUMES`                      | `/data/rustfs{0..3}` | 存储卷路径（支持展开语法）               |
| `RUSTFS_ADDRESS`                      | `:9000`              | S3 API 监听地址                 |
| `RUSTFS_CONSOLE_ADDRESS`              | `:9001`              | 控制台监听地址                     |
| `RUSTFS_CONSOLE_ENABLE`               | `true`               | 是否启用 Web 控制台                |
| `RUSTFS_ACCESS_KEY`                   | `rustfsadmin`        | 管理员 Access Key              |
| `RUSTFS_SECRET_KEY`                   | `rustfsadmin`        | 管理员 Secret Key              |
| `RUSTFS_CORS_ALLOWED_ORIGINS`         | `*`                  | S3 API CORS 允许的源            |
| `RUSTFS_CONSOLE_CORS_ALLOWED_ORIGINS` | `*`                  | 控制台 CORS 允许的源               |
| `RUSTFS_TLS_PATH`                     | `/opt/tls`           | TLS 证书目录                    |
| `RUSTFS_OBS_LOGGER_LEVEL`             | `info`               | 日志级别（debug/info/warn/error） |
| `RUSTFS_OBS_ENDPOINT`                 | -                    | OpenTelemetry Collector 地址  |

### 6.2 完整 Docker Compose 配置

```yaml
services:
  rustfs:
    image: rustfs/rustfs:latest
    container_name: rustfs-server
    ports:
      - "9000:9000"    # S3 API 端口
      - "9001:9001"    # 控制台端口
    environment:
      - RUSTFS_VOLUMES=/data/rustfs{0..3}    # 4 个存储卷
      - RUSTFS_ADDRESS=0.0.0.0:9000
      - RUSTFS_CONSOLE_ADDRESS=0.0.0.0:9001
      - RUSTFS_CONSOLE_ENABLE=true
      - RUSTFS_CORS_ALLOWED_ORIGINS=*
      - RUSTFS_CONSOLE_CORS_ALLOWED_ORIGINS=*
      - RUSTFS_ACCESS_KEY=rustfsadmin
      - RUSTFS_SECRET_KEY=rustfsadmin
      - RUSTFS_OBS_LOGGER_LEVEL=info
      - RUSTFS_TLS_PATH=/opt/tls
    volumes:
      - ./data:/data
      - ./logs:/app/logs
      - ./certs/:/opt/tls    # TLS 证书（可选）
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "sh", "-c",
        "curl -f http://127.0.0.1:9000/health && curl -f http://127.0.0.1:9001/rustfs/console/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

### 6.3 可观测性 Stack（推荐）

RustFS 提供了完整的可观测性 Docker Compose Profile：

```bash
# 启动 RustFS + Grafana + Prometheus + Tempo + Jaeger + Loki
docker compose --profile observability up -d
```

这会启动以下服务：

| 服务 | 端口 | 用途 |
|---|---|---|
| **RustFS** | 9000/9001 | 对象存储 + 控制台 |
| **Prometheus** | 9090 | 指标采集 |
| **Grafana** | 3000 | 可视化仪表盘（admin/admin） |
| **Tempo** | 3200 | 分布式链路追踪 |
| **Jaeger** | 16686 | 链路追踪 UI |
| **Loki** | 3100 | 日志聚合 |
| **OTel Collector** | 4317/4318 | OpenTelemetry 数据收集 |

### 6.4 NGINX 反向代理（生产推荐）

```bash
# 启动带 NGINX 反向代理的完整栈
docker compose --profile observability --profile proxy up -d
```

---

## 7. TLS / HTTPS 配置

```bash
# 1. 创建证书目录
mkdir -p certs

# 2. 放入 TLS 证书文件
cp your-cert.pem certs/public.crt
cp your-key.pem certs/private.key

# 3. 启动时挂载证书目录
docker run -d \
    -p 9000:9000 \
    -p 9001:9001 \
    -v $(pwd)/data:/data \
    -v $(pwd)/certs:/opt/tls \
    -e RUSTFS_TLS_PATH=/opt/tls \
    rustfs/rustfs:latest
```

> [!tip] 证书要求
> 证书文件放置在 `RUSTFS_TLS_PATH` 目录下，命名通常为 `public.crt`（证书）和 `private.key`（私钥）。详细配置请参考 [TLS 配置文档](https://docs.rustfs.com/integration/tls-configured.html)。

---

## 8. OIDC / SSO 集成（Microsoft Entra ID 示例）

RustFS 支持 OIDC 认证和角色映射，可直接对接 Microsoft Entra ID（原 Azure AD）：

```bash
# 环境变量配置
RUSTFS_IDENTITY_OPENID_ENABLE=on
RUSTFS_IDENTITY_OPENID_CONFIG_URL="https://login.microsoftonline.com/<tenant-id>/v2.0/.well-known/openid-configuration"
RUSTFS_IDENTITY_OPENID_CLIENT_ID="<client-id>"
RUSTFS_IDENTITY_OPENID_CLIENT_SECRET="<client-secret>"
RUSTFS_IDENTITY_OPENID_SCOPES="openid,profile,email"
RUSTFS_IDENTITY_OPENID_GROUPS_CLAIM="groups"
RUSTFS_IDENTITY_OPENID_ROLES_CLAIM="roles"
```

对应的 IAM 策略示例：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["admin:*"],
      "Resource": ["arn:aws:s3:::*"],
      "Condition": {
        "ForAnyValue:StringEquals": {
          "jwt:roles": ["RustFS.ConsoleAdmin"]
        }
      }
    }
  ]
}
```

---

## 9. Kubernetes 部署

### 9.1 Helm Chart 安装

```bash
# 添加 Helm 仓库
helm repo add rustfs https://charts.rustfs.com
helm repo update

# 安装
helm install rustfs rustfs/rustfs \
    -n rustfs-system \
    --create-namespace \
    --set accessKey=rustfsadmin \
    --set secretKey=your-strong-secret-key \
    --set persistence.size=100Gi \
    --set replicas=4
```

### 9.2 常用 Helm 配置

```yaml
# values.yaml
replicaCount: 4

image:
  repository: rustfs/rustfs
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  ports:
    api: 9000
    console: 9001

ingress:
  enabled: true
  hostname: rustfs.example.com

persistence:
  enabled: true
  size: 100Gi
  storageClassName: local-path

resources:
  requests:
    cpu: 500m
    memory: 512Mi
  limits:
    cpu: "2"
    memory: 2Gi

accessKey: admin-access-key
secretKey: admin-secret-key
```

### 9.3 在 K8s 中使用 RustFS 作为存储

```yaml
# 创建使用 RustFS 的应用
apiVersion: v1
kind: ConfigMap
metadata:
  name: s3-config
data:
  S3_ENDPOINT: "http://rustfs.rustfs-system:9000"
  S3_ACCESS_KEY: "rustfsadmin"
  S3_SECRET_KEY: "your-secret-key"
  S3_BUCKET: "my-data"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: data-processor
spec:
  replicas: 3
  selector:
    matchLabels:
      app: data-processor
  template:
    metadata:
      labels:
        app: data-processor
    spec:
      containers:
        - name: app
          image: your-app:latest
          envFrom:
            - configMapRef:
                name: s3-config
```

---

## 10. 性能基准测试

### 10.1 官方测试环境

| 类型 | 参数 | 备注 |
|---|---|---|
| CPU | 2 核 | Intel Xeon (Sapphire Rapids) Platinum 8475B, 2.7/3.2 GHz |
| 内存 | 4 GB | |
| 网络 | 15 Gbps | |
| 硬盘 | 40GB × 4 | IOPS 3800 / Drive |

> [!note] 性能数据
> 官方基准测试表明，在相同硬件条件下，RustFS 的吞吐量和延迟均优于 MinIO。详细基准测试结果请参考 GitHub 仓库中的性能对比图表。

### 10.2 性能优势分析

```
RustFS 性能优势来源：
├── 无 GC 停顿       → 延迟更稳定，无毛刺
├── 零拷贝 I/O       → io-core / io-metrics 模块
├── 纠删码优化       → ecstore 引擎高效编解码
├── 异步运行时       → tokio 高并发调度
├── 编译时优化       → Rust Release 模式 LLVM 后端优化
└── 内存效率         → 无 GC 开销，内存占用更低
```

---

## 11. RustFS 与 JuiceFS 的关系

> [!abstract] 核心关系
> **RustFS 是对象存储（S3 兼容），JuiceFS 是文件系统（POSIX 兼容）。两者互补，可组合使用。**

### 11.1 定位对比

```
┌──────────────────────────────────────────────────────┐
│                    应用层                              │
│    AI/ML  大数据  容器  数据库  Web 应用  文件管理      │
└──────┬──────────────┬────────────────┬────────────────┘
       │              │                │
       ▼              ▼                ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│   POSIX 挂载 │ │   S3 API     │ │  Web 控制台   │
│   (FUSE)     │ │   (HTTP)     │ │              │
└──────┬───────┘ └──────┬───────┘ └──────┬───────┘
       │                │                │
       ▼                ▼                ▼
┌──────────────┐ ┌──────────────────────────────────┐
│   JuiceFS    │ │           RustFS                 │
│              │ │                                  │
│ 元数据引擎    │ │  S3 API + Swift API + 控制台      │
│ (Redis/TiKV) │ │  纠删码引擎 + Bitrot 保护         │
│              │ │  IAM + KMS + 审计                 │
└──────┬───────┘ └──────┬───────────────────────────┘
       │                │
       ▼                ▼
┌──────────────────────────────────────────────────┐
│              底层存储介质                           │
│    本地磁盘 / 对象存储（S3/MinIO/RustFS）           │
└──────────────────────────────────────────────────┘
```

### 11.2 功能对比

| 维度 | **RustFS** | **JuiceFS** |
|---|---|---|
| **类型** | 对象存储（S3 兼容） | 分布式文件系统（POSIX 兼容） |
| **访问协议** | S3 API、Swift API、Web 控制台 | POSIX（FUSE）、S3 网关、WebDAV、HDFS |
| **数据接口** | HTTP REST API | 内核 FUSE 模块 |
| **元数据** | 内置（纠删码引擎管理） | 独立元数据引擎（Redis/TiKV/SQL） |
| **数据保护** | 纠删码 + Bitrot 校验 | 依赖对象存储的冗余 |
| **文件语义** | 对象（Key-Value，扁平命名空间） | 文件（目录树、inode、权限、锁） |
| **POSIX 兼容** | ❌ | ✅ 完整兼容 |
| **S3 兼容** | ✅ 100% 原生 | ✅ 网关模式 |
| **适合场景** | S3 应用、数据湖、备份、Web 资源 | 文件共享、AI 训练数据、K8s PV |

### 11.3 组合使用方案

> [!success] RustFS 作为 JuiceFS 的数据后端
> JuiceFS 支持使用 S3 兼容的对象存储作为数据后端。RustFS 既然是 100% S3 兼容，就可以**直接作为 JuiceFS 的数据存储层**。

#### 方案 A：RustFS 作为 JuiceFS 数据后端

```
┌────────────────────────────────────────────────────┐
│  应用 / 容器 / AI 训练任务                           │
└───────────────────┬────────────────────────────────┘
                    │ POSIX 文件操作
                    ▼
┌────────────────────────────────────────────────────┐
│              JuiceFS (FUSE Mount)                   │
│  ┌─────────────┐  ┌──────────────────────────┐    │
│  │ 元数据引擎   │  │ 数据路由 + 多级缓存       │    │
│  │ (Redis/TiKV)│  │ (内存 + SSD)              │    │
│  └──────┬──────┘  └──────────┬───────────────┘    │
└─────────┼────────────────────┼─────────────────────┘
          │                    │
          │ 元数据操作          │ 数据块读写
          ▼                    ▼
   ┌─────────────┐    ┌──────────────────┐
   │ Redis / TiKV│    │  RustFS          │
   │             │    │  S3 API (:9000)  │
   │ 元数据存储   │    │  纠删码 + Bitrot │
   └─────────────┘    └────────┬─────────┘
                               │
                      ┌────────▼─────────┐
                      │   本地磁盘 / SSD  │
                      └──────────────────┘
```

**配置步骤**：

```bash
# 1. 启动 RustFS
docker run -d --name rustfs \
    -p 9000:9000 -p 9001:9001 \
    -v $(pwd)/rustfs-data:/data \
    -v $(pwd)/rustfs-logs:/logs \
    -e RUSTFS_ACCESS_KEY=rustfsadmin \
    -e RUSTFS_SECRET_KEY=rustfsadmin \
    rustfs/rustfs:latest

# 2. 在 RustFS 中为 JuiceFS 创建专用 Bucket
aws --endpoint-url http://localhost:9000 s3 mb s3://juicefs-data

# 3. 启动 Redis 作为元数据引擎
docker run -d --name redis \
    -p 6379:6379 \
    redis:7-alpine redis-server --appendonly yes --requirepass "yourpassword"

# 4. 创建 JuiceFS 文件系统（使用 RustFS 作为数据后端）
juicefs format \
    --storage s3 \
    --bucket http://localhost:9000/juicefs-data \
    --access-key rustfsadmin \
    --secret-key rustfsadmin \
    redis://:yourpassword@127.0.0.1:6379/0 \
    myjfs

# 5. 挂载 JuiceFS（获得 POSIX 文件系统接口）
juicefs mount \
    -d \
    --cache-size 10G \
    --cache-dir /tmp/jfs-cache \
    redis://:yourpassword@127.0.0.1:6379/0 \
    /mnt/jfs

# 6. 验证
echo "Hello from RustFS + JuiceFS!" > /mnt/jfs/test.txt
cat /mnt/jfs/test.txt
# "Hello from RustFS + JuiceFS!"

# 同时，通过 RustFS 控制台也能看到 juicefs-data bucket 中存储的数据块
```

#### 方案 B：JuiceFS 与 RustFS 独立并行使用

```
┌──────────────────────────────────────────────────────┐
│                    应用层                              │
│                                                      │
│  ┌──────────────┐              ┌──────────────────┐  │
│  │ 文件型应用    │              │ S3 型应用         │  │
│  │ (AI训练, NFS)│              │ (数据湖, 备份)    │  │
│  └──────┬───────┘              └────────┬─────────┘  │
└─────────┼──────────────────────────────┼─────────────┘
          │ POSIX                        │ S3 API
          ▼                              ▼
   ┌──────────────┐              ┌──────────────────┐
   │  JuiceFS     │              │   RustFS          │
   │  (FUSE Mount)│              │  (HTTP :9000)     │
   │              │              │                   │
   │ 元数据:Redis │              │ 内置纠删码引擎     │
   │ 数据: RustFS │─────────────▶│ S3 + Swift API    │
   └──────────────┘              │ Web 控制台 :9001  │
                                 └──────────────────┘
                                          │
                                   ┌──────▼──────┐
                                   │  本地磁盘    │
                                   └─────────────┘
```

### 11.4 组合使用的优势

| 优势         | 说明                                                          |
| ---------- | ----------------------------------------------------------- |
| **双协议访问**  | JuiceFS 提供 POSIX 文件接口，RustFS 提供 S3 对象接口，同一数据支持两种访问方式        |
| **数据安全增强** | JuiceFS 缓存加速 + RustFS 纠删码 + Bitrot 保护，双重保障                  |
| **弹性存储**   | RustFS 管理底层磁盘，JuiceFS 管理文件语义，各司其职                           |
| **成本优化**   | RustFS (Apache 2.0) + JuiceFS (Apache 2.0)，双 Apache 协议无商业限制 |
| **K8s 原生** | JuiceFS CSI Driver + RustFS Helm Chart，完整 K8s 存储方案          |

### 11.5 选型决策树

```
你需要什么？
│
├── S3 对象存储（应用直接用 S3 SDK / aws-cli 访问）
│   ├── 需要 POSIX 文件挂载吗？
│   │   ├── 否 → ✅ 单独使用 RustFS
│   │   └── 是 → ✅ RustFS + JuiceFS 组合
│   │
│   └── 需要多协议（S3 + WebDAV + HDFS）？
│       └── 是 → ✅ RustFS（数据后端）+ JuiceFS（多协议前端）
│
├── POSIX 文件系统（应用需要像本地磁盘一样挂载）
│   ├── 已有 S3/MinIO 对象存储？
│   │   ├── 是 → ✅ 用 JuiceFS 封装现有对象存储
│   │   └── 否 → ✅ 先部署 RustFS，再用 JuiceFS 封装
│   │
│   └── 需要 K8s 持久卷（PV/PVC）？
│       ├── RWX（多 Pod 共享） → ✅ JuiceFS CSI Driver
│       └── RWO（单 Pod 独占） → ✅ RustFS 或 JuiceFS 均可
│
└── 数据湖 / 大数据平台
    ├── Spark/Flink/Hive → ✅ JuiceFS（HDFS 兼容）
    ├── S3 数据湖格式（Parquet/Iceberg/Delta） → ✅ RustFS
    └── 两者都要 → ✅ RustFS + JuiceFS 组合
```

---

## 12. 与其他方案对比

### 12.1 RustFS vs MinIO vs Ceph 对比

| 维度            | **RustFS**   | **MinIO**  | **Ceph**      |
| ------------- | ------------ | ---------- | ------------- |
| **语言**        | Rust         | Go         | C++           |
| **许可证**       | Apache 2.0 ✅ | AGPL v3 ⚠️ | LGPL 2.1      |
| **定位**        | S3 兼容对象存储    | S3 兼容对象存储  | 统一存储（块+文件+对象） |
| **S3 兼容**     | 100%         | 100%       | 通过 RGW        |
| **Swift API** | ✅ 原生支持       | ❌          | 通过 RGW        |
| **POSIX**     | ❌            | ❌          | ✅ CephFS      |
| **纠删码**       | ✅ 内置         | ✅ 内置       | ✅ 内置          |
| **Bitrot 保护** | ✅            | ✅          | ✅             |
| **部署复杂度**     | ⭐ 简单         | ⭐ 简单       | ⭐⭐⭐⭐ 复杂       |
| **GC 停顿**     | 无（Rust）      | 有（Go）      | 无（C++）        |
| **内存安全**      | 编译时保证        | GC 管理      | 手动管理（有风险）     |
| **边缘/IoT**    | ✅ 轻量         | ⚠️ 偏重      | ❌ 太重          |
| **遥测**        | 无            | 可能有        | 无             |
| **商业使用**      | ✅ 无限制        | ⚠️ AGPL 限制 | ✅ 无限制         |

### 12.2 RustFS vs MinIO vs JuiceFS

| 维度 | **RustFS** | **MinIO** | **JuiceFS** |
|---|---|---|---|
| **角色** | 对象存储 | 对象存储 | 文件系统 |
| **替代关系** | MinIO 替代品 | — | NFS/HDFS 替代品 |
| **可否作为 JuiceFS 后端** | ✅ 可以 | ✅ 可以 | — |
| **POSIX 挂载** | ❌ | ❌ | ✅ |
| **S3 API** | ✅ 原生 | ✅ 原生 | ✅ 网关模式 |
| **数据冗余** | 纠删码 | 纠删码 | 依赖后端 |

---

## 13. 运维管理

### 13.1 健康检查

```bash
# S3 API 健康检查
curl -f http://localhost:9000/health

# 控制台健康检查
curl -f http://localhost:9001/rustfs/console/health

# Docker healthcheck（已内置）
docker inspect --format='{{.State.Health.Status}}' rustfs
```

### 13.2 监控

RustFS 内置 OpenTelemetry 支持：

```yaml
# 环境变量配置
RUSTFS_OBS_LOGGER_LEVEL: info
RUSTFS_OBS_ENDPOINT: http://otel-collector:4318
```

接入 Prometheus + Grafana：
1. 使用 `--profile observability` 启动完整栈
2. 访问 Grafana（`http://localhost:3000`，admin/admin）
3. 查看预配置的 RustFS 仪表盘

### 13.3 数据管理

```bash
# 使用 AWS CLI 管理数据

# 查看所有 Bucket
aws --endpoint-url http://localhost:9000 s3 ls

# 查看 Bucket 大小
aws --endpoint-url http://localhost:9000 s3 ls s3://my-bucket --recursive | \
    awk '{sum+=$3} END {print "Total: " sum/1024/1024/1024 " GB"}'

# 设置 Bucket 版本控制
aws --endpoint-url http://localhost:9000 s3api put-bucket-versioning \
    --bucket my-bucket \
    --versioning-configuration Status=Enabled

# 配置生命周期规则（需 RustFS 生命周期功能就绪）
aws --endpoint-url http://localhost:9000 s3api put-bucket-lifecycle-configuration \
    --bucket my-bucket \
    --lifecycle-configuration file://lifecycle.json
```

---

## 14. 最佳实践

> [!success] 生产环境 Checklist
> 1. **修改默认凭证**：不要使用 `rustfsadmin/rustfsadmin`，设置强密码
> 2. **启用 TLS**：生产环境必须启用 HTTPS
> 3. **多盘配置**：使用 `{0..N}` 展开语法配置多个存储卷（至少 4 个，纠删码需要）
> 4. **资源限制**：Docker/K8s 中设置合理的 CPU 和内存限制
> 5. **监控告警**：接入 Prometheus + Grafana，配置关键指标告警
> 6. **定期备份**：使用 Bucket Replication 复制到远程实例
> 7. **日志持久化**：将日志目录挂载到持久存储
> 8. **健康检查**：配置 Docker/K8s 健康检查
> 9. **反向代理**：使用 NGINX/Caddy 做反向代理和负载均衡
> 10. **配合 JuiceFS**：需要 POSIX 文件语义时，用 JuiceFS 封装 RustFS

---

## 15. 常见问题排查

| 问题 | 可能原因 | 解决方案 |
|---|---|---|
| Permission Denied | 挂载目录 UID 不是 10001 | `chown -R 10001:10001 data logs` |
| 无法访问控制台 :9001 | 未启用控制台 | 设置 `RUSTFS_CONSOLE_ENABLE=true` |
| S3 上传失败 | Access Key 不正确 | 检查 `RUSTFS_ACCESS_KEY` / `RUSTFS_SECRET_KEY` |
| 容器启动失败 | 磁盘空间不足 | 检查 `/data` 挂载点空间 |
| 性能不达预期 | 单盘或磁盘 IOPS 不足 | 配置多个存储卷，使用 SSD |
| CORS 错误 | CORS 配置不正确 | 设置 `RUSTFS_CORS_ALLOWED_ORIGINS` |
| Bucket 已存在 | Bucket 名称全局唯一 | 换一个 Bucket 名称 |

---

## 16. 迁移指南（从 MinIO 迁移到 RustFS）

### 16.1 数据迁移

```bash
# 使用 rclone 迁移数据
rclone sync minio:my-bucket rustfs:my-bucket

# 或使用 mc（MinIO Client）
mc alias set minio http://old-minio:9000 old-access old-secret
mc alias set rustfs http://new-rustfs:9000 rustfsadmin rustfsadmin
mc mirror minio/my-bucket rustfs/my-bucket
```

### 16.2 应用迁移

由于 RustFS 100% S3 兼容，应用只需修改 endpoint URL：

```bash
# 修改前
AWS_ENDPOINT_URL=http://minio:9000

# 修改后
AWS_ENDPOINT_URL=http://rustfs:9000
```

### 16.3 环境变量映射

| MinIO | RustFS | 说明 |
|---|---|---|
| `MINIO_ROOT_USER` | `RUSTFS_ACCESS_KEY` | 管理员用户 |
| `MINIO_ROOT_PASSWORD` | `RUSTFS_SECRET_KEY` | 管理员密码 |
| `MINIO_VOLUMES` | `RUSTFS_VOLUMES` | 存储卷路径 |
| `MINIO_SERVER_URL` | `RUSTFS_ADDRESS` | 监听地址 |
| `MINIO_BROWSER` | `RUSTFS_CONSOLE_ENABLE` | 控制台开关 |

---

## 17. 参考资源

- [RustFS GitHub 仓库](https://github.com/rustfs/rustfs) — 源码与 Issue
- [RustFS 官方文档](https://docs.rustfs.com) — 配置、API、高级用法
- [RustFS Helm Chart](https://charts.rustfs.com) — Kubernetes 部署
- [RustFS Discussions](https://github.com/rustfs/rustfs/discussions) — 社区问答
- [ROSS Index Q4 2025](https://runacap.com/ross-index/q4-2025/) — 增长最快的开源项目
- [[JuiceFS 介绍与部署指南]] — 互补方案：POSIX 文件系统 + 对象存储
