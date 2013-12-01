---
layout: article
title: 用C语言写解释器[4] - 语句分析
date: 2009-11-02 20:33
category: BASIC解释器
excerpt:
  控制语句的解析与实现
---

# 声明

为提高教学质量，我所在的学院正在筹划编写C语言教材。《用C语言写解释器》系列文章经整理后将收入书中“综合实验”一章。因此该系列的文章主要阅读对象定为刚学完C语言的学生（不要求有数据结构等其他知识），所以行文比较罗嗦，请勿见怪。本人水平有限，如有描述不恰当或错误之处请不吝赐教！特此声明。

# 语句

在前面的章节中已经成功实现了内存管理和表达式求值模块。之所以称表达式求值是解释器的核心部分，是因为几乎所有语句的操作都伴随着表达式求值。也许你已经迫不及待地给 eval 传值让它执行复杂的运输了，但目前来讲它充其量只是一个计算器。要想成为一门语言，还需要一套自成体系的语法，包括输入输出语句和控制语句。但在进行语法分析之前，首先需要将 BASIC 源码载入到内存中。

# BASIC 源码载入

在《[用C语言写解释器（一）]({% post_url 2009-10-18-basic-interpreter-in-c-1 %})》中附了一段 BASIC 参考代码，每一行的结构是一个行号+一条语句。其中行号为 1-9999 之间的正整数，且当前行号大于前面的行号；语句则由以下即将介绍的 3 条 I/O 语句和 8 条控制语句组成。为方便编码，程序中采用静态数组来保存源代码，读者可以尝试用链表结构实现动态申请的版本。下面是代码结构的定义。

```c
// in basic_io.h
#define PROGRAM_SIZE (10000)

typedef struct {
    int ln;     // line number
    STRING line;
} CODE;

extern CODE code[PROGRAM_SIZE];
extern int cp;
extern int code_size;
```

其中 `code_size` 的作用顾名思义：记录代码的行数。cp （0 ≤ cp < code_size）记录当前行的下标（比如 cp 等于5时表明执行到第5行）。下面是载入 BASIC 源码的参考代码，在载入源码的同时会去除两端的空白字符。

```c
// in basic_io.c
void load_program ( STRING filename )
{
    FILE *fp = fopen ( filename, "r" );
    int bg, ed;

    if ( fp == NULL ) {
        fprintf ( stderr, "文件 %s 无法打开！/n", filename );
        exit ( EXIT_FAILURE );
    }

    while ( fscanf ( fp, "%d", &code[cp].ln ) != EOF ) {
        if ( code[cp].ln <= code[cp-1].ln ) {
            fprintf ( stderr, "Line %d: 标号错误！/n", cp );
            exit ( EXIT_FAILURE );
        }

        fgets ( code[cp].line, sizeof(code[cp].line), fp );
        for ( bg = 0; isspace(code[cp].line[bg]); bg++ );
        ed = (int)strlen ( code[cp].line + bg ) - 1;
        while ( ed >= 0 && isspace ( code[cp].line[ed+bg] ) ) {
            ed--;
        }
        if ( ed >= 0 ) {
            memmove ( code[cp].line, code[cp].line + bg, ed + 1 );
            code[cp].line[ed + 1] = 0;
        } else {
            code[cp].line[0] = 0;
        }

        cp++;
        if ( cp >= PROGRAM_SIZE ) {
            fprintf ( stderr, "程序%s太大，代码空间不足！/n", filename );
            exit ( EXIT_FAILURE );
        }
    }

    code_size = cp;
    cp = 1;
}
```

# 语法分析

源码载入完成后就要开始逐行分析语句了，程序中总共能处理以下 11 种语句：

```c
// in main.c
typedef enum {
    key_input = 0,  // INPUT
    key_print,      // PRINT
    key_for,        // FOR .. TO .. STEP
    key_next,       // NEXT
    key_while,      // WHILE
    key_wend,       // WEND
    key_if,         // IF
    key_else,       // ELSE
    key_endif,      // END IF
    key_goto,       // GOTO
    key_let         // LET
} keywords;
```

《[用C语言写解释器（一）]({% post_url 2009-10-18-basic-interpreter-in-c-1 %})》中详细描述了每个语句的语法，本程序中所谓的语法其实就是字符串匹配，参考代码如下：

