---
layout: article
title: 在 Cygwin 下编译 netcat_1.10-38
date: 2011-03-18 08:06
category: 双向管道
excerpt:
  Cygwin 用的是 netcat 原生的 1.10 版本，该版本没提供 -c 选项。
  因为我最近在做的项目需要一个这样的工具来帮助测试，
  于是就决定自己编译 1.10-38 版的 nc。
---

我在《[Linux下用nc实现DuplexPipe]({% post_url 2010-01-25-duplexpipe-8-linux-nc %})》中提到，Win 版的 netcat 无法实现 DuplexPipe 的功能。

其实 Cygwin 版的 netcat 也是如此。Cygwin 用的是 netcat 原生的 1.10 版本（项目地址是：[http://nc110.sourceforge.net/](http://nc110.sourceforge.net/)），该版本没提供 -c 选项。因为我最近在做的项目需要一个这样的工具来帮助测试，于是就决定自己编译 1.10-38 版的 nc。

# 编译

首先到 debian 下 netcat 的主页下载源代码（[http://packages.qa.debian.org/n/netcat.html](http://packages.qa.debian.org/n/netcat.html)），测试版已经到 1.10-39 版本了，我选择稳定版 v1.10-38（[http://packages.debian.org/source/stable/netcat](http://packages.debian.org/source/stable/netcat)）。需要下载两个文件：其中 [netcat_1.10.orig.tar.gz](http://ftp.de.debian.org/debian/pool/main/n/netcat/netcat_1.10.orig.tar.gz) 是原生的 nc；[netcat_1.10-38.diff.gz](http://ftp.de.debian.org/debian/pool/main/n/netcat/netcat_1.10-38.diff.gz) 是升级包。

下载完成后先解压：

```bash
$ ls
netcat_1.10-38.diff.gz*  netcat_1.10.orig.tar.gz*
$ tar xvf netcat_1.10.orig.tar.gz
...
$ ls
netcat-1.10.orig/  netcat_1.10-38.diff.gz*  netcat_1.10.orig.tar.gz*
```

接着是释放补丁

```bash
$ cd netcat-1.10.orig
$ ls
Changelog  README  generic.h     netcat.c  stupidh*
Makefile   data/   netcat.blurb  scripts/
$ zcat ../netcat_1.10-38.diff.gz | patch -p1
$ ls
Changelog  README  debian/    nc.1          netcat.c  stupidh*
Makefile   data/   generic.h  netcat.blurb  scripts/
```

补丁被释放到 debian 目录下。然后就是给源代码打补丁了。

```bash
$ sed 's#^#patch -Np1 -i debian/patches/#' debian/patches/series | sh
patching file Makefile
patching file netcat.c
patching file netcat.c
patching file netcat.c
patching file netcat.c
patching file netcat.c
patching file netcat.c
patching file netcat.c
patching file netcat.c
patching file netcat.c
patching file netcat.c
patching file netcat.c
patching file debian/nc.1
patching file netcat.c
patching file debian/nc.1
patching file netcat.c
patching file debian/nc.1
patching file data/rservice.c
patching file netcat.c
patching file netcat.c
patching file netcat.c
patching file netcat.c
patching file netcat.c
patching file scripts/webproxy
$
```

最后，为了能使用 `-c` 和 `-e` 选项，还要再修改一下 Makefile 文件。将 Makefile 的第 11 行“`# DFLAGS = -DTEST -DDEBUG`”替换成“`DFLAGS = -DDEBIAN_VERSION='"1.10-38"' -DGAPING_SECURITY_HOLE -DIP_TOS -DTELNET`”，保存退出。这样，就可以运行“`make nc`”来编译了。

# 安装

编译完成后会在当前目录下生成一个“nc.exe”。将它拷贝到“/bin”目录下，并将“nc.1”复制到“/usr/share/man/man1”目录里。这样就安装完成了。你可以试试运行“nc -h”看看版本号是否正确。

```bash
$ nc -h
[v1.10-38]
connect to somewhere:   nc [-options] hostname port[s] [ports] ...
listen for inbound:     nc -l -p port [-options] [hostname] [port]
options:
        -c shell commands       as `-e'; use /bin/sh to exec [dangerous!!]
        -e filename             program to exec after connect [dangerous!!]
        -b                      allow broadcasts
        -g gateway              source-routing hop point[s], up to 8
        -G num                  source-routing pointer: 4, 8, 12, ...
        -h                      this cruft
        -i secs                 delay interval for lines sent, ports scanned
        -k                      set keepalive option on socket
        -l                      listen mode, for inbound connects
        -n                      numeric-only IP addresses, no DNS
        -o file                 hex dump of traffic
        -p port                 local port number
        -r                      randomize local and remote ports
        -q secs                 quit after EOF on stdin and delay of secs
        -s addr                 local source address
        -T tos                  set Type Of Service
        -t                      answer TELNET negotiation
        -u                      UDP mode
        -v                      verbose [use twice to be more verbose]
        -w secs                 timeout for connects and final net reads
        -z                      zero-I/O mode [used for scanning]
port numbers can be individual or ranges: lo-hi [inclusive];
hyphens in port names must be backslash escaped (e.g. 'ftp/-data').
```

# 模拟 WireShark

在项目中，我们有时会需要截获客户端和服务端之间发送的消息。WireShark 可以监听本地的通信，可是我觉得它操作比较麻烦。现在，有了 1.10-38 版的 nc，就可以很简单地实现这个功能！

```bash
nc -l -p 1234 -c 'tee 1234.txt | nc remotehost 1235 | tee 1235.txt'
```

执行了上面的命令后，再用客户端连接本地的 1234 端口，nc 就会自动去连接远程的 1235 端口。并且客户端发送的消息会保留一份副本到 1234.txt 文件中；服务器反馈的信息则会保存到 1235.txt 中。
