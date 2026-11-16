---
layout: default
title: "Backtracking & Recursion"
---

# Backtracking & Recursion

> **Estimated Time**: 11 Days • **Pace**: 1.5–2 hours/day (Strictly 1 problem/day) • **Language**: C# (.NET 6+)

---

## 🛠️ C# Mental Model & Gotchas

### 1. The Three Steps of Backtracking
Every backtracking algorithm is an explicit depth-first search over a state decision tree:
1. **Choose**: Add choice to current candidate path / mutate state.
2. **Explore**: Recurse down the decision tree to explore all consequences.
3. **Unchoose (Backtrack)**: Undo the mutation / remove choice (`path.RemoveAt(path.Count - 1)`) so sibling branches receive a pristine state.

### 2. Deep Copying Result Lists in C#
A ubiquitous beginner bug in C# backtracking:
```csharp
// ❌ WRONG: Saves the reference to the mutable working list!
result.Add(currentPath);

// ✅ CORRECT: Takes an immutable snapshot of the current state
result.Add(new List<int>(currentPath));
```
Because `currentPath` is an object reference on the heap, failing to clone it results in an output list filled with empty lists once backtracking unwinds!

### 3. Deduplication Invariant
When duplicate elements exist in the input array (e.g. `Subsets II`, `Combination Sum II`):
1. Sort the input array first: `Array.Sort(nums)`.
2. Skip sibling branches that choose the same value at the current tree depth:
```csharp
if (i > startIdx && nums[i] == nums[i - 1]) continue;
```

---

## 📅 Day-by-Day Schedule

