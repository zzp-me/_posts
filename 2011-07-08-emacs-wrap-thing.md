---
layout: article
title: 文本加引号的插件
date: 2011-07-08 20:50
category: Emacs
excerpt:
  编辑过程中经常遇到在文本两端添加内容的情况，例如添加双引号、添加XML标签等。
  开发一个Emacs插件来简化操作。
---

前几天[@刘鑫-MarchLiu](http://weibo.com/marchliu)在微博上发布了一个给文本加引号的插件：[http://weibo.com/1729408273/eDcC8e8w6aD](http://weibo.com/1729408273/eDcC8e8w6aD)。不过用起来有点小问题：

1. 两头都只能插入一个字符，因此不能用于添加 XML 标签；
1. 光标控制上有个 bug，每次执行后光标会往左移动一个字符。

我自己刚刚也实现了一下，允许用户输入两端符号，因此能支持XML等更复杂的功能

```common-lisp
(defun wrap-thing (thing)
  "Wrap the thing at point.
THING is a symbol which specifies the kind of syntactic entity you want.
Possibilities include `region', `symbol', `list', `sexp', `defun', `filename',
`url', `email', `word', `sentence', `whitespace', `line', `page' and so on."
  (interactive)
  (let ((range (if (eq thing 'region)
                   (cons (region-beginning) (region-end))
                 (bounds-of-thing-at-point thing)))
        (wrapper (cons (read-string "Left: ")
                       (read-string "Right: "))))
    (save-excursion
      (goto-char (cdr range))
      (insert (cdr wrapper))
      (goto-char (car range))
      (insert (car wrapper)))))

(defmacro make-wrap-for (&rest things)
  "A tool for define wrap-region, wrap-word etc."
  `(progn
     ,@(mapcar
        (lambda (e)
          `(defun ,(intern (concat "wrap-" (symbol-name e))) ()
             (interactive)
             (wrap-thing ',e (cons (read-string "Left: ")
                                   (read-string "Right: ")))))
        things)))

(make-wrap-for region symbol list sexp defun
               filename url email word sentence
               whitespace lint page)

(global-set-key (kbd "C-.") 'wrap-region)
```

把上面的代码放到你的 .emacs 文件中，就能用 C-. 来调用了。如果有需要，你还可以再将 wrap-word、wrap-sentence 等绑定到其他键。
