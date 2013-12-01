---
layout: article
title: 用C语言写解释器[3] - 中缀转后缀
date: 2009-11-01 22:25
category: BASIC解释器
excerpt:
  处理表达式时如何中缀转后缀
---

# 声明

为提高教学质量，我所在的学院正在筹划编写C语言教材。《用C语言写解释器》系列文章经整理后将收入书中“综合实验”一章。因此该系列的文章主要阅读对象定为刚学完C语言的学生（不要求有数据结构等其他知识），所以行文比较罗嗦，请勿见怪。本人水平有限，如有描述不恰当或错误之处请不吝赐教！特此声明。

# 操作符排序

如果你忘记了后缀表达式的概念，赶紧翻回上一篇《[用C语言写解释器（二）]({% post_url 2009-11-01-basic-interpreter-in-c-2 %})》回顾一下。简单地说，将中缀表达式转换成后缀表达式，就是将操作符的执行顺序由“优先级顺序”转换成“在表达式中的先后顺序”。因此，所谓的中缀转后缀，其实就是给原表达式中的操作符排序。

比如将中缀表达式 `5 * ((10 - 1) / 3)` 转换成后缀表达式为 5 10 1 - 3 / *。其中数字 5 10 1 3 仍然按照原先的顺序排列，而操作符的顺序变为 - / ×，这意味着减号最先计算、其次是除号、最后才是乘号。也许你还在担心如何将操作符从两个操作数的中间移到它们的后边。其实不用担心，在完成了排序工作后你就发现它已经跑到操作数的后面了 ^_^。

从中缀表达式 1+2×3+4 中逐个获取操作符，依次是 + × +。如果当前操作符的优先级不大于前面的操作符时，前面操作符就要先输出。比如例子中的第二个加号，它前面是乘号，因此乘号从这个队伍中跑到输出的队伍中当了“老大”；此时第二个加号再前面的加号比较，仍然没有比它大，因此第一个加号也排到新队伍中去了；最后队伍中只剩下加号自己了，所以它也走了。得到新队伍里的顺序 × + + 就是所求解。下面的表格中详细展示每一个步骤。

| 序号 | 输入 | 临时空间 | 输出 |
| --- | ---- | ------- | --- |
| 1 | + |   |   |
| 2 | × | + |   |
| 3 | + | + × |   |
| 4 |   | + × + |   |
| 5 |   | + + | × |
| 6 |   | + | × + |
| 7 |   |   | × + + |

相信你心里还是牵挂着那些操作数。很简单，如果碰到的是操作符就按上面的规则处理，如果是操作数就直接输出！下面的表格加上了操作数，将输出完整的后缀表达式。

| 序号 | 输入 | 临时空间 | 输出 |
| --- | ---- | ------- | --- |
| 1 | 1 |   |   |
| 2 | + |   | 1 |
| 3 | 2 | + | 1 |
| 4 | × | + | 1 2 |
| 5 | 3 | + × | 1 2 |
| 6 | + | + × | 1 2 3 |
| 7 |   | + × + | 1 2 3 |
| 8 |   | + + | 1 2 3 × |
| 9 | 4 | + | 1 2 3 × + |
| 10 |   | + | 1 2 3 × + 4 |
| 11 |   |   | 1 2 3 × + 4 + |

得到最终结果 1 2 3 × + 4 + 就是所求的后缀表达式。下面是程序中的参考代码（有删减）。

```c
// in expression.c
PTLIST infix2postfix ()
{
    PTLIST list = NULL, tail, p;
    PTLIST stack = NULL;
    // 初始时在临时空间放一个优先级最低的操作符
    // 这样就不用判断是否为空了，方便编码
    stack = (PTLIST)calloc(1, sizeof(TOKEN_LIST));
    stack->next = NULL;
    stack->token.type = token_operator;
    stack->token.ator = operators[oper_min];
    // before 为全局变量，用于保存之前的操作符
    // 具体作用参看下面的章节
    memset ( &before, 0, sizeof(before) );
    for (;;) {
        p = (PTLIST)calloc(1, sizeof(TOKEN_LIST));
        // calloc 自动初始化
        p->next = NULL;
        p->token = next_token ();
        if ( p->token.type == token_operand ) {
            // 如果是操作数，就不用客气，直接输出
            if ( !list ) {
                list = tail = p;
            } else {
                tail->next = p;
                tail = p;
            }
        } else if ( p->token.type == token_operator) {
            if ( p->token.ator.oper == oper_rparen ) {
                // 右括号
                free ( p );
                while ( stack->token.ator.oper != oper_lparen ) {
                    p = stack;
                    stack = stack->next;
                    tail->next = p;
                    tail = p;
                    tail->next = NULL;
                }
                p = stack;
                stack = stack->next;
                free ( p );
            } else {
                while ( stack->token.ator.isp >= p->token.ator.icp ) {
                    tail->next = stack;
                    stack = stack->next;
                    tail = tail->next;
                    tail->next = NULL;
                }
                p->next = stack;
                stack = p;
            }
        } else {
            free ( p );
            break;
        }
    }
    while ( stack ) {
        p = stack;
        stack = stack->next;
        if ( p->token.ator.oper != oper_min ) {
            p->next = NULL;
            tail->next = p;
            tail = p;
        } else {
            free ( p );
        }
    }
    return list;
}
```

