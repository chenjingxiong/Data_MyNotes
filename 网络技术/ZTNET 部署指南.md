---
title: 使用 ZTNET 管理 ZeroTier 网络服务器
created: 2026-02-08
tags:
  - ZeroTier
  - ZTNET
  - 网络管理
  - 容器化
---

# 使用 ZTNET 管理 ZeroTier 网络服务器

## 概述

ZTNET 是一个功能强大的 Web UI 界面，用于管理自托管的 ZeroTier 网络控制器。相比传统的 Moon 服务器，ZTNET 提供的 Private Root Server（私有根服务器）具有以下优势：

### ZTNET vs Moon 服务器对比

| 特性         | Moon 服务器            | ZTNET (Private Root Server) |
| ---------- | ------------------- | --------------------------- |
| **管理界面**   | 命令行                 | 🌐 Web UI 界面                |
| **网络管理**   | 需要 ZeroTier Central | ✅ 完全自托管                     |
| **配置复杂度**  | 手动编辑 JSON           | 🖱️ 可视化操作                   |
| **多节点支持**  | 需要手动配置              | ✅ 一键添加                      |
| **根服务器类型** | 辅助中继节点              | 完全替换公共根服务器                  |
| **数据持久化**  | 手动备份                | ✅ 自动备份                      |
| **用户管理**   | ❌ 不支持               | ✅ 多用户支持                     |

### 2026 年官方政策更新

> ⚠️ **重要提醒**：ZeroTier 官方在 2026 年已不再推荐部署私有 Moon 服务器，且不受 SLA 支持。建议使用 **Private Root Server（通过 ZTNET）** 或使用官方的 ZeroTier Central 服务。

## 环境要求

### 服务器要求

**最低配置：**
- CPU：2 核心
- 内存：1 GB
- 磁盘：20 GB
- 操作系统：Ubuntu 20.04/22.04 或 CentOS 8/9

**推荐配置：**
- CPU：4 核心
- 内存：2 GB
- 磁盘：50 GB SSD
- 操作系统：Ubuntu 22.04 LTS

### 网络要求

- 公网 IP 地址（静态或动态）
- 开放 TCP 和 UDP 端口 9993
- 支持的协议：IPv4 和 IPv6

## 快速部署

### Docker Compose 部署（官方推荐）

根据官方文档，ZTNET 需要三个服务组件：

- **PostgreSQL**：数据库服务（存储网络配置、用户数据等）
- **ZeroTier**：核心网络服务（使用 `zyclonite/zerotier` 镜像）
- **ZTNET**：Web UI 管理界面

> ⚠️ **重要说明**：`sinamics/ztnet` 镜像**不包含** ZeroTier 核心服务和数据库，必须使用 Docker Compose 部署完整的三个服务。

创建工作目录：
```bash
mkdir -p ~/ztnet-deploy
cd ~/ztnet-deploy
```

创建 `docker-compose.yml` 文件：
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15.2-alpine
    container_name: ztnet-postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres        # ⚠️ 生产环境请修改此密码
      POSTGRES_DB: ztnet
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - ztnet-network

  zerotier:
    image: zyclonite/zerotier:1.14.0
    hostname: zerotier
    container_name: ztnet-zerotier
    restart: unless-stopped
    volumes:
      - zerotier-data:/var/lib/zerotier-one
    cap_add:
      - NET_ADMIN
      - SYS_ADMIN
    devices:
      - /dev/net/tun:/dev/net/tun
    networks:
      - ztnet-network
    ports:
      - "9993:9993/udp"
    environment:
      - ZT_OVERRIDE_LOCAL_CONF=true
      - ZT_ALLOW_MANAGEMENT_FROM=172.31.255.0/29

  ztnet:
    image: sinamics/ztnet:latest
    container_name: ztnet-web
    working_dir: /app
    volumes:
      - zerotier-data:/var/lib/zerotier-one
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      # 数据库配置
      POSTGRES_HOST: postgres
      POSTGRES_PORT: 5432
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres        # ⚠️ 与上面设置的密码一致
      POSTGRES_DB: ztnet
      # 认证配置
      NEXTAUTH_URL: "http://localhost:3000"    # ⚠️ 修改为你的服务器 IP 或域名
      NEXTAUTH_SECRET: "random_secret"          # ⚠️ 生产环境请使用随机字符串
      NEXTAUTH_URL_INTERNAL: "http://ztnet:3000"
    networks:
      - ztnet-network
    depends_on:
      - postgres
      - zerotier

volumes:
  postgres-data:
  zerotier-data:

networks:
  ztnet-network:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 172.31.255.0/29
