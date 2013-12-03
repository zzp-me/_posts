---
layout: article
title: DOS批处理做图形边缘检测
date: 2009-07-03 14:03
category: 图像处理
excerpt:
  不要小看批处理，它也是图灵完备的！今天就用批处理来执行一个拉普拉斯算子求边缘检测给你瞧瞧~
---

图灵机是由输入、输出和状态转移函数三要素组成的，广义上的自动机模型。理论上讲任何任何完备图灵机语言都可用于通用编程，并且和其他完备图灵机语言一样有效。但实际上有些此类语言作用在其特定领域之外时可能令人非常痛苦。例如m4是一种有意的完备图灵机，但实践中把m4当作通用语言使用则非常困难。

最近对一些计算机语言进行分析，总结一门语言做到可编程至少具备以下几点：

+ 输入输出。获得待处理的数据，可以是从标准输入输出，也可以从文件；
+ 算数运算。计算机的核心当然是“计算”，就是普通计算机上提供的加减乘除运算；
+ 内存管理：临时变量值的获取和存储管理，其实有点像通过变量名来查找值的Hash表（这在DOS的批处理中体现的很到位）；
+ 按条件跳转：拥有条件判断（test），加上语句跳转（jump），就能模拟出if、while、for和goto等语句（其中goto的条件为永真，就执行“test(true) jump xxx”一样）。

按照上面的说法，程序解释器很像是一个功能加强了的计算器（原来写个计算器也这么不容易，以前低估它了T_T）。纵观周围的工具，很多看似简陋的小工具原来都符合上面的要求。比如UNIX里的命令行计算器`bc(1)/dc(1)`，都是完备图灵机；DOS的批处理同样也具备，下面来详细讨论。

批处理中可以用set 给变量赋值； `set /a` 可以进行算数运算，在命令行中执行`set /?` 可以查看所有支持的运算符；另外还有if、for、call、goto等语句支持跳转； `set /p` 和echo 可以实现从键盘和屏幕上输出文本信息，但对二进制文件的操作显得有点力不从心（可以用debug 来实现，但貌似 挺复杂）。所以我用C语言写了两个小程序（Bmp2Txt和Txt2Bmp，可到子清行下载二进制文件和C语言源码），解决批处理对BMP文件的输入输出。下面是我用这两个小工具写的拉普拉斯算子求边缘检测的批处理源码：

```bat
::用拉普拉斯算子来做边缘检测
@echo off
setlocal enabledelayedexpansion
if not "%~x1"==".bmp" goto error
if not "%~x2"==".bmp" goto error
Bmp2Txt %1 $$temp$$.txt
::将BMP的像素集合保存到数b
call Txt2Array b < $$temp$$.txt
set t.width=!b.width!
set t.height=!b.height!
for /l %%y in (0,1,!b.height!) do (
	for /l %%x in (0,1,!b.width!) do (
		set ny=%%y
		set nx=%%x
		call :calc
	)
)
::将数组转换成文本文件
call Array2Txt t > $$temp$$.txt
Txt2Bmp %2 $$temp$$.txt
del /q $$temp$$.txt
goto end
::Functions
:error
echo.&echo 　usage: %0 Input_BMP_File Output_BMP_File
goto end
:calc
set /a t[%ny%][%nx%]=4*!b[%ny%][%nx%]!
set /a dy=%ny%-1
set /a dx=%nx%
if defined b[%dy%][%dx%] set /a t[%ny%][%nx%]-=!b[%dy%][%dx%]!
set /a dy=%ny%+1
set /a dx=%nx%
if defined b[%dy%][%dx%] (set /a t[%ny%][%nx%]-=!b[%dy%][%dx%]!)
set /a dy=%ny%
set /a dx=%nx%-1
if defined b[%dy%][%dx%] (set /a t[%ny%][%nx%]-=!b[%dy%][%dx%]!)
set /a dy=%ny%
set /a dx=%nx%+1
if defined b[%dy%][%dx%] (set /a t[%ny%][%nx%]-=!b[%dy%][%dx%]!)
goto :eof
:end
```

在我的机子上跑了一两分钟居然也跑出结果来了，相应的效果图如下。

原始图：

{% img 1.jpg %}

处理后的图片：

{% img 2.jpg %}