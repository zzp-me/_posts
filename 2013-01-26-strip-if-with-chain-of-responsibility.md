---
layout: article
title: 消灭分支语句之责任链模式
date: 2013-01-26 22:47
category: 设计模式
excerpt:
  许多代码都会滥用if else和switch语句，
  本文介绍如何避免写大量的if else，
  并提高代码的可维护性和灵活性。
---

分支语句是所有编程语言的基本元素，比如Java语言中的if else和switch语句，它们提供一种能力允许程序根据一些条件动态地选择执行某些代码块。这种动态性给程序带来了很多的灵活性！

正因为if else如此方便如此灵活，很多代码中它都会被滥用，就像下面这样让人崩溃的、嵌套的、成堆的分支语句：

{% highlight java %}
if (context.equals("tutorial-room")) {
    if (pageNumber == 1) {
        if (input.equals("2")) {
            // go next
            // output step-2 prompt
        } else {
            // warning xxx
            // output step-1 prompt
        }
    } else if (pageNumber == 2) {
        if (state == State.QUITING) {
            if (input.equals("y")) {
                // xxx
            } else {
                // restore
            }
        } else if (input.equals("22")) {
            // put chess
        } else if (input.equals("q")) {
            // quiting?
        } else {
            // unknow instruct
        }
    } else ...
} else if (context.equals("newbie-room")) {
    ...
} else if (context.equals("easy-room")) {
    ...
} else if (context.equals("normal-room")) {
    ...
} else ...
{% endhighlight %}

本文会讨论一些程序设计的方法，把诸如上述的混乱代码重构成更清晰更优雅的代码。注：文中的代码皆为Java代码片段，仅使用标准JDK的类库。

# 问题

说上述代码结构让人崩溃，我们得有理有据。

首先，它的可读性不好。这里说的可读性不好并非指变量名命名不规范、花括号风格不一致、对齐不统一等问题，而是指代码是否方便理解。比如：

{% highlight java %}
if (cash < price) {
    // block A
} else if (onSale) {
    // block B
} else ... block C
{% endhighlight %}

这段代码先检查用户的现金是否足够支付当前货物的价格，如果余额不足则执行代码块A，否则再查看当前货物是否有促销活动，有就执行代码块B。其中代码块B咋眼看只有if (onSale)这一个条件，但因为它处于else块中，所以还隐含了(cash &gt;= price)这一条件。在代码规模不是很大的时候，这样的隐含条件影响可能不大，但如果有很多个else条件并且里面同时还嵌套着很深的分支结构，当你看到最深层的代码时，你是否还确信自己能清楚地记得所有的前提条件？

其次，它的维护性不好。比如在上面代码中加入会员机制，会员在购买商品时有积分，那相应的积分模块调用代码要同时出现在block B和block C中。如果之后会员又分了多个等级，那这段代码很快就成了庞然大物，任何的修改都会牵一发而动全身！

# 查表法

根据分支语句的特点，它可用于根据不同的输入返回特定的输出。比如《如此理解面向对象》一文中要根据系统名字，输出不同的提示语：

{% highlight java %}
String osName = System.getProperty("os.name");
if (osName.equals("SunOS")) {
    System.out.println("This is a UNIX box and therefore good.");
} else if (osName.equals("Linux")) {
    System.out.println("This is a Linux box and good as well.");
} else if (osName.equals("Windows NT")) {
    System.out.println("This is a Windows box and therefore bad.");
} else {
    System.out.println("Unknow box.");
}
{% endhighlight %}

我们暂且成这类分支为“数据型分支”。它犹如数学中的映射（Mapping），每一组特定的输入数据对应一组唯一的输出数据。因此，在输入数据比较简单时（比如第一个例子，输入数据只有系统名字一项），可以使用 java.util.Map 或 java.util.Properties 把映射关系持久化到配置文件中，程序启动时再加载到内存：

{% highlight java %}
import java.io.FileInputStream;
import java.util.Properties;

public class Main {
    public static void main(String[] args) throws Exception {
        Properties options = new Properties();
        options.load(new FileInputStream("options.properties"));

        String osName = System.getProperty("os.name");
        String prompt = options.getProperty(osName);
        if (prompt == null) {
            prompt = "Unknow box.";
        }

        System.out.println(prompt);
    }
}
{% endhighlight %}

其中配置文件信息如下：

{% highlight ini %}
SunOS=This is a UNIX box and therefore good.
Linux=This is a Linux box and good as well.
Windows\ NT=This is a Windows box and therefore bad.
{% endhighlight %}

