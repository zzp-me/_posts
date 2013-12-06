---
layout: article
title: 相似行编辑模式
date: 2012-03-21 17:05
category: Emacs
excerpt:
  在日常开发的过程中，经常遇到要输入相似的信息，即大部分内容均一致仅小部分内容不同的信息。
  本文针对此类问题提出了一些解决方法，并不断改进使之更加快捷有效。
---

# 绪论

作为一个程序员，日常生活就是做软件开发。该过程需要编写大量的代码（code）、配置文件（Configuration file）、数据库原始信息（Database Meta Data）等。开发期间常常会遇到需要反复地输入一段相似的内容的情况。

对于编程而言，有重复的或者类似的代码，就意味着需要思考是否能提炼出一种抽象来重构代码；但对于编码意外的场景往往只能按部就班地输入。下文将列举三个常见的场景。

# 场景一：数据库脚本

软件产品上线，需要向DBA（数据库管理员）提供SQL语句来插入一些原始信息。假设系统中有一张如下表：

```sql
create table users (
  id int,
  name varchar(20),
  description varchar(255)
)
```

现在需要往这张表中插入5行原始记录：

```sql
insert into users (id, name, description) values (1, 'zhang zepeng', 'Hangzhou, Zhang Zepeng');
insert into users (id, name, description) values (2, 'chen liang', 'Hangzhou, Chen Liang');
insert into users (id, name, description) values (3, 'xin zengfeng', 'Hangzhou, Xin Zengfeng');
insert into users (id, name, description) values (4, 'wang wei', 'Hangzhou, Wang Wei');
insert into users (id, name, description) values (5, 'gao feng', 'Hangzhou, Gao Feng');
```

容易发现，这五行代码除了具体的名字不一样，其他信息完全相同。那些经常和数据库打交道的同学应该对这个场景非常熟悉。

# 场景二：课堂Menu 程序

在上一堂课上，邢老师不断改进菜单程序，最终变成可通过配置来改变程序行为的、非常灵活的、非常通用的程序。最终的配置文件格式如下：

    # index name description
    1 exit Exit_is_great
    2 help Help_is_great
    3 enquery Enquery_is_great

这个配置文件中的三行信息同样也是相似的。无独有偶，真实的开发环境往往会根据不同的运行环境提供不同的配置文件，这样文件里的信息大多也都是相似的（比如只有机器名不同而已）。

# 场景三：本文标题

再举一个更近的例子，本中的标题依次有“场景一”、“场景二”和“场景三”，下文还会有“解决方案一”、“解决方案二”、“解决方案三”等等。除了最后的序数不同，其他内容完全一样！这说明，不仅仅出现在程序开发中，日常的写作、笔记等也充斥着相似内容的问题。

# 解决方案一：从记事本开始

我观察了一下周围，碰到这样的问题时，有少数会很老实地打开记事本，很耐心地逐字输入。其中有不少也是程序员。

他们认为这种是临时的、一次性的工作，而且内容并不多（通常都是在10行以内），所以想其他解决方法可能好没有直接动手敲键盘来得快。

<dl>
  <dt>优点</dt>
  <dd>实施简单，无需借助任何特殊的工具，因此几乎没有学习成本，任何会使用计算机的人都能完成这个任务。</dd>
  <dt>缺点</dt>
  <dd>容易犯错；对未来类似的场景没有帮助；而且这个方法工作量非常大，对操作者要求有一定的耐心。</dd>
</dl>

# 解决方案二：C-c C-v

第一种方法无论对于当前的工作还是未来的工作，都没有抽象出可参考的模型。所以，在方案二中，首先考虑提高当前操作的效率。

例如场景一中，每行均有70个字符是一模一样的，真正不同的内容只有10来个。因此，多数人都会在完成第一行内容后，复制当前行（C-c），然后连续粘贴4次，最后逐行修改，填入需要的数据。

<dl>
  <dt>优点</dt>
  <dd>和第一种方法相比，重复的内容只需要输入一次；后面的内容都是通过复制粘贴得到，因此不容易出错；复制粘贴代价很低，所以可以处理比第一种方法更大量的数据；成本低廉，不需要借助特殊的工具，因此这也是多数人使用的方法。</dd>
  <dt>缺点</dt>
  <dd>如果你自己动手操作一下，你就会发现 ，在开始填入不同的内容之前，文件里已经布满了密密麻麻的内容，需要很频繁地在多个位置之间来回切换，容易遗漏。</dd>
</dl>

# 解决方案三：键盘宏

使用方法二可以避免多次输入重复的内容，已经省下不少功夫。但定位频繁、容易遗漏等缺点也非常显著。到目前为止，方法一、方法二中都使用记事本作为编辑器，但记事本只提供了一小部分基础的编辑功能，为了进一步提高效率，可以使用其他一些提供更丰富功能的优秀编辑器，例如下文要介绍的GNU Emacs（下文简称Emacs）为例。

