---
layout: article
title: C实验报告[1]
date: 2007-02-06 11:21
category: C语言实验报告
excerpt:
  熟悉上机环境(VC++)，
  基础输入输出
  和简单表达式计算
---

实验目的及要求：熟悉上机环境(VC++)和简单表达式计算。

实验内容：表达式计算，简单程序调试

1. 编写一个输出一句话的程序，上机调试，熟悉系统的各个有关程序的编辑和编译命令的使用。
2. 完成3个数据的输入、求和，并输出计算结果的程序。

要求：请同学把预备知识、步骤、程序框图、调试好的程序及存在的问题写在下面（不够可以附页）。

# 1.c

{% highlight c %}
#include <stdlib.h>
#include <stdio.h>

int main(void) {
  printf("Hello redraiment!\n");

  return EXIT_SUCCESS;
}
{% endhighlight %}

# 2.c

{% highlight c %}
#include <stdlib.h>
#include <stdio.h>

int main(void) {
  int a, b, c;

  scanf("%d%d%d", &a, &b, &c);
  printf("%d\n", a + b + c);

  return EXIT_SUCCESS;
}
{% endhighlight %}
