# Nginx 实战: 部署 React 前端项目

@[TOC](文章目录)

<!-- TOC -->

- [Nginx 实战: 部署 React 前端项目](#nginx-实战-部署-react-前端项目)
- [正文](#正文)
  - [1. 准备 React 项目 & 完成打包](#1-准备-react-项目--完成打包)
  - [2. 准备 Docker 镜像 & 配置文件](#2-准备-docker-镜像--配置文件)
  - [3. 启动 / 停止服务](#3-启动--停止服务)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

## 1. 准备 React 项目 & 完成打包

先准备一个 React 项目，使用 `create-react-app` 脚手架搭建的也好，自己手动搭建的也行，通常是使用 webpack 作为打包工具

1. 这里推荐一个作者自己写的脚手架

```bash
$ yarn global add @youxian/cli
$ yx-cli deploy_react
```

然后选择前端项目

![](https://picures.oss-cn-beijing.aliyuncs.com/img/nginx_deploy_react_1_create_app.png)

2. 接下来进入项目根目录后运行打包指令

```bash
$ cd deploy_react
$ yarn build
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/nginx_deploy_react_2_build_app.png)

## 2. 准备 Docker 镜像 & 配置文件

接下来要准备好 Docker 环境，安装就不说了，请先安装好 Docker 和 Docker Compose 两个工具（Linux 环境下 Docker Compose 需要额外安装）

1. 拉取 nginx 镜像

```bash
$ docker pull nginx
```

2. 准备 `docker-compose.yml` 配置文件

- `/deploy_react/docker-compose.yml`

```yml
version: '3'
services:
  # 服务名称
  nginx:
    # 镜像:版本
    image: nginx:latest
    # 映射容器80端口到本地80端口
    ports:
      - '3000:80'
    # 数据卷 映射本地文件到容器
    volumes:
      # 映射nginx.conf文件到容器的/etc/nginx/conf.d目录并覆盖default.conf文件
    #   - ./nginx.conf:/etc/nginx/conf.d/default.conf
      # 映射 build 文件夹到容器的 /usr/share/nginx/html 文件夹
      - ./dist:/usr/share/nginx/html
    # 覆盖容器启动后默认执行的命令。
    command: /bin/bash -c "nginx -g 'daemon off;'"
```

当然你可以提供自己的 `nginx.conf` 来替换默认的，这里先注释掉使用默认的配置文件

当你找不到 nginx 的默认配置文件的时候可以使用 `nginx -t` 指令来查找

```bash
$ nginx -t
```

## 3. 启动 / 停止服务

1. 最后使用 `docker-compose up` 指令启动容器

```bash
$ docker-compose up -d
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/nginx_deploy_react_3_run_container.png)

2. 访问应用端口

![](https://picures.oss-cn-beijing.aliyuncs.com/img/nginx_deploy_react_4_access_app.png)

3. 要停止服务则是使用 `docker-compose down`

```bash
$ docker-compose down
```

# 其他资源

## 参考连接

| Title                                                                                       | Link                                                                                                                               |
| ------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| Docker 使用: docker-compose 实现按配置启动容器                                              | [https://blog.csdn.net/weixin_44691608/article/details/120624526](https://blog.csdn.net/weixin_44691608/article/details/120624526) |
| Webpack配置区分开发环境和生产环境                                                           | [https://www.cnblogs.com/dengyao-blogs/p/11598431.html](https://www.cnblogs.com/dengyao-blogs/p/11598431.html)                     |
| React 实践项目 （五）                                                                       | [https://github.com/DigAg/digag-pc-react/issues/10](https://github.com/DigAg/digag-pc-react/issues/10)                             |
| Nginx安装，目录结构与配置文件详解                                                           | [https://www.cnblogs.com/liang-io/p/9340335.html](https://www.cnblogs.com/liang-io/p/9340335.html)                                 |
| linux下如何查找nginx配置文件的位置                                                          | [https://www.cnblogs.com/jpfss/p/10077004.html](https://www.cnblogs.com/jpfss/p/10077004.html)                                     |
| Nginx —— nginx的命令行控制（nginx的启动与停止、重载配置文件、回滚日志文件、平滑升级等操作） | [https://blog.csdn.net/weixin_42167759/article/details/84999456](https://blog.csdn.net/weixin_42167759/article/details/84999456)   |
| docker运行nginx为什么要使用 daemon off                                                      | [https://www.cnblogs.com/weifeng1463/p/10277178.html](https://www.cnblogs.com/weifeng1463/p/10277178.html)                         |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/deployment/docker/deploy_react](https://github.com/superfreeeee/Blog-code/tree/main/deployment/docker/deploy_react)
