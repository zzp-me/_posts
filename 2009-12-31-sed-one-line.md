---
layout: article
title: sed单行脚本学习笔记
date: 2009-12-31 18:07
category: 命令行超级工具
excerpt:
  sed是UNIX下的流编辑器（stream editor），sed脚本通常以开始都很小，并且写和读都很简单。
  在测试脚本时，可能会发现不适用于一般规则的特殊情况。
  为了解决这些问题，可以给脚本增加行，生成更长、更复杂并且更完整的脚本。
  虽然花费在细化脚本上的时间抵消了不用手动编辑而节省下来的时间，但至少在这段时间内，
  你的头脑被自己的这个似乎熟悉的想法占据：“看！计算机完成的。”
---

# 回家真好

前段时间忙着找工作、项目结题、写报告……反正是总有做不完的事情，哈哈。好在暂时告一段落了，应老妈强烈要求回家休息几天。这次回家除了这身衣服，只带了一本《sed与awk》，我觉得这种小册子最适合茶余饭后休闲之用。如果你也有兴趣学 sed ，推荐你一起看（可以在[谷歌图书](http://books.google.com/)在线阅读英文版:D）。

花了两天时间，看完了前面 sed 的部分。要掌握一个工具就要熟悉它的规则，man 等参考手册向我们介绍这些规则，教程则演示如何使用这些规则，但要将这些规则运用自如，还需要去理解别人的代码并尝试自己解决问题。在[SourceForge](http://sed.sourceforge.net/)上有份经典的文档：《[SED单行脚本快速参考](http://sed.sourceforge.net/sed1line_zh-CN.html)》（单行脚本要求命令行长度小于65个字符），由[Eric Pement](mailto:pemente@northpark.edu)整理，[Joe Hong](mailto:hq00e@126.com)翻译，通篇阅读后获益良多，故撰此文和大家分享。

# 精彩脚本摘录

```bash
# 在每一行后面增加一空行
sed G
```

在参考手册中，命令G的作用是“将换行符后的保持空间内容追加到模式空间”。就像前文提到的，看过教程后只是熟悉了规则，还不能将规则运用自如，我自己写的代码是：sed 's/$/\n/'，就是因为我还不熟悉每个命令会对模式空间产生什么影响。所以看到这段参考代码时感觉眼前一亮：“原来还可以这样写！”

```bash
# 显示文件中的最后10行 （模拟“tail”）
sed -e :a -e '$q;N;11,$D;ba'
```

假设文件有 N 行（N 大于10），显示最后10行也就意味着删除前的 N-10 行。在多行模式中，命令“D”可以删除模式空间中第一行；命令“N”可以将下一行追加到模式空间中，建立多行模式。因此问题转化为：“1)将整个文件的内容放入一个模式空间中；2)删除前 N-10 行。”其中问题1)通过控制语句“b”来解决：sed ':a; N; ba'；至于问题二，模式“1,$”代表文本中的所有行，因此紧跟着的命令被执行N次，同理，模式“11,$”匹配后面的 N-10 行，因此“11,$D”一个执行了 N-10 次。

其实，在 GNU sed 中，命令“$q”是可以删掉的，因为在最后一行执行命令“N”就会因出错而自动退出。

另外，在 info 手册中也有一个解决方法：sed '1h;2,10{H;g};$q;1,9d;N;D'。他的思路差不多，只不过是将中间文本保持在“保持空间”而不是“模式空间”，因此无需通过控制语句来制造循环。但它繁琐一些，需要在两处指定地址范围。

```bash
# 显示文件中的最后2行（模拟“tail -2”命令）
sed '$!N;$!D'
```

这条脚本也很精彩，命令“$!N”只能执行到倒数第二行，除了倒数第二行，命令“$!D”都能被执行，因此仅剩下最后两行未被删除。我的解决方法要麻烦一些：sed -n 'N;$p;D'，命令“$p”只能执行在倒数第二行执行，并且输出最后两行。当文件只有一行时，两段脚本都没有输出。

# 我的解决方法

显示文件中的倒数第二行

<dl>
  <dt>当文件中只有一行时，输入空行</dt>
  <dd>sed -e &#39;$!{h;d;}&#39; -e x</dd>
  <dt>当文件中只有一行时，显示该行</dt>
  <dd>sed -e &#39;1{$q;}&#39; -e &#39;$!{h;d;}&#39; -e x</dd>
</dl>

我的解决方法

<dl>
  <dd>sed -e &#39;1{$d;}&#39; -e &#39;$!{h;d;}&#39; -e x  # 当文件中只有一行时，不输出</dd>
  <dt>当文件中只有一行时，输入空行</dt>
  <dd>sed &#39;x;$!d&#39;</dd>
  <dt>当文件中只有一行时，显示该行</dt>
  <dd>sed &#39;1h;1!x;$!d&#39;</dd>
  <dt>当文件中只有一行时，不输出</dt>
  <dd>sed -n &#39;N;$P;D&#39;</dd>
</dl>

