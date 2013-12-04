---
layout: article
title: 10种编程语言实现Y组合子 
date: 2013-09-05 10:28:39
categories: [Lisp, Lambda演算]
excerpt:
  Y组合子是Lambda演算的一部分，也是函数式编程的理论基础。
  它是一种方法/技巧，在没有赋值语句的前提下定义递归的匿名函数。
  即仅仅通过Lambda表达式这个最基本的“原子”实现循环/迭代。
  颇有道生一、一生二、二生三、三生万物的感觉。
---

# Y-Combinator

转眼又过了一年，好像每年这个时候我都要和匿名递归函数较劲。今年我掌握了更多函数式编程的理论知识，了解到“匿名递归函数”已经有相应的研究结果：Y组合子（Y-Combinator）。

Y组合子是Lambda演算的一部分，也是函数式编程的理论基础。它是一种方法/技巧，在没有赋值语句的前提下定义递归的匿名函数。即仅仅通过Lambda表达式这个最基本的“原子”实现循环/迭代，颇有道生一、一生二、二生三、三生万物的感觉。


## 从递归的阶乘函数开始

先不考虑效率等其他因素，写一个最简单的递归阶乘函数。此处采用Scheme，你可以选择自己熟悉的编程语言跟着我一步一步实现Y-Combinator版的阶乘函数。

```scheme
(define (fab n)
  (if (zero? n)
    1
    (* n (fab (- n 1)))))
```

Scheme中`(define (fn-name))`是`(define fn-name (lambda))`的简写，就像JS中，`function foo() {}`等价于`var foo = function() {}`。把上面的定义展开成Lambda的定义：

```scheme
(define fab
  (lambda (n)
    (if (zero? n)
      1
      (* n (fab (- n 1))))))
```

## 绑定函数名

想要递归地调用一个函数，就必须给这个函数取一个名字。匿名函数想要实现递归，就得取一个临时的名字。所谓临时，指这个名字只在此函数体内有效，函数执行完成后，这个名字就伴随函数一起消失。为解决这个问题，[第一篇文章]({% post_url 2011-08-05-recursive-lambda %})中强制规定匿名函数有一个隐藏的名字this指向自己，这导致this这个变量名被强行占用，并不优雅，因此[第二篇文章]({% post_url 2012-08-04-clojure-style-lambda-in-common-lisp %})借鉴Clojure的方法，允许自定义一个名字。

但在lambda演算中，只有最普通的lambda，没有赋值语句，如何绑定一个名字呢？答案是使用lambda的参数列表！

```scheme
(lambda (fab)
  (lambda (n)
    (if (zero? n)
      1
      (* n (fab (- n 1))))))
```

## 生成阶乘函数的函数

虽然通过参数列表，即使用闭包技术给匿名函数取了一个名字，但此函数并不是我们想要的阶乘函数，而是阶乘函数的元函数（meta-fab），即生成阶乘函数的函数。因此需要执行这个元函数，获得想要的阶乘函数：

```scheme
((lambda (fab)
   (lambda (n)
     (if (zero? n)
       1
       (* n (fab (- n 1))))))
 xxx)
```

此时又出现另一个问题：实参xxx，即形参fab该取什么值？从定义来看，fab就是函数自身，既然是“自身”，首先想到的就是复制一份一模一样的代码：

```scheme
((lambda (fab)
   (lambda (n)
     (if (zero? n)
       1
       (* n (fab (- n 1))))))
 (lambda (fab)
   (lambda (n)
     (if (zero? n)
       1
       (* n (fab (- n 1)))))))
```

看起来已经把自己传递给了自己，但马上发现`(fab (- n 1))`会失败，因为此时的`fab`不是一个阶乘函数，而是一个包含阶乘函数的函数，即要获取包含在内部的函数，因此调用方式要改成`((meta-fab meta-fab) (- n 1))`：

```scheme
((lambda (meta-fab)
   (lambda (n)
     (if (zero? n)
       1
       (* n ((meta-fab meta-fab) (- n 1))))))
 (lambda (meta-fab)
   (lambda (n)
     (if (zero? n)
       1
       (* n ((meta-fab meta-fab) (- n 1)))))))
```

把名字改成meta-fab就能清晰地看出它是阶乘的元函数，而不是阶乘函数本身。

## 去除重复

以上代码已经实现了lambda的自我调用，但其中包含重复的代码，meta-fab即做函数又做参数，即`(meta meta)`：

```scheme
((lambda (meta)
   (meta meta))
 (lambda (meta-fab)
   (lambda (n)
     (if (zero? n)
       1
       (* n ((meta-fab meta-fab) (- n 1)))))))
```

## 提取阶乘函数

因为我们想要的是阶乘函数，所以用fab取代`(meta-fab meta-fab)`，方法同样是使用参数列表命名：

```scheme
((lambda (meta)
   (meta meta))
 (lambda (meta-fab)
   ((lambda (fab)
      (lambda (n)
        (if (zero? n)
          1
          (* n (fab (- n 1))))))
    (meta-fab meta-fab))))
```

这段代码还不能正常运行，因为Scheme以及其他主流的编程语言实现都采用“应用序”，即执行函数时先计算参数的值，因此`(meta-fab meta-fab)`原来是在求阶乘的过程中才被执行，现在提取出来后执行的时间被提前，于是陷入无限循环。解决方法是把它包装在lambda中（你学到了lambda的另一个用处：延迟执行）。

