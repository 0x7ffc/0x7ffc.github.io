---
layout: post
title: "Solution Bags"
description: "Solution bags is an alternative way of thinking about dynamic programming that yields itself to optimizations using data structures."
---

__A summary of this [video]( https://youtu.be/oaYsWnohXpA?t=139)__.

Solution bags is an alternative way of thinking about dynamic programming that yields itself to optimizations using data structures. Using solution bag we can solve the [shortest subarray with sum at least k](https://leetcode.com/problems/shortest-subarray-with-sum-at-least-k/description/) problem with `O(N)` time complexity. 

Here is the problem description: return the length of the shortest, non-empty, contiguous subarray of an array `a` with sum at least `k`. If `a` only contains positive number then this problem is quite simple which can be solved in `O(N)` time using two pointers technique, but it's not the case here, `a` contains negative number. Then the `O(N^2)` solution is obvious: just iterate through all the subarrays and take advantage of precomputations which in this case is the prefix sum. If we want to optimize it further, we must think differently.

Let's first define what a bag is, you can think of a bag as a space which contains the answer to our problem, the space could be ordered like a stack/queue, or it could be a generic 'bag' which hold no special meaning yet. In this problem I'll define a bag as follow: for a subarray `a` of size `m+1`, the bag `b` is: `{[0,a[0]+...+a[m]], ..., [i,a[i]+...+a[m]], ..., [k,a[m]]}`.  In this single bag we can easily find the minimum length which makes `a[i]+...+a[m]>=k`(or `sum(i,m)>=k` for short). When adding a new item `x` into `a`, we only need to increment the bag with this new item `x` to get the next bag: `b[i][1]+=x for all i`. Given an array we can generate all the bags by adding one item at a time, and for each bag we have `{sum(0,j),...,sum(i,j),...,sum(j,j)}`, and we can find the minimum length which makes `sum(i,j)>=k`.

So far the method above using our fancy bags is still `O(N^2)`, nothing special. But if for each bag we can find some invariant to reduce its size, we can have better time complexity overall:

-  If `sum(i1,j)<sum(i2,j)` and `i1<i2` then there is no point of keeping
   `sum(i1,j)` because `sum(i2,j`) is closer to our goal. In other words,
   we have this invariant: if `i1<i2`, then `sum(i1,j)>sum(i2,j)`.
-  If we find that `sum(i,j)>=k`, then there is no point of keeping
   it in the bag for later use because extending `j` makes `j-i` bigger
   and we already have `sum(i, j)>=k`.

With these two invariants we can achieve much better time complexity already, there is a slight problem though: to get the next bag we have to do `b[i][1]+=x for all i`, and that is pretty slow. To get rid of that we use a technique called delting. We keep an auxiliary variable `delta` outside of the bag to represent the overall change to the bag. Each time we take something out of the bag we add `delta` to that value, and each time before we put something in the bag, we subtract `delta` from that value.

```ruby
def shortest_subarray(a, k)
  delta, b, min = 0, [], Float::INFINITY
  a.size.times do |j|
    delta += a[j]
    b.pop while !b.empty? && b[-1][1]+delta < a[j] # first invariant
    b << [j, a[j]-delta]
    last = b.shift while !b.empty? && b[0][1]+delta >= k # second invariant
    min = [min, j-last[0]+1].min if last
  end
  min == Float::INFINITY ? -1 : min
end
```

For each item, we add it to the bag once and remove from bag once, so the total time complexity is `O(N)`.

Let's use solution bag to solve an easy problem: [Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/description/). This time we'll keep a bag of monotonically decreasing numbers.

1. If `i<j` and `nums[i]<nums[j]`, then no matter what window
   `nums[i]` is in, it won't contribute to the maximum number inside
   that window. In other words we need `i<j` && `nums[i]>=nums[j]`.
2. We need to remove the first number from the bag while sliding to
   the right if it equals to the maximum number in the bag.

```ruby
def max_sliding_window(nums, k)
  q, ans, i = [], [], 0
  while i < nums.size
    q.pop while !q.empty? && q[-1] < nums[i] # first invariant
    q << nums[i]
    q.shift if i >= k && nums[i-k] == q[0] # second invariant
    ans << q[0] if i >= k-1
    i += 1
  end
  ans
end
```

Anyway, I hope you get the idea.
