---
layout: article
title: C实验报告[5]
date: 2007-02-06 11:27
category: C语言实验报告
excerpt:
  学习函数的编程思想，
  掌握函数中2种参数的传递方式
  和函数的相互调用。
---

实验内容及要求：学习函数的编程思想，编写一个程序包括3~4个函数。掌握函数中2种参数的传递方式和函数的相互调用。

1. 写一个函数int  digit(int n,int k)，它返回数n的从右边向左的第k个十进数字位值。例如，函数调用digit(1234，2)将返回值3。
2. 写一个函数int isprime(int n)，当n是质数时，函数返回非零值；当n是合数时，函数返回零值。
3. 写一个函数reverse(char s[])，将字符串s[]中的字符存储位置颠倒后重新存于s[]中。试分别用递归和非递归两种形式编写。
4. 写一个主函数输入测试数据（自己指定），并调用上述函数，检查函数功能的正确性。

要求：请同学把预备知识、步骤、程序框图、调试好的程序及存在的问题写在下面（不够可以附页）。

{% highlight c %}
#include <string.h>
#include <stdlib.h>
#include <stdio.h>

int digit(int n, int k) {
  for (k--; k > 0; k--)
    n /= 10;
  return n % 10;
}

int isprime(int n) {
  int i;
  for (i = 2; i * i <= n && n % i; i++);
  return i * i > n;
}

void reverse(char s[]) {
  int i, j = strlen(s) - 1;
  for (i = 0; i < j; i++, j--) {
    char tmp = s[i];
    s[i] = s[j];
    s[j] = tmp;
  }
}

int main(void) {
  char s[] = "redraiment";

  printf("%d\n", digit(1234, 1));
  printf("%d\n", digit(1234, 2));
  printf("%d\n", digit(1234, 3));
  printf("%d\n", digit(1234, 4));

  puts(isprime(2)? "Y": "N");
  puts(isprime(3)? "Y": "N");
  puts(isprime(4)? "Y": "N");
  puts(isprime(9)? "Y": "N");

  reverse(s);
  puts(s);

  return EXIT_SUCCESS;
}
{% endhighlight %}
