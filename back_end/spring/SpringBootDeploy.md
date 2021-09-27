# SpringBoot 部署: 项目打包 & 手动部署到阿里云服务器上

@[TOC](文章目录)

<!-- TOC -->

- [SpringBoot 部署: 项目打包 & 手动部署到阿里云服务器上](#springboot-部署-项目打包--手动部署到阿里云服务器上)
- [前言](#前言)
- [正文](#正文)
  - [1. 环境准备 & 部署目标](#1-环境准备--部署目标)
  - [2. 准备 SpringBoot 项目](#2-准备-springboot-项目)
  - [3. 准备云服务器](#3-准备云服务器)
  - [4. 服务器环境配置](#4-服务器环境配置)
    - [4.1 域名别名 & ssh 免密登录](#41-域名别名--ssh-免密登录)
      - [4.1.1 域名别名](#411-域名别名)
      - [4.1.2 ssh 免密登陆](#412-ssh-免密登陆)
    - [4.2 Java 环境准备（使用 yum 自动安装）](#42-java-环境准备使用-yum-自动安装)
    - [4.3 MySQL 环境准备](#43-mysql-环境准备)
    - [4.3.1 MySQL 下载](#431-mysql-下载)
    - [4.3.2 修改密码](#432-修改密码)
    - [4.3.3 修改访问权限](#433-修改访问权限)
    - [4.3.4 修改 mysql 编码](#434-修改-mysql-编码)
    - [4.3.5 阿里云服务器添加安全组](#435-阿里云服务器添加安全组)
  - [5. 手动部署项目](#5-手动部署项目)
    - [5.1 打包项目](#51-打包项目)
    - [5.2 启动项目](#52-启动项目)
    - [5.3 停止后台运行项目](#53-停止后台运行项目)
  - [6. Docker 上部署](#6-docker-上部署)
    - [6.1 Docker 安装](#61-docker-安装)
    - [6.2 Docker 打包配置](#62-docker-打包配置)
    - [6.3 制作镜像 & 运行到容器当中](#63-制作镜像--运行到容器当中)
    - [6.4 访问应用端口](#64-访问应用端口)
    - [6.5 停止 docker 服务](#65-停止-docker-服务)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

之前写过一些 SpringBoot，也摸过一些 Spring Cloud 的东西，但是还没有针对后端的部署好好做过一次整理。

本篇就要带大家来做一次将 SpringBoot 项目部署到阿里云或是其他任意的云服务器上。

# 正文

## 1. 环境准备 & 部署目标

首先在开始之前我们先明确一下各个依赖的版本、操作系统环境等信息

- 项目依赖
  - SpringBoot `2.5.5`
  - Swagger2 `2.7.0`
- 系统环境
  - CentOS `7.9 x64位` 
  - MySQL Community Server `5.7.35`

列出来主要是避免版本不一致产生的问题，下面我们马上开始后端环境的部署准备

## 2. 准备 SpringBoot 项目

准备项目的部分我们就不再多说了，网上教程一大把，最终的目标就是准备一个 SpringBoot 的项目，然后使用 MySQL 数据库支持就行啦。真的想搞一个全新的就上下面的网址自己初始化一下啦

- SpringBoot 项目初始化入口

[https://start.spring.io/](https://start.spring.io/)

## 3. 准备云服务器

云服务器的部分我选择的是阿里云，当然什么腾讯云、华为云都差不多，哪个便宜好用用哪个咯。不过不同服务器用起来大同小异，不同服务器的操作也都可以互相借鉴咯。

- 阿里云服务器

[https://www.aliyun.com/product/ecs](https://www.aliyun.com/product/ecs)

## 4. 服务器环境配置

SpringBoot 项目和云服务器都准备好之后，我们就可以开始来配置啦

### 4.1 域名别名 & ssh 免密登录

首先第一步是先配置一下 ssh 的访问方式，后面会进行多次 ssh 访问，所以配一下域名别名和免密登陆还是比较方便的哈

#### 4.1.1 域名别名

我们知道我们访问云服务器的时候通常都需要使用 `[username]@[ip]` 的形式，有可能就是会出现这样 `root@2.123.231.3` 这种难记的 ip，这时候配置一个别名就相当方便，例如取名为 `remote` 就可以使用 remote 来取代 ip 地址，如 `root@remote`

作者使用的是 macOS 系统，linux 的用户应该大同小异，window 我就不清楚啦hh

首先去到 `~/.ssh/` 目录下看到 ssh 相关的配置

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_deploy_1_ssh_config.png)

由于作者已经配置过一些东西，所以如果读者找不到这个目录或文件的话可以自己手动建一下，接下来往这个文件内填入下面的内容

```
Host remote
  HostName x.x.x.x
  Port 22
  User root
  IdentityFile ~/.ssh/id_rsa
```

`remote` 为你想取的别名，取别的也行；HostName 后面则是放原始的 ip 地址

#### 4.1.2 ssh 免密登陆

看到下面有一个 IdentityFile 指向了 id_rsa 相信也有人好奇这是个啥吧，这其实就是接下来我们要做的免密登陆需要用到的文件

我们使用 `ssh-keygen` 指令来生成 ssh 访问的公钥和私钥

```bash
$ ssh-keygen -t rsa
```

然后一路回车下去看到 `Your public key has been saved in ~/.ssh/id_rsa.pub.` 就行啦，接下来则是将里面的公钥复制出来等下要用到

```bash
$ cat ~/.ssh/id_rsa.pub  # 复制下面出现的内容
```

接下来先登录云服务器，由于这次还没添加好 ssh 密钥所以还是需要输入密码（不过域名别名已经可以用了！）

```bash
$ ssh remote
```

进入云服务器的命令行界面后，一样找到 `~/.ssh` 目录下，没有就自己创一个（由于使用的是 root 用户登陆，所以 `~/.ssh` 的实际绝对路径为 `/root/.ssh`）。

接下来在这个目录下创建 `authorized_keys` 文件

```bash
$ cd /root
$ mkdir .ssh
$ cd .ssh
$ touch authorized_keys
```

接下来则是把刚刚复制过来的公钥贴到这个文件里面就大功告成啦，退出再重新登陆就可以看到效果啦

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_deploy_2_ssh_not_password.png)

不用密码也不用记难记的 ip 地址啦

### 4.2 Java 环境准备（使用 yum 自动安装）

配置好 ssh 的访问之后接下来就先进入到云服务器的命令行下，我们有几个东西要装一装，首先第一步当然是先把 Java 环境配置好。本篇使用的是 Java 8 的版本

第一步我们可以使用 `yum -y list java*` 指令查询符合条件的依赖列表

```bash
$ yum -y list java*
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_deploy_3_java_from_yum.png)

从列表中找到 `java-1.8.0-openjdk.x86_64` 就是我们要装的依赖，执行

```bash
$ yum install -y java-1.8.0-openjdk.x86_64
```

安装完之后就可以运行 `java` 命令看有没有安装成功咯

```bash
$ java -version
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_deploy_4_java_version.png)

### 4.3 MySQL 环境准备

下一步则是 MySQL 的环境准备，一样我们使用 yum 来安装

### 4.3.1 MySQL 下载

首先我们先安装 mysql 需要的 yum 源

```bash
$ wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
```

然后就可以使用 yum 进行安装

```bash
$ yum -y install mysql57-community-release-el7-10.noarch.rpm
```

最后是安装 MySQL 服务器

```bash
$ yum -y install mysql-community-server
```

都安装成功之后我们需要启动一下 mysql 服务

```bash
$ systemctl start mysqld.service
```

然后可以使用指令查看 mysql 服务的运行状态

```bash
$ systemctl status mysqld.service
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_deploy_5_mysql_active.png)

看到 `active (running)` 表示正常运行

### 4.3.2 修改密码

运行成功之后，我们第一步要做的就是先改掉默认配置的密码，执行下面命令来找到这个密码

```bash
$ grep "password" /var/log/mysqld.log
2021-09-27T03:17:50.521321Z 1 [Note] A temporary password is generated for root@localhost: blablablabla
```

找到上面这一行 blablabla 位置的你的密码，用来登陆 mysql

```bash
$ mysql -u root -p
```

输入完密码之后应该就能够正常进入了

下一步则是要修改密码

```bash
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'your new password';
```

在上面的 `your new password` 的部分改成你的密码，注意要同时存在大写、小写字母和特殊密码才能配置成功

### 4.3.3 修改访问权限

改完密码先不急，我们顺便把访问权限也开起来，毕竟我们可能在本地测试的时候也是需要连上远程的 mysql 服务器的，所以需要把访问权限打开（这时候密码就很重要了，保护好密码避免被盗数据）

```bash
mysql> grant all privileges on *.* to 'root'@'%' identified by 'password' with grant option;
```

这里直接使用 `%` 开启对所有域名的权限，如果你只想限定指定的 ip 地址能够访问 mysql 的话将 % 改成你的 ip 就行啦

修改完权限之后需要执行下面命令才能生效

```bash
mysql> flush privileges;
```

### 4.3.4 修改 mysql 编码

除了权限之外我们还需要修改一下 mysql 的编码

```bash
$ vim /etc/my.cnf
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_deploy_13_mysql_charset.png)

在该文件里面加入下面两段

```
[client]
default-character-set=utf8
```

```
character-set-server=utf8
collation-server=utf8_general_ci
```

接下来重新启动 mysql 服务就行啦

```bash
$ systemctl restart mysqld.service
```

一样可以运行一下 status 查看运行状况

```bash
$ systemctl status mysqld.service
```

然后我们还可以进到 mysql 里面看看有没有成功配置编码规则

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_deploy_14_mysql_charset_result.png)

### 4.3.5 阿里云服务器添加安全组

都配置好之后还要到阿里云上添加安全组配置，才能够从公网访问到阿里云服务器的指定端口哦

路径为：阿里云ECS控制台 > 实例 > 点击实例 > 安全组 > 配置规则

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_deploy_6_open_port.png)

接下来在安全组里面添加开放 3306 端口

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_deploy_7_open_port_config.png)

后续服务部署在如 8080、8081 等端口也是要在这边开放，当然你也可以现在一起配好

到此 mysql 相关的配置就搞定啦

## 5. 手动部署项目

接下来的步骤比较傻，我们先来搞一次手动部署项目的方式，一种是直接在服务器上安装 git 和 maven，直接在服务器上完成打包的步骤，不过本篇就傻一些，先在本地打包好之后直接传输到远程启动

### 5.1 打包项目

前面准备好的 SpringBoot 项目需要用下面命令进行打包的，当然你也可以直接在如 IDEA 里使用双击图形化命令

```bash
$ mvn clean package
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_deploy_8_idea_mvn_package.png)

项目打包好之后应该会在 target 目录下产生一个 jar 包

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_deploy_9_target_jar.png)

接下来我们使用 `scp [targetFile] [user]@[ip]:[path]` 命令将它复制到远程服务器上

```bash
$ scp target/logger-0.0.1-SNAPSHOT.jar root@remote:/home/DataBG
```

文件名、路径、ip 地址/别名、目标目录等都可以依据实际情况修改

### 5.2 启动项目

复制上去之后我们就可以到服务器上看看有没有上传上去的文件啦（**记得打包的时候数据库配置要正确啊**）

```bash
$ ssh remote
[xxx]$ cs /home/DataBG
[xxx]$ cs ls
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_deploy_10_jar_on_server.png)

当然你的目录应该跟我的是不一样的，直接放在 home 目录下其实就可以了

接下来我们就可以使用 `java` 命令直接启动项目啦

```bash
$ java -jar logger-0.0.1-SNAPSHOT.jar
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_deploy_11_run_project_local.png)

看到如上面的输出就代表启动成功啦，我们还可以稍微修改一下启动指令

```bash
$ nohup java -jar logger-0.0.1-SNAPSHOT.jar > $filename 2>&1 &
```

`nohup` 表示指令运行在后台；`> filename` 则是将标准输出流写入到指令文件里面啦

### 5.3 停止后台运行项目

接下来如果你是在后台运行项目的话，就需要找到他把他停下来才可以重新启动修改过并重新打包的项目啦

```bash
$ ps aux | grep "java -jar" | grep -v "grep"
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_deploy_12_nohup_killing.png)

root 后面的数字就是进城 ID，以本次运行来说进程号为 `18121`，要停止该进程则要运行

```bash
$ kill -9 18121
```

才能停止进程，再次运行上面那条查询指令就能看到没有相关进程了

## 6. Docker 上部署

前面的部分我们与读者一起在阿里云上手动部署了一个项目，然而这个项目属于一个裸进程，直接运行在服务器环境上的，这样其实不太好，一直找进程杀进程也是挺危险的。

这时候我们就可以选择使用 Docker 来部署我们的项目

### 6.1 Docker 安装

首先我们需要在服务器上安装一下 Docker，官方的教程挺好的简单易懂，看看菜鸟的也还行（都贴在参考链接里面啦）

```bash
$ yum install -y yum-utils device-mapper-persistent-data lvm2
$ yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
$ yum install docker-ce docker-ce-cli containerd.io
```

依序执行上述指令，如果权限不够自己在前面加个 `sudo`

安装完之后应该能够看到下面两个包被安装了

```bash
$ yum list docker*
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_deploy_15_docker_installed.png)

接下来可以启动 docker

```bash
$ systemctl start docker
$ systemctl status docker
$ docker version
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_deploy_16_docker_server_active.png)

如果有正确启动的话应该能够看到下半部关于 Server 的信息，另外你也可以透过 `docker images` 命令来检查是否正确启动

### 6.2 Docker 打包配置

接下来我们到项目的根目录下建立一个 `Dockerfile` 文件，也是我们的项目部署到 docker 上的时候会用到的配置文件

```bash
$ touch Dockerfile
```

填入一下内容

```dockerfile
# Docker image for springboot file run
# VERSION 0.0.1
# Author: superfree
# environment
FROM java:8
# author
MAINTAINER superfree <superfreeeee@gmail.com>
# volume 宗卷
VOLUME /tmp

ADD logger-0.0.1-SNAPSHOT.jar app.jar
RUN bash -c 'touch /app.jar'
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

内容就不详细解释，大致上就是说：
- `FROM java:8` 以 java8 的镜像为基础
- `ADD xxx yyy` 将 xxx 文件导入并改名为 yyy
- `RUN xxx ENTRYPOINT` 运行指令 & 指定运行时参数等

接下来我们一样将这个文件复制到远程服务器上

```bash
$ scp Dockerfile root@remote:/home/DataBG
```

记得改成自己的域名和路径哈，还有要与刚刚的 jar 包放一起，否则路径不对

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_deploy_17_dockerfile_upload.png)

### 6.3 制作镜像 & 运行到容器当中

接下来我们就在服务器上运行几个指令

第一个是根据刚才的 Dockerfile 制作一个镜像

```bash
$ docker build -t youxian-logger .
```

`-t` 参数后面接的是你的项目名称，同时别忘了指定运行目录为 `.` 当前目录

接下来我们就可以用 `docker images` 看到制作好的镜像，第一次运行的时候还会下载 java8 的镜像比较久

```bash
$ docker images
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_deploy_18_build_image.png)

有了镜像之后我们就能够拿来运行了，运行的时候会生成一个容器

```bash
$ docker run -d -p 8080:8080 youxian-logger
```

`-d` 参数表示运行在后台，`-p` 则是指定端口映射（`a:b` 意思是：将机器本身的 a 端口映射到 docker 容器的 b 端口），最后则是刚刚制作的镜像名称 `youxian-logger`，你也可以填上镜像的 ID

然后我们可以运行 `docker ps` 查看容器运行情况

```bash
$ docker ps -a
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_deploy_19_docker_container.png)

### 6.4 访问应用端口

最后我们就可以访问自己写的应用看到端口运行的结果啦

![](https://picures.oss-cn-beijing.aliyuncs.com/img/springboot_deploy_20_application_result.png)

### 6.5 停止 docker 服务

停止 docker 上的应用就比较简单了，不用查什么傻逼端口

```bash
$ docker stop [Container ID]
```

停止后就可以删除容器或是镜像

```bash
$ docker rm [Container ID]
$ docker rmi [Image ID]
```

# 结语

其实阿里云部署 SpringBoot 的操作已经玩过好多遍了，从大二就开始尝试到现在才真正比较知道自己在干嘛，赶紧记录下来，之后更多的做后端相关的开发相信也会越来越顺手。能够整好自己的部署环境就已经成功一大半啦哈哈。下次可能会再尝试加上 jenkins 服务来做 CI/CD 的工作流

# 其他资源

## 参考连接

| Title                                                       | Link                                                                                                                                 |
| ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| Spring Initializr                                           | [https://start.spring.io/](https://start.spring.io/)                                                                                 |
| 阿里云ECS                                                   | [https://www.aliyun.com/product/ecs](https://www.aliyun.com/product/ecs)                                                             |
| ssh免密码登录配置方法                                       | [https://www.cnblogs.com/wenxingxu/p/9597307.html](https://www.cnblogs.com/wenxingxu/p/9597307.html)                                 |
| 阿里云部署SpringBoot                                        | [https://blog.csdn.net/shuwanghong/article/details/88547315](https://blog.csdn.net/shuwanghong/article/details/88547315)             |
| CentOS7安装MySQL（完整版）                                  | [https://blog.csdn.net/qq_36582604/article/details/80526287](https://blog.csdn.net/qq_36582604/article/details/80526287)             |
| 利用ssh传输文件                                             | [https://www.cnblogs.com/jiangyao/archive/2011/01/26/1945570.html](https://www.cnblogs.com/jiangyao/archive/2011/01/26/1945570.html) |
| ERROR 1273 (HY000): Unknown collation: 'utf8mb4_0900_ai_ci' | [https://blog.csdn.net/weixin_43728574/article/details/103592533](https://blog.csdn.net/weixin_43728574/article/details/103592533)   |
| CentOS7设置环境变量                                         | [https://www.cnblogs.com/wucongzhou/p/12579468.html](https://www.cnblogs.com/wucongzhou/p/12579468.html)                             |
| springboot 在linux后台运行                                  | [https://blog.csdn.net/yuhui123999/article/details/80593750](https://blog.csdn.net/yuhui123999/article/details/80593750)             |
| Linux下 SpringBoot jar项目后台运行、查看、停用              | [https://www.cnblogs.com/xiaoshen666/p/11021160.html](https://www.cnblogs.com/xiaoshen666/p/11021160.html)                           |
| shell脚本按当前日期输出日志的实现                           | [https://www.jb51.net/article/186140.htm](https://www.jb51.net/article/186140.htm)                                                   |
| springboot 配置多环境切换                                   | [https://blog.csdn.net/weixin_42130471/article/details/90413324](https://blog.csdn.net/weixin_42130471/article/details/90413324)     |
| 使用spring profile实现多环境切换                            | [https://www.cnblogs.com/hero123/p/10861693.html](https://www.cnblogs.com/hero123/p/10861693.html)                                   |
| Install Docker Engine on CentOS                             | [https://docs.docker.com/engine/install/centos/](https://docs.docker.com/engine/install/centos/)                                     |
| CentOS Docker 安装 - 菜鸟                                   | [https://www.runoob.com/docker/centos-docker-install.html](https://www.runoob.com/docker/centos-docker-install.html)                 |
| Centos7上安装docker                                         | [https://www.cnblogs.com/yufeng218/p/8370670.html](https://www.cnblogs.com/yufeng218/p/8370670.html)                                 |
| Docker部署SpringBoot项目                                    | [https://www.jianshu.com/p/397929dbc27d](https://www.jianshu.com/p/397929dbc27d)                                                     |
| 查看docker运行中的命令行输出                                | [https://blog.csdn.net/zcgyq/article/details/83084017](https://blog.csdn.net/zcgyq/article/details/83084017)                         |
| Docker容器应用日志查看                                      | [https://blog.csdn.net/benben_2015/article/details/80708723](https://blog.csdn.net/benben_2015/article/details/80708723)             |

## 完整代码示例

这里就不贴啦，关键代码都写在文章中了
