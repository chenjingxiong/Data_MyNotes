---
title: 搭建ZeroTier Moon服务器
created: 2026-02-08
tags:
---
# 搭建ZeroTier Moon服务器

## 🧑‍💻 User


建立zerotier 的moon服务器 自己的网络使用

---
## 🤖 Assistant

<div style="border: 2px solid #dddddd; border-radius: 10px;">
  <details style="padding: 5px;">
    <summary>已深度思考</summary>
    用户要求建立ZeroTier的moon服务器来在自己的网络中使用。这是一个关于ZeroTier私有网络部署的技术请求。<br><br>我需要搜索关于ZeroTier moon服务器搭建的最新信息，以确保提供准确和最新的指导。<br><br><br>用户询问如何建立ZeroTier的moon服务器来在自己的网络中使用。我需要提供详细的ZeroTier moon服务器搭建教程。<br><br>让我直接基于我的知识来回答这个问题，因为ZeroTier的moon服务器搭建是一个相对标准的技术流程。<br><br>我需要提供：<br>1. 什么是ZeroTier moon<br>2. moon服务器的作用<br>3. 详细的搭建步骤<br>4. 配置方法<br>5. 客户端连接方法<br>6. 注意事项<br><br>让我按照系统提示的要求，使用中文回复，提供完整的技术文档。<br>
  </details>
</div>

# ZeroTier Moon 服务器搭建完整教程

## 一、概述

ZeroTier 是一款优秀的软件定义网络（SDN）解决方案，能够帮助用户快速构建虚拟局域网。而 Moon 服务器是 ZeroTier 网络中的中继服务器，能够显著提升网络连接速度和稳定性，特别是在跨运营商、跨地区的网络环境中表现优异。

### 1.1 为什么需要 Moon 服务器

在默认情况下，ZeroTier 客户端之间会尝试直接连接（P2P直连），这种方式的延迟最低、性能最好。然而，现实网络环境中存在复杂的网络拓扑结构：

- **NAT 穿透失败**：部分网络环境（如对称型 NAT、企业防火墙等）会导致 P2P 连接失败
- **跨运营商通信**：电信和联通、移动之间的互联存在天然的延迟和丢包问题
- **地理位置影响**：跨国家或跨大洲的连接质量难以保证

Moon 服务器的出现就是为了解决这些问题。当 P2P 直连失败时，客户端会自动降级到通过 Moon 服务器中继流量，确保网络的可用性。

### 1.2 Moon 服务器的优势

部署私有的 Moon 服务器能够带来以下优势：

- **提升连接稳定性**：确保在复杂网络环境下也能保持连接
- **降低跨运营商延迟**：选择最优的网络路径
- **增强隐私保护**：使用私有服务器避免公共服务器的安全隐患
- **更高的可控性**：可以随时监控和调整服务器配置
- **带宽保障**：专用的服务器资源提供更稳定的传输性能

## 二、环境准备

### 2.1 服务器要求

搭建 ZeroTier Moon 服务器需要一台具有公网 IP 的服务器，以下是具体配置要求：

**最低配置：**
- CPU：1 核心
- 内存：512 MB
- 带宽：1 Mbps（根据实际需求调整）
- 磁盘：5 GB

**推荐配置：**
- CPU：2 核心及以上
- 内存：1 GB 及以上
- 带宽：10 Mbps 及以上
- 磁盘：20 GB SSD

**操作系统支持：**
- Ubuntu 18.04/20.04/22.04 LTS（推荐）
- Debian 10/11/12
- CentOS 7/8/9
- Rocky Linux 8/9
- 任何支持 Systemd 的 Linux 发行版

### 2.2 网络要求

- 必须拥有公网 IP 地址（静态或动态均可）
- 需要开放 UDP 端口 9993（ZeroTier 默认通信端口）
- 确保防火墙允许该端口的入站流量

### 2.3 获取网络 ID

在开始搭建之前，你需要准备：

