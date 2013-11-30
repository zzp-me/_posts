---
layout: article
title: DuplexPipe-0.1_0发布
date: 2009-09-02 01:25
category: 双向管道
excerpt:
  DuplexPipe 是我开发的一个开源网络小工具——双向管道，
  目的是让网络上的两个程序能进行自动化交流。
  当初写这个小工具的原因是为了能在外网远程控制内网计算机。
---

项目主页：[http://code.google.com/p/duplexpipe/](http://code.google.com/p/duplexpipe/)

DuplexPipe 是我开发的一个开源网络小工具——双向管道，目的是让网络上的两个程序能进行自动化交流。当初写这个小工具的原因是为了能在外网远程控制内网计算机。

传统的管道只能从一端输入、一端输出。双向管道不仅可以让进程 A 的输出作为进程 B 的输入，也会让进程 B 的输出作为进程 A 的输入。这样就可以让两个进程实现交流。

本程序主要是 TCP 转发工具。允许监听本地端口，也可以主动连接远程端口。如果和瑞士军刀“nc -e”配合使用， 就能实现本地进程和网络进程任意沟通。程序的具体使用方法可以参见项目主页。

接下来我会写几篇关于这个小工具原理及应用的文章。本程序开放源代码，目前使用 Java 程序实现，诚邀有兴趣的朋友一起维护，我希望能再用其他语言开发（尤其是 C 语言）。

1. [DuplexPipe二三事（一）——有趣的起因：算24]({% post_url 2009-09-03-duplexpipe-1 %})
1. [DuplexPipe二三事（二）——瑞士军刀再显锋芒：让程序相互聊天]({% post_url 2009-09-03-duplexpipe-2 %})
1. [DuplexPipe二三事（三）——网络中转站：端口映射]({% post_url 2009-09-04-duplexpipe-3 %})
1. [DuplexPipe二三事（四）——网络连接方式随心换]({% post_url 2009-09-04-duplexpipe-4 %})
1. [DuplexPipe二三事（五）——来自内网的呼唤]({% post_url 2009-09-04-duplexpipe-5 %})
1. [DuplexPipe二三事（六）——没有第七]({% post_url 2009-09-05-duplexpipe-6 %})