```

启动服务：
```bash
docker-compose up -d

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs -f ztnet
```

## 服务说明

### 架构说明

ZTNET 采用三服务架构：

| 服务 | 镜像 | 作用 |
|------|------|------|
| **postgres** | `postgres:15.2-alpine` | 存储 ZTNET 的网络配置、用户、组织等数据 |
| **zerotier** | `zyclonite/zerotier:1.14.0` | ZeroTier 核心网络服务 |
| **ztnet** | `sinamics/ztnet:latest` | Web UI 管理界面，依赖前两个服务 |

### 启动服务

```bash
docker-compose up -d

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs -f ztnet
```

## 详细配置步骤

### 1. 访问 ZTNET Web UI

1. 打开浏览器访问 `http://your_server_ip:3000`
2. 首次访问会自动创建管理员账户
3. 设置管理员用户名和密码

### 2. 配置网络控制器

1. 登录后进入 `Admin → ZT Controller` 页面
2. 点击 `Generate Planet` 按钮
3. 系统会自动检测服务器外网 IP
4. 如需自定义，可以手动输入 `<IP>:9993`
5. 添加身份注释（可选）
6. 点击 `CREATE PLANET`

**注意**：
- 请将 `NEXTAUTH_URL` 修改为你的服务器实际 IP 地址或域名
- 生产环境务必修改 `POSTGRES_PASSWORD` 和 `NEXTAUTH_SECRET`

### 3. 验证 Private Root 创建

```bash
# 进入 ZeroTier 容器检查
docker exec -it ztnet-zerotier zerotier-cli listpeers | grep PLANET
```

成功后应该只显示你的私有根服务器，不会出现公共的 PLANET 服务器。

### 4. 下载配置文件

1. 在 ZTNET 界面点击 `DOWNLOAD CONFIG`
2. 下载的压缩包包含：
   - `planet.custom` - 自定义的 planet 文件
   - `mkworld.config.json` - 配置模板
   - `current.c25519` 和 `previous.c25519` - 密钥文件

## 配置客户端设备

### Windows 客户端

1. 备份原有 planet 文件：
   ```cmd
   copy %PROGRAMDATA%\ZeroTier\One\planet %PROGRAMDATA%\ZeroTier\One\planet.bak
   ```

2. 将 `planet.custom` 重命名为 `planet`
3. 复制到 `%PROGRAMDATA%\ZeroTier\One\` 目录
4. 重启 ZeroTier 服务：
   ```cmd
   net stop ZeroTierOne
   net start ZeroTierOne
   ```

### Linux 客户端

1. 备份原有文件：
   ```bash
   sudo cp /var/lib/zerotier-one/planet /var/lib/zerotier-one/planet.bak
   ```

2. 替换新文件：
   ```bash
   sudo cp planet.custom /var/lib/zerotier-one/planet
   ```

3. 重启服务：
   ```bash
   sudo systemctl restart zerotier-one
   ```

### macOS 客户端

1. 备份原有文件：
   ```bash
   sudo cp /Library/Application\ Support/ZeroTier/One/planet /Library/Application\ Support/ZeroTier/One/planet.bak
   ```

2. 替换新文件：
   ```bash
   sudo cp planet.custom /Library/Application\ Support/ZeroTier/One/planet
   ```

3. 重启 ZeroTier 服务：
   ```bash
   sudo launchctl kickstart -k system/com.zerotier.one
   ```

## 添加多个 Private Root 服务器

多根服务器配置可以提高网络的高可用性。

### 步骤 1：部署第二个服务器

在第二个服务器上使用相同的 docker-compose.yml 配置部署 ZTNET。

### 步骤 2：获取第二个服务器的身份

在第二个服务器上运行：
```bash
docker exec ztnet-zerotier zerotier-cli getpublicidentity
```

复制输出的 identity 内容。

### 步骤 3：在 ZTNET 中添加第二个根

1. 在第一个服务器的 ZTNET Web UI 中
2. 进入 `Admin → ZT Controller`
3. 点击 `Generate Planet`
4. 点击 `Add root server`
5. 填入第二个服务器的 identity 和 IP 地址
6. 点击 `CREATE PLANET`

### 步骤 4：更新所有客户端

1. 下载新的 `planet.custom`
2. 更新所有客户端设备
3. 重启 ZeroTier 服务

## 网络管理功能

### 创建网络

1. 在 ZTNET 主界面点击 `Create New Network`
2. 设置网络名称（可选）
3. 选择网络配置模板
4. 创建网络

### 管理成员

1. 进入特定网络的管理页面
2. 查看所有已加入的设备
3. 可以：
   - 授权/拒绝设备加入
   - 分配固定 IP 地址
   - 设置成员标签
   - 查看设备状态

### 配置网络规则

1. 进入 `Network Settings`
2. 配置：
   - 路由规则
   - 防火墙规则
   - DNS 设置
   - MTU 设置

## 高级配置

## 运维指南

### 升级部署

```bash
# 停止服务
docker-compose down

