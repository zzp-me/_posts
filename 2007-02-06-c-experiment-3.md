---
layout: article
title: C实验报告[3]
date: 2007-02-06 11:25
category: C语言实验报告
excerpt:
  使用循环语句完成累乘、
  图形输出的程序编写。
  掌握较复杂结构程序的编写。
---

实验内容及要求：使用循环语句完成累乘、图形输出的程序编写。掌握较复杂结构程序的编写。

1. 已知xyz+yzz=532，其中x、y、z都是数字（0~9），编写一个程序求出x、y、z分别代表什么数字。
2. 编写一个程序打印如下图形：（行数由键盘输入（1~9）范围的值）例下面是输入4的情况。

        4444444
         33333
          222
           1
          222
         33333
        4444444

3. 学校有近千名学生，在操场上排队，5人一行余2人，7人一行余3人，3人一行余1人，编写一个程序求该校的学生人数。

要求：请同学把预备知识、步骤、程序框图、调试好的程序及存在的问题写在下面（不够可以附页）。

# 1.c

{% highlight c %}
#include <stdlib.h>
#include <stdio.h>

int main(void) {
  int x, y, z;

  for (z = 1; z <= 6; z += 5) {
    y = (13 - z) % 10 - 2 * z / 10;
    x = 4 + (13 - z) / 10 - y;
    if (0 <= x && x <= 9
     && 0 <= y && y <= 9
     && x != y && x != z
     && y != z) {
      printf("x=%d y=%d z=%d\n", x, y, z);
    }
  }

  return EXIT_SUCCESS;
}
{% endhighlight %}

# 2.c

{% highlight c %}
#include <stdlib.h>
#include <stdio.h>
#include <math.h>

int main(void) {
  int x, y;

  for (y = -3; y <= 3; y++) {
    printf("%*d", 4 - abs(y), 1 + abs(y));
    for (x = 0; x < 2 * abs(y); x++) {
      printf("%d", 1 + abs(y));
    }
    putchar('\n');
  }

  return EXIT_SUCCESS;
}
{% endhighlight %}

# 3.c

{% highlight c %}
#include <stdlib.h>
#include <stdio.h>
#include <math.h>

int main(void) {
  int n;

  for (n = 1;
       n % 5 != 2
    || n % 7 != 3
    || n % 3 != 1;
       n++);
  printf("%d\n", n);

  return EXIT_SUCCESS;
}
{% endhighlight %}
