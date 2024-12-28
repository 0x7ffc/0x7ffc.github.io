---
layout: post
title: "Lambda Caculus - Part 1"
description: "Part 1 of an introduction to Lambda Caculus which includes lambda caculus grammar and syntax, evaluation rules, De Bruijn encoding, Church numerals and others."
lang: zh
mathjax: true
---

### Lambda Calculus

上个世纪30年代，Alonzo Church和Stephen Cole Kleene为了清晰、简洁的描述函数发明了λ演算（Lambda caculus），一个函数也是值，并且是第一公民的系统。在λ演算中，函数可以当成参数传给其他函数，函数也可以返回函数，而且一切值只有函数。“lambda”（λ）只是个用在定义函数时的符号，“calculus”意为这是一个操作符号的计算系统，函数是一个给定特定参数输出值得规则，比如\\(g(x)=x^3-2x^2+5x-6\\)。

λ演算是一切的开端，要说λ演算的最主要贡献：

*   函数式语言。有很多语言是直接基于λ演算的，比如Lisp、ML。
*   Proof assistant（比如Coq、Agda、Isabelle）。
*   Concurrent Model。

如果你对这些领域有兴趣，那λ演算是逃不了的。这一章就主要介绍λ演算。

### λ演算的语法

纯λ演算语法只包含三个东西：函数抽象（称作abstraction）、变量和函数应用（应用参数到函数上，称作application）。如果再加上额外的类型或函数（比如整型、加法），就叫applied λ演算。下面为了简化，我们假设λ演算里有整数和算术运算（后面会讲如何用纯λ演算定义整数和加法运算）。

纯λ演算的语法如下：


$$
\begin{align*}
e::=&x \qquad\qquad & variable\\
| & \lambda x.e \qquad\qquad & abstraction\\
| & e_1\;e_2 \qquad\qquad & application
\end{align*}
$$

\\(\\lambda x.e\\)是个函数：x是参数，表达式e是函数体。此函数并没有名字。比如\\(\\lambda x.x^2\\)是个求x平方的函数。

\\(e\_1\\;e\_2\\)是个把\\(e\_2\\)当做参数传给函数\\(e\_1\\)的函数应用。比如\\((\\lambda x.x^2)5\\)的值为25。

仅仅利用这三个规则，就能写出完整的程序，做完整的数学运算，多么简洁的计算系统。

下面是一些λ演算的例子：

\\\[ \\begin{align\*} &\\lambda x.x \\qquad & \\text 一个返回自己的identity函数 \\\\ &\\lambda x.(f(g\\;x))) \\qquad & \\text 一个函数抽象 \\\\ &(\\lambda x.x)42 \\qquad & \\text 一个应用到identity函数的函数应用 \\\\ &\\lambda y.\\lambda x.x \\qquad & \\text 一个忽略输入并返回identity函数的函数 \\end{align\*} \\\]

λ表达式是向最右扩展的，意味着\\(\\lambda x.x \\lambda y.y\\)是\\(\\lambda x.x(\\lambda y.y)\\)，而不是\\((\\lambda x.x)(\\lambda y.y)\\)；函数应用是左结合的，意味着\\(e\_1\\;e\_2\\;e\_3\\)是\\((e\_1\\;e\_2)e\_3\\)。

表达式中的变量要么是绑定的（bound），要么是自由的（free）：对于变量x，如果函数定义的参数中包含了x，那么x就是绑定的，否则就是自由的。如果一个表达式所有的变量都是绑定的，那么就称之为闭合的。比如\\(\\lambda x.(x(\\lambda y.y\\;a)x)y\\)中，所有的x都是绑定的，第一个y是绑定的，最后的一个y是自由的，a也是自由的。很显然，一个合理的程序应该不包含自由变量，也就是说，一个良构的程序应该是一个闭合的λ表达式。可以把λ看成一个把变量绑定在一个作用域的绑定操作符，比如\\(\\lambda x.e\\)中，x被绑定在了e中。绑定变量的名字并不重要，\\(\\lambda x.x\\)和\\(\\lambda y.y\\)是同一个函数，像这种\\(e\_1\\)和\\(e\_2\\)只有绑定变量名不同的叫作\\(\\alpha\\)变换，通常写成\\(e\_1=\_\\alpha e\_2\\)，这是λ演算中一条重要的规约规则。

