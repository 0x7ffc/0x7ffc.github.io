---
layout: post
title: "Graph Traversal Made Easy - Part 2"
description: "Part 2 of a comprehensive guide to Graph Traversal algorithms which includes Depth First Search and solves some leetcode problems with it."
---

Now you have an overview of DFS, I want to generalize DFS a little bit. Firstly, the way the graph is specified is as an adjacency lists:

- The graph contains vertices (V) and edges (E).
- For each vertex v, adj[v] is the list of vertices it's connected to. We can represent it using Hash table.

```ruby
# Visit all the vertices.
def visit_all(vertices, adj)
  parent = {}
  vertices.each do |v|
    if !parent.has_key?(v)
      parent[v] = nil
      dfs(adj, v, parent)
    end
  end
end

# Visit all the vertices reachable from a given source, vertex v.
def dfs(adj, v, parent)
  adj[v].each do |x|
    if !parent.has_key?(x)
      parent[x] = v
      dfs(adj, x, parent)
    end
  end
end
```

- The first step is building the adjacency lists.
- We use `parent` (a hash table) to mark vertices as visited. If we've visited it, we just skip it.

---

### Cycle Detection

The graph has cycle when we visit a vertex (v) which is being processed, by that I mean there are vertices inside `adj[v]` we have not yet visited. We can use another hash table to store this:

```ruby
def dfs_have_cycle?(adj, v, parent, processing=Hash.new(false))
  processing[v] = true		       # new line
  adj[v].each do |x|
    return true if processing[x]       # cycle detected
    if !parent.has_key?(x)
      parent[x] = v
      return true if dfs_have_cycle?(adj, x, parent, processing) # slight change
    end
  end
  processing[v] = false		       # new line
  false				       # new line
end
```

---

### Topological Sort

Topological sorting for Directed Acyclic Graph (DAG) is a linear ordering of vertices such that for every directed edge `u->v`, vertex u comes before v in the ordering. We can't have cycle here because the ordering would be impossible. The reverse order of the finishing times of DFS is the topological sort. It's better for you to understand if we just solve a problem with it, It's a variation of the `Job Scheduling` problem: [alien dictionary](https://leetcode.com/problems/alien-dictionary/description). There is a new alien language which uses the latin alphabet. However, the order among letters are unknown to you. You receive a list of non-empty words from the dictionary, where words are sorted lexicographically by the rules of this new language. Derive the order of letters in this language. E.g. `["wrt", "wrf", "er", "ett", "rftt"] => "wertf"`.

Let's build the graph (adjacency lists) first.

For each two words (w1 and w2) (w1 comes first) in the dictionary we can determine their order:

- a pointer points to both w1 an w2 with the same offset `p`.
- if w1[p] equals w2[p], then go to the next character: `p++`.
- otherwise w1[p] comes before w2[p]: add w2[p] to adj[w1[p]]. Then we break. We only consider the first different character.

```ruby
def build_graph(words)
  g = Hash.new { |h, k| h[k] = Set.new }
  words.combination(2).each do |w1, w2|
    p = 0
    while p < w1.size && p < w2.size
      if w1[p] == w2[p]
	p += 1
      else
	g[w1[p]] << w2[p]
	break
      end
    end
  end
  g
end
```

After building the graph, we can just use our DFS "template" to solve it:

```ruby
def solve(words)
  g, parent, topo = build_graph(words), {}, []
  g.keys.each do |v|
    if !parent.has_key?(v)
      parent[v] = nil
      return "" if dfs(g, v, parent, topo)
    end
  end
  topo.join
end

def dfs(g, v, parent, topo, processing=Hash.new(false))
  processing[v] = true
  g[v].each do |x|
    return true if processing[x]
    if !parent.has_key?(x)
      parent[x] = v
      return true if dfs(g, x, parent, topo, processing)
    end
  end
  processing[v] = false
  topo.unshift(v)
  false
end
```

Notice that we can actually get rid of `processing` and instead use `topo` to detect cycle: `topo.include?(x)`. `include?` is an `O(n)` operation which can be optimized by switching `topo` to a hash table.

It may worries you that in this problem it takes some effort to build the graph, but sometimes it's quite simple. For example in [this problem](https://leetcode.com/problems/course-schedule-ii/description), I can build the graph in one line of ruby and the rest is exactly the same:

```ruby
g = prerequisites.reduce(Array.new(num_courses) {[]}) { |h, (u, v)| h[v] << u; h}
```

That pretty sums up DFS, we talked about cycle detection and topological sort. Maybe I'll talk about DFS on trees later.

We'll look at BFS and solve some shortest path problems next.