---
layout: post
title: "Fibonacci and Catamorphism"
description: "Catamorphism is an interesting concept in Category Theroy. I explore these ideas and in the end implement Fibonacci using Catamorphism."
lang: zh
mathjax: true
---

没想到吧，Fibonacci还能编出Part2。实际上这个Part2跟Fibonacci没啥大关系，千错万错我不该去看什么Category Theroy，又费解又没用，现在满脑子都是Recursion Scheme，我真是闲。

首先看大佬们的文章吧：

* [Category Theory for Programmers](https://bartoszmilewski.com/2014/10/28/category-theory-for-programmers-the-preface/)
* [Recursion Schemes](http://blog.sumtypeofway.com/)
* [Understanding F-Algebras](https://www.schoolofhaskell.com/user/bartosz/understanding-algebras)

看完这些还是半懂不懂，当时看的时候有段时间一直想不明白Category Theroy和Haskell里的类型到底有啥对应关系，总结一下比较关键的东西：

* Category里的object对应着Haskell类型，morphism对应函数。
* 给定一个category C，C的一个object‘长啥样’取决于它跟其它所有C的object的关系，比如你在族谱里是谁就取决于你跟其它人的关系。族谱有个functor是映射族谱里的名字到真人，这个functor既要映射名字（object），也要映射族谱里的关系（morphism）。
* Functor是categories之间的morphism，可以把functor想象成容器，fmap改容器里的东西，但保留结构（preserve structure）。
* Natural transformation是functor之间的morphism，它改变容器，不动内容。
* Haskell里所有的functor都是endofunctor（？）。
* 在Haskell里，没人喜欢谈论bottom。
* Kind是`* -> *`的要么是covariant functor，要么是contravariant functor（Thanks ADT）。
* 因为ADT的product和coproduct都是functor，GHC利用这一点提供了`DeriveFunctor`给你自动实现covariant functor的fmap。
* Monad的`return :: a -> m a`和`join :: m m a -> m a`都是natural transformation，前者是因为有`Identity` functor： `return :: Identity a -> m a`。
* 因为有参数多态的存在，Category Theroy里的那些law在Haskell里都是白拿的，theorem for free。

Haskell里类型的不动点我感觉和数学里的不大一样。数学里满足$f(x)=x$的x是不动点；在程序语言里，不动点意味着不停的自我应用，举个例子，我们有这个类型`NatF`：

```haskell
data NatF a  = ZeroF | SuccF a deriving Functor
```

明眼人已经看出来这就是`Maybe`。这里的`a`我可以选任何类型，当然这个任何包括`NatF`自己：

```haskell
one = SuccF ZeroF
```

这只是自我应用了一层，当然你可以有很多层：

```haskell
five = SuccF (SuccF (SuccF (SuccF (SuccF ZeroF))))
```

明眼人又看出来了，这不就是丘奇数嘛。`NatF`中的F是为了指出这是个Functor，我们取这个Functor的不动点，这个不动点于下面的类型同构：

```haskell
data Nat = Zero | Succ Nat
```

感觉这种对Functor应用自己的套路就是Functor的不动点，`Maybe`的不动点就是自然数。再举个例子比如`data Pair a b = Pair a b`（就是个Product）的Functor`Pair a`的不动点其实跟List同构。`Nat`就是个丘奇数，各种运算可以定义起来了：

```haskell
plus m Zero = m
plus m (Succ n) = Succ $ plus m n
minus m Zero = m
minus (Succ m) (Succ n) = minus m n
mult m Zero = Zero
mult m (Succ n) = plus m $ mult m n

-- 其实这个时候我们就能定义Fibonacci了

fib Zero = Zero
fib (Succ Zero) = Succ Zero
fib (Succ (Succ n)) = plus (fib (Succ n)) (fib n)
```

另一种方法是对自然数进行结构归纳（structural recursion），记得数学归纳法嘛，如果`n=1`时`P(n)`成立，并且有`n=k,P(k) => n=k+1,P(k+1)`那么P对所有的自然数都成立。这里就要感谢Curry和Howard了，程序即证明，structural induction对应到程序里是fold，数学归纳对应到程序里是在自然数上fold。

```haskell
-- f 0 = c
-- f 1 = h c
-- f (n+1) = h (f n) = h ....... h c
-- Haskell定义如下

foldn c _ Zero = c
foldn c h (Succ n) = h $ foldn c h n
```

利用Curring，可以重新实现上面的运算：

```haskell
plus' m = foldn m Succ
mult' m = foldn Zero $ plus' m
```

有一种理解foldn的方法，把丘奇数x的Succ换成h，Zero换成c，结果就是`foldn c h x`，比如`mult (Succ Zero) (Succ (Succ Zero))`就是把第二个参数的所有Succ换成`plus (Succ Zero)`（Zero没变），于是就是加了两次`Succ Zero`，得到`Succ (Succ Zero)`(2)。

为了方便，我们可以把它实现成`Num`的实例：

```haskell
instance Num Nat where
  (+) = plus
  (-) = minus
  (\*) = mult
  fromInteger n = case n of
		    0 -> Zero
		    n -> Succ (fromInteger (n - 1))
  abs = undefined
  signum = undefined
```

这样就可以直接写阿拉伯数字，而不是复杂的类型：

```haskell
foldn c _ 0 = c
foldn c h n = h $ foldn c h (n-1)
plus' m = foldn m (+1)
mult' m = foldn 0 $ plus' m
-- mult' 2 3 => 6
```

利用foldn可以实现Fibonnaci，基本和上节的动态规划一样了：

```haskell
fib = fst . foldn (0, 1) f
  where f (m, n) = (n, m + n)
```

话说回来这个Nat是个递归数据类型，想搞个`Functor`都没有。回头看`NatF`，你是`* -> *`，要是能用你就好了，这样`Functor`，`Traversable`，`Foldable`啥的就都有了，就是有个问题，每次在`NatF`应用自己它类型都会变，需要给个类型表示‘NatF的不动点’，参考`f(x)=x`：

> newtype Fix f = Fix (f (Fix f))

x是Fix f，f就是f，这样Nat就是：

```haskell
type Nat = Fix NatF

example1 :: Nat
example1 = Fix (SuccF (Fix (SuccF (Fix ZeroF))))  -- => 2
```

为了方便，把Fix换个写法：

```haskell
newtype Fix f = Fix { unFix :: f (Fix f) }

-- 相当于两个方法
-- Fix加一层  :    Fix :: f (Fix f) -> Fix f
-- unFix剥一层:  unFix :: Fix f -> f (Fix f)
```

接下来我们定义一个方法，如果它知道如何把`NatF a`简化成`a`，那它就知道怎么把`Nat`简化成a：

> cata :: (NatF a -> a) -> (Nat -> a)

对于形如`F a -> a`的函数类型，我们称之为F-algebra，把箭头反过来`a -> F a`就是F-coalgebra，具体为啥看[这里](https://stackoverflow.com/questions/16015020/what-does-coalgebra-mean-in-the-context-of-programming)。

在实现cata之前，先看下面这个东西：

$$\require{amsCd}$$

$$
\begin{CD}
F\;a @>fmap\,g>> F\;b \\
@V alga VV @VV algb V \\
a @>>g> b
\end{CD}
$$

非常直观，有两个Algebra：alga和algb，举个例子：

```haskell
alga :: Maybe Int -> Int
alga (Just x) = x
alga Nothing = 0

algb :: Maybe Char -> Char
algb (Just c) = c
algb Nothing = '0'

g :: Int -> Char
g = intToDigit
```

这个图应该满足：先拆包装然后改内容(path2)，和先fmap改内容然后拆包装(path1)应该是一样的：

```
path1 = algb . fmap g
path2 = g . alga

-- path1 ~ path2 : path1 is isomorphic to path2
```

要是这个alga能反过来多好，这样我们就能用3条边定义g了：

```haskell
m' = algb . fmap g . alga_inv
```

你可以强行写出来一个，不过并不是所以的方法都能反过来的：

```
alga_inv :: Int -> Maybe Int
alga_inv 0 = Nothing
alga_inv x = Just x
```

下一步，该想想cata咋实现了。我们把这个图里的a用`Fix F`替代：

$$
\begin{CD}
F\;(Fix\;F) @>fmap\,g>> F\;a \\
@VFixVV  @VValgV \\
Fix\;F @>>g> a
\end{CD}
$$

看看cata的函数类型，`(NatF a -> a)`不就是右边的alg吗，`Nat -> a`不就是`Fix F -> a`吗，这个图就差一个东西了，就是`Fix F -> F (Fix F)`，要是有这个东西，三条边整一起不就能递归定义g了，回头看Fix的定义，这玩意就是`unFix`啊！

$$
\begin{CD}
F\;(Fix\;F) @>fmap\,g>> F\;a \\
@AunFixAA  @VValgV \\
Fix\;F @>>g> a
\end{CD}
$$

这种`F(Fix F)`和`Fix F`同构的叫做initail algebra。这个图满足：

```haskell
g = alg . fmap g . unFix
```

Nice，翻译成Haskell：

```haskell
type Algebra f a = f a -> a

cata :: (Functor f) => Algebra f a -> Fix f -> a
cata alg = alg . fmap (cata alg) . unFix
```

Catamorphism和fold干的事情是一样的，但是更general，对所有Functor都能用，我是这么理解的：Catamorphism就像AST的一个个解释器，AST的结点是Functor，解释器的意义由alg定义，fmap会遍历求值每个子结点，有点像面向对象里的Visitor模式，最后AST会坍塌成一个结果。比如你想美化打印AST，最后会得到一个字符串，再比如有个四则运算的AST，你想求值运算的结果，最后会得到一个数字。首先unFix拆开得到Functor，fmap进到Functor里递归的eval，最后自底向上一层层alg的结果上来把Functor树坍塌成一个结果。

理解了这些，是时候搞点事了：

```haskell
-- 求值成Int（这里的Nat是Fix NatF）
eval_int :: Nat -> Int
eval_int = cata phi where
  phi ZeroF = 0
  phi (SuccF x) = x + 1

-- eval_int (Fix (SuccF (Fix (SuccF (Fix (SuccF (Fix (SuccF (Fix ZeroF)))))))))
-- => 4

-- 加法
plus :: Nat -> Nat -> Nat
plus n = cata phi where
  phi ZeroF = n
  phi (SuccF m) = Fix $ SuccF m

-- eval_int $ plus (Fix (SuccF (Fix (SuccF (Fix ZeroF))))) (Fix (SuccF (Fix ZeroF)))
-- => 3

-- Pretty Print
import Text.PrettyPrint (Doc, render)
import qualified Text.PrettyPrint as P

pretty :: NatF Doc -> Doc
pretty ZeroF = P.text "0"
pretty (SuccF x) = P.parens (P.cat [P.text "+ 1 ", x])

instance Show Nat where
  show = render . cata pretty

-- show $ plus (Fix (SuccF (Fix (SuccF (Fix ZeroF))))) (Fix (SuccF (Fix ZeroF)))
-- => "(+ 1 (+ 1 (+ 1 0)))"
```

如果我们把图里的箭头都反过来，我们可以得到一个叫Anamorphism的东西，Catamorphism把复杂的东西简化，Anamorphism从简单的东西搭建出复杂的类型：

```haskell
type CoAlgebra f a = a -> f a

ana :: Functor f => CoAlgebra f a -> a -> Fix f
ana coalg = Fix . fmap (ana coalg) . coalg
```

举个例子，我们可以从Int反向造出一个`Nat`来：

```haskell
toNatF :: Int -> Nat
toNatF = ana phi where
  phi 0 = ZeroF
  phi n = SuccF (n-1)

-- toNatF 3
-- => (Fix (SuccF (Fix (SuccF (Fix (SuccF (Fix ZeroF)))))))
```

最后试试Fibonacci：

```haskell
fib_cata = fst . cata phi where
  phi ZeroF = (0, 1)
  phi (SuccF (a, b)) = (b, a+b)

-- fib_cata $ toNatF 20
-- 6765
```

Catamorphism就这么多了，以后说不定会编个Part 3出来， 再说吧。