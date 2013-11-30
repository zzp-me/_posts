---
layout: article
title: DuplexPipe二三事[4] - 网络连接方式随心换
date: 2009-09-04 13:18
category: 双向管道
excerpt:
  有没有办法让“监听-监听”、“连接-连接”这两种方式也能通信呢？
  或者说，有没有办法在不修改程序的前提下转换它的连接方式？
  这就是 DuplexPipe 提供的功能！
---

# 连接方式

在《[DuplexPipe二三事（一）]({% post_url 2009-09-03-duplexpipe-1 %})》中提到建立连接有两种方式：监听本地端口，等待其他程序来连接（以下简称“监听方式”）；或者主动连接其他程序（以下简称“连接方式”）。排列组合一下，会得到三种结果：监听-连接、监听-监听、连接-连接。其中只有“监听-连接”方式能正确地建立连接，《[DuplexPipe二三事（三）]({% post_url 2009-09-04-duplexpipe-3 %})》中介绍的 FPipe 只是在其中添加了一节“监听-连接-监听-连接”，其中粗体部分就是 FPipe 的工作。

那有没有办法让“监听-监听”、“连接-连接”这两种方式也能通信呢？或者说，有没有办法在不修改程序的前提下转换它的连接方式？这就是 DuplexPipe 提供的功能！

# DuplexPipe简介

DuplexPipe 和 FPipe 类似，也是命令行下的转发工具，但它允许你自己选择连接方式，因此比 FPipe 更加灵活！DuplexPipe 目前还只有 Java 实现方式，开放源代码，目前还不支持 UDP 模式。欢迎有兴趣的同学一起参与开发和维护！项目主页：[http://code.google.com/p/duplexpipe](http://code.google.com/p/duplexpipe)。下面来介绍一下它的选项和用法：

<dl>
  <dt>用法: java -jar DuplexPipe [-vh] model model</dt>
  <dt>选项:</dt>
  <dd>-v              输出一些提示信息</dd>
  <dd>-h              显示本帮助文档</dd>
  <dt>模式:</dt>
  <dd>-l port         监听本地端口</dd>
  <dd>-c host port    连接远程端口</dd>
  <dt>示例:</dt>
  <dd>java pipe.DuplexPipe -c 192.168.1.100 3389 -l 1234</dd>
  <dd>将本地 1234 号端口上的信息转发给 192.168.1.100 的 3389 端口，这样的用法类似 FPipe。</dd>
</dl>

由上述可知，执行“java -jar DuplexPipe -c host1 port1 -c host2 port2”就能让“监听-监听”方式的两个程序通信；执行“java -jar DuplexPipe -l port1 -l port2”可以让“连接-连接”方式的两个程序交流。

# 总结

从功能上来说，DuplexPipe 和 FPipe 有部分重合，不过这并不是 DuplexPipe 的亮点。思索一下你会发现它能做很多事情！你不想知道我是如何用它将“Windows 远程桌面”改造成反弹式连接的吗？你不想看看怎么让这些小工具配合来创造一个后门？请看《[DuplexPipe二三事（五）]({% post_url 2009-09-04-duplexpipe-5 %})》。
