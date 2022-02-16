# 一、Docker概述

具体参考：https://docs.docker.com/



# 二、安装及初步使用

## 1. 安装docker及启动

```shell
$ systemctl start docker.service 
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
$ docker version		# 显示docker的版本信息
$ docker info			# 显示docker的系统信息，包括镜像和容器的数量
$ docker 命令 --help	# 显示命令的帮助信息
```

帮助文档的地址：https://docs.docker.com/engine/reference -> Command-line reference



## 2. 镜像命令

### Get started

```shell
$ docker images		# 查看本地主机上所有的镜像
$ docker images -aq	# 列出所有镜像的id

# 可选项
	-a, --all		# 列出所有镜像
	-q, --quiet		# 只显示镜像的id

$ docker images                                              (base)
REPOSITORY               TAG       IMAGE ID       CREATED        SIZE
docker/getting-started   latest    613921574f76   4 months ago   26.7MB

# 解释
REPOSITORY	镜像的仓库源
TAG			镜像的标签
IMAGE ID	镜像的ID
CREATED		镜像的创建时间
SIZE		镜像的大小

```

### 搜索镜像

```shell
$ docker search mysql                                        (base)
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   11593     [OK]
mariadb                           MariaDB Server is a high performing open sou…   4409      [OK]
mysql/mysql-server                Optimized MySQL Server Docker images. Create…   857       [OK]

# 可选项，通过搜索来过滤
	--filter=STARS=3000	# 搜索结果即为STARS > 3000的
```

### 下载镜像

```shell
$ docker pull 镜像名[:tag]	# 下载镜像

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
$ docker pull mysql
$ docker pull docker.io/library/mysql:latest

# 指定版本下载
$ docker pull mysql:5.7
```

### 删除镜像

```shell
$ docker rmi images

$ # docker rmi -f 镜像id    # 删除指定的id
$ # docker rmi -f 镜像id 镜像id 镜像id    # 删除多个指定的id
$ # docker rmi -f $(docker images -aq) # 删除全部镜像
	$(docker images -aq) 相当于把括号内命令的执行结果当做参数传进去
```



## 3.容器命令

### 新建容器并启动

```shell
$ docker run[可选参数〕image
# 参数说明
--name="Name"	指定容器名字
-d				后台方式运行
-it				使用交互方式运行，进入容器查看内容
-p				端口映射 指定容器的端口 -p 8080:8080
	-p ip:主机端口:容器端口
	-p 主机端口：容器端口（常用）
	-p 容器端口
-p 				随机指定端囗

# 测试，启动并进入容器
$ docker run -it centos /bin/bash

$ ls #查看容器内的centos，基础版本，很多命令都是不完善的！
bin dev etc home 1ib 1ib64 1ost+found media mnt opt proc root run shin srv svs var
tmo usr

# 坑
$ docker run -d centos
	# 问题docker ps，发现 centos 停止了
	# 常见的坑：docker 容器使用后台运行，就必须要有要一个前台进程，docker run -it centos /bin/bash 就是前台进程，可以使用docker ps查看一下有没有这个服务
	# docker发现没有应用，就会自动停止nginx，容器启动后，发现自己没有提供服务，就会立刻停止，就是没有程序了
	
# 用完即删，测试
$ docker run -i -rm container
```

### 列出所有容器

```shell
$ docker ps # 列出当前正在运行的容器
-a		# 列出当前正在运行的容器+带出历史运行过的容器
-n=?	# 显示最近创建的容器
-q 		# 只显示容器的编号
```

### 启动和停止容器

```shell
$ docker start 容器id		# 启动容器
$ docker restart 容器id	# 重启容器
$ docker stop 容器id		# 停止当前正在运行的容器
$ docker ki1l 容器id		# 强制停止当前容器
```

### 进入正在运行的容

```shell
$ docker exec -it container bashshell(bin/bash)	# 进入容器后开启一个新的终端，可以在里面操作（常用）
$ docker attach	container	# 进入容器正在执行的終端，不会启动新的进程！
	# Docker attach可以attach到一个已经运行的容器的stdin(标准输入，一般指键盘输入到缓冲区里的东西)，然后进行命令执行的动作。
	# 但是需要注意的是，如果从这个stdin中exit，会导致容器的停止。
```



## 4.容器与宿主机交互

### 拷贝文件

```shell
$ docker cp container:/home/test.java /home	# 从容器拷贝到宿主机
```



## 5.日志与信息

```shell
$ docker logs --help
```

### 查看容器中进程信息

```shell
$ docker top container
```

### 查看镜像的元数据

```shell
$ docker inspect container
```



# 四、镜像 联合文件系统

### UnionFs （ 联合文件系统）

