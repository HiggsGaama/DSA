---
layout: default
title: "Binary Trees & BSTs"
---

# Binary Trees & BSTs

> **Estimated Time**: 12 Days • **Pace**: 1.5–2 hours/day (Strictly 1 problem/day) • **Language**: C# (.NET 6+)

---

## 🛠️ C# Mental Model & Gotchas

### 1. The Two Fundamental Tree Mental Models
Every tree problem reduces to one of two structural mindsets:
- **Bottom-Up Induction (Postorder / Divide & Conquer)**: Ask: *"If my left and right subtrees give me their answers (e.g. depth, diameter, validity), how do I combine them at the current node?"*
- **Top-Down State Passing (Preorder / DFS)**: Pass information down from parents to children (e.g. current path sum, maximum ancestor value, or valid BST range $(min, max)$).

### 2. Level-Order BFS Invariant
In C#, when running BFS level-order traversal, snapshot `int levelSize = queue.Count` **before** the inner loop starts. This ensures the loop processes exactly one horizontal tree level at a time without mixing in newly enqueued child nodes.

### 3. BST Range Invariant
Never validate a Binary Search Tree by only comparing a node to its immediate left and right children! A node in the right subtree of the left child could violate the root's value. Always propagate bounds:
```csharp
bool Validate(TreeNode node, long min, long max)
```
Using `long` prevents overflow when tree node values equal `int.MinValue` or `int.MaxValue`.

---

## 📅 Day-by-Day Schedule

