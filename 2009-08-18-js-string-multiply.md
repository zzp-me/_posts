---
layout: article
title: JavaScript中的字符串乘法
date: 2009-08-18 15:21
category: JavaScript
excerpt:
  为JavaScript提供类似Ruby中字符串乘法的功能。
---

原文地址：[http://www.davidflanagan.com/2009/08/string-multipli.html](http://www.davidflanagan.com/2009/08/string-multipli.html)

作者简介：David Flanagan 是一个醉心于Java写作的计算机程序员，他的大部分时间都致力于编写Java相关图书。David 在麻省理工学院获得了计算机科学于工程学位。他生活在地处西雅图和温哥华之间的美国太平洋西北海岸。他在O'Reilly出版的畅销书有《Java in a Nutshell》、《Java Foundation Classes in a Nutshell》、《Java Enterprise in a Nutshell》、《JavaScript: The Definitive Guide》、《JavaScript Pocket Reference》以及《The Ruby Programming Language》等。

# 原文

In Ruby, the `"*"` operator used with a string on the left and a number on the right does string repetition. `"Ruby"*2` evaluates to "RubyRuby", for example. This is only occasionally useful (when creating lines of hyphens for ASCII tables, for example) but it seems kind of neat. And it sure beats having to write a loop and concatenate n copies of a string one at a time--that just seems really inefficient.

I just realized that there is a clever way to implement string multiplication in JavaScript:

```javascript
String.prototype.times = function(n) {
    return Array.prototype.join.call({length:n+1}, this);
};
"js".times(5) // => "jsjsjsjsjs"
```

This method takes advantage of the behavior of the Array.join() method for arrays that have undefined elements. But it doesn't even bother creating an array with n+1 undefined elements. It fakes it out using and object with a length property and relies on the fact that Array.prototype.join() is defined generically. Because this object isn't an array, we can't invoke join() directly, but have to go through the prototype and use call(). Here's a simpler version that might be just as efficient:

```javascript
String.prototype.times = function(n) { return (new Array(n+1)).join(this);};
```

When you call the Array() constructor with a single numeric argument, it just sets the length of the returned array, and doesn't actually create any elements for the array.

I've only tested these in Firefox. I'm assuming that either is more efficient than anything that involves an actual loop, but I haven't run any benchmarks.

# 解释

非逐句翻译，只解释一下原文大概的意思。

在Ruby中，“*”操作符用一个字符串作为左边参数，一个数字作为右边参数，来实现字符串重复。例如，"Ruby" * 2 的值为 "RubyRuby"。这仅在少数地方有用（例如，生成一张由连字符等ASCII 码字符构成的表格），但是非常简洁。而且好过写一个循环来连接n次字符串&mdash;&mdash;这样显得很没效率。

我刚刚发现在JavaScript中有个聪明的技巧来实现字符串的乘法：

> 代码请参见原文

这个方法是调用一个由元素全为“undefined”的数组的Array.join()行为。但是它并没有真正创建一个包含 n+1 个“undefined”元素的数组。它利用一个包含 length 属性的匿名对象，依靠 Array 对象的原型函数 join()。因为 “Object” 不是数组，不能直接调用 join()，因此不得不通过原型的 call() 来实现。下面给出一个同样效果的简单版本：

> 代码请参见原文

当我们调用 Array 的带一个参数的构造器时，仅仅是设置了数组的长度，实际上并没有创建数组的元素。

我仅在 Firefox 下对其做了测试，我估计它会比普通的循环更加有效，但我并没有进行基准测试。

# 我的评论

如果要考虑效率的话，对循环迭代稍作优化可能效率更高。比如下面这段递归调用，算法复杂度是O(log<sub>2</sub><sup>n</sup>)。在Google Chrome下测试结果是比 David 的方法执行更快，但不得不承认他的方法很优雅！

```javascript
String.prototype.times = function(n) {
    if ( n == 1 ) {
        return this;
    }
    var midRes = this.times(Math.floor(n/2));
    midRes += midRes;
    if ( n % 2 ) {
        midRes += this;
    }
    return midRes;
}
```

# 后记

David 采纳了我的建议，他又为我们写了一段非递归的版本。请参看他的博客原文：[http://www.davidflanagan.com/2009/08/good-algorithms.html](http://www.davidflanagan.com/2009/08/good-algorithms.html)
