# Docker: 部署 Nginx 反向代理

@[TOC](文章目录)

<!-- TOC -->

- [Docker: 部署 Nginx 反向代理](#docker-部署-nginx-反向代理)
- [环境准备](#环境准备)
- [实验一：容器内部署 Nginx 静态资源服务器](#实验一容器内部署-nginx-静态资源服务器)
  - [目录结构](#目录结构)
  - [静态资源准备](#静态资源准备)
  - [Docker Compose 配置](#docker-compose-配置)
  - [nginx 配置](#nginx-配置)
  - [启动 & 效果](#启动--效果)
- [实验二：Nginx 反向代理本机服务](#实验二nginx-反向代理本机服务)
  - [准备 Express 服务](#准备-express-服务)
  - [修改 Docker Compose 配置](#修改-docker-compose-配置)
  - [修改 nginx 配置](#修改-nginx-配置)
  - [测试](#测试)
- [实验小结](#实验小结)
- [参考连接](#参考连接)
- [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 环境准备

要求安装下列服务

- Docker
- Docker Compose(Windows/MacOS 下 Docker Desktop 自带，可以不用额外安装)
- Node.js(用于启动一个 express 后端服务，如果有其他如 java+tomcat、golang 等服务的话可以不用下载)

# 实验一：容器内部署 Nginx 静态资源服务器

第一个实验的目标是在 docker 内启动一个 nginx 容器，访问并返回一个 html 页面

## 目录结构

```
.
├── Dockerfile
├── dist
│   └── index.html
├── docker-compose.yml
└── nginx.conf
```

## 静态资源准备

先准备一个 `index.html`

- `/dist/index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Nginx config test</title>
  </head>
  <body>
    <h1>/dist/index.html by nginx container</h1>
  </body>
</html>
```

## Docker Compose 配置

接下来准备一个 `docker-compose.yml` 配置项

- `/docker-compose.yml`

```yml
version: '3.7'

services:
  nginx:
    image: nginx
    ports:
      - 8991:80
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./dist:/usr/share/nginx/html
    command: /bin/bash -c "nginx -g 'daemon off;'"
```

从 volumes 可以看到我们将本地的 nginx.conf 来取代默认配置文件，并将静态文件目录 dist 映射到容器内

## nginx 配置

最后根据 docker-compose 的端口映射，指定 nginx 服务监听的端口(docker 将本地 8991 端口映射到容器内的 80 端口，因此 nginx 配置应该设置为 80 端口；上面静态资源目录 `./dist` 映射到 `/usr/share/nginx/html`，作为容器内的静态资源根目录)

- `/nginx.conf`

```nginx
events {
    worker_connections 1024;
}

http {
    server {
        listen          80;
        server_name     localhost;

        location / {
            root        /usr/share/nginx/html;
        }
    }
}
```

## 启动 & 效果

最后启动服务检查效果

```bash
docker-compose up -d
```

- `-d` 表示启动在后台
- 如果没有事先拉 nginx 镜像，会先自动拉
- 根据 `docker-compose.yml` 启动容器
- 根据 `nginx.conf` 启动 nginx 服务

最后访问 8991 应该就能看到页面了

![](https://picures.oss-cn-beijing.aliyuncs.com/img/docker_http_proxy_nginx_1.png)

# 实验二：Nginx 反向代理本机服务

实验二的目标是利用 nginx 容器反向代理回本地端口上的服务

## 准备 Express 服务

本篇使用 express 启动一个后端服务，其他任意一个服务也都可以

- `server.js`

```js
const express = require('express');

const app = express();

const port = 8080;

app.get('/', (req, res) => {
  res.send('Response from localhost:8080');
});

app.listen(port, () => {
  console.log(`Example app listening on port ${port}!`);
});
```

然后在本地安装依赖

```bash
npm init -y
npm install express
npm install nodemon -D
```

添加启动脚本

- `package.json`

```json
{
  // ...
  "scripts": {
    "start": "nodemon server.js"
  }
}
```

启动服务

```bash
npm run start
```

在本地访问端口测试服务是否启动成功

![](https://picures.oss-cn-beijing.aliyuncs.com/img/docker_http_proxy_nginx_2_app.png)

## 修改 Docker Compose 配置

实验二相较于实验一只需要增加一个新的端口映射即可

```yml
ports:
  - 8992:8999
```

完整的配置项

- `docker-compose.yml`

```yml
version: '3.7'

services:
  nginx:
    image: nginx
    ports:
      - 8991:80
      - 8992:8999
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./dist:/usr/share/nginx/html
    command: /bin/bash -c "nginx -g 'daemon off;'"
```

## 修改 nginx 配置

由于我们为容器添加了一个新的端口映射，我们只需要在 nginx 内多配置一个 server 即可

- `nginx.conf`

```nginx
events {
    worker_connections 1024;
}

http {
    server {
        listen          8999;
        server_name     localhost;

        location / {
            proxy_pass  http://host.docker.internal:8080;
        }
    }
}
```

注意这里使用一个特别的域名 `host.docker.internal`，docker 会自动将发向该 host 的请求转发到本地机器上，也就能够访问到容器外部的本地服务了

## 测试

最后我们在本地访问 8992 端口，会先访问到 nginx 容器的 8999，并透过 proxy_pass 重新转发会本机上的 8080 端口

![](https://picures.oss-cn-beijing.aliyuncs.com/img/docker_http_proxy_nginx_3_proxy.png)

# 实验小结

本篇实践了以下技术

- 使用 docker-compose 启动 docker 容器
- 使用 nginx 进行静态资源分发 & 服务转发
- 使用特殊域名 `host.docker.internal` 从容器内部指向本地机器

---

# 参考连接

| Title                            | Link                                                                                                                               |
| -------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| Use Docker Compose - docker docs | [https://docs.docker.com/get-started/08_using_compose/](https://docs.docker.com/get-started/08_using_compose/)                     |
| nginx - DockerHub                | [https://hub.docker.com/_/nginx](https://hub.docker.com/_/nginx)                                                                   |
| Nginx 实战: 部署 React 前端项目  | [https://blog.csdn.net/weixin_44691608/article/details/120624970](https://blog.csdn.net/weixin_44691608/article/details/120624970) |

# 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/deployment/docker/docker_http_proxy_nginx](https://github.com/superfreeeee/Blog-code/tree/main/deployment/docker/docker_http_proxy_nginx)
