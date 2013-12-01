---
layout: article
title: 用C语言写解释器[2] - 表达式求值
date: 2009-11-01 14:05
category: BASIC解释器
excerpt:
  如何用C语言实现表达式解析、括号优先级判断、求值等。
---

# 声明

为提高教学质量，我所在的学院正在筹划编写C语言教材。《用C语言写解释器》系列文章经整理后将收入书中“综合实验”一章。因此该系列的文章主要阅读对象定为刚学完C语言的学生（不要求有数据结构等其他知识），所以行文比较罗嗦，请勿见怪。本人水平有限，如有描述不恰当或错误之处请不吝赐教！特此声明。

# 内存管理

既然是表达式求值，自然需要在内存中保存计算结果以及中间值。在《[用C语言写解释器（一）]({% post_url 2009-10-18-basic-interpreter-in-c-1 %})》中提过：变量要求是若类型，而 C 语言中的变量是强类型，为实现这个目标就需要定义自己的变量类型，参考代码如下（注释部分指出代码所在的文件名，下同）：

```c
// in basic_io.h
#define MEMORY_SIZE (26)

typedef enum {
    var_null = 0,
    var_double,
    var_string
} variant_type;
typedef char STRING[128];
typedef struct {
    variant_type type;
    union {
        double i;
        STRING s;
    };
} VARIANT;

extern VARIANT memory[MEMORY_SIZE];

// in expression.h
typedef VARIANT OPERAND;
```

程序自带 A-Z 26个可用变量，初始时都处于未赋值（ver_null)状态。所有变量必须先赋值再使用，否则就会报错！至于赋值语句的实现请参见后面语法分析的章节。

# 操作符

表达式中光有数值不行，还需要有操作符。在《一》中“表达式运算”一节已经给出了解释器需要实现的所有操作符，包括“算术运算”、“关系运算”和“逻辑运算”。下面给出程序中操作符的定义和声明：

```c
// in expression.h
typedef enum {
    /* 算数运算 */
    oper_lparen = 0,    // 左括号
    oper_rparen,        // 右括号
    oper_plus,          // 加
    oper_minus,         // 减
    oper_multiply,      // 乘
    oper_divide,        // 除
    oper_mod,           // 模
    oper_power,         // 幂
    oper_positive,      // 正号
    oper_negative,      // 负号
    oper_factorial,     // 阶乘
    /* 关系运算 */
    oper_lt,            // 小于
    oper_gt,            // 大于
    oper_eq,            // 等于
    oper_ne,            // 不等于
    oper_le,            // 不大于
    oper_ge,            // 不小于
    /* 逻辑运算 */
    oper_and,           // 且
    oper_or,            // 或
    oper_not,           // 非
    /* 赋值 */
    oper_assignment,    // 赋值
    oper_min            // 栈底
} operator_type;
typedef enum {
    left2right,
    right2left
} associativity;
typedef struct {
    int numbers;        // 操作数
    int icp;            // 优先级
    int isp;            // 优先级
    associativity ass;  // 结合性
    operator_type oper; // 操作符
} OPERATOR;

// in expression.c
static const OPERATOR operators[] = {
    /* 算数运算 */
    {2, 17, 1, left2right, oper_lparen},     // 左括号
    {2, 17, 17, left2right, oper_rparen},    // 右括号
    {2, 12, 12, left2right, oper_plus},      // 加
    {2, 12, 12, left2right, oper_minus},     // 减
    {2, 13, 13, left2right, oper_multiply},  // 乘
    {2, 13, 13, left2right, oper_divide},    // 除
    {2, 13, 13, left2right, oper_mod},       // 模
    {2, 14, 14, left2right, oper_power},     // 幂
    {1, 16, 15, right2left, oper_positive},  // 正号
    {1, 16, 15, right2left, oper_negative},  // 负号
    {1, 16, 15, left2right, oper_factorial}, // 阶乘
    /* 关系运算 */
    {2, 10, 10, left2right, oper_lt},        // 小于
    {2, 10, 10, left2right, oper_gt},        // 大于
    {2, 9, 9, left2right, oper_eq},          // 等于
    {2, 9, 9, left2right, oper_ne},          // 不等于
    {2, 10, 10, left2right, oper_le},        // 不大于
    {2, 10, 10, left2right, oper_ge},        // 不小于
    /* 逻辑运算 */
    {2, 5, 5, left2right, oper_and},         // 且
    {2, 4, 4, left2right, oper_or},          // 或
    {1, 15, 15, right2left, oper_not},       // 非
    /* 赋值 */
    // BASIC 中赋值语句不属于表达式！
    {2, 2, 2, right2left, oper_assignment},  // 赋值
    /* 最小优先级 */
    {2, 0, 0, right2left, oper_min}          // 栈底
};
```

你也许会问为什么需要 `icp（incoming precedence）`、`isp（in-stack precedence）` 两个优先级，现在不用着急，以后会详细解释！

# 后缀表达式

现在操作数（operand）和操作符（operator）都有了，一个表达式就是由它们组合构成的，我们就统称它们为标记（token）。在程序中定义如下：

