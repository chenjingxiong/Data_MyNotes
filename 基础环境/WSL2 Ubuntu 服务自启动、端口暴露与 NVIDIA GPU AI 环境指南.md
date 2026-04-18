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
- [[#四、常见问题与排查]]

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

即使使用镜像模式，Windows 防火墙可能仍会阻止外部访问。需要添加入站规则：

```powershell
# PowerShell（管理员）
# 示例：开放 8080 端口
New-NetFirewallRule -DisplayName "WSL2 Port 8080" -Direction Inbound -LocalPort 8080 -Protocol TCP -Action Allow

# 示例：开放常用开发端口范围
New-NetFirewallRule -DisplayName "WSL2 Dev Ports" -Direction Inbound -LocalPort 3000-9999 -Protocol TCP -Action Allow
```

或者通过 **Hyper-V 防火墙** 配置（更底层的方式）：

```powershell
# 允许 WSL 所有入站连接（开发环境可用，生产环境慎用）
Set-NetFirewallHyperVVMSetting -Name '{40E0AC32-46A5-438A-A0B2-2B479E8F2E90}' -DefaultInboundAction Allow

# 或者只允许特定端口
Set-NetFirewallHyperVRule -Name "WSL" -Direction Inbound -Action Allow -LocalPorts 8080 -Protocol TCP
```

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

### 3.6 步骤 4：安装 cuDNN

```bash
# 通过 apt 安装 cuDNN（需要 CUDA keyring 已配置）
sudo apt-get install -y cudnn

# 或者安装特定版本
# sudo apt-get install -y cudnn=9.6.0.*-1+cuda12.6

# 验证
cat /usr/include/cudnn_version.h 2>/dev/null || dpkg -l | grep cudnn
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

## 四、常见问题与排查

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

---

## 五、完整配置速查

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

## 参考资料

- [Use systemd to manage Linux services with WSL — Microsoft Learn](https://learn.microsoft.com/en-us/windows/wsl/systemd)
- [Accessing network applications with WSL — Microsoft Learn](https://learn.microsoft.com/en-us/windows/wsl/networking)
- [CUDA on WSL User Guide — NVIDIA Docs](https://docs.nvidia.com/cuda/wsl-user-guide/index.html)
- [Enable NVIDIA CUDA on WSL 2 — Microsoft Learn](https://learn.microsoft.com/en-us/windows/ai/directml/gpu-cuda-in-wsl)
- [Systemd support lands in WSL — Ubuntu Blog](https://ubuntu.com/blog/ubuntu-wsl-enable-systemd)
- [Getting Started with CUDA on Ubuntu on WSL 2 — Ubuntu Blog](https://ubuntu.com/blog/getting-started-with-cuda-on-ubuntu-on-wsl-2)
- [How to Set Up CUDA and WSL2 for Windows 11 — freeCodeCamp](https://www.freecodecamp.org/news/how-to-set-up-cuda-and-wsl2-for-windows-11-including-pytorch-and-tensorflow-gpu/)
- [Install PyTorch on Windows, Linux and macOS in 2025 — Hugging Face](https://huggingface.co/blog/daya-shankar/pytorch-install-guide)
