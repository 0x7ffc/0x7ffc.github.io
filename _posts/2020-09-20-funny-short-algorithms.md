---
layout: post
title: "Funny Short Algorithms"
description: "Some funny short algorithms that you probably don't know."
mathjax: true
---

Here are some funny short algorithms that you probably don't know.

## Jay Kadane's Algorithm

Let's see the problem first: given an integer array `nums`, find the contiguous subarray (containing at least one number) which has the largest sum and return its sum. A naive $O(n^3)$ solution would be iterating through all the subarrays. We can optimize to $O(n^2)$ by using prefix sum. Upon further inspection, we can come up with this $O(n)$ solution:

1. Let `S` be the prefix sum: `s[i] == nums[0]+nums[1]+...+nums[i]`.
2. Given `r`, we want to find such `l <= r`, so that `s[r]-s[l-1]` is
   maximal, clearly we need `s[l-1]` to be minimum of `s[0..r-1]`.
3. Better yet, we don't actually need to build `S`, we just need
   the current prefix sum and the minimum prefix sum before it.

```ruby
def max_sub_array_a(nums)
  ans = -Float::INFINITY
  sum = min_sum = 0
  (0...nums.size).each do |i|
    sum += nums[i]
    ans = [ans, sum-min_sum].max
    min_sum = [min_sum, sum].min
  end
  ans
end
```

The above function only return a numerical answer, we can find the
boundaries of the subarray as well:

```ruby
def max_sub_array_indices_a(nums)
  ans = -Float::INFINITY
  lo = hi = 0
  sum = min_sum = 0
  min_p = -1
  (0...nums.size).each do |i|
    sum += nums[i]
    k = sum - min_sum
    if k > ans
      ans = k
      lo = min_p + 1
      hi = i
    end
    if sum < min_sum
      min_sum = sum
      min_p = i
    end
  end
  [lo, hi]
end
```

[Jay Kadane's algorithm](https://en.wikipedia.org/wiki/Maximum_subarray_problem#Kadane's_algorithm) basically has tha same idea as before except that if at some point `s[i]` is negative, we just assign `s[i]=0`. Why you ask? Funny you should ask because I don't know. This version of the algorithm will return 0 if the input contains no positive elements so keep that in mind. With small modifications it can also keep track of the starting and ending indices, but I'll leave that one to you.

```ruby
def max_sub_array(nums)
  ans = -Float::INFINITY
  sum = 0
  (0...nums.size).each do |i|
    sum += nums[i]
    ans = [ans, sum].max
    sum = [sum, 0].max
  end
  ans
end
```

## Boyer–Moore majority vote algorithm

The problem is simple: given an array of size `n`, find the majority element. The majority
element is the element that appears more than `n/2` times. Let's just ignore the `more than n/2`
part for now and find the element that occurs most often. It's quite simple if we use a hash table:

```ruby
def majority_element(nums)
  freq, max_f, max_n = Hash.new(0), 0
  nums.each do |n|
    freq[n] += 1
    max_f, max_n = freq[n], n if freq[n] > max_f
  end
  max_n
end
```

[Boyer-Moore's algorithm](https://en.wikipedia.org/wiki/Boyer%E2%80%93Moore_majority_vote_algorithm) on
the other hand doesn't require extra space. The picture on the wiki illustrates this algorithm very well.

```ruby
def majority_element(nums)
  i, m = 0
  nums.each do |x|
    if i == 0
      m = x
      i = 1
    elsif m == x
      i += 1
    else
      i -= 1
    end
  end
  m
end
```

If there is a majority element, the algorithm will always find it. 
However keep in mind that even when the input sequence has no majority, the algorithm
will report one of the sequence elements as its result, and it doesn't guaranteed
to be the element that occurs most often.

## Floyd's Tortoise and Hare

Cycle detection problem: given $x_0; x_1=f(x_0); x_2=f(x_1); ...; x_i=f(x_{i-1})$, find if it has
cycle in it, that is there exists `i` and `j` such that $x_i==x_j$. Of course you can do that with
 a hash table, but [floyd's algorithm](https://en.wikipedia.org/wiki/Cycle_detection#Floyd's_tortoise_and_hare) can solve this using constant space and much less equality tests, see wikipedia for a detailed explanation.

```ruby
def has_cycle?(x)
  t = f(x)
  h = f(f(x))
  while t != h && some_stop_condition
    t = f(t)
    h = f(f(h))
  end
  t == h
end
```

For example you can use this algorithm when you want to determine if a linked list has a cycle in it:

```ruby
def hasCycle(head)
  return false if head.nil? || head.next.nil?
  slow, fast = head, head.next
  while !fast.nil? && !fast.next.nil?
    return true if fast == slow
    slow, fast = slow.next, fast.next.next
  end
  false
end
```

In this case the tortoise is `slow = slow.next` and the hare is `fast = fast.next.next`. If you
understand how floyd's algorithm works, then you'll have no trouble explain above code to others.