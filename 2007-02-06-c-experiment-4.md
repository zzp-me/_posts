---
layout: article
title: C实验报告[4]
date: 2007-02-06 11:26
category: C语言实验报告
excerpt:
  使用数组构造类型编写
  矩阵转置运算和排序程序。
  掌握一维和二维数组的使用技巧。
---

实验内容及要求：使用数组构造类型编写一个矩阵转置运算的程序和一个排序程序。掌握一维和二维数组的使用技巧。

1. 编写一个程序判定用户输入的一串字符是否为“回文”，所谓回文就是这串字符正读反读都一样。例、“aba”，“bhuuhb”是回文。
2. 设数组a的定义如下：int  a[20]={2,4,6,8,10,12,14,16}; 已存入数组中的数据值已经按由小到大存放，现从键盘输入一个数据，把它插入到数组中，要求插入新数据以后，数组中值的序仍然保持不变，即升序。请编写一个程序实现上述功能。
3. 写一个3*5矩阵的转置程序，输出其原矩阵的值和转置以后的结果。

要求：请同学把预备知识、步骤、程序框图、调试好的程序及存在的问题写在下面（不够可以附页）。

# 1.c

{% highlight c %}
#include <string.h>
#include <stdlib.h>
#include <stdio.h>

int main(void) {
  char s[64];
  int i, j;

  scanf("%s", s);

  j = strlen(s) - 1;
  for (i = 0; i < j && s[i] == s[j]; i++, j--);

  puts(i >= j? "Yes": "No");

  return EXIT_SUCCESS;
}
{% endhighlight %}

# 2.c

{% highlight c %}
#include <stdlib.h>
#include <stdio.h>

int main(void) {
  int a[20] = {2, 4, 6, 8, 10, 12, 14, 16};
  int len = 8;
  int n;
  int i;

  scanf("%d", &n);
  for (i = len; i && n < a[i-1]; i--) {
    a[i] = a[i-1];
  }
  a[i] = n;

  for (i = 0; i <= len; i++) {
    printf("%d\n", a[i]);
  }

  return EXIT_SUCCESS;
}
{% endhighlight %}

# 3.c

{% highlight c %}
#include <stdlib.h>
#include <stdio.h>

#define SIZE 5

typedef int Matrix[SIZE][SIZE];

Matrix m;

void input(int r, int c) {
  int x, y;

  for (y = 0; y < r; y++) {
    for (x = 0; x < c; x++) {
      scanf("%d", &m[y][x]);
    }
  }
}

void display(int r, int c) {
  int x, y;

  for (y = 0; y < r; y++) {
    for (x = 0; x < c; x++) {
      printf("%d ", m[y][x]);
    }
    putchar('\n');
  }
}

void turn() {
  int x, y;

  for (y = 0; y < SIZE; y++) {
    for (x = 0; x < y; x++) {
      int tmp = m[y][x];
      m[y][x] = m[x][y];
      m[x][y] = tmp;
    }
  }  
}

int main(void) {
  input(3, 5);
  display(3, 5);
  turn();
  display(5, 3);

  return EXIT_SUCCESS;
}
{% endhighlight %}

