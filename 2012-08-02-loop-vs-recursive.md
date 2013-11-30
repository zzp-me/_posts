---
layout: article
title: 循环 vs 递归
date: 2012-08-02 22:54
category: 编程语言生态
excerpt:
  一些同学对递归的理解还停留在“是一种求阶乘比循环低效的方法”，
  但其实递归和循环处理的问题是不同。
---

注：本文代码使用 JavaScript。

一些同学对递归的理解还停留在“是一种求阶乘比循环低效的方法”。但其实递归和循环处理的问题是不同。拿“遍历数组”这个问题来说：循环适合同一维度（单层长度不限）上的遍历，而递归则适合跨维度（层数不限）的遍历。

比如遍历以下一维数组：

```javascript
var a1 = [1];
var a2 = [1, 2];
var a3 = [1, 2, 3];
```

虽然它们长度不一，但循环应付它们非常容易，也很优雅：

```javascript
var flattenByLoop = function(a) {
    for (var i = 0; i < a.length; i++) {
        println(a[i]);
    }
};
```

如果改用递归，则看起来比较别扭：

```javascript
var flattenByRecur = function(i, a) {
    if (i < a.length) {
        println(a[i]);
        flattenByRecur(i + 1, a);
    }
};
```

它们能输出同样的结果，但相比之下递归版本看起来很笨拙。

现在想想，如果元数据变化了：维度扩大到二维。

```javascript
var a = [[1, 2, 3], [4, 5, 6], [7, 8, 9]];
```

此时需要再外面再套一层循环变成双重循环：

```javascript
var flattenByLoop = function(a) {
    for (var i = 0; i < a.length; i++) {
        for (var j = 0; j < a[i].length; j++) {
            println(a[i][j]);
        }
    }
};
```

如果数据的维度再继续扩大，变成三维、四维……甚至动态的N维数组。使用循环该怎么处理呢？

在这种“层数”很深，甚至不确定的情况下，就需要用“递归”来解决跨“层”的问题。

```javascript
var isArray = function(a) {
    return Object.prototype.toString.call(a) === '[object Array]';
};

var flattenByRecur = function(a) {
    if (isArray(a)) {
        for (var i = 0; i < a.length; i++) {
            flatten(a[i]);
        }
    } else {
        println(a);
    }
};
```

上面的代码中，如果发现子节点是一个数组，就使用递归进入下一层；而同一层上的遍历则使用循环来完成。
