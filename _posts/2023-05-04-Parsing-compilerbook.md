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

扫描将文本中的一个个字符整合成一个个词法单元，解析将这些词法单元整合成一个个句子。解析计算机程序，首先需要描述语言中合法句子的形式。这种正式陈述称之为上下文无关语法（Context Free Grammar，CFG）。CFG允许递归，拥有比正则表达式更为丰富的结构。CFG容易写，但不一定容易解析，常见的两种CFG为LL(1)语法和LR(1)语法。

LL(1)可以只考虑当前规则和输入流中的下一个词法单元来实现求值。这个性质降低了手写递归下降解析器的难度。然而，LL(1)对语言的语法具有严格的要求，并不是所有的语法结构可以表示成LL(1)语法。

相比LL(1)，LR(1)更加通用和强大。基本上所有可用的编程语言可以写成LR(1)形式。但是，LR(1)语法的解析算法更为复杂，很难手写。通常使用解析器生成器接收LR(1)语法，自动生成解析代码。

## CFG
首先给出几个定义。终结符（terminal）是不能进一步重写的符号，包含关键字、操作符和标识符等；非终结符（non-terminal）表示可以重写的符号，包含声明、陈述和表达式；句子（sentence）表示语言中终结符的一个有效序列，句子形式（sentential form）表示终结符和非终结符的一个有效序列。

CFG则是正式描述语言中允许句子的规则列表。规则的左边是单个非终结符，规则的右边是描述非终结符的允许形式的句子形式。规则的右边也可以是$\epsilon$，表示没有操作。CFG的第一条规则是特殊的，它是程序的顶层定义，左侧的非终结符成为起始符。下面的CFG描述了包含加法、整数和标识符的表达式。

|语法$G_2$|
|---|
|1. P $\rightarrow$ E|
|2. E $\rightarrow$ E + E|
|3. E $\rightarrow$ ident|
|4. E $\rightarrow$ int|

上面的规则1表示完整的程序由一个表达式构成；规则2表示一个表达式可以是任意两个表达式的相加；规则3表示表达式可以是标识符；规则4表示表达式可以是整数。规则2-4可以压缩如下：

$E \rightarrow E + E \| ident \| int$

### 推断语句
每个语法描述了一个语句集合，这个语句集合也称为语法的语言。为了证明给定的语句是语言中的一员，我们需要证明存在一个规则应用序列可以将起始符和给定的句子连接起来。这样的规则应用序列也称作推导。$\Rightarrow$表示通过应用一个语法规则，一个句子形式和另一个句子形式等价。

有两种推断方式，其一是自上而下，其二是自下而上。在自上而下的方式中，我们从起始符出发，应用CFG中的规则来展开非终结符，最后得到目标语句；在自下而上的方式中，我们从目标语句出发，反向应用CFG中的规则，最后到达起始符。

### 模糊语法
模糊语法允许多种推断方式得到相同的目标语句。