---
layout: article
title: iKnowledge诞生记：知识vs经验
date: 2013-03-13 21:31:46
category: iKnowledge
excerpt:
  写了多年的博客，对博客的使用有了自己的方式。
  现有的免费空间或博客系统或多或少不符合我的风格。
  经过几天自学ExtJS，终于开发出自己的博客系统，
  实现了一些很想要的功能！
---

# 免费博客空间的尴尬史

我其实开过好几个博客：

1. 第一个是网易163 博客，在高中时赶时髦开的。那时候忙着高考，加上高中的生活很单调（没有一丝爆点），所以成了摆设。毕业后尝试更新过几篇，但首页对文章的预览功能总让我不满意（好吧，我承认我有强迫症，看见那长短不一的预览就纠结），把心一狠就关掉了。
1. 后来一位高中同学帮我开通了QQ 空间（那时QQ 空间还不能自己申请，需要其他已开通的用户帮忙开通），我想QQ 上人气也旺，可以遥相呼应、一石激起千层浪！可惜QQ 空间对文章大小作了很严格的限制，容不下我那么多废话，只能痛并快乐着。后来发生意外，腾讯突然说我的QQ 号是靓号，要求续费才能继续使用。苦于连换了三张手机卡都不成功，后来发现那些手机号也被移动和联通收回了，那可是我花钱买的号码呀！受不了这些霸王条约，再次狠心把帐号仍掉。
1. 第三回转到[百度空间](http://hi.baidu.com/redraiment)，上大一时开的。那时我觉得百度的产品都不会很激进地更新，这比较符合我的口味。尤其是能直接粘贴带HTML 格式的文本，写ACM 等解题报告感觉很方便。结果百度空间改版后对文章的大小也作了限制，贴的代码格式也经常被打乱格式，于是再次考虑搬家，可找半天也没找到能关闭空间的按钮，感觉很无奈...
1. 第四回搬家到[CSDN](http://blog.csdn.net/redraiment)，那是在08年的时候。我想在国内CSDN 算得上专业的程序员站点了，博客系统自然也应该有“程序员特色”吧。可它偏偏很不稳定，我刚夸完它就发现文章不能发表了，看了官方博客才知道“评论系统”也已停用半个多月了。这让我感觉不是很好，觉得他们有点不负责，拿用户当小白鼠。当然，CSDN 的新功能倒是不断添加，也自带插入代码的功能，用起来很方便。而且在那里也有很多志同道合的朋友会关注我的空间，让我很有存在感。只可惜Bug 修复效率不高。
1. 第五回我选择了[Google Blogger](http://redraiment.blogspot.com/)，在09年的时候。那段时间我是Google 产品的重度用户：浏览器主页是iGoogle、用Gmail 收发信件、在Reader 上吸收大师们的观点、用Docs 在线编写和共享我的文档、用Notebook 随时记录稍瞬即逝的灵感。那时候他们表现得如此稳定，比如用Gmail 发送大附件时没遇到苦苦等待十几分钟后却返回一个发送超时的尴尬，把事情交给它们处理我觉得很安心。“漂泊”这么久，我本希望Blogger 能让我安心地住下来，遗憾的是不到三天Blogger 就被和谐了。

古人有孟母三迁，我的空间都搬五回家了，始终没找到称心如意的博客空间。Blogger被墙后我不得不回到CSDN，结果还混上了社区专家。

# 知识vs经验

从第一个163博客到现在，已经快有8个年头了。对于我而言，博客也从最初的吐槽工具变成了知识库。

说到知识（Knowledge），很多人都会把它与经验（experience）划等号，在我看来它们是不一样的：知识是抽象的，而经验是具体的。人们先在实践中获得经验，然后结合以前获得的经验在脑海中进一步分析和挖掘，融入自己的知识体系中。形成知识后就可以举一反三，去预测其他同类情况的结果。

比如幼儿园的小朋友学加法，老师举了1+1=2、2+2=4等一系列具体的例子，如果小朋友们紧紧在脑海中强记住这些等式，那他们知识掌握了几条独立的经验，这些经验就像一张Hash表一样，告诉小朋友1+1的结果是2、2+2的结果是4，一旦遇到新的加法问题（如1+2）就无从下手了；在将来小朋友们接触到了更多的数字和等式后，脑海中分析和挖掘这些等式的异同，最后提炼出整数和加法等抽象的概念，形成了知识，就能解决所有加法问题。

因此，我认为经验是通用的，经验只是对某个现象的描述，比如被针扎到会痛、1+1等于2；而知识是属于个人的，由于每个人不同的世界观，以及脑海中积累不同数量的经验，导致同一条经验不同的人会有不同的归类方式。比如提到Bash，有些人会把它与Linux划等号，因为他们只知道Linux下需要用bash；而另一些人知道它是一种Shell，因为他们知道还有csh、tcsh、zsh等其他shell，并且它们还能用于Linux意外的系统中。

# iKnowledge诞生

每当我们学到新东西就能写一篇博文来记录，但这些记录还仅仅是一条条独立的经验，需要按照某种方式去组织它们，这样才能融入我们的知识体系。

纵观现有的博客系统，对于博文的组织无非是时间的倒序，或给文章贴标签。我发现很多人压根不会回头去看自己写的博客，即使博文排版混乱、文章主题不鲜明、错别字无数也都无所谓。

对我而言，博客不仅用于记录我的知识，方便以后查阅；更重要的是它要帮助我梳理已掌握的经验以形成知识体系，反而时间的先后并不重要。因此，我需要一套树状的结构来展示主题与主题之间的层次关系，而不是一堆摊平的标签。

由于上面种种原因，经过几天自学ExtJS，我终于开发出自己的博客系统，实现以层次结构组织的标签系统，并取名叫iKnowledge，以强调这套系统帮助大家积累知识，而不仅仅是记录经验。

{% img 1.png %}

# iKnowledge的特色

## iKnowledge是一个纯静态网站

自从Paul Graham 开发了全世界第一个Web 应用程序Viaweb 之后，应用程序Web化变得一发不可收拾，甚至出现了Web OS 要完全取代桌面应用。主流博客系统中，既有前台展示还有后台管理，能在线发文章、编辑、删除等，什么都做结果什么都没做好。比如上面提的展示形式一成不变、在线编辑器功能单一、误删误改恢复困难等。正因如此，iKnowledge只做一件事——展示。文章编辑、版本控制等工作则交给专业的编辑器和版本控制工具来完成！

现有的博客系统几乎清一色使用数据库保存文章内容，通常还只使用一个字段。且不说性能如何，就使用上也是非常的不方便。诸如批量统计文章行数的功能，不得不写一个程序来解析字段里的字符串；而处理纯文本文件却有一堆现成的命令行工具。再说博客系统的数据通常是一次写入、偶尔编辑、多次读取，即频繁地查询。为了提高性能，又往往对动态页面做缓存处理生成静态页面。既然到头来都是生成静态页面，前面那些琐碎的事情简直是脱裤子放屁多此一举。

根据上面的需求描述，说白了iKnowledge就是左侧树状菜单、右侧选项卡页面的静态页面。为了快速实现这套系统，我选择站在巨人的肩膀上，很多功能使用现成的工具或库。

## iKnowledge基于ExtJS

的确，它很臃肿。所以，非常感谢你耐心地浏览这个被全国99&#37电脑击败的龟速博客。

JavaScript被称为Web时代的汇编语言，虽然它相较其他高级语言已经灵活很多！Web程序员在经历这么多年的积累和沉淀后渐渐挖掘出一些Web编程的模式，于是有了jQuery等JS库来简化开发，这一飞跃犹如把汇编码中成堆的JNZ、JMP等语句组织成了if、for等C语言控制语句。但这还不够，开发者还得自己定义HTML、设计CSS样式等。于是ExtJS来了，犹如当年Delphi带来RAD的春风一般。它带来的不仅仅是漂亮的界面，还有开发过程的改进：程序员不再需要关心如何定义HTML、怎么写跨浏览器的CSS，只需确定数据展示的形式再调用相应的组件即可。

我有幸从JavaScript一直实践到ExtJS，所以我能体会每一次改进带来的喜悦！回到iKnowledge，正如上面说的，它只是一颗树加一个选项卡。如果从零开始实现，工作量无法想象；即使使用插件丰富的jQuery工作量也不小，何况jQuery的插件往往各自为政，相互协作总会出这样或那样的问题；而使用ExtJS，一切都这么简单！

前端相关的代码都在app这个目录中，整个工程只有32k。

## 文件目录结构即标签层次结构

提到树状结构，最常见的莫过于系统的文件目录结构，为什么我们不好好利用呢？iKnowledge就是利用文件目录结构生成左侧分类标签层次结构，转换工作由Rhino（纯Java实现的JavaScript引擎）自动完成。由此可见，iKnowledge是重度依赖JavaScript，无论前端还是后端，无不享受着它的强大与便利。

为了方便在命令行下切换目录，目录名统一用英文，但前端展示我又希望用中文，因此在生成的过程中做了额外的翻译工作。

## 代码高亮

作为程序员的博客，少不了在博文中分享代码，语法高亮的代码有助于读者理解代码逻辑。iKnowledge选用了一个原生的JavaScript插件——[Rainbow](http://rainbowco.de/)，所谓原生的指不依赖注入jQuery等JavaScript库。

这个库很轻量（1.4kb），而且扩展也很方便，但在使用中发现它在处理较长的代码段时偶尔会堆栈溢出，作者表示无法重现问题。

## Q: 为什么不能评论？

在这个Web 2.x时代，一个只能看的网站简直一无是处！毕竟提供评论等互动的渠道可以和读者交流，相互学习等。但愿望总是好的，以我CSDN上的博客为例，近70万的访问量咋看很厉害的样子，但总共只有680条评论，除去大部分单纯叫好、求帮的，真正有价值的评论没几条！

此外，要处理评论对服务器也有更多的要求；而放弃评论就只需一台能处理静态页面的Web服务器。这样我就能把iKnowledge托管在Github Pages上。
