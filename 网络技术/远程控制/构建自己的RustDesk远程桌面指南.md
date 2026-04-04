---
title: 构建自己的 RustDesk 远程桌面指南
date: 2026-04-04
tags:
  - RustDesk
  - 远程桌面
  - 自建服务器
  - Docker
  - API
source: https://www.smianao.com/1414.html
---

# 构建自己的 RustDesk 远程桌面指南

> [!info] 前言
> RustDesk 是一款开源的远程桌面软件，支持自建中继服务器。本指南基于 [鼠标迁徙](https://www.smianao.com/1414.html) 的教程，涵盖 **服务端部署、API 防滥用、源码修改、自动编译客户端** 的完整流程。

---

## 一、整体架构概览

```
┌─────────────┐       ┌──────────────────────────────────┐
│  RustDesk    │◄─────►│  你的云服务器                      │
│  自编译客户端 │       │                                  │
│  (内置服务器  │       │  ┌─────────────────────────┐     │
│   信息)      │       │  │ Docker Compose           │     │
└─────────────┘       │  │  ├─ hbbs (ID服务器) :21116│     │
                      │  │  ├─ hbbr (中继服务器) :21117│    │
                      │  │  └─ API (Web管理) :21114  │     │
                      │  └─────────────────────────┘     │
                      └──────────────────────────────────┘
```

**核心组件：**
- **hbbs**：ID 服务器，负责设备注册和发现（端口 21116）
- **hbbr**：中继服务器，负责远程桌面数据转发（端口 21117）
- **API**：Web 管理后台，控制用户注册/登录，防止滥用（端口 21114）
- **自编译客户端**：将服务器信息内置到客户端，用户无需手动配置

---

## 二、服务端及 API 安装

### 2.1 准备工作

- 一台具有**公网 IP** 的云服务器（推荐阿里云、腾讯云等）
- SSH 远程连接工具（如 [WindTerm](https://sbqx.cc/233.html)）

### 2.2 安装 1Panel 管理面板

1Panel 提供可视化的 Docker Compose 管理，界面简洁易用。

```bash
# RedHat / CentOS
curl -sSL https://resource.fit2cloud.com/1panel/package/quick_start.sh -o quick_start.sh && sh quick_start.sh

# Ubuntu
curl -sSL https://resource.fit2cloud.com/1panel/package/quick_start.sh -o quick_start.sh && sudo bash quick_start.sh

# Debian
curl -sSL https://resource.fit2cloud.com/1panel/package/quick_start.sh -o quick_start.sh && bash quick_start.sh
```

> [!tip] 安装时注意
> - 可自定义面板访问端口，例如 `6666`
> - 可自定义用户名和密码

### 2.3 开放防火墙端口

在云服务器后台的**安全组**中开放以下端口：

| 协议 | 端口 | 用途 |
|------|------|------|
| TCP | 21114 | API Web 管理后台 |
| TCP | 21115 | NAT 测试 |
| TCP | 21116 | ID 服务器 (hbbs) |
| UDP | 21116 | ID 服务器 (hbbs) |
| TCP | 21117 | 中继服务器 (hbbr) |
| TCP | 21118 | Web 客户端 |
| TCP | 21119 | Web 客户端 |
| TCP | 6666 | 1Panel 面板（自定义） |

### 2.4 Docker Compose 一键部署

使用 [lejianwen](https://github.com/lejianwen) 修改的服务端镜像（集成 API 功能）。

在 1Panel 面板中：**容器 → 编排 → 创建编排**，粘贴以下配置：

```yaml
networks:
  rustdesk-net:
    external: false
services:
  rustdesk:
    ports:
      - 21114:21114
      - 21115:21115
      - 21116:21116
      - 21116:21116/udp
      - 21117:21117
      - 21118:21118
      - 21119:21119
    image: lejianwen/rustdesk-server-s6:latest
    environment:
      - RELAY=<你的服务器IP或域名>:21117
      - ENCRYPTED_ONLY=1
      - MUST_LOGIN=Y
      - TZ=Asia/Shanghai
      - RUSTDESK_API_RUSTDESK_ID_SERVER=<你的服务器IP或域名>:21116
      - RUSTDESK_API_RUSTDESK_RELAY_SERVER=<你的服务器IP或域名>:21117
      - RUSTDESK_API_RUSTDESK_API_SERVER=http://<你的服务器IP或域名>:21114
      - RUSTDESK_API_KEY_FILE=/data/id_ed25519.pub
      - RUSTDESK_API_JWT_KEY=你的自定义密钥字符串
    volumes:
      - /data/rustdesk/server:/data
      - /data/rustdesk/api:/app/data
    networks:
      - rustdesk-net
    restart: unless-stopped
```

> [!warning] 重要参数说明
> - **RELAY**：中继服务器地址 + 端口（默认 21117）
> - **MUST_LOGIN**：设为 `Y` 表示**必须登录才能发起远程连接**（防滥用关键设置）
> - **RUSTDESK_API_JWT_KEY**：自定义一个随机字符串即可
> - 将所有 `<...>` 替换为你的实际服务器 IP 或域名

### 2.5 获取 Key 并登录 API 后台

**查看 Key：**

```bash
cat /data/rustdesk/server/id_ed25519.pub
```

**登录 API 后台：**
- 地址：`http://你的服务器IP:21114`
- 默认用户名：`admin`
- 默认密码：查看 Docker 容器日志获取

**修改管理员密码：**

```bash
./apimain reset-admin-pwd <你的新密码>
```

> [!danger] 安全提醒
> 登录 API 后台后，**务必第一时间修改默认密码**！

### 2.6 测试部署

将以下信息填入 RustDesk 客户端进行测试：
- **ID 服务器**：你的服务器 IP:21116
- **中继服务器**：你的服务器 IP:21117
- **API 服务器**：http://你的服务器IP:21114
- **Key**：从服务器获取的公钥

确认未登录状态下无法发起远程连接。

---

## 三、修改 RustDesk 源码（将服务器信息内置客户端）

### 3.1 准备工具

| 工具 | 用途 |
|------|------|
| [Git](https://git-scm.com/) | 克隆/推送源码 |
| [HBuilderX](https://sbqx.cc/231.html) | 代码编辑器（轻量免安装） |
| [DevSidecar](https://sbqx.cc/229.html) | 解决 GitHub 访问问题 |

### 3.2 Fork 项目

1. 注册/登录 GitHub
2. Fork [rustdesk/rustdesk](https://github.com/rustdesk/rustdesk) 主项目
3. 进入主项目的 `libs/hbb_common`，点击链接跳转到 [rustdesk/hbb_common](https://github.com/rustdesk/hbb_common) 子模块，也 Fork 到自己账号

> [!tip] 安全建议
> Fork 的仓库默认为**公开**。为防止服务器信息泄露，建议：
> - 创建**私有仓库**，分别导入主项目和子模块源码
> - 或者在编译完成后删除 Fork 仓库

### 3.3 配置 GitHub SSH Key

打开 Git Bash，依次执行：

```bash
# 配置用户名和邮箱
git config --global user.name "你的用户名"
git config --global user.email "你的邮箱"

# 生成 SSH 密钥（一路回车）
ssh-keygen -t rsa -C "你的邮箱"

# 查看公钥（复制输出内容）
cat ~/.ssh/id_rsa.pub
```

将公钥添加到 GitHub：**个人中心 → Settings → SSH and GPG keys → New SSH key**

```bash
# 测试连接
ssh -T git@github.com
```

出现 `Hi xxx! You've successfully authenticated...` 即表示成功。

### 3.4 克隆源码到本地

```bash
# 递归克隆（包含子模块）
git clone --recurse-submodules <你的Fork仓库SSH地址>
cd rustdesk
```

### 3.5 修改子模块路径

编辑主目录下的 `.gitmodules` 文件：

```ini
[submodule libs/hbb_common]
    path = libs/hbb_common
    url = <替换为你Fork的hbb_common仓库SSH地址>
```

同步并提交：

```bash
git submodule sync
git add .gitmodules
git commit -m "更新子模块地址至我的Fork"
git push origin master
```

### 3.6 修改 ID/中继服务器地址

这是**最关键的一步**——将你的服务器信息写入源码。

```bash
# 进入子模块目录
cd libs/hbb_common

# 创建新分支
git checkout -b custom-server
```

用编辑器打开 `libs/hbb_common/src/config.rs`，找到约 **101-102 行**，修改为你的服务器信息：

```rust
// 原代码类似：
// ...
// 修改为你的服务器地址和 Key
```

> [!info] 行号说明
> 具体行号可能随源码版本变化，搜索 `HARD_SETTINGS` 或 `id_server`、`relay_server` 等关键词定位。

提交并推送：

```bash
git add .
git commit -m "改为自己的ID/中继服务器"
git push origin custom-server

# 返回主仓库
cd ../../..
```

更新子模块的 Commit ID：

```bash
git add libs/hbb_common
git commit -m "更新修改后的子模块"
git push origin master
```

### 3.7 修改 API 地址

编辑 `src/common.rs`，找到约 **947 行**，替换 API 地址为你的服务器地址。

```bash
git add .
git commit -m "修改替换API地址"
git push origin master
```

### 3.8 删除客户端广告提示（可选）

编辑 `flutter/lib/desktop/pages/connection_page.dart`，将约 **81-110 行** 替换为：

```dart
Widget setupServerWidget() => Flexible(
   child: Offstage(
     offstage: !(!_svcStopped.value &&
         stateGlobal.svcStatus.value == SvcStatus.ready &&
         _svcIsUsingPublicServer.value),
     child: Row(
       crossAxisAlignment: CrossAxisAlignment.center,
       children: [],
     ),
   ),
 );
```

```bash
git add .
git commit -m "修改删除客户端广告"
git push origin master
```

---

## 四、本地编译 RustDesk 客户端（Windows）

> [!abstract] 概述
> 如果不想依赖 GitHub Actions，可以在本地 Windows 电脑上编译 RustDesk。整体流程：**安装工具链 → 配置 vcpkg → 编译 Rust 后端 → 编译 Flutter 前端 → 合并产物**。

### 4.1 环境要求

| 工具                 | 版本要求                | 用途               |
| ------------------ | ------------------- | ---------------- |
| Visual Studio 2022 | 含 **C++ 桌面开发** 工作负载 | C/C++ 编译环境       |
| Rust               | 1.75（推荐）            | Rust 编译器         |
| Flutter SDK        | 3.24.5（stable）      | GUI 前端框架         |
| LLVM / Clang       | 15.x                | bindgen 依赖       |
| vcpkg              | 特定 commit（见下方）      | C/C++ 依赖管理       |
| Python 3           | 3.x                 | 运行 `build.py` 脚本 |
| Git                | 最新                  | 源码管理             |

### 4.2 安装 Visual Studio 2022

下载 [Visual Studio Build Tools 2022](https://visualstudio.microsoft.com/zh-hans/downloads/)，安装时勾选：

- ✅ **使用 C++ 的桌面开发**（Desktop development with C++）
- ✅ Windows 10/11 SDK
- ✅ MSVC v143 生成工具

### 4.3 安装 Rust 工具链

```powershell
# 安装 rustup（Rust 版本管理器）
winget install Rustlang.Rustup
# 或访问 https://rustup.rs 下载安装器

# 安装指定版本
rustup install 1.75
rustup default 1.75

# 验证
rustc --version
cargo --version
```

### 4.4 安装 LLVM / Clang

RustDesk 的 bindgen 需要 LLVM：

```powershell
winget install LLVM.LLVM
```

安装后确保 `clang` 在 PATH 中：

```powershell
clang --version
```

> [!tip] 如果 clang 不在 PATH
> 将 LLVM 安装路径（默认 `C:\Program Files\LLVM\bin`）添加到系统环境变量 `PATH` 中。

### 4.5 安装 Flutter SDK

```powershell
# 方式一：winget
winget install Flutter.Flutter

# 方式二：手动下载
# 访问 https://docs.flutter.dev/get-started/install/windows
# 下载 Flutter 3.24.5 stable 版本并解压
```

配置环境变量并验证：

```powershell
flutter doctor
# 确保 Flutter 和 Visual Studio 项都打勾 ✓
```

**替换 Flutter 引擎**（RustDesk 使用自定义引擎）：

```powershell
flutter precache --windows

# 下载自定义引擎
Invoke-WebRequest -Uri https://github.com/rustdesk/engine/releases/download/main/windows-x64-release.zip -OutFile windows-x64-release.zip
Expand-Archive -Path windows-x64-release.zip -DestinationPath windows-x64-release

# 替换引擎文件（路径根据实际 Flutter 安装位置调整）
# 将 windows-x64-release 中的文件覆盖到：
# <Flutter安装路径>\bin\cache\artifacts\engine\windows-x64-release\
```

### 4.6 安装和配置 vcpkg

vcpkg 是微软的 C++ 包管理器，RustDesk 依赖它来管理视频编解码等底层库。

#### 4.6.1 克隆 vcpkg

```powershell
# 选择一个安装目录（路径不要含中文和空格）
cd C:\
git clone https://github.com/microsoft/vcpkg
cd vcpkg

# 切换到 RustDesk CI 使用的特定版本（重要！）
git checkout 120deac3062162151622ca4860575a33844ba10b
```

#### 4.6.2 初始化 vcpkg

```powershell
# 在 vcpkg 目录下执行
.\bootstrap-vcpkg.bat
```

> [!warning] 初始化可能较慢
> 首次运行会编译 vcpkg 自身，需要几分钟。确保网络通畅。

#### 4.6.3 设置环境变量

```powershell
# 设置 VCPKG_ROOT（永久生效）
[Environment]::SetEnvironmentVariable("VCPKG_ROOT", "C:\vcpkg", "User")

# 当前会话也生效
$env:VCPKG_ROOT = "C:\vcpkg"
```

#### 4.6.4 安装 RustDesk 所需依赖

在 RustDesk 源码根目录（包含 `vcpkg.json` 的目录）下执行：

```powershell
# 进入 RustDesk 源码目录
cd <你的RustDesk源码目录>

# 安装所有 vcpkg 依赖（manifest 模式，自动读取 vcpkg.json）
$env:VCPKG_ROOT\vcpkg install --triplet x64-windows-static
```

或者手动指定每个包：

```powershell
$env:VCPKG_ROOT\vcpkg install libvpx:x64-windows-static libyuv:x64-windows-static opus:x64-windows-static aom:x64-windows-static libjpeg-turbo:x64-windows-static mfx-dispatch:x64-windows-static ffmpeg[core,avcodec,avformat,swresample,amf,nvcodec,qsv]:x64-windows-static
```

> [!info] vcpkg.json 依赖列表（Windows）
> RustDesk 的 `vcpkg.json` 定义了以下 C/C++ 依赖：
>
> | 包名 | 用途 |
> |------|------|
> | `libvpx` | VP8/VP9 视频编解码 |
> | `libyuv` | YUV 格式转换 |
> | `opus` | Opus 音频编解码 |
> | `aom` | AV1 视频编解码 |
> | `libjpeg-turbo` | JPEG 图像处理 |
> | `mfx-dispatch` | Intel Quick Sync 硬件加速 |
> | `ffmpeg` | 多媒体框架（含 AMF/NVCodec/QSV 硬件编码） |

#### 4.6.5 验证 vcpkg 安装

```powershell
# 查看已安装的包
$env:VCPKG_ROOT\vcpkg list

# 确认静态库文件存在
dir $env:VCPKG_ROOT\installed\x64-windows-static\lib\*.lib
```

### 4.7 编译 RustDesk

#### 4.7.1 克隆源码

```bash
# 如果之前已经按第三章修改过源码，直接用那个目录
# 否则重新克隆
git clone --recurse-submodules <你的Fork仓库地址>
cd rustdesk
```

#### 4.7.2 编译虚拟显示模块

```powershell
cd libs\virtual_display\dylib
cargo build --release
cd ..\..\..
```

#### 4.7.3 编译 Rust 核心库

```powershell
# 编译 Rust 后端（动态链接库）
cargo build --features "flutter,hwcodec,vram" --lib --release
```

> [!note] features 说明
> - `flutter`：使用 Flutter UI（必须）
> - `hwcodec`：启用硬件编解码（AMF/NVCodec/QSV）
> - `vram`：启用 VRAM 直接渲染（仅 Windows）
>
> 如果不需要硬件加速，可以去掉 `hwcodec` 和 `vram`。

编译成功后检查：

```powershell
# 确认生成了动态库
dir target\release\librustdesk.dll
# 应该存在该文件
```

#### 4.7.4 编译 Flutter 前端

```powershell
cd flutter
flutter pub get
flutter build windows --release
cd ..
```

#### 4.7.5 合并产物

```powershell
# 将 Rust 编译产物复制到 Flutter 构建目录
copy target\release\librustdesk.dll flutter\build\windows\x64\runner\Release\
copy target\release\deps\dylib_virtual_display.dll flutter\build\windows\x64\runner\Release\

# 将编译好的整个目录复制出来
xcopy /E /I flutter\build\windows\x64\runner\Release C:\RustDesk-build
```

最终 `C:\RustDesk-build` 目录中的 `rustdesk.exe` 即可直接运行。

#### 4.7.6 使用 build.py 一键编译（推荐）

RustDesk 提供了 `build.py` 脚本，可自动化上述流程：

```powershell
# 确保在源码根目录
python build.py --portable --hwcodec --flutter --vram --skip-portable-pack

# 编译产物在：
# flutter\build\windows\x64\runner\Release\
```

> [!tip] build.py 参数说明
> | 参数 | 说明 |
> |------|------|
> | `--flutter` | 构建 Flutter 版本 |
> | `--hwcodec` | 启用硬件编解码 |
> | `--vram` | 启用 VRAM 渲染 |
> | `--portable` | 构建便携版 |
> | `--skip-portable-pack` | 跳过打包为单文件 exe |

### 4.8 编译时间参考

| 阶段 | 首次编译 | 增量编译 |
|------|----------|----------|
| vcpkg 依赖 | 30-60 分钟 | 已缓存 |
| Rust 编译 | 15-30 分钟 | 2-5 分钟 |
| Flutter 编译 | 5-10 分钟 | 1-2 分钟 |

### 4.9 常见编译错误

#### 找不到 vcpkg 包

```
error: could not find native static library
```

**解决：** 确认 `$env:VCPKG_ROOT` 设置正确，且已执行 `vcpkg install`。

#### libclang 未找到

```
error: `libclang` not found
```

**解决：** 安装 LLVM 并确保在 PATH 中，或设置 `LIBCLANG_PATH`：

```powershell
$env:LIBCLANG_PATH = "C:\Program Files\LLVM\bin"
```

#### Flutter 编译失败

```
CMake Error at flutter/...
```

**解决：** 运行 `flutter doctor` 检查所有依赖，确认 Visual Studio 已正确安装。

---

## 五、GitHub Actions 自动编译客户端（云端方案）

> [!note] 本地编译 vs GitHub Actions
> 如果本地电脑配置不够，或者不想折腾环境搭建，可以使用 GitHub Actions 云端编译，但需要稳定的网络环境。

### 5.1 触发编译

在 GitHub 仓库中进入 **Actions** 标签页，选择对应的编译工作流（如 `Build Windows`），点击 **Run workflow**。

### 5.2 等待编译完成

- Windows 客户端编译约需 **1 小时**
- 编译完成后在 Artifacts 中下载

### 5.3 验证

下载并运行自编译的客户端，确认：
1. 服务器信息已自动填入（无需手动配置）
2. 广告提示已移除
3. 未登录时无法发起远程连接
4. 登录后可正常远程

---

## 六、附加：内置固定连接密码

来自评论区用户的补充，在 `libs/hbb_common/src/config.rs` 中找到 `HARD_SETTINGS`（约第 73 行），修改为：

```rust
pub static ref HARD_SETTINGS: RwLock<HashMap<String, String>> = {
    let mut map = HashMap::new();
    map.insert("password".to_string(), "你的固定密码".to_string());
    RwLock::new(map)
};
```

> [!caution] 安全风险
> 固定密码会编译到所有客户端中，适合个人或小团队内网使用。**不建议在公网场景使用**。

---

## 七、端口速查表

| 端口 | 协议 | 服务 | 说明 |
|------|------|------|------|
| 21114 | TCP | API | Web 管理后台 |
| 21115 | TCP | hbbs | NAT 类型测试 |
| 21116 | TCP/UDP | hbbs | ID 注册/心跳 |
| 21117 | TCP | hbbr | 中继转发 |
| 21118 | TCP | Web Client | 网页端 |
| 21119 | TCP | Web Client | 网页端 |

---

## 八、关于 P2P 直连

### P2P 原理

RustDesk 默认就支持 P2P（点对点）直连，**无需修改 Docker Compose 的 `networks` 配置**。Docker 中的 `rustdesk-net` 只是容器间内部通信网络，与 P2P 无关。

```
客户端A ──── NAT穿透(打洞) ──── 客户端B
   │                                │
   └──── 先通过 hbbs(21116) 信令协商 ────┘
   │                                │
   └──── 打洞成功 → P2P 直连 ─────────┘
   └──── 打洞失败 → 走 hbbr(21117) 中继 ──┘
```

连接流程：
1. 两个客户端通过 **hbbs (21116)** 完成信令交换
2. 自动尝试 NAT 打洞（UDP Hole Punching）
3. 打洞成功 → **P2P 直连**（数据不经过服务器）
4. 打洞失败 → 走 **hbbr (21117)** 中继转发

### 影响 P2P 成功率的因素

| 因素 | 说明 |
|------|------|
| 端口 21115 开放 | 用于 NAT 类型探测，决定打洞策略 |
| 双方 NAT 类型 | 全锥形 NAT 成功率最高，对称型几乎必定走中继 |
| 运营商限制 | 某些运营商会阻止 UDP 打洞 |
| Docker NAT 层 | Docker 自身的 NAT 可能影响打洞 |

> [!tip] Docker 网络模式优化
> 如果 P2P 始终不成功，可将 `network_mode` 改为 `host`，绕过 Docker 自身的 NAT 层：
> ```yaml
> services:
>   rustdesk:
>     image: lejianwen/rustdesk-server-s6:latest
>     network_mode: host  # 直接使用宿主机网络，替代 ports 映射
>     environment:
>       - RELAY=<你的服务器IP>:21117
>       # ... 其余 environment 不变
>     volumes:
>       - /data/rustdesk/server:/data
>       - /data/rustdesk/api:/app/data
>     restart: unless-stopped
>     # 注意：使用 host 模式后，删除 ports 和 networks 部分
> ```
> 大多数情况下标准的端口映射就够用，仅在 P2P 问题时考虑此方案。

---

## 九、常见问题

### Docker 镜像拉取失败

如果在国内服务器拉取 Docker 镜像超时，可配置镜像加速器或使用代理。

### 客户端编译失败

- 确认 Fork 的是最新版本
- 检查子模块是否正确同步
- 确认源码修改没有语法错误

### 忘记 API 密码

通过服务器终端重置：

```bash
./apimain reset-admin-pwd <新密码>
```

---

## 参考资源

- 原文：[RustDesk通过API防止服务器被滥用 - 鼠标迁徙](https://www.smianao.com/1414.html)
- [RustDesk 官方文档](https://rustdesk.com/docs/)
- [lejianwen/rustdesk-server-s6](https://github.com/lejianwen/rustdesk-server-s6)（带 API 的服务端镜像）
- [RustDesk GitHub](https://github.com/rustdesk/rustdesk)
- 工具下载：[WindTerm](https://sbqx.cc/233.html) | [HBuilderX](https://sbqx.cc/231.html) | [DevSidecar](https://sbqx.cc/229.html)
