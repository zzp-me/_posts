---
layout: article
title: C实验报告[9]
date: 2007-02-06 11:31
category: C语言实验报告
excerpt:
  编写程序完成一个简单职工信息系统设计。
---

实验内容及要求：编写程序完成一个简单职工信息系统设计。

1. 编写一个建立一个输入职工信息（职工号，姓名，年龄，工资）的函数，并把职工信息存入文件“work.txt”中；写一个按照职工号查一个职工工资的函数；再写一个修改函数，实现工资小于800元的职工加90元，并把修改工资的职工信息建成一个文件“workmodify.txt”；最后写一个主函数先调用建立函数，再调用查询函数对查到的职工信息进行显示，最后调用修改函数修改职工工资。
2. 要求职工信息存储在文件或结构体数组（自己选择），编写程序实现上述功能。

要求：请同学把预备知识、步骤、程序框图、调试好的程序及存在的问题写在下面（不够可以附页）。

{% highlight c %}
#include <stdlib.h>
#include <stdio.h>

/** Data Type */
typedef struct employee_node {
  int id;
  char name[128];
  int age;
  double wage;
  struct employee_node* next;
} *Employee;

typedef void (*Callback)(Employee);
typedef int (*Predicate)(Employee);

/** API */
Employee employee_make_node() {
  Employee e;

  e = (Employee)calloc(1, sizeof(struct employee_node));
  if (e != NULL) {
    scanf("%d%s%d%le", &e->id, e->name, &e->age, &e->wage);
  }
  e->next = NULL;

  return e;
}

Employee employee_make_list() {
  Employee e = NULL, head = NULL;
  int length;

  scanf("%d", &length);
  while (length--) {
    Employee p = employee_make_node();
    if (e) {
      e = e->next = p;
    } else {
      e = head = p;
    }
  }

  return head;
}

Employee employee_map(Employee e, Callback fn) {
  Employee p, head = e;

  while (p = e) {
    e = e->next;
    fn(p);
  }

  return head;
}

Employee employee_some(Employee e, Predicate fn) {
  for (; e; e = e->next) {
    if (fn(e)) {
      return e;
    }
  }
  return NULL;
}

/** Interface */
/* IO */
FILE* FOUT;

void employee_output_node(Employee e) {
  fprintf(FOUT, "%d %s %d %.2f\n", e->id, e->name, e->age, e->wage);
}

void employee_display_node(Employee e) {
  FOUT = stdout;
  if (e != NULL) {
    employee_output_node(e);
  }
}

void employee_display(Employee e) {
  FOUT = stdout;
  employee_map(e, employee_output_node);
}

void employee_save(Employee e, char* filename) {
  FOUT = fopen(filename, "w");
  employee_map(e, employee_output_node);
  fclose(FOUT);
}

/* Search */
int search_key_id;

int employee_cmp_id(Employee e) {
  return e != NULL && e->id == search_key_id;
}

Employee employee_search_by_id(Employee e) {
  scanf("%d", &search_key_id);
  return employee_some(e, employee_cmp_id);
}

/* Modify */
void employee_adjust_wage_node(Employee e) {
  if (e != NULL && e->wage < 800) {
    e->wage += 90;
  }
}

void employee_adjust_wage(Employee e) {
  employee_map(e, employee_adjust_wage_node);
}

/* Free */
void employee_free_node(Employee e) {
  free(e);
}

Employee employee_free(Employee e) {
  return employee_map(e, employee_free_node);
}

int main(void) {
  Employee e = NULL;

  e = employee_make_list();
  employee_save(e, "work.txt");

  employee_display_node(employee_search_by_id(e));

  employee_adjust_wage(e);
  employee_save(e, "workmodify.txt");

  employee_display(e);

  employee_free(e);

  return EXIT_SUCCESS;
}
{% endhighlight %}