使用这种方法，能很方便地支持新的系统或修改现有系统的提示语，且无须修改程序。不过开发中真实的输入项远不止一个字符串，正如 [@jxqlove?](http://my.oschina.net/feichexia) 同学之前在 (http://www.oschina.net/code/snippet_111708_17599) 中提的：根据交易类型、支付方式等多个条件返回一个字符串。处理这种Key有多个元素构成的情况，解决方案的思想和单元素是一致的，只是把元数据移到了数据库中：

{% highlight sql %}
create table metadata (
  trade_type varchar(16), -- 交易类型，比如收入、支出等
  payment varchar(16), -- 支付类型，比如现金、信用卡等
  code varchar(4) -- 最终的返回值
);
insert into metadata values ('income', 'cash', '001');
insert into metadata values ('income', 'credit card', '002');
insert into metadata values ('income', 'alipay', '003');
insert into metadata values ('expense', 'cash', '101');
insert into metadata values ('expense', 'credit card', '102');
insert into metadata values ('expense', 'alipay', '103');
{% endhighlight %}

在应用程序这一端则需要动态地构造查询语句：

{% highlight java %}
public String queryStatement(Properties options) {
    StringBuilder query = new StringBuilder("select code from metadata");
    Enumeration names = options.propertyNames();
    for (int i = 0; names.hasMoreElements(); i++) {
        String key = names.nextElement().toString();
        String value = options.getProperty(key);
        query.append(i == 0? " where ": " and ");
        query.append(String.format("%s = '%s'", key, value));
    }
    return query.toString();
}
{% endhighlight %}

根据实际的情况，代码可能更复杂一些，比如value的内容需要转义等。这样设计的系统会非常灵活，比如输入端新增了一个选项，只需给metadata添加新的字段，并根据所有的合法值插入新的记录或更新现有记录，而代码无须修改。

这种持久化到数据库的方法适用于一对一的无规律映射，即不存在或者只有少量的映射存在多组key对应同一个value的情况。它和数据的规模无关，比如一个字典程序的数据同样适用这种方式，数据量虽然很大但并不稀疏。

与之相对的是稀疏的数据，比如有一项值域范围是[1，100]，其中1到50应对的值是无规律，从51到100的值全部是一个固定的常量（比如0）。这时候有一半的存储空间是浪费的，真心不如在代码里用 if (value &gt; 50) 来判断。下文会提供另一种方法处理这类问题。

# 类责任链模式

上面介绍的查表法把元数据从逻辑代码中剥离出来，避免因元数据（Metadata）变化导致修改程序。但从某种意义上来说，程序本该如此：程序本身只是逻辑的集合；元数据（辅助程序行为，诸如语言包文件）集中在配置文件里；待处理的数据来自外部输入（用户手工录入、本地文件、数据库等）。因此本节讨论分支语句更常用的方式：选择执行某段代码。

{% highlight java %}
if (optionA) {
    if (optionB) {
        doSomething1();
    } else {
        doSomething2();
    }
} else {
    doSomething3();
}
{% endhighlight %}

类似上面的代码，根据不同的输入选项或命令行参数等调用不同的方法来完成某些操作，而不是单纯的返回数据。因此，这些选项是为了确定现在这个request是谁的职责，而这正是“责任链模式”要解决的问题！本节的标题为“类责任链模式”，表示我的解决方案是类似“责任链模式”，并不严格和它保持一致，但核心思想是一致的：使多个对象都有机会处理请求。

因此，每个RequestHandler都需提供一个接口判断自己能否处理当前请求；如果能处理，则Client调用另一个执行的接口：

{% highlight java %}
public interface Handler {
    public boolean accept(Properties options);
    public void execute();
}
{% endhighlight %}

于是，上面的分支结构对应三个独立的Handler类：

{% highlight java %}
public class RequestHandler1 implements Handler {
    public boolean accept(Properties options) {
        return options.getProperty("A") != null
            &amp;&amp; options.getProperty("B") != null;
    }

    public void execute() {
        doSomething1();
    }
}

public class RequestHandler2 implements Handler {
    public boolean accept(Properties options) {
        return options.getProperty("A") != null
            &amp;&amp; options.getProperty("B") == null;
    }

    public void execute() {
        doSomething2();
    }
}

public class RequestHandler3 implements Handler {
    public boolean accept(Properties options) {
        return options.getProperty("A") == null;
    }

    public void execute() {
        doSomething3();
    }
}
{% endhighlight %}

接下来还需要一个额外的管理类负责这些类的实例化的请求的分发：

{% highlight java %}
import java.util.ServiceLoader;
import java.util.Iterator;

public class Manager {
    private static Arraylist;
    static {
        list = new Array();

        ServiceLoaderloader = ServiceLoader.load(Handler.class);
        Iteratorit = loader.iterator();
        while (it.hasNext()) {
            list.add(it.next());
        }
    }

    public static void process(Properties options) {
        for (Handler handler : list) {
            if (handler.accept(options)) {
                handler.execute();
            }
        }
    }
}
{% endhighlight %}

上面代码使用了服务加载功能自动实例化所有注册过的Handler子类，如果你还不了解它的原理，可查看相应的API文档。有了这些代码，已经万事具备！也许你已经发现，这样的设计和JDBC的接口不谋而合：Manager对应java.sql.DriverManager、Handler对应java.sql.Driver、RequestHandler这些类则对应数据库厂商自己实现的驱动程序。

基于这样的框架，它的代码总量也许比原来的要多，但你不再需要在一堆if else中仔细推敲代码执行的前提条件，所有的前提条件都在accept函数里；添加新的功能所要做的仅需实现一个新的类，无须修改现有代码，符合开闭原则。

# 总结

本文中介绍了两种方法在我的实际开发中运用很多。比如昨天分享的“微信版开窗游戏”就是用“类责任链模式”结合“状态模式”实现的（不过它不是用Java写的）。如果你有其他方法来处理上述问题，欢迎留言交流。感想你耐心地读完全文！

PS：其实消灭分支语句的方法还有很多，也许可以继续写一个系列～嘿嘿。
