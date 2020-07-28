---
layout: post
title: "Programming Languages and Logic Part 2"
description: "Programming languages share a lot in common with Logic. Here i'll introduce formal semantics including Operational Semantics, Denotational Semantics and Axiomatic Semantics through a simple language we made called IMP"
mathjax: true
---

### 指称语义

已经学习了两种操作语义，这一节，我们学习新的语义模型–指称语义。指称语义意用数学函数表达程序在计算什么，以IMP为例子就是一个把store转换为另一个store的函数：给定一个store，程序输出一个最终的store。举个例子，可以把程序\\(foo:=bar+1\\)想成一个函数，此函数接受一个store\\(\\;\\sigma\\)，输出另一个和\\(\\sigma差不多的\\)store\\(\\;\\sigma'\\)，只不过\\(\\sigma'\\)另外包含了foo到\\(\\sigma(bar)+1\\)的映射：\\(\\sigma'=\\sigma\[foo\\mapsto\\sigma(bar)+1\]\\)。程序就是store到store的函数。操作语义告诉我们程序如何运行，指称语义展示的是程序在计算什么。

#### IMP的指称语义

给定程序c，我们用\\(C\[c\]\\)表示c的指称（denotation），用数学函数表示就是：

$$
C[c]:Store\rightharpoonup{Store}
$$

其中\\(C\[c\]\\)其实是个偏函数，因为既可能输入的store不是由程序的自由变量定义的，也可能程序不会终止。不终止的程序的\\(C\[c\]\\)是未定义的，因为没有对应的store作为结果。把\\(C\[c\]\\)应用到store\\(\\;\\sigma\\)上则写成\\(C\[c\]\\sigma\\)，如果用\\(f\\)表示\\(C\[c\]\\)，那\\(f(\\sigma)\\)和\\(C\[c\]\\)意思是一样的。同时定义下面的两个函数，\\(A\[a\]\\)是算术表达式a的指称，\\(B\[b\]\\)是布尔表达式b的指称：

$$
\begin{align*}
A[a]&:Store\rightharpoonup{Int}\\
B[b]&:Store\rightharpoonup\{true,false\}
\end{align*}
$$

现在我们能定义IMP的的指称语义了。首先定义算术表达式的布尔表达式的指称：

$$
\begin{align*}
A[n]=&\{(\sigma,n)\}\\
A[x]=&\{(\sigma,\sigma(x))\}\\
A[a_1+a_2]=&\{(\sigma,n)|(\sigma,n_1)\in{A[a_1]}\land(\sigma,n_2)\in{A[a_2]}\land{n=n_1+n_2}\}\\
B[true]=&\{(\sigma,true)\}\\
B[false]=&\{(\sigma,false)\}\\
B[a_1\lt a_2]=&\{(\sigma,true)|(\sigma,n_1)\in{A[a_1]}\land(\sigma,n_2)\in{A[a_2]\land{n_1\lt n_2}}\}\cup\\
&\{(\sigma,false)|(\sigma,n_1)\in{A[a_1]}\land(\sigma,n_2)\in{A[a_2]}\land{n_1\geq{n_2}}\}
\end{align*}
$$

命令的指称如下：

$$
\begin{align*}
C[skip]=&\{(\sigma,\sigma)\}\\
C[x:=a]=&\{(\sigma,\sigma[x\mapsto{n}])|(\sigma,n)\in{A[a]})\}\\
C[c_1;c_2]=&\{(\sigma,\sigma')|\exists\sigma''.((\sigma,\sigma'')\in{C[c_1]}\land(\sigma'',\sigma'\in{C[c_2]})\}
\end{align*}
$$

定义关系运算\\(C\[c\_1;c\_2\]=C\[c\_2\]\\circ{C\[c\_1\]}\\)，\\(\\circ\\)是关系的复合运算，定义如下：

$$
if\;R_1\subseteq{A\times{b}}\;and\;R_2\subseteq{B\times{C}},R_2\circ{R_1}\subseteq{A\times{C}},then\;R_2\circ{R_1}=\{(a,c)|\exists{b\in{B}}.(a,b)\in{R_1}\land(b,c)\in{R_2}.\}
$$

