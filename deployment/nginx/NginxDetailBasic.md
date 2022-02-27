# 细说 Nginx: 静态资源服务器基础 - server,location,root

@[TOC](文章目录)

<!-- TOC -->

- [细说 Nginx: 静态资源服务器基础 - server,location,root](#细说-nginx-静态资源服务器基础---serverlocationroot)
- [Nginx 概述 & 安装简要](#nginx-概述--安装简要)
  - [安装](#安装)
- [Nginx 配置文件结构](#nginx-配置文件结构)
- [实验1: 第一个路由](#实验1-第一个路由)
- [实验2: 更多的服务器](#实验2-更多的服务器)
  - [2.1 location 路径匹配](#21-location-路径匹配)
- [实验3: 代理服务器](#实验3-代理服务器)
- [实验小结](#实验小结)
- [参考连接](#参考连接)
- [完整代码示例](#完整代码示例)

<!-- /TOC -->

# Nginx 概述 & 安装简要

Nginx 作为 http 服务器可说是大获成功，不论是在静态资源分发或是反向代理上的负载均衡性能都非常好。

本篇将带大家来认识 Nginx 的基础用法之一：静态资源服务器的基础配置方式

## 安装

- Linux

Linux 系统方面的支持都挺好，可以直接参考下面链接进行安装

[传送门：nginx: Linux packages](https://nginx.org/en/linux_packages.html)

- MacOS

macOS 上稍微特别，这里推荐使用 homebrew 进行安装

```bash
brew install nginx
```

homebrew 提供了类似 linux 系统的 service 指令，可用来启动 nginx 服务

```bash
brew services start nginx
```

后续提到的相关目录（如 `/var/log`、`/var/run`、`/etc/nginx.conf` 等配置可能会被迁移到 `/opt/homebrew` 目录下，具体需要看 homebrew 的配置，应该还是蛮好找的）

- Docker

Docker 算是一个非常好的部署平台，弭平了系统间的差异。基于 Docker 启动 nginx 也非常简单，直接拉一个官方的镜像就好了，可以跟着下面教程走

[传送门：nginx(Official Image) - Official build of Nginx.](https://hub.docker.com/_/nginx)

- Windows

Nginx 作为服务器通常还是用 Linux 居多，退一步在同样基于 Unix 内核上的 MacOS 也勉强还行，但是真的不推荐装在 windows 上。下面提供一个安装连接，相关命令应该是保持一致的，但是目录结构的不相同就需要用户自己去探索了

[传送门：nginx for Windows](http://nginx.org/en/docs/windows.html)

# Nginx 配置文件结构

Nginx 官方对于配置文件的结构说明和书写主要是按具体用例一一说明，官方似乎没有对所有可配置项有一个统整的说明。

不过这样也并不是什么大问题，用到啥功能再去学也不迟

本篇先介绍一些非常基本的配置结构，更细节的后面几篇会再说到hh

- 配置文件结构

`nginx.conf` 的配置文件结构可以分成指令（`directive`）和块（`block`），块同时也作为内部指令的上下文(`context`)存在，而所有块的最外层则称为全局上下文(`main context`)。

全局上下问只允许包含两种块体：`events`、`http`

```nginx
events {
}

http {
}
```

本篇关注在 http 静态资源服务器的搭建，所以我们具体只关注 http 块结构

http 块当中又可以包含多个 `server` 块，称为虚拟服务器(`virtual server`)；而 server 块当中则可以再包含 `location` 块，结构如下

```nginx
http {
  server {
    location / {
    }
  }
}
```

下面我们直接带大家来看看简单的 http 服务器要怎么写

# 实验1: 第一个路由

上面提过的配置文件结构直接拿下来用，并且填充第一个 location 块就能完成第一个最基础的静态资源服务器啦

```nginx
http {
    server {
        listen          8080;
        server_name     localhost;

        location / {
            root ~/data/images;
        }
    }
}
```

这里我们在 server 块中指定监听 8080 端口以及 host 名(`localhost`)，并且写了第一个 location 块

如此一来 nginx 就能够响应 `http://localhost:8080/` 为前缀的任何资源，并从 `root` 指定的路径查询相关数据。例如对于 `http://localhost:8080/sample.jpg` 的请求，nginx 就会去查找 `~/Desktop/tmp/test-root/data/images/sample.jpg` 的文件是否存在

- 示例

![](https://picures.oss-cn-beijing.aliyuncs.com/img/nginx_deploy_react_1_jpg.png)

# 实验2: 更多的服务器

接下来我们创建第二个 server，以及更多的 location

```nginx
http {
    server {
        listen          8080;
        server_name     localhost;

        location / {
            root ~/data;
        }

        location ~ \.(gif|jpg|png)$ {
            root ~/data/images;
        }
    }

    server {
        listen  8081;

        location / {
            root ~/data/images;
        }
    }
}
```

这次我们指定了两个 server，分别监听 8080、8081 的请求，并且在 8080 的 server 下添加了第二个 location，当我们遇到指定后缀名的时候，则从 `/data/images` 路径下寻找资源

也就是说上面的例子我们不论是访问 `http://localhost:8080/sample.jpg` 或是 `http://localhost:8081/sample.jpg`，实际上都是访问到 `~/data/images/sample.jpg` 的图片

- 示例

![](https://picures.oss-cn-beijing.aliyuncs.com/img/nginx_deploy_react_1_jpg.png) ![](https://picures.oss-cn-beijing.aliyuncs.com/img/nginx_deploy_react_2_jpg.png)

## 2.1 location 路径匹配

这里还要再特别介绍一下 location 的路径匹配问题，当我们添加更多 location 或是在 location 的路径上做文章的时候，实际上只是给予 nginx 一个选择，也就是仅仅描述一个匹配规则。

匹配到目标 location 之后，还是会用整个 pathname 与 root 合并作为最终路径，而不会自动舍弃匹配的部分

```nginx
location /images {
  root /data/images
}
```

如上面这种匹配方式，如果我们发起的是 `http://localhost:8080/images/sample.jpg`，实际上真正去寻找的文件则是 `/data/images/images/sample.jpg`，这点需要读者小心（作者在使用初期常常陷入这种误区）

# 实验3: 代理服务器

到目前为止相信透过 server、location 的组合已经能够基本满足一些简单静态资源的分发（注意我们这边说的都是单体应用的情况下）

最后我们额外介绍一种代理，简单的将对指定站点的请求转发到另一个站点

```nginx
server {
    listen          8082;
    server_name     localhost;

    location /www {
        proxy_pass  http://localhost:8080;
    }
}
```

这样写的目标是去匹配 `http://localhost:8082/www` 为前缀的路径，并将其转发给 8080 的 server（注意根据前面 2.1 的路径匹配规则），转发之后实际上还是对 `8080/www` 进行匹配，而不会将匹配过的 www 删掉，这点也要小心。

# 实验小结

本质上我们可以将 nginx 视为一个单一的反向代理服务器，所以对于 location 的设计可以从用户可能发起的方式来进行配置，然后最后将可能匹配的路径集合导向正确的资源存放位置

---

# 参考连接

| Title     | Link                                                     |
| --------- | -------------------------------------------------------- |
| nginx     | [https://nginx.org/en/docs/](https://nginx.org/en/docs/) |
| NginxDocs | [https://docs.nginx.com/](https://docs.nginx.com/)       |

# 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/deployment/nginx/nginx_detail_basic](https://github.com/superfreeeee/Blog-code/tree/main/deployment/nginx/nginx_detail_basic)