在λ演算中，函数是值，能当参数也能当返回值，在纯λ演算中，每个值也是函数，每个结果都是函数。比如\\(\\lambda f.f\\;42\\)是个接受函数并应用在42上的函数；\\(\\lambda v.\\lambda f.(f\\;v)\\)返回了一个应用参数v到函数自身的函数。

介绍完了\\(\\alpha\\)变换，下面介绍λ演算中另外一个重要的规约规则–\\(\\beta\\)规约

### λ演算的语义

函数应用\\((\\lambda x. e\_1)e\_2\\)可以看成把\\(e\_1\\)单独拿出来，并把其中所有的自由x替换成\\(e\_2\\)，比如\\((\\lambda x.x^2)5\\)就是\\(5^2\\)，我们把这种替换写成\\(e\_1\\{e\_2/x\\}\\)，这种替换叫作\\(\\beta归约\\)。给定一个表达式，根据不同的规约顺序，可能有多种\\(\\beta\\)规约的方式，比如(λx. x+x)((λy. y) 5)，可以归约成((λy. y) 5)+((λy. y) 5) 或者 (λx. x+x) 5，不同的归约方式意味着不同的语义：Call-by-name和Call-by-value。

#### Call-by-value

Call-by-value(CBV)语义保证了应用函数时参数都是值(value)。在纯λ演算中，唯一的值是函数，所有的抽象都是函数，所有的抽象都是值（在包含整数和算数运算的applied λ演算中，整数也是值），简单的讲，不能再简化或者求值的表达式都是值。

下面是CBV的small-step操作语义：

\\\[ \\frac{e\_1\\to e\_1'}{e\_1 e\_2\\to e\_1'e\_2} \\quad \\frac{e\\to e'}{v\\;e\\to v\\;e'} \\quad {\\bf{\\beta}\\scriptsize{-REDUCTION}} \\frac{}{(\\lambda x.e)v\\to e\\{v/x\\}} \\\]

从上面的语义可以看出CBV的求值策略如下：给定应用\\(e\_1\\;e\_2\\)，先对\\(e\_1\\)求值直到它是一个值，再对\\(e\_2\\)求值直到它是一个值，然后对两者进行\\(\\beta\\)归约。举个例子：

\\\[ \\begin{align\*} (\\lambda x.\\lambda y.y\\;x)(5+2)\\lambda x.x+1 & \\to (\\lambda x.\\lambda y.y\\;x)7\\lambda x.x+1\\\\ & \\to (\\lambda y.y\\;7)\\lambda x.x+1\\\\ & \\to (\\lambda x.x+1)7\\\\ & \\to 7 + 1\\\\ & \\to 8 \\end{align\*} \\\]

#### Call-by-name

Call-by-name(CBN)语义与CBV不同，在函数应用时CBN会优先应用参数，而不是化简参数变成值。CBN得small-step语义要简单的多，因为它不强制要求参数在应用时为值：

\\\[ \\frac{e\_1\\to e\_1'}{e\_1 e\_2\\to e\_1'e\_2} \\quad {\\bf{\\beta}\\scriptsize{-REDUCTION}} \\frac{}{(\\lambda x.e\_1)e\_2\\to e\_1\\{e\_2/x\\}} \\\]

举个CBN的例子：

\\\[ \\begin{align\*} (\\lambda x.\\lambda y.y\\;x)(5+2)\\lambda x.x+1 & \\to (\\lambda y.y(5+2))\\lambda x.x+1\\\\ & \\to (\\lambda x.x+1)(5+2)\\\\ & \\to (5+2)+1\\\\ & \\to 7+1\\\\ & \\to 8 \\end{align\*} \\\]

### λ演算的求值策略

如上所诉Lambda Calculus的求值策略有很多种，其中最宽松的是完全\\(\\beta\\)规约，表达式\\((\\lambda x.e\_1)e\_2\\)在任何情况下都能规约成\\(e\_1\\{e\_2/x\\}\\)，完全\\(\\beta\\)规约的small-step语义如下：

