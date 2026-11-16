---
layout: default
title: "Graphs II: Topological Sort, Dijkstra & Union-Find"
---

# Graphs II: Topological Sort, Dijkstra & Union-Find

> **Estimated Time**: 12 Days • **Pace**: 1.5–2 hours/day (Strictly 1 problem/day) • **Language**: C# (.NET 6+)

---

## 🛠️ C# Mental Model & Gotchas

### 1. Disjoint Set Union (DSU) with Path Compression & Rank
The golden standard DSU in C#:
```csharp
public class UnionFind {
  private readonly int[] _parent;
  private readonly int[] _rank;
  public int Components { get; private set; }

  public UnionFind(int n) {
      _parent = new int[n];
      _rank = new int[n];
      Components = n;
      for (int i = 0; i < n; i++) _parent[i] = i;
  }

  public int Find(int i) {
      if (_parent[i] == i) return i;
      return _parent[i] = Find(_parent[i]); // Path compression
  }

  public bool Union(int i, int j) {
      int rootI = Find(i);
      int rootJ = Find(j);
      if (rootI == rootJ) return false; // Already connected

      // Union by rank
      if (_rank[rootI] < _rank[rootJ]) {
          _parent[rootI] = rootJ;
      } else if (_rank[rootI] > _rank[rootJ]) {
          _parent[rootJ] = rootI;
      } else {
          _parent[rootJ] = rootI;
          _rank[rootI]++;
      }
      Components--;
      return true;
  }
}
```

### 2. Dijkstra's Algorithm with Modern `PriorityQueue`
In C# (.NET 6+), Dijkstra's algorithm uses `PriorityQueue<int, int>` where the priority is current distance:
- Always skip stale nodes: `if (d > dist[u]) continue;`.
- Initialize `dist` array with `int.MaxValue` (or `long.MaxValue` for large weights).
- Dijkstra is **only valid for non-negative edge weights**. Negative edges violate the greedy property and require Bellman-Ford or SPFA.

---

## 📅 Day-by-Day Schedule

