---
layout: post
title: "Programming Languages and Logic Part 1"
description: "Programming languages share a lot in common with Logic. Here i'll introduce formal semantics including Operational Sematics, Denotational Semantics and Aximatic Semantics through a simple language we made called IMP"
mathjax: true
---

### 什么是语义

什么是程序？当我们编写程序的时候，写下的只是一串串的字符而已，这些字符只是特定的语法，并不能告诉我们程序到底在干什么，于是我们就通过编译器或解释器运行程序，以此来定义程序。但是在这个过程中也会出现问题，比如编译器如果有bug，就会影响我们对程序功能的判断。

一个更好的定义程序的办法是：给程序语言的语义建立一个形式的数学定义。这种方法不仅更清晰，精确，更重要的是给严谨的形式证明提供了入口。这种方法唯一的缺点是：语义本身会非常的复杂，特别是当你想为成熟的现代语言建模的时候。

有三种传统的定义语言语义的方法：

*   **操作语义** 在抽象机器层面定义；
*   **指称语义** 在数学对象（比如函数）层面定义；
*   **公理语义** 在数理逻辑层面定义；

这三种方法在数学复杂度上各有利弊，有的好证明，有的容易实现解释器或者编译器。下面就对三种方法逐个分析；

#### 一个简单的语言：算术表达式

为了理解语义，首先让我们构思一个特别简单的语言，这个语言只包含整数的加、乘运算和赋值操作。用这个语言写的程序是一个表达式；运行这个程序就是对表达式求值。我们用下面的变量来描述语法结构：

$$
\begin{align*} x, y, z & \in Var \\ n, m & \in Int \\ e & \in Exp \end{align*}
$$

_Var_ 是程序变量的集合；_Int_ 是整数集合；_Exp_ 是表达式定义域，我们用下面的BNF语法表示 _Exp_：


$$
\begin{align*} e ::= & x \\ | & n \\ | & e_1 + e_2 \\ | & e_1 * e_2 \\ | & x:=e_1;e_2 \end{align*}
$$

\\(x:=e\_1;e\_2\\)的意思是，在求\\(e\_2\\)值之前把\\(e\_1\\)的值赋给x，整个表达式的值就是求值\\(e\_2\\)的结果。


文法（grammar）决定了语言的语法（syntax），很显然上面的文法很模糊，比如1+2*3，既可以是(1+2)*3，也可以是1+(2*3)。有很多种办法可以解决这个问题，我们可以重写一个不模糊的文法，但这样文法会变得复杂难懂；我们也可以把现有的文法进行一点小小的改动：

$$
\begin{align*} e ::= & x \\ | & n \\ | & (e_1 + e_2) \\ | & (e_1 * e_2) \\ | & x:=e_1;e_2 \end{align*}
$$

但这样文法又显得冗杂；这里，我们把语音“具体语法”和“抽象语法”分开，前者指的是怎么不模糊的把字符解析成程序语句，后者则描述了，可能模糊的程序结构，下面的文章，为了方便，使用了后者，所有的抽象语法树结构都将用带括号的表达式代替，但是注意括号本身并不是语言的一部分。

#### 操作语义

我们对表达式都有直观的感觉，比如，7+(4*2)的值是15，i:=6+1;2*3*i的值是42。下面，我们就把这种直觉给精确的表示出来。

操作语义描述了程序在抽象机器上是怎么运行的。操作语义包含small-step和large-step两种，我们先讲small-step。small-step操作语义描述了程序是怎么逐步规约的，表达式会逐步规约直到一个特定的值为止。我们称抽象机器的一个状态为配置（configuration），对于我们的语言，一个配置必须包含下面两条信息：

*   _store_：又叫做环境或状态，包含了整数和变量的映射。当程序运行时，我们会从store中找出与变量相对应的整数；有时也会更新store，比如，添加新变量，修改现有变量的映射。
*   _expression_：要规约的表达式。

我们用从Var到Int和映射表示store；用一对store和expression表示配置：

$$
\begin{align*}
Store&\triangleq{Var}\rightharpoonup{Int}\\
Config&\triangleq{Store}\times{Exp}
\end{align*}
$$

我们用’\\(\\langle\\)’和’\\(\\rangle\\)’符号表示配置，small-step操作语义在其中就是个描述从一个配置变为另一个配置的关系运算（relation）：\\(\\rightarrow \\subseteq Config \\times Config\\)。关系\\(\\rightarrow\\)描述了程序如何逐步规约，我们下面用中缀法表示关系\\(\\rightarrow\\)：给定配置\\(\\langle\\sigma\_1,e\_1\\rangle\\)和配置\\(\\langle\\sigma\_2,e\_2\\rangle\\)，如果\\((\\langle\\sigma\_1,e\_1\\rangle,\\langle\\sigma\_2,e\_2\\rangle)\\)在关系\\(\\rightarrow\\)中，我们就写成\\(\\langle\\sigma\_1,e\_1\\rangle \\rightarrow\\langle\\sigma\_2,e\_2\\rangle\\)。

用这种方法，定义语言语义的问题就化简成了定义关系\\(\\rightarrow\\)。

很显然，整数有无数个，表达式有无数个，于是配置也有无数个，配置之间的单步规约也有无数个，我们需要用有限的方式来描述无限的东西。利用推导规则，我们能简介的定义\\(\\rightarrow\\)：


$$
\frac{n=\sigma(x)}{\langle\sigma,x\rangle\rightarrow\langle\sigma,n\rangle} {\bf{V}}\scriptsize AR
$$

