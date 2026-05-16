---
title: Ansible 自动化运维指南
date: 2026-05-16
tags:
  - ansible
  - automation
  - devops
  - infrastructure
  - configuration-management
source: https://github.comansible/ansible
---

# Ansible 自动化运维指南

> **项目地址**: [github.com/ansible/ansible](https://github.com/ansible/ansible)
> **官方文档**: [docs.ansible.com](https://docs.ansible.com/ansible/latest/)
> **许可证**: GPL v3.0 | **语言**: Python | **作者**: Michael DeHaan / Red Hat

---

## 一、什么是 Ansible

Ansible 是一个**极简的 IT 自动化系统**，由 Red Hat 维护，用于：

| 能力 | 说明 |
|------|------|
| **配置管理** | 统一管理服务器配置状态 |
| **应用部署** | 自动化应用的构建、发布、更新 |
| **云资源编排** | AWS / Azure / GCP / OpenStack 资源管理 |
| **任务自动化** | 日常运维任务的批量执行 |
| **网络自动化** | 网络设备的配置和管理 |
| **零停机更新** | 滚动更新 + 负载均衡协调 |

### 核心设计原则

1. **无代理 (Agentless)** — 不需要在目标机器上安装任何客户端，直接利用 SSH (Linux) / WinRM (Windows)
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

| 功能 | 说明 |
|------|------|
| **项目管理** | 多项目隔离，每项目独立的 Inventory / Keys |
| **任务模板** | 绑定 Playbook + Inventory + 凭据，一键执行 |
| **密钥管理** | SSH 密钥、登录密码的安全存储 |
| **环境管理** | 多环境支持（开发/测试/生产） |
| **Inventory** | 静态文件 / 动态脚本 / 手动添加 |
| **任务日志** | 实时查看 Playbook 执行输出 |
| **定时任务** | Cron 风格的调度执行 |
| **API** | RESTful API，可集成到 CI/CD |
| **通知** | 邮件 / Slack / Webhook 通知 |
| **多用户** | 基础的 RBAC 权限管理 |

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
