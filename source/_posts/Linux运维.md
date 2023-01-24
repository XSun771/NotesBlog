---
title: Linux运维
date: 2023-01-24 17:19:56
tags: Linux
---

# 指定全局别名(alias)/环境变量

## 为 Java 11 配置 java 命令

可以从[JDK国内镜像站](https://www.injdk.cn/)下载对应的JDK编译后压缩包，解压缩到合适位置。一般是/usr文件夹下。我放在了/user/java/jdk-11下。

之后开始配置环境变量。

首先 `vim /etc/profile` 对 `/etc/profile` 进行编辑，这个文件是 Linux 系统下环境变量的大管家。

在文件最下方加入：

```bash
COPYexport JAVA_HOME=/usr/java/jdk-11
export PATH=$JAVA_HOME/bin:$PATH
```

完成后输入 `:wq` 退出vim的编辑视图。

随后 source /etc/profile 使得修改立刻生效。

再输入 `java -version` 应当有如下输出：

```bash
COPYopenjdk version "11" 2018-09-25
OpenJDK Runtime Environment 18.9 (build 11+28)
OpenJDK 64-Bit Server VM 18.9 (build 11+28, mixed mode)
```

说明安装已经完成。

## 同时安装 Java 8

给 Java 8 的 JDK 使用 java8 的别名去调用，即运行它等于运行 jJava 8 所在目录（下载 Java 8 的压缩包解压后出来的文件夹）的 bin 目录下的 java。

对应操作：

在`/etc/profile`文件中写入一行 `alias java11='/usr/openjdk8/bin/java'`，然后在终端中输入并回车`source /etc/profile`


# 设置软连接

`ln -s 要被链接上的文件（夹）的路径 用来链接到那个文件（夹）的文件`

在 nginx 配置自己的 webapp 时有这样一个操作：

COPYln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
这里 /etc/nginx/sites-available/example.com 是被链接的对象

/etc/nginx/sites-enabled/ 是链接？并不。因为它是一个目录，而目标却是一个文件。那么此时会在目录下自动生成一个名为 example.com 的文件用于对应目标文件以进行连接。但如果这里就是一个第一个参数就是一个文件，那么这个文件就会成为链接，即便文件名不同。

# Shell 脚本

## $用法

| 变量 | 含义                                                         |
| ---- | ------------------------------------------------------------ |
| $0   | 当前脚本的文件名                                             |
| $n   | 传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是$1，第二个参数是$2。 |
| $#   | 传递给脚本或函数的参数个数。                                 |
| $*   | 传递给脚本或函数的所有参数。被双引号(" ")包含时，返回用空格分隔开各参数的字符串。 |
| $@   | 传递给脚本或函数的所有参数。被双引号(" ")包含时，返回一个参数的数组。 |
| $?   | 上个命令的退出状态，或函数的返回值。                         |
| $$   | 当前Shell进程ID。在 Shell 脚本中的话就对应运行这脚本所在的进程ID。 |

# tar 解压文件

指令是 tar

对于后缀名中有 .tar 的，需要带参数 x 表示解压缩，与之对应的是参数 c 表示将制定目录压缩成一个压缩文件。

对于后缀名带有 .gz 的，要带参数 z 表示需要 gzip 参与。解压缩与压缩都是这个参数。

带 v 参数则会输出解压缩过程。

带 f 参数后才可以跟待解压文件的文件名，如果仅使用上述参数就跟文件名（tar -xvz nginx-1.18.0.tar.gz） 会得到这样的错误提示： 

```bash
COPYtar: Refusing to read archive contents from terminal (missing -f option?)
tar: Error is not recoverable: exiting now
```

在参数 C 后跟压缩文件被解压到的文件夹。注意：文件夹自身必须已经存在。

综上所述，解压一个 /nginx-1.18.0.tar.gz 可以使用如下代码

`tar -xvzf ./nginx-1.18.0.tar.gz -C /usr/nginx/1.18.0/`

# Is the system based on AMD64

在下载软件的时候，经常会看到文件名后面跟着架构 -amd64, -arm64 之类的。一般 PC 系统都是基于 AMD64 的 CPU 架构，可以通过如下方式确认：

`uname -a` 或者 `cat /proc/version`

得到如下结果，其中 -amd64 表明是 AMD64 架构。

`Linux VM-133-145-debian 4.9.0-12-amd64 #1 SMP Debian 4.9.210-1 (2020-01-20) x86_64 GNU/Linux`

# 当前系统是什么版本

遗忘了自己当初选择安装了什么版本的系统，只记得是 debian？可以用如下命令检查：

`lsb_release -a`

```bash
No LSB modules are available.
Distributor ID:    Debian
Description:    Debian GNU/Linux 9.0 (stretch)
Release:    9.0
Codename:    stretch
```

[How to Check Debian Linux Version](https://www.tecmint.com/check-debian-version/)

> It’s quite often that we keep forgetting which version of the Debian operating system we are using and this mostly happens when you log in to the Debian server after a long time or are you looking for a software that is available for a specific version of Debian only.
 
# systemctl 系统服务管理器

通过 apt-get 等包管理安装的软件一般会向系统注册自己的服务，之后可通过 systemctl 来管理。

从源码编译安装（比如Redis就可以这么装）的软件也可能配置了向 systemctl 注册自己服务的步骤。

## 确认 systemctl 本身已经安装

`systemctl --version`

如果 systemctl 本身已经安装，应该得到其版本：

```bash
root@VM-133-145-debian:~# clear
root@VM-133-145-debian:~# systemctl --version
systemd 232
+PAM +AUDIT +SELINUX +IMA +APPARMOR +SMACK +SYSVINIT +UTMP +LIBCRYPTSETUP +GCRYPT +GNUTLS +ACL +XZ +LZ4 +SECCOMP +BLKID +ELFUTILS +KMOD +IDN
```

## 使用 systemctl 列出所有服务

包括此时并不在运行的。

`systemctl`

可以结合 `grep` 使用

比如，找到 nginx 服务：

```bash
root@VM-133-145-debian:~# systemctl | grep nginx
nginx.service    loaded active running   A high performance web server and a reverse proxy server        enabled
```

比如，只看正在运行中的

```bash
root@VM-133-145-debian:~# systemctl | grep running
acpid.path                                                        loaded active running   ACPI Events Check                                                 
init.scope                                                        loaded active running   System and Service Manager                                        
session-147197.scope                                              loaded active running   Session 147197 of user root                                       
acpid.service                                                     loaded active running   ACPI event daemon                                                 
cron.service                                                      loaded active running   Regular background program processing daemon                      
#...
```

## 检查某个服务的运行状态

`systemctl status ${service-name}`

检查名为 service-name 的服务的运行状态。

一个正常运行的程序在查询时得到的结果应该和如下类似：

```bash
root@VM-4-3-debian:~/downloads# systemctl status mysql
● mysql.service - MySQL Community Server
   Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2020-11-06 13:39:01 HKT; 51min ago
  Process: 15291 ExecStartPre=/usr/share/mysql/mysql-systemd-start pre (code=exited, status=0/SUCCESS)
  Process: 15326 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid (code=exited, status=0/SUCCESS)
 Main PID: 15328 (mysqld)
    Tasks: 28 (limit: 2202)
   Memory: 181.6M
   CGroup: /system.slice/mysql.service
           └─15328 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid

Nov 06 13:39:01 VM-4-3-debian systemd[1]: Starting MySQL Community Server...
Nov 06 13:39:01 VM-4-3-debian systemd[1]: Started MySQL Community Server.
```

一个不存在的程序：

```bash
root@VM-4-3-debian:~/downloads# systemctl status redis
Unit redis.service could not be found.
```

## 控制一个服务启停

```
systemctl start httpd
systemctl restart httpd
systemctl stop httpd
systemctl reload httpd
systemctl kill httpd
```

启动，重启，停止，重载和杀死 httpd 服务。这里 httpd 可以更换为其它服务。注意到之前列出所有服务时，被列出的服务都跟着一个 .service，这个其实是可以省略的。

`systemctl enable xxx.service` 可以使某个服务开机自动启动

`systemctl disable xxx.service` 则是禁止某个服务开机自动启动

## 将一个软件注册为 systemctl 的服务

鉴于 apt-get 上的很多软件的版本都相比最新稳定版要旧，不可避免地要遇到自行手动编译安装一个软件的情况。将这样的软件托管到 systemctl 将便于调整其开机启动与否的配置以及控制服务启动，重启，停止。

以手动编译安装的 Nginx 为例。编译安装的过程与本节无关，结果上最后 Nginx 被安装在了 `/usr/local/nginx/` 中。若不使用 systemctl ，则启动 Nginx 的指令为 `/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf`，重启 Nginx 的命令为 `/usr/local/nginx/sbin/nginx -s reload`，停止 Nginx 的命令为 `/usr/local/nginx/sbin/nginx -s stop`。

而我们想要可以通过

```
systemctl start nginx
systemctl restart nginx
systemctl stop nginx
```

来控制 nginx 服务。

实现方法：

` systemctl edit --force nginx.service`

调用 systemctl 为 nginx 服务编辑配置文件，由于 nginx 之前并未注册，所以不存在编辑。但因为 `--force` 参数，所以 systemctl 就会为其创建一个配置文件以编辑。这个文件在 `/etc/systemd/system/nginx.service` 目录下。

编辑 `/etc/systemd/system/nginx.service/nginx.service` 文件，添加如下内容：

```
[Unit]
Description=nginx - high performance web server
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
ExecStart=/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s stop

[Install]
WantedBy=multi-user.target
```

其中各字段的意义是：

> [Unit]：systemctl 认为每个被管理的资源都是一个 Unit。
> Description：描述该服务。
>
> After：依赖，当依赖的服务启动之后再启动自定义的服务。
> 
> [Service]：服务运行参数的设置
> 
> Type：程序运行的形式。forking指程序在后台运行。
> 
> ExecStart：systemctl start nginx.service 时调用这个字段的值。这里不能使用别名，依赖PATH路径和相对路径之类的偷懒写法。
> 
> ExecReload：对应 systemctl reload
> 
> ExecStop：对应 systemctl stop
> 
> 注意：启动、重启、停止等命令的值中必须使用绝对路径。
> 
> [Install] 服务安装的相关设置
> 
> wantedBy：挂载在 /etc/systemd/system/ 目录下的哪个文件夹里。这个字段的值还有其他指导服务挂载和安装的语义的作用，但我没有搜索到靠谱资料，一般都是 multi-user.target。

使用 `systemctl daemon-reload nginx.service` 让 systemctl 重载服务的配置文件。之后`systemctl start nginx`等就可用了。

# /usr 与 /usr/local 与 /opt

linux 中 `/opt` 目录用来安装附加软件包，是用户级的程序目录，可以理解为 `D:/Software`。被安装到 /opt 目录下的程序，应当满足所有的数据、库文件等等都是放在同个目录下面。类比于 windows 系统，有点像我们下载到了一个软件的绿色解压即用版，然后我们把压缩包解压缩出来得到一个文件夹，里面包含这个软件的所有文件。这个文件夹就该放到`D:/Software`。

opt 有可选的意思，这里可以用于放置第三方大型软件。当你不需要时，直接 rm -rf 掉即可。在硬盘容量不够时，也可将 /opt 单独挂载到其他磁盘上使用。

linux 中 `/usr` 为系统级的目录，可以理解为 C:/Windows/。

linux 中 `/usr/local` 目录为用户级的程序目录，可以理解为 C:/Progrem Files/。用户自己编译安装的软件一般默认会安装到这个目录下。apt-get 安装的软件一般也是。

# 确认某个进程使用的用户

在编译安装了 nginx 后，在访问网页资源时遇到了 403 错误，猜测原因是 Nginx 并无权限访问文档根目录。此时确认自己的猜测原因就需要了解 nginx 的主进程是以什么用户在运行的。

因此输入 `ps aux | grep nginx` 得到：

```
nobody 31731 0.0 0.1 25376 2304 ? S 16:39 0:00 nginx: worker process
```

第一列的结果 nobody 表明 Nginx 的主进程以 nobody 用户运行而非 root，因此猜想基本是正确的。

既然都说到这里了解决方案也就说了吧。在 Nginx 的配置文件（一般是 `/usr/local/nginx/conf/nginx.conf` ）中最高层级添加 user root 使得 Nginx 以 root 身份运行。

# 修改某个文件/文件夹的权限控制

chmod 命令，其最常见的用法的参数搭配是这样的：

`chmod [ugoa][+-=][rwxX]`

其中三个参数依次意义为

u 表示该文件的拥有者，g 表示与该文件的拥有者属于同一个群体(group)者，o 表示其他以外的人，a 表示这三者皆是。
+ 表示增加权限、- 表示取消权限、= 表示唯一设定权限。
r 表示可读取，w 表示可写入，x 表示可执行。

# 管理 Java 程序

如果在一个终端上输入 `java -jar a-project.jar` 或 `mvn spring-boot:run`，则这个终端将被这个指令的后续输出给占用。同时如果关闭这个终端，其它终端也管不到这个 java 程序。这两件事都是不运维的。

## nohup

`nohup command [>file.out] [2>&1] [&]`

command 将标准输出输出到 file.out 文件，从而避免了 command 命令的标准输出占用当前终端的问题。这里的 file.out 当然是可以随便改名的，my-project-name.log什么的都行。但是文件所在的文件夹必须都已经存在，nohup只会创建文件，不创建文件夹。

如果省略 `>file.out`，则默认重定向到 ./nohup.out。

如果最终的结尾不带 &，则 command 的标准输出被重定向了，但nohup命令的标准输出没重定向，还是会占用当前终端。而带上 & 则不会有此问题。

`[2>&1]` 中 2 代表的是标准错误输出（stderr），&1 代表的是标准输出（stdout），这整个参数的意思就是将标准错误输出输出的目标重定向到标准输出（的输出目标）。而由于前面已经将标准输出重定向到 file.out，故最终 stderr 也重定向到 file.out。

因此我们可以写出这样一个 bash 脚本来管理 Java 程序的启动：

```bash
COPY#!/bin/bash

cd ~/projects/example/example-backend/

nohup mvn spring-boot:run >~/projects/example/logs/example.out 2>&1 &
```

## netstat 与 grep 与 awk 与 xargs

在用 bash 脚本管理了 Java 程序的启动后，剩下的问题是如何结束这个 Java 程序。

此处先给出解决方案再来分析:

`netstat -lp | grep 7500 | awk '{print $7}' | awk -F/ '{print $1}' | xargs kill -15;`

然后分析这个命令是怎么回事儿。

首先是管道符 |，它能够将前一个命令的标准输出作为后一个命令的标准输入传过去。

然后是 netstat，它可以显示网络状态。我这里的思路是建立在我的 Java 程序是一个 Web 后台应用，并且我配置了这个 java 程序监听 7500 端口。因此相比 `ps -ef | grep java` 我觉得去筛选出监听 7500 端口的那行程序更精确一点。

而 `netstat -lp -lp` 中，p 参数较为重要，它的意义是显示正在使用Socket的程序识别码和程序名称，如果没有这个参数，即便得到了那一行，那一行里也没有那个 Java 程序的 PID，也就不可能发给 `kill -15` 去关闭。另一个 l 参数则是将展示的 Socket 过滤一遍，只展示其中我方主机作为 server 的套接字。这个其实加不加对于脚本命令影响不大，因为最后都能被 grep 7500 筛出那一行 Java 程序。不过加了之后会让输出的内容在人阅读时更友好。

再然后是 awk 命令。

awk 默认将空格（或者说连续的空格）和 Tab 作为分隔符，而 '{print $7}' 的意思就是利用分隔符分隔后，把从左往右数的第7个内容给输出到标准输出上。

`netstat -lp | grep 7500` 产生的输出是这样的：

```
tcp6 0 0 [::]:7500 [::]:* LISTEN 21494/java
```

以空格（连续的空格）为分隔符进行分隔的话，则分别有tcp6，0，0，[::]:7500，[::]:*，LISTEN和21494/java。其中第七个的前半部分21494就是Java程序的PID了。

所以接下来要对 21494/java 执行 `awk -F/ '{print $1}` ,这里-F手动指定分隔符，-F/的意思就是手动指定/为分隔符，然后输出第一个，也就得到了 PID。

最后是 xargs 命令。为什么这里不能直接 | kill -15 呢？因为kill -15 期待的是一个作为参数的 PID 而不从标准输入里取 PID。而 xargs 的作用正是将自己的标准输入的内容转换为后面的指令的参数。如此搭配，也就最终等价于kill -15 21494了。