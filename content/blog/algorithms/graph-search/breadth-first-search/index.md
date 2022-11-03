---
title: Breadth First Search
date: 2022-11-03
description: A variation of DFS guaranteed to find the path with the fewest edges
category: graph-search
tags: ["graph-search", "graph"]
---

## Trace

- Worklist: Queue
- Guaranteed to find a path with the fewest edges

```
  Worklist Backpointers

  ***

  (⊥, a) ✓ a → (⊥, a)
  (a, c) ✓ c → (a, c)
  (c, e) ✓ e → (c, e)
  (c, b) ✓ b → (c, b)
  (e, d) ✓ d → (e, d)
  (e, f)
  (b, e)
  (b, a)

  ## Reversed Path

  d ← e ← c ← a
```

## DFS basis

This is a very simple variation on DFS that uses a queue instead of a stack. Here is DFS:

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

## Final Algorithm

By just making a tweak to the worklist data structure, we can implement BFS!

```js
function bfs(graph, start, destination) {
  const worklist = new Queue()
  const backpointers = {}

  // we must kick of the process with a "dummy" value
  worklist.insert(new Edge(null, start))

  while (worklist.length > 0) {
    // we always consider the most-recently added edges
    const workitem = worklist.remove()

    // in case of cyclic graph, skip already visited nodes
    if (backpointers.has(workitem.to)) {
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
      worklist.insert(new Edge(workitem.destination, incidence.destination))
    }
  }

  // we cound't find a path :(
  return null
}
```