\\\[ \\frac{e\_1\\to e\_1'}{e\_1 e\_2\\to e\_1'e\_2} \\quad \\frac{e\_2\\to e\_2'}{e\_1\\;e\_2\\to e\_1\\;e\_2'} \\quad \\frac{e\_1\\to e\_1'}{\\lambda x.e\_1\\to \\lambda x.e\_1'} \\quad {\\bf{\\beta}} \\frac{}{(\\lambda x.e\_1)e\_2\\to e\_1\\{e\_2/x\\}} \\\]

call by value(CBV)比完全\\(\\beta\\)规约要严格一点：CBV只允许参数为value时的规约，并且不允许λ下的规约；call by name(CBN)允许参数不是value的规约，同时也不允许λ下的规约。

很明显完全\\(\\beta\\)规约是non-determinitic（不确定）的，因为对同一个表达式它可以有不同的求值步骤。这时就有一个很显然的问题了，不同的求值步骤会产生不同的结果吗？答案是，不会。完全\\(\\beta\\)规约满足**confluence（合流性）**特性：

\\\[ Therom. 如果e\\to^\*e\_1 \\land e\\to^\*e\_2，那么存在e'使得e\_1\\to^\*e' \\land e\_2\\to^\*e'. \\\]

confluece就是大家常说的**Church-Rosser定理**，在λ演算中求值的顺序并不重要，结果是一样的，如果表达式a能规约成b和c，那么b和c肯定能进一步规约到表达式d。Church-Rosser定理在好多λ演算变种中都成立，如简单类型λ演算，还有许多带复杂类型系统的λ系统。

但不管什么求值策略，都要有一个应用函数的过程，也就是替换操作。下面是替换操作的定义：

\\\[ \\begin{align\*} y\\{e/x\\} & = \\begin{cases} e, & \\text {if y=x} \\\\ y, & \\text{otherwise} \\end{cases} \\\\ (e\_1\\;e\_2)\\{e/x\\} & = (e\_1\\{e/x\\})(e\_2\\{e/x\\}) \\\\ (\\lambda y.e\_1)\\{e/x\\} & = \\lambda.(e\_1\\{e/x\\}) \\qquad y\\neq x \\land y\\notin fv(e) \\end{align\*} \\\]

注意其中最后一个在λ抽象上的替换，需要y和x不同，并且y不在e的自由变量中，不然会出问题，比如：

\\\[ (\\lambda y.x)\\{y/x\\}=(\\lambda y.y) \\\]

当变量名字重复的时候，可以配合\\(\\alpha\\)规则，使得替换操作总能成立。虽然这样可以正确求值，但这些替换操作一般会非常麻烦，下一节就会讲如何用编码解决重名的问题。

### De Bruijn编码

上一节中\\(\\beta\\)替换操作时遇到的有关自由、绑定名字的麻烦有很多，什么重名，\\(\\alpha\\)变换还要重命名，好麻烦，要没有解决办法呢，能不能不用名字呢？是的，不用名字是可以的，de Bruijn编码就是一种不用名字的编码方法，de Bruijn编码中，把绑定变量看出指向绑定它的λ和指针，de Bruijn编码的语法如下：

$$
e::= n\;|\;\lambda.e\;|\;e\;e
$$

变量用整数表示，其数值就是绑定的下标，下面是一些de Bruijn编码的例子：

|Standard | de Bruijn|
|+----|+---|
|λx.x | λ.0|
|λz.z | λ.0|
|λx.λy.x | λ.λ.1|
|λx.λy.λs.λz.x s (y s z) | λ.λ.λ.λ.3 1 (2 1 0)|
|(λx.x x) (λx.x x) | (λ.0 0) (λ.0 0)|
|(λx.λx.x) (λy.y) | (λ.λ.0) (λ.0)|

为了用de Bruijn编码表示带自由变量的λ表达式，我们需要一套映射自由变量到整数的规则，这种映射我们称为上下文，用\\(\\Gamma\\)表示，比如有个映射x到0和y到1的映射\\(\\Gamma\\)，那么包含\\(\\Gamma\\)的\\(\\lambda z.x\\;y\\;z\\)编码后就是\\(\\lambda.1\\;2\\;0\\)，注意其中的x和y为了不被捕捉下标都升了1。这种上升变化的函数为shift操作，其定义如下：

\\\[ \\begin{align\*} \\uparrow^i\_c(n) & = \\begin{cases} n, & \\text {if n<c} \\\\ n+i, & \\text{otherwise} \\end{cases} \\\\ \\uparrow^i\_c(\\lambda.e) & = \\lambda.(\\uparrow^i\_{c+1}e) \\\\ \\uparrow^i\_c(e\_1\\;e\_2) & = (\\uparrow^i\_c e\_1)(\\uparrow^i\_c e\_2) \\end{align\*} \\\]

偏移量c和表达式里绑定的变量是对应的，c控制着哪个变量应该shift。c的初始值为0（所有的变量都要shift），每经过一个函数抽象时c都加1。对于\\(\\uparrow^i\_c(t)\\)中t的每个标识符k，我们知道当\\(k<c\\)，k是经过c已绑定的变量，不应该shift，当$ k c$，k是自由的，可以shift。利用shift函数，可以重新定义替换操作：

\\\[ \\begin{align\*} n\\{e/m\\} & = \\begin{cases} e, & \\text {if n=m} \\\\ n, & \\text{otherwise} \\end{cases} \\\\ (e\_1\\;e\_2)\\{e/m\\} & = (e\_1\\{e/m\\})(e\_2\\{e/m\\}) \\\\ (\\lambda.e\_1)\\{e/m\\} & = \\lambda.e\_1\\{(\\uparrow^1\_0e)/m+1\\})) \\end{align\*} \\\]

