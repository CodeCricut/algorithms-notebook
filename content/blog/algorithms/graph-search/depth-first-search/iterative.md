---
title: Depth First Search - Iterative Approach
date: 2022-11-02
description: A simple graph search algorithm guaranteed to find a path if it exists.
category: graph-search
tags: ["graph-search", "graph"]
---

<!-- https://git.unl.edu/soft-core/soft-260/graph-search-in-class/-/tree/main/graph-search -->

![Directed graph](./directed-graph.svg)

## Trace

- Worklist: Stack
- Guaranteed to find a path if one exists
- Not guaranteed to find a good path
- Can be made very memory efficient (topic for Thursday)

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

It's important that you understand how to write the search algorithm, as it will serve as the basis for many other search algorithms such as dfs, Dijkstra's, etc.

Signature

```js
function dfs(graph, start, destination) {
  return []
}
```

Obviously, we must start with our worklist and backpointers:

```js
function dfs(graph, start, destination) {
  const worklist = []
  const backpointers = {}

  return []
}
```

If we follow the same process as the trace, the next logical step is to iterate through the worklist, adding more items to the stack when the current item has incidences:

```js
function dfs(graph, start, destination) {
  const worklist = []
  const backpointers = {}

  // we must kick of the process with a "dummy" value
  worklist.push(new Edge(null, start))

  while (worklist.length > 0) {
    // we always consider the most-recently added edges
    const workitem = worklist.pop()

    for (const incidence of graph.getIncidences(workitem.destination)) {
      worklist.push(new Edge(workitem.destination, incidence.destination))
    }
  }

  return []
}
```

Now we can begin thinking about our return statement. There are two situations we want to return:

1. when we've searched the entire graph and didn't find a path, return `null`
2. when we've found a path, return the path

In the second case, we need to maintain the path using `backpointers`. Then, we return early and use `backpointers` to resolve the path:

```js
function dfs(graph, start, destination) {
  const worklist = []
  const backpointers = {}

  // we must kick of the process with a "dummy" value
  worklist.push(new Edge(null, start))

  while (worklist.length > 0) {
    // we always consider the most-recently added edges
    const workitem = worklist.pop()

    backpointers.set(workitem.destination, workitem) // e.g., c -> (a, c)

    if (workitem.destination === destination) {
      // we've found the path
      const reversed = []

      let curr = backpointers.get(destination)
      while (curr !== null) {
        reversed.push(curr.end)
        curr = backpointers.get(curr.start)
      }

      return reversed.reverse()
    }

    for (const incidence of graph.getIncidences(workitem.destination)) {
      worklist.push(new Edge(workitem.destination, incidence.destination))
    }
  }

  // we cound't find a path :(
  return null
}
```

This is almost correct, but we need to consider cyclical graphs. If the current work item's destination is already in the backpointers, we can just skip it.

## Final result

The final algorithm looks like this:

```js
function dfs(graph, start, destination) {
  const worklist = []
  const backpointers = {}

  // we must kick of the process with a "dummy" value
  worklist.push(new Edge(null, start))

  while (worklist.length > 0) {
    // we always consider the most-recently added edges
    const workitem = worklist.pop()

    // in case of cyclic graph, skip already visited nodes
    if (backpointers.has(edge.to)) {
      continue
    }

    backpointers.set(workitem.destination, workitem) // e.g., c -> (a, c)

    if (workitem.destination === destination) {
      // we've found the path
      const reversed = []

      let curr = backpointers.get(destination)
      while (curr !== null) {
        reversed.push(curr.end)
        curr = backpointers.get(curr.start)
      }

      return reversed.reverse()
    }

    for (const incidence of graph.getIncidences(workitem.destination)) {
      worklist.push(new Edge(workitem.destination, incidence.destination))
    }
  }

  // we cound't find a path :(
  return null
}
```