### Day 1: Theory Anchor & Baseline Pattern (Kahn's Algorithm)
- **Step 1 (45 mins) | Concept Video**: [Topological Sort & Kahn's Algorithm](https://www.youtube.com/watch?v=dis_c/TopologicalSort) or [Graph Topological Sorting](https://www.youtube.com/watch?v=eL-KzMXSXXI) by *Abdul Bari*
  - Key focus: What is an in-degree? Why do zero in-degree nodes represent valid starting tasks?
- **Step 2 (20 mins) | Pen & Paper Invariant**:
  - Draw a DAG of 5 courses. Compute in-degrees, push 0 in-degree nodes into a queue, and trace the topological ordering.
- **Step 3 (45 mins) | Problem 1 (Warm-Up)**:
  - [Course Schedule II](https://leetcode.com/problems/course-schedule-ii/) (NeetCode - Medium)
  - Objective: Return topological ordering or empty array if cycle exists.
- **Step 4 (10 mins) | Active Recall**:

<details markdown="1">
<summary><b>🧠 Self-Test: How does Kahn's algorithm detect if a graph contains a cycle?</b></summary>

<b>Answer:</b> If the total number of nodes dequeued and added to the topological ordering is strictly less than $V$, a cycle exists! Nodes participating in a directed cycle never reach an in-degree of 0.

</details>

---

### Day 2: DSU Cycle Detection in Undirected Graphs (Redundant Connection)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: In an undirected graph with $N$ nodes and $N$ edges, how does Union-Find locate the edge that creates a cycle?</b></summary>

<b>Answer:</b> For each edge $(u, v)$, call <code>Union(u, v)</code>. If <code>Find(u) == Find(v)</code> before merging, $u$ and $v$ are already connected, meaning this edge is the redundant cycle creator!

</details>

- **Deep Practice (75–85 mins) | Problem 2**:
  - [Redundant Connection](https://leetcode.com/problems/redundant-connection/) (NeetCode - Medium)
  - Thinking prompts: Implement standard Union-Find with path compression.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int[] FindRedundantConnection(int[][] edges) {
        int n = edges.Length;
        int[] parent = new int[n + 1];
        for (int i = 1; i <= n; i++) parent[i] = i;

        int Find(int x) {
            if (parent[x] == x) return x;
            return parent[x] = Find(parent[x]);
        }

        foreach (var edge in edges) {
            int rootU = Find(edge[0]);
            int rootV = Find(edge[1]);
            if (rootU == rootV) return edge; // Redundant edge
            parent[rootU] = rootV;
        }
        return Array.Empty<int>();
    }
}
```

<b>Complexity:</b> Time: $O(N \cdot \alpha(N)) \approx O(N)$, Space: $O(N)$.

</details>

---

### Day 3: Component Counting via DSU
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: Starting with $N$ isolated nodes, how does calling <code>Union(u, v)</code> track the count of connected components?</b></summary>

<b>Answer:</b> Every successful union that merges two previously disjoint roots decrements the total component count by exactly 1.

</details>

- **Deep Practice (75–85 mins) | Problem 3**:
  - [Number of Connected Components in an Undirected Graph](https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/) (NeetCode - Medium)
  - Thinking prompts: Start with `count = n`. For each edge, decrement `count` if roots differ.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int CountComponents(int n, int[][] edges) {
        int[] parent = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i;
        int components = n;

        int Find(int i) {
            if (parent[i] == i) return i;
            return parent[i] = Find(parent[i]);
        }

        foreach (var edge in edges) {
            int r1 = Find(edge[0]);
            int r2 = Find(edge[1]);
            if (r1 != r2) {
                parent[r1] = r2;
                components--;
            }
        }
        return components;
    }
}
```

<b>Complexity:</b> Time: $O(V + E \cdot \alpha(V))$, Space: $O(V)$.

</details>

---

### Day 4: Tree Validity Invariants (Graph Valid Tree)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: What are the two necessary and sufficient conditions for an undirected graph to be a valid tree?</b></summary>

<b>Answer:</b> 1. Exactly $N - 1$ edges (no cycles); 2. All $N$ nodes form exactly 1 connected component.

</details>

- **Deep Practice (75–85 mins) | Problem 4**:
  - [Graph Valid Tree](https://leetcode.com/problems/graph-valid-tree/) (NeetCode - Medium)
  - Thinking prompts: Quick exit `if (edges.Length != n - 1) return false;`. Then test connectivity with DSU or BFS.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public bool ValidTree(int n, int[][] edges) {
        if (edges.Length != n - 1) return false;

        int[] parent = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i;

        int Find(int i) {
            if (parent[i] == i) return i;
            return parent[i] = Find(parent[i]);
        }

        foreach (var edge in edges) {
            int r1 = Find(edge[0]);
            int r2 = Find(edge[1]);
            if (r1 == r2) return false; // Cycle detected
            parent[r1] = r2;
        }
        return true;
    }
}
```

<b>Complexity:</b> Time: $O(N \cdot \alpha(N))$, Space: $O(N)$.

</details>

---

### Day 5: Midpoint Shift — Dijkstra's Algorithm
- **Step 1 (20 mins) | Focused Video**: [Dijkstra's Algorithm Explained](https://www.youtube.com/watch?v=XB4MIexjvY0) by *Abdul Bari*
  - Key focus: Relaxation step `if (dist[u] + weight < dist[v])`. Why greedy choice guarantees optimality with non-negative edges.
- **Deep Practice (75 mins) | Problem 5**:
  - [Network Delay Time](https://leetcode.com/problems/network-delay-time/) (NeetCode - Medium)
  - Thinking prompts: Use modern .NET 6+ `PriorityQueue<(int node, int d), int>`.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int NetworkDelayTime(int[][] times, int n, int k) {
        List<(int dest, int w)>[] graph = new List<(int, int)>[n + 1];
        for (int i = 1; i <= n; i++) graph[i] = new List<(int, int)>();
        foreach (var t in times) {
            graph[t[0]].Add((t[1], t[2]));
        }

        int[] dist = new int[n + 1];
        Array.Fill(dist, int.MaxValue);
        dist[k] = 0;

        PriorityQueue<int, int> pq = new();
        pq.Enqueue(k, 0);

        while (pq.Count > 0) {
            pq.TryDequeue(out int u, out int d);
            if (d > dist[u]) continue; // Stale path

            foreach (var (v, weight) in graph[u]) {
                if (dist[u] + weight < dist[v]) {
                    dist[v] = dist[u] + weight;
                    pq.Enqueue(v, dist[v]);
                }
            }
        }

        int maxDelay = 0;
        for (int i = 1; i <= n; i++) {
            if (dist[i] == int.MaxValue) return -1;
            maxDelay = Math.Max(maxDelay, dist[i]);
        }
        return maxDelay;
    }
}
```

<b>Complexity:</b> Time: $O(E \log V)$, Space: $O(V + E)$.

</details>

---

### Day 6: Minimum Spanning Tree (Min Cost to Connect All Points)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: What is the difference between Prim's algorithm and Kruskal's algorithm?</b></summary>

<b>Answer:</b> Prim's builds an MST by growing a single connected tree outward using a priority queue. Kruskal's sorts all edges globally and adds them one-by-one using Union-Find to prevent cycles.

</details>

- **Deep Practice (75–85 mins) | Problem 6**:
  - [Min Cost to Connect All Points](https://leetcode.com/problems/min-cost-to-connect-all-points/) (NeetCode - Medium)
  - Thinking prompts: Complete graph of $N$ points. Prim's algorithm with Manhattan distance $|x_1 - x_2| + |y_1 - y_2|$.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int MinCostConnectPoints(int[][] points) {
        int n = points.Length;
        bool[] inMst = new bool[n];
        int[] minDist = new int[n];
        Array.Fill(minDist, int.MaxValue);
        minDist[0] = 0;

        int totalCost = 0;
        for (int step = 0; step < n; step++) {
            int u = -1;
            for (int i = 0; i < n; i++) {
                if (!inMst[i] && (u == -1 || minDist[i] < minDist[u])) {
                    u = i;
                }
            }

            inMst[u] = true;
            totalCost += minDist[u];

            for (int v = 0; v < n; v++) {
                if (!inMst[v]) {
                    int dist = Math.Abs(points[u][0] - points[v][0]) + Math.Abs(points[u][1] - points[v][1]);
                    if (dist < minDist[v]) {
                        minDist[v] = dist;
                    }
                }
            }
        }
        return totalCost;
    }
}
```

<b>Complexity:</b> Time: $O(N^2)$ (dense graph Prim's), Space: $O(N)$.

</details>

---

### Day 7: Minimax Path via Dijkstra (Swim in Rising Water)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: How does Dijkstra adapt to find the path that minimizes the MAXIMUM edge weight encountered?</b></summary>

<b>Answer:</b> The edge relaxation formula changes from additive <code>dist[u] + weight</code> to bottleneck <code>Math.Max(dist[u], grid[nr][nc])</code>. The priority queue always expands the cell with the lowest water elevation.

</details>

- **Deep Practice (75–85 mins) | Problem 7**:
  - [Swim in Rising Water](https://leetcode.com/problems/swim-in-rising-water/) (NeetCode - Hard)
  - Thinking prompts: Modified Dijkstra where distance is $\max(\text{water elevation})$.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int SwimInWater(int[][] grid) {
        int n = grid.Length;
        PriorityQueue<(int r, int c), int> pq = new();
        bool[,] visited = new bool[n, n];

        pq.Enqueue((0, 0), grid[0][0]);
        visited[0, 0] = true;

        int[] dr = { -1, 1, 0, 0 };
        int[] dc = { 0, 0, -1, 1 };

        while (pq.Count > 0) {
            pq.TryDequeue(out var cell, out int time);
            int r = cell.r, c = cell.c;

            if (r == n - 1 && c == n - 1) return time;

            for (int i = 0; i < 4; i++) {
                int nr = r + dr[i];
                int nc = c + dc[i];

                if (nr >= 0 && nr < n && nc >= 0 && nc < n && !visited[nr, nc]) {
                    visited[nr, nc] = true;
                    int nextTime = Math.Max(time, grid[nr][nc]);
                    pq.Enqueue((nr, nc), nextTime);
                }
            }
        }
        return -1;
    }
}
```

<b>Complexity:</b> Time: $O(N^2 \log N)$, Space: $O(N^2)$.

</details>

---

### Day 8: String Lexicographical DAG (Alien Dictionary)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: In Alien Dictionary, what invalid prefix case makes an alien order impossible?</b></summary>

<b>Answer:</b> If an earlier word has a later word as a prefix but is strictly longer (e.g. <code>["apple", "app"]</code>). In lexicographical ordering, shorter prefixes must always come first.

</details>

- **Deep Practice (75–85 mins) | Problem 8**:
  - [Alien Dictionary](https://leetcode.com/problems/alien-dictionary/) (NeetCode - Hard)
  - Thinking prompts: Compare adjacent words. Find first differing character $c_1 \ne c_2$, add directed edge $c_1 \to c_2$. Run Kahn's algorithm.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public string AlienOrder(string[] words) {
        Dictionary<char, HashSet<char>> adj = new();
        Dictionary<char, int> inDegree = new();

        foreach (var w in words) {
            foreach (var c in w) {
                adj.TryAdd(c, new HashSet<char>());
                inDegree.TryAdd(c, 0);
            }
        }

        for (int i = 0; i < words.Length - 1; i++) {
            string w1 = words[i], w2 = words[i + 1];
            if (w1.Length > w2.Length && w1.StartsWith(w2)) return ""; // Invalid prefix order

            for (int j = 0; j < Math.Min(w1.Length, w2.Length); j++) {
                if (w1[j] != w2[j]) {
                    if (adj[w1[j]].Add(w2[j])) {
                        inDegree[w2[j]]++;
                    }
                    break;
                }
            }
        }

        Queue<char> queue = new();
        foreach (var kvp in inDegree) {
            if (kvp.Value == 0) queue.Enqueue(kvp.Key);
        }

        var sb = new System.Text.StringBuilder();
        while (queue.Count > 0) {
            char curr = queue.Dequeue();
            sb.Append(curr);

            foreach (char next in adj[curr]) {
                if (--inDegree[next] == 0) {
                    queue.Enqueue(next);
                }
            }
        }
        return sb.Length == inDegree.Count ? sb.ToString() : "";
    }
}
```

<b>Complexity:</b> Time: $O(C)$ where $C$ is total characters across all words, Space: $O(U + E)$ where $U \le 26$.

</details>

---

### Day 9: State-Bounded Dijkstra / Bellman-Ford (Cheapest Flights)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: Why does standard Dijkstra fail on Cheapest Flights Within K Stops?</b></summary>

<b>Answer:</b> Standard Dijkstra greedily rejects a path to node $u$ with higher cost, even if that higher-cost path uses fewer stops and could reach the destination within $K$ stops while the cheaper path runs out of stops.

</details>

- **Deep Practice (75–85 mins) | Problem 9**:
  - [Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/) (NeetCode - Medium)
  - Thinking prompts: Bellman-Ford using $K + 1$ relaxations with a temporary copy of `dist` array.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int FindCheapestPrice(int n, int[][] flights, int src, int dst, int k) {
        int[] dist = new int[n];
        Array.Fill(dist, int.MaxValue);
        dist[src] = 0;

        for (int i = 0; i <= k; i++) {
            int[] temp = (int[])dist.Clone();
            foreach (var f in flights) {
                int u = f[0], v = f[1], price = f[2];
                if (dist[u] != int.MaxValue && dist[u] + price < temp[v]) {
                    temp[v] = dist[u] + price;
                }
            }
            dist = temp;
        }
        return dist[dst] == int.MaxValue ? -1 : dist[dst];
    }
}
```

