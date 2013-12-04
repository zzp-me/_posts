---
layout: article
title: 用awk去除C风格注释
date: 2010-01-06 22:16
category: 命令行超级工具
excerpt:
  使用awk脚本取出C语言风格的代码注释：单行注释（//）与多行注释（/**/）
---

今天闲逛Linux宝库，看到论坛里有人在讨论如何用 shell 脚本来处理 C 语言注释，发帖时间是 08-10-23（以前怎么都没注意到，失败...），但问题好像并没被解决。正好这两天玩 sed & awk，来小试一下身手。

# C语句注释

本文讨论的是 C99 标准，它支持单行注释（“// ...”）和块注释（“/*...*/”），并且当单行注释以“/”结尾时也可以跨多行。测试代码如下：

```c
#include <stdlib.h>
#include <stdio.h>
int main (int argc, char *argv[])
{
// not show/
not show/
not show
// not show
/* not show */
    int is; // not show
    int/* not show */ ms; /* not show */
    double ds; // not show/
    not show/
    not show
    double dm; /* ...
    not show
    not show */ float fs; /**
                           * now show
                           */
    float/**/ fm;
    char cs[] = "aaa // /***/";
    char cm1[] = /* not show */"hello*/";
    char cm2[] = "/*redraiment"/* not show */;
    /* printf("/////"); */
    return EXIT_SUCCESS;
}
```

其中灰色部分就是注释，经过处理后需要将它们全部移除或用替换成空字符。论坛原帖中没有处理以“/”结尾的单行注释，也没处理注释关键字出现在字符串中的情况。

# 工具的选择

sed 是一个流编辑器，它能对文件进行“插入”、“删除”、“替换”、“追加”等编辑操作；而 awk 是一门模式匹配的程序设计语言，它除了能编辑文本还可以统计信息，你可以把它看成基于文本文件的数据库系统。原帖中作者使用 sed 来解决，因为问题涉及的操作仅仅是删除 C 代码中的注释。但由于以下原因导致 sed 心有余而力不足：

## 一、不支持最小匹配

正则表达式默认采用贪心匹配策略，在正则的标准中通过在量词后面加“?”来使用最小匹配策略，详细规则介绍请参见[这里](http://deerchao.net/tutorials/regex/regex.htm#greedyandlazy)。问题中多行注释必须使用最小匹配原则，如果关键词只有一个字符，就可以通过排除字符集来模拟，比如我们经常用“"[^"]*"”来匹配一个字符串。可惜 C 语言的注释关键词都是多字符。

## 二、排序字符（Collating Symbols）只是一个美丽的梦想

排序字符用于字符列表（character list）中。按照[文档](http://www.gnu.org/software/sed/manual/html_node/Regular-Expressions.html#Regular-Expressions)的描述，排序字符可将多个字符当一个字符来匹配。比如模式“[[.ch.]i]”可以匹配行“char”和“int”，但不能匹配“coho”。如果它能被支持，就可以把“/*”、“*/”、“//”都看做一个字符，通过（一）中的排除字符集来实现最小匹配。可惜到目前为止，我接触的工具中没有一款支持这个特性，更不用说对正则支持平平的 sed 了。

## 三、字符串来捣乱

在《[sed单行脚本学习笔记]({% post_url 2009-12-31-sed-one-line %})》中我给出了一段 sed 单行脚本，用于替换不在字符串中的模式。看起来正适合解决这个问题，但它的前提是模式要和待修改的文本完全匹配。由（一）、（二）两个条件决定了 sed 实现的正则表达式无法匹配 C 语言所有类型的注释。另外，sed 提供的控制语句是“b、t”，它们的功能是类似于 C 语言的 goto，因此它不能像“if ... else ...”一样方便地判断某个注释的起始位置是否在字符串中。

由于上述原因，我们需要一个变量来记录当前状态——是否在字符串中。因此我使用 awk 来解决。

# 我的解决方案

```awk
# filename: strip_c_comment.awk
# issue: awk -f scrip_c_comment.awk test.c
BEGIN {
  FS=""
}

!(ignore_line && $NF == "//") && !ignore_line-- {
  ignore_line = 0;

  for(i = 1; i <= NF; i++) {
    if (ignore_block) {
      if ($i $(i+1) == "*/") {
        ignore_block = 0
        i++ # remove '*'
      }
      continue
    }

    if (!instr && $i $(i+1) == "/*") {
      ignore_block = 1
      i++ # remove '/'
      continue
    }

    if (!instr && $i $(i+1) == "//") {
      ignore_line = ($NF == "//")? 1: 0
      break
    }

    if ($i == "/"") {
      instr = 1 - instr
    }

    printf($i)
  }

  printf("/n")
}
```

在开始时将 FS 设为空字符串，使得输入记录的每个字符都成为一个独立的字段。代码中的三个布尔变量分别代表：

<dl>
  <dt>ignore_line</dt>
  <dd>如果上一行是以“/”结尾的单行注释则为“True”；</dd>
  <dt>ignore_block</dt>
  <dd>如果当前字符在块注释中则为“True”；</dd>
  <dt>instr</dt>
  <dd>如果当前字符在非注释的字符串内则为“True”。</dd>
</dl>

脚本的工作就是保留“`ignore_line`”和“`ignore_block`”都为“False”时的字符。

# 执行结果

```c
#include <stdlib.h>
#include <stdio.h>
int main (int argc, char *argv[])
{
    int is; 
    int ms; 
    double ds; 
    double dm; 
 float fs; 
    float fm;
    char cs[] = "aaa // /***/";
    char cm1[] = "hello*/";
    char cm2[] = "/*redraiment";
    
    return EXIT_SUCCESS;
}
```
