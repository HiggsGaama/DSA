---
layout: default
title: "Graphs I: BFS, DFS & Grid Traversals"
---

# Graphs I: BFS, DFS & Grid Traversals

> **Estimated Time**: 11 Days • **Pace**: 1.5–2 hours/day (Strictly 1 problem/day) • **Language**: C# (.NET 6+)

---

## 🛠️ C# Mental Model & Gotchas

### 1. Explicit Graph Representation in C#
- **Adjacency List**: Always use array of lists `List<int>[] graph = new List<int>[n]` instead of `Dictionary<int, List<int>>` when nodes are $0$-indexed integers. It provides $O(1)$ random access without hashing overhead.
```csharp
List<int>[] graph = new List<int>[n];
for (int i = 0; i < n; i++) graph[i] = new List<int>();
foreach (var edge in edges) {
  graph[edge[0]].Add(edge[1]);
  graph[edge[1]].Add(edge[0]); // if undirected
}
```

### 2. Implicit Grid Traversals & Direction Vectors
In grid problems, avoid copy-pasting four directional calls. Use compact direction arrays:
```csharp
int[] dr = { -1, 1, 0, 0 };
int[] dc = { 0, 0, -1, 1 };
for (int i = 0; i < 4; i++) {
  int nr = r + dr[i];
  int nc = c + dc[i];
  if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && !visited[nr, nc]) {
      // process neighbor
  }
}
```

### 3. Multi-Source BFS Invariant
When calculating shortest paths from multiple simultaneous starting points (e.g. `Rotting Oranges`, `Walls and Gates`):
- Enqueue **all initial source nodes into the BFS queue at time 0**.
- Mark them visited immediately upon enqueuing to prevent duplicate processing.
- A single BFS queue guarantees that nodes are visited in strictly non-decreasing order of distance.

---

## 📅 Day-by-Day Schedule

