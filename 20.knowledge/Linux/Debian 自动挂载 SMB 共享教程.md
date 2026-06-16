# Debian 自动挂载 SMB 共享教程

## 1. 安装必要软件包

```bash
sudo apt update
sudo apt install cifs-utils
```

> `cifs-utils` 提供 SMB/CIFS 协议的挂载支持。

## 2. 创建挂载点

```bash
sudo mkdir -p /mnt/nas
```

> 挂载点路径可自定义，如 `/mnt/share`、`/media/smb` 等。

## 3. 创建凭据文件（推荐）

将用户名密码存放在安全文件中，避免明文暴露在 `/etc/fstab`：

```bash
sudo nano /etc/smbcredentials
```

写入以下内容：

```
username=你的用户名
password=你的密码
domain=WORKGROUP
```

设置权限，仅 root 可读：

```bash
sudo chmod 600 /etc/smbcredentials
```

## 4. 配置 /etc/fstab 自动挂载

编辑 fstab：

```bash
sudo nano /etc/fstab
```

在文件末尾添加一行：

```fstab
//192.168.1.100/share  /mnt/nas  cifs  credentials=/etc/smbcredentials,uid=1000,gid=1000,iocharset=utf8,vers=3.0  0  0
```

### 参数说明

| 参数                      | 作用                                  |
| ----------------------- | ----------------------------------- |
| `//192.168.1.100/share` | SMB 服务器地址和共享名                       |
| `/mnt/nas`              | 本地挂载点                               |
| `cifs`                  | 文件系统类型                              |
| `credentials=...`       | 凭据文件路径                              |
| `uid=1000,gid=1000`     | 挂载后文件所属用户/组（用 `id` 命令查看你的 uid）      |
| `iocharset=utf8`        | 字符编码                                |
| `vers=3.0`              | SMB 协议版本（可选 `2.0`、`3.0`，老设备用 `1.0`） |
| `0 0`                   | 不备份、不 fsck                          |

### 可选附加参数

- `nofail` — 服务器不可用时**不阻止开机启动**
- `x-systemd.automount` — 按需挂载（首次访问时才挂载，节省资源）
- `file_mode=0644,dir_mode=0755` — 自定义文件/目录权限
- `sec=ntlmssp` — 兼容旧版认证方式

**带可选参数的完整示例：**

```fstab
//192.168.1.100/share  /mnt/nas  cifs  credentials=/etc/smbcredentials,uid=1000,gid=1000,iocharset=utf8,vers=3.0,nofail,x-systemd.automount  0  0
```

## 5. 测试挂载

先测试能否正常挂载（不重启）：

```bash
sudo mount -a
```

检查挂载状态：

```bash
df -h | grep /mnt/nas
# 或
mount | grep /mnt/nas
```

能正常看到容量信息即表示成功 ✅

## 6. 开机自动加载（Systemd Mount Unit）

除了 `/etc/fstab`，更推荐使用 **systemd mount unit**，它对网络挂载的控制更精细，支持依赖管理和自动重试。

### 6.1 创建 Mount Unit

> [!important] 文件名必须与挂载点路径对应
> 挂载点 `/mnt/nas` → 文件名 `mnt-nas.mount`（路径中的 `/` 替换为 `-`）。

```bash
sudo nano /etc/systemd/system/mnt-nas.mount
```

写入以下内容：

```ini
[Unit]
Description=Mount SMB Share
Requires=network-online.target
After=network-online.target

[Mount]
What=//192.168.1.100/share
Where=/mnt/nas
Type=cifs
Options=credentials=/etc/smbcredentials,uid=1000,gid=1000,iocharset=utf8,vers=3.0,_netdev

[Install]
WantedBy=multi-user.target
```

### 6.2 配合 Automount 按需挂载（推荐）

Automount 单元可以在**首次访问挂载点时**才触发挂载，避免开机时因网络未就绪而失败：

```bash
sudo nano /etc/systemd/system/mnt-nas.automount
```

```ini
[Unit]
Description=Automount SMB Share

[Automount]
Where=/mnt/nas
TimeoutIdleSec=300

[Install]
WantedBy=multi-user.target
```

> `TimeoutIdleSec=300` 表示空闲 5 分钟后自动卸载，节省连接资源。设为 `0` 则永不卸载。

### 6.3 启用并启动

```bash
# 如果之前在 fstab 中配置过，先移除对应行并卸载
sudo umount /mnt/nas

# 启用 mount unit（不用 automount 时）
sudo systemctl enable --now mnt-nas.mount

# 或者：启用 automount（推荐，二选一）
sudo systemctl enable --now mnt-nas.automount
```

验证：

```bash
systemctl status mnt-nas.mount
# 或
systemctl status mnt-nas.automount
```

> [!warning] 注意
> 启用了 automount 后，**不要同时手动 `mount -a`**，由 systemd 管理挂载生命周期即可。

## 7. 网络变化自动重连与刷新

网络断开恢复后，SMB 挂载可能变成僵死状态（`df` 卡住、`ls` 无响应）。以下提供两种自动重连方案。

### 方案 A：NetworkManager Dispatcher（推荐）

当 NetworkManager 检测到网络状态变化时自动触发重连脚本。

**1. 创建 dispatcher 脚本：**

```bash
sudo nano /etc/NetworkManager/dispatcher.d/99-smb-reconnect.sh
```

