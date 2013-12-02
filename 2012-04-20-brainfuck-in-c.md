---
layout: article
title: C语言实现BrainFuck解释器
category: BrainFuck
date: 2012-04-20 22:25
excerpt:
  用ANSI C实现一个轻量级的BrainFuck解释器
---

码农的业余休闲活动就是去学习一门冷门的语言或者研究一项非主流的技术。BrainFuck 是一门小巧的编程语言，顾名思义，阅读这门语言的代码就像在强奸你的大脑一样。事实证明开发它的解释器比读懂它的 Hello World 要快。

BrainFuck只有八条指令：

| 指令 | 含义 | 等价的C代码 |
| ---- | ---- | ---------- |
| > | 指针加一 | ++ptr; |
| < | 指针减一 | --ptr; |
| + | 指针指向的字节的值加一 | ++*ptr; |
| - | 指针指向的字节的值减一 | --*ptr; |
| . | 输出指针指向的单元内容（ASCII码） | putchar(*ptr); |
| , | 输入内容到指针指向的单元（ASCII码） | *ptr = getchar(); |
| [ | 如果指针指向的单元值为零，向后跳转到对应的]指令的次一指令处 | while (*ptr) { |
| ] | 如果指针指向的单元值不为零，向前跳转到对应的[指令的次一指令处 | } |

解释器代码如下：

```c
#include <stdlib.h>
#include <string.h>
#include <stdio.h>

#define TOKENS "><+-.,[]"
#define CODE_SEGMENT_SIZE 30000
#define STACK_SEGMENT_SIZE 1000
#define DATA_SEGMENT_SIZE 30000

typedef void (*Callback)(void);

struct {
  char cs[CODE_SEGMENT_SIZE];   /* Code Segment */
  long ip;                      /* Instruction Pointer */
  char ss[STACK_SEGMENT_SIZE];  /* Stack Segment */
  long sp;                      /* Stack Pointer */
  char ds[DATA_SEGMENT_SIZE];   /* Data Segment */
  long bp;                      /* Base Pointer */
  Callback fn[128];
} vm;

void vm_forward() {
  vm.bp = (vm.bp + 1) % DATA_SEGMENT_SIZE;
}

void vm_backward() {
  vm.bp = (vm.bp + DATA_SEGMENT_SIZE - 1) % DATA_SEGMENT_SIZE;
}

void vm_increment() {
  vm.ds[vm.bp]++;
}

void vm_decrement() {
  vm.ds[vm.bp]--;
}

void vm_input() {
  vm.ds[vm.bp] = getchar();
}

void vm_output() {
  putchar(vm.ds[vm.bp]);
}

void vm_while_entry() {
  if (vm.ds[vm.bp]) {
    vm.ss[vm.sp] = vm.ip - 1;
    vm.sp++;
  } else {
    int c = 1;
    for (vm.ip++; vm.cs[vm.ip] && c; vm.ip++) {
      if (vm.cs[vm.ip] == '[') {
        c++;
      } else if (vm.cs[vm.ip] == ']') {
        c--;
      }
    }
  }
}

void vm_while_exit() {
  if (vm.ds[vm.bp]) {
    vm.sp--;
    vm.ip = vm.ss[vm.sp];
  }
}

void setup() {
  int c;
  int i;
  memset(&vm, 0, sizeof(vm));
  vm.fn['>'] = vm_forward;
  vm.fn['<'] = vm_backward;
  vm.fn['+'] = vm_increment;
  vm.fn['-'] = vm_decrement;
  vm.fn['.'] = vm_output;
  vm.fn[','] = vm_input;
  vm.fn['['] = vm_while_entry;
  vm.fn[']'] = vm_while_exit;
  for (i = 0; (c = getchar()) != EOF;) {
    if (strchr(TOKENS, c)) {
      vm.cs[i] = c;
      i++;
    }
  }
}

void run() {
  while (vm.cs[vm.ip]) {
    vm.fn[vm.cs[vm.ip]]();
    vm.ip++;
  }
}

int main(int argc, char* argv[]) {
  if (argc > 1) {
    freopen(argv[1], "r", stdin);
  }
  setup();
  run();
  return 0;
}
```

Brainfuck版Hello world的代码如下：

```brainfuck
++++++++++
[>+++++++>++++++++++>+++>+<<<<-]
>++.>+.+++++++..+++.>++.<<+++++++++++++++.
>.+++.------.--------.>+.>.
```

执行结果为：

```bash
$ gcc brainfuck.c -o bf
$ ./bf helloword.bf
Hello World!
```
