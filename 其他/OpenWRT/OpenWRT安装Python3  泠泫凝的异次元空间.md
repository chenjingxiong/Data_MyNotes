---
title: "OpenWRT安装Python3 | 泠泫凝的异次元空间"
source: "https://lxnchan.cn/openwrt-python.html"
author:
  - "[[Lxn's Area仅剩下另一个世界怀3)r     var binft=function(r){var n=[\"《大切な人と再会できる日を》\"]]"
  - "[[\"有一种谎言，以死亡为真相\"]]"
  - "[[\"有一种寻花，以永别为代价\"]]"
  - "[[\"有一种毕业，以弑师为证书\"]]"
  - "[[\"为了每秒五厘米的樱花\"]]"
  - "[[\"我苦等多年\"]]"
  - "[[\"为了两千小时的记忆\"]]"
  - "[[\"我忍痛伴你\"]]"
  - "[[\"为了已被忘却的名字\"]]"
  - "[[\"我苦追千里\"]]"
published:
created: 2025-09-04
description: "科技、Linux、运维、云与虚拟化；研究有趣的新技术，分享运维操作，记录我的运维实战经验、问题和解决方案；敢于探索，勤于记录。"
tags:
  - "clippings"
---
OpenWRT安装Python3相关命令和依赖包

1. 首先确认剩余空间，opt文件夹要有足够的空间
```shell
root@XiaoQiang:~# df

Filesystem           1K-blocks      Used Available Use% Mounted on

rootfs                  130255    102809     20893  83% /

none                    126972         0    126972   0% /dev

tmpfs                   127580      6200    121380   5% /tmp

/dev/sda1               130255    102809     20893  83% /

tmpfs                   127580      6200    121380   5% /extdisks

/dev/sda3               267728     83116    171096  33% /data

/dev/sda4            975918408 489647556 486168452  50% /userdisk

/dev/sda3               267728     83116    171096  33% /etc

/dev/sda1               130255    102809     20893  83% /rom

/dev/sda4            975918408 489647556 486168452  50% /userdisk/appdata/-/lib

tmpfs                      128         0       128   0% /userdisk/appdata/-/tmp

/dev/sda4            975918408 489647556 486168452  50% /userdisk/appdata/-/bin

/dev/sda4            975918408 489647556 486168452  50% /userdisk/appdata/-/userdata

devtmpfs                126972         0    126972   0% /userdisk/appdata/-/dev

tmpfs                   127580      6200    121380   5% /etc/sysapihttpd/miwifi-webinitrd.conf

/dev/sda4            975918408 489647556 486168452  50% /opt
```
1. 更新包列表
```shell
root@XiaoQiang:~# opkg update
```
1. 安装主要组件
```shell
安装主要组件

root@Openwrt:/mnt# opkg install python3-base

root@Openwrt:~# opkg install python3-pip

root@Openwrt:~# opkg install python3

 换源（可选）

root@Openwrt:~# /usr/bin/python3 -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple  --upgrade pip
```
1. 依赖包（供离线安装参考）
```erlang
libffi.

libtirpc.

python3-light.

python3-email.

python3-urllib.

python3-pydoc.

python3-cgi.

python3-cgitb.

python3-decimal.

python-pip-conf.

libexpat.

python3-xml.

python3-unittest.

python3-ncurses.

python3-ctypes.

python3-logging.

python3-multiprocessing.

python3-readline.

libgdbm.

python3-gdbm.

libsqlite3.

python3-sqlite3.

python3-distutils.

python3-openssl.

libdb47.

python3-dbm.

python3-codecs.

python3-asyncio.

python3-lzma.

python3.

python3-pkg-resources.

python3-setuptools.

python3-pip.
```