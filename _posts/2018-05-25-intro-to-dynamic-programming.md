---
layout: post
title: "Intro to Dynamic Programming"
description: "Dynamic Programming although easy to describe: it's just recursion + memoization, but it's hard to fully grasp. Here I'll explore these ideas step by step providing a comprehensive guide and solve some leetcode problems with it."
---

I was really confused when I was first introduced to the concept of dynamic programing, partly because I wasn't good at understanding recursion and mostly because people who taught me did a bad job. They didn't tell me about recursion and memoization, all they told me was "states" and "state transformation formulas", so at that time DP felt like a gimmick to me. Now I've clear this all up and I want to share it with you.

Firstly, let's try to solve this problem: [leetcode 10](https://leetcode.com/problems/regular-expression-matching/description). Given an input string `s` and a pattern `p`, implement regular expression matching with support for '.' and '\*'. '.' Matches any single character. '\*' Matches zero or more of the preceding element, for example "aab" matches "c\*a\*b". Quite hard at first glance, but if we break it down to smaller problems we can solve it recursively.

Given `s`, and we want to match it against pattern `p`, we'll describe it as `m(s, p)`. Now if the second character of p is not '\*' which means that the only way to get around it is that if the first character of s and p matches or the first character of p is '.', once that is done we can match the tails of s an p. However if the second character of p is '\*' then we need to break it down a little bit:

- Given patter "c\*a\*b", it's the same as `m(s, "a*b")` if we abandon "c\*" and the rest still matches: `m(s, p[2..])`.
- Given patter "c\*a\*b", it's the same as `m(s[1..], "c*a*b")` if the first character of s is 'c'.
- Given pattern ".\*a\*b" then we can always check whether the rest of s matches: `m(s[1..], ".*a*b")` if s is not empty.

We reach the end if p is empty and s is also empty (no more string to match).

Believe it or not that sums up all the conditions. Let's translate that into code:

```ruby
def m(s, p)
  # the end condition
  return s.empty? if p.empty?
  # otherwise we try to match
  first = !s.empty? && (s[0] == p[0] || p[0] == '.') 
  if p[1] == '*'
    m(s, p[2..]) || (first && m(s[1..], p))
  else
    first && m(s[1..], p[1..]) 
  end
end
```

You should be able to understand above code, then the rest of the article will make more sense. Ruby's slicing string is actually an O(n) operation, so instead we should use indices to mark the starting point of the substring and subpattern we're trying to match. i points to s, j points to p, when j reaches the end, so should i.

```ruby
def m(s, p, i=0, j=0)
  return i == s.length if j == p.length
  first = i < s.length && (s[i] == p[j] || p[j] == '.')
  if p[j+1] == '*'
    m(s, p, i, j+2) || (first && m(s, p, i+1, j)
  else
    first && m(s, p, i+1, j+1)
  end
end
```

As you can see, the result of `m` can be memorized using i and j, but we'll actually never hit the memo because i and j are always moving forward. Anyway, it still shows the motivation behind DP.

```ruby
def m(s, p, i=0, j=0, memo=Hash.new)
  return i == s.length if j == p.length
  return memo[[i,j]] if memo.has_key?([i,j])
  first = i < s.length && (s[i] == p[j] || p[j] == '.')
  if p[j+1] == '*'
    m(s, p, i, j+2, memo) || (first && m(s, p, i+1, j, memo))
  else
    first && m(s, p, i+1, j+1, memo)
  end
end
```

Now instead of recursion, let's use a table to represent the problem.

- dp[i][j] represents whether s and p matches starting from i and j respectively.
- the initial state is dp[s.length][p.length] and it is true, empty string matches empty pattern.
- we'll fill the table from dp[s.length][p.length] up to dp[0][0].
- the result of m(s, p) is stored inside dp[0][0].

```ruby
def m(s, p)
  m, n = s.length, p.length
  dp = Array.new(m+1) { Array.new(n+1, false) }
  dp[m][n] = true
  m.downto(0) do |i|
    (n-1).downto(0) do |j|
      first = i < m && (s[i] == p[j] || p[j] == '.')
      if p[j+1] == '*'
        dp[i][j] = dp[i][j+2] || (first && dp[i+1][j])
      else
        dp[i][j] = first && dp[i+1][j+1]
      end
    end
  end
  dp[0][0]
end
```

As you can see recursion is top-down and DP is bottom-up, and DP is really just a iterative version of the same recursive program with the benefit of memoization.

Key points:

- Identify the sub problems.
- Solve your problem recursively.
- Check what part of the problem can be memorized.
- Check the end condition.
- Check edge cases.
- Rewrite the recursive problem with bottom-up DP.

---

### House Robber