```bash
#!/bin/bash

INTERFACE="$1"
ACTION="$2"
MOUNT_POINT="/mnt/nas"

case "$ACTION" in
    up|vpn-up|connectivity-change)
        # 网络恢复：等待 5 秒确保连接稳定，然后重新挂载
        logger "NM Dispatcher: Network up on $INTERFACE, reconnecting SMB..."
        sleep 5

        # 先卸载（可能已僵死，用 lazy 确保成功）
        umount -l "$MOUNT_POINT" 2>/dev/null

        # 重新挂载
        mount "$MOUNT_POINT" 2>/dev/null

        if mountpoint -q "$MOUNT_POINT"; then
            logger "NM Dispatcher: SMB reconnected successfully."
        else
            logger "NM Dispatcher: SMB reconnect failed, will retry on next network event."
        fi
        ;;
    down)
        # 网络断开：懒卸载，防止僵死挂载阻塞 I/O
        logger "NM Dispatcher: Network down on $INTERFACE, lazy unmounting SMB..."
        umount -l "$MOUNT_POINT" 2>/dev/null
        ;;
esac
```

**2. 设置权限：**

```bash
sudo chmod 755 /etc/NetworkManager/dispatcher.d/99-smb-reconnect.sh
sudo chown root:root /etc/NetworkManager/dispatcher.d/99-smb-reconnect.sh
```

**3. 确保 NetworkManager 服务已启用：**

```bash
sudo systemctl enable --now NetworkManager
```

### 方案 B：Systemd 定时健康检查

不依赖 NetworkManager，通过定时器周期性检查挂载状态并自动修复。

**1. 创建健康检查服务：**

```bash
sudo nano /etc/systemd/system/smb-healthcheck.service
```

```ini
[Unit]
Description=SMB Mount Health Check
After=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/smb-healthcheck.sh
```

**2. 创建检查脚本：**

```bash
sudo nano /usr/local/bin/smb-healthcheck.sh
```

```bash
#!/bin/bash

MOUNT_POINT="/mnt/nas"
TEST_FILE="$MOUNT_POINT/.smb-healthcheck"

# 检查挂载点是否存在且为 CIFS 挂载
if ! mountpoint -q "$MOUNT_POINT"; then
    logger "SMB Healthcheck: Not mounted, attempting mount..."
    mount "$MOUNT_POINT" 2>/dev/null
    exit 0
fi

# 尝试读取测试文件（设置 5 秒超时防止卡住）
if ! timeout 5 ls "$MOUNT_POINT" > /dev/null 2>&1; then
    logger "SMB Healthcheck: Mount appears stale, reconnecting..."
    umount -l "$MOUNT_POINT" 2>/dev/null
    sleep 2
    mount "$MOUNT_POINT" 2>/dev/null

    if mountpoint -q "$MOUNT_POINT"; then
        logger "SMB Healthcheck: Reconnected successfully."
    else
        logger "SMB Healthcheck: Reconnect failed."
    fi
fi
```

```bash
sudo chmod 755 /usr/local/bin/smb-healthcheck.sh
```

**3. 创建定时器（每 3 分钟检查一次）：**

```bash
sudo nano /etc/systemd/system/smb-healthcheck.timer
```

```ini
[Unit]
Description=SMB Mount Health Check Timer

[Timer]
OnBootSec=1min
OnUnitActiveSec=3min
AccuracySec=30sec

[Install]
WantedBy=timers.target
```

**4. 启用定时器：**

```bash
sudo systemctl enable --now smb-healthcheck.timer
```

查看定时器状态：

```bash
systemctl list-timers smb-healthcheck.timer
```

> [!tip] 方案选择建议
> - **桌面环境 / 笔记本** → 选 **方案 A**（NetworkManager），响应网络切换更及时。
> - **服务器 / 无头设备** → 选 **方案 B**（定时器），不依赖 NetworkManager，兼容性更广。
> - **两者可同时启用**，互不冲突，实现双层保障。

## 8. 常见问题排查

| 问题 | 解决方案 |
|------|----------|
| `Host is down` | 检查 IP 地址、网络连通性 (`ping`) |
| `Permission denied` | 检查凭据文件中的用户名密码 |
| `Operation not supported` | 尝试更换 `vers=` 参数（如 `vers=2.0` 或 `vers=1.0`） |
| `Resource temporarily unavailable` | 服务器可能已挂载，用 `sudo umount /mnt/nas` 后重试 |
| 开机卡住无法启动 | 确保添加了 `nofail` 参数，或改用 systemd automount 方案 |
| 挂载僵死（`df` / `ls` 卡住） | `sudo umount -l /mnt/nas` 懒卸载后重新挂载；配置健康检查自动修复 |
| 网络切换后无法访问 | 配置 NetworkManager dispatcher 自动重连（见第 7 节） |
| systemd mount unit 启动失败 | 用 `journalctl -u mnt-nas.mount` 查看详细日志 |

---

> [!tip] 小贴士
> - 用 `smbclient -L //192.168.1.100 -U 用户名` 可列出服务器上所有可用共享。
> - 路径中的空格用 `\040` 转义，如 `//server/my\040share`。
> - `x-systemd.idle-timeout=300` 可在 fstab 条目中实现 automount 的空闲超时效果。
> - 使用 `lazy unmount`（`umount -l`）是处理僵死 SMB 挂载的安全方式，不会强制中断正在进行的 I/O。
