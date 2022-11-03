---
title: Depth First Search - Recursive Approach
date: 2022-11-02
description: A recursive twist on the classic DFS.
category: graph-search
tags: ["graph-search", "graph", "dynamic-programming"]
---

<!-- https://git.unl.edu/soft-core/soft-260/graph-search-in-class/-/tree/main/graph-search -->

![Directed graph](./directed-graph.svg)

The main benefit of using a recursive algorithm is that it's very memory efficient.

This can be considered a dynamic programming problem because it breaks the problem down into smaller sub-problems, and can use memoization to remember answers to subproblems.

## Trace

- Worklist: Recursive frames
- Guaranteed to find a path if one exists
- Not guaranteed to find a good path
- Very memory efficient

```
    Worklist    Backpointers
    --------    ------------
    (⊥, a) ✓    a → (⊥, a)
    (a, c) ✓    c → (a, c)
    (c, e)      b → (c, b)
    (c, b) ✓    e → (b, e)
    (b, e) ✓    f → (e, f)
    (b, a) ✓    d → (f, d)
    (e, d)
    (e, f) ✓
    (f, c)
    (f, d) ✓

    Reversed Path
    ----
    d ← f ← e ← b ← c ← a
```

## Algorithm

First, we consider the base case: when `start === destination`, it's fair to say the only vertex in the path is `destination`:

```js
function bfs(graph, start, destination) {
  if (start === destination) return [destination]

  // ...
  return []
}
```

Otherwise, we need to break the problem into a slightly simpler problem: finding the path for the incidences of the start node:

```js
function bfs(graph, start, destination = {}) {
  if (start === destination) return [destination]

  for (const incidence of graph.getIncidences(start)) {
    const postfix = bfs(graph, incidence.destination, destination)
    // ...
  }

  // ...
  return []
}
```

Let's suppose that bfs returns a path for the subproblem. In that case, we want to return that path appended to the current `start` to give the complete path. In another case, if `postfix`
is undefined (meaning no path could be found for that incidence), we want to consider all other incidences. If we consider all incidences and _still_ can't find a path, then it's fair to return
`null`:

```js
function bfs(graph, start, destination) {
  if (start === destination) return [destination]

  for (const incidence of graph.getIncidences(start)) {
    const postfix = bfs(graph, incidence.destination, destination)
    if (postfix !== null) return [start, ...postfix]
  }

  return null
}
```

## Final Algorithm

But we are forgetting a critical thing: what if the graph is cylical? In that case, we need to ensure we don't search vertices we've already visited.

```js
function bfs(graph, start, destination, visited = []) {
  // don't try to find the path if we've already searched this vertex
  if (visited.has(start)) return null
  visited.add(start)

  if (start === destination) return [destination]

  for (const incidence of graph.getIncidences(start)) {
    const postfix = bfs(graph, incidence.destination, destination)
    if (postfix !== null) return [start, ...postfix]
  }

  return null
}
```
