---
layout: article
title: 安装Jekyll
category: Jekyll
date: 2013-12-02 14:45:41
excerpt: 在不同的平台（Windows、Linux和OS X）上安装Jekyll
---

# Mac OS X 用户

如果你使用的是苹果电脑，那么恭喜你，Ruby等一切已经准备就绪，你所要做的就是在终端执行以下命令：

```bash
gem install jekyll
```

# Linux 用户

如果你用的是Linux，或许Ruby不是默认安装的，但安装起来也很方便。以Debian系统为例，安装Jekyll需要执行以下命令：

```bash
apt-get install ruby ruby-dev
gem install jekyll
```

# Windows 用户

如果你是Windows用户，这篇文章就是为你准备的！

## 1. 下载Cygwin

到[www.cygwin.com](http://www.cygwin.com)下载最新的安装包：[32位](http://cygwin.com/setup-x86.exe)、[64位](http://cygwin.com/setup-x86_64.exe)。

## 2. 安装Ruby和GCC

启动第一步下载的安装包，在选择软件列表中选中Ruby，如下图所示：

{% img 1.png %}

使用相同的方法，安装gcc、make以及libcrypt-devel。

## 3. 安装Jekyll

经过以上的准备步骤，可以开始安装Jekyll了：

```bash
gem install posix-spawn
gem install jekyll
```

## 4. 降级pygments.rb

安装完Jekyll后，默认会安装以来的pygments.rb库，这个库用于高亮代码。但最新版的pygments.rb无法在cygwin中正常使用，需要降级成0.5.0版本。

```bash
gem uninstall pygments.rb --version '>0.5.0'
gem install pygments.rb --version 0.5.0
```