下面是de Bruijn编码的\\(\\beta\\)规约的规则：

\\\[ {\\bf{\\beta}} \\frac{}{(\\lambda .e\_1)e\_2\\to\\uparrow^{-1}\_0(e\_1\\{\\uparrow^1\_0e\_2/0\\}) } \\\]

注意在规约时，因为外层λ消失了，需要重新标记变量向下shift。比如\\((\\lambda.1\\;0\\;2)(\\lambda.0)\\to 0(\\lambda.0)1\\)，而不是\\(1(\\lambda.0)2\\)

举个计算的例子，给定\\((\\lambda u.\\lambda v.u\\;x)y\\)的编码形式：\\((\\lambda.\\lambda.1\\;2)1\\)，其中\\(\\Gamma(x)=0,\\Gamma(y)=1\\)：

\\\[ \\begin{align\*} & (\\lambda.\\lambda.1\\;2)1 \\\\ \\to & \\uparrow^{-1}\_0((\\lambda.1\\;2)\\{( \\uparrow^1\_01)0\\}) \\\\ \\to & \\uparrow^{-1}\_0((\\lambda.1\\;2)\\{2/0\\}) \\\\ \\to & \\uparrow^{-1}\_0\\lambda.((1\\;2)\\{(\\uparrow^1\_02)/0+1)\\}) \\\\ \\to & \\uparrow^{-1}\_0\\lambda.((1\\;2)\\{3/1\\}) \\\\ \\to & \\uparrow^{-1}\_0\\lambda.(1\\{3/1\\})(2\\{3/1\\}) \\\\ \\to & \\uparrow^{-1}\_0\\lambda.3\\;2 \\\\ \\to & \\lambda.2\\;1 \\end{align\*} \\\]

最终的结果就是\\(\\lambda v.y\\;x\\)。

### Combinators

另外一种对付自由、绑定变量的方法就是利用`combinator`，也就是组合子，组合子就是闭合的λ表达式，闭合的\\(\\lambda表达式\\)就是没有自由变量的表达式。利用一些特定的组合子，我们能编码整个λ演算，其中一种组合就是S，K组合子和辅助的I组合子：

\\\[ \\begin{align\*} & K \\; x \\; y \\to \\; x \\\\ & S \\; x \\; y \\; z \\to \\; x \\; z \\; (y \\; z) \\\\ & I \\; x \\to x \\end{align\*} \\\]

上面是S、K的应用规则，下面是S、K的闭合λ表示：

\\\[ \\begin{align\*} & K = \\lambda x.\\lambda y.x \\\\ & S = \\lambda x.\\lambda y.\\lambda z.x\\;z\\;(y\\;z) \\\\ & I = \\lambda x.x \\end{align\*} \\\]

之所以I是辅助的是因为I可以用S和K编码出来：`S K K`。注意尽管对任意x都有`((S K K)x) = (I x)`，但`(S K K)`自身并不不等于I。我们称这种项是外延相等的。外延相等：两个函数是相等的，如果它们对于相同的参数总是生成相同的结果。相反的，内涵相等：两个函数是相等的，当且仅当它们有相同的实现。

