---
title: 在Debian安装软件
date: 2023-01-28 14:22:14
tags: Linux
---

# 安装工具

`dpkg -i ...` 用于 .deb 结尾的文件。.deb 结尾的文件通常是一个软件的安装包，可以借此直接安装软件。

`dpkg -l` 展示本地已安装的所有软件。与 `| grep ...` 结合可以查看本地已安装哪些包含 ... 的软件。

`apt-cache policy ...` 可以查看软件在当前 apt 源中的可用版本Version table 属性）。如果所查询的软件已经安装在系统上，则 Installed 属性会指示安装在系统上的属性。

```bash
root@debian11-cicd:~# apt-cache policy nginx
nginx:
  已安装：(无)
  候选： 1.18.0-6.1+deb11u3
  版本列表：
     1.18.0-6.1+deb11u3 500
        500 http://mirrors.huaweicloud.com/debian bullseye/main amd64 Packages
        500 http://security.debian.org/debian-security bullseye-security/main amd64 Packages
```

`apt-get install ...` 时的默认版本是 `apt-cache policy` 查询的结果的 Candidate 属性）以及当前是否安装（Installed 属性）。

`apt-cache madison ...` 可以查看软件在当前 apt 源中的所有版本以及如果使用这个版本的话将使用什么源。

```bash
root@VM-133-145-debian:~/download# apt-cache madison erlang
    erlang | 1:19.2.1+dfsg-2+deb9u3 | http://mirrors.tencentyun.com/debian stretch/main amd64 Packages
    erlang | 1:19.2.1+dfsg-2+deb9u3 | http://mirrors.tencentyun.com/debian stretch/main i386 Packages
    erlang | 1:19.2.1+dfsg-2+deb9u1 | http://mirrors.tencentyun.com/debian-security stretch/updates/main amd64 Packages
    erlang | 1:19.2.1+dfsg-2+deb9u1 | http://mirrors.tencentyun.com/debian-security stretch/updates/main i386 Packages
    erlang | 1:19.2.1+dfsg-2+deb9u3 | http://mirrors.tencentyun.com/debian stretch/main Sources
    erlang | 1:19.2.1+dfsg-2+deb9u1 | http://mirrors.tencentyun.com/debian-security stretch/updates/main Sources
N: Ignoring file 'rabbitmq_rabbitmq-server' in directory '/etc/apt/sources.list.d/' as it has no filename extension
N: Ignoring file 'bintray.erlang' in directory '/etc/apt/sources.list.d/' as it has an invalid filename extension
```

`apt-get install ...=version` 可以在软件名后增补 `=version` 来强制指定安装软件的版本。这里的 version 只能是 apt-cache policy 或 apt-cache madison 中查出来的可用版本号。

关于软件卸载：

`apt-get purge` 或 `apt-get --purge remove` 都是删除指定软件且不保留配置文件。

`apt-get autoremove` 移除系统当前不再需要的软件。会出现这种软件，一般是在安装A软件时，需要B软件作为基础库，随后A软件被移除了。

`apt-get remove` 删除已安装的软件包但保留配置文件。

在删除过程中可能会出现居然需要用户同意安装某个软件的操作。为什么会这样？因为现在被删除的这个软件是其它某个软件的依赖，提示用户允许的要新安装的软件则是替换被删除软件来充当依赖的。

实际上，这种情况的出现意味着用户执行删除软件的操作是不慎重的。如果要使用的软件B还依赖着软件A，就不应该移除软件A。或者应该先完成对软件B的不使用和移除。

# Docker