### Day 1: Theory Anchor & Baseline Pattern
- **Step 1 (45 mins) | Concept Video**: [Graph Traversal BFS and DFS](https://www.youtube.com/watch?v=pcKY4hjDrxk) by *Abdul Bari*
  - Key focus: What is the difference between BFS (level exploration via Queue) and DFS (branch exploration via recursion/stack)?
- **Step 2 (20 mins) | Pen & Paper Invariant**:
  - Draw a $4 \times 5$ island matrix. Manually trace the DFS flood-fill that converts land `'1'` to water `'0'`.
- **Step 3 (45 mins) | Problem 1 (Warm-Up)**:
  - [Number of Islands](https://leetcode.com/problems/number-of-islands/) (NeetCode - Medium)
  - Objective: In-place flood fill marking visited cells.
- **Step 4 (10 mins) | Active Recall**:

<details markdown="1">
<summary><b>🧠 Self-Test: When running BFS, at what exact moment should a node be marked as visited?</b></summary>

<b>Answer:</b> Immediately upon being <b>enqueued</b> into the queue, NEVER when it is dequeued! If you mark nodes as visited upon dequeuing, multiple paths can enqueue the same node multiple times, causing exponential memory blowup and TLE.

</details>

---

### Day 2: Connected Component Area Aggregation
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: How does DFS aggregate the total area of an island?</b></summary>

<b>Answer:</b> The base case returns 0 for water/out-of-bounds cells. For a valid land cell, mark it as visited and return $1 + \sum \text{DFS}(\text{all 4 neighbors})$.

</details>

- **Deep Practice (75–85 mins) | Problem 2**:
  - [Max Area of Island](https://leetcode.com/problems/max-area-of-island/) (NeetCode - Medium)
  - Thinking prompts: In-place mutation `grid[r][c] = 0` avoids allocating a visited array.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int MaxAreaOfIsland(int[][] grid) {
        int maxArea = 0;
        for (int r = 0; r < grid.Length; r++) {
            for (int c = 0; c < grid[0].Length; c++) {
                if (grid[r][c] == 1) {
                    maxArea = Math.Max(maxArea, Dfs(r, c, grid));
                }
            }
        }
        return maxArea;
    }

    private int Dfs(int r, int c, int[][] grid) {
        if (r < 0 || r >= grid.Length || c < 0 || c >= grid[0].Length || grid[r][c] == 0) {
            return 0;
        }

        grid[r][c] = 0; // Mark visited
        return 1 + Dfs(r + 1, c, grid) + Dfs(r - 1, c, grid) +
                   Dfs(r, c + 1, grid) + Dfs(r, c - 1, grid);
    }
}
```

<b>Complexity:</b> Time: $O(M \cdot N)$, Space: $O(M \cdot N)$ recursion depth.

</details>

---

### Day 3: Graph Deep Copy & Reference Mapping (Clone Graph)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: How do you handle cycles when cloning a graph with DFS or BFS?</b></summary>

<b>Answer:</b> Maintain a <code>Dictionary&lt;Node, Node&gt;</code> mapping each original node to its newly created clone. If an encountered neighbor already exists in the dictionary, reuse the existing cloned node reference instead of re-cloning.

</details>

- **Deep Practice (75–85 mins) | Problem 3**:
  - [Clone Graph](https://leetcode.com/problems/clone-graph/) (NeetCode - Medium)
  - Thinking prompts: Map original `Node` to cloned `Node`.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    private readonly Dictionary<Node, Node> _clones = new();

    public Node CloneGraph(Node node) {
        if (node == null) return null;
        if (_clones.TryGetValue(node, out var clone)) return clone;

        Node copy = new(node.val);
        _clones[node] = copy;

        foreach (var neighbor in node.neighbors) {
            copy.neighbors.Add(CloneGraph(neighbor));
        }
        return copy;
    }
}
```

<b>Complexity:</b> Time: $O(V + E)$, Space: $O(V)$ auxiliary dictionary and stack.

</details>

---

### Day 4: Reverse Reachability from Ocean Boundaries
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: Why is searching outward from the Pacific and Atlantic ocean borders $O(M \cdot N)$ while searching from each cell to the ocean is $O(M^2 \cdot N^2)$?</b></summary>

<b>Answer:</b> Starting from oceans allows us to walk "uphill" ($\text{neighbor} \ge \text{current}$) once for each ocean into two boolean matrices <code>pacificReachable</code> and <code>atlanticReachable</code>. The intersection cells are those that can reach both.

</details>

- **Deep Practice (75–85 mins) | Problem 4**:
  - [Pacific Atlantic Water Flow](https://leetcode.com/problems/pacific-atlantic-water-flow/) (NeetCode - Medium)
  - Thinking prompts: Run two multi-source DFS/BFS passes from Pacific and Atlantic borders.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public IList<IList<int>> PacificAtlantic(int[][] heights) {
        int m = heights.Length, n = heights[0].Length;
        bool[,] pac = new bool[m, n];
        bool[,] atl = new bool[m, n];

        for (int c = 0; c < n; c++) {
            Dfs(0, c, pac, heights[0][c], heights);
            Dfs(m - 1, c, atl, heights[m - 1][c], heights);
        }
        for (int r = 0; r < m; r++) {
            Dfs(r, 0, pac, heights[r][0], heights);
            Dfs(r, n - 1, atl, heights[r][n - 1], heights);
        }

        List<IList<int>> result = new();
        for (int r = 0; r < m; r++) {
            for (int c = 0; c < n; c++) {
                if (pac[r, c] && atl[r, c]) {
                    result.Add(new List<int> { r, c });
                }
            }
        }
        return result;
    }

    private void Dfs(int r, int c, bool[,] ocean, int prevHeight, int[][] heights) {
        if (r < 0 || r >= heights.Length || c < 0 || c >= heights[0].Length) return;
        if (ocean[r, c] || heights[r][c] < prevHeight) return;

        ocean[r, c] = true;
        Dfs(r + 1, c, ocean, heights[r][c], heights);
        Dfs(r - 1, c, ocean, heights[r][c], heights);
        Dfs(r, c + 1, ocean, heights[r][c], heights);
        Dfs(r, c - 1, ocean, heights[r][c], heights);
    }
}
```

<b>Complexity:</b> Time: $O(M \cdot N)$, Space: $O(M \cdot N)$.

</details>

---

### Day 5: Boundary Flood-Fill & Inversion (Surrounded Regions)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: Which 'O' cells can NEVER be captured in Surrounded Regions?</b></summary>

<b>Answer:</b> Any 'O' connected directly or indirectly to the outer boundary of the board.

</details>

- **Deep Practice (75–85 mins) | Problem 5**:
  - [Surrounded Regions](https://leetcode.com/problems/surrounded-regions/) (NeetCode - Medium)
  - Thinking prompts: 1. Flood-fill from border `'O'`s changing them to `'T'`; 2. Flip all remaining `'O'` to `'X'`; 3. Restore `'T'` back to `'O'`.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public void Solve(char[][] board) {
        int m = board.Length, n = board[0].Length;

        // Phase 1: Mark non-surrounded boundary regions as 'T'
        for (int r = 0; r < m; r++) {
            Dfs(r, 0, board);
            Dfs(r, n - 1, board);
        }
        for (int c = 0; c < n; c++) {
            Dfs(0, c, board);
            Dfs(m - 1, c, board);
        }

        // Phase 2: Invert surrounded 'O' -> 'X' and restore 'T' -> 'O'
        for (int r = 0; r < m; r++) {
            for (int c = 0; c < n; c++) {
                if (board[r][c] == 'O') board[r][c] = 'X';
                else if (board[r][c] == 'T') board[r][c] = 'O';
            }
        }
    }

    private void Dfs(int r, int c, char[][] board) {
        if (r < 0 || r >= board.Length || c < 0 || c >= board[0].Length || board[r][c] != 'O') {
            return;
        }
        board[r][c] = 'T';
        Dfs(r + 1, c, board);
        Dfs(r - 1, c, board);
        Dfs(r, c + 1, board);
        Dfs(r, c - 1, board);
    }
}
```

<b>Complexity:</b> Time: $O(M \cdot N)$, Space: $O(M \cdot N)$ recursion depth.

</details>

---

### Day 6: Multi-Source Simultaneous BFS (Rotting Oranges)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: In Rotting Oranges, why can standard single-source BFS or DFS fail to compute the minimum elapsed time?</b></summary>

<b>Answer:</b> Multiple rotten oranges rot adjacent fresh oranges simultaneously. Enqueuing all rotten oranges at time $0$ simulates parallel time steps.

</details>

- **Deep Practice (75–85 mins) | Problem 6**:
  - [Rotting Oranges](https://leetcode.com/problems/rotting-oranges/) (NeetCode - Medium)
  - Thinking prompts: Count initial fresh oranges. Enqueue all rotten cells `(r, c)` at time 0. Process BFS in level waves.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int OrangesRotting(int[][] grid) {
        int m = grid.Length, n = grid[0].Length;
        Queue<(int r, int c)> queue = new();
        int fresh = 0;

        for (int r = 0; r < m; r++) {
            for (int c = 0; c < n; c++) {
                if (grid[r][c] == 2) queue.Enqueue((r, c));
                else if (grid[r][c] == 1) fresh++;
            }
        }

        if (fresh == 0) return 0;

        int minutes = -1;
        int[] dr = { -1, 1, 0, 0 };
        int[] dc = { 0, 0, -1, 1 };

        while (queue.Count > 0) {
            int levelSize = queue.Count;
            minutes++;

            for (int i = 0; i < levelSize; i++) {
                var (r, c) = queue.Dequeue();
                for (int d = 0; d < 4; d++) {
                    int nr = r + dr[d];
                    int nc = c + dc[d];

                    if (nr >= 0 && nr < m && nc >= 0 && nc < n && grid[nr][nc] == 1) {
                        grid[nr][nc] = 2; // Mark rotten
                        fresh--;
                        queue.Enqueue((nr, nc));
                    }
                }
            }
        }
        return fresh == 0 ? minutes : -1;
    }
}
```

<b>Complexity:</b> Time: $O(M \cdot N)$, Space: $O(M \cdot N)$.

</details>

---

### Day 7: Reverse Distance Propagation (Walls and Gates)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: In Walls and Gates, why do we start BFS from Gates instead of Rooms?</b></summary>

<b>Answer:</b> Searching from each room takes $O(R \times M \cdot N)$. Searching simultaneously from all gates takes $O(M \cdot N)$ total time because each empty room is updated with its shortest distance on the first visit.

</details>

- **Deep Practice (75–85 mins) | Problem 7**:
  - [Walls and Gates](https://leetcode.com/problems/walls-and-gates/) (NeetCode - Medium)
  - Thinking prompts: Enqueue all cells equal to `0` (gates). Distance updates to `rooms[r][c] + 1`.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public void WallsAndGates(int[][] rooms) {
        int m = rooms.Length, n = rooms[0].Length;
        Queue<(int r, int c)> queue = new();

        for (int r = 0; r < m; r++) {
            for (int c = 0; c < n; c++) {
                if (rooms[r][c] == 0) {
                    queue.Enqueue((r, c));
                }
            }
        }

        int[] dr = { -1, 1, 0, 0 };
        int[] dc = { 0, 0, -1, 1 };

        while (queue.Count > 0) {
            var (r, c) = queue.Dequeue();
            for (int i = 0; i < 4; i++) {
                int nr = r + dr[i];
                int nc = c + dc[i];

                if (nr >= 0 && nr < m && nc >= 0 && nc < n && rooms[nr][nc] == 2147483647) {
                    rooms[nr][nc] = rooms[r][c] + 1;
                    queue.Enqueue((nr, nc));
                }
            }
        }
    }
}
```

<b>Complexity:</b> Time: $O(M \cdot N)$, Space: $O(M \cdot N)$.

</details>

---

### Day 8: Cycle Detection in Directed Graphs (Course Schedule)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: Why is a simple boolean <code>visited[]</code> array insufficient for cycle detection in a directed graph?</b></summary>

<b>Answer:</b> In a directed graph, visiting an already visited node is not a cycle unless that node is currently part of the active recursion stack! We need a 3-color state: unvisited (0), currently visiting (1), and finished (2).

</details>

- **Deep Practice (75–85 mins) | Problem 8**:
  - [Course Schedule](https://leetcode.com/problems/course-schedule/) (NeetCode - Medium)
  - Thinking prompts: Use 3-state tracking (`0 = unvisited`, `1 = visiting`, `2 = visited`) or Kahn's indegree BFS algorithm.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public bool CanFinish(int numCourses, int[][] prerequisites) {
        List<int>[] graph = new List<int>[numCourses];
        for (int i = 0; i < numCourses; i++) graph[i] = new List<int>();

        foreach (var pre in prerequisites) {
            graph[pre[1]].Add(pre[0]);
        }

        int[] state = new int[numCourses]; // 0: unvisited, 1: visiting, 2: visited
        for (int i = 0; i < numCourses; i++) {
            if (HasCycle(i, graph, state)) return false;
        }
        return true;
    }

    private bool HasCycle(int u, List<int>[] graph, int[] state) {
        if (state[u] == 1) return true;  // Found back-edge to ancestor in call stack
        if (state[u] == 2) return false; // Already verified acyclic

        state[u] = 1; // Mark visiting
        foreach (int v in graph[u]) {
            if (HasCycle(v, graph, state)) return true;
        }
        state[u] = 2; // Mark finished
        return false;
    }
}
```

<b>Complexity:</b> Time: $O(V + E)$, Space: $O(V + E)$.

</details>

---

### Day 9: Abstract State Space BFS (Open the Lock)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: How many transitions exist from any 4-digit lock state like <code>"0000"</code>?</b></summary>

<b>Answer:</b> Exactly 8 transitions (each of the 4 wheels can be rotated +1 or -1 modulo 10).

</details>

- **Deep Practice (75–85 mins) | Problem 9**:
  - [Open the Lock](https://leetcode.com/problems/open-the-lock/) (LeetCode 752 - Medium)
  - Thinking prompts: Put deadends into a `HashSet<string>`. Run standard BFS on 4-digit strings.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int OpenLock(string[] deadends, string target) {
        HashSet<string> dead = new(deadends);
        if (dead.Contains("0000")) return -1;
        if (target == "0000") return 0;

        Queue<string> queue = new();
        HashSet<string> visited = new();
        queue.Enqueue("0000");
        visited.Add("0000");

        int turns = 0;
        while (queue.Count > 0) {
            int count = queue.Count;
            turns++;

            for (int i = 0; i < count; i++) {
                string curr = queue.Dequeue();
                foreach (string next in GetNeighbors(curr)) {
                    if (next == target) return turns;
                    if (!dead.Contains(next) && visited.Add(next)) {
                        queue.Enqueue(next);
                    }
                }
            }
        }
        return -1;
    }

    private IEnumerable<string> GetNeighbors(string s) {
        char[] arr = s.ToCharArray();
        for (int i = 0; i < 4; i++) {
            char orig = arr[i];
            arr[i] = orig == '9' ? '0' : (char)(orig + 1);
            yield return new string(arr);
            arr[i] = orig == '0' ? '9' : (char)(orig - 1);
            yield return new string(arr);
            arr[i] = orig;
        }
    }
}
```

<b>Complexity:</b> Time: $O(10^4 \cdot 8)$, Space: $O(10^4)$.

</details>

---

### Day 10: Pattern Graph Shortest Path (Word Ladder)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: In Word Ladder with $N$ words of length $L$, why is checking all pairs $O(N^2 \cdot L)$ suboptimal, and how do wildcard patterns help?</b></summary>

<b>Answer:</b> By mapping words to wildcard patterns (e.g. <code>*ot</code>, <code>h*t</code>, <code>ho*</code>) in a <code>Dictionary&lt;string, List&lt;string&gt;&gt;</code>, graph construction and neighbor lookup drop to $O(N \cdot L^2)$.

</details>

- **Deep Practice (75–85 mins) | Problem 10**:
  - [Word Ladder](https://leetcode.com/problems/word-ladder/) (NeetCode - Hard)
  - Thinking prompts: Bi-directional BFS or standard BFS level tracking.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int LadderLength(string beginWord, string endWord, IList<string> wordList) {
        HashSet<string> wordSet = new(wordList);
        if (!wordSet.Contains(endWord)) return 0;

        Queue<string> queue = new();
        queue.Enqueue(beginWord);
        int step = 1;

        while (queue.Count > 0) {
            int size = queue.Count;
            step++;

            for (int i = 0; i < size; i++) {
                char[] curr = queue.Dequeue().ToCharArray();
                for (int j = 0; j < curr.Length; j++) {
                    char original = curr[j];
                    for (char c = 'a'; c <= 'z'; c++) {
                        if (c == original) continue;
                        curr[j] = c;
                        string next = new string(curr);

                        if (next == endWord) return step;
                        if (wordSet.Remove(next)) {
                            queue.Enqueue(next);
                        }
                    }
                    curr[j] = original;
                }
            }
        }
        return 0;
    }
}
```

<b>Complexity:</b> Time: $O(M^2 \cdot N)$ where $M$ is word length and $N$ is word list count. Space: $O(M \cdot N)$.

</details>

---

### Day 11: CSES & Competitive Benchmarks + Spaced Retrieval
- **Benchmark 1**: [CSES: Counting Rooms](https://cses.fi/problemset/task/1192) (Number of Islands on massive $1000 \times 1000$ grid)
- **Benchmark 2**: [CSES: Labyrinth](https://cses.fi/problemset/task/1193) (BFS shortest path with predecessor tracking)
- **Benchmark 3**: [CSES: Building Roads](https://cses.fi/problemset/task/1666) (Finding connected components and connecting bridges)
- **Benchmark 4**: [CSES: Message Route](https://cses.fi/problemset/task/1667) (Unweighted shortest path in communication network)
- **Benchmark 5**: [Codeforces 520B - Two Buttons](https://codeforces.com/problemset/problem/520/B) (BFS on numerical operations)

---

### Spaced Retrieval Flashcards

<details markdown="1">
<summary><b>Flashcard 1: When must you use BFS instead of DFS?</b></summary>

When searching for the <b>shortest path</b> in an unweighted graph or state space. BFS explores in concentric waves of equal distance, ensuring the first time the target is reached, it is via the minimum number of edges.
</details>

<details markdown="1">
<summary><b>Flashcard 2: How do you prevent stack overflow during large grid DFS in C#?</b></summary>

By replacing recursion with BFS using an explicit <code>Queue&lt;(int, int)&gt;</code>, or using an explicit <code>Stack&lt;(int, int)&gt;</code> allocated on the heap rather than consuming the thread's 1MB call stack.
</details>

<details markdown="1">
<summary><b>Flashcard 3: What is the purpose of the 3-color state in directed graph cycle detection?</b></summary>

State 0 (white) = unvisited; State 1 (grey) = currently on the active recursion stack; State 2 (black) = fully processed and confirmed acyclic. A cycle exists if and only if an edge points to a State 1 node (a back-edge).
</details>

---

## 🎯 Topic Checklist (15 Problems)

| Day | # | Problem | Platform | Difficulty | Status |
|:---:|:---:|:---|:---:|:---:|:---:|
| Day 1 | 1 | [Number of Islands](https://leetcode.com/problems/number-of-islands/) | NeetCode | Medium | - [ ] |
| Day 2 | 2 | [Max Area of Island](https://leetcode.com/problems/max-area-of-island/) | NeetCode | Medium | - [ ] |
| Day 3 | 3 | [Clone Graph](https://leetcode.com/problems/clone-graph/) | NeetCode | Medium | - [ ] |
| Day 4 | 4 | [Pacific Atlantic Water Flow](https://leetcode.com/problems/pacific-atlantic-water-flow/) | NeetCode | Medium | - [ ] |
| Day 5 | 5 | [Surrounded Regions](https://leetcode.com/problems/surrounded-regions/) | NeetCode | Medium | - [ ] |
| Day 6 | 6 | [Rotting Oranges](https://leetcode.com/problems/rotting-oranges/) | NeetCode | Medium | - [ ] |
| Day 7 | 7 | [Walls and Gates](https://leetcode.com/problems/walls-and-gates/) | NeetCode | Medium | - [ ] |
| Day 8 | 8 | [Course Schedule](https://leetcode.com/problems/course-schedule/) | NeetCode | Medium | - [ ] |
| Day 9 | 9 | [Open the Lock](https://leetcode.com/problems/open-the-lock/) | LeetCode | Medium | - [ ] |
| Day 10 | 10 | [Word Ladder](https://leetcode.com/problems/word-ladder/) | NeetCode | Hard | - [ ] |
| Day 11 | 11 | [Counting Rooms](https://cses.fi/problemset/task/1192) | CSES | Easy | - [ ] |
| Day 11 | 12 | [Labyrinth](https://cses.fi/problemset/task/1193) | CSES | Medium | - [ ] |
| Day 11 | 13 | [Building Roads](https://cses.fi/problemset/task/1666) | CSES | Medium | - [ ] |
| Day 11 | 14 | [Message Route](https://cses.fi/problemset/task/1667) | CSES | Medium | - [ ] |
| Day 11 | 15 | [Two Buttons (520B)](https://codeforces.com/problemset/problem/520/B) | Codeforces | Rating 1000 | - [ ] |

---

[⬅️ Previous Topic: Backtracking](09-backtracking.html) | [Home](index.html) | [Next Topic: Graphs II: Topological Sort, Dijkstra & Union-Find ➡️](11-graphs-advanced.html)
