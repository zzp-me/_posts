---
layout: article
title: C实验报告[2]
date: 2007-02-06 11:23
category: C语言实验报告
excerpt:
  了解基本的表达式语句
  和分支语句的使用，
  写5个表达式，
  要求运算符各不相同。
---

实验内容及要求：了解基本的表达式语句和分支语句的使用，写5个表达式，要求运算符各不相同。

上机实验要求：

1. 写一个输入7个数据的程序，把输入的数据代入a+b*(c-d)/e*f-g表达式进行运算，输出计算的结果。
1. 编写一个程序完成输入一个整数，输出它的符号。
1. 有一函数如下：

            x       (x<1)
        y = 2x-1    (1<=x<10)
            3x-11   (x>=10)

    试编程根据输入的x的值计算出y的值。

要求：请同学把预备知识、步骤、程序框图、调试好的程序及存在的问题写在下面（不够可以附页）。

# 1.c

{% highlight c %}
#include <stdlib.h>
#include <stdio.h>

int main(void) {
  double a, b, c, d, e, f, g;

  scanf("%le%le%le%le%le%le%le", &a, &b, &c, &d, &e, &f, &g);
  printf("%g\n", a + b * (c - d) / e * f - g);

  return EXIT_SUCCESS;
}
{% endhighlight %}

# 2.c

{% highlight c %}
#include <stdlib.h>
#include <stdio.h>

int main(void) {
  int n;

  scanf("%d", &n);

  if (n < 0) {
    n = -1;
  } else if (n > 0) {
    n = 1;
  }
  printf("%d\n", n);

  return EXIT_SUCCESS;
}
{% endhighlight %}

#3.c

{% highlight c %}
#include <stdlib.h>
#include <stdio.h>

int main(void) {
  int x, y;

  scanf("%d", &x);

  if (x < 1) {
    y = x;
  } else if (x < 10) {
    y = 2 * x - 1;
  } else {
    y = 3 * x - 11;
  }
  printf("%d\n", y);

  return EXIT_SUCCESS;
}
{% endhighlight %}
