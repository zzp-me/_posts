---
layout: article
title: 杨辉三角
date: 2011-09-22 13:00
category: 基础算法
excerpt:
  如果你了解函数式编程，可能知道map和reduce两个运算符：
  map可以保持数据维度不变，
  reduce则把一个n维的数据降到1维。
  杨辉三角属于第三类范畴：把维度为1扩展成n维。
---

杨辉三角形，又称贾宪三角形、帕斯卡三角形，是二项式系数在三角形中的一种几何排列。其中第n行第i列的值为：C<sup>i</sup><sub>n</sub>= n!/(i! * (n - i)!)。如下图所示。

为了输出一个规模为n的杨辉三角，如果根据定义来做，那每一个元素至少要计算两个阶乘，效率是很低的。从下图可知，第i行可以通过第i-1行的元素两两相加得到，这样乘法就简化成一组加法，效率会提高很多！

{% img 1.gif %}

如果你了解函数式编程，可能知道map和reduce两个运算符：map可以保持数据维度不变，reduce则把一个n维的数据降到1维。杨辉三角属于第三类范畴：把维度为1的数据扩展成n维。

本文提供多种不同的编程语言的解决方法，来比较各语言解决此类问题的优劣。

# 批处理版

```bat
::Show a Pascal/Yanhui Triangle
@echo off
setlocal enabledelayedexpansion
set /a line[0]=1
for /l %%i in (0,1,10) do (
  for /l %%j in (%%i,-1,1) do (
    if defined line[%%j] (
      set n=%%j
      call :calc
    ) else (
      set /a line[%%j]=1
    )
  )
  for /l %%j in (0,1,%%i) do set /p=!line[%%j]! <nul
  echo.
)
goto end

:calc
set /a prev=%n%-1
set /a line[%n%]+=!line[%prev%]!
goto :eof

:end
pause
```

# Bash 版

```bash
#!/usr/bin/bash

a[0]=1
for ((i=0;i<=10;i++)); do
  for ((j=$i;j>0;j--)); do
    ((a[$j]+=a[$j-1]))
  done
  for ((j=0;j<=$i;j++)); do
    printf "%d " ${a[$j]}
  done
  echo
done
```

C语言版

```c
#include <stdlib.h>
#include <stdio.h>

#define SIZE 11

int main(void) {
  int line[SIZE] = {1};
  int row;
  int i;

  for (row = 0; row < SIZE; row++) {
    for (i = row; i > 0; i--) {
      line[i] += line[i - 1];
    }
    for (i = 0; i <= row; i++) {
      printf("%d%c", line[i], i == row? '\n': ' ');
    }
  }

  return EXIT_SUCCESS;
}
```

Common Lisp 版

```common-lisp
(do ((a '(1) (mapcar #'+ `(0 ,@a) `(,@a 0))))
    ((= (length a) 10))
  (format t "~{~A~^ ~}~%" a))
```

Clojure版

```clojure
(defn pascal []
  (iterate #(map + `(0 ~@%) `(~@% 0)) [1]))

(take 5 (pascal)) ; 取前面5行
; ->((1) (1 1) (1 2 1) (1 3 3 1) (1 4 6 4 1))
```
