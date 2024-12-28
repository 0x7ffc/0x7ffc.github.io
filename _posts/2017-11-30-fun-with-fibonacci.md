---
layout: post
title: "Fun with Fibonacci"
description: "Many interesting ways to implement Fibonacci in Haskell which utilize laziness and Functional Programming."
lang: zh
mathjax: true
---

Fibonacci，想试试用Haskell写，没想到[有很多](https://wiki.haskell.org/The_Fibonacci_sequence)有很多有意思的东西。一步一步来，首先是定义：

> 定义：Fibonacci数列是这样`0,1,1,2,3,5,8...`一串满足`F(0)=0`,`F(1)=1`并且当`n>=2`时有`F(n)=F(n-1)+F(n-2)`的数列。能算出`F(n)`的算法有很多，但速度却差别很大：

### 最慢的递归

```haskell
fibs 0 = 0
fibs 1 = 1
fibs n = fib (n-1) + fib (n-2)
```

想象一下展开的树，自顶向下有很多重复的分支，我试了一下，算`fibs 40`都有明显的迟钝了。空间复杂度为$\Theta(n)$，时间复杂度为$\Theta(\phi^n)$，其中$\phi = \frac{\sqrt{5}+1}{2}$，呈指数型增长。

### 快一点的迭代

上来肯定想到的就是动态规划（DP）了。如果已经有`F(k-2)`和`F(k-1)`，那么直接加一起就是`F(k)`，然后重复此过程把`F(k-1)`和`F(k)`加一起得到`F(k+1)`直到`k=n`：

```c
int fib(int n) {
  int a = 0;
  int b = 1;
  while (n-- > 1) {
    int t = a;
    a = b;
    b += t;
  }
  return b;
}
```

自底向上，没有重复的计算，没有膨胀的堆栈，空间、时间复杂度分别为$\Theta(1)$，$\Theta(n)$。这个在Haskell中借助`iterate`有个很有意思的实现。首先要知道Haskell是惰性求值的，比如有个tuple，`(2 + 3, 4)`在其他语言中存在内存里一般就是`(5, 4)`, 而在Haskell里不会计算`2 + 3`，只有访问第一个元素的时候（实际是pattern matching的时候）才会计算，可以把第一个元素想象成一个没有参数的lambda，在访问的时候才会求值：`(<thunk 2 + 3>, 4)`。借助`iterate`我们能生成一个存有所有Fibonacci数的List，用的时候拿就行了。

回头看上面c程序，每次循环会算出下一对`a`和`b`，类比成下面的过程，`x`就是`a`和`b`的tuple，`f`负责算出下一个tuple：

```haskell
x           = (0, 1)
f x         = (1, 1)
f $ f x     = (1, 2)
f $ f $ f x = (2, 3)
...
```

f 就是一个简单的lambda： `f = \(a, b) -> (b, a + b)`

这是Haskell中的iterate：

> iterate f x == [x, f x, f (f x), ...]

借助iterate就能算出所有的tuple：

> iterate (\\(a, b) -> (b, a + b)) (0, 1) ==> [(0,1),(1,1),(1,2),...]

最后就简单了，只要把每个tuple的一个元素拿出来就是所有的Fibonacci数了：

> fibs = map fst $ iterate f x

看起来很正常，但其实有点问题. 可以[看这里](https://www.reddit.com/r/haskell/comments/6dp9iw/is_haskell_doing_some_magic_with_iterate/)，总结下来就是Full Laziness会用更多内存，并带来性能问题，这也就是问什么[wiki](https://wiki.haskell.org/The_Fibonacci_sequence)里尾递归版本使用了[Bang Patterns](https://ocharles.org.uk/blog/posts/2014-12-05-bang-patterns.html)，而`iterate`借助fold/build避免了构建list和pattern matching所带来的性能消耗。

按照这个思路，我试着用ruby实现了一下：

```ruby
iterate = -> (x, &block) do
  Enumerator.new do |yielder|
    loop do
      yielder << x
      x = block.call(x)
    end
  end.lazy
end

x = [0, 1]
f = -> (ab) { [ab[1], ab[0]+ab[1]] }
fibs = iterate.call(x, &f).map(&:first)
```

但很快就stack too big了..., 实际上`f`可以很灵活，比如：

> f = \\(a, b) -> (a + b, a + 2 \* b)

这样product列表就是这样`[(1,1),(2,3),(5,8),...]`(从1开始)，正是所有的Fibonacci数，接下来flatmap就行了。 还有一个很fancy的方法是这个：

> fibs = 0 : 1 : zipWith (+) fibs (tail fibs)

Stackoverflow上有两个答案解释的比较清楚：

* [Understanding concatMap recursion](https://stackoverflow.com/a/31675593)
* [Understanding a recursively defined list (fibs in terms of zipWith)](https://stackoverflow.com/a/6274016)

当你把它塞到[point free converter](http://pointfree.io/)里，得到的是这坨WTF的东西：

> fib = fix ((0 :) . (1 :) . ap (zipWith (+)) tail)

首先`ap (zipWith (+)) tail`其实是`\xs -> zipWith (+) xs (tail xs)`（可以看[这里](https://stackoverflow.com/questions/19181917/how-does-the-expression-ap-zip-tail-work)）。ok，这个fix看名字应该是不动点，根据不动点的定义应该是这样的`fix f = f (fix f)`，但Haskell实际是这么实现的`fix f = let x = f x in x`。[查了一下](https://stackoverflow.com/questions/37366222/why-is-this-version-of-fix-more-efficient-in-haskell)，这样写会快一些。让我奇怪的不是这个，而是：

* 并不是所以的`fix f`都是收敛的，但似乎Haskell通过bottom类型hack了这个问题。
* fix怎么能生成无穷数列，像tail这种函数为什么能用在无穷数列上。

我知道你又要说惰性求值，但是似乎无论用`iterate`还是`zipWith`，无论用Haskell还是Ruby，我感觉这些好像都有某种内在的联系，某种无穷数据类型和递归直接的联系。以前看SICP时没深究，只知道[迭代](https://github.com/ACEMerlin/hackerrank-fp-answers/blob/5602130f741e23707cba9d68ef6869f40bb54f42/recursion/fibonacci.lisp#L10)需要多加点变量来记录计算的状态。现在看了[这篇文章](http://blog.sigfpe.com/2007/07/data-and-codata.html)，才知道它们都是Corecursion。

有关Corecursion的东西可以看[Wikipedia](https://en.wikipedia.org/wiki/Corecursion)，我没大看懂，不过圈一下重点：

* 递归自顶向下拆数据，递归不一定终止，但structural recursion可以终止，利用归纳推理可以证明。
* Corecursion自底向上生成数据，这个过程不一定有限，所以在严格规约下不会终止，所以一般和惰性求值配合使用。生成的数据可以有限或者无限。
* Corecursion在FP语言里很重要，因为可以用来处理无穷数据结构，比如网络流什么的。
* 在命令式语言里，Corecursion一般用[Generator](https://en.wikipedia.org/wiki/Generator_%28computer_programming%29)实现。正好对应着上面Ruby版本里的Enumerator。

往下好像涉及到范畴论什么的，真是让人头大，有时间再看吧。

搞了这么多都是迭代，时间复杂度还是$\Theta(n)$，算第1000000个会很久。想更快就要用数学了。

### Faster!!

$$
\begin{align*}
F(n+1) &= 1\,F(n) + 1\,F(n-1)\\
F(n) &= 1\,F(n) + 0\,F(n-1)\\
\\
\begin{bmatrix} F(n+1) \\ F(n) \end{bmatrix} &= \begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix} \begin{bmatrix} F(n) \\ F(n - 1) \end{bmatrix} \\
\begin{bmatrix} F(n+1) \\ F(n) \end{bmatrix} &= \begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix}^n \begin{bmatrix} F(1) \\ F(0) \end{bmatrix} \\
\\
\text{并且} \\
\begin{bmatrix} F(n) \\ F(n-1) \end{bmatrix} &= \begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix}^n \begin{bmatrix} F(0) \\ F(-1) \end{bmatrix} \\
\\
\text{于是得到}\\
\begin{bmatrix} F(n+1) & F(n) \\ F(n) & F(n-1) \end{bmatrix} &= \begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix}^n \begin{bmatrix} F(1) & F(0) \\ F(0) & F(-1) \end{bmatrix} \\
\\
\text{其中} \\
F(1) &= 1 \\
F(0) &= 0 \\
F(-1) &= 1
\end{align*}
$$

右边的矩阵是个单位矩阵，于是就得出结论啦：

$$
\begin{pmatrix}
1 & 1\\
1 & 0
\end{pmatrix}^n
=
\begin{pmatrix}
F_{n+1} & F_n\\
F_{n} & F_{n-1}
\end{pmatrix}
$$

于是算第n个Fibonacci数的问题就简化成了求矩阵n次方的问题，注意这里要用矩阵快速幂算法才能达到\\(\\Theta(logN)\\)（可参考TAOCP卷二4.63节），不然就会退化成动态规划。

Haskell的实现利用Num的特性，只需实现`*`就行了：

```haskell
newtype Mat a =
  Mat [[a]]
  deriving (Eq, Show)

instance Num a =>
 Num (Mat a) where
  Mat x * Mat y =
    Mat
      [ [ sum $ zipWith (*) xs ys
	| ys <- transpose y ]
      | xs <- x ]
  negate = undefined
  (+) = undefined
  abs = undefined
  fromInteger _ = undefined
  signum = undefined

fib_matrix 0 = 0
fib_matrix n = l $ Mat [[1, 1], [1, 0]] ^ n
  where l (Mat a) = head a !! 1
```

[这里](https://www.nayuki.io/page/fast-fibonacci-algorithms)还有个据说更快的方法Fast doubling，利用上面的矩阵能推导出来，时间复杂度同样是$\Theta(logN)$，似乎快那么一点。

嗯，Fibonacci就这样了，茴香豆的茴字有很多种写法。

有空还是要看看范畴论，不然总感觉缺了点啥。