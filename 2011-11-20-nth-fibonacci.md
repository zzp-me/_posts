---
layout: article
title: 高效地计算第n个斐波那契数
date: 2011-11-20 11:00
category: 基础算法
excerpt:
  使用通项公式快速地求第N个斐波那契数
---

大家都知道费波那契数列的定义是：

<ol>
  <li><i>F</i> = 0</li>
  <li><i>F</i><sub>1</sub> = 1</li>
  <li><i>F</i><sub>n</sub> = <i>F</i><sub>n-1</sub> + <i>F</i><sub>n-2</sub></li>
</ol>

如果直接按照这个定义来计算效率是很低的。所有，为获得第 n 个数，一般都会采用迭代的方式计算整个 0-n 的斐波那契数列，在 n 比较小的时候（小于十万）效率还是很不错的（在我的机器上 Java 程序计算 F<sub>100000</sub>耗时 1s）。但如果仅仅是为了获得第 n 个数，这又显得太低效了。这种时候就适合使用通项公式来计算：

{% img 1.png %}

本来是想着用 Java 写一个，一想到又要定义两三个类代码会显得冗长，倒不如用 Clojure 来实现。好在 Clojure 里也没太多稀奇古怪的符号，即使你没接触过 Clojure，应该还是能看懂。

```clojure
(ns redraiment.math.fibonacci
  "Some utils for generate fibonacci number.")

(def ^{:private true :dynamic true} *base*
  "Base number for composite number.
As you know, it's complex when *base* = -1."
  5)

(defn- multiply
  "Returns the product of composite nums. Supports arbitrary precision.
(multiply) returns [1 0]."
  [& args]
  (reduce
   #(let [a1 (first %1)
          b1 (second %1)
          a2 (first %2)
          b2 (second %2)]
      [(+ (* a1 a2) (* b1 b2 *base*))
       (+ (* a1 b2) (* a2 b1))])
   (cons [1 0] args)))

(defn- pow
  "Returns a composite num whose value is c^n."
  [c n]
  (cond
   (== n 0) [1 0]
   (== n 1) c
   (>n 1) (let [sub (pow c (quot n 2))]
             (multiply sub sub (if (odd? n) c [1 0])))
   :else (throw (IllegalArgumentException.
                 "n MUST be a natural number"))))

(defn fibonacci-nth
  "Returns the n-th fibonacci in high performance.
The algorithm is: fib(n) = (a^n - b^n) / (a - b)
where a = (1 + sqrt(5)) / 2 and b = -1 / a.
See http://en.wikipedia.org/wiki/Fibonacci_number."
  [n]
  (* (second (pow [1/2 1/2] n)) 2))
```

计算第一百万个斐波那契数，耗时 4秒。大家可以自己试试用迭代法计算，在我的电脑上耗时 1分钟：

```clojure
redraiment.math.fibonacci=>(time (do (fibonacci-nth 1000000) 0))
"Elapsed time: 4272.027189 msecs"
0
```