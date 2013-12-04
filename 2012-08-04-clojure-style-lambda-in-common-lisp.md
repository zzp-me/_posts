---
layout: article
title: 在Common Lisp中实现Clojure的fn
date: 2012-08-04 19:39
category: Lisp
excerpt:
  Clojure中Lambda支持递归，所以我要在Common Lisp中也实现这样的功能！
---

一年前，我写了一篇在Common Lisp中实现匿名递归函数的[文章]({% post_url 2011-08-05-recursive-lambda %})，提到 Emacs Lisp、Scheme 和 Common Lisp 中默认都没提供定义可递归的 lambda 函数的方法。并在文章里提供了我自己实现的 Emacs Lisp 版本和 Common Lisp 版本。在那之后，我学习了 Clojure，发现 Clojure 中的 fn 在定义 lambda 函数的同时还允许给它取一个临时的名字，这样就能在函数体中递归地调用自己了，比如下面用来临时求第12个斐波那契数的匿名函数：

```clojure
((fn fibonacci [n]
   (if (<= 2 n)
     (+ (fibonacci (- n 1))
        (fibonacci (- n 2)))
     1))
 12)
```

这个方法比我之前实现的要高明的多！我的方法会额外“霸占”一个名字“this”来代表自己，这样很容易有命名冲突的问题。但像 fn 这样，名字由开发者自己提供，就能避免这样的问题。因此，我开始琢磨怎么把 fn 迁移到 Common Lisp 中。

有了之前开发的经验，这一次实现起来顺手多了。观察 fn 的语法，与 lambda 相比它多了一个可选的名字。所以，当函数名未提供时和 lambda 无区别：

```common-lisp
(defmacro fn (&rest body)
  (if (listp (car body))
    `(lambda ,@body)
    ))
```

然后是 else 块，这部分和之前文章里介绍的一样，都是需要让返回的匿名函数能识别一个额外的函数名，并且那个函数名指向函数本身。区别仅是之前 hard code 成 this，而这回名字由开发者指定。所以，也是用“局部函数”来实现，即 Common Lisp 中的 labels，或 Emacs Lisp 中的 flet：

```common-lisp
(defmacro fn (&rest body)
  (if (listp (car body))
    `(lambda ,@body)
    `(lambda (&rest args)
       (labels (,body)
         (apply (function ,(car body)) args)))))
```

来写个函数测试一下。比如输出一棵树的所有子节点：

```common-lisp
(funcall
  (fn dump-list (o)
    (if (consp o)
      (dolist (item o)
        (dump-list item))
      (format t "~A~%" o)))
  '(1 (2 (3 (4) 5) 6) 7 (8 9)))
```

在 clisp 中执行结果如下：

```common-lisp
λ clisp -q
[1]> (defmacro fn (&rest body)
  (if (listp (car body))
    `(lambda ,@body)
    `(lambda (&rest args)
       (labels (,body)
         (apply (function ,(car body)) args)))))
FN
[2]> (funcall
  (fn dump-list (o)
    (if (consp o)
      (dolist (item o)
        (dump-list item))
      (format t "~A~%" o)))
  '(1 (2 (3 (4) 5) 6) 7 (8 9)))
1
2
3
4
5
6
7
8
9
NIL
[3]>
```