在解决这个问题上，参考代码显得有些繁琐。只有将每行都交换“模式空间”和“保持空间”的内容（命令“x”），并将除最后一行外所有内容删除（命令“$!d”），就能获得倒数第二行，因为“保持空间”初始化时为空，因此当文件中只有一行时输入空行；为了在文件中只有一行时能显示该行，要对第一行特殊照顾：覆盖保持空间；第三条命令你很熟悉，咋一看以为是上面“tail -2”的解决方法，它们很像，差别仅仅是“tail -2”中“p”是小写，此处是大写。

<dl>
  <dt>删除文件中的重复行，不管有无相邻。注意hold space所能支持的缓存大小，或者使用GNU sed。</dt>
  <dd>sed -n &#39;G; s/\n/&amp;&amp;/; /^/([ -~]*\n/).*\n\1/d; s/\n//; h; P&#39;</dd>
  <dt>我的解决方案</dt>
  <dd>sed -n &#39;G; s/\n/&amp;&amp;/; /^/([^\n]*\n/).*\n\1/d; s/\n//; h; P&#39;</dd>
</dl>

参考代码中运用了一个小技巧：用模式“[ -~]”来匹配所有可打印字符。可打印字符的ASCII范围是0x20-0x7F，而0x20和0x7F分别是空格和波浪线。但这个技巧不能在 GNU sed v4.1.5 中使用（但在 v4.0.7 中却可以使用）。为了使代码通用，需要改为“[^\n]”。

```bash
# 只保留多个相邻空行的第一行。并且删除文件顶部和尾部的空行。
# （模拟“cat -s”）
sed '/./,/^$/!d'        #方法1，删除文件顶部的空行，允许尾部保留一空行
sed '/^$/N;/\n$/D'      #方法2，允许顶部保留一空行，尾部不留空行
```

在我的环境里测试，方法2尾部同样保留一个空行。

# 我的单行脚本

我看的兴起，也设计了一个单行脚本。问题来源于设计宏替换器，比如C语言中有如下定义：

```bash
#define PRINT printf
PRINT("printf with PRINT");
```

此时使用 s/PRINT/printf/g 就会把字符串中的“PRINT”也替换掉。因此需要使用一下脚本：

```bash
# 只替换不在字符串内的模式
sed -r 's/^|"[^"]*"/&\n/g; :a; s/(\n[^"]*)foo/\1bar/; ta; s/\n//g'
# 只替换字符串中的模式
sed -r 's/^|"[^"]*"/\n&/g; :a; s/(\n"[^"]*)1/\1x/; ta; s/\n//g'
```

替换不在字符串内模式的原理是：

1. 先在起始位置和字符串的第二个引号后面添加换行符（s/^|"[^"]*"/&\n/g）；
1. 替换所有将换行符和第一个引号之间的模式，这些字符都不在字符串里面（s/(\n[^"]*)foo/\1bar/）；
1. 迭代执行第二部，直到替换所有模式（ta）；
1. 删除所有换行符（s/\n//g）。

其中换行符是作分隔符用，你也可以使用其他的、不在该行中的字符。替换字符串内模式的原理基本相同，只是分隔符放到字符串第一个引号的前面，并替换以引号开头的模式。下面是测试结果（将字符‘1’替换为‘x’）：

文件string的内容：

```bash
123"123"123
111"44a"jjl
dad"111"ddd
"111"44"5555"
"111""""333"
1122
"1111"2211"1111"
4455
"9988"
"1155"
```

执行 sed -r 's/^|"[^"]*"/&\n/g;:a;s/(\n[^"]*)1/\1x/;ta;s/\n//g' string

```bash
x23"123"x23
xxx"44a"jjl
dad"111"ddd
"111"44"5555"
"111""""333"
xx22
"1111"22xx"1111"
4455
"9988"
"1155"
```

执行 sed -r 's/^|"[^"]*"/\n&/g;:a;s/(\n"[^"]*)1/\1x/;ta;s/\n//g' string

```bash
123"x23"123
111"44a"jjl
dad"xxx"ddd
"xxx"44"5555"
"xxx""""333"
1122
"xxxx"2211"xxxx"
4455
"9988"
"xx55"
```

# 附

我的环境是 Debian Lenny + GNU sed 4.1.5。Windows版本可以到 GNUWin32 下载最新的 Sed for Windows，也可以发邮件向我索取 GNU sed.exe v4.0.7。GNU sed 自 v3.02.80 起可以使用转义字符'\t'来代替制表符，其他大部分他版本还不能识别'\t'的简写方式。下面摘录《sed与awk》中很好玩的一段话：

> *像许多程序一样，sed脚本通常以开始都很小，并且写和读都很简单。在测试脚本时，可能会发现不适用于一般规则的特殊情况。为了解决这些问题，可以给脚本增加行，生成更长、更复杂并且更完整的脚本。虽然花费在细化脚本上的时间抵消了不用手动编辑而节省下来的时间，但至少在这段时间内，你的头脑被自己的这个似乎熟悉的想法占据：“看！计算机完成的。”*
> 
> ——《sed与awk》 P119