$$
\frac{\langle\sigma,e_1\rangle\rightarrow\langle\sigma',e_1'\rangle}{\langle\sigma,e_1+e_2\rangle\rightarrow\langle\sigma',e_1'+e_2\rangle}
{\bf{LA}\scriptsize DD}\quad \frac{\langle\sigma,e_2\rangle\rightarrow\langle\sigma',e_2'\rangle}{\langle\sigma,n+e_2\rangle\rightarrow\langle\sigma',n+e_2'\rangle}
{\bf{RA}\scriptsize DD}\quad
\frac{p=m+n}{\langle\sigma,n+m\rangle\rightarrow\langle\sigma,p\rangle}
{\bf{A}\scriptsize DD}
$$

$$
\frac{\langle\sigma,e_1\rangle\rightarrow\langle\sigma',e_1'\rangle}{\langle\sigma,x:=e1;e2\rangle\rightarrow\langle\sigma',x:=e_1';e2\rangle}
{\bf{A}\scriptsize SSGN1}\quad \frac{\sigma'=\sigma[x\mapsto{n}]}{\langle\sigma,x:=n;e_2\rangle\rightarrow\langle\sigma',e_2\rangle}
{\bf{A}\scriptsize SSGN}
$$

推导规则的意思就是如果分子成立，那么分母也成立。分子叫做前提，分母叫做结论；\\(\\sigma\[x\\mapsto{n}\]\\)意思是给store \\(\\sigma\\)添加一个x到n的映射：

\\\[f(y)= \\begin{cases} n, & \\text {if $y$=$n$} \\\\ \\sigma(y) & \\text{otherwise}\\end{cases}\\\]

那么怎么使用这些推导规则呢，我们举个例子，如果要求表达式\\(\\langle\\sigma,(foo+2)*(bar+1)\\rangle\\)的值，其中\\(\\sigma{(foo)}=4\\)，\\(\\sigma{(bar)}=3\\)：

$$
\begin{align*}
\langle\sigma,(foo+2)*(bar+1)\rangle&\rightarrow\langle\sigma,(4+2)*(bar+1)\rangle\quad&
{\bf{V}\scriptsize AR},{\bf{LA}\scriptsize DD},{\bf{LM}\scriptsize UL}\\
&\rightarrow\langle\sigma,6*(bar+1)\rangle&
{\bf{A}\scriptsize DD},{\bf{LM}\scriptsize UL}\\
&\rightarrow\langle\sigma,6*(3+1)\rangle&
{\bf{V}\scriptsize AR},{\bf{LA}\scriptsize DD},{\bf{RM}\scriptsize UL}\\
&\rightarrow\langle\sigma,6*4\rangle&
{\bf{A}\scriptsize DD},{\bf{RM}\scriptsize UL}\\
&\rightarrow\langle\sigma,24\rangle&
{\bf{M}\scriptsize UL} \end{align*}
$$

最终计算的结果是24，当机器规约结束，只有一个含有最终结果的配置，对于上文的表达式，最终配置的样式是
$\langle\sigma,n\rangle$ 。另外，
$\rightarrow$ 关系的自反、传递版本是
$\rightarrow^\*$ ，也就是说，如果能直接或多步的从
$\langle\sigma,e\rangle$规约到
$\langle\sigma',e'\rangle$，我们就写成
$\langle\sigma,e\rangle\rightarrow^\*\langle\sigma',e'\rangle$ ，所以：

$$
\langle\sigma,(foo+2)*(bar+1)\rangle\rightarrow^*\langle\sigma,24\rangle
$$

### 归纳定义与归纳证明

#### 程序属性

我们继续使用我们简单的语言：
$ e::=x\,|\,n\,|\,e_1+e_2\,|\,e_1*e_2\,|\,x:=e_1;e_2 $ ，用它描述一些程序特有的属性，并用归纳法证明这些属性。

对于一个程序语言，我们能问好多有意思的问题，比如：它是确定（deterministic）的吗？有没有无法终止的程序？求值的时候会出现哪些错误？有了前面操作语义的定义，就能准确的描述出这些问题了：

*   确定性（Determinism）：求值是确定、明确的，

$$
\forall{e}\in{Exp}.\forall{\sigma,\sigma',\sigma''}\in{Store}.\forall{e',e''}\in{Exp}
$$

$$
if\langle\sigma,e\rangle\rightarrow\langle\sigma',e'\rangle\,and\,\langle\sigma,e\rangle\rightarrow\langle\sigma'',e''\rangle\,then\,e'=e''\,and\;\sigma'=\sigma''.
$$

*   终止性（Termination）：每个对表达式的求值都能终止，

$$
\forall{e}\in{Exp}.\forall{\sigma}\in{Store}.\exists{\sigma'}\in{Store}.\exists{e'}\in{Exp}.\langle\sigma,e\rangle\rightarrow^*\langle\sigma',e'\rangle\,and\,\langle\sigma',e'\rangle\nrightarrow,
$$

$$
where\,\langle\sigma',e'\rangle\nrightarrow\,is\,shorthand\,for\,\neg(\exists\sigma''\in{Store}.\exists{e''}\in{Exp}.\langle\sigma',e'\rangle\rightarrow\langle\sigma'',e''\rangle).
$$

*   完备性（Soundness）：对每个表达式求值的结果是个整数，

$$
\forall{e}\in{Exp}.\forall\sigma\in{Store}.\exists\sigma'\in{Store}.\exists{n'}\in{Int}.\langle\sigma,e\rangle\rightarrow^*\langle\sigma',n'\rangle.
$$

