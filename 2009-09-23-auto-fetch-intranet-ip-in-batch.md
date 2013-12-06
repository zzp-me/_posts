---
layout: article
title: 自动获取内网可用IP
date: 2009-09-23 14:11
category: 命令行超级工具
excerpt:
  在寝室上内网真的是很闹心呀～稍微迟点开机，IP 地址就被别人抢去了。
  为此，我开了这个批处理脚本，自动检查并设置可用的IP地址。
---

在寝室上内网真的是很闹心呀～稍微迟点开机，IP 地址就被别人抢去了。昨晚九点放学后，一直等到十点半才连上，实在忍无可忍了！

IP 被抢后解决方法无非就是找一个没被占用的地址，但手工去测试 256 个地址太折磨人了，早些时候我们班的两个同学就分别用 Java 和 C# 来实现过类似的小工具。但解决这样的小问题有点杀鸡用牛刀的感觉，不符合我的性格^_^，我当然还是用批处理来解决。

批处理中核心的两条语句就是：

<dl>
  <dt>设置静态IP地址</dt>
  <dd>netsh int ip set address &quot;本地连接&quot; static IP地址 子网掩码 网关 1</dd>
  <dt>设置域名解析器</dt>
  <dd>netsh int ip set dns &quot;本地连接&quot; static 域名解析器地址 primary</dd>
</dl>

程序使用时需要提供一个网关地址，为了方便大家使用，在 XP 和 Vista 系统能自动检测网关地址，如果正确的话直接敲一个回车就可以了。下面是批处理的代码，大家只要将下面的代码复制到记事本里，另存为 sip.bat，双击即可运行。

```bat
:: 自动查找可用 IP
:: 适用于浙江工商大学
:: by redraiment
@echo off
setlocal enabledelayedexpansion

if NOT "%1"=="" goto scan

:: 检测可识别的系统版本
:: XP
ver | find "[版本 5"
if NOT ERRORLEVEL 1 (
  set SV=XP
  set CG='"ipconfig|find /I "gate""'
  set TS=13
)

:: VISTA
ver | find "[版本 6"
if NOT ERRORLEVEL 1 (
  set SV=VISTA
  set CG='"ipconfig|find /V "::"|find "网关""'
  set TS=15
)

if NOT "%SV%"=="" (for /F "tokens=%TS%" %%i in (%CG%) do set g=%%i)
echo 您的网关可能是：!g!
set /P gateway=如果不正确请重新输入，否则输入回车：
if "!gateway!"=="" set gateway=!g!
echo %0 !gateway!
pause
exit

:scan
for /l %%d in (2,1,253) do (
  echo 正在验证 %~n1.%%d 是否被占用 ...
  ping -w 10 -n 1 %~n1.%%d | find /I "TTL"
  if ERRORLEVEL 1 (
    echo 设置本地IP ...
    netsh int ip set address "本地连接" static %~n1.%%d 255.255.255.0 %1 1

    echo 测试连接网关 ...
    ping -w 10 -n 1 %1 | find /I "TTL"
    if NOT ERRORLEVEL 1 (
      echo 恭喜，%~n1.%%d 可用!
      echo 设置域名解析器 ...
      netsh int ip set dns "本地连接" static 210.33.88.1 primary
      exit
    )
  )
)
```

下面是在 XP 环境下运行的效果图，它将本地 IP 设置成 10.230.71.6：

{% img 1.jpg %}
