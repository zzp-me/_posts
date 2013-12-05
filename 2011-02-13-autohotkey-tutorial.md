---
layout: article
title: AutoHotKey入门教程
date: 2011-02-13 23:21
category: AutoHotKey
excerpt: 官方AutoHotKey入门教程中文翻译
---

AutoHotKey 真是一个好玩的工具！短短几行代码就是先了“窗口置顶”、“窗口透明”等功能，之前我还特意为此装了好几个小工具，现在都可以卸掉了。闲来无事，就把 Quick Start 翻译了一下，我没有逐字逐句地翻译，有时候我嫌原文罗嗦就用自己的话概括地描述了一下。

原文地址：http://www.autohotkey.com/docs/Tutorial.htm

# 教程目录

+ [创建脚本](#Create)
+ [启动程序](#Launch)
+ [模拟鼠标键盘](#Send)
+ [操纵窗口](#Activate)
+ [输入](#Dialogs)
+ [变量与剪切板](#Variables)
+ [循环](#Loop)
+ [操纵文件](#Files)
+ [其他特性](#Overview)

# 创建脚本

每个脚本都是一个纯文本文件，由一些能被 AutoHotKey.exe 执行的命令组成。一个脚本可能还包含[热键](http://www.autohotkey.com/docs/Hotkeys.htm)和[热字符串](http://www.autohotkey.com/docs/Hotstrings.htm)。如果没有热键和热字符串，脚本在启动的时候就会从头依次执行到尾。

创建一个新的脚本：

1. [下载](http://www.autohotkey.com/download/)并安装 AutoHotkey。
1. 右击鼠标，选择*新建 -> 文本文档*。
1. 输入文件名并确保以*.ahk*结尾。例如：Test.ahk。
1. 右击文件，选择*编辑脚本*。
1. 输入以下内容：#space::Run www.google.com

上一行的第一个字符 "#" 代表键盘上的 Windows 键；所以 #space 表示在按住 Windows 键后再按空格键。"::" 后面的命令会在热键激活后执行，在本例中则会打开谷歌主页。继续按下面步骤操作，来执行这个脚本：

1. 保存并关闭该文件。
1. 双击该文件来启动它。在系统托盘里会出现一个新图标。
1. 按下 Windows 和空格键，网页会在默认的浏览器里打开。
1. 右击系统托盘里的绿色图标可以退出或编辑当前脚本。

注意：

+ 可以同时启动多个脚本，并且在系统托盘里都会有一个相应的图标。
+ 每个脚本都能定义多个[热键](http://www.autohotkey.com/docs/Hotkeys.htm)和[热字符串](http://www.autohotkey.com/docs/Hotstrings.htm)。
+ 想让某个脚本开机即启动，可以将它的[快捷方式放到开始菜单的启动目录里](http://www.autohotkey.com/docs/FAQ.htm#Startup)。

# 启动程序

命令[Run](http://www.autohotkey.com/docs/commands/Run.htm)可以运行程序、打开文档、网页链接或快捷键。请参看以下示例：

```bash
Run Notepad
Run C:/My Documents/Address List.doc
Run C:/My Documents/My Shortcut.lnk
Run www.yahoo.com
Run mailto:someone@somedomain.com
```

可以给这些命令设置任何[热键](http://www.autohotkey.com/docs/Hotkeys.htm)。下面第一个例子的快捷键是 Win+N ，第二个是 Control+Alt+C：

```bash
#n::Run Notepad
^!c::Run calc.exe
```

上例是单行热键，因为每个热键之包含一条命令。如果命令多余一条，则热键定义必须单独放一行，其后每条命令放一行，且最后一行必须是[return](http://www.autohotkey.com/docs/commands/Return.htm)。例如：

```bash
#n::
Run http://www.google.com
Run Notepad.exe
return
```

如果待执行的程序或文档没有集成到系统中，则需要指定完整路径：

```bash
Run %A_ProgramFiles%/Winamp/Winamp.exe
```

上例中 `%A_ProgramFiles%` 是[内建变量](http://www.autohotkey.com/docs/Variables.htm#BuiltIn)。比起直接使用诸如 `C:/Program Files` 这样的绝对路径，推荐使用内建变量，它使得脚本的可移植性更好，即能在其他机器上正常运行。注意：命令和变量都是大小写无关的。例如 "Run" 与 "run"、"A_ProgramFiles"与"a_programfiles"都是一样的。

用[RunWait](http://www.autohotkey.com/docs/commands/Run.htm)代替 Run，脚本就会一直等在，直到刚才运行程序退出。在下例中[MsgBox](http://www.autohotkey.com/docs/commands/MsgBox.htm)命令直到记事本被关闭后才执行：

```bash
RunWait Notepad
MsgBox The user has finished (Notepad has been closed).
```

更多关于启动程序的内容 -- 像附带参数、制定工作路径以及获取程序返回值 -- 请点击[这里](http://www.autohotkey.com/docs/TutorialLaunch.htm)。

# 模拟鼠标键盘

用[Send](http://www.autohotkey.com/docs/commands/Send.htm)命令可以向当前活动窗口发送键盘击键消息。下例中定义了热键 Control+Alt+S 来输入签名：

```bash
^!s::
Send Sincerely,{Enter}John Smith
return
```

上例中{Enter}是模拟回车键，其他字符都是字面意思。下一个例子展示了其他几个特殊字符：

```bash
Send ^c!{tab}pasted:^v
```

代码会依次发送 Control+C、Alt+Tab、字符串 "pasted:"和 Control+V。完整的特殊字符列表请参阅[Send](http://www.autohotkey.com/docs/commands/Send.htm)命令。

最后，按键序列还可以用于定义字符串的缩写，即[热字符串](http://www.autohotkey.com/docs/Hotstrings.htm)。例如，下例将 btw 定义为 by the way 的缩写，无论何时你输入 btw 后再输入空格或逗号，都会被替换成 "by the way"：

```bash
::btw::by the way
```

鼠标点击：在发送鼠标点击事件前要先确定鼠标的位置（X, Y 坐标值）。AutoHotKey 自带的 Window Spy 可以很方便地确定鼠标的位置：

1. 启动 Window Spy 。
1. 激活你感兴趣的窗口 (Window Spy 默认置于窗口顶端)。
1. 鼠标指针移到目标位置，Window Spy 就能显示出鼠标的坐标位置（在 Windows XP 或之前的版本中，按下 Shift-Alt-Tab 来激活 Window Spy，以便复制和粘贴 "冻结" 的坐标位置）。
1. 将上面的坐标位置应用于[Click](http://www.autohotkey.com/docs/commands/Click.htm)命令。下例中鼠标在 112, 223 出单击一下左键：*Click 112, 223*

移动鼠标（未按键）用[MouseMove](http://www.autohotkey.com/docs/commands/MouseMove.htm)；拖动鼠标（有按键）[MouseClickDrag](http://www.autohotkey.com/docs/commands/MouseClickDrag.htm)。

# 操纵窗口

用[WinActivate](http://www.autohotkey.com/docs/commands/WinActivate.htm)来激活一个窗口；用[IfWinExist](http://www.autohotkey.com/docs/commands/IfWinExist.htm)或[WinWait](http://www.autohotkey.com/docs/commands/WinWait.htm)判断某个窗口是否存在。以下示例演示这些命令的用法：

```bash
IfWinExist Untitled - Notepad
{
    WinActivate
}
else
{
    Run Notepad
    WinWait Untitled - Notepad
    WinActivate
}
```

例子中首先搜索标题以 "Untitled - Notepad"（忽略大小写）开头的窗口。如果找到了就激活它；否则就启动记事本程序 ，等到窗口以出现就激活它。上例中还使用了[上一次找到的的窗口](http://www.autohotkey.com/docs/LastFoundWindow.htm)避免在每个 WinActivate 后面再次指定标题。

一些常用的窗口管理命令：

+ [IfWinActive](http://www.autohotkey.com/docs/commands/IfWinActive.htm)：检查指定的窗口是否处于激活状态。
+ [WinWaitActive](http://www.autohotkey.com/docs/commands/WinWaitActive.htm)：等待指定的窗口处于激活状态（通常在发送窗口激活指令——比如按下 Control-F 来弹出查找窗口——后使用）。
+ [WinClose](http://www.autohotkey.com/docs/commands/WinClose.htm)：关闭指定的窗口。
+ [WinMove](http://www.autohotkey.com/docs/commands/WinMove.htm)：移动或/且调整窗口大写。
+ [WinMinimize](http://www.autohotkey.com/docs/commands/WinMinimize.htm)，[WinMaximize](http://www.autohotkey.com/docs/commands/WinMaximize.htm)，[WinRestore](http://www.autohotkey.com/docs/commands/WinRestore.htm)：分别是最小化、最大化和恢复指定的窗口。

# 输入

下例显示一个对话框，它有两个按钮（YES 和 NO）：

```bash
MsgBox, 4, , Would you like to continue?
IfMsgBox, No
    return
; Otherwise, the user picked yes.
MsgBox You pressed YES.
```

使用[InputBox](http://www.autohotkey.com/docs/commands/InputBox.htm)提示用户输入一个字符串；[FileSelectFile](http://www.autohotkey.com/docs/commands/FileSelectFile.htm)或[FileSelectFolder](http://www.autohotkey.com/docs/commands/FileSelectFolder.htm)供用户选择文件和文件夹；更高级的任务使用[Gui](http://www.autohotkey.com/docs/commands/Gui.htm)命令来自定义界面。

提示：从其他例子中你可能已经注意到第一个逗号往往被忽略（除非第一个参数为空或执行 := 、 = 或当前命令是[后续语句](http://www.autohotkey.com/docs/Scripts.htm#continuation)唯一的顶部命令)。例如：

```bash
MsgBox This is ok.
MsgBox, This is ok too (it has an explicit comma).
```

# 变量与剪切板

*变量*是脚本用于存储文本或数值的内存区域。在数学运算和比较运算里，只包含数字（可以附带可选的小数点）的变量会自动解析成数值。

除了[函数](http://www.autohotkey.com/docs/Functions.htm#Locals)里的局部变量，所有其他变量都是全局变量。即，它们的值可在脚本的任意地方读取和修改。此外，变量在使用前无需声明，直接使用即可（初始值为空）。

下例演示如何给变量赋字符串：

```bash
MyVar1 = 123
MyVar2 = my string
```

下例演示将变量的值与字符串或数值进行比较：

```bash
if MyVar2 = my string
{
    MsgBox MyVar2 contains the string "my string".
}
if MyVar1 >= 100
{
    MsgBox MyVar1 contains %MyVar1%, which is a number greater than or equal to 100.
}
```

你可能注意到第二个 MsgBox 语句中*MyVar1*被两个百分号包围着。此处会显示*MyVar1*变量的值。在给变量赋值时也可以使用相同的技巧。例如：

```bash
MyVarConcatenated = %MyVar1% %MyVar2%
```

上例中变量 MyVarConcatenated 的值是 "123 my string"（不含引号）。

考虑下面这个例子，它演示两个变量相互比较：

```bash
if (ItemCount > ItemLimit)
{
    MsgBox The value in ItemCount, which is %ItemCount%, is greater than %ItemLimit%.
}
```

注意到上例的第一行包含一对小括号。小括号表示这个 if 语句包含一个[表达式](http://www.autohotkey.com/docs/Variables.htm#Expressions)；缺少它们则看作 "非表达式 if 语句"，于是需要用百分号将 ItemLimit 括起（这样的 if 语句限定只能使用一个比较运算符，即它们不能包含数学运算符或连词 "AND" 和 "OR" 等）。

数学：如下例所示，使用冒号等号（:=）运算符可以将[表达式](http://www.autohotkey.com/docs/Variables.htm#Expressions)的计算结果赋值给一个变量：

NetPrice := Price * (1 - Discount/100)

完整的数学运算符请参阅[表达式](http://www.autohotkey.com/docs/Variables.htm#Expressions)一节。

剪切板：*Clipboard*是一个特殊变量，它代表 Windows 的系统剪切板。虽然如此，你还是可以像普通变量一样使用它。下例会显示当前剪切板里的内容：

```bash
MsgBox %clipboard%
```

考虑下面的例子，它用一段新的文本替代剪切板里当前的内容：

```bash
clipboard = A line of text.`r`nA second line of text.`r`n
```

上面的 \`r 和 \`n （第一个字符是倒引号，键盘 1 左边的那个键）分别代表回车符和换行符。这两个字符会将文本换到新的一行，效果等价于键入键盘上的 Enter 键。

下例演示将文本追加到剪切板（或其他任何变量）：

```bash
clipboard = %clipboard% And here is the text to append.
```

更多内容请参阅[剪切板](http://www.autohotkey.com/docs/misc/Clipboard.htm)和[变量](http://www.autohotkey.com/docs/Variables.htm)两节。

# 循环

[loop](http://www.autohotkey.com/docs/commands/Loop.htm)可以连续地执行一段代码。下例中弹出[MsgBox](http://www.autohotkey.com/docs/commands/MsgBox.htm)窗口三次：

```bash
Loop 3
{
    MsgBox This window will be displayed three times.
}
```

你也可以在 Loop 后面指定一个变量，在迭代次数不确定时这个方法很奏效：

```bash
Loop %RunCount%
{
    Run C:/Check Server Status.exe
    Sleep 60000  ; Wait 60 seconds.
}
```

上例中，除非 RunCount 小于或等于 0，否则循环体至少能执行一次。

根据不同的条件，可以中途跳出循环。执行下例程序后，当用户按住 F1 键，程序就会不断地发送点击鼠标左键事件：

```bash
$F1::  ; Make the F1 key into a hotkey
       ; (the $ symbol facilitates the "P" mode of GetKeyState below).
Loop  ; Since no number is specified with it, this is an infinite loop
      ; unless "break" or "return" is encountered inside.
{
    ; If this statement is true, the user has physically released the F1 key.
    if not GetKeyState("F1", "P")
        break  ; Break out of the loop.
    ; Otherwise (since the above didn't "break"), keep clicking the mouse.
    Click  ; Click the left mouse button at the cursor's current position.
}
return
```

上例中，当用户放开 F1 键，程序就能知道并通过[break](http://www.autohotkey.com/docs/commands/Break.htm)命令终止循环。*Break*使得程序跳到循环体外。

一个替代方案是使用["while" 循环](http://www.autohotkey.com/docs/commands/While.htm)：

```bash
$F1::
while GetKeyState("F1", "P")  ; While the F1 key is being held down physically.
{
    Click
}
return
```

上例演示了通用的循环结构。下面的循环结构适用于一些特殊场合：

[File-reading/writing loop](http://www.autohotkey.com/docs/commands/LoopReadFile.htm)：逐行获取文件内容。它可用于将基于行结构的文件转换成其他格式。还可以用于搜索特定内容。

[Files and folders loop](http://www.autohotkey.com/docs/commands/LoopFile.htm)：逐个遍历文件和文件夹。它允许你逐个处理文件或文件夹。

[Parsing loop](http://www.autohotkey.com/docs/commands/LoopParse.htm)：从字符串中逐个获取子串。比如将 "Red,Green,Blue" 分成三个字段。

[Registry loop](http://www.autohotkey.com/docs/commands/LoopReg.htm)：逐个遍历注册表中的键。

# 操纵文件

下例中使用[FileAppend](http://www.autohotkey.com/docs/commands/FileAppend.htm)将文本追加到文件末尾（或创建一个新文件）。注意：它使用 `n （换行符）来换行：

```bash
FileAppend, A line of text to append.`n, C:/My Documents/My Text File.txt
```

使用[FileDelete](http://www.autohotkey.com/docs/commands/FileDelete.htm)删除一个已存在的文件。例如：

```bash
FileDelete, C:/My Documents/My Text File.txt
```

其他一些常用的、和文件和文件夹相关的命令：

+ [FileRead](http://www.autohotkey.com/docs/commands/FileRead.htm)：将整个文件的内容读取到变量中。
+ [File-reading Loop](http://www.autohotkey.com/docs/commands/LoopReadFile.htm)：逐行读取文件。
+ [IfExist](http://www.autohotkey.com/docs/commands/IfExist.htm)：判断一个文件或文件夹是否存在。
+ [FileSelectFile](http://www.autohotkey.com/docs/commands/FileSelectFile.htm)和[FileSelectFolder](http://www.autohotkey.com/docs/commands/FileSelectFolder.htm)：弹出一个对话框让用户选择文件或文件夹。
+ [FileDelete](http://www.autohotkey.com/docs/commands/FileDelete.htm)/[FileRecycle](http://www.autohotkey.com/docs/commands/FileRecycle.htm)：删除/回收文件。使用[FileRemoveDir](http://www.autohotkey.com/docs/commands/FileRemoveDir.htm)来删除整个文件夹。
+ [FileCopy](http://www.autohotkey.com/docs/commands/FileCopy.htm)/[FileMove](http://www.autohotkey.com/docs/commands/FileMove.htm)：复制/移动文件。使用[FileCopyDir](http://www.autohotkey.com/docs/commands/FileCopyDir.htm)/[FileMoveDir](http://www.autohotkey.com/docs/commands/FileMoveDir.htm)来拷贝/移动整个文件夹。
+ [Files-and-folders Loop](http://www.autohotkey.com/docs/commands/LoopFile.htm)：逐个遍历文件和文件夹。
+ [FileSetAttrib](http://www.autohotkey.com/docs/commands/FileSetAttrib.htm)和[FileSetTime](http://www.autohotkey.com/docs/commands/FileSetTime.htm)：修改文件的属性和时间戳。
+ [IniRead](http://www.autohotkey.com/docs/commands/IniRead.htm)、[IniWrite](http://www.autohotkey.com/docs/commands/IniWrite.htm)和[IniDelete](http://www.autohotkey.com/docs/commands/IniDelete.htm)：创建、访问和维护 INI 文件。
+ [RegRead](http://www.autohotkey.com/docs/commands/RegRead.htm)、[RegWrite](http://www.autohotkey.com/docs/commands/RegWrite.htm)、[RegDelete](http://www.autohotkey.com/docs/commands/RegDelete.htm)和[Registry Loop](http://www.autohotkey.com/docs/commands/LoopReg.htm)：访问 Windows 注册表。

# 其他特性

参阅[命令列表](http://www.autohotkey.com/docs/commands/index.htm)。