悲催的是，完备性在我们的语言中不成立。比如给定一个完全没有定义过的\\(\\sigma\\)和表达式\\(i+j\\)，配置\\(\\langle\\sigma,i+j\\rangle\\)就卡住了。问题出现在i和j都是自由变量，而\\(\\sigma\\)中却没有相应的映射。

为了解决这个问题，我们定义个新的概念：良构的配置；当\\(\\sigma\\)至少由\\(e\\)中的自由变量定义的时候，就称配置\\(\\langle\\sigma,e\\rangle\\)是良构的，下面就是表达式中自由变量集合的定义：

$$
\begin{align*}
fvs(x)&\triangleq{\{x\}}\\
fvs(n)&\triangleq\{\}\\
fvs(e_1+e_2)&\triangleq{fvs(e_1)\cup{fvs(e_2)}}\\
fvs(e_1*e_2)&\triangleq{fvs(e_1)\cup{fvs(e_2)}}\\
fvs(x:=e_1;e_2)&\triangleq{fvs(e_1)\cup{(fvs(e_2)\setminus\{x\})}}
\end{align*}
$$

*   Progress：给定表达式e和store\\(\\,\\sigma\\)，其中e的自由变量都包含在\\(\\sigma\\)的定义域中，e要么是个整数，要么还能往下推，

\\\[\\forall{e}\\in{Exp}.\\forall{\\sigma}\\in{Store}.\\\\fvs(e)\\subseteq{dom(\\sigma)}\\implies{e\\in{Int}\\lor(\\exists{e'}\\in{Exp}.\\exists\\sigma'\\in{Store}.\\langle\\sigma,e\\rangle\\to\\langle\\sigma',e'\\rangle)} \\\]

*   Preservation：良构的配置，单步求值后还是良构的，

\\\[\\forall{e,e'}\\in{Exp}.\\forall{\\sigma,\\sigma'}\\in{Store}.\\\\fvs(e)\\subseteq{dom(\\sigma)}\\land{\\langle\\sigma,e\\rangle\\to\\langle\\sigma',e'\\rangle}\\implies{fvs(e')\\subseteq{dom(\\sigma')}}\\\]

#### 归纳集

归纳法在PLT中是个重要的概念。归纳定义集是由有限的公理和归纳规则组成的。其中，公理的结构：
$\frac{}{a\in{A}}$，表明了a在A集合中；归纳规则的结构：$ \frac{a_1\in{A}\quad...\quad a_n\in{A}}{a\in{A}}$ ，表明了如果\\(a\_1,...,a\_n\\)是A的元素，那a也是A的元素。集合A是由所有能从公理和归纳规则推导出的推论组成的，换句话说，对于A的每个元素a，一定能构建一个有限的证明树证明\\(a\\in{A}\\)。举个归纳集的例子，下面是自然数的归纳定义：

$$
\frac{}{0\in{N}}\quad\frac{n\in{N}}{succ(n)\in{N}}
$$

文法所描述的集合就是归纳集。比如用我们简单的语言：
$e::=x\,|\,n\,|\,e_1+e_2\,|\,e_1*e_2\,|\,x:=e_1;e_2$ 就可以用下面的归纳集定义，此归纳集包含了2个公理和3个归纳规则：

$$
\frac{}{x\in{Exp}}\qquad\qquad\frac{}{n\in Exp}\\\frac{e_1\in Exp\quad e_2\in Exp}{e_1+e_2\in Exp}\quad\frac{e_1\in Exp\quad e_2\in Exp}{e_1*e_2\in Exp}\quad\frac{e_1\in Exp\quad e_2\in Exp}{x:=e_1;e_2\in Exp}
$$

#### Progress的归纳证明

还记得上面讲得结构归纳法吗，现在就用这种方法证明Soundness的Progress属性：

_Proof_. e为一个表达式，需要通过对e进行结构归纳证明：

