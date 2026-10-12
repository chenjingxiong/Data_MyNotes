---
title: "阿里云qcow2镜像转vmdk，导入ESXi"
source: "https://www.cnblogs.com/tangjizhong/p/12559710.html"
author:
  - "[[唐际忠]]"
published: 2020-03-24
created: 2025-09-13
description: "简介： 阿里云的ecs服务器打包镜像后是qcow2格式，如果要将镜像导入到VMware Workstation Pro或者VMware ESXi，就需要将qcow2格式转成vmdk格式才可以。 笔者在此以ESXi 6.7导入为例来演示整个过程。 主要分以下几个步骤: 1、工具准备。下载并安装镜像格式"
tags:
  - "clippings"
---


## [阿里云qcow2镜像转vmdk，导入ESXi](https://www.cnblogs.com/tangjizhong/p/12559710.html "发布于 2020-03-24 16:21")

### 简介：

阿里云的ecs服务器打包镜像后是qcow2格式，如果要将镜像导入到VMware Workstation Pro或者VMware ESXi，就需要将qcow2格式转成vmdk格式才可以。

笔者在此以ESXi 6.7导入为例来演示整个过程。

主要分以下几个步骤:

1、工具准备。下载并安装镜像格式转换工具；

2、开始转换镜像格式；

```
qemu-img convert -f qcow2 -O vmdk aliyun-ecs.qcow2 aliyun-ecs.vmdk
```

3、上传镜像到ESXi存储；

4、二次转换，将vmdk转换成ESXi可认的格式；

```
	vmkfstools -i aliyun-ecs.vmdk -d thin out-aliyun-ecs.vmdk
```

5、新建虚拟机并导入