---
layout: post
title:  Scanning to compilerbook
date: 2023-04-29 20:30:30 +0800
category: Compiler
---
<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>

## 词法单元的种类
扫描指的是从程序的原生文本源码中识别出词法单元的过程，需要编程语言设计者讲清楚精密的细节——哪些是允许的，哪些是不允许的。大多数语言的词法单元包含以下系列：
- 关键字
- 标识符
- 数字
- 字符串
- 注释和空格

设计新的编程语言语言或者为已有编程语言设计编译器的时候，首先要做的就是讲清楚每个类型的词法单元所允许的字符。下面是一种非正式的实现方式。

## 手制的扫描器
```
token_t scan_token( FILE *fp ) {
    int c = fgetc(fp);
    if(c == '*')
    {
        return TOKEN_MULTIPLY;
    }
    else if(c == '!')
    {
        char d = fgetc(fp);
        if(d == '=')
        {
            return TOKEN_NOT_EQUAL;
        }
        else
        {
            ungetc(d, fp);
            return TOKEN_NOT;
        }
    }
    else if(isalpha(c))
    {
        do
        {
            char d = fgetc(fp);
        }
        while(isalnum(d));
        ungetc(d, fp);
        return TOKEN_IDENTIFIER;
    }
    else if(...)
    {
        ...
    }
}
```

如上所示，手写的扫描器非常冗长。当加入更多类型的词法单元时，扫描器会显得错综复杂，尤其是一些包含相同序列的词法单元。开发者也很难将扫描器代码和每个词法单元的预期定义对应起来。在输入复杂的情况下，可能会导致难以预料的错误。也就是说，对于词法单元不多的小语言，手写的扫描器可以是合适的方案。

对于包含大规模词法单元的复杂语言，我们需要更加形式化的方法来定义和扫描词法单元。形式化方法可以让我们更有把握词法单元之间并不冲突和扫描器实现的正确性。此外，形式化方法也有助于我们得到紧凑的扫描器，提高扫描性能。

正则表达式和有限状态机允许我们准确地声明给定的词法单元。接着，自动化工具可以处理这些定义，寻找错误和模糊性，输出紧凑、高性能的代码。

## 正则表达式