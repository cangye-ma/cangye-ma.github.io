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

<script type="module">
import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@10.0.2/+esm';
mermaid.initialize({ startOnLoad: false });
await mermaid.run({
  querySelector: '.language-mermaid',
});
</script>

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
正则表达式是表达模式的一种语言。我们可用正则表达式来作为一种紧凑和形式化的方法来识别出编译器扫描器接收的词法单元，并将这些表达式翻译成可以工作的代码。让我们准确地定义正则表达式。

正则表达式$s$是代表$L(s)$的字符串，而$L(s)$是一个字符串的集合，这个字符串可看作“$s$的语言”。首先，由如下的基础定义。
- 如果$a \in \Sigma$,那么$a$是正则表达式且L(a) = {a}。
- $\epsilon$是正则表达式，且$L(\epsilon)$只包含空字符串。

接着，对于任意的正则表达式$s$和$t$,有
- s\|t 是正则表达式，满足$ L(s \| t) = L(s) \cup L(t)$。
- st是正则表达式，满足L(st)包含了L(s)和L(t)组合起来的所有可能。
- s* 是正则表达式，满足L(s*)包含L(s)的0次或多次连接。

上面定义的语法可以用来表示任意的正则表达式。下面是建立在基础语法上的一些辅助语法。

|辅助语法|含义|
| --- | --- |
|s?|表示s是可选的，可以写成($s \| \epsilon$)|
|s+|表示s可以重复1到多次，可以写成$ss*$|
|[a-z]|表示范围内的任意字符，可以写成(a\|b\|c\|...\|z)|
|[^x]|表示除了x的任意字符，可以写成$\Sigma$ - x|

正则表达式也遵守一些代数性质，如可结合性、可交换性、可分配性和幂等性：

|性质|表示|
| --- | --- |
|可结合性|a \| (b \| c) = (a \| b) \| c|
|可交换性|a \| b = b \| a|
|可分配性|a(b \| c) = ab \| ac|
|幂等性|a** = a*|

## 有限自动机
有限自动机是可以用来表示特定计算形式的抽象机器。以图表形式，有限自动机由一系列状态和这些状态之间的一系列边构成。每条边都会从字母表$\Sigma$中抽出一到多个符号进行标记。

有限自动机起始于初始态$S_0$，根据输入符号进入下一个状态。有限自动机的一些状态称作接收态，用矩形表示。当输入完成时，有限自动机处于接收态，我们称有限自动机接收该输入；当输入完成时有限自动机不处于接收态，我们称有限自动机拒绝该输入。正则表达式和有限自动机一一对应。对于简单的正则表达式，我们可以自行构造有限自动机。如下是关键字for的自动状态机。
```mermaid
flowchart LR
   A((0)) -- f --> B((1)) -- o --> C((2)) -- r --> D[3]
```

如下是[a-z][a-z0-9]+形式标识符的有限自动机。
```mermaid
flowchart LR
    A((0)) -- a-z --> B((1)) -- a-z0-9 --> C[2]
    C[2] -- a-z0-9 --> C[2]
```

如下是([1-9] [0-9]*) | 0形式数字的有限自动机。
```mermaid
flowchart LR
    A((0)) -- 1-9 --> B[1] -- 0-9 --> C[2]
    C[2] -- 0-9 --> C[2]
    A((0)) -- 0 --> D[3]
```

### 确定的有限自动机

上面的3个例子均为确定的有限自动机（Deterministic Finite Automation， DFA）。DFA是有限自动机的一种特殊情况，其中对于每个给定的输入符号，每个状态最多只有一条输出边。这也意味着已知当前状态和输出符号时，将要做的选择是确定的，因此在软件或硬件中，都是容易实现的。

### 不确定的有限自动机

DFA的一个替代品是不确定的有限自动机（Nondeterministic Finite Automation， NFA）。考虑正则表达式[a-z]*ing，其表示所有后缀为ing的小写单词。它可以表示成如下的自动机。
```mermaid
flowchart LR
    A((0)) -- i --> B((1)) -- n --> C((2)) -- g --> D[3]
    A((0)) -- a-z --> A((0)) 
```
对于单词sing， 初始状态为0。输入s时保留在初始状态0。继续输入s时，既可以保留在初始状态0，也可以进入初始状态1。若保留在初始状态0，最后处理完整个单词时出于初始状态0，为非接收态；若进入初始状态1，最后进入状态3，为接收态。两种选择都遵守传输规则，但得到的结果并不一样。

NFA也可以接收空字符，记作$\epsilon$，也就以为不需要接收字符就可以完成状态切换。下面用NFA来表示a*(ab\|ac)。