```c
// in main.c
keywords yacc ( const STRING line )
{
    if ( !strnicmp ( line, "INPUT ", 6 ) ) {
        return key_input;
    } else if ( !strnicmp ( line, "PRINT ", 6 ) ) {
        return key_print;
    } else if ( !strnicmp ( line, "FOR ", 4 ) ) {
        return key_for;
    } else if ( !stricmp ( line, "NEXT" ) ) {
        return key_next;
    } else if ( !strnicmp ( line, "WHILE ", 6 ) ) {
        return key_while;
    } else if ( !stricmp ( line, "WEND" ) ) {
        return key_wend;
    } else if ( !strnicmp ( line, "IF ", 3 ) ) {
        return key_if;
    } else if ( !stricmp ( line, "ELSE" ) ) {
        return key_else;
    } else if ( !stricmp ( line, "END IF" ) ) {
        return key_endif;
    } else if ( !strnicmp ( line, "GOTO ", 5 ) ) {
        return key_goto;
    } else if ( !strnicmp ( line, "LET ", 4 ) ) {
        return key_let;
    } else if ( strchr ( line, '=' ) ) {
        return key_let;
    }

    return -1;
}
```

每个语句对应有一个执行函数，在分析出是哪种语句后，就可以调用它了！为了编码方便，我们将这些执行函数保存在一个函数指针数组中，请看下面的参考代码：

```c
// in main.c
void (*key_func[])( const STRING ) = {
    exec_input,
    exec_print,
    exec_for,
    exec_next,
    exec_while,
    exec_wend,
    exec_if,
    exec_else,
    exec_endif,
    exec_goto,
    exec_assignment
};

int main ( int argc, char *argv[] )
{
    if ( argc != 2 ) {
        fprintf ( stderr, "usage: %s basic_script_file/n", argv[0] );
        exit ( EXIT_FAILURE );
    }

    load_program ( argv[1] );

    while ( cp < code_size ) {
        (*key_func[yacc ( code[cp].line )]) ( code[cp].line );
        cp++;
    }

    return EXIT_SUCCESS;
}
```

以上代码展示的就是整个程序的基础框架，现在欠缺的只是每个语句的执行函数，下面将逐个详细解释。

# I/O语句

输入输出是一个宽泛的概念，并不局限于从键盘输入和显示到屏幕上，还包括操作文件、连接网络、进程通信等。《我们的目标》中指出只需实现从键盘输入（INPUT）和显示到屏幕上（PRINT），事实上还应该包括赋值语句，只不过它属于程序内部的I/O。

## INPUT 语句

INPUT 语句后面跟着一堆变量名（用逗号隔开）。因为变量是弱类型，你可以输入数字或字符串。但C语言是强类型语言，为实现这个功能就需要判断一下 scanf 的返回值。我们执行 `scanf ( "%lf", &memory[n].i )`，如果你输入的是一个数字，就能成功读取一个浮点数，函数返回 1、否则就返回 0；不能读取时就采用 getchar 来获取字符串！参考代码如下：

```c
// in basic_io.c
void exec_input ( const STRING line )
{
    const char *s = line;
    int n;

    assert ( s != NULL );
    s += 5;

    while ( *s ) {
        while ( *s && isspace(*s) ) {
            s++;
        }
        if ( !isalpha(*s) || isalnum(*(s+1)) ) {
            perror ( "变量名错误！/n" );
            exit ( EXIT_FAILURE );
        } else {
            n = toupper(*s) - 'A';
        }

        if ( !scanf ( "%lf", &memory[n].i ) ) {
            int i;
            // 用户输入的是一个字符串
            memory[n].type = var_string;
            if ( (memory[n].s[0] = getchar()) == '"' ) {
                for ( i = 0; (memory[n].s[i]=getchar())!='"'; i++ );
            } else {
                for ( i = 1; !isspace(memory[n].s[i]=getchar()); i++ );
            }
            memory[n].s[i] = 0;
        } else {
            memory[n].type = var_double;
        }

        do {
            s++;
        } while ( *s && isspace(*s) );
        if ( *s && *s != ',' ) {
            perror ( "INPUT 表达式语法错误！/n" );
            exit ( EXIT_FAILURE );
        } else if ( *s ) {
            s++;
        }
    }
}
```

## PRINT 语句

输出相对简单些，PRINT 后面跟随的是一堆表达式，表达式只需委托给 eval 来求值即可，因此 PRINT 要做的仅仅是按照值的类型来输出结果。唯一需要小心的就是类似 `PRINT "hello, world"` 这样字符串中带有逗号的情况，以下是参考代码：

```c
// in basic_io.c
void exec_print ( const STRING line )
{
    STRING l;
    char *s, *e;
    VARIANT v;
    int c = 0;

    strcpy ( l, line );
    s = l;

    assert ( s != NULL );
    s += 5;

    for (;;) {
        for ( e = s; *e && *e != ','; e++ ) {
            // 去除字符串
            if ( *e == '"' ) {
                do {
                    e++;
                } while ( *e && *e != '"' );
            }
        }
        if ( *e ) {
            *e = 0;
        } else {
            e = NULL;
        }

        if ( c++ ) putchar ( '/t' );
        v = eval ( s );
        if ( v.type == var_double ) {
            printf ( "%g", v.i );
        } else if ( v.type == var_string ) {
            printf ( v.s );
        }

        if ( e ) {
            s = e + 1;
        } else {
            putchar ( '/n' );
            break;
        }
    }
}
```

