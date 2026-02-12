---
title: 搭建ZeroTier Moon服务器
created: 2026-02-08
tags:
---
# 搭建ZeroTier Moon服务器

> **Moon服务器**：ZeroTier的中继服务器，当P2P直连失败时自动中继流量，提升复杂网络环境下的连接稳定性。

## 前置要求

- 服务器需有**公网IP**
- 开放 **UDP 9993** 端口
- 在 [ZeroTier Central](https://my.zerotier.com/) 创建网络并获取网络ID

## 方法一：Linux 服务器部署

### 1. 安装 ZeroTier

```bash
curl -s https://install.zerotier.com | sudo bash
sudo systemctl start zerotier-one
sudo systemctl enable zerotier-one
```

### 2. 加入网络

```bash
# 替换为你的网络ID
sudo zerotier-cli join 8056c2e21c000000
```

→ 到 [ZeroTier Central](https://my.zerotier.com/) 授权该设备

### 3. 生成 Moon 配置

```bash
cd /var/lib/zerotier-one
# 生成配置模板
sudo zerotier-idtool initmoon identity.public > moon.json
```

编辑 `moon.json`，添加 `stableEndpoints`：

```json
{
  "roots": [
    {
      "identity": "xxxxx:0:xxxxx...",
      "stableEndpoints": ["你的公网IP/9993"]
    }
  ]
}
```

> **注**：`stableEndpoints` 是 ZeroTier 1.6.0+ 推荐的配置方式，比传统的 `scores` 数组更简洁。
```

### 4. 启动 Moon

```bash
# 生成 .moon 文件
sudo zerotier-idtool genmoon moon.json
# 重启服务
sudo systemctl restart zerotier-one
```

### 5. 验证

```bash
sudo zerotier-cli listmoons
```

---

## 方法二：OpenWrt 路由器部署

### 1. 安装 ZeroTier

OpenWrt 18.06+ 可通过 opkg 安装：

```bash
# 更新软件源
opkg update

# 安装 ZeroTier
opkg install zerotier
```

### 2. 配置 ZeroTier

编辑 `/etc/config/zerotier`：

```bash
config zerotier 'sample_config'
    option enabled '1'
    option join '8056c2e21c000000'  # 替换为你的网络ID
    # option port '9993'  # 可选，默认端口
```

启动服务：

```bash
/etc/init.d/zerotier start
/etc/init.d/zerotier enable
```

### 3. 配置防火墙

编辑 `/etc/config/firewall`，添加 ZeroTier 区域：

```bash
# ZeroTier 接口规则
config zone
    option name 'zerotier'
    option input 'ACCEPT'
    option output 'ACCEPT'
    option forward 'ACCEPT'
    option mtu_fix '1'

# 允许 zerotier 接口到 LAN 的转发
config forwarding
    option src 'zerotier'
    option dest 'lan'

config forwarding
    option src 'lan'
    option dest 'zerotier'

# 开放 UDP 9993 端口
config rule
    option name 'Allow-ZeroTier'
    option src 'wan'
    option dest_port '9993'
    option proto 'udp'
    option target 'ACCEPT'
```

重载防火墙：

```bash
/etc/init.d/firewall restart
```

或通过 LuCI 界面：网络 → 防火墙 → 添加 zerotier 接口到允许区域

### 4. 配置 Moon

```bash
cd /etc/zerotier

# 生成配置模板
zerotier-idtool initmoon identity.public > moon.json
```

编辑 `moon.json`，添加 `stableEndpoints`（支持动态域名）：

```json
{
  "roots": [
    {
      "identity": "xxxxx:0:xxxxx...",
      "stableEndpoints": ["your.domain.com/9993"]  # 或直接用公网IP/9993
    }
  ]
}
```

生成并启动 Moon：

```bash
# 生成 .moon 文件
zerotier-idtool genmoon moon.json
# 重启服务
/etc/init.d/zerotier restart
```

### 5. 路由器端口转发

如果路由器背后还有上级路由：

```bash
# 在上级路由配置端口转发
# 外部 UDP 9993 → 路由器内网IP UDP 9993
```

### 6. 验证

```bash
zerotier-cli listmoons
zerotier-cli listnetworks
```

---

## 客户端连接 Moon

### Windows/macOS/Linux

```bash
# 方式一：命令行添加（将MOON_ID替换为实际的moon id）
zerotier-cli orbit MOON_ID MOON_ID

# 验证
zerotier-cli listmoons
```
 
 
> 不同客户端版本 moon-id 不一致，需要检查本机 zerotier 版本（zerotier-cli -v 命令可查看版本），如：
> - 在 zerotier 1.10.6 版本中，moon-id 是 16 位，前面有多余 0
> - 在 zerotier 1.12.2 版本后，moon-id 是 10 位字符串
> 
> 总的来说，在新版中 moon-id 是 10 位字符串，需要注意删除前面多余的 0



**方式二**：将服务器生成的 `.moon` 文件复制到客户端：

| 平台      | 配置目录                                                 |
| ------- | ---------------------------------------------------- |
| Windows | `C:\ProgramData\ZeroTier\One\moons.d\`               |
| macOS   | `/Library/Application Support/ZeroTier/One/moons.d/` |
| Linux   | `/var/lib/zerotier-one/moons.d/`                     |

### OpenWrt 客户端连接

```bash
# 添加 moon
zerotier-cli orbit MOON_ID MOON_ID
# 验证
zerotier-cli listmoons
```

> 
> 不同客户端版本 moon-id 不一致，需要检查本机 zerotier 版本（zerotier-cli -v 命令可查看版本），如：
> 
> - 在 zerotier 1.10.6 版本中，moon-id 是 16 位，前面有多余 0
> - 在 zerotier 1.12.2 版本后，moon-id 是 10 位字符串
> 
> 总的来说，在新版中 moon-id 是 10 位字符串，需要注意删除前面多余的 0
> 
 

```

zerotier-cli orbit <moon-id> <moon-id>

```

注意版本差异：

- 1.10.6 版本：使用 16 位 moon-id（含前导零）

```

zerotier-cli orbit 000000a62f602019 000000a62f602019

```

- 1.12.2+ 版本：使用 10 位 moon-id（去除前导零）

```

zerotier-cli orbit a62f602019 a62f602019

```

orbit 命令会返回 200 状态码表示添加成功。如果 orbit 一直返回 404 状态需要检查上文的版本，多重试几次 orbit

#### 3.2 检查所添加的 moon 状态

可以通过 listmoons 和 listpeers 检查 moon 状态
---

## 防火墙配置

### ufw (Ubuntu/Debian)

```bash
sudo ufw allow 9993/udp
```

### firewalld (CentOS/Rocky)

```bash
sudo firewall-cmd --permanent --add-port=9993/udp
sudo firewall-cmd --reload
```

### 云服务商安全组

记得在控制台开放 **UDP 9993**

---

## 常见问题

**Q: 客户端无法连接 Moon？**
- 检查服务器 UDP 9993 端口是否开放：`ss -ulnp \| grep 9993`
- 确认 `.moon` 文件已生成：`ls /var/lib/zerotier-one/*.moon`

**Q: OpenWrt 无法连接？**
- 检查 `/etc/config/firewall` 是否配置 zerotier 区域
- 确认上级路由已做端口转发

**Q: 如何获取 Moon ID？**
- 在服务器执行：`zerotier-cli listmoons`
- 或查看 `moon.json` 中的 `id` 字段

