---
title: "ESXi 8.0.2 或更高版本上 SSH 服务器的可配置属性"
source: "https://knowledge.broadcom.com/external/article?articleNumber=408319"
author:
published:
created: 2025-09-14
description:
tags:
  - "clippings"
---
ESXi 8.0.2 或更高版本上 SSH 服务器的可配置属性

搜索 取消

### ESXi 8.0.2 或更高版本上 SSH 服务器的可配置属性

书

#### 文章 ID： 408319

calendar\_today

#### 更新于： 08-26-2025

#### 产品

VMware vSphere ESXi

#### 问题/介绍

安装或升级到 ESX 8.0.2 或更高版本后，无法通过在“/etc/ssh/sshd\_config”文件中直接编辑来修改 SSH 服务器属性。

- 提供“esxcli”命令来列出 SSH 服务器属性并设置属性值。

#### 环境

VMware vSphere ESX 8.0 U2

#### 分辨率

从 8.0.2 开始，用户可以使用“esxcli 命令”修改 SSH 服务器配置的子集;因此，不再支持直接修改 /etc/ssh/sshd\_config 文件。

#### 列出可用和可配置的 SSH 属性

- `esxcli system ssh server config list`

#### 设置属性值

- `esxcli system ssh server config set -k <key/property name> -v <value>`

#### 附加信息

似乎没有 API 调用可用于使用 PowerCLI 编辑 ESX 服务器配置“密码身份验证”的值。

- [API Broadcom 开发人员文档](https://developer.broadcom.com/xapis?tab=App%2520Modernization)
	- [VMware vSphere 和 vSAN 命令索引](https://developer.broadcom.com/powercli/latest/products/vmwarevsphereandvsan/commands-index)

#### 反馈

本文对您有帮助吗？

thumb\_up 是的

thumb\_down 不