# 操作符优先级

上一节介绍了中缀转后缀的方法。其中关键的部分就是比较两个操作符的优先级大小。通常情况下这都很简单：比如乘除的优先级比加减大，但括号需要特殊考虑。

中缀表达式中用括号来提升运算符的优先级，因此左括号正在放入（incoming）临时空间时优先级比任何操作符都大；一旦左括号已经放入（in-stack）空间中，此时它优先级如果还是最大，那无论什么操作符过来它就马上被踢出去，而我们想要的是任何操作符过来都能顺利放入临时空间，因此它放入空间后优先级需要变为最小。这意味着左括号在放入空间前后的优先级是不同的，所以我们需要用两个优先级变量 icp 和 isp 来分别记录操作符在两个状态下的优先级（还记得上一篇的问题吗）。

另一个是右括号，它本身和优先级无关，它会将临时空间里的操作符一个个输出，直到碰到左括号为止。下面是本程序中中缀转后缀的代码（有删减）。

# 获取标识符

在上面的代码中你会看到一个陌生的函数 next_taken() 。它会从中缀表达式中获得一个标记，方法类似从字符串中提取单词（参看课后习题）。在本程序中能识别的标记除了操作符，还有纯数字、字符串、变量名等操作数。唯一要注意的就是操作符和操作数之间可以存在零到多个空格。下面是参考代码（有删减）。

```c
// in expression.c
static TOKEN next_token ()
{
    TOKEN token = {0};
    STRING s;
    int i;
    if ( e == NULL ) {
        return token;
    }
    // 去掉前导空格
    while ( *e && isspace(*e) ) {
        e++;
    }
    if ( *e == 0 ) {
        return token;
    }
    if ( *e == '"' ) {
        // 字符串
        token.type = token_operand;
        token.var.type = var_string;
        e++;
        for ( i = 0; *e && *e != '"'; i++ ) {
            token.var.s[i] = *e;
            e++;
        }
        e++;
    } else if ( isalpha(*e) ) {
        // 如果首字符为字母则有两种情况
        // 1. 变量
        // 2. 逻辑操作符
        token.type = token_operator;
        for ( i = 0; isalnum(*e); i++ ) {
            s[i] = toupper(*e);
            e++;
        }
        s[i] = 0;
        if ( !strcmp ( s, "AND" ) ) {
            token.ator = operators[oper_and];
        } else if ( !strcmp ( s, "OR" ) ) {
            token.ator = operators[oper_or];
        } else if ( !strcmp ( s, "NOT" ) ) {
            token.ator = operators[oper_not];
        } else if ( i == 1 ) {
            token.type = token_operand;
            token.var = memory[s[0]-'A'];
            if ( token.var.type == var_null ) {
                memset ( &token, 0, sizeof(token) );
                fprintf ( stderr, "变量%c未赋值！/n", s[0] );
                exit ( EXIT_FAILURE );
            }
        } else {
            goto errorhandler;
        }
    } else if ( isdigit(*e) || *e == '.' ) {
        // 数字
        token.type = token_operand;
        token.var.type = var_double;
        for ( i = 0; isdigit(*e) || *e == '.'; i++ ) {
            s[i] = *e;
            e++;
        }
        s[i] = 0;
        if ( sscanf ( s, "%lf", &token.var.i ) != 1 ) {
            // 读取数字失败！
            // 错误处理
        }
    } else {
        // 剩下算数运算符和关系运算符
        token.type = token_operator;
        switch (*e) {
        case '(':
            token.ator = operators[oper_lparen];
            break;
        // ...
        // 此处省略其他操作符的代码
        default:
            // 不可识别的操作符
            break;
        }
        e++;
    }
    before = token;
    return token;
}
```

# 总结

本章主要介绍中缀表达式转后缀表达式的方法，并给出了相应的参考代码。和前一篇文章结合起来就完成了解释器中“表达式求值”和“内存管理”两部分，在下一篇文章中我们将介绍语句的解析，其中包含了输入/输出、分支以及循环语句。

