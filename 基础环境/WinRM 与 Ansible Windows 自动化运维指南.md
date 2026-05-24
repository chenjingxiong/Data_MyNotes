---
title: WinRM 与 Ansible Windows 自动化运维指南
date: 2026-05-24
tags:
  - winrm
  - ansible
  - windows
  - hyper-v
  - automation
  - devops
related:
  - "[[基础环境/Ansible 自动化运维指南]]"
---

# WinRM 与 Ansible Windows 自动化运维指南

> **WinRM 官方文档**: [learn.microsoft.com - Windows Remote Management](https://learn.microsoft.com/en-us/windows/win32/winrm/portal)
> **Ansible Windows 指南**: [docs.ansible.com - Windows Guides](https://docs.ansible.com/ansible/latest/os_guide/windows_guide.html)
> **Hyper-V 模块**: [community.windows Collection](https://galaxy.ansible.com/ui/repo/published/community/windows/)

---

## 一、WinRM 基础概念

### 1.1 什么是 WinRM

**Windows Remote Management (WinRM)** 是微软实现的 **WS-Management 协议**，用于远程管理 Windows 系统。它是 Ansible 管理 Windows 主机的核心通信协议。

```
┌──────────────────────────────────────────────────────────┐
│                  WinRM 架构                               │
│                                                          │
│   ┌─────────────────┐          ┌──────────────────────┐  │
│   │  Ansible 控制    │  HTTP/   │  Windows 目标主机     │  │
│   │  节点 (Linux)   │  HTTPS   │                      │  │
│   │                 │ ──────▶  │  ┌────────────────┐  │  │
│   │  pywinrm 库     │  5985/   │  │ WinRM Service  │  │  │
│   │  (Python)       │  5986    │  │ (wsmauth)      │  │  │
│   │                 │          │  └───────┬────────┘  │  │
│   │                 │          │          │           │  │
│   │                 │          │  ┌───────▼────────┐  │  │
│   │                 │          │  │ WinRS / WMI    │  │  │
│   │                 │          │  │ (执行命令)      │  │  │
│   │                 │          │  └───────┬────────┘  │  │
│   │                 │          │          │           │  │
│   │                 │          │  ┌───────▼────────┐  │  │
│   │                 │          │  │ PowerShell     │  │  │
│   │                 │          │  │ (实际执行)      │  │  │
│   └─────────────────┘          │  └────────────────┘  │  │
│                                └──────────────────────┘  │
└──────────────────────────────────────────────────────────┘
```

### 1.2 WinRM vs SSH 对比

| 特性 | WinRM (Windows) | SSH (Linux) |
|------|-----------------|-------------|
| **默认端口** | 5985 (HTTP) / 5986 (HTTPS) | 22 |
| **协议** | WS-Management (SOAP) | SSH |
| **认证** | Basic / NTLM / Kerberos / CredSSP | 密钥 / 密码 |
| **Agent** | 系统内置，无需安装 | 需要 sshd 服务 |
| **传输格式** | SOAP/XML | 二进制 / 文本 |
| **加密** | HTTP 明文 / HTTPS 加密 | 始终加密 |
| **多平台** | Windows 原生支持 | 全平台 |
| **传输效率** | XML 开销较大 | 更轻量 |

> [!note] 新趋势
> Windows Server 2019+ 和 Windows 10+ 已内置 **OpenSSH Server**，Ansible 也支持通过 SSH 连接 Windows。但 WinRM 仍然是目前最成熟的方案。

### 1.3 认证方式对比

| 认证方式 | 安全性 | 适用场景 | 需要域 | 性能 |
|---------|--------|---------|--------|------|
| **Basic** | ⚠️ 低（需 HTTPS） | 工作组环境、测试 | 否 | 高 |
| **NTLM** | ★★☆ 中 | 工作组/域环境 | 否 | 中 |
| **Kerberos** | ★★★ 高 | 域环境（推荐） | **是** | 高 |
| **CredSSP** | ★★★ 高 | 需要双跳（Double Hop） | 否 | 低 |
| **Certificate** | ★★★ 高 | 高安全要求 | 否 | 高 |

---

## 二、WinRM 服务端配置（Windows 主机）

### 2.1 快速配置脚本

在目标 Windows 主机上以 **管理员 PowerShell** 运行：

```powershell
# ============================================
# WinRM 快速配置脚本（适用于工作组/测试环境）
# ============================================

# 1. 启动 WinRM 服务并设置自动启动
winrm quickconfig -force

# 2. 设置 WinRM 服务为自动启动
Set-Service -Name WinRM -StartupType Automatic

# 3. 配置 WinRM 监听器（HTTP）
winrm set winrm/config/listener?Address=*+Transport=HTTP '@{Port="5985"}'

# 4. 配置 Basic 认证（测试环境用）
winrm set winrm/config/service/auth '@{Basic="true"}'
winrm set winrm/config/service '@{AllowUnencrypted="true"}'

# 5. 配置 TrustedHosts（允许所有主机连接，生产环境应限制）
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "*" -Force

# 6. 验证配置
winrm enumerate winrm/config/listener
Write-Host "WinRM 配置完成！HTTP 端口: 5985" -ForegroundColor Green
```

### 2.2 生产环境 HTTPS 配置

> [!warning] 生产环境务必使用 HTTPS
> HTTP + Basic 认证会以明文传输密码，仅用于测试。生产环境必须配置 HTTPS。

#### 方式一：自签名证书（适合内部测试）

```powershell
# 创建自签名证书
$Cert = New-SelfSignedCertificate -DnsName "$env:COMPUTERNAME" `
    -CertStoreLocation "Cert:\LocalMachine\My" `
    -FriendlyName "WinRM HTTPS"

# 创建 HTTPS 监听器
$Thumbprint = $Cert.Thumbprint
New-Item -Path WSMan:\localhost\Listener -Transport HTTPS -Address * `
    -CertificateThumbprint $Thumbprint -Force

# 防火墙放行 5986
New-NetFirewallRule -DisplayName "WinRM HTTPS" -Direction Inbound `
    -Action Allow -Protocol TCP -LocalPort 5986

# 验证
winrm enumerate winrm/config/listener
Write-Host "HTTPS 监听器已创建，Thumbprint: $Thumbprint" -ForegroundColor Green
```

#### 方式二：企业 CA 证书（推荐生产环境）

```powershell
# 假设已从企业 CA 获取证书并导入
$Thumbprint = "YOUR_CERT_THUMBPRINT"

# 删除旧的 HTTPS 监听器（如果有）
Get-ChildItem WSMan:\localhost\Listener |
    Where-Object { $_.Transport -eq "HTTPS" } |
    Remove-Item -Force

# 创建 HTTPS 监听器
New-Item -Path WSMan:\localhost\Listener -Transport HTTPS -Address * `
    -CertificateThumbprint $Thumbprint -Force

# 确保仅使用 HTTPS
winrm set winrm/config/service '@{AllowUnencrypted="false"}'

# 关闭 HTTP 监听器（可选）
Get-ChildItem WSMan:\localhost\Listener |
    Where-Object { $_.Transport -eq "HTTP" } |
    Remove-Item -Force

# 确认
winrm enumerate winrm/config/listener
```

### 2.3 Ansible 官方配置脚本

Ansible 提供了官方的 WinRM 配置脚本，推荐使用：

```powershell
# 下载并执行 Ansible 官方 ConfigureRemotingForAnsible 脚本
# 该脚本会自动配置 WinRM 服务、证书和防火墙
$url = "https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1"
$file = "$env:temp\ConfigureRemotingForAnsible.ps1"

# 下载脚本
Invoke-WebRequest -Uri $url -OutFile $file

# 执行脚本（启用所有认证方式）
powershell.exe -ExecutionPolicy RemoteSigned -File $file -EnableCredSSP -SkipNetworkProfileCheck

# 验证
winrm enumerate winrm/config/listener
```

#### 脚本参数说明

| 参数 | 说明 |
|------|------|
| `-EnableCredSSP` | 启用 CredSSP 认证（支持双跳） |
| `-SkipNetworkProfileCheck` | 跳过网络配置文件检查（公有网络也生效） |
| `-ForceNewSSLCert` | 强制生成新的自签名 SSL 证书 |
| `-CertValidityDays` | 证书有效天数（默认 1095 天） |

### 2.4 域环境 Kerberos 配置

```powershell
# 在域控制器或域成员主机上
# WinRM 默认已支持 Kerberos，确保以下配置：

# 检查 WinRM 服务配置
winrm get winrm/config/service

# 确认 Kerberos 认证已启用
# 默认情况下 auth\Configuration=Kerberos=true
```

### 2.5 防火墙配置

```powershell
# WinRM HTTP (5985)
New-NetFirewallRule -DisplayName "WinRM HTTP" -Direction Inbound `
    -Action Allow -Protocol TCP -LocalPort 5985 -Profile Domain,Private

# WinRM HTTPS (5986)
New-NetFirewallRule -DisplayName "WinRM HTTPS" -Direction Inbound `
    -Action Allow -Protocol TCP -LocalPort 5986 -Profile Domain,Private,Public

# 查看规则
Get-NetFirewallRule -DisplayName "WinRM*" | Format-Table DisplayName, Enabled, Direction, Action
```

---

## 三、Ansible 控制节点配置

### 3.1 安装依赖

```bash
# 安装 pywinrm（WinRM Python 库）
pip install pywinrm

# 安装额外的认证库（按需）
pip install pywinrm[credssp]    # CredSSP 认证
pip install pywinrm[kerberos]   # Kerberos 认证（需系统级 Kerberos）
pip install requests-ntlm       # NTLM 认证

# 一次性安装全部
pip install "pywinrm[credssp,kerberos]" requests-ntlm
```

> [!tip] Kerberos 依赖
> 在 Linux 上使用 Kerberos 认证还需要安装系统级的 `krb5` 包：
> ```bash
> # Debian/Ubuntu
> sudo apt install krb5-user libkrb5-dev
>
> # RHEL/CentOS
> sudo dnf install krb5-workstation krb5-devel
> ```

### 3.2 Inventory 配置

#### 基础 HTTP 配置（测试环境）

```ini
# inventory/windows.ini

[windows]
win-dc01    ansible_host=192.168.1.10
win-web01   ansible_host=192.168.1.20
win-db01    ansible_host=192.168.1.30

[windows:vars]
ansible_connection = winrm
ansible_winrm_transport = basic
ansible_winrm_server_cert_validation = ignore
ansible_user = Administrator
ansible_password = YourPassword123!
ansible_port = 5985
```

#### HTTPS + NTLM 配置（推荐）

```ini
# inventory/windows.ini

[windows]
win-dc01    ansible_host=192.168.1.10
win-web01   ansible_host=192.168.1.20
win-db01    ansible_host=192.168.1.30

[windows:vars]
ansible_connection = winrm
ansible_winrm_transport = ntlm
ansible_winrm_server_cert_validation = ignore
ansible_user = Administrator
ansible_password = YourPassword123!
ansible_port = 5986
```

#### Kerberos 配置（域环境推荐）

```ini
# inventory/windows.ini

[domain_controllers]
win-dc01  ansible_host=dc01.corp.example.com

[domain_members]
win-web01 ansible_host=web01.corp.example.com
win-db01  ansible_host=db01.corp.example.com

[windows:children]
domain_controllers
domain_members

[windows:vars]
ansible_connection = winrm
ansible_winrm_transport = kerberos
ansible_user = admin@CORP.EXAMPLE.COM
ansible_password = "{{ vault_domain_admin_password }}"
ansible_port = 5986
ansible_winrm_server_cert_validation = ignore
```

Kerberos 的 `/etc/krb5.conf`：

```ini
# /etc/krb5.conf
[libdefaults]
    default_realm = CORP.EXAMPLE.COM
    dns_lookup_realm = true
    dns_lookup_kdc = true
    ticket_lifetime = 24h
    forwardable = true

[realms]
    CORP.EXAMPLE.COM = {
        kdc = dc01.corp.example.com
        admin_server = dc01.corp.example.com
    }

[domain_realm]
    .corp.example.com = CORP.EXAMPLE.COM
    corp.example.com = CORP.EXAMPLE.COM
```

#### 使用 Ansible Vault 保护密码

```bash
# 创建加密的变量文件
ansible-vault create inventory/group_vars/windows/vault.yml
```

```yaml
# inventory/group_vars/windows/vault.yml（加密）
vault_domain_admin_password: "YourSecurePassword123!"
```

```ini
# inventory/windows.ini
[windows:vars]
ansible_password = "{{ vault_domain_admin_password }}"
```

### 3.3 连接参数参考

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `ansible_connection` | 连接类型 | `ssh` |
| `ansible_port` | WinRM 端口 | `5985` |
| `ansible_user` | Windows 用户名 | — |
| `ansible_password` | Windows 密码 | — |
| `ansible_winrm_transport` | 认证方式 | `basic` |
| `ansible_winrm_server_cert_validation` | 证书验证 | `validate` |
| `ansible_winrm_operation_timeout_sec` | 操作超时 | `20` |
| `ansible_winrm_read_timeout_sec` | 读取超时 | `30` |
| `ansible_winrm_message_encryption` | 消息加密 | `auto` |
| `ansible_winrm_ca_trust_path` | CA 证书路径 | — |
| `ansible_winrm_cert_pem` | 客户端证书 | — |
| `ansible_winrm_cert_key_pem` | 客户端密钥 | — |

### 3.4 测试连接

```bash
# 测试 WinRM 连通性
ansible windows -i inventory/windows.ini -m win_ping

# 成功输出示例:
# win-web01 | SUCCESS => {
#     "changed": false,
#     "ping": "pong"
# }

# 收集 Windows 主机信息
ansible windows -i inventory/windows.ini -m setup

# 查看特定 facts
ansible windows -i inventory/windows.ini -m setup \
  -a "filter=ansible_os_family"
```

---

## 四、Ansible Windows 常用模块

### 4.1 核心模块速查

| FQCN | 用途 | 替代 Linux 模块 |
|------|------|----------------|
| `ansible.windows.win_ping` | 测试连通性 | `ping` |
| `ansible.windows.win_shell` | 执行 PowerShell/命令 | `shell` |
| `ansible.windows.win_command` | 执行命令（非 Shell） | `command` |
| `ansible.windows.win_copy` | 复制文件到 Windows | `copy` |
| `ansible.windows.win_template` | 渲染 Jinja2 模板 | `template` |
| `ansible.windows.win_file` | 文件/目录管理 | `file` |
| `ansible.windows.win_stat` | 检查文件状态 | `stat` |
| `ansible.windows.win_owner` | 文件所有者 | `owner` |
| `ansible.windows.win_acl` | ACL 权限管理 | — |
| `ansible.windows.win_user` | 本地用户管理 | `user` |
| `ansible.windows.win_group` | 本地组管理 | `group` |
| `ansible.windows.win_service` | Windows 服务管理 | `service` |
| `ansible.windows.win_feature` | 安装 Windows 功能 | `apt`/`yum` |
| `ansible.windows.win_package` | 安装软件包 | `package` |
| `ansible.windows.win_updates` | Windows 更新 | — |
| `ansible.windows.win_regedit` | 注册表管理 | — |
| `ansible.windows.win_hostname` | 设置主机名 | `hostname` |
| `ansible.windows.win_dns_client` | DNS 客户端配置 | — |
| `ansible.windows.win_firewall` | 防火墙管理 | — |
| `ansible.windows.win_firewall_rule` | 防火墙规则 | — |
| `ansible.windows.win_disk_facts` | 磁盘信息 | — |
| `ansible.windows.win_format` | 格式化磁盘 | — |
| `ansible.windows.win_partition` | 分区管理 | — |
| `ansible.windows.win_share` | SMB 共享管理 | — |
| `ansible.windows.win_scheduled_task` | 计划任务 | `cron` |
| `ansible.windows.win_uri` | HTTP 请求（类似 curl） | `uri` |
| `ansible.windows.win_get_url` | 下载文件 | `get_url` |
| `ansible.windows.win_unzip` | 解压文件 | `unarchive` |
| `ansible.windows.win_zip` | 压缩文件 | `archive` |
| `community.windows.win_iis_website` | IIS 网站管理 | — |
| `community.windows.win_iis_webapppool` | IIS 应用池管理 | — |
| `community.windows.win_psmodule` | PowerShell 模块管理 | — |
| `chocolatey.chocolatey.win_chocolatey` | Chocolatey 包管理 | `apt`/`yum` |

### 4.2 实战示例

#### 系统管理

```yaml
---
- name: Windows 系统基础配置
  hosts: windows
  gather_facts: yes

  tasks:
    # 设置主机名
    - name: 设置计算机名
      ansible.windows.win_hostname:
        name: "{{ inventory_hostname_short }}"
      register: hostname_result

    - name: 重启以应用主机名
      ansible.windows.win_reboot:
      when: hostname_result.reboot_required

    # 创建本地用户
    - name: 创建管理员用户
      ansible.windows.win_user:
        name: deploy
        password: "{{ vault_deploy_password }}"
        state: present
        groups:
          - Administrators
          - "Remote Desktop Users"
        password_never_expires: yes
        user_cannot_change_password: no

    # 创建本地组
    - name: 创建运维组
      ansible.windows.win_group:
        name: OpsTeam
        description: "Operations Team"
        state: present

    # Windows 防火墙
    - name: 启用防火墙（所有配置文件）
      ansible.windows.win_firewall:
        profiles:
          - Domain
          - Private
          - Public
        state: enabled

    # 防火墙规则
    - name: 放行 RDP 端口
      ansible.windows.win_firewall_rule:
        name: "Allow RDP"
        localport: 3389
        action: allow
        direction: in
        protocol: tcp
        state: present
        enabled: yes

    - name: 放行 WinRM HTTPS
      ansible.windows.win_firewall_rule:
        name: "Allow WinRM HTTPS"
        localport: 5986
        action: allow
        direction: in
        protocol: tcp
        state: present
        enabled: yes

    # DNS 配置
    - name: 配置 DNS 服务器
      ansible.windows.win_dns_client:
        adapter_names: "*"
        dns_servers:
          - 192.168.1.1
          - 8.8.8.8

    # 注册表
    - name: 禁用 UAC 远程限制
      ansible.windows.win_regedit:
        path: HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System
        name: LocalAccountTokenFilterPolicy
        data: 1
        type: dword

    - name: 设置时区
      ansible.windows.win_shell: |
        Set-TimeZone -Id "China Standard Time"
```

#### 软件安装与包管理

```yaml
---
- name: 软件安装与包管理
  hosts: windows

  tasks:
    # 安装 Windows 功能
    - name: 安装 Telnet 客户端
      ansible.windows.win_feature:
        name: Telnet-Client
        state: present
      register: feature_result

    - name: 安装 IIS（Web 服务器）
      ansible.windows.win_feature:
        name:
          - Web-Server
          - Web-Mgmt-Tools
          - Web-Scripting-Tools
        state: present
        include_sub_features: yes
        include_management_tools: yes
      register: iis_result

    - name: 安装后重启（如果需要）
      ansible.windows.win_reboot:
      when: iis_result.reboot_required

    # 使用 Chocolatey 安装软件
    - name: 安装 Chocolatey
      ansible.windows.win_shell: |
        Set-ExecutionPolicy Bypass -Scope Process -Force
        [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
        Invoke-Expression ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
      args:
        creates: C:\ProgramData\chocolatey\bin\choco.exe

    - name: 通过 Chocolatey 安装软件
      chocolatey.chocolatey.win_chocolatey:
        name: "{{ item }}"
        state: present
      loop:
        - git
        - vim
        - 7zip
        - curl
        - vscode
        - nodejs-lts
        - python

    # 安装 MSI 包
    - name: 下载并安装 MSI 包
      ansible.windows.win_package:
        path: https://example.com/software/app-1.0.msi
        state: present
        arguments: '/quiet /norestart'

    # 安装 EXE 包
    - name: 安装 Visual Studio Code
      ansible.windows.win_package:
        path: https://update.code.visualstudio.com/latest/win32-x64/stable
        product_id: "{AEB7D607-DA44-4255-8FF1-5A019652B40D}"
        arguments:
          - /VERYSILENT
          - /NORESTART
          - /MERGETASKS=!runcode
        state: present

    # Windows 更新
    - name: 安装 Windows 更新
      ansible.windows.win_updates:
        category_names:
          - SecurityUpdates
          - CriticalUpdates
        state: installed
        reboot: yes
      register: update_result

    - name: 显示更新结果
      ansible.builtin.debug:
        msg: "已安装 {{ update_result.installed_update_count }} 个更新"
```

#### 文件与目录管理

```yaml
---
- name: 文件与目录管理
  hosts: windows

  tasks:
    # 创建目录
    - name: 创建应用目录
      ansible.windows.win_file:
        path: "C:\\Apps\\MyApplication"
        state: directory

    # 复制文件
    - name: 复制配置文件
      ansible.windows.win_copy:
        src: ./files/app.config
        dest: "C:\\Apps\\MyApplication\\app.config"

    # 使用模板
    - name: 渲染并部署配置
      ansible.windows.win_template:
        src: ./templates/web.config.j2
        dest: "C:\\inetpub\\wwwroot\\web.config"

    # 下载文件
    - name: 下载安装包
      ansible.windows.win_get_url:
        url: https://example.com/app-v2.0.zip
        dest: "C:\\Downloads\\app-v2.0.zip"
        checksum: "sha256:ABCDEF1234567890..."
        force: no

    # 解压文件
    - name: 解压应用
      community.windows.win_unzip:
        src: "C:\\Downloads\\app-v2.0.zip"
        dest: "C:\\Apps\\MyApplication"
        delete_archive: yes

    # 搜索并替换文件内容
    - name: 修改配置文件
      ansible.windows.win_lineinfile:
        path: "C:\\Apps\\MyApplication\\app.config"
        regexp: '^<add key="ServerURL"'
        line: '<add key="ServerURL" value="https://new-server.example.com" />'
        state: present

    # ACL 权限
    - name: 设置目录权限
      ansible.windows.win_acl:
        path: "C:\\Apps\\MyApplication"
        user: "IIS_IUSRS"
        rights: "ReadAndExecute"
        type: allow
        state: present
        inherit: ContainerInherit,ObjectInherit
        propagation: None
```

#### 服务与计划任务

```yaml
---
- name: 服务与计划任务管理
  hosts: windows

  tasks:
    # 服务管理
    - name: 确保 IIS 服务运行
      ansible.windows.win_service:
        name: W3SVC
        state: started
        start_mode: auto

    - name: 停止并禁用不需要的服务
      ansible.windows.win_service:
        name: "{{ item }}"
        state: stopped
        start_mode: disabled
      loop:
        - Spooler          # 打印服务
        - WSearch          # Windows Search
        - SysMain

    # 计划任务
    - name: 创建日志清理计划任务
      ansible.windows.win_scheduled_task:
        name: "CleanupLogs"
        description: "每日清理过期日志文件"
        actions:
          - path: powershell.exe
            arguments: >
              -ExecutionPolicy Bypass
              -File "C:\\Scripts\\Cleanup-Logs.ps1"
        triggers:
          - type: daily
            start_boundary: "2026-01-01T02:00:00"
        username: SYSTEM
        run_level: highest
        state: present
        enabled: yes

    - name: 创建系统状态检查任务
      ansible.windows.win_scheduled_task:
        name: "SystemHealthCheck"
        description: "每 15 分钟检查系统状态"
        actions:
          - path: powershell.exe
            arguments: >
              -ExecutionPolicy Bypass
              -Command "Get-Process |
              Where-Object { $_.WorkingSet64 -gt 1GB } |
              ConvertTo-Json |
              Out-File C:\\Logs\\heavy-processes.json"
        triggers:
          - type: boot
          - type: registration
            repetition:
              interval: "PT15M"
              duration: "P1D"
        username: SYSTEM
        state: present
        enabled: yes
```

#### IIS Web 服务器管理

```yaml
---
- name: IIS Web 服务器部署
  hosts: windows
  vars:
    site_name: "MyApp"
    site_path: "C:\\inetpub\\wwwroot\\MyApp"
    app_pool_name: "MyAppPool"
    bindings:
      - protocol: http
        port: 80
        hostname: "myapp.example.com"
      - protocol: https
        port: 443
        hostname: "myapp.example.com"
        certificate_hash: "YOUR_CERT_THUMBPRINT"

  tasks:
    # 安装 IIS
    - name: 安装 IIS 角色
      ansible.windows.win_feature:
        name:
          - Web-Server
          - Web-Mgmt-Tools
          - Web-Scripting-Tools
          - Web-Asp-Net45
        state: present
        include_management_tools: yes
      register: iis_install

    - name: 重启（如果需要）
      ansible.windows.win_reboot:
      when: iis_install.reboot_required

    # 创建应用池
    - name: 创建应用池
      community.windows.win_iis_webapppool:
        name: "{{ app_pool_name }}"
        state: present
        attributes:
          managedRuntimeVersion: "v4.0"
          processModel.identityType: ApplicationPoolIdentity
          processModel.idleTimeout: "00:30:00"
          recycling.periodicRestart.time: "24:00:00"

    # 创建网站目录
    - name: 创建网站目录
      ansible.windows.win_file:
        path: "{{ site_path }}"
        state: directory

    # 部署应用文件
    - name: 部署应用
      ansible.windows.win_copy:
        src: "./files/myapp/"
        dest: "{{ site_path }}\\"

    # 创建网站
    - name: 创建 IIS 网站
      community.windows.win_iis_website:
        name: "{{ site_name }}"
        state: started
        physical_path: "{{ site_path }}"
        application_pool: "{{ app_pool_name }}"
        binding:
          - protocol: http
            port: 80
            hostname: "myapp.example.com"
      register: website

    # 设置目录权限
    - name: 设置网站目录权限
      ansible.windows.win_acl:
        path: "{{ site_path }}"
        user: "IIS_IUSRS"
        rights: "ReadAndExecute"
        type: allow
        state: present
```

### 4.3 执行 PowerShell 脚本

```yaml
---
- name: PowerShell 脚本执行示例
  hosts: windows

  tasks:
    # 执行简单命令
    - name: 获取系统信息
      ansible.windows.win_shell: |
        $os = Get-CimInstance Win32_OperatingSystem
        [PSCustomObject]@{
          Hostname  = $env:COMPUTERNAME
          OS        = $os.Caption
          Version   = $os.Version
          Memory_GB = [math]::Round($os.TotalVisibleMemorySize / 1MB, 2)
          Uptime    = (Get-Date) - $os.LastBootUpTime
        } | ConvertTo-Json
      register: system_info

    - name: 显示系统信息
      ansible.builtin.debug:
        msg: "{{ system_info.stdout | from_json }}"

    # 执行本地 PS1 脚本
    - name: 上传并执行脚本
      ansible.windows.win_copy:
        src: ./scripts/Configure-Windows.ps1
        dest: "C:\\Temp\\Configure-Windows.ps1"

    - name: 执行配置脚本
      ansible.windows.win_shell: |
        powershell.exe -ExecutionPolicy Bypass -File "C:\\Temp\\Configure-Windows.ps1"
      register: script_result

    - name: 清理脚本
      ansible.windows.win_file:
        path: "C:\\Temp\\Configure-Windows.ps1"
        state: absent

    # 带条件的 PowerShell
    - name: 检查并重启服务（如果卡死）
      ansible.windows.win_shell: |
        $svc = Get-Service -Name "MyApp" -ErrorAction SilentlyContinue
        if ($svc.Status -eq 'Stopped' -and $svc.StartType -eq 'Automatic') {
            Start-Service -Name "MyApp"
            Write-Output "restarted"
        } else {
            Write-Output "ok"
        }
      register: svc_check
      changed_when: "'restarted' in svc_check.stdout"
```

---

## 五、Ansible 管理 Hyper-V

### 5.1 Hyper-V 环境准备

#### 控制节点安装依赖

```bash
# 安装 Ansible Windows Collection（包含 Hyper-V 模块）
ansible-galaxy collection install ansible.windows
ansible-galaxy collection install community.windows

# 安装 pywinrm
pip install pywinrm
```

#### Hyper-V 主机要求

| 要求 | 说明 |
|------|------|
| **操作系统** | Windows Server 2016+ (含 Hyper-V 角色) 或 Windows 10/11 Pro/Enterprise |
| **Hyper-V 角色** | 已安装并启用 |
| **WinRM** | 已配置并可连接 |
| **PowerShell** | 5.1+（建议 7.x） |
| **权限** | 需要管理员权限或 Hyper-V Administrators 组成员 |

```yaml
# 检查 Hyper-V 状态
- name: 检查 Hyper-V 角色
  ansible.windows.win_feature:
    name: Hyper-V
    state: present
  register: hyperv_check

- name: 安装 Hyper-V 角色
  ansible.windows.win_feature:
    name:
      - Hyper-V
      - Hyper-V-PowerShell
      - RSAT-Hyper-V-Tools
    state: present
    include_management_tools: yes
  register: hyperv_install

- name: 重启以完成安装
  ansible.windows.win_reboot:
  when: hyperv_install.reboot_required
```

### 5.2 Inventory 配置

```ini
# inventory/hyperv.ini

[hyperv_hosts]
hyperv-01  ansible_host=192.168.1.50
hyperv-02  ansible_host=192.168.1.51

[hyperv_hosts:vars]
ansible_connection = winrm
ansible_winrm_transport = ntlm
ansible_winrm_server_cert_validation = ignore
ansible_user = Administrator
ansible_password = "{{ vault_hyperv_password }}"
ansible_port = 5986

# 虚拟机目标（通过 delegate_to 指定 Hyper-V 主机）
[vm_guests]
vm-web01   hyperv_host=hyperv-01
vm-db01    hyperv_host=hyperv-01
vm-app01   hyperv_host=hyperv-02
```

### 5.3 虚拟机生命周期管理

#### 创建虚拟机

```yaml
---
- name: 创建 Hyper-V 虚拟机
  hosts: hyperv_hosts
  gather_facts: no

  vars:
    vm_name: "VM-Web01"
    vm_path: "D:\\Hyper-V\\VMs"
    vhd_path: "D:\\Hyper-V\\VHDs"
    iso_path: "D:\\ISO\\WindowsServer2022.iso"
    vm_cpu: 4
    vm_memory: 4096MB
    vm_disk_size: 80GB
    vm_switch: "External Switch"

  tasks:
    # 创建 VHD（虚拟硬盘）
    - name: 创建虚拟硬盘
      ansible.windows.win_shell: |
        New-VHD -Path "{{ vhd_path }}\\{{ vm_name }}.vhdx" `
          -SizeBytes {{ vm_disk_size }} `
          -Dynamic `
          -BlockSizeBytes 1MB
      register: vhd_result
      changed_when: "'Created' in vhd_result.stdout"

    # 创建虚拟机
    - name: 创建虚拟机
      ansible.windows.win_shell: |
        $vm = New-VM -Name "{{ vm_name }}" `
          -Path "{{ vm_path }}" `
          -MemoryStartupBytes {{ vm_memory }} `
          -VHDPath "{{ vhd_path }}\\{{ vm_name }}.vhdx" `
          -SwitchName "{{ vm_switch }}" `
          -Generation 2

        # 配置 CPU
        Set-VMProcessor -VMName "{{ vm_name }}" -Count {{ vm_cpu }}

        # 配置动态内存
        Set-VMMemory -VMName "{{ vm_name }}" `
          -DynamicMemoryEnabled $true `
          -MinimumBytes 1GB `
          -MaximumBytes 8GB `
          -StartupBytes {{ vm_memory }}

        # 启用嵌套虚拟化（可选）
        Set-VMProcessor -VMName "{{ vm_name }}" -ExposeVirtualizationExtensions $true

        Write-Output "VM {{ vm_name }} created successfully"
      register: vm_result
      changed_when: "'created successfully' in vm_result.stdout"

    # 挂载 ISO（安装介质）
    - name: 挂载安装 ISO
      ansible.windows.win_shell: |
        Add-VMDvdDrive -VMName "{{ vm_name }}" -Path "{{ iso_path }}"

    # 设置启动顺序
    - name: 设置从 ISO 启动
      ansible.windows.win_shell: |
        $vm = Get-VM -Name "{{ vm_name }}"
        $dvd = $vm | Get-VMDvdDrive
        $firmware = Get-VMFirmware -VMName "{{ vm_name }}"
        $bootOrder = $firmware.BootOrder | Where-Object {
          $_.BootType -ne 'Network' -and $_.Device -ne $dvd
        }
        $newBootOrder = @($dvd) + @($firmware.BootOrder | Where-Object {
          $_.Device -ne $dvd
        })
        Set-VMFirmware -VMName "{{ vm_name }}" -BootOrder $newBootOrder

    # 启动虚拟机
    - name: 启动虚拟机
      ansible.windows.win_shell: |
        Start-VM -Name "{{ vm_name }}"
      register: start_result
      changed_when: "'Running' in start_result.stdout or start_result.changed"
```

#### 批量创建虚拟机

```yaml
---
- name: 批量创建虚拟机
  hosts: hyperv_hosts
  gather_facts: no

  vars:
    vm_switch: "External Switch"
    vm_path: "D:\\Hyper-V\\VMs"
    vhd_path: "D:\\Hyper-V\\VHDs"
    vm_base_image: "D:\\Hyper-V\\Templates\\Base-Win2022.vhdx"
    vms:
      - name: "VM-Web01"
        cpu: 2
        memory: 2048MB
        disk: 40GB
      - name: "VM-Web02"
        cpu: 2
        memory: 2048MB
        disk: 40GB
      - name: "VM-DB01"
        cpu: 4
        memory: 8192MB
        disk: 200GB
      - name: "VM-App01"
        cpu: 4
        memory: 4096MB
        disk: 100GB

  tasks:
    - name: 从模板创建虚拟机
      ansible.windows.win_shell: |
        $vmInfo = @{
          Name       = "{{ item.name }}"
          Path       = "{{ vm_path }}"
          MemoryStartupBytes = {{ item.memory }}
          SwitchName = "{{ vm_switch }}"
          Generation = 2
        }

        # 检查 VM 是否已存在
        $existing = Get-VM -Name "{{ item.name }}" -ErrorAction SilentlyContinue
        if ($existing) {
            Write-Output "VM {{ item.name }} already exists"
            exit 0
        }

        # 从模板 VHD 复制
        $destVhd = "{{ vhd_path }}\\{{ item.name }}.vhdx"
        Copy-Item -Path "{{ vm_base_image }}" -Destination $destVhd

        # 创建 VM
        $vm = New-VM @vmInfo -VHDPath $destVhd

        # 配置 CPU
        Set-VMProcessor -VMName "{{ item.name }}" -Count {{ item.cpu }}

        # 配置动态内存
        Set-VMMemory -VMName "{{ item.name }}" `
          -DynamicMemoryEnabled $true `
          -MinimumBytes 512MB `
          -MaximumBytes {{ item.memory }}

        # 扩展系统盘（如果需要更大）
        $currentSize = (Get-VHD -Path $destVhd).Size
        $targetSize = {{ item.disk }}
        if ($targetSize -gt $currentSize) {
            Resize-VHD -Path $destVhd -SizeBytes $targetSize
        }

        Write-Output "VM {{ item.name }} created"
      loop: "{{ vms }}"
      register: vm_create
      changed_when: "'created' in item.stdout | default('')"
```

#### 虚拟机状态管理

```yaml
---
- name: 虚拟机生命周期管理
  hosts: hyperv_hosts

  vars:
    vm_name: "VM-Web01"

  tasks:
    # 查看所有虚拟机状态
    - name: 获取所有虚拟机状态
      ansible.windows.win_shell: |
        Get-VM | Select-Object Name, State, CPUUsage, MemoryAssigned,
          Uptime, Status, IntegrationServicesState |
          ConvertTo-Json
      register: all_vms

    - name: 显示虚拟机列表
      ansible.builtin.debug:
        msg: "{{ all_vms.stdout | from_json }}"

    # 启动虚拟机
    - name: 启动虚拟机
      ansible.windows.win_shell: |
        Start-VM -Name "{{ vm_name }}"
      when: true

    # 正常关机
    - name: 优雅关机
      ansible.windows.win_shell: |
        Stop-VM -Name "{{ vm_name }}"
      # 等待 Guest OS 正常关闭

    # 强制关机
    - name: 强制关机（断电）
      ansible.windows.win_shell: |
        Stop-VM -Name "{{ vm_name }}" -Force
      # 等同于拔电源

    # 保存状态（休眠）
    - name: 保存虚拟机状态
      ansible.windows.win_shell: |
        Save-VM -Name "{{ vm_name }}"

    # 暂停/恢复
    - name: 暂停虚拟机
      ansible.windows.win_shell: |
        Suspend-VM -Name "{{ vm_name }}"

    - name: 恢复虚拟机
      ansible.windows.win_shell: |
        Resume-VM -Name "{{ vm_name }}"

    # 重启
    - name: 重启虚拟机
      ansible.windows.win_shell: |
        Restart-VM -Name "{{ vm_name }}" -Force

    # 删除虚拟机
    - name: 停止并删除虚拟机
      block:
        - name: 停止虚拟机
          ansible.windows.win_shell: |
            Stop-VM -Name "{{ vm_name }}" -Force -ErrorAction SilentlyContinue

        - name: 删除虚拟机及其文件
          ansible.windows.win_shell: |
            Remove-VM -Name "{{ vm_name }}" -Force
            $vhdPath = "D:\\Hyper-V\\VHDs\\{{ vm_name }}.vhdx"
            if (Test-Path $vhdPath) {
                Remove-Item $vhdPath -Force
            }
      when: false  # 安全开关，防止误删
```

### 5.4 虚拟机快照管理

```yaml
---
- name: 虚拟机快照（Checkpoint）管理
  hosts: hyperv_hosts

  vars:
    vm_name: "VM-Web01"

  tasks:
    # 创建快照
    - name: 创建更新前快照
      ansible.windows.win_shell: |
        Checkpoint-VM -Name "{{ vm_name }}" -SnapshotName "Before-Update-20260524"

    # 查看所有快照
    - name: 列出所有快照
      ansible.windows.win_shell: |
        Get-VMSnapshot -VMName "{{ vm_name }}" |
          Select-Object Name, CreationTime, ParentSnapshotName, State |
          ConvertTo-Json
      register: snapshots

    - name: 显示快照列表
      ansible.builtin.debug:
        msg: "{{ snapshots.stdout | from_json }}"

    # 恢复快照
    - name: 恢复到指定快照
      ansible.windows.win_shell: |
        Restore-VMSnapshot -VMName "{{ vm_name }}" -Name "Before-Update-20260524"
        Start-VM -Name "{{ vm_name }}"
      when: false  # 按需启用

    # 删除快照
    - name: 删除旧快照
      ansible.windows.win_shell: |
        Get-VMSnapshot -VMName "{{ vm_name }}" |
          Where-Object { $_.CreationTime -lt (Get-Date).AddDays(-30) } |
          Remove-VMSnapshot
      # 自动清理 30 天前的快照

    # 导出虚拟机
    - name: 导出虚拟机（完整备份）
      ansible.windows.win_shell: |
        Export-VM -Name "{{ vm_name }}" -Path "E:\\Backup\\VMs\\"
```

### 5.5 虚拟网络管理

```yaml
---
- name: Hyper-V 虚拟网络管理
  hosts: hyperv_hosts

  tasks:
    # 查看现有虚拟交换机
    - name: 列出虚拟交换机
      ansible.windows.win_shell: |
        Get-VMSwitch | Select-Object Name, SwitchType, NetAdapterInterfaceDescription |
          ConvertTo-Json
      register: switches

    - name: 显示交换机列表
      ansible.builtin.debug:
        msg: "{{ switches.stdout | from_json }}"

    # 创建外部虚拟交换机（绑定物理网卡）
    - name: 创建外部虚拟交换机
      ansible.windows.win_shell: |
        $adapter = Get-NetAdapter | Where-Object { $_.Status -eq 'Up' -and $_.MediaType -eq '802.3' } | Select-Object -First 1
        New-VMSwitch -Name "External Switch" -NetAdapterName $adapter.Name `
          -AllowManagementOS $true
      register: switch_result
      changed_when: "'External Switch' in switch_result.stdout | default('')"

    # 创建内部虚拟交换机（仅宿主机与虚拟机通信）
    - name: 创建内部虚拟交换机
      ansible.windows.win_shell: |
        New-VMSwitch -Name "Internal Switch" -SwitchType Internal

    # 创建专用虚拟交换机（仅虚拟机之间通信）
    - name: 创建专用虚拟交换机
      ansible.windows.win_shell: |
        New-VMSwitch -Name "Private Switch" -SwitchType Private

    # 配置内部交换机的 NAT（让内部 VM 可以上网）
    - name: 配置 NAT 网络
      ansible.windows.win_shell: |
        # 给内部交换机的 vEthernet 分配 IP
        $adapter = Get-NetAdapter | Where-Object { $_.Name -like "*Internal Switch*" }
        New-NetIPAddress -InterfaceAlias $adapter.Name `
          -IPAddress 192.168.100.1 `
          -PrefixLength 24

        # 创建 NAT
        New-NetNat -Name "InternalNAT" `
          -InternalIPInterfaceAddressPrefix 192.168.100.0/24

    # 修改虚拟机的网络适配器
    - name: 修改 VM 网络适配器
      ansible.windows.win_shell: |
        # 连接到指定交换机
        Get-VMNetworkAdapter -VMName "VM-Web01" |
          Connect-VMNetworkAdapter -SwitchName "External Switch"

    # 添加额外的网络适配器
    - name: 添加额外的网络适配器
      ansible.windows.win_shell: |
        Add-VMNetworkAdapter -VMName "VM-Web01" `
          -Name "Storage" `
          -SwitchName "Private Switch"

    # 设置 VLAN
    - name: 配置 VLAN
      ansible.windows.win_shell: |
        Set-VMNetworkAdapterVlan -VMName "VM-Web01" `
          -Access -VlanId 100
```

### 5.6 虚拟硬盘管理

```yaml
---
- name: 虚拟硬盘管理
  hosts: hyperv_hosts

  vars:
    vhd_path: "D:\\Hyper-V\\VHDs"

  tasks:
    # 创建动态扩展 VHD
    - name: 创建动态 VHD
      ansible.windows.win_shell: |
        New-VHD -Path "{{ vhd_path }}\\data-disk.vhdx" `
          -SizeBytes 200GB `
          -Dynamic

    # 创建固定大小 VHD
    - name: 创建固定 VHD（高性能）
      ansible.windows.win_shell: |
        New-VHD -Path "{{ vhd_path }}\\high-perf.vhdx" `
          -SizeBytes 100GB `
          -Fixed

    # 创建差异磁盘（链接到父磁盘）
    - name: 创建差异磁盘
      ansible.windows.win_shell: |
        New-VHD -ParentPath "{{ vhd_path }}\\Base-Win2022.vhdx" `
          -Path "{{ vhd_path }}\\vm-diff.vhdx" `
          -Differencing

    # 扩展 VHD
    - name: 扩展虚拟硬盘
      ansible.windows.win_shell: |
        Resize-VHD -Path "{{ vhd_path }}\\data-disk.vhdx" -SizeBytes 300GB

    # 压缩 VHD（减少文件大小）
    - name: 压缩虚拟硬盘
      ansible.windows.win_shell: |
        # 需要先关闭 VM
        Optimize-VHD -Path "{{ vhd_path }}\\data-disk.vhdx" -Mode Full

    # 合并差异磁盘
    - name: 合并差异磁盘到父磁盘
      ansible.windows.win_shell: |
        Merge-VHD -Path "{{ vhd_path }}\\vm-diff.vhdx" -DestinationPath "{{ vhd_path }}\\merged.vhdx"

    # 将 VHD 挂载到虚拟机
    - name: 添加数据磁盘到虚拟机
      ansible.windows.win_shell: |
        Add-VMHardDiskDrive -VMName "VM-DB01" `
          -Path "{{ vhd_path }}\\data-disk.vhdx" `
          -ControllerType SCSI

    # 查看磁盘信息
    - name: 获取 VHD 信息
      ansible.windows.win_shell: |
        Get-VHD -Path "{{ vhd_path }}\\data-disk.vhdx" |
          Select-Object Path, VhdType, FileSize, Size, FragmentationPercentage |
          ConvertTo-Json
      register: vhd_info

    - name: 显示磁盘信息
      ansible.builtin.debug:
        msg: "{{ vhd_info.stdout | from_json }}"
```

### 5.7 虚拟机监控与维护

```yaml
---
- name: Hyper-V 监控与维护
  hosts: hyperv_hosts

  tasks:
    # 获取虚拟机详细状态
    - name: 获取 VM 资源使用情况
      ansible.windows.win_shell: |
        Get-VM | ForEach-Object {
            $vm = $_
            [PSCustomObject]@{
                Name           = $vm.Name
                State          = $vm.State
                CPUUsage_pct   = $vm.CPUUsage
                MemoryAssigned_MB = [math]::Round($vm.MemoryAssigned / 1MB, 0)
                MemoryDemand_MB   = [math]::Round($vm.MemoryDemand / 1MB, 0)
                Uptime         = $vm.Uptime.ToString()
                Status         = $vm.Status
                Version        = $vm.Version
            }
        } | ConvertTo-Json
      register: vm_status

    - name: 显示 VM 状态
      ansible.builtin.debug:
        msg: "{{ vm_status.stdout | from_json }}"

    # 获取 Hyper-V 主机资源
    - name: 获取 Hyper-V 主机信息
      ansible.windows.win_shell: |
        $os = Get-CimInstance Win32_OperatingSystem
        $cpu = Get-CimInstance Win32_Processor
        [PSCustomObject]@{
            Hostname    = $env:COMPUTERNAME
            CPU_Model   = $cpu[0].Name
            CPU_Cores   = ($cpu | Measure-Object -Property NumberOfCores -Sum).Sum
            Total_RAM_GB  = [math]::Round($os.TotalVisibleMemorySize / 1MB, 2)
            Free_RAM_GB   = [math]::Round($os.FreePhysicalMemory / 1MB, 2)
            Used_RAM_pct  = [math]::Round(($os.TotalVisibleMemorySize - $os.FreePhysicalMemory) / $os.TotalVisibleMemorySize * 100, 1)
            VM_Count    = (Get-VM).Count
            VM_Running  = (Get-VM | Where-Object { $_.State -eq 'Running' }).Count
        } | ConvertTo-Json
      register: host_info

    - name: 显示主机信息
      ansible.builtin.debug:
        msg: "{{ host_info.stdout | from_json }}"

    # 获取磁盘 I/O 统计
    - name: 获取 VM 磁盘 I/O
      ansible.windows.win_shell: |
        Get-VM | Get-VMHardDiskDrive | ForEach-Object {
            $disk = $_
            $path = $disk.Path
            $vhd = Get-VHD -Path $path -ErrorAction SilentlyContinue
            [PSCustomObject]@{
                VMName       = $disk.VMName
                Controller   = $disk.ControllerType
                VHD_Path     = $path
                VHD_Type     = $vhd.VhdType
                Size_GB      = [math]::Round($vhd.Size / 1GB, 2)
                FileSize_GB  = [math]::Round($vhd.FileSize / 1GB, 2)
                Fragmentation = $vhd.FragmentationPercentage
            }
        } | ConvertTo-Json
      register: disk_io

    # 集成服务状态检查
    - name: 检查集成服务状态
      ansible.windows.win_shell: |
        Get-VM | Select-Object Name, IntegrationServicesState,
          @{N='IntegrationServicesVersion';E={$_.IntegrationServicesVersion}} |
          ConvertTo-Json
      register: integration_services

    - name: 显示集成服务状态
      ansible.builtin.debug:
        msg: "{{ integration_services.stdout | from_json }}"

    # 更新集成服务
    - name: 更新虚拟机集成服务
      ansible.windows.win_shell: |
        Get-VM | ForEach-Object {
            Enable-VMIntegrationService -VMName $_.Name -Name "Guest Service Interface"
        }
```

---

## 六、完整实战：自动化部署 Hyper-V 虚拟机集群

### 6.1 项目结构

```
ansible-hyperv/
├── ansible.cfg
├── inventory/
│   ├── hyperv.ini
│   ├── group_vars/
│   │   ├── all.yml
│   │   └── hyperv_hosts.yml
│   └── host_vars/
│       └── hyperv-01.yml
├── playbooks/
│   ├── hyperv-setup.yml          # Hyper-V 主机初始化
│   ├── vm-create.yml             # 创建虚拟机
│   ├── vm-configure.yml          # 配置虚拟机
│   ├── vm-lifecycle.yml          # 生命周期管理
│   ├── vm-backup.yml             # 备份与快照
│   └── vm-monitoring.yml         # 监控巡检
├── templates/
│   ├── unattend.xml.j2           # Windows 无人值守应答文件
│   └── vm-config.ps1.j2          # VM 配置脚本模板
├── files/
│   └── Configure-WinRM.ps1       # WinRM 配置脚本
├── roles/
│   ├── hyperv_prerequisites/     # Hyper-V 环境准备
│   ├── vm_create/                # 创建虚拟机
│   ├── vm_network/               # 网络配置
│   └── vm_guest_setup/           # Guest OS 初始化
├── group_vars/
│   └── vault.yml                 # 加密变量
└── site.yml                      # 主入口
```

### 6.2 主 Playbook

```yaml
# site.yml — Hyper-V 虚拟化环境完整部署
---
- name: 1. 初始化 Hyper-V 主机环境
  import_playbook: playbooks/hyperv-setup.yml
  tags: [setup, hyperv]

- name: 2. 创建虚拟网络
  import_playbook: playbooks/vm-network-setup.yml
  tags: [network]

- name: 3. 创建虚拟机
  import_playbook: playbooks/vm-create.yml
  tags: [create]

- name: 4. 配置虚拟机 Guest OS
  import_playbook: playbooks/vm-configure.yml
  tags: [configure]

- name: 5. 执行日常监控
  import_playbook: playbooks/vm-monitoring.yml
  tags: [monitoring]
```

### 6.3 Hyper-V 主机初始化 Playbook

```yaml
# playbooks/hyperv-setup.yml
---
- name: 初始化 Hyper-V 主机
  hosts: hyperv_hosts
  gather_facts: yes

  tasks:
    - name: 检查操作系统版本
      ansible.builtin.debug:
        msg: "{{ ansible_distribution }} {{ ansible_distribution_version }}"

    - name: 安装 Hyper-V 角色
      ansible.windows.win_feature:
        name:
          - Hyper-V
          - Hyper-V-PowerShell
          - Hyper-V-Tools
        state: present
        include_management_tools: yes
      register: hyperv_install

    - name: 重启完成 Hyper-V 安装
      ansible.windows.win_reboot:
        reboot_timeout: 600
      when: hyperv_install.reboot_required

    - name: 创建 VM 存储目录
      ansible.windows.win_file:
        path: "{{ item }}"
        state: directory
      loop:
        - "D:\\Hyper-V\\VMs"
        - "D:\\Hyper-V\\VHDs"
        - "D:\\Hyper-V\\Templates"
        - "D:\\Hyper-V\\ISOs"

    - name: 设置默认 VM 路径
      ansible.windows.win_shell: |
        Set-VMHost -VirtualMachinePath "D:\\Hyper-V\\VMs" `
          -VirtualHardDiskPath "D:\\Hyper-V\\VHDs"

    - name: 配置 Hyper-V 主机参数
      ansible.windows.win_shell: |
        Set-VMHost -MaximumStorageMigrations 2 `
          -MaximumVirtualMachineMigrations 2 `
          -VirtualMachineMigrationEnabled $true

    - name: 验证 Hyper-V 安装
      ansible.windows.win_shell: |
        Get-WindowsFeature -Name Hyper-V | Select-Object Name, InstallState | ConvertTo-Json
      register: verify

    - name: 显示安装结果
      ansible.builtin.debug:
        msg: "{{ verify.stdout | from_json }}"
```

### 6.4 虚拟机创建 Playbook

```yaml
# playbooks/vm-create.yml
---
- name: 创建虚拟机
  hosts: hyperv_hosts

  vars_files:
    - ../group_vars/vault.yml

  vars:
    vm_switch: "External Switch"
    vm_path: "D:\\Hyper-V\\VMs"
    vhd_path: "D:\\Hyper-V\\VHDs"
    template_path: "D:\\Hyper-V\\Templates\\Base-WinServer2022.vhdx"

    # 虚拟机定义列表
    virtual_machines:
      - name: "VM-DC01"
        cpu: 2
        memory_startup: 2048MB
        memory_min: 1024MB
        memory_max: 4096MB
        disk_size: 60GB
        notes: "Domain Controller"
      - name: "VM-Web01"
        cpu: 4
        memory_startup: 4096MB
        memory_min: 2048MB
        memory_max: 8192MB
        disk_size: 80GB
        notes: "Web Server"
        extra_disks:
          - name: "logs"
            size: 50GB
      - name: "VM-DB01"
        cpu: 4
        memory_startup: 8192MB
        memory_min: 4096MB
        memory_max: 16384MB
        disk_size: 100GB
        notes: "Database Server"
        extra_disks:
          - name: "data"
            size: 200GB
          - name: "logs"
            size: 100GB

  tasks:
    - name: 检查模板文件存在
      ansible.windows.win_stat:
        path: "{{ template_path }}"
      register: template_check
      run_once: yes

    - name: 失败提示（模板不存在）
      ansible.builtin.fail:
        msg: "模板文件 {{ template_path }} 不存在！请先准备基础模板。"
      when: not template_check.stat.exists

    - name: 创建虚拟机
      ansible.windows.win_shell: |
        $vmName = "{{ item.name }}"
        $vhdFile = "{{ vhd_path }}\\$vmName\\$vmName.vhdx"

        # 检查是否已存在
        if (Get-VM -Name $vmName -ErrorAction SilentlyContinue) {
            Write-Output "EXISTS:$vmName"
            exit 0
        }

        # 创建 VM 目录
        New-Item -ItemType Directory -Path "{{ vhd_path }}\\$vmName" -Force | Out-Null
        New-Item -ItemType Directory -Path "{{ vm_path }}\\$vmName" -Force | Out-Null

        # 从模板复制 VHD
        Copy-Item "{{ template_path }}" $vhdFile

        # 创建虚拟机
        $vm = New-VM -Name $vmName `
          -Path "{{ vm_path }}\\$vmName" `
          -MemoryStartupBytes {{ item.memory_startup }} `
          -VHDPath $vhdFile `
          -SwitchName "{{ vm_switch }}" `
          -Generation 2

        # 配置 CPU
        Set-VMProcessor -VMName $vmName -Count {{ item.cpu }}

        # 配置动态内存
        Set-VMMemory -VMName $vmName `
          -DynamicMemoryEnabled $true `
          -MinimumBytes {{ item.memory_min | default('1024MB') }} `
          -MaximumBytes {{ item.memory_max | default('8192MB') }} `
          -StartupBytes {{ item.memory_startup }}

        # 设置 VM 备注
        Set-VM -Name $vmName -Notes "{{ item.notes | default('') }}"

        # 设置自动启动操作
        Set-VM -Name $vmName -AutomaticStartAction StartIfRunning
        Set-VM -Name $vmName -AutomaticStopAction Save

        Write-Output "CREATED:$vmName"
      loop: "{{ virtual_machines }}"
      register: vm_create_results

    - name: 创建额外数据磁盘
      ansible.windows.win_shell: |
        $vmName = "{{ item.0.name }}"
        $diskName = "{{ item.1.name }}"
        $diskSize = "{{ item.1.size }}"
        $vhdFile = "{{ vhd_path }}\\$vmName\\${vmName}-${diskName}.vhdx"

        if (Test-Path $vhdFile) {
            Write-Output "EXISTS:$vhdFile"
            exit 0
        }

        # 创建新 VHD
        New-VHD -Path $vhdFile -SizeBytes $diskSize -Dynamic | Out-Null

        # 挂载到 VM
        Add-VMHardDiskDrive -VMName $vmName -Path $vhdFile -ControllerType SCSI

        Write-Output "CREATED:$vhdFile"
      loop: "{{ virtual_machines | subelements('extra_disks', skip_missing=True) }}"
      register: disk_create_results

    - name: 启动所有新创建的虚拟机
      ansible.windows.win_shell: |
        $vm = Get-VM -Name "{{ item.name }}" -ErrorAction SilentlyContinue
        if ($vm -and $vm.State -eq 'Off') {
            Start-VM -Name "{{ item.name }}"
            Write-Output "STARTED:{{ item.name }}"
        } else {
            Write-Output "SKIP:{{ item.name }}"
        }
      loop: "{{ virtual_machines }}"

    - name: 等待虚拟机启动完成
      ansible.windows.win_shell: |
        $vm = Get-VM -Name "{{ item.name }}"
        $timeout = 120
        $elapsed = 0
        while ($vm.State -ne 'Running' -and $elapsed -lt $timeout) {
            Start-Sleep -Seconds 5
            $elapsed += 5
            $vm = Get-VM -Name "{{ item.name }}"
        }
        if ($vm.State -eq 'Running') {
            Write-Output "READY:{{ item.name }}"
        } else {
            Write-Error "TIMEOUT:{{ item.name }}"
        }
      loop: "{{ virtual_machines }}"
```

### 6.5 虚拟机监控 Playbook

```yaml
# playbooks/vm-monitoring.yml
---
- name: Hyper-V 监控巡检
  hosts: hyperv_hosts
  gather_facts: yes

  vars:
    report_path: "C:\\Reports"
    alert_cpu_threshold: 80      # CPU 使用率告警阈值 %
    alert_memory_threshold: 90   # 内存使用率告警阈值 %

  tasks:
    - name: 创建报告目录
      ansible.windows.win_file:
        path: "{{ report_path }}"
        state: directory

    - name: 收集 Hyper-V 主机信息
      ansible.windows.win_shell: |
        $hostInfo = @{
            Hostname    = $env:COMPUTERNAME
            Timestamp   = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
            OS          = (Get-CimInstance Win32_OperatingSystem).Caption
            CPU_Usage   = (Get-Counter '\Processor(_Total)\% Processor Time').CounterSamples.CookedValue
            Total_RAM_GB   = [math]::Round((Get-CimInstance Win32_OperatingSystem).TotalVisibleMemorySize / 1MB, 2)
            Free_RAM_GB    = [math]::Round((Get-CimInstance Win32_OperatingSystem).FreePhysicalMemory / 1MB, 2)
            VM_Total    = (Get-VM).Count
            VM_Running  = (Get-VM | Where-Object State -eq 'Running').Count
            VM_Off      = (Get-VM | Where-Object State -eq 'Off').Count
        }
        $hostInfo | ConvertTo-Json
      register: host_report

    - name: 收集虚拟机状态
      ansible.windows.win_shell: |
        Get-VM | ForEach-Object {
            $vm = $_
            @{
                Name         = $vm.Name
                State        = $vm.State.ToString()
                CPU_Usage    = $vm.CPUUsage
                Memory_MB    = [math]::Round($vm.MemoryAssigned / 1MB, 0)
                Uptime       = if ($vm.Uptime) { $vm.Uptime.ToString() } else { "N/A" }
                Status       = $vm.Status
                Health       = $vm.HealthState.ToString()
            }
        } | ConvertTo-Json
      register: vm_report

    - name: 生成 HTML 报告
      ansible.windows.win_shell: |
        $hostData = '{{ host_report.stdout | from_json | to_json }}' | ConvertFrom-Json
        $vmData = '{{ vm_report.stdout | from_json | to_json }}' | ConvertFrom-Json

        $html = @"
        <!DOCTYPE html>
        <html>
        <head>
            <title>Hyper-V Monitor Report - $(Get-Date -Format 'yyyy-MM-dd')</title>
            <style>
                body { font-family: 'Segoe UI', sans-serif; margin: 20px; background: #f5f5f5; }
                h1 { color: #0078d4; }
                h2 { color: #333; }
                table { border-collapse: collapse; width: 100%; margin: 10px 0; background: white; }
                th { background: #0078d4; color: white; padding: 10px; text-align: left; }
                td { padding: 8px 10px; border-bottom: 1px solid #ddd; }
                tr:hover { background: #f0f8ff; }
                .alert { color: #d13438; font-weight: bold; }
                .ok { color: #107c10; }
                .summary { display: flex; gap: 20px; margin: 20px 0; }
                .card { background: white; padding: 15px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
            </style>
        </head>
        <body>
            <h1>🖥️ Hyper-V Monitor Report</h1>
            <p>Host: $($hostData.Hostname) | Time: $($hostData.Timestamp)</p>
            <div class="summary">
                <div class="card"><strong>Total VMs:</strong> $($hostData.VM_Total)</div>
                <div class="card"><strong>Running:</strong> <span class="ok">$($hostData.VM_Running)</span></div>
                <div class="card"><strong>Stopped:</strong> $($hostData.VM_Off)</div>
                <div class="card"><strong>Host RAM Free:</strong> $($hostData.Free_RAM_GB) GB</div>
            </div>
            <h2>Virtual Machines</h2>
            <table>
                <tr><th>Name</th><th>State</th><th>CPU %</th><th>Memory (MB)</th><th>Uptime</th><th>Health</th></tr>
        "@

        foreach ($vm in $vmData) {
            $cpuClass = if ($vm.CPU_Usage -gt {{ alert_cpu_threshold }}) { 'alert' } else { 'ok' }
            $html += "<tr><td>$($vm.Name)</td><td>$($vm.State)</td><td class='$cpuClass'>$($vm.CPU_Usage)%</td><td>$($vm.Memory_MB)</td><td>$($vm.Uptime)</td><td>$($vm.Health)</td></tr>"
        }

        $html += @"
            </table>
        </body>
        </html>
        "@

        $reportFile = "{{ report_path }}\\HyperV-Report-$(Get-Date -Format 'yyyyMMdd-HHmmss').html"
        $html | Out-File -FilePath $reportFile -Encoding UTF8
        Write-Output $reportFile
      register: report_file

    - name: 显示报告路径
      ansible.builtin.debug:
        msg: "报告已生成: {{ report_file.stdout }}"
```

---

## 七、Windows SSH 连接（替代 WinRM）

> [!info] Windows OpenSSH
> Windows 10 1809+ 和 Windows Server 2019+ 内置 OpenSSH Server，Ansible 可以通过 SSH 连接 Windows。

### 7.1 启用 Windows SSH

```powershell
# 在 Windows 主机上执行

# 安装 OpenSSH Server
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# 启动并设置自动启动
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic

# 确认防火墙规则（默认自动添加）
Get-NetFirewallRule -Name *ssh*

# 配置默认 Shell 为 PowerShell
New-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH" `
  -Name DefaultShell -Value "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" `
  -PropertyType String -Force
```

### 7.2 Ansible SSH 连接配置

```ini
# inventory/windows-ssh.ini

[windows_ssh]
win-web01  ansible_host=192.168.1.20
win-db01   ansible_host=192.168.1.30

[windows_ssh:vars]
ansible_connection = ssh
ansible_shell_type = powershell
ansible_user = Administrator
ansible_password = "{{ vault_win_password }}"
ansible_port = 22
```

```yaml
# 等同的 YAML inventory
all:
  children:
    windows_ssh:
      hosts:
        win-web01:
          ansible_host: 192.168.1.20
        win-db01:
          ansible_host: 192.168.1.30
      vars:
        ansible_connection: ssh
        ansible_shell_type: powershell
        ansible_user: Administrator
        ansible_port: 22
```

> [!warning] WinRM vs SSH
> | 特性 | WinRM | SSH |
> |------|-------|-----|
> | 模块兼容性 | ✅ 完全支持 | ⚠️ 部分限制 |
> | 成熟度 | ★★★★★ | ★★★☆☆ |
> | 性能 | 中等 | 较好 |
> | 文件传输 | 更可靠 | 更快 |
> | 推荐度 | **生产环境首选** | 实验性 / 简单场景 |

---

## 八、故障排查指南

### 8.1 WinRM 连接问题

| 错误 | 原因 | 解决方案 |
|------|------|---------|
| `504 WinRM timeout` | WinRM 服务未启动 | `Start-Service WinRM` |
| `401 Unauthorized` | 认证失败 | 检查用户名密码、认证方式 |
| `500 WinRM Shell` | Shell 初始化失败 | 检查 PowerShell 版本 |
| `SSL: CERTIFICATE_VERIFY_FAILED` | 证书验证失败 | 设置 `ansible_winrm_server_cert_validation=ignore` |
| `Connection refused` | 端口未监听 | 检查防火墙和 WinRM 监听器 |
| `kerberos: authGSSClientInit()` | Kerberos 未配置 | 安装 `krb5` 并配置 `/etc/krb5.conf` |

### 8.2 诊断命令

```bash
# 从 Ansible 控制节点测试 WinRM 连接
# 使用 Python 直接测试
python3 -c "
import winrm
s = winrm.Session('192.168.1.10', auth=('Administrator', 'Password'), transport='ntlm', server_cert_validation='ignore')
r = s.run_cmd('hostname')
print(r.status_code)
print(r.std_out.decode())
"

# 测试连接
ansible windows -i inventory/windows.ini -m win_ping -vvv

# 查看详细 WinRM 通信日志
ansible windows -i inventory/windows.ini -m win_ping -vvvv
```

```powershell
# 在 Windows 主机上诊断
# 检查 WinRM 服务状态
Get-Service WinRM

# 检查监听器
winrm enumerate winrm/config/listener

# 检查配置
winrm get winrm/config

# 测试本地 WinRM 连接
Test-WSMan -ComputerName localhost

# 测试远程连接（从另一台 Windows）
Test-WSMan -ComputerName 192.168.1.10 -Authentication Default
```

### 8.3 CredSSP 双跳问题

> [!warning] 双跳问题 (Double Hop)
> 当 Ansible 通过 WinRM 连接 Windows，再从 Windows 访问网络资源（如共享文件夹、远程数据库）时，认证凭据不会传递到第二个跃点。

**解决方案一：CredSSP**

```yaml
# ansible.cfg 或 inventory
ansible_winrm_transport: credssp
```

```powershell
# 在 Windows 主机上启用 CredSSP
Enable-WSManCredSSP -Role Server -Force

# 在 Ansible 控制节点（Linux）
# 安装依赖
pip install pywinrm[credssp]
```

**解决方案二：使用 `become`**

```yaml
- name: 访问网络共享
  ansible.windows.win_shell: |
    Get-ChildItem \\file-server\share
  become: yes
  become_method: runas
  become_user: "{{ domain_admin }}"
  vars:
    ansible_become_password: "{{ vault_domain_password }}"
```

---

## 九、安全最佳实践

### 9.1 连接安全

| 实践 | 说明 |
|------|------|
| ✅ 使用 HTTPS | 生产环境始终使用 5986 端口 |
| ✅ 使用域证书 | 从企业 CA 获取证书，避免自签名 |
| ✅ 限制 TrustedHosts | 仅允许可信的 Ansible 控制节点 IP |
| ✅ 使用 Kerberos | 域环境首选 Kerberos 认证 |
| ✅ 启用防火墙 | 仅放行必要的端口和来源 IP |
| ❌ 避免明文 | 不要在 HTTP 模式下使用 Basic 认证 |
| ❌ 避免硬编码密码 | 使用 Ansible Vault 加密所有凭据 |

### 9.2 Ansible Vault 保护 Windows 凭据

```yaml
# group_vars/windows/vault.yml
vault_win_admin_password: "SecurePassword123!"
vault_domain_admin: "admin@CORP.EXAMPLE.COM"
vault_domain_password: "DomainPassword456!"
```

```bash
# 加密
ansible-vault encrypt group_vars/windows/vault.yml

# 执行时提供密码
ansible-playbook site.yml --ask-vault-pass
```

### 9.3 最小权限原则

```yaml
# 创建专用的 Ansible 服务账户
- name: 创建 Ansible 服务用户
  ansible.windows.win_user:
    name: ansible_svc
    password: "{{ vault_ansible_svc_password }}"
    password_never_expires: yes
    user_cannot_change_password: yes
    groups:
      - "Hyper-V Administrators"    # 仅 Hyper-V 管理权限
      - "Remote Management Users"   # WinRM 远程管理
    state: present
```

---

## 十、相关资源

- 📘 [Ansible Windows 指南](https://docs.ansible.com/ansible/latest/os_guide/windows_guide.html)
- 📘 [WinRM 官方文档](https://learn.microsoft.com/en-us/windows/win32/winrm/portal)
- 📘 [Hyper-V PowerShell 模块](https://learn.microsoft.com/en-us/powershell/module/hyper-v/)
- 📦 [ansible.windows Collection](https://galaxy.ansible.com/ui/repo/published/ansible/windows/)
- 📦 [community.windows Collection](https://galaxy.ansible.com/ui/repo/published/community/windows/)
- 🔧 [ConfigureRemotingForAnsible.ps1](https://github.com/ansible/ansible/blob/devel/examples/scripts/ConfigureRemotingForAnsible.ps1)
- 🔧 [pywinrm (Python)](https://github.com/diyan/pywinrm)
- 📰 [[基础环境/Ansible 自动化运维指南|Ansible 自动化运维指南]]
