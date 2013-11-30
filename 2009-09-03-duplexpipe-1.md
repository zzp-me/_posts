---
layout: article
title: DuplexPipe二三事[1] - 有趣的起因：算24
date: 2009-09-03 10:56
category: 双向管道
excerpt:
  写 DuplexPipe 是因为“无聊”！真的，那天很无聊，
  想起小时候用扑克牌和姐姐比赛算24，就随手写了一个 Shell 脚本重温一下。
---

写 DuplexPipe 是因为“无聊”！真的，那天很无聊，想起小时候用扑克牌和姐姐比赛算24，就随手写了一个 Shell 脚本重温一下：

```bash
#!/bin/sh
for ((i=0;i<4;i++))
do
    ((n=$RANDOM%10+1))
    echo -n "$n "
done
echo
i=0
while read exp
do
    ((i++))
    ((value=$exp))
    if [[ $value -ne 24 ]]
    then
        echo -n "Wrong! "
        echo "$exp=$value"
        echo "try again!"
    else
        echo "OK!"
        echo "Total use ${i}s"
        break
    fi
done
```

最近好像养成一个习惯：能让计算机做则尽量不自己做。才玩了几盘，就开始动歪脑筋要写一个求24的程序来自动计算。方法就是最简单的穷举，下面是一个最初的简陋版本（有兴趣的朋友可以在此再添加五种括号）：

```bash
#!/bin/sh
s[0]='+'
s[1]='-'
s[2]='*'
s[3]='/'
read a b c d
for ((i=0;i<64;i++))
do
    ((k=i%4))
    x=${s[$k]}
    ((k=i/4%4))
    y=${s[$k]}
    ((k=i/16))
    z=${s[$k]}
    e="$a$x$b$y$c$z$d"
    r=`echo $e | bc -l | sed -e 's//.0/+$//'`
    if [[ "$r" == "24" ]]
    then
        echo "$e=$r"
    fi
done
```

既然能让计算机自动计算结果，那我先运行程序1，显示4个随机数；运行程序2，输入程序1生成的4个数字，获得结果后再反馈给程序1。但转念一想：我这样手工进行输入输出的工作，不是比算24本身更枯燥、单调？而且是重复的劳动！理所当然应该由计算机自动完成。

为完成这个任务，计算机要做的就是让“程序1”和“程序2”进行聊天：“程序1”的输出作为“程序2”的输入、“程序2”的输出作为“程序1”的输入。这一定义会让人立刻联想到“管道”（`exe1 | exe2`），但遗憾的是“管道”是单向的——只能满足“程序1”的输出作为“程序2”的输入。

其实这就是一个“进程通信”（IPC）的问题，如果你熟悉 UNIX 的历史，也会和我一样联想到 Socket 。我随即想起了网络瑞士军刀——nc（netcat），并用它顺利解决了这个问题！我会在《[DuplexPipe二三事（二）]({% post_url 2009-09-03-duplexpipe-2 %})》中介绍这个方法。