Since we've "mastered" DP, let's practice with this really easy problem: [House Robber](https://leetcode.com/problems/house-robber/description): You are a professional robber planning to rob houses along a street. Each house has a certain amount of money stashed, the only constraint stopping you from robbing each of them is that adjacent houses have security system connected and it will automatically contact the police if two adjacent houses were broken into on the same night. Given a list of non-negative integers representing the amount of money of each house, determine the maximum amount of money you can rob tonight without alerting the police. Without actually writing the recursive version, one can find these key points easily:

1. dp[i]: the maximum amount of money we can rob from house 0 to i.
2. If we rob the ith house, then dp[i] = dp[i-2]+nums[i].
3. If we don't rob the ith house, then dp[i] = dp[i-1].
4. Pick the maximum of step 2 and step 3 as dp[i].
5. Only one house, just rob it: dp[0] = nums[0].
6. No house, can't rob anyone: dp[-1] = 0.
7. The result is stored in dp[n-1].


```ruby
def robe(nums)
  return 0 if nums.empty?
  n = nums.size
  dp = Array.new(n+1, 0)
  dp[0] = nums[0]
  (1...n).each do |i|
    dp[i] = [dp[i-2]+nums[i], dp[i-1]].max
  end
  dp[n-1]
end
```

This is what I call a classic DP problem, actually it's a simplified variation of the `0/1 Knapsack Problem`.

---

### Coin Change

Let's see another classic, the infamous "Coin Change" problem. There're two variation of this problem, the first one: You are given coins of different denominations and a total amount of money amount. Write a function to compute the fewest number of coins that you need to make up that amount. If that amount of money cannot be made up by any combination of the coins, return -1. For example given coins [1, 2, 5] and the total amount 11, the result is 3 because 11 = 5 + 5 + 1. And you may assume that you have an infinite number of each kind of coin. Let's break it down:

1. dp[i]: the fewest number of coins that makes up amount i.
2. For amount i, we can choose which kind of coin we use, for certain coin x, we get dp[i] = dp[i-x] + 1, and we do this for all kinds of coins: `dp[i] = min(dp[i-{for x in coins if x <= i}] + 1)`.
3. dp table is initialized with INFINITY which makes comparision easier.
4. dp[0] is 0, because we need 0 coins to making up amount 0.

```ruby
def change(coins, amount)
  dp = Array.new(amount+1, Float::INFINITY)
  (0..amount).each do |i|
    coins.each do |x|
      dp[i] = [dp[i], dp[i-x]+1].min if x <= i 
    end
  end
  dp[amount] == Float::INFINITY ? -1 : dp[amount]
end
```

The second variation of the coin change problem: given coins of different denominations and a total amount of money. Write a function to compute the number of combinations that make up that amount. You may assume that you have infinite number of each kind of coin. Unlike the first one, we have to decide whether to use certain coin or not, so a one dimensional DP table won't suffice. To simplify this problem for my brain, I see this problem as Integer partition. Integer partition is number of ways a number can be represented as sum of positive integers. e.g. 4 => 4 / 3+1 / 2+2 / 2+1+1 / 1+1+1+1, so there are 4 ways, also 3+1 and 1+3 are considered as only one. you can imagine a big table of size N*N where N is size of all natural number, e.g. table[4][4] = 4, so it contains all the solutions for any combination of coins and given amount. Now instead of all natural number, you're given only some certain integers, aka coins.

For coins: [2,3,5], amount: 7, we have this table

|i\j| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
|---|---|---|---|---|---|---|---|---|
| 0 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 2 | 1 | 0 | 1 | 0 | 1 | 0 | 1 | 0 |
| 3 | 1 | 0 | 1 | 1 | 1 | 1 | 2 | 1 |
| 5 | 1 | 0 | 1 | 1 | 1 | 2 | 2 | 2 |

1. dp[i][j]: number of ways that make up amount j using first i coins.
2. the first row should all be zero except for the dp[0][0].
3. dp[0][0] is 1. It's weird I know, but when you think about it, dp[x][x] should never be zero.
4. To determine dp[i][j] we have two choices:
   - Use current coin: dp[i][j-x] where x is the current coin denomination.
   - Don't use current coin: dp[i-1][j], just ignore current coin.
   - dp[i][j] is the sum of these two choices.

```ruby
def change(coins, amount)
  dp = Array.new(coins.size+1) { Array.new(amount+1, 0) }
  dp[0][0] = 1
  (1..coins.size).each do |i|
    (0..amount).each do |j|
      dp[i][j] = dp[i-1][j] + (j >= coins[i-1] ? dp[i][j-coins[i-1]] : 0)
    end
  end
  dp[coins.size][amount]
end
```

Although a bit more complex, it still follows the same principle.

### Conclusion

Divide and Conquer also works by dividing the problem into subproblems, but don't confuse it with DP. DP actually emphasize on using memoization to solve overlapping subproblems efficiently. Remember:

> DP = recursion + memoization
