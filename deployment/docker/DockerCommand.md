# Docker：部署入門 + 常用指令

@[TOC](文章目錄)

<!-- TOC -->

- [Docker：部署入門 + 常用指令](#docker部署入門--常用指令)
  - [簡介](#簡介)
  - [參考](#參考)
- [正文](#正文)
  - [Docker 三元素](#docker-三元素)
    - [Image 鏡像（映像檔）](#image-鏡像映像檔)
    - [Container 容器](#container-容器)
      - [堆疊](#堆疊)
    - [Repository 倉庫](#repository-倉庫)
  - [基於 node 環境的簡單 Web 項目](#基於-node-環境的簡單-web-項目)
    - [Install 安裝](#install-安裝)
      - [Windows / Mac 環境](#windows--mac-環境)
      - [Ubuntu Linux 環境](#ubuntu-linux-環境)
      - [CentOS Linux 環境](#centos-linux-環境)
    - [項目內容](#項目內容)
    - [建立 Image 鏡像](#建立-image-鏡像)
    - [建立並運行 Container](#建立並運行-container)
    - [停止和銷毀](#停止和銷毀)
  - [Docker 常用指令](#docker-常用指令)
    - [基本信息](#基本信息)
    - [Image 鏡像相關](#image-鏡像相關)
    - [Container 容器相關](#container-容器相關)
- [結語](#結語)

<!-- /TOC -->

## 簡介

Docker 作為容器化部署最成功的應用之一，相較於虛擬機除了節省了多餘的操作系統空間之外，也一併解決了"我的電腦上就 OK"的問題（環境不相容、不支援等問題）。只要安裝 Docker 環境，就可以在任意一台服務器上啟動相同的 Image 而有相同的表現。本篇將不會深究`虛擬機`和`容器化部署`的差異和優劣，相關內容可以查詢其他文章，接下來就讓我們來看看 docker 的核心三元素以及使用方法吧。

## 參考

<table>
  <tr>
    <td>Docker 基礎教學與介紹 101</td>
    <td><a href="https://medium.com/unorthodox-paranoid/docker-tutorial-101-c3808b899ac6">https://medium.com/unorthodox-paranoid/docker-tutorial-101-c3808b899ac6</a></td>
  </tr>
  <tr>
    <td>Docker 基本指令操作</td>
    <td><a href="https://ithelp.ithome.com.tw/articles/10186431">https://ithelp.ithome.com.tw/articles/10186431</a></td>
  </tr>
  <tr>
    <td>Docker</td>
    <td><a href="https://www.docker.com/">https://www.docker.com/</a></td>
  </tr>
  <tr>
    <td>Docker Hub</td>
    <td><a href="https://hub.docker.com/">https://hub.docker.com/</a></td>
  </tr>
</table>

# 正文

## Docker 三元素

Docker 有三個核心要素：`Image 鏡像`、`Container 容器`、`Repository 倉庫`

首先就跟 Docker 的 logo 一樣，我們可以把 Docker 看作一艘船，他在不同`海域(主機)`上都會有`一致的環境(大大解決了部署時環境不一致的世紀難題)`，而所有`應用(application)`就像是船上的`貨櫃(Container)`，我們透過將不同應用連同它的必須環境封裝成一個`容器(Container)` ，如此能夠避免為不同應用安裝多餘的`虛擬機(VM)`來運行`操作系統(OS)`。

有關`容器化部署`與`虛擬機部署`的差異和優劣本篇不打算深究，詳細可以查看參考鏈接中的第一條、或是第三條的官方網站會有更詳細的解說。

### Image 鏡像（映像檔）

Docker 中的 `Image 映像` 好比一個應用的`模板(Template)`，應用的`環境`、`驅動`、`應用本身` 等都將封裝到一個 Image 中。當我們將寫好的應用源碼包裝成一個 Image 之後，我們就可以以此為模板創建出許多個`運行實例`，也就是接下來要提到的 `Container 容器`。

### Container 容器

`Image` 和 `Container` 的關係與 JS 中`對象(object)`和`原型(prototype)`的關係相似：Image 作為應用的`環境模板`，而 Container 則是基於 Image 創建出來的`應用實例`。由於 Docker 中 `Image` 是`唯獨(read-only)`的，因此創建出來的 Container 將在此基礎之上會建立一層可寫層來保存運行時的修改。

#### 堆疊

Docker 團隊建議，一個 Image 最好只包含一個應用(application)，而一個實際的生產環境的程序往往會用到多個不同的應用來組合出一個服務，為此我們就必須使用`堆疊`的方式來構建出最終的產物。

我們可以先拉取最基礎的 Image，運行後產生一個 Container，並在運行的 Container 之上進行一次修改，然後將修改後的 Container 重新封裝成第一層的 Image。如此循環之下我們就能夠透過堆疊建造出一個包含多個環境的應用程序。

### Repository 倉庫

了解了 Image 和 Container 之間的關係之後，我們發現 Docker 的中間產物單位就是一個個的 Image（一個可運行的鏡像）。`Docker Hub` 就是一個與 Github 類似的存在，Github 保存了許多項目的`源代碼(Source Code)`，而 Docker Hub 則是保存了許多`可運行的鏡像(Image)`，我們將 Image `拉(pull)` 下來之後就可以基於 Image 建立一個`運行實例(Container)`。

## 基於 node 環境的簡單 Web 項目

接下來我們以一個基於 node 環境的簡單 Web 項目的部署作為範例

### Install 安裝

首先要使用 docker 當然第一步就是安裝，以下給出四種操作系統環境的安裝（這邊就簡單略過，網上都有很多資源）：

#### Windows / Mac 環境

可以直接到官網：<a href="https://www.docker.com/get-started">https://www.docker.com/get-started</a>

安裝 Docker Desktop 桌面客戶端啟用 Docker

#### Ubuntu Linux 環境

使用 atp 安裝：<a href="https://blog.csdn.net/LuffysMan/article/details/89393965">https://blog.csdn.net/LuffysMan/article/details/89393965</a>

#### CentOS Linux 環境

使用 yum 安裝：<a href="https://www.cnblogs.com/tomingto/p/11439478.html">https://www.cnblogs.com/tomingto/p/11439478.html</a>

### 項目內容

接下來我們先創建一個極簡的前端小項目：

- 項目結構

```
test-docker/
|- index.html
|- index.js
|- package.json
|- Dockerfile
```

- index.html：主頁面

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>test-docker</title>
  </head>
  <body>
    <div>Hello World</div>

    <script src="index.js"></script>
  </body>
</html>
```

- index.js：node 運行入口（使用 http 模塊啟動一個簡易的 node 服務）

```js
const http = require('http')
const fs = require('fs')

const ip = 'localhost'
const port = 3000

http
  .createServer(function (req, res) {
    console.log(`path: ${req.url}`)
    res.writeHead(200, { 'Content-Type': 'text/html' })
    const page = fs.readFileSync('./index.html', 'utf-8')
    res.end(page)
  })
  .listen(port, function () {
    console.log(`Server running at http://${ip}:${port}`)
  })
```

- package.json：node 項目配置項

```json
{
  "name": "test-docker",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

- Dockerfile：最重要的 Docker 配置文件（用於生成 Image）

```docker
FROM node:10.15.3-alpine  # node 環境
WORKDIR /app  # 工作目錄
ADD . /app  # 複製所有文件到工作目錄
RUN npm install  # 運行指令
EXPOSE 300  # 開放端口
CMD node index  # 命令行指令
```

### 建立 Image 鏡像

以上項目內容都配置好之後就能在根目錄執行

```bash
$ docker build . -t test-docker-demo
```

然後我們可以透過 `docker images` 來查看所有鏡像

```bash
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
test-docker-demo    latest              b3e614711748        3 seconds ago       71MB
node                10.15.3-alpine      56bc3a1ed035        14 months ago       71MB
```

### 建立並運行 Container

有了 `Image 鏡像`之後，我們就可以以 Image 為基礎創建出`運行實例(Container 容器)`了！

接下來我們就使用 `docker run` 來建立並運行一個新的 `Container`，並使用 `docker ps` 來查看所有容器：

```bash
$ docker run -d -p 3000:3000 test-docker-demo
f254b660a993bf56103496d16f69b262690ae2da8113e12814d87233c9b7010c

$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                   PORTS                             NAMES
f254b660a993        test-docker-demo    "/bin/sh -c 'node in…"   18 seconds ago      Up 17 seconds            300/tcp, 0.0.0.0:3000->3000/tcp   zen_golick
```

到此我們就可以訪問 http://localhost:3000 來訪問我們的網頁囉！

### 停止和銷毀

由於我們前面使用 `docker run -d` 使項目在後端運行，如果我們需要將應用停下來就要使用 `docker stop` 指令：

```bash
# docker stop <container-id>
$ docker stop f254b660a993
```

當我們需要銷毀舊的 Image 和 Container 時，要使用 `docker rm`、`docker rmi`

```bash
# docker rm <container-id>
$ docker rm f254b660a993
# docker rmi <image-id> 或 docker rmi <repository:tag>
$ docker rmi b3e614711748
$ docker rmi test-docker-demo
```

## Docker 常用指令

最後的最後我們來統一整理一下整個流程中會用到的相關 Docker 指令囉：

### 基本信息

```bash
# Docker 信息
$ docker info
# Docker 版本
$ docker version
```

### Image 鏡像相關

```bash
### 查看 Image
$ docker images

### 從 Docker Hub 查找 image
$ docker search [repo-name]
# ex:
$ docker search node

### 拉取 Image
$ docker pull [repo-name]:[tag]
# ex:
$ docker pull nginx
$ docker pull node:10.15.3-alpine

### 建立 Image 鏡像
$ docker build [OPTIONS] PATH | URL | -
# ex:
$ docker build . -t test-docker-demo # . 為指定 Dockerfile 的位置，-t 為指定 tag

### 刪除Image 鏡像（必須沒有 Container 使用才能刪除）
$ docker rmi [image-id | repo-name:tag]
# ex:
$ docker rmi 73c078631474
```

- 相關參數

| Command | Option | Description   |
| ------- | ------ | ------------- |
| build   | -t     | 指定 tag 名稱 |

### Container 容器相關

```bash
# 查詢正在執行的 Container
$ docker ps
# ex:
$ docker ps
$ docker ps -a # -a 查詢全部

# 基於 Image 創建並運行新的 Container
$ docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
# ex:
$ docker run -d -p 3000:3000 test-docker-demo
$ docker run -p 8000:8000 -it test-vue-nginx

# 查詢正在執行的 Container 輸出
$ docker exec -i -t [container-id] bash
# ex:
$ docker exec -i -t 017648f95503 bash

# 啟動已創建容器
$ docker start [container-id]

# 停止運行中容器
$ docker stop [container-id]

# 重啟運行中容器（不推薦，也不等價於 stop 和 start 連用）
$ docker restart [container-id]

# 刪除 Container（必須 stop 才能刪除）
$ docker rm [container-id]
```

- 相關參數

| Command | Option | Description                                |
| ------- | ------ | ------------------------------------------ |
| run     | -d     | 後台運行                                   |
|         | -p     | 指定端口映射                               |
|         | -i     | 交互模式                                   |
|         | -t     | 配置一個終端機                             |
| ps      | -a     | 查詢所有 Container(默認只查詢運行中的容器) |

# 結語

到此就全部結束啦！本篇簡單介紹了一下 Docker 的核心三個元素：Image、Container、Repository，並且建立部署一個極簡的 node 項目，最後附上常用的 docker 指令，供大家參考。下一篇將會紀錄使用 Docker 部署 vue 項目在 nginx 服務器中的操作過程。