## LET 语句

在 BASIC 中，“赋值”和“等号”都使用“=”，因此不能像 C 语言中使用 A = B = C 这样连续赋值，在 BASIC 中它的意思是判断 B 和 C 的值是否相等并将结果赋值给 A 。而且关键字 LET 是可选的，即 `LET A = 1` 和 `A = 1` 是等价的。剩下的事情那个就很简单了，只要将表达式的值赋给变量即可。以下是参考代码：

```c
// in basic_io.c
void exec_assignment ( const STRING line )
{
    const char *s = line;
    int n;

    if ( !strnicmp ( s, "LET ", 4 ) ) {
        s += 4;
    }
    while ( *s && isspace(*s) ) {
        s++;
    }
    if ( !isalpha(*s) || isalnum(*(s+1)) ) {
        perror ( "变量名错误！/n" );
        exit ( EXIT_FAILURE );
    } else {
        n = toupper(*s) - 'A';
    }

    do {
        s++;
    } while ( *s && isspace(*s) );
    if ( *s != '=' ) {
        fprintf ( stderr, "赋值表达式 %s 语法错误！/n", line );
        exit ( EXIT_FAILURE );
    } else {
        memory[n] = eval ( s + 1 );
    }
}
```

# 控制语句

现在是最后一个模块——控制语句。控制语句并不参与交互，它们的作用只是根据一定的规则来改变代码指针（cp）的值，让程序能到指定的位置去继续执行。限于篇幅，本节只介绍 for、next 以及 goto 三个控制语句的实现方法，读者可以尝试自己完成其他函数，也可以参看附带的完整代码。

## FOR 语句

先来看一下 FOR 语句的结构：

    FOR var = expression1 TO expression2 [STEP expression3]

它首先要计算三个表达式，获得 v1、v2、v3 三个值，然后让变量（var）从 v1 开始，每次迭代都加 v3，直到超出 v2 的范围位置。因此，每一个 FOR 语句，我们都需要保存这四个信息：变量名、起始值、结束值以及步长。另外，不要忘记 FOR 循环等控制语句可以嵌套使用，因此需要开辟一组空间来保存这些信息，参考代码如下：

```c
// in grammar.h
static struct {
    int id;           // memory index
    int ln;           // line number
    double target;    // target value
    double step;
} stack_for[MEMORY_SIZE];
static int top_for = -1;
```

分析的过程就是通过 strstr 在语句中搜索“=”、“TO”、“STEP”等字符串，然后将提取的表达式传递给 eval 计算，并将值保存到 stack_for 这个空间中。参考代码如下：

```c
// in grammar.c
void exec_for ( const STRING line )
{
    STRING l;
    char *s, *t;
    int top = top_for + 1;

    if ( strnicmp ( line, "FOR ", 4 ) ) {
        goto errorhandler;
    } else if ( top >= MEMORY_SIZE ) {
        fprintf ( stderr, "FOR 循环嵌套过深！/n" );
        exit ( EXIT_FAILURE );
    }

    strcpy ( l, line );

    s = l + 4;
    while ( *s && isspace(*s) ) s++;
    if ( isalpha(*s) && !isalnum(s[1]) ) {
        stack_for[top].id = toupper(*s) - 'A';
        stack_for[top].ln = cp;
    } else {
        goto errorhandler;
    }

    do {
        s++;
    } while ( *s && isspace(*s) );
    if ( *s == '=' ) {
        s++;
    } else {
        goto errorhandler;
    }

    t = strstr ( s, " TO " );
    if ( t != NULL ) {
        *t = 0;
        memory[stack_for[top].id] = eval ( s );
        s = t + 4;
    } else {
        goto errorhandler;
    }

    t = strstr ( s, " STEP " );
    if ( t != NULL ) {
        *t = 0;
        stack_for[top].target = eval ( s ).i;
        s = t + 5;
        stack_for[top].step = eval ( s ).i;
        if ( fabs ( stack_for[top].step ) < 1E-6 ) {
            goto errorhandler;
        }
    } else {
        stack_for[top].target = eval ( s ).i;
        stack_for[top].step = 1;
    }

    if ( (stack_for[top].step > 0 && 
         memory[stack_for[top].id].i > stack_for[top].target)||
         (stack_for[top].step < 0 && 
         memory[stack_for[top].id].i < stack_for[top].target)) {
        while ( cp < code_size && strcmp(code[cp].line, "NEXT") ) {
            cp++;
        }
        if ( cp >= code_size ) {
            goto errorhandler;
        }
    } else {
        top_for++;
    }

    return;

errorhandler:
    fprintf ( stderr, "Line %d: 语法错误！/n", code[cp].ln );
    exit ( EXIT_FAILURE );
}
```

