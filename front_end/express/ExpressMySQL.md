# Express 实战: 连接 MySQL 数据库

@[TOC](文章目录)

<!-- TOC -->

- [Express 实战: 连接 MySQL 数据库](#express-实战-连接-mysql-数据库)
  - [简介](#简介)
  - [参考](#参考)
  - [完整示例代码](#完整示例代码)
- [正文](#正文)
  - [准备 MySQL 环境](#准备-mysql-环境)
    - [安装 MySQL](#安装-mysql)
    - [启动 MySQL](#启动-mysql)
    - [配置路径并进入 CLI](#配置路径并进入-cli)
    - [创建数据库、数据表、填充基本数据](#创建数据库数据表填充基本数据)
    - [导入数据](#导入数据)
  - [Express 项目](#express-项目)
    - [创建项目 & 引入依赖](#创建项目--引入依赖)
    - [项目结构](#项目结构)
    - [代码实现细节](#代码实现细节)
    - [运行 & 测试](#运行--测试)
- [结语](#结语)

<!-- /TOC -->

## 简介

本篇来介绍如何在 express 中台（or 后端）项目中连接 MySQL 数据库。借由 Node.js 的威力使得 JavaScript 的触手也得以探入后端开发的领域，形成一个全栈开发的体系。而在服务端的应用中为了进行数据持久化，势必离不开`数据库(database)`。

数据库也存在各式各样的类型和产品，按存储结构分可以分成`关系型数据库(RDBMS)`和`非关系型数据库(NoSQL)`。本篇要介绍并连接的是 MySQL，属于一个常见的关系型数据库，免费的嗯，马上来用用看。

## 参考

<table>
  <tr>
    <td>node连接Mysql报错ER_NOT_SUPPORTED_AUTH_MODE</td>
    <td><a href="https://www.cnblogs.com/jing-tian/p/11688073.html">https://www.cnblogs.com/jing-tian/p/11688073.html</a></td>
  </tr>
</table>

## 完整示例代码

<a href="https://github.com/superfreeeee/Blog-code/tree/main/front_end/node:express/express-mysql">https://github.com/superfreeeee/Blog-code/tree/main/front_end/node:express/express-mysql</a>

# 正文

## 准备 MySQL 环境

首先在连接 MySQL 之前，他得要先存在啊，所以我们第一步骤先来准备 MySQL 数据库环境并插入一个表和三条数据做示例

### 安装 MySQL

傻瓜式安装，一路"下一步"就得了hh，附上安装地址

<a href="https://dev.mysql.com/downloads/mysql/">https://dev.mysql.com/downloads/mysql/</a>

注意！windows 用户别装错了，直接下 `MySQL Installer MSI` 不需要下 `ZIP Archive` 的版本

账号默认使用 `root` 用户，密码就随你高兴吧，本篇使用 `123456789` 做示例

### 启动 MySQL

mac 用户直接到 `系统偏好设定 > MySQL > Start MySQL Server` 就启动成功了（记得第一次启动前先 `Initialize Database` 初始化数据库并设置 root 密码）

windows 则要进入到刚刚安装好的 MySQL 目录下的 bin 目录中运行 `mysqld-nt --install` 安装为一个 windows 服务，再使用 `net start mysql` 来启动 mysql 服务

### 配置路径并进入 CLI

mac 用户可以在 `~/.bash_profile` 中添加 mysql 的路径（默认应该在 `/usr/local/mysql`）

windows 则到高级环境配置中配置 MYSQL_HOME 和 PATH

| OS      | Variable   | Value                                   |
| ------- | ---------- | --------------------------------------- |
| mac     | MYSQL_HOME | /usr/local/mysql                        |
|         | PATH       | \$PATH:\$MYSQL_HOME/bin                 |
| windows | MYSQL_HOME | C:\Program Files\MySQL\MySQL Server xxx |
|         | PATH       | %MYSQL_HOME%\bin                        |

下面的步骤就统一了：

```bash
# root 用户登入，-p 指定输入密码
$ mysql -u root -p
Enter password:  # 输入密码，不会显示字符，敲就对了

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 13
Server version: 8.0.22 MySQL Community Server - GPL

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
# 到此变成功进入 MySQL CLI
```

### 创建数据库、数据表、填充基本数据

接下来我们使用 .sql 文件来统一创建数据库、数据表，并填充三条基本数据方便等下使用

> express_mysql_db.sql

```sql
# 创建空数据库
DROP DATABASE IF EXISTS express_mysql_db;
CREATE DATABASE express_mysql_db;

USE express_mysql_db;

# 创建 user 表
CREATE TABLE `user` (
    `id` INT(11) NOT NULL AUTO_INCREMENT,
    `name` VARCHAR(20) NOT NULL,
    `password` VARCHAR(60) NOT NULL,
    PRIMARY KEY (`id`)
)ENGINE=InnoDB DEFAULT CHARSET=utf8;

# 插入三条测试用数据
INSERT INTO user (`name`, `password`) VALUES ('user1', 'password1');
INSERT INTO user (`name`, `password`) VALUES ('user2', 'password2');
INSERT INTO user (`name`, `password`) VALUES ('user3', 'password3');
```

### 导入数据

一条一条指令敲太麻烦了，而且敲错还要重来，所以我们上面才使用 .sql 文件将指令脚本化，接下来我们就能够直接在 mysql CLI 里面一键导入：

```bash
# 这边填上自己的文件的真实路径
mysql> source xxx/express_mysql_db.sql;
# 查看数据库中所有表
mysql> show tables;
+----------------------------+
| Tables_in_express_mysql_db |
+----------------------------+
| user                       |
+----------------------------+
1 row in set (0.00 sec)
# 查看 user 表的列说明
mysql> show columns from user;
+----------+-------------+------+-----+---------+----------------+
| Field    | Type        | Null | Key | Default | Extra          |
+----------+-------------+------+-----+---------+----------------+
| id       | int         | NO   | PRI | NULL    | auto_increment |
| name     | varchar(20) | NO   |     | NULL    |                |
| password | varchar(60) | NO   |     | NULL    |                |
+----------+-------------+------+-----+---------+----------------+
3 rows in set (0.03 sec)
# 查看 user 表的所有信息
mysql> select * from user;
+----+-------+-----------+
| id | name  | password  |
+----+-------+-----------+
|  1 | user1 | password1 |
|  2 | user2 | password2 |
|  3 | user3 | password3 |
+----+-------+-----------+
3 rows in set (0.00 sec)
```

到此我们就完整数据库环境和测试数据的准备了

## Express 项目

数据库环境准备还是比较麻烦的，占用了不少篇幅，下面就来构建我们的主角：express 搭建的后端并连接 MySQL 数据库

### 创建项目 & 引入依赖

透过 `yarn init -y` 命令初始化 node 项目，并引入所需要的依赖并加上启动脚本，完整操作流程如下

1. `yarn init -y`：初始化 node 项目，会产生一个 `package.json` 文件
2. `yarn add express mysql`：引入 express、mysql 第三方库
3. 创建 `src` 目录并加入入口文件和启动脚本

> package.json

```json
{
  "name": "express-mysql",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "scripts": {
    // 项目启动脚本
    "start": "node src/index.js"
  },
  // 运行时依赖
  "dependencies": {
    "express": "^4.17.1",
    "mysql": "^2.18.1"
  }
}
```

### 项目结构

老样子，先给出项目结构

```
/express-mysql
├── express_mysql_db.sql // 数据库初始化脚本
├── package.json
├── src
│   ├── config.js // 启动配置信息
│   ├── db.js     // 数据库连接
│   └── index.js  // 服务主入口
└── yarn.lock
```

`express_mysql_db.sql` 上面已经介绍过了，下面我们就来填充三个主要的 js 文件

### 代码实现细节

首先先定义我们的配置信息 `config.js`

> config.js：启动配置

```js
module.exports = {
  port: 3000, // express 服务启动端口
  /* 数据库相关配置 */
  db: {
    host: 'localhost', // 主机名
    port: 3306,        // MySQL 默认端口为 3306
    user: 'root',          // 使用 root 用户登入 MySQL
    password: '123456789', // MySQL 密码，用你自己的
    database: 'express_mysql_db' // 使用数据库
  }
}
```

第二步我们需要建立 MySQL 的`连接(connection)`，并将连接实例导出(export)

> db.js：创建连接实例

```js
const mysql = require('mysql')

const config = require('./config').db // 获取数据库配置信息

module.exports = mysql.createConnection(config) // mysql.createConnection 方法创建连接实例
```

最后我们在默认路由中加入查找 user 表的方法并返回结果

> index.js：启动 express 服务

```js
const express = require('express')

const connection = require('./db') // 获取连接实例
const { port } = require('./config') // 获取启动参数

const app = express()

app.get('/', (req, res, next) => {
  /* 使用 connection.query 来执行 sql 语句 */
  // 第一个参数为 sql 语句，可以透过 js 自由组合
  // 第二个参数为回调函数，err 表示查询异常、第二个参数则为查询结果（这里的查询结果为多个用户行）
  connection.query('select * from user', (err, users) => {
    if (err) {
      res.send('query error')
    } else {
      // 将 MySQL 查询结果作为路由返回值
      res.send(users)
    }
  })
})

app.listen(port, () => {
  console.log(`express server listen at http://localhost:${port}`)
})
```

### 运行 & 测试

最后我们透过 `yarn start` 运行 express 服务：

```bash
$ yarn start
yarn run v1.22.10
node src/index.js
express server listen at http://localhost:3000
```

并访问 `http://localhost:3000` 就能看到运行结果辣！

![](https://picures.oss-cn-beijing.aliyuncs.com/img/express_mysql_sample.png)

# 结语

Node.js 使得 JS 语言能够涉足服务端开发的部分，成功引起全栈开发的浪潮。诸多数据库连接的 js 第三方库也到处都是，不只是本篇用到的 mysql，其他例如 mongodb 作为非关系型数据库的代表，说明 express 作为轻量型后端模版或是 demo 开发的能力也是越来越完备，供大家参考。（有空读者再来写一篇关于连接 MongoDB 的，作为 NoSQL 数据库的代表）
