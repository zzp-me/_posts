---
layout: article
title: 除余法求素数
date: 2008-01-29 18:33
category: 基础算法
excerpt:
  使用除余法求2-N之间的所有素数
---

# 问题描述

试编写一个程序，找出 2-N 之间的所有素数。

# 问题分析

这个问题可以有两种解法：一种是用“筛法”，另一种是除余法。本文使用除余法，“筛法”详情请参见《[筛法求素数]({% post_url 2008-01-29-prime-filter-in-c %})》。

题目要求找出 2-N 之间的所有素数，但并非每个数都需要检查。如2、3是素数，把数列按 2×3=6 来分组，即 6n, 6n+1, 6n+2, 6n+3, 6n+4, 6n+5。其中 6n, 6n+2, 6n+4 是 2 的倍数、6n+3 是 3 的倍数，如果直接跳过 2 与 3 的倍数，则任意连续的 6 个数中仅 2 个数（6n+1 和 6n+5）需要测试，工作量变成原来的 2/6 = 1/3。

再来看每个数的检查过程，根据素数的定义：素数只能被 1 与本身整除。即每个数都要检查 2-n-1 是否能被 n 整除。这其中也有许多冗余的计算：假设 n 不是素数，即存在 n=a×b，且 a、b 不为 1 和 n；则 a≥2、b≤n/2。意味着，检查过 2 之后，大于 n/2 的数都可以不用检查，因为它们不可能整除 n。因此，只需要检查 a≤b，即 2-sqrt(n)。

经过上面的优化，搜索的范围已经大大缩小，但中间还是有不少冗余。例如一个数不能被 2 整除，那它同样不能被 2 的倍数整除。因此在检查 2-sqrt(n) 的过程中 4、6、8……等2的倍数是可以忽略的，最理想的方法就是用已经找到的素数去除n。

# 编码技巧

根据上述优化策略编码代码，其中涉及一些编程小技巧。

<dl>
  <dt>如何做到只检查 6n+1 和 6n+5？</dt>
  <dd>不难发现，它们的距离是4、2、4、2……，可以先定义一个变量 factor=4，然后 factor=6-factor。</dd>
  <dt>sqrt(n)有精度问题</dt>
  <dd>如sqrt(121)有可能得到 10.999999。使用 i*i &lt;= n 替代 i &lt;= sqrt(n)。</dd>
</dl>

# 程序清单

```c
#include <stdlib.h>
#include <stdio.h>

int prime[30000] = { 2, 3, 5 }; /* 保存已找到的素数 */
int count = 3;                  /* 已找到素数的个数 */
int MAX = 200000;               /* 要查找的素数范围 */

void generate_prime() {
  int i, n;
  int factor = 2;
  for (n = 7; n <= MAX; n += factor) {
    factor = 6 - factor;
    for (i = 0; prime[i] * prime[i] <= n; i++) {
      if (!(n % prime[i])) {
        break;
      }
    }
    if (prime[i] * prime[i] > n) {
      prime[count] = n;
      count++;
    }
  }
}

int main(void) {
  int i;
  
  generate_prime();
  for (i = 0; i < count; i++) {
    printf("%d\n", prime[i]);
  }

  return EXIT_SUCCESS;
}
```