```c
// in expression.h
typedef enum {
    token_operand = 1,
    token_operator
} token_type;
typedef struct {
    token_type type;
    union {
        OPERAND var;
        OPERATOR ator;
    };
} TOKEN;
typedef struct tlist {
    TOKEN token;
    struct tlist *next;
} TOKEN_LIST, *PTLIST;
```

我们平时习惯将表达式符写作：`operand operator operand`（比如1+1），这是一个递归的定义，表达式本身也可作为操作数。像这种将操作符放在两个操作数之间的表达式称为中缀表达式，中缀表达式的好处是可读性强，操作数之间泾渭分明（尤其是手写体中）。但它有自身的缺陷：操作符的位置说明不了它在运算的先后问题。例如 1+2×3 中，虽然 + 的位置在 × 之前，但这并不表示先做加运算再做乘运算。为解决这个问题，数学中给操作符分了等级，级别高的操作符先计算（乘号的级别比加号高），并用*括号*提高操作符优先级。因此上例表达式的值是 7 而不是 `(1+2)*3=9`。

但对于计算机来说，优先级是一个多余的概念。就像上面提到的，中缀表达式中操作符的顺序没有提供运算先后关系的信息，这就好比用4个字节的空间仅保存1个字节数据——太浪费了！索性将操作符按照运算的先后排序：先计算的排最前面。此时操作符就不适合再放中间了，可以将它移到被操作数的后面：`operand operand operator`（比如 1 1 +）。上例中 `1+2×3` 就变化为 `1 2 3 × +；(1+2)×3` 变化成 `1 2 + 3 ×`，这种将操作符符放到操作数后面的表达式称为后缀表达式。同理还有将操作符符按照逆序放到操作数的前面的前缀表达式。

无论是前缀表达式还是后缀表达式，它们的优点都是用操作符的顺序来代替优先级，这样就可以舍弃括号等概念，化繁为简。

# 后缀表达式求值

请看下面的梯等式计算，比较中缀表达式和后缀表达式的求值过程。

<pre>  8 × <span style="background-color: #ffff00;">( 2 + 3 )</span>        8 <span style="background-color: #ffff00;">2 3 +</span> ×
= <span style="background-color: #ffff00;">8 * 5</span>              = <span style="background-color: #ffff00;">8 5 ×</span>
= <span style="background-color: #ffff00;">40</span>                 = <span style="background-color: #ffff00;">40</span></pre>

后缀表达式的求值方式：从头开始一个标记（token）一个标记地往后扫描，碰到操作数时先放到一个临时的空间里；碰到操作符就从空间里取出最后两个操作数，做相应的运算，然后将结果再次放回空间中。到了最后，空间中就只剩下操作数即运算结果！这个中缀表达式求值类似，只不过中缀表达式操作数取的是前后各一个。下面的代码是程序中后缀表达式求值的节选，其中只包含加法运算，其他运算都是类似的。

```c
// in expression.c
VARIANT eval ( const char expr[] )
{
    // ...
    // 一些变量的定义和声明

    // 将中缀表达式转换成后缀表达式
    // 转换方法将在后续文章中介绍
    list = infix2postfix ();
    while ( list ) {
        // 取出一个 token
        p = list;
        list = list-&gt;next;

        // 如果是操作数就保存到 stack 中
        if ( p-&gt;token.type == token_operand ) {
            p-&gt;next = stack;
            stack = p;
            continue;
        }

        // 如果是操作符...
        switch ( p-&gt;token.ator.oper ) {
        // 加法运算
        case oper_plus:
            // 取出 stack 中最末两个操作数
            op2 = stack;
            op1 = stack = stack-&gt;next;

            if ( op1-&gt;token.var.type == var_double &amp;&amp;
                 op2-&gt;token.var.type == var_double ) {
                op1-&gt;token.var.i += op2-&gt;token.var.i;
            } else {
                // 字符串的加法即合并两个字符串
                // 如果其中一个是数字则需要先转换为字符串
                if ( op1-&gt;token.var.type == var_double ) {
                    sprintf ( s1, &quot;%g&quot;, op1-&gt;token.var.i );
                } else {
                    strcpy ( s1, op1-&gt;token.var.s );
                }
                if ( op2-&gt;token.var.type == var_double ) {
                    sprintf ( s2, &quot;%g&quot;, op2-&gt;token.var.i );
                } else {
                    strcpy ( s2, op2-&gt;token.var.s );
                }
                op1-&gt;token.type = var_string;
                strcat ( s1, s2 );
                strcpy ( op1-&gt;token.var.s, s1 );
            }
            free ( op2 );
            break;
        // ...
        // 其他操作符方法类似
        default:
            // 无效操作符处理
            break;
        }
        free ( p );
    }

    value = stack-&gt;token.var;
    free ( stack );

    // 最后一个元素即表达式的值
    return value;
}
```

# 总结

这一篇文章主要介绍了表达式中的操作符、操作数在程序内部的表示方法、后缀表达式的相关知识以及后缀表达式求值的方法。在下一篇文章中将着重介绍如何将中缀表达式转换成后缀表达式。
