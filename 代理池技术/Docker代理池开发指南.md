# Docker 代理池开发完整指南

## 目录
1. [项目概述](#项目概述)
2. [GitHub 开源代理项目](#github-开源代理项目)
3. [项目结构](#项目结构)
4. [Docker 部署配置](#docker-部署配置)
5. [TUN 透明代理设置](#tun-透明代理设置)
6. [代理池管理系统](#代理池管理系统)
7. [测试与验证](#测试与验证)

---

## 项目概述

本文档介绍如何使用 Docker 构建一个完整的代理池系统，实现：
- **透明代理**: 通过 TUN 虚拟网卡为容器内所有进程提供代理
- **代理池管理**: 自动获取、测试、筛选可用代理
- **自动故障转移**: 代理失效时自动切换
- **监控面板**: 实时查看代理状态

---

## GitHub 开源代理项目

### 推荐的代理池项目

| 项目名 | Stars | 语言 | 特点 | 仓库地址 |
|--------|-------|------|------|----------|
| **proxy_pool** | 18k+ | Python | 最流行的代理池，支持爬虫验证 | `https://github.com/jhao104/proxy_pool` |
| **ProxyPool** | 5k+ | Python | 简洁易用，API 友好 | `https://github.com/Germey/ProxyPool` |
| **proxy_pool** | 2k+ | Python | Redis 存储，高性能 | `https://github.com/Python3WebSpider/ProxyPool` |
| **tun2socks** | 3k+ | Go | TUN 透明代理核心 | `https://github.com/xjasonlyu/tun2socks` |
| **v2rayA** | 8k+ | Go | Web GUI 代理工具 | `https://github.com/v2rayA/v2rayA` |
| **clash** | 55k+ | Go | 规则强大的代理工具 | `https://github.com/Dreamacro/clash` |

### 本项目选用方案
- **代理池核心**: 基于 `jhao104/proxy_pool` 二次开发
- **TUN 代理**: 使用 `xjasonlyu/tun2socks` 实现透明代理
- **API 服务**: Flask 提供 RESTful API
- **存储**: Redis 存储代理队列

---

## 项目结构

```
proxy-pool-docker/
├── docker-compose.yml           # Docker 编排配置
├── Dockerfile                    # 应用容器镜像
├── .env                          # 环境变量配置
├── requirements.txt              # Python 依赖
├── src/
│   ├── __init__.py
│   ├── config.py                 # 配置管理
│   ├── proxy_fetcher.py          # 代理获取模块
│   ├── proxy_validator.py        # 代理验证模块
│   ├── proxy_manager.py          # 代理池管理
│   ├── api_server.py             # API 服务
│   └── tun_setup.py              # TUN 设备配置
├── scripts/
│   ├── init_tun.sh               # TUN 初始化脚本
│   └── health_check.sh           # 健康检查脚本
├── tests/
│   ├── test_proxy.py             # 代理测试
│   └── test_tun.py               # TUN 测试
└── README.md
```

---

## Docker 部署配置

### 1. Dockerfile

```dockerfile
# syntax=docker/dockerfile:1.4
FROM ubuntu:22.04

LABEL maintainer="your-email@example.com"
LABEL description="Docker Proxy Pool with TUN transparent proxy"

# 设置非交互模式
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Asia/Shanghai

# 安装基础依赖
RUN apt-get update && apt-get install -y \
    python3.10 \
    python3-pip \
    python3-dev \
    redis-server \
    wget \
    curl \
    iproute2 \
    iptables \
    wireguard-tools \
    iputils-ping \
    net-tools \
    tzdata \
    && rm -rf /var/lib/apt/lists/*

# 设置时区
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# 安装 tun2socks (透明代理核心)
RUN wget -O /usr/local/bin/tun2socks \
    https://github.com/xjasonlyu/tun2socks/releases/download/v2.5.1/tun2socks-linux-amd64 \
    && chmod +x /usr/local/bin/tun2socks

# 创建工作目录
WORKDIR /app

# 复制项目文件
COPY requirements.txt .
RUN pip3 install --no-cache-dir -r requirements.txt

COPY src/ /app/src/
COPY scripts/ /app/scripts/

# 创建 TUN 设备所需的目录
RUN mkdir -p /var/run/tun && chmod 755 /app/scripts/*.sh

# 暴露端口
# 5010: Flask API
# 6379: Redis (仅内部)
# 7890: 代理端口
EXPOSE 5010 7890

# 设置启动脚本权限
RUN chmod +x /app/scripts/*.sh

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD /app/scripts/health_check.sh

# 启动命令
CMD ["/app/scripts/init_tun.sh"]
```

### 2. docker-compose.yml

```yaml
version: '3.8'

services:
  # Redis 数据存储
  redis:
    image: redis:7-alpine
    container_name: proxy-pool-redis
    restart: unless-stopped
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    networks:
      - proxy_network
    healthcheck:
      test: ["CMD", "redis-cli", "-a", "${REDIS_PASSWORD}", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  # 代理池主服务
  proxy-pool:
    build: .
    container_name: proxy-pool
    restart: unless-stopped
    privileged: true  # TUN 需要特权模式
    cap_add:
      - NET_ADMIN      # 网络管理权限
      - NET_RAW        # 原始套接字权限
    network_mode: "service:redis"  # 共享网络命名空间
    environment:
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - REDIS_PASSWORD=${REDIS_PASSWORD}
      - PROXY_API_PORT=5010
      - PROXY_PORT=7890
      - TUN_NAME=tun0
      - TUN_IP=10.0.0.1
      - FETCH_INTERVAL=300
      - VALIDATE_TIMEOUT=10
      - LOG_LEVEL=INFO
    volumes:
      - ./logs:/app/logs
      - ./config:/app/config
    depends_on:
      redis:
        condition: service_healthy
    ports:
      - "5010:5010"   # API 端口
      - "7890:7890"   # 代理端口

  # 可选: Web 监控面板
  proxy-dashboard:
    image: rediscommander/redis-commander:latest
    container_name: proxy-dashboard
    restart: unless-stopped
    environment:
      - REDIS_HOSTS=local:redis:6379:0:${REDIS_PASSWORD}
    ports:
      - "8081:8081"
    networks:
      - proxy_network
    depends_on:
      - redis

networks:
  proxy_network:
    driver: bridge

volumes:
  redis_data:
    driver: local
```

### 3. requirements.txt

```txt
# Web 框架
Flask==3.0.0
Flask-CORS==4.0.0

# HTTP 客户端
requests==2.31.0
httpx==0.25.2
aiohttp==3.9.1

# 数据库
redis==5.0.1

# HTML 解析
beautifulsoup4==4.12.2
lxml==4.9.3

# 异步支持
asyncio==3.4.3

# 工具库
pytz==2023.3
python-dateutil==2.8.2

# 日志
loguru==0.7.2

# 验证和测试
ping3==4.0.4

# 配置管理
python-dotenv==1.0.0
pyyaml==6.0.1
```

### 4. .env 环境变量

```bash
# Redis 配置
REDIS_PASSWORD=your_secure_password_here

# 代理池配置
PROXY_API_PORT=5010
PROXY_PORT=7890

# TUN 配置
TUN_NAME=tun0
TUN_IP=10.0.0.1
TUN_NETMASK=255.255.255.0

# 抓取配置
FETCH_INTERVAL=300
FETCHER_COUNT=5

# 验证配置
VALIDATE_TIMEOUT=10
VALIDATE_THREADS=50

# 日志配置
LOG_LEVEL=INFO
LOG_DIR=/app/logs
```

---

## TUN 透明代理设置

### 1. init_tun.sh - TUN 初始化脚本

```bash
#!/bin/bash
set -e

echo "🚀 初始化 TUN 透明代理..."

# 加载环境变量
source /app/src/config.py 2>/dev/null || true

# 配置变量
TUN_NAME="${TUN_NAME:-tun0}"
TUN_IP="${TUN_IP:-10.0.0.1}"
TUN_NETMASK="${TUN_NETMASK:-255.255.255.0}"
PROXY_PORT="${PROXY_PORT:-7890}"

# 创建 TUN 设备
echo "📡 创建 TUN 设备: $TUN_NAME"
ip tuntap add dev $TUN_NAME mode tun
ip addr add $TUN_IP/$TUN_NETMASK dev $TUN_NAME
ip link set dev $TUN_NAME up

# 配置路由规则 - 将流量转发到 TUN
echo "🔄 配置路由规则..."
ip route add 0.0.0.0/1 dev $TUN_NAME
ip route add 128.0.0.0/1 dev $TUN_NAME

# 启动 Redis
echo "🔴 启动 Redis..."
redis-server --daemonize yes --appendonly yes --requirepass $REDIS_PASSWORD

# 等待 Redis 就绪
sleep 2

# 启动 tun2socks (透明代理核心)
echo "🌐 启动 tun2socks..."
tun2socks \
    -device $TUN_NAME \
    -proxy "socks5://127.0.0.1:$PROXY_PORT" \
    -loglevel debug \
    &

# 启动本地 SOCKS5 代理 (连接到代理池)
echo "🔧 启动本地代理转发..."
python3 /app/src/proxy_forwarder.py &

# 启动代理池服务
echo "🎯 启动代理池管理服务..."
python3 /app/src/proxy_manager.py &

# 启动 API 服务
echo "📡 启动 API 服务..."
python3 /app/src/api_server.py &

# 等待所有服务启动
sleep 3

echo "✅ TUN 透明代理初始化完成!"
echo "   - TUN 设备: $TUN_NAME ($TUN_IP)"
echo "   - API 端口: $PROXY_API_PORT"
echo "   - 代理端口: $PROXY_PORT"
echo ""
echo "📊 健康检查:"
ip addr show $TUN_NAME

# 保持容器运行
wait
```

### 2. proxy_forwarder.py - 本地代理转发

```python
#!/usr/bin/env python3
"""
本地代理转发服务
监听本地 7890 端口，将请求转发到代理池中的代理
"""

import socket
import asyncio
from loguru import logger
from src.config import Config
from src.proxy_manager import ProxyManager


class ProxyForwarder:
    def __init__(self):
        self.config = Config()
        self.proxy_manager = ProxyManager()
        self.local_port = self.config.PROXY_PORT
        self.current_proxy = None

    async def get_proxy(self):
        """获取可用代理"""
        if self.current_proxy:
            return self.current_proxy

        proxy = await self.proxy_manager.get_best_proxy()
        if proxy:
            self.current_proxy = proxy
            logger.info(f"使用代理: {proxy}")
        return proxy

    async def handle_client(self, reader, writer):
        """处理客户端连接"""
        try:
            # 获取代理
            proxy = await self.get_proxy()
            if not proxy:
                writer.close()
                return

            # 连接到代理
            proxy_host, proxy_port = proxy.split(':')
            proxy_reader, proxy_writer = await asyncio.open_connection(
                proxy_host, int(proxy_port)
            )

            # 双向转发数据
            async def forward(client_reader, proxy_writer):
                try:
                    while True:
                        data = await client_reader.read(4096)
                        if not data:
                            break
                        proxy_writer.write(data)
                        await proxy_writer.drain()
                except Exception as e:
                    logger.error(f"转发错误: {e}")

            # 并发转发
            task1 = asyncio.create_task(forward(reader, proxy_writer))
            task2 = asyncio.create_task(forward(proxy_reader, writer))

            await asyncio.gather(task1, task2)

        except Exception as e:
            logger.error(f"处理客户端错误: {e}")
        finally:
            writer.close()
            await writer.wait_closed()

    async def start(self):
        """启动转发服务"""
        server = await asyncio.start_server(
            self.handle_client,
            '0.0.0.0',
            self.local_port
        )

        logger.info(f"代理转发服务启动: 0.0.0.0:{self.local_port}")

        async with server:
            await server.serve_forever()


if __name__ == '__main__':
    forwarder = ProxyForwarder()
    asyncio.run(forwarder.start())
```

### 3. health_check.sh - 健康检查脚本

```bash
#!/bin/bash

# 检查 TUN 设备
if ! ip addr show tun0 >/dev/null 2>&1; then
    echo "❌ TUN 设备不存在"
    exit 1
fi

# 检查代理端口
if ! nc -z 127.0.0.1 7890; then
    echo "❌ 代理端口不可用"
    exit 1
fi

# 检查 API 服务
if ! curl -s http://127.0.0.1:5010/health >/dev/null; then
    echo "❌ API 服务不可用"
    exit 1
fi

# 测试代理连接
if ! curl -x socks5://127.0.0.1:7890 -s --connect-timeout 5 https://www.google.com >/dev/null 2>&1; then
    echo "⚠️  代理连接测试失败 (可能是正常的)"
fi

echo "✅ 健康检查通过"
exit 0
```

---

## 代理池管理系统

### 1. config.py - 配置管理

```python
#!/usr/bin/env python3
"""
配置管理模块
"""

import os
from dotenv import load_dotenv

load_dotenv()


class Config:
    """全局配置类"""

    # Redis 配置
    REDIS_HOST = os.getenv('REDIS_HOST', 'redis')
    REDIS_PORT = int(os.getenv('REDIS_PORT', 6379))
    REDIS_PASSWORD = os.getenv('REDIS_PASSWORD', '')
    REDIS_DB = int(os.getenv('REDIS_DB', 0))

    # 代理配置
    PROXY_API_PORT = int(os.getenv('PROXY_API_PORT', 5010))
    PROXY_PORT = int(os.getenv('PROXY_PORT', 7890))

    # TUN 配置
    TUN_NAME = os.getenv('TUN_NAME', 'tun0')
    TUN_IP = os.getenv('TUN_IP', '10.0.0.1')
    TUN_NETMASK = os.getenv('TUN_NETMASK', '255.255.255.0')

    # 抓取配置
    FETCH_INTERVAL = int(os.getenv('FETCH_INTERVAL', 300))  # 秒
    FETCHER_COUNT = int(os.getenv('FETCHER_COUNT', 5))

    # 验证配置
    VALIDATE_TIMEOUT = int(os.getenv('VALIDATE_TIMEOUT', 10))
    VALIDATE_THREADS = int(os.getenv('VALIDATE_THREADS', 50))

    # 测试 URL
    TEST_URL = 'https://www.google.com'
    TEST_API = 'https://api.ipify.org?format=json'

    # 日志配置
    LOG_LEVEL = os.getenv('LOG_LEVEL', 'INFO')
    LOG_DIR = os.getenv('LOG_DIR', '/app/logs')

    # 代理源配置
    PROXY_SOURCES = [
        'https://www.proxy-list.download/api/v1/get?type=http',
        'https://proxylist.geonode.com/api/proxy-list',
        'https://api.proxyscrape.com/v2/?request=get&protocol=http',
    ]

    @classmethod
    def get_redis_url(cls):
        """获取 Redis 连接 URL"""
        return f"redis://:{cls.REDIS_PASSWORD}@{cls.REDIS_HOST}:{cls.REDIS_PORT}/{cls.REDIS_DB}"
```

### 2. proxy_fetcher.py - 代理获取模块

```python
#!/usr/bin/env python3
"""
代理获取模块
从各种免费代理源获取代理
"""

import asyncio
import aiohttp
from bs4 import BeautifulSoup
from loguru import logger
from typing import List
import re
from src.config import Config


class ProxyFetcher:
    """代理获取器"""

    def __init__(self):
        self.config = Config()
        self.sources = self.config.PROXY_SOURCES

    async def fetch_from_api(self, url: str) -> List[str]:
        """从 API 获取代理"""
        proxies = []
        try:
            async with aiohttp.ClientSession(timeout=aiohttp.ClientTimeout(total=10)) as session:
                async with session.get(url) as response:
                    if response.status == 200:
                        text = await response.text()
                        # 解析不同格式的代理列表
                        lines = text.strip().split('\n')
                        for line in lines:
                            # 匹配 IP:PORT 格式
                            match = re.match(r'(\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}):(\d+)', line)
                            if match:
                                ip, port = match.groups()
                                proxies.append(f"{ip}:{port}")
        except Exception as e:
            logger.error(f"从 {url} 获取代理失败: {e}")

        return proxies

    async def fetch_from_web(self, url: str) -> List[str]:
        """从网页抓取代理"""
        proxies = []
        try:
            async with aiohttp.ClientSession(timeout=aiohttp.ClientTimeout(total=10)) as session:
                async with session.get(url) as response:
                    if response.status == 200:
                        html = await response.text()
                        soup = BeautifulSoup(html, 'lxml')

                        # 查找所有包含 IP:PORT 的文本
                        for td in soup.find_all('td'):
                            text = td.get_text(strip=True)
                            match = re.match(r'(\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}):(\d+)', text)
                            if match:
                                ip, port = match.groups()
                                proxies.append(f"{ip}:{port}")
        except Exception as e:
            logger.error(f"从 {url} 抓取代理失败: {e}")

        return proxies

    async def fetch_all(self) -> List[str]:
        """从所有源获取代理"""
        all_proxies = []
        tasks = []

        for source in self.sources:
            if 'api' in source:
                tasks.append(self.fetch_from_api(source))
            else:
                tasks.append(self.fetch_from_web(source))

        results = await asyncio.gather(*tasks, return_exceptions=True)

        for result in results:
            if isinstance(result, list):
                all_proxies.extend(result)

        # 去重
        unique_proxies = list(set(all_proxies))
        logger.info(f"获取到 {len(unique_proxies)} 个唯一代理")

        return unique_proxies
```

### 3. proxy_validator.py - 代理验证模块

```python
#!/usr/bin/env python3
"""
代理验证模块
验证代理的可用性和速度
"""

import asyncio
import aiohttp
from loguru import logger
from typing import List, Dict, Tuple
import time
from src.config import Config


class ProxyValidator:
    """代理验证器"""

    def __init__(self):
        self.config = Config()
        self.test_url = self.config.TEST_URL
        self.test_api = self.config.TEST_API
        self.timeout = self.config.VALIDATE_TIMEOUT

    async def validate_proxy(self, proxy: str, session: aiohttp.ClientSession) -> Dict:
        """验证单个代理"""
        result = {
            'proxy': proxy,
            'valid': False,
            'response_time': None,
            'anonymous': False,
            'ip': None
        }

        proxy_url = f"http://{proxy}"

        try:
            start_time = time.time()

            # 测试连接
            async with session.get(
                self.test_url,
                proxy=proxy_url,
                timeout=aiohttp.ClientTimeout(total=self.timeout),
                allow_redirects=True
            ) as response:
                if response.status == 200:
                    result['valid'] = True
                    result['response_time'] = time.time() - start_time

                    # 检查匿名性
                    async with session.get(
                        self.test_api,
                        proxy=proxy_url,
                        timeout=aiohttp.ClientTimeout(total=5)
                    ) as ip_response:
                        if ip_response.status == 200:
                            data = await ip_response.json()
                            result['ip'] = data.get('ip')
                            result['anonymous'] = (result['ip'] != self.get_local_ip())

        except asyncio.TimeoutError:
            pass
        except Exception as e:
            pass

        return result

    def get_local_ip(self) -> str:
        """获取本地 IP"""
        import socket
        try:
            s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
            s.connect(("8.8.8.8", 80))
            ip = s.getsockname()[0]
            s.close()
            return ip
        except:
            return "127.0.0.1"

    async def validate_batch(self, proxies: List[str]) -> List[Dict]:
        """批量验证代理"""
        connector = aiohttp.TCPConnector(limit=self.config.VALIDATE_THREADS)

        valid_proxies = []

        async with aiohttp.ClientSession(connector=connector) as session:
            tasks = [self.validate_proxy(proxy, session) for proxy in proxies]
            results = await asyncio.gather(*tasks, return_exceptions=True)

            for result in results:
                if isinstance(result, dict) and result['valid']:
                    valid_proxies.append(result)

        logger.info(f"验证完成: {len(valid_proxies)}/{len(proxies)} 个代理可用")

        return valid_proxies
```

### 4. proxy_manager.py - 代理池管理

```python
#!/usr/bin/env python3
"""
代理池管理模块
管理代理的获取、验证、存储和分发
"""

import asyncio
import redis
import json
from loguru import logger
from typing import List, Dict, Optional
from datetime import datetime, timedelta
from src.config import Config
from src.proxy_fetcher import ProxyFetcher
from src.proxy_validator import ProxyValidator


class ProxyManager:
    """代理池管理器"""

    # Redis 键名
    VALID_PROXIES_KEY = 'proxy:valid'
    ALL_PROXIES_KEY = 'proxy:all'
    PROXY_STATS_KEY = 'proxy:stats'

    def __init__(self):
        self.config = Config()
        self.redis_client = redis.from_url(
            self.config.get_redis_url(),
            decode_responses=True
        )
        self.fetcher = ProxyFetcher()
        self.validator = ProxyValidator()

    def add_proxy(self, proxy: str, score: float = 100):
        """添加代理到有效池"""
        self.redis_client.zadd(self.VALID_PROXIES_KEY, {proxy: score})

    def remove_proxy(self, proxy: str):
        """移除代理"""
        self.redis_client.zrem(self.VALID_PROXIES_KEY, proxy)
        self.redis_client.zrem(self.ALL_PROXIES_KEY, proxy)

    def get_best_proxy(self) -> Optional[str]:
        """获取最佳代理 (分数最高)"""
        result = self.redis_client.zrevrange(self.VALID_PROXIES_KEY, 0, 0, withscores=True)
        if result:
            return result[0][0]
        return None

    def get_random_proxy(self) -> Optional[str]:
        """获取随机代理"""
        count = self.redis_client.zcard(self.VALID_PROXIES_KEY)
        if count > 0:
            import random
            index = random.randint(0, count - 1)
            result = self.redis_client.zrevrange(self.VALID_PROXIES_KEY, index, index)
            if result:
                return result[0]
        return None

    def get_proxies(self, count: int = 10) -> List[str]:
        """获取多个代理"""
        return self.redis_client.zrevrange(self.VALID_PROXIES_KEY, 0, count - 1)

    def update_proxy_score(self, proxy: str, delta: float):
        """更新代理分数"""
        self.redis_client.zincrby(self.VALID_PROXIES_KEY, delta, proxy)

    def mark_proxy_failed(self, proxy: str):
        """标记代理失败"""
        self.update_proxy_score(proxy, -10)
        score = self.redis_client.zscore(self.VALID_PROXIES_KEY, proxy)
        if score and score < 50:
            self.remove_proxy(proxy)
            logger.warning(f"移除低分代理: {proxy}")

    async def refresh_pool(self):
        """刷新代理池"""
        logger.info("开始刷新代理池...")

        # 获取新代理
        new_proxies = await self.fetcher.fetch_all()

        if not new_proxies:
            logger.warning("未获取到新代理")
            return

        # 验证代理
        valid_proxies = await self.validator.validate_batch(new_proxies)

        # 添加到池中
        for proxy_info in valid_proxies:
            proxy = proxy_info['proxy']
            score = 100

            # 根据响应时间调整分数
            if proxy_info['response_time']:
                score -= proxy_info['response_time'] * 10

            # 匿名代理加分
            if proxy_info['anonymous']:
                score += 20

            self.add_proxy(proxy, score)

        logger.info(f"代理池刷新完成: 新增 {len(valid_proxies)} 个有效代理")

    async def auto_refresh_loop(self):
        """自动刷新循环"""
        while True:
            try:
                await self.refresh_pool()
                await asyncio.sleep(self.config.FETCH_INTERVAL)
            except Exception as e:
                logger.error(f"自动刷新错误: {e}")
                await asyncio.sleep(60)

    def get_stats(self) -> Dict:
        """获取代理池统计"""
        valid_count = self.redis_client.zcard(self.VALID_PROXIES_KEY)
        all_count = self.redis_client.zcard(self.ALL_PROXIES_KEY)

        return {
            'valid_proxies': valid_count,
            'total_proxies': all_count,
            'last_refresh': datetime.now().isoformat(),
            'fetch_interval': self.config.FETCH_INTERVAL
        }


if __name__ == '__main__':
    # 启动代理池管理服务
    manager = ProxyManager()
    asyncio.run(manager.auto_refresh_loop())
```

### 5. api_server.py - API 服务

```python
#!/usr/bin/env python3
"""
API 服务模块
提供 RESTful API 访问代理池
"""

from flask import Flask, jsonify, request
from flask_cors import CORS
from loguru import logger
from src.config import Config
from src.proxy_manager import ProxyManager
import threading
import asyncio


app = Flask(__name__)
CORS(app)

config = Config()
proxy_manager = ProxyManager()


@app.route('/health', methods=['GET'])
def health_check():
    """健康检查"""
    return jsonify({
        'status': 'healthy',
        'timestamp': proxy_manager.get_stats()['last_refresh']
    })


@app.route('/api/proxy', methods=['GET'])
def get_proxy():
    """获取一个代理"""
    proxy_type = request.args.get('type', 'best')  # best, random

    if proxy_type == 'random':
        proxy = proxy_manager.get_random_proxy()
    else:
        proxy = proxy_manager.get_best_proxy()

    if proxy:
        return jsonify({
            'success': True,
            'proxy': proxy,
            'proxy_url': f'http://{proxy}'
        })
    else:
        return jsonify({
            'success': False,
            'error': 'No available proxy'
        }), 503


@app.route('/api/proxies', methods=['GET'])
def get_proxies():
    """获取多个代理"""
    count = min(int(request.args.get('count', 10)), 100)

    proxies = proxy_manager.get_proxies(count)

    return jsonify({
        'success': True,
        'count': len(proxies),
        'proxies': [{'proxy': p, 'url': f'http://{p}'} for p in proxies]
    })


@app.route('/api/stats', methods=['GET'])
def get_stats():
    """获取统计信息"""
    return jsonify(proxy_manager.get_stats())


@app.route('/api/refresh', methods=['POST'])
def refresh_pool():
    """手动刷新代理池"""
    # 在后台线程中运行异步任务
    def run_refresh():
        loop = asyncio.new_event_loop()
        asyncio.set_event_loop(loop)
        loop.run_until_complete(proxy_manager.refresh_pool())
        loop.close()

    thread = threading.Thread(target=run_refresh)
    thread.start()

    return jsonify({
        'success': True,
        'message': 'Proxy pool refresh started'
    })


@app.route('/api/report', methods=['POST'])
def report_proxy():
    """报告代理失败"""
    data = request.json
    proxy = data.get('proxy')

    if proxy:
        proxy_manager.mark_proxy_failed(proxy)
        return jsonify({
            'success': True,
            'message': f'Proxy {proxy} marked as failed'
        })
    else:
        return jsonify({
            'success': False,
            'error': 'Proxy not provided'
        }), 400


if __name__ == '__main__':
    logger.info(f"启动 API 服务: 端口 {config.PROXY_API_PORT}")
    app.run(host='0.0.0.0', port=config.PROXY_API_PORT, debug=False)
```

---

## 测试与验证

### 1. test_proxy.py - 代理测试

```python
#!/usr/bin/env python3
"""
代理功能测试
"""

import requests
import time
from loguru import logger


def test_api_connection():
    """测试 API 连接"""
    logger.info("测试 API 连接...")

    try:
        response = requests.get('http://localhost:5010/health', timeout=5)
        assert response.status_code == 200
        logger.info("✅ API 连接正常")
        return True
    except Exception as e:
        logger.error(f"❌ API 连接失败: {e}")
        return False


def test_get_proxy():
    """测试获取代理"""
    logger.info("测试获取代理...")

    try:
        response = requests.get('http://localhost:5010/api/proxy', timeout=5)
        data = response.json()

        assert data['success'] == True
        assert 'proxy' in data

        logger.info(f"✅ 获取代理成功: {data['proxy']}")
        return data['proxy']
    except Exception as e:
        logger.error(f"❌ 获取代理失败: {e}")
        return None


def test_proxy_connection(proxy):
    """测试代理连接"""
    logger.info(f"测试代理连接: {proxy}")

    try:
        proxies = {
            'http': f'http://{proxy}',
            'https': f'http://{proxy}'
        }

        start = time.time()
        response = requests.get(
            'https://api.ipify.org?format=json',
            proxies=proxies,
            timeout=10
        )
        elapsed = time.time() - start

        if response.status_code == 200:
            ip = response.json().get('ip')
            logger.info(f"✅ 代理连接成功! IP: {ip}, 耗时: {elapsed:.2f}s")
            return True
        else:
            logger.error(f"❌ 代理返回错误状态: {response.status_code}")
            return False
    except Exception as e:
        logger.error(f"❌ 代理连接失败: {e}")
        return False


def test_stats():
    """测试统计信息"""
    logger.info("测试统计信息...")

    try:
        response = requests.get('http://localhost:5010/api/stats', timeout=5)
        data = response.json()

        logger.info(f"📊 代理池统计:")
        logger.info(f"   - 有效代理: {data['valid_proxies']}")
        logger.info(f"   - 总代理数: {data['total_proxies']}")
        logger.info(f"   - 最后刷新: {data['last_refresh']}")

        return True
    except Exception as e:
        logger.error(f"❌ 获取统计失败: {e}")
        return False


def run_all_tests():
    """运行所有测试"""
    logger.info("=" * 50)
    logger.info("开始代理池测试")
    logger.info("=" * 50)

    results = {
        'api_connection': test_api_connection(),
        'get_proxy': test_get_proxy() is not None,
        'stats': test_stats()
    }

    # 获取代理并测试
    proxy = test_get_proxy()
    if proxy:
        results['proxy_connection'] = test_proxy_connection(proxy)

    # 总结
    logger.info("=" * 50)
    logger.info("测试总结:")
    passed = sum(1 for v in results.values() if v)
    total = len(results)
    logger.info(f"通过: {passed}/{total}")
    for name, result in results.items():
        status = "✅" if result else "❌"
        logger.info(f"  {status} {name}")
    logger.info("=" * 50)


if __name__ == '__main__':
    run_all_tests()
```

### 2. test_tun.py - TUN 测试

```python
#!/usr/bin/env python3
"""
TUN 透明代理测试
验证容器内流量是否通过代理
"""

import subprocess
import requests
import socket
from loguru import logger


def test_tun_device():
    """测试 TUN 设备是否存在"""
    logger.info("检查 TUN 设备...")

    try:
        result = subprocess.run(
            ['ip', 'addr', 'show', 'tun0'],
            capture_output=True,
            text=True
        )

        if result.returncode == 0:
            logger.info("✅ TUN 设备 tun0 存在")
            logger.info(result.stdout)
            return True
        else:
            logger.error("❌ TUN 设备不存在")
            return False
    except Exception as e:
        logger.error(f"❌ 检查 TUN 设备失败: {e}")
        return False


def test_proxy_port():
    """测试本地代理端口"""
    logger.info("测试本地代理端口 7890...")

    try:
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sock.settimeout(5)
        result = sock.connect_ex(('127.0.0.1', 7890))
        sock.close()

        if result == 0:
            logger.info("✅ 代理端口 7890 可访问")
            return True
        else:
            logger.error("❌ 代理端口 7890 不可访问")
            return False
    except Exception as e:
        logger.error(f"❌ 测试代理端口失败: {e}")
        return False


def test_transparent_proxy():
    """测试透明代理是否生效"""
    logger.info("测试透明代理...")

    try:
        # 通过代理端口请求
        proxies = {
            'http': 'socks5://127.0.0.1:7890',
            'https': 'socks5://127.0.0.1:7890'
        }

        response = requests.get(
            'https://api.ipify.org?format=json',
            proxies=proxies,
            timeout=10
        )

        if response.status_code == 200:
            ip = response.json().get('ip')
            logger.info(f"✅ 透明代理生效! 外部 IP: {ip}")

            # 获取本地 IP 进行对比
            try:
                local_ip = socket.gethostbyname(socket.gethostname())
                logger.info(f"   本地 IP: {local_ip}")
                if ip != local_ip:
                    logger.info("   ✅ IP 不同，确认通过代理")
                else:
                    logger.warning("   ⚠️ IP 相同，可能未使用代理")
            except:
                pass

            return True
        else:
            logger.error(f"❌ 请求失败: {response.status_code}")
            return False
    except Exception as e:
        logger.error(f"❌ 测试透明代理失败: {e}")
        return False


def test_route_table():
    """测试路由表配置"""
    logger.info("检查路由表...")

    try:
        result = subprocess.run(
            ['ip', 'route'],
            capture_output=True,
            text=True
        )

        routes = result.stdout
        logger.info("当前路由表:")
        logger.info(routes)

        # 检查是否有通过 tun0 的默认路由
        if 'tun0' in routes:
            logger.info("✅ 发现 tun0 路由")
            return True
        else:
            logger.warning("⚠️ 未发现 tun0 路由")
            return False
    except Exception as e:
        logger.error(f"❌ 检查路由表失败: {e}")
        return False


def run_all_tests():
    """运行所有 TUN 测试"""
    logger.info("=" * 50)
    logger.info("TUN 透明代理测试")
    logger.info("=" * 50)

    results = {
        'tun_device': test_tun_device(),
        'proxy_port': test_proxy_port(),
        'route_table': test_route_table(),
        'transparent_proxy': test_transparent_proxy()
    }

    # 总结
    logger.info("=" * 50)
    logger.info("测试总结:")
    passed = sum(1 for v in results.values() if v)
    total = len(results)
    logger.info(f"通过: {passed}/{total}")
    for name, result in results.items():
        status = "✅" if result else "❌"
        logger.info(f"  {status} {name}")
    logger.info("=" * 50)


if __name__ == '__main__':
    run_all_tests()
```

### 3. 部署和测试步骤

```bash
# 1. 构建镜像
docker-compose build

# 2. 启动服务
docker-compose up -d

# 3. 查看日志
docker-compose logs -f proxy-pool

# 4. 进入容器测试
docker-compose exec proxy-pool bash

# 在容器内运行测试
cd /app
python3 tests/test_proxy.py
python3 tests/test_tun.py

# 5. 测试 API
curl http://localhost:5010/health
curl http://localhost:5010/api/proxy
curl http://localhost:5010/api/stats

# 6. 手动刷新代理池
curl -X POST http://localhost:5010/api/refresh

# 7. 查看监控面板
# 浏览器访问: http://localhost:8081
```

---

## 常见问题排查

### 问题 1: TUN 设备创建失败
```bash
# 检查权限
docker-compose.yml 中确保有:
privileged: true
cap_add:
  - NET_ADMIN
  - NET_RAW
```

### 问题 2: 代理验证失败
```bash
# 调整超时时间
export VALIDATE_TIMEOUT=15

# 减少并发数
export VALIDATE_THREADS=20
```

### 问题 3: Redis 连接失败
```bash
# 检查 Redis 是否启动
docker-compose ps

# 查看 Redis 日志
docker-compose logs redis
```

### 问题 4: 代理获取量少
```bash
# 添加更多代理源
# 编辑 src/config.py 中的 PROXY_SOURCES

# 调整抓取间隔
export FETCH_INTERVAL=60  # 秒
```

---

## 扩展功能建议

1. **代理协议支持**: 添加 SOCKS5、HTTPS 代理支持
2. **地理位置过滤**: 根据国家/城市筛选代理
3. **白名单模式**: 只访问特定网站时使用代理
4. **代理轮换**: 每次请求自动切换代理
5. **性能监控**: 使用 Prometheus + Grafana 监控
6. **代理评分**: 根据稳定性、速度智能评分
7. **加密传输**: 使用 TLS 加密代理流量

---

## 许可证

MIT License

---

## 相关资源

- [Docker 官方文档](https://docs.docker.com/)
- [tun2socks 项目](https://github.com/xjasonlyu/tun2socks)
- [proxy_pool 项目](https://github.com/jhao104/proxy_pool)
- [Flask 文档](https://flask.palletsprojects.com/)
- [Redis 文档](https://redis.io/docs/)
