---
layout: article
title: 正则表达式匹配double值
date: 2008-07-11 12:38
category: 正则表达式
excerpt:
  介绍如何用正则表达式匹配多种情况，以及匹配一段数值区间。
---

近来在学正则表达式，看到有其他人在尝试用正则表达式来匹配检查一个字符串是否是合法的Double类型了，我也手痒来发挥一下。这在实际开发中可能用不上，例如Java中判断一个字符串是否是合法的数值，可以用现成的Double.parseDouble等来完成。不过，这样的练习有助于掌握正则表达式的使用。

本文代码用Perl实现，因为Perl对正则表达式的支持是最好的。让我们一步一步来解刨double浮点数：

# 一、普通整数

普通的整数都是合法double型，带上后缀字母d或者D也是double型。比如：1、+10、-100、100d、987D。即：

```perl
/^[-+]?\d+[dD]$/
```

# 二、实数

double型的变量还能保存实数。比如：`1.0`、`+12.34`、`-45.896`、`-563.887d`甚至`124.`、`.123`、`-.687`在Java中也都是合法double值。规则如下：

1. 如果小数点前有整数的话，那小数点和小数的出现可以随意；
2. 如果小数点前面没有数字的话，默认以0做整数，但小数必须出现。

也就是说，只有一个小数点的话，那就是非法的double数。因此，我们要用多选分支来做：

```perl
/^[-+]?(\d+(\.\d*)?|\.\d+)[dD]?$/
```

# 三、科学计数法

double变量还有一种形式——科学计数法。指数部分只能是十进制整数，允许为负数。比如：`1e123`、`12.546e54d`、`4335.546E33`、`-.54e-7D`，因此还要在尾部加上指数的匹配。指数没有小数，所以只要匹配整数就可以。

```perl
/^[-+]?(\d+(\.\d*)?|\.\d+)([eE][-+]?\d+)?[dD]?$/
```

# 四、取值范围

最后，也是最麻烦的，就是Java中double的取值范围：负数范围：从-1.7976931348623157×10<sup>+308</sup>到-4.94065645841246544×10<sup>-324</sup>正数范围：从4.94065645841246544×10<sup>-324</sup>到1.7976931348623157×10<sup>+308</sup>这也就是说，指数的取值范围在 -324 到 308 之间。如果仅仅是这样，那还好说，关键是前面的小数还这么复杂。为了简化问题，我就把指数的范围在 -323 到 307 之间，对前面的小数没有限制。那首先来匹配 -307 到 307

```perl
/[-+]?([012]?\d{1,2}|30[0-7])/
```

然后另外在匹配 -308 到 -324

```perl
/-3([01]?[4-9]|[012]?[0-3])/
```

合并起来，就是：

```perl
/^[-+]?(\d+(\.\d*)?|\.\d+)([eE]([-+]?([012]?\d{1,2}|30[0-7])|-3([01]?[4-9]|[012]?[0-3])))?[dD]?$/
```

# 实例测试

用这个正则表达式来编写一个perl脚本：

```perl
#!/usr/bin/perl

while ($line = <STDIN>) {
  chomp($line);
  if ($line =~ /^[-+]?(\d+(\.\d*)?|(\.\d+))([eE]([-+]?([012]?\d{1,2}|30[0-7])|-3([01]?[4-9]|[012]?[0-3])))?[dD]?$/) {
    print $line, " is Java double!\n";
  } else {
    print $line, " is not Java double\n";
  }
}
```

测试数据：

    +
    -
    .
    e
    a
    1.
    .1
    0.1
    -.
    -.1
    +.1
    1e123d
    -3.543E4456D
    -45.54879e-323d
    -777.1234E-324
    4553.876e357
    1543e307
    13.54e15.78d
    213.e123d
    1.2.3
    -.1.1
    +e
    e12

输出结果：

    + is not Java double
    - is not Java double
    . is not Java double
    e is not Java double
    a is not Java double
    1. is Java double!
    .1 is Java double!
    0.1 is Java double!
    -. is not Java double
    -.1 is Java double!
    +.1 is Java double!
    1e123d is Java double!
    -3.543E4456D is not Java double
    -45.54879e-323d is Java double!
    -777.1234E-324 is not Java double
    4553.876e357 is not Java double
    1543e307 is Java double!
    13.54e15.78d is not Java double
    213.e123d is Java double!
    1.2.3 is not Java double
    -.1.1 is not Java double
    +e is not Java double
    e12 is not Java double
