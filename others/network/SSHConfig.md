# 服务器 SSH 访问配置(主机别名 + 免密登陆)

@[TOC](文章目录)

<!-- TOC -->

- [服务器 SSH 访问配置(主机别名 + 免密登陆)](#服务器-ssh-访问配置主机别名--免密登陆)
- [正文](#正文)
  - [1. 流程](#1-流程)
  - [2. 本地配置](#2-本地配置)
    - [2.1 别名配置](#21-别名配置)
    - [2.2 生成密钥](#22-生成密钥)
  - [3. 远程服务器配置](#3-远程服务器配置)
    - [3.1 公钥配置](#31-公钥配置)
  - [4. 测试](#4-测试)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

之前在[SpringBoot 部署: 项目打包 & 手动部署到阿里云服务器上](https://blog.csdn.net/weixin_44691608/article/details/120517833)中有写过，这里单独提出来再做一次记录

## 1. 流程

本地服务器使用 ssh 协议访问远程服务器，同时配置免密码登陆与域名映射

![](https://picures.oss-cn-beijing.aliyuncs.com/img/ssh_config_1_target.png)

## 2. 本地配置

### 2.1 别名配置

修改 `~/.ssh/config` 文件，添加以下内容

```txt
Host remote
  HostName x.x.x.x
  Port 22
  User root
  IdentityFile ~/.ssh/id_rsa
```

`x.x.x.x` 改成远程服务器的公网 IP，`remote` 可以改成你喜欢的别名，`root` 则是登入用户

### 2.2 生成密钥

运行下面指令生成密钥

```bash
$ ssh-keygen -t rsa
```

密钥生成在 `~/.ssh/id_rsa`

公钥则是 `~/.ssh/id_rsa.pub`

## 3. 远程服务器配置

原始用法是(以 root 用户登陆目标 IP 机器)

```bash
$ ssh root@x.x.x.x
```

配置好 2.1 的别名之后就可以使用别名替换用户 + IP(如前面配置的别名是 `remote`)

```bash
$ ssh remote
```

第一次登陆需要输入密码

### 3.1 公钥配置

将本地主机上的公钥内容(`~/.ssh/id_rsa.pub` 的内容)写到远程服务器上的 `~/.ssh/authorized_keys` 文件内

```bash
[root@VM-12-2-centos ~]$ touch ~/.ssh/authorized_keys
[root@VM-12-2-centos ~]$ vim ~/.ssh/authorized_keys
```

## 4. 测试

最后输入 `exit` 退出远程服务器命令行，再次登陆测试免密登陆是否成功

```bash
[root@VM-12-2-centos ~]$ exit
$ ssh remote
```

# 其他资源

## 参考连接

| Title                                                     | Link                                                                                                                               |
| --------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| SpringBoot 部署: 项目打包 & 手动部署到阿里云服务器上 #4.1 | [https://blog.csdn.net/weixin_44691608/article/details/120517833](https://blog.csdn.net/weixin_44691608/article/details/120517833) |

## 完整代码示例

[]()