```mermaid
flowchart LR
    A((0)) -- $\epsilon$ --> B((1)) -- a --> C((2)) -- b --> D[3]
    A((0)) -- $\epsilon$ --> E((4)) -- a --> F((5)) -- c
    --> G[6]
    A((0)) -- a --> A((0))
```

这个NFA包含了很多模糊的选择。在状态0，它可以消耗一个a保持在状态0；或者，它接收一个$\epsilon$进入状态1或状态4，同样消耗一个a进入下一个状态。有两种常见的思路来理解这里的模糊选择。其一是存在上帝视角，NFA总能做出正确的选择；其二是同时存在多个允许的状态，在输入完成时选择处于接收态的状态。第一种思路不太可行，第二种思路将NFA转换成了DFA。

原则上，我们可以通过简单地跟踪所有可能的状态来在软件或者硬件上实现NFA。但此举过于低效，更好的策略是将NFA转换成等价的DFA。

## 转换算法

如下是正则表达式、NFA和DFA的关系。
![正则表达式、NFA和DFA的关系](/public/images/re_nfa_dfa.png)

### 将正则表达式转换成NFA
首先，我们定义与正则表达式基础条件对应的自动机。

任意字符a的NFA如下所示：
```mermaid
flowchart LR
    A((0)) -- a --> B[1]
```

任意$\epsilon$传输的NFA如下所示：
```mermaid
flowchart LR
    A((0)) -- $\epsilon$ --> B[1]
```

现在假设我们已经为正则表达式A和正则表达式B构造了NFA，其中A和B都只包含一个初始态和一个接收态。那么AB的NFA如下所示：
```mermaid
flowchart LR
    A((0)) -- A --> B((1)) -- $\epsilon$ --> C((2)) -- B --> D[3]
```

A\|B的NFA如下所示：
```mermaid
flowchart LR
    A((0)) -- $\epsilon$ --> B((1)) -- A --> C((2)) -- $\epsilon$ --> D[3]
    A((0)) -- $\epsilon$ --> E((4)) -- B --> F((5)) -- $\epsilon$ --> D[3]    
```

Kleene闭包A*的NFA如下所示：
```mermaid
flowchart LR
    A((0)) -- $\epsilon$ --> B((1)) -- A --> C((2)) -- $\epsilon$ --> D[3]
    A((0)) -- $\epsilon$ --> D[3]
    D[3] -- $\epsilon$ --> A((0))
```

a(cat\|cow)*的NFA如下所示：
```mermaid
flowchart LR
    A((0)) -- a --> B((1)) -- $\epsilon$ --> C((2)) -- $\epsilon$ --> D((3)) -- $\epsilon$ --> E((4)) -- c --> F((5)) -- o --> G((6)) -- w --> H((7)) -- $\epsilon$ --> I((8)) -- $\epsilon$ --> J[9]
    D((3)) -- $\epsilon$ --> K((10)) -- c --> L((11)) -- a --> M((12)) -- t --> N((13)) -- $\epsilon$ --> I((8))
    C((2)) -- $\epsilon$ --> J[9]
    J[9] -- $\epsilon$ --> C((2))
```

表示完整语言词法单元的NFA可能拥有上千个状态，难以实现。一般通过将NFA转换成等价的DFA来处理。

### 将NFA转换成DFA

我们可以使用子集构造将NFA转化成等价的DFA，其基本思想是构造这样一个DFA，其中DFA中的一个状态对应NFA中的多个状态。考虑一个由状态集N和初始状态$N_0$构成的NFA，我们想将其转化成由状态集D和初始状态$D_0$构成的DFA。每个D状态对应多个N状态。

首先定义辅助函数$\epsilon$闭包$\epsilon$ - closure(n),其指的是NFA状态n通过0次或者多次$\epsilon$传输。子集构造算法如下所示：

|子集构造算法|
|---|
|输入：给定由状态集N和初始状态$N_0$构成的NFA，字母表$\Sigma$|
|输出：由状态集D和初始状态$D_0$构成的DFA|
|构建初始态$D_0$ = $\epsilon$ - closure($N_0$)|
|将$D_0$加入至列表|
|While 列表中包含DFA状态|
|$\quad$设d为下一个从列表中去除的DFA状态|
|$\quad$ For $\Sigma$ 中的每个字符|
|$\quad\quad$ 让T包含所有的NFA状态$N_k$,这些状态满足$N_j \in d$并且$N_j \stackrel{c}{\rightarrow} N_k$|
|$\quad\quad$ 构造新的DFA状态$D_i$ = $\epsilon$ - closure($T$)|
|$\quad\quad$ 如果$D_i$不在列表中，将其添加到列表尾部。|

### 最小化DFA
子集构造算法可以帮助我们得到有效的DFA。但与此同时，得到的DFA中可能包含大量的状态，需要一个很大的传输矩阵来表示，可能影响访存性能。为了解决这个问题，我们可以采用Hopcroft算法来缩小DFA。下面是DFA算法。