为了解释怎么用组合子编码λ演算，我们需要一个翻译闭合λ表达式到等同组合子的方法–bracket abstraction：首先，我们定义一个函数\[x\]，它接受一个组合子M作为参数，并生成另一个等同于\\(\\lambda x.M\\)的表达式，其中对任意N都有\\((\[x\] M)N \\to M\\{N/x\\}\\)，下面是函数\[x\]的定义：

\\\[ \\begin{align\*} \[x\]x & = I \\\\ \[x\]N & = K N \\qquad\\qquad 其中x\\notin fv(N)\\\\ \[x\]N\_1 N\_2 & = S(\[x\]N\_1)(\[x\]N\_2) \\\\ \\end{align\*} \\\]

然后，我们定义一个映射λ表达式到组合子表达式的函数(e)\*：

\\\[ \\begin{align\*} (x)\* & = x \\\\ (e\_1 e\_2)\* & = (e\_1)\* (e\_2)\* \\\\ (\\lambda x.e)\* & = \[x\](e)\* \\\\ \\end{align\*} \\\]

这样我们就能编码λ表达式了，比如：

\\\[ \\begin{align\*} & (\\lambda x.\\lambda y.x)\* \\\\ = & \[x\] (\\lambda y.x)\* \\\\ = & \[x\] (\[y\] x) \\\\ = & \[x\] (K x) \\\\ = & (S (\[x\] K) (\[x\] x)) \\\\ = & S (K K) I \\end{align\*} \\\]

不相信？可以算算看：

\\\[ \\begin{align\*} & (\\lambda x.\\lambda y.x)e\_1 e\_2 \\\\ = & (\\lambda y.e\_1)e\_2 \\\\ = & e\_1 \\end{align\*} \\\]

\\\[ \\begin{align\*} & (S(KK)I)e\_1 e\_2 \\\\ = & (KKe\_1)(Ie\_1)e\_2 \\\\ = & Ke\_1e\_2 \\\\ = & e\_1 \\end{align\*} \\\]

组合子和直觉逻辑的联系：组合子K和S分别对应着命题逻辑里的两条公理：

\\\[ AK: A \\to (B \\to A) \\\\ AS: (A \\to (B \\to C)) \\to ((A \\to B) \\to (A \\to C)) \\\]

函数应用则对应着肯定前件：

\\\[ MP: A \\to B, A \\vdash B \\\]

在Curry-Howard同构层面，AK、AS和MP对于直觉逻辑的蕴含片段是完备的，但对于经典逻辑的蕴含片段需要组合子逻辑模拟排中律（比如皮尔士定律）。后面会有更多关于数理逻辑的介绍，这里就不多说了。

### 编码λ演算

纯λ演算只有函数，用纯函数写程序绝对不是啥开心的事儿，为了方便，我们可以编码一些对象，比如布尔值和整数，怎么用函数“实现”对象呢，这里的实现其实是一种约定，比如某个函数就代表整数2，以后遇到这个函数就知道他是整数2了，这种约定就是编码，编码并没有什么神奇的，神奇的在于，当我们合理的编码，比如说整数后，我们就能实现真正的加法、乘法等算数运算了。

#### Booleans

为了实现常见的逻辑操作，比如`if`、`NOT`和`AND`，我们首先要定义布尔值，也就是`true`和`false`：

\\\[ \\begin{align\*} true & \\triangleq \\lambda x.\\lambda y.x \\\\ false & \\triangleq \\lambda x.\\lambda y.y \\end{align\*} \\\]

`true`和`false`都是接受两个参数的函数，`true`返回第一个参数，`false`则返回第二个，至于为什么，这就是我们的编码，它就是这样定义的，神奇的是，有了这个我们就能定义逻辑运算了，比如当我们要定义如下的if函数：

\\\[ \\lambda b.\\lambda t.\\lambda f.if\\;b = \\text {true then t else f}. \\\]

有了`true`和`false`，写`if`就变得很简单：

\\\[ if \\triangleq \\lambda b.\\lambda t .\\lambda f.b\\;t\\;f \\\]

很显然，根据定义，当`b`为`true`时就取第一个参数–`t`，当`b`为`false`时就取第二个参数–`f`，符合`if`的需求。以此类推，其他的逻辑运算也可以很简单的定义出来：