## NEXT 语句

NEXT 的工作就简单得多了。它从 stack_for 这个空间中取出最后一组数据，让变量的值累加上步长，并判断循环是否结束。如果结束就跳出循环执行下一条语句；否则就将代码指针移回循环体的顶部，继续执行循环体。下面是参考代码。

```c
// in grammar.c
void exec_next ( const STRING line )
{
    if ( stricmp ( line, "NEXT" ) ) {
        fprintf ( stderr, "Line %d: 语法错误！/n", code[cp].ln );
        exit ( EXIT_FAILURE );
    }
    if ( top_for < 0 ) {
        fprintf ( stderr, "Line %d: NEXT 没有相匹配的 FOR！/n", code[cp].ln );
        exit ( EXIT_FAILURE );
    }

    memory[stack_for[top_for].id].i += stack_for[top_for].step;
    if ( stack_for[top_for].step > 0 && 
         memory[stack_for[top_for].id].i > stack_for[top_for].target ) {
        top_for--;
    } else if ( stack_for[top_for].step < 0 && 
         memory[stack_for[top_for].id].i < stack_for[top_for].target ) {
        top_for--;
    } else {
        cp = stack_for[top_for].ln;
    }
}
```

## GOTO 语句

也许你认为 GOTO 语句只是简单的将 cp 的值设置为指定的行，但事实上它比想象中的要复杂些。考虑下面的 BASIC 代码：

```
0010 I = 5
0020 GOTO 40
0030 FOR I = 1 TO 10
0040   PRINT I
0050 NEXT
```

像这类代码，直接跳到循环体内部，如果只是简单地将 cp 移动到指定位置，当代码继续执行到 NEXT 时就会报告没有对应的 FOR 循环！跳到其他的控制结构，如 WHILE、IF 等，也会出现相同的问题。以下是参考代码（有删减）。

```c
// in grammar.c
void exec_goto ( const STRING line )
{
    int ln;

    if ( strnicmp ( line, "GOTO ", 5 ) ) {
        fprintf ( stderr, "Line %d: 语法错误！/n", code[cp].ln );
        exit ( EXIT_FAILURE );
    }

    ln = (int)eval ( line + 5 ).i;
    if ( ln > code[cp].ln ) {
        // 往下跳转
        while ( cp < code_size && ln != code[cp].ln ) {
            if ( !strnicmp ( code[cp].line, "IF ", 3 ) ) {
                top_if++;
                stack_if[top_if] = 1;
            } else if ( !stricmp ( code[cp].line, "ELSE" ) ) {
                stack_if[top_if] = 1;
            } else if ( !stricmp ( code[cp].line, "END IF" ) ) {
                top_if--;
            } else if ( !strnicmp ( code[cp].line, "WHILE ", 6 ) ) {
                top_while++;
                stack_while[top_while].isrun = 1;
                stack_while[top_while].ln = cp;
            } else if ( !stricmp ( code[cp].line, "WEND" ) ) {
                top_while--;
            } else if ( !strnicmp ( code[cp].line, "FOR ", 4 ) ) {
                int i = 4;
                VARIANT v;
                while ( isspace(code[cp].line[i]) ) i++;
                v = memory[toupper(code[cp].line[i])-'A'];
                exec_for ( code[cp].line );
                memory[toupper(code[cp].line[i])-'A'] = v;
            } else if ( !stricmp ( code[cp].line, "NEXT" ) ) {
                top_for--;
            }
            cp++;
        }
    } else if ( ln < code[cp].ln ) {
        // 往上跳转
        // 代码类似，此处省略
    } else {
        // 我不希望出现死循环，你可能有其他处理方式
        fprintf ( stderr, "Line %d: 死循环！/n", code[cp].ln );
        exit ( EXIT_FAILURE );
    }

    if ( ln == code[cp].ln ) {
        cp--;
    } else {
        fprintf ( stderr, "标号 %d 不存在！/n", ln );
        exit ( EXIT_FAILURE );
    }
}
```

# 总结

本章介绍了源码载入、语法分析以及部分语句的实现，WHILE 和 IF 等控制语句方法和 FOR、NEXT 类似，有兴趣的读者请尝试自己实现（或者参看附带的完整源码）。这样一个解释器的四个关键部分“内存管理”、“表达式求值”、“输入输出”和“控制语句”就全部介绍完了，希望你也能写出自己的解释器。下一篇我将总结一下我个人对编程语言的一些思考，如果你也有兴趣请继续关注《用C语言写解释器（五）》！
