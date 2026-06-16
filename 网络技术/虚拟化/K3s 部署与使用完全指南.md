# K3s 部署与使用完全指南：家庭网络集群实战

> K3s 是由 Rancher Labs（现 SUSE）开发的轻量级 Kubernetes 发行版，专为资源受限环境设计——完美契合家庭实验室、边缘计算和 IoT 场景。本指南覆盖从零搭建家庭 K3s 集群的全流程，并与标准 K8s 进行深入对比。

---

## 目录

- [[#1. K3s 概述与核心架构]]
- [[#2. K3s vs K8s 深度对比]]
- [[#3. 家庭集群规划]]
- [[#4. 部署实战]]
- [[#5. 网络与 Ingress 配置]]
- [[#6. 持久化存储方案]]
- [[#7. 常用应用部署]]
- [[#8. 运维与监控]]
- [[#9. 家庭集群的限制与应对]]
- [[#10. 故障排查手册]]
- [[#11. 参考资源]]

---

## 1. K3s 概述与核心架构

### 什么是 K3s

| 特性 | 说明 |
|------|------|
| **定位** | CNCF 认证的轻量级 Kubernetes 发行版 |
| **二进制大小** | ~100 MB（单文件） vs K8s 数 GB |
| **内存占用** | 控制面最低 512 MB（K8s 通常需要 2 GB+） |
| **维护方** | SUSE / Rancher，CNCF 沙箱项目 |
| **License** | Apache 2.0 |
| **架构支持** | x86_64、ARM64、ARMv7 |
| **版本策略** | 与上游 K8s 同步发布（仅落后约 1-2 周） |

### 架构对比

```
┌─────────────── 标准K8s 控制面 ───────────────┐     ┌──────── K3s 控制面 ────────┐
│                                              │     │                            │
│  ┌─────┐ ┌──────┐ ┌─────┐ ┌─────┐ ┌──────┐ │     │  ┌──────────────────────┐ │
│  │ API │ │Scheduler│ │C-M │ │etcd │ │ ...  │ │     │  │     k3s server       │ │
│  │ Svr │ │      │ │     │ │     │ │      │ │     │  │  (单进程包含所有组件)   │ │
│  └─────┘ └──────┘ └─────┘ └─────┘ └──────┘ │     │  │  ┌──────┐ ┌───────┐  │ │
│                                              │     │  │  │SQLite│ │traefik│  │ │
│  5+ 独立进程                                 │     │  │  │/etcd │ │coredns│  │ │
│                                              │     │  │  └──────┘ └───────┘  │ │
└──────────────────────────────────────────────┘     │  └──────────────────────┘ │
                                                     └────────────────────────────┘
```

**K3s 的关键设计决策：**

1. **单二进制文件**：将 API Server、Scheduler、Controller Manager、etcd/SQLite 等所有组件编译为一个可执行文件
2. **内置容器运行时**：默认使用 `containerd`，无需额外安装 Docker
3. **自动 TLS**：节点间通信自动生成并管理 TLS 证书
4. **内置工具链**：CoreDNS、Traefik Ingress、ServiceLB、本地存储驱动开箱即用
5. **自动清单目录**：放在 `/var/lib/rancher/k3s/server/manifests/` 的 YAML 会自动部署

---

## 2. K3s vs K8s 深度对比

### 2.1 K3s 精简了什么

K3s 并非简单"阉割"K8s，而是**有选择地移除了云厂商相关的、遗留的和非核心的组件**：

| 移除/替换的组件 | K8s 默认 | K3s 替代方案 | 影响 |
|----------------|---------|-------------|------|
| **etcd** | 内置 etcd | SQLite（单节点）/ 嵌入式 etcd（HA） | 功能等价，简化运维 |
| **in-tree 云提供商** | AWS/GCP/Azure 原生插件 | 移除，使用 out-of-tree CSI/CCM | 家庭场景无影响 |
| **in-tree 存储插件** | 内置各种云存储驱动 | 移除，使用 CSI 驱动 | 需要手动安装 CSI |
| **Legacy/Alpha 功能** | 全部包含 | 剔除不稳定/废弃特性 | 正面影响，更稳定 |
| **Docker 运行时** | 可选 Docker | containerd（内置） | Docker CLI 不可用 |
| **自带 add-ons** | 无 | Traefik、CoreDNS、ServiceLB | 开箱即用 |

### 2.2 功能完整性对比

| 功能 | K8s | K3s | 家庭场景影响 |
|------|-----|-----|------------|
| **Pod/Deployment/Service** | ✅ | ✅ | 无差异 |
| **Namespace/RBAC** | ✅ | ✅ | 无差异 |
| **ConfigMap/Secret** | ✅ | ✅ | 无差异 |
| **HPA/VPA 自动伸缩** | ✅ | ✅ | 无差异 |
| **NetworkPolicy** | ✅ | ✅（需安装 Calico 等替代 Flannel） | 需额外配置 |
| **Ingress** | 需手动安装 | ✅ 内置 Traefik | K3s 更方便 |
| **持久化存储** | 需手动安装 CSI | ✅ 内置 local-path | K3s 更方便 |
| **负载均衡器** | 需外部 LB | ✅ 内置 ServiceLB（简化版） | 可用但功能有限 |
| **Helm** | 需安装 Helm | ✅ 内置 Helm Controller | K3s 更方便 |
| **证书管理** | 需安装 cert-manager | 需安装 cert-manager | 无差异 |
| **仪表盘** | 需安装 Dashboard | 需安装 Dashboard | 无差异 |
| **API 聚合层** | ✅ | ✅ | 无差异 |
| **CRD/Operator** | ✅ | ✅ | 无差异 |
| **kubectl** | 需安装 | ✅ 内置 `k3s kubectl` | K3s 更方便 |
| **多集群管理** | 需工具链 | 需工具链（Rancher 原生支持） | K3s + Rancher 更方便 |
| **规模上限** | 5000 节点 | 数百节点 | 家庭环境无影响 |
| **GPU 调度** | ✅ | ✅（需安装 NVIDIA device plugin） | 无差异 |

### 2.3 资源消耗对比

| 指标 | K8s 控制面 | K3s 控制面 |
|------|-----------|-----------|
| **最低内存** | 2 GB | 512 MB |
| **推荐内存** | 4 GB+ | 1-2 GB |
| **二进制体积** | ~2 GB+ | ~100 MB |
| **启动时间** | 2-5 分钟 | 30-60 秒 |
| **进程数** | 5+ 独立进程 | 1 个进程 |
| **CPU 闲置占用** | 5-10% | 1-3% |

> [!tip] 关键结论
> 对于家庭集群（通常 2-5 台机器），K3s 移除的都是**云厂商特有功能**，家庭环境根本用不到。真正缺失的几乎为零——K3s 是家庭 K8s 的最佳选择。

---

## 3. 家庭集群规划

### 3.1 硬件推荐

| 方案 | 节点配置 | 适用场景 | 预算参考 |
|------|---------|---------|---------|
| **入门级** | 1 台旧电脑/NUC（4C/8G/256G SSD） | 学习、轻量服务 | ¥0（利用闲置） |
| **标准级** | 3 台 mini PC（如 Dell Wyse / HP T640） | HA 集群、媒体服务 | ¥1500-3000 |
| **进阶级** | 3 台 x86 台式机 + NAS | 全功能私有云 | ¥5000+ |
| **ARM 集群** | 3-5 台 Raspberry Pi 5 | 低功耗实验集群 | ¥2000-3000 |
| **混合架构** | x86 主节点 + ARM 工作节点 | 异构实验 | 因硬件而异 |

### 3.2 网络拓扑

```
                          ┌─────────────────┐
                          │   家庭路由器      │
                          │ 192.168.1.1/24  │
                          └────────┬────────┘
                                   │
                    ┌──────────────┼──────────────┐
                    │              │              │
              ┌─────┴─────┐ ┌─────┴─────┐ ┌─────┴─────┐
              │  k3s-srv1  │ │  k3s-srv2  │ │  k3s-srv3  │
              │(Server+Agent)│ │(Server+Agent)│ │(Server+Agent)│
              │ 192.168.1.10│ │ 192.168.1.11│ │ 192.168.1.12│
              └─────┬──────┘ └─────┬──────┘ └─────┬──────┘
                    │              │              │
                    └──── etcd HA 集群 (端口 2379-2380) ────┘

                    ┌──────────────┐
                    │  k3s-agent1  │
                    │ (Agent Only) │
                    │ 192.168.1.20 │
                    └──────────────┘
```

### 3.3 端口规划

| 端口 | 协议 | 用途 | 节点类型 |
|------|------|------|---------|
| 6443 | TCP | Kubernetes API Server | Server |
| 2379-2380 | TCP | etcd 客户端/集群通信 | Server (HA) |
| 2381 | TCP | etcd 健康检查 (嵌入式) | Server (HA) |
| 8472 | UDP | Flannel VXLAN | 所有节点 |
| 10250 | TCP | kubelet metrics | 所有节点 |
| 80 | TCP | Ingress HTTP | Server |
| 443 | TCP | Ingress HTTPS | Server |
| 6443 | TCP | k3s agent 注册 | Server |

### 3.4 操作系统准备（所有节点）

```bash
# 1. 更新系统
apt update && apt upgrade -y

# 2. 设置主机名（每台机器不同）
hostnamectl set-hostname k3s-srv1    # Server 节点
hostnamectl set-hostname k3s-agent1  # Agent 节点

# 3. 配置静态 IP（编辑 netplan 或 /etc/network/interfaces）
# 确保 IP 永久固定

# 4. 配置 hosts 文件（所有节点）
cat >> /etc/hosts << 'EOF'
192.168.1.10  k3s-srv1
192.168.1.11  k3s-srv2
192.168.1.12  k3s-srv3
192.168.1.20  k3s-agent1
EOF

# 5. 关闭 Swap（重要！）
swapoff -a
sed -i '/swap/d' /etc/fstab

# 6. 时间同步
apt install -y chrony
systemctl enable --now chrony

# 7. （可选）安装并配置 NTP 确保时间精确同步
timedatectl set-ntp true

# 8. 内核参数优化
cat >> /etc/sysctl.conf << 'EOF'
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
modprobe br_netfilter
sysctl -p /etc/sysctl.conf
```

---

## 4. 部署实战

### 4.1 方案 A：单节点部署（入门/测试）

适合：只有一台机器，想快速体验。

```bash
# 安装 K3s Server（含 Agent）
curl -sfL https://get.k3s.io | sh -

# 检查状态
systemctl status k3s
k3s kubectl get nodes

# 获取 kubeconfig（用于远程管理）
cat /etc/rancher/k3s/k3s.yaml
```

```bash
# 自定义安装（指定数据目录、禁用 Traefik 等）
curl -sfL https://get.k3s.io | sh -s - \
  --data-dir /mnt/data/k3s \
  --disable traefik \
  --write-kubeconfig-mode 644
```

> [!warning] 单节点风险
> 单节点部署无高可用。Server 挂掉 = 整个集群不可用。仅建议用于开发/测试。

### 4.2 方案 B：嵌入式 etcd 高可用集群（推荐）

适合：3+ 台 Server 节点，生产级可用性。

```
┌─────────────────────────────────────────────────┐
│              K3s HA 集群架构                      │
│                                                   │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│   │ Server-1 │  │ Server-2 │  │ Server-3 │     │
│   │(Leader)  │  │(Follower)│  │(Follower)│     │
│   │ API+Sched│  │ API+Sched│  │ API+Sched│     │
│   │ C-M+etcd │  │ C-M+etcd │  │ C-M+etcd │     │
│   └────┬─────┘  └────┬─────┘  └────┬─────┘     │
│        │              │              │            │
│        └──── embedded etcd Raft ────┘            │
│                                                   │
│   ┌──────────┐  ┌──────────┐                    │
│   │ Agent-1  │  │ Agent-2  │  ...               │
│   │(工作节点) │  │(工作节点) │                    │
│   └──────────┘  └──────────┘                    │
└─────────────────────────────────────────────────┘
```

#### 步骤 1：初始化第一个 Server 节点

```bash
# 在 k3s-srv1 (192.168.1.10) 上执行
curl -sfL https://get.k3s.io | sh -s - server \
  --cluster-init \
  --tls-san 192.168.1.10 \
  --tls-san k3s-srv1 \
  --tls-san k3s-api.local \
  --data-dir /mnt/data/k3s
```

> [!important] `--tls-san` 很重要
> 所有后续加入的节点都通过这个地址连接。如果你以后想通过负载均衡 IP 或域名访问 API，**现在就要加进去**，否则以后改证书很麻烦。

安装完成后，获取 Token：

```bash
cat /var/lib/rancher/k3s/server/token
# 输出示例：K10xxxxxxxxxxxx::server:xxxxxxxxxxxxxxxx
```

#### 步骤 2：加入更多 Server 节点

```bash
# 在 k3s-srv2 (192.168.1.11) 和 k3s-srv3 (192.168.1.12) 上分别执行
curl -sfL https://get.k3s.io | sh -s - server \
  --server https://192.168.1.10:6443 \
  --token <从步骤1获取的TOKEN> \
  --tls-san 192.168.1.11 \
  --data-dir /mnt/data/k3s
```

#### 步骤 3：加入 Agent（工作）节点

```bash
# 在 k3s-agent1 (192.168.1.20) 上执行
curl -sfL https://get.k3s.io | K3S_URL=https://192.168.1.10:6443 \
  K3S_TOKEN=<从步骤1获取的TOKEN> \
  sh -
```

#### 步骤 4：验证集群

```bash
# 在任意 Server 节点执行
k3s kubectl get nodes
# 期望输出：
# NAME        STATUS   ROLES                       AGE   VERSION
# k3s-srv1    Ready    control-plane,etcd,master   5m    v1.31.x+k3s1
# k3s-srv2    Ready    control-plane,etcd,master   3m    v1.31.x+k3s1
# k3s-srv3    Ready    control-plane,etcd,master   1m    v1.31.x+k3s1
# k3s-agent1  Ready    <none>                      30s   v1.31.x+k3s1

# 检查 etcd 集群健康
k3s etcdctl member list
k3s etcdctl endpoint health
```

### 4.3 方案 C：外部数据库 HA（MySQL/PostgreSQL）

适合：已有外部数据库的环境（家用较少使用）。

```bash
# 使用外部 MySQL
curl -sfL https://get.k3s.io | sh -s - server \
  --datastore-endpoint="mysql://k3s:password@tcp(192.168.1.50:3306)/k3s"
```

### 4.4 离线/ air-gapped 部署

适合：内网环境无法访问外网。

```bash
# 1. 下载 K3s 二进制（在有网络的机器上）
wget https://github.com/k3s-io/k3s/releases/latest/download/k3s
chmod +x k3s

# 2. 下载离线镜像
wget https://github.com/k3s-io/k3s/releases/latest/download/k3s-airgap-images-amd64.tar

# 3. 传输到目标机器
scp k3s k3s-airgap-images-amd64.tar user@target:~

# 4. 在目标机器上安装
mkdir -p /var/lib/rancher/k3s/agent/images/
cp k3s-airgap-images-amd64.tar /var/lib/rancher/k3s/agent/images/
cp k3s /usr/local/bin/
chmod +x /usr/local/bin/k3s

# 5. 创建 systemd 服务
curl -sfL https://get.k3s.io | INSTALL_K3S_SKIP_DOWNLOAD=true sh -s - server --cluster-init
```

### 4.5 从远程机器管理集群

```bash
# 1. 从 Server 节点复制 kubeconfig
scp root@192.168.1.10:/etc/rancher/k3s/k3s.yaml ~/.kube/config

# 2. 修改 server 地址
sed -i 's/127.0.0.1/192.168.1.10/g' ~/.kube/config

# 3. 安装 kubectl（如果还没有）
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl && mv kubectl /usr/local/bin/

# 4. 测试连接
kubectl get nodes
```

---

## 5. 网络与 Ingress 配置

### 5.1 默认网络：Flannel VXLAN

K3s 默认使用 Flannel + VXLAN，开箱即用：

```bash
# 查看集群 Pod CIDR 和 Service CIDR
k3s kubectl cluster-info dump | grep -i cidr

# 默认值：
# Cluster CIDR: 10.42.0.0/16
# Service CIDR: 10.43.0.0/1
```

### 5.2 替换为 Calico（支持 NetworkPolicy）

```bash
# 安装 K3s 时禁用 Flannel
curl -sfL https://get.k3s.io | sh -s - server \
  --flannel-backend=none \
  --disable-network-policy \
  --cluster-init

# 安装 Calico
k3s kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/calico.yaml
```

### 5.3 MetalLB（裸金属负载均衡器）

K3s 内置的 ServiceLB 功能有限。MetalLB 提供真正的 LoadBalancer：

```bash
# 1. 安装 MetalLB
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.8/config/manifests/metallb.yaml

# 2. 配置 IP 地址池
cat << 'EOF' | kubectl apply -f -
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: home-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.1.100-192.168.1.120   # 家庭网络中的保留 IP 段
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: home-l2
  namespace: metallb-system
spec:
  ipAddressPools:
  - home-pool
EOF
```

### 5.4 Traefik Ingress 配置

K3s 默认安装 Traefik v2，通过 IngressRoute 或标准 Ingress 暴露服务：

```yaml
# 示例：通过 Ingress 暴露 Web 应用
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app
  annotations:
    traefik.ingress.kubernetes.io/router.entrypoints: websecure
    traefik.ingress.kubernetes.io/router.tls: "true"
spec:
  rules:
  - host: app.home.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app
            port:
              number: 80
  tls:
  - hosts:
    - app.home.local
```

```bash
# 自定义 Traefik 配置（如增加入口点）
cat << 'EOF' | kubectl apply -f -
apiVersion: helm.cattle.io/v1
kind: HelmChartConfig
metadata:
  name: traefik
  namespace: kube-system
spec:
  valuesContent: |-
    ports:
      web:
        redirectTo: websecure
      websecure:
        tls:
          enabled: true
    additionalArguments:
      - "--certificatesresolvers.le.acme.email=you@example.com"
      - "--certificatesresolvers.le.acme.storage=/data/acme.json"
      - "--certificatesresolvers.le.acme.tlschallenge=true"
EOF
```

> [!tip] 替换为 Nginx Ingress
> 如果你更熟悉 Nginx：
> ```bash
> # 禁用默认 Traefik，安装 Nginx
> curl -sfL https://get.k3s.io | sh -s - --disable traefik
> kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
> ```

---

## 6. 持久化存储方案

### 6.1 内置 local-path（默认）

K3s 自带 `local-path-provisioner`，自动在节点本地创建存储：

```yaml
# 自动创建 PV
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-data
spec:
  accessModes: ["ReadWriteOnce"]
  storageClassName: local-path
  resources:
    requests:
      storage: 10Gi
```

```bash
# 修改默认存储路径
cat << 'EOF' | kubectl apply -f -
apiVersion: helm.cattle.io/v1
kind: HelmChartConfig
metadata:
  name: local-path-provisioner
  namespace: kube-system
spec:
  valuesContent: |-
    storageClass:
      defaultClass: true
    nodePathMap:
    - node: DEFAULT_PATH_FOR_NON_LISTED_NODES
      paths:
      - /mnt/data/k3s-storage
EOF
```

> [!warning] local-path 的局限
> - 数据绑定在单个节点，Pod 迁移后数据不跟随
> - 无副本、无快照、无扩容
> - 节点故障 = 数据丢失

### 6.2 Longhorn（推荐：分布式块存储）

Longhorn 是专为 K8s 设计的分布式块存储，完美适配家庭集群：

```bash
# 安装 Longhorn
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.7.2/deploy/longhorn.yaml

# 检查安装状态
kubectl -n longhorn-system get pods -w
```

```bash
# 安装 iscsid（Longhorn 依赖，每个节点都要装）
apt install -y open-iscsi
systemctl enable --now iscsid
```

```yaml
# 使用 Longhorn 存储
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-longhorn-data
spec:
  accessModes: ["ReadWriteOnce"]
  storageClassName: longhorn
  resources:
    requests:
      storage: 20Gi
```

**Longhorn 优势：**
- ✅ 数据跨节点副本（默认 2 副本）
- ✅ Web 管理界面
- ✅ 支持快照和备份（可备份到 NFS/S3）
- ✅ 在线扩容
- ✅ 节点故障自动恢复

**家庭集群调优建议：**

```yaml
# 修改 Longhorn 默认设置（适配合庭节点数少的特点）
apiVersion: longhorn.io/v1beta2
kind: Setting
metadata:
  name: default-replica-count
  namespace: longhorn-system
value: "2"          # 2 副本（3 节点集群推荐）
---
apiVersion: longhorn.io/v1beta2
kind: Setting
metadata:
  name: storage-minimal-available-percentage
  namespace: longhorn-system
value: "10"         # 存储告警阈值
```

### 6.3 NFS 存储（共享文件存储）

```bash
# 安装 NFS CSI 驱动
kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/v4.6.0/deploy/install-driver.sh | bash

# 创建 NFS StorageClass
cat << 'EOF' | kubectl apply -f -
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-csi
provisioner: nfs.csi.k8s.io
parameters:
  server: 192.168.1.50      # NFS 服务器 IP
  share: /volume1/k8s-data  # NFS 共享路径
reclaimPolicy: Retain
volumeBindingMode: Immediate
EOF
```

---

## 7. 常用应用部署

### 7.1 K3s 内置 Helm Controller

K3s 支持通过 CRD 直接部署 Helm Chart，无需安装 Helm CLI：

```yaml
# /var/lib/rancher/k3s/server/manifests/ 下放置的 YAML 会自动部署
apiVersion: helm.cattle.io/v1
kind: HelmChart
metadata:
  name: nextcloud
  namespace: kube-system
spec:
  chart: nextcloud
  repo: https://nextcloud.github.io/helm/
  targetNamespace: nextcloud
  valuesContent: |-
    persistence:
      enabled: true
      size: 50Gi
      storageClass: longhorn
    ingress:
      enabled: true
      hostname: cloud.home.local
```

### 7.2 家庭常用应用清单

| 应用 | 用途 | Helm Chart |
|------|------|-----------|
| **Nextcloud** | 私有云盘 | `nextcloud/nextcloud` |
| **Jellyfin** | 媒体服务器 | 自建 YAML 或 `jellyfin/jellyfin` |
| **Home Assistant** | 智能家居 | 自建 YAML |
| **Pi-hole / AdGuard Home** | DNS 广告过滤 | `mojo2600/pihole` |
| **Gitea** | 自托管 Git | `gitea-charts/gitea` |
| **Vaultwarden** | 密码管理器 | `guerzon/vaultwarden` |
| **Uptime Kuma** | 服务监控 | 自建 YAML |
| **Immich** | 照片管理 | `immich-charts/immich` |
| **Frigate** | 视频监控 NVR | `blakeblackshear/frigate` |
| **Traefik Dashboard** | 反向代理管理 | 已内置 |

### 7.3 cert-manager（HTTPS 证书管理）

```bash
# 安装 cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.16.0/cert-manager.yaml

# 使用 Let's Encrypt（需要公网域名）
cat << 'EOF' | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: traefik
EOF
```

> [!note] 家庭内网 HTTPS
> 如果没有公网域名，可以使用自签名 CA 或 [Step CA](https://smallstep.com/docs/step-ca/) 搭建内部 CA。

### 7.4 KubeVirt（在 K3s 上运行虚拟机）

```bash
# 安装 KubeVirt
kubectl apply -f https://github.com/kubevirt/kubevirt/releases/download/v1.3.0/kubevirt-operator.yaml
kubectl apply -f https://github.com/kubevirt/kubevirt/releases/download/v1.3.0/kubevirt-cr.yaml

# 安装 CDI（容器化数据导入）
kubectl apply -f https://github.com/kubevirt/containerized-data-importer/releases/download/v1.60.0/cdi-operator.yaml
kubectl apply -f https://github.com/kubevirt/containerized-data-importer/releases/download/v1.60.0/cdi-cr.yaml
```

---

## 8. 运维与监控

### 8.1 Kubernetes Dashboard

```bash
# 安装 Dashboard
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

# 创建管理员 ServiceAccount
cat << 'EOF' | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF

# 获取登录 Token
kubectl -n kubernetes-dashboard create token admin-user

# 通过 kubectl proxy 或 Ingress 访问
kubectl proxy
# 浏览器访问：http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

### 8.2 Prometheus + Grafana 监控

```bash
# 使用 kube-prometheus-stack 一键安装
kubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/example/rbac/prometheus-operator/prometheus-operator_clusterRole.yaml
kubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/example/rbac/prometheus-operator/prometheus-operator_clusterRoleBinding.yaml
kubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/example/rbac/prometheus-operator/prometheus-operator_serviceAccount.yaml
kubectl apply --server-side -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/example/rbac/prometheus-operator/prometheus-operator_deployment.yaml

# 或使用 Helm（推荐）
cat << 'EOF' | kubectl apply -f -
apiVersion: helm.cattle.io/v1
kind: HelmChart
metadata:
  name: kube-prometheus-stack
  namespace: kube-system
spec:
  repo: https://prometheus-community.github.io/helm-charts
  chart: kube-prometheus-stack
  targetNamespace: monitoring
  valuesContent: |-
    grafana:
      ingress:
        enabled: true
        hosts:
        - grafana.home.local
    prometheus:
      prometheusSpec:
        retention: 7d
        resources:
          requests:
            memory: 512Mi
EOF
```

### 8.3 常用运维命令

```bash
# 集群信息
kubectl cluster-info
kubectl get nodes -o wide
kubectl top nodes

# 查看所有 Pod
kubectl get pods -A

# 查看事件
kubectl get events -A --sort-by='.lastTimestamp'

# 查看节点详情
kubectl describe node k3s-srv1

# etcd 备份（HA 模式）
k3s etcdctl snapshot save /backup/k3s-snapshot-$(date +%Y%m%d).tar.gz

# etcd 恢复
k3s server --cluster-reset --cluster-reset-restore-path=/backup/k3s-snapshot-20260530.tar.gz

# 升级 K3s（滚动升级）
# 在每个 Server 节点上依次执行
curl -sfL https://get.k3s.io | sh -

# 查看集群版本
kubectl version -o yaml
```

### 8.4 自动备份策略

```bash
# 创建自动备份 CronJob
cat << 'EOF' | kubectl apply -f -
apiVersion: batch/v1
kind: CronJob
metadata:
  name: k3s-etcd-backup
  namespace: kube-system
spec:
  schedule: "0 2 * * *"    # 每天凌晨 2 点
  jobTemplate:
    spec:
      template:
        spec:
          nodeSelector:
            node-role.kubernetes.io/control-plane: "true"
          containers:
          - name: backup
            image: rancher/k3s:latest
            command:
            - /bin/sh
            - -c
            - |
              k3s etcdctl snapshot save /backup/snapshot-$(date +%Y%m%d-%H%M%S).tar.gz
              # 保留最近 7 个备份
              ls -t /backup/snapshot-*.tar.gz | tail -n +8 | xargs -r rm
          restartPolicy: OnFailure
          volumes:
          - name: backup
            hostPath:
              path: /mnt/backup/k3s
              type: DirectoryOrCreate
EOF
```

---

## 9. 家庭集群的限制与应对

### 9.1 网络限制

| 限制 | 说明 | 应对方案 |
|------|------|---------|
| **无公网 IP** | 大多数家庭宽带没有固定公网 IP | 使用 DDNS + 端口映射，或 Cloudflare Tunnel |
| **上行带宽低** | 家庭宽带上行通常只有下行的 1/5-1/10 | 集群内部流量走局域网，仅暴露必要服务 |
| **NAT 穿越** | 家庭路由器 NAT 限制外部访问 | Tailscale / WireGuard VPN 组网 |
| **DNS 解析** | 家庭路由器 DNS 功能有限 | 集群内部用 CoreDNS，配合 Pi-hole/AdGuard |
| **mDNS/LLB** | 组播在 Wi-Fi 上常不工作 | MetalLB 使用 L2 模式 + 有线网络 |

### 9.2 硬件限制

| 限制 | 说明 | 应对方案 |
|------|------|---------|
| **内存有限** | 消费级硬件通常 8-32 GB | 合理规划资源请求/限制，使用请求调度的优先级 |
| **存储可靠性** | 消费级 SSD/HDD 无 ECC | Longhorn 多副本 + 定期备份到 NAS |
| **断电风险** | 无 UPS 或 UPS 容量有限 | 数据库定期备份，Pod 设置合理的优雅关闭 |
| **散热/噪音** | 家用环境需要安静 | 选择低功耗硬件（如 N100/J4125），或 Pi 集群 |
| **ARM 兼容性** | 部分 Docker 镜像无 ARM 版本 | 构建多架构镜像，或使用 `--platform linux/amd64`（QEMU 模拟慢） |

### 9.3 K3s 自身的限制（相比完整 K8s）

| 限制 | 影响 | 严重程度 | 应对 |
|------|------|---------|------|
| **ServiceLB 功能简单** | 不支持高级负载均衡策略 | 低 | 替换为 MetalLB |
| **默认无 NetworkPolicy** | Flannel 不支持网络策略 | 中 | 替换为 Calico/Cilium |
| **Traefik 版本可能滞后** | 内置 Traefik 版本更新慢 | 低 | 手动升级或替换 |
| **无内置监控** | 需手动安装 Prometheus 栈 | 低 | 标准 K8s 也需手动安装 |
| **etcd 嵌入式** | 运维工具不如独立 etcd 丰富 | 低 | 使用 `k3s etcdctl` 子命令 |
| **规模上限** | 官方测试到数百节点 | 无 | 家庭环境远达不到上限 |
| **无 Dynamic Admission** | 默认不启用 Admission Webhook | 低 | 可手动配置 |
| **Secret 加密较弱** | 默认 etcd 中 Secret 明文存储 | 中 | 启用 Secret 加密或使用外部 Sealed Secrets |

### 9.4 安全加固建议

```bash
# 1. 启用 Secret 加密
cat << 'EOF' >> /etc/rancher/k3s/config.yaml
secrets-encryption: true
EOF

# 2. 限制 API 访问（仅允许内网）
# 在路由器上不要将 6443 端口映射到公网

# 3. 使用 Sealed Secrets（安全存储 Secret 到 Git）
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.27.0/controller.yaml

# 4. 启用审计日志
cat << 'EOF' >> /etc/rancher/k3s/config.yaml
kube-apiserver-arg:
  - "audit-log-path=/var/log/k3s/audit.log"
  - "audit-log-maxage=30"
  - "audit-log-maxbackup=10"
  - "audit-policy-file=/etc/rancher/k3s/audit-policy.yaml"
EOF
```

### 9.5 家庭集群 vs 生产集群的差异总结

```
┌───────────────────────────────────────────────────────────────┐
│                    家庭集群 vs 生产集群                         │
├───────────────────┬──────────────────┬────────────────────────┤
│     维度          │   家庭集群        │    生产集群              │
├───────────────────┼──────────────────┼────────────────────────┤
│ 节点数            │ 1-5              │ 10-1000+               │
│ 可用性要求        │ 尽量可用          │ 99.9%+ SLA             │
│ 数据备份          │ 手动/简易自动     │ 专业备份策略            │
│ 监控告警          │ 可选              │ 必须                    │
│ 安全加固          │ 基础              │ 全面                    │
│ 网络架构          │ 单层 LAN          │ 多层 VPC/SDN            │
│ 存储方案          │ Longhorn/NFS      │ Ceph/Gluster/云盘       │
│ CI/CD            │ 可选              │ 必须                    │
│ 人员             │ 1人（你自己）      │ SRE 团队                │
│ 预算             │ 0-5000 元         │ 不限                    │
└───────────────────┴──────────────────┴────────────────────────┘
```

---

## 10. 故障排查手册

### 10.1 常见问题速查表

| 症状 | 可能原因 | 排查命令 |
|------|---------|---------|
| Node NotReady | containerd/kubelet 异常 | `journalctl -u k3s -f` |
| Pod CrashLoopBackOff | 镜像拉取失败/配置错误 | `kubectl describe pod <name>` |
| Pod Pending | 资源不足/PVC 未绑定 | `kubectl describe pod <name>` |
| Service 无法访问 | Endpoints 为空 | `kubectl get endpoints <svc>` |
| Ingress 503 | 后端 Pod 不健康 | `kubectl get pods -n <ns>` |
| etcd 集群不健康 | 网络分区/磁盘满 | `k3s etcdctl endpoint health` |
| 证书过期 | K3s 自动续签（1年），检查时钟 | `k3s kubectl get --raw /readyz` |
| 镜像拉取超时 | 网络问题/镜像仓库不可达 | `crictl pull <image>` |
| 磁盘满 | 日志/镜像堆积 | `df -h && docker/crictl image prune` |

### 10.2 日志查看

```bash
# K3s 服务日志
journalctl -u k3s -f                        # Server 节点
journalctl -u k3s-agent -f                   # Agent 节点

# Pod 日志
kubectl logs -f <pod-name> -n <namespace>

# 之前崩溃的容器日志
kubectl logs <pod-name> -n <namespace> --previous

# 事件日志
kubectl get events -A --sort-by='.lastTimestamp' | tail -30
```

### 10.3 重置/卸载

```bash
# 卸载 K3s Server
/usr/local/bin/k3s-uninstall.sh

# 卸载 K3s Agent
/usr/local/bin/k3s-agent-uninstall.sh

# 清理残留数据
rm -rf /etc/rancher/k3s
rm -rf /var/lib/rancher/k3s
rm -rf /run/k3s
```

### 10.4 网络故障排查流程

```
Pod 无法通信？
  │
  ├─ 检查 Pod 是否 Running
  │   └─ kubectl get pods -A
  │
  ├─ 检查 DNS 解析
  │   └─ kubectl exec -it <pod> -- nslookup kubernetes.default
  │
  ├─ 检查跨节点通信（VXLAN）
  │   └─ 在节点间 ping 10.42.x.x（Pod CIDR）
  │   └─ 检查 UDP 8472 端口是否放行
  │
  ├─ 检查 Service
  │   └─ kubectl get endpoints <svc>
  │   └─ kubectl get svc <svc>
  │
  └─ 检查 Ingress
      └─ kubectl describe ingress <name>
      └─ kubectl logs -n kube-system -l app.kubernetes.io/name=traefik
```

---

## 11. 参考资源

### 官方文档

| 资源 | 链接 |
|------|------|
| K3s 官方文档 | [docs.k3s.io](https://docs.k3s.io/) |
| K3s GitHub | [github.com/k3s-io/k3s](https://github.com/k3s-io/k3s) |
| K3s HA 嵌入式 etcd | [docs.k3s.io/datastore/ha-embedded](https://docs.k3s.io/datastore/ha-embedded) |
| K3s Server 配置 | [docs.k3s.io/cli/server](https://docs.k3s.io/cli/server) |
| K3s Agent 配置 | [docs.k3s.io/cli/agent](https://docs.k3s.io/cli/agent) |

### 社区指南

| 资源 | 链接 |
|------|------|
| K3s vs K8s (2026) | [Clever Cloud Blog](https://www.clever.cloud/blog/features/2026/05/28/k3s-vs-k8s-what-are-the-differences-and-which-one-should-you-choose-in-2026/) |
| How to Set Up a K3S Cluster in 2025 | [Merox Blog](https://merox.dev/blog/k3s-cluster-in-2025/) |
| 3-VPS HA K3s Cluster Playbook | [DCHost Blog](https://www.dchost.com/blog/en/i-built-a-3%E2%80%91vps-ha-k3s-cluster-with-traefik-cert%E2%80%91manager-and-longhorn-heres-the-playbook/) |
| Building a Declarative K3s Homelab | [Dreams of Code](https://dreamsofcode.io/blog/building-a-perfectly-declarative-k3s-homelab-cluster) |
| K3s on Proxmox Best Practices | [Geek Bacon](https://geekbacon.com/2085/) |
| SUSE: K3s and K8s Key Differences | [SUSE Blog](https://www.suse.com/c/k3s-and-k8s-key-differences-and-use-cases-explained/) |

### 生态工具

| 工具 | 用途 | 链接 |
|------|------|------|
| Longhorn | 分布式块存储 | [longhorn.io](https://longhorn.io/) |
| MetalLB | 裸金属负载均衡 | [metallb.universe.tf](https://metallb.universe.tf/) |
| Rancher | 多集群管理 UI | [rancher.com](https://www.rancher.com/) |
| Helm | 包管理 | [helm.sh](https://helm.sh/) |
| ArgoCD | GitOps 持续部署 | [argoproj.github.io/cd](https://argoproj.github.io/cd/) |
| Tailscale | 零配置 VPN 组网 | [tailscale.com](https://tailscale.com/) |
| Renovate | 自动更新依赖 | [github.com/renovatebot](https://github.com/renovatebot/renovate) |

---

> [!summary] 快速决策树
>
> ```
> 你有几种机器？
>   │
>   ├─ 1 台 → 单节点 K3s（方案 A），简单够用
>   │
>   ├─ 2 台 → 1 Server + 1 Agent（无 HA），或 2 Server + 外部 DB
>   │
>   ├─ 3+ 台 → 嵌入式 etcd HA（方案 B），推荐！
>   │
>   └─ 树莓派？→ K3s 原生支持 ARM64/ARMv7，安装命令一样
> ```

---

*最后更新：2026-05-30*