\\\[ \\begin{align\*} not & \\triangleq \\lambda b.b \\; \\text{false true} \\\\ and & \\triangleq \\lambda b\_1.\\lambda b\_2.b\_1\\;b\_2 \\; \\text{false} \\\\ or & \\triangleq \\lambda b\_1.\\lambda b\_2.b\_1 \\; \\text{true} \\; b\_2 \\end{align\*} \\\]

#### Church numerals

丘奇数是一种编码整数的方法，在丘奇数里，整数就是一个接受两个参数–f和x的函数，其中把整数n表示为x在f应用了n次的函数：

\\\[ \\begin{align\*} \\overline 0 & \\triangleq \\lambda f.\\lambda x.x \\\\ \\overline 1 & \\triangleq \\lambda f.\\lambda x.f\\;x \\\\ \\overline 2 & \\triangleq \\lambda f.\\lambda x.f(f\\;x) \\\\ succ & \\triangleq \\lambda n.\\lambda f.\\lambda x.f(n\\;f\\;x) \\end{align\*} \\\]

其中的`succ`函数其实很好理解，它接受另一个丘奇数n作为参数，首先表达式\\(n\\;f\\;x\\)把x应用到f上n次，然后把结果再应用到f上一次，也就是总共把x应用到f上n+1次，最终结果也就是丘奇数n+1，没错，`succ`函数的作用就是给丘奇数加1。定义了`succ`函数，加法也就很好定义了，m+n就是在n上应用m次`succ`函数的结果：

\\\[ plus \\triangleq \\lambda m.\\lambda n.m\\;\\text{succ}\\;n \\\]

比如1 + 2：

\\\[ \\begin{align\*} & 1 + 2 \\\\ \\to & (\\lambda m.\\lambda n.m\\;\\text{succ}\\;n)\\overline 1\\;\\overline 2 \\\\ \\to & \\overline 1 \\; succ \\; \\overline 2 \\\\ \\to & (\\lambda f.\\lambda x.f\\;x)succ(\\lambda f.\\lambda x.f(f\\;x)) \\\\ \\to & (\\lambda n.\\lambda f.\\lambda x.f(n\\;f\\;x))(\\lambda f.\\lambda x.f(f\\;x)) \\\\ \\to & \\lambda f.\\lambda x.f((\\lambda f.\\lambda x.f(f\\;x))f\\;x) \\\\ \\to & \\lambda f.\\lambda x.f(f(f\\;x)) \\\\ \\to & \\overline 3 \\end{align\*} \\\]

这种利用`succ`函数的加法定义其实挺显而易见的，它把m的f变成了`succ`、x变成了n，最后的结果肯定是(succ(succ(succ…n))…))的样式，而且`succ`的个数是m个，那自然结果就是m+n啦。如果不借助`succ`函数，加法的定义是这样的：

\\\[ plus' \\triangleq \\lambda m.\\lambda n.\\lambda f.\\lambda x.m\\;f(n\\;f\\;x) \\\]

乍一看，这什么鬼啊，算起来好麻烦，但借助一点小小的变换，理解起来就很简单了。首先所有的丘奇数都是用f应用到x上多少次所表示出的–(f(f(f…x))…))，也就是f的N次幂，可以把丘奇数n简单的看成\\(f^N\\)，再看上面`plus'`的定义：

\\\[((m\\;f)((n\\;f)x)\\\]

其中m和n都是丘奇数，上面说了丘奇数可以看成求f的N次幂的函数，于是就变成了：

\\\[(f^m(f^n\\;x))\\\]

剩下的就简单了，在x上应用了n次f然后又应用了m次f，结果自然是：

\\\[(f^{m+n}x)\\\]

于是结果就是m+n了，用数学的方式表示就是：

\\\[ f^{m+n}(x) = f^m(f^n(x)) \\\]

这么一想，上面的`plus'`的定义也就显而易见了，如果你理解了`plus'`，那下面的乘法定义也就不需要更多的解释了，乘法用到了恒等式\\(f^{(m\*n)} = (f^m)^n\\)：

\\\[ times \\triangleq \\lambda m.\\lambda n.\\lambda f.n(m\\;f) \\\]

这一章就差不多了，[下一章](/2017/lambda-caculus-part-2)我们会介绍更多更复杂的概念。