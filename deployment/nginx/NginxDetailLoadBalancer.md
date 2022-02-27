# 细说 Nginx: 负载均衡 Load Balance

@[TOC](文章目录)

<!-- TOC -->

- [细说 Nginx: 负载均衡 Load Balance](#细说-nginx-负载均衡-load-balance)
- [准备服务](#准备服务)
- [负载均衡配置项](#负载均衡配置项)
  - [负载均衡策略](#负载均衡策略)
  - [更多配置项](#更多配置项)
- [示例](#示例)
  - [ip_hash](#ip_hash)
  - [轮询 + 权重](#轮询--权重)
- [参考连接](#参考连接)
- [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 准备服务

首先我们先准备三个后端服务，起在 8081～8083 端口上

- `server.js`

```js
const express = require('express');

const createServer = (ports) => {
  const countMap = {}; // port => count

  ports.forEach((port, i) => {
    const id = i + 1;

    const app = express();

    let count = 0;
    app.get('/', (req, res) => {
      count += 1;
      countMap[id] = count;
      res.send({
        server: id,
        count: countMap,
      });
    });

    app.listen(port, () => {
      console.log(`server${id} listen on http://localhost:${port}`);
    });
  });
};

createServer([8081, 8082, 8083]);
```

我们使用 countMap 来记录每个服务响应请求的次数

# 负载均衡配置项

接下来是 nginx 的配置项，核心的指令是 `upstream`

```nginx
http {
    upstream hello_server {
        server localhost:8081;
        server localhost:8082;
        server localhost:8083;
    }

    server {
        listen          8999;
        server_name     localhost;

        location / {
          proxy_pass  http://hello_server;
        }
    }
}
```

我们在 http 块里面添加 upstream 块，upstream 块内部再用 server 声明候选服务的域；最后再将 8999 代理到该服务上能够实现负载均衡

## 负载均衡策略

默认的情况下 nginx 采用轮询每个单体服务的方式，我们也可以指定负载策略

- 最少连接数优先(least connection)

```nginx
upstream hello_server {
    least_conn;

    # servers ...
}
```

改策略可以从连接数量上平衡请求

- ip 映射

```nginx
upstream hello_server {
    ip_hash;

    # servers ...
}
```

有些时候需要保证同一个 client 持续跟同一个 server 对接（可能使用 session 持久化等），这时候就使用 ip_hash 来保证多次连接使用同一个服务器

## 更多配置项

nginx 还提供了更多关于负载均衡的配置项，例如为 server 配置权重(weight)、server 的健康检查（失败次数限制 max_fails、超时检测 fail_timeout 等），需要用到再去配配看

[传送门：Using nginx as HTTP load balancer - Health checks](https://nginx.org/en/docs/http/load_balancing.html)

# 示例

下面我们演示几个例子，直接使用上面的[配置项](#负载均衡配置项)并稍微做一点点的修改

## ip_hash

```nginx
upstream hello_server {
    ip_hash;

    server localhost:8081;
    server localhost:8082;
    server localhost:8083;
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/nginx_detail_load_balancer_1_hash.png)

使用 ip_hash 之后我们可以看到多次请求之后一直都是同一个 server 被选中

## 轮询 + 权重

```nginx
upstream hello_server {
    server localhost:8081 weight=3;
    server localhost:8082;
    server localhost:8083;
}
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/nginx_detail_load_balancer_2_weight.png)

默认的轮询策略下同时为 server:8081 加上权重 3，可以看到连接的处理次数大致是其他服务的三倍

---

# 参考连接

| Title                             | Link                                                                                                     |
| --------------------------------- | -------------------------------------------------------------------------------------------------------- |
| Using nginx as HTTP load balancer | [https://nginx.org/en/docs/http/load_balancing.html](https://nginx.org/en/docs/http/load_balancing.html) |

# 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/deployment/nginx/nginx_detail_load_balancer](https://github.com/superfreeeee/Blog-code/tree/main/deployment/nginx/nginx_detail_load_balancer)
