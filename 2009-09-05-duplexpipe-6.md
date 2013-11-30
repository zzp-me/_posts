---
layout: article
title: DuplexPipe二三事[6] - 没有第七
date: 2009-09-05 12:42
category: 双向管道
excerpt: DuplexPipe的设计文档
---

# 我的设想

在着手编写 DuplexPipe 之前，我规划过我的需求：我想要一个最通用的通信工具，换言之就是能让所有具有输入/输出的程序都可以相互通信。DuplexPipe 本身远没达到这个设想，至少还得具备以下几中模式：

    -f file       # 通过读写文件获得数据
    -s            # 从 stdio 中获得数据
    -e exefile    # 从本地程序的输入输出中获得数据
    -r url        # 这是一个附加功能。如果你玩过几天木马，
                  # 你可能也渴望将它变成一个强大的后门！
                  # 通过这个选项可以从URL中获得IP地址和端口，主动进行连接。

# 没有第七

但我会很遗憾的宣布，上述功能不会被加入 DuplexPipe 中。这也意味着介绍 DuplexPipe 到此为止，不会再有《DuplexPipe二三事（七）》，关于 DuplexPipe 维护的动态则会更新在项目主页[http://code.google.com/p/duplexpipe/](http://code.google.com/p/duplexpipe/)中。

你可能听说过：“一篇文章的完成不是再也不能往里面加内容，而是再也没法删内容时。”写程序也是如此。我们来看上面提到的四个功能：

    -f file       # 这条可以最先被排除，因为主流的系统都支持输入输出重定向（'<'、'>'和'>>'）。
    -s            # -s 和 -e 两种模式 nc 都支持！写程序也很机会重复制造车轮。
    -e exefile    # 理由同上
    -r url        # 这个功能虽然诱人，但很明显和主要功能无关，没必要保留。

因此，前三个功能都可以通过和 nc 配合来完成（瑞士军刀 nc 的使用方法请参看《[DuplexPipe二三事（二）]({% post_url 2009-09-03-duplexpipe-2 %})》）！而且现在 Http Client 遍地开花（例如 wget 和 curl），要完成第四个功能也很简单。比如我用来搞定我们校园内部计算机的 Shell 脚本：

```bash
#!/bin/sh

# 外网计算机的IP文件，格式是ip port
IP_FILE='http://www.xxx.com/ip.txt'

while true
do
    wget -O ip.txt $IP_FILE
    read ip port < ip.txt
    java -jar DuplexPipe-0.1_0.jar -v -c $ip $port -c 10.21.*.* 3389
    # 如果失败，每分钟尝试连接一次
    sleep 60
done
```

# 曲终

善用身边的小工具，这是我推崇的“[物尽其（奇）用]({% post_url 2009-07-02-the-best-use %})”！也体现了 UNIX 的哲学：只提供机制，不提供策略。通过善用他人的成果，可以降低我们编码的复杂度，节省下更多的时间做更有意义的事情！
