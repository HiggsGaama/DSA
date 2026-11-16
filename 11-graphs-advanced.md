---
layout: default
title: "Topic 11: Graphs II - Topo Sort, Dijkstra & DSU"
---

# Topic 11: Graphs II - Topo Sort, Dijkstra & DSU

[← Topic 10: Graphs I](10-graphs-traversal.md) \| [Schedule](schedule.md) \| [Next: 1-D Dynamic Programming →](12-1d-dp.md)

---

## Daily Timeboxing (1.5 – 2.0 Hours)
- **00:00 – 00:15 (15m) | Active Recall Pre-Test**: Answer the conceptual questions below before reading further.
- **00:15 – 00:40 (25m) | YouTube Conceptual Anchor**: Watch the curated video segment on Kahn's algorithm, Dijkstra relaxation, and DSU.
- **00:40 – 01:30 (50m) | Coding Deep Dive**: Pen-and-paper dry run of Course Schedule II & Network Delay Time.
- **01:30 – 01:45 (15m) | Post-Session Flashcards**: Complete the flashcard prompts from memory.

---

## 1. Active Recall Pre-Test (15 Minutes)
Attempt to answer these questions on paper before expanding the solutions.

<details>
<summary><strong>Prompt 1: Why does Dijkstra's Algorithm fail on graphs containing negative edge weights?</strong></summary>

Dijkstra relies on a **greedy invariant**: once a vertex $u$ is popped from the min-heap with the minimum tentative distance $d[u]$, its shortest path is finalized and will never be relaxed again. 
This invariant holds strictly when all edge weights are non-negative ($w(u, v) \ge 0$), because any alternative path to $u$ through an unvisited vertex $x$ already has length $d[x] \ge d[u]$, and adding non-negative weights cannot decrease it. 
With negative weights, an alternative longer-hop path can yield a smaller total distance after $u$ was already finalized, violating the greedy assumption. (Negative weights require Bellman-Ford or SPFA).
</details>

<details>
<summary><strong>Prompt 2: How does Kahn's algorithm detect directed cycles during Topological Sort?</strong></summary>

Kahn's algorithm maintains an `in_degree` array and enqueues all vertices with `in_degree == 0`. 
When a node is popped, it is appended to the topological order, and the in-degrees of all its neighbors are decremented. 
Any nodes participating in a directed cycle will always have an in-degree $\ge 1$ from their cycle predecessor, so they can never reach an in-degree of 0. 
If the total number of nodes in the output topological order is strictly less than $|V|$, a cycle exists.
</details>

<details>
<summary><strong>Prompt 3: What is the time complexity of Disjoint Set Union (DSU) when both Path Compression and Union by Rank are used?</strong></summary>

$O(\alpha(N))$ amortized per operation, where $\alpha(N)$ is the **Inverse Ackermann Function**. 
For all practically conceivable values of $N$ in our universe ($N \le 10^{80}$ atoms in the observable universe), $\alpha(N) < 5$. 
Therefore, DSU operations run in effectively constant $O(1)$ time in practice.
</details>

---

## 2. YouTube Conceptual Anchor (25 Minutes)