\\\[\\forall{\\sigma}\\in{Store}.fvs(e)\\subseteq{dom(\\sigma)}\\implies{e\\in{Int}\\lor(\\exists{e'}.\\exists\\sigma'\\in.\\langle\\sigma,e\\rangle\\to\\langle\\sigma',e'\\rangle)} \\\]

*   Case \\(e=x\\)：由fvs的定义，\\(\\therefore{fvs(x)=\\{x\\}}\\)，\\(\\because{fvs(e)\\subseteq{dom(\\sigma)}}\\)，\\(\\therefore{\\{x\\}\\subseteq{dom(\\sigma)}}\\)，\\(\\therefore{x\\subseteq{dom(\\sigma)}}\\)，设\\(n=\\sigma(x)\\)。由Var公理可得\\(\\langle\\sigma,x\\rangle\\rightarrow\\langle\\sigma,n\\rangle\\)。Case finished;
*   Case \\(e=n\\)：直接有\\(e\\in{Int}\\)。Case finished;
*   Case \\(e=e\_1+e\_2\\)：首先设\\(P(e\_1),P(e\_2)\\)如下，

$$
P(e_1)=\forall{\sigma}\in{Store}.fvs(e_1)\subseteq{dom(\sigma)}\implies{e_1\in{Int}\lor(\exists{e'}.\exists\sigma'.\langle\sigma,e_1\rangle\to\langle\sigma',e'\rangle)}
$$

$$
P(e_2)=\forall{\sigma}\in{Store}.fvs(e_2)\subseteq{dom(\sigma)}\implies{e_2\in{Int}\lor(\exists{e'}.\exists\sigma'.\langle\sigma,e_2\rangle\to\langle\sigma',e'\rangle)}
$$

然后只需证明

$$
P(e_1+e_2)=\forall{\sigma}\in{Store}.fvs(e_1+e_2)\subseteq{dom(\sigma)}\implies{e_1+e_2\in{Int}\lor(\exists{e'}.\exists\sigma'.\langle\sigma,e_1+e_2\rangle\to\langle\sigma',e'\rangle)}
$$

即可:
    
*   Subcase $e_1=n_1\land e_2=n_2$ ：由ADD公理，即可得到 $\langle\sigma,n_1+n_2\rangle\rightarrow\langle\sigma,p\rangle$ ，其中
    $p=n_1+n_2$ 。

*   Subcase \\(e1\\notin Int\\)：由fvs的定义可得，\\(\\therefore{fvs(e\_1)\\subseteq{fvs*(e\_1+e\_2)}\\subseteq{dom(\\sigma)}}\\)，又\\(\\because{P(e\_1)}\\)，\\(\\therefore{\\langle\\sigma,e\_1\\rangle\\to\\langle\\sigma',e'\\rangle}\\)，由LADD公理可得\\(\\langle\\sigma,e\_1+e\_2\\rangle\\to\\langle\\sigma',e'+e\_2\\rangle\\)。

*   Subcase \\(e\_1=n1\\land{e\_2\\notin{Int}}\\)：由fvs的定义可得，\\(\\therefore{fvs(e\_2)\\subseteq{fvs*(e\_1+e\_2)}\\subseteq{dom(\\sigma)}}\\)，又\\(\\because{P(e\_2)}\\)，\\(\\therefore{\\langle\\sigma,e\_2\\rangle\\to\\langle\\sigma',e'\\rangle}\\)，由LADD公理可得\\(\\langle\\sigma,e\_1+e\_2\\rangle\\to\\langle\\sigma',e\_1+e'\\rangle\\)。Case finished;

*   Case \\(e=e\_1*e\_2\\)：同上个Case证明方法大致相同；
*   Case \\(e=x:=e\_1;e2\\)：同样，我们先假设\\(P(e\_1),P(e\_2)\\)如下，

$$
P(e_1)=\forall{\sigma}\in{Store}.fvs(e_1)\subseteq{dom(\sigma)}\implies{e_1\in{Int}\lor(\exists{e'}.\exists\sigma'.\langle\sigma,e_1\rangle\to\langle\sigma',e'\rangle)}
$$

$$
P(e_2)=\forall{\sigma}\in{Store}.fvs(e_2)\subseteq{dom(\sigma)}\implies{e_2\in{Int}\lor(\exists{e'}.\exists\sigma'.\langle\sigma,e_2\rangle\to\langle\sigma',e'\rangle)}
$$

然后只需证明

\\\[P(e=x:=e\_1;e2)=x:=e\_1;e\_2\\in{Int}{\\lor(\\exists{e'}.\\exists\\sigma'.\\langle\\sigma,x:=e\_1;e\_2\\rangle\\to\\langle\\sigma',e'\\rangle)}\\\]

即可:

*   Subcase \\(e\_1=n\_1\\)：由ASSGN公理可得，\\(\\langle\\sigma,x:=n\_1;e\_2\\rangle \\to \\langle\\sigma',e\_2\\rangle\\)， 其中\\(\\sigma'=\\sigma\[x\\mapsto{n\_1}\]\\)。
*   Subcase \\(e\_1\\notin{Int}\\)：由fvs的定义可得，\\(\\therefore{fvs(e\_1)\\subseteq{fvs(x:=e\_1;e\_2)}\\subseteq{dom(\\sigma)}}\\)，\\(\\because{\\langle\\sigma,e\_1\\rangle\\to\\langle\\sigma',e'\\rangle}\\)，\\(\\therefore{\\langle\\sigma,x:=e\_1;e\_2\\rangle\\to\\langle\\sigma',x:=e\_1';e\_2\\rangle}\\)。Case finished。证明完毕。

### Large-step 操作语义

我们已经用求值关系\\(\\rightarrow\\subseteq{Config}\\times{Config}\\)定义了small-step操作语义，现在我们就讲讲另外一个操作语义–Large-step操作语义。不同于small-step语义，large-step直接输出对表达式求值的最终结果。我们用关系\\(\\Downarrow\\)来定义large-step语义：

\\\[\\Downarrow\\subseteq(Store\\times{Exp})\\times(Store\\times{Int})\\\]

我们用\\(\\langle\\sigma,e\\rangle\\Downarrow\\langle\\sigma',n\\rangle\\)来表示\\(((\\sigma,e),(\\sigma',n))\\in\\Downarrow\\)，换句话说就是，表达式e和store\\(\\,\\sigma\\)一步求值到最终store\\(\\,\\sigma'\\)和整数n。同样，large-step语义也能用归纳集表示：

\\\[ \\frac{}{\\langle\\sigma,n\\rangle\\Downarrow\\langle\\sigma,n\\rangle} {\\bf{I}\\scriptsize{NT}}\\qquad \\frac{n=\\sigma(x)}{\\langle\\sigma,x\\rangle\\Downarrow\\langle\\sigma,n\\rangle} {\\bf{V}\\scriptsize{AR}} \\\]

\\\[ \\frac{\\langle\\sigma,e\_1\\rangle\\Downarrow\\langle\\sigma',n\_1\\rangle\\quad\\langle\\sigma',e\_2\\rangle\\Downarrow\\langle\\sigma'',n\_2\\rangle\\quad{n=n\_1+n\_2}}{\\langle\\sigma,e\_1+e\_2\\rangle\\Downarrow\\langle\\sigma'',n\\rangle} {\\bf{A}\\scriptsize{DD}} \\\]

\\\[ \\frac{\\langle\\sigma,e\_1\\rangle\\Downarrow\\langle\\sigma',n\_1\\rangle\\quad\\langle\\sigma',e\_2\\rangle\\Downarrow\\langle\\sigma'',n\_2\\rangle\\quad{n=n\_1\\times{n\_2}}}{\\langle\\sigma,e\_1*e\_2\\rangle\\Downarrow\\langle\\sigma'',n\\rangle} {\\bf{M}\\scriptsize{UL}} \\\]

\\\[ \\frac{\\langle\\sigma,e\_1\\rangle\\Downarrow\\langle\\sigma',n\_1\\rangle\\quad\\langle\\sigma'\[x\\mapsto{n\_1}\],e\_2\\rangle\\Downarrow\\langle\\sigma'',n\_2\\rangle}{\\langle\\sigma,x:=e\_1;e\_2\\rangle\\Downarrow\\langle\\sigma'',n\_2\\rangle} {\\bf{A}\\scriptsize{SSGN}} \\\]

下面的证明树显示的是\\(\\langle\\sigma,foo:=3;foo*bar\\rangle\\)的求值过程，其中\\(\\sigma(bar)=7,\\sigma'=sigma\[foo\\mapsto3\]\\)，21是最后的结果：

$$
\cfrac{\cfrac{}{\langle\sigma,3\rangle\Downarrow\langle\sigma,3\rangle}{\bf{I}\scriptsize{NT}}\quad\cfrac{\cfrac{}{\langle\sigma',foo\rangle\Downarrow\langle\sigma',3\rangle}{\bf{V}\scriptsize{AR}}\quad\cfrac{}{\langle\sigma',bar\rangle\Downarrow\langle\sigma',7\rangle}{\bf{V}\scriptsize{AR}}}{\langle\sigma',foo*bar\rangle\Downarrow\langle\sigma',21\rangle}{\bf{M}\scriptsize{UL}}}{\langle\sigma,foo:=3;foo*bar\rangle\Downarrow\langle\sigma',21\rangle}{\bf{A}\scriptsize{SSGN}}
$$

从这棵树可以看出，对large-step证明树进行深度优先遍历的一个结果就是small-step中的一串一小步求值过程。

那么small-step和large-step语义是否是等价的呢，答案是肯定的：

\\\[Theorem(文法等价).对所有的表达式e，store\\,\\sigma和\\,\\sigma'，还有整数n\\\\ 都有：\\langle\\sigma,e\\rangle\\Downarrow\\langle\\sigma',n\\rangle\\iff\\langle\\sigma,e\\rangle\\to^*\\langle\\sigma',n\\rangle\\\]

具体证明不是很复杂，同样用到了结构归纳，有兴趣的读者可以尝试一下。

### 一个简单的指令式语言：IMP

#### IMP语言（A simple imperative language）

显然上面的算术表达式语言有点过于简单，现在我们就要考虑定义一个更加真实的程序语言，一个有if和while控制结构的语言，这个简单的指令式语言–IMP的语法如下：

$$
\begin{align*}
arithmetic\;expressions\quad&a\in{Aexp}\quad{a::=\,x\,|\,n\,|\,a_1+a_2\,|\,a_1\times{a_2}}\\
boolean\;expressions\quad&b\in{Bexp}\quad{b::=\,true\,|\,false\,|\,a_1\lt a_2}\\
commands\quad&c\in{Com}\quad{c::=\,skip\,|\,x:=a\,|\,c_1;c_2\,|if\;b\;then\,c_1\,else\,c_2\,|\,while\;b\;do\;c}
\end{align*}
$$
#### IMP的small-step操作语义

我们先给出IMP的small-step操作语义。这个语言的配置：\\(\\langle{c},\\sigma\\rangle\\)，\\(\\langle\\sigma,b\\rangle\\)，\\(\\langle\\sigma,a\\rangle\\)；求值后的最终配置：\\(\\langle\\sigma,skip\\rangle,\\langle\\sigma,true\\rangle\\)，\\(\\langle\\sigma,false\\rangle\\)，\\(\\langle\\sigma,n\\rangle\\)。分别对应算术表达式、布尔表达式和命令，我们定义三种不同的small-step操作关系：

$$
\begin{align*}
\to_{Com}&\subseteq{Com\times{Store}\times{Com}\times{Store}}\\
\to_{Bexp}&\subseteq{Bexp\times{Store}\times{Bexp}\times{Store}}\\
\to_{Aexp}&\subseteq{Aexp\times{Store}\times{Aexp}\times{Store}}
\end{align*}
$$

为了简洁，这三个关系统一用\\(\\to\\)表示；注意其中算术表达式没有赋值操作了，算术表达式和布尔表达式不会更新store。

**Arithmetic expressions**

\\\[\\frac{n=\\sigma(x)}{\\langle\\sigma,x\\rangle\\to\\langle\\sigma,n\\rangle}\\\]

\\\[ \\frac{\\langle\\sigma,a\_1\\rangle\\to\\langle\\sigma,a\_1'\\rangle}{\\langle\\sigma,a\_1+a\_2\\rangle\\to\\langle\\sigma,a\_1'+a\_2\\rangle}\\quad \\frac{\\langle\\sigma,a\_2\\rangle\\to\\langle\\sigma,a\_2'\\rangle}{\\langle\\sigma,n+a\_2\\rangle\\to\\langle\\sigma,n+a\_2'\\rangle}\\quad \\frac{p=n+m}{\\langle\\sigma,n+m\\rangle\\to\\langle\\sigma,p\\rangle} \\\]

\\\[ \\frac{\\langle\\sigma,a\_1\\rangle\\to\\langle\\sigma,a\_1'\\rangle}{\\langle\\sigma,a\_1\\times{a\_2}\\rangle\\to\\langle\\sigma,a\_1'\\times{a\_2}\\rangle}\\quad \\frac{\\langle\\sigma,a\_2\\rangle\\to\\langle\\sigma,a\_2'\\rangle}{\\langle\\sigma,n\\times{a\_2}\\rangle\\to\\langle\\sigma,n\\times{a\_2'}\\rangle}\\quad \\frac{p=n\\times{m}}{\\langle\\sigma,n\\times{m}\\rangle\\to\\langle\\sigma,p\\rangle} \\\]

**Boolean expressions**

\\\[ \\frac{\\langle\\sigma,a\_1\\rangle\\to\\langle\\sigma,a\_1'\\rangle}{\\langle\\sigma,a\_1\\lt a\_2\\rangle\\to\\langle\\sigma,a\_1'<a\_2\\rangle}\\qquad \\frac{\\langle\\sigma,a\_2\\rangle\\to\\langle\\sigma,a\_2'\\rangle}{\\langle\\sigma,n<{a\_2}\\rangle\\to\\langle\\sigma,n<a\_2'\\rangle} \\\]

\\\[ \\frac{n<{m}}{\\langle\\sigma,n<{m}\\rangle\\to\\langle\\sigma,true\\rangle}\\qquad \\frac{n\\ge{m}}{\\langle\\sigma,n<{m}\\rangle\\to\\langle\\sigma,false\\rangle} \\\]

**Commands**

\\\[ \\frac{\\langle\\sigma,e\\rangle\\to\\langle\\sigma,e'\\rangle}{\\langle\\sigma,x:=e\\rangle\\to\\langle\\sigma,x:=e'\\rangle}\\qquad \\frac{}{\\langle\\sigma,x:=n\\rangle\\to\\langle\\sigma\[x\\mapsto{n}\],skip\\rangle} \\\]

\\\[ \\frac{\\langle\\sigma,c\_1\\rangle\\to\\langle\\sigma',c\_1'\\rangle}{\\langle\\sigma,c\_1;c\_2\\rangle\\to\\langle\\sigma',c\_1';c\_2\\rangle}\\qquad \\frac{}{\\langle\\sigma,skip;c\_2\\rangle\\to\\langle\\sigma,c\_2\\rangle} \\\]

对于if命令，先求值测试表达式直到true或false，然后再选择正确的分支：

\\\[ \\frac{\\langle\\sigma,b\\rangle\\to\\langle\\sigma,b'\\rangle}{\\langle\\sigma,if\\;b\\;then\\;c\_1\\;else\\;c\_2\\rangle\\to\\langle\\sigma,if\\;b'\\;then\\;c\_1\\;else\\;c\_2\\rangle} \\\]

\\\[ \\frac{}{\\langle\\sigma,if\\;true\\;then\\;c\_1\\;else\\;c\_2\\rangle\\to\\langle\\sigma,c\_1\\rangle}\\qquad \\frac{}{\\langle\\sigma,if\\;false\\;then\\;c\_1\\;else\\;c\_2\\to\\langle\\sigma,c\_2\\rangle} \\\]

对于while命令，上面的策略就不行了（Why？）。我们可以使用下面的规则，你可以把循环想象成把if命令一层一层展开：

\\\[\\frac{}{\\langle\\sigma,while\\;b\\;do\\;c\\rangle\\to\\langle\\sigma,if\\;b\\;then\\,(c;while\\;b\\;do\\;c)\\,else\\;skip\\rangle}\\\]

现在我们就能利用这些规则来运行程序了，比如（其中W是\\(while\\;foo<{4}\\;do\\;foo:=foo+5\\)的缩写）：

$$
\begin{align*}
& \langle\sigma,foo:=3;while\;foo<{4}\;do\;foo:=foo+5\rangle\\
\to&\langle\sigma',skip;while\;foo<{4}\;do\;foo:=foo+5\rangle\quad{(其中\sigma'=\sigma[foo\mapsto3])}\\
\to&\langle\sigma',while\;foo<{4}\;do\;foo:=foo+5\rangle\\
\to&\langle\sigma',if\;foo<{4}\;then\;(foo:=foo+5;W)\;else\;skip\rangle\\
\to&\langle\sigma',if\;3<{4}\;then\;(foo:=foo+5;W)\;else\;skip\rangle\\
\to&\langle\sigma',if\;true\;then\;(foo:=foo+5;W)\;else\;skip\rangle\\
\to&\langle\sigma',foo:=foo+5;while\;foo<{4}\;do\;foo:=foo+5\rangle\\
\to&\langle\sigma',foo:=3+5;while\;foo<{4}\;do\;foo:=foo+5\rangle\\
\to&\langle\sigma',foo:=8;while\;foo<{4}\;do\;foo:=foo+5\rangle\\
\to&\langle\sigma'',while\;foo<{4}\;do\;foo:=foo+5\rangle\quad{(其中\sigma''=\sigma'[foo\mapsto8])}\\
\to&\langle\sigma'',if\;foo<{4}\;then\;(foo:=foo+5;W)\;else\;skip\rangle\\
\to&\langle\sigma'',if\;8<{4}\;then\;(foo:=foo+5;W)\;else\;skip\rangle\\
\to&\langle\sigma'',if\;false\;then\;(foo:=foo+5;W)\;else\;skip\rangle\\
\to&\langle\sigma'',skip\rangle
\end{align*}
$$

#### IMP的large-step操作语义

同样对算术表达式、布尔表达式和命令我们定义以下不同的large-step求值关系，其中算数表达式关系把表达式和store映射为整数值；布尔表达式的最终值为\\(Bool=\\{true,false\\}\\)；命令的最终值为一个store：

$$
\begin{align*}
\Downarrow_{Aexp}&\subseteq{Aexp\times{Store}\times{Int}}\\
\Downarrow_{Bexp}&\subseteq{Bexp\times{Store}\times{Bool}}\\
\Downarrow_{Com}&\subseteq{Com\times{Store}\times{Store}}
\end{align*}
$$

同样，为了简化，对三种关系我们都用\\(\\Downarrow\\)表示；\\(\\Downarrow\\)也使用中缀表示法：\\(\\langle\\sigma,c\\rangle\\Downarrow\\sigma'\\)的意思就是\\((c,\\sigma,\\sigma')\\in\\Downarrow\_{Com}\\)。

**Arithmetic expressions**

\\\[ \\frac{}{\\langle\\sigma,n\\rangle\\Downarrow{n}}\\qquad \\frac{\\sigma(x)=n}{\\langle\\sigma,x\\rangle\\Downarrow{n}} \\\]

\\\[ \\frac{\\langle\\sigma,a\_1\\rangle\\Downarrow{n\_1}\\quad\\langle\\sigma,a\_2\\rangle\\Downarrow{n\_2}\\quad{n=n\_1+n\_2}}{\\langle\\sigma,a\_1+a\_2\\rangle\\Downarrow{n}}\\quad \\frac{\\langle\\sigma,a\_1\\Downarrow{n\_1}\\rangle\\quad\\langle\\sigma,a\_2\\rangle\\Downarrow{n\_2}\\quad{n=n\_1\\times{n\_2}}}{\\langle\\sigma,a\_1\\times{a\_2}\\rangle\\Downarrow{n}} \\\]

**Boolean expressions**

\\\[ \\frac{}{\\langle\\sigma,true\\rangle\\Downarrow{true}}\\qquad \\frac{}{\\langle\\sigma,false\\rangle\\Downarrow{false}} \\\]

\\\[ \\frac{\\langle\\sigma,a\_1\\rangle\\Downarrow{n\_1}\\quad\\langle\\sigma,a\_2\\rangle\\Downarrow{n\_2}\\quad{n=n\_1<n\_2}}{\\langle\\sigma,a\_1<a\_2\\rangle\\Downarrow{true}}\\quad \\frac{\\langle\\sigma,a\_1\\Downarrow{n\_1}\\rangle\\quad\\langle\\sigma,a\_2\\rangle\\Downarrow{n\_2}\\quad{n=n\_1\\ge{n\_2}}}{\\langle\\sigma,a\_1<a\_2\\rangle\\Downarrow{false}} \\\]

**Commands**

\\\[ {\\bf{S}\\scriptsize{KIP}} \\frac{}{\\langle\\sigma,skip\\rangle\\Downarrow\\sigma}\\quad {\\bf{A}\\scriptsize{SSGN}} \\frac{\\langle\\sigma,e\\rangle\\Downarrow{n}}{\\langle\\sigma,x:=e\\rangle\\Downarrow\\sigma\[x\\mapsto{n}\]} \\quad{\\bf{S}\\scriptsize{EQ}} \\frac{\\langle\\sigma,c\_1\\rangle\\Downarrow\\sigma'\\quad\\langle\\sigma',c\_2\\rangle\\Downarrow\\sigma''}{\\langle\\sigma,c\_1;c\_2\\rangle\\Downarrow\\sigma''} \\\]

\\\[ {\\bf{I}\\scriptsize{F-T}} \\frac{\\langle\\sigma,b\\rangle\\Downarrow{true}\\quad\\langle\\sigma,c\_1\\rangle\\Downarrow\\sigma'}{\\langle\\sigma,if\\;b\\;then\\;c\_1\\;else\\;c\_2\\rangle\\Downarrow\\sigma'}\\quad {\\bf{I}\\scriptsize{F-F}} \\frac{\\langle\\sigma,b\\rangle\\Downarrow{false}\\quad\\langle\\sigma,c\_2\\rangle\\Downarrow\\sigma'}{\\langle\\sigma,if\\;b\\;then\\;c\_1\\;else\\;c\_2\\rangle\\Downarrow\\sigma'} \\\]

\\\[ {\\bf{W}\\scriptsize{HILE-F}} \\frac{\\langle\\sigma,b\\rangle\\Downarrow{false}}{\\langle\\sigma,while\\;b\\;do\\;c\\rangle\\Downarrow\\sigma} \\\]

\\\[ {\\bf{W}\\scriptsize{HILE-T}} \\frac{\\langle\\sigma,b\\rangle\\Downarrow{true}\\quad\\langle\\sigma,c\\rangle\\Downarrow\\sigma'\\quad\\langle\\sigma',while\\;b\\;do\\;c\\rangle\\Downarrow\\sigma''}{\\langle\\sigma,while\\;b\\;do\\;c\\rangle\\Downarrow\\sigma''} \\\]

有意思的是，large-step中的while命令就没有像small-step中的那样依赖if命令定义（Why？）。

### 小结：IMP的一个实现

如果你能坚持到这里，好吧，要么你是个爱学习的好孩子，要么你就一直边看边问：“这些都有什么用！？”。讲了这么多理论，是时候搞点实际的东西了。这一节，我们会实现算术表达式语言和这个IMP语言。

因为这篇文章是针对程序语言的，这里并不会针对编译原理展开讨论，如果你对编译原理不太熟悉，也不要紧，这里的实现所使选用的语言是OCaml，OCaml自带lexer和parser generator，词法分析就已经差不多了，剩下的就是语法分析和求值eval了；另外，OCaml的语法也非常简单，基本一看就懂，不懂的看看文档也就差不多了。

当然你也可以选定自己的语言，然后实现IMP，基本原理都差不多，没有lexer可以用正则表达式凑合，没有parser可以recursive descendant parser堆一个嘛。这个IMP的实现依据的是large-step语义，lexer.mll负责把文件输入变成token流，parser.mly负责接受token流输出ast.ml里定义的ast，eval.ml负责接受ast并输出求值ast的结果。下面分别是变量、算术表达式、布尔表达式和命令的类型定义，特别的直观：

```ocaml
type var = string

type aexp =
  | Int of int                (* n *)
  | Var of var                (* x *)
  | Plus of aexp * aexp       (* a0 + a1 *)
  | Times of aexp * aexp      (* a0 * a1 *)

type bexp =
  | True
  | False
  | Less of aexp * aexp       (* a0 < a1 *)

type com =
  | Seq of com * com          (* c0 ; c1 *)
  | Assign of var * aexp      (* x := a *)
  | Aexp of aexp              (* a *)
  | If of bexp * com * com    (* if b then c0 else c1 *)
  | While of bexp * com       (* while b do c *)
```  

需要实现的就是下面三个求值方法，分别求值算术表达式、波尔表达式和命令，分别负责输出一个整数、一个布尔值和一个store：

```ocaml
let rec eval_aexp (s,e) : int = 
  match e with
    | Int(n) -> n
    | Var(x) -> lookup s x
    | Plus(a1,a2) -> eval_aexp (s,a1) + eval_aexp (s,a2)
    | Times(a1,a2) -> eval_aexp (s,a1) * eval_aexp (s,a2)

let rec eval_bexp (s,e) : bool =
  match e with
    | True -> true
    | False -> false
    | Less(a1,a2) -> eval_aexp (s,a1) < eval_aexp (s,a2)

let rec eval_com (s,e) : store =
  match e with
    | Assign(x, a1) ->
      let n1 = eval_aexp (s,a1) in
      update s x n1
    | Seq(c1,c2) ->
      let s' = eval_com (s,c1) in
      eval_com (s', c2)
    | Aexp a1 ->
      s
    | If(b1,c1,c2) ->
      if eval_bexp (s,b1) then eval_com (s,c1) else eval_com (s,c2)
    | While(b1,c1) ->
      if eval_bexp (s,b1) then eval_com (s,Seq(c1,While(b1,c1))) else s
```

为了使IMP更合理和更好实现，我扩展了IMP的语法，最终你要可以解释运行下面的程序，命令由‘;’隔开，while循环体由‘{’、‘}’包含：

```
x := 2;
y := x + 3;
if y< 6 then z:=x else z:=y;
while y <6 do {
  x:=x+1;
  y:=y+1;
}
x := x * x;
```

### IMP的属性

#### 文法等价

IMP的small-step语义和large-step语义是等价的：

\\\[ Therom. \\forall{c\\in{Com}\\land\\sigma,\\sigma'\\in{Store}}.\\\\ \\langle\\sigma,c\\rangle\\to^*\\langle\\sigma',skip\\rangle\\iff\\langle\\sigma,c\\rangle\\Downarrow\\sigma' \\\]

#### Non-Termination

给定命令c的起始状态\\(\\sigma\\)，这个命令可能终止，输出\\(\\sigma'\\)，但也有可能发散，永远不会终止，永远不会输出最终状态。比如：

\\\[while\\;true\\;do\\;foo:=foo+1\\\]

永远发散；

\\\[while\\;0<i\\;do\\;i:i+1\\\]

只有当i在初始\\(\\sigma\\)中值大于0时才会发散。如果\\(\\langle\\sigma,c\\rangle\\)是一个发散的配置，那么不存在\\(\\sigma\\)使得：

\\\[\\langle\\sigma,c\\rangle\\Downarrow\\sigma'\\lor\\langle\\sigma,c\\rangle\\to^*\\langle\\sigma',skip\\rangle.\\\]

在small-step语义中，发散的配置生成了无限的序列：

\\\[\\langle\\sigma,c\\rangle\\to\\langle\\sigma\_1,c\_1\\rangle\\to\\langle\\sigma\_2,c\_2\\rangle\\to\\dots\\\]

所以，small-step语义让我们有机会表达和证明程序可能发散的属性，large-step语义则不能。

#### Determinism

IMP的small-step语义和large-step语义都是确定的。每个IMP命令c和其初始状态\\(\\sigma\\)最多求值到一个最终的store：

\\\[ Theorem. \\forall{c\\in{Com}\\land\\sigma\_1,\\sigma\_2\\in{Store}}.\\\\ \\langle\\sigma,c\\rangle\\Downarrow\\sigma\_1\\land\\langle\\sigma,c\\rangle\\Downarrow\\sigma\_2\\implies\\sigma\_1=\\;\\sigma\_2 \\\]

下一章我们会探索指称语义和公理语义。