### Day 1: Theory Anchor & Baseline Pattern
- **Step 1 (45 mins) | Concept Video**: [Backtracking Algorithm & State Space Trees](https://www.youtube.com/watch?v=xFv_Hl4B83A) by *Abdul Bari*
  - Key focus: What is the state space tree? How does depth-first tree traversal explore candidate configurations?
- **Step 2 (20 mins) | Pen & Paper Invariant**:
  - Draw the binary decision tree for `nums = [1, 2, 3]` (include vs exclude choice at each level).
- **Step 3 (45 mins) | Problem 1 (Warm-Up)**:
  - [Subsets](https://leetcode.com/problems/subsets/) (NeetCode - Medium)
  - Objective: Classic inclusion/exclusion recursion branching.
- **Step 4 (10 mins) | Active Recall**:
  <details>
  <summary><b>🧠 Self-Test: How many total subsets exist for a set of size $N$, and what is the time complexity?</b></summary>
  <br>
  <b>Answer:</b> There are exactly $2^N$ subsets. Since copying each subset of average length $N/2$ into the result list takes $O(N)$ time, the total time complexity is $O(N \cdot 2^N)$.
  </details>

---

### Day 2: Unbounded Knapsack-Style Choice (Combination Sum)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: In Combination Sum, how do we allow an element to be reused unlimited times without causing infinite recursion loops?</b></summary>
  <br>
  <b>Answer:</b> Pass the same index $i$ in the recursive call <code>Backtrack(i, remaining - candidates[i])</code> rather than advancing to $i + 1$. Recursion terminates when <code>remaining &lt;= 0</code>.
  </details>
- **Deep Practice (75–85 mins) | Problem 2**:
  - [Combination Sum](https://leetcode.com/problems/combination-sum/) (NeetCode - Medium)
  - Thinking prompts: Prune early if `candidates[i] > remaining` after sorting.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public IList<IList<int>> CombinationSum(int[] candidates, int target) {
          Array.Sort(candidates);
          List<IList<int>> result = new();
          List<int> current = new();
          Dfs(0, target, candidates, current, result);
          return result;
      }
      
      private void Dfs(int start, int target, int[] candidates, List<int> current, List<IList<int>> result) {
          if (target == 0) {
              result.Add(new List<int>(current));
              return;
          }
          
          for (int i = start; i < candidates.Length; i++) {
              if (candidates[i] > target) break; // Prune remaining larger numbers
              
              current.Add(candidates[i]);
              Dfs(i, target - candidates[i], candidates, current, result); // Reuse index i
              current.RemoveAt(current.Count - 1); // Backtrack
          }
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N^{\frac{T}{M}})$ where $T$ is target and $M$ is minimal candidate value. Space: $O(\frac{T}{M})$.
  </details>

---

### Day 3: Ordered Placement Invariant (Permutations)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: What is the difference in branching between Combinations and Permutations?</b></summary>
  <br>
  <b>Answer:</b> Combinations restrict candidate choices to indices $\ge start$ to avoid duplicate orderings. Permutations can choose any element from index $0$ to $N - 1$, using a <code>bool[] used</code> array or in-place swapping to avoid reusing the same element.
  </details>
- **Deep Practice (75–85 mins) | Problem 3**:
  - [Permutations](https://leetcode.com/problems/permutations/) (NeetCode - Medium)
  - Thinking prompts: Compare `bool[] used` vs in-place swapping technique.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public IList<IList<int>> Permute(int[] nums) {
          List<IList<int>> result = new();
          List<int> current = new();
          bool[] used = new bool[nums.Length];
          Dfs(nums, used, current, result);
          return result;
      }
      
      private void Dfs(int[] nums, bool[] used, List<int> current, List<IList<int>> result) {
          if (current.Count == nums.Length) {
              result.Add(new List<int>(current));
              return;
          }
          
          for (int i = 0; i < nums.Length; i++) {
              if (used[i]) continue;
              
              used[i] = true;
              current.Add(nums[i]);
              Dfs(nums, used, current, result);
              current.RemoveAt(current.Count - 1);
              used[i] = false;
          }
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N \cdot N!)$, Space: $O(N)$ recursion depth.
  </details>

---

### Day 4: Horizontal Sibling Deduplication (Subsets II)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: Why is sorting essential before running backtracking on inputs containing duplicates?</b></summary>
  <br>
  <b>Answer:</b> Sorting groups identical numbers adjacent to each other. This enables the sibling check <code>i &gt; start &amp;&amp; nums[i] == nums[i - 1]</code> to cleanly skip duplicate branches.
  </details>
- **Deep Practice (75–85 mins) | Problem 4**:
  - [Subsets II](https://leetcode.com/problems/subsets-ii/) (NeetCode - Medium)
  - Thinking prompts: Trace `nums = [1, 2, 2]`. Distinguish between vertical recursion depth (reusing identical values across levels) vs horizontal iteration (skipping duplicate values at the same level).
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public IList<IList<int>> SubsetsWithDup(int[] nums) {
          Array.Sort(nums);
          List<IList<int>> result = new();
          List<int> current = new();
          Dfs(0, nums, current, result);
          return result;
      }
      
      private void Dfs(int start, int[] nums, List<int> current, List<IList<int>> result) {
          result.Add(new List<int>(current));
          
          for (int i = start; i < nums.Length; i++) {
              if (i > start && nums[i] == nums[i - 1]) continue; // Skip horizontal duplicate
              
              current.Add(nums[i]);
              Dfs(i + 1, nums, current, result);
              current.RemoveAt(current.Count - 1);
          }
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N \cdot 2^N)$, Space: $O(N)$.
  </details>

---

### Day 5: Bounded Combination Sum with Duplicates (Combination Sum II)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: What are the two key differences between Combination Sum I and Combination Sum II?</b></summary>
  <br>
  <b>Answer:</b> In Combination Sum II, each number may only be used once (recurse with <code>i + 1</code> instead of <code>i</code>), and the input array contains duplicates, requiring horizontal sibling skipping.
  </details>
- **Deep Practice (75–85 mins) | Problem 5**:
  - [Combination Sum II](https://leetcode.com/problems/combination-sum-ii/) (NeetCode - Medium)
  - Thinking prompts: Combine sorting, `i > start && candidates[i] == candidates[i - 1]` skipping, and `i + 1` recursion.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public IList<IList<int>> CombinationSum2(int[] candidates, int target) {
          Array.Sort(candidates);
          List<IList<int>> result = new();
          List<int> current = new();
          Dfs(0, target, candidates, current, result);
          return result;
      }
      
      private void Dfs(int start, int target, int[] candidates, List<int> current, List<IList<int>> result) {
          if (target == 0) {
              result.Add(new List<int>(current));
              return;
          }
          
          for (int i = start; i < candidates.Length; i++) {
              if (candidates[i] > target) break;
              if (i > start && candidates[i] == candidates[i - 1]) continue;
              
              current.Add(candidates[i]);
              Dfs(i + 1, target - candidates[i], candidates, current, result);
              current.RemoveAt(current.Count - 1);
          }
      }
  }
  ```
  <b>Complexity:</b> Time: $O(2^N)$, Space: $O(N)$.
  </details>

---

### Day 6: 2D Grid In-Place Backtracking (Word Search)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: How do you mark a grid cell $(r, c)$ as visited without allocating an extra 2D boolean array?</b></summary>
  <br>
  <b>Answer:</b> Mutate the cell in-place to a sentinel character (e.g. <code>board[r][c] = '#'</code>) before recursing, and restore its original character <code>board[r][c] = originalChar</code> when backtracking.
  </details>
- **Deep Practice (75–85 mins) | Problem 6**:
  - [Word Search](https://leetcode.com/problems/word-search/) (NeetCode - Medium)
  - Thinking prompts: Explore 4 cardinal directions. Return `true` immediately upon finding full word to prune further exploration.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public bool Exist(char[][] board, string word) {
          int m = board.Length;
          int n = board[0].Length;
          
          for (int r = 0; r < m; r++) {
              for (int c = 0; c < n; c++) {
                  if (Dfs(r, c, 0, board, word)) return true;
              }
          }
          return false;
      }
      
      private bool Dfs(int r, int c, int idx, char[][] board, string word) {
          if (idx == word.Length) return true;
          if (r < 0 || r >= board.Length || c < 0 || c >= board[0].Length || board[r][c] != word[idx]) {
              return false;
          }
          
          char temp = board[r][c];
          board[r][c] = '#'; // Mark visited
          
          bool found = Dfs(r + 1, c, idx + 1, board, word) ||
                       Dfs(r - 1, c, idx + 1, board, word) ||
                       Dfs(r, c + 1, idx + 1, board, word) ||
                       Dfs(r, c - 1, idx + 1, board, word);
                       
          board[r][c] = temp; // Unmark / backtrack
          return found;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(M \cdot N \cdot 3^L)$ where $L$ is word length, Space: $O(L)$ recursion stack.
  </details>

---

### Day 7: String Partitioning Invariants (Palindrome Partitioning)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: In Palindrome Partitioning, what condition must hold before making a recursive call on substring $[start \dots i]$?</b></summary>
  <br>
  <b>Answer:</b> The substring $s[start \dots i]$ must be a valid palindrome. If not, this cut is discarded immediately without branching further.
  </details>
- **Deep Practice (75–85 mins) | Problem 7**:
  - [Palindrome Partitioning](https://leetcode.com/problems/palindrome-partitioning/) (NeetCode - Medium)
  - Thinking prompts: Slice string at each index $i \ge start$. If substring is a palindrome, add to `current` and recurse on `i + 1`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public IList<IList<string>> Partition(string s) {
          List<IList<string>> result = new();
          List<string> current = new();
          Dfs(0, s, current, result);
          return result;
      }
      
      private void Dfs(int start, string s, List<string> current, List<IList<string>> result) {
          if (start == s.Length) {
              result.Add(new List<string>(current));
              return;
          }
          
          for (int i = start; i < s.Length; i++) {
              if (IsPalindrome(s, start, i)) {
                  current.Add(s.Substring(start, i - start + 1));
                  Dfs(i + 1, s, current, result);
                  current.RemoveAt(current.Count - 1);
              }
          }
      }
      
      private bool IsPalindrome(string s, int l, int r) {
          while (l < r) {
              if (s[l++] != s[r--]) return false;
          }
          return true;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N \cdot 2^N)$, Space: $O(N)$.
  </details>

---

### Day 8: Fixed Digit String Mapping (Letter Combinations)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: What is the branching factor for digits '7' and '9' vs other digits in phone keypad mapping?</b></summary>
  <br>
  <b>Answer:</b> Digits '7' (pqrs) and '9' (wxyz) have a branching factor of 4, while other digits have 3.
  </details>
- **Deep Practice (75–85 mins) | Problem 8**:
  - [Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number/) (NeetCode - Medium)
  - Thinking prompts: Base case `if (digits.Length == 0) return empty`. Maintain a `char[] path` of size `digits.Length` to avoid string concatenation overhead.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      private static readonly string[] Pad = {
          "", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"
      };

      public IList<string> LetterCombinations(string digits) {
          List<string> result = new();
          if (digits.Length == 0) return result;
          
          char[] path = new char[digits.Length];
          Dfs(0, digits, path, result);
          return result;
      }
      
      private void Dfs(int idx, string digits, char[] path, List<string> result) {
          if (idx == digits.Length) {
              result.Add(new string(path));
              return;
          }
          
          string letters = Pad[digits[idx] - '0'];
          foreach (char c in letters) {
              path[idx] = c;
              Dfs(idx + 1, digits, path, result);
          }
      }
  }
  ```
  <b>Complexity:</b> Time: $O(4^N)$, Space: $O(N)$.
  </details>

---

### Day 9: Constraint Propagation & Bitmasks (N-Queens)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: How are positive and negative diagonals indexed in an $N \times N$ chessboard?</b></summary>
  <br>
  <b>Answer:</b> Positive diagonal (top-left to bottom-right): $r - c$ (or $r - c + N - 1$ to map to positive indices). Negative diagonal (top-right to bottom-left): $r + c$.
  </details>
- **Deep Practice (75–85 mins) | Problem 9**:
  - [N-Queens](https://leetcode.com/problems/n-queens/) (NeetCode - Hard)
  - Thinking prompts: Place one queen per row. Track attacked columns, positive diagonals ($r + c$), and negative diagonals ($r - c$) using `HashSet<int>` or integer bitmasks.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public IList<IList<string>> SolveNQueens(int n) {
          List<IList<string>> result = new();
          char[][] board = new char[n][];
          for (int i = 0; i < n; i++) {
              board[i] = new char[n];
              Array.Fill(board[i], '.');
          }
          
          HashSet<int> cols = new();
          HashSet<int> posDiag = new(); // r + c
          HashSet<int> negDiag = new(); // r - c
          
          Dfs(0, n, board, cols, posDiag, negDiag, result);
          return result;
      }
      
      private void Dfs(int r, int n, char[][] board, HashSet<int> cols, HashSet<int> posDiag, HashSet<int> negDiag, List<IList<string>> result) {
          if (r == n) {
              List<string> solution = new(n);
              for (int i = 0; i < n; i++) solution.Add(new string(board[i]));
              result.Add(solution);
              return;
          }
          
          for (int c = 0; c < n; c++) {
              if (cols.Contains(c) || posDiag.Contains(r + c) || negDiag.Contains(r - c)) {
                  continue;
              }
              
              board[r][c] = 'Q';
              cols.Add(c);
              posDiag.Add(r + c);
              negDiag.Add(r - c);
              
              Dfs(r + 1, n, board, cols, posDiag, negDiag, result);
              
              // Backtrack
              board[r][c] = '.';
              cols.Remove(c);
              posDiag.Remove(r + c);
              negDiag.Remove(r - c);
          }
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N!)$, Space: $O(N^2)$ for board.
  </details>

---

### Day 10: State Space Pruning (Sudoku Solver)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: In Sudoku Solver, how does returning a boolean from <code>Solve()</code> prune dead ends?</b></summary>
  <br>
  <b>Answer:</b> If a candidate number leads to a dead end (no valid number for a subsequent cell), <code>Solve()</code> returns <code>false</code>, which triggers resetting <code>board[r][c] = '.'</code> and attempting the next digit. If a full solution is reached, it returns <code>true</code> and halts all recursion.
  </details>
- **Deep Practice (75–85 mins) | Problem 10**:
  - [Sudoku Solver](https://leetcode.com/problems/sudoku-solver/) (NeetCode - Hard)
  - Thinking prompts: Search for empty cell `'.'`. Attempt digits `'1'` through `'9'`. Validate row, column, and $3 \times 3$ box.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public void SolveSudoku(char[][] board) {
          Solve(board);
      }
      
      private bool Solve(char[][] board) {
          for (int r = 0; r < 9; r++) {
              for (int c = 0; c < 9; c++) {
                  if (board[r][c] == '.') {
                      for (char d = '1'; d <= '9'; d++) {
                          if (IsValid(board, r, c, d)) {
                              board[r][c] = d;
                              if (Solve(board)) return true;
                              board[r][c] = '.'; // Backtrack
                          }
                      }
                      return false; // No digit fits
                  }
              }
          }
          return true; // All cells filled
      }
      
      private bool IsValid(char[][] board, int r, int c, char d) {
          int boxRow = 3 * (r / 3);
          int boxCol = 3 * (c / 3);
          
          for (int i = 0; i < 9; i++) {
              if (board[r][i] == d) return false;
              if (board[i][c] == d) return false;
              if (board[boxRow + i / 3][boxCol + i % 3] == d) return false;
          }
          return true;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(9^{81})$ worst-case (drastically pruned in practice), Space: $O(81)$ recursion depth.
  </details>

---

### Day 11: CSES & Competitive Benchmarks + Spaced Retrieval
- **Benchmark 1**: [CSES: Apple Division](https://cses.fi/problemset/task/1623) (Partition $N$ weights into two sets minimizing weight difference)
- **Benchmark 2**: [CSES: Creating Strings](https://cses.fi/problemset/task/1622) (Unique string permutations)
- **Benchmark 3**: [CSES: Chessboard and Queens](https://cses.fi/problemset/task/1624) (8-Queens with reserved blocked squares)
- **Benchmark 4**: [Codeforces 1097B - Petr and a Combination Lock](https://codeforces.com/problemset/problem/1097/B) (Bitmask subset sum modulo 360)
- **Benchmark 5**: [Codeforces 1352C - K-th Not Divisible by n](https://codeforces.com/problemset/problem/1352/C) (Mathematical search)

---

### Spaced Retrieval Flashcards

<details>
<summary><b>Flashcard 1: When must you clone a collection when adding it to the final result in C#?</b></summary>
<br>
Always when backtracking with a shared mutable list (e.g. <code>result.Add(new List&lt;int&gt;(current))</code>). If you do not clone it, every entry in <code>result</code> will point to the exact same list instance, which ends up empty after unwinding.
</details>

<details>
<summary><b>Flashcard 2: How do you differentiate between horizontal pruning and vertical branching?</b></summary>
<br>
Vertical branching is the recursive call depth (moving from index $i$ to $i+1$). Horizontal pruning is the <code>for</code> loop iteration at the current depth; duplicate checks like <code>if (i &gt; start &amp;&amp; nums[i] == nums[i-1]) continue;</code> prune horizontal duplicates while allowing identical values to be picked vertically.
</details>

<details>
<summary><b>Flashcard 3: How does in-place array mutation simplify memory usage in grid backtracking?</b></summary>
<br>
Mutating a grid cell directly (e.g. setting to <code>'#'</code> and restoring upon return) eliminates the need to allocate an $M \times N$ boolean visited matrix, cutting heap allocations to zero.
</details>

---

## 🎯 Topic Checklist (15 Problems)

| Day | # | Problem | Platform | Difficulty | Status |
|:---:|:---:|:---|:---:|:---:|:---:|
| Day 1 | 1 | [Subsets](https://leetcode.com/problems/subsets/) | NeetCode | Medium | - [ ] |
| Day 2 | 2 | [Combination Sum](https://leetcode.com/problems/combination-sum/) | NeetCode | Medium | - [ ] |
| Day 3 | 3 | [Permutations](https://leetcode.com/problems/permutations/) | NeetCode | Medium | - [ ] |
| Day 4 | 4 | [Subsets II](https://leetcode.com/problems/subsets-ii/) | NeetCode | Medium | - [ ] |
| Day 5 | 5 | [Combination Sum II](https://leetcode.com/problems/combination-sum-ii/) | NeetCode | Medium | - [ ] |
| Day 6 | 6 | [Word Search](https://leetcode.com/problems/word-search/) | NeetCode | Medium | - [ ] |
| Day 7 | 7 | [Palindrome Partitioning](https://leetcode.com/problems/palindrome-partitioning/) | NeetCode | Medium | - [ ] |
| Day 8 | 8 | [Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number/) | NeetCode | Medium | - [ ] |
| Day 9 | 9 | [N-Queens](https://leetcode.com/problems/n-queens/) | NeetCode | Hard | - [ ] |
| Day 10 | 10 | [Sudoku Solver](https://leetcode.com/problems/sudoku-solver/) | NeetCode | Hard | - [ ] |
| Day 11 | 11 | [Apple Division](https://cses.fi/problemset/task/1623) | CSES | Easy | - [ ] |
| Day 11 | 12 | [Creating Strings](https://cses.fi/problemset/task/1622) | CSES | Easy | - [ ] |
| Day 11 | 13 | [Chessboard and Queens](https://cses.fi/problemset/task/1624) | CSES | Medium | - [ ] |
| Day 11 | 14 | [Petr and a Combination Lock (1097B)](https://codeforces.com/problemset/problem/1097/B) | Codeforces | Rating 1000 | - [ ] |
| Day 11 | 15 | [K-th Not Divisible by n (1352C)](https://codeforces.com/problemset/problem/1352/C) | Codeforces | Rating 1200 | - [ ] |

---

[⬅️ Previous Topic: Heaps & Priority Queues](08-heaps.html) | [Home](index.html) | [Next Topic: Graphs I: BFS, DFS & Grid Traversals ➡️](10-graphs-traversal.html)

