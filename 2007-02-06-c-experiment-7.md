---
layout: article
title: C实验报告[7]
date: 2007-02-06 11:29
category: C语言实验报告
excerpt:
  用指针做函数参数完成字符串的传递，
  对10个字符串用指针完成其字典排序。
---

实验内容及要求：用指针做函数参数完成字符串的传递，对10个字符串用指针完成其字典排序。

1. 编写一个书名排序程序，输入10个书名存入一个二维数组，用函数void sortstring(char s[][100])实现他们的字典顺序,在主函数输出结果。
2. 编写函数search()

        void search(char *s1, char *s2, char *s3)

    函数search()从已知两个字符串s1与s2中，找出都包含的最长的单词送字符串s3，约定字符串中只有小写字母和空格字符，单词用1个或1个以上空格分隔。写一个主函数测试函数的正确性。

要求：请同学把预备知识、步骤、程序框图、调试好的程序及存在的问题写在下面（不够可以附页）。

{% highlight c %}
#include <string.h>
#include <stdlib.h>
#include <stdio.h>
#include <ctype.h>

void sortstring(char s[][100]) {
  qsort(s, 10, 100, strcmp);
}

void search(char* s1, char* s2, char* s3) {
  int i, from = 0, len = 0, max = 0;
  char word[100], *p;
  for (i = 0; s1[i]; i++) {
    if (islower(s1[i])) {
      if (!islower(s1[i+1])) {
        len = i - from + 1;
        if (max < len) {
          strncpy(word, s1 + from, len);
          word[len] = 0;
          p = strstr(s2, word);
          if (p && !islower(p[len])) {
            max = len;
            strcpy(s3, word);
          }
        }
      }
    } else {
      from = i + 1;
    }
  }
}

int main(void) {
  char s[10][100];
  int i;

  for (i = 0; i < 2; i++) {
    scanf("%s", s[i]);
  }
  sortstring(s);
  for (i = 0; i < 10; i++) {
    puts(s[i]);
  }

  gets(s[0]);
  gets(s[1]);
  search(s[0], s[1], s[2]);
  puts(s[2]);

  return EXIT_SUCCESS;
}
{% endhighlight %}
