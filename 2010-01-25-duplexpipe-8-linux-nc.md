---
layout: article
title: Linux下用nc实现DuplexPipe
date: 2010-01-25 23:03
category: 双向管道
excerpt:
  Linux下使用nc实现DuplexPipe的功能
---

nc 是一把网络的瑞士军刀，我以前在介绍 DuplexPipe 时也提到过，如果你没接触过它，可以先参看一下《[DuplexPipe二三事（二）]({% post_url 2009-09-03-duplexpipe-2 %})》。再来简单地介绍一下 DuplexPipe，顾名思义，它是一个“双向管道”。在 shell 中，我们通过“|”使用匿名管道，让前一条命令的输出作为后一条命令的输入；双向管道即在此基础上在加上“后一条命令的输入作为前一条命令的输入”。这是最初开发它的原因，但后来发现它更像是一个网络接口转换器，“DuplexPipe”这个名字反而不能体现它的功能。

# 留言

今天有网友给我留言，他通过用 nc 的 -e 选项来执行 nc 本身来实现 DuplexPipe。留言原文如下：

> 哥们，你写的那个DuplexPipe, 我很欣赏。
> 不过近日于网上逛发现此工具的功能竟然完全可以用netcat做到，
> 有两种方法，我的博客上载了一种。简单描述如下：

    在windows下：
    echo nc [ip] [port] > relay.bat
    nc -l -p [port2] -e relay.bat
    其余的类推
    第二种方法是用命名管道：(linux下)
    mknod backpipe p
    nc -l -p [port] 0<backpipe | nc [ip] [port12] | tee backpipe

其中选项 -e 的作用是：

```
for NT:    -e prog        inbound program to exec [dangerous!!]
for Linux: -e filename    program to exec after connect [dangerous!!]
```

# Windows下不行

在我开发 DuplexPipe 时确实考虑过功能会不会和 nc 重叠，当时只想着通过 shell 管道来连接，忘了 nc 自带了一个双向管道！我首先在 Vista 下做了测试，nc(win32) 是从 [http://www.securityfocus.com/tools/139](http://www.securityfocus.com/tools/139) 下载。开启三个命令提示符，分别执行：

    1) nc -l -p 1234
    2) nc localhost 1234 -e "nc -l -p 1235"
    3) nc localhost 1235

其中第二条和留言中使用批处理等价。理论上在提示符(3)中输入一行数据，提示符(1)三中马上显示。但我每次在提示符(3)中输入一堆数据后，提示符(1)要输入两个回车才会把数据显示出来。我又另外开启四个命令提示符，模拟 DuplexPipe：

    1) nc -l -p 1234
    2) nc -l -p 1235 -e "nc -l -p 1236"
    3) nc localhost 1234 -e "nc localhost 1235"
    4) nc localhost 1236

此时提示符(1)中的数据能发送到(4)中，而提示符(4)中的数据却倒不了(1)。调整(2)、(3)中端口的顺序会出现不同结果，但都达不到理想效果。后来又下了其他几个不同版本的 nc，并在 WinXP 下也进行了测试，但都不成功。

# Linux下成功

Linux下有些不同，nc 不能脱离 shell 执行。可以像留言中一样先创建一个脚本文件，也可以使用另一个选项：

    -c shell commands      as `-e'; use /bin/sh to exec [dangerous!!]

我的系统是 Debian Lenny+nc v1.10-38，经测试能实现 DuplexPipe 的全部功能！最新版的 DuplexPipe 能实现[九种连接模式]({% post_url 2009-10-01-duplexpipe-7 %})，对应的指令分别是：

1. nc -l -p port1 -c 'nc -l -p port2'
1. nc -l -p port1 -c 'nc host2 port2'
1. nc -l -p port1 -c 'nc -u -l -p port2'
1. nc -l -p port1 -c 'nc -u host2 port2'
1. nc host1 port1 -c 'nc host2 port2'
1. nc host1 port1 -c 'nc -u -l -p port2'
1. nc host1 port1 -c 'nc -u host2 port2'
1. nc -u -l -p port1 -c 'nc -u -l -p port2'
1. nc -u -l -p port1 -c 'nc -u host2 port2'

在留言中还提到用命名管道来实现。

    mknod pipe p
    nc -l -p 1234 0< pipe | nc -l -p 1235 > pipe

shell 的匿名管道让前一条命令的标准输出作为后一条命令的标准输入，手工创建的命名管道则是让后一条命令的标准输出作为前一条命令的标准输入。你会发现这已经解决了文章开头的问题，而且不需要通过网络。

# 下一步任务

我想，DuplexPipe 已经没必要继续维护了。虽然 Windows 下测试不成功，但所幸它的源码开放，也许接下来我们应该考虑转向去完善 netcat for NT。
