---
layout: article
title: 汝当分离软件与信仰
date: 2009-06-21 17:41
category: 我的世界观
excerpt:
  “宗教信仰”在软件开发中有着多种表现形式——非要坚持某种设计方法，
  笃信特定的布局或注释风格，激励避免全局数据。
  不管是哪种情况，都是不合适的。
---

摘自《代码大全2》，有删减，希望和大家共勉。

“宗教信仰”在软件开发中有着多种表现形式——非要坚持某种设计方法，笃信特定的布局或注释风格，激励避免全局数据。不管是哪种情况，都是不合适的。

# 软件先知

糟糕的是，一些专业优秀人员往往更容易偏执。革新方法需要公开，才能让别人尝试。尝试这些方法后才能充分证实或反驳之。研究结果向实践者的传播称为“技术转移”，对于推动软件开发的时间水平有重要作用。然而，传播新方法和兜售狗皮膏药是不同的概念。对于后者，兜售者只是不懈地让你确信他们的方法如何灵验、如何放至四海而皆准。他们要你忘掉所学的一切，因为新方法如此伟大，能在任何方面将你的效率提高100%。

不要盲目跟风，而应使用一种混合的方法。可用激动人心的最新方法做做试验，但仍扎根于传统的可靠的方法。

# 折中主义

要对编程问题找出最有效的解决方案时，盲目迷信某种方法只会缩小你的选择余地。要是软件开发是确定的精确过程，就能按固定的套路解决问题；但软件开发并非确定过程，是需要逐步细化的，因而生硬的过程是不合适的，很难指望会成功。例如，设计中有时自顶向下分解法行得通，有时面向对象的方法、自底向上综合法或数据结构法会更好些。你应有意识地尝试几种途径，明知这些方法中有的可能成功、有的可能失败，但只有通过实践才能知道哪些好使。你必须采取折中的态度。

一味矜持某种方法，还会将问题强行塞进其解决方案中。如果没有充分了解问题就定下解决方法，说明你还不够成熟。受限于所坚持的思路，你很可能与最有效的方法失之交臂。

刚开始接触人呢和新方法时，人们会感到不自在。而建议你不要编程“信仰”，并不是说用新方法解决问题遇到麻烦时就马上停用新方法。对新方法有个合适的定位，但同样也要对老方法有合适定位。

对于书本暂时的技术，和其他途径所给出的技术一样，折中主义都是一种有益的态度。这里对若干问题的讨论提供了许多可以相互替代的高级方法，但毕竟不能同时用。对于每个特定问题，你需要选择这种或者那种方法。你应该将这些方法视为工具箱中的工具，工作时自己判断挑选最好的工具。多数时候工具的选择关系不大，你可以选择老虎钳或者尖嘴钳。但有些场合，工具选择至关重要，故而要仔细做出取舍。工程学的规则之一就是权衡各种技术。如果早早将自己的选择限制在单一工具上，就无法做出权衡。

由于“工具箱”形象地比喻出抽象的折中思想，所以这个比喻很有用。设想你是机械师，你的伙伴 Simple Simon 老爱用老虎钳，加入他不愿使用其他种类的钳子，你可能会觉得这家伙真稀奇，因为他不会用给他配的那么多工具。软件开发中的道理也是一样。在高层会有几种可用的设计方法；在细化，可对布局方案、代码注释、变量命名、定义类接口和传递子程序参数方面有多种选择。

顽固的态度与软件创建中的可选择工具箱方法背道而驰，也不是创建高质量软件所应持的态度。

# 试验

折中主义和试验之间有着密切联系。试验应贯穿于整个开发过程，但固执会妨碍你这么做。要想有效地试验，应能基于试验结果改变思路；否则试验只会白白浪费时间。

软件开发中许多顽固的方法源于对错误的畏惧心理。“试图没有错误”是最大的错误。设计正好是仔细地规划小错误以避免大错误的过程。软件开发中的试验是为了验证某种方法是否可行——只要它解决了问题，就算成功了。

试验可以用在许多层次上，和折中主义一样。每一层上，当你要做出选择时，就要搞个相应的试验，看看哪个方法最佳。在架构设计层，试验可以是对软件结构的勾勒（使用三种不同方法）；在详细设计层，试验也许是对上层结构指示遵循（对应使用三种不同的低层设计方法）；在编程语言层，试验室能够编个简短的测试程序，以验证自己不太熟悉的语法特性。试验还包括调整代码，并测量其效率，看程序是否真变短变快了。在软件开发过程的整体层上，可能通过试验收集质量和效率数据，以了解正式检查是不是比走查找出的错误更多。

关键是对软件开发的各方面，你都应保持开放的心态。这样才能从开发过程和产品中有所收获。开放性的试验和顽固坚持某种方法可完全是两码事。