如果\\(C\[c\_1\]\\)和\\(C\[c\_2\]\\)是全函数，那么\\(\\circ\\)则是函数复合。下面是if命令和while命令的定义：

$$
\begin{align*}
C[if\;b\;then\;c_1\;else\;c_2]=&\{(\sigma,\sigma')|(\sigma,true)\in{B[b]}\land(\sigma,\sigma')\in{C[c_1]}\}\cup\\
&\{(\sigma,\sigma')|(\sigma',false)\in{B[b]}\land(\sigma,\sigma')\in{C[c_2]}\}\\
C[while\;b\;do\;c]=&\{(\sigma,\sigma)|(\sigma,false)\in{B[b]}\}\cup\\
&\{(\sigma,\sigma')|(\sigma,true)\in{B[b]}\land\exists\sigma''.((\sigma,\sigma'')\in{C[c]}\land(\sigma'',\sigma')\in{C[while\;b\;do\;c])}\}
\end{align*}
$$

但是只要你仔细一看，就会发现while的定义是不对的，它在自己的定义中用到了自己，这就不是一个定义了，而是一个递归等式，解决这个问题的方法就是不动点。

#### 不动点（Fixed points）

为了解决while定义的问题，我们必须找出符合递归等式的函数。为了理解这个问题，我们先理解下面这个例子，给定函数\\(f:\\mathbb{N}\\to\\mathbb{N}\\):

$$
f(x)= \begin{cases} 0, & \text {if $x$=0} \\ f(x-1)+2x-1 & \text{otherwise}\end{cases}
$$

这不是\\(f\\)的定义，而是\\(f\\)满足的等式，而且这里似乎只有一个函数\\(f\\)满足这个等式：\\(f(x)=x^2\\)，这个函数\\(f\\)就是不动点。通常来说，给定一个递归等式，不一定有解（\\(g:\\mathbb{N}\\to\\mathbb{N},g(x)=g(x)+1\\)就没解）。

利用逐步渐近法，我们可以计算出不动点。每次计算就越来越逼近最终的答案。为了求出满足递归等式的\\(f\\)，首先给定函数\\(f\_0=\\emptyset\\)（\\(f\_0\\)是空关系，一个定义域为空的偏函数），然后用逐步渐进法逐步计算：

$$
\begin{align*}
f_0&=\emptyset\\
f_1&= \begin{cases} 0, & \text {if $x$=0} \\ f_0(x-1)+2x-1 & \text{otherwise}\end{cases}\\
&=\{(0,0)\}\\
f_2&= \begin{cases} 0, & \text {if $x$=0} \\ f_1(x-1)+2x-1 & \text{otherwise}\end{cases}\\
&=\{(0,0),(1,1)\}\\
f_3&= \begin{cases} 0, & \text {if $x$=0} \\ f_2(x-1)+2x-1 & \text{otherwise}\end{cases}\\
&=\{(0,0),(1,1),(2,4)\}
\end{align*}
$$

\\(f\_i\\)逐步呈现出\\(f(x)=x^2\\)。

但是数学中没有‘显然’，接下来就是给逐步渐进法建立一个数学模型：一个高阶函数F，F接受一个\\(f\_k\\)，输出下一个\\(f\_{k+1}\\):

$$
F:(\mathbb{N}\rightharpoonup\mathbb{N})\to(\mathbb{N}\rightharpoonup\mathbb{N})
$$

其中 \\\[(F(f))(x)= \\begin{cases} 0, & \\text {if $x$=0} \\\\ f(x-1)+2x-1 & \\text{otherwise}\\end{cases}\\\]

上述等式的一个解就是\\(f=F(f)\\)。通常来说，给定函数\\(F:A\\to{A}\\)，如果\\(a\\in{A},F(a)=a\\)，那么a就是F的不动点。于是，F的所有不动点的集合就是递归等式最终的解：

$$
\begin{align*}
f&=fix(F)\\
&=f_0\cup f_1\cup f_2\cup f_3\cup \dots\\
&=\emptyset\cup F(\emptyset)\cup F(F(\emptyset))\cup F(F(F(\emptyset)))\cup \dots\\
&=\bigcup_{i\geq0}F^i(\emptyset)
\end{align*}
$$

### Kleene不动点定理与IMP

让我们利用不动点重新定义IMP的while语义：

$$
\begin{align*}
C[while\;b\;do\;c]=&fix(F)\\
F(f)=&\{(\sigma,\sigma)|(\sigma,false)\in{B[b]}\}\cup\\
&\{(\sigma,\sigma')|(\sigma,true)\in{B[b]}\land\exists\sigma''.(\sigma,\sigma'')\in{C[c]}\land(\sigma'',\sigma')\in{f}\}
\end{align*}
$$

但是认真的读者肯定心中已经有个疑问了：“你怎么确定不动点存在？”，是的，就像上一节所说的那样，递归等式不一定有解，不动点不一定存在。这一节就要利用Kleen不动点定理证明while命令的语义是存在不动点的。

#### Kleene不动点定理

$$
Definition(Scott连续). 给定函数f: Cl(X) \rightarrow Cl(Y)，如果\forall x \in Cl(X), \forall b \in f(x), \exists x_0 \subseteq_{fin} x, b \in f(x_0)\\
$$

$$
并且f是单调的a \subseteq b \Rightarrow f(a) \subseteq f(b)，则说f是Scott连续的。
$$

$$
Definition(有向集合). 当D是个偏序集合，如果\forall x, x' \in D,\exists z \in D\\
使得x\subseteq D,x'\subseteq D，则说D是有向集合。
$$

\\(b\\in{f(x)}\\)，意思是给定x，其中x可能是由无限的元素组成的，函数接受x输出一个b；函数是连续的意思就是，其实不需要所有的x，只需要x的一个有限的子集就能同样达到输出b的效果。这就是连续的本质。为了得到函数包含有限信息的输出，只要包含有限信息的输入。有限信息的输入的每个元素的有限渐进的总和组成了输出，其实输入从有限到无限时的输出，没有一个神奇的跳跃，你在无限得到的b，其实在有限的某个时候就已经得到了。如下是证明，首先根据上文的解释翻译一下Scott连续的定义，其中D是有向集合：

$$
f(\bigcup_{x \in D} x) = \bigcup_{x \in D} f(x)
$$

_proof_. 如要证明A=B，可以先证明\\(A\\subseteq{B}\\)，再证明\\(B\\subseteq{A}\\)：

*   **Case** \\(\\bigcup\_{x \\in D} f(x) \\subseteq f(\\bigcup\_{x \\in D} x)\\)：因为\\(x \\subseteq \\bigcup\_{x \\in D} x\\)，根据单调性可得\\(f(x) \\subseteq f(\\bigcup\_{x \\in D} x)\\)，又因为对所有\\(x\\in{D}\\)，\\(f(x)\\)都满足\\(f(x) \\subseteq f(\\bigcup\_{x \\in D} x)\\)，所以\\(\\bigcup\_{x \\in D} f(x) \\subseteq f(\\bigcup\_{x \\in D} x)\\)。Case finished;
*   **Case** \\(f(\\bigcup\_{x \\in D} x) \\subseteq \\bigcup\_{x \\in D} f(x)\\)：假设左边的\\(f(\\bigcup\_{x \\in D} x)\\)的结果是b，根据上面的解释，其实只需要输入的有限子集\\(x\_0\\)就能输出b，\\(x\_0 \\subseteq\_{fin} \\bigcup\_{x \\in D} x\\)，所以\\(b \\in f(x\_0)\\)，假设有一个比\\(x\_0\\)稍微大一点的z，根据单调性有\\(f(x\_0) \\subseteq f(z)\\)，所以有\\(b \\in f(z)\\)，根据有向集合的定义，有\\(z\\in{D}\\)，所以\\(f(z) \\subseteq \\bigcup\_{x \\in D} f(x)\\)。Case finished，得证。

$$
Theorem(Kleene不动点).给定Scott连续的函数f，f的最小不动点是\bigcup_iF^i(\emptyset)
$$

_proof_. 设\\(X=\\bigcup\_iF^i(\\emptyset)\\)，首先证明\\(X\\)是\\(F\\)的不动点–\\(F(X)=X\\)：

$$
\begin{align*}
F(X)&=F(\bigcup_iF^i(\emptyset)) \qquad &\text{By definition of X}\\
&=\bigcup_iF(F^i(\emptyset)) \qquad &\text{By Scott continuity}\\
&=\bigcup_iF^{i+1}(\emptyset) \\
&=\emptyset\cup\bigcup_iF^{i+1}(\emptyset) \\
&=F^0(\emptyset)\cup\bigcup_iF^{i+1}(\emptyset) \\
&=\bigcup_iF^i(\emptyset)\\
&=X
\end{align*}
$$

然后，我们证明\\(X\\)是\\(F\\)的最小不动点。假设\\(Y\\)是\\(F\\)的另一个不动点，我们要证明的是，对所有得i，\\(F^i(\\emptyset)\\subseteq{Y}\\)都成立。当i是0的时候，\\(F^0(\\emptyset)=\\emptyset\\subseteq{Y}\\)；对剩下的，我们先假设有\\(F^i(\\emptyset)\\subseteq{Y}\\)，由单调性可知，所以\\(F(F^{i+1}(\\emptyset))\\subseteq{F(Y)}\\)，因为\\(Y\\)是一个不动点，所以\\(F(Y)=Y\\)，然后\\(F^{i+1}(\\emptyset)\\subseteq{Y}\\)。最后，所有的元素都是\\(Y\\)的子集：

$$
F^0\emptyset\subseteq F^1\emptyset\subseteq \dots
$$

所以所有元素的并集\\(X\\)也属于\\(Y\\)：\\(X=\\bigcup\_iF^i(\\emptyset)\\subseteq Y\\)，故\\(X\\)是\\(F\\)的最小不动点。得证。

#### 指称语义的论证

相比操作语义，指称语义的一大好处就是，在论证等价问题的时候，只需要看程序指称的计算结果就行了，而操作语义则必须把抽象机器底层变换、衍生出的都论证清楚。

比如，要证明\\(skip;c\\)和\\(c;skip\\)是等价的，计算如下：

$$
\begin{align*}
C[skip;c]&=\{(\sigma,\sigma'')|\exists\sigma'.(\sigma,\sigma')\in{C[skip]}\land(\sigma',\sigma'')\in{C[c]}\}\\
&=\{(\sigma,\sigma'')|(\sigma,\sigma'')\in{C[c]}\}\\
&=\{(\sigma,\sigma'')|\exists\sigma'.(\sigma,\sigma')\in{C[c]}\land(\sigma',\sigma'')\in{C[skip]}\}\\
&=C[c;skip]
\end{align*}
$$

利用偏函数，映射关系和集合，证明变得容易多了。再举个例子\\(C\[while\\;false\\;do\\;c\]\\)，根据定义，只需证\\(fix(F)\\)，其中\\(F\\)为：

$$
\begin{align*}
F(f)=&\{(\sigma,\sigma)|(\sigma,false)\in{B[b]}\}\cup\\
&\{(\sigma,\sigma'')|\exists\sigma'.(\sigma,true)\in{B[b]}\land(\sigma,\sigma')\in{C[c]}\land(\sigma',\sigma'')\in{f}\}
\end{align*}
$$

由Kleene不动点定理可得\\(fixF=\\bigcup\_iF^i(\\emptyset)\\)，因为对所有的i，都有\\(F^i(\\emptyset)=\\{(\\sigma,\\sigma)\\}\\)，所以\\(fixF=\\{(\\sigma,\\sigma)\\}\\)，即\\(C\[skip\]\\)。

### 公理语义

#### 什么是公理语义

公理语义用逻辑规则来定义程序（操作语义模型展示程序如何运行，指称语义模型展示程序计算什么）。公理语义最初由Floyd和Hoare提出，后来由Dijkstra和Gries进一步发展。

公理语义通常用前置条件和后置条件来描述程序规则：

$$
\{Pre\}c\{Post\}
$$

其中c是程序，Pre和Post是描述程序状态的公式，通常称之为断言；这种三元逻辑又称之为部分正确规则，或者霍尔三元组，其意思如下：如果在运行c前Pre成立，而且c是能终止的，那么Post在c终止后也成立。也可以这样说，给定满足Pre的store \\(\\sigma\\)，运行c并终止后，Post在输出的store \\(\\sigma'\\)中也成立。

前置条件和后置条件可以看成程序和用户的接口、约定，用户只需关系程序运行的结果，而不用理解程序是怎么运行的。通常，为了使程序更好维护，程序员用写注释的方法给方法、函数添加文档，特别是对那些闭源的库来说，这就是库使用者和库开发者之间的约定。

但是，谁也不能保证写在注释里的前置条件和后置条件是正确的，注释描述的是开发者的意图，而不能保证程序的正确性。公理语义解决的就是这个问题。

公理语义展示了如何描述部分正确语句以及用论证证明程序的程序性。但是，部分正确规则不能保证程序会终止，这也是它叫‘部分’的原因，而完全正确规则保证了程序在满足前置条件时一定会终止，我们用方括号表示完全正确规则：

$$
[Pre]c[Post]
$$

其意思：如果在执行c之前满足Pre，那么c会终止并且终止后满足Post。

大体上，在霍尔三元组中，前置条件指明了在运行程序前的需求，后置条件指明了程序的期望结果（如果程序会终止）。举个例子：

$$
\{foo=0\land bar=i\}baz:=0;while\;foo\neq bar \; do(baz:=baz-2;foo:=foo+1)\{baz=-2i\}
$$

表明了如果有个store中有foo和bar，分别映射到0和i，那么如果程序终止，那么最终的store中应该包含baz到-2i的映射。注意其中i只是个逻辑需要的变量，程序中并没有i，它的作用只是表述bar的初始值，通常称这种变量为‘伪变量’（ghost variables）。

例子中的部分正确语句是正确的：给定任何满足\\(\\sigma(foo)=0\\)的store\\(\\;\\sigma\\)，和

$$
C[baz:=0;while\;foo\neq bar do(baz:=baz-2;foo:=foo+1)]\sigma=\sigma'
$$

则\\(\\sigma'(baz)=-2\\sigma(bar)\\)。注意这只是个部分正确语句，只有程序能终止时才成立，有些初始store不能让程序终止。但下面的完全正确语句是成立的：

$$
[bar=0\land bar=i\land i\geq0]baz:=0;while\;foo\neq bar\;do(baz:=baz-2;foo:=foo+1)[baz=-2i].
$$

表明了如果有个包含foo到0、bar到正整数映射的store\\(\\;\\sigma\\)，那么程序会终止，并输出最终store\\(\\;\\sigma'\\)：\\(\\sigma'(baz)=-2\\sigma(bar)\\)。

我们所讲的公理语义会主要集中在部分正确断言。

#### 断言

*   怎么写断言？怎么描述前置条件和后置条件？
*   一个断言合理是什么意思？一个部分正确语句合理是什么意思？
*   怎么证明部分正确语句是合理的？

怎么表述前置条件或者后置条件？上面的例子中，我们已经用过了程序变量、逻辑相等、逻辑变量、逻辑合取（\\(\\land\\)）。能用什么直接影响到部分正确语句描述出的程序属性，对于IMP，我们用算术表达式之间的比较、逻辑操作符和量词（全部量词和部分量词）来表示断言：

$$
\begin{align*}
i,j&\in LVar \\
a&\in Axep::=x\;|\;i\;|\;n\;|\;a_1+a_2\;|a_1 \\
P,Q&\in Assn::=true\;|\;false\;|\;a_1\lt a_2\;|\;P_1\land P_2\;|\;P_11\lor P_2\;|\;P_1\implies P_2\;|\;\neg P\;|\;\forall i.P\;|\;\exists i.P
\end{align*}
$$

为了表示什么是“断言P在store\\(\\;\\sigma\\)中成立”，我们还要定义一些新东西，因为除了store，我们还需要知道逻辑变量的值，首先我们定义一个逻辑变量到整数的映射I：

$$
I:LVar\to Int
$$

接着定义方法\\(A\_i\[a\]\\)，它是逻辑变量的指称语义：

$$
\begin{align*}
A_i[n](\sigma,I)&=n \\
A_i[x](\sigma,I)&=\sigma(x) \\
A_i[i](\sigma,I)&=I(i) \\
A_i[a_1+a_2](\sigma,I)&=A_i[a_1](\sigma,I)+A_i[a_2](\sigma,I)
\end{align*}
$$

现在我们能表示断言的正确性了，\\(\\sigma\\vDash\_IP\\)，意为，在I解释下，P在store\\(\\;\\sigma\\)中成立：

$$
\begin{align*}
&\sigma\vDash_I true             \qquad\qquad &(always) \\
&\sigma\vDash_I a_1\lt a_2          \qquad\qquad &if\;A_i[a_1](\sigma,I)\lt A_i[a_2](\sigma,I) \\
&\sigma\vDash_I P_1 \land P_2    \qquad\qquad &if\;\sigma\vDash_IP_1\;and\;\sigma\vDash_IP_2 \\
&\sigma\vDash_I P_1 \lor P_2     \qquad\qquad &if\;\sigma\vDash_IP_1\;or\;\sigma\vDash_IP_2 \\
&\sigma\vDash_I P_1 \implies P_2 \qquad\qquad &if\;s\nvDash_IP_1\;or\;\sigma\vDash_IP_2 \\
&\sigma\vDash_I \neg P           \qquad\qquad &if\;s\nvDash_IP \\
&\sigma\vDash_I \forall i.P      \qquad\qquad &if\;\forall k \in Int.\sigma\vDash_{I[i\mapsto k]}P \\
&\sigma\vDash_I \exists i.P      \qquad\qquad &if\;\exists k \in Int.\sigma\vDash_{I[i\mapsto k]}P
\end{align*}
$$

定义了断言的正确性，现在我们就可以定义一个部分正确语句的正确性了：

$$
\forall \sigma'.if\;\sigma\vDash_IP\;and\;C[c]\sigma=\sigma'\;then\;\sigma'\vDash_IQ
$$

当一个霍尔三元组是合理的（写作\\(\\vDash\\{P\\}c\\{Q\\}\\)），那它在所有的store和解释中都是正确的：\\(\\forall \\sigma,I.\\sigma\\vDash\_I\\{P\\}c\\{Q\\}\\)

现在我们知道当我们说断言P成立、部分正确语句\\(\\{P\\}c\\{Q\\}\\)成立是什么意思了。

### 霍尔逻辑

利用公理和推论，我们可以直接推导出合理的部分正确语句，而不用关心store或者程序，这些就叫做霍尔规则，由霍尔规则组成的证明系统就是霍尔逻辑：

$$
{\bf{S}\scriptsize{KIP}}
\frac{}{\{P\}skip\{P\}}\quad
{\bf{A}\scriptsize{SSGN}}
\frac{}{\{P[a/x]\}x:=a\{P\}}
$$

$$
{\bf{S}\scriptsize{EQ}}
\frac{\{P\}c_1\{R\}\quad\{R\}c_2\{Q\}}{\{P\}c_1;c_2\{Q\}}\quad
{\bf{I}\scriptsize{F}}
\frac{\{P\land b\}c_1\{Q\}\quad\{P\land\neg b\}c_2\{Q\}}{\{P\}if\;b\;then\;c_1\;else\;c_2\{Q\}}
$$

$$
{\bf{W}\scriptsize{HILE}}
\frac{\{P\land b\}c\{P\}}{\{P\}while\;b\;do\;c\{P\land\neg b\}}
$$

$$
{\bf{C}\scriptsize{ONSEQUENCE}}
\frac{\vDash(P\implies P')\quad\{P'\}c\{Q'\}\quad\vDash(Q'\implies Q)}{\{P\}c\{Q\}}
$$

while中的断言\\(P\\)是个循环不变式，它既是前置条件又是后置条件，它在循环前后都成立，这一点在while规则的结论中也体现出来了。consequence规则加强了前置条件，弱化了后置条件。

这些霍尔逻辑组成了部分正确语句的归纳定义。当我们能为\\(\\{P\\}c\\{Q\\}\\)构建一个证明树，那么就说它是霍尔逻辑的一个公理，写成\\(\\vdash\\{P\\}c\\{Q\\}\\)。

#### soundness和completeness

现在我们已经有两种断言：

*   合理的部分正确语句\\(\\vDash\\{P\\}c\\{Q\\}\\)。其在所有的store和解释中都成立。
*   霍尔逻辑公理\\(\\vdash\\{P\\}c\\{Q\\}\\)。能从霍尔逻辑的公理和推论中推导出的部分正确语句。

这两者之间有什么关系呢？首先，第一个问题，是不是每个霍尔逻辑公理都是合理的部分正确三元组呢（\\(\\vdash\\{P\\}c\\{Q\\}\\implies\\vDash\\{P\\}c\\{Q\\}\\)）？答案是肯定的，霍尔逻辑是sound的，这一点很重要，这意味着我们不能推导出不成立的部分正确三元组。

第二个问题，对每个合理的断言，我们都能构造出一个霍尔逻辑吗（\\(\\vDash\\{P\\}c\\{Q\\}\\implies\\vdash\\{P\\}c\\{Q\\}\\)）？这个答案也是肯定的，这就是霍尔逻辑相对完整性，由Cook在1974年证明。

#### 例子：阶乘

```
{x=n ∧ n>0}
y:=1;
while x>0 do {
    y:=y*x;
    x:=x-1;
}
{y=n!}
```

我们会用霍尔逻辑证明上面是计算n的阶乘的程序。

要证明上面的程序，因为这是一个包含一个赋值语句和一个while语句的程序，就要利用SEQ规则，于是我们要证明下面的两个三元组：

$$
\{x=n\land n\gt 0\}y:=1\{I\} \\
\{I\}while\;x\gt 0\;do\{y:=y*x;x:=x-1\}\{y=n!\}
$$

但要想利用SEQ规则，要先满足SEQ规则的分子（前提条件），也就是要先找到满足上面两个三元组的\\(I\\)。首先\\(I\\)需要满足在循环前后都满足，我们已经说过，\\(I\\)需要是个循环不变量。只看循环体可知（不要看while的条件，因为我们要找的是循环前后都满足的断言），y的值是先乘x的初始值n，再乘下一个x，也就是n-1，然后一步步累乘的结果：

$$
y=n*(n-1)*\cdots*(x+1)
$$

两边同时乘x！，得到x!*y=n!，当然其中的x是正整数。于是我们得到\\(I\\)：

$$
I=x!*y=n!\land x\geq0
$$

要证明\\(I\\)是循环不变量，即必须满足while规则的前提:

$$
\{I\land x\gt 0\}y:=y*x;x:=x-1\{I\}
$$

要证明上式，我们倒着走一遍：

$$
\{(x-1)!*y=n!\land(x-1)\geq0\}x:=x-1\{I\}
$$

$$
\{(x-1)!*y*x=n!\land(x-1)\geq0\}y:=y*x\{(x-1)!*y=n!\land(x-1)\geq0\}
$$

又因为\\(I\\land x\\gt 0\\implies(x-1)!\*y\*x=n!\\land(x-1)\\geq0\\)，再由CONSEQUENCE规则，可得上式。现在满足了while的前提条件，终于可以使用while规则了，得到：

$$
\{I\}while\;x\gt 0\;do\{y:=y*x;x:=x-1\}\{I\land x\leq0\}
$$

剩下的只需证明\\(I\\land x\\leq0\\implies y=n!\\)：

$$
x!*y=n!\land x\geq0\land x\leq0\implies y=n!
$$

由CONSEQUENCE规则可得第二个三元组。

下面，证前一个三元组。首先，利用不需要前提条件的赋值规则：\\(\\{I\[1/y\]\\}y:=1\\{I\\}\\)，展开得到：

$$
\{x!*1=n!\land x\geq0\}y:=1\{x!*y=n!\land x\geq0\}
$$

又因为\\(x=n\\land n\\gt 0 \\implies x!\*1=n!\\land x\\geq0\\)，再由CONSEQUENCE规则可得第一个三元组。

小结
--

到这里就告一段落了，写了不少，基本介绍了语义，以及证明语义属性的方法。