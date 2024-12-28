---
layout: post
title: "Binary Search"
description: "Binary search although easy to describe, but hard to write it bug free. Here I'll explore the idea of Discrete Binary Search and solve some leetcode problems with it."
---

When I was learning binary search and saw other people's implementation, I was confused, I'm sure you did as well. Let's clear it up. Binary search in the first glance is used to find a value in a sorted sequence, you may write something like this:

```ruby
def bs(arr, target)
  lo, hi = 0, arr.size-1
  while lo <= hi
    mid = lo + (hi - lo) / 2
    if arr[mid] == target
      return mid
    else if arr[mid] < target
      lo = mid + 1
    else
      hi = mid - 1
  end
  // target not found
end
```

Easy right? However sometimes you need more than finding a target. Sometimes you may not have a sequence like an array. Let's take it further, array lookup is essentially a function: given x, f(x) is the search target, and the equality check for finding the target is also just a predicate over f(x): given predicate p, we want to find the first x such that p(f(x)) is true:

```ruby
def bs(f, p, lo, hi)
  while lo < hi
    mid = lo + (hi - lo) / 2
    if p(f(mid))
      hi = mid
    else
      lo = mid + 1
    end
  end
  return f(lo) if p(f(lo))
end
```
The two crucial lines are `hi = mid` and `lo = mid+1`. When `p(mid)` is true, we can discard the second half of the search space, since the predicate is true for all elements in it. However, we can not discard mid itself, since it may well be the first element for which p is true. In a similar vein, if `p(mid)` is false, we can discard the first half of the search space, but this time including mid. `p(mid)` is false so we don’t need it in our search space. This effectively means we can move the lower bound to mid+1. Also notice that we use `lo < hi` stead of `lo <= hi`.

For an example, see [Leetcode problem 74](https://leetcode.com/problems/search-a-2d-matrix/description):

```ruby
def search_matrix_a(matrix, target)
  return false if matrix[0].nil? || matrix[0].empty?
  sub = bsearch(matrix) { |sub| target <= sub[-1] }
  found = bsearch(sub) { |x| x >= target }
  !found.nil? && found == target
end

def bsearch(objs)
  return if objs.nil? || objs.empty?
  lo, hi = 0, objs.length-1
  while lo < hi
    mid = lo + (hi-lo) / 2
    if yield(objs[mid])
      hi = mid
    else
      lo = mid + 1
    end
  end
  return objs[lo] if yield(objs[lo])
end
```

A similar problem is [Leetcode 378](https://leetcode.com/problems/kth-smallest-element-in-a-sorted-matrix/description). Given a matrix: [⁠[1, 5, 9], [10, 11, 13], [12, 13, 15]]. When `k=8`, `mid=(1+15)/2=8`, for each row we find the index of the first element that is larger than mid. Then sum up all the indices and compare it with k: `2+0+0 < k`, which means mid is too big, we need to go 'left'. If it equals k, it means that we have k numbers that smaller or equal than that number, which is exactly what we want -- the kth smallest number. Here we use ruby's build-in `bsearch` method.

```ruby
def kth_smallest(matrix, k)
  lo, hi = matrix[0][0], matrix[-1][-1]
  (lo..hi).bsearch do |m|
    matrix.map do |x|
      (0..x.length).bsearch { |i| i == x.length || x[i] > m }
    end.sum >= k
  end
end
```

For a more interesting problem see [leetcode 1014](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/description), we want to find the least weight capacity(the search space) of the ship such that packages can be shipped within D days(the predicate):

```ruby
def ship_within_days(weights, d)
  lo, hi = weights.max, weights.sum
  (lo..hi).bsearch do |load|
    days_needed(weights, load) <= d
  end
end

def days_needed(weights, load)
  current_load, days= 0, 1
  weights.each do |w|
    current_load += w
    if current_load > load
      days += 1
      current_load = w
    end
  end
  days
end
```

In conclusion, binary search is much more than something for searching elements in a sorted array. Just keep these steps in mind:

- Find the search space and determine the lower and upper bounds.
- Design the predicate.
- Deal with edge cases.
- Write and test your binary search.