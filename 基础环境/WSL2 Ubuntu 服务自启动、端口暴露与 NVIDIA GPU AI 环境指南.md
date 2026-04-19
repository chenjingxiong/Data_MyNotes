---
tags:
  - WSL2
  - Ubuntu
  - NVIDIA
  - CUDA
  - AI
  - systemd
  - networking
date: 2026-04-18
---

# WSL2 Ubuntu 服务自启动、端口暴露与 NVIDIA GPU AI 环境指南

> 本指南覆盖三大主题：WSL2 内服务开机自启动、端口暴露给局域网访问、以及 NVIDIA GPU AI 开发环境搭建。
> 适用环境：Windows 11 22H2+ / WSL2 / Ubuntu 22.04 或 24.04

---

## 目录

- [[#一、服务自启动 — 启用 systemd]]
- [[#二、端口暴露 — 让局域网访问 WSL2 服务]]
- [[#三、NVIDIA GPU AI 环境搭建]]
- [[#四、在 1Panel 部署 vLLM 运行 Qwen3]]
- [[#五、常见问题与排查]]
- [[#六、完整配置速查]]
- [[#七、动态模型切换]]

---

## 一、服务自启动 — 启用 systemd

### 1.1 为什么需要 systemd

WSL2 默认使用简单的 init 进程，不支持 `systemctl`。启用 systemd 后，你可以像在真正的 Linux 服务器上一样：
- 用 `systemctl enable` 设置服务开机自启
- 用 `systemctl start/stop/restart` 管理服务
- Docker、Nginx、SSH、PostgreSQL 等服务都能自动启动

### 1.2 启用 systemd

**步骤 1：编辑 `/etc/wsl.conf`**

```bash
sudo nano /etc/wsl.conf
```

添加以下内容：

```ini
[boot]
systemd=true

# 可选：设置默认用户
[user]
default=你的用户名

[interop]
enabled=true
appendWindowsPath=true
```

**步骤 2：重启 WSL**

在 **Windows PowerShell（管理员）** 中执行：

```powershell
wsl.exe --shutdown
```

然后重新打开 Ubuntu。

**步骤 3：验证 systemd 是否生效**

```bash
# 检查 PID 1 是否为 systemd
ps -p 1 -o comm=

# 应输出: systemd


# 检查 systemctl 是否可用
systemctl list-unit-files --type=service

```

### 1.3 常用服务自启动配置

```bash
# SSH 服务
sudo systemctl enable ssh
sudo systemctl start ssh

# Docker（如果已安装）
sudo systemctl enable docker
sudo systemctl start docker

# Nginx
sudo systemctl enable nginx
sudo systemctl start nginx

# PostgreSQL
sudo systemctl enable postgresql
sudo systemctl start postgresql

# 查看服务状态
systemctl status ssh
systemctl is-enabled docker
```

### 1.4 启动自定义脚本/服务

如果你的服务没有 systemd unit 文件，可以自己创建：

```bash
# 创建自定义服务
sudo nano /etc/systemd/system/my-ai-service.service
```

写入以下内容：

```ini
[Unit]
Description=My AI Service
After=network.target

[Service]
Type=simple
User=你的用户名
WorkingDirectory=/home/你的用户名/my-project
ExecStart=/usr/bin/python3 /home/你的用户名/my-project/app.py
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

然后启用：

```bash
sudo systemctl daemon-reload
sudo systemctl enable my-ai-service
sudo systemctl start my-ai-service
```

### 1.5 方案 B：不使用 systemd（旧方法）

如果不想启用 systemd，可以在 `/etc/wsl.conf` 中使用 `[boot]command`：

```ini
[boot]
command="service ssh start && service docker start && service postgresql start"
```

或使用 shell profile（`~/.bashrc` 或 `~/.profile`）添加启动命令。但这种方式不如 systemd 可靠。

---

## 二、端口暴露 — 让局域网访问 WSL2 服务

### 2.1 网络模式概述

WSL2 有两种主要网络模式：

| 模式 | 特点 | 适用场景 |
|------|------|----------|
| **NAT（默认）** | WSL 有独立虚拟 IP（172.x.x.x），Windows 自动做 localhost 端口转发 | 个人开发 |
| **Mirrored（镜像）** | WSL 与 Windows 共享网络栈，使用相同 IP | 局域网暴露、VPN 兼容 |

### 2.2 方案 A：启用 Mirrored 网络模式（推荐）

> ⚠️ 要求：Windows 11 22H2 及以上版本

**步骤 1：创建/编辑 `.wslconfig`**

在 Windows 用户目录下（`%USERPROFILE%\.wslconfig`，即 `C:\Users\你的用户名\.wslconfig`）创建文件：

```ini
[wsl2]
# 启用镜像网络模式
networkingMode=mirrored

# 自动继承 Windows 代理设置
autoProxy=true

# 启用 DNS 隧道（改善 DNS 解析）
dnsTunneling=true

# 自动代理 Windows 防火墙规则
firewall=true

# 嵌套虚拟化支持（Docker 等）
nestedVirtualization=true
```

**步骤 2：重启 WSL**

```powershell
wsl.exe --shutdown
```

**步骤 3：验证镜像模式**

```bash
# 在 WSL 中检查 IP 地址
ip addr show eth0

# 镜像模式下，IP 应与 Windows 主机 IP 相同
# 在 Windows 中运行 ipconfig 确认
```

```powershell
# Windows PowerShell 中查看主机 IP
ipconfig | findstr "IPv4"
```

**步骤 4：配置 Windows 防火墙（局域网访问必需）**

> [!important] 需要开放两层防火墙！
> Windows 有 **两层** 防火墙会阻止局域网访问 WSL2 服务：
> 1. **Windows Defender 防火墙** — 普通防火墙规则
> 2. **Hyper-V 防火墙** — 专门控制 WSL2 虚拟机流量的独立防火墙层
>
> 只开放其中一层是不够的！如果遇到"localhost 能访问但局域网不能"的问题，详见 [[#Q12：localhost/127.0.0.1 可以访问，但局域网其他设备无法访问 WSL2 服务]]。

**第 4a 步：普通防火墙规则**

```powershell
# PowerShell（管理员）
# 示例：开放常用开发端口范围
New-NetFirewallRule -DisplayName "WSL2 Dev Ports" -Direction Inbound -LocalPort 3000-9999 -Protocol TCP -Action Allow
```

**第 4b 步：Hyper-V 防火墙（⭐ 关键！不开放这层局域网无法访问）**

> [!warning] 需要管理员权限 + Hyper-V Administrators 组
> 此命令需要 **以管理员身份运行 PowerShell**，且当前用户必须在 **Hyper-V Administrators** 组中。
> 如果仍然报"拒绝访问"，请先执行下面的权限配置步骤。

```powershell
# ⚠️ 必须以管理员身份运行 PowerShell（右键 → 以管理员身份运行）

# 如果报"拒绝访问"，先将当前用户加入 Hyper-V Administrators 组
# 方法1：命令行（替换为你的 Windows 用户名）
Add-LocalGroupMember -Group "Hyper-V Administrators" -Member "你的Windows用户名"

# 方法2：通过图形界面
# 运行 lusrmgr.msc → 组 → Hyper-V Administrators → 添加你的用户

# 添加后需要注销并重新登录，然后再执行以下命令：

# 允许 WSL 所有入站连接（开发环境可用，生产环境慎用）
Set-NetFirewallHyperVVMSetting -Name '{40E0AC32-46A5-438A-A0B2-2B479E8F2E90}' -DefaultInboundAction Allow

# 或者只允许特定端口
New-NetFirewallHyperVRule -Name "WSL-8080" -DisplayName "WSL2 Port 8080" -Direction Inbound -Action Allow -LocalPorts 8080 -Protocol TCP -VMCreatorId '{40E0AC32-46A5-438A-A0B2-2B479E8F2E90}'
```

> [!tip] 如果 Hyper-V 防火墙命令始终无法执行
> 确保你的 Windows 用户在 **Hyper-V Administrators** 组中（上面有说明），
> 并且 **注销重新登录** 后再试。如果你使用的是 Windows 11 23H2+，
> 也可以尝试在 `.wslconfig` 中添加 `firewall=true` 让 WSL 自动同步防火墙规则。

### 2.3 方案 B：NAT 模式 + 端口转发（经典方案）

如果你无法使用镜像模式，可以手动设置端口转发：

**步骤 1：获取 WSL IP**

```bash
# 在 WSL 中
hostname -I
# 输出类似: 172.25.123.45
```

**步骤 2：在 Windows 中配置端口转发**

```powershell
# PowerShell（管理员）
# 格式: netsh interface portproxy add v4tov4 listenport=端口 listenaddress=0.0.0.0 connectport=端口 connectaddress=WSL_IP

netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=8080 connectaddress=172.25.123.45
```

**步骤 3：开放防火墙**

```powershell
New-NetFirewallRule -DisplayName "WSL2 Port 8080" -Direction Inbound -LocalPort 8080 -Protocol TCP -Action Allow
```

**步骤 4：查看已有的端口转发规则**

```powershell
netsh interface portproxy show all
```

**步骤 5：删除端口转发规则**

```powershell
netsh interface portproxy delete v4tov4 listenport=8080 listenaddress=0.0.0.0
```

> ⚠️ **NAT 模式的问题**：每次 WSL 重启后 IP 可能变化，需要重新设置端口转发。可以写一个启动脚本自动处理。

### 2.4 自动端口转发脚本（NAT 模式）

创建 Windows 批处理脚本 `wsl-port-forward.bat`：

```bat
@echo off
REM 自动获取 WSL IP 并设置端口转发

for /f "tokens=*" %%i in ('wsl hostname -I') do set WSL_IP=%%i
set WSL_IP=%WSL_IP: =%

echo WSL IP: %WSL_IP%

REM 清除旧的转发规则
netsh interface portproxy reset

REM 添加端口转发（按需修改端口列表）
for %%p in (8080 3000 5432 6379 8888) do (
    netsh interface portproxy add v4tov4 listenport=%%p listenaddress=0.0.0.0 connectport=%%p connectaddress=%WSL_IP%
    echo Forwarded port %%p to %WSL_IP%:%%p
)

echo Done!
```

将此脚本加入 **Windows 任务计划程序**，在用户登录时自动运行。

### 2.5 验证端口可访问性

```bash
# 在 WSL 中启动一个测试服务
python3 -m http.server 8080

# 在局域网其他设备上访问
# http://你的Windows主机IP:8080
```

```powershell
# Windows 中检查端口监听状态
netstat -an | findstr "8080"
```


### 检查网络配置文件类型

```powershell
# PowerShell（管理员）
Get-NetConnectionProfile
```

如果看到 `NetworkCategory : Public`，公共网络配置文件默认阻止所有入站。改为专用：

```powershell
# 将网络改为"专用"（假设你的网卡名是 "以太网"）
Set-NetConnectionProfile -InterfaceAlias "以太网" -NetworkCategory Private

# 或者查看所有接口后指定
Get-NetAdapter | Select-Object Name, Status
```

---

## 三、NVIDIA GPU AI 环境搭建

### 3.1 架构说明

WSL2 GPU 加速的工作原理：

```
┌─────────────────────────────────────────┐
│  Windows 11 Host                         │
│  ┌───────────────────────────────────┐  │
│  │  NVIDIA Windows Driver (最新版)    │  │
│  │  ↓ GPU-PV (半虚拟化)              │  │
│  │  ┌─────────────────────────────┐  │  │
│  │  │  WSL2 Ubuntu                │  │  │
│  │  │  ┌───────────────────────┐  │  │  │
│  │  │  │  CUDA Toolkit (WSL)   │  │  │  │
│  │  │  │  ↓                    │  │  │  │
│  │  │  │  cuDNN                │  │  │  │
│  │  │  │  ↓                    │  │  │  │
│  │  │  │  PyTorch / TensorFlow │  │  │  │
│  │  │  │  ↓                    │  │  │  │
│  │  │  │  vLLM / Ollama 等     │  │  │  │
│  │  │  └───────────────────────┘  │  │  │
│  │  └─────────────────────────────┘  │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

**关键点**：
- **Windows 端**只需安装 NVIDIA 的 Windows 驱动（不需要在 Windows 装 CUDA Toolkit）
- **WSL2 端**安装 CUDA Toolkit（专为 WSL 构建的版本）和 AI 框架
- GPU 通过 **GPU-PV（半虚拟化）** 传递给 WSL2，性能接近原生

### 3.2 前置要求

| 项目 | 要求 |
|------|------|
| GPU | NVIDIA GTX 1060 6GB 及以上（推荐 RTX 3060 12GB+） |
| 显存 | 最低 6GB，推荐 12GB+（运行大模型需要更多） |
| 驱动 | NVIDIA 驱动 ≥ 510.xx（推荐最新版） |
| Windows | Windows 11 22H2+ |
| WSL2 | 最新版本 |

### 3.3 步骤 1：安装 NVIDIA Windows 驱动

在 Windows 端安装最新 NVIDIA 驱动：

```powershell
# 方法1：从 NVIDIA 官网下载
# https://www.nvidia.com/Download/index.aspx
# 选择 "Game Ready Driver" 或 "Studio Driver" 均可

# 方法2：使用 winget 安装
winget install Nvidia.GeForceExperience
```

> ⚠️ **不要**在 WSL 内部安装 NVIDIA Linux 驱动！Windows 驱动已经包含了 WSL GPU 支持。

### 3.4 步骤 2：验证 WSL2 GPU 可用性

```bash
# 在 WSL 中验证
nvidia-smi

# 应看到类似输出：
# +-----------------------------------------------------------------------------------------+
# | NVIDIA-SMI 560.xx              Driver Version: 560.xx          CUDA Version: 12.6         |
# |-----------------------------------------+------------------------+------------------------+
# | GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC   |
# ...
```

如果 `nvidia-smi` 无法运行，说明驱动或 WSL 配置有问题。

**第一步：确认 nvidia-smi 的实际路径**

```bash
which nvidia-smi
# 大概率输出: /usr/lib/wsl/lib/nvidia-smi
```

**第二步：创建软链接（最简单有效）**

```bash
sudo ln -s /usr/lib/wsl/lib/nvidia-smi /usr/bin/nvidia-smi
```

**第三步：重启 1Panel 服务**

```bash
sudo systemctl restart 1panel
```

然后刷新 1Panel 的 GPU 面板，应该就能检测到了。

### 3.5 步骤 3：安装 CUDA Toolkit（WSL 版）

```bash
# 检查 Ubuntu 版本
lsb_release -a

# 方法1：使用 NVIDIA 官方 apt 仓库（推荐）
# 以 Ubuntu 24.04 为例
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt-get update
sudo apt-get install -y cuda-toolkit-12-6

# 方法2：使用 Ubuntu 22.04 的仓库
# wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu-22.04/x86_64/cuda-keyring_1.1-1_all.deb
# sudo dpkg -i cuda-keyring_1.1-1_all.deb
# sudo apt-get update
# sudo apt-get install -y cuda-toolkit-12-6
```

**配置环境变量：**

```bash
# 添加到 ~/.bashrc
cat >> ~/.bashrc << 'EOF'

# CUDA 环境变量
export PATH=/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
EOF

source ~/.bashrc

# 验证
nvcc --version
```

### 3.6 步骤 4：安装 cuDNN（可选）

> [!important] 大多数情况下不需要单独安装 cuDNN
> - **PyTorch**：pip 安装时已自带 cuDNN，无需单独安装
> - **vLLM Docker**：Docker 镜像内已包含 cuDNN
> - **TensorFlow**：pip 版本自带 cuDNN
>
> **只有**在你需要从源码编译某些项目时，才需要手动安装 cuDNN。

#### 方案 A：使用 pip 安装（推荐，最简单）

```bash
# nvidia-cudnn-cu12 包会安装到 Python 环境中
pip install nvidia-cudnn-cu12

# 然后设置环境变量让系统找到它
# 添加到 ~/.bashrc
echo 'export LD_LIBRARY_PATH=$(python -c "import nvidia.cudnn; print(nvidia.cudnn.__path__[0])")/lib:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc
```

#### 方案 B：通过 apt 安装（需要编译 C++ 项目时）

WSL 专用 CUDA 仓库中 cuDNN 包名不是 `cudnn`，需要先搜索可用包名：

```bash
# 先确保 CUDA keyring 已配置（步骤 3 中已做过）
sudo apt-get update

# 搜索可用的 cuDNN 包名
apt-cache search cudnn | grep cuda

# 根据搜索结果安装对应版本，常见包名：
# CUDA 12.x: sudo apt-get install -y cudnn9-cuda-12
# 或: sudo apt-get install -y libcudnn9-cuda-12
# CUDA 11.x: sudo apt-get install -y cudnn-cuda-11
```

> [!warning] 如果 apt-cache 搜不到 cuDNN 包
> WSL 专用仓库（`wsl-ubuntu`）可能不包含 cuDNN 包。此时可以改用普通 Ubuntu 仓库：
> ```bash
> # 临时添加 Ubuntu 24.04 CUDA 仓库
> wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64/cuda-keyring_1.1-1_all.deb
> sudo dpkg -i cuda-keyring_1.1-1_all.deb
> sudo apt-get update
> apt-cache search cudnn | grep cuda
> ```
> 安装完 cuDNN 后建议移除该仓库，避免与 WSL 仓库冲突。

#### 方案 C：使用 .tar.gz 手动安装

从 [NVIDIA cuDNN 下载页](https://developer.nvidia.com/cudnn-downloads) 下载 Linux tarball：

```bash
# 解压到 CUDA 目录
tar -xvf cudnn-linux-x86_64-9.x.x.x_cudaX.Y-archive.tar.xz
sudo cp cudnn-*-archive/include/* /usr/local/cuda/include/
sudo cp cudnn-*-archive/lib/* /usr/local/cuda/lib64/
sudo chmod a+r /usr/local/cuda/include/cudnn* /usr/local/cuda/lib64/libcudnn*
```

#### 验证 cuDNN 安装

```bash
# 方法1：检查头文件
ls /usr/local/cuda/include/cudnn*.h 2>/dev/null

# 方法2：检查库文件
ldconfig -p | grep cudnn

# 方法3：通过 Python 验证（如果用 pip 安装）
python -c "import nvidia.cudnn; print(nvidia.cudnn.__version__)"
```

### 3.7 步骤 5：搭建 AI 开发环境

#### 方案 A：使用 Conda（推荐）

```bash
# 安装 Miniconda
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh -b
~/miniconda3/bin/conda init bash
source ~/.bashrc

# 创建 AI 环境
conda create -n ai python=3.11 -y
conda activate ai

# 安装 PyTorch（CUDA 12.6 版本）
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu126

# 验证 PyTorch GPU
python -c "import torch; print(f'CUDA available: {torch.cuda.is_available()}'); print(f'GPU: {torch.cuda.get_device_name(0)}')"
```

#### 方案 B：使用 uv（更快的新工具）

```bash
# 安装 uv
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.bashrc

# 创建虚拟环境
uv venv ai-env --python 3.11
source ai-env/bin/activate

# 安装 PyTorch
uv pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu126
```

#### 安装常用 AI 工具

```bash
# Hugging Face 工具
pip install transformers accelerate datasets tokenizers

# 模型推理引擎
pip install vllm              # 高性能 LLM 推理

# 或安装 Ollama（本地大模型运行）
curl -fsSL https://ollama.com/install.sh | sh

# Stable Diffusion 相关
pip install diffusers transformers accelerate safetensors

# ONNX Runtime GPU
pip install onnxruntime-gpu
```

### 3.8 步骤 6：运行 AI 服务的端口暴露

AI 服务通常需要暴露到网络，结合前面 [[#二、端口暴露 — 让局域网访问 WSL2 服务]] 的配置：

```bash
# Ollama — 默认端口 11434
OLLAMA_HOST=0.0.0.0 ollama serve

# vLLM — OpenAI 兼容 API
python -m vllm.serve --model meta-llama/Llama-3.1-8B --host 0.0.0.0 --port 8080

# Jupyter Notebook
jupyter notebook --ip=0.0.0.0 --port=8888 --no-browser

# FastAPI 服务
uvicorn app:app --host 0.0.0.0 --port 8000

# Streamlit 应用
streamlit run app.py --server.address 0.0.0.0 --server.port 8501
```

在镜像网络模式下，这些服务启动后局域网设备即可通过 `http://Windows主机IP:端口` 直接访问。

---

## 四、在 1Panel 部署 vLLM 运行 Qwen3

> [!note] 模型说明
> **Qwen3-30B-A3B** 是通义千问 Qwen3 系列中的 MoE（混合专家）模型：
> - 总参数量：**300 亿**（30B）
> - 每次推理仅激活：**30 亿**（3B）
> - FP16 精度显存需求：**约 60-65GB**
> - AWQ 4-bit 量化后：**约 15-16GB**（纯权重），但 vLLM 还需要 KV 缓存空间
> - 最低 vLLM 版本：**≥ 0.8.5**
>
> **16GB 显存**跑 Qwen3-30B-A3B-AWQ 非常紧张（权重几乎占满），建议优先考虑下方替代模型。

### 4.1 显存需求与方案选择

#### 如果你的显存 ≥ 24GB（RTX 3090/4090）

| 方案 | 显存需求 | 说明 |
|------|---------|------|
| Qwen3-30B-A3B-AWQ (4-bit) | ~16-20GB | ✅ 推荐，质量损失极小 |
| Qwen3-30B-A3B FP16 | ~60-65GB | 需要多卡或 A100 |

#### 如果你的显存 = 16GB（RTX 4080 / RTX 4060 Ti 16GB 等）

> [!warning] 16GB 显存模型选择
> Qwen3-30B-A3B-AWQ 权重约 15-16GB，加上 KV 缓存和 CUDA 上下文会超出 16GB。
> 推荐以下替代方案：

| 推荐模型                  | 量化    | 权重大小  | 16GB 可行性 | 说明           |
| --------------------- | ----- | ----- | -------- | ------------ |
| **Qwen3-30B-A3B-AWQ** | 4-bit | ~15GB | ⚠️ 勉强可行  | 需极限优化（见下方配置） |
| **Qwen3-14B-AWQ**     | 4-bit | ~8GB  | ✅ 轻松运行   | 质量优秀，推荐首选    |
| **Qwen3-8B**          | FP16  | ~16GB | ⚠️ 勉强    | 无量化，上下文受限    |
| **Qwen3-8B-AWQ**      | 4-bit | ~5GB  | ✅ 非常轻松   | 预留大量空间给长上下文  |
| **Qwen3-4B**          | FP16  | ~8GB  | ✅ 轻松     | 小模型，速度快      |

> **16GB 显卡推荐**：**Qwen3-14B-AWQ**（最佳性价比）或 **Qwen3-30B-A3B-AWQ**（极限模式，见 [[#4.5 16GB 显存极限运行 Qwen3-30B-A3B]]）

### 4.2 前置条件

#### （1）安装 NVIDIA Container Toolkit

让 Docker 容器能访问宿主机 GPU：

```bash
# 添加 NVIDIA Container Toolkit 仓库
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | \
  sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit

# 配置 Docker runtime
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker

# 验证 Docker GPU 支持
docker run --rm --gpus all ubuntu nvidia-smi
```

#### （2）确认 1Panel 已安装并运行

```bash
systemctl status 1panel
```

### 4.3 下载模型

> 推荐在国内网络使用 ModelScope 下载，速度远快于 HuggingFace。
> 根据你的显存大小选择对应模型（参考 [[#4.1 显存需求与方案选择]]）。

```bash
# 安装 modelscope
pip install modelscope
```

#### 16GB 显卡推荐：Qwen3-14B-AWQ（~8GB，最佳性价比）

```bash
modelscope download --model Qwen/Qwen3-14B-AWQ \
  --local_dir /opt/1panel/models/Qwen3-14B-AWQ
```

#### 24GB+ 显卡推荐：Qwen3-30B-A3B-AWQ（~15GB，最强质量）

```bash
modelscope download --model Qwen/Qwen3-30B-A3B-AWQ \
  --local_dir /opt/1panel/models/Qwen3-30B-A3B-AWQ
```

#### 其他选项

```bash
# Qwen3-8B-AWQ（~5GB，16GB 显卡可跑长上下文）
modelscope download --model Qwen/Qwen3-8B-AWQ \
  --local_dir /opt/1panel/models/Qwen3-8B-AWQ

# FP16 原版（需要 60GB+ 显存）
modelscope download --model Qwen/Qwen3-30B-A3B \
  --local_dir /opt/1panel/models/Qwen3-30B-A3B
```

#### 通过 HuggingFace 下载（需要代理/海外网络）

```bash
pip install huggingface_hub

# 下载 AWQ 量化版
huggingface-cli download Qwen/Qwen3-14B-AWQ \
  --local-dir /opt/1panel/models/Qwen3-14B-AWQ
```

> [!tip] 模型存放路径
> 建议统一放在 `/opt/1panel/models/` 下，方便 1Panel 应用和 Docker 挂载访问。

### 4.4 通过 1Panel 编排（Compose）部署（推荐，免费）

> 1Panel 的 AI 模型管理功能需要专业版，**社区版用户**可以直接使用 **容器 → 编排** 功能部署，完全免费且功能等价。

**步骤 1：在 1Panel 中创建 Compose**

1. 打开 **1Panel** → **容器** → **编排**
2. 点击 **创建编排**
3. 输入名称：`vllm-qwen3`

**步骤 2：编写 docker-compose.yml**

> [!important] 镜像版本必须匹配你的 CUDA 版本
> 先在 WSL 中运行 `nvidia-smi` 查看右上角的 `CUDA Version`，然后对照下表选择镜像标签：
>
> | 你的 CUDA 版本 | 推荐镜像标签 | 说明 |
> |---------------|-------------|------|
> | CUDA 12.9+ | `vllm/vllm-openai:latest` | 最新版 |
> | CUDA 12.6 | `vllm/vllm-openai:v0.8.5` | 稳定版 |
> | CUDA 12.4 | `vllm/vllm-openai:v0.7.3` | 旧稳定版 |
> | CUDA 12.1 | `vllm/vllm-openai:v0.6.6.post1` | 更旧版本 |
>
> **不要用 `latest`，除非你的驱动足够新。** 查看所有可用标签：https://hub.docker.com/r/vllm/vllm-openai/tags

```yaml
services:
  vllm-qwen3:
    image: vllm/vllm-openai:v0.8.5    # ← 根据你的 CUDA 版本修改（见上方表格）
    container_name: vllm-qwen3
    restart: unless-stopped
    ports:
      - "8000:8000"
    volumes:
      - /opt/1panel/models:/models    # 挂载模型目录
    environment:
      - MODEL_NAME=Qwen3-14B
    command: >
      --model /models/Qwen3-14B-AWQ
      --served-model-name Qwen3-14B
      --host 0.0.0.0
      --port 8000
      --max-model-len 8192
      --gpu-memory-utilization 0.90
      --trust-remote-code
      --quantization awq
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
```

> [!warning] FP16 原版需要多卡并行时
> 使用 FP16 原版模型（非 AWQ）时，需要增加 `--tensor-parallel-size` 参数：
> ```yaml
> command: >
>   --model /models/Qwen3-30B-A3B
>   --tensor-parallel-size 2
>   --host 0.0.0.0
>   --port 8000
>   --gpu-memory-utilization 0.90
>   --trust-remote-code
> ```

**步骤 3：启动服务**

在 1Panel 编排页面点击 **启动**，或通过命令行：

```bash
cd /opt/1panel/docker/compose/vllm-qwen3
docker compose up -d
```

### 4.6 验证部署

```bash
# 1. 检查容器是否运行
docker ps | grep vllm

# 2. 查看模型列表
curl http://localhost:8000/v1/models

# 3. 测试对话（OpenAI 兼容 API）
#    注意：model 名称需要与 docker-compose.yml 中的 --served-model-name 一致
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "Qwen3-14B",
    "messages": [
      {"role": "user", "content": "你好，请介绍一下你自己"}
    ],
    "max_tokens": 256,
    "temperature": 0.7
  }'

# 4. 测试流式输出
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "Qwen3-14B",
    "messages": [
      {"role": "user", "content": "用Python写一个快速排序"}
    ],
    "max_tokens": 512,
    "stream": true
  }'
```

成功返回 JSON 响应即表示部署完成。

### 4.7 从 Python 调用

```python
from openai import OpenAI

# 连接本地 vLLM 服务
client = OpenAI(
    base_url="http://localhost:8000/v1",
    api_key="not-needed"  # vLLM 默认不需要 API key
)

# 发送对话
response = client.chat.completions.create(
    model="Qwen3-14B",    # 需与 docker-compose.yml 中的 --served-model-name 一致
    messages=[
        {"role": "system", "content": "你是一个有帮助的AI助手。"},
        {"role": "user", "content": "解释一下什么是MoE架构？"}
    ],
    max_tokens=512,
    temperature=0.7
)

print(response.choices[0].message.content)
```

### 4.8 配合 Open WebUI 使用（可选）

如果想要一个类似 ChatGPT 的 Web 界面，建议将 Open WebUI 和 vLLM 放在**同一个 Compose 编排**中，确保网络互通且数据持久化。

> [!warning] 数据持久化关键
> Open WebUI 的账号、对话历史、设置等数据存储在 `/app/backend/data` 目录。
> **必须**通过 `volumes` 将该目录映射到 WSL2 宿主机，否则容器重建后数据全部丢失。

**将以下内容合并到之前的 `docker-compose.yml` 中（同一个文件）：**

```yaml
services:
  # ===== vLLM 服务（之前已有的配置）=====
  vllm-qwen3:
    image: vllm/vllm-openai:v0.8.5
    container_name: vllm-qwen3
    restart: unless-stopped
    ports:
      - "8000:8000"
    volumes:
      - /opt/1panel/models:/models
    command: >
      --model /models/Qwen3-14B-AWQ
      --served-model-name Qwen3-14B
      --host 0.0.0.0
      --port 8000
      --max-model-len 8192
      --gpu-memory-utilization 0.90
      --trust-remote-code
      --quantization awq
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]

  # ===== Open WebUI（新增）=====
  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    restart: unless-stopped
    ports:
      - "3000:8080"
    volumes:
      - /opt/1panel/data/open-webui:/app/backend/data    # ← 关键：数据持久化
    environment:
      # 连接 vLLM（同一 compose 内可直接用服务名）
      - OPENAI_API_BASE_URL=http://vllm-qwen3:8000/v1
    depends_on:
      - vllm-qwen3
```

> [!important] 数据不丢失的检查清单
> 1. **volume 映射正确**：`/opt/1panel/data/open-webui:/app/backend/data` 必须存在
> 2. **不要用 `docker compose down -v`**：`-v` 会删除 volume，用 `docker compose down` 即可
> 3. **确认数据目录存在**：`ls -la /opt/1panel/data/open-webui/` 应能看到数据库文件
> 4. **备份**（推荐）：定期拷贝该目录到 Windows 侧
>    ```bash
>    cp -r /opt/1panel/data/open-webui /mnt/c/Users/你的用户名/Desktop/open-webui-backup
>    ```

部署完成后访问：`http://你的主机IP:3000`，首次打开需要注册管理员账号。

### 4.9 动态模型切换（按需加载不同模型）

> [!note] 适用场景
> 你有多个模型（如 Qwen3-14B、Qwen3-8B、Qwen3-30B-A3B），但显存只够同时运行一个。
> 在 Open WebUI 中选择不同模型时，自动卸载旧模型、加载新模型。
>
> **原理**：vLLM **不支持运行时热切换模型架构**，但我们可以用一个轻量级 **代理服务（Proxy）** 拦截 Open WebUI 的请求，
> 检测到模型变化后，自动停止旧容器、启动新容器加载目标模型，然后转发请求。

#### 架构说明

```
┌───────────────┐     ┌───────────────────┐     ┌─────────────────────┐
│  Open WebUI   │────▶│  vLLM Switcher    │────▶│  vLLM Container     │
│  (port 3000)  │     │  Proxy (port 9000) │     │  (port 8000)        │
│               │     │                   │     │  ┌───────────────┐  │
│  用户选择模型   │     │  1. 拦截请求        │     │  │ Qwen3-14B    │  │
│  → 发送请求     │     │  2. 检查模型名称    │     │  │ or Qwen3-8B  │  │
│               │     │  3. 模型不同→切换    │     │  │ or 30B-A3B   │  │
│               │◀────│  4. 转发请求/响应    │◀────│  └───────────────┘  │
└───────────────┘     └───────────────────┘     └─────────────────────┘
                           systemd 服务              Docker 容器
                            (宿主机)               (按需启停)
```

**切换流程**：
1. 用户在 Open WebUI 选择新模型（如从 Qwen3-14B 切换到 Qwen3-8B）
2. Open WebUI 发送请求到 Proxy（端口 9000）
3. Proxy 检测到模型变化 → `docker stop` 旧容器 → `docker run` 新容器加载目标模型
4. 等待 vLLM 就绪（`/health` 检查）
5. 转发请求到新容器 → 返回响应给 Open WebUI
6. **切换延迟**：约 30~90 秒（取决于模型大小和磁盘速度）

#### 步骤 1：安装依赖

```bash
pip install fastapi uvicorn httpx pyyaml
```

#### 步骤 2：创建目录和模型配置文件

```bash
sudo mkdir -p /opt/1panel/vllm-switcher
```

创建 `/opt/1panel/vllm-switcher/models.yaml`：

```yaml
# ============================================================
# vLLM 动态模型切换 — 模型配置
# ============================================================
# 每个模型定义了 Docker 启动参数。
# 切换模型时，Proxy 会根据此配置重新创建 vLLM 容器。

# Docker 全局配置
docker:
  image: vllm/vllm-openai:v0.8.5   # ← 根据你的 CUDA 版本修改
  gpu_count: all                     # GPU 数量（all = 所有 GPU）
  model_dir: /opt/1panel/models      # 模型文件存放目录

# 默认启动的模型（Proxy 启动时自动加载）
default_model: Qwen3-14B

# ──── 可用模型列表 ────
models:
  # 推荐首选：16GB 显卡最佳性价比
  - name: Qwen3-14B
    path: /models/Qwen3-14B-AWQ
    served_name: Qwen3-14B
    quantization: awq
    max_model_len: 8192
    gpu_memory_utilization: 0.90
    extra_args:
      - --enable-prefix-caching
      - --enable-chunked-prefill

  # 轻量模型：速度快，适合简单任务
  - name: Qwen3-8B
    path: /models/Qwen3-8B-AWQ
    served_name: Qwen3-8B
    quantization: awq
    max_model_len: 8192
    gpu_memory_utilization: 0.85
    extra_args:
      - --enable-prefix-caching
      - --enable-chunked-prefill

  # 小模型：极快，适合测试
  - name: Qwen3-4B
    path: /models/Qwen3-4B
    served_name: Qwen3-4B
    max_model_len: 8192
    gpu_memory_utilization: 0.85
    extra_args:
      - --enable-prefix-caching

  # 大模型 MoE：最强质量，16GB 需极限优化
  - name: Qwen3-30B-A3B
    path: /models/Qwen3-30B-A3B-AWQ
    served_name: Qwen3-30B-A3B
    quantization: awq
    max_model_len: 4096         # 16GB 显存需限制上下文长度
    gpu_memory_utilization: 0.95
    extra_args:
      - --enforce-eager         # 节省约 1-2GB 显存
```

> [!tip] 添加新模型
> 只需在 `models` 列表中添加新条目，然后重启 Proxy 服务：
> ```bash
> sudo systemctl restart vllm-switcher
> ```
> 新模型会自动出现在 Open WebUI 的模型列表中。

#### 步骤 3：创建代理脚本

创建 `/opt/1panel/vllm-switcher/vllm_switcher.py`：

```python
#!/usr/bin/env python3
"""
vLLM 动态模型切换代理 (Model Switching Proxy)
==============================================

在 Open WebUI 和 vLLM 之间充当智能代理，根据请求中的模型名称
自动切换 vLLM 加载的模型。

工作流程：
1. 拦截 OpenAI 兼容 API 请求
2. 检查请求中的 model 字段
3. 如果模型不同于当前加载的模型 → 停止旧容器，启动新容器
4. 等待 vLLM 就绪后转发请求

依赖：pip install fastapi uvicorn httpx pyyaml
"""

import asyncio
import json
import logging
import os
import time
from typing import Optional

import httpx
import yaml
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse, Response, StreamingResponse
import uvicorn

# ──── 日志配置 ────
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
)
logger = logging.getLogger("vllm-switcher")

# ──── 环境变量 / 配置 ────
CONFIG_PATH = os.environ.get("CONFIG_PATH", "/opt/1panel/vllm-switcher/models.yaml")
PROXY_PORT = int(os.environ.get("PROXY_PORT", "9000"))
VLLM_PORT = int(os.environ.get("VLLM_PORT", "8000"))
VLLM_HOST = os.environ.get("VLLM_HOST", "127.0.0.1")
VLLM_URL = f"http://{VLLM_HOST}:{VLLM_PORT}"
CONTAINER_NAME = "vllm-dynamic"
SWITCH_TIMEOUT = int(os.environ.get("SWITCH_TIMEOUT", "300"))  # 切换超时（秒）

# ──── 全局状态 ────
current_model: Optional[str] = None
switch_lock = asyncio.Lock()

app = FastAPI(title="vLLM Dynamic Model Switcher")


# ──────────────────── 工具函数 ────────────────────


def load_config() -> dict:
    """加载 models.yaml 配置"""
    with open(CONFIG_PATH, "r") as f:
        return yaml.safe_load(f)


def get_model_config(model_name: str) -> Optional[dict]:
    """根据模型名获取配置"""
    for m in load_config().get("models", []):
        if m["name"] == model_name:
            return m
    return None


async def run_cmd(cmd: list) -> str:
    """异步执行 shell 命令"""
    logger.info(f"执行: {' '.join(cmd)}")
    proc = await asyncio.create_subprocess_exec(
        *cmd, stdout=asyncio.subprocess.PIPE, stderr=asyncio.subprocess.PIPE
    )
    stdout, stderr = await proc.communicate()
    if proc.returncode != 0:
        raise RuntimeError(f"命令失败: {stderr.decode().strip()}")
    return stdout.decode().strip()


def build_docker_cmd(model_cfg: dict) -> list:
    """构建 docker run 命令"""
    cfg = load_config()
    dock = cfg.get("docker", {})
    image = dock.get("image", "vllm/vllm-openai:v0.8.5")
    gpu = dock.get("gpu_count", "all")
    mdir = dock.get("model_dir", "/opt/1panel/models")

    cmd = [
        "docker", "run", "-d",
        "--name", CONTAINER_NAME,
        "--gpus", gpu,
        "-v", f"{mdir}:/models",
        "-p", f"{VLLM_PORT}:8000",
        image,
        "--model", model_cfg["path"],
        "--served-model-name", model_cfg.get("served_name", model_cfg["name"]),
        "--host", "0.0.0.0",
        "--port", "8000",
        "--max-model-len", str(model_cfg.get("max_model_len", 8192)),
        "--gpu-memory-utilization", str(model_cfg.get("gpu_memory_utilization", 0.90)),
        "--trust-remote-code",
    ]
    if model_cfg.get("quantization"):
        cmd.extend(["--quantization", model_cfg["quantization"]])
    if model_cfg.get("extra_args"):
        cmd.extend(model_cfg["extra_args"])
    return cmd


async def stop_vllm():
    """停止并移除 vLLM 容器"""
    for c in [
        ["docker", "stop", "-t", "30", CONTAINER_NAME],
        ["docker", "rm", "-f", CONTAINER_NAME],
    ]:
        try:
            await run_cmd(c)
        except RuntimeError:
            pass  # 容器可能已不存在


async def wait_vllm_ready(timeout: int = SWITCH_TIMEOUT) -> bool:
    """等待 vLLM 就绪"""
    t0 = time.time()
    while time.time() - t0 < timeout:
        try:
            async with httpx.AsyncClient() as c:
                r = await c.get(f"{VLLM_URL}/health", timeout=5.0)
                if r.status_code == 200:
                    logger.info("✅ vLLM 就绪")
                    return True
        except (httpx.ConnectError, httpx.TimeoutException):
            pass
        await asyncio.sleep(3)
    raise TimeoutError(f"vLLM 在 {timeout}s 内未就绪")


async def switch_model(target: str):
    """切换到目标模型（加锁防止并发切换）"""
    global current_model

    model_cfg = get_model_config(target)
    if not model_cfg:
        available = [m["name"] for m in load_config().get("models", [])]
        raise ValueError(f"未知模型 '{target}'，可用: {available}")

    async with switch_lock:
        # 双重检查：可能另一个协程已经切换到目标模型
        if current_model == target:
            try:
                async with httpx.AsyncClient() as c:
                    r = await c.get(f"{VLLM_URL}/health", timeout=5.0)
                    if r.status_code == 200:
                        return
            except Exception:
                pass

        logger.info(f"🔄 切换模型: {current_model} → {target}")
        await stop_vllm()
        await run_cmd(build_docker_cmd(model_cfg))
        await wait_vllm_ready()
        current_model = target
        logger.info(f"✅ 模型切换完成: {target}")


def extract_model(body: bytes) -> Optional[str]:
    """从请求体提取模型名"""
    if not body:
        return None
    try:
        return json.loads(body).get("model")
    except (json.JSONDecodeError, UnicodeDecodeError):
        return None


# ──────────────────── API 路由 ────────────────────


@app.get("/v1/models")
async def list_models():
    """返回所有配置的模型列表（Open WebUI 会调用此接口展示模型下拉框）"""
    models = load_config().get("models", [])
    return JSONResponse(content={
        "object": "list",
        "data": [
            {"id": m["name"], "object": "model", "owned_by": "local"}
            for m in models
        ],
    })


@app.get("/health")
async def health():
    """代理自身健康检查 + 当前状态"""
    vllm_ok = False
    try:
        async with httpx.AsyncClient() as c:
            r = await c.get(f"{VLLM_URL}/health", timeout=5.0)
            vllm_ok = r.status_code == 200
    except Exception:
        pass
    return JSONResponse(content={
        "status": "ok",
        "current_model": current_model,
        "vllm_status": "ready" if vllm_ok else "unavailable",
    })


@app.api_route("/{path:path}", methods=["GET", "POST", "PUT", "DELETE", "PATCH"])
async def proxy(path: str, request: Request):
    """通用代理：检测模型 → 切换 → 转发"""
    body = await request.body()

    # 检查是否需要切换模型
    if body and path.startswith("v1/"):
        model_name = extract_model(body)
        if model_name:
            try:
                await switch_model(model_name)
            except Exception as e:
                logger.error(f"❌ 切换失败: {e}")
                return JSONResponse(
                    status_code=503,
                    content={"error": {"message": str(e), "type": "server_error"}},
                )

    # 转发请求到 vLLM
    try:
        is_stream = False
        if body:
            try:
                is_stream = json.loads(body).get("stream", False)
            except Exception:
                pass

        url = f"{VLLM_URL}/{path}"
        headers = {k: v for k, v in request.headers.items() if k != "host"}

        if is_stream:
            async def stream():
                async with httpx.AsyncClient(timeout=httpx.Timeout(300.0)) as c:
                    async with c.stream(request.method, url, content=body, headers=headers) as r:
                        async for chunk in r.aiter_bytes():
                            yield chunk
            return StreamingResponse(stream(), media_type="text/event-stream")

        async with httpx.AsyncClient(timeout=httpx.Timeout(300.0)) as c:
            r = await c.request(request.method, url, content=body, headers=headers)
            return Response(
                content=r.content,
                status_code=r.status_code,
                media_type=r.headers.get("content-type"),
            )
    except httpx.ConnectError:
        return JSONResponse(
            status_code=503,
            content={"error": {"message": "vLLM 不可用，模型可能正在加载", "type": "server_error"}},
        )


@app.on_event("startup")
async def on_startup():
    """启动时加载默认模型"""
    cfg = load_config()
    default = cfg.get("default_model")
    if default:
        logger.info(f"🚀 加载默认模型: {default}")
        try:
            await switch_model(default)
        except Exception as e:
            logger.error(f"默认模型加载失败: {e}")


if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=PROXY_PORT)
```

#### 步骤 4：创建 systemd 服务

创建 `/etc/systemd/system/vllm-switcher.service`：

```ini
[Unit]
Description=vLLM Dynamic Model Switcher Proxy
After=docker.service
Requires=docker.service

[Service]
Type=simple
ExecStart=/usr/bin/python3 /opt/1panel/vllm-switcher/vllm_switcher.py
Restart=on-failure
RestartSec=10
Environment=CONFIG_PATH=/opt/1panel/vllm-switcher/models.yaml
Environment=PROXY_PORT=9000
Environment=VLLM_PORT=8000

[Install]
WantedBy=multi-user.target
```

启用并启动：

```bash
sudo systemctl daemon-reload
sudo systemctl enable vllm-switcher
sudo systemctl start vllm-switcher

# 查看日志
sudo journalctl -u vllm-switcher -f
```

#### 步骤 5：更新 Open WebUI 的 Docker Compose

> [!important] 关键变更
> - **移除**原 docker-compose.yml 中的 `vllm-qwen3` 服务（不再需要，由 Proxy 动态管理）
> - Open WebUI 的 `OPENAI_API_BASE_URL` **改为指向 Proxy 端口 9000**

更新后的 `docker-compose.yml`：

```yaml
services:
  # ===== Open WebUI =====
  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    restart: unless-stopped
    ports:
      - "3000:8080"
    volumes:
      - /opt/1panel/data/open-webui:/app/backend/data    # 数据持久化
    environment:
      # ← 指向 Proxy（不再是 vLLM 直接地址）
      - OPENAI_API_BASE_URL=http://host.docker.internal:9000/v1
    extra_hosts:
      - "host.docker.internal:host-gateway"    # 让容器能访问宿主机
```

重启 Open WebUI：

```bash
cd /opt/1panel/docker/compose/vllm-qwen3
docker compose up -d
```

#### 步骤 6：验证

```bash
# 1. 检查 Proxy 状态
curl http://localhost:9000/health
# → {"status":"ok","current_model":"Qwen3-14B","vllm_status":"ready"}

# 2. 查看可用模型列表
curl http://localhost:9000/v1/models
# → 列出 models.yaml 中配置的所有模型

# 3. 测试模型切换（从 Qwen3-14B 切换到 Qwen3-8B）
curl http://localhost:9000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "Qwen3-8B",
    "messages": [{"role": "user", "content": "你好"}],
    "max_tokens": 64
  }'
# 首次请求会等待 ~30-60 秒（切换模型），之后同模型请求即时响应

# 4. 再次检查当前模型
curl http://localhost:9000/health
# → {"current_model":"Qwen3-8B",...}
```

> [!warning] 首次切换会有延迟
> 当请求的模型与当前加载的不同时，需要 **30~90 秒** 的切换时间：
> - 停止旧容器：~5 秒
> - 启动新容器 + 加载模型：20~80 秒（取决于模型大小）
>
> 切换完成后，后续相同模型的请求会 **即时响应**（无额外延迟）。

#### 管理命令速查

```bash
# 查看 Proxy 状态和当前模型
curl -s http://localhost:9000/health | python3 -m json.tool

# 查看 Proxy 日志
sudo journalctl -u vllm-switcher -f

# 重启 Proxy（修改 models.yaml 后需要）
sudo systemctl restart vllm-switcher

# 查看 vLLM 容器状态
docker ps -f name=vllm-dynamic

# 查看 vLLM 容器日志
docker logs vllm-dynamic --tail 50

# 手动停止 Proxy + vLLM
sudo systemctl stop vllm-switcher
docker stop vllm-dynamic
```

> [!info] 未来优化：vLLM Sleep Mode
> vLLM 在 0.8+ 版本引入了 **Sleep Mode**（睡眠模式），支持在不重启容器的情况下快速切换模型：
>
> | Sleep 级别 | 行为 | 切换速度 |
> |-----------|------|---------|
> | **Level 1** | 权重卸载到 CPU 内存 | 亚秒级唤醒 |
> | **Level 2** | 权重和 KV Cache 全部丢弃 | 需重新加载权重 |
>
> 目前 Sleep Mode 主要用于 **同架构模型** 的权重更新（如 RLHF 训练场景）。
> 对于不同架构的模型切换（如 Qwen3-14B → Qwen3-8B），仍需重启容器。
>
> 参考：[vLLM Sleep Mode 官方文档](https://docs.vllm.ai/en/latest/features/sleep_mode/)
> 社区工具：[LLMSnap](https://www.reddit.com/r/LocalLLaMA/comments/1p4k9is/llmsnap_fast_model_swapping_for_vllm_using_sleep/)

### 4.10 性能优化参数参考

| vLLM 参数                    | 说明                  | 推荐值             |
| -------------------------- | ------------------- | --------------- |
| `--gpu-memory-utilization` | GPU 显存利用率           | `0.85-0.95`     |
| `--max-model-len`          | 最大上下文长度             | `8192`（更长需更多显存） |
| `--max-num-seqs`           | 最大并发序列数             | `16-64`         |
| `--tensor-parallel-size`   | 多卡并行数               | 单卡 `1`，双卡 `2`   |
| `--quantization`           | 量化方式                | `awq`（AWQ 模型时）  |
| `--enforce-eager`          | 禁用 CUDA Graph（节省显存） | 调试时加上           |
| `--enable-prefix-caching`  | 前缀缓存（加速重复 prompt）   | 建议开启            |
| `--enable-chunked-prefill` | 分块预填充（降低延迟）         | 建议开启            |

完整启动示例（AWQ + 性能优化，以 Qwen3-14B-AWQ 为例）：

```yaml
command: >
  --model /models/Qwen3-14B-AWQ
  --served-model-name Qwen3-14B
  --host 0.0.0.0
  --port 8000
  --quantization awq
  --max-model-len 8192
  --gpu-memory-utilization 0.90
  --max-num-seqs 32
  --enable-prefix-caching
  --enable-chunked-prefill
  --trust-remote-code
```

---

## 五、常见问题与排查

> 也包含之前在「三、NVIDIA GPU AI 环境搭建」中积累的排障经验。

### Q1：`nvidia-smi` 在 WSL 中不可用

```bash
# 检查是否能看到 GPU 设备
ls /dev/dxg

# 如果不存在，说明 GPU-PV 未启用
# 解决方法：
# 1. 确保 Windows NVIDIA 驱动是最新版
# 2. 确保 WSL2 是最新版
wsl --update

# 3. 重启 WSL
wsl --shutdown
```

### Q2：systemd 服务没有自动启动

```bash
# 检查 systemd 是否启用
cat /etc/wsl.conf | grep systemd

# 检查服务状态
systemctl status <service-name>

# 查看启动日志
journalctl -u <service-name> -n 50
```

### Q3：镜像模式下 WSL 无网络

```bash
# 检查网络模式
cat /mnt/c/Users/你的Windows用户名/.wslconfig

# 尝试重置网络
# PowerShell（管理员）
wsl --shutdown
# 等待几秒后重新打开 WSL

# 如果仍不行，检查 Hyper-V 服务是否运行
# PowerShell（管理员）
Get-Service vmcompute, vmms | Select-Object Name, Status
```

### Q4：PyTorch 检测不到 GPU

```python
import torch

# 详细排查
print(f"PyTorch version: {torch.__version__}")
print(f"CUDA available: {torch.cuda.is_available()}")
print(f"CUDA version (PyTorch): {torch.version.cuda}")
print(f"cuDNN version: {torch.backends.cudnn.version()}")

if torch.cuda.is_available():
    print(f"GPU: {torch.cuda.get_device_name(0)}")
    print(f"GPU Memory: {torch.cuda.get_device_properties(0).total_mem / 1024**3:.1f} GB")
else:
    # 常见原因：
    # 1. PyTorch CUDA 版本与驱动不匹配
    # 2. 安装了 CPU 版本的 PyTorch
    # 3. NVIDIA 驱动过旧
    print("GPU not available - check driver and PyTorch CUDA version")
```

### Q5：显存不足

```bash
# 检查当前显存使用
nvidia-smi

# PyTorch 中限制显存使用
# export PYTORCH_CUDA_ALLOC_CONF=max_split_size_mb:128

# 使用 4-bit 量化加载模型（节省显存）
from transformers import AutoModelForCausalLM, BitsAndBytesConfig

quantization_config = BitsAndBytesConfig(load_in_4bit=True)
model = AutoModelForCausalLM.from_pretrained(
    "model_name",
    quantization_config=quantization_config,
    device_map="auto"
)
```

### Q6：WSL2 占用过多内存

在 `%USERPROFILE%\.wslconfig` 中限制资源：

```ini
[wsl2]
memory=16GB          # 限制最大内存
processors=8         # 限制 CPU 核心数
swap=8GB             # 交换空间大小
localhostForwarding=true
```

### Q7：vLLM 容器启动报错 "CUDA out of memory"

```bash
# 降低显存利用率
--gpu-memory-utilization 0.80

# 缩短最大上下文
--max-model-len 4096

# 禁用 CUDA Graph（额外节省约 1-2GB）
--enforce-eager

# 如果仍然不够，考虑换用更小的量化版或使用 GPTQ-Int4 模型
```

### Q8：vLLM 报错 "MoE kernels not available"

Qwen3-30B-A3B 是 MoE 架构，需要 vLLM ≥ 0.8.5 并支持 MoE 内核：

```bash
# 检查 vLLM 版本（pip 安装时）
pip show vllm

# Docker 中拉取匹配你 CUDA 版本的镜像
# 先用 nvidia-smi 查看你的 CUDA 版本，然后选择对应标签
docker pull vllm/vllm-openai:v0.8.5    # CUDA 12.6
```

### Q9：Docker 容器内无法访问 GPU

```bash
# 1. 检查 NVIDIA Container Toolkit 是否安装
nvidia-ctk --version

# 2. 检查 Docker runtime 配置
cat /etc/docker/daemon.json | grep nvidia

# 应该有类似内容：
# "runtimes": { "nvidia": { "path": "nvidia-container-runtime" } }

# 3. 重新配置并重启
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker

# 4. 测试
docker run --rm --gpus all ubuntu nvidia-smi
```

### Q10：1Panel 面板中 vLLM 服务显示异常

```bash
# 查看 vLLM 容器日志
docker logs $(docker ps -a | grep vllm | awk '{print $1}') --tail 100

# 常见原因：
# 1. 模型路径挂载错误 → 检查 volumes 配置
# 2. 端口被占用 → sudo lsof -i :8000
# 3. GPU 未正确映射 → 检查 deploy.resources.reservations.devices 配置
# 4. 模型文件不完整 → 重新下载模型

# 重启容器
docker restart <容器名>
```

### Q11：容器启动报错 "unsatisfied condition: cuda>=12.9"

```
nvidia-container-cli: requirement error: unsatisfied condition: cuda>=12.9,
please update your driver to a newer version, or use an earlier cuda container
```

**原因**：Docker 镜像中的 CUDA 版本高于你的显卡驱动支持的版本。

```bash
# 1. 查看你的驱动支持的 CUDA 版本
nvidia-smi
# 右上角: CUDA Version: 12.x

# 2. 选择匹配的镜像版本（不要用 latest）
# CUDA 12.6 → vllm/vllm-openai:v0.8.5
# CUDA 12.4 → vllm/vllm-openai:v0.7.3

# 3. 更新 docker-compose.yml 中的 image 字段后重新创建
docker compose down
docker compose up -d
```

> [!tip] 驱动版本与 CUDA 版本对应关系
> | NVIDIA 驱动版本 | 最高支持 CUDA |
> |----------------|-------------|
> | ≥ 560.x | CUDA 12.6 |
> | ≥ 550.x | CUDA 12.4 |
> | ≥ 535.x | CUDA 12.2 |
> | ≥ 525.x | CUDA 12.0 |
> | ≥ 520.x | CUDA 11.8 |
>
> 如果驱动太旧，也可以更新 Windows 端的 NVIDIA 驱动到最新版。

### Q12：localhost/127.0.0.1 可以访问，但局域网其他设备无法访问 WSL2 服务

> [!bug] 典型表现
> - WSL2 内 `curl localhost:端口` 正常
> - Windows 上 `curl localhost:端口` 也正常
> - **但局域网其他电脑/手机** `curl Windows主机IP:端口` 连接超时/拒绝
>
> 这几乎是 WSL2 部署服务后最常见的问题，原因通常是以下几层防火墙叠加导致的。

#### 排查步骤（按优先级从高到低）

**第 1 步：确认实际网络模式**

```bash
# 在 WSL 中
ip addr show eth0 | grep inet
```

- 如果 IP 是 **172.x.x.x** → 你在 **NAT 模式**（不是镜像模式！），见下方方案 A
- 如果 IP 与 Windows 的 `ipconfig` 结果 **相同** → 你在 **镜像模式**，见下方方案 B

**第 2 步：确认服务绑定地址**

```bash
# 在 WSL 中检查服务监听地址
ss -tlnp | grep -E '(8000|9000|3000)'   # 替换为你的端口
```

- ✅ `0.0.0.0:8000` → 监听所有网卡，正确
- ❌ `127.0.0.1:8000` → 只监听本地回环，**外部无法访问**
  - Docker 容器：确认 `-p` 参数写的是 `"8000:8000"`（不是 `"127.0.0.1:8000:8000"`）
  - Python/Node 服务：确认启动参数是 `--host 0.0.0.0` 而不是默认的 `127.0.0.1`

**第 3 步：检查 Windows 防火墙（普通规则）**

```powershell
# PowerShell（管理员）
# 查看是否已有 WSL 相关的入站规则
Get-NetFirewallRule | Where-Object { $_.DisplayName -like "*WSL*" -or $_.DisplayName -like "*Dev*" } | Select-Object DisplayName, Enabled, Direction, Action

# 如果没有，添加规则（一次性开放所有常用端口）
New-NetFirewallRule -DisplayName "WSL2 Services" -Direction Inbound -LocalPort 3000,8000,8080,9000 -Protocol TCP -Action Allow

# 或者开放一个范围
New-NetFirewallRule -DisplayName "WSL2 Dev Ports" -Direction Inbound -LocalPort 3000-9999 -Protocol TCP -Action Allow
```

**第 4 步：⭐ 检查 Hyper-V 防火墙（最常被遗漏的一层！）**

> [!important] 这是 WSL2 网络的最关键一层
> Windows 有 **两层防火墙**：
> 1. **Windows Defender 防火墙**（上面的第 3 步）— 控制普通程序的网络访问
> 2. **Hyper-V 防火墙** — **专门**控制进出 WSL2 虚拟机的流量
>
> 即使你开放了 Windows 防火墙规则，Hyper-V 防火墙默认仍然会 **阻止外部到 WSL2 的入站连接**。
> 这就是"localhost 能访问但局域网不能"的 **最常见原因**。

```powershell
# ⚠️ 以管理员身份运行 PowerShell

# 1. 先查看当前 Hyper-V 防火墙策略
Get-NetFirewallHyperVVMSetting -Name '{40E0AC32-46A5-438A-A0B2-2B479E8F2E90}'

# 如果显示 DefaultInboundAction 为 Block，这就是问题所在！
```

```powershell
# 2. 开放 Hyper-V 防火墙（方案一：全部放行，推荐开发环境使用）
Set-NetFirewallHyperVVMSetting -Name '{40E0AC32-46A5-438A-A0B2-2B479E8F2E90}' -DefaultInboundAction Allow

# 3. 开放 Hyper-V 防火墙（方案二：只放行特定端口，更安全）
New-NetFirewallHyperVRule -Name "WSL-HTTP-3000" -DisplayName "WSL2 Open WebUI" `
  -Direction Inbound -Action Allow -LocalPorts 3000 -Protocol TCP `
  -VMCreatorId '{40E0AC32-46A5-438A-A0B2-2B479E8F2E90}'

New-NetFirewallHyperVRule -Name "WSL-VLLM-8000" -DisplayName "WSL2 vLLM" `
  -Direction Inbound -Action Allow -LocalPorts 8000 -Protocol TCP `
  -VMCreatorId '{40E0AC32-46A5-438A-A0B2-2B479E8F2E90}'

New-NetFirewallHyperVRule -Name "WSL-Proxy-9000" -DisplayName "WSL2 vLLM Proxy" `
  -Direction Inbound -Action Allow -LocalPorts 9000 -Protocol TCP `
  -VMCreatorId '{40E0AC32-46A5-438A-A0B2-2B479E8F2E90}'

New-NetFirewallHyperVRule -Name "WSL-1Panel" -DisplayName "WSL2 1Panel" `
  -Direction Inbound -Action Allow -LocalPorts 8090 -Protocol TCP `
  -VMCreatorId '{40E0AC32-46A5-438A-A0B2-2B479E8F2E90}'
```

> [!tip] 如果 Hyper-V 防火墙命令报"拒绝访问"
> 需要先将当前用户加入 **Hyper-V Administrators** 组：
> ```powershell
> Add-LocalGroupMember -Group "Hyper-V Administrators" -Member "你的Windows用户名"
> ```
> 然后 **注销并重新登录 Windows**，再执行上面的命令。

**第 5 步：检查网络配置文件类型**

```powershell
# PowerShell（管理员）
Get-NetConnectionProfile
```

如果 `NetworkCategory` 显示为 `Public`，改为 `Private`：

```powershell
# 查看网卡名称
Get-NetAdapter | Select-Object Name, Status

# 改为专用网络（替换为你的网卡名）
Set-NetConnectionProfile -InterfaceAlias "以太网" -NetworkCategory Private
# 或 WiFi：
Set-NetConnectionProfile -InterfaceAlias "WLAN" -NetworkCategory Private
```

#### 方案 A：你在 NAT 模式（IP 是 172.x.x.x）

NAT 模式下，WSL2 有独立的虚拟 IP，Windows 默认只做 localhost 端口转发，**不转发局域网流量**。
需要手动添加端口转发：

```powershell
# PowerShell（管理员）
# 获取 WSL IP
$wslIp = (wsl hostname -I).Trim()

# 添加端口转发（每个需要暴露的端口都要添加）
netsh interface portproxy add v4tov4 listenport=3000 listenaddress=0.0.0.0 connectport=3000 connectaddress=$wslIp
netsh interface portproxy add v4tov4 listenport=8000 listenaddress=0.0.0.0 connectport=8000 connectaddress=$wslIp
netsh interface portproxy add v4tov4 listenport=9000 listenaddress=0.0.0.0 connectport=9000 connectaddress=$wslIp
netsh interface portproxy add v4tov4 listenport=8090 listenaddress=0.0.0.0 connectport=8090 connectaddress=$wslIp

# 验证
netsh interface portproxy show all
```

> ⚠️ NAT 模式每次 WSL 重启后 IP 可能变化，需要重新执行。建议 **切换到镜像模式**（见下方方案 B）。

#### 方案 B：确认启用镜像模式（推荐）

编辑 `C:\Users\你的用户名\.wslconfig`，确认包含以下内容：

```ini
[wsl2]
networkingMode=mirrored
firewall=true
localhostForwarding=true
```

保存后在 PowerShell（管理员）中重启 WSL：

```powershell
wsl.exe --shutdown
```

然后重新打开 WSL，验证：

```bash
# IP 应与 Windows 主机相同
ip addr show eth0 | grep inet
```

#### 完整验证清单

```bash
# ═══ 在 WSL 中 ═══
# 确认服务在监听 0.0.0.0
ss -tlnp | grep -E '(3000|8000|8090|9000)'

# ═══ 在 Windows PowerShell 中 ═══
# 确认 Windows 在监听这些端口
netstat -an | findstr "LISTENING" | findstr "3000 8000 8090 9000"

# ═══ 在局域网其他设备上 ═══
# 用 Windows 主机 IP 访问
curl http://Windows主机IP:3000    # Open WebUI
curl http://Windows主机IP:8090    # 1Panel
curl http://Windows主机IP:9000/v1/models  # vLLM Proxy
```

> [!quote] 三层防火墙总结
> ```
> 局域网设备 →①→ Windows Defender 防火墙 →②→ Hyper-V 防火墙 →③→ WSL2 服务
>                  (第 3 步)                    (第 4 步 ⭐)
> ```
> - 第 ① 层：`New-NetFirewallRule`（普通防火墙规则）
> - 第 ② 层：`Set-NetFirewallHyperVVMSetting`（**最常被遗漏！**）
> - 第 ③ 层：服务自身绑定地址（`0.0.0.0` vs `127.0.0.1`）
>
> **localhost 能访问但局域网不能 = 第 ② 层没开放**，这是 90% 情况的根本原因。

### Q13：模型切换 Proxy 相关问题

```bash
# 查看切换代理状态
curl -s http://localhost:9000/health | python3 -m json.tool

# 查看代理日志（实时）
sudo journalctl -u vllm-switcher -f

# 重启代理
sudo systemctl restart vllm-switcher

# 查看当前 vLLM 动态容器
docker ps -f name=vllm-dynamic

# 如果模型切换卡住，手动清理
docker stop vllm-dynamic && docker rm vllm-dynamic
sudo systemctl restart vllm-switcher
```

> [!tip] Open WebUI 中看不到模型列表？
> 1. 确认 Proxy 服务已启动：`systemctl status vllm-switcher`
> 2. 确认 `OPENAI_API_BASE_URL` 指向 Proxy（端口 9000），不是 vLLM 直接地址（端口 8000）
> 3. 在 Open WebUI → 设置 → 连接 中刷新模型列表

---

## 六、完整配置速查

### `/etc/wsl.conf`（WSL 内部）

```ini
[boot]
systemd=true

[user]
default=你的用户名

[interop]
enabled=true
appendWindowsPath=true
```

### `%USERPROFILE%\.wslconfig`（Windows 端）

```ini
[wsl2]
networkingMode=mirrored
autoProxy=true
dnsTunneling=true
firewall=true
nestedVirtualization=true
memory=16GB
processors=8
swap=8GB
localhostForwarding=true
```

### 验证一切正常的命令清单

```bash
# 1. systemd
ps -p 1 -o comm=                    # 应输出 systemd

# 2. 网络
ip addr show eth0                    # 镜像模式下与 Windows IP 相同
curl -s ifconfig.me                  # 检查外部 IP

# 3. GPU
nvidia-smi                           # 显示 GPU 信息
python -c "import torch; print(torch.cuda.is_available())"  # True

# 4. 服务
systemctl is-enabled docker          # enabled
systemctl is-enabled ssh             # enabled
```

---

## 七、动态模型切换

> 完整的动态模型切换方案（包括代理脚本、配置文件、systemd 服务）详见 [[#4.9 动态模型切换（按需加载不同模型）]]。

### WSL2 基础

- [Use systemd to manage Linux services with WSL — Microsoft Learn](https://learn.microsoft.com/en-us/windows/wsl/systemd)
- [Accessing network applications with WSL — Microsoft Learn](https://learn.microsoft.com/en-us/windows/wsl/networking)
- [Systemd support lands in WSL — Ubuntu Blog](https://ubuntu.com/blog/ubuntu-wsl-enable-systemd)

### GPU / CUDA

- [CUDA on WSL User Guide — NVIDIA Docs](https://docs.nvidia.com/cuda/wsl-user-guide/index.html)
- [Enable NVIDIA CUDA on WSL 2 — Microsoft Learn](https://learn.microsoft.com/en-us/windows/ai/directml/gpu-cuda-in-wsl)
- [Getting Started with CUDA on Ubuntu on WSL 2 — Ubuntu Blog](https://ubuntu.com/blog/getting-started-with-cuda-on-ubuntu-on-wsl-2)
- [How to Set Up CUDA and WSL2 for Windows 11 — freeCodeCamp](https://www.freecodecamp.org/news/how-to-set-up-cuda-and-wsl2-for-windows-11-including-pytorch-and-tensorflow-gpu/)
- [Install PyTorch on Windows, Linux and macOS in 2025 — Hugging Face](https://huggingface.co/blog/daya-shankar/pytorch-install-guide)

### vLLM 与 Qwen3 部署

- [1Panel vLLM 官方文档](https://1panel.cn/docs/v2/user_manual/ai/vllm/)
- [1Panel GPU 监控文档](https://1panel.cn/docs/v1/user_manual/ai/gpu/)
- [Qwen3-30B-A3B — ModelScope](https://modelscope.cn/models/Qwen/Qwen3-30B-A3B)
- [vLLM Docker 部署 — vLLM 中文站](https://vllm.hyper.ai/docs/deployment/docker/)
- [vLLM 官方 Docker 部署文档](https://docs.vllm.com.cn/en/latest/deployment/docker/)
- [Docker Compose 编排 vLLM 多模型服务 — 知乎](https://zhuanlan.zhihu.com/p/1985432653625844472)
- [vLLM 部署 Qwen3 方案 — 俊瑶先森](http://junyao.tech/posts/2a08b1ba.html)

### vLLM 动态模型切换

- [vLLM Sleep Mode 官方文档](https://docs.vllm.ai/en/latest/features/sleep_mode/)
- [LLMSnap — 基于 Sleep Mode 的快速模型切换工具 (Reddit)](https://www.reddit.com/r/LocalLLaMA/comments/1p4k9is/llmsnap_fast_model_swapping_for_vllm_using_sleep/)
- [vLLM Sleep Mode 零重载模型切换 — 知乎](https://zhuanlan.zhihu.com/p/1967692417659537009)
- [Model Swapping with vLLM — Reddit 讨论](https://www.reddit.com/r/LocalLLaMA/comments/1kg6tk3/model_swapping_with_vllm/)
