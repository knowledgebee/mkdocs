---
title: 使用 Docker
date: 2022-04-29 17:06:55
tags: 
    - Docker
    - Software
---

如何使用开源容器引擎 Docker

<!--more-->

Docker 是一个开源的容器引擎。Docker 相对于虚拟机来说启动快，占用资源少，迁移部署方便，在迁移时可以节省大量重新部署和调试的时间。Docker 将应用与运行平台解耦，应用与服务运行在容器中，迁移时无论新旧平台是否相同，直接迁移容器即可。Docker 的特性让 Devop 的 CI/CD ( 持续集成/持续部署 ) 十分方便。

Docker 运行需要守护进程，如果未进行超级管理员权限设置或者未在 root账户下操作，需要在 docker 命令前加上 *`sudo`*

### 启动一个容器 ( *`docker run`* )

***

Docker 启动和创建容器十分简单，仅需一个命令便可启动容器：

> *docker run \[OPTIONS\] \<image:tag\>*

Docker的 *run* 命令有较多的可选操作和参数，命令中的参数 *`<image:tag>`* 为必选选项，指明所需要运行的容器的镜像名称，如果本地不存在此镜像，Docker 会自动从 Docker Hub 上拉取该镜像。Docker 的容器可以基于镜像 ( image ) 生成，也可以使用 Dockerfile 构建 ( builde ) 生成。

命令中的 *`[OPTIONS]`* 操作选项为可选选项：

可以使用 *`-d`* 参数让容器在后台运行，否则创建的容器默认是停止状态 ( *Exited* )

如果容器运行的应用程序需要对外通信或提供服务，可以将容器的端口映射到主机端口上，使用 *`-p`* 参数，使用 *`-p`* 参数需要指明映射的端口，例如将使用 Ubuntu 镜像创建的容器的 80 端口映射到主机的 8000 端口并设置后台运行：

> *docker run -d -p 8000:80 ubuntu:latest*

*docker run* 命令执行完成后容器已经开始运行，查看本地的容器可以使用 *`ps`* 命令，该命令会列出本地运行中的容器，查看停止或者所有容器需要加参数 *`-a`* ：

> *docker ps -a*

### 容器网络

***

容器与容器之间是相互隔离的，容器之间的通信需要借助 Docker 的 Bridge 网络来进行通信。Docker 默认 Bridge 网络的网段为 *172.17.0.0/16* ，创建容器时未指定容器网络将默认连接到此网络，Docker 会为连接到此网络的容器分配一个 *172.17.0.0/16* 网段内的IP地址用来与主机和 Docker 默认 Bridge 内的容器通信。

容器在默认的 Bride 网络将处在相同的网段，即 *172.17.0.0/16* ，在相同的网段中容器可以相互访问，这将带来安全隐患。安全的做法是为需要通信的容器建立一个新的 Bridge 网络，使得不同网段中的容器相互隔离，在相同网段中的容器可以相互访问。

创建名为 *New* 的 Bridge 网络：

> *docker network create New*

查看已有的 Bridge 网络：

> *docker network ls*

如果需要将容器加入到创建的 Bridge 网络需要在创建容器时指定加入的网络，指定其加入创建的 Bridge 网络 New ，需要在创建容器时加上 *`--network New`* 参数。

查看 Bridge 网络 New 的详细参数使用 *`inspect`* :

> *docker network inspect New*

### 容器卷

***

容器的存储设计在容器被删除后容器内的所有更改都会被删除，当需要将容器中的更改变为持久化存储时就需要容器的卷 ( *volume* ) 的帮助。将容器中指定的路径挂载到卷后，该路径内的所有操作都会同步在指定的卷中，在容器被删除后该容器的卷并不会被删除，卷的操作与网络的操作相同，将 *`network`* 改为 *`volume`* 即可。

创建名为 *New-volume* 的 *`volume`* ：

> *docker volume creat New-volume*

查看所有的 volume ：

> *docker volume ls*

查看 *New-volume* 的详细参数

> *docker volume inspect New-volume*

容器的 volume 的设计让容器的迁移变得十分方便，在迁移时迁移 volume 即可，在创建新的容器时指定该 volume 即可。

### Docker Compose

***

在使用 Docker 时，启动容器时有个性化需求时需要指定很多参数，启动一个容器简单，但是要启动多个容器每个都要指定许多参数是非常麻烦的。我们可以使用 Docker 官方的容器编排工具 Docker Compose (需要安装) 实现快速批量启动容器。

使用 Docker Compose 需要编写 *`docker compose.yaml`* 文件或者 *`compose.yaml`* 文件，启动十分简单：

> *docker compose up -d*

在启动之后，文件中所指定的的参数会自动生成，不再需要手动创建，关闭也同样简单：

> *docker compose down*

当需要修改更新编排容器的镜像时，操作同样简单，修改 *`compose.yaml`* 文件后执行以下命令即可：

> *docker compose pull && docker compose up -d*

### **Notice**

***

\* *当/etc/docker目录下的 `daemon.json` 文件格式不正确时 Docker 将无法启动*

\* 直接将 volume 直接复制到 Docker 目录下需要在 Docker 中执行创建同名 volume

\* entrypoint.sh 需要注意换行标记，CRLF (Windows) 和 LF (Linux) 是不一样的，否则会造成容器无法启动