### Day 1: Theory Anchor & Baseline Pattern
- **Step 1 (45 mins) | Concept Video**: [Binary Tree Traversal Techniques](https://www.youtube.com/watch?v=gm8DUJJhmY4) by *Abdul Bari*
  - Key focus: Preorder, Inorder, and Postorder recursive traversal mechanics on the call stack.
- **Step 2 (20 mins) | Pen & Paper Invariant**:
  - Draw a 3-level binary tree. Write the trace of recursive child swapping for Invert Binary Tree.
- **Step 3 (45 mins) | Problem 1 (Warm-Up)**:
  - [Invert Binary Tree](https://leetcode.com/problems/invert-binary-tree/) (NeetCode - Easy)
  - Objective: Invert left and right subtrees recursively.
- **Step 4 (10 mins) | Active Recall**:
  <details>
  <summary><b>🧠 Self-Test: What is the base case of almost every binary tree recursion?</b></summary>
  <br>
  <b>Answer:</b> <code>if (root == null) return ...;</code>. When hitting a null pointer, immediately return the neutral identity element (e.g. 0 for height/count, null for nodes, true for boolean checks).
  </details>

---

### Day 2: Postorder Height Induction (Maximum Depth & Diameter)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: How is the diameter of a binary tree related to the heights of its subtrees?</b></summary>
  <br>
  <b>Answer:</b> The longest path passing through any node $u$ as its highest turning point is $\text{height}(u.left) + \text{height}(u.right)$.
  </details>
- **Deep Practice (75–85 mins) | Problem 2**:
  - [Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree/) (NeetCode - Easy)
  - Thinking prompts: Write a helper function that returns the height of the current node, while updating a global or reference variable `maxDiameter` with `leftHeight + rightHeight`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      private int _maxDiameter = 0;

      public int DiameterOfBinaryTree(TreeNode root) {
          _maxDiameter = 0;
          Height(root);
          return _maxDiameter;
      }
      
      private int Height(TreeNode node) {
          if (node == null) return 0;
          
          int left = Height(node.left);
          int right = Height(node.right);
          
          _maxDiameter = Math.Max(_maxDiameter, left + right);
          return 1 + Math.Max(left, right);
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N)$ single pass, Space: $O(H)$ recursion stack where $H$ is tree height.
  </details>

---

### Day 3: Balanced Tree Postorder Pruning
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: Why is checking <code>Math.Abs(Height(left) - Height(right)) &lt;= 1</code> naively at every node $O(N^2)$?</b></summary>
  <br>
  <b>Answer:</b> Because calculating heights repeatedly for each node recalculates subtree depths over and over. By returning $-1$ immediately from the bottom up as soon as any imbalance is detected, we prune in $O(N)$ time.
  </details>
- **Deep Practice (75–85 mins) | Problem 3**:
  - [Balanced Binary Tree](https://leetcode.com/problems/balanced-binary-tree/) (NeetCode - Easy)
  - Thinking prompts: Return $-1$ from helper when subtree is unbalanced.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public bool IsBalanced(TreeNode root) {
          return CheckHeight(root) != -1;
      }
      
      private int CheckHeight(TreeNode node) {
          if (node == null) return 0;
          
          int left = CheckHeight(node.left);
          if (left == -1) return -1;
          
          int right = CheckHeight(node.right);
          if (right == -1) return -1;
          
          if (Math.Abs(left - right) > 1) return -1;
          return 1 + Math.Max(left, right);
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N)$, Space: $O(H)$.
  </details>

---

### Day 4: Dual-Tree Structural Comparison (Same Tree & Subtree)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: What three conditions confirm that two binary trees $p$ and $q$ are identical?</b></summary>
  <br>
  <b>Answer:</b> 1. Both are null (true); 2. Exactly one is null (false); 3. <code>p.val == q.val &amp;&amp; IsSameTree(p.left, q.left) &amp;&amp; IsSameTree(p.right, q.right)</code>.
  </details>
- **Deep Practice (75–85 mins) | Problem 4**:
  - [Subtree of Another Tree](https://leetcode.com/problems/subtree-of-another-tree/) (NeetCode - Easy)
  - Thinking prompts: A tree $S$ contains $T$ if `IsSameTree(S, T)` or `HasSubtree(S.left, T)` or `HasSubtree(S.right, T)`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public bool IsSubtree(TreeNode root, TreeNode subRoot) {
          if (root == null) return false;
          if (IsSame(root, subRoot)) return true;
          return IsSubtree(root.left, subRoot) || IsSubtree(root.right, subRoot);
      }
      
      private bool IsSame(TreeNode s, TreeNode t) {
          if (s == null && t == null) return true;
          if (s == null || t == null || s.val != t.val) return false;
          return IsSame(s.left, t.left) && IsSame(s.right, t.right);
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N \cdot M)$ worst-case, Space: $O(H)$.
  </details>

---

### Day 5: BST Invariant Exploitation (LCA of BST)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: In a Binary Search Tree, how do we know whether the Lowest Common Ancestor of $P$ and $Q$ is the current node?</b></summary>
  <br>
  <b>Answer:</b> If $P$ and $Q$ split across the current node (one is smaller and one is greater) or if the current node equals $P$ or $Q$, the current node is their Lowest Common Ancestor.
  </details>
- **Deep Practice (75–85 mins) | Problem 5**:
  - [Lowest Common Ancestor of a Binary Search Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/) (NeetCode - Medium)
  - Thinking prompts: If both $P$ and $Q$ are smaller than `curr.val`, move `curr = curr.left`. If both are greater, move `curr = curr.right`. Otherwise, you have found the LCA!
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public TreeNode LowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
          TreeNode curr = root;
          while (curr != null) {
              if (p.val < curr.val && q.val < curr.val) {
                  curr = curr.left;
              } else if (p.val > curr.val && q.val > curr.val) {
                  curr = curr.right;
              } else {
                  return curr; // Split point or match
              }
          }
          return null;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(H)$, Space: $O(1)$ iterative.
  </details>

---

### Day 6: BFS Level-Order Traversal Invariant
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: How do you isolate levels in a BFS queue without maintaining separate queues for each depth?</b></summary>
  <br>
  <b>Answer:</b> Store <code>int count = queue.Count</code> at the start of the outer loop. Iterate exactly <code>count</code> times to dequeue all nodes belonging to the current level.
  </details>
- **Deep Practice (75–85 mins) | Problem 6**:
  - [Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/) (NeetCode - Medium)
  - Thinking prompts: Standard BFS with `Queue<TreeNode>`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public IList<IList<int>> LevelOrder(TreeNode root) {
          List<IList<int>> result = new();
          if (root == null) return result;
          
          Queue<TreeNode> queue = new();
          queue.Enqueue(root);
          
          while (queue.Count > 0) {
              int levelSize = queue.Count;
              List<int> currentLevel = new(levelSize);
              
              for (int i = 0; i < levelSize; i++) {
                  TreeNode node = queue.Dequeue();
                  currentLevel.Add(node.val);
                  
                  if (node.left != null) queue.Enqueue(node.left);
                  if (node.right != null) queue.Enqueue(node.right);
              }
              result.Add(currentLevel);
          }
          return result;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N)$, Space: $O(N)$ (at most $N/2$ leaf nodes in queue).
  </details>

---

### Day 7: Level-Order Boundary Observation (Right Side View)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: Which node in a BFS level represents the right-side view?</b></summary>
  <br>
  <b>Answer:</b> The very last node processed in that level's <code>for (int i = 0; i &lt; levelSize; i++)</code> loop (i.e. when <code>i == levelSize - 1</code>).
  </details>
- **Deep Practice (75–85 mins) | Problem 7**:
  - [Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view/) (NeetCode - Medium)
  - Thinking prompts: Compare BFS level capture vs DFS visiting `right` child before `left`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public IList<int> RightSideView(TreeNode root) {
          List<int> result = new();
          if (root == null) return result;
          
          Queue<TreeNode> queue = new();
          queue.Enqueue(root);
          
          while (queue.Count > 0) {
              int count = queue.Count;
              for (int i = 0; i < count; i++) {
                  TreeNode node = queue.Dequeue();
                  if (i == count - 1) {
                      result.Add(node.val);
                  }
                  if (node.left != null) queue.Enqueue(node.left);
                  if (node.right != null) queue.Enqueue(node.right);
              }
          }
          return result;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N)$, Space: $O(N)$.
  </details>

---

### Day 8: Path Parameter Accumulation (Count Good Nodes)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: In Count Good Nodes in Binary Tree, what state must be passed down to each recursive call?</b></summary>
  <br>
  <b>Answer:</b> The maximum value encountered along the path from the root to the current node: <code>Math.Max(maxSoFar, node.val)</code>.
  </details>
- **Deep Practice (75–85 mins) | Problem 8**:
  - [Count Good Nodes in Binary Tree](https://leetcode.com/problems/count-good-nodes-in-binary-tree/) (NeetCode - Medium)
  - Thinking prompts: Top-down recursion propagating `maxVal`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public int GoodNodes(TreeNode root) {
          return Dfs(root, root.val);
      }
      
      private int Dfs(TreeNode node, int maxSoFar) {
          if (node == null) return 0;
          
          int count = (node.val >= maxSoFar) ? 1 : 0;
          int newMax = Math.Max(maxSoFar, node.val);
          
          count += Dfs(node.left, newMax);
          count += Dfs(node.right, newMax);
          return count;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N)$, Space: $O(H)$.
  </details>

---

### Day 9: Open-Interval Invariant Validation (Validate BST)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: Why can validating BST fail if using <code>int.MinValue</code> and <code>int.MaxValue</code> as bounds?</b></summary>
  <br>
  <b>Answer:</b> If a valid tree node has value equal to <code>int.MinValue</code> or <code>int.MaxValue</code>, strict inequality <code>val &gt; min &amp;&amp; val &lt; max</code> will incorrectly reject it. Using <code>long.MinValue</code> and <code>long.MaxValue</code> provides headroom.
  </details>
- **Deep Practice (75–85 mins) | Problem 9**:
  - [Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/) (NeetCode - Medium)
  - Thinking prompts: Validate node with open interval $(lower, upper)$. When moving left, upper bound becomes `node.val`. When moving right, lower bound becomes `node.val`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public bool IsValidBST(TreeNode root) {
          return Validate(root, long.MinValue, long.MaxValue);
      }
      
      private bool Validate(TreeNode node, long min, long max) {
          if (node == null) return true;
          if (node.val <= min || node.val >= max) return false;
          
          return Validate(node.left, min, node.val) &&
                 Validate(node.right, node.val, max);
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N)$, Space: $O(H)$.
  </details>

---

### Day 10: Inorder Traversal Monotonicity (Kth Smallest in BST)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: What order of traversal visits BST nodes in strictly sorted ascending order?</b></summary>
  <br>
  <b>Answer:</b> Inorder traversal (Left $\to$ Root $\to$ Right).
  </details>
- **Deep Practice (75–85 mins) | Problem 10**:
  - [Kth Smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/) (NeetCode - Medium)
  - Thinking prompts: Iterative inorder traversal with `Stack<TreeNode>`. Stop when counter reaches $K$.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public int KthSmallest(TreeNode root, int k) {
          Stack<TreeNode> stack = new();
          TreeNode curr = root;
          
          while (curr != null || stack.Count > 0) {
              while (curr != null) {
                  stack.Push(curr);
                  curr = curr.left;
              }
              
              curr = stack.Pop();
              k--;
              if (k == 0) return curr.val;
              
              curr = curr.right;
          }
          return -1;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(H + K)$, Space: $O(H)$.
  </details>

---

### Day 11: Tree Reconstruction via Hash Maps (Preorder & Inorder)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: In Construct Binary Tree from Preorder and Inorder Traversal, what does the first element of preorder tell us, and what does its index in inorder tell us?</b></summary>
  <br>
  <b>Answer:</b> The first element of preorder is the root. Finding its index $M$ in inorder reveals that all elements to the left of $M$ form the left subtree, and all elements to the right form the right subtree.
  </details>
- **Deep Practice (75–85 mins) | Problem 11**:
  - [Construct Binary Tree from Preorder and Inorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/) (NeetCode - Medium)
  - Thinking prompts: Use `Dictionary<int, int>` to find root index in inorder array in $O(1)$ time.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      private int _preIndex = 0;
      private readonly Dictionary<int, int> _inMap = new();

      public TreeNode BuildTree(int[] preorder, int[] inorder) {
          _preIndex = 0;
          _inMap.Clear();
          for (int i = 0; i < inorder.Length; i++) {
              _inMap[inorder[i]] = i;
          }
          return Build(preorder, 0, inorder.Length - 1);
      }
      
      private TreeNode Build(int[] preorder, int inStart, int inEnd) {
          if (inStart > inEnd) return null;
          
          int rootVal = preorder[_preIndex++];
          TreeNode root = new(rootVal);
          int mid = _inMap[rootVal];
          
          root.left = Build(preorder, inStart, mid - 1);
          root.right = Build(preorder, mid + 1, inEnd);
          return root;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N)$, Space: $O(N)$.
  </details>

---

### Day 12: Advanced Benchmarks & Spaced Retrieval
- **Benchmark 1**: [Binary Tree Maximum Path Sum](https://leetcode.com/problems/binary-tree-maximum-path-sum/) (NeetCode - Hard: subtree postorder max path)
- **Benchmark 2**: [Serialize and Deserialize Binary Tree](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/) (NeetCode - Hard: preorder serialization)
- **Benchmark 3**: [CSES: Tree Diameter](https://cses.fi/problemset/task/1131) (Two BFS runs on general tree)
- **Benchmark 4**: [CSES: Subtree Queries](https://cses.fi/problemset/task/1137) (Euler tour / tree flattening)
- **Benchmark 5**: [Codeforces 580C - Kefa and Park](https://codeforces.com/problemset/problem/580/C) (Tree DFS with consecutive cat condition)

---

### Spaced Retrieval Flashcards

<details>
<summary><b>Flashcard 1: When should you use Postorder traversal instead of Preorder?</b></summary>
<br>
Use Postorder when computing an aggregate property that depends on the results of child subtrees (e.g. tree height, subtree size, diameter, maximum path sum).
</details>

<details>
<summary><b>Flashcard 2: In Binary Tree Maximum Path Sum, why can a helper function return only <code>root.val + Math.Max(0, Math.Max(leftGain, rightGain))</code> to its parent?</b></summary>
<br>
Because a valid simple path cannot branch into both child subtrees AND continue upward to the parent node. It can only pick at most one branch to extend upward.
</details>

<details>
<summary><b>Flashcard 3: How does Euler Tour tree flattening simplify subtree queries?</b></summary>
<br>
It records entry and exit timestamps during DFS. A node's entire subtree maps to a contiguous contiguous segment $[in[u], out[u]]$ in an array, allowing range query structures like Fenwick Trees or Segment Trees to query subtrees in $O(\log N)$.
</details>

---

## 🎯 Topic Checklist (16 Problems)

| Day | # | Problem | Platform | Difficulty | Status |
|:---:|:---:|:---|:---:|:---:|:---:|
| Day 1 | 1 | [Invert Binary Tree](https://leetcode.com/problems/invert-binary-tree/) | NeetCode | Easy | - [ ] |
| Day 2 | 2 | [Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree/) | NeetCode | Easy | - [ ] |
| Day 3 | 3 | [Balanced Binary Tree](https://leetcode.com/problems/balanced-binary-tree/) | NeetCode | Easy | - [ ] |
| Day 4 | 4 | [Subtree of Another Tree](https://leetcode.com/problems/subtree-of-another-tree/) | NeetCode | Easy | - [ ] |
| Day 5 | 5 | [LCA of a BST](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/) | NeetCode | Medium | - [ ] |
| Day 6 | 6 | [Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/) | NeetCode | Medium | - [ ] |
| Day 7 | 7 | [Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view/) | NeetCode | Medium | - [ ] |
| Day 8 | 8 | [Count Good Nodes in Binary Tree](https://leetcode.com/problems/count-good-nodes-in-binary-tree/) | NeetCode | Medium | - [ ] |
| Day 9 | 9 | [Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/) | NeetCode | Medium | - [ ] |
| Day 10 | 10 | [Kth Smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/) | NeetCode | Medium | - [ ] |
| Day 11 | 11 | [Construct Tree from Preorder & Inorder](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/) | NeetCode | Medium | - [ ] |
| Day 12 | 12 | [Binary Tree Maximum Path Sum](https://leetcode.com/problems/binary-tree-maximum-path-sum/) | NeetCode | Hard | - [ ] |
| Day 12 | 13 | [Serialize and Deserialize Binary Tree](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/) | NeetCode | Hard | - [ ] |
| Day 12 | 14 | [Tree Diameter](https://cses.fi/problemset/task/1131) | CSES | Medium | - [ ] |
| Day 12 | 15 | [Subtree Queries](https://cses.fi/problemset/task/1137) | CSES | Hard | - [ ] |
| Day 12 | 16 | [Kefa and Park (580C)](https://codeforces.com/problemset/problem/580/C) | Codeforces | Rating 1500 | - [ ] |

---

[⬅️ Previous Topic: Linked Lists](06-linked-lists.html) | [Home](index.html) | [Next Topic: Heaps & Priority Queues ➡️](08-heaps.html)

