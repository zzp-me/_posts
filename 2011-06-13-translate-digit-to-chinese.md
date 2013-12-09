---
layout: article
title: 整数转汉语大写
date: 2011-06-13 22:35
category: 中文计算器
excerpt:
  在网上看到的数字转中文大写的程序都太繁琐，而且普遍功能有缺陷，
  例如100被翻译成“壹佰零拾零”。实在看不下去，就自己研究汉语中数字的读写方法，
  并实现一个优雅的算法。
---

# 起因

前几天用网银给朋友转账，在金额一栏中输入阿拉伯数字，右边会立即显示出相应的汉语数字大写。感觉挺有意思，就到网上搜索一下现成代码，找到一段 Java 的和一段 C# 的，但它们的实现都太繁琐，并且功能上有缺陷，例如 100 被翻译成“壹佰零拾零”，在汉语中应该是“壹佰元整”。因此决定自己研究汉语中数字的读写方法，实现正确并且优雅的算法。

# 代码

下面是我的代码，目前只处理正整数。

```javascript
function digit_uppercase(n) {
    var digit = [
        '零', '壹', '贰', '叁', '肆',
        '伍', '陆', '柒', '捌', '玖'
    ];
    var unit = [
        ['元', '万', '亿'],
        ['', '拾', '佰', '仟']
    ];
    var s = '';
    for (var i = 0; i < unit[0].length && n > 0; i++) {
        var p = '';
        for (var j = 0; j < unit[1].length && n > 0; j++) {
            p = digit[n % 10] + unit[1][j] + p;
            n = Math.floor(n / 10);
        }
        s = p.replace(/(零.)*零$/, '')
             .replace(/^$/, '零')
          + unit[0][i] + s;
    }
    return s.replace(/(零.)*零元/, '元')
            .replace(/(零.)+/g, '零')
            .replace(/^$/, '零元') + '整';
}
```

# 功能测试

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