docker 官方提供了如何在 debian 上安装 docker 的方式。[传送门](https://docs.docker.com/engine/install/debian/#install-using-the-repository)。

另外由于众所周知的原因，直接访问境外的 docker 下载源通常会很慢。清华大学镜像站为此提供了镜像服务以及相关的[安装文档](https://mirrors.tuna.tsinghua.edu.cn/help/docker-ce/)。

安装好 docker 之后需要使用 docker hub 下载和安装 docker image，对于国内服务器又需要配置 docker hub 的镜像。这里建议买的云服务器是谁，配谁的镜像。我用的是腾讯云，参考腾讯云的[快速入门文档](https://cloud.tencent.com/document/product/1141/63910)。

# Nginx

## 从 apt-get 安装 nginx

`sudo apt install nginx`

安装完成后，nginx 服务自动被注册在 systemctl 中。

## 编译安装

### 安装

从 [Nginx 官网](http://nginx.org/) 的下载页面中选择 Stable version 的不带 windows 的那个下载到云主机中进行手动编译安装。

2022年12月13日，nginx 已经发布了稳定的 1.22 版本，而在 debian 11 上使用 ·apt-get cache policy nginx· 得到的 nginx 版本还在 1.18.x，所以我选择自己装。

下载源码后需要编译安装，故需要阅读官方文档中 [Building nginx from Sources](http://nginx.org/en/docs/configure.html)。其安装方法简单来说就是解压缩下载到的源码压缩包后进入其解压后目录（该目录下应当有一个 configure 文件），随后以该目录为工作目录完成后续步骤：

执行 `./configure` 命令

如果只是简单学习使用 Nginx，所以啥配置项都不指定也够用。但如果要带 https 支持的话要带上参数 `--with-http_ssl_module` 即 `./configure --with-http_ssl_module`。

但第一次执行并不能安装成功，遇到缺失 pcre 支持无法支持 http 模块功能的错误：

```
./configure: error: the HTTP rewrite module requires the PCRE library.
You can either disable the module by using --without-http_rewrite_module
option, or install the PCRE library into the system, or build the PCRE library
statically from the source with nginx by using --with-pcre=<path> option.
```

以 pcre debian 为关键词谷歌，得到 apt-get install libpcre3 libpcre3-dev 来安装相关库，解决这一问题。

但在安装了这些之后仍然不成功，遇到 zlib 库的问题：

```
./configure: error: the HTTP gzip module requires the zlib library.
You can either disable the module by using --without-http_gzip_module
option, or install the zlib library into the system, or build the zlib library
statically from the source with nginx by using --with-zlib=<path> option.
```

以 zlib debian 为关键词谷歌，得到 sudo apt install zlib1g-dev 来安装 zlib 库，解决这一问题。

第三次执行 ./configure 终于成功，最后输出：

```
Configuration summary
  + using system PCRE library
  + OpenSSL library is not used
  + using system zlib library

  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  nginx modules path: "/usr/local/nginx/modules"
  nginx configuration prefix: "/usr/local/nginx/conf"
  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
  nginx pid file: "/usr/local/nginx/logs/nginx.pid"
  nginx error log file: "/usr/local/nginx/logs/error.log"
  nginx http access log file: "/usr/local/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"
```

之后再输入 make 指令即可完成安装。

### 交付 systemctl

参考[将编译安装的软件托管 systemctl](https://xaiversun.gitee.io/blog/2021/05/11/0xC/#将编译安装的软件托管-systemctl)一节。

## 配置文件

`nginx -t` 可以检测配置文件是否存在问题。

没有问题时会有如下输出：

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

输出中也透露了配置文件的具体路径：`/etc/nginx/nginx.conf`

## 网站启用 https

nginx 提供了 [Configuring HTTPS servers](http://nginx.org/en/docs/http/configuring_https_servers.html) 文档可以参考。

同时随着 ACME2 技术的普及，SSL 证书的供应商可能会给出基于 ACME 技术部署 nginx https 证书的脚本。比如[ACME v2证书自动化快速入门](https://blog.freessl.cn/acme-quick-start/)一文。

# MYSQL 5.7

## 安装

依据MySQL 官方的[安装文档](https://dev.mysql.com/doc/refman/5.7/en/linux-installation-debian.html)：

首先是下载一个 .deb 文件到云服务器中，随后对那个文件执行 `dpkg -i path_of_the_deb_file`，目的是将包含 MYSQL 5.7 的安装包的 APT 仓库加入到云服务器的 APT 仓库列表中。

但是，这里使用 .deb 文件添加 MySQL 5.7 的官方安装源，会让 apt 从 MySQL 官方下载 MySQL，即可能出现之前安装 Docker 时下载速度比较慢的问题。所以还是需要使用[清华大学镜像站](https://mirrors.tuna.tsinghua.edu.cn/help/mysql/)。


在安装时遇到了 gnupg is not installed 的问题：

```
root@VM-4-3-debian:~/downloads# dpkg -i mysql-apt-config_0.8.16-1_all.deb
dpkg: regarding mysql-apt-config_0.8.16-1_all.deb containing mysql-apt-config, pre-dependency problem:
 mysql-apt-config pre-depends on gnupg
  gnupg is not installed.

dpkg: error processing archive mysql-apt-config_0.8.16-1_all.deb (--install):
 pre-dependency problem - not installing mysql-apt-config
Errors were encountered while processing:
 mysql-apt-config_0.8.16-1_all.deb
```

解决方法就是把它装了：`apt install gnupg`。

```
apt-get install mysql-server
```


如果使用清华镜像站的源，本应使用 `deb https://mirrors.tuna.tsinghua.edu.cn/mysql/apt/debian stretch mysql-5.6 mysql-5.7 mysql-8.0 mysql-tools` 填入 `/etc/apt/sources.list.d/mysql-community.list` 文件，但我只想安装 MySQL 5.7，所以删减为 `deb https://mirrors.tuna.tsinghua.edu.cn/mysql/apt/debian stretch mysql-5.7 mysql-tools`。

帮助文档中的 `/etc/apt/sources.list.d/mysql-community.list` 要是不存在，就新建一个写。此后还需要 `apt-get update` 让 apt 更新使这一修改生效，然后再执行 “Installing MySQL with APT” 章节的内容。

如果 MySQL 5.7 的安装是配置云服务器的第一步，那么很有可能无法使用 https 协议的清华镜像源。这件事会暴露在编辑 mysql-community.list 文件后的 apt-get update 后，错误信息如下：

```
E: The method driver /usr/lib/apt/methods/https could not be found.
N: Is the package apt-transport-https installed?
E: Failed to fetch https://mirrors.cloud.tencent.com/mysql/apt/debian/dists/stretch/InRelease  
E: Some index files failed to download. They have been ignored, or old ones used instead.
```

错误提示中的第二行其实已经给出了解决方案。使用 `apt install apt-transport-https` 将缺失的包安装。再度执行 `apt update`，不再报错。

现在，使用 `apt-cache policy mysql-server` 确定要安装的 mysql 版本为 5.7：

```bash
root@VM-4-3-debian:~/downloads# apt-cache policy mysql-server
mysql-server:
  Installed: (none)
  Candidate: 5.7.32-1debian10
  Version table:
     5.7.32-1debian10 500
        500 https://mirrors.cloud.tencent.com/mysql/apt/debian buster/mysql-5.7 amd64 Packages
        500 http://repo.mysql.com/apt/debian buster/mysql-5.7 amd64 Packages
```

确认过后可以输入执行 `apt-get install mysql-server` 开始安装。

## 允许被远程接入

MySQL 5.7 默认不允许远程主机接入。搜索 mysql 5.7 allow remote access，找到 StackOverflow 上的一个相关问题 [How to allow remote connection to mysql](https://stackoverflow.com/questions/14779104/how-to-allow-remote-connection-to-mysql)。

> What is disabled by default is remote root access. If you want to enable that, run this SQL command locally:
> 
> ```
> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;
>  FLUSH PRIVILEGES;
> ```
> 
> And then find the following line and comment it out in your my.cnf file, which usually lives on /etc/mysql/my.cnf on Unix/OSX systems. In some cases the location for the file is /etc/mysql/mysql.conf.d/mysqld.cnf).
> 
> If it’s a Windows system, you can find it in the MySQL installation directory, usually something like C:\Program Files\MySQL\MySQL Server 5.5\ and the filename will be my.ini.
> 
> Change line
> ```
> bind-address = 127.0.0.1
> ```
> 
> to
> 
> ```
> #bind-address = 127.0.0.1
> ```
> 
> And restart the MySQL server (Unix/OSX, and Windows) for the changes to take effect.
>

回答前半部分照做即可，只有代码中的 ‘password’ 里的 password 要换成真实的 mysql 的 root 用户的密码。

但后半段，并不存在 `/etc/mysql/my.cnf` 这一文件，即便我自己新建然后添加 `#bind-address = 127.0.0.1` 的内容也仍然无用。但思路是明确的，我们要找到 mysql 的有效配置文件。

翻看官方文档 [5.1.2 Server Configuration Defaults](https://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html)

> For non-Windows platforms, no default option file is created during either the server installation or the data directory initialization process. Create your option file by following the instructions given in [Section 4.2.2.2](https://dev.mysql.com/doc/refman/5.7/en/option-files.html), “Using Option Files”. Without an option file, the server just starts with its default settings—see Section 5.1.2, “Server Configuration Defaults” on how to check those settings.

[Section 4.2.2.2](https://dev.mysql.com/doc/refman/5.7/en/option-files.html)

> Most MySQL programs can read startup options from option files (sometimes called configuration files). Option files provide a convenient way to specify commonly used options so that they need not be entered on the command line each time you run a program.
> 
> To determine whether a program reads option files, invoke it with the --help option. (For mysqld, use --verbose and --help.) If the program reads option files, **the help message indicates which files it looks for and which option groups it recognizes.**

所以在主机上输入使用 `mysqld --help` 命令。得到：

```
mysqld  Ver 8.0.21 for Linux on x86_64 (MySQL Community Server - GPL)
Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Starts the MySQL database server.

Usage: mysqld [OPTIONS]

For more help options (several pages), use mysqld --verbose --help.
```

由于

> For more help options (several pages), use mysqld --verbose --help.

执行 `mysqld --verbose --help > ~/mysql-config.txt`，阅读时发现其中说

> Default options are read from the following files in the given order:
> /etc/my.cnf /etc/mysql/my.cnf ~/.my.cnf

其中 `·~/.my.cnf` 和 `/etc/my.cnf` 均不存在，而 `/etc/mysql/my.cnf` 存在且包含如下两行：
```
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mysql.conf.d/
```

在 /etc/mysql/conf.d/ 下有一个 mysql.cnf 文件，不过其中啥也没有。

而 /etc/mysql/mysql.conf.d/ 下有一个 mysqld.cnf 文件，其中有东西：

```bash
#/etc/mysql/mysql.conf.d/mysqld.cnf
[mysqld]
pid-file    = /var/run/mysqld/mysqld.pid
socket        = /var/run/mysqld/mysqld.sock
datadir        = /var/lib/mysql
log-error    = /var/log/mysql/error.log
# By default we only accept connections from localhost
bind-address    = 127.0.0.1
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
```

有 stackoverflow 回答里说的要注释的东西了，把它注释掉。

然后 systemctl restart mysql 。这之后就可以连接了。

# Redis

[Redis官方安装文档](https://redis.io/docs/getting-started/installation/install-redis-on-linux/)

# Java 与 Maven

## 安装

在 [Java I tell you](https://www.injdk.cn/) 中选择合适的 jdk 版本并下载到本地进行安装。此处我选择 Temurin 版本 （Adoptopenjdk自2020年7月交给Eclipse基金会，后改名Temurin）以及 jdk 17，[linux64.tar.gz](https://d6.injdk.cn/openjdk/adoptopenjdk/17/OpenJDK17U-jdk_x64_linux_hotspot_17.0.1_12.tar.gz) 文件。

```bash
mkdir ~/download
cd ~/download
wget https://d6.injdk.cn/openjdk/adoptopenjdk/17/
OpenJDK17U-jdk_x64_linux_hotspot_17.0.1_12.tar.gz
mkdir /usr/local/temurin/
tar -xzf OpenJDK17U-jdk_x64_linux_hotspot_17.0.1_12.tar.gz -C /usr/local/temurin/
cd /usr/local/temurin/
root@debian11-cicd:/usr/local/temurin# ls
jdk-17.0.1+12
```

然后编辑 `/etc/profile`，在最后新增如下几行
```
export JAVA_HOME=/usr/local/temurin/jdk-17.0.1+12
PATH=$PATH:$JAVA_HOME/bin
```
执行 `source /etc/profile`

通过 `java --version` 验证，应该得到如下结果：

```
root@debian11-cicd:/usr/local/temurin# java --version
openjdk 17.0.1 2021-10-19
OpenJDK Runtime Environment Temurin-17.0.1+12 (build 17.0.1+12)
OpenJDK 64-Bit Server VM Temurin-17.0.1+12 (build 17.0.1+12, mixed mode, sharing)
```

maven 的安装与之类似。从[官网](https://maven.apache.org/download.cgi)获取最新包的路径，解压缩，配置环境变量。

选 Binary tar.gz archive 是 maven 程序编译后的产品。Source tar.gz archive 是 maven 的源代码还没有编译过，怎么用。

```bash
cd ~/download
wget https://dlcdn.apache.org/maven/maven-3/3.8.7/binaries/apache-maven-3.8.7-bin.tar.gz
tar -xzf apache-maven-3.8.7-bin.tar.gz -C /usr/local/
```

同样进行环境变量的配置
```
export MAVEN_HOME=/usr/local/apache-maven-3.8.7
PATH=$PATH:$MAVEN_HOME/bin
```

## 配置 maven 镜像

在 maven 的配置文件 `$MAVEN_HOME/conf/settings.xml` 中加入[阿里云的 maven 镜像站](https://developer.aliyun.com/mirror/maven/)的配置：

```xml
<mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>阿里云公共仓库</name>
    <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```

# NodeJs 和 npm

## NodeJs

[Node.js 官网](https://nodejs.org/en/download/)

[Node.js Linux 安装指南](https://github.com/nodejs/help/wiki/Installation)

但我们不从官网下载，而是从[清华的镜像站](https://nodejs.org/dist/latest/)下载。

```bash
cd ~/download
wget https://nodejs.org/dist/latest/node-v19.5.0-linux-x64.tar.xz
```

随后按照文档操作即可

```
mkdir -p /usr/local/lib/nodejs
tar -xJvf node-v19.5.0-linux-x64.tar.xz -C /usr/local/lib/nodejs
```

文档说在 `~/.profile` 里设置环境变量，保守了。直接在 `/etc/profile` 最末继续插入就好。

```
export NODE_JS_HOME=/usr/local/lib/nodejs/node-v19.5.0-linux-x64
export PATH=$NODE_JS_HOME/bin:$PATH
```

## npm 配置镜像

[阿里云 npm 镜像站](https://developer.aliyun.com/mirror/NPM?spm=a2c6h.13651102.0.0.30da1b11l49Cj4)

```
npm config set registry https://registry.npmmirror.com/
npm config get registry
```



