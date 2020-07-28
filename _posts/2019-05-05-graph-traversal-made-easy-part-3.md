---
layout: post
title: "Graph Traversal Made Easy - Part 3"
description: "Part 3 of a comprehensive guide to Graph Traversal algorithms which includes (Bi-Directional) Breadth First Search and solves some leetcode problems with it."
---

DFS is often used to explore the whole graph. BFS on the other hand explore the graph layer by layer and that'll help us finding the shortest paths. It also can tells us which vertices are unreachable from s, in that case the shortest path is infinity.

BFS by itself is simple:

```ruby
def bfs(s, adj)
  level = { s=>0 }
  parent = { s=>nil }
  i = 1
  frontier = [s]
  while !frontier.empty?
    next = []
    frontier.each do |u|
      adj[u].each do |v|
	next if level.has_key?(v)
	level[v] = i
	parent[v] = u
	next << v
      end
    end
    frontier = next
    i += 1
  end
end
```

- `frontier`, things you can reach from s using `i-1` moves
- `next`, things you can reach from s using `i` moves
- `parent` forms the shortest paths.

A lot of variables in above code is optional:

- `level` is optional if `i` will suffice.
- `parent` is optional if we don't need the actual shortest path and we can use other tricks to mark vertices as `seen`.
- `next` also can be eliminated by treating frontier as a Queue.

You'll see what I mean shortly, let's start with a classic BFS problem: [Word Ladder](https://leetcode.com/problems/word-ladder/description). Given two words (beginWord and endWord), and a dictionary's word list, find the length of shortest transformation sequence from beginWord to endWord, such that: Only one letter can be changed at a time. Each transformed word must exist in the word list. Note that beginWord is not a transformed word. Return 0 if there is no such transformation sequence. E.g. given begin word "hit", end word "cog" and the dictionary `["hot","dot","dog","lot","log","cog"]`, the result is 5 (`"hit" -> "hot" -> "dot" -> "dog" -> "cog"`).

```ruby
def solve(begin_word, end_word, word_list)
  word_list = word_list.to_set
  return 0 if !word_list.include?(end_word)   # edge case

  q, d = [begin_word], 1                      # BFS start
  while !q.empty?
    q.size.times do
      x = q.shift
      x.size.times do |i|                     # check adj
	('a'..'z').each do |c|
	  w = x[0, i] + c + x[i+1, x.size-i]
	  next if !word_list.include?(w)
	  return d+1 if w == end_word
	  word_list.delete(w)
	  q << w
	end
      end
    end
    d += 1
  end
  0
end
```

- Transform `word_list` to a Set so that lookup time is O(1).
- We can treat Ruby array as a Queue using `shift` and `<<`.
- We used a trick: `q.size.times` and `x = q.shift` so that after each loop `q` will be the new frontier.
- We skipped building the graph. Just checking neighbors inside BFS.
- We delete the word in the `word_list` as marking it as "seen".

In this problem it doesn't require to output the actual path, but we can get it easily using `parent`:

```ruby
path, p = [], end_word
while p != parent[p]
  path.unshift(p)
  p = parent[p]
end
path.unshift(p)
```

There is a [variation of this problem](https://leetcode.com/problems/word-ladder-ii/description) where you need to return all the shortest paths instead of just one. You can't just use `parent` because a word can have multiple parents. A simple solution would be changing `parent` to `{v=>Set}` and use DFS in the end to generate all the paths. I'll leave to to you since it's not that different and you can practice writing DFS.

There is a common optimization called Bi-Directional BFS, simply put, you do BFS on both ends and always search from the one with smaller frontier.

```ruby
def solve(begin_word, end_word, word_list)
  word_list = word_list.to_set
  return 0 if !word_list.include?(end_word)

  qb, qe, d = Set[begin_word], Set[end_word], 0
  while !qb.empty? && !qe.empty?
    temp = Set.new
    qb.each do |x|
      x.size.times do |i|
	('a'..'z').each do |c|
	  w = x[0, i] + c + x[i+1, x.size-i]
	  return d+1 if qe.include?(w)
	  next if !word_list.include?(w)
	  word_list.delete(w)
	  temp << w
	end
      end
    end
    if temp.size < qe.size
      qb = temp
    else
      qb, qe = qe, temp
    end
    d += 1
  end
  0
end
```

If you want the actual path you'll need two parent pointers for both front and backward search, then tape them together.

We talked about BFS, Bi-Directional BFS and solved some shortest path problems. Next We'll look into more complicated concepts like Dijkstra and Bellman-Ford.