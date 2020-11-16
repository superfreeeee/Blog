# Docker: Mac 上的 Ubuntu 环境搭建

@[TOC](文章目录)

<!-- TOC -->

- [Docker: Mac 上的 Ubuntu 环境搭建](#docker-mac-上的-ubuntu-环境搭建)
  - [简介](#简介)
  - [参考](#参考)
- [正文](#正文)
  - [搭建目标](#搭建目标)
  - [安装 Docker for Mac](#安装-docker-for-mac)
  - [使用阿里加速器](#使用阿里加速器)
  - [获取镜像并启动容器](#获取镜像并启动容器)
    - [确认安装并启动](#确认安装并启动)
    - [获取 Ubuntu20.04 镜像](#获取-ubuntu2004-镜像)
    - [创建容器并进入伪终端](#创建容器并进入伪终端)
  - [为 Ubuntu 配置 ssh、vim 相关工具](#为-ubuntu-配置-sshvim-相关工具)
    - [安装工具](#安装工具)
    - [配置 ssh](#配置-ssh)
  - [制作镜像以保存环境](#制作镜像以保存环境)
  - [从宿主机透过 ssh 访问虚拟机](#从宿主机透过-ssh-访问虚拟机)
- [结语](#结语)

<!-- /TOC -->

## 简介

Docker 容器化管理的功能，几乎能够替代虚拟机的作用。本篇就来介绍使用 Docker Image 创建环境并作为虚拟机使用的详细步骤。

## 参考

<table>
  <tr>
    <td>Docker for Mac安装与入门</td>
    <td><a href="https://www.jianshu.com/p/a414f081d296">https://www.jianshu.com/p/a414f081d296</a></td>
  </tr>
  <tr>
    <td>配置阿里云镜像加速器</td>
    <td><a href="https://blog.csdn.net/niukaoying6674/article/details/87788282">https://blog.csdn.net/niukaoying6674/article/details/87788282</a></td>
  </tr>
  <tr>
    <td>docker安装Ubuntu以及ssh连接</td>
    <td><a href="https://www.cnblogs.com/mengw/p/11413461.html">https://www.cnblogs.com/mengw/p/11413461.html</a></td>
  </tr>
  <tr>
    <td>docker for mac官方链接</td>
    <td><a href="https://download.docker.com/mac/stable/Docker.dmg">https://download.docker.com/mac/stable/Docker.dmg</a></td>
  </tr>
  <tr>
    <td>docker阿里镜像</td>
    <td><a href="http://mirrors.aliyun.com/docker-toolbox/">http://mirrors.aliyun.com/docker-toolbox/</a></td>
  </tr>
</table>

# 正文

## 搭建目标

本篇要搭建的目标环境是在 MacOS 上的 Docker 环境中安装 Ubuntu 镜像并作为虚拟机使用

- 宿主机器操作系统：MacOS
- 容器环境：Docker for Mac
- 镜像版本：Ubuntu20.04

## 安装 Docker for Mac

首先我们需要在 mac 上安装 Docker，我们可以透过以下三种途径安装：

- 使用 Homebrew 直接下载

```bash
$ brew cask install docker
```

- 透过 Docker 官方链接手动安装 Docker Desktop

<a href="https://download.docker.com/mac/stable/Docker.dmg">https://download.docker.com/mac/stable/Docker.dmg</a>

- 透过阿里云镜像安装 Docker Desktop（国内较快）

<a href="http://mirrors.aliyun.com/docker-toolbox/mac/docker-for-mac/stable/Docker.dmg">http://mirrors.aliyun.com/docker-toolbox/mac/docker-for-mac/stable/Docker.dmg</a>

## 使用阿里加速器

由于国内安装 Docker 镜像速度较慢，我们可以使用阿里提供的`容器镜像加速器`的服务

1. 首先当然得先到阿里云上面办一个号

2. 进入阿里云`容器镜像服务`中

![](https://picures.oss-cn-beijing.aliyuncs.com/img/docker-aliyun-accelerate1.png)

3. 选择`镜像加速器`并获得加速器地址

![](https://picures.oss-cn-beijing.aliyuncs.com/img/docker-aliyun-accelerate2.png)

3. 由于本篇环境使用 Docker for Mac，所以到 Docker Desktop，选择 **Preference** -> **Docker Engine**，并在右侧 JSON 配置中添加以下配置，然后点击右下角的`Apply & Restart`就配置成功了

```json
{
    // others...
    "registry-mirrors": ["自己的加速器地址"]
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/docker-aliyun-accelerate3.png)

## 获取镜像并启动容器

### 确认安装并启动

启动 Docker 之后检查 Docker 安装和启动情形

```bash
$ docker --version
Docker version xxx
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
```

若出现`Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?`则表示 Docker Desktop 尚未启动

### 获取 Ubuntu20.04 镜像

Docker 相关指令可以参考前一篇：<a href="https://blog.csdn.net/weixin_44691608/article/details/107208251">Docker：部署入門 + 常用指令</a>

我们可以使用`docker search <image-name>`查找 DockerHub 上现有的镜像

```bash
$ docker search ubuntu
```

接下来我们使用`docker pull <image-name>:<tag>`指令获取 Ubuntu 镜像

```bash
$ docker pull ubuntu:20.04
```

可以使用`docker images`查看本地所有镜像

```bash
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              20.04               d70eaf7277ea        2 weeks ago         72.9MB
```

### 创建容器并进入伪终端

- 创建并启动容器

```bash
$ docker run -itd -p 3316:22 ubuntu:20.04
```

参数说明：
1. `-i`: 交互式模式开启；`-t`: 分配 tty 伪终端；`-it` 通常同时使用
2. `-d`: 后台运行容器
3. `-p`: 指定端口映射，`3316:22` 表示将宿主机器的 3316 端口映射到容器内部的 22 端口(sshd 端口)

使用 `docker ps` 检查容器运行情况

```bash
$ docker ps -a
docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                        PORTS                  NAMES
f6d1867fa1eb        ubuntu:20.04        "/bin/bash"         3 seconds ago       Up 2 seconds                  0.0.0.0:3316->22/tcp   sad_margulis
```

- 进入容器终端

`docker exec` 和 `docker attach` 都能达成我们进入终端的目的，但是使用 `attach` 后在终端 `exit` 会导致容器的停止，所以这边采用 `docker exec <container-id>` 的用法

```bash
$ docker exec -it f6d1867fa1eb /bin/bash
root@f6d1867fa1eb:/# 
```

## 为 Ubuntu 配置 ssh、vim 相关工具

### 安装工具

进入 Ubuntu 伪终端之后，接下来我们需要安装一些工具如 ssh、vim

- ssh 相关：`openssh-client`(ssh 客户端)、`openssh-server`(ssh 服务端)
- vim 文字编辑工具

```bash
root@f6d1867fa1eb:/# apt-get install openssh-client openssh-server vim
```

ssh-server 安装过程可能需要配置所在地区和时区信息，选择 `Asia/Shanghai`

### 配置 ssh

安装完毕后我们需要修改 `sshd_config` 的配置：`PermitRootLogin` 来允许使用 root 权限登入

- 首先启动 ssh 服务，并查看是否启动成功

```bash
root@f6d1867fa1eb:/# /etc/init.d/ssh start
root@f6d1867fa1eb:/# ps -e | grep ssh
  3882 ?        00:00:00 sshd
```

- 修改 `sshd_config`，添加 `PermitRootLogin yes`

```bash
root@f6d1867fa1eb:/# vim /etc/ssh/sshd_config
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ssh-rootlogin.png)

- 重新启动 ssh 服务

```bash
root@f6d1867fa1eb:/# service ssh restart
```

- 退出终端

到此就全部配置完毕了，输入 `exit` 退出终端

```bash
root@f6d1867fa1eb:/# exit
```

## 制作镜像以保存环境

到此我们已经配置完成我们的虚拟机容器了，我们可以使用 `docker commit <container-id> <repository>:<tag>` 来将容器重新制作成镜像，相当于一个系统配置后初始状态的备份

```
$ docker commit f6d1867fa1eb vm-ubuntu
sha256:14eb657556198e5d92fa66a15766af0203cb3cbb21b84e6a4f65385c7b36ca7d
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
vm-ubuntu           latest              14eb65755619        14 seconds ago      267MB
ubuntu              20.04               d70eaf7277ea        2 weeks ago         72.9MB
```

## 从宿主机透过 ssh 访问虚拟机

最后我们就能够在宿主机使用 ssh 访问到虚拟机了。由于我们启动容器时使用端口映射 `3316 -> 20` 所以 ssh 时需要指定端口(指令：`ssh -p <指定端口> <用户>@<主机名>`)：

```bash
$ ssh -p 3316 root@localhost
Welcome to Ubuntu 20.04.1 LTS (GNU/Linux 5.4.39-linuxkit x86_64)
root@localhost's password: 
 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Last login: Fri Nov 13 14:18:21 2020 from 172.17.0.1
root@f6d1867fa1eb:~# 
```

# 结语

容器化技术相对于虚拟机更为轻量化，本篇一步一脚印使用 Docker 在 Mac 上安装一台 Ubuntu 容器化虚拟机，供大家参考。