<b>Complexity:</b> Time: $O(K \cdot E)$, Space: $O(V)$.

</details>

---

### Day 10: Multi-Field String Equivalence (Accounts Merge)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: In Accounts Merge, what should be the elements connected by DSU?</b></summary>

<b>Answer:</b> The account indices $(0 \dots N - 1)$ or the emails themselves! If an email maps to an earlier account index, union the current account index with that earlier account index.

</details>

- **Deep Practice (75–85 mins) | Problem 10**:
  - [Accounts Merge](https://leetcode.com/problems/accounts-merge/) (NeetCode - Medium)
  - Thinking prompts: Map `email -> first_account_index`. Union account indices that share emails.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public IList<IList<string>> AccountsMerge(IList<IList<string>> accounts) {
        int n = accounts.Count;
        int[] parent = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i;

        int Find(int i) {
            if (parent[i] == i) return i;
            return parent[i] = Find(parent[i]);
        }

        Dictionary<string, int> emailToAcc = new();
        for (int i = 0; i < n; i++) {
            for (int j = 1; j < accounts[i].Count; j++) {
                string email = accounts[i][j];
                if (emailToAcc.TryGetValue(email, out int prevAcc)) {
                    int r1 = Find(i), r2 = Find(prevAcc);
                    if (r1 != r2) parent[r1] = r2;
                } else {
                    emailToAcc[email] = i;
                }
            }
        }

        Dictionary<int, List<string>> leaderEmails = new();
        foreach (var kvp in emailToAcc) {
            int root = Find(kvp.Value);
            if (!leaderEmails.TryGetValue(root, out var list)) {
                list = new List<string>();
                leaderEmails[root] = list;
            }
            list.Add(kvp.Key);
        }

        List<IList<string>> result = new();
        foreach (var kvp in leaderEmails) {
            kvp.Value.Sort(StringComparer.Ordinal);
            List<string> row = new() { accounts[kvp.Key][0] };
            row.AddRange(kvp.Value);
            result.Add(row);
        }
        return result;
    }
}
```

<b>Complexity:</b> Time: $O(N \cdot K \log(N \cdot K))$, Space: $O(N \cdot K)$.

</details>

---

### Day 11: Hierholzer's Eulerian Path (Reconstruct Itinerary)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: In Hierholzer's algorithm, why do we add nodes to the result list in postorder (after all outgoing edges are exhausted)?</b></summary>

<b>Answer:</b> Postorder insertion ensures that dead-end cycles and sub-tours are resolved first and prepended in reverse, guaranteeing a continuous Eulerian path that visits every edge.

</details>

- **Deep Practice (75–85 mins) | Problem 11**:
  - [Reconstruct Itinerary](https://leetcode.com/problems/reconstruct-itinerary/) (NeetCode - Hard)
  - Thinking prompts: Min-heap or sorted list of airport destinations for lexicographical tie-breaking.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public IList<string> FindItinerary(IList<IList<string>> tickets) {
        Dictionary<string, PriorityQueue<string, string>> adj = new();
        foreach (var t in tickets) {
            if (!adj.TryGetValue(t[0], out var pq)) {
                pq = new PriorityQueue<string, string>();
                adj[t[0]] = pq;
            }
            pq.Enqueue(t[1], t[1]);
        }

        LinkedList<string> route = new();
        Dfs("JFK", adj, route);
        return route.ToList();
    }

    private void Dfs(string airport, Dictionary<string, PriorityQueue<string, string>> adj, LinkedList<string> route) {
        if (adj.TryGetValue(airport, out var pq)) {
            while (pq.Count > 0) {
                string next = pq.Dequeue();
                Dfs(next, adj, route);
            }
        }
        route.AddFirst(airport); // Postorder insertion
    }
}
```

