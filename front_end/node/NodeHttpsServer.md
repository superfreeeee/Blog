# Node: 本地使用 HTTPS 协议

@[TOC](文章目录)

<!-- TOC -->

- [Node: 本地使用 HTTPS 协议](#node-本地使用-https-协议)
- [完整代码示例](#完整代码示例)
- [本地开发证书生成工具 mkcert](#本地开发证书生成工具-mkcert)
- [基于 https 模块创建 https 服务器](#基于-https-模块创建-https-服务器)
- [使用 express 创建 https 服务器](#使用-express-创建-https-服务器)
- [参考连接](#参考连接)

<!-- /TOC -->

# 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/node/node_https_server](https://github.com/superfreeeee/Blog-code/tree/main/front_end/node/node_https_server)

# 本地开发证书生成工具 mkcert

开源工具：[FiloSottile/mkcert - Github](https://github.com/FiloSottile/mkcert)

本篇使用 MacOS 系统 + Homebrew 安装方式，其他系统或是安装方式请查阅上面链接 readme

- 安装

```sh
brew install mkcert  # 安装工具
mkcert -install  # 添加本地 CA
```

- 生成证书：可改成自己服务器启动所使用的域名，会生成在当前命令行所在目录下

```sh
mkcert localhost
```

> 注意：切记不要把生成的 key 与 cert (.pem文件)分享到非本地主机上，会产生网站访问攻击的风险

# 基于 https 模块创建 https 服务器

首先我们先构建一个简单的 npm 项目（本篇使用 pnpm），并建立下列文件

![](https://picures.oss-cn-beijing.aliyuncs.com/img/node_https_server_1_structure.png)

- 安装依赖：本篇直接使用 Node 内置的 https 模块，所以只需要安装开发用的 nodemon，以及下一小节所用到的 express 即可

```sh
pnpm i -D nodemon
pnpm i express
```

- `server.js`

最后就是创建一个最小型的服务器即可

```js
const https = require('https');
const fs = require('fs');

const options = {
  key: fs.readFileSync('./localhost-key.pem'),
  cert: fs.readFileSync('./localhost.pem'),
};

https
  .createServer(options, (req, res) => {
    res.write(fs.readFileSync('./index.html'));
    res.end();
  })
  .listen(3000, () => {
    console.log(`Server listen at https://localhost:${3000}`);
  });
```

最后访问页面成功说明配置正确（可以看到访问网址已经从原来的 `http://localhost` 变成 `https://localhost` 就算成功了）

![](https://picures.oss-cn-beijing.aliyuncs.com/img/node_https_server_2_https.png)

# 使用 express 创建 https 服务器

原来使用 express 我们会这样写

```js
const express = express()
const app = express()

// routers ...

app.listen(PORT)
```

如果使用 generator 或是进行过其他配置可能知道可以改成作为 http 模块的中间件如下

```js
const express = express()
const app = express()

// routers ...

http.createServer(app).listen(PORT)
```

那其实改成 https 就非常简单了，直接把 http 改成使用 https 模块就可以了

- `server2.js`

```js
const https = require('https');
const fs = require('fs');
const express = require('express');
const path = require('path');

const options = {
  key: fs.readFileSync('./localhost-key.pem'),
  cert: fs.readFileSync('./localhost.pem'),
};

const app = express();

app.get('/', (req, res) => {
  res.sendFile(path.join(__dirname, '/index.html'));
});

https.createServer(options, app).listen(3000, () => {
  console.log(`Server listen at https://localhost:${3000}`);
});
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/node_https_server_3_express.png)

# 参考连接

| Title                                                         | Link                                                                                                                                                                                                                     |
| ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 如何使用 HTTPS 进行本地开发                                   | [https://web.dev/how-to-use-local-https/](https://web.dev/how-to-use-local-https/)                                                                                                                                       |
| FiloSottile/mkcert - Github                                   | [https://github.com/FiloSottile/mkcert](https://github.com/FiloSottile/mkcert)                                                                                                                                           |
| Rendering HTML Pages as an HTTP Server Response Using Node.js | [https://www.section.io/engineering-education/rendering-html-pages-as-a-http-server-response-using-node-js/](https://www.section.io/engineering-education/rendering-html-pages-as-a-http-server-response-using-node-js/) |
| Enabling HTTPS on express.js - stackoverflow                  | [https://stackoverflow.com/questions/11744975/enabling-https-on-express-js](https://stackoverflow.com/questions/11744975/enabling-https-on-express-js)                                                                   |
| How To Deliver HTML Files with Express                        | [https://www.digitalocean.com/community/tutorials/use-expressjs-to-deliver-html-files](https://www.digitalocean.com/community/tutorials/use-expressjs-to-deliver-html-files)                                             |