```scheme
((lambda (meta)
   (meta meta))
 (lambda (meta-fab)
   ((lambda (fab)
      (lambda (n)
        (if (zero? n)
          1
          (* n (fab (- n 1))))))
    (lambda args
      (apply (meta-fab meta-fab) args)))))
```

此时，代码中第4行到第8行正是最初定义的匿名递归阶乘函数，我们终于得到了阶乘函数本身！

## 形成模式

如果把其中的阶乘函数作为一个整体提取出来，那就是得到一种“模式”，即能生成任意匿名递归函数的模式：

```scheme
((lambda (fn)
   ((lambda (meta)
      (meta meta))
    (lambda (meta-fn)
      (fn
        (lambda args
          (apply (meta-fn meta-fn) args))))))
 (lambda (fab)
   (lambda (n)
     (if (zero? n)
       1
       (* n (fab (- n 1)))))))
```

Lambda演算中称这个模式为Y组合子（Y-Combinator），即：

```scheme
(define (y-combinator fn)
  ((lambda (meta)
     (meta meta))
   (lambda (meta-fn)
     (fn
       (lambda args
         (apply (meta-fn meta-fn) args))))))
```

有了Y组合子，我们就能定义任意的匿名递归函数。前文中定义的是递归求阶乘，再定义一个递归求斐波那契数：

```scheme
(y-combinator
  (lambda (fib)
    (lambda (n)
      (if (< n 3)
        1
      (+ (fib (- n 1))
         (fib (- n 2)))))))
```

# 10种实现

下面用10种不同的编程语言实现Y组合子，以及Y版的递归阶乘函数。实际开发中可能不会用上这样的技巧，但这些代码分别展示了这10种语言的诸多语法特性，能帮助你了解如何在这些语言中实现以下功能：

1. 如何定义匿名函数；
1. 如何就地调用一个匿名函数；
1. 如何将函数作为参数传递给其他函数；
1. 如何定义参数数目不定的函数；
1. 如何把函数作为值返回；
1. 如何将数组里的元素平坦开来传递给函数；
1. 三元表达式的使用方法。

这十种编程语言，有Python、PHP、Perl、Ruby等大家耳熟能详的脚本语言，估计最让大家惊讶的应该是其中有Java！

## Scheme

我始终觉得Scheme版是这么多种实现中最优雅的！它没有“刻意”的简洁，读起来很自然。

{% gist 6445345 y-combinator.scm %}

## Clojure

其实Clojure不需要借助Y-Combinator就能实现匿名递归函数，它的lambda——fn——支持传递一个函数名，为这个临时函数命名。也许Clojure的fn不应该叫匿名函数，应该叫临时函数更贴切。

同样是Lisp，Clojure版本比Scheme版本更简短，却让我感觉是一种刻意的简洁。我喜欢用fn取代lambda，但用稀奇古怪的符号来缩减代码量会让代码的可读性变差（我最近好像变得不太喜欢用符号，哈哈）。

{% gist 6445345 y-combinator.clj %}

## Common Lisp

Common Lisp版和Scheme版其实差不多，只不过Common Lisp属于Lisp-2，即函数命名空间与变量命名空间不同，因此调用匿名函数时需要额外的funcall。我个人不喜欢这个额外的调用，觉得它是冗余信息，位置信息已经包含了角色信息，就像命令行的第一个参数永远是命令。

{% gist 6445345 y-combinator.lisp %}

## Ruby

Ruby从Lisp那儿借鉴了许多，包括它的缺点。和Common Lisp一样，Ruby中执行一个匿名函数也需要额外的“.call”，或者使用中括号“`[]`”而不是和普通函数一样的小括号“`()`”，总之在Ruby中匿名函数与普通函数不一样！还有繁杂的符号也影响我在Ruby中使用匿名函数的心情，因此我会把Ruby看作语法更灵活、更简洁的Java，而不会考虑写函数式风格的代码。

{% gist 6445345 y-combinator.rb %}

## Python

Python中匿名函数的使用方式与普通函数一样，就这段代码而言，Python之于Ruby就像Scheme之于Common Lisp。但Python对Lambda的支持简直弱爆了，函数体只允许有一条语句！我决定我的工具箱中用Python取代C语言，虽然Python对匿名函数的支持只比C语言好一点点。

{% gist 6445345 y-combinator.py %}

## Perl

我个人对Perl函数不能声明参数的抱怨更甚于繁杂的符号！

{% gist 6445345 y-combinator.pl %}

假设Perl能像其他语言一样声明参数列表，代码会更简洁直观：

```perl
sub y_combinator($f) {
  sub($u) { $u->($u); }->(sub($x) {
    $f->(sub { $x->($x)->(@_); });
  });
}

print y_combinator(sub($fab) {
  sub($n) { $n < 2? 1: $n * $fab->($n - 1); };
})->(10);
```

## JavaScript

JavaScript无疑是脚本语言中最流行的！但冗长的function、return等关键字总是刺痛我的神经：

{% gist 6445345 y-combinator.pl %}

## Lua

Lua和JavaScript有相同的毛病，最让我意外的是它没有三元运算符！不过没有使用花括号让代码看起来清爽不少~

{% gist 6445345 y-combinator.lua %}

## PHP

PHP也是JavaScript的难兄难弟，function、return……

此外，PHP版本是脚本语言中符号（$、_、()、{}）用的最多的！是的，比Perl还多。

{% gist 6445345 y-combinator.pl %}

## Java

最后，Java登场。我说的不是Java 8，即不是用Lambda表达式，而是匿名类！匿名函数的意义是把代码块作为参数传递，这正是匿名类所做得事情。

{% gist 6445345 YCombinator.java %}
