---
title:
source: "https://metaso.cn/search-v2/8708834967134314497"
author:
published:
created: 2026-02-08
description:
tags:
  - "clippings"
---
根据您的报错信息：“Provider API创建实例失败: 虚拟机支持检查失败: LXD不支持虚拟机”，这是一个非常典型的 **LXD 虚拟化支持检查失败** 的错误。LXD 在尝试创建虚拟机（VM）实例时，会首先检测宿主机是否支持 KVM 虚拟化。如果宿主机不具备此能力，LXD 就会报出您看到的错误。

以下是针对该问题的详细排查和解决方案：

### 1\. 核心原因分析

LXD 创建虚拟机实例（ `virtual-machine` ）与创建容器实例（ `container` ）的区别在于：

- **容器** ：依赖于 Linux 的 Namespace 和 Cgroup 功能，不需要硬件虚拟化支持。
- **虚拟机** ：需要宿主机的硬件虚拟化支持（Intel VT-x 或 AMD-V）以及相应的 KVM 模块支持。

报错 “LXD不支持虚拟机” 通常意味着 **宿主机当前被识别为仅支持容器** ，不支持虚拟机。

### 2\. 具体排查步骤

#### 步骤一：检查宿主机硬件和 BIOS 设置

1. **检查 CPU 是否支持硬件虚拟化** ：
	```bash
	grep -E --color '(vmx|svm)' /proc/cpuinfo
	```
	- 如果有输出（ `vmx` 表示 Intel， `svm` 表示 AMD），说明 CPU 支持。
	- 如果没有输出，说明 CPU 硬件本身不支持，或者 BIOS 中关闭了此功能。
2. **检查 BIOS 设置** ：
	- 重新启动服务器，进入 BIOS/UEFI 设置界面。
	- 找到 “Intel VT-x”, “Intel Virtualization Technology”, “AMD-V” 或 “SVM Mode” 选项。
	- 确保其已 **Enabled** （启用）后保存并退出。

#### 步骤二：检查 Linux 内核是否加载了 KVM 模块

即使硬件支持，如果 KVM 模块未加载，也会导致 LXD 认为不支持 VM。

```bash
# 检查模块是否加载
lsmod | grep kvm

# 检查 /dev/kvm 设备节点
ls -l /dev/kvm
```
- 正常情况下，应该能看到类似 `kvm_intel` 或 `kvm_amd` 的模块被加载。
- 如果没有加载，尝试手动加载：
	```bash
	sudo modprobe kvm_intel   # 如果是 Intel CPU
	sudo modprobe kvm_amd     # 如果是 AMD CPU
	```

#### 步骤三：检查 LXD 是否识别了虚拟化能力

您可以直接通过 LXD 命令查看宿主机支持的实例类型。

```bash
lxc info | grep -A2 'instance_types'
```
- 如果输出中仅包含 `container` ，而没有 `virtual-machine` ，说明 LXD 当前检测不到虚拟化能力（这是您报错的直接原因）。
- 正常情况下应该包含两项：
	```
	instance_types:
	- container
	- virtual-machine
	```

#### 步骤四：检查 LXD 版本和安装方式

- **版本检查** ：确保 LXD 版本足够新（LXD 4.x 及以上才正式支持虚拟机）。您可以通过 `snap info lxd` 或 `lxc version` 查看版本。
- **Snap 权限** ：如果是通过 Snap 安装的 LXD，确保 Snap 有访问 KVM 的权限：
	```bash
	sudo snap connect lxd:kvm-support :kvm-support
	```

### 3\. 解决方案

#### 场景一：如果是因为 BIOS 虚拟化未开启

这是最常见的情况。 **解决方法** ：进入 BIOS 开启虚拟化（VT-x/VV-d/SVM），保存退出，重启系统后重新检查。

#### 场景二：如果是因为宿主机不支持硬件虚拟化

- **方案一** ：如果您确实需要运行虚拟机，且当前硬件不支持，您需要更换支持 VT-x/VV-d 的服务器。
- **方案二** ：如果您可以接受，改用 **容器** （Container）来部署服务。LXD 的容器性能与虚拟机相近，但不需要硬件虚拟化支持。

#### 场景三：如果是因为 LXD 未加载 KVM 模块或权限不足

- **加载模块** ：执行 `sudo modprobe kvm_intel` （Intel）或 `sudo modprobe kvm_amd` （AMD）。
- **Snap 权限** ：确保 Snap 有权限访问 KVM，执行：
	```bash
	sudo snap connect lxd:kvm-support :kvm-support
	```

#### 场景四：如果是因为 LXD 版本过低或配置问题

- **升级 LXD** ：如果您使用的是较旧版本的 LXD，建议通过 Snap 或官方包进行升级。
	```bash
	sudo snap refresh lxd
	```
- **重新初始化 LXD** ：有时 LXD 初始化时没有检测到虚拟化能力，您可以尝试重新初始化。
	```bash
	sudo lxd init
	```
	在初始化过程中，LXD 会自动检查虚拟化支持。

### 4\. 小结

请先确认宿主机的 **BIOS 虚拟化设置** 和 **KVM 模块加载** 是否正常，这是最关键的一步。您可以先执行 `lxc info | grep -A2 'instance_types'` 看看当前 LXD 认知的能力。如果您对硬件不熟悉，建议联系负责硬件管理的同事（如有），检查 BIOS 设置是最快解决问题的办法。