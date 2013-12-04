---
layout: article
title: Lisp中实现匿名递归函数
date: 2011-08-05 22:22
category: Lisp
excerpt:
  主流的 Lisp 实现（Common Lisp、Guile、Emacs Lisp 等）
  中默认都没提供定义匿名的递归函数的方法。
  不过 Lisp 一个特色就是你可以自己动手添加需要的语言特性！
  于是我就尝试着自己写一个宏来实现这个功能。
---

主流的 Lisp 实现（Common Lisp、Guile、Emacs Lisp 等）中默认都没提供定义匿名的递归函数的方法。上 Google 搜索了一下，看到不少人也都在抱怨。不过 Lisp 一个特色就是你可以自己动手添加需要的语言特性！于是我就尝试着自己写一个宏来实现这个功能。

用 Lisp recursive lambda 做关键词搜索，找到老外一篇 08 年的[文章](http://ewger.blogspot.com/2008/03/recursive-lambda-factorial-in-lisp.html)，里面提到一种用两个 Lambda 实现的递归方法。我将格式整理了一下：

```common-lisp
(funcall
 (lambda (fn n)
   (funcall fn n fn))
 (lambda (n this)
   (cond ((> n 0)
          (* n (funcall this (- n 1) this)))
         (t 1)))
 10)
```

方法就是额外添加一个参数，用于传递函数本身。个人觉得这个方法不够优雅，如果需要定义多个匿名的递归函数，冗余的代码也会相当多。不过它已经反映出实现匿名递归函数的本质方法：将匿名函数赋给一个临时变量。

我的想法是在定义 lambda 函数时使用 this 来代表自身。例如编写一个用递归实现的匿名阶乘函数：

```common-lisp
((lambda (n)
   (if (> n 1)
       (* n (this (1- n)))
     1))
 10)
```

根据上述期望的代码，目标是：

1. 定义一个能创建并返回匿名函数的宏；
1. 这个匿名函数要绑定到 this 符号（Symbol）上。

我一开始先尝试用 Emcas Lisp 来实现。实现第一个目标很简单，只要将宏的参数和 lambda 这个符号链接起来即可：

```common-lisp
(defmacro re-lambda (&rest cdr)
   (cons 'lambda cdr))
```

第二个目标可以使用 defalias。它能将符号绑定到函数空间里（setq 只能绑定到变量空间，因此通过 setq 绑定的函数只能用 funcall 来调用）：

```scheme
(defmacro re-lambda (&rest cdr)
  `(defalias 'this
     ,(cons 'lambda cdr)))
```

这样就拥有了符合上述要求，可以定义匿名递归函数的方法了。用这个宏来编写阶乘函数并执行：

```common-lisp
(funcall
 (re-lambda (n)
   (if (> n 1)
       (* n (this (1- n)))
     1))
 10)
; 执行结果：3628800
```

执行结果正确，实现的代码也很简短，看起来很不错的样子。可惜它有个不足之处：Emacs Lisp 中有 let 等函数来定义局部变量，却没用类似的方法来定义局部函数，而且也不支持闭包，所以 this 这个符号是所有匿名函数共用的。如果同时用 re-lambda 定义了两个匿名函数，前一个函数就会被覆盖：

```common-lisp
(setq factorial
 (re-lambda (n)
   (if (> n 1)
       (* n (this (1- n)))
     1)))

(funcall factorial 10) ; 执行结果：3628800

(setq sum
 (re-lambda (n)
   (if (> n 1)
       (+ n (this (1- n)))
     1)))

(funcall sum 10)       ; 执行结果：55
(funcall factorial 10) ; 执行结果：55
```

好在 Common Lisp 中提供的 labels 可以定义局部函数，有了它就不用担心命名空间被污染了：

```common-lisp
(defmacro re-lambda (&rest body)
  `(lambda (&rest args)
     (labels (,(cons 'this body))
        (apply #'this args))))
```

上面代码中，因为 this 是作为返回的匿名函数中的一个局部函数，因此不必担心命名冲突问题。你可能会想，为什么不用 re-lambda 代替 this？这样也能避免 this 和其他全局变量或函数冲突。即：

```common-lisp
((lambda (n)
   (if (> n 1)
       (* n (lambda (1- n)))
     1))
 10)
```

这看起来很不错：lambda 在不同的上下文里表现出不同的角色。而且实现的方法也很简单，只要将上述代码中的 this 替换成 re-lambda 即可。但这会引起另一个问题：不能定义嵌套匿名递归函数。例如，求 1 到 n 所有阶乘的和：

```common-lisp
(format t "~A~%"
  (funcall
    (re-lambda (n)
      (if (> n 1)
          (+ (this (1- n))
	     (funcall (re-lambda (n)
	                 (if (> n 1)
	                     (* (this (1- n)) n)
			   1))
		      n))
	1))
    10))
```

如果没用 this 的话，就很难分辨哪些 re-lambda 是定义函数，哪些是计算结果。

最后，还有一个遗憾：你可能也发现了，我给的期望结果里调用 lambda 都不需要 funcall。因为它是按 Scheme 的语法写的，而实现的代码是按 Common Lisp 的语法写的。就这方面来说，我觉得 Scheme 的语法更优雅些。
