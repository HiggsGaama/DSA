---
layout: default
title: "Topic 10: Graphs I - BFS, DFS & Grid Traversals"
---

# Topic 10: Graphs I - BFS, DFS & Grid Traversals

[← Topic 09: Backtracking](09-backtracking.md) \| [Schedule](schedule.md) \| [Next: Graphs II (Advanced) →](11-graphs-advanced.md)

---

## Daily Timeboxing (1.5 – 2.0 Hours)
- **00:00 – 00:15 (15m) | Active Recall Pre-Test**: Answer the conceptual questions below before reading further.
- **00:15 – 00:40 (25m) | YouTube Conceptual Anchor**: Watch the curated video segment on multi-source BFS and graph connectivity.
- **00:40 – 01:30 (50m) | Coding Deep Dive**: Pen-and-paper dry run of Rotting Oranges & Pacific Atlantic Water Flow.
- **01:30 – 01:45 (15m) | Post-Session Flashcards**: Complete the flashcard prompts from memory.

---

## 1. Active Recall Pre-Test (15 Minutes)
Attempt to answer these questions on paper before expanding the solutions.

<details>
<summary><strong>Prompt 1: Why does Breadth-First Search (BFS) guarantee the shortest path in an unweighted graph, whereas Depth-First Search (DFS) cannot?</strong></summary>

BFS explores nodes strictly in non-decreasing order of their distance from the source: depth 0, depth 1, depth 2, etc. 
Because all edges have uniform weight 1, the first time a node is reached and popped from the queue is guaranteed to be via the minimal number of edges. 
In contrast, DFS explores along one path as deeply as possible before backtracking; it can reach a target node via a long, circuitous path before ever exploring a direct edge.
</details>

<details>
<summary><strong>Prompt 2: What is the Multi-Source BFS pattern, and how does it prevent redundant traversals?</strong></summary>

Instead of running separate BFS passes from each individual source ($K$ sources $\implies O(K \cdot (V + E))$), Multi-Source BFS initializes the BFS queue with **all sources simultaneously at time $t = 0$**, marking all of them as visited. 
The wavefront expands uniformly from all sources in lockstep. Each vertex is visited at most once, achieving the optimal shortest distance to the *closest* source in $O(V + E)$ total time.
</details>

<details>
<summary><strong>Prompt 3: In grid problems with boundary constraints (e.g., Pacific Atlantic Water Flow, Surrounded Regions), why is reverse-traversal from the edges more efficient than testing every interior cell?</strong></summary>

Testing from the interior requires initiating a new search from every cell $(r, c)$ to check if water can reach the boundary ($O((M \times N)^2)$ work).
By reversing the problem—starting DFS/BFS directly from boundary cells and moving inwards to reach higher or uncaptured interior cells—each cell is visited at most once per boundary, reducing complexity to $O(M \times N)$.
</details>

---

## 2. YouTube Conceptual Anchor (25 Minutes)

Recommended Video Anchors:
1. **William Fiset**: [Graph Theory Introduction & BFS/DFS Grid Traversals](https://www.youtube.com/watch?v=09_LlHjoEiY) (22 mins)
2. **NeetCode**: [Rotting Oranges Multi-Source BFS Intuition](https://www.youtube.com/watch?v=y704fEOx0i0) (12 mins)

### Core Invariants & Mental Model
1. **The Visited Invariant**: In any cyclic graph or grid, mark a cell as visited *at the moment it is pushed into the queue*, not when it is popped. Delaying until popping allows duplicate entries of the same node, leading to exponential memory bloat and TLE.
2. **Direction Vectors**: Replace repetitive neighbor checks with standard coordinate delta arrays:
   `DIRECTIONS = [(0, 1), (0, -1), (1, 0), (-1, 0)]`
3. **Bipartite Invariant (2-Colorability)**: A graph is bipartite if and only if it contains **no odd-length cycles**. During BFS/DFS, color the start node with 0. For every neighbor, if uncolored, color with $1 - \text{current}$; if already colored with the same color, an odd cycle exists.

---

## 3. Canonical Code Templates

### Pattern A: Connected Components via Grid DFS ($O(M \times N)$ Time)
```python
from typing import List

def num_islands(grid: List[List[str]]) -> int:
    if not grid:
        return 0
        
    rows, cols = len(grid), len(grid[0])
    island_count = 0
    
    def dfs(r: int, c: int):
        if r < 0 or r >= rows or c < 0 or c >= cols or grid[r][c] != "1":
            return
            
        grid[r][c] = "0" # Sink island cell directly to save visited set space
        dfs(r + 1, c)
        dfs(r - 1, c)
        dfs(r, c + 1)
        dfs(r, c - 1)
        
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == "1":
                island_count += 1
                dfs(r, c)
                
    return island_count
```

### Pattern B: Multi-Source BFS Wavefront ($O(M \times N)$ Time, $O(M \times N)$ Space)
```python
from collections import deque
from typing import List

def oranges_rotting(grid: List[List[int]]) -> int:
    rows, cols = len(grid), len(grid[0])
    queue = deque()
    fresh_count = 0
    
    # 1. Enqueue all initial sources and count fresh targets
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == 2:
                queue.append((r, c))
            elif grid[r][c] == 1:
                fresh_count += 1
                
    if fresh_count == 0:
        return 0
        
    minutes = 0
    directions = [(0, 1), (0, -1), (1, 0), (-1, 0)]
    
    # 2. Wavefront expansion
    while queue and fresh_count > 0:
        minutes += 1
        for _ in range(len(queue)):
            r, c = queue.popleft()
            for dr, dc in directions:
                nr, nc = r + dr, c + dc
                if 0 <= nr < rows and 0 <= nc < cols and grid[nr][nc] == 1:
                    grid[nr][nc] = 2 # Mark visited / rotten
                    fresh_count -= 1
                    queue.append((nr, nc))
                    
    return minutes if fresh_count == 0 else -1
```

---

## 4. Coding Deep Dive: Worked Walkthrough (50 Minutes)

### Problem: Rotting Oranges (LeetCode 994)
**Problem Statement**: You are given an $m \times n$ grid where `0` is empty, `1` is fresh orange, and `2` is rotten orange. Every minute, any fresh orange 4-directionally adjacent to a rotten orange becomes rotten. Return the minimum minutes until no fresh orange remains, or `-1` if impossible.

#### Pen-and-Paper Dry Run Trace
Let grid be:
```text
[ [2, 1, 1],
  [1, 1, 0],
  [0, 1, 1] ]
```
Initial: `fresh_count = 6`. Queue at $t = 0$: `[(0, 0)]`.

| Minute $t$ | Current Queue Tier | Neighbors Processed | Fresh Oranges Rotted | Remaining Fresh | Next Queue |
|:---:|:---:|:---|:---:|:---:|:---:|
| 1 | `[(0, 0)]` | $(0, 1) \to 2$, $(1, 0) \to 2$ | 2 | 4 | `[(0, 1), (1, 0)]` |
| 2 | `[(0, 1), (1, 0)]` | From $(0, 1): (0, 2) \to 2, (1, 1) \to 2$<br>From $(1, 0)$: none | 2 | 2 | `[(0, 2), (1, 1)]` |
| 3 | `[(0, 2), (1, 1)]` | From $(1, 1): (2, 1) \to 2$ | 1 | 1 | `[(2, 1)]` |
| 4 | `[(2, 1)]` | From $(2, 1): (2, 2) \to 2$ | 1 | **0** | `[(2, 2)]` |

All fresh oranges rotted. Elapsed time: **4 minutes**.
Time Complexity: $O(M \times N)$ as each cell is visited at most once.
Space Complexity: $O(M \times N)$ for queue.

---

## 5. Curated Problem Set (16 Problems)

Track your daily solutions by checking off the markdown boxes.

### NeetCode / LeetCode (Core)
- [ ] [LeetCode 200: Number of Islands](https://leetcode.com/problems/number-of-islands/) | Medium | Connected Components DFS Sinking
- [ ] [LeetCode 695: Max Area of Island](https://leetcode.com/problems/max-area-of-island/) | Medium | Area Accumulation Recursive DFS
- [ ] [LeetCode 133: Clone Graph](https://leetcode.com/problems/clone-graph/) | Medium | Node Mapping Hash Table + DFS/BFS
- [ ] [LeetCode 994: Rotting Oranges](https://leetcode.com/problems/rotting-oranges/) | Medium | Multi-Source BFS Wavefront
- [ ] [LeetCode 417: Pacific Atlantic Water Flow](https://leetcode.com/problems/pacific-atlantic-water-flow/) | Medium | Reverse Boundary Search Intersection
- [ ] [LeetCode 130: Surrounded Regions](https://leetcode.com/problems/surrounded-regions/) | Medium | Border-Connected Inverse DFS
- [ ] [LeetCode 1091: Shortest Path in Binary Matrix](https://leetcode.com/problems/shortest-path-in-binary-matrix/) | Medium | 8-Directional BFS Shortest Path
- [ ] [LeetCode 785: Is Graph Bipartite?](https://leetcode.com/problems/is-graph-bipartite/) | Medium | 2-Coloring BFS / Odd Cycle Detection
- [ ] [LeetCode 127: Word Ladder](https://leetcode.com/problems/word-ladder/) | Hard | Bidirectional BFS / Word Transformation
- [ ] [LeetCode 261: Graph Valid Tree](https://leetcode.com/problems/graph-valid-tree/) | Medium | $E = V - 1$ & Cycle-Free Connectivity

### CSES Problem Set (Free Tier, No Login Wall)
- [ ] [CSES 1192: Counting Rooms](https://cses.fi/problemset/task/1192) | Graph Algorithms | 2D Grid Room DFS
- [ ] [CSES 1193: Labyrinth](https://cses.fi/problemset/task/1193) | Graph Algorithms | BFS Shortest Path with String Direction Reconstruction
- [ ] [CSES 1666: Building Roads](https://cses.fi/problemset/task/1666) | Graph Algorithms | Component Leaders & Bridge Linking
- [ ] [CSES 1667: Message Route](https://cses.fi/problemset/task/1667) | Graph Algorithms | Shortest Path Parent Pointers
- [ ] [CSES 1194: Monsters](https://cses.fi/problemset/task/1194) | Graph Algorithms | Multi-Source BFS Monster Race Condition

### Codeforces & AtCoder (Rating 800–1400)
- [ ] [Codeforces 1033A: King Escape](https://codeforces.com/problemset/problem/1033/A) | 1000 | Chessboard Quadrant Graph Search
- [ ] [Codeforces 580C: Kefa and Park](https://codeforces.com/problemset/problem/580/C) | 1500 | Tree DFS with Consecutive Cat Counter

---

## 6. Post-Session Active Recall Flashcards (15 Minutes)

<details>
<summary><strong>Flashcard 1: Why does Bidirectional BFS significantly outperform standard BFS in Word Ladder?</strong></summary>

Standard BFS searches outward in a expanding sphere with branching factor $b$ and distance $d$, visiting roughly $O(b^d)$ states. 
Bidirectional BFS runs two concurrent searches—one from `beginWord` and one from `endWord`—until their frontiers intersect. 
Each search only reaches depth $d/2$, reducing total visited states to:
$$O(b^{d/2} + b^{d/2}) = O(2 \cdot b^{d/2}) \ll O(b^d)$$
For large search spaces, this cuts execution time by several orders of magnitude.
</details>

<details>
<summary><strong>Flashcard 2: In CSES Labyrinth, how do you reconstruct the exact path string ("U", "D", "L", "R") after BFS completes?</strong></summary>

Maintain a 2D array `parent[r][c]` which stores the coordinate `(prev_r, prev_c)` and the move taken to reach `(r, c)`. 
Once target $B$ is reached, backtrack from $B$ through `parent` until reaching starting point $A$. 
Collect the directions in reverse order, then reverse the final string.
</details>

<details>
<summary><strong>Flashcard 3: What condition determines whether an undirected graph is a valid tree?</strong></summary>

An undirected graph with $V$ vertices is a valid tree if and only if:
1. It has exactly $V - 1$ edges: $|E| = |V| - 1$.
2. It is completely connected (single connected component via BFS/DFS).
These two conditions together mathematically preclude cycles and disconnected partitions.
</details>

---

[← Topic 09: Backtracking](09-backtracking.md) \| [Next Topic: Graphs II (Advanced) →](11-graphs-advanced.md)