# 备份数据（可选但推荐）
docker run --rm -v ztnet-deploy_postgres-data:/data -v $(pwd):/backup alpine tar -czf /backup/postgres_backup_$(date +%Y%m%d).tar.gz -C /data .
docker run --rm -v ztnet-deploy_zerotier-data:/data -v $(pwd):/backup alpine tar -czf /backup/zerotier_backup_$(date +%Y%m%d).tar.gz -C /data .

# 拉取新镜像
docker-compose pull

# 重新启动
docker-compose up -d

# 查看升级日志
docker-compose logs -f ztnet
```

### 容器间通信

ZTNET 需要与 PostgreSQL 和 ZeroTier 容器通信。确保：

1. **网络配置正确**：
   ```yaml
   networks:
     ztnet-network:
       driver: bridge
   ```

2. **服务发现**：
   - ZTNET 通过容器名 `zerotier` 访问 ZeroTier
   - ZeroTier 监听 `0.0.0.0` 所有接口

3. **连接测试**：
   ```bash
   # 测试容器间通信
   docker exec ztnet-web curl http://zerotier:9993
   ```

### 故障排查

1. **网络连接问题**：
   ```bash
   # 检查网络
   docker network ls
   docker network inspect ztnet-deploy_ztnet-network

   # 测试容器间连接
   docker exec ztnet-web ping zerotier
   ```

2. **端口冲突**：
   ```bash
   # 检查端口占用
   ss -tulnp | grep 9993
   ss -tulnp | grep 3000
   ```

3. **服务启动问题**：
   ```bash
   # 查看各服务日志
   docker-compose logs postgres
   docker-compose logs zerotier
   docker-compose logs ztnet
   ```

## 1. 反向代理配置（可选）

如果需要使用域名访问 ZTNET，可以配置 Nginx：

```nginx
server {
    listen 80;
    server_name ztnet.yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### 2. HTTPS 配置

使用 Let's Encrypt 免费证书：

```bash
# 安装 certbot
sudo apt install certbot python3-certbot-nginx

# 获取证书
sudo certbot --nginx -d ztnet.yourdomain.com
```

### 3. 数据备份

创建备份脚本 `backup.sh`：
```bash
#!/bin/bash
BACKUP_DIR="/opt/backups/ztnet"
DATE=$(date +%Y%m%d_%H%M%S)

# 创建备份目录
mkdir -p $BACKUP_DIR

# 备份 PostgreSQL 数据库
docker exec ztnet-postgres pg_dump -U postgres ztnet > $BACKUP_DIR/ztnet_db_$DATE.sql

# 备份 ZeroTier 配置
docker exec ztnet-zerotier tar -czf - /var/lib/zerotier-one > $BACKUP_DIR/zerotier_data_$DATE.tar.gz

# 保留最近7天的备份
find $BACKUP_DIR -name "*.sql" -mtime +7 -delete
find $BACKUP_DIR -name "*.tar.gz" -mtime +7 -delete

echo "Backup completed: $DATE"
```

设置定时任务：
```bash
echo "0 2 * * * /opt/scripts/backup.sh" | sudo tee /etc/cron.d/ztnet-backup
```

## 监控与维护

### 1. 服务监控

创建监控脚本 `monitor.sh`：
```bash
#!/bin/bash
LOG_FILE="/var/log/ztnet-monitor.log"

log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a $LOG_FILE
}

check_services() {
    # 检查 PostgreSQL 容器
    if ! docker ps | grep -q ztnet-postgres; then
        log "错误: PostgreSQL 容器未运行"
        docker restart ztnet-postgres
    fi

    # 检查 ZeroTier 容器
    if ! docker ps | grep -q ztnet-zerotier; then
        log "错误: ZeroTier 容器未运行"
        docker restart ztnet-zerotier
    fi

    # 检查 ZTNET 容器
    if ! docker ps | grep -q ztnet-web; then
        log "错误: ZTNET Web 容器未运行"
        docker restart ztnet-web
    fi

    # 检查端口
    if ! ss -tulnp | grep -q ":9993 "; then
        log "警告: 端口 9993 未监听"
    fi

    if ! ss -tulnp | grep -q ":3000 "; then
        log "警告: 端口 3000 未监听"
    fi
}

check_ztnet() {
    # 检查 ZTNET API
    if curl -s http://localhost:3000/api/health >/dev/null; then
        log "ZTNET 服务正常"
    else
        log "警告: ZTNET API 无响应"
    fi
}

log "开始 ZTNET 服务健康检查..."
check_services
check_ztnet
log "检查完成"
```

### 2. 性能监控

查看资源使用情况：
```bash
# 查看 CPU 和内存使用
docker stats ztnet-postgres ztnet-zerotier ztnet-web

# 查看磁盘使用
du -sh ~/ztnet-deploy/

# 查看 ZeroTier 统计
docker exec ztnet-zerotier zerotier-cli networkstats
```

## 故障排除

### 1. 容器启动失败

```bash
# 查看详细错误
docker-compose logs postgres
docker-compose logs zerotier
docker-compose logs ztnet

# 检查容器状态
docker ps -a

# 重新启动容器
docker-compose down
docker-compose up -d
```

### 2. 端口冲突

```bash
# 检查端口占用
sudo ss -tulnp | grep 9993
sudo ss -tulnp | grep 3000

# 修改端口（在 docker-compose.yml 中）
# 将 3000:3000 改为其他端口，如 8888:3000
```

### 3. 网络连接问题

```bash
# 测试 ZeroTier 连接
docker exec ztnet-zerotier zerotier-cli listnetworks
docker exec ztnet-zerotier zerotier-cli listpeers

# 测试端口连通性
telnet your_server_ip 9993
```

### 4. 重置 ZTNET

如果需要重置 ZTNET：

```bash
# 停止并删除容器
docker-compose down

# 删除数据目录
sudo rm -rf ~/ztnet-deploy/

# 重新部署
docker-compose up -d
```

## 移动设备支持

### Android 解决方案

1. **ZeroTierFix**（推荐）
   - 下载地址：https://github.com/kaaass/ZerotierFix
   - 支持自定义 planet 配置
   - 无需 root 设备

2. **Zerotier-Magisk**（需 root）
   - 适用于已 root 的设备
   - 替换 `/data/adb/zerotier/home/planet` 文件

### iOS 解决方案

1. **ZeroTieriOSFix**
   - 需要自签名应用
   - 使用 Apple Developer 账号
   - 证书需要定期更新

### 注意事项

- 第三方修改可能存在安全风险
- 请仔细检查应用来源
- 官方移动应用暂不支持自定义 planet

## 安全建议

### 1. 基础安全

```bash
# 防火墙配置
sudo ufw default deny incoming
sudo ufw allow ssh
sudo ufw allow 9993/tcp
sudo ufw allow 9993/udp
sudo ufw enable

# 禁用密码登录（使用 SSH 密钥）
sudo nano /etc/ssh/sshd_config
# 找到 PasswordAuthentication no
sudo systemctl restart sshd
```

### 2. ZTNET 安全配置

1. 设置强密码
2. 启用两步验证（如果可用）
3. 定期更新 ZTNET
4. 限制访问 IP（通过防火墙）

### 3. 网络安全

1. 启用 ZeroTier 的加密
2. 配置细粒度的访问控制
3. 定期轮换密钥
4. 监控异常连接

## 性能优化

### 1. 系统优化

```bash
# 优化网络参数
echo 'net.core.rmem_max = 134217728
net.core.wmem_max = 134217728
net.ipv4.udp_rmem_min = 8192
net.ipv4.udp_wmem_min = 8192' | sudo tee /etc/sysctl.d/99-network.conf

sysctl -p
```

### 2. 容器优化

```bash
# 限制容器资源使用
# 在 docker-compose.yml 中添加：
deploy:
  resources:
    limits:
      cpus: '1.0'
      memory: 1G
    reservations:
      cpus: '0.5'
      memory: 512M
```

### 3. ZeroTier 优化

在 `/var/lib/zerotier-one/local.conf` 中添加：
```json
{
  "settings": {
    "portMappingEnabled": true,
    "softwareUpdate": "disable",
    "allowTCPFallback": true,
    "allowManagementFrom": ["127.0.0.1"],
    "allowGlobal": false
  }
}
```

## 总结

使用 ZTNET 管理的 ZeroTier Private Root Server 提供了：

- ✅ **完全自托管**：不依赖 ZeroTier Central
- ✅ **Web UI 管理**：可视化操作界面
- ✅ **多根服务器**：高可用性配置
- ✅ **自动备份**：数据安全保护
- ✅ **多用户支持**：团队协作管理

相比传统的 Moon 服务器，ZTNET 提供了更完整、更易用的网络管理解决方案。对于需要在 2026 年及以后使用 ZeroTier 的用户，这是目前推荐的部署方式。

**官方资源：**
- [ZTNET GitHub](https://github.com/sinamics/ztnet)
- [ZTNET 官方文档](https://ztnet.network)
- [ZeroTier 官方文档](https://docs.zerotier.com)