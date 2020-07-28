---
layout: post
title: "Things I Wish I Knew About Bitwise Operators"
description: "Lowest one bit and other common bitwise hacks are interesting yet difficult to understand. This article provides some explanation and motivation behind some hacks."
---

First you need to see [this](https://graphics.stanford.edu/~seander/bithacks.html) amazing article. If you haven't, I don't know what to say, it's your loss.

## Lowest set bit

Signed integers are commonly represented using two's complement, and the two's complement is calculated by inverting the digits and adding one. For example:

```
010       + 110       = 1000
010       + (101 + 1) = 1000
(001 + 1) + 110       = 1000
```

When you add 1 to a number, it'll set all bits to 0 up to the first 0 and change that 0 to 1, which means that bit at the original number before inverting is 1. So the negation of the number has the same "right part", up to and including the lowest set bit, but everything to the left of the lowest set bit is the complement of the input. Thus the bitwise AND of a number with its negation yields the lowest set bit.

```c
int lowest_set_bit(int n) {
  return n & -n;
}
```

You may wonder when this'll be useful, go check out `Fenwick Tree`. You can use `Fenwick Tree` to solve this [problem](https://leetcode.com/problems/range-sum-query-mutable/description/)

---

## Highest set bit

The idea is simple, set all bits right of the highest set bit to 1, then shifts right and subtracts that.

```c
int highest_set_bit(int n) {
  n |= (n >> 1);
  n |= (n >> 2);
  n |= (n >> 4);
  n |= (n >> 8);
  n |= (n >> 16);
  return n - (n >> 1);
}
```

Also GCC provides `__builtin_clz`.

One usage of this is when you want to do binary search on `Fenwick Tree`.

---

## x modulo 2^n

Of course you can just do `x % (2^n)` but the modulo operation often is much slower than other arithmetic operators. Can you be faster? Think about it, when you're working with base-10 numbers, `x % 10` gives you the last digit of `x`, `x % 100` gives you the last two digits and so on. So with base-2 number, `x % (2^n)` actually gives you the last n bits of `x`.

```
(x % (2^n)) === (x & (2^n - 1))
```

Once again you may wonder what's the usage of this. Well if you want to implement hash table, one thing you need to decide on is the growth factor. One may choose [1.5](https://stackoverflow.com/a/1100426), and the benefit is that you can reuse the memory holes after you free the memory, but it can be solved if you have a compact garbage collector. Another pretty common growth factor is 2, thats where this trick comes into play. With hash table, you first need to calculate the hash, then module that with the length of the array to get the index of where you actually put the data. Since modulo operation is expensive on most cpu architecture, it'll have an massive impact on the hash table performance, not if you use the logic `AND` which is a pretty cheap operation.