UnionFS（联合文件系统）：Union文件系统(UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。 Union 文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录

### 镜像加载原理

docker的镜像实际上由一层一层的文件系统组成。  

bootfs(boot file system)主要包含 bootloader 和 kernel,bootloader 主要是引导加载 kernel, Linux 刚启动时会加载 bootfs 文件系统，在Docker镜像的最底层是 bootfs。这一层与我们典型的Linux/Unix系统是一样的，包含 boot 加载器和内核。当 boot 加载完成之后整个内核就都在内存中了，此时内存的使用权已由 bootfs 转交给内核，此时系统也会卸载 bootfs。

rootfs (root file system），在 bootfs 之上。包含的就是典型 Linux 系统中的 /dev,/proc,/bin,letc  等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如Ubuntu， Centos等等。

<img src="image-20220111111904832.png" alt="image-20220111111904832" style="zoom:50%;" />

对于一个精简的OS，rootfs 可以很小，只需要包含最基本的命令，工具和程序库就可以了，因为底层直接用Host的kernel，自己只
需要提供rootfs就可以了。由此可见对于不同的linux发行版，bootfs基本是一致的，rootfs会有差别，因此不同的发行版可以公用bootfs

### 分层理解

下载一个镜像，注意观察下载的日志输出，可以看到是一层一层的在下载

资源共享：比如有多个镜像都从相同的Base镜像构建而来，那么宿主机只需在磁盘上保留一份base镜像，同时内存中也只需要加载一份base镜像，这样就可以为所有的容器服务了，而且镜像的每一层都可以被共享。

查看镜像分层的方式可以通过 docker image inspect 命令



所有的 Docker 镜像都起始于一个基础镜像层 ，当进行修改或增加新的内容时，就会在当前镜像层之上，创建新的镜像层。举一个简单的例子，假如基于 Ubuntu Linux 16.04 创建一个新的镜像，这就是新镜像的第一层；如果在该镜像中添加 Python包，就会在基础镜像层之上创建第二个镜像层；如果继续添加一个安全补丁 ，就会创建第三个镜像层。

该镜像当前已经包含 3 个镜像层，如下图所示（这只是一个用于演示的很简单的例子）。

<img src="image-20220111112211346.png" alt="image-20220111112211346" style="zoom:50%;" />

在添加额外的镜像层的同时 ，镜像始终保持是当前所有镜像的组合 ，理解这一点非常重要。下图中举了一个简单的例子，每个镜像
层包含 了个文件，而镜像包含了来自两个镜像层的6个文件

<img src="image-20220111112330566.png" alt="image-20220111112330566" style="zoom:50%;" />

上图中的镜像层跟之前图中的略有区别 ，主要目的是便于展示文件。

下图中展示了一个稍微复杂的三层镜像，在外部看来整个镜像只有 6 个文件，这是因为最上层中的文件7是文件5的一个更新版
本

<img src="image-20220111112414957.png" alt="image-20220111112414957" style="zoom:50%;" />

这种情况下，上层镜像层中的文件覆盖了底层镜像层中的文件。这样就使得文件的更新版本作为一个新镜像层添加到镜像当中。

Docker 通过存储引擎（新版本采用快照机制）的方式来实现镜像层堆栈，并保证多镜像层对外展示为统一的文件系统。

Linux 上可用的存储引1擎有 AUFS、Overlay2、Device Mapper、Btrfs 以及 ZFS。顾名思义，每种存储引擎都基于 Linux 中对应的文件系统或者块设备技术，并且每种存储引l擎都有其独有的性能特点。

Docker 在Windows 上仅支持 windowsfilter一种存储引擎，该引擎基于 NTFS 文件系统之上实现了分层和 CoW[1]。

下图展示了与系统显示相同的三层镜像。所有镜像层堆叠并合并，对外提供统一的视图。

<img src="image-20220111112458296.png" alt="image-20220111112458296" style="zoom:50%;" />

### 特点

Docker镜像都是只读的，当容器启动时，一个新的可写层被加载到镜像的顶部！

这一层就是我们通常说的容器层，容器之下的都叫镜像层！


### commit 镜像

```shell
$ docker commit		# 提交容器成为一个新的副本

# 命令和git原理类似
docker commit -m="提交的描述信息” -a="作者" 容器id [TAG]
```



# 五、容器数据卷

需求：数据可以持久化，容器可以数据共享概念股，同步到本地

通过目录的挂载解决

<img src="image-20220111112959843.png" alt="image-20220111112959843" style="zoom:50%;" />

```shell
docker run -it -v 主机目录：容器内目录
#测试
$ docker run -it -v /home/ceshi: /home centos /bin/bash	# 启动起来时候我们可以通过docker inspect 容器id
```

![image-20220215100110498](image-20220215100110498.png)

容器与物理主机会占用两倍内存。

容器删除，主机储存的数据不会删除

如果 docker inspect后没有显示挂载，就说明挂载失败
