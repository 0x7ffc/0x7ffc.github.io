---
layout: post
title: "Lambda Caculus - Part 2"
description: "Part 2 of an introduction to Lambda Caculus which includes Y combinator, evaluation context, curring, Continuation-passing style and others."
lang: zh
mathjax: true
---

用λ演算编程
------

### 不会终止的程序

在用LC写程序时，要注意有可能你的程序永远不会终止。正如下面所示，这是一个永远不会终止的程序，我们叫它`omega`：

\\\[ \\begin{align\*} omega & = (\\lambda x.x\\;x)(\\lambda x.x\\;x) \\\\ & \\to (\\lambda x.x\\;x)(\\lambda x.x\\;x) \\\\ & = omega \\end{align\*} \\\]

`omega`会求值到自己，永远不会结束。更有意思的是当你把`omega`当做函数的参数时，程序并不一定是不会终止的。比如这个程序：\\((\\lambda x.(\\lambda y.y))omega\\)，很显然，终止与否和求值策略有关。当使用CBV求值，我们需要参数先求值到值才应用，但是我们的`omega`永远求不到一个值，自然程序永远不会终止；如果使用CBN求值，由于可以直接应用参数，马上就得到了结果\\(\\lambda y.y\\)，也就是说程序是可以终止的。CBV和CBN都是常见的求值策略，大多数语言用的是CBV，后面还会介call-by-need，也就是按需求值，它和CBN很相似，不到必要时候不求值参数，而且效率更高。

### 递归

要想写出有用的程序，我们还需要递归，需要一种定义递归程序的方法。没有递归，就写不了下面的阶乘函数：

\\\[ fact \\triangleq \\lambda n.if(iszero\\;n)1(times\\;n(fact(pred\\;n))) \\\]

为了读起来方便，换了如下的写法，这个写法已经和很多真正的程序语言写法差不多了：

\\\[ fact \\triangleq \\lambda n.if\\;n=0\\;then\\;1\\;else\\;n\\times fact(n-1) \\\]

`fact`只是等式右边的缩写，但这里`fact`定义显然是不合理的，`fact`在右边也出现了啊，这不是个定义，而是个递归等式，想起了Part 1讲的Kleen不动点了吗？同样我们需要一个在LC里处理递归等式的方法。那怎么才能在所有函数都是匿名函数的LC中实现递归呢，怎么才能让一个没有名字的函数调用自己呢？

一种巧妙的方法是定义一个函数`fact'`，`fact`不是出现在右边了吗，那就在`fact`外再包一层λ，把自己当成参数传进去：

