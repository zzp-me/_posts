---
layout: article
title: 替换成连续的数列
date: 2013-04-28 10:13:54
category: Emacs
excerpt:
  日常开发中有一类编辑操作，需要用一组连续的数列依次替换某个内容。
  本文向你展示在Emacs中如何实现这个功能！
---

# 问题描述

日常开发中有一类编辑操作，需要用一组连续的数列依次替换某个内容。比如要把下面的内容：

    insert into tbl values ('JOE0');
    insert into tbl values ('JOE0');
    insert into tbl values ('JOE0');
    insert into tbl values ('JOE0');
    insert into tbl values ('JOE0');

替换成如下内容：

    insert into tbl values ('JOE1');
    insert into tbl values ('JOE2');
    insert into tbl values ('JOE3');
    insert into tbl values ('JOE4');
    insert into tbl values ('JOE5');

即把0自动替换成一个递增的序列。下面用Emacs抛砖引玉。

# 相关Emacs特性

大部分编辑器都支持正则表达式替换，允许使用小括号分组并在后面的替换中使用诸如`\1`、`\2`、`\3`等向前引用。比如把`<h1>`和`<h2>`等标题的内容清空可以使用“`\(<h[12]>\)[^<]*`”替换成“`\1`”，其中`\(`和`\)`就是捕捉分组，`\1`则是引用捕捉到的内容。

Emacs中不仅能支持`\1`、`\2`等向前引用的方法，还提供了更高级的功能：“`\,`”。“`\,`”后面可以接一句Emacs Lisp表达式，它会把执行的结果替换到目标位置。比如把所有的HTML标签都替换成大写，即`<backquote>`替换`<BACKQUOTE>`，可以使用“`\(<[^>]+>\)`”替换成“`\,(upcase \1)`”，其中upcase是Emacs Lisp的函数，可以把一个字符串中的小写字母转成大写。

# 解决方法

使用上述特性解决文章开头的问题，只需使用incf函数即可，它的功能类似C语言中的 ++i。操作如下（其中“|”为光标所在位置）：

1. 将光标移到起始位置

        |insert into tbl values ('JOE0');
        insert into tbl values ('JOE0');
        insert into tbl values ('JOE0');
        insert into tbl values ('JOE0');
        insert into tbl values ('JOE0');

1. 按C-M-%，即执行正则表达式替换
1. 根据提示输入要替换的内容“0”，敲回车
1. 再输入目标内容“\,(incf count)”，其中count为变量名
1. 敲回车，得到如下内容。

        insert into tbl values ('JOE1');
        insert into tbl values ('JOE2');
        insert into tbl values ('JOE3');
        insert into tbl values ('JOE4');
        insert into tbl values ('JOE5|');
