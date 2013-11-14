---
layout: article
title: C实验报告[8]
date: 2007-02-06 11:30
category: C语言实验报告
excerpt:
  编写程序完成单链表的建立、查询，
  掌握结构体的定义、数据输入方法。
---

实验内容及要求：编写程序完成单链表的建立、查询，掌握结构体的定义、数据输入方法。

1. 编写一个建立一个单链表的函数。设链表的表元素信息包含学号、姓名、一门课的成绩；写一个按照学号查一个学生成绩的函数；最后写一个主函数先调用建立函数，再调用查询函数，显示查到学生的姓名和成绩。
2. 编一程序，能把从终端输入的一个字符串中的小写字母全部转换成大写字母，要求输入字符的同时指定该字符在字符串中的序号（即字符在字符串中的顺序号，例如第1个字符的序号为1），字符和序号存入结构体中，字符串存入结构体数组中，然后显示结构体数组的结果。（用字符！表示输入字符串的结束）。

要求：请同学把预备知识、步骤、程序框图、调试好的程序及存在的问题写在下面（不够可以附页）。

# 1.c

{% highlight c %}
#include <stdlib.h>
#include <stdio.h>

typedef struct student_node {
  int id;
  char name[128];
  double score;
  struct student_node* next;
} *Student;

Student student_new() {
  Student s;

  s = (Student)calloc(1, sizeof(struct student_node));
  if (s != NULL) {
    scanf("%d%s%le", &s->id, s->name, &s->score);
  }
  s->next = NULL;

  return s;
}

Student student_list() {
  Student s = NULL, head = NULL;
  int length;

  scanf("%d", &length);
  while (length--) {
    if (s) {
      s = s->next = student_new();
    } else {
      s = head = student_new();
    }
  }

  return head;
}

Student student_search(Student s, int id) {
  Student p;

  for (p = s; p; p = p->next) {
    if (p->id == id) {
      return p;
    }
  }

  return NULL;
}

void student_display(Student s) {
  if (s != NULL) {
    printf("%d %s %g\n", s->id, s->name, s->score);
  }
}

void student_free(Student s) {
  while (s) {
    Student p = s;
    s = s->next;
    free(p);
  }
}

int main(void) {
  Student s = NULL;
  int id;

  s = student_list();
  scanf("%d", &id);
  student_display(student_search(s, id));
  student_free(s);

  return EXIT_SUCCESS;
}
{% endhighlight %}

# 2.c

{% highlight c %}
#include <stdlib.h>
#include <stdio.h>
#include <ctype.h>

#define MAX 128

typedef struct {
  struct {
    char value;
    int index;
  } data[MAX];
  int length;
} CharSequence;

int main(void) {
  CharSequence seq;
  int i;
  char c;

  for (i = 0; (c = getchar()) != '!'; i++) {
    if (islower(c)) {
      c = toupper(c);
    }
    seq.data[i].value = c;
    seq.data[i].index = i + 1;
  }
  seq.length = i;

  for (i = 0; i < seq.length; i++) {
    printf("%d: %c\n", seq.data[i].index, seq.data[i].value);
  }

  return EXIT_SUCCESS;
}
{% endhighlight %}