为了避免方法二中的问题，最好能编辑的过程改成增量式的、交互式的。Emacs提供了一种叫做“键盘宏”的功能，可以将一组操作录制下来反复播放。比如处理场景一可以作如下操作录制一段宏，其中“#”后面为注释，粗体部分为输入的纯文本信息：

1. C-x ( # 开始录制
1. insert into users (id, name, description) values (
1. C-u C-x q # 进入递归编辑，等待用户输入
1. C-M-c # 结束递归编辑
1. , '
1. C-u C-x q
1. C-M-c
1. ', 'Hangzhoud,
1. C-u C-x q
1. C-M-c
1. ');<BR>
1. C-x ) #结 结束录制

录制完后，接着开始执行：

1. C-u 5 C-x e # 其中 C-x e 为播放宏，C-u 5表示重复执行5次
1. 1
1. C-M-c
1. zhang zepeng
1. C-M-c
1. Zhang Zepeng
1. C-M-c
1. …

<dl>
  <dt>优点</dt>
  <dd>和第二种方法一样，使用宏同样不需要重复输入；另外，宏是逐步得填入内容，光标无需四处游走，不会眼花缭乱。</dd>
  <dt>缺点</dt>
  <dd>录制过程比较麻烦，当待编辑内容较少时有点本末倒置；录制的宏不通用，不能被以后的用例使用。</dd>
</dl>

# 解决方案四：Emacs Lisp

使用Emacs宏已经能很有效地解决问题，但可惜不通用。仔细观察各个场景，它们的共同特征是：从第二行开始，都可通过替换第一行中某些关键词来得到。

抛开现有的工具，想象怎样一个工具才能很顺手地解决这个问题。我想我会这么使用：

1. 输入第一行内容；
1. 复制第一行内容；
1. 告诉工具要粘贴多少次；
1. 工具首先询问我有哪些关键字需要替换，我回答它一些“正则表达式”；
1. 工具开始粘帖，并且每行都提醒我给出之前那些正则表达式在当前行相应值。

纵观现有的编辑器，没发现有提供类似功能的工具。好在，诸如Emacs这样优秀的编辑器都允许扩展它的功能。我使用Emacs的扩展语言Emacs Lisp开发了下面这个小插件：

```common-lisp
(defun yank-with-regexp-replace (&optional arg)
  "Replace regexp while yanking.
With argument N, yank N times."
  (interactive "P")
  (let ((exp-list '())
        exp)
    (setq exp-list
          (do ((i 1 (1+ i)))
              ((equalp (setq exp (read-regexp (format "%d" i))) "")
               (reverse exp-list))
            (push exp exp-list)))
    (dotimes (_ (parse-number-arg arg))
      (let ((line (current-kill 0)))
        (insert
         (dolist (from exp-list line)
           (setq line
                 (replace-regexp-in-string
                  from
                  (read-string (format "Replace %s with: " from))
                  line))))))))

(global-set-key (kbd "C-c C-y") #'yank-with-regexp-replace)
```

将上面的代码添加到自己的“.emacs”文件里，这样就是通过按“C-c C-y”来使用这个功能。如果需要粘帖多行就使用C-u N前缀，比如场景一就使用C-u 4 C-c C-y来粘贴后面4行。

这里要介绍一个Emacs给我面的糖果：Emacs在做替换时能很智能地处理大小写问题。比如将“1, zhang zepeng, Zhang Zepeng”这一行中的“zhang zepeng”替换成”chen liang”，会得到“1, chen liang, Chen Liang”。之前三种方法都认为一行有三个关键词，使用这种方法一行只需要处理两个关键词即可！

<dl>
  <dt>优点</dt>
  <dd>更通用，在安装完上述插件后，就可以反复使用；更智能，诸如“zhang zepeng”和“Zhang Zepeng”这样仅大小写区别的字段只需输入一次即可；更灵活，Emacs的正则表达式替换可以使用“\,(expression)”来执行Emacs Lisp代码，借助这一强大的特性，该方法几乎能处理所有场景！</dd>
  <dt>缺点</dt>
  <dd>学习成本高，不仅需要你熟悉一种编辑器（比如本文的Emacs），还需要了解如何扩展它（比如本文的Emacs Lisp）；平台依赖性大，如果你使用EditPlus或其他编辑器，很遗憾上面的Emacs Lisp不能为你效劳。</dd>
</dl>

# 结论

至此，介绍了四种方法解决文章开头提出的问题。从第一种最原始的方法开始，不断挖掘方法中重复的部分，最后抽象出一种通用的解决方案，并使用Emacs Lisp开发出相应的工具，使得以后遇到此类问题时也能很高效地解决。

在解决这个问题的过程中，我们经过“改良方法”、“寻找工具”、“开发自己工具”三个步骤，对比各种方法的优缺点后可以发现：学习成本越低的方法越麻烦。比如前两种使用记事本的方法，操作非常犯错且容易出错；最后一种方法需要了解很多，甚至还要学一门冷僻的编程语言Lisp，但却可以一劳永逸。正验了那句话：如果一开始就怕麻烦，后面只会越来越麻烦！