Recommended Video Anchors:
1. **Abdul Bari**: [Dijkstra's Algorithm Shortest Path](https://www.youtube.com/watch?v=XB4MIexjvY0) (20 mins)
2. **William Fiset**: [Topological Sort (Kahn's Algorithm) & Union Find / DSU](https://www.youtube.com/watch?v=eL-KzMXSXXI) (22 mins)

### Core Invariants & Mental Model
1. **Kahn's In-Degree Invariant**: A task can be executed if and only if all of its prerequisite dependencies have been satisfied (`in_degree == 0`).
2. **Dijkstra Relaxation Invariant**: For every directed edge $(u, v)$ with weight $w$:
   $$\text{if } \text{dist}[u] + w < \text{dist}[v] \implies \text{dist}[v] = \text{dist}[u] + w$$
   Always store `(dist, node)` in the min-heap. If `d > dist[u]`, skip processing (stale heap entry).
3. **DSU Representative Root Invariant**: Two elements $x$ and $y$ belong to the same connected component if and only if `find(x) == find(y)`. Path compression flattens the tree so that `parent[x]` points directly to the component root.

---

## 3. Canonical Code Templates

### Pattern A: Kahn's Algorithm for Topological Sort ($O(V + E)$ Time)
```python
from collections import deque, defaultdict
from typing import List

def topological_sort(num_courses: int, prerequisites: List[List[int]]) -> List[int]:
    adj = defaultdict(list)
    in_degree = [0] * num_courses
    
    for dest, src in prerequisites:
        adj[src].append(dest)
        in_degree[dest] += 1
        
    queue = deque([i for i in range(num_courses) if in_degree[i] == 0])
    topo_order = []
    
    while queue:
        node = queue.popleft()
        topo_order.append(node)
        
        for neighbor in adj[node]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)
                
    return topo_order if len(topo_order) == num_courses else []
```

### Pattern B: Dijkstra's Shortest Path Algorithm ($O((V + E) \log V)$ Time)
```python
import heapq
from collections import defaultdict
from typing import List

def network_delay_time(times: List[List[int]], n: int, k: int) -> int:
    adj = defaultdict(list)
    for u, v, w in times:
        adj[u].append((v, w))
        
    # (distance_so_far, current_node)
    min_heap = [(0, k)]
    shortest_dist = {}
    
    while min_heap:
        d, u = heapq.heappop(min_heap)
        
        if u in shortest_dist:
            continue
        shortest_dist[u] = d
        
        for v, weight in adj[u]:
            if v not in shortest_dist:
                heapq.heappush(min_heap, (d + weight, v))
                
    return max(shortest_dist.values()) if len(shortest_dist) == n else -1
```

### Pattern C: Disjoint Set Union (DSU with Rank & Path Compression)
```python
class DSU:
    def __init__(self, size: int):
        self.parent = list(range(size))
        self.rank = [0] * size
        self.components = size

    def find(self, x: int) -> int:
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x]) # Path compression
        return self.parent[x]

    def union(self, x: int, y: int) -> bool:
        root_x = self.find(x)
        root_y = self.find(y)
        
        if root_x == root_y:
            return False # Already in same component (cycle detected)
            
        # Union by rank
        if self.rank[root_x] < self.rank[root_y]:
            self.parent[root_x] = root_y
        elif self.rank[root_x] > self.rank[root_y]:
            self.parent[root_y] = root_x
        else:
            self.parent[root_y] = root_x
            self.rank[root_x] += 1
            
        self.components -= 1
        return True
```

---

## 4. Coding Deep Dive: Worked Walkthrough (50 Minutes)

### Problem: Course Schedule II (LeetCode 210)
**Problem Statement**: There are `numCourses` courses labeled `0` to `n-1`. Given `prerequisites[i] = [a, b]` (must take `b` before `a`), return any valid course ordering, or `[]` if impossible.

#### Pen-and-Paper Dry Run Trace
Let `numCourses = 4`, `prerequisites = [[1, 0], [2, 0], [3, 1], [3, 2]]`.
Edges: $0 \to 1$, $0 \to 2$, $1 \to 3$, $2 \to 3$.
Initial In-Degrees:
- Node 0: 0
- Node 1: 1
- Node 2: 1
- Node 3: 2
Queue at start: `[0]`.

| Step | Popped Node | Neighbors Updated | New In-Degrees | New Nodes Enqueued | Current Output Order |
|:---:|:---:|:---|:---|:---:|:---|
| 1 | `0` | $1: 1 - 1 = 0$<br>$2: 1 - 1 = 0$ | Node 1: 0, Node 2: 0 | `[1, 2]` | `[0]` |
| 2 | `1` | $3: 2 - 1 = 1$ | Node 3: 1 | None | `[0, 1]` |
| 3 | `2` | $3: 1 - 1 = 0$ | Node 3: 0 | `[3]` | `[0, 1, 2]` |
| 4 | `3` | None | - | None | `[0, 1, 2, 3]` |

Queue is empty. Total courses ordered: $4 == \text{numCourses}$.
Valid Topological Sort: `[0, 1, 2, 3]` (or `[0, 2, 1, 3]`).
Time Complexity: $O(V + E) = O(4 + 4) = O(8)$.
Space Complexity: $O(V + E)$ for adjacency list and in-degree array.

---

## 5. Curated Problem Set (16 Problems)

Track your daily solutions by checking off the markdown boxes.

### NeetCode / LeetCode (Core)
- [ ] [LeetCode 207: Course Schedule](https://leetcode.com/problems/course-schedule/) | Medium | Kahn's Cycle Detection
- [ ] [LeetCode 210: Course Schedule II](https://leetcode.com/problems/course-schedule-ii/) | Medium | Topological Ordering Construction
- [ ] [LeetCode 269: Alien Dictionary](https://leetcode.com/problems/alien-dictionary/) | Hard | Character Ordering Extraction & Topo Sort
- [ ] [LeetCode 802: Find Eventual Safe States](https://leetcode.com/problems/find-eventual-safe-states/) | Medium | Reverse Graph Kahn's Algorithm
- [ ] [LeetCode 743: Network Delay Time](https://leetcode.com/problems/network-delay-time/) | Medium | Classic Dijkstra Min-Heap
- [ ] [LeetCode 1631: Path With Minimum Effort](https://leetcode.com/problems/path-with-minimum-effort/) | Medium | Modified Dijkstra / Minimax Effort
- [ ] [LeetCode 787: Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/) | Medium | Bellman-Ford / Bounded BFS
- [ ] [LeetCode 684: Redundant Connection](https://leetcode.com/problems/redundant-connection/) | Medium | DSU Cycle Detection in Tree
- [ ] [LeetCode 323: Number of Connected Components in an Undirected Graph](https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/) | Medium | DSU Component Count Tracking
- [ ] [LeetCode 1584: Min Cost to Connect All Points](https://leetcode.com/problems/min-cost-to-connect-all-points/) | Medium | Kruskal's Algorithm with DSU / Prim's

### CSES Problem Set (Free Tier, No Login Wall)
- [ ] [CSES 1679: Course Schedule](https://cses.fi/problemset/task/1679) | Graph Algorithms | Classic Topological Sort Verification
- [ ] [CSES 1669: Round Trip](https://cses.fi/problemset/task/1669) | Graph Algorithms | Cycle Detection and Reconstruction
- [ ] [CSES 1671: Shortest Routes I](https://cses.fi/problemset/task/1671) | Graph Algorithms | Non-Negative Single-Source Dijkstra
- [ ] [CSES 1672: Shortest Routes II](https://cses.fi/problemset/task/1672) | Graph Algorithms | Floyd-Warshall All-Pairs $O(V^3)$
- [ ] [CSES 1675: Road Reparation](https://cses.fi/problemset/task/1675) | Graph Algorithms | Minimum Spanning Tree (Kruskal's)

### Codeforces & AtCoder (Rating 800–1400)
- [ ] [Codeforces 20C: Dijkstra?](https://codeforces.com/problemset/problem/20/C) | 1400 | Shortest Path with Full Node Array Reconstruction
- [ ] [Codeforces 25D: Road Construction](https://codeforces.com/problemset/problem/25/D) | 1600 | DSU Redundant Edge Rerouting
- [ ] [AtCoder ABC218_E: Destruction](https://atcoder.jp/contests/abc218/tasks/abc218_e) | 1200 | Maximum Weight Edge Pruning via DSU

---

## 6. Post-Session Active Recall Flashcards (15 Minutes)

<details>
<summary><strong>Flashcard 1: In Dijkstra's Algorithm, why is it critical to check `if d > shortest_dist[u]: continue` after popping from the min-heap?</strong></summary>

Python's `heapq` does not support an efficient `decrease_key` operation. When a shorter distance to node $u$ is discovered, a new `(new_dist, u)` pair is pushed onto the heap, leaving the older, larger `(old_dist, u)` in the heap. 
Skipping stale entries when popped guarantees that each vertex's adjacency list is relaxed at most once, maintaining $O((V + E) \log V)$ runtime instead of degrading exponentially.
</details>

<details>
<summary><strong>Flashcard 2: How does Kruskal's algorithm use DSU to construct a Minimum Spanning Tree (MST)?</strong></summary>

1. Sort all edges $E$ in ascending order of their weights.
2. Initialize a DSU with all $V$ vertices.
3. For each edge $(u, v, w)$ in sorted order:
   - Call `dsu.union(u, v)`.
   - If `union` returns `True`, $u$ and $v$ were previously disconnected; include this edge in the MST and accumulate its weight.
   - If `union` returns `False`, adding this edge would form a cycle; discard it.
4. Stop when $V - 1$ edges have been accepted.
</details>

<details>
<summary><strong>Flashcard 3: What is the difference between Path Compression and Union by Rank?</strong></summary>

- **Path Compression**: Executed during `find(x)`. It points every node along the lookup traversal directly to the root, dramatically flattening tree depth for subsequent queries.
- **Union by Rank / Size**: Executed during `union(x, y)`. It attaches the tree with smaller depth (or smaller size) under the root of the larger tree, preventing tree depth from degenerating into a tall line.
Together, they achieve $O(\alpha(N))$ amortized complexity.
</details>

---

[← Topic 10: Graphs I](10-graphs-traversal.md) \| [Next Topic: 1-D Dynamic Programming →](12-1d-dp.md)
