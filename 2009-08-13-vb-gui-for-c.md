---
layout: article
title: 用VB为C程序添加图形界面
date: 2009-08-13 20:36
category: C语言
excerpt:
  很多初学者发现，即便自己把C语言教材从头啃到尾，依然只能写出命令行下的程序。
  本文提供一种简便的方法，使用VB拖出一个图形界面，然后调用现有的C函数。
---

本文的读者定为C语言初学者，介绍的技巧适用于开发迷你型项目或自娱自乐的玩具程序。读者可以抱着茶余饭后休闲娱乐的心态来围观，至于CLI、GUI等名词解释请参看维基百科。

# 诱人的GUI程序

程序的作用就是化繁为简，让计算机高效地帮我们完成枯燥的工作。写程序最大的动力就是你精心设计的程序能获得大家的认可和好评，这其中伴随着发布程序给大家使用。

在C语言编写的操作系统（比如UNIX、Windows等）上，C语言可以说是“无所不能”。但很多初学者发现，即便自己把C语言教材从头啃到尾，依然只能写出命令行下的程序。程序是CLI还是GUI本无可厚非，对我们程序员来说更重要的是程序本身提供的功能，而且CLI相对于GUI还更灵活一些。也许你可以尝试一下把程序发送给一个非计算机科班出身的朋友，估计得大费唇舌来解释程序如何运行，最后还落得一个“不方便”的抱怨。撇开这些不说，至少一个活泼的桌面图标也比死气沉沉的终端图标更吸引人。

但编写GUI程序从来不是一件容易的事情，MFC也好、Swing也罢，都是一堆烦人的接口。用VB可以屏蔽这些细节，界面设计是所见即所得的，拖拖拽拽就能堆出一个像样的界面。本文的原理就是用VB来设计前台界面，C做后台逻辑处理。实现方法就是将C程序打包成DLL文件，由VB程序来调用。

# 所需软件

Dev-C++ 4.9.9.2或以上版本，VB 6.0精简版。这两款软件在华军软件园都能下载到，合起来大小也就15MB左右。如果你有完整版的VB当然更好，不过有精简版的也够用了。

# Dev-C++生成DLL的方法

1. 打开Dev-C++；
1. 点击“文件”-“新建”-“工程”，工程类型选择“DLL”；
1. 语言必须选择“C”，不能选择“C++”；
1. 为工程取一个名字，比如“hello”。确定后会自动生成“dllmain.c”和“dll.h”两个文件；
1. “dllmain.c”里自带了一个函数“DLLIMPORT void HelloWord ()”，但为了能在VB里调用，需要在DLLIMPORT后面添加一个“__stdcall”，即“DLLIMPORT __stdcall void HelloWord ()”，“dll.h”文件中也做同样的修改。没有写“__stdcall”的话会弹出“DLL调用约定错误，错误号：49”，所以请务必小心。
1. 保存文件后，按Ctrl+F9编译。就会在工程的目录下生成一个hello.dll。

# 用VB设计程序界面

1. 打开VB；
1. 新建一个“标准EXE”工程；
1. 拖一个按钮控件（工具箱的第三行第二列）到界面上，如下图。

{% img 1.jpg %}

# 在VB中调用DLL里的函数

在VB中调用DLL里的函数，首先要声明才可调用。声明的格式是

    [Public|Private] Declare [Sub|Function] 函数名 Lib "DLL路径" (参数列表) [as 返回值类型]

其中DLL路径可以是相对路径，也可以是绝对路径；参数列表使用ByVal或ByRef修饰，表示传值还是传引用

1. 双击VB的窗口界面，打开代码窗口；
1. 将一下代码复制到代码窗口里

        Private Declare Sub HelloWorld Lib "hello.dll" ()
        Private Sub Command1_Click()
            HelloWorld
        End Sub

1. 点击“文件”-“生成工程1.exe”，路径选择刚才Dev-C++生成hello.dll所在的目录。
1. 确保“工程1.exe”和“hello.dll”两个文件放在同一个目录下，双击运行“工程1.exe”。
1. 点击按钮“Command1”，就会弹出一个窗口，显示“Hello World from DLL!”则表示调用DLL成功！如下图

{% img 2.jpg %}

现在，你可以自由发挥来编写丰富多彩的GUI程序了！

# 附录：C参数在VB中的声明

| C类型 | VB类型 |
| ----- | ------ |
| char * | ByVal args As String |
| short | Integer |
| int | Long |
| long | Long |
| UINT | Long |
| ULONG | Long |
| WORD,DWORD | Long |
| WPARAM,LPARAM | Long |
| WMSG,UMSG | Long |
| HRESULT | Long |
| BOOL | Boolean |
| COLORREF | Long |
| HWND,HDC,HBRUSH,HKEY,等等. | Long |
| LPSTR,LPCSTR | String |
| LPWSTR,OLECHAR,BSTR | String |
| LPTSTR | String |
| VARIANT_BOOL | Boolean |
| unsignedchar | Byte |
| BYTE | Byte |
| VARIANT | Variant |