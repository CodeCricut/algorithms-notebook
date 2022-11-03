---
title: Dijkstra's Algorithm (and Best First)
date: 2022-11-02
description: A variation on BFS guaranteed to find the path with the least edge weight.
category: graph-search
tags: ["graph-search", "graph"]
---

## Trace

```
*   Worklist: Priority Queue
*   Priority: Distance travelled from the source
*   Guaranteed to find a path with least weight

      Worklist       Backpointers
    ------------    --------------
    (⊥, a, 0) ✓     a → (⊥, a, 0)
    (a, c, 7) ✓     c → (a, c, 7)
    (c, e, 12) ✓    b → (c, b, 8)
    (c, b, 8) ✓     e → (c, e, 12)
    (b, e, 17)      f → (e, f, 14)
    (b, a, 12) ✓    d → (f, d, 14)
    (e, d, 15)
    (e, f, 14) ✓
    (f, c, 22)
    (f, d, 14) ✓

    Reversed Path
    ----
    d ← f ← e ← c ← a

```

## Explanation

Dijkstra's algorithm is a simple variation on BFS that uses a _PriorityQueue_ instead of a regular _Queue_. Specifically, we prioritize edges with the least total edge weight.

Here's BFS for a refresher:

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

## Final Algorithm

We'll make a few tweaks to make this code use a priority queue where the priority is the total edge weight.

```js
function dijkstras(graph, start, destination) {
  const worklist = new PriorityQueue()
  const backpointers = {}

  // Edge(start, end, totalDistance)
  worklist.insert(new Edge(null, start, 0), 0)

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
      const newTotalWeight = workitem.totalDistance + incidence.weight

      // We must keep track of the total distance traveled for each worklist item, AND use it as the priority so lower-weight vertices are considered first
      worklist.insert(
        new Edge(workitem.destination, incidence.destination, newTotalWeight),
        newTotalWeight
      )
    }
  }

  // we cound't find a path :(
  return null
}
```

## Best First

Best first is a very simple variation on Dijkstra's where a heuristic function is also used to predict the viability of a path.

```js
// Here we use an arbitrary heuristic. Actual heuristics might include Manhattan distance to the destination.
function heuristic(vertex, destination) {
  if ("acf".includes(vertex)) {
    return 2
  }
  if ("be".includes(vertex)) {
    return 1
  }
  return 0
}

export function bestFirst(graph, source, destination) {
  const backpointers = new Map()
  const worklist = new PriorityQueue()
  worklist.insert(
    new Edge(undefined, source, 0),
    heuristic(source, destination)
  )
  while (worklist.size > 0) {
    const workitem = worklist.remove()
    if (
      backpointers.has(workitem.to) &&
      backpointers.get(workitem.to).distance <= workitem.distance
    ) {
      continue
    }
    backpointers.set(workitem.to, workitem)
    if (workitem.to === destination) {
      const reversedPath = []
      for (
        let current = destination;
        current !== undefined;
        current = backpointers.get(current).from
      ) {
        reversedPath.push(current)
      }
      return reversedPath.reverse()
    }
    for (const incidence of graph.getIncidences(workitem.to)) {
      worklist.insert(
        new Edge(
          workitem.to,
          incidence.destination,
          workitem.distance + incidence.weight
        ),
        workitem.distance +
          incidence.weight +
          heuristic(incidence.destination, destination)
      )
    }
  }
  return undefined
}
```