|DFA最小化算法|
|---|
|输入： 由状态集S组成的DFA|
|输出： 由状态集T组成的等价DFA，其中T的状态数不超过S|
|首先将S切分为$T_0$和$T_1$，其中$T_0$表示S的非接收态，$T_1$表示S的接收态|
|重复：|
|$\quad$ $\forall T_i \in T$ :|
|$\quad\quad$ $\forall c \in \Sigma$ :|
|$\quad\quad\quad$ if $T_i \stackrel{c}{\rightarrow}$ {超过一个T中的状态}|
|$\quad\quad\quad$ then 将$T_i$切分成多个状态，满足每个状态接收c进入不超过1个状态|
|直到没有状态可以切分|

## 有限自动机的限制
有限自动机识别词法单元的复杂度为指数复杂度，不够高效。

## 使用扫描器生成器
正则表达式精准地描述了词法单元所允许的所有形式，我们可以使用一个程序来将一系列正则表达式转换成扫描器代码。这个程序也称作扫描器生成器。Flex是一种基于C语言实现的扫描器，遵从Berkeley License， 如今广泛用于类Unix系统得到C/C++实现的扫描器。使用Flex，我们需要写出扫描器的规格，其中包含正则表达式、C代码片段以及一些专业术语。Flex接收这个规格，输出可以正常编译的C代码。

```
%{
    (C Preamble Code)
%}
    (Character Classes)
%%
    (Regular Expression Rules)
%%
    (Additional Code)
```

如上是Flex文件的整体框架。其中第一部分包含放置在scanner.c开端的任意C代码，比如include文件、类型定义等，用于包含含有词法单元符号常量的文件。第二部分描述字符类，对常见的正则表达式是符号缩写。比如声明DIGIT [0-9]，这个类在后面以{DIGIT}引用。第三部分陈述了你想要匹配的词法单元的每种类型的正则表达式，伴随的是表达式匹配后就会执行的C代码段。第四部分是scanner尾部，通常是额外的辅助函数。Flex必须定义yywrap()函数，若文件结束时输入已经完成返回1，否则yywrap()函数将会打开下一个文件并返回0。

FLEX接受的正则表达式语言与前面介绍的正则表达式语言类似。主要区别在于具有特殊含义的字符（如括号、方括号和星号）需要用反斜杠转义或者套上双引号；此外，（.）可用于匹配任意的字符，有助于捕捉错误条件。下面是一个简单的示例。

```
%{
#include "token.h"
%}
DIGIT   [0-9]
LETTER  [a-zA-Z]
%%
(" " | \t | \n) /* skip whitespace */
\+          {   return TOKEN_ADD;  }
while       {   return TOKEN_WHILE;}
{LETTER}+   {   return TOKEN_IDENT;}
{DIGIT}+    {   return TOKEN_NUMBER;}
.           {   return TOKEN_ERROR;}
%%
int yywrap(){   return 1;}
```

FLEX生成扫描器代码，但并非完整的程序，因此需要写一个main函数。下面是一个基于上面例子的简单驱动程序。

```
#include "token.h"
#include <stdio.h>

extern  FILE    *yyin;
extern  int     yylex();
extern  char    *yytext;

int main()
{
    yyin = fopen("program.c", "r");
    if(!yyin)
    {
        printf("could not open program.c\n");
        return 1;
    }

    while(1)
    {
        token_t t = yylex();
        if (t == TOKEN_EOF) break;
        printf(token: %d text: %s\n", t, yytext);
    }
}
```

上面是main.c文件的内容。首先，主程序必须将它将在生成的扫描代码里面使用的符号为extern，yyin是文本将会读取的文件，yylex是实现扫描器的函数，数组yytex包含每个词法单元发现的实际文本。程序中不同地方词法单元的类型必须有一致的定义，因此在token.h文件中列出了枚举新类型token_t，如下所示。

```
typedef enum
{
    TOKEN_EOF = 0,
    TOKEN_WHILE,
    TOKEN_ADD,
    TOKEN_IDENT,
    TOKEN_NUMBER,
    TOKEN_ERROR
} token_t;
```

如下是一个Flex程序的构造过程。

```mermaid
flowchart LR
    A((scanner.flex)) --> B[Flex] --> B1((scanner.c)) --> C[Compiler] --> C1((scanner.o)) --> D[Linker] --> E((scanner.exe))
    F((main.c)) --> G[Compiler] --> G1((main.o)) --> D[Linker]
    H((token.h)) --> B1((scanner.c))
    H((token.h)) --> F((main.c))
```

## 实际考虑
### 处理关键字
### 记录原来的位置
### 清洗词法单元
### 约束词法单元
### 错误处理





