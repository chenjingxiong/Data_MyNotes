---
created: 2025年8月24日 15:51
title: Docker Compose搭建博客程序wordpress，并于已经部署好的另一个容器mariadb通信_网络存储_什么值得买
tags: 
source: https://post.smzdm.com/p/admdlx6k/
description: Docker Compose搭建博客程序wordpress，并于已经部署好的另一个容器mariadb通信。机器：绿联4600+mariadb已用Do
author: 浮生不得闲
published: 
---




#                                 Docker Compose搭建博客程序wordpress，并于已经部署好的另一个容器mariadb通信                            
2024-09-10 17:37:140点赞6收藏0评论
 **Docker Compose** 搭建博客程序 **wordpress** ，并于已经部署好的另一个容器 **mariadb** 通信。
 **机器： [绿联 ** 关注粉丝：

- 商品百科
- 好价
- 社区文章
 ](https://pinpai.smzdm.com/4769/)4600+** 
 **mariadb** 已用 **Docker Compose** 提前部署好，我想再创建一个博客程序 **wordpress** ，数据库采用已经部署好了的 **mariadb** ，难点在于 **wordpress** 要怎样与 **mariadb** 建立通信。
刚开始，我被难住了半天，明明 **yaml** 文件没错，数据库连接地址也没问题，为什么数据库就连不上。后来发现是 **mariadb** 定义的网络名称引起的。
首先，要 **SSH** 下查看 **mariadb** 所在的网络。
我采用的是 **WindTerm** ， **SSH** 登陆 ** [NAS](https://www.smzdm.com/ju/sp3qr1k/) ** ， ** *sudo -i* ** 输入管理员密码后换成 ** root**  账号，输入命令： ** *docker network ls* ** 指令查询所有容器的网络，在列出来的容器网络里找到 **mariadb** 网络名称。
我在部署 **mariad** b时， ** *networks* ** 填的是： **appnet** ，但在容器里却变成了 **mariadb_appnet** 。
经过了解，我才知道，在  **Docker Compose**  中，当我们定义服务并指定网络时， **Docker Compose**  会自动创建一个以服务名称和网络名称组合命名的网络。
比如我在前面容器  `mariadb`  的  `docker-compose.yml`  文件中定义了服务  `mariadb`  并将其连接到  `appnet`  网络， **Docker Compose**  可能就自动创建一个名为  `mariadb_appnet`  的网络，这就是服务名称和网络名称的组合。
当我们知道 **mariadb** 的网络后，yaml就好写了。
 **wordpress** 的 **yaml** 文件如下：
 `version: '3.8'  # 指定 Docker Compose 文件的版本

services:
  wordpress:
    container_name: wordpress  # 为 WordPress 容器指定一个名称
    image: wordpress:latest  # 使用 WordPress 的最新版镜像
    ports:
      - "8010:80"  # 将容器的 80 端口映射到宿 [主机](https://www.smzdm.com/ju/sp3rz02/) 的 8010 端口
    restart: always  # 总是重启容器，确保 WordPress 服务的高可用性
    environment:
      WORDPRESS_DB_HOST: mariadb:3306  # WordPress 将连接到名为 mariadb 的容器的 3306 端口
      WORDPRESS_DB_USER: root  # 数据库用户
      WORDPRESS_DB_NAME: wordpress  # 数据库名称
      WORDPRESS_DB_PASSWORD: 12345  # 数据库用户的密码
    volumes:
      - /volume1/docker/wordpress/data:/var/www/html  # 将宿主机的目录挂载到容器的 WordPress 安装目录
    networks:
      - mariadb_appnet  # 将 WordPress 服务连接到名为 mariadb_appnet 的网络

networks:
  mariadb_appnet:
    driver: bridge  # 定义一个桥接网络，允许服务之间相互通信` 


### 说明：

-  **服务（services）** ：定义了一个名为  `wordpress`  的服务。
-  **容器名称（container_name）** ：为容器指定了一个名称，方便管理和引用。
-  **镜像（image）** ：指定了服务使用的 Docker 镜像。
-  **端口映射（ports）** ：定义了容器端口和宿主机端口之间的映射关系。
-  **重启策略（restart）** ：设置了容器的重启策略， `always`  表示容器总是尝试重启。
-  **环境变量（environment）** ：用于配置服务的运行环境，如数据库连接信息。
-  **卷映射（volumes）** ：用于数据持久化，将宿主机的目录挂载到容器内部，确保数据不会在容器重启后丢失。
-  **网络（networks）** ：定义了服务之间的网络连接，使用桥接网络模式，允许服务之间相互通信。



### 注意事项：

-  **环境变量** ：确保  `WORDPRESS_DB_HOST`  中的  `mariadb:3306`  是正确的。如果 MariaDB 容器不叫  `mariadb` ，请相应地更改此值。
-  **卷映射** ：确保  `/volume1/docker/wordpress/data`  这个宿主机目录存在，并且 Docker 有权限访问它。
-  **网络** ：确保  `mariadb_appnet`  网络已经存在，并且 MariaDB 容器也已经连接到这个网络。

 **哦，对了，Docker Compose部署好wordpress后，记得将整个docker，或者mariadb容器重启下，不然的话，可能打不开网页。**  [
![Docker Compose搭建博客程序wordpress，并于已经部署好的另一个容器mariadb通信](https://am.zdmimg.com/202409/10/66e011484bf322399.png_e1080.jpg)  ](https://post.smzdm.com/p/admdlx6k/pic_2/) [
![Docker Compose搭建博客程序wordpress，并于已经部署好的另一个容器mariadb通信](https://am.zdmimg.com/202409/10/66e0111e3a8444274.png_e1080.jpg)  ](https://post.smzdm.com/p/admdlx6k/pic_3/) [
![Docker Compose搭建博客程序wordpress，并于已经部署好的另一个容器mariadb通信](https://am.zdmimg.com/202409/10/66e011489e9012399.png_e1080.jpg)  ](https://post.smzdm.com/p/admdlx6k/pic_4/)
![](https://res.smzdm.com/pc/pc_shequ/dist/img/the-end.png) 
文章很值，打赏犒劳作者一下“首席”打赏官正虚席以待！


#### 相关文章推荐

- 
- 

 [更多精彩文章](https://www.smzdm.com/fenlei/wangluocunchu/) 
 [更多精彩文章](https://www.smzdm.com/fenlei/wangluocunchu/) 