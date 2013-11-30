---
layout: article
title: 猜数字游戏及自动解猜数字程序
date: 2012-08-21 20:02
category: 双向管道
excerpt:
  实在无聊，于是写了一个猜数字游戏：
  随机生成一个[0, 99]之间的整数，
  如果猜得小了就显示 Too small，
  大了显示 Too big，
  否则显示 You are right。
---

都是寂寞惹得祸...网络故障已经四天了，强烈谴责华数网通这种低效率的行为（好吧，谴责有个屁用）。

实在无聊，于是写了一个猜数字游戏：随机生成一个`[0, 99]`之间的整数，如果猜得小了就显示 Too small，大了显示 Too big，否则显示 You are right。作为添头，前面会以英文序数词输出 The first time, The second time...

```common-lisp
(setf *random-state* (make-random-state t))
(do ((target (random 100))
     (guess 0)
     (times 1 (1+ times)))
    ((= target guess))
  (format t "Give a number between 0 to 99: ")
  (format t "The ~:r time: ~[You are right!~;Too big.~:;Too small.~]~%"
          times (signum (- (setq guess (read)) target))))
```

让我很欣喜的是核心代码只有 do 循环这么一句话。故事当然还没完，我玩来几盘后，想着要是有一个自动化的程序能自己玩这个游戏就好了。转念一想，最近刚接触的 Tcl 有一个扩展叫 Expect，就是能自动化这种交互式的程序！继续捣鼓一阵：

```bash
#!/usr/bin/expect -f

spawn clisp guess.lisp

set leftBound 0
set rightBound 100
while 1 {
    expect {
        right {break}
        small {set leftBound $guess}
        big {set rightBound $guess}
        {Give a number between 0 to 99: } {
            set guess [expr ($leftBound + $rightBound) / 2]
            send "$guess\n"
        }
        eof {exit 0}
    }
}
```

算法就是二分。看看执行效果：

```
expect$ ./main.tcl
spawn clisp guess.lisp
Give a number between 0 to 99: 50
The first time: Too small.
Give a number between 0 to 99: 75
The second time: Too small.
Give a number between 0 to 99: 87
The third time: Too small.
Give a number between 0 to 99: 93
The fourth time: Too big.
Give a number between 0 to 99: 90
The fifth time: Too big.
Give a number between 0 to 99: 88
The sixth time: Too small.
Give a number between 0 to 99: 89
The seventh time: You are right!
expect$
```
