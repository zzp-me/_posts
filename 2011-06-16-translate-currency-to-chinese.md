---
layout: article
title: 金额转中文大写
date: 2011-06-16 12:32
category: 中文计算器
excerpt:
  支持负数和小数，即能处理欠款、“角”和“分”。
---

几天前分享了一段 JavaScript 代码《[整数转汉语大写]({% post_url 2011-06-13-translate-digit-to-chinese %})》，刚刚做了一个改进，可以处理两位小数（角和分）和负数（欠款）。

# 算法分析

## 负数

如果参数是负数，只需把前缀设成“欠”即可，同时把参数改成正数。

```javascript
var prefix = n < 0? '欠': '';
n = Math.abs(n);
```

## 小数

首先要逐个取出小数部分，例如“0.12”要分别取出1和2。

```javascript
var d = Math.floor(n * Math.pow(10, i + 1)) % 10;
```

小数部分不像整数部分需要分组，因此只需添加单位即可：

```javascript
s += digit[d] + fraction[i];
```

最后，小数部分的零不用读，例如“0.01”读成一分，而不是零角一分。因此去除所有零的部分：

```javascript
s += (digit[d] + fraction[i]).replace(/零./, '');
```

## 整数部分

整数部分完全沿用之前的代码，只需去掉参数的小数部分。

```javascript
n = Math.floor(n);
```

# 代码

完整代码如下：

```javascript
var digit_uppercase = function(n) {
    var digit = [
        '零', '壹', '贰', '叁', '肆',
        '伍', '陆', '柒', '捌', '玖'
    ];

    // 负数
    var prefix = n < 0? '欠': '';
    n = Math.abs(n);

    // 小数
    var fraction = ['角', '分'];
    var s = '';
    for (var i = 0; i < fraction.length; i++) {
        var d = Math.floor(n * Math.pow(10, i + 1)) % 10;
        s += (digit[d] + fraction[i]).replace(/零./, '');
    }
    s = s || '整';

    // 整数
    n = Math.floor(n);
    var unit = ['', '拾', '佰', '仟'];
    var group = ['元', '万', '亿'];
    for (var i = 0; i < group.length && n > 0; i++) {
        var p = '';
        for (var j = 0; j < unit.length && n > 0; j++) {
            p = digit[n % 10] + unit[j] + p;
            n = Math.floor(n / 10);
        }
        s = p.replace(/(零.)*零$/, '')
             .replace(/^$/, '零')
          + group[i] + s;
    }
    return prefix +　s.replace(/(零.)*零元/, '元')
            　　　　　.replace(/(零.)+/g, '零')
            　　　　　.replace(/^整$/, '零元整');
};
```

# 功能测试

## 整数测试数据

```javascript
alert(digit_uppercase(0));          // 零元整
alert(digit_uppercase(123));        // 壹佰贰拾叁元整
alert(digit_uppercase(1000000));    // 壹佰万元整
alert(digit_uppercase(100000001));  // 壹亿零壹元整
alert(digit_uppercase(1000000000)); // 壹拾亿元整
alert(digit_uppercase(1234567890)); // 壹拾贰亿叁仟肆佰伍拾陆万柒仟捌佰玖拾元整
alert(digit_uppercase(1001100101)); // 壹拾亿零壹佰壹拾万零壹佰零壹元整
alert(digit_uppercase(110101010));  // 壹亿壹仟零壹拾万壹仟零壹拾元整
```

## 小数测试数据

```javascript
alert(digit_uppercase(0.12));          // 壹角贰分
alert(digit_uppercase(123.34));        // 壹佰贰拾叁元叁角肆分
alert(digit_uppercase(1000000.56));    // 壹佰万元伍角陆分
alert(digit_uppercase(100000001.78));  // 壹亿零壹元柒角捌分
alert(digit_uppercase(1000000000.90)); // 壹拾亿元玖角
alert(digit_uppercase(1234567890.03)); // 壹拾贰亿叁仟肆佰伍拾陆万柒仟捌佰玖拾元叁分
alert(digit_uppercase(1001100101.00)); // 壹拾亿零壹佰壹拾万零壹佰零壹元整
alert(digit_uppercase(110101010.10));  // 壹亿壹仟零壹拾万壹仟零壹拾元壹角
```

## 负数测试数据

```javascript
alert(digit_uppercase(-0.12));          // 欠壹角贰分
alert(digit_uppercase(-123.34));        // 欠壹佰贰拾叁元叁角肆分
alert(digit_uppercase(-1000000.56));    // 欠壹佰万元伍角陆分
alert(digit_uppercase(-100000001.78));  // 欠壹亿零壹元柒角捌分
alert(digit_uppercase(-1000000000.90)); // 欠壹拾亿元玖角
alert(digit_uppercase(-1234567890.03)); // 欠壹拾贰亿叁仟肆佰伍拾陆万柒仟捌佰玖拾元叁分
alert(digit_uppercase(-1001100101.00)); // 欠壹拾亿零壹佰壹拾万零壹佰零壹元整
alert(digit_uppercase(-110101010.10));  // 欠壹亿壹仟零壹拾万壹仟零壹拾元壹角
```