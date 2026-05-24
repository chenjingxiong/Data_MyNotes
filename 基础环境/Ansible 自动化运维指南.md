---
title: Ansible 自动化运维指南
date: 2026-05-16
tags:
  - ansible
  - automation
  - devops
  - infrastructure
  - configuration-management
  - windows
  - hyper-v
  - firecracker
  - microvm
  - serverless
related:
  - "[[基础环境/WinRM 与 Ansible Windows 自动化运维指南]]"
  - "[[Firecracker microVM 介绍与使用指南]]"
  - "[[Firecracker 生态与配合项目指南]]"
source: https://github.com/ansible/ansible
---

# Ansible 自动化运维指南

> **项目地址**: [github.com/ansible/ansible](https://github.com/ansible/ansible)
> **官方文档**: [docs.ansible.com](https://docs.ansible.com/ansible/latest/)
> **许可证**: GPL v3.0 | **语言**: Python | **作者**: Michael DeHaan / Red Hat

---

## 一、什么是 Ansible

Ansible 是一个**极简的 IT 自动化系统**，由 Red Hat 维护，用于：

| 能力        | 说明                                 |
| --------- | ---------------------------------- |
| **配置管理**  | 统一管理服务器配置状态                        |
| **应用部署**  | 自动化应用的构建、发布、更新                     |
| **云资源编排** | AWS / Azure / GCP / OpenStack 资源管理 |
| **任务自动化** | 日常运维任务的批量执行                        |
| **网络自动化** | 网络设备的配置和管理                         |
| **零停机更新** | 滚动更新 + 负载均衡协调                      |

### 核心设计原则

1. **无代理 (Agentless)** — 不需要在目标机器上安装任何客户端，直接利用 SSH (Linux) / WinRM (Windows) → 详见 [[基础环境/WinRM 与 Ansible Windows 自动化运维指南|WinRM 指南]]
2. **幂等性 (Idempotent)** — 同一个 Playbook 执行多次结果一致，不会重复操作
3. **人类可读** — 使用 YAML 描述自动化逻辑，易读易写
4. **最小依赖** — 控制节点仅需 Python，无需数据库或守护进程
5. **模块化** — 支持用任何动态语言开发模块（不限于 Python）

---

## 二、架构概览

```
┌─────────────────────────────────────────────┐
│              控制节点 (Control Node)          │
│                                             │
│  ┌───────────┐  ┌──────────┐  ┌──────────┐ │
│  │ Playbooks │  │ Inventory│  │ Modules  │ │
│  │  (YAML)   │  │ (主机清单)│  │ (功能模块)│ │
│  └─────┬─────┘  └─────┬────┘  └────┬─────┘ │
│        └───────────────┼───────────┘        │
│                        │                    │
│                   ansible-core               │
│                        │                    │
└────────────────────────┼────────────────────┘
                         │ SSH / WinRM
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
   ┌────────────┐ ┌────────────┐ ┌────────────┐
   │ Managed    │ │ Managed    │ │ Managed    │
   │ Node 1     │ │ Node 2     │ │ Node N     │
   │ (Linux)    │ │ (Windows)  │ │ (Network)  │
   └────────────┘ └────────────┘ └────────────┘
```

### 关键概念

| 概念 | 说明 |
|------|------|
| **Control Node** | 安装 Ansible 的管理机器 |
| **Managed Node** | 被管理的目标主机 |
| **Inventory** | 主机清单，定义管理哪些机器 |
| **Module** | 执行具体操作的代码单元（如 `yum`、`copy`、`service`） |
| **Task** | 调用单个模块的一次操作 |
| **Playbook** | 一组有序 Task 的 YAML 编排文件 |
| **Role** | 可复用的 Playbook 组织结构 |
| **Collection** | 模块、Role、插件的分发包（FQCN 命名） |
| **Handler** | 在 Task 触发变更时才执行的回调操作 |

---

## 三、安装

### 方式一：pip（推荐，全平台通用）

```bash
# 安装最新稳定版
pip install ansible

# 安装特定版本
pip install ansible==11.0.0

# 仅安装核心引擎（不含社区 Collection）
pip install ansible-core
```

### 方式二：系统包管理器

```bash
# Debian / Ubuntu
sudo apt update && sudo apt install ansible

# RHEL / CentOS / Fedora
sudo dnf install ansible        # Fedora / RHEL 8+
sudo yum install ansible        # 旧版 RHEL/CentOS

# macOS
brew install ansible

# Alpine
apk add ansible
```

### 方式三：从源码运行（开发者）

```bash
git clone https://github.com/ansible/ansible.git
cd ansible
source hacking/env-setup
pip install -r requirements.txt
```

### 验证安装

```bash
ansible --version
# 输出示例:
# ansible [core 2.18.x]
#   config file = /etc/ansible/ansible.cfg
#   python version = 3.12.x
```

---

## 四、快速上手

### 4.1 配置 SSH 免密登录

Ansible 通过 SSH 连接目标机器，需先配置免密：

```bash
# 生成密钥（如果还没有）
ssh-keygen -t ed25519

# 将公钥复制到目标主机
ssh-copy-id user@target-host
```

### 4.2 创建主机清单 (Inventory)

创建 `inventory.ini` 文件：

```ini
[webservers]
web1.example.com
web2.example.com
192.168.1.10

[dbservers]
db1.example.com ansible_user=admin

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

### 4.3 第一个 Ad-Hoc 命令

```bash
# 测试连通性
ansible all -i inventory.ini -m ping

# 查看主机信息
ansible webservers -i inventory.ini -m setup

# 批量执行命令
ansible all -i inventory.ini -a "uptime"
```

### 4.4 第一个 Playbook

创建 `site.yml`：

```yaml
---
- name: 部署 Web 服务器
  hosts: webservers
  become: yes          # 使用 sudo 提权

  tasks:
    - name: 安装 Nginx
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: yes

    - name: 上传配置文件
      ansible.builtin.copy:
        src: ./nginx.conf
        dest: /etc/nginx/nginx.conf
        mode: '0644'
      notify: Reload Nginx      # 变更时触发 Handler

    - name: 确保 Nginx 运行
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: yes

  handlers:
    - name: Reload Nginx
      ansible.builtin.service:
        name: nginx
        state: reloaded
```

执行：

```bash
ansible-playbook -i inventory.ini site.yml
```

### 4.5 常用执行参数

```bash
# 检查模式（Dry Run，不实际执行）
ansible-playbook site.yml --check

# 限制目标主机
ansible-playbook site.yml --limit "web1.example.com"

# 按标签执行
ansible-playbook site.yml --tags "install"

# 查看详细输出
ansible-playbook site.yml -vv

# 指定私钥文件
ansible-playbook site.yml --private-key ~/.ssh/my_key
```

---

## 五、核心语法详解

### 5.1 变量 (Variables)

```yaml
# 在 Playbook 中定义
- name: 使用变量示例
  hosts: webservers
  vars:
    http_port: 8080
    server_name: "myapp.example.com"

  tasks:
    - name: 配置 Nginx
      ansible.builtin.template:
        src: nginx.conf.j2
        dest: /etc/nginx/sites-available/default
      vars:
        # 任务级变量（覆盖 Play 级）
        http_port: 9090
```

#### 变量优先级（从低到高）

```
role defaults → inventory file vars → inventory group_vars →
inventory host_vars → playbook vars → playbook vars_prompt →
playbook vars_files → role vars → block vars → task vars →
extra vars (命令行 -e)
```

#### `group_vars` / `host_vars` 组织

```
project/
├── inventory/
│   ├── inventory.ini
│   ├── group_vars/
│   │   ├── webservers.yml     # web 服务器组的变量
│   │   └── dbservers.yml      # 数据库组的变量
│   └── host_vars/
│       ├── web1.yml           # 单台主机的变量
│       └── db1.yml
├── site.yml
└── roles/
```

### 5.2 模板 (Jinja2)

Ansible 使用 Jinja2 模板引擎，文件后缀为 `.j2`：

```nginx
# templates/nginx.conf.j2
server {
    listen {{ http_port }};
    server_name {{ server_name }};

    {% for upstream in upstream_servers %}
    location {{ upstream.path }} {
        proxy_pass http://{{ upstream.host }}:{{ upstream.port }};
    }
    {% endfor %}

    # 条件渲染
    {% if enable_ssl %}
    ssl_certificate     {{ ssl_cert_path }};
    ssl_certificate_key {{ ssl_key_path }};
    {% endif %}
}
```

常用 Jinja2 过滤器：

```yaml
# 默认值
port: "{{ http_port | default(80) }}"

# 类型转换
items: "{{ my_string | int }}"

# 列表操作
first: "{{ my_list | first }}"
last: "{{ my_list | last }}"
unique: "{{ my_list | unique }}"

# 字典操作
keys: "{{ my_dict | dict2items }}"

# 加密
hashed: "{{ password | password_hash('sha512') }}"

# JSON 查询
users: "{{ data | json_query('users[*].name') }}"
```

### 5.3 条件判断 (When)

```yaml
tasks:
  - name: Debian 系列安装
    ansible.builtin.apt:
      name: nginx
      state: present
    when: ansible_os_family == "Debian"

  - name: RedHat 系列安装
    ansible.builtin.yum:
      name: nginx
      state: present
    when: ansible_os_family == "RedHat"

  - name: 仅在磁盘空间不足时清理
    ansible.builtin.shell: cleanup.sh
    when: ansible_mounts | selectattr('mount', 'equalto', '/') |
          map(attribute='size_available') | first | int < 1073741824
```

### 5.4 循环 (Loop)

```yaml
tasks:
  # 标准循环
  - name: 安装多个软件包
    ansible.builtin.apt:
      name: "{{ item }}"
      state: present
    loop:
      - git
      - vim
      - htop
      - tmux

  # 字典循环
  - name: 创建多个用户
    ansible.builtin.user:
      name: "{{ item.name }}"
      groups: "{{ item.groups }}"
      shell: "{{ item.shell }}"
    loop:
      - { name: 'alice', groups: 'sudo', shell: '/bin/bash' }
      - { name: 'bob', groups: 'docker', shell: '/bin/zsh' }

  # 在文件中查找
  - name: 检查文件存在
    ansible.builtin.stat:
      path: "{{ item }}"
    loop: "{{ lookup('fileglob', '/etc/conf/*.conf', wantlist=True) }}"
    register: conf_files
```

### 5.5 角色与集合

#### 角色 (Roles)

```bash
# 初始化角色骨架
ansible-galaxy init myrole
```

角色结构：

```
roles/myrole/
├── defaults/
│   └── main.yml       # 默认变量（最低优先级）
├── vars/
│   └── main.yml       # 角色变量（高优先级）
├── tasks/
│   └── main.yml       # 主任务列表
├── handlers/
│   └── main.yml       # Handler 定义
├── templates/
│   └── *.j2           # Jinja2 模板
├── files/
│   └── *              # 静态文件
├── meta/
│   └── main.yml       # 角色依赖和元数据
└── tests/
    ├── inventory
    └── test.yml
```

使用角色：

```yaml
- name: 完整部署
  hosts: webservers
  roles:
    - role: nginx
      vars:
        nginx_port: 8080
    - role: mysql
      when: db_required | bool
```

#### 集合 (Collections)

```bash
# 安装集合
ansible-galaxy collection install community.docker
ansible-galaxy collection install ansible.utils

# 列出已安装集合
ansible-galaxy collection list
```

`requirements.yml`：

```yaml
collections:
  - name: community.docker
    version: ">=3.0.0"
  - name: ansible.posix
  - name: ansible.utils

roles:
  - name: geerlingguy.nginx
    version: "3.1.0"
```

批量安装：

```bash
ansible-galaxy install -r requirements.yml
```

---

## 六、推荐项目结构

```
ansible-project/
├── ansible.cfg                # 项目配置
├── inventory/
│   ├── production.ini         # 生产环境清单
│   ├── staging.ini            # 测试环境清单
│   ├── group_vars/
│   │   ├── all.yml
│   │   ├── webservers.yml
│   │   └── dbservers.yml
│   └── host_vars/
│       └── web1.yml
├── playbooks/
│   ├── site.yml               # 主入口
│   ├── webservers.yml
│   └── dbservers.yml
├── roles/
│   ├── nginx/
│   ├── mysql/
│   └── common/
├── collections/
│   └── requirements.yml
├── templates/
├── files/
├── filter_plugins/
├── library/                   # 自定义模块
└── tests/
```

`ansible.cfg` 示例：

```ini
[defaults]
inventory      = inventory/production.ini
roles_path     = roles
host_key_checking = False
retry_files_enabled = False
stdout_callback = yaml
timeout = 30

[privilege_escalation]
become = True
become_method = sudo

[ssh_connection]
pipelining = True
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
```

---

## 七、高级主题

### 7.1 Ansible Vault（加密敏感数据）

```bash
# 创建加密文件
ansible-vault create secrets.yml

# 编辑加密文件
ansible-vault edit secrets.yml

# 加密已有文件
ansible-vault encrypt group_vars/production.yml

# 解密文件
ansible-vault decrypt group_vars/production.yml

# 更改密码
ansible-vault rekey secrets.yml

# 执行时提供密码
ansible-playbook site.yml --ask-vault-pass
ansible-playbook site.yml --vault-password-file ~/.vault_pass
```

加密变量示例：

```yaml
# secrets.yml（加密后）
db_password: "SuperSecret123!"
api_key: "sk-xxxxxxxxxxxxxxxx"
```

### 7.2 动态清单 (Dynamic Inventory)

适用于云环境，自动发现主机：

```bash
# AWS
pip install boto3
ansible-inventory -i aws_ec2.yml --list

# Azure
pip install azure-mgmt-compute
ansible-inventory -i azure_rm.yml --list
```

AWS 动态清单配置 `aws_ec2.yml`：

```yaml
plugin: amazon.aws.aws_ec2
regions:
  - us-east-1
  - ap-northeast-1
filters:
  tag:Environment: production
keyed_groups:
  - key: tags.Role
    prefix: role
hostvars:
  - ansible_user: ubuntu
```

### 7.3 异步任务 (Async)

长时间运行的任务不阻塞：

```yaml
- name: 异步执行备份
  ansible.builtin.shell: /opt/backup.sh
  async: 3600           # 最长等待 1 小时
  poll: 0               # 不等待，立即返回
  register: backup_job

- name: 做其他事情...

- name: 检查备份是否完成
  ansible.builtin.async_status:
    jid: "{{ backup_job.ansible_job_id }}"
  register: job_result
  until: job_result.finished
  retries: 60
  delay: 60
```

### 7.4 委托 (Delegate) 与本地操作

```yaml
# 在本地执行（而非目标主机）
- name: 本地生成配置
  ansible.builtin.template:
    src: config.j2
    dest: /tmp/generated_config
  delegate_to: localhost
  run_once: true

# 在另一台主机上执行
- name: 从负载均衡器摘除节点
  ansible.builtin.shell: disable_node.sh {{ inventory_hostname }}
  delegate_to: lb.example.com
```

### 7.5 错误处理 (Block/Rescue)

```yaml
tasks:
  - block:
      - name: 尝试部署
        ansible.builtin.shell: deploy.sh

    rescue:
      - name: 部署失败，回滚
        ansible.builtin.shell: rollback.sh

      - name: 发送告警
        ansible.builtin.mail:
          to: admin@example.com
          subject: "部署失败: {{ inventory_hostname }}"

    always:
      - name: 清理临时文件
        ansible.builtin.file:
          path: /tmp/deploy_temp
          state: absent
```

---

## 八、常用模块速查

### 系统管理

| FQCN | 用途 | 示例 |
|------|------|------|
| `ansible.builtin.apt` | Debian 系包管理 | `name: nginx state: present` |
| `ansible.builtin.yum` | RHEL 系包管理 | `name: nginx state: latest` |
| `ansible.builtin.service` | 服务管理 | `name: nginx state: started` |
| `ansible.builtin.user` | 用户管理 | `name: deploy shell: /bin/bash` |
| `ansible.builtin.file` | 文件/目录管理 | `path: /app state: directory` |
| `ansible.builtin.copy` | 复制文件 | `src: app.conf dest: /etc/` |
| `ansible.builtin.template` | 渲染模板 | `src: nginx.j2 dest: /etc/nginx/` |
| `ansible.builtin.lineinfile` | 修改单行 | `path: /etc/hosts line: '...'` |
| `ansible.builtin.replace` | 替换文本 | `regexp: 'old' replace: 'new'` |
| `ansible.builtin.shell` | Shell 命令 | `cmd: ls /tmp` |
| `ansible.builtin.command` | 命令执行 | `cmd: /usr/bin/make` |
| `ansible.builtin.cron` | 定时任务 | `name: backup minute: 0 job: '...'` |
| `ansible.builtin.wait_for` | 等待端口/文件 | `port: 8080 delay: 5` |

### 云与容器

| Collection             | 用途                      |
| ---------------------- | ----------------------- |
| `amazon.aws`           | AWS EC2 / S3 / IAM      |
| `azure.azcollection`   | Azure 资源管理              |
| `google.cloud`         | GCP 资源管理                |
| `community.docker`     | Docker / Docker Compose |
| `kubernetes.core`      | Kubernetes 资源管理         |
| `community.kubernetes` | K8s 部署和管理               |

### 网络设备

| Collection | 用途 |
|------------|------|
| `cisco.ios` | Cisco IOS 设备 |
| `cisco.nxos` | Cisco NX-OS 设备 |
| `arista.eos` | Arista EOS 设备 |
| `junipernetworks.junos` | Juniper 设备 |

---

## 九、最佳实践

### 9.1 代码质量

| 实践 | 说明 |
|------|------|
| **使用 FQCN** | `ansible.builtin.yum` 而非 `yum` |
| **始终声明 `become`** | 显式声明权限提升 |
| **幂等设计** | 避免使用 `shell`/`command`，优先用声明式模块 |
| **命名所有 Task** | `name` 字段清晰描述意图 |
| **版本锁定** | `requirements.yml` 中指定版本号 |
| **使用 `ansible-lint`** | 代码静态检查 |

### 9.2 安全

| 实践 | 说明 |
|------|------|
| **Vault 加密** | 所有敏感数据必须用 Ansible Vault 加密 |
| **外部密钥管理** | 考虑集成 HashiCorp Vault / AWS Secrets Manager |
| **SSH 硬化** | 使用非 root 用户 + 密钥认证 |
| **不存储密码** | 使用 `--ask-vault-pass` 或 CI 密钥注入 |

### 9.3 测试

```bash
# 语法检查
ansible-playbook site.yml --syntax-check

# Dry Run
ansible-playbook site.yml --check --diff

# 使用 Molecule 测试角色
pip install molecule molecule-docker
cd roles/myrole
molecule init scenario
molecule test
```

### 9.4 性能优化

```ini
# ansible.cfg 性能配置
[defaults]
forks = 50                           # 并发数（默认 5）
host_key_checking = False            # 跳过 SSH 主机验证
gathering = smart                    # 智能收集 facts

[ssh_connection]
pipelining = True                    # 减少 SSH 连接数
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
```

```yaml
# Playbook 级优化
- name: 优化示例
  hosts: all
  gather_facts: no                   # 不需要时可关闭
  strategy: free                     # 自由策略，不等待最慢主机
```

---

## 十、与 CI/CD 集成

### GitHub Actions 示例

```yaml
# .github/workflows/ansible.yml
name: Ansible Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install Ansible
        run: pip install ansible

      - name: Install dependencies
        run: ansible-galaxy install -r requirements.yml

      - name: Syntax check
        run: ansible-playbook site.yml --syntax-check

      - name: Dry run
        run: ansible-playbook site.yml --check
        env:
          ANSIBLE_HOST_KEY_CHECKING: "False"

      - name: Deploy
        run: ansible-playbook site.yml
        env:
          VAULT_PASSWORD: ${{ secrets.VAULT_PASSWORD }}
```

---

## 十一、Docker 部署

### 11.1 Docker 方式运行 Ansible 控制节点

将 Ansible 控制节点容器化，实现环境隔离、版本锁定和可复现的自动化环境。

#### 基础 Dockerfile

```dockerfile
# Dockerfile
FROM python:3.12-slim

LABEL maintainer="ops-team"

# 安装系统依赖
RUN apt-get update && apt-get install -y --no-install-recommends \
        openssh-client \
        sshpass \
        git \
        curl \
    && rm -rf /var/lib/apt/lists/*

# 安装 Ansible + 常用 Collection
RUN pip install --no-cache-dir \
        ansible-core==2.18.0 \
        ansible-builder \
        ansible-lint

# 安装常用 Collections
RUN ansible-galaxy collection install \
        community.docker \
        community.general \
        ansible.posix \
        ansible.utils

# 工作目录
WORKDIR /ansible

# 入口
ENTRYPOINT ["ansible-playbook"]
CMD ["--help"]
```

构建并运行：

```bash
# 构建镜像
docker build -t ansible-control:latest .

# 运行 Ad-Hoc 命令
docker run --rm \
  -v $(pwd)/inventory:/ansible/inventory \
  -v ~/.ssh:/root/.ssh:ro \
  ansible-control:latest \
  ansible all -i inventory/hosts -m ping

# 运行 Playbook
docker run --rm \
  -v $(pwd):/ansible \
  -v ~/.ssh:/root/.ssh:ro \
  ansible-control:latest \
  -i inventory/hosts site.yml

# 交互式进入容器调试
docker run --rm -it \
  -v $(pwd):/ansible \
  -v ~/.ssh:/root/.ssh:ro \
  --entrypoint /bin/bash \
  ansible-control:latest
```

#### Docker Compose 完整编排

```yaml
# docker-compose.yml
version: "3.8"

services:
  ansible-control:
    build:
      context: .
      dockerfile: Dockerfile
    image: ansible-control:latest
    container_name: ansible-control
    volumes:
      - ./playbooks:/ansible/playbooks:ro
      - ./inventory:/ansible/inventory:ro
      - ./roles:/ansible/roles:ro
      - ./group_vars:/ansible/group_vars:ro
      - ./host_vars:/ansible/host_vars:ro
      - ./secrets:/ansible/secrets:ro            # Vault 文件
      - ~/.ssh:/root/.ssh:ro                      # SSH 密钥
      - ./logs:/ansible/logs                       # 执行日志
    environment:
      - ANSIBLE_HOST_KEY_CHECKING=False
      - ANSIBLE_STDOUT_CALLBACK=yaml
      - ANSIBLE_LOG_PATH=/ansible/logs/ansible.log
    networks:
      - mgmt-net

networks:
  mgmt-net:
    driver: bridge
```

执行：

```bash
# 启动服务
docker compose up -d

# 在容器内执行 Playbook
docker compose exec ansible-control \
  ansible-playbook -i inventory/hosts playbooks/site.yml

# 使用 Vault
docker compose exec ansible-control \
  ansible-playbook -i inventory/hosts playbooks/site.yml \
  --vault-password-file /ansible/secrets/vault_pass
```

### 11.2 Ansible 执行环境 (Execution Environment)

Ansible 官方推荐的容器化方案，基于 `ansible-builder` 构建：

```bash
# 安装 ansible-builder
pip install ansible-builder
```

创建 `execution-environment.yml`：

```yaml
version: 3
build_arg_defaults:
  ANSIBLE_VERSION: "2.18.0"
dependencies:
  galaxy: requirements.yml           # Collection 依赖
  python: requirements.txt           # Python 包依赖
  system: bindep.txt                 # 系统包依赖
additional_build_steps:
  prepend_base:
    - RUN echo "Building EE..."
  append_final:
    - COPY --chmod=0755 scripts/entrypoint.sh /entrypoint.sh
    - ENTRYPOINT ["/entrypoint.sh"]
```

`requirements.yml`（Collection 依赖）：

```yaml
collections:
  - name: community.docker
    version: ">=3.0.0"
  - name: ansible.posix
  - name: ansible.utils
  - name: community.general
```

`requirements.txt`（Python 依赖）：

```text
boto3>=1.28.0
azure-mgmt-compute>=30.0.0
kubernetes>=28.0.0
docker>=6.0.0
```

`bindep.txt`（系统依赖）：

```text
git [platform:rpm]
openssh-clients [platform:rpm]
git [platform:dpkg]
openssh-client [platform:dpkg]
```

构建与使用：

```bash
# 构建执行环境镜像
ansible-builder build -f execution-environment.yml \
  -t my-ansible-ee:latest \
  -v 3

# 用 ansible-navigator 运行
pip install ansible-navigator
ansible-navigator run site.yml \
  --execution-environment-image my-ansible-ee:latest \
  --mode stdout

# 直接用 Docker 运行
docker run --rm \
  -v $(pwd):/ansible \
  -v ~/.ssh:/home/runner/.ssh:ro \
  my-ansible-ee:latest \
  ansible-playbook -i inventory/hosts site.yml
```

### 11.3 Ansible 管理 Docker 容器

使用 `community.docker` Collection 从 Ansible 直接管理 Docker：

```bash
# 安装 Collection
ansible-galaxy collection install community.docker
```

```yaml
# playbooks/docker-deploy.yml
- name: 部署 Docker 容器化应用
  hosts: dockerservers
  become: yes
  vars:
    app_image: "nginx:latest"
    app_name: "my-web"
    app_port: 8080

  tasks:
    # 安装 Docker
    - name: 安装 Docker
      ansible.builtin.include_role:
        name: geerlingguy.docker

    # 拉取镜像
    - name: 拉取应用镜像
      community.docker.docker_image:
        name: "{{ app_image }}"
        source: pull

    # 启动容器
    - name: 启动 Web 容器
      community.docker.docker_container:
        name: "{{ app_name }}"
        image: "{{ app_image }}"
        state: started
        restart_policy: always
        published_ports:
          - "{{ app_port }}:80"
        volumes:
          - "/data/html:/usr/share/nginx/html:ro"
        env:
          NGINX_PORT: "80"
        labels:
          managed_by: "ansible"
          environment: "production"
        healthcheck:
          test: ["CMD", "curl", "-f", "http://localhost/"]
          interval: 30s
          timeout: 10s
          retries: 3

    # Docker Compose 编排
    - name: 部署 Compose 项目
      community.docker.docker_compose_v2:
        project_src: /opt/myapp/
        state: present
        pull: always            # 始终拉取最新镜像
        recreate: always        # 镜像变更时重建容器
      register: compose_result

    - name: 显示部署结果
      ansible.builtin.debug:
        var: compose_result

    # 管理容器生命周期
    - name: 停止旧版容器
      community.docker.docker_container:
        name: "my-app-v1"
        state: absent

    # 清理无用镜像
    - name: 清理悬空镜像
      community.docker.docker_prune:
        images: yes
        images_filters:
          dangling: true
```

---

## 十二、Web UI 管理

### 方案对比

| 平台 | 类型 | 适用场景 | 复杂度 |
|------|------|---------|--------|
| **AWX** | 开源 | 企业级、团队协作 | ★★★★☆ |
| **Ansible Semaphore** | 开源 | 个人/小团队、轻量级 | ★★☆☆☆ |
| **Ansible Automation Platform** | 商业 | 大型企业、付费支持 | ★★★★★ |
| **Rundeck** | 开源 | 多工具编排、作业调度 | ★★★☆☆ |

### 12.1 AWX — 官方开源 Web UI

AWX 是 Ansible Tower 的上游开源项目，提供完整的 Web 管理界面。

#### Docker Compose 部署 AWX

```bash
# 克隆 AWX Operator 或 AWX 仓库
git clone https://github.com/ansible/awx.git
cd awx

# 使用官方安装脚本（推荐）
# 基于 docker-compose 的快速部署
curl -O https://raw.githubusercontent.com/ansible/awx/devel/tools/docker-compose/Makefile
```

推荐使用 **awx-operator** 在 K8s 上部署，或使用以下独立 Docker Compose 方案：

```yaml
# awx-docker-compose.yml
# ⚠️ AWX 官方推荐 K8s 部署，以下为简化独立方案
version: "3.8"

services:
  awx_web:
    image: ghcr.io/ansible/awx:24.6.1
    container_name: awx_web
    ports:
      - "8043:8043"        # HTTPS
      - "8080:8080"        # HTTP
    depends_on:
      - awx_task
      - awx_postgres
      - awx_redis
    environment:
      HOSTNAME: awx_web
      AWX_ADMIN_USER: admin
      AWX_ADMIN_PASSWORD: "{{ vault_awx_password }}"
      DATABASE_HOST: awx_postgres
      DATABASE_PORT: "5432"
      DATABASE_NAME: awx
      DATABASE_USER: awx
      DATABASE_PASSWORD: "{{ vault_db_password }}"
      REDIS_HOST: awx_redis
      SECRET_KEY: "{{ vault_secret_key }}"
    volumes:
      - awx_projects:/var/lib/awx/projects
      - awx_joblogs:/var/log/tower

  awx_task:
    image: ghcr.io/ansible/awx:24.6.1
    container_name: awx_task
    depends_on:
      - awx_postgres
      - awx_redis
    environment:
      HOSTNAME: awx_task
      SUPERVISOR_WEB_CHILD_PROCESS_NUM: "1"
      DATABASE_HOST: awx_postgres
      DATABASE_PORT: "5432"
      DATABASE_NAME: awx
      DATABASE_USER: awx
      DATABASE_PASSWORD: "{{ vault_db_password }}"
      REDIS_HOST: awx_redis
      SECRET_KEY: "{{ vault_secret_key }}"
    volumes:
      - awx_projects:/var/lib/awx/projects
      - awx_joblogs:/var/log/tower
      - /var/run/docker.sock:/var/run/docker.sock   # 调用 Docker

  awx_postgres:
    image: postgres:16-alpine
    container_name: awx_postgres
    environment:
      POSTGRES_DB: awx
      POSTGRES_USER: awx
      POSTGRES_PASSWORD: "{{ vault_db_password }}"
    volumes:
      - awx_db:/var/lib/postgresql/data

  awx_redis:
    image: redis:7-alpine
    container_name: awx_redis

volumes:
  awx_db:
  awx_projects:
  awx_joblogs:
```

#### AWX 核心功能

```
AWX Web UI
├── 📋 仪表盘 (Dashboard)
│   ├── 作业执行概览
│   ├── 主机状态统计
│   └── 最近活动时间线
│
├── 📁 项目管理 (Projects)
│   ├── Git 仓库同步
│   ├── SVN / Mercurial / 本地目录
│   └── Playbook 版本追踪
│
├── 📦 清单管理 (Inventories)
│   ├── 静态清单
│   ├── 动态清单（AWS/Azure/GCP/VMware）
│   ├── 主机分组与变量
│   └── 智能主机过滤器
│
├── 🚀 作业模板 (Job Templates)
│   ├── 绑定 Project + Inventory
│   ├── 凭据管理（SSH / Vault / Cloud）
│   ├── 参数化执行（调查问卷 Survey）
│   ├── 定时调度（Schedules）
│   └── 工作流编排（Workflow）
│
├── 🔐 凭据管理 (Credentials)
│   ├── SSH 密钥
│   ├── Ansible Vault 密码
│   ├── AWS / Azure / GCP 凭据
│   ├── 自定义凭据类型
│   └── 外部密钥管理集成
│
└── 👥 权限控制 (RBAC)
    ├── 团队与组织
    ├── 角色权限分配
    └── 审计日志
```

#### AWX API 自动化

```bash
# 登录获取 Token
TOKEN=$(curl -sk -X POST https://awx.example.com/api/v2/tokens/ \
  -u admin:password | jq -r '.token')

# 获取所有作业模板
curl -sk -H "Authorization: Bearer $TOKEN" \
  https://awx.example.com/api/v2/job_templates/ | jq

# 触发执行
curl -sk -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"extra_vars": "{\"target\": \"web1\"}"}' \
  https://awx.example.com/api/v2/job_templates/12/launch/

# 查看执行结果
curl -sk -H "Authorization: Bearer $TOKEN" \
  https://awx.example.com/api/v2/jobs/42/stdout/ | jq -r '.content'
```

#### AWX 与 CI/CD 集成

```yaml
# .github/workflows/awx-deploy.yml
name: Trigger AWX Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: 触发 AWX 作业
        run: |
          RESPONSE=$(curl -sk -X POST \
            -H "Authorization: Bearer ${{ secrets.AWX_TOKEN }}" \
            -H "Content-Type: application/json" \
            -d "{\"extra_vars\": \"{\\\"branch\\\": \\\"${{ github.sha }}\\\"}\"}" \
            "${{ secrets.AWX_URL }}/api/v2/job_templates/${{ secrets.AWX_TEMPLATE_ID }}/launch/")
          echo "Job URL: $(echo $RESPONSE | jq -r '.url')"
```

### 12.2 Ansible Semaphore — 轻量级 Web UI

**Semaphore** 是一个轻量级的 Ansible Web UI，适合个人开发者和小团队，部署极简。

> **项目地址**: [github.com/ansible-semaphore/semaphore](https://github.com/ansible-semaphore/semaphore)

#### Docker Compose 部署

```yaml
# semaphore-docker-compose.yml
version: "3.8"

services:
  semaphore:
    image: semaphoreui/semaphore:latest
    container_name: semaphore
    ports:
      - "3000:3000"
    environment:
      SEMAPHORE_DB_DRIVER: mysql
      SEMAPHORE_DB_HOST: semaphore_db
      SEMAPHORE_DB_PORT: "3306"
      SEMAPHORE_DB_USER: semaphore
      SEMAPHORE_DB_PASS: semaphore_pass
      SEMAPHORE_DB_NAME: semaphore
      SEMAPHORE_ADMIN: admin
      SEMAPHORE_ADMIN_PASSWORD: "StrongAdminPass!"
      SEMAPHORE_ADMIN_EMAIL: admin@example.com
      SEMAPHORE_ADMIN_NAME: "System Admin"
      SEMAPHORE_ACCESS_KEY_ENCRYPTION: "your-32-char-encryption-key-here!"
    depends_on:
      - semaphore_db
    volumes:
      - ./inventory:/inventory:ro
      - ./playbooks:/playbooks:ro
      - ~/.ssh:/root/.ssh:ro
    restart: unless-stopped

  semaphore_db:
    image: mariadb:11
    container_name: semaphore_db
    environment:
      MYSQL_RANDOM_ROOT_PASSWORD: "yes"
      MYSQL_DATABASE: semaphore
      MYSQL_USER: semaphore
      MYSQL_PASSWORD: semaphore_pass
    volumes:
      - semaphore_db:/var/lib/mysql
    restart: unless-stopped

volumes:
  semaphore_db:
```

部署步骤：

```bash
# 1. 启动服务
docker compose -f semaphore-docker-compose.yml up -d

# 2. 访问 Web UI
# http://your-server:3000
# 默认账号: admin / StrongAdminPass!

# 3. 首次配置
# - 创建项目
# - 添加 Git 仓库（Playbook 源）
# - 添加 Inventory 文件
# - 添加 SSH 凭据
# - 创建并运行任务模板
```

#### Semaphore 核心功能

| 功能            | 说明                                |
| ------------- | --------------------------------- |
| **项目管理**      | 多项目隔离，每项目独立的 Inventory / Keys     |
| **任务模板**      | 绑定 Playbook + Inventory + 凭据，一键执行 |
| **密钥管理**      | SSH 密钥、登录密码的安全存储                  |
| **环境管理**      | 多环境支持（开发/测试/生产）                   |
| **Inventory** | 静态文件 / 动态脚本 / 手动添加                |
| **任务日志**      | 实时查看 Playbook 执行输出                |
| **定时任务**      | Cron 风格的调度执行                      |
| **API**       | RESTful API，可集成到 CI/CD            |
| **通知**        | 邮件 / Slack / Webhook 通知           |
| **多用户**       | 基础的 RBAC 权限管理                     |

#### Semaphore API 调用

```bash
# 登录
COOKIE=$(curl -s -c - http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"auth": "admin", "password": "StrongAdminPass!"}' \
  | grep semaphore | awk '{print $NF}')

# 获取项目列表
curl -s -b "semaphore=$COOKIE" \
  http://localhost:3000/api/projects | jq

# 触发任务
curl -s -b "semaphore=$COOKIE" \
  -X POST \
  -H "Content-Type: application/json" \
  http://localhost:3000/api/project/1/tasks \
  -d '{"template_id": 1, "debug": false}' | jq
```

### 12.3 方案选择建议

```
你的场景？
│
├── 个人 / 学习 / 小项目
│   └── ✅ Semaphore（轻量、5 分钟部署）
│
├── 小团队 (5-20 人)
│   └── ✅ Semaphore + Docker（够用、维护简单）
│
├── 中大型团队 (20+ 人)
│   └── ✅ AWX（RBAC、审计、工作流）
│
├── 已有 K8s 集群
│   └── ✅ AWX Operator（云原生部署）
│
└── 企业级 + 需要商业支持
    └── ✅ Ansible Automation Platform (AAP)
```

---

## 十三、生态与扩展

### 核心生态

| 工具 | 说明 |
|------|------|
| **Ansible Core** | 核心引擎（模块、CLI） |
| **Ansible Galaxy** | 社区 Role / Collection 市场 |
| **AWX** | Ansible Tower 的开源版本（Web UI） |
| **Ansible Semaphore** | 轻量级 Web UI，适合小团队 |
| **Ansible Automation Platform (AAP)** | Red Hat 企业版 |
| **ansible-navigator** | 新一代 CLI 工具 |
| **ansible-builder** | 构建执行环境 (EE) 镜像 |
| **Execution Environments** | 容器化执行环境 |
| **Event-Driven Ansible (EDA)** | 事件驱动自动化 |
| **Molecule** | 角色测试框架 |

### 常用 Galaxy 角色

```bash
# Nginx
ansible-galaxy install geerlingguy.nginx

# Docker
ansible-galaxy install geerlingguy.docker

# MySQL
ansible-galaxy install geerlingguy.mysql

# Node.js
ansible-galaxy install geerlingguy.nodejs
```

---

## 十四、常见问题排查

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| `UNREACHABLE` | SSH 连接失败 | 检查网络、密钥、防火墙 |
| `Permission denied` | 权限不足 | 添加 `become: yes` |
| `Module not found` | Collection 未安装 | `ansible-galaxy collection install` |
| 幂等性问题 | 使用 `shell` 模块 | 改用声明式模块或添加 `changed_when` |
| 执行缓慢 | 默认并发 5 | 调高 `forks` + 启用 `pipelining` |
| 变量未生效 | 优先级覆盖 | 检查变量优先级链 |
| Template 报错 | Jinja2 语法错误 | 使用 `ansible-playbook --syntax-check` |

调试技巧：

```bash
# 查看详细输出 (-v 到 -vvvv 四级)
ansible-playbook site.yml -vvv

# 查看变量值
ansible -m debug -a "var=hostvars[inventory_hostname]" all

# 交互式调试
- name: Debug breakpoint
  ansible.builtin.debug:
    var: my_variable
```

---

## 十五、学习路线

```
🟢 入门
  │  ├── 安装 Ansible，配置 SSH 免密
  │  ├── 编写 Inventory，运行 Ad-Hoc 命令
  │  └── 编写第一个 Playbook
  │
🟡 进阶
  │  ├── 掌握变量、模板 (Jinja2)、条件、循环
  │  ├── 使用 Roles 组织代码
  │  ├── Ansible Vault 加密敏感数据
  │  └── 动态 Inventory (AWS/Azure)
  │
🔴 高级
  │  ├── 自定义模块和插件
  │  ├── AWX / Ansible Tower 部署
  │  ├── Execution Environments 容器化
  │  ├── Event-Driven Ansible (EDA)
  │  └── Molecule 自动化测试
  │
⚫ 专家
     ├── 贡献 Collection 到社区
     ├── 性能调优大规模集群 (1000+ 节点)
     └── 与 Terraform / K8s 深度集成
```

---

## 十六、相关资源

- 📘 [Ansible 官方文档](https://docs.ansible.com/ansible/latest/)
- 📦 [Ansible Galaxy](https://galaxy.ansible.com/) — 社区角色和集合
- 💬 [Ansible Forum](https://forum.ansible.com/) — 社区讨论
- 📰 [Ansible Blog](https://www.ansible.com/blog) — 官方博客
- 📺 [Ansible YouTube](https://www.youtube.com/ansibleautomation) — 视频教程
- 🐙 [GitHub Repository](https://github.com/ansible/ansible) — 源代码
- 📋 [Ansible Examples](https://github.com/ansible/ansible-examples) — 官方示例
- 🖥️ [AWX](https://github.com/ansible/awx) — 官方开源 Web UI
- 🖥️ [Ansible Semaphore](https://github.com/ansible-semaphore/semaphore) — 轻量级 Web UI
- 🔧 [ansible-builder](https://github.com/ansible/ansible-builder) — 执行环境构建工具
- 🔧 [Molecule](https://github.com/ansible/molecule) — 角色测试框架
- 📰 [[基础环境/WinRM 与 Ansible Windows 自动化运维指南|WinRM 与 Ansible Windows 自动化运维指南]] — WinRM 配置 + Windows 管理 + Hyper-V 虚拟化
- 🔥 [[Firecracker microVM 介绍与使用指南]] — Firecracker 轻量级 microVM 虚拟化
- 🔥 [[Firecracker 生态与配合项目指南]] — Firecracker 生态项目与集成方案

---

## 十七、Firecracker microVM 自动化部署与管理

> [!info] 前置阅读
> - [[Firecracker microVM 介绍与使用指南]] — Firecracker 基础概念、架构、API 与操作
> - [[Firecracker 生态与配合项目指南]] — 生态项目全景与选型指南

**Firecracker** 是 AWS 开源的轻量级虚拟机监视器（VMM），能在 <125ms 启动一个 microVM，提供 VM 级安全隔离和容器级速度。结合 Ansible 可以实现：

| 场景 | 说明 |
|------|------|
| **批量部署** | 在多台宿主机上一键安装 Firecracker + Jailer |
| **生命周期管理** | 通过 Ansible 调用 Firecracker API 创建/销毁/快照 microVM |
| **基础设施即代码** | 将 microVM 定义（内核、rootfs、网络、规格）全部 YAML 化 |
| **CI/CD 集成** | 与 AWX/Semaphore 集成，实现 Web 化的 microVM 管理平台 |

### 17.1 宿主机环境准备 Playbook

在一台或多台 Linux 宿主机上安装 Firecracker 运行环境：

```yaml
# playbooks/firecracker-setup.yml
- name: 部署 Firecracker 宿主机环境
  hosts: fc_hosts
  become: yes
  vars:
    firecracker_version: "v1.9.0"
    firecracker_install_dir: /usr/local/bin
    jailer_user: "fc-jailer"
    jailer_uid: 1234
    jailer_gid: 1234
    jailer_chroot_base: /srv/jailer

  tasks:
    # 检查 KVM 支持
    - name: 检查 KVM 支持
      ansible.builtin.stat:
        path: /dev/kvm
      register: kvm_check

    - name: 断言 KVM 可用
      ansible.builtin.assert:
        that: kvm_check.stat.exists
        fail_msg: "KVM 不可用！请确保 CPU 支持虚拟化并已加载 kvm 模块"

    # 安装系统依赖
    - name: 安装系统依赖
      ansible.builtin.apt:
        name:
          - curl
          - iproute2
          - iptables
          - bridge-utils
          - util-linux
        state: present
        update_cache: yes
      when: ansible_os_family == "Debian"

    # 下载 Firecracker 二进制
    - name: 下载 Firecracker
      ansible.builtin.get_url:
        url: "https://github.com/firecracker-microvm/firecracker/releases/download/{{ firecracker_version }}/firecracker-{{ firecracker_version }}-{{ ansible_architecture }}.tgz"
        dest: "/tmp/firecracker-{{ firecracker_version }}.tgz"
        mode: '0644'
      register: fc_download

    - name: 解压并安装 Firecracker
      ansible.builtin.unarchive:
        src: "/tmp/firecracker-{{ firecracker_version }}.tgz"
        dest: "/tmp/"
        remote_src: yes
      when: fc_download.changed

    - name: 安装 Firecracker 二进制
      ansible.builtin.copy:
        src: "/tmp/release-{{ firecracker_version }}-{{ ansible_architecture }}/firecracker-{{ firecracker_version }}-{{ ansible_architecture }}"
        dest: "{{ firecracker_install_dir }}/firecracker"
        mode: '0755'
        remote_src: yes

    - name: 安装 Jailer 二进制
      ansible.builtin.copy:
        src: "/tmp/release-{{ firecracker_version }}-{{ ansible_architecture }}/jailer-{{ firecracker_version }}-{{ ansible_architecture }}"
        dest: "{{ firecracker_install_dir }}/jailer"
        mode: '0755'
        remote_src: yes

    # 创建 Jailer 用户
    - name: 创建 Jailer 专用用户
      ansible.builtin.user:
        name: "{{ jailer_user }}"
        uid: "{{ jailer_uid }}"
        group: "kvm"
        shell: /usr/sbin/nologin
        system: yes
        create_home: no

    # 创建目录结构
    - name: 创建 Jailer chroot 基础目录
      ansible.builtin.file:
        path: "{{ jailer_chroot_base }}/firecracker"
        state: directory
        mode: '0755'

    # 验证安装
    - name: 验证 Firecracker 安装
      ansible.builtin.command: "{{ firecracker_install_dir }}/firecracker --version"
      register: fc_version
      changed_when: false

    - name: 显示安装结果
      ansible.builtin.debug:
        msg: "Firecracker 安装成功: {{ fc_version.stdout }}"
```

### 17.2 microVM 镜像准备 Playbook

批量准备 Guest 内核和 rootfs 镜像：

```yaml
# playbooks/firecracker-images.yml
- name: 准备 Firecracker 镜像资源
  hosts: fc_hosts
  become: yes
  vars:
    fc_images_dir: /opt/firecracker/images
    kernel_url: "https://s3.amazonaws.com/spec.ccfc.min/firecracker-ci/vmlinux-5.10.186"
    rootfs_size_mb: 500

  tasks:
    - name: 创建镜像目录
      ansible.builtin.file:
        path: "{{ fc_images_dir }}"
        state: directory
        mode: '0755'

    - name: 下载 Guest 内核
      ansible.builtin.get_url:
        url: "{{ kernel_url }}"
        dest: "{{ fc_images_dir }}/vmlinux-5.10"
        mode: '0644'

    - name: 创建 Alpine rootfs（基于 Docker）
      ansible.builtin.shell: |
        docker run --rm -v {{ fc_images_dir }}:/output alpine:latest \
          sh -c '
            mkdir -p /tmp/rootfs/{bin,sbin,etc,proc,sys,dev,tmp,lib,lib64,var,run}
            cp -a /bin/* /tmp/rootfs/bin/
            cp -a /sbin/* /tmp/rootfs/sbin/
            cp -a /lib/* /tmp/rootfs/lib/
            cp -a /lib64/* /tmp/rootfs/lib64/ 2>/dev/null || true
            cp /etc/passwd /tmp/rootfs/etc/
            cp /etc/group /tmp/rootfs/etc/
            echo "root:x:0:0:root:/root:/bin/sh" > /tmp/rootfs/etc/passwd
            dd if=/dev/zero of=/output/rootfs.ext4 bs=1M count={{ rootfs_size_mb }}
            mkfs.ext4 -F /output/rootfs.ext4
            mount -o loop /output/rootfs.ext4 /mnt
            cp -a /tmp/rootfs/* /mnt/
            echo "#!/bin/sh" > /mnt/init
            echo "mount -t proc proc /proc" >> /mnt/init
            echo "mount -t sysfs sysfs /sys" >> /mnt/init
            echo "mount -t devtmpfs devtmpfs /dev" >> /mnt/init
            echo "ip link set lo up" >> /mnt/init
            echo "ip link set eth0 up" >> /mnt/init
            echo "exec /bin/sh" >> /mnt/init
            chmod +x /mnt/init
            umount /mnt
          '
      args:
        creates: "{{ fc_images_dir }}/rootfs.ext4"

    - name: 验证镜像文件
      ansible.builtin.stat:
        path: "{{ fc_images_dir }}/{{ item }}"
      loop:
        - vmlinux-5.10
        - rootfs.ext4
      register: image_check

    - name: 显示镜像信息
      ansible.builtin.debug:
        msg: "{{ item.item }}: {{ (item.stat.size / 1024 / 1024) | round(1) }}MB"
      loop: "{{ image_check.results }}"
```

### 17.3 microVM 生命周期管理 Playbook

通过 Ansible 的 `uri` 模块调用 Firecracker REST API 管理 microVM：

```yaml
# playbooks/firecracker-vm-manage.yml
- name: 管理 Firecracker microVM
  hosts: fc_hosts
  become: yes
  vars:
    vm_id: "my-vm-001"
    socket_path: "/tmp/firecracker-{{ vm_id }}.socket"
    fc_images_dir: /opt/firecracker/images
    vm_vcpu: 2
    vm_mem_mib: 256
    tap_device: "tap-{{ vm_id }}"
    vm_guest_mac: "AA:FC:00:00:00:01"
    vm_ip: "172.16.0.2"
    tap_ip: "172.16.0.1"

  tasks:
    # ===== 创建 TAP 网络接口 =====
    - name: 创建 TAP 网络接口
      ansible.builtin.command:
        cmd: ip tuntap add dev {{ tap_device }} mode tap
      ignore_errors: yes
      changed_when: false

    - name: 配置 TAP 接口 IP
      ansible.builtin.command:
        cmd: ip addr add {{ tap_ip }}/24 dev {{ tap_device }}
      ignore_errors: yes
      changed_when: false

    - name: 启用 TAP 接口
      ansible.builtin.command:
        cmd: ip link set {{ tap_device }} up
      changed_when: false

    - name: 配置 NAT 转发
      ansible.builtin.iptables:
        table: nat
        chain: POSTROUTING
        out_interface: "{{ ansible_default_ipv4.interface }}"
        jump: MASQUERADE
      become: yes

    - name: 允许 TAP 转发
      ansible.builtin.iptables:
        chain: FORWARD
        in_interface: "{{ tap_device }}"
        out_interface: "{{ ansible_default_ipv4.interface }}"
        policy: ACCEPT

    # ===== 启动 Firecracker 进程 =====
    - name: 清理旧 Socket
      ansible.builtin.file:
        path: "{{ socket_path }}"
        state: absent

    - name: 启动 Firecracker VMM 进程
      ansible.builtin.shell: |
        nohup /usr/local/bin/firecracker \
          --api-sock {{ socket_path }} > /var/log/firecracker-{{ vm_id }}.log 2>&1 &
        echo $!
      register: fc_pid
      changed_when: true

    - name: 等待 API 就绪
      ansible.builtin.wait_for:
        path: "{{ socket_path }}"
        timeout: 5

    # ===== 配置 microVM =====
    - name: 设置 Guest 内核
      ansible.builtin.uri:
        url: "http://localhost/boot-source"
        unix_socket: "{{ socket_path }}"
        method: PUT
        body_format: json
        body:
          kernel_image_path: "{{ fc_images_dir }}/vmlinux-5.10"
          boot_args: "console=ttyS0 reboot=k panic=1 pci=off"
        status_code: 204

    - name: 设置 RootFS 磁盘
      ansible.builtin.uri:
        url: "http://localhost/drives/rootfs"
        unix_socket: "{{ socket_path }}"
        method: PUT
        body_format: json
        body:
          drive_id: rootfs
          path_on_host: "{{ fc_images_dir }}/rootfs.ext4"
          is_root_device: true
          is_read_only: false
        status_code: 204

    - name: 配置网络接口
      ansible.builtin.uri:
        url: "http://localhost/network-interfaces/eth0"
        unix_socket: "{{ socket_path }}"
        method: PUT
        body_format: json
        body:
          iface_id: eth0
          host_dev_name: "{{ tap_device }}"
          guest_mac: "{{ vm_guest_mac }}"
        status_code: 204

    - name: 设置虚拟机规格
      ansible.builtin.uri:
        url: "http://localhost/machine-config"
        unix_socket: "{{ socket_path }}"
        method: PUT
        body_format: json
        body:
          vcpu_count: "{{ vm_vcpu }}"
          mem_size_mib: "{{ vm_mem_mib }}"
        status_code: 204

    # ===== 启动虚拟机 =====
    - name: 启动 microVM
      ansible.builtin.uri:
        url: "http://localhost/actions"
        unix_socket: "{{ socket_path }}"
        method: PUT
        body_format: json
        body:
          action_type: InstanceStart
        status_code: 204

    - name: 显示 microVM 信息
      ansible.builtin.debug:
        msg: |
          ✅ microVM '{{ vm_id }}' 已启动！
          vCPU: {{ vm_vcpu }} | 内存: {{ vm_mem_mib }}MB
          Guest IP: {{ vm_ip }} | PID: {{ fc_pid.stdout }}
          日志: /var/log/firecracker-{{ vm_id }}.log
```

### 17.4 批量 microVM 编排

使用变量驱动批量创建多台 microVM：

```yaml
# playbooks/firecracker-batch.yml
- name: 批量创建 microVM 集群
  hosts: fc_hosts
  become: yes
  vars:
    fc_images_dir: /opt/firecracker/images
    microvms:
      - id: "web-001"
        vcpu: 2
        mem: 512
        ip: "172.16.1.10"
        mac: "AA:FC:01:00:00:01"
        rootfs: "rootfs-web.ext4"
      - id: "web-002"
        vcpu: 2
        mem: 512
        ip: "172.16.1.11"
        mac: "AA:FC:01:00:00:02"
        rootfs: "rootfs-web.ext4"
      - id: "db-001"
        vcpu: 4
        mem: 1024
        ip: "172.16.1.20"
        mac: "AA:FC:01:00:00:03"
        rootfs: "rootfs-db.ext4"
      - id: "cache-001"
        vcpu: 1
        mem: 256
        ip: "172.16.1.30"
        mac: "AA:FC:01:00:00:04"
        rootfs: "rootfs-cache.ext4"

  tasks:
    - name: 为每台 VM 准备独立 rootfs
      ansible.builtin.command:
        cmd: >
          cp {{ fc_images_dir }}/rootfs.ext4
          {{ fc_images_dir }}/{{ item.rootfs }}
      args:
        creates: "{{ fc_images_dir }}/{{ item.rootfs }}"
      loop: "{{ microvms }}"

    - name: 批量创建 microVM
      ansible.builtin.include_tasks: tasks/create-microvm.yml
      loop: "{{ microvms }}"
      loop_control:
        loop_var: vm
        label: "{{ vm.id }}"
```

`tasks/create-microvm.yml`（单台 VM 创建任务）：

```yaml
# tasks/create-microvm.yml
- name: "准备 {{ vm.id }} — 创建 TAP 接口"
  ansible.builtin.shell: |
    ip tuntap add dev tap-{{ vm.id }} mode tap 2>/dev/null || true
    ip addr add {{ vm.ip | regex_replace('\\.[0-9]+$', '.1') }}/24 dev tap-{{ vm.id }} 2>/dev/null || true
    ip link set tap-{{ vm.id }} up
  changed_when: false

- name: "准备 {{ vm.id }} — 清理旧 Socket"
  ansible.builtin.file:
    path: "/tmp/fc-{{ vm.id }}.socket"
    state: absent

- name: "启动 {{ vm.id }} — 启动 VMM 进程"
  ansible.builtin.shell: |
    nohup /usr/local/bin/firecracker \
      --api-sock /tmp/fc-{{ vm.id }}.socket \
      > /var/log/fc-{{ vm.id }}.log 2>&1 &
  changed_when: true

- name: "启动 {{ vm.id }} — 等待 API 就绪"
  ansible.builtin.wait_for:
    path: "/tmp/fc-{{ vm.id }}.socket"
    timeout: 5

- name: "配置 {{ vm.id }} — 内核 + 磁盘 + 网络 + 规格"
  ansible.builtin.uri:
    url: "http://localhost{{ item.path }}"
    unix_socket: "/tmp/fc-{{ vm.id }}.socket"
    method: PUT
    body_format: json
    body: "{{ item.body }}"
    status_code: 204
  loop:
    - path: "/boot-source"
      body:
        kernel_image_path: "{{ fc_images_dir }}/vmlinux-5.10"
        boot_args: "console=ttyS0 reboot=k panic=1 pci=off"
    - path: "/drives/rootfs"
      body:
        drive_id: rootfs
        path_on_host: "{{ fc_images_dir }}/{{ vm.rootfs }}"
        is_root_device: true
        is_read_only: false
    - path: "/network-interfaces/eth0"
      body:
        iface_id: eth0
        host_dev_name: "tap-{{ vm.id }}"
        guest_mac: "{{ vm.mac }}"
    - path: "/machine-config"
      body:
        vcpu_count: "{{ vm.vcpu }}"
        mem_size_mib: "{{ vm.mem }}"
  loop_control:
    label: "{{ item.path }}"

- name: "启动 {{ vm.id }} — InstanceStart"
  ansible.builtin.uri:
    url: "http://localhost/actions"
    unix_socket: "/tmp/fc-{{ vm.id }}.socket"
    method: PUT
    body_format: json
    body:
      action_type: InstanceStart
    status_code: 204
```

### 17.5 快照管理 Playbook

通过 Ansible 管理 Firecracker 快照（创建/恢复/批量克隆）：

```yaml
# playbooks/firecracker-snapshots.yml
- name: Firecracker 快照管理
  hosts: fc_hosts
  become: yes
  vars:
    snapshot_dir: /opt/firecracker/snapshots
    source_vm: "web-001"
    source_socket: "/tmp/fc-{{ source_vm }}.socket"
    clone_count: 3

  tasks:
    # ===== 创建快照 =====
    - name: 创建快照目录
      ansible.builtin.file:
        path: "{{ snapshot_dir }}/{{ source_vm }}"
        state: directory
        mode: '0755'

    - name: 对运行中的 microVM 创建快照
      ansible.builtin.uri:
        url: "http://localhost/snapshot/create"
        unix_socket: "{{ source_socket }}"
        method: PUT
        body_format: json
        body:
          snapshot_type: "Full"
          snapshot_path: "{{ snapshot_dir }}/{{ source_vm }}/vm-state"
          mem_file_path: "{{ snapshot_dir }}/{{ source_vm }}/vm-memory"
          version: "1.9.0"
        status_code: 204

    # ===== 批量从快照恢复（克隆） =====
    - name: 批量克隆 microVM
      ansible.builtin.include_tasks: tasks/restore-microvm.yml
      loop: "{{ range(1, clone_count + 1) | list }}"
      loop_control:
        loop_var: clone_idx

    - name: 显示克隆结果
      ansible.builtin.debug:
        msg: |
          ✅ 快照创建完成: {{ source_vm }}
          ✅ 已克隆 {{ clone_count }} 台 microVM
          快照位置: {{ snapshot_dir }}/{{ source_vm }}/
```

`tasks/restore-microvm.yml`：

```yaml
# tasks/restore-microvm.yml
- name: "克隆 #{{ clone_idx }} — 准备 rootfs 副本"
  ansible.builtin.command:
    cmd: >
      cp {{ snapshot_dir }}/{{ source_vm }}/vm-state
      {{ snapshot_dir }}/clone-{{ clone_idx }}-vm-state
    creates: "{{ snapshot_dir }}/clone-{{ clone_idx }}-vm-state"

- name: "克隆 #{{ clone_idx }} — 准备内存副本"
  ansible.builtin.command:
    cmd: >
      cp {{ snapshot_dir }}/{{ source_vm }}/vm-memory
      {{ snapshot_dir }}/clone-{{ clone_idx }}-vm-memory
    creates: "{{ snapshot_dir }}/clone-{{ clone_idx }}-vm-memory"

- name: "克隆 #{{ clone_idx }} — 启动新 VMM 进程"
  ansible.builtin.shell: |
    rm -f /tmp/fc-clone-{{ clone_idx }}.socket
    nohup /usr/local/bin/firecracker \
      --api-sock /tmp/fc-clone-{{ clone_idx }}.socket \
      > /var/log/fc-clone-{{ clone_idx }}.log 2>&1 &
  changed_when: true

- name: "克隆 #{{ clone_idx }} — 等待 API"
  ansible.builtin.wait_for:
    path: "/tmp/fc-clone-{{ clone_idx }}.socket"
    timeout: 5

- name: "克隆 #{{ clone_idx }} — 加载快照"
  ansible.builtin.uri:
    url: "http://localhost/snapshot/load"
    unix_socket: "/tmp/fc-clone-{{ clone_idx }}.socket"
    method: PUT
    body_format: json
    body:
      snapshot_path: "{{ snapshot_dir }}/clone-{{ clone_idx }}-vm-state"
      mem_file_path: "{{ snapshot_dir }}/clone-{{ clone_idx }}-vm-memory"
      enable_diff_snapshots: false
    status_code: 204

- name: "克隆 #{{ clone_idx }} — 恢复执行"
  ansible.builtin.uri:
    url: "http://localhost/actions"
    unix_socket: "/tmp/fc-clone-{{ clone_idx }}.socket"
    method: PUT
    body_format: json
    body:
      action_type: InstanceStart
    status_code: 204
```

### 17.6 推荐项目结构

```
ansible-firecracker/
├── ansible.cfg
├── inventory/
│   ├── hosts.ini                        # FC 宿主机清单
│   ├── group_vars/
│   │   └── fc_hosts.yml                 # 全局变量（版本、路径、网络）
│   └── host_vars/
│       └── host1.yml                    # 单台宿主机变量
├── playbooks/
│   ├── firecracker-setup.yml            # 宿主机环境安装
│   ├── firecracker-images.yml           # 镜像准备
│   ├── firecracker-vm-manage.yml        # 单台 VM 管理
│   ├── firecracker-batch.yml            # 批量 VM 编排
│   └── firecracker-snapshots.yml        # 快照管理
├── tasks/
│   ├── create-microvm.yml               # 单台 VM 创建任务
│   └── restore-microvm.yml              # 快照恢复任务
├── templates/
│   └── fc-service.json.j2               # microVM 配置模板
├── roles/
│   └── firecracker/                     # 可复用 Role
│       ├── defaults/main.yml
│       ├── tasks/
│       │   ├── install.yml
│       │   ├── configure-network.yml
│       │   └── create-vm.yml
│       ├── templates/
│       └── handlers/
└── group_vars/
    └── fc_hosts.yml
```

`inventory/group_vars/fc_hosts.yml` — 全局配置变量：

```yaml
# Firecracker 全局配置
firecracker_version: "v1.9.0"
firecracker_install_dir: /usr/local/bin
fc_images_dir: /opt/firecracker/images
fc_snapshots_dir: /opt/firecracker/snapshots
jailer_user: fc-jailer
jailer_uid: 1234
jailer_gid: 1234
jailer_chroot_base: /srv/jailer

# 网络配置
fc_subnet: "172.16.0.0/16"
fc_tap_base_ip: "172.16.0.1"

# 默认 microVM 规格
default_vcpu: 2
default_mem_mib: 256
```

`inventory/hosts.ini`：

```ini
[fc_hosts]
fc-host1 ansible_host=192.168.1.100
fc-host2 ansible_host=192.168.1.101

[fc_hosts:vars]
ansible_user=ubuntu
ansible_python_interpreter=/usr/bin/python3
```

### 17.7 Web UI 集成管理

将 Firecracker 管理集成到 Web UI 平台（AWX / Semaphore），实现可视化运维。

#### 方案一：AWX 集成

```
┌──────────────────────────────────────────────────────────┐
│                   AWX Web UI                             │
│  ┌─────────────────────────────────────────────────────┐│
│  │  📋 Firecracker 仪表盘 (自定义 Dashboard)           ││
│  │  ├── 运行中 microVM 数量                             ││
│  │  ├── 各宿主机资源使用率                               ││
│  │  └── 最近执行历史                                    ││
│  └─────────────────────────────────────────────────────┘│
│                                                          │
│  ┌─────────────────────────────────────────────────────┐│
│  │  🚀 作业模板 (Job Templates)                        ││
│  │                                                     ││
│  │  ┌──────────────────┐  ┌──────────────────────┐     ││
│  │  │ FC-Setup         │  │ FC-Create-VM         │     ││
│  │  │ 安装宿主机环境    │  │ 创建单台 microVM     │     ││
│  │  │ Survey: 无       │  │ Survey: vm_id, vcpu  │     ││
│  │  └──────────────────┘  │          mem, ip     │     ││
│  │                        └──────────────────────┘     ││
│  │  ┌──────────────────┐  ┌──────────────────────┐     ││
│  │  │ FC-Batch-Create  │  │ FC-Snapshot           │     ││
│  │  │ 批量创建 VM      │  │ 快照创建/恢复/克隆    │     ││
│  │  │ Survey: 配置文件 │  │ Survey: vm_id, action │     ││
│  │  └──────────────────┘  └──────────────────────┘     ││
│  │  ┌──────────────────┐  ┌──────────────────────┐     ││
│  │  │ FC-List-VMs      │  │ FC-Destroy-VM        │     ││
│  │  │ 列出所有 VM      │  │ 销毁指定 microVM     │     ││
│  │  │ Survey: host     │  │ Survey: vm_id        │     ││
│  │  └──────────────────┘  └──────────────────────┘     ││
│  └─────────────────────────────────────────────────────┘│
│                                                          │
│  ┌─────────────────────────────────────────────────────┐│
│  │  ⏰ 定时调度 (Schedules)                             ││
│  │  ├── 每日凌晨 2:00 — 清理过期 VM                    ││
│  │  ├── 每小时 — 健康检查 + 自动重启失败 VM            ││
│  │  └── 每周日 — 创建全量快照                           ││
│  └─────────────────────────────────────────────────────┘│
└──────────────────────────────────────────────────────────┘
```

在 AWX 中创建作业模板的配置：

```yaml
# AWX 作业模板配置示例（通过 API 或 Web UI 创建）
templates:
  - name: "FC-Create-VM"
    description: "创建一台 Firecracker microVM"
    playbook: playbooks/firecracker-vm-manage.yml
    inventory: "FC-Production"
    credential: "FC-SSH-Key"
    survey_enabled: true
    survey_spec:
      - variable: vm_id
        type: text
        required: true
        description: "microVM 标识符"
      - variable: vm_vcpu
        type: integer
        required: true
        default: 2
        description: "vCPU 数量"
      - variable: vm_mem_mib
        type: integer
        required: true
        default: 256
        description: "内存大小 (MB)"
      - variable: vm_ip
        type: text
        required: true
        description: "Guest IP 地址"

  - name: "FC-Snapshot"
    description: "快照管理（创建/恢复/克隆）"
    playbook: playbooks/firecracker-snapshots.yml
    survey_enabled: true
    survey_spec:
      - variable: source_vm
        type: text
        required: true
        description: "源 microVM 标识"
      - variable: clone_count
        type: integer
        default: 1
        min: 1
        max: 20
        description: "克隆数量"

  - name: "FC-Batch-Create"
    description: "批量创建 microVM 集群"
    playbook: playbooks/firecracker-batch.yml
    extra_vars:
      microvms: "{{ survey_microvms }}"
```

#### 方案二：Semaphore 集成

Semaphore 更适合小团队的轻量管理：

```
┌─────────────────────────────────────────────────────┐
│              Semaphore Web UI                        │
│                                                      │
│  📁 项目: Firecracker 管理                           │
│  ├── 🔑 密钥: SSH-Key (宿主机连接)                   │
│  ├── 📋 Inventory: fc-hosts                          │
│  ├── 📦 仓库: git@repo:ansible-firecracker.git      │
│  │                                                   │
│  ├── 📝 任务模板:                                    │
│  │   ├── [FC-Setup]        安装宿主机环境             │
│  │   ├── [FC-Create-VM]    创建 microVM              │
│  │   ├── [FC-Batch]        批量创建                   │
│  │   ├── [FC-Snapshot]     快照管理                   │
│  │   ├── [FC-Destroy]      销毁 VM                    │
│  │   └── [FC-Status]       查看运行状态               │
│  │                                                   │
│  ├── ⏰ 定时任务:                                    │
│  │   ├── 每日 02:00 — 清理过期 VM                    │
│  │   └── 每周日 03:00 — 全量快照                     │
│  │                                                   │
│  └── 📜 执行历史: 所有操作的完整日志                  │
└─────────────────────────────────────────────────────┘
```

Semaphore 项目配置步骤：

```bash
# 1. 在 Semaphore 中创建项目 "Firecracker 管理"

# 2. 添加 Inventory
#    名称: fc-hosts
#    类型: Static
#    内容: 直接引用仓库中的 inventory/hosts.ini

# 3. 添加密钥
#    名称: FC-SSH-Key
#    类型: SSH Private Key
#    内容: ~/.ssh/id_firecracker

# 4. 添加 Git 仓库
#    URL: git@your-repo:ansible-firecracker.git
#    分支: main

# 5. 创建任务模板
#    模板名: FC-Create-VM
#    Playbook: playbooks/firecracker-vm-manage.yml
#    参数: vm_id=my-app vcpu=2 mem=512
```

#### 方案三：Flint + 自定义 Web Dashboard

针对需要更丰富 UI 交互的场景，使用 Flask/FastAPI 构建自定义管理面板：

```yaml
# playbooks/firecracker-dashboard.yml
- name: 部署 Firecracker 管理 Dashboard
  hosts: fc_dashboard
  become: yes
  vars:
    dashboard_dir: /opt/fc-dashboard
    dashboard_port: 8080

  tasks:
    - name: 安装 Python 依赖
      ansible.builtin.pip:
        name:
          - fastapi
          - uvicorn
          - jinja2
          - httpx
        virtualenv: "{{ dashboard_dir }}/venv"

    - name: 部署 Dashboard 代码
      ansible.builtin.git:
        repo: "https://github.com/your-org/fc-dashboard.git"
        dest: "{{ dashboard_dir }}/app"
        version: main

    - name: 创建 systemd 服务
      ansible.builtin.template:
        src: fc-dashboard.service.j2
        dest: /etc/systemd/system/fc-dashboard.service
        mode: '0644'
      notify: Restart Dashboard

    - name: 启动 Dashboard
      ansible.builtin.service:
        name: fc-dashboard
        state: started
        enabled: yes

  handlers:
    - name: Restart Dashboard
      ansible.builtin.service:
        name: fc-dashboard
        state: restarted
```

Dashboard 核心功能模块参考：

```
┌──────────────────────────────────────────────────────────────┐
│              Firecracker 管理 Dashboard                       │
│                                                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐   │
│  │  宿主机概览    │  │  VM 列表     │  │  快照管理        │   │
│  │              │  │              │  │                  │   │
│  │ • CPU 使用率  │  │ • 运行状态   │  │ • 创建快照       │   │
│  │ • 内存占用    │  │ • vCPU/内存  │  │ • 恢复快照       │   │
│  │ • VM 数量    │  │ • IP 地址    │  │ • 批量克隆       │   │
│  │ • KVM 状态   │  │ • 运行时长   │  │ • 定时快照策略   │   │
│  └──────────────┘  └──────────────┘  └──────────────────┘   │
│                                                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐   │
│  │  创建 VM     │  │  控制台      │  │  监控指标        │   │
│  │              │  │              │  │                  │   │
│  │ • 表单创建   │  │ • 串口输出   │  │ • CPU/内存图表   │   │
│  │ • YAML 导入  │  │ • 实时日志   │  │ • 网络吞吐       │   │
│  │ • 模板选择   │  │ • 历史日志   │  │ • 磁盘 I/O      │   │
│  └──────────────┘  └──────────────┘  └──────────────────┘   │
│                                                               │
│  后端: FastAPI → 调用 Ansible Playbook / Firecracker API     │
│  前端: Vue.js / React                                         │
└──────────────────────────────────────────────────────────────┘
```

### 17.8 监控集成 Playbook

将 Firecracker 指标接入 Prometheus + Grafana：

```yaml
# playbooks/firecracker-monitoring.yml
- name: 部署 Firecracker 监控
  hosts: fc_hosts
  become: yes
  vars:
    prometheus_config: /etc/prometheus/prometheus.yml
    metrics_script: /usr/local/bin/fc-metrics-collector.sh

  tasks:
    # 指标采集脚本（定时调用 Firecracker Metrics API）
    - name: 部署指标采集脚本
      ansible.builtin.copy:
        dest: "{{ metrics_script }}"
        mode: '0755'
        content: |
          #!/bin/bash
          # 采集所有运行中 microVM 的指标并暴露给 Prometheus
          METRICS_DIR="/var/lib/fc-metrics"
          mkdir -p "$METRICS_DIR"
          echo "# HELP fc_uptime_ms microVM uptime in milliseconds" > "$METRICS_DIR/metrics.prom"
          echo "# TYPE fc_uptime gauge" >> "$METRICS_DIR/metrics.prom"

          for socket in /tmp/fc-*.socket; do
            [ -S "$socket" ] || continue
            VM_ID=$(basename "$socket" .socket | sed 's/fc-//')
            METRICS=$(curl -s --unix-socket "$socket" 'http://localhost/metrics' 2>/dev/null)
            if [ -n "$METRICS" ]; then
              UPTIME=$(echo "$METRICS" | jq -r '.vmm.uptime_ns // 0')
              UPTIME_MS=$(( UPTIME / 1000000 ))
              echo "fc_uptime{vm=\"$VM_ID\"} $UPTIME_MS" >> "$METRICS_DIR/metrics.prom"
            fi
          done

    - name: 配置定时采集（Cron）
      ansible.builtin.cron:
        name: "Firecracker metrics collection"
        minute: "*/1"
        job: "{{ metrics_script }}"

    - name: 配置 Prometheus 采集目标
      ansible.builtin.blockinfile:
        path: "{{ prometheus_config }}"
        marker: "# {mark} ANSIBLE MANAGED - Firecracker"
        block: |
          - job_name: 'firecracker'
            scrape_interval: 15s
            static_configs:
              - targets:
                {% for host in groups['fc_hosts'] %}
                - '{{ hostvars[host].ansible_host }}:9090'
                {% endfor %}
                labels:
                  group: 'firecracker'
      when: inventory_hostname == groups['fc_hosts'][0]
```

### 17.9 完整部署工作流

```
┌─────────────────────────────────────────────────────────────┐
│              Firecracker + Ansible 完整工作流                  │
│                                                              │
│  Step 1: 环境准备                                            │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  ansible-playbook playbooks/firecracker-setup.yml      │ │
│  │  → 安装 Firecracker + Jailer                           │ │
│  │  → 创建用户、目录、权限                                  │ │
│  └────────────────────────────────────────────────────────┘ │
│                         ↓                                    │
│  Step 2: 镜像准备                                            │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  ansible-playbook playbooks/firecracker-images.yml     │ │
│  │  → 下载 Guest 内核                                     │ │
│  │  → 构建 rootfs 镜像                                    │ │
│  └────────────────────────────────────────────────────────┘ │
│                         ↓                                    │
│  Step 3: 监控部署                                            │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  ansible-playbook playbooks/firecracker-monitoring.yml │ │
│  │  → 部署指标采集脚本                                     │ │
│  │  → 配置 Prometheus + Grafana                           │ │
│  └────────────────────────────────────────────────────────┘ │
│                         ↓                                    │
│  Step 4: 创建 VM（两种方式）                                  │
│  ┌───────────────────┐  ┌────────────────────────────┐     │
│  │ CLI 方式          │  │ Web UI 方式               │     │
│  │ ansible-playbook  │  │ AWX/Semaphore 一键触发     │     │
│  │ firecracker-      │  │ → 选择模板                │     │
│  │ vm-manage.yml     │  │ → 填写参数                │     │
│  │ -e "vm_id=xxx"    │  │ → 点击执行                │     │
│  └───────────────────┘  └────────────────────────────┘     │
│                         ↓                                    │
│  Step 5: 日常运维（通过 Web UI 或 CLI）                       │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  • 查看运行状态         • 创建/恢复快照                  │ │
│  │  • 批量扩容             • 监控告警                      │ │
│  │  • 销毁 VM             • 日志审计                      │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 17.10 Jailer 生产部署 Playbook

生产环境必须使用 Jailer 安全沙箱启动 microVM：

```yaml
# playbooks/firecracker-jailer.yml
- name: 使用 Jailer 安全启动 microVM
  hosts: fc_hosts
  become: yes
  vars:
    vm_id: "prod-app-001"
    jailer_chroot_base: /srv/jailer
    jailer_uid: 1234
    jailer_gid: 1234
    fc_images_dir: /opt/firecracker/images
    vm_vcpu: 2
    vm_mem_mib: 512
    tap_device: "tap-{{ vm_id }}"
    guest_mac: "AA:FC:00:00:00:01"

  tasks:
    - name: 准备 Jailer chroot 环境
      ansible.builtin.file:
        path: "{{ jailer_chroot_base }}/firecracker/{{ vm_id }}/root"
        state: directory
        mode: '0755'

    - name: 使用 Jailer 启动 Firecracker
      ansible.builtin.shell: |
        jailer \
          --id {{ vm_id }} \
          --exec-file /usr/local/bin/firecracker \
          --uid {{ jailer_uid }} \
          --gid {{ jailer_gid }} \
          --chroot-base-dir {{ jailer_chroot_base }} \
          --cgroup-version 2 \
          --resource-limit "no-file=1024" \
          -- /run/firecracker.socket
      async: 86400
      poll: 0
      register: jailer_job

    - name: 等待 Jailer Socket 就绪
      ansible.builtin.wait_for:
        path: "{{ jailer_chroot_base }}/firecracker/{{ vm_id }}/root/run/firecracker.socket"
        timeout: 10

    - name: 配置 microVM（通过 Jailer chroot 内的 Socket）
      ansible.builtin.uri:
        url: "http://localhost{{ item.path }}"
        unix_socket: "{{ jailer_chroot_base }}/firecracker/{{ vm_id }}/root/run/firecracker.socket"
        method: PUT
        body_format: json
        body: "{{ item.body }}"
        status_code: 204
      loop:
        - path: "/boot-source"
          body:
            kernel_image_path: "/opt/firecracker/images/vmlinux-5.10"
            boot_args: "console=ttyS0 reboot=k panic=1 pci=off"
        - path: "/drives/rootfs"
          body:
            drive_id: rootfs
            path_on_host: "/opt/firecracker/images/rootfs.ext4"
            is_root_device: true
            is_read_only: true
        - path: "/network-interfaces/eth0"
          body:
            iface_id: eth0
            host_dev_name: "{{ tap_device }}"
            guest_mac: "{{ guest_mac }}"
        - path: "/machine-config"
          body:
            vcpu_count: "{{ vm_vcpu }}"
            mem_size_mib: "{{ vm_mem_mib }}"

    - name: 启动 microVM
      ansible.builtin.uri:
        url: "http://localhost/actions"
        unix_socket: "{{ jailer_chroot_base }}/firecracker/{{ vm_id }}/root/run/firecracker.socket"
        method: PUT
        body_format: json
        body:
          action_type: InstanceStart
        status_code: 204

    - name: 显示部署信息
      ansible.builtin.debug:
        msg: |
          🔒 microVM '{{ vm_id }}' 已通过 Jailer 安全启动
          chroot: {{ jailer_chroot_base }}/firecracker/{{ vm_id }}/root
          uid/gid: {{ jailer_uid }}/{{ jailer_gid }}
          vCPU: {{ vm_vcpu }} | 内存: {{ vm_mem_mib }}MB
```

### 17.11 常见问题与排查

| 问题                     | 原因                | 解决方案                                        |
| ---------------------- | ----------------- | ------------------------------------------- |
| `uri` 模块 Socket 连接失败   | Socket 路径错误或权限不足  | 检查 `unix_socket` 参数和文件权限                    |
| `Cannot open /dev/kvm` | 用户不在 kvm 组        | `usermod -aG kvm $USER`                     |
| 批量创建 TAP 冲突            | IP 地址重复           | 使用 Jinja2 计算 IP（`{{ base_ip + item_idx }}`） |
| 快照恢复版本不匹配              | Firecracker 版本不一致 | 确保 `version` 字段与安装版本一致                      |
| Jailer chroot 权限问题     | 文件未正确绑定挂载         | 检查 Jailer 日志，确保资源文件在 chroot 内可见             |
| VM 启动后无网络              | TAP + NAT 未配置     | 先运行网络配置 Task，确保 iptables 规则生效               |
| Ansible `uri` 超时       | Firecracker 进程未启动 | 先用 `wait_for` 确认 Socket 就绪                  |

### 17.12 Web UI 方案选择建议

```
你的 Firecracker 管理场景？
│
├── 个人 / 学习 / 少量 VM（< 10 台）
│   └── ✅ Ansible CLI 直接管理（无需 Web UI）
│
├── 小团队 (3-10 人) + 定期创建/销毁 VM
│   └── ✅ Semaphore（5 分钟部署，够用）
│
├── 中大型团队 + 生产环境 + 多租户
│   └── ✅ AWX（RBAC 权限、审计日志、工作流）
│
├── 需要自定义 Dashboard 和丰富交互
│   └── ✅ FastAPI + Vue/React 自建（完全定制）
│
└── 已有 Kubernetes 集群
    └── ✅ Flintlock + Liquid Metal（云原生编排）
        详见 [[Firecracker 生态与配合项目指南]]
```
