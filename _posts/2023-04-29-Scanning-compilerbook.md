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