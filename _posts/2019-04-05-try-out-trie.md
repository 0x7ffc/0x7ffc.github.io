---
layout: post
title: "Try Out Trie"
description: "Trie is an efficient data structure to check whether a word or a word prefix is in a dictionary. We'll implement a simple Trie and solve some leetcode problems with it."
mathjax: true
---

Let $s$ be some string and $W$ be a sequence of strings.

Trie is a data structure which:

* answer whether $s$ is in $W$.
* answer whether $s$ is a prefix of some string in $W$.
* requires $O(m)$ time when m exists in $W$ where $m$ is the length of $s$.
* requires less than $O(m)$ when $s$ is not in $W$.

The difference with Hash table is that it can answer prefix-related queries efficiently, otherwise you'll have to store all the prefix strings in the Hash table.

The implementation of Trie is simple, it's a tree like data structure where each node consists of a mapping from character to child nodes, usually represented by an Array size of 256 when $W$ only consists of ASCII character. You can store additional information in the tree nodes, such as a boolean to indicate the end of a word. Let's start with the Class definitions.

```ruby
class Trie
  def initialize
    @root = Node.new
  end

  class Node
    attr_accessor :val, :next
    def initialize
      @val, @next = nil, Array.new(256)
    end
  end

  def insert(s) end   # TODO
  def exitst?(s) end  # TODO
  def prefix?(s) end  # TODO
end
```

It always have a dummy node `@root` as an entry point.

Next is `insert`, that is adding a new word $s$ to the Trie:

```ruby
def insert(s, x=@root, k=0)
  x = Node.new if x.nil?
  if k == s.size		# reach the end of s
    x.val = true
    return x
  end
  c = s[k].ord - 'a'.ord	# get the index
  x.next[c] = insert(s, x.next[c], k+1)
  x
end
```

This is a recursive version, if you've written BST before, this type of code should be familiar to you. We pass the link down the tree and build the tree bottom-up. Given $W$ you can build the Trie by inserting each word in it.

Next are queries, the only difference is that we check `val` in `exist?` which indicate the end of a word.
:

```ruby
def exist?(s, x=@root, k=0)
  return false if x.nil?
  return !x.val.nil? if k == s.size
  c = s[k].ord - 'a'.ord
  exists?(s, x.next[c], k+1)
end

def prefix?(s, x=@root, k=0)
  return false if x.nil?
  return true if k == s.size
  c = s[k].ord - 'a'.ord
  prefix?(s, x.next[c], k+1)
end
```

This API for Trie is nice and all, but most of the time you should expose the Node and work on it directly. Let's look at a problem called ["Word Search"](https://leetcode.com/problems/word-search-ii/description): Given a 2D board and a list of words from the dictionary, find all words in the board. Each word must be constructed from letters of sequentially adjacent cell, where "adjacent" cells are those horizontally or vertically neighboring. The same letter cell may not be used more than once in a word.

Example:

```ruby
board = [
  ['o','a','a','n'],
  ['e','t','a','e'],
  ['i','h','k','r'],
  ['i','f','l','v']]
words = ["oath","pea","eat","rain"]
Output: ["eat","oath"]
```

If you're not familiar with DFS, check my [last post](/2019/graph-traversal-made-easy-part-1). If the current candidate does not exist in all words' prefix, you could stop backtracking immediately which makes Trie a great option:

```ruby
# b: board; i: row; j: column;
# t: Trie; ans: the result.
def dfs(b, t, i, j, ans)
  return if i < 0 || i > b.size-1 ||
    j < 0 || j > b[0].size-1 || b[i][j] == "*"
  t = t.next[b[i][j]]		# go to next Trie node
  return if t.nil?		# no such prefix
  if !t.word.nil?		# word detected
    ans << t.word
    t.word = nil
  end
  b[i][j], y = "*", b[i][j]	# sink the island
  dfs(b, t, i-1, j, ans);	# up
  dfs(b, t, i+1, j, ans);	# down
  dfs(b, t, i, j-1, ans);	# left
  dfs(b, t, i, j+1, ans);	# right
  b[i][j] = y			# restore the island
end
```

How do we build the Trie in the first place? Well, the same as before, this time I'll write a iterative version:

```ruby
class Trie
  attr_accessor :word, :next
  def initialize
    @word, @next = nil, {}	# use Hash to ease things a little bit
  end
end

def build_trie(words)
  root = Trie.new
  words.each do |w|
    t = root
    w.each_char do |c|
      t.next[c] = Trie.new if t.next[c].nil?
      t = t.next[c]
    end
    t.word = w
  end
  root
end
```

In the end we glue all these together:

```ruby
def find_words(board, words)
  trie, ans = build_trie(words), []
  board.each_with_index do |r, i|
    r.each_with_index do |c, j|
      dfs(board, trie, i, j, ans)
    end
  end
  ans
end
```

When you boil it down, it really is just DFS + Trie, pretty straightforward.

For a far more complex problem see [Palindrome Paris](https://leetcode.com/problems/palindrome-pairs/description). I'll give you a hint: build the Trie with each word reversed.

In conclusion, Trie is an efficient data structure to check whether a word or a word prefix is in a dictionary. Next time you jump into Hash table, think twice and give Trie a try 😉.