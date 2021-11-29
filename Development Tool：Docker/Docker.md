# 一、Docker概述

具体参考：https://docs.docker.com/



# 二、安装及初步使用

## 1. 安装docker及启动

```shell
systemctl start docker.service 
```

  

## 2. 配置阿里云镜像

https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors



## 3. Docker run hello-world的流程

<img src="/Users/wenxuanan/Desktop/10concerto/NOTES/development tool：Docker/image-20211025205910946-5166986.png" alt="image-20211025205910946" style="zoom:50%;" />

## 4. 底层原理

Docker是一个Client-Server结构的系统，Docker的守护进程运行在主机上，通过Socket从客户端访问。DockerServer执行从Docker-Client接收到的命令。

<img src="/Users/wenxuanan/Desktop/10concerto/NOTES/Development Tool：Docker/image-20211025210650553.png" alt="image-20211025210650553" style="zoom: 50%;" />

## 5. Docker为什么比VM快？

- Docker有着比虚拟机更少的抽象层
- Docker 利用的是宿主机(物理机)的内核，VM需要的是Guest OS



# 三、命令使用

## 1. 帮助命令

```shell
docker version		# 显示docker的版本信息
docker info			# 显示docker的系统信息，包括镜像和容器的数量
docker 命令 --help	# 显示命令的帮助信息
```

帮助文档的地址：https://docs.docker.com/engine/reference -> Command-line reference



## 2. 镜像命令

```shell
docker images		# 查看本地主机上所有的镜像
docker images -aq	# 列出所有镜像的id

# 可选项
	-a, --all		# 列出所有镜像
	-q, --quiet		# 只显示镜像的id

~> docker images                                              (base)
REPOSITORY               TAG       IMAGE ID       CREATED        SIZE
docker/getting-started   latest    613921574f76   4 months ago   26.7MB

# 解释
REPOSITORY	镜像的仓库源
TAG			镜像的标签
IMAGE ID	镜像的ID
CREATED		镜像的创建时间
SIZE		镜像的大小

```



## 3. 搜索镜像

```shell
~> docker search mysql                                        (base)
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   11593     [OK]
mariadb                           MariaDB Server is a high performing open sou…   4409      [OK]
mysql/mysql-server                Optimized MySQL Server Docker images. Create…   857       [OK]

# 可选项，通过搜索来过滤
	--filter=STARS=3000	# 搜索结果即为STARS > 3000的
```



## 4. 下载镜像

```shell
docker pull 镜像名[:tag]	# 下载镜像

~> docker pull mysql	# 不加tag，默认下载 lastest
~ [1]> docker pull --platform linux/x86_64 mysql	# m1 mac 需要加--platform linux/x86_64
Using default tag: latest	# 如果不写tag就是latest
latest: Pulling from library/mysql
b380bbd43752: Pull complete	# 分层下载，docker image的核心 联合文件系统
f23cbf2ecc5d: Pull complete
30cfc6c29c0a: Pull complete
b38609286cbe: Pull complete
8211d9e66cd6: Pull complete
2313f9eeca4a: Pull complete
7eb487d00da0: Pull complete
4d7421c8152e: Pull complete
77f3d8811a28: Pull complete
cce755338cba: Pull complete
69b753046b9f: Pull complete
b2e64b0ab53c: Pull complete
Digest: sha256:6d7d4524463fe6e2b893ffc2b89543c81dec7ef82fb2020a1b27606666464d87
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest	# 真实文件地址

# 等价
docker pull mysql
docker pull docker.io/library/mysql:latest

# 指定版本下载
docker pull mysql:5.7
```



## 5. 删除镜像

```shell
```





















