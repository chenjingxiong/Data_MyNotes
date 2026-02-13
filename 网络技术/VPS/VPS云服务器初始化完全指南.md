# VPS 云服务器初始化完全指南

> 本指南涵盖VPS服务器从购买到生产环境可用的完整初始化流程，包括1Panel安装、SSL证书申请、安全配置和性能优化。

---

## 📋 目录

- [一、系统基础配置](#一系统基础配置)
- [二、1Panel 面板安装](#二1panel-面板安装)
- [三、SSL 证书申请](#三ssl-证书申请)
- [四、安全加固配置](#四安全加固配置)
- [五、性能优化](#五性能优化)
- [六、一键初始化脚本](#六一键初始化脚本)
- [七、常见问题排查](#七常见问题排查)

---

## 一、系统基础配置

### 1.1 更新系统

```bash
# Debian/Ubuntu
apt update && apt upgrade -y

# CentOS/Rocky/AlmaLinux
yum update -y
# 或
dnf update -y
```

### 1.2 设置时区

```bash
# 设置为上海时区
timedatectl set-timezone Asia/Shanghai

# 验证
timedatectl status
```

### 1.3 创建普通用户（推荐）

```bash
# 创建用户
adduser admin

# 添加到 sudo 组
usermod -aG sudo admin

# 设置 sudo 免密（可选）
echo "admin ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/admin
chmod 440 /etc/sudoers.d/admin
```

### 1.4 配置 SSH 密钥登录

```bash
# 在本地生成密钥（如果没有）
ssh-keygen -t ed25519 -C "your_email@example.com"

# 复制公钥到服务器
ssh-copy-id -i ~/.ssh/id_ed25519.pub root@your_server_ip

# 或手动添加
mkdir -p ~/.ssh
echo "你的公钥内容" >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

### 1.5 优化 SSH 配置

编辑 `/etc/ssh/sshd_config`：

```bash
cat > /etc/ssh/sshd_config.d/security.conf << 'EOF'
# 禁用 root 登录
PermitRootLogin no

# 禁用密码认证，仅允许密钥
PasswordAuthentication no
PubkeyAuthentication yes

# 禁用空密码
PermitEmptyPasswords no

# 更改默认端口（可选）
# Port 22222

# 限制登录尝试
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2

# 禁用反向解析
UseDNS no

# 仅允许特定用户
# AllowUsers admin
EOF

# 重启 SSH 服务
systemctl restart sshd
```

---

## 二、1Panel 面板安装

### 2.1 官方一键安装脚本

```bash
curl -sSL https://resource.fit2cloud.com/1panel/package/quick_start.sh -o quick_start.sh && \
sh quick_start.sh
```

### 2.2 安装后配置

```bash
# 查看面板信息
1pctl user info
1pctl reset  # 重置端口/用户/密码

# 防火墙放行面板端口（默认随机端口）
# 需要在云服务商安全组也放行此端口
```

### 2.3 Docker 安装（1Panel 依赖）

如果系统未安装 Docker，1Panel 会自动安装，也可以手动安装：

```bash
# 使用阿里云镜像（国内推荐）
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

# 配置 Docker 镜像加速
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",
    "https://hub-mirror.c.163.com",
    "https://mirror.ccs.tencentyun.com"
  ],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
EOF

# 启动 Docker
sudo systemctl daemon-reload
sudo systemctl enable docker
sudo systemctl start docker
```

---

## 三、SSL 证书申请

### 3.1 使用 acme.sh 申请证书

```bash
# 安装 acme.sh
curl https://get.acme.sh | sh -s email=your_email@example.com

# 重新加载配置
source ~/.bashrc

# 或手动创建 alias（如果没有自动添加）
alias acme.sh=~/.acme.sh/acme.sh
```

### 3.2 申请证书（HTTP 验证）

```bash
# 需要先在 1Panel 中配置网站，确保域名解析到服务器

# 方式1：使用 1Panel 内置证书申请（推荐）
# 登录 1Panel -> 网站 -> 创建网站 -> SSL -> Let's Encrypt

# 方式2：使用 acme.sh 手动申请
acme.sh --issue -d yourdomain.com -d www.yourdomain.com --webroot /www/wwwroot/yourdomain.com
```

### 3.3 申请证书（DNS API 验证）

```bash
# Cloudflare
export CF_Token="your_api_token"
acme.sh --issue -d yourdomain.com -d *.yourdomain.com --dns dns_cf

# 阿里云
export Ali_Key="your_access_key"
export Ali_Secret="your_access_secret"
acme.sh --issue -d yourdomain.com -d *.yourdomain.com --dns dns_ali

# 腾讯云
export DP_Id="your_id"
export DP_Key="your_key"
acme.sh --issue -d yourdomain.com -d *.yourdomain.com --dns dns_dp
```

### 3.4 安装证书到指定目录

```bash
acme.sh --install-cert -d yourdomain.com \
  --key-file /path/to/keyfile/in/nginx/key.pem \
  --fullchain-file /path/to/fullchain/nginx/cert.pem \
  --reloadcmd "service nginx force-reload"
```

### 3.5 设置证书自动续期

```bash
# acme.sh 默认自动续期，查看定时任务
crontab -l | grep acme

# 手动续期
acme.sh --renew -d yourdomain.com --force
```

---

## 四、安全加固配置

### 4.1 配置防火墙（UFW）

```bash
# 安装 UFW
apt install ufw -y

# 默认策略
ufw default deny incoming
ufw default allow outgoing

# 允许 SSH（修改后的端口）
ufw allow 22222/tcp comment 'SSH'

# 允许 HTTP/HTTPS
ufw allow 80/tcp comment 'HTTP'
ufw allow 443/tcp comment 'HTTPS'

# 允许 1Panel 面板端口（替换为实际端口）
ufw allow 10086/tcp comment '1Panel'

# 启用防火墙
ufw enable

# 查看状态
ufw status numbered
```

### 4.2 配置防火墙（firewalld）

```bash
# 安装 firewalld（CentOS/Rocky）
yum install firewalld -y
systemctl enable firewalld
systemctl start firewalld

# 添加服务
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --permanent --add-port=22222/tcp  # SSH端口
firewall-cmd --permanent --add-port=10086/tcp  # 1Panel端口

# 重载配置
firewall-cmd --reload

# 查看规则
firewall-cmd --list-all
```

### 4.3 安装配置 Fail2ban

```bash
# 安装
apt install fail2ban -y

# 创建本地配置
cat > /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
# 封禁时间（秒）
bantime = 3600

# 检测时间窗口（秒）
findtime = 600

# 失败次数
maxretry = 5

[sshd]
enabled = true
port = 22222
logpath = /var/log/auth.log
maxretry = 3

[nginx-http-auth]
enabled = true
filter = nginx-http-auth
port = http,https
logpath = /var/log/nginx/error.log

[nginx-limit-req]
enabled = true
filter = nginx-limit-req
port = http,https
logpath = /var/log/nginx/error.log
maxretry = 10
EOF

# 启动服务
systemctl enable fail2ban
systemctl start fail2ban

# 查看状态
fail2ban-client status sshd
```

### 4.4 禁用不必要的服务

```bash
# 禁用 IPv6（如果不需要）
cat >> /etc/sysctl.conf << EOF
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
EOF
sysctl -p

# 禁用不用的服务
systemctl disable postfix  # 如果不需要邮件服务
```

### 4.5 配置系统日志轮转

```bash
cat > /etc/logrotate.d/custom << 'EOF'
/var/log/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 0640 root root
}
EOF
```

---

## 五、性能优化

### 5.1 系统内核参数优化

```bash
cat > /etc/sysctl.d/99-custom.conf << 'EOF'
# 网络性能优化
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 65536 16777216
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.tcp_window_scaling = 1

# TCP 快速打开
net.ipv4.tcp_fastopen = 3

# SYN 队列
net.ipv4.tcp_max_syn_backlog = 8192
net.core.somaxconn = 1024

# TIME_WAIT 优化
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_max_tw_buckets = 5000

# 防止 SYN 洪水攻击
net.ipv4.tcp_syncookies = 1

# 文件描述符限制
fs.file-max = 2097152

# 交换分区使用（适当使用 swap）
vm.swappiness = 10
vm.vfs_cache_pressure = 50
EOF

# 应用配置
sysctl -p /etc/sysctl.d/99-custom.conf
```

### 5.2 文件描述符限制

```bash
cat >> /etc/security/limits.conf << 'EOF'
* soft nofile 65535
* hard nofile 65535
* soft nproc 65535
* hard nproc 65535
EOF

# 重新登录生效
```

### 5.3 MySQL 优化（如果使用）

```bash
# 在 1Panel 中安装 MySQL 后，编辑配置
cat > /etc/mysql/conf.d/custom.cnf << 'EOF'
[mysqld]
# 连接数
max_connections = 500

# 缓冲池大小（根据内存调整，建议为物理内存的 50-70%）
innodb_buffer_pool_size = 1G

# 日志文件大小
innodb_log_file_size = 256M

# 刷新策略
innodb_flush_log_at_trx_commit = 2

# 查询缓存（MySQL 5.7 及以下）
query_cache_size = 128M
query_cache_type = 1

# 慢查询日志
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 2
EOF
```

### 5.4 Nginx 优化

在 1Panel 的 Nginx 配置中添加：

```nginx
# /etc/nginx/nginx.conf
user www-data;
worker_processes auto;
worker_rlimit_nofile 65535;

events {
    use epoll;
    worker_connections 10240;
    multi_accept on;
}

http {
    # 基础配置
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # 日志格式
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    # 性能优化
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 60;
    keepalive_requests 1000;

    # Gzip 压缩
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_comp_level 6;
    gzip_types text/plain text/css text/xml text/javascript
               application/json application/javascript application/xml+rss
               application/rss+xml font/truetype font/opentype
               application/vnd.ms-fontobject image/svg+xml;

    # 缓冲区设置
    client_body_buffer_size 128k;
    client_max_body_size 50m;
    client_header_buffer_size 1k;
    large_client_header_buffers 4 4k;

    # 超时设置
    client_body_timeout 60;
    client_header_timeout 60;
    send_timeout 60;

    # 包含站点配置
    include /etc/nginx/conf.d/*.conf;
}
```

### 5.5 PHP 优化（如果使用）

```bash
# /etc/php/8.x/fpm/pool.d/www.conf
pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35
pm.max_requests = 500

# 内存限制
php_admin_value[memory_limit] = 256M

# 执行时间
php_admin_value[max_execution_time] = 300
```

### 5.6 定时清理任务

```bash
# 添加清理脚本
cat > /root/cleanup.sh << 'EOF'
#!/bin/bash
# 清理 Docker 未使用的镜像
docker system prune -a -f --volumes

# 清理日志
find /var/log -name "*.log" -type f -size +100M -exec truncate -s 0 {} \;

# 清理 apt 缓存
apt-get clean
apt-get autoclean
EOF

chmod +x /root/cleanup.sh

# 添加到 crontab（每周日凌晨 3 点执行）
(crontab -l 2>/dev/null; echo "0 3 * * 0 /root/cleanup.sh >> /var/log/cleanup.log 2>&1") | crontab -
```

---

## 六、一键初始化脚本

以下是整合所有步骤的自动化脚本，**使用前请先阅读注意事项**！

### ⚠️ 使用前必读

1. **修改脚本中的配置项**：SSH端口、用户名、域名等
2. **确保云服务商安全组已放行相应端口**
3. **首次运行建议先在测试环境验证**
4. **脚本会修改SSH配置，请确保已配置密钥登录**

### 完整脚本

```bash
#!/bin/bash

################################################################################
# VPS 云服务器一键初始化脚本
# 功能：系统更新、安全加固、1Panel安装、证书配置、性能优化
#
# 使用方法：
# 1. 修改下方配置区域
# 2. chmod +x vps_init.sh
# 3. ./vps_init.sh
################################################################################

set -e  # 遇到错误立即退出

################################################################################
# 配置区域 - 请根据实际情况修改
################################################################################

# SSH 配置
SSH_PORT="22222"                          # SSH 端口
ADMIN_USER="admin"                        # 普通用户名
SSH_KEY="ssh-ed25519 AAAA..."             # SSH 公钥

# 域名配置（用于证书申请）
DOMAIN="example.com"                      # 主域名
EMAIL="admin@example.com"                 # Let's Encrypt 邮箱

# 1Panel 配置
PANEL_PORT="10086"                        # 1Panel 端口

# DNS API 配置（用于通配符证书，可选）
# CLOUDFLARE_TOKEN="your_token"
# ALI_KEY="your_key"
# ALI_SECRET="your_secret"

################################################################################
# 颜色输出函数
################################################################################

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

info() {
    echo -e "${GREEN}[INFO]${NC} $1"
}

warn() {
    echo -e "${YELLOW}[WARN]${NC} $1"
}

error() {
    echo -e "${RED}[ERROR]${NC} $1"
}

################################################################################
# 检测系统类型
################################################################################

detect_os() {
    if [ -f /etc/os-release ]; then
        . /etc/os-release
        OS=$ID
        OS_VERSION=$VERSION_ID
    else
        error "无法检测系统类型"
        exit 1
    fi

    info "检测到系统: $OS $OS_VERSION"
}

################################################################################
# 更新系统
################################################################################

update_system() {
    info "开始更新系统..."

    if [[ "$OS" == "ubuntu" || "$OS" == "debian" ]]; then
        export DEBIAN_FRONTEND=noninteractive
        apt update && apt upgrade -y
        apt install -y curl wget git vim ufw fail2ban ca-certificates \
            gnupg lsb-release software-properties-common
    elif [[ "$OS" == "centos" || "$OS" == "rocky" || "$OS" == "almalinux" ]]; then
        yum update -y
        yum install -y curl wget git vim firewalld fail2ban
    else
        warn "未知的系统类型，跳过更新"
    fi

    info "系统更新完成"
}

################################################################################
# 设置时区
################################################################################

setup_timezone() {
    info "设置时区为 Asia/Shanghai..."
    timedatectl set-timezone Asia/Shanghai
    info "当前时区: $(timedatectl | grep 'Time zone' | awk '{print $3}')"
}

################################################################################
# 创建普通用户
################################################################################

create_user() {
    if id "$ADMIN_USER" &>/dev/null; then
        warn "用户 $ADMIN_USER 已存在，跳过创建"
        return
    fi

    info "创建用户 $ADMIN_USER..."
    adduser --disabled-password --gecos "" "$ADMIN_USER"
    usermod -aG sudo "$ADMIN_USER"

    # 配置 SSH 密钥
    mkdir -p /home/$ADMIN_USER/.ssh
    echo "$SSH_KEY" > /home/$ADMIN_USER/.ssh/authorized_keys
    chmod 700 /home/$ADMIN_USER/.ssh
    chmod 600 /home/$ADMIN_USER/.ssh/authorized_keys
    chown -R $ADMIN_USER:$ADMIN_USER /home/$ADMIN_USER/.ssh

    # 配置 sudo 免密
    echo "$ADMIN_USER ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/$ADMIN_USER
    chmod 440 /etc/sudoers.d/$ADMIN_USER

    info "用户 $ADMIN_USER 创建完成"
}

################################################################################
# 配置 SSH
################################################################################

setup_ssh() {
    info "配置 SSH 安全设置..."

    cat > /etc/ssh/sshd_config.d/security.conf << EOF
# 禁用 root 登录
PermitRootLogin no

# 禁用密码认证
PasswordAuthentication no
PubkeyAuthentication yes
PermitEmptyPasswords no

# 更改端口
Port $SSH_PORT

# 限制登录尝试
MaxAuthTries 3
ClientAliveInterval 300
ClientAliveCountMax 2

# 禁用反向解析
UseDNS no
EOF

    # 重启 SSH
    if [[ "$OS" == "ubuntu" || "$OS" == "debian" ]]; then
        systemctl restart sshd
    else
        systemctl restart sshd
    fi

    info "SSH 配置完成，端口已改为: $SSH_PORT"
    warn "请确保云服务商安全组已放行端口 $SSH_PORT"
}

################################################################################
# 安装 Docker
################################################################################

install_docker() {
    if command -v docker &>/dev/null; then
        warn "Docker 已安装，跳过"
        return
    fi

    info "安装 Docker..."

    # 使用阿里云镜像
    curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

    # 配置镜像加速和日志
    mkdir -p /etc/docker
    cat > /etc/docker/daemon.json << EOF
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",
    "https://hub-mirror.c.163.com",
    "https://mirror.ccs.tencentyun.com"
  ],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
EOF

    systemctl daemon-reload
    systemctl enable docker
    systemctl start docker

    info "Docker 安装完成"
}

################################################################################
# 安装 1Panel
################################################################################

install_1panel() {
    if command -v 1pctl &>/dev/null; then
        warn "1Panel 已安装，跳过"
        return
    fi

    info "安装 1Panel..."
    curl -sSL https://resource.fit2cloud.com/1panel/package/quick_start.sh -o quick_start.sh
    sh quick_start.sh

    info "1Panel 安装完成"
    info "使用以下命令查看面板信息:"
    info "  1pctl user info"
    warn "请确保云服务商安全组已放行 1Panel 端口"
}

################################################################################
# 配置防火墙
################################################################################

setup_firewall() {
    info "配置防火墙..."

    if [[ "$OS" == "ubuntu" || "$OS" == "debian" ]]; then
        # UFW
        ufw default deny incoming
        ufw default allow outgoing
        ufw allow $SSH_PORT/tcp comment "SSH"
        ufw allow 80/tcp comment "HTTP"
        ufw allow 443/tcp comment "HTTPS"
        ufw --force enable
        ufw status
    else
        # firewalld
        systemctl enable firewalld
        systemctl start firewalld
        firewall-cmd --permanent --add-service=http
        firewall-cmd --permanent --add-service=https
        firewall-cmd --permanent --add-port=$SSH_PORT/tcp
        firewall-cmd --reload
        firewall-cmd --list-all
    fi

    info "防火墙配置完成"
}

################################################################################
# 配置 Fail2ban
################################################################################

setup_fail2ban() {
    info "配置 Fail2ban..."

    cat > /etc/fail2ban/jail.local << EOF
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 5

[sshd]
enabled = true
port = $SSH_PORT
logpath = $( [ -f /var/log/auth.log ] && echo "/var/log/auth.log" || echo "/var/log/secure" )
maxretry = 3
EOF

    systemctl enable fail2ban
    systemctl restart fail2ban

    info "Fail2ban 配置完成"
}

################################################################################
# 系统优化
################################################################################

optimize_system() {
    info "优化系统参数..."

    # 内核参数优化
    cat > /etc/sysctl.d/99-custom.conf << EOF
# 网络性能优化
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 65536 16777216
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_max_syn_backlog = 8192
net.core.somaxconn = 1024
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_syncookies = 1
fs.file-max = 2097152
vm.swappiness = 10
vm.vfs_cache_pressure = 50
EOF

    sysctl -p /etc/sysctl.d/99-custom.conf

    # 文件描述符限制
    cat >> /etc/security/limits.conf << EOF
* soft nofile 65535
* hard nofile 65535
* soft nproc 65535
* hard nproc 65535
EOF

    # 禁用 IPv6（可选）
    # echo "net.ipv6.conf.all.disable_ipv6 = 1" >> /etc/sysctl.conf
    # echo "net.ipv6.conf.default.disable_ipv6 = 1" >> /etc/sysctl.conf
    # sysctl -p

    info "系统优化完成"
}

################################################################################
# 安装 acme.sh（证书工具）
################################################################################

install_acme() {
    if [ -f ~/.acme.sh/acme.sh ]; then
        warn "acme.sh 已安装，跳过"
        return
    fi

    info "安装 acme.sh..."
    curl https://get.acme.sh | sh -s email=$EMAIL

    # 重新加载环境
    source ~/.bashrc 2>/dev/null || true

    info "acme.sh 安装完成"
    info "证书申请需要在 1Panel 中配置网站后进行"
}

################################################################################
# 创建清理脚本
################################################################################

create_cleanup_script() {
    info "创建定时清理脚本..."

    cat > /root/cleanup.sh << 'EOF'
#!/bin/bash
echo "[$(date)] 开始清理..."

# 清理 Docker 未使用的镜像
docker system prune -a -f --volumes

# 清理大日志文件
find /var/log -name "*.log" -type f -size +100M -exec truncate -s 0 {} \;

# 清理包管理器缓存
if command -v apt-get &>/dev/null; then
    apt-get clean
    apt-get autoclean
elif command -v yum &>/dev/null; then
    yum clean all
fi

echo "[$(date)] 清理完成"
EOF

    chmod +x /root/cleanup.sh

    # 添加到 crontab
    (crontab -l 2>/dev/null | grep -v "cleanup.sh"; echo "0 3 * * 0 /root/cleanup.sh >> /var/log/cleanup.log 2>&1") | crontab -

    info "清理脚本已创建，每周日凌晨 3 点执行"
}

################################################################################
# 显示安装信息
################################################################################

show_info() {
    echo ""
    echo "=================================="
    echo "  初始化完成！"
    echo "=================================="
    echo ""
    echo "SSH 端口: $SSH_PORT"
    echo "SSH 用户: $ADMIN_USER"
    echo "1Panel 端口: $(1pctl user info 2>/dev/null | grep 'Entrance' | awk '{print $2}' || echo '请运行 1pctl user info 查看')"
    echo ""
    echo "重要提醒:"
    echo "1. 请确保云服务商安全组已放行以下端口:"
    echo "   - SSH: $SSH_PORT"
    echo "   - HTTP: 80"
    echo "   - HTTPS: 443"
    echo "   - 1Panel: 请运行 '1pctl user info' 查看"
    echo ""
    echo "2. 使用新用户登录: ssh $ADMIN_USER@your_server_ip -p $SSH_PORT"
    echo ""
    echo "3. 查看 1Panel 信息: 1pctl user info"
    echo "4. 重置 1Panel: 1pctl reset"
    echo ""
    echo "5. SSL 证书申请:"
    echo "   - 推荐: 登录 1Panel -> 网站 -> SSL -> Let's Encrypt"
    echo "   - 或使用: acme.sh --issue -d $DOMAIN -d www.$DOMAIN --webroot /www/wwwroot/$DOMAIN"
    echo ""
}

################################################################################
# 主函数
################################################################################

main() {
    echo "=================================="
    echo "  VPS 云服务器初始化脚本"
    echo "=================================="
    echo ""

    detect_os
    update_system
    setup_timezone
    create_user
    install_docker
    install_1panel
    setup_firewall
    setup_fail2ban
    setup_ssh
    optimize_system
    install_acme
    create_cleanup_script

    show_info

    echo "=================================="
    warn "SSH 配置已修改，端口改为 $SSH_PORT"
    warn "请保持当前连接，新开窗口测试新端口登录成功后再关闭"
    warn "如果无法登录，请通过云服务商 VNC 登录检查"
    echo "=================================="
}

# 运行主函数
main
```

### 脚本使用说明

1. **保存脚本**
```bash
nano vps_init.sh
# 粘贴上述脚本内容
# 修改配置区域的参数
```

2. **赋予执行权限**
```bash
chmod +x vps_init.sh
```

3. **运行脚本**
```bash
./vps_init.sh
```

4. **修改配置区域的关键参数**
   - `SSH_PORT`: SSH 端口（建议改为非标准端口）
   - `ADMIN_USER`: 普通用户名
   - `SSH_KEY`: 你的 SSH 公钥
   - `DOMAIN`: 主域名
   - `EMAIL`: 用于 Let's Encrypt 的邮箱

---

## 七、常见问题排查

### 7.1 SSH 无法连接

```bash
# 通过云服务商 VNC 登录

# 检查 SSH 状态
systemctl status sshd

# 检查配置
sshd -t

# 查看日志
tail -f /var/log/auth.log  # Debian/Ubuntu
tail -f /var/log/secure    # CentOS/Rocky
```

### 7.2 1Panel 无法访问

```bash
# 检查服务状态
systemctl status 1panel

# 查看端口
1pctl user info

# 检查防火墙
ufw status
firewall-cmd --list-all

# 查看日志
tail -f /opt/1panel/conf/log/panel.log
```

### 7.3 Docker 问题

```bash
# 检查 Docker 状态
systemctl status docker

# 查看日志
journalctl -u docker -n 50

# 重启 Docker
systemctl restart docker
```

### 7.4 证书申请失败

```bash
# 检查 acme.sh 日志
acme.sh --issue -d example.com --dry-run  # 测试模式

# 检查 80 端口是否被占用
netstat -tlnp | grep :80

# 确保 DNS 解析正确
ping example.com
```

### 7.5 性能问题

```bash
# 查看系统负载
top
htop

# 查看磁盘使用
df -h

# 查看内存使用
free -h

# 查看 Docker 资源使用
docker stats
```

---

## 📚 相关资源

- [1Panel 官方文档](https://1panel.cn/docs/)
- [acme.sh 文档](https://github.com/acmesh-official/acme.sh/wiki)
- [Let's Encrypt](https://letsencrypt.org/)
- [Docker 官方文档](https://docs.docker.com/)

---

## 📝 更新日志

| 日期 | 版本 | 说明 |
|------|------|------|
| 2026-02-09 | v1.0 | 初始版本，包含完整的初始化流程 |

---

> 💡 **提示**: 本指南适用于 Debian、Ubuntu、CentOS、Rocky Linux、AlmaLinux 等主流 Linux 发行版。如有问题，欢迎反馈。
