---
layout: post
title: "Union Find"
description: "Here I demonstrate Ruby implementation of the Weighed Quick-Union find data structure and solve some leecode problems with it."
---

The motivation behind Union-Find is that given some points on an axis, you want to be able to group some points together and answer the query of whether two points are in the same group. It works on 2D panel as well since you can always transform 2D point to 1D.

Here is the Ruby implementation of the weighed quick-union with path compression described [here](https://www.coursera.org/learn/algorithms-part1/lecture/ZgecU/quick-union).

```ruby
class UF
  def initialize(size)
    @arr = Array.new(size, &:itself)
    @weight = Array.new(size, 1)
  end

  def connected?(p, q)
    root(p) == root(q)
  end

  def root(p)
    until p == @arr[p]
      @arr[p] = @arr[@arr[p]]	# path compression
      p = @arr[p]
    end
    p
  end

  def union(p, q)
    i, j = root(p), root(q)
    return if i == j
    if @weight[i] < @weight[j]	# union according to weight
      @arr[i] = j
      @weight[j] += @weight[i]
    else
      @arr[j] = i
      @weight[i] += @weight[j]
    end
  end
end
```

[Leetcode 130](https://leetcode.com/problems/surrounded-regions/description) is an interesting problem that can be solved using Union-Find: Given a 2D board containing 'X' and 'O' (the letter O), capture all regions surrounded by 'X'. A region is captured by flipping all 'O's into 'X's in that surrounded region. The trick is to create a dummy node which connects to all border nodes. This trick is pretty common when solving this kind of problem. Using Union-Find, the idea is simple:

1. Union all the 'O's together.
2. Union all 'O's on border to the dummy node.
3. Flip all 'O's to 'X's which are not connected to the dummy node.

```ruby
def solve(board)
  return if board.empty?	# edge case
  w, h = board[0].length, board.length
  uf = UF.new(w*h+1)		# +1 for the additional dummy node
  dummy_node = w*h		# use the last unused index as the dummy node
  region = []			# all the 'O's indices are stored here
  (0..(w*h-1)).each do |i|
    if board[i / w][i % w] == 'O'
      region << i
      neighbors = []
      neighbors << i+1 unless ((i+1) % w).zero?       # right
      neighbors << i-1 unless (i % w).zero?	      # left
      neighbors << i-w unless i >= 0 && i < w	      # up
      neighbors << i+w unless i >= w*(h-1) && i < w*h # down
      uf.union(i, dummy_node) unless neighbors.length == 4
      neighbors.each do |n|
	uf.union(n, i) if board[n / w][n % w] == 'O'
      end
    end
  end
  region.each do |i|
    board[i / w][i % w] = 'X' unless uf.connected?(i, dummy_node)
  end
end
```

Here are a list of similar problems, I'll leave them to you:

- [Number of Islands](https://leetcode.com/problems/number-of-islands/description)
- [Max Area of Island](https://leetcode.com/problems/max-area-of-island/description)
- [Most Stones Removed with Same Row or Column](https://leetcode.com/problems/most-stones-removed-with-same-row-or-column/description)

These problems can also be solved using DFS with a trick I call "Sink the Island", I'll talk about it in later article.