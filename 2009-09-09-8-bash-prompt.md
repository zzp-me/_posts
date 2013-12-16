---
layout: article
title: 八个有趣实用的Bash提示符
date: 2009-09-09 15:02
category: 命令行超级工具
excerpt:
  很多人并不关心命令提示符，觉得它没用。
  刚从互联网上搜罗了几个有趣且实用的 Bash 提示符。
  好的提示符或许能改善你的工作方式～
---

原文链接：http://maketecheasier.com/8-useful-and-interesting-bash-prompts/2009/09/04

很多人并不关心命令提示符，觉得它没用。刚从互联网上搜罗了几个有趣且实用的 Bash 提示符。好的提示符或许能改善你的工作方式～

注：要使用下面的效果，只需将“PS1=”部分复制粘贴到终端执行即可。如果要保持修改，可以将它追加到“~/.bashrc”文件中。

# 一、执行成功就显示笑脸

如果命令执行成功，就显示一张笑脸。效果和代码如下：

![prompt](http://mcdn.maketecheasier.com/wp-content/uploads/2009/08/bashprompts-happyface.jpg)

```bash
PS1="\`if [ \$? = 0 ]; then echo \[\e[33m\]^_^\[\e[0m\]; else echo \[\e[31m\]O_O\[\e[0m\]; fi\`[\u@\h:\w]\\$ "
```

# 二、遇错误命令改变颜色

效果和上面类似，只是改变了提示符的颜色；另一个亮点就是它在最左边显示历史记录的数量。效果和代码如下：

![prompt](http://mcdn.maketecheasier.com/wp-content/uploads/2009/08/bashprompts-hurring.jpg)

```bash
PROMPT_COMMAND='PS1="\[\033[0;33m\][\!]\`if [[ \$? = "0" ]]; then echo "\\[\\033[32m\\]"; else echo "\\[\\033[31m\\]"; fi\`[\u.\h: \`if [[ `pwd|wc -c|tr -d " "` > 18 ]]; then echo "\\W"; else echo "\\w"; fi\`]\$\[\033[0m\] "; echo -ne "\033]0;`hostname -s`:`pwd`\007"'
```

# 三、多行提示符

利用多行可以显示更多的内容。比如下列中显示了当前日期和时间、完整路径、用户和主机名、活动的终端以及当前目录文件数量和所用空间。

![prompt](http://mcdn.maketecheasier.com/wp-content/uploads/2009/08/bashprompts-informant.jpg)

```bash
PS1="\n\[\033[35m\]\$(/bin/date)\n\[\033[32m\]\w\n\[\033[1;31m\]\u@\h: \[\033[1;34m\]\$(/usr/bin/tty | /bin/sed -e 's:/dev/::'): \[\033[1;36m\]\$(/bin/ls -1 | /usr/bin/wc -l | /bin/sed 's: ::g') files \[\033[1;33m\]\$(/bin/ls -lah | /bin/grep -m 1 total | /bin/sed 's/total //')b\[\033[0m\] -> \[\033[0m\]"
```

# 四、七彩提示符

这个的特点就是用不同的颜色来显示不同的信息。如下列中分别显示了时间、用户名、主机名以及当前目录：

![prompt](http://mcdn.maketecheasier.com/wp-content/uploads/2009/08/bashprompts-4.jpg)

```bash
PS1="\[\033[35m\]\t\[\033[m\]-\[\033[36m\]\u\[\033[m\]@\[\033[32m\]\h:\[\033[33;1m\]\w\[\033[m\]\$ "
```

# 五、显示完整路径

这是一个简短的提示符，第一行显示完整路径，第二行显示当前用户：

![prompt](http://mcdn.maketecheasier.com/wp-content/uploads/2009/08/bashprompts-5.jpg)

```bash
PS1="[\[\033[32m\]\w]\[\033[0m\]\n\[\033[1;36m\]\u\[\033[1;33m\]-> \[\033[0m\]"
```

# 六、显示后台任务数量

这个提示符的第一行还是显示用户名、主机名以及完整路径。特色在第二行，不仅显示历史记录数量，还会显示后台作业的数量：

![prompt](http://mcdn.maketecheasier.com/wp-content/uploads/2009/08/bashprompts-61.jpg)


```bash
PS1='\[\e[1;32m\]\u@\H:\[\e[m\] \[\e[1;37m\]\w\[\e[m\]\n\[\e[1;33m\]hist:\! \[\e[0;33m\] \[\e[1;31m\]jobs:\j \$\[\e[m\] '
```

# 七、显示目录信息

这是一个看起来很酷的设计：第一行显示用户名、主机名、后台作业数量、以及日期时间。第二行显示当前路径、文件数量以及所用空间：

![prompt](http://mcdn.maketecheasier.com/wp-content/uploads/2009/08/bashprompts-7.jpg)

```bash
PS1="\n\[\e[30;1m\]\[\016\]l\[\017\](\[\e[34;1m\]\u@\h\[\e[30;1m\])-(\[\e[34;1m\]\j\[\e[30;1m\])-(\[\e[34;1m\]\@ \d\[\e[30;1m\])->\[\e[30;1m\]\n\[\016\]m\[\017\]-(\[\[\e[32;1m\]\w\[\e[30;1m\])-(\[\e[32;1m\]\$(/bin/ls -1 | /usr/bin/wc -l | /bin/sed 's: ::g') files, \$(/bin/ls -lah | /bin/grep -m 1 total | /bin/sed 's/total //')b\[\e[30;1m\])--> \[\e[0m\]"
```

# 八、作者所用的提示符

这是本文原作者自己的提示符，这是基于 #7 修改的：第一行显示用户名、后台作业数、当前路径；第二行显示历史记录数。

![prompt](http://mcdn.maketecheasier.com/wp-content/uploads/2009/08/bashprompts-8.jpg)

```bash
PS1="\n\[\e[32;1m\](\[\e[37;1m\]\u\[\e[32;1m\])-(\[\e[37;1m\]jobs:\j\[\e[32;1m\])-(\[\e[37;1m\]\w\[\e[32;1m\])\n(\[\[\e[37;1m\]! \!\[\e[32;1m\])-> \[\e[0m\]"
```

# 九、我的提示符

我的提示符非常简洁，就是 PS1='$ '。我大概就是原作者说的不在乎提示符的那类人，呵呵。通常命令返回的结果并不多，如果提示符过于复杂，屏幕上提示符的数量反而比真正的命令还多（多行提示符尤甚）。我的机子就我一个人用，像用户名、主机名等信息并不需要强调，真的想知道就临时执行一下`whoami`。不知道大家都搜罗了什么实用的提示符，来一起分享一下吧。
