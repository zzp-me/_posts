---
layout: article
title: 批量重命名文件
date: 2009-08-29 12:52
category: 命令行超级工具
excerpt:
  批量重命名文件，你会怎么做？下载现有的软件工具？自己开发工具？用Java、C#……？
  如果我是你，我会用批处理或Shell脚本。
---

# 无心插柳

不久前，我们软件工程系举行了全系大会。我在大会上做了简短的报告，主题是“学以致用、动手实践”。报告期间我说了一个亲身经历：以前校园内U盘病毒肆虐，病毒会把U盘里所有的文本文件加上系统属性和隐藏属性，并添加“.tmp”扩展名（例如原文件名为“a.txt”，病毒修改为“a.txt.tmp”），然后生成一个和原文件同名的病毒文件。我不幸中招，于是用我所学的知识写了一个小程序，几秒钟就解决了。

原以为它就像插播广告一样随听随忘，不料言者无心听者有意。几个学弟回去后也都竭尽所能玩了一把：有用 Java 的，也有用 C# 的……功能也五花八门，不仅能批量修改扩展名，还能按 1-N 的顺序批量重命名文件等等。看他们玩得不亦乐乎，想来那次报告还有点作用嘛^_^。

# 我的解决方法

今天又被问起我那时候的程序是怎么写的。其实在我们大一那会儿，要用计算机只能到图书馆的电子阅览室，那里的系统“干干净净”，不会有 VC++ 等编译器，而且那种打开网页需要半分钟的速度也不会让你考虑去下载（估计其他高校情况也差不多），所以我是用批处理解决的。我用了下面三条命令：

```bat
::a.bat
@echo off
::删除所有病毒文件
del /F /S /Q *
::递归地去掉文件的隐藏属性
attrib -s -h -r * /S /D
::遍历目录，去掉 tmp 扩展名
for /R . %%f in (*.tmp) do ren %%f %%~nf
```

上面的程序中是为了去掉“.tmp”这个扩展名，所以用 for 命令来遍历。但如果仅仅是修改扩展名则更简单，只要运行 ren *.bat *.txt 即可。另附一段好玩的批处理：将下面的代码保存成 bat 文件，双击运行就能将当前目录下的 jpg 图片文件按 1-N 的顺序重新命名（用来批量处理从网上下载下来的图片非常有效）。大家可以自行修改扩展名。

```bat
::b.bat
@echo off
::开启延迟的变量扩充
setlocal enabledelayedexpansion
::计数器
set /a i=1
for /R . %%f in (*.jpg) do (
  ren %%f !i!.jpg
  set /a i=!i!+1
)
```

# 批量重命名汇总

只要大家悉心观察，还有很多好玩的问题可以去解决。现将各种批量重命名的批处理代码汇总如下：

## 按顺序重命名文件

将当前目录下不规则命名的 jpg 文件依次重命名成 1.jpg、2.jpg 等

```bat
@echo off
::开启延迟的变量扩充
setlocal enabledelayedexpansion
::计数器
set /a i=1
for %%f in (*.jpg) do (
  ren %%f !i!.jpg
  set /a i=!i!+1
)
```

## 递归地处理子目录文件

其他代码都是处理当前目录下的文件，这个程序给出递归处理子目录的模板

```bat
for /R . %f in (*) do echo %f
```

## 去掉文件名日期前缀

需求出处：http://www.oschina.net/code/snippet_125800_4330

```bat
for %f in (*.sc2replay) do for /F "delims=- tokens=4*" %t in ("%f") do move %f %t
```

## 替换文件名中的字串

需求出处：http://www.oschina.net/code/snippet_143158_4337

```bat
@echo off
setlocal enabledelayedexpansion
for %%f in (*) do (
  set name=%%~nf
  set ext=%%~xf
  move !name!.!ext! !name:%1=%2!.!ext!
)
```

## 将文件名变成大写

需求出处：http://www.oschina.net/code/snippet_99867_4340

```bat
:: convert file name to upper case
@echo off
setlocal enabledelayedexpansion
set LowerCase=abcdefghijklmnopqrstuvwxyz
set UpperCase=ABCDEFGHIJKLMNOPQRSTUVWXYZ

for %%f in (*.txt) do (
  set string=%%f
  for /L %%d in (0,1,25) do (
    set from=!LowerCase:~%%d,1!
    set to=!UpperCase:~%%d,1!
    call :convert !from! !to!
  )
  move %%f !string!
)

goto end

:convert
set string=!string:%1=%2!
goto :eof

:end
```

# Shell版批量重命名

功能和上述批处理一致，但使用bash来完成

## 按顺序重命名文件

将当前目录下不规则命名的 jpg 文件依次重命名成 1.jpg、2.jpg 等

```bash
i=1
for f in *.jpg; do
  mv $f ${i}.jpg
  ((i++))
done
```

## 递归地处理子目录文件

其他代码都是处理当前目录下的文件，这个程序给出递归处理子目录的模板

```bash
# 处理子目录文件
find . -name '*.log' -type f | while read p; do
  fname=$(basename $p)
  # do some thing for fname
  mv $p $(dirname $p)/$fname
done
```

## 去掉文件名日期前缀

需求出处：http://www.oschina.net/code/snippet_125800_4330

```bash
for f in *.sc2replay; do mv $f ${f##*-}; done
```

## 替换文件名中的字串

需求出处：http://www.oschina.net/code/snippet_143158_4337

```bash
for f in *; do
  name=${f%.*}
  ext=${f##*.}
  mv $f `sed 's/e/x/g' <<< $name`.$ext
done
```

## 将文件名变成大写

需求出处：http://www.oschina.net/code/snippet_99867_4340

```bash
for f in *; do
  name=${f%.*}
  ext=${f##*.}
  mv $f `tr 'a-z' 'A-Z' <<< $name`.$ext
done
```
