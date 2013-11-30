---
layout: article
title: 迎国庆，DuplexPipe 发布 0.3.0 版
date: 2009-10-01 22:07
category: 双向管道
excerpt: 实现了 UDP 通信模式
---

今天是中华人民共和国建国六十周年，普天同庆！作为一个程序员，当然是努力工作报效祖国啦～特地抽空完善 DuplexPipe，主要更新如下：

1. 实现了 UDP 通信模式；
1. 增加了对多语言的支持，Download 中提供中文版，你还可以通过源码自行编译英文版；
1. 修正了一些 v0.1.0 中的小错误。

最新版的源码以及 JAR 包请到项目主页（[http://code.google.com/p/duplexpipe/](http://code.google.com/p/duplexpipe/)）下载。有了 UDP 模式，现在连接模式一共有以下四种：

1. TCP 监听模式
1. TCP 连接模式
1. UDP 监听模式
1. UDP 连接模式

根据两个连接模式排列，可以得到 4×4=16 种模式。由于类似“TCP 监听模式 - TCP 连接模式”和“TCP 连接模式 - TCP 监听模式”等模式属于同一种，排除重复项后总共剩下以下十种模式：

1. TCP 监听模式 - TCP 监听模式
1. TCP 监听模式 - TCP 连接模式
1. TCP 监听模式 - UDP 监听模式
1. TCP 监听模式 - UDP 连接模式
1. TCP 连接模式 - TCP 连接模式
1. TCP 连接模式 - UDP 监听模式
1. TCP 连接模式 - UDP 连接模式
1. UDP 监听模式 - UDP 监听模式
1. UDP 监听模式 - UDP 连接模式
1. UDP 连接模式 - UDP 连接模式

到这里还没有结束。我们知道 UDP 属于非安全连接，通讯之前没有经过三次握手确认。因此在“UDP 连接模式端”发送数据包到“UDP 监听模式端”之前，“监听端”并不知道“连接端”的位置，所以也就无法主动给“连接端”发送数据。而我们的 DuplexPipe 只是一个数据转发工具，本身并不向两端程序发送任何多余的信息。因此第十种模式“UDP 连接模式 - UDP 连接模式”并不能正常工作。于是，真正能建立通讯的模式就只剩下前面九种。我暂时想不出解决方法，如果其他朋友知道如何解决（当然不能让 DuplexPipe 主动发送冗余数据），欢迎联系我（[redraiment@gmail.com](mailto:redraiment@gmail.com)），谢谢！
