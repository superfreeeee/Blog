# Nginx 踩坑笔记: 部署 React/Vue 前端应用路由 404 Not Found(Error: 404 Not Found | Nginx 1.21.6)

@[TOC](文章目录)

<!-- TOC -->

- [Nginx 踩坑笔记: 部署 React/Vue 前端应用路由 404 Not Found(Error: 404 Not Found | Nginx 1.21.6)](#nginx-踩坑笔记-部署-reactvue-前端应用路由-404-not-founderror-404-not-found--nginx-1216)
- [0. 项目背景](#0-项目背景)
- [1. 问题描述](#1-问题描述)
- [2. 问题排查 & 解决方案](#2-问题排查--解决方案)
  - [解决方案](#解决方案)
- [附录：示例项目运行文档](#附录示例项目运行文档)
- [参考连接](#参考连接)

<!-- /TOC -->

# 0. 项目背景

React/Vue 等 SPA 前端应用使用 history 路由模式，打包部署到 nginx 服务器上

# 1. 问题描述

本地开发环境下路由表现正常，部署到 nginx 上后在非根路由下刷新发生 404 Not Found

![](https://picures.oss-cn-beijing.aliyuncs.com/img/nginx_note_spa_router_4_notfound.png)

# 2. 问题排查 & 解决方案

这是由于页面刷新/首次进入页面的时候，会先访问 nginx 服务器获取相应的 html 资源，因此 SPA 应用访问非根目录是会进入错误的路由而产生 404 报错

## 解决方案

在没有特殊配置的情况下，可以直接在 nginx 配置文件内添加 `try_files` 指令，使得指定路径下的所有资源请求失败后会默认尝试 `/index.html` 文件，SPA 项目会在自动根据路由进行 SPA 应用内部的路由判断

在 `nginx.conf` 内的 location 块添加下面这句话即可

```nginx
try_files $uri /index.html;
```

完整配置示例

```nginx
events {
    worker_connections 512;
}

http {
    server {
        listen 80;
        server_name localhost;

        location / {
            root /app;

            try_files $uri /index.html;
        }
    }
}
```

# 附录：示例项目运行文档

[https://github.com/superfreeeee/Blog-code/tree/main/deployment/nginx/nginx_note_spa_router](https://github.com/superfreeeee/Blog-code/tree/main/deployment/nginx/nginx_note_spa_router)

1. 运行 `npm install && npm run build` 进行项目打包
2. 启动 Docker 服务，并运行 `docker-compose up -d`
3. 访问 `http://localhost:3001/login` 查看效果

# 参考连接

| Title                                    | Link                                                                                                                                                                                           |
| ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 實際理解 nginx try_files                 | [https://insights.rytass.com/%E5%AF%A6%E9%9A%9B%E7%90%86%E8%A7%A3-nginx-try-files-409361637081](https://insights.rytass.com/%E5%AF%A6%E9%9A%9B%E7%90%86%E8%A7%A3-nginx-try-files-409361637081) |
| Deploying NGINX and NGINX Plus on Docker | [https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-docker/](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-docker/)                       |
