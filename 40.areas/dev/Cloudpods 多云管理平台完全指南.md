---
title: Cloudpods 多云管理平台完全指南
date: 2026-05-17
tags:
  - cloud
  - cloudpods
  - multi-cloud
  - hybrid-cloud
  - open-source
  - golang
  - devops
  - ai
url: https://github.com/yunionio/cloudpods
---

# Cloudpods 多云管理平台完全指南

> **"云上之云"** — 一个开源的、Golang 实现的云原生的融合多云/混合云管理平台。

---

## 目录

- [[#1. 项目概述]]
- [[#2. 核心架构]]
- [[#3. 支持的云平台]]
- [[#4. 支持的云资源]]
- [[#5. AI 云能力]]
- [[#6. 核心功能详解]]
- [[#7. 部署方式]]
- [[#8. 快速上手]]
- [[#9. API 与 CLI]]
- [[#10. 适用场景]]
- [[#11. 生态与社区]]
- [[#12. 参考链接]]

---

## 1. 项目概述

**Cloudpods** 是由 [Yunion（云联信息）](https://www.yunion.io) 团队开发和维护的开源多云/混合云管理平台，采用 **Apache License 2.0** 许可证发布，后端使用 **Golang** 开发，前端基于 **Vue.js/TypeScript**。

### 核心定位

| 定位 | 说明 |
|------|------|
| **多云管理** | 统一管理多个公有云、私有云账号 |
| **混合云编排** | 在一个界面访问私有云和公有云 |
| **轻量私有云** | 将几台物理服务器虚拟化成私有云 |
| **裸金属管理** | 全自动化的物理机生命周期管理 |
| **AI 云** | LLM 推理与 AI 容器应用统一管理 |

### 项目基础信息

| 项目 | 信息 |
|------|------|
| GitHub | https://github.com/yunionio/cloudpods |
| 语言 | Go (后端) + TypeScript/Vue.js (前端) |
| 协议 | Apache License 2.0 |
| 官方文档 | https://www.cloudpods.org |
| API 文档 | https://www.cloudpods.org/en/docs/swagger/ |
| Telegram | https://t.me/cloudpods_org |
| AI 文档 | https://deepwiki.com/yunionio/cloudpods |

---

## 2. 核心架构

Cloudpods 采用**微服务架构**，借鉴了 OpenStack 的服务模型设计，但使用 Go 语言重写，性能更高、部署更轻量。

### 2.1 架构图

```
┌─────────────────────────────────────────────────────────┐
│                     用户层 (User Layer)                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐ │
│  │ Web UI   │  │ CLI      │  │ Terraform│  │ REST API│ │
│  │ (Vue.js) │  │ (climc)  │  │ Provider │  │         │ │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬────┘ │
└───────┼──────────────┼────────────┼──────────────┼──────┘
        │              │            │              │
┌───────┴──────────────┴────────────┴──────────────┴──────┐
│                   API 网关 (Region API)                   │
├─────────────────────────────────────────────────────────┤
│                  控制面 (Control Plane)                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐ │
│  │ Keystone │  │ Compute  │  │ Network  │  │ Image   │ │
│  │ (身份)   │  │ (计算)   │  │ (网络)   │  │ (镜像)  │ │
│  └──────────┘  └──────────┘  └──────────┘  └─────────┘ │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐ │
│  │ Block    │  │ DNS      │  │ Load     │  │ Billing │ │
│  │ Storage  │  │ (域名)   │  │ Balancer │  │ (计费)  │ │
│  └──────────┘  └──────────┘  └──────────┘  └─────────┘ │
├─────────────────────────────────────────────────────────┤
│                  云代理层 (Cloud Provider Proxy)           │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐ ┌───────┐ │
│  │  AWS   │ │ Azure  │ │ GCP    │ │阿里云  │ │华为云 │ │
│  └────────┘ └────────┘ └────────┘ └────────┘ └───────┘ │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐           │
│  │VMware  │ │OpenStack│ │ ZStack │ │ KVM    │ ...       │
│  └────────┘ └────────┘ └────────┘ └────────┘           │
├─────────────────────────────────────────────────────────┤
│                  基础设施层 (Infrastructure)               │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐ │
│  │ 公有云   │  │ 私有云   │  │ 虚拟化   │  │裸金属   │ │
│  │ VM/容器  │  │ OpenStack│  │ VMware   │  │ IPMI    │ │
│  └──────────┘  └──────────┘  └──────────┘  └─────────┘ │
└─────────────────────────────────────────────────────────┘
```

### 2.2 核心组件

| 组件 | 类比 OpenStack | 功能说明 |
|------|---------------|---------|
| **Region** | — | 区域控制器，API 网关入口，负责请求路由和调度 |
| **Keystone** | Keystone | 身份认证与授权服务，支持 RBAC、LDAP/AD、SAML/OAuth |
| **Compute** | Nova | 计算服务，管理虚拟机和裸金属的完整生命周期 |
| **Image** | Glance | 镜像服务，管理操作系统镜像的上传、存储和分发 |
| **Network** | Neutron | 网络服务，管理 VPC、子网、安全组、弹性 IP 等 |
| **Block Storage** | Cinder | 块存储服务，管理云磁盘的创建、挂载和快照 |
| **DNS** | Designate | DNS 域名解析管理 |
| **Load Balancer** | Octavia | 负载均衡器管理 |
| **Cloudproxy** | — | 云代理，与各云平台 API 交互的适配层 |
| **Host Agent** | — | 部署在计算节点上的轻量 Agent，管理 KVM 虚拟机 |
| **Web Console** | Horizon | 基于 Vue.js 的 Web 管理控制台 |
| **Meter/Billing** | Ceilometer | 资源计量与费用管理 |

### 2.3 数据模型层级

```
Domain (域)
  └── Project/Tenant (项目/租户)
        └── Cloud Account (云账号)
              └── Region (区域)
                    └── Zone (可用区)
                          └── VPC (虚拟私有云)
                                └── Network (网络)
                                      └── VM Instance (虚拟机实例)
```

### 2.4 通信方式

- **组件间通信**：REST API + 消息队列（类似 RabbitMQ 的机制）
- **与云平台通信**：通过各云平台的 SDK/REST API
- **与宿主机通信**：通过 Host Agent（SSH/API）
- **配置管理**：Etcd 分布式键值存储

---

## 3. 支持的云平台

Cloudpods 的多云纳管能力覆盖极为广泛，是业界支持云平台数量最多的开源 CMP 之一。

### 3.1 公有云

| 云平台                             | 全球/中国 | 纳管能力 |
| ------------------------------- | ----- | ---- |
| **AWS** (Amazon Web Services)   | 全球    | 完整   |
| **Azure** (Microsoft Azure)     | 全球    | 完整   |
| **GCP** (Google Cloud Platform) | 全球    | 完整   |
| **阿里云** (Alibaba Cloud)         | 中国    | 完整   |
| **华为云** (Huawei Cloud)          | 中国    | 完整   |
| **腾讯云** (Tencent Cloud)         | 中国    | 完整   |
| **UCloud**                      | 中国    | 完整   |
| **天翼云** (Ctyun / China Telecom) | 中国    | 完整   |
| **移动云** (ECloud / China Mobile) | 中国    | 完整   |
| **京东云** (JDCloud)               | 中国    | 完整   |

### 3.2 私有云 / 虚拟化

| 平台 | 类型 | 说明 |
|------|------|------|
| **OpenStack** | 私有云 | 支持对接已有 OpenStack 集群 |
| **ZStack** | 私有云 | 国产私有云平台 |
| **Alibaba Cloud Aspara** (阿里飞天) | 私有云 | 阿里云专有云 |
| **Huawei HCSO** | 私有云 | 华为云 Stack |
| **Nutanix** | 超融合 | 超融合基础设施 |

### 3.3 本地基础设施

| 资源类型 | 技术 | 说明 |
|---------|------|------|
| **轻量私有云** | KVM | Cloudpods 内置的 KVM 虚拟化管理 |
| **VMware vSphere** | vCenter/ESXi | 将 VMware 集群转为自助服务私有云 |
| **裸金属** | IPMI, Redfish API | 物理机自动化 Provisioning |
| **对象存储** | Minio, Ceph, XSky | 兼容 S3 协议的对象存储 |
| **NAS** | Ceph | 文件存储服务 |

---

## 4. 支持的云资源

Cloudpods 对纳管的云资源种类覆盖非常全面，几乎涵盖 IaaS 的所有核心资源类型：

### 4.1 服务器资源

| 资源类型                     | 说明          |
| ------------------------ | ----------- |
| Instances (实例)           | 虚拟机的全生命周期管理 |
| Disks (磁盘)               | 云磁盘/块存储     |
| Network Interfaces (网卡)  | 虚拟网络接口      |
| Networks (网络)            | 虚拟子网        |
| VPCs (虚拟私有云)             | 隔离的虚拟网络环境   |
| Storages (存储)            | 存储后端管理      |
| Hosts (宿主机)              | 物理服务器/计算节点  |
| Wires (物理网络)             | 二层网络映射      |
| Snapshots (快照)           | 磁盘快照        |
| Snapshot Policies (快照策略) | 自动快照策略      |
| Security Groups (安全组)    | 防火墙规则       |
| Elastic IPs (弹性 IP)      | 公网 IP 管理    |
| SSH Keypairs (密钥对)       | SSH 公钥管理    |
| Images (镜像)              | 操作系统镜像管理    |

### 4.2 网络资源

| 资源类型                           | 说明         |
| ------------------------------ | ---------- |
| VPC Peering (VPC 对等连接)         | 跨 VPC 互通   |
| Inter-VPC Network (跨 VPC 网络)   | 多 VPC 网络互联 |
| NAT Gateway (NAT 网关)           | 地址转换网关     |
| DNAT/SNAT Rules (DNAT/SNAT 规则) | 端口/地址映射规则  |
| Route Tables (路由表)             | VPC 路由管理   |
| Route Entries (路由条目)           | 路由规则       |

### 4.3 其他资源

| 类别 | 资源 |
|------|------|
| **负载均衡** | 实例、监听器、后端服务器组、后端服务器、TLS 证书、ACL |
| **对象存储** | Bucket、Object |
| **NAS 文件存储** | 文件系统、访问组、挂载点 |
| **RDS 数据库** | 实例、账号、备份、数据库、参数、权限 |
| **弹性缓存** | 实例、账号、备份、参数 |
| **DNS** | DNS 区域、DNS 记录 |

---

## 5. AI 云能力

> [!note] 新特性
> Cloudpods 近期新增了 **AI 云** 模块，提供 LLM 推理与 AI 容器应用的统一管理，是多云管理与 AI 工作负载的结合创新。

### 5.1 AI 推理服务

| 能力 | 说明 |
|------|------|
| **GPU 调度** | 自动 GPU 设备探测、注册与分配 |
| **模型挂载** | 模型文件自动挂载到推理实例 |
| **服务地址分配** | 自动分配推理服务的访问地址 |
| **多实例管理** | 支持多个推理实例的并发运行 |

### 5.2 AI 应用管理

支持一站式的 AI 容器应用部署和管理：

| 应用           | 类型    | 说明             |
| ------------ | ----- | -------------- |
| **Ollama**   | AI 推理 | 轻量级本地 LLM 推理引擎 |
| **OpenClaw** | AI 应用 | AI Agent 智能体助手 |
| **Dify**     | AI 应用 | LLM 应用编排平台     |
| **ComfyUI**  | AI 应用 | AI 图像生成工具      |

### 5.3 模型库

- **统一管理**：模型来源、版本与缓存的集中管理
- **多实例复用**：同一模型可被多个推理实例共享使用
- **离线分发**：支持模型文件的离线分发，避免重复下载

### 5.4 GPU 运维

- **自动探测**：GPU 设备的自动发现和注册
- **统一配置**：NVIDIA/CUDA 环境的统一配置和管理
- **资源规格**：通过模板定义 CPU/内存/GPU 资源规格
- **镜像管理**：通过镜像管理容器运行环境，标准化交付

---

## 6. 核心功能详解

### 6.1 多云统一管理

Cloudpods 通过**云账号抽象层**实现多云纳管：

1. **注册云账号**：在各云平台创建 API 密钥，录入 Cloudpods
2. **资源发现**：Cloudpods 自动同步各云账号下的资源
3. **统一视图**：在单一控制台查看所有云上的资源
4. **统一操作**：通过统一 API 对异构云资源进行 CRUD 操作

```
用户 → Cloudpods 统一 API → 云代理适配层 → 各云平台原生 API
                                      ├→ AWS SDK
                                      ├→ Azure SDK
                                      ├→ 阿里云 SDK
                                      ├→ VMware vSphere API
                                      └→ KVM Host Agent
```

### 6.2 身份与访问管理 (IAM)

| 功能 | 说明 |
|------|------|
| **多租户隔离** | Domain → Project 的层级隔离 |
| **RBAC** | 基于角色的细粒度访问控制 |
| **LDAP/AD 对接** | 企业目录服务集成 |
| **SAML/OAuth** | 企业级单点登录 (SSO) |
| **API 密钥** | 支持 Access Key / Secret Key 认证 |

### 6.3 费用管理 (FinOps)

| 功能                      | 说明                |
| ----------------------- | ----------------- |
| **账单汇总**                | 多云账单的统一收集和汇总      |
| **费用分析**                | 按项目、云平台、资源类型的费用分析 |
| **趋势预测**                | 费用趋势分析与预测         |
| **优化建议**                | 资源利用率分析，提供降本建议    |
| **Showback/Chargeback** | 内部成本分摊            |

### 6.4 裸金属管理

Cloudpods 提供了完整的裸金属服务器全生命周期管理：

| 阶段                | 技术                       |
| ----------------- | ------------------------ |
| **发现**            | IPMI/Redfish API 自动发现物理机 |
| ** Provisioning** | PXE 网络启动，自动安装操作系统        |
| **管理**            | 开关机、重装系统、固件升级            |
| **监控**            | 硬件状态监控（温度、风扇、电源）         |
| **回收**            | 资源释放和重新分配                |

### 6.5 VMware 迁移

Cloudpods 可以将 VMware vSphere 虚拟化集群转变为自助服务的私有云：

- 自动导入 VMware 集群中的 VM、网络、存储信息
- 提供自助服务门户，用户可自行创建和管理 VM
- 支持从 VMware 到 KVM 的 V2V 迁移

---

## 7. 部署方式

### 7.1 部署模式对比

| 模式                    | 适用场景       | 高可用 | 复杂度 | 节点数 |
| --------------------- | ---------- | --- | --- | --- |
| **ocboot All-in-One** | 测试、PoC、小规模 | ❌   | 低   | 1   |
| **Docker Compose**    | 快速体验、开发测试  | ❌   | 低   | 1   |
| **标准集群**              | 生产环境       | ✅   | 中   | 3+  |
| **Kubernetes (Helm)** | K8s 环境     | ✅   | 中高  | 3+  |

### 7.2 系统要求

#### All-in-One 模式

| 项目     | 最低要求                                         | 推荐配置             |
| ------ | -------------------------------------------- | ---------------- |
| CPU    | 8 核                                          | 16+ 核            |
| 内存     | 16 GB                                        | 32+ GB           |
| 磁盘     | 100 GB SSD                                   | 500+ GB SSD      |
| 操作系统   | CentOS 7/8, Ubuntu 18.04/20.04, Debian 10/11 | Ubuntu 20.04 LTS |
| Docker | 20.10+                                       | 最新稳定版            |

#### 生产集群模式

| 角色 | 最低数量 | 推荐配置 |
|------|---------|---------|
| 控制节点 | 3 | 8C/16G 以上 |
| 计算节点 | 按需 | 16C/64G 以上 |
| 存储节点 | 按需 | 取决于存储需求 |

### 7.3 部署架构选择

```
方案 A: All-in-One (单机全栈)
┌─────────────────────────────┐
│         单台服务器            │
│  ┌───────────────────────┐  │
│  │  控制面 + 计算面       │  │
│  │  (Docker 容器化)      │  │
│  └───────────────────────┘  │
└─────────────────────────────┘

方案 B: 标准集群 (分离部署)
┌──────────┐  ┌──────────┐  ┌──────────┐
│ 控制节点1 │  │ 控制节点2 │  │ 控制节点3 │
│ (HA)     │  │ (HA)     │  │ (HA)     │
└─────┬────┘  └─────┬────┘  └─────┬────┘
      └──────────────┼──────────────┘
                     │
       ┌─────────────┼─────────────┐
  ┌────┴────┐  ┌────┴────┐  ┌────┴────┐
  │计算节点1│  │计算节点2│  │计算节点3│
  │(KVM)   │  │(KVM)   │  │(KVM)   │
  └─────────┘  └─────────┘  └─────────┘

方案 C: Kubernetes 部署
┌──────────────────────────────┐
│       Kubernetes 集群         │
│  ┌─────┐ ┌─────┐ ┌─────┐   │
│  │Pod  │ │Pod  │ │Pod  │   │
│  │服务1│ │服务2│ │服务N│   │
│  └─────┘ └─────┘ └─────┘   │
│  (通过 Helm Chart 管理)      │
└──────────────────────────────┘
```

---

## 8. 快速上手

### 8.1 方式一：ocboot 官方安装（推荐）

> [!info] 官方安装方式
> Cloudpods 官方使用 **ocboot**（基于 Ansible + buildah）进行部署。内部会拉起一套轻量 Kubernetes，将所有 Cloudpods 服务以容器方式运行。

**环境要求**：CentOS 7.x / Kylin V10 / Debian 10，最低 4C/8G。

```bash
# 1. 克隆 ocboot 安装器
git clone https://github.com/yunionio/ocboot.git
cd ocboot

# 2. 一键 All-in-One 部署（包含私有云 + 多云管理）
# 首次部署会自动安装依赖（buildah, ansible 等）
python3 run.py full

# 3. 部署完成后访问
# 地址: https://<your-server-ip>
# 默认账号: admin / admin@openstack.org
# 首次登录请修改密码
```

**常用运维命令**：

```bash
# 查看服务状态
kubectl get pods -n onecloud

# 添加计算节点
python3 run.py add-node <master-ip> <node-ip>

# 添加负载均衡节点
python3 run.py add-lbagent <master-ip> <node-ip>

# 数据库备份
python3 run.py backup <master-ip>

# 数据库恢复
python3 run.py restore <master-ip>
```

---

### 8.2 方式二：Docker Compose 快速部署（社区方案）

> [!warning] 社区方案
> Docker Compose 部署并非官方推荐方式，适用于**快速体验和开发测试**。生产环境请使用 ocboot 或 Helm 部署。
> 镜像均来自 Docker Hub 官方 `yunion/*` 仓库。

**环境要求**：任意 Linux + Docker 20.10+ + Docker Compose v2。

#### 步骤 1：创建项目目录

```bash
mkdir -p /opt/cloudpods && cd /opt/cloudpods
```

#### 步骤 2：创建 `docker-compose.yml`

```yaml
version: "3.8"

# ============================================================
# Cloudpods All-in-One Docker Compose
# 使用 yunion 官方预编译镜像（Docker Hub: yunion/*）
# 适用场景：快速体验、开发测试
# ============================================================

x-common-env: &common-env
  # ---- 数据库连接 ----
  MYSQL_HOST: mariadb
  MYSQL_PORT: "3306"
  MYSQL_USER: root
  MYSQL_PASSWORD: "your-sql-password"
  # ---- Redis 连接 ----
  REDIS_HOST: redis
  REDIS_PORT: "6379"
  # ---- 区域配置 ----
  REGION: "cloudpods-region"
  ZONE: "zone0"
  # ---- 认证 ----
  ADMIN_USER: "admin"
  ADMIN_PASSWORD: "admin@openstack.org"
  AUTH_URL: "https://keystone:5000/v3"

services:
  # ========================
  #  基础设施层
  # ========================

  etcd:
    image: yunion/etcd:latest
    container_name: cloudpods-etcd
    restart: unless-stopped
    volumes:
      - etcd_data:/etcd-data
    command: >
      etcd
      --data-dir /etcd-data
      --listen-client-urls http://0.0.0.0:2379
      --advertise-client-urls http://etcd:2379
    healthcheck:
      test: ["CMD", "etcdctl", "endpoint", "health"]
      interval: 10s
      timeout: 5s
      retries: 5

  mariadb:
    image: mariadb:10.5
    container_name: cloudpods-mariadb
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: "your-sql-password"
      MYSQL_DATABASE: "cloudpods"
    volumes:
      - mysql_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:6-alpine
    container_name: cloudpods-redis
    restart: unless-stopped
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  # ========================
  #  Cloudpods 核心服务层
  # ========================

  keystone:
    image: yunion/keystone:latest
    container_name: cloudpods-keystone
    restart: unless-stopped
    depends_on:
      mariadb:
        condition: service_healthy
      etcd:
        condition: service_healthy
    environment:
      <<: *common-env
    ports:
      - "5000:5000"
    volumes:
      - cert_data:/etc/yunion/pki

  region:
    image: yunion/region:latest
    container_name: cloudpods-region
    restart: unless-stopped
    depends_on:
      - keystone
    environment:
      <<: *common-env
    ports:
      - "3080:3080"

  glance:
    image: yunion/glance:latest
    container_name: cloudpods-glance
    restart: unless-stopped
    depends_on:
      - keystone
    environment:
      <<: *common-env
    volumes:
      - glance_data:/opt/cloud/workspace

  scheduler:
    image: yunion/scheduler:latest
    container_name: cloudpods-scheduler
    restart: unless-stopped
    depends_on:
      - keystone
    environment:
      <<: *common-env

  apigateway:
    image: yunion/apigateway:latest
    container_name: cloudpods-apigateway
    restart: unless-stopped
    depends_on:
      - keystone
      - region
    environment:
      <<: *common-env
    ports:
      - "443:443"
      - "80:80"

  cloudproxy:
    image: yunion/cloudproxy:latest
    container_name: cloudpods-cloudproxy
    restart: unless-stopped
    depends_on:
      - keystone
    environment:
      <<: *common-env

  cloudnet:
    image: yunion/cloudnet:latest
    container_name: cloudpods-cloudnet
    restart: unless-stopped
    depends_on:
      - keystone
    environment:
      <<: *common-env

  cloudid:
    image: yunion/cloudid:latest
    container_name: cloudpods-cloudid
    restart: unless-stopped
    depends_on:
      - keystone
    environment:
      <<: *common-env

  webconsole:
    image: yunion/webconsole:latest
    container_name: cloudpods-webconsole
    restart: unless-stopped
    depends_on:
      - keystone
    environment:
      <<: *common-env

  # ========================
  #  可选：运维增强服务
  # ========================

  meter:
    image: yunion/meter:latest
    container_name: cloudpods-meter
    restart: unless-stopped
    depends_on:
      - keystone
    environment:
      <<: *common-env
    profiles:
      - full

  notify:
    image: yunion/notify:latest
    container_name: cloudpods-notify
    restart: unless-stopped
    depends_on:
      - keystone
    environment:
      <<: *common-env
    profiles:
      - full

  monitor:
    image: yunion/telegraf:latest
    container_name: cloudpods-telegraf
    restart: unless-stopped
    depends_on:
      - keystone
    environment:
      <<: *common-env
    profiles:
      - full

  # ========================
  #  前端 Web 控制台
  # ========================

  web:
    image: yunion/web:latest
    container_name: cloudpods-web
    restart: unless-stopped
    depends_on:
      - apigateway
    ports:
      - "9900:80"

volumes:
  etcd_data:
  mysql_data:
  redis_data:
  glance_data:
  cert_data:
```

#### 步骤 3：修改配置并启动

```bash
# 1. 修改数据库密码（建议替换默认值）
# 编辑 docker-compose.yml 中的 MYSQL_ROOT_PASSWORD 和 MYSQL_PASSWORD

# 2. 拉取所有镜像
docker compose pull

# 3. 启动核心服务（不含运维增强）
docker compose up -d

# 如果需要完整服务（含计费、通知、监控），使用：
# docker compose --profile full up -d

# 4. 查看服务状态
docker compose ps
docker compose logs -f --tail=50
```

#### 步骤 4：访问控制台

```
Web 控制台: https://<your-server-ip>      (通过 apigateway)
           或 http://<your-server-ip>:9900  (直接访问 web 容器)
API 地址:   https://<your-server-ip>:5000/v3
默认账号:   admin / admin@openstack.org
```

#### 步骤 5：常用运维命令

```bash
# 查看所有服务日志
docker compose logs -f

# 查看特定服务日志
docker compose logs -f region
docker compose logs -f keystone

# 重启某个服务
docker compose restart region

# 停止所有服务
docker compose down

# 停止并清除数据（慎用！会删除所有数据）
docker compose down -v

# 升级镜像版本
docker compose pull && docker compose up -d
```

> [!tip] Docker Compose 部署注意事项
> - **首次启动**：各服务依赖数据库初始化，首次启动可能需要 2-3 分钟完成自动初始化
> - **内存要求**：建议至少 **16GB** 内存，8 个核心以上
> - **数据持久化**：所有数据通过 Docker Volumes 持久化，`docker compose down` 不会丢失数据
> - **镜像版本**：可指定版本号（如 `yunion/keystone:v3.10`）替代 `latest` 以固定版本
> - **TLS 证书**：首次启动会自动生成自签名证书，浏览器访问时需接受安全例外

---

### 8.3 方式三：Helm 部署（Kubernetes）

```bash
# 1. 添加 Helm 仓库
helm repo add cloudpods https://yunionio.github.io/cloudpods
helm repo update

# 2. 创建命名空间
kubectl create namespace cloudpods

# 3. 安装 Cloudpods
helm install cloudpods cloudpods/cloudpods \
  -n cloudpods \
  -f values.yaml

# 4. 检查部署状态
kubectl get pods -n cloudpods
```

---

### 8.4 首次登录后的配置流程

1. **登录管理控制台**：通过浏览器访问 Web UI
2. **创建域和项目**：建立组织架构（Domain → Project）
3. **创建用户和角色**：配置用户权限
4. **接入云平台**：
   - 进入「云账号」管理
   - 选择要接入的云平台类型
   - 填入 API 密钥（Access Key / Secret Key）
   - 点击「同步」开始资源发现
5. **查看纳管资源**：在资源列表中查看同步过来的云资源
6. **开始使用**：通过统一界面创建、管理云资源

---

## 9. API 与 CLI

### 9.1 climc — 命令行工具

`climc` 是 Cloudpods 的命令行管理工具，类似于 OpenStack 的 `openstack` CLI。

```bash
# 认证登录
climc login --auth-url https://<cloudpods-api>:5000/v3 \
            --username admin \
            --password <password> \
            --domain Default

# 列出所有服务器
climc server-list

# 创建虚拟机
climc server-create --name my-vm \
                    --image <image-id> \
                    --spec <spec-id> \
                    --network <network-id>

# 列出云账号
climc cloud-account-list

# 同步云账号资源
climc cloud-account-sync --cloudaccount <account-id>
```

### 9.2 REST API

Cloudpods 提供了完整的 RESTful API，兼容 OpenStack 风格：

```bash
# 获取认证 Token
curl -X POST https://<cloudpods-api>:5000/v3/auth/tokens \
  -H "Content-Type: application/json" \
  -d '{
    "auth": {
      "identity": {
        "methods": ["password"],
        "password": {
          "user": {
            "name": "admin",
            "domain": {"name": "Default"},
            "password": "<password>"
          }
        }
      }
    }
  }'

# 列出所有服务器
curl -X GET https://<cloudpods-api>:5000/v2/servers \
  -H "X-Auth-Token: <token>"

# Swagger API 文档
# https://www.cloudpods.org/en/docs/swagger/
```

### 9.3 Terraform Provider

Cloudpods 提供了 Terraform Provider，支持 Infrastructure as Code：

```hcl
terraform {
  required_providers {
    cloudpods = {
      source  = "yunionio/cloudpods"
    }
  }
}

provider "cloudpods" {
  auth_url    = "https://<cloudpods-api>:5000/v3"
  username    = "admin"
  password    = "<password>"
  domain_name = "Default"
  project_name = "system"
}

resource "cloudpods_server" "my_vm" {
  name        = "my-vm"
  image       = "<image-id>"
  spec        = "<spec-id>"
  network     = "<network-id>"
}
```

---

## 10. 适用场景

### 10.1 典型使用场景

```
场景 1: 多云费用治理
━━━━━━━━━━━━━━━━━━━━
企业使用多个公有云（阿里云 + AWS + 腾讯云）
→ 通过 Cloudpods 统一纳管
→ 费用分析和优化
→ 年度云成本降低 20-30%

场景 2: VMware 替换/迁移
━━━━━━━━━━━━━━━━━━━━━━━━
企业现有大量 VMware 虚拟化资产
→ Cloudpods 纳管 VMware 集群
→ 提供 Self-Service 门户
→ 逐步迁移到 KVM 节点

场景 3: 混合云统一管理
━━━━━━━━━━━━━━━━━━━━━━
核心系统在私有云，弹性业务在公有云
→ Cloudpods 提供统一视图
→ 统一身份认证和权限管理
→ 跨云资源编排

场景 4: 轻量私有云建设
━━━━━━━━━━━━━━━━━━━━━━
几台物理服务器需要虚拟化
→ Cloudpods All-in-One 部署
→ 无需 OpenStack 等重型方案
→ 开箱即用的私有云

场景 5: AI 工作负载管理
━━━━━━━━━━━━━━━━━━━━━━
需要部署 LLM 推理服务
→ Cloudpods AI 云模块
→ GPU 资源调度和管理
→ Ollama / Dify 一键部署
```

### 10.2 谁需要 Cloudpods？

| 用户画像 | 需求 |
|---------|------|
| 运维团队 | 需要在一个界面管理多个云平台 |
| FinOps 团队 | 需要统一的多云费用分析和优化 |
| 安全团队 | 需要统一的身份认证和访问控制 |
| 开发团队 | 需要自助式的云资源申请 |
| AI/ML 工程师 | 需要 GPU 调度和 LLM 部署平台 |
| 企业 CIO | 需要避免供应商锁定，实施多云策略 |

---

## 11. 生态与社区

### 11.1 相关资源

| 资源 | 链接 |
|------|------|
| GitHub 仓库 | https://github.com/yunionio/cloudpods |
| 官方文档 (EN) | https://www.cloudpods.org/en |
| 官方文档 (CN) | https://www.cloudpods.org/zh |
| Swagger API | https://www.cloudpods.org/en/docs/swagger/ |
| AI 生成文档 | https://deepwiki.com/yunionio/cloudpods |
| Telegram 群 | https://t.me/cloudpods_org |
| 用户列表 | https://github.com/yunionio/cloudpods/issues/11427 |

### 11.2 贡献指南

- 代码贡献遵循 [CONTRIBUTING](https://github.com/yunionio/cloudpods/blob/master/CONTRIBUTING.md) 规范
- 欢迎 Bug 报告、功能建议、文档改进等任何形式的贡献
- 用户可在 [Issue #11427](https://github.com/yunionio/cloudpods/issues/11427) 留下使用信息

### 11.3 技术栈总结

```
┌── 后端 ──────────────────────────┐
│ Golang                           │
│ 微服务架构                        │
│ RESTful API                      │
│ Docker 容器化                    │
└──────────────────────────────────┘

┌── 前端 ──────────────────────────┐
│ Vue.js                           │
│ TypeScript                       │
│ Web 控制台                       │
└──────────────────────────────────┘

┌── 基础设施 ──────────────────────┐
│ Kubernetes / Docker              │
│ MySQL / MariaDB                  │
│ Etcd (配置/发现)                 │
│ 消息队列                         │
└──────────────────────────────────┘

┌── 存储后端 ──────────────────────┐
│ 本地存储                         │
│ Ceph (块/对象/文件)              │
│ NFS                              │
│ MinIO (S3 兼容)                  │
└──────────────────────────────────┘
```

---

## 12. 参考链接

- [Cloudpods GitHub 仓库](https://github.com/yunionio/cloudpods)
- [Cloudpods 官方文档](https://www.cloudpods.org)
- [Yunion 官网](https://www.yunion.io)
- [Cloudpods Swagger API](https://www.cloudpods.org/en/docs/swagger/)
- [DeepWiki AI 文档](https://deepwiki.com/yunionio/cloudpods)
- [用户列表 Issue](https://github.com/yunionio/cloudpods/issues/11427)
- [贡献指南](https://github.com/yunionio/cloudpods/blob/master/CONTRIBUTING.md)

---

> [!info] 更新说明
> 本指南基于 2026 年 5 月 Cloudpods GitHub 仓库最新信息编写。项目持续迭代中，建议定期关注 [Release Notes](https://www.cloudpods.org/en/docs/release-notes/) 获取最新更新。
