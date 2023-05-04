---
layout: post
title:  Parsing to compilerbook
date: 2023-05-04 20:30:30 +0800
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

<script type="module">
import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@10.0.2/+esm';
mermaid.initialize({ startOnLoad: false });
await mermaid.run({
  querySelector: '.language-mermaid',
});
</script>

## 概览

扫描将文本中的一个个字符整合成一个个词法单元，解析将这些词法单元整合成一个个句子。解析计算机程序，首先需要描述语言中合法句子的形式。这种正式陈述称之为上下文无关语法（Context Free Grammar，CFG）。CFG允许递归，拥有比正则表达式更为丰富的结构。CFG容易写，但不一定容易解析，常见的两种CFG为LL（1）语法和LR（1）语法。

LL (1)可以只考虑当前规则和输入流中的下一个词法单元来实现求值。这个性质降低了手写递归下降解析器的难度。然而，LL（1）对语言的语法具有严格的要求，并不是所有的语法结构可以表示成LL（1）语法。

相比LL（1），LR（1）更加通用和强大。基本上所有可用的编程语言可以写成LR（1）形式。但是，LR（1）语法的解析算法更为复杂，很难手写。通常使用解析器生成器接收LR（1）语法，自动生成解析代码。

## CFG