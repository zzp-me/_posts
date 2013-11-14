---
layout: article
title: C实验报告[6]
date: 2007-02-06 11:28
category: C语言实验报告
excerpt:
  用指针做函数参数完成字符串的传递。
  掌握函数中2种参数的传递方式。
---

上机内容及要求：用指针做函数参数完成字符串的传递。掌握函数中2种参数的传递方式。

1. 编写一个函数char *delk(char *sp),把sp所指向的字符串中所有的“$”字符删除，并把处理后的字符串指针返回。
1. 写函数find(char *s1, char *s2)，函数find查找串s1中是否包含词s2。约定串中的词由1个或1个以上空格符分隔。

请同学把调试好的程序写在下面和背面。

{% highlight c %}
#include <string.h>
#include <stdlib.h>
#include <stdio.h>
#include <ctype.h>

char* delk(char* sp) {
  char* p = sp;
  int i;
  for (i = 0; sp[i]; i++) {
    if (sp[i] != '$') {
      *p = sp[i];
      p++;
    }
  }
  *p = 0;
  return sp;
}

int find(char* s1, char* s2) {
  int len1 = strlen(s1);
  int len2 = strlen(s2);
  int i;

  for (i = 0; i <= len1 - len2; i++) {
    if (!strncmp(s1 + i, s2, len2)
     && (!s1[i+len2] || isspace(s1[i+len2]))) {
      return 1;
    }
  }
  return 0;
}

int main(void) {
  char s[] = "$$$$$h$e$l$$$$lo, red$raiment$$$$$";

  puts(s);
  puts(delk(s));
  puts(find(s, "hello")? "Yes": "No");

  return EXIT_SUCCESS;
}
{% endhighlight %}
