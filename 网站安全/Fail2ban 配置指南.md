# Fail2ban 配置指南

> Fail2ban 是一款入侵防御系统，通过监控系统日志自动封禁恶意 IP 地址。它可以有效防范暴力破解、DDoS 攻击、扫描探测等威胁，是服务器安全的第一道防线。

## 📋 目录

- [功能特点](#功能特点)
- [安装部署](#安装部署)
- [基础配置](#基础配置)
- [SSH 监控配置](#ssh-监控配置)
- [网站监控配置](#网站监控配置)
- [高级配置](#高级配置)
- [管理与维护](#管理与维护)
- [常见问题](#常见问题)

---

## ✨ 功能特点

- 🔐 **SSH 防护**：自动封禁暴力破解 SSH 的 IP
- 🌐 **Web 防护**：保护 Nginx/Apache 等网站服务器
- 🚫 **防爬虫**：封禁恶意爬虫和扫描器
- 📊 **日志监控**：实时分析系统日志
- ⏱️ **自动解封**：支持设定封禁时间，自动解封
- 🎯 **灵活规则**：支持自定义过滤规则和正则表达式

---

## 🚀 安装部署

### 方式一：使用包管理器安装

#### Debian / Ubuntu

```bash
# 更新软件包列表
sudo apt update

# 安装 Fail2ban
sudo apt install fail2ban -y
```

#### CentOS / Rocky Linux / AlmaLinux

```bash
# 安装 EPEL 源
sudo yum install epel-release -y

# 安装 Fail2ban
sudo yum install fail2ban -y
```

#### Arch Linux / Manjaro

```bash
sudo pacman -S fail2ban
```

### 方式二：Docker 部署

```bash
# 拉取镜像
docker pull crazymax/fail2ban:latest

# 运行容器
docker run -d \
  --name fail2ban \
  --network host \
  --cap-add NET_ADMIN \
  --cap-add NET_RAW \
  -v /var/log:/var/log:ro \
  -v /path/to/config:/data \
  -v /var/run/fail2ban:/var/run/fail2ban \
  --restart unless-stopped \
  crazymax/fail2ban:latest
```

### 安装后验证

```bash
# 检查服务状态
sudo systemctl status fail2ban

# 查看版本
fail2ban-client version

# 查看当前状态
sudo fail2ban-client status
```

---

## ⚙️ 基础配置

### 配置文件结构

```
/etc/fail2ban/
├── fail2ban.conf         # 主配置文件
├── jail.conf             # 监控规则配置（参考文件）
├── jail.d/               # 自定义监控规则目录
├── filter.d/             # 过滤规则目录
└── action.d/             # 动作脚本目录
```

### 创建本地配置

**⚠️ 重要提示**：不要直接修改 `jail.conf`，应在 `jail.d/` 目录创建自定义配置。

#### 1. 创建主配置文件

创建 `/etc/fail2ban/jail.d/local.conf`：

```ini
[DEFAULT]
# 封禁时间（秒），1天 = 86400
bantime = 86400

# 检测时间窗口（秒）
findtime = 600

# 最大失败次数
maxretry = 5

# 忽略 IP（空格分隔，支持 CIDR）
ignoreip = 127.0.0.1/8 ::1 192.168.1.0/24

# 发送通知邮件
# destemail = your-email@example.com
# sendername = Fail2ban
# action = %(action_mwl)s

# 白名单文件路径（可选）
# ignorecommand = /path/to/whitelist.sh
```

---

## 🔐 SSH 监控配置

### 启用 SSH 防护

创建 `/etc/fail2ban/jail.d/sshd.conf`：

```ini
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
# 或 CentOS/RHEL: /var/log/secure
maxretry = 3
bantime = 86400
findtime = 600
```

### SSH 高级防护配置

```ini
[sshd]
enabled = true
port = ssh,22222           # 如果使用非标准端口
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 86400
findtime = 600

# 附加动作
action = iptables-allports[name=SSH]

# 封禁所有端口，不只是 SSH
# action = iptables-allports[name=SSH]
```

### 检查 SSH 监控状态

```bash
# 查看 SSH jail 状态
sudo fail2ban-client status sshd

# 查看被封禁的 IP
sudo fail2ban-client status sshd | grep "Banned IP"

# 手动解封 IP
sudo fail2ban-client set sshd unbanip 1.2.3.4

# 手动封禁 IP
sudo fail2ban-client set sshd banip 1.2.3.4
```

---

## 🌐 网站监控配置

### 1Panel + OpenResty 防护配置

> **1Panel 环境特殊说明**：
> - 1Panel 安装的 OpenResty 日志路径通常在 `/opt/1panel/apps/openresty/openresty/logs/` 或 `/var/log/openresty/`
> - 需要先确认实际的日志路径再配置

#### 1. 确认 OpenResty 日志路径

```bash
# 查找 OpenResty 日志目录
sudo find /opt/1panel -name "*.log" -type f 2>/dev/null | grep -E "(access|error)" | head -10

# 常见的 1Panel OpenResty 日志路径
ls -la /opt/1panel/apps/openresty/openresty/logs/
# 或
ls -la /var/log/openresty/
# 或
ls -la /usr/local/openresty/nginx/logs/
```

#### 2. 创建 1Panel OpenResty 专用配置

创建 `/etc/fail2ban/jail.d/1panel-openresty.conf`：

```ini
# ============================================
# 1Panel OpenResty 防护配置
# ============================================

# ========== 基础认证失败防护 ==========
[openresty-http-auth]
enabled = true
filter = nginx-http-auth
port = http,https
logpath = /opt/1panel/apps/openresty/openresty/logs/error.log
          /var/log/openresty/error.log
          /usr/local/openresty/nginx/logs/error.log
maxretry = 5
findtime = 600
bantime = 86400
action = iptables-multiport[name=OpenResty-Auth, port="http,https"]

# ========== 恶意扫描防护 ==========
[openresty-noscript]
enabled = true
filter = nginx-noscript
port = http,https
logpath = /opt/1panel/apps/openresty/openresty/logs/access.log
          /var/log/openresty/access.log
          /usr/local/openresty/nginx/logs/access.log
maxretry = 3
findtime = 60
bantime = 86400
action = iptables-multiport[name=OpenResty-NoScript, port="http,https"]

# ========== Bad Bot 爬虫防护 ==========
[openresty-badbots]
enabled = true
filter = nginx-badbots
port = http,https
logpath = /opt/1panel/apps/openresty/openresty/logs/access.log
          /var/log/openresty/access.log
          /usr/local/openresty/nginx/logs/access.log
maxretry = 2
findtime = 600
bantime = 86400
action = iptables-multiport[name=OpenResty-BadBots, port="http,https"]

# ========== 恶意 User-Agent 防护 ==========
[openresty-baduseragent]
enabled = true
port = http,https
filter = openresty-baduseragent
logpath = /opt/1panel/apps/openresty/openresty/logs/access.log
          /var/log/openresty/access.log
          /usr/local/openresty/nginx/logs/access.log
maxretry = 1
findtime = 600
bantime = 86400
action = iptables-multiport[name=OpenResty-BadUA, port="http,https"]

# ========== SQL 注入防护 ==========
[openresty-sql-injection]
enabled = true
port = http,https
filter = openresty-sql-injection
logpath = /opt/1panel/apps/openresty/openresty/logs/access.log
          /var/log/openresty/access.log
          /usr/local/openresty/nginx/logs/access.log
maxretry = 3
findtime = 600
bantime = 86400
action = iptables-multiport[name=OpenResty-SQL, port="http,https"]

# ========== 目录遍历攻击防护 ==========
[openresty-dir-traversal]
enabled = true
port = http,https
filter = openresty-dir-traversal
logpath = /opt/1panel/apps/openresty/openresty/logs/access.log
          /var/log/openresty/access.log
          /usr/local/openresty/nginx/logs/access.log
maxretry = 2
findtime = 600
bantime = 86400
action = iptables-multiport[name=OpenResty-DirTraversal, port="http,https"]

# ========== 限制请求速率防护 ==========
[openresty-limit-req]
enabled = true
filter = nginx-limit-req
port = http,https
logpath = /opt/1panel/apps/openresty/openresty/logs/error.log
          /var/log/openresty/error.log
          /usr/local/openresty/nginx/logs/error.log
maxretry = 10
findtime = 60
bantime = 3600
action = iptables-multiport[name=OpenResty-Limit, port="http,https"]

# ========== 403/404 异常访问防护 ==========
[openresty-forbidden]
enabled = true
filter = nginx-forbidden
port = http,https
logpath = /opt/1panel/apps/openresty/openresty/logs/access.log
          /var/log/openresty/access.log
          /usr/local/openresty/nginx/logs/access.log
maxretry = 10
findtime = 600
bantime = 3600
action = iptables-multiport[name=OpenResty-Forbidden, port="http,https"]

# ========== WordPress 登录防护 ==========
[openresty-wordpress]
enabled = true
filter = wordpress
port = http,https
logpath = /opt/1panel/apps/openresty/openresty/logs/access.log
          /var/log/openresty/access.log
          /usr/local/openresty/nginx/logs/access.log
maxretry = 5
findtime = 300
bantime = 86400
action = iptables-multiport[name=OpenResty-WP, port="http,https"]

# ========== 代理访问防护 ==========
[openresty-proxy]
enabled = true
port = http,https
filter = openresty-proxy
logpath = /opt/1panel/apps/openresty/openresty/logs/access.log
          /var/log/openresty/access.log
          /usr/local/openresty/nginx/logs/access.log
maxretry = 2
findtime = 600
bantime = 86400
action = iptables-multiport[name=OpenResty-Proxy, port="http,https"]
```

#### 3. 创建自定义过滤规则

创建 `/etc/fail2ban/filter.d/openresty-baduseragent.conf`：

```ini
[Definition]
failregex = ^<HOST> -.*"GET.*HTTP.*".*"(?:curl|wget|python-requests|python-urllib|libwww-perl|java|go-http-client|scanner|crawler|spider|bot|scraper)".*$
            ^<HOST> -.*".*(Nmap|nikto|wikto|sf|sqlmap|bsqlbf|w3af|acunetix|burpsuite|openvas|core-impact|netsparker|wpscan|snort|ids|webinspect|apachebench).*".*$
ignoreregex =
```

创建 `/etc/fail2ban/filter.d/openresty-sql-injection.conf`：

```ini
[Definition]
failregex = ^<HOST> -.*"(GET|POST).*(\?|%3F)(union|select|insert|delete|update|drop|create|alter|declare|cast|exec|execute|\bcast\b|\bexec\b).*HTTP.*"
            ^<HOST> -.*"(GET|POST).*(or|and).*(\=|%3D).*(\=|%3D).*HTTP.*"
            ^<HOST> -.*"(GET|POST).*(\%27|\%22|\;).*HTTP.*"
            ^<HOST> -.*"(GET|POST).*waitfor\%20delay.*HTTP.*"
            ^<HOST> -.*"(GET|POST).*\.\.(\/|%2F|%5C).*HTTP.*"
ignoreregex =
```

创建 `/etc/fail2ban/filter.d/openresty-dir-traversal.conf`：

```ini
[Definition]
failregex = ^<HOST> -.*"(GET|POST).*(\.\.\/|\.\.\\|%2e%2e%2f|%2e%2e%5c|%252e|%252e%252f|%c0%ae|%c1%9c).*HTTP.*"
            ^<HOST> -.*"(GET|POST).*\/etc\/passwd.*HTTP.*"
            ^<HOST> -.*"(GET|POST).*\/proc\/.*HTTP.*"
ignoreregex =
```

创建 `/etc/fail2ban/filter.d/openresty-proxy.conf`：

```ini
[Definition]
failregex = ^<HOST> -.*"GET.*(http|https)\:\/\/.*HTTP.*"
            ^<HOST> -.*"(CONNECT|PROXY|HEAD).*HTTP.*"
ignoreregex =
```

#### 4. 1Panel 环境配置验证脚本

创建 `/usr/local/bin/check-openresty-logs.sh`：

```bash
#!/bin/bash
# 1Panel OpenResty 日志路径检查脚本

echo "=== 检查 1Panel OpenResty 日志路径 ==="
echo ""

# 可能的日志路径
LOG_PATHS=(
    "/opt/1panel/apps/openresty/openresty/logs"
    "/var/log/openresty"
    "/usr/local/openresty/nginx/logs"
    "/var/log/nginx"
)

for path in "${LOG_PATHS[@]}"; do
    if [ -d "$path" ]; then
        echo "✓ 目录存在: $path"
        echo "  内容:"
        ls -lh "$path" 2>/dev/null | grep -E "\.log$|total"
        echo ""
    else
        echo "✗ 目录不存在: $path"
    fi
done

echo "=== 推荐的 Fail2ban logpath 配置 ==="
for path in "${LOG_PATHS[@]}"; do
    if [ -d "$path" ]; then
        echo "logpath = $path/access.log"
        echo "          $path/error.log"
    fi
done
```

```bash
# 运行检查脚本
sudo chmod +x /usr/local/bin/check-openresty-logs.sh
sudo /usr/local/bin/check-openresty-logs.sh
```

#### 5. 1Panel 防火墙配置

如果 1Panel 使用 firewalld：

```ini
# 在 /etc/fail2ban/jail.d/local.conf 中添加
[DEFAULT]
banaction = firewallcmd-ipset
```

如果使用 ufw：

```ini
[DEFAULT]
banaction = ufw
```

#### 6. 测试配置

```bash
# 1. 重启 Fail2ban
sudo systemctl restart fail2ban

# 2. 检查 jail 状态
sudo fail2ban-client status

# 3. 测试正则表达式
sudo fail2ban-regex /opt/1panel/apps/openresty/openresty/logs/access.log /etc/fail2ban/filter.d/openresty-sql-injection.conf

# 4. 查看特定 jail 状态
sudo fail2ban-client status openresty-sql-injection
```

---

### Nginx 防护配置

创建 `/etc/fail2ban/jail.d/nginx.conf`：

#### 1. Nginx 基础认证防护

```ini
[nginx-http-auth]
enabled = true
filter = nginx-http-auth
port = http,https
logpath = /var/log/nginx/error.log
maxretry = 5
bantime = 86400
```

#### 2. Nginx 限制请求防护（防 DDoS）

```ini
[nginx-limit-req]
enabled = true
filter = nginx-limit-req
port = http,https
logpath = /var/log/nginx/error.log
maxretry = 10
findtime = 60
bantime = 3600
```

#### 3. Nginx 禁止访问防护

```ini
[nginx-forbidden]
enabled = true
filter = nginx-forbidden
port = http,https
logpath = /var/log/nginx/access.log
maxretry = 5
findtime = 600
bantime = 86400
```

#### 4. Nginx 恶意扫描防护

```ini
[nginx-noscript]
enabled = true
filter = nginx-noscript
port = http,https
logpath = /var/log/nginx/access.log
maxretry = 3
findtime = 60
bantime = 86400
```

#### 5. Nginx 无主页访问防护

```ini
[nginx-nohome]
enabled = true
filter = nginx-nohome
port = http,https
logpath = /var/log/nginx/access.log
maxretry = 3
findtime = 60
bantime = 86400
```

#### 6. Nginx Bad Bot 爬虫防护

```ini
[nginx-badbots]
enabled = true
filter = nginx-badbots
port = http,https
logpath = /var/log/nginx/access.log
maxretry = 2
findtime = 600
bantime = 86400
```

### Apache 防护配置

创建 `/etc/fail2ban/jail.d/apache.conf`：

#### 1. Apache 基础防护

```ini
[apache]
enabled = true
port = http,https
filter = apache-auth
logpath = /var/log/apache2/error.log
maxretry = 5
bantime = 86400

[apache-overflows]
enabled = true
port = http,https
filter = apache-overflows
logpath = /var/log/apache2/error.log
maxretry = 3
bantime = 86400

[apache-badbots]
enabled = true
port = http,https
filter = apache-badbots
logpath = /var/log/apache2/access.log
maxretry = 2
bantime = 86400
```

### WordPress 防护配置

创建 `/etc/fail2ban/jail.d/wordpress.conf`：

```ini
[wordpress]
enabled = true
filter = wordpress
port = http,https
logpath = /var/log/nginx/access.log  # 或 Apache 日志路径
maxretry = 5
findtime = 300
bantime = 86400
```

#### 创建 WordPress 过滤规则

创建 `/etc/fail2ban/filter.d/wordpress.conf`：

```ini
[Definition]
failregex = ^<HOST> .* "POST .*wp-login.php.*" 200
            ^<HOST> .* "POST .*xmlrpc.php.*" 200
ignoreregex =
```

### 网站监控综合配置示例

创建 `/etc/fail2ban/jail.d/web-protection.conf`：

```ini
# 网站综合防护配置

# Nginx 认证失败
[nginx-http-auth]
enabled = true
port = http,https
logpath = /var/log/nginx/error.log
maxretry = 5
bantime = 86400

# 恶意扫描
[nginx-noscript]
enabled = true
port = http,https
logpath = /var/log/nginx/access.log
maxretry = 3
bantime = 86400

# Bad Bot 爬虫
[nginx-badbots]
enabled = true
port = http,https
logpath = /var/log/nginx/access.log
maxretry = 2
bantime = 86400

# 搜索型注入
[nginx-search]
enabled = true
port = http,https
logpath = /var/log/nginx/access.log
maxretry = 3
bantime = 86400

# SQL 注入尝试
[nginx-sql-injection]
enabled = true
port = http,https
logpath = /var/log/nginx/access.log
maxretry = 3
bantime = 86400
```

#### 创建自定义过滤规则

创建 `/etc/fail2ban/filter.d/nginx-sql-injection.conf`：

```ini
[Definition]
failregex = ^<HOST> -.*"(GET|POST).*(\?|%3F)(union|select|insert|delete|update|drop|create|alter).*HTTP.*"
            ^<HOST> -.*"(GET|POST).*(or|and).*(\=|%3D).*(\=|%3D).*HTTP.*"
ignoreregex =
```

---

## 🔧 高级配置

### 1. 自定义白名单脚本

创建 `/usr/local/bin/fail2ban-whitelist.sh`：

```bash
#!/bin/bash
# Fail2ban IP 白名单检查脚本

WHITELIST_FILE="/etc/fail2ban/whitelist.txt"

if [ -f "$WHITELIST_FILE" ]; then
    grep -q "$1" "$WHITELIST_FILE" && echo "1" || echo "0"
else
    echo "0"
fi
```

配置权限：
```bash
sudo chmod +x /usr/local/bin/fail2ban-whitelist.sh
```

在 `jail.local` 中使用：
```ini
ignorecommand = /usr/local/bin/fail2ban-whitelist.sh <ip>
```

### 2. 多次违规递增封禁

创建 `/etc/fail2ban/jail.d/recidive.conf`：

```ini
[recidive]
enabled = true
filter = recidive
logpath = /var/log/fail2ban.log
action = iptables-allports[name=recidive]
bantime = 2592000  # 30天
findtime = 86400   # 24小时内
maxretry = 3       # 3次违规
```

### 3. 封禁所有端口

修改 action 配置：
```ini
action = iptables-allports[name=SERVICE]
```

### 4. 邮件通知配置

```ini
[DEFAULT]
# 邮件设置
destemail = admin@example.com
sendername = Fail2ban-Alert
action = %(action_mwl)s

# 说明：
# %(action_)s     仅封禁，不通知
# %(action_mw)s   封禁 + 发邮件
# %(action_mwl)s  封禁 + 发邮件 + 记录日志
```

### 5. 使用防火墙后端

#### 使用 firewalld（CentOS/RHEL 7+）

```ini
[DEFAULT]
banaction = firewallcmd-ipset
```

#### 使用 ufw（Ubuntu）

```ini
[DEFAULT]
banaction = ufw
```

#### 使用 nftables

```ini
[DEFAULT]
banaction = nftables-allports
```

### 6. 集成 Cloudflare 封禁

创建 `/etc/fail2ban/action.d/cloudflare.conf`：

```ini
[Definition]
actionstart =
actionstop =
actioncheck =
actionban = curl -s -X POST "https://api.cloudflare.com/client/v4/user/firewall/access_rules/rules" \
    -H "X-Auth-Email: <cf_email>" \
    -H "X-Auth-Key: <cf_api_key>" \
    -H "Content-Type: application/json" \
    --data '{"mode":"block","configuration":{"target":"ip","value":"<ip>"},"notes":"Fail2ban ban"}'

actionunban = curl -s -X DELETE "https://api.cloudflare.com/client/v4/user/firewall/access_rules/rules/$(curl -s -X GET "https://api.cloudflare.com/client/v4/user/firewall/access_rules/rules?mode=block&configuration_value=<ip>&page=1&per_page=1" \
    -H "X-Auth-Email: <cf_email>" \
    -H "X-Auth-Key: <cf_api_key>" \
    -H "Content-Type: application/json" | jq -r '.result[0].id')"

[Init]
cf_email = your-email@example.com
cf_api_key = your_api_key_here
```

---

## 🛠️ 管理与维护

### 服务管理命令

```bash
# 启动服务
sudo systemctl start fail2ban

# 停止服务
sudo systemctl stop fail2ban

# 重启服务
sudo systemctl restart fail2ban

# 重新加载配置
sudo systemctl reload fail2ban

# 开机自启
sudo systemctl enable fail2ban

# 查看状态
sudo systemctl status fail2ban
```

### 查看 Jail 状态

```bash
# 查看所有启用的 jail
sudo fail2ban-client status

# 查看特定 jail 状态
sudo fail2ban-client status sshd
sudo fail2ban-client status nginx-http-auth

# 查看被封禁的 IP
sudo fail2ban-client get sshd banip

# 查看所有 jail 的被封 IP
for jail in $(sudo fail2ban-client status | grep "Jail list" | sed 's/.*: //;s/,//g'); do
    echo "=== $jail ==="
    sudo fail2ban-client status "$jail" | grep "Banned IP"
done
```

### 手动管理封禁

```bash
# 手动封禁 IP
sudo fail2ban-client set sshd banip 1.2.3.4

# 手动解封 IP
sudo fail2ban-client set sshd unbanip 1.2.3.4

# 解封指定 jail 的所有 IP
sudo fail2ban-client unban --all

# 设置封禁时间
sudo fail2ban-client set sshd bantime 3600
```

### 配置测试

```bash
# 测试正则表达式是否匹配日志
sudo fail2ban-regex /var/log/nginx/access.log /etc/fail2ban/filter.d/nginx-sql-injection.conf

# 测试 jail 配置（不实际封禁）
sudo fail2ban-client -d

# 检查配置文件语法
sudo fail2ban-client -t
```

### 查看日志

```bash
# 查看 Fail2ban 日志
sudo tail -f /var/log/fail2ban.log

# 查看最近的封禁记录
grep "Ban " /var/log/fail2ban.log | tail -20

# 统计被封禁最多的 IP
grep "Ban " /var/log/fail2ban.log | awk '{print $5}' | sort | uniq -c | sort -rn | head -10
```

### 配置持久化

```bash
# 确保封禁在重启后仍然生效（重启 iptables 时）
# 使用 ipset 或 firewalld 后端可以实现持久化
```

---

## ❓ 常见问题

### Q0: 1Panel + OpenResty 环境常见问题

#### Q0.1: 找不到 OpenResty 日志文件

**解决方法：**

```bash
# 方法1：查找实际日志路径
sudo find /opt/1panel -name "*.log" 2>/dev/null

# 方法2：检查 OpenResty 配置
sudo cat /opt/1panel/apps/openresty/openresty/conf/nginx.conf | grep error_log
sudo cat /opt/1panel/apps/openresty/openresty/conf/nginx.conf | grep access_log

# 方法3：运行检查脚本
sudo /usr/local/bin/check-openresty-logs.sh
```

找到正确路径后，更新 jail 配置中的 `logpath`。

#### Q0.2: Fail2ban 无法读取 1Panel 日志

**原因：** 通常是权限问题

**解决方法：**

```bash
# 1. 检查日志文件权限
ls -la /opt/1panel/apps/openresty/openresty/logs/

# 2. 将 Fail2ban 用户添加到日志组
sudo usermod -aG www-data fail2ban  # 或 nginx 组

# 3. 修改日志权限
sudo chmod 750 /opt/1panel/apps/openresty/openresty/logs/
sudo chmod 640 /opt/1panel/apps/openresty/openresty/logs/*.log

# 4. 或使用 ACL
sudo setfacl -R -m u:fail2ban:rx /opt/1panel/apps/openresty/openresty/logs/
```

#### Q0.3: 1Panel 容器内的 OpenResty 如何监控

**1Panel 容器日志监控配置：**

```ini
# /etc/fail2ban/jail.d/1panel-openresty.conf
[openresty-container]
enabled = true
filter = openresty-sql-injection
port = http,https
logpath = /var/lib/docker/containers/*openresty*/*-json.log
maxretry = 3
bantime = 86400
action = iptables-multiport[name=OpenResty, port="http,https"]
```

#### Q0.4: 1Panel 防火墙与 Fail2ban 冲突

**解决方法：**

1Panel 默认可能使用 firewalld 或 ufw，需要配置 Fail2ban 使用相同的后端：

```bash
# 检查当前防火墙
sudo firewall-cmd --state 2>/dev/null && echo "使用 firewalld"
sudo ufw status 2>/dev/null | head -1 && echo "使用 ufw"

# 配置 Fail2ban 使用对应后端
# /etc/fail2ban/jail.d/local.conf
[DEFAULT]
# firewalld 用户
banaction = firewallcmd-ipset

# ufw 用户
# banaction = ufw
```

#### Q0.5: 1Panel 升级后配置丢失

**预防措施：**

```bash
# 备份 Fail2ban 配置
sudo cp -r /etc/fail2ban /etc/fail2ban.backup.$(date +%Y%m%d)

# 添加到 1Panel 钩子（如果支持）
# 或创建恢复脚本
```

创建恢复脚本 `/usr/local/bin/restore-fail2ban.sh`：

```bash
#!/bin/bash
BACKUP_DIR="/etc/fail2ban.backup"
if [ -d "$BACKUP_DIR" ]; then
    sudo cp -r "$BACKUP_DIR"/* /etc/fail2ban/
    sudo systemctl restart fail2ban
    echo "Fail2ban 配置已恢复"
fi
```

---

### Q1: 如何检查 Fail2ban 是否正常工作？

**验证步骤：**

1. **检查服务状态**
   ```bash
   sudo systemctl status fail2ban
   ```

2. **查看启用的 jail**
   ```bash
   sudo fail2ban-client status
   ```

3. **模拟触发封禁**
   ```bash
   # 在另一台机器上多次尝试 SSH 登录失败
   ssh root@your-server-ip
   # 输入错误密码，直到超过 maxretry 次数
   ```

4. **查看封禁状态**
   ```bash
   sudo fail2ban-client status sshd
   sudo iptables -L -n | grep f2b
   ```

### Q2: IP 被误封了怎么办？

**解决方案：**

```bash
# 方法1：临时解封
sudo fail2ban-client set sshd unbanip 1.2.3.4

# 方法2：添加到白名单
# 编辑 /etc/fail2ban/jail.d/local.conf
ignoreip = 127.0.0.1/8 ::1 1.2.3.4

# 方法3：永久白名单
echo "1.2.3.4" | sudo tee -a /etc/fail2ban/whitelist.txt
```

### Q3: 如何调整封禁策略？

**根据需求调整参数：**

| 场景 | maxretry | findtime | bantime |
|------|----------|----------|---------|
| 基础防护 | 5 | 600 | 3600 |
| 高风险服务 | 3 | 300 | 86400 |
| 临时防护 | 10 | 60 | 600 |
| 严格防护 | 2 | 600 | 604800 |

### Q4: 为什么 IP 没有被封禁？

**排查清单：**

- ✅ 确认 jail 已启用：`sudo fail2ban-client status`
- ✅ 检查日志路径是否正确
- ✅ 测试正则表达式：`sudo fail2ban-regex /var/log/auth.log /etc/fail2ban/filter.d/sshd.conf`
- ✅ 检查 IP 是否在白名单中
- ✅ 确认防火墙后端配置正确

### Q5: 如何查看封禁历史？

```bash
# 查看完整日志
sudo cat /var/log/fail2ban.log | grep "Ban\|Unban"

# 按日期查看
sudo grep "2024-01-15" /var/log/fail2ban.log

# 统计今日封禁数
sudo grep "$(date +%Y-%m-%d)" /var/log/fail2ban.log | grep -c "Ban"
```

### Q6: Fail2ban 与其他安全工具配合？

**推荐组合：**

| 工具 | 作用 | 配合方式 |
|------|------|----------|
| **UFW/Firewalld** | 防火墙 | 作为 Fail2ban 的后端 |
| **CrowdSec** | 行为分析 | 可共存，CrowdSec 更智能 |
| **Port Knocking** | 端口隐藏 | Fail2ban 处理已暴露的服务 |
| **VPN/ZeroTier** | 访问控制 | 限制 SSH 仅 VPN 访问 + Fail2ban |

### Q7: Docker 容器如何使用 Fail2ban？

**方案一：宿主机 Fail2ban 监控容器日志**

```ini
[jail]
logpath = /var/lib/docker/containers/<container_id>/*-json.log
```

**方案二：Docker 内运行 Fail2ban**

```yaml
version: '3'
services:
  fail2ban:
    image: crazymax/fail2ban:latest
    network_mode: "host"
    cap_add:
      - NET_ADMIN
      - NET_RAW
    volumes:
      - ./fail2ban:/data
      - /var/log:/var/log:ro
```

### Q8: 如何减少性能影响？

**优化建议：**

1. **使用高效的日志后端**
   ```ini
   [DEFAULT]
   backend = systemd  # 或 auto
   ```

2. **限制监控的日志大小**
   ```ini
   [jail]
   maxlines = 1000  # 每次读取的最大行数
   ```

3. **减少不必要的 jail**
   - 只启用真正需要的服务

4. **使用 ipset 代替 iptables**
   - 大量 IP 时性能更好

---

## 📚 相关资源

- **官方文档**: [Fail2ban 官方网站](https://www.fail2ban.org/)
- **GitHub**: [fail2ban/fail2ban](https://github.com/fail2ban/fail2ban)
- **社区 Wiki**: [Fail2ban Wiki](https://github.com/fail2ban/fail2ban/wiki)

---

## 📝 最佳实践

1. **定期检查日志**：每周查看封禁记录，评估安全状况
2. **合理设置白名单**：避免封锁自己或信任的 IP
3. **渐进式封禁**：对重复违规者递增封禁时间
4. **组合防护**：配合 VPN、密钥登录等多层防护
5. **备份配置**：保存自定义配置到版本控制
6. **测试规则**：新规则上线前先用 `fail2ban-regex` 测试
7. **监控资源**：监控 Fail2ban 的 CPU/内存使用

---

## 🎯 快速参考

| 监控目标 | Jail 名称 | 配置文件 |
|---------|-----------|----------|
| SSH | sshd | `/etc/fail2ban/jail.d/sshd.conf` |
| Nginx 认证 | nginx-http-auth | `/etc/fail2ban/jail.d/nginx.conf` |
| Apache 认证 | apache | `/etc/fail2ban/jail.d/apache.conf` |
| WordPress | wordpress | `/etc/fail2ban/jail.d/wordpress.conf` |
| 重复违规 | recidive | `/etc/fail2ban/jail.d/recidive.conf` |

---

**配置完成后，您的服务器将具备强大的自动防护能力！** 🛡️

---

## 📦 1Panel + OpenResty 快速配置模板

> **快速复制粘贴配置** - 适用于 1Panel 环境的完整配置模板

### 第一步：检查日志路径

```bash
# 复制粘贴以下脚本运行
cat << 'EOF' | sudo tee /usr/local/bin/check-openresty-logs.sh
#!/bin/bash
echo "=== 1Panel OpenResty 日志检查 ==="
LOG_PATHS=(
    "/opt/1panel/apps/openresty/openresty/logs"
    "/var/log/openresty"
    "/usr/local/openresty/nginx/logs"
)
for path in "${LOG_PATHS[@]}"; do
    if [ -d "$path" ]; then
        echo "✓ $path"
        ls -lh "$path" 2>/dev/null | grep -E "\.log$"
    fi
done
EOF

sudo chmod +x /usr/local/bin/check-openresty-logs.sh
sudo /usr/local/bin/check-openresty-logs.sh
```

### 第二步：创建主配置

```bash
# 创建 Fail2ban 主配置
cat << 'EOF' | sudo tee /etc/fail2ban/jail.d/local.conf
[DEFAULT]
# 封禁 1 天
bantime = 86400

# 检测时间窗口 10 分钟
findtime = 600

# 最大失败次数
maxretry = 5

# 忽略本地 IP
ignoreip = 127.0.0.1/8 ::1

# 防火墙后端（根据实际情况选择）
# firewalld: banaction = firewallcmd-ipset
# ufw: banaction = ufw
# iptables: banaction = iptables-allports
banaction = iptables-allports
EOF
```

### 第三步：创建 OpenResty 防护配置

```bash
# 创建 1Panel OpenResty 完整防护配置
cat << 'EOF' | sudo tee /etc/fail2ban/jail.d/1panel-openresty.conf
# 1Panel OpenResty 防护配置

# SSH 防护
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
          /var/log/secure
maxretry = 3
bantime = 86400

# OpenResty 认证失败
[openresty-auth]
enabled = true
filter = nginx-http-auth
port = http,https
logpath = /opt/1panel/apps/openresty/openresty/logs/error.log
maxretry = 5
bantime = 86400

# OpenResty 恶意扫描
[openresty-scan]
enabled = true
filter = nginx-noscript
port = http,https
logpath = /opt/1panel/apps/openresty/openresty/logs/access.log
maxretry = 3
findtime = 60
bantime = 86400

# OpenResty Bad Bot
[openresty-badbots]
enabled = true
filter = nginx-badbots
port = http,https
logpath = /opt/1panel/apps/openresty/openresty/logs/access.log
maxretry = 2
bantime = 86400

# OpenResty SQL 注入
[openresty-sqli]
enabled = true
filter = openresty-sql-injection
port = http,https
logpath = /opt/1panel/apps/openresty/openresty/logs/access.log
maxretry = 3
bantime = 86400

# OpenResty 恶意 UA
[openresty-badua]
enabled = true
filter = openresty-baduseragent
port = http,https
logpath = /opt/1panel/apps/openresty/openresty/logs/access.log
maxretry = 1
bantime = 86400

# OpenResty 目录遍历
[openresty-traversal]
enabled = true
filter = openresty-dir-traversal
port = http,https
logpath = /opt/1panel/apps/openresty/openresty/logs/access.log
maxretry = 2
bantime = 86400

# WordPress 防护
[openresty-wordpress]
enabled = true
filter = wordpress
port = http,https
logpath = /opt/1panel/apps/openresty/openresty/logs/access.log
maxretry = 5
findtime = 300
bantime = 86400
EOF
```

### 第四步：创建自定义过滤规则

```bash
# SQL 注入过滤规则
cat << 'EOF' | sudo tee /etc/fail2ban/filter.d/openresty-sql-injection.conf
[Definition]
failregex = ^<HOST> -.*"(GET|POST).*(\?|%3F)(union|select|insert|delete|update|drop|create|alter|declare|cast|exec|execute).*HTTP.*"
            ^<HOST> -.*"(GET|POST).*(or|and).*(\=|%3D).*(\=|%3D).*HTTP.*"
            ^<HOST> -.*"(GET|POST).*(\%27|\%22|\;).*HTTP.*"
ignoreregex =
EOF

# 恶意 UA 过滤规则
cat << 'EOF' | sudo tee /etc/fail2ban/filter.d/openresty-baduseragent.conf
[Definition]
failregex = ^<HOST> -.*"GET.*HTTP.*".*"(?:curl|wget|python-requests|python-urllib|libwww-perl|java|go-http-client|scanner|crawler|spider|bot|scraper|sqlmap|nmap|nikto)".*$
            ^<HOST> -.*".*(sqlmap|nikto|nmap|w3af|acunetix|burpsuite|wpscan|snort).*".*$
ignoreregex =
EOF

# 目录遍历过滤规则
cat << 'EOF' | sudo tee /etc/fail2ban/filter.d/openresty-dir-traversal.conf
[Definition]
failregex = ^<HOST> -.*"(GET|POST).*(\.\.\/|\.\.\\|%2e%2e%2f|%2e%2e%5c|%252e|%252e%252f|%c0%ae).*HTTP.*"
            ^<HOST> -.*"(GET|POST).*\/etc\/passwd.*HTTP.*"
ignoreregex =
EOF
```

### 第五步：设置日志权限

```bash
# 确保 Fail2ban 可以读取日志
sudo usermod -aG www-data fail2ban 2>/dev/null || sudo usermod -aG nginx fail2ban
sudo chmod 750 /opt/1panel/apps/openresty/openresty/logs/ 2>/dev/null
sudo chmod 640 /opt/1panel/apps/openresty/openresty/logs/*.log 2>/dev/null

# 或使用 ACL
sudo setfacl -R -m u:fail2ban:rx /opt/1panel/apps/openresty/openresty/logs/ 2>/dev/null
```

### 第六步：重启并验证

```bash
# 重启 Fail2ban
sudo systemctl restart fail2ban

# 等待几秒
sleep 3

# 检查状态
sudo fail2ban-client status

# 查看所有 jail
echo "=== 启用的 Jail ==="
sudo fail2ban-client status | grep "Jail list"

# 测试正则规则
echo "=== 测试 SQL 注入规则 ==="
sudo fail2ban-regex /opt/1panel/apps/openresty/openresty/logs/access.log /etc/fail2ban/filter.d/openresty-sql-injection.conf | tail -20
```

### 第七步：日常管理命令

```bash
# 添加到 ~/.bashrc 快速访问
cat << 'EOF' | tee -a ~/.bashrc

# Fail2ban 快捷命令
alias f2b-status='sudo fail2ban-client status'
alias f2b-banned='sudo fail2ban-client status sshd | grep "Banned IP"'
alias f2b-unban='sudo fail2ban-client set sshd unbanip'
alias f2b-log='sudo tail -f /var/log/fail2ban.log'
alias f2b-restart='sudo systemctl restart fail2ban'
alias f2b-check='sudo /usr/local/bin/check-openresty-logs.sh'
EOF

source ~/.bashrc
```

### 配置文件清单

完成以上步骤后，你应该有以下配置文件：

```
/etc/fail2ban/
├── jail.d/
│   ├── local.conf                    # 主配置
│   ├── 1panel-openresty.conf         # OpenResty 防护
│   └── sshd.conf                     # SSH 防护（可选）
└── filter.d/
    ├── openresty-sql-injection.conf  # SQL 注入规则
    ├── openresty-baduseragent.conf   # 恶意 UA 规则
    └── openresty-dir-traversal.conf  # 目录遍历规则
```

### 验证清单

- [ ] 服务运行正常：`sudo systemctl status fail2ban`
- [ ] Jail 已启用：`sudo fail2ban-client status`
- [ ] 日志路径正确：`sudo fail2ban-client get openresty-sqli logpath`
- [ ] 正则规则有效：`sudo fail2ban-regex` 测试通过
- [ ] 防火墙后端正确：`sudo iptables -L -n | grep f2b`
- [ ] 权限配置正确：`sudo -u fail2ban cat /opt/1panel/apps/openresty/openresty/logs/access.log` 能读取

---

**🎉 配置完成！您的 1Panel + OpenResty 服务器现在已具备强大的安全防护能力。**
