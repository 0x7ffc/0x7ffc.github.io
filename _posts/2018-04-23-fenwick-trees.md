---
layout: post
title: "Fenwick Trees"
description: "Fenwick Tree (Binary Indexed Tree) is a common data structure used to solve range sum problem. Here I provide some intuition and solves some leetcode problems with it."
---

Given some sequence and you want to determine range sum queries, for example:

```
[2 4 1 5 6 7 7]

range_sum(3, 3) => 5
range_sum(2, 5) => 19
```

Of course the easiest way to do this is just run a for loop on the sequence and query each of the array locations. And the worst case would be a query on the entire array which is O(N) and that's a big no-no.

And the next easy step is to memorize more information by doing some pre computation, for example by computing prefix sums:

```
[2 4 1 5  6  7  7 ]
[2 6 7 12 18 25 32]

range_sum(2, 5) => 25 - 6 = 19
```

But if we want to do it dynamically, aka we might change the sequence, which means we've to recompute the prefix sums and that's O(N) which makes it another big no-no. Now you can see the real motivation behind fenwick tree: we want a data structure that stores information at various array locations and then we can easily compute a prefix sum up to including that position.

For an typical array an element at certain index is only responsible for itself. Fenwick tree however is organized by the lowest one bit and it has a range of responsibility instead of just being responsible for its single index. For example when we look at index 6(one-based array) which in binary is `0110`, and lowest one bit `0010`(2) says that it's responsible for two indices.

```
1000       |
 111 |     |
 110   |   |
 101 | |   |
 100     | |
 011 |   | |
 010   | | |
 001 | | | |
```

For example, in order to get the prefix sum up to index `110`(6), we first get its value which is responsible for two elements a[6] and a[5], then get rid of the lowest one bit and get `100` which its index is responsible for the first 4 elements. In short we staircase down until no bits are left. The time complexity is `O(log(N))` where N is the number of bits.

And if we want to update certain element in the sequence, we can just draw a straight line to the right at that index and see all the different cells that the fenwick tree must update. For example for `101` we need to update three cells -- `101`, `110` and `1000`. To get the next cell we just have to add the current index's lowest one bit to the current index, `101 + 001 => 110`, `110 + 010 => 1000`. The time complexity of update is `O(log(N))` where N is the number of bits.

As you can see the lowest one bit is important, and the easiest why to calculate it is `i & -i` where i is the index(see [here](/2018/things-I-wish-I-knew-about-bitwise-operators) for detailed explanation).

The implementation is really simple though.

```ruby
# API is zero-based, implementation is one-based.

class BIT

  def initialize(arg)
	@size = arg + 1
	@table = Array.new(@size, 0)
  end

  # update position i by delta
  def update(i, delta)
	i += 1
	while i < @size
	  @table[i] += delta
	  i += lowest_one_bit(i)
	end
  end

  # compute the prefix sum value[0, i]
  def sum(i)
	i += 1
	sum = 0
	while i > 0
	  sum += @table[i]
	  i -= lowest_one_bit(i)
	end
	sum
  end

  # compute the range sum value[i, j]
  def range_sum(i, j)
	sum(j) - sum(i-1)
  end

  private

  def lowest_one_bit(i)
	i & -i
  end
end
```

There are some interesting problems you can solve using Fenwick Tree, for example [leetcode problem 315](https://leetcode.com/problems/count-of-smaller-numbers-after-self/description), basically a counting inversions problem. We can convert this problem to range sum query problem, thus using Fenwick Tree:

1. Compress the input by rank, map the input to its rank: [5,2,6,1] => [2,1,3,0], the result would be the same.
2. Iterate compressed input from right to left, given current rank x, update Fenwick Tree at index x by 1 since it represents that we've seen the number one more time, the count of elements smaller to the right of x would just be the range sum of Fenwick Tree from 0 to x-1, hence bit.sum(x-1).

```ruby
def count_smaller(nums)
  rank = {}
  Set.new(nums.sort).each_with_index { |n, i| rank[n] = i }
  bit = BIT.new(rank.size)
  [].tap do |r|
	nums.map { |n| rank[n] }.reverse_each do |i|
	  bit.update(i, 1)
	  r.unshift(bit.sum(i-1))
	end
  end
end
```

The time complexity is `O(N*log(N))` which is better than a naive implementation using two for loops. [Leetcode problem 493](https://leetcode.com/problems/reverse-pairs/description) is another variation of the same problem, I'll leave it to you. See also [Leetcode problem 327](https://leetcode.com/problems/count-of-range-sum/description) for a more complex example.

Another data structure commonly used to deal with range queries is Segment Tree, it's more flexible and powerful. We'll talk about it in another article.