\\\[ fact' \\triangleq \\lambda f.\\lambda n.if\\;n=0\\;then\\;1\\;else\\;n\\times(f f(n -1)) \\\]

这里的\\(ff(n-1)\\)中\\(f\\)其实是`fact'`。注意与`fact`不同，在调用`fact'`时，不是直接\\(fact'(n-1)\\)，还要传入`fact'`自身（看到这里递归的思想了吗）。下面是利用`fact'`定义出的`fact`：

\\\[ fact \\triangleq fact' fact' \\\]

不相信？那举个例子：

\\\[ \\begin{align\*} fact \\; 3 & = (fact' fact')3 \\\\ & = (\\lambda f.\\lambda n.if\\;n=0\\;then\\;1\\;else\\;n\\times(f f(n -1)) fact')3 \\\\ & \\to (\\lambda n.if\\;n=0\\;then\\;1\\;else\\;n\\times(fact' fact'(n -1)))3 \\\\ & \\to if\\;3=0\\;then\\;1\\;else\\;3\\times(fact' fact'(3-1)) \\\\ & \\to 3\\times (fact' fact'(3-1)) \\\\ & \\to \\cdots \\\\ & \\to 3 \\times 2 \\times 1 \\times 1 \\\\ & \\to^\* 6 \\end{align\*} \\\]

神奇吧，if的判断确保了这里并不会无限递归。总结下来，我们有了一个定义递归函数\\(f\\)的技巧：写一个接受自己作为参数的\\(f'\\)，然后\\(f\\)就是\\(f\\triangleq f' f'\\)。

如果上面的方法难以理解的话，那还有一种方法，如果你还记得Part 1中指称语义中定义`while`的方法的话，那这种方法跟它差不多，没错，就是利用不动点，函数没有名字那就利用高阶函数算出“自己”：把函数表示成一个高阶函数的不动点，这个不动点就是我们想要的函数。比如下面函数G的不动点就是我们想要的`fact`函数：

\\\[ G \\triangleq \\lambda f.\\lambda n.if\\;n=0\\;then\\;1\\;else\\;n\\times(f(n-1)) \\\]

还记得什么是不动点吗？如果g是G的不动点，那就有\\(G\\;g=g\\)，也就是说如果有求G不动点的方法，那就可以定义`fact`了。幸运的是，有很多种可以用来求不动点的组合子，其中最著名的要数Haskell Curry发现的Y组合子了（著名的创业孵化器公司Y Combinator就是以此命名的）：

\\\[ Y \\triangleq \\lambda f.(\\lambda x.f(x\\;x))(\\lambda x.f(x\\;x)). \\\]

尽管Y组合子是最出名的不动点组合子，它有一个很大的问题：Y组合子只能在non-strict的规约策略下使用，否则会发散（diverge）。如下所示，在CBV规则下，会无限递归：

\\\[ \\begin{align\*} & fact \\\\ = & Y G \\\\ = & (\\lambda f.(\\lambda x.f(x\\;x))(\\lambda x.f(x\\;x)))G \\\\ \\to & (\\lambda x.G(x\\;x))(\\lambda x.G(x\\;x)) \\\\ \\to & G((\\lambda x.G(x\\;x))(\\lambda x.G(x\\;x))) \\\\ \\to & G(G((\\lambda x.G(x\\;x))(\\lambda x.G(x\\;x)))) \\\\ \\to & \\cdots \\end{align\*} \\\]

为了解决这个问题，我们需要把表达式包装一下以延迟计算（看到惰性求值的思想了吗？后面会详细介绍），包装后的组合子就是Z组合子，也称作CBV Y组合子：

\\\[ Z \\triangleq \\lambda f.(\\lambda x.f(\\lambda y.x\\;x\\;y))(\\lambda x.f(\\lambda y.x\\;x\\;y)) \\\]

下面是用Z组合子求fact的过程，注意其中是如何利用\\(\\lambda y\\)包住(x x)以延迟计算的：

\\\[ \\begin{align\*} & fact \\\\ = & Z\\;G \\\\ = & (\\lambda f.(\\lambda x.f(\\lambda y.x\\;x\\;y))(\\lambda x.f(\\lambda y.x\\;x\\;y)))G \\\\ \\to & (\\lambda x.G(\\lambda y.x\\;x\\;y))(\\lambda x.G(\\lambda y.x\\;x\\;y)) \\\\ \\to & G(\\lambda y.(\\lambda x.G(\\lambda y.x\\;x\\;y))(\\lambda x.G(\\lambda y.x\\;x\\;y)) y) \\\\ = & (\\lambda f.\\lambda n.if\\;n=0\\;then\\;1\\;else\\;n\\times(f(n-1))) \\\\ & \\quad (\\lambda y.(\\lambda x.G(\\lambda y.x\\;x\\;y))(\\lambda x.G(\\lambda y.x\\;x\\;y)) y) \\\\ \\to & \\lambda n.if\\;n=0\\;then\\;1\\;else\\;n\\times((\\lambda y.(\\lambda x.G(\\lambda y.x\\;x\\;y))(\\lambda x.G(\\lambda y.x\\;x\\;y)) y)(n-1)) \\\\ =\_\\beta & \\lambda n.if\\;n=0\\;then\\;1\\;else\\;n\\times(\\lambda y.(Z\\;G)y)(n-1) \\\\ =\_\\beta & \\lambda n.if\\;n=0\\;then\\;1\\;else\\;n\\times(Z\\;G(n-1)) \\\\ = & \\lambda n.if\\;n=0\\;then\\;1\\;else\\;n\\times(fact(n-1)) \\end{align\*} \\\]

下面是另一个特别有趣的不动点组合子：

\\\[ Y\_k \\triangleq (L L L L L L L L L L L L L L L L L L L L L L L L L L) \\\\ L \\triangleq \\lambda abcdefghijklmnopqstuvwxyzr. (r (t h i s i s a f i x e d p o i n t c o m b i n a t o r)) \\\]

像这样的组合子有很多，实际上是无穷多的，鉴于有无穷多的不动点组合子，为了更好的理解，我们还是看看最初Alan Turing发现的不动点组合子吧。假设我们有一个高阶函数\\(f\\)，要算\\(f\\)的不动点，首先假设\\(\\Theta f\\)是\\(f\\)的不动点，于是根据不动点的定义有：

\\\[ \\Theta f = f(\\Theta f) \\\]

把左边的\\(f\\)当成右边的参数变化一下：

\\\[ \\Theta = \\lambda f.f(\\Theta f) \\\]

现在再利用我们上面学到的第一个定义递归函数的技巧，定义\\(\\Theta' = \\lambda t.\\lambda f.f(t\\;t\\;f)\\)，于是就得到：

\\\[ \\begin{align\*} \\Theta & = \\Theta'\\Theta' \\\\ & = (\\lambda t.\\lambda f.f(t\\;t\\;f))\\Theta' \\\\ & \\to \\lambda f.f(\\Theta'\\Theta'f) \\\\ & = \\lambda f.f(\\Theta f) \\end{align\*} \\\]

Ok，现在让我们试试它对不对，同样，CBV是不行的，这次我们用直接用CBN规则了：

\\\[ \\begin{align\*} fact & = \\Theta \\; G \\\\ & = ((\\lambda t.\\lambda f.f(t\\;t\\;f))(\\lambda t.\\lambda f.f(t\\;t\\;f)))G \\\\ & \\to (\\lambda f.f((\\lambda t.\\lambda f.f(t\\;t\\;f))(\\lambda t.\\lambda f.f(t\\;t\\;f)) f)) G \\\\ & \\to G((\\lambda t.\\lambda f.f(t\\;t\\;f))(\\lambda t.\\lambda f.f(t\\;t\\;f))G) \\\\ & = G(\\Theta \\; G) \\\\ & = (\\lambda f.\\lambda n.if\\;n=0\\;then\\;1\\;else\\;n\\times(f(n-1)))(\\Theta \\; G) \\\\ & \\to \\lambda n.if\\;n=0\\;then\\;1\\;else\\;n\\times((\\Theta \\; G)(n-1)) \\\\ & = \\lambda n.if\\;n=0\\;then\\;1\\;else\\;n\\times(fact(n-1)) \\end{align\*} \\\]

完美。

虽然我们这里讲了这么多怎么在λ演算里实现递归，但其实在实现真正的程序语言时是不会利用到Y或者Z组合子的，不动点组合子的意义在于展示在无类型λ演算中，递归（满足fix f = f (fix f)的fix）可以用纯λ演算自身来实现，而不需要扩充λ演算的定义。如果你要问为何不直接扩充λ演算，你需要看看[奥卡姆剃刀](https://en.wikipedia.org/wiki/Occam%27s_razor)：如无必要，勿增实体。

Definitional translation
------------------------

我们已经看到了如何在λ演算里编码出高级语言中才有的结构（布尔值、条件语句、自然数和递归），但这些结构都还是数学函数，并不是人们熟悉的程序语言，是时候看看Definational translation了（后面用DT代替）：通过把当前语言翻译成另外一种语言来定义语言的意思。这是一种指称语义的手法，但翻译的结果并不像指称语义那样还是数学函数，而是一个简单的程序语言。注意DT并不会生成更简洁或者更高效的代码，它的作用只是用目标语言来描述当前语言的意思。

对每个语言结构，我们都会定义一个对应的操作语义，然后利用DT翻译成一个简单的语言。不过首先要介绍的是求值上下文（evaluation contexts），它会帮助我们简便的定义新语言特性。

### Evaluation contexts

让我们回忆一下λ演算的CBV操作语义：

$$
\begin{align*}
e & ::= x\;|\;\lambda x.e\;|\;e_1\;e_2 \\
v & ::= \lambda x.e
\end{align*}
$$

\\\[ \\frac{e\_1\\to e\_1'}{e\_1 e\_2\\to e\_1'e\_2} \\quad \\frac{e\\to e'}{v\\;e\\to v\\;e'} \\quad {\\bf{\\beta}\\scriptsize{-REDUCTION}} \\frac{}{(\\lambda x.e)v\\to e\\{v/x\\}} \\\]

这三个规则中只有\\(\\beta\\)规约在干“实事”：规约化简一个表达式，其它两个只是告诉我们求值的顺序：先把第一个参数规约成值，再把第二参数规约成值。大多数的语言都有上述特性：一种规则（congruence rule）定义求值的顺序，另一种规则（computation rule）定义真正的规约计算。

利用求值上下文我们就能方便的区分这两种规则。我们用\\(E\[\\cdot\]\\)表示一个求值上下文，其中的点表示一个待填充的洞，以后会填进其它的表达式。跟我们上面CBV λ演算的定义一样，求值上下文也可以用BNF来表示：

$$
E ::= [\cdot]\;|\;E\;e\;|\;v\;E
$$

上面就是CBV λ演算的求值上下文定义。当一个求值上下文E的洞被表达式e填充过都后就写成\\(E\[e\]\\)：

\\\[ \\begin{align\*} E\_1 &=\[\\cdot\](\\lambda x.x) \\qquad \\qquad & E\_1\[\\lambda y.y\\;y\]=(\\lambda y.y\\;y)\\lambda x.x \\\\ E\_2 &=(\\lambda z.z\\;z)\[\\cdot\] & E\_2\[\\lambda x.\\lambda y.x\]=(\\lambda z.z\\;z)(\\lambda x.\\lambda y.x) \\\\ \\end{align\*} \\\]

有了求值上下文，上面的CBV λ演算只需两条规则就能定义了：

\\\[ \\frac{e\\to e'}{E\[e\]\\to E\[e'\]} \\qquad {\\bf{\\beta}\\scriptsize{-REDUCTION}} \\frac{}{(\\lambda x.e)v\\to e\\{v/x\\}} \\\]

注意上面的第一个规则是怎么利用E的定义巧妙的实现和CBV一样的求值顺序的。理解了CBV，那实现CBN也很简单了：

$$
E ::= [\cdot]\;|\;E\;e \qquad \frac{e\to e'}{E[e]\to E[e']} \qquad {\bf{\beta}\scriptsize{-REDUCTION}} \frac{}{(\lambda x.e_1)e_2\to e_1\{e_2/x\}}
$$

求值上下文的好处远远不止这些，下面会构造一些更复杂的语言结构。

### 多参函数和柯里化

λ演算中的函数只允许一个参数，利用求值上下文我们可以方便的定义一个支持多参函数的语言：

$$
e ::= x\;|\;\lambda x_1,...,x_n.e\;|\;e_0\;e_1\;...\;e_n
$$

多参λ演算的CBV语义如下：

$$
E ::= [\cdots]\;|\;v_0 ... v_{i-1}\;E\;e_{i+1}...e_n \qquad \frac{e\to e'}{E[e]\to E[e']}
$$

$$
{\bf{\beta}\scriptsize{-REDUCTION}}  \frac{}{(\lambda x_1,...,x_n.e_0)v_1...v_n\to e_0\{v_1/x_1\}\{v_2/x_2\}...\{v_n/x_n\}}
$$

E保证了先把\\(e\_0\\;e\_1\\;...\\;e\_n\\)从左至右依次求值为value从而实现CBV。注意这种多参λ演算和纯λ演算其实是一样的，多参λ演算并不比纯λ演算高到哪里去。为了展示它们是一样的，这里定义了一个函数\\(T\[\\cdot\]\\)，它的作用就是把多参λ演算转换成纯λ演算：

\\\[ \\begin{align\*} T\[x\] & = x \\\\ T\[\\lambda x\_1,...,x\_n.e\] & = \\lambda x\_1...\\lambda x\_n.T\[e\] \\\\ T\[e\_0\\;e\_1\\;e\_2\\;...\\;e\_n\] & = (...((T\[e\_0\]T\[e\_1\])T\[e\_2\])...T\[e\_n\]) \\end{align\*} \\\]

这种把多参函数转换为单参函数调用链的技术就是柯里化（curring）。想象一个接受两个参数的函数，第一个参数定义域为A，第二个参数定义域为B，最后返回定义域C中的某个结果，用数学的方式表示就是\\(A\\times B \\to C\\)，这个函数经过柯里化之后是\\(A\\to (B \\to C)\\)，也就是接受A后返回一个函数，此函数接受B返回C。

这些对DT来说都是小儿科，是时候扩展λ演算展示下DT真正的实力了。

### Products 、let

下面是我们加入`product`和`let`之后的λ演算：

$$
\begin{align*}
e & ::=x\;|\;\lambda x.e\;|\;e_1\;e_2 \\
  & |\;(e_1,e_2)\;|\;fst\;e\;|\;snd\;e \\
  & |\;let\;x=e_1\;in\;e_2 \\
v & ::= \lambda x.e\;|\;(v_1,v_2)
\end{align*}
$$

扩展后的λ演算包含

*   **product**：一对表达式就是一个`product`，如，如\\((e\_1,e\_2)\\)，当\\(e\_1\\)和\\(e\_2\\)都是值的时候，`product`才是一个值；我们可以用`fst`运算符获取第一个元素，用`snd`获取第二个，如\\(fst\\;(v\_1,v\_2)\\to v\_1\\)。
*   **let**，`let`把\\(e\_2\\)中所有的x当成\\(e\_1\\)，也就是和\\((\\lambda x.e\_2)e\_1\\)是等价的。

利用求值上下文我们可以写出上述语言的small-step CBV操作语义：

$$
E ::= [\cdot]\;|\;E\;e\;|\;v\;E\;|\;(E,e)\;|\;\;(v,E)\;|\;fst\;E\;|\;snd\;E|\;let\;x=E\;in\;e_2
$$

$$
\frac{e\to e'}{E[e]\to E[e']} \qquad \qquad {\bf{\beta}\scriptsize{-REDUCTION}} \frac{}{(\lambda x.e)v \to e\{v/x\}} \\
$$

$$
\frac{}{fst\;(v_1,v_2)\to v_1} \qquad \qquad \qquad \frac{}{snd\;(v_1,v_2)\to v_2} \\ \\
$$

$$
\frac{}{let\;x=v\;in\;e\to e\{v/x\}}
$$

最后只需要翻译成纯λ演算就行了：

\\\[ \\begin{align\*} T\[x\] & = x \\\\ T\[\\lambda x.e\] & = \\lambda x.T\[e\] \\\\ T\[e\_1\\;e\_2\] & = T\[e\_1\]T\[e\_2\] \\\\ T\[(e\_1,e\_2)\] & = (\\lambda x.\\lambda y.\\lambda f.f\\;x\\;y)T\[e\_1\]T\[e\_2\] \\\\ T\[fst\\;e\] & = T\[e\](\\lambda x.\\lambda y.x) \\\\ T\[snd\\;e\] & = T\[e\](\\lambda x.\\lambda y.y) \\\\ T\[let\\;x=e\_1\\;in\\;e\_2\] & = (\\lambda x.T\[e\_2\])T\[e\_1\] \\end{align\*} \\\]

### Laziness

我们定义过了λ演算的CBV语义的CBN语义：CBV中参数在应用时必须是值，CBN中参数随时可以应用，两种是截然不同的语义，有些程序在CBV中不会终止，而在CBN中可以终止。但其实，CBV是可以从CBN翻译得到的：在参数上包一层函数以延迟计算。这种在计算上包函数来延迟求值的方法叫做`thunk`：

\\\[ \\begin{align\*} T\[x\] & = x(\\lambda y.y) \\\\ T\[\\lambda x.e\] & = \\lambda x.T\[e\] \\\\ T\[e\_1\\;e\_2\] & = T\[e\_1\](\\lambda z.T\[e\_2\]) \\quad \\text{其中z不是} e\_2 \\text{的自由变量} \\end{align\*} \\\]

所有的参数都变成了`thunk`，用的时候应用thunk就可以得到真正的参数了，thunk的参数并不重要，因为并不会用到它。

### References

像引用这种可以在内存创建、读、写的高级概念利用DT也能实现，而且最终的语言依然是函数式的，但表达式就会有副作了–因为可以改变程序状态。此语言的定义如下：

$$
\begin{align*}
e & ::= x\;|\;\lambda x.e\;|\;e_0\;e_1\;|\;ref\;e\;|\;!e\;|\;e_1:=e_2\;|\;\ell \\
v & ::= \lambda x.e\;|\;\ell
\end{align*}
$$

`ref e`像`malloc`一样创建一个新的内存区域，并初始化区域内容为`e`的结果，`ref e`表达式本身求值的结果是内存地址\\(\\ell\\)，\\(\\ell\\)像指针一样指向创建的内存区域；`!e`把`e`当做内存地址，求值的结果为地址所保存的内容；\\(e\_1:=e\_2\\)把\\(e\_1\\)当做内存地址\\(\\ell\\)，并把\\(\\ell\\)的内容更新为\\(e\_2\\)的结果。注意程序员并不会直接使用\\(\\ell\\)，它只是语言操作语义中的东西。这个带引用类型的语言的small-step CBV操作语义如下，其中配置\\(\\langle\\sigma,e\\rangle\\)中`e`是表达式，\\(\\sigma\\)是地址到值得映射。：

$$
E ::= [\cdot]\;|\;E\;e\;|\;v\;E\;|\;ref\;E\;|\;!E\;|\;E:=e\;|\;v:=E \quad \frac{\langle\sigma,e\rangle\to\langle\sigma',e'\rangle}{\langle\sigma,E[e]\rangle\to\langle\sigma',E[e']\rangle} \\
$$

$$
{\bf{\beta}\scriptsize{-REDUCTION}} \frac{}{\langle\sigma,(\lambda x.e)v\rangle\to\langle\sigma,e\{v/x\}\rangle} \quad {\bf{A}\scriptsize{LLOC}} \frac{}{\langle\sigma,ref\;v\rangle\to\langle\sigma[\ell\mapsto v],\ell\rangle}\ell\notin dom(\sigma) \\
$$

$$
{\bf{D}\scriptsize{EREF}} \frac{}{\langle\sigma,!\ell\rangle\to\langle\sigma,v\rangle}\sigma(\ell)=v \qquad \qquad {\bf{A}\scriptsize{SSIGN}} \frac{}{\langle\sigma,\ell:=v\rangle\to\langle\sigma[\ell\mapsto v],v\rangle}
$$

这个新语言并不比λ演算高到哪里去，它也能翻译成λ演算，不过很麻烦就是了。

Continuations
-------------

前面的翻译中，源语言的控制结构直接翻译成了目标语言额结构，比如：

\\\[ \\begin{align\*} T\[\\lambda x.e\] & = \\lambda x.T\[e\] \\\\ T\[e\_1\\;e\_2\] & = T\[e\_1\]T\[e\_2\] \\end{align\*} \\\]

当目标语言的控制结构和源语言很相似的时候，这种翻译还是够用的，但当两者有较大区别时（比如`goto`命令），就需要借助其它东西了–Continuation。Continuation是个很重要的编程技巧，因为它能让程序的控制流变成明确的东西。Continuation还可以用来定义复杂的控制结构，比如异常处理、goto。一般来说，一个continuation代表着“剩下的程序”，虽然“剩下的程序”在命令式语言中很显而易见（因为是一句一句statement组成的，剩下的程序就是剩下的statement），但在函数式语言中就不一样了，因为不能写出\\(fun1();fun2()\\)这种代码。举个例子，给定下面的代码：

\\\[ \\text {if foo < 10 then 32 + 6 else 7 + bar} \\\]

在求值`if`语句之前要先求值`foo < 10`，求值`if`语句已经之后发生的事情就是`foo < 10`的continuation，我们可以这样表示：

\\\[ (\\lambda y.\\text{if y then 32 + 6 else 7 + bar)(foo < 10)} \\\]

虽然这和原本的表达式是等价的，但这里的控制流更明显了，表达出了先求值\\(foo < 10\\)这个需求，这一点其实是很重要的，因为在函数式程序里，控制流是隐藏起来的。再举个例子：

\\\[ (\\lambda x.x)((1+2)+3)+4 \\\]

首先，最外层的求值结果是一个`value`，也就是说它的continuation如下：

\\\[ k\_0 = \\lambda v.(\\lambda x.x)v \\\]

下一层接受一个值，然后把它加4，并把结果传给\\(k\_0\\)：

\\\[ k\_1 = \\lambda a.k\_0(a+4) \\\]

以此类推：

\\\[ k\_2 = \\lambda b.k\_1(b+3) \\\\ k\_3 = \\lambda c.k\_2(c+2) \\\]

最后程序就是\\(k\_3 1\\)，又因为\\(let x=e\\;in\\;e'\\)语法糖和\\(\\lambda x.e')e\\)是等价的：

\\\[ \\begin{align\*} & let\\;c = 1\\;in \\\\ & let\\;b = 2\\;in \\\\ & let\\;a = 3\\;in \\\\ & let\\;v = a + 4\\;in \\\\ & (\\lambda x.x)v \\end{align\*} \\\]

这就和机器指令很像了：

\\\[ \\begin{align\*} & set\\;c,1 \\\\ & set\\;b,c,2 \\\\ & set\\;a,b,3 \\\\ & set\\;v,a,4 \\\\ & call\\;id,v \\end{align\*} \\\]

利用这些continuation，函数都变成“没有返回值”，并在原本的参数之上又加了个参数用来表示continuation。当函数结束时，会调用continuation并把结果当成参数传给continuation，而不是直接把结果返回给调用者，这种风格的代码通常叫做Continuation-Passing Style，简写是CPS。举个例子，下面是CPS风格的阶乘函数：

\\\[ FACT\_{cps} = Y\\;\\lambda f.\\lambda n,k.if\\;n=0\\;then\\;k\\;1\\;else\\;f(n-1)(\\lambda v.k(n\*v)) \\\]

其中k是continuation，因为用到了Y组合子，可能不容易理解，下面是Haskell实现：

```haskell
fac :: Int -> Int
fac 0 = 1
fac n = n * fac (n -1)

facCPS :: Int -> (Int -> a) -> a
facCPS 0 k = k 1
facCPS n k = facCPS (n-1) $ \res -> k $ n * res

print $ fac 6                                       -- 720
print $ facCPS 6 id                                 -- 720
print $ fmap ((*2) . (+10) . (fac)) [1..3]          -- [22,24,32]
print $ fmap (flip facCPS ((*2) . (+10))) [1..3]    -- [22,24,32]
```

CPS在实现函数式语言编译器中是很重要的概念，常常被当做编译器IR。正如上面展示的那样，CPS让控制结构变明显，其最终形式和机器指令很像，非常利于翻译函数到机器码。比如，CPS调用可以翻译成函数jump，因为被调用的函数并不会返回控制。

### CPS λ演算

我们可以把λ演算翻译成CPS样式，我们要翻译的语言语法如下：

$$
e ::= x\;|\;\lambda x.e\;|\;e_1\;e_2\;|\;n\;|\;e_1+e_2\;|\;(e_1,e_2)\;|\;fst\;e\;|\;snd\;e
$$

首先定义一个\\(CPS\[\\cdot\]\\)函数，它接受一个CBV λ表达式，并返回CPS样式的λ表达式。对所有的表达式`e`，都有\\(CPS\[e\] = \\lambda k.\\cdots\\)，其中`k`是continuation，\\(\\cdots\\)是求值`e`的结果。并且为了简介，把\\(CPS\[e\]=\\lambda xk.\\cdots\\)写成\\(CPS\[e\]\\;k\\)：

\\\[ \\begin{align\*} CPS\[n\]\\;k = & k\\;n \\\\ CPS\[e\_1+e\_2\]\\;k = & CPS\[e\_1\](\\lambda n.CPS\[e\_2\](\\lambda m.k(n+m))) \\quad n\\text{不是}e\_2\\text{的自由变量} \\\\ CPS\[(e\_1,e\_2)\]\\;k = & CPS\[e\_1\](\\lambda v.CPS\[e\_2\](\\lambda w.k(v,w))) \\quad v\\text{不是}e\_2\\text{的自由变量} \\\\ CPS\[fst\\;e\]\\;k = & CPS\[e\](\\lambda v.k(fst\\;v)) \\\\ CPS\[snd\\;e\]\\;k = & CPS\[e\](\\lambda v.k(snd\\;v)) \\\\ CPS\[x\]\\;k = & k\\;x \\\\ CPS\[\\lambda x.e\]\\;k = & k (\\lambda x.\\lambda k'.CPS\[e\]k') \\quad k'\\text{不是}e\_2\\text{的自由变量} \\\\ CPS\[e\_1\\;e\_2\]\\;k = & CPS\[e\_1\](\\lambda f.CPS\[e\_2\](\\lambda v.f\\;v\\;k)) \\quad f\\text{不是}e\_2\\text{的自由变量} \\end{align\*} \\\]

利用这种翻译，我们可以方便的写出一个简单的λ演算编译器。像这种的例子有很多，如果一个程序语言支持First-class Continuation，那么我们程序员就能写出操作程序控制流的代码，创造出自己的流程结构，这样才是完整掌控了自己的代码。很多“复杂”概念其背后的原理都是CPS，比如异常（exception）、协程（fiber）、生成器（generator）。有了First-class Continuation就像有了时光机，能在代码时空穿梭，随心所欲。

小结
--

这一章主要讲解了λ演算，准确的说是untyped Lambda Calculus，也就是无类型λ演算，重点介绍了：

*   λ演算定义，语法，语义以及求值策略；
*   De Bruijn编码、SKI编码以及如何编码高级语言结果（如布尔值和布尔运算、整数和算数运算）；
*   如何在λ演算中实现递归；
*   如何利用Definitional translation扩展λ演算并构造新语言；
*   什么是Continuations以及无敌的CPS；