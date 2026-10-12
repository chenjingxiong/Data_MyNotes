---
title: Docker Compose搭建博客程序wordpress，并于已经部署好的另一个容器mariadb通信_网络存储_什么值得买
source: https://post.smzdm.com/p/admdlx6k/
author:
published: 2024-09-10
created: 2025-08-24
description: Docker Compose搭建博客程序wordpress，并于已经部署好的另一个容器mariadb通信。机器：绿联4600+mariadb已用Do
tags:
  - clippings
---
当前位置：

[社区首页](https://post.smzdm.com/)

[电脑数码](https://www.smzdm.com/fenlei/diannaoshuma/)

[存储设备](https://www.smzdm.com/fenlei/cunchushebei/)

[网络存储](https://www.smzdm.com/fenlei/wangluocunchu/)

文章详情

![](https://a.zdmimg.com/202409/10/66e011268f5f17556.png_fo742.jpg)

## Docker Compose搭建博客程序wordpress，并于已经部署好的另一个容器mariadb通信

**Docker Compose** 搭建博客程序 **wordpress** ，并于已经部署好的另一个容器 **mariadb** 通信。

**机器： [绿联](https://pinpai.smzdm.com/4769/)**

**[关注](https://pinpai.smzdm.com/4769/)**

**品牌

粉丝：

- 商品百科
- 好价
- 社区文章
**

**4600+**

**mariadb** 已用 **Docker Compose** 提前部署好，我想再创建一个博客程序 **wordpress** ，数据库采用已经部署好了的 **mariadb** ，难点在于 **wordpress** 要怎样与 **mariadb** 建立通信。

刚开始，我被难住了半天，明明 **yaml** 文件没错，数据库连接地址也没问题，为什么数据库就连不上。后来发现是 **mariadb** 定义的网络名称引起的。

首先，要 **SSH** 下查看 **mariadb** 所在的网络。

我采用的是 **WindTerm** ， **SSH** 登陆 **[NAS](https://www.smzdm.com/ju/sp3qr1k/)** ， ***sudo -i*** 输入管理员密码后换成 **root** 账号，输入命令： ***docker network ls*** 指令查询所有容器的网络，在列出来的容器网络里找到 **mariadb** 网络名称。

我在部署 **mariad** b 时， ***networks*** 填的是： **appnet** ，但在容器里却变成了 **mariadb\_appnet** 。

经过了解，我才知道， 在 **Docker Compose** 中，当我们定义服务并指定网络时， **Docker Compose** 会自动创建一个以服务名称和网络名称组合命名的网络。

比如我在前面容器 `mariadb` 的 `docker-compose.yml` 文件中定义了服务 `mariadb` 并将其连接到 `appnet` 网络， **Docker Compose** 可能就自动创建一个名为 `mariadb_appnet` 的网络，这就是服务名称和网络名称的组合。

当我们知道 **mariadb** 的网络后，yaml就好写了。

**wordpress** 的 **yaml** 文件如下：

`version: '3.8'  # 指定 Docker Compose 文件的版本  services:   wordpress:     container_name: wordpress  # 为 WordPress 容器指定一个名称     image: wordpress:latest  # 使用 WordPress 的最新版镜像     ports:       - "8010:80"  # 将容器的 80 端口映射到宿[主机](https://www.smzdm.com/ju/sp3rz02/)的 8010 端口     restart: always  # 总是重启容器，确保 WordPress 服务的高可用性     environment:       WORDPRESS_DB_HOST: mariadb:3306  # WordPress 将连接到名为 mariadb 的容器的 3306 端口       WORDPRESS_DB_USER: root  # 数据库用户       WORDPRESS_DB_NAME: wordpress  # 数据库名称       WORDPRESS_DB_PASSWORD: 12345  # 数据库用户的密码     volumes:       - /volume1/docker/wordpress/data:/var/www/html  # 将宿主机的目录挂载到容器的 WordPress 安装目录     networks:       - mariadb_appnet  # 将 WordPress 服务连接到名为 mariadb_appnet 的网络  networks:   mariadb_appnet:     driver: bridge  # 定义一个桥接网络，允许服务之间相互通信`

### 说明：

- **服务（services）** ：定义了一个名为 `wordpress` 的服务。
- **容器名称（container\_name）** ：为容器指定了一个名称，方便管理和引用。
- **镜像（image）** ：指定了服务使用的 Docker 镜像。
- **端口映射（ports）** ：定义了容器端口和宿主机端口之间的映射关系。
- **重启策略（restart）** ：设置了容器的重启策略， `always` 表示容器总是尝试重启。
- **环境变量（environment）** ：用于配置服务的运行环境，如数据库连接信息。
- **卷映射（volumes）** ：用于数据持久化，将宿主机的目录挂载到容器内部，确保数据不会在容器重启后丢失。
- **网络（networks）** ：定义了服务之间的网络连接，使用桥接网络模式，允许服务之间相互通信。

### 注意事项：

- **环境变量** ：确保 `WORDPRESS_DB_HOST` 中的 `mariadb:3306` 是正确的。如果 MariaDB 容器不叫 `mariadb` ，请相应地更改此值。
- **卷映射** ：确保 `/volume1/docker/wordpress/data` 这个宿主机目录存在，并且 Docker 有权限访问它。
- **网络** ：确保 `mariadb_appnet` 网络已经存在，并且 MariaDB 容器也已经连接到这个网络。

**哦，对了，Docker Compose部署好wordpress后，记得将整个docker，或者mariadb容器重启下，不然的话，可能打不开网页。**

[![Docker Compose搭建博客程序wordpress，并于已经部署好的另一个容器mariadb通信](https://am.zdmimg.com/202409/10/66e011484bf322399.png_e1080.jpg)](https://post.smzdm.com/p/admdlx6k/pic_2/) [![Docker Compose搭建博客程序wordpress，并于已经部署好的另一个容器mariadb通信](https://am.zdmimg.com/202409/10/66e0111e3a8444274.png_e1080.jpg)](https://post.smzdm.com/p/admdlx6k/pic_3/) [![Docker Compose搭建博客程序wordpress，并于已经部署好的另一个容器mariadb通信](https://am.zdmimg.com/202409/10/66e011489e9012399.png_e1080.jpg)](https://post.smzdm.com/p/admdlx6k/pic_4/)

- [docker](https://m.smzdm.com/theme/a5zokog/?zdm_feature=%7B%22fs%22%3A1%2C%22ta%22%3A1%7D)
	话题：docker
- [绿联nas](https://m.smzdm.com/theme/ao5vz14/?zdm_feature=%7B%22fs%22%3A1%2C%22ta%22%3A1%7D)
	话题：绿联nas
- [wordpress](https://m.smzdm.com/theme/aj21xww/?zdm_feature=%7B%22fs%22%3A1%2C%22ta%22%3A1%7D)
	话题：wordpress

文章很值，打赏犒劳作者一下

“首席”打赏官正虚席以待！## [浮生不得闲](https://zhiyou.smzdm.com/member/8474560793/)

[

5文章 | 2爆料 | 2粉丝

](https://zhiyou.smzdm.com/member/8474560793/)

- [![](https://qna.smzdm.com/202412/27/676e74ae1f1f69023.jpg_a320.jpg)](https://post.smzdm.com/p/ag523g3d/)
	[天钡 7 盘位 nas，预计年后发布](https://post.smzdm.com/p/ag523g3d/)
	0 0
	[天钡 7 盘位 nas，预计年后发布](https://post.smzdm.com/p/ag523g3d/)
- [![](https://a.zdmimg.com/202409/10/66dffa86da3bd8762.png_a320.jpg)](https://post.smzdm.com/p/akld83x8/)
	[Docker Compose搭建jellyfin影音库](https://post.smzdm.com/p/akld83x8/)
	2 0
	[Docker Compose搭建jellyfin影音库](https://post.smzdm.com/p/akld83x8/)
- [![](https://a.zdmimg.com/202409/10/66e004ea8e2795018.png_a320.jpg)](https://post.smzdm.com/p/a9687lmo/)
	[fnOS飞牛私有云Docker Compose搭建mariadb+ phpmyadmin](https://post.smzdm.com/p/a9687lmo/)
	3 13
	[fnOS飞牛私有云Docker Compose搭建mariadb+ phpmyadmin](https://post.smzdm.com/p/a9687lmo/)
- [![](https://qna.smzdm.com/202411/23/67416a4039a233651.jpg_a320.jpg)](https://post.smzdm.com/p/azo3vpv0/)
	[绿联 UGOS Pro 11 月份更新计划](https://post.smzdm.com/p/azo3vpv0/)
	0 0
	[绿联 UGOS Pro 11 月份更新计划](https://post.smzdm.com/p/azo3vpv0/)

- [![](https://a.zdmimg.com/202508/24/68aabd6b729175332.jpg_a320.jpg)](https://post.smzdm.com/p/azzl34oo/)
	[前情回顾：《会计刺客》是专注的天才魅力](https://post.smzdm.com/p/azzl34oo/)
	0 0
	[前情回顾：《会计刺客》是专注的天才魅力](https://post.smzdm.com/p/azzl34oo/)
- [![](https://a.zdmimg.com/202508/24/68aac165722995393.jpg_a320.jpg)](https://post.smzdm.com/p/ado52wpd/)
	[奔驰GLC大幅优惠引热议，理性消费成购车新趋势](https://post.smzdm.com/p/ado52wpd/)
	0 0
	[奔驰GLC大幅优惠引热议，理性消费成购车新趋势](https://post.smzdm.com/p/ado52wpd/)
- [![](https://a.zdmimg.com/202508/24/68aac16387a631374.jpg_a320.jpg)](https://post.smzdm.com/p/a0v5d799/)
	[岚图追光L官图发布，搭载华为乾崑智驾系统，成都车展即将亮相](https://post.smzdm.com/p/a0v5d799/)
	0 0
	[岚图追光L官图发布，搭载华为乾崑智驾系统，成都车展即将亮相](https://post.smzdm.com/p/a0v5d799/)
- [![](https://a.zdmimg.com/202508/24/68aabd3014c2b488.jpg_a320.jpg)](https://post.smzdm.com/p/an5ex40p/)
	[七夕甜蜜约会穿搭宝典：让心动从穿搭开始](https://post.smzdm.com/p/an5ex40p/)
	0 0
	[七夕甜蜜约会穿搭宝典：让心动从穿搭开始](https://post.smzdm.com/p/an5ex40p/)
- [![](https://a.zdmimg.com/202508/24/68aa2d67a722e6731.jpg_a320.jpg)](https://post.smzdm.com/p/ako60v99/)
	[告别手机存储焦虑，麦沃M2助你畅享自由创作](https://post.smzdm.com/p/ako60v99/)
	0 0
	[告别手机存储焦虑，麦沃M2助你畅享自由创作](https://post.smzdm.com/p/ako60v99/)
- [![](https://a.zdmimg.com/202508/20/68a5a564f919e6709.jpg_a320.jpg)](https://post.smzdm.com/p/a6pqxgxn/)
	[【投票有奖】家居清洁大对决你站哪一派？参与投票瓜分碎银！](https://post.smzdm.com/p/a6pqxgxn/)
	130 1234
	[【投票有奖】家居清洁大对决你站哪一派？参与投票瓜分碎银！](https://post.smzdm.com/p/a6pqxgxn/)
- [![](https://a.zdmimg.com/202508/17/68a1b4caab4b98471.jpg_a320.jpg)](https://post.smzdm.com/p/ax6w34g2/)
	[穷鬼天堂！这些东西“按原料买”，能省一个亿，贫穷激发出的潜力](https://post.smzdm.com/p/ax6w34g2/)
	113 55
	[穷鬼天堂！这些东西“按原料买”，能省一个亿，贫穷激发出的潜力](https://post.smzdm.com/p/ax6w34g2/)
- [![](https://a.zdmimg.com/202508/21/68a6d909571d82652.png_a320.jpg)](https://post.smzdm.com/p/al4vr3m0/)
	[股神上线！极空间NAS部署金融交易决策框架『TradingAgents-CN』](https://post.smzdm.com/p/al4vr3m0/)
	74 25
	[股神上线！极空间NAS部署金融交易决策框架『TradingAgents-CN』](https://post.smzdm.com/p/al4vr3m0/)
- [![](https://a.zdmimg.com/202508/19/68a3c67a9f95d4723.jpg_a320.jpg)](https://post.smzdm.com/p/a65m8lqg/)
	[老游戏不退出市场，年轻人还在玩，背后藏着啥猫腻.](https://post.smzdm.com/p/a65m8lqg/)
	91 231
	[老游戏不退出市场，年轻人还在玩，背后藏着啥猫腻.](https://post.smzdm.com/p/a65m8lqg/)
- [![](https://a.zdmimg.com/202508/23/68a9315b6a1e05060.jpg_a320.jpg)](https://post.smzdm.com/p/aogkdwk6/)
	[晒8小时太阳，续航增加100里！比亚迪把太阳能板，一年省3000油费](https://post.smzdm.com/p/aogkdwk6/)
	85 441
	[晒8小时太阳，续航增加100里！比亚迪把太阳能板，一年省3000油费](https://post.smzdm.com/p/aogkdwk6/)

- [![](https://qna.smzdm.com/202412/27/676e74ae1f1f69023.jpg_a320.jpg)](https://post.smzdm.com/p/ag523g3d/)
	[天钡 7 盘位 nas，预计年后发布](https://post.smzdm.com/p/ag523g3d/)
	0 0
	[天钡 7 盘位 nas，预计年后发布](https://post.smzdm.com/p/ag523g3d/)
- [![](https://a.zdmimg.com/202409/10/66dffa86da3bd8762.png_a320.jpg)](https://post.smzdm.com/p/akld83x8/)
	[Docker Compose搭建jellyfin影音库](https://post.smzdm.com/p/akld83x8/)
	2 0
	[Docker Compose搭建jellyfin影音库](https://post.smzdm.com/p/akld83x8/)
- [![](https://a.zdmimg.com/202409/10/66e004ea8e2795018.png_a320.jpg)](https://post.smzdm.com/p/a9687lmo/)
	[fnOS飞牛私有云Docker Compose搭建mariadb+ phpmyadmin](https://post.smzdm.com/p/a9687lmo/)
	3 13
	[fnOS飞牛私有云Docker Compose搭建mariadb+ phpmyadmin](https://post.smzdm.com/p/a9687lmo/)
- [![](https://qna.smzdm.com/202411/23/67416a4039a233651.jpg_a320.jpg)](https://post.smzdm.com/p/azo3vpv0/)
	[绿联 UGOS Pro 11 月份更新计划](https://post.smzdm.com/p/azo3vpv0/)
	0 0
	[绿联 UGOS Pro 11 月份更新计划](https://post.smzdm.com/p/azo3vpv0/)

已收藏  
[去我的收藏夹](https://post.smzdm.com/p/admdlx6k/#)

![](https://res.smzdm.com/pc/common/talent-report/talent-report-icon.png)

举报成功

该举报信息「小小值」会优先核实并处理哦！