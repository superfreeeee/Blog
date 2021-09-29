# Jenkins 自动化部署 SpringBoot 项目配置

@[TOC](文章目录)

<!-- TOC -->

- [Jenkins 自动化部署 SpringBoot 项目配置](#jenkins-自动化部署-springboot-项目配置)
- [前言](#前言)
- [正文](#正文)
  - [1. 环境准备](#1-环境准备)
    - [1.1 安装 Java](#11-安装-java)
      - [1.1.1 卸载 OpenJDK](#111-卸载-openjdk)
      - [1.1.2 下载 Java 8 压缩包](#112-下载-java-8-压缩包)
      - [1.1.3 设置环境变量](#113-设置环境变量)
    - [1.2 安装 Maven](#12-安装-maven)
      - [1.2.1 下载 maven](#121-下载-maven)
      - [1.2.2 配置环境变量](#122-配置环境变量)
      - [1.2.3 maven 源 & 本地仓库配置](#123-maven-源--本地仓库配置)
    - [1.3 安装 Git](#13-安装-git)
    - [1.4 安装 Jenkins](#14-安装-jenkins)
      - [1.4.1 下载 Jenkins](#141-下载-jenkins)
      - [1.4.2 Jenkins 启动配置](#142-jenkins-启动配置)
      - [1.4.3 启动 Jenkins 服务](#143-启动-jenkins-服务)
    - [1.5 安装 Docker](#15-安装-docker)
  - [2. Jenkins 流水线配置](#2-jenkins-流水线配置)
    - [2.1 初始化配置](#21-初始化配置)
    - [2.2 配置本地工具](#22-配置本地工具)
      - [2.2.1 JDK 配置](#221-jdk-配置)
      - [2.2.2 Git 路径](#222-git-路径)
      - [2.2.3 Maven 配置](#223-maven-配置)
    - [2.3 配置凭证](#23-配置凭证)
      - [2.3.1 Github 用户凭证](#231-github-用户凭证)
      - [2.3.2 Github Webhook 访问 Token](#232-github-webhook-访问-token)
    - [2.4 创建 SpringBoot 部署任务](#24-创建-springboot-部署任务)
      - [2.4.1 基本信息](#241-基本信息)
      - [2.4.2 仓库源码](#242-仓库源码)
      - [2.4.3 构建触发时机](#243-构建触发时机)
      - [2.4.4 具体构建任务步骤](#244-具体构建任务步骤)
      - [2.4.5 运行构建任务](#245-运行构建任务)
- [结语](#结语)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 前言

今天带大家来配置并部署一个 jenkins 服务来实现开发流程的 CI/CD，本篇就不做过多的解释了，主要是阐述操作流程

下面在命令行中看到 `[root@remote]%` 开头表示在服务器的命令行环境下，`$` 开头则是本地的命令行

# 正文

## 1. 环境准备

### 1.1 安装 Java

如果你（其实是说我自己啦hh）之前是使用 yum 安装 Java，那你也有可能不小心安装到了 open-jdk 的旧版本，然而我们还是更推荐安装 Oracle 上比较完整的 JDK 版本

- 两者者的差异看这里：[https://www.cnblogs.com/sxdcgaq8080/p/7487369.html](https://www.cnblogs.com/sxdcgaq8080/p/7487369.html)

#### 1.1.1 卸载 OpenJDK

卸载 open-jdk 的步骤比较特殊

先查找已安装的相关包

```bash
[root@remote]% rpm -qa | grep openjdk
```

接下来把所有相关包都删了（对每一个包替换下面的 `pkg-name` 一个一个删干净）

```bash
[root@remote]% rpm -e --nodeps [pkg-name]
```

#### 1.1.2 下载 Java 8 压缩包

- Oracle 上的 Java8 JDK 安装路径：[https://www.oracle.com/java/technologies/javase/javase8u211-later-archive-downloads.html](https://www.oracle.com/java/technologies/javase/javase8u211-later-archive-downloads.html)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_1_java_jdk.png)

我们要选的是这个 linux64 位的 tar.gz 版本

下载下来之后，将其上传到服务器的 `/usr/local/java` 目录下，并进行解压缩

```bash
$ scp jdk-8u291-linux-x64.tar.gz root@remote:/usr/local/java/
$ ssh remote
[root@remote ~]% cd /usr/local/java
[root@remote java]% tar -xzvf jdk-8u291-linux-x64.tar.gz
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_2_jdk_tar.png)

#### 1.1.3 设置环境变量

接下来配置一下服务器上的 java 环境变量，也就是大家耳熟能详的 JAVA_HOME

```bash
[root@remote]% vim ~/.bash_profile
```

在里面添加两个环境变量

```sh
export JAVA_HOME=/usr/local/java/jdk1.8.0_291
export PATH=$PATH:$JAVA_HOME/bin
```

然后使用 `source` 使其生效

```bash
[root@remote]% source ~/.bash_profile
```

设置好之后检查一下 java 命令的版本

```bash
[root@remote]% java -version
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_3_java_version.png)

看到上面的输出表示配置正确啦

### 1.2 安装 Maven

第二部分我们先来安装一下 maven，后面打包 springboot 的时候会用到

#### 1.2.1 下载 maven

首先是去 apache 上下载 maven 的可运行包

- apache 下载链接：[https://maven.apache.org/download.cgi](https://maven.apache.org/download.cgi)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_4_maven_apache.png)

版本倒是无所谓，反正我们只是需要这个包而已

一样下载下来之后先上传到服务器上（路径 `/usr/local/maven`），然后解压缩

```bash
$ scp apache-maven-3.8.2-bin.tar.gz root@remote:/usr/local/maven/
$ ssh remote
[root@remote ~]% cd /usr/local/java
[root@remote maven]% tar -xzvf apache-maven-3.8.2-bin.tar.gz
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_5_maven_pkg.png)

#### 1.2.2 配置环境变量

一样 maven 也要配置一下环境变量，才能使用 `mvn` 指令

```bash
[root@remote]% vim ~/.bash_profile
```

加入下面两行

```sh
export MAVEN_HOME=/usr/local/maven/apache-maven-3.8.2
export PATH=$PATH:$MAVEN_HOME/bin
```

生效之后查看命令是否正常运行

```bash
[root@remote]% source ~/.bash_profile
[root@remote]% mvn -v
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_6_mvn_version.png)

#### 1.2.3 maven 源 & 本地仓库配置

接下来我们需要修改一下 maven 的默认配置

```bash
[root@remote]% vim /usr/local/maven/apache-maven-3.8.2/conf/settings.xml
```

- 配置 maven 源仓库（使用阿里源）

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_7_maven_config_mirror.png)

在 `<mirrors>` 标签下加入下面这段

```xml
    <mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>
    </mirror>
```

- 指定本地仓库位置（资源下载目录）

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_8_maven_config_localRepository.png)

在接近 setting 顶部的地方找到 `<localRepository>`，加入下面这段

```xml
  <localRepository>/usr/local/maven/repository</localRepository>
```

- 指定 JDK 版本

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_9_maven_config_jdk_version.png)

找到 `<profiles>` 加入下面这段

```xml
    <profile>
      <id>jdk-1.8</id>

      <activation>
        <jdk>1.8</jdk>
      </activation>
      
      <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
      </properties>
    </profile>
```

到此为止就配置好啦～

### 1.3 安装 Git

由于待会我们要建设的工作流是使用 jenkins 拉取 Github 的代码到本地然后进行打包后部署，所以 git 工具也是不能少

```bash
[root@remote]% yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker
```

我也不知道是不是这些工具其实，都装一装hh，下面这个才是 git 其实；最后查看一下版本就行了

```bash
[root@remote]% yum install git
[root@remote]% git --version
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_10_git_version.png)

版本低一些也无所谓，能用就行hh（git 应该也不会低到哪里去啦哈哈）

### 1.4 安装 Jenkins

第四个就是我们的主角啦 - Jenkins 老爷子

#### 1.4.1 下载 Jenkins

这次我们先直接使用 yum 来安装 jenkins，官方其实更推荐在 docker 容器内进行部署（不过 docker 目录映射的部分博主还没搞很懂，先这样玩hh）

```bash
[root@remote]% $ wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
[root@remote]% $ rpm --import http://pkg.jenkins.io/redhat-stable/jenkins.io.key
[root@remote]% $ yum install -y jenkins
```

依次运行上面几个指令直到 Completed，就算是安装完成啦

#### 1.4.2 Jenkins 启动配置

接下来我们到 `/etc/sysconfig/jenkins` 内进行一些启动相关的配置

```bash
[root@remote]% vim /etc/sysconfig/jenkins
```

- 修改启动端口

由于我们是在本地启动，还是不要占用 8080 这个热门的 tomcat 端口，改成 `8900`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_11_jenkins_start_config_port.png)

- 增加运行内存

接下来是由于 jenkins 执行任务的时候还是比较耗内存的，我们也可以在这里先配置好给他更多的启动可用内存

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_12_jenkins_start_config_memory.png)

找到 `JENKINS_JAVA_OPTIONS` 的选项，改成下面这句（其实就是加上后面几个启动参数）

```sh
JENKINS_JAVA_OPTIONS="-Djava.awt.headless=true -Xms1024m -Xmx2048m -XX:PermSize=256m -XX:MaxPermSize=512m"
```

- 修改权限

由于之后我们在 jenkins 内进行自动化部署的时候会使用到 docker 指令，然而可能存在权限不足的问题，因此我们可以把启动权限改成 `root`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_13_jenkins_start_config_root.png)

同时修改一下用户组

```bash
[root@remote]% gpasswd -a jenkins root
```

#### 1.4.3 启动 Jenkins 服务

都配置好了之后我们就可以来启动 Jenkins 服务了，顺便检查一下启动状态

```bash
[root@remote]% systemctl start jenkins
[root@remote]% systemctl status jenkins
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_14_jenkins_start_status.png)

### 1.5 安装 Docker

关于 Docker 的安装前一篇说过啦，自己去看

传送门：[SpringBoot 部署: 项目打包 & 手动部署到阿里云服务器上 #6.1 Docker 安装](https://blog.csdn.net/weixin_44691608/article/details/120517833)

## 2. Jenkins 流水线配置

安装安装了老半天，终于都搞好了，接下来打开 `http://[your-ip]:8900/` 开始正式的体验 jenkins 啦（记得改成你的 ip 啊）

### 2.1 初始化配置

刚开的时候应该是从这个页面开始

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_15_jenkins_init.png)

嗯人家也说的够清楚了，去 `/var/jenkins_home/secrets/initialAdminPassword` 找你的密码

```bash
[root@remote]% cat /var/jenkins_home/secrets/initialAdminPassword
```

注意如果你是使用 docker 安装的话可以试试

```bash
[root@remote]% docker logs [Container ID]
```

从标准输出来找到密码

进去之后选下面左边的，安装推荐的插件组就好啦

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_16_jenkins_plugins.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_17_jenkins_plugins_progress.png)

等到进度条跑完之后，是设置管理员用户密码，还是要保管好，忘记密码还是挺麻烦的哈

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_18_jenkins_user.png)

设置好之后他会给你一个重启按钮，接下来就进入到登陆页面啦

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_19_jenkins_login.png)

登陆之后就是熟悉的首页啦

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_20_jenkins_front_page.png)

### 2.2 配置本地工具

接下来 jenkins 是需要使用本地的 git、jdk、maven 等工具，所谓也需要配置一下路径，选择 `管理 Jenkins` > `Global Tool Configuration`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_41_jenkins_global_configuration.png)

#### 2.2.1 JDK 配置

首先是 JDK 的 JAVA_HOME 填上，也就是与系统上环境变量相同的路径就行了

```bash
[root@remote]% echo $JAVA_HOME
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_42_jenkins_global_configuration_1_java_home.png)

#### 2.2.2 Git 路径

第二个是 Git 路径

```bash
[root@remote]% which git
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_43_jenkins_global_configuration_2_git_path.png)

#### 2.2.3 Maven 配置

第三个是 Maven 路径的配置

```bash
[root@remote]% echo $MAVEN_HOME
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_44_jenkins_global_configuration_3_maven_home.png)

### 2.3 配置凭证

初始化好 jenkins 之后，先别急着创建流水线。我们先帮系统加上两个需要用到的凭证

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_21_jenkins_credential.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_22_jenkins_credential_entry.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_23_jenkins_credential_entry2.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_24_jenkins_credential_entry3.png)

#### 2.3.1 Github 用户凭证

第一个凭证我们先要在 jenkins 里面注册我们的 Github 用户，这样才可以正常的 pull 代码

首先先在远程机器上创建 ssh key

```bash
[root@remote]% ssh-keygen -t rsa
```

创建好之后第一步先把公钥加入到你的 Github 账号里面

点击用户头像 > `Setting`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_26_Github_sshkey_1.png)

找到 `SSH Key` 的选项 > `New SSH key`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_27_Github_sshkey_2.png)

接下来将你本地上的 `~/.ssh/id_rsa.pub` 也就是公钥贴到 key 的部分，然后取一个名字（title）

```bash
[root@remote]% cat ~/.ssh/id_rsa.pub
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_28_Github_sshkey_3.png)

接下来就可以在 jenkins 上的 `Add Credential` 中选择 `SSH Username with private key`，然后在下面的 `private key` 选择 `Add` 然后填入 `~/.ssh/id_rsa` 的私钥

```bash
[root@remote]% cat ~/.ssh/id_rsa
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_25_jenkins_credential_Github_user.png)

#### 2.3.2 Github Webhook 访问 Token

第二种凭证则是 Webhook 用到的，先到 Github 账户上 > `Settings` > `Developer Settings` > `Personal Access Tokens` > `Generate New Token` 创建一个可访问 token

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_29_Github_webhook_1.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_30_Github_webhook_2.png)

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_31_Github_webhook_3.png)

接下来则是把时间设为 `No expired`，然后勾选 `repo` 跟 `repo_hook` 两项

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_32_Github_webhook_4.png)

创建完成之后把 token 内容记好，接下来在 jenkins 里面再创建一个新的 credential，选择 `Secret text`，然后将 token 填在 `Secret` 的栏位

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_33_jenkins_credential_webhook.png)

### 2.4 创建 SpringBoot 部署任务

完成凭证之后就可以来创建任务啦，到主面板选择 `创建任务（新建专案）`

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_34_jenkins_job_1.png)

想一个名字之后，选择 `Free Style` 风格

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_35_jenkins_job_2.png)

#### 2.4.1 基本信息

接下来先来填任务的基本信息，把 Github 仓库的路径体填上

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_36_jenkins_job_3.png)

#### 2.4.2 仓库源码

后续我们需要透过 jenkins 自己操作 git 来下载源码，选择 git，填上仓库路径（用来 `git clone` 的那个哦），然后选择刚刚创建过的 Github 账户的凭证，以及想要部署的分支名称

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_37_jenkins_job_4.png)

#### 2.4.3 构建触发时机

另外我们可以设定构建的触发时机，平常可以使用手动点击建置一键部署，这边我们再额外加上刚刚创建的 Github webhook

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_38_jenkins_job_5.png)

同时别忘了到 Github 的仓库内开始 webhook：选择 `Settings` > `Webhooks` > `Add Webhook`，然后填入 jenkins 的路径（`http://[youre-ip]:8900/github-webhook/`），然后选择 json

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_40_jenkins_job_7.png)

#### 2.4.4 具体构建任务步骤

最后一部分就是具体的构建任务啦，假设我们的项目已经存在如下的 `Dockerfile` 配置

```Dockerfile
# Docker image for springboot file run
# VERSION 0.0.1
# Author: superfree
# environment
FROM java:8
# author
MAINTAINER superfree <superfreeeee@gmail.com>
# volume 宗卷
VOLUME /tmp

ADD target/logger-0.0.1-SNAPSHOT.jar app.jar
RUN bash -c 'touch /app.jar'
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

这时候我们就可以创造两个构建步骤

1. 第一个是调用顶层的 maven 指令进行项目打包（图片里面错了，不用加 maven 指令）

```
clean package
```

2. 第二步则是编写 docker 运行脚本

```sh
docker stop youxian-logger-pre ||true
docker rmi youxian-logger-pre ||true
docker build -t youxian-logger-pre .
docker run --rm --name youxian-logger-pre -d -p 8081:8081 youxian-logger-pre
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_39_jenkins_job_6.png)

#### 2.4.5 运行构建任务

最后去到任务里面点一下构建就能等待结果啦

![](https://picures.oss-cn-beijing.aliyuncs.com/img/jenkins_deploy_springboot_45_jenkins_build.png)

# 结语

整体来说其实配置是不难的，就是很繁琐，还会遇到一堆奇奇怪怪的问题，而且 Jenkins 消耗的资源还是挺多的，我用的阿里云 1 核 2 G 的一次运行 mysql + docker + jenkins 有点扛不住，内存平均消耗都在 90% 以上hh，之后看看能不能尝试部署到容器内来进行统一的内存管理调度

# 其他资源

## 参考连接

| Title                                                                                                    | Link                                                                                                                                                                                 |
| -------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| yum安装java的位置_Centos 7 查看使用YUM 安装的JDK路径并配置JAVA_HOME                                      | [https://blog.csdn.net/weixin_32375369/article/details/114555129](https://blog.csdn.net/weixin_32375369/article/details/114555129)                                                   |
| Centos7安装maven                                                                                         | [https://www.cnblogs.com/116970u/p/11211963.html](https://www.cnblogs.com/116970u/p/11211963.html)                                                                                   |
| Downloading Apache Maven 3.8.2                                                                           | [https://maven.apache.org/download.cgi](https://maven.apache.org/download.cgi)                                                                                                       |
| CentOS压缩与解压                                                                                         | [https://blog.csdn.net/shumeigang/article/details/80694719](https://blog.csdn.net/shumeigang/article/details/80694719)                                                               |
| 《阿里云服务器搭建》------ 安装Git                                                                       | [https://blog.csdn.net/tomatocc/article/details/83933041](https://blog.csdn.net/tomatocc/article/details/83933041)                                                                   |
| 安装Jenkins - 安装Jenkins                                                                                | [https://www.jenkins.io/zh/doc/book/installing/](https://www.jenkins.io/zh/doc/book/installing/)                                                                                     |
| centos下搭建Jenkins持续集成环境(安装jenkins)                                                             | [https://www.cnblogs.com/loveyouyou616/p/8714544.html](https://www.cnblogs.com/loveyouyou616/p/8714544.html)                                                                         |
| Public key for jenkins-2.288-1.1.noarch.rpm is not installed的解决方法                                   | [https://blog.csdn.net/u010662249/article/details/115896468](https://blog.csdn.net/u010662249/article/details/115896468)                                                             |
| jenkins credentials & git ssh 认证                                                                       | [https://blog.csdn.net/gw569453350game/article/details/51911179](https://blog.csdn.net/gw569453350game/article/details/51911179)                                                     |
| 怎么样使用yum来安装、卸载jdk                                                                             | [https://www.cnblogs.com/dingjiaoyang/p/5102827.html](https://www.cnblogs.com/dingjiaoyang/p/5102827.html)                                                                           |
| 【JDK和Open JDK】平常使用的JDK和Open JDK有什么区别                                                       | [https://www.cnblogs.com/sxdcgaq8080/p/7487369.html](https://www.cnblogs.com/sxdcgaq8080/p/7487369.html)                                                                             |
| 【Linux】CentOS7下安装JDK详细过程                                                                        | [https://www.cnblogs.com/sxdcgaq8080/p/7492426.html](https://www.cnblogs.com/sxdcgaq8080/p/7492426.html)                                                                             |
| Java SE 8 Archive Downloads - Oracle                                                                     | [https://www.oracle.com/java/technologies/javase/javase8u211-later-archive-downloads.html](https://www.oracle.com/java/technologies/javase/javase8u211-later-archive-downloads.html) |
| Jenkins安装之踩坑记录                                                                                    | [https://blog.csdn.net/qq_39884410/article/details/100073607](https://blog.csdn.net/qq_39884410/article/details/100073607)                                                           |
| Jenkins用户授予root权限                                                                                  | [https://www.cnblogs.com/heyongboke/p/11996774.html](https://www.cnblogs.com/heyongboke/p/11996774.html)                                                                             |
| 解决jenkins中运行docker无权限问题 以root用户运行jenkins中shell命令                                       | [https://blog.csdn.net/qq_39507276/article/details/103315466](https://blog.csdn.net/qq_39507276/article/details/103315466)                                                           |
| docker常规操作——启动、停止、重启容器实例                                                                 | [https://www.cnblogs.com/personblog/p/10762875.html](https://www.cnblogs.com/personblog/p/10762875.html)                                                                             |
| jenkins+docker实现自动编译、打包、构建镜像、容器部署                                                     | [https://blog.csdn.net/xiaoxiangzi520/article/details/88842200/](https://blog.csdn.net/xiaoxiangzi520/article/details/88842200/)                                                     |
| docker 权限问题 Got permission denied while trying to connect to the Docker daemon socket at 。。。      | [https://blog.csdn.net/u011337602/article/details/104541261](https://blog.csdn.net/u011337602/article/details/104541261)                                                             |
| 使用Jenkins+docker 部署springboot项目                                                                    | [https://blog.csdn.net/qq_38157516/article/details/90139120](https://blog.csdn.net/qq_38157516/article/details/90139120)                                                             |
| 使用Jenkins配置SpringBoot的自动化构建                                                                    | [https://blog.csdn.net/xlgen157387/article/details/78733729](https://blog.csdn.net/xlgen157387/article/details/78733729)                                                             |
| 使用Jenkins配置Git+Maven的自动化构建                                                                     | [https://blog.csdn.net/xlgen157387/article/details/50353317](https://blog.csdn.net/xlgen157387/article/details/50353317)                                                             |
| bash忽略错误继续执行_忽略Shell脚本中的特定错误                                                           | [https://blog.csdn.net/weixin_39614754/article/details/111854550](https://blog.csdn.net/weixin_39614754/article/details/111854550)                                                   |
| SpringBoot升级2.4.0所出现的问题：When allowCredentials is true, allowedOrigins cannot contain the specia | [https://blog.csdn.net/jxysgzs/article/details/110818712](https://blog.csdn.net/jxysgzs/article/details/110818712)                                                                   |
| IllegalArgumentException: When allowCredentials is true, allowedOrigins cannot contain the special v     | [https://blog.csdn.net/weixin_51579730/article/details/111142073](https://blog.csdn.net/weixin_51579730/article/details/111142073)                                                   |
| 手把手教你使用 Jenkins 配合 Github hook 持续集成                                                         | [https://blog.csdn.net/weixin_38320674/article/details/110412976](https://blog.csdn.net/weixin_38320674/article/details/110412976)                                                   |
| Jenkins内存溢出的处理方法                                                                                | [https://www.cnblogs.com/EasonJim/p/6394434.html](https://www.cnblogs.com/EasonJim/p/6394434.html)                                                                                   |
| BUILD STEP 'INVOKE TOP-LEVEL MAVEN TARGETS' MARKED BUILD AS FAILURE FINISHED解决方案                     | [https://www.freesion.com/article/8213184540/](https://www.freesion.com/article/8213184540/)                                                                                         |
| Build step 'Invoke top-level Maven targets' marked build as failure Finished解决                         | [https://www.cnblogs.com/yangyuke1994/p/10592847.html](https://www.cnblogs.com/yangyuke1994/p/10592847.html)                                                                         |

## 完整代码示例

代码都在过程中啦，项目自己搞一个吧
