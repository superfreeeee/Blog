# Docker 使用: docker-compose 实现按配置启动容器

@[TOC](文章目录)

<!-- TOC -->

- [Docker 使用: docker-compose 实现按配置启动容器](#docker-使用-docker-compose-实现按配置启动容器)
- [正文](#正文)
  - [1. docker-compose 作用](#1-docker-compose-作用)
  - [2. 使用示例：基于 nginx 部署前端项目](#2-使用示例基于-nginx-部署前端项目)
  - [3. 常见指令](#3-常见指令)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

## 1. docker-compose 作用

- 可将 docker 运行命令写成配置文件
- 同一个系统的多个服务同时运行在隔离环境下

## 2. 使用示例：基于 nginx 部署前端项目

- 使用 docker-compose 的时候，需要写 `docker-compose.yml` 作为启动的配置文件

1. 编写启动时的配置文件

- `docker-compose.yml`

```yml
version: '3'
services:
  # 服务名称
  nginx:
    # 镜像:版本
    image: nginx:latest
    # 本地 300 -> 容器 80
    ports:
      - '3000:80'
    # 数据卷 映射本地文件到容器
    volumes:
      # 本地 nginx.conf -> 容器配置文件
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
      # 前端打包目录 -> nginx 静态资源目录
      - ./dist:/usr/share/nginx/html
    # 启动命令：daemon off 将 nginx 提到前台避免容器退出
    command: /bin/bash -c "nginx -g 'daemon off;'"
```

2. 启动容器（需要在 `docker-compose.yml` 文件所在的目录下）

```bash
$ docker-compose up -d
$ docker-compose ps
```

- `-d` 表示后台运行
- `ps` 为查看运行服务组

当然在使用原本的 `docker ps` 也能看到运行中的容器

![](https://picures.oss-cn-beijing.aliyuncs.com/img/docker_compose_1_start.png)

## 3. 常见指令

- 构建容器 & 启动服务

```bash
$ docker-compose up -d
```

- 停止服务 & 删除容器

```bash
$ docker-compose down
```

- 查看当前服务

```bash
$ docker-compose ps
```

- 其他指令

```bash
$ docker-compose -h
Define and run multi-container applications with Docker.

Usage:
  docker-compose [-f <arg>...] [--profile <name>...] [options] [--] [COMMAND] [ARGS...]
  docker-compose -h|--help

Options:
  ...

Commands:
  build              Build or rebuild services
  config             Validate and view the Compose file
  create             Create services
  down               Stop and remove resources
  events             Receive real time events from containers
  exec               Execute a command in a running container
  help               Get help on a command
  images             List images
  kill               Kill containers
  logs               View output from containers
  pause              Pause services
  port               Print the public port for a port binding
  ps                 List containers
  pull               Pull service images
  push               Push service images
  restart            Restart services
  rm                 Remove stopped containers
  run                Run a one-off command
  scale              Set number of containers for a service
  start              Start services
  stop               Stop services
  top                Display the running processes
  unpause            Unpause services
  up                 Create and start containers
  version            Show version information and quit
```

# 其他资源

## 参考连接

| Title                            | Link                                                                                                                             |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Docker Compose - 菜鸟            | [https://www.runoob.com/docker/docker-compose.html](https://www.runoob.com/docker/docker-compose.html)                           |
| Docker中进入容器命令行及后台运行 | [https://blog.csdn.net/weixin_39750084/article/details/82085888](https://blog.csdn.net/weixin_39750084/article/details/82085888) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/deployment/docker/deploy_react](https://github.com/superfreeeee/Blog-code/tree/main/deployment/docker/deploy_react)