<b>Complexity:</b> Time: $O(E \log E)$, Space: $O(V + E)$.

</details>

---

### Day 12: CSES & Competitive Benchmarks + Spaced Retrieval
- **Benchmark 1**: [CSES: Course Schedule](https://cses.fi/problemset/task/1679) (Kahn's topological sort on $10^5$ nodes)
- **Benchmark 2**: [CSES: Shortest Routes I](https://cses.fi/problemset/task/1671) (High-speed Dijkstra with 64-bit distances)
- **Benchmark 3**: [CSES: Road Reparation](https://cses.fi/problemset/task/1675) (Kruskal's MST / total road cost)
- **Benchmark 4**: [CSES: Road Construction](https://cses.fi/problemset/task/1676) (DSU maintaining max component size)
- **Benchmark 5**: [Codeforces 20C - Dijkstra?](https://codeforces.com/problemset/problem/20/C) (Dijkstra with full path reconstruction)

---

### Spaced Retrieval Flashcards

<details markdown="1">
<summary><b>Flashcard 1: What is Path Compression in DSU, and why is its complexity near-constant?</b></summary>

Path compression flattens the tree during <code>Find(x)</code> by directly attaching every visited node to the root. Combined with union by rank/size, the amortized time per operation becomes $O(\alpha(N))$ (inverse Ackermann function), which is $\le 4$ for all realistic values of $N$.
</details>

<details markdown="1">
<summary><b>Flashcard 2: Why can Dijkstra fail when negative edge weights exist?</b></summary>

Dijkstra assumes that once a node is dequeued from the priority queue, its shortest distance is finalized and will never decrease. A negative edge encountered later could lower that finalized distance, violating the greedy invariant.
</details>

<details markdown="1">
<summary><b>Flashcard 3: How does Kahn's algorithm break ties when multiple valid topological sorts exist?</b></summary>

By replacing the standard FIFO <code>Queue&lt;int&gt;</code> with a <code>PriorityQueue&lt;int, int&gt;</code> (min-heap), you can guarantee that the lexicographically smallest topological sort is produced.
</details>

---

## 🎯 Topic Checklist (16 Problems)

| Day | # | Problem | Platform | Difficulty | Status |
|:---:|:---:|:---|:---:|:---:|:---:|
| Day 1 | 1 | [Course Schedule II](https://leetcode.com/problems/course-schedule-ii/) | NeetCode | Medium | - [ ] |
| Day 2 | 2 | [Redundant Connection](https://leetcode.com/problems/redundant-connection/) | NeetCode | Medium | - [ ] |
| Day 3 | 3 | [Number of Connected Components](https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/) | NeetCode | Medium | - [ ] |
| Day 4 | 4 | [Graph Valid Tree](https://leetcode.com/problems/graph-valid-tree/) | NeetCode | Medium | - [ ] |
| Day 5 | 5 | [Network Delay Time](https://leetcode.com/problems/network-delay-time/) | NeetCode | Medium | - [ ] |
| Day 6 | 6 | [Min Cost to Connect All Points](https://leetcode.com/problems/min-cost-to-connect-all-points/) | NeetCode | Medium | - [ ] |
| Day 7 | 7 | [Swim in Rising Water](https://leetcode.com/problems/swim-in-rising-water/) | NeetCode | Hard | - [ ] |
| Day 8 | 8 | [Alien Dictionary](https://leetcode.com/problems/alien-dictionary/) | NeetCode | Hard | - [ ] |
| Day 9 | 9 | [Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/) | NeetCode | Medium | - [ ] |
| Day 10 | 10 | [Accounts Merge](https://leetcode.com/problems/accounts-merge/) | NeetCode | Medium | - [ ] |
| Day 11 | 11 | [Reconstruct Itinerary](https://leetcode.com/problems/reconstruct-itinerary/) | NeetCode | Hard | - [ ] |
| Day 12 | 12 | [Course Schedule (CSES)](https://cses.fi/problemset/task/1679) | CSES | Medium | - [ ] |
| Day 12 | 13 | [Shortest Routes I](https://cses.fi/problemset/task/1671) | CSES | Medium | - [ ] |
| Day 12 | 14 | [Road Reparation](https://cses.fi/problemset/task/1675) | CSES | Medium | - [ ] |
| Day 12 | 15 | [Road Construction](https://cses.fi/problemset/task/1676) | CSES | Medium | - [ ] |
| Day 12 | 16 | [Dijkstra? (20C)](https://codeforces.com/problemset/problem/20/C) | Codeforces | Rating 1400 | - [ ] |

---

[⬅️ Previous Topic: Graphs I](10-graphs-traversal.html) | [Home](index.html) | [Next Topic: 1-D Dynamic Programming ➡️](12-1d-dp.html)