1. 访问 [ZeroTier Central](https://my.zerotier.com/) 并注册账号
2. 创建一个新的网络（Network），获取 16 位网络 ID（例如：`8056c2e21c000000`）
3. 将此网络 ID 记录下来，后续配置需要使用

## 三、在 Linux 服务器上搭建 Moon 服务器

以下步骤详细说明如何在 Linux 服务器上部署 ZeroTier Moon 服务器。

### 3.1 安装 ZeroTier

首先，通过官方脚本一键安装 ZeroTier：

```bash
# 下载并执行官方安装脚本
curl -s https://install.zerotier.com | sudo bash
```

安装完成后，启动并设置开机自启：

```bash
# 启动 ZeroTier 服务
sudo systemctl start zerotier-one

# 设置开机自启
sudo systemctl enable zerotier-one

# 验证服务状态
sudo systemctl status zerotier-one
```

### 3.2 加入网络

使用刚才准备的网络 ID 将服务器加入 ZeroTier 网络：

```bash
# 将服务器加入网络（将 8056c2e21c000000 替换为你的网络 ID）
sudo zerotier-cli join 8056c2e21c000000
```

加入网络后，需要在 ZeroTier Central 管理界面中进行授权：

1. 登录 ZeroTier Central
2. 进入对应的网络设置页面
3. 在 "Members" 区域找到刚加入的服务器（显示为未知设备）
4. 勾选该设备的授权选项
5. （可选）为设备设置固定的 IP 地址

### 3.3 创建 Moon 配置文件

在服务器上创建 Moon 配置文件，该文件定义了 Moon 服务器的身份和配置参数：

```bash
# 创建配置文件目录（如果不存在）
sudo mkdir -p /var/lib/zerotier-one

# 创建 moon.json 配置文件
sudo tee /var/lib/zerotier-one/moon.json << 'EOF'
{
    "id": "0000000000000001", 
    "objtype": "moon",
    "roots": [
        {
            "identity": "identity.pub 文件的内容",
            "depth": 32,
            "scores": [
                {
                    "address": "服务器公网IP",
                    "weight": 128
                }
            ]
        }
    ],
    "signingKey": "signingKey.pub 文件的内容",
    "revision": 1
}
EOF
```

### 3.4 生成 Moon 配置

实际上，更简单的方法是使用 ZeroTier 提供的工具来自动生成 Moon 配置：

```bash
# 进入 ZeroTier 数据目录
cd /var/lib/zerotier-one

# 生成 Moon 配置文件
sudo zerotier-idtool initmoon /var/lib/zerotier-one/identity.public > /var/lib/zerotier-one/moon.json
```

生成的 moon.json 文件内容类似如下：

```json
{
  "id": "7axxxxxxxxxx0001",
  "objtype": "moon",
  "signingKey": "bxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "signingKey_SECRET": "axxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "vMajor": 1,
  "vMinor": 2,
  "vRev": 1,
  "roots": [
    {
      "depth": 32,
      "identity": "7xxxxxxxxxx:0:xxxxxxxxxx...xxxxxx",
      "scores": []
    }
  ],
  "revision": 1
}
```

接下来，修改 moon.json 文件，在 scores 数组中添加服务器信息：

```bash
# 使用编辑器修改配置文件
sudo nano /var/lib/zerotier-one/moon.json
```

将文件修改为以下格式（注意将 IP 地址替换为你的服务器公网 IP）：

```json
{
  "id": "7axxxxxxxxxx0001",
  "objtype": "moon",
  "signingKey": "bxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "vMajor": 1,
  "vMinor": 2,
  "vRev": 1,
  "roots": [
    {
      "depth": 32,
      "identity": "7xxxxxxxxxx:0:xxxxxxxxxx...xxxxxx",
      "scores": [
        {
          "address": "你的服务器公网IP",
          "weight": 128
        }
      ]
    }
  ],
  "revision": 1
}
```

### 3.5 创建 Moon 数据文件并启动服务

使用生成的配置文件创建 Moon 数据文件：

```bash
# 生成 .moon 文件
sudo zerotier-idtool genmoon /var/lib/zerotier-one/moon.json
```

此命令会在 `/var/lib/zerotier-one/` 目录下生成一个名为 `<moon_id>.moon` 的文件（例如：`7axxxxxxxxxx0001.moon`）。

重启 ZeroTier 服务以加载 Moon 配置：

```bash
# 重启服务
sudo systemctl restart zerotier-one
```

### 3.6 验证 Moon 服务器状态

验证 Moon 服务器是否正常运行：

```bash
# 查看 Moon 列表
sudo zerotier-cli listmoons

# 查看 zerotier 服务状态
sudo systemctl status zerotier-one

# 查看监听端口
sudo ss -ulnp | grep 9993

# 验证网络连接状态
sudo zerotier-cli listnetworks

# 查看详细状态信息
sudo zerotier-cli info
```

如果一切正常，输出应该显示你的 Moon 服务器信息，包括 Moon ID 和服务器地址。

### 3.7 测试服务器基础连接

在配置客户端之前，建议先验证服务器的基础连接功能：

```bash
# 确认 Moon 文件已生成
ls -la /var/lib/zerotier-one/*.moon

# 检查 ZeroTier 服务日志
sudo journalctl -u zerotier-one -n 50 --no-pager

# 测试本地网络接口
sudo zerotier-cli listnetworks -j
```

确保 Moon 文件存在且服务运行正常后，再进行客户端配置。

### 3.8 配置防火墙

确保服务器的防火墙允许 UDP 9993 端口的流量：

**对于 Ubuntu/Debian（使用 ufw）：**

```bash
# 开放 UDP 9993 端口
sudo ufw allow 9993/udp

# 重新加载防火墙规则
sudo ufw reload

# 查看防火墙状态
sudo ufw status
```

**对于 CentOS/Rocky Linux（使用 firewalld）：**

```bash
# 开放 UDP 9993 端口
sudo firewall-cmd --permanent --add-port=9993/udp

# 重新加载防火墙规则
sudo firewall-cmd --reload

# 查看已开放的端口
sudo firewall-cmd --list-ports
```

**对于阿里云、腾讯云等云服务商：**

还需要在云控制台的安全组规则中开放 UDP 9993 端口。

## 四、在客户端连接 Moon 服务器

**重要提示**：在开始配置客户端之前，请确保：
1. Moon 服务器已成功启动并运行
2. `.moon` 文件已成功生成（路径：`/var/lib/zerotier-one/*.moon`）
3. 防火墙已开放 UDP 9993 端口
4. 已获取正确的 Moon ID（可通过 `sudo zerotier-cli listmoons` 查看）

### 4.1 Windows 客户端

**方法一：通过命令行添加 Moon**

1. 以管理员身份打开命令提示符或 PowerShell
2. 执行以下命令添加 Moon（将 IP 地址替换为你的 Moon 服务器地址）：

```cmd
zerotier-cli orbit <Moon_ID> <Moon_ID>
```

Moon ID 可以在服务器的 `moon.json` 文件中找到，格式为 `7xxxxxxxxxx0001`。

**方法二：通过配置文件添加 Moon**

1. 找到 ZeroTier 安装目录下的配置文件目录，通常位于：
```
   C:\ProgramData\ZeroTier\One\
   ```

2. 在该目录下创建 `moons.d` 文件夹（如果不存在）

3. 将服务器上生成的 `.moon` 文件复制到该文件夹中

4. 重启 ZeroTier 服务或计算机

### 4.2 macOS 客户端

**方法一：通过命令行**

1. 打开终端
2. 执行以下命令添加 Moon：

```bash
sudo zerotier-cli orbit <Moon_ID> <Moon_ID>
```

**方法二：通过配置文件**

1. 在终端中执行以下命令创建配置目录：

```bash
sudo mkdir -p /Library/Application\ Support/ZeroTier/One/moons.d
```

2. 将服务器上的 `.moon` 文件复制到该目录

3. 重启 ZeroTier 服务：

```bash
sudo launchctl kickstart -k com.zerotier.one
```

### 4.3 Linux 客户端

在 Linux 客户端上添加 Moon 服务器的步骤如下：

1. 确保已安装 ZeroTier 客户端：

```bash
curl -s https://install.zerotier.com | sudo bash
```

2. 执行以下命令添加 Moon：

```bash
sudo zerotier-cli orbit <Moon_ID> <Moon_ID>
```

3. 验证 Moon 连接：

```bash
sudo zerotier-cli listmoons
```

### 4.4 移动设备客户端

在 iOS 和 Android 设备上：

1. 安装 ZeroTier 官方应用
2. 连接到你的网络
3. **目前移动客户端的限制**：
   - 官方移动客户端暂不支持手动添加 Moon 服务器
   - 移动设备会自动尝试使用 ZeroTier 官方的根服务器进行中继
   - 如需在移动设备上使用私有 Moon，可以考虑：
     - 使用支持 ZeroTier 的第三方路由器固件（如 OpenWrt）
     - 通过 VPN 方式连接到已配置 Moon 的设备
     - 等待官方更新支持

### 4.5 验证连接状态

在任意客户端上验证是否成功连接到 Moon 服务器：

```bash
# 查看当前加入的网络
zerotier-cli listnetworks

# 查看 Moon 列表
zerotier-cli listmoons

# 查看详细的状态信息
zerotier-cli status
```

当成功连接到 Moon 服务器后，`listmoons` 命令应该能显示你的 Moon 服务器信息。

## 五、在 Docker 中部署 Moon 服务器

对于喜欢使用容器化的用户，也可以通过 Docker 部署 Moon 服务器。

### 5.1 准备工作

```bash
# 安装 Docker（如果尚未安装）
curl -s https://get.docker.com | sudo bash

# 启动 Docker 服务
sudo systemctl start docker
sudo systemctl enable docker
```

### 5.2 创建配置文件

```bash
# 创建工作目录
mkdir -p ~/zerotier-moon
cd ~/zerotier-moon

# 创建 moon.json 配置文件
cat > moon.json << 'EOF'
{
  "id": "0000000000000001",
  "objtype": "moon",
  "vMajor": 1,
  "vMinor": 2,
  "vRev": 1,
  "roots": [
    {
      "identity": "auto_generated_identity",
      "depth": 32,
      "scores": [
        {
          "address": "your_server_ip",
          "weight": 128
        }
      ]
    }
  ],
  "signingKey": "generated_signing_key",
  "signingKey_SECRET": "generated_signing_key_secret"
}
EOF
```

### 5.3 运行 Moon 容器

```bash
# 运行 ZeroTier Moon 容器
docker run -d \
--name zerotier-moon \
--restart always \
--device /dev/net/tun \
--cap-add NET_ADMIN \
-p 9993:9993/udp \
-v ~/zerotier-moon:/var/lib/zerotier-one \
erinn/zerotier:latest
```

### 5.4 配置 Moon

```bash
# 进入容器
docker exec -it zerotier-moon /bin/bash

# 生成配置文件
cd /var/lib/zerotier-one
zerotier-idtool initmoon identity.public > moon.json

# 退出容器
exit

# 重启容器
docker restart zerotier-moon
```

### 5.5 获取 .moon 文件

```bash
# 从容器中复制 .moon 文件
docker cp zerotier-moon:/var/lib/zerotier-one/*.moon ./

# 查看生成的 moon 文件
ls -la *.moon
```

## 六、高级配置与优化

### 6.1 自定义 Moon ID

每个 Moon 服务器都有唯一的 ID，格式为 `xxxxxxxxxx0001`。最后四位始终是 `0001` 表示这是 Moon 服务器。

### 6.2 负载均衡配置

如果需要更高的可用性和性能，可以部署多个 Moon 服务器：

```json
{
"roots": [
 {
 "identity": "moon1_identity:0:xxxxx",
 "depth": 32,
 "scores": [
 { "address": "moon1_ip", "weight": 128 }
 ]
 },
 {
 "identity": "moon2_identity:0:xxxxx",
 "depth": 32,
 "scores": [
 { "address": "moon2_ip", "weight": 128 }
 ]
 }
]
}
```

### 6.3 性能优化建议

**系统层面：**

```bash
# 优化网络缓冲区
sysctl -w net.core.rmem_max=26214400
sysctl -w net.core.wmem_max=26214400
sysctl -w net.ipv4.udp_rmem_min=16384
sysctl -w net.ipv4.udp_wmem_min=16384
```

**ZeroTier 配置优化：**

在 `/var/lib/zerotier-one/local.conf` 中添加：

```json
{
"settings": {
 "portMappingEnabled": true,
 "softwareUpdate": "disable",
 "allowTCPFallback": true
}
}
```

### 6.4 日志管理

查看 Moon 服务器日志：

```bash
# 使用 journalctl 查看日志
sudo journalctl -u zerotier-one -f

# 查看最近的日志
sudo journalctl -u zerotier-one -n 100
```

配置日志轮转：

```bash
# 创建日志配置
sudo tee /etc/logrotate.d/zerotier << 'EOF'
/var/log/zerotier-one/*.log {
 daily
 rotate 7
 compress
 delaycompress
 missingok
 notifempty
}
EOF
```

### 6.5 监控脚本

创建一个简单的监控脚本来跟踪 Moon 服务器状态：

```bash
#!/bin/bash
# moon-monitor.sh

LOG_FILE="/var/log/zerotier-moon-monitor.log"

log() {
 echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a $LOG_FILE
}

check_port() {
 if ss -ulnp | grep -q ":9993 "; then
 log "端口 9993 正常"
 return 0
 else
 log "警告: 端口 9993 未监听"
 return 1
 fi
}

check_service() {
 if systemctl is-active --quiet zerotier-one; then
 log "ZeroTier 服务运行正常"
 return 0
 else
 log "错误: ZeroTier 服务未运行"
 return 1
 fi
}

check_moon() {
 MOONS=$(zerotier-cli listmoons 2>/dev/null)
 if [ -n "$MOONS" ]; then
 log "Moon 服务器信息: $MOONS"
 return 0
 else
 log "警告: 未找到 Moon 配置"
 return 1
 fi
}

check_bandwidth() {
 # 检查网络接口流量
 RX_BYTES=$(cat /sys/class/net/zt*/statistics/rx_bytes 2>/dev/null | awk '{sum+=$1} END {print sum}')
 TX_BYTES=$(cat /sys/class/net/zt*/statistics/tx_bytes 2>/dev/null | awk '{sum+=$1} END {print sum}')
 if [ -n "$RX_BYTES" ]; then
   log "接收字节: ${RX_BYTES}, 发送字节: ${TX_BYTES}"
 fi
}

log "开始 Moon 服务器健康检查..."
check_port
check_service
check_moon
check_bandwidth
log "检查完成"
```

部署监控脚本：

```bash
# 保存脚本到系统目录
sudo tee /usr/local/bin/moon-monitor.sh > /dev/null << 'EOF'
#!/bin/bash
# [复制上面的完整脚本内容到这里]
EOF

# 设置执行权限
sudo chmod +x /usr/local/bin/moon-monitor.sh

# 创建日志目录
sudo touch /var/log/zerotier-moon-monitor.log
sudo chmod 644 /var/log/zerotier-moon-monitor.log

# 添加定时任务（假设脚本保存在 /usr/local/bin/）
echo "* * * * * root /usr/local/bin/moon-monitor.sh" | sudo tee /etc/cron.d/moon-monitor
```

## 七、常见问题排查

### 7.1 连接问题

**问题：客户端无法连接到 Moon 服务器**

**排查步骤：**

1. 检查服务器防火墙是否开放 UDP 9993 端口：
```bash
sudo ss -ulnp | grep 9993
```

2. 确认服务器公网 IP 可达：
```bash
telnet your_server_ip 9993
```

3. 检查 Moon 配置文件是否正确：
```bash
cat /var/lib/zerotier-one/moon.json
```

4. 验证 .moon 文件是否存在：
```bash
ls -la /var/lib/zerotier-one/*.moon
```

### 7.2 性能问题

**问题：网络延迟高或带宽不足**

**解决方案：**

1. 选择网络条件更好的服务器位置
2. 增加服务器带宽配置
3. 部署多个 Moon 服务器进行负载均衡
4. 检查服务器资源使用情况：
```bash
top
iostat -x 1
```

### 7.3 服务启动问题

**问题：ZeroTier 服务无法启动**

**排查步骤：**

1. 检查系统日志：
```bash
sudo journalctl -xe
```

2. 检查端口是否被占用：
```bash
sudo ss -tulnp | grep 9993
```

3. 验证配置文件语法：
```bash
python3 -m json.tool /var/lib/zerotier-one/moon.json
```

4. 重新生成身份信息：
```bash
sudo rm -f /var/lib/zerotier-one/identity*
sudo systemctl restart zerotier-one
```

### 7.4 认证问题

**问题：设备已加入网络但无法通信**

**解决方案：**

1. 确认已在 ZeroTier Central 中授权该设备
2. 检查网络配置中的 IP 地址分配
3. 验证网络成员列表中的设备状态
4. 检查是否有 IP 地址冲突

## 八、安全建议

### 8.1 服务器安全

1. **定期更新系统**：
```bash
sudo apt update && sudo apt upgrade -y
```

2. **配置 SSH 密钥认证**：
```bash
# 生成 SSH 密钥
ssh-keygen -t ed25519

# 将公钥复制到服务器
ssh-copy-id user@your_server_ip
```

3. **配置防火墙规则**：
```bash
# 仅允许必要的端口
sudo ufw default deny incoming
sudo ufw allow ssh
sudo ufw allow 9993/udp
sudo ufw enable
```

### 8.2 ZeroTier 安全

1. **禁用旧版加密**：
在 `/var/lib/zerotier-one/local.conf` 中添加：
```json
{
 "settings": {
 "peerChecks": {
 "cryptoEnabled": true
 }
 }
}
```

2. **启用网络访问控制**：
在 ZeroTier Central 中配置细粒度的网络访问规则。

3. **定期轮换密钥**：
定期更换 Moon 服务器的身份密钥。

4. **限制管理访问**：
在 `/var/lib/zerotier-one/local.conf` 中添加：
```json
{
  "settings": {
    "allowManagementFrom": ["127.0.0.1", "your.trusted.ip"],
    "allowTcpFallback": true,
    "allowGlobal": false
  }
}
```

### 8.3 监控与日志

1. **启用入侵检测**：
```bash
sudo apt install fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

2. **配置日志集中管理**：
使用 rsyslog 或 ELK Stack 集中管理日志。

## 九、总结

通过本教程，你应该已经掌握了以下技能：

- **理解 ZeroTier Moon 服务器的原理和作用**：Moon 服务器作为中继节点，能够在 P2P 连接失败时提供稳定的网络连接
- **在 Linux 服务器上完整部署 Moon 服务器**：从安装、配置到启动的完整流程
- **在各种客户端平台上连接 Moon 服务器**：Windows、macOS、Linux 等平台的连接方法
- **使用 Docker 容器化部署**：灵活的容器化解决方案
- **高级配置和优化技巧**：性能优化、负载均衡、监控等
- **问题排查方法**：常见问题的诊断和解决方案

部署私有 Moon 服务器能够显著提升 ZeroTier 网络的稳定性和性能，特别是在复杂的网络环境下效果明显。建议根据实际需求选择合适的服务器配置，并定期监控和维护服务器状态。

如果你在使用过程中遇到任何问题，可以参考官方文档或社区资源获取更多帮助。ZeroTier 的官方文档地址为：https://docs.zerotier.com/
