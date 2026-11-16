---
layout: default
title: "Topic 07: Binary Trees & BSTs"
---

# Topic 07: Binary Trees & BSTs

[← Topic 06: Linked Lists](06-linked-lists.md) \| [Schedule](schedule.md) \| [Next: Heaps & Priority Queues →](08-heaps.md)

---

## Daily Timeboxing (1.5 – 2.0 Hours)
- **00:00 – 00:15 (15m) | Active Recall Pre-Test**: Answer the conceptual questions below before reading further.
- **00:15 – 00:40 (25m) | YouTube Conceptual Anchor**: Watch the curated video segment on recursive subtree aggregation and BST properties.
- **00:40 – 01:30 (50m) | Coding Deep Dive**: Pen-and-paper dry run of Validate BST & Binary Tree Maximum Path Sum.
- **01:30 – 01:45 (15m) | Post-Session Flashcards**: Complete the flashcard prompts from memory.

---

## 1. Active Recall Pre-Test (15 Minutes)
Attempt to answer these questions on paper before expanding the solutions.

<details>
<summary><strong>Prompt 1: Why is checking `node.left.val < node.val < node.right.val` locally insufficient to validate a Binary Search Tree?</strong></summary>

The BST invariant is a **global** property: *every* node in the left subtree must be strictly less than the root, and *every* node in the right subtree must be strictly greater than the root.
Checking only immediate parent-child relationships permits violations elsewhere in the tree (e.g., a node in the right subtree having a left child whose value is smaller than the top root: `5 -> right: 10 -> left: 3`, which satisfies local conditions but violates the global root bound). 
Correct validation requires passing valid open ranges `(low, high)` down the recursive tree.
</details>

<details>
<summary><strong>Prompt 2: What is the difference between the Height of a node and the Depth of a node?</strong></summary>

- **Depth**: The number of edges along the path from the **root** down to that specific node. Computed top-down. The root has depth 0.
- **Height**: The number of edges on the longest downward path from that node to a **leaf**. Computed bottom-up. A leaf has height 0 (or 1 depending on convention), and an empty tree has height $-1$ (or 0).
</details>

<details>
<summary><strong>Prompt 3: In Binary Tree Maximum Path Sum, why can the recursive helper function return only one branch path, while updating the global maximum with both branches?</strong></summary>

A valid path cannot contain branching; it must be a simple path of distinct nodes. 
At any given root node, a path *originating* below that node can travel up the left branch, pass through the node, and continue down the right branch ($\text{left\_gain} + \text{val} + \text{right\_gain}$). 
However, to extend this path to the node's parent, the path can only continue down *one* of the subtrees: $\text{val} + \max(\text{left\_gain}, \text{right\_gain})$.
</details>

---

## 2. YouTube Conceptual Anchor (25 Minutes)

Recommended Video Anchors:
1. **NeetCode**: [Binary Tree Maximum Path Sum Detailed Intuition](https://www.youtube.com/watch?v=Hr5cWUld4vU) (15 mins)
2. **Abdul Bari**: [Binary Search Trees & Traversal Properties](https://www.youtube.com/watch?v=pYT9F8_LFTM) (22 mins)

### Core Invariants & Mental Model
1. **Subtree Decomposition Invariant**: Solve for the left subtree, solve for the right subtree, and combine the results at the current root. Bottom-up post-order recursion computes properties like height, balance, and diameter in a single $O(N)$ pass.
2. **BST Open-Interval Invariant**: 
   - When recursing left: `validate(node.left, low, node.val)`
   - When recursing right: `validate(node.right, node.val, high)`
3. **BFS Level-Delineation Invariant**: When doing level-order traversal with a queue, snapshot `level_size = len(queue)` before the inner loop. Iterating exactly `level_size` times processes one complete depth tier before any newly appended children are handled.

---

## 3. Canonical Code Templates

### Pattern A: Post-Order Subtree Aggregation (Diameter of Tree — $O(N)$ Time)
```python
from typing import Optional

class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

def diameter_of_binary_tree(root: Optional[TreeNode]) -> int:
    max_diameter = 0
    
    def max_depth(node: Optional[TreeNode]) -> int:
        nonlocal max_diameter
        if not node:
            return 0
            
        left_h = max_depth(node.left)
        right_h = max_depth(node.right)
        
        # Longest path through current node spans left height + right height
        max_diameter = max(max_diameter, left_h + right_h)
        
        return 1 + max(left_h, right_h)
        
    max_depth(root)
    return max_diameter
```

### Pattern B: Validate BST with Range Invariant ($O(N)$ Time, $O(H)$ Space)
```python
def is_valid_bst(root: Optional[TreeNode]) -> bool:
    def validate(node: Optional[TreeNode], low: float, high: float) -> bool:
        if not node:
            return True
            
        if not (low < node.val < high):
            return False
            
        return (validate(node.left, low, node.val) and 
                validate(node.right, node.val, high))
                
    return validate(root, float('-inf'), float('inf'))
```

### Pattern C: Level-Order Traversal Snapshot (BFS — $O(N)$ Time, $O(W)$ Space)
```python
from collections import deque
from typing import List

def level_order(root: Optional[TreeNode]) -> List[List[int]]:
    if not root:
        return []
        
    result = []
    queue = deque([root])
    
    while queue:
        level_size = len(queue)
        current_level = []
        
        for _ in range(level_size):
            node = queue.popleft()
            current_level.append(node.val)
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)
                
        result.append(current_level)
        
    return result
```

---

## 4. Coding Deep Dive: Worked Walkthrough (50 Minutes)

### Problem: Binary Tree Maximum Path Sum (LeetCode 124)
**Problem Statement**: A path in a binary tree is a sequence of nodes where each pair of adjacent nodes has an edge connecting them. Return the maximum path sum of any non-empty path.

#### Pen-and-Paper Dry Run Trace
Let tree be:
```text
       -10
       /  \
      9    20
          /  \
         15   7
```
Initial: `global_max = -inf`.
Post-order traversal execution:

| Step | Subtree Root | Left Branch Gain | Right Branch Gain | Subtree Path (`val + L + R`) | `global_max` Updated | Return to Parent (`val + max(L, R)`) |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| 1 | `9` (leaf) | 0 | 0 | $9 + 0 + 0 = 9$ | $\max(-\infty, 9) = 9$ | $\max(0, 9) = 9$ |
| 2 | `15` (leaf) | 0 | 0 | $15 + 0 + 0 = 15$ | $\max(9, 15) = 15$ | $\max(0, 15) = 15$ |
| 3 | `7` (leaf) | 0 | 0 | $7 + 0 + 0 = 7$ | $\max(15, 7) = 15$ | $\max(0, 7) = 7$ |
| 4 | `20` | 15 | 7 | $20 + 15 + 7 = 42$ | $\max(15, 42) = \mathbf{42}$ | $20 + \max(15, 7) = 35$ |
| 5 | `-10` | 9 | 35 | $-10 + 9 + 35 = 34$ | $\max(42, 34) = \mathbf{42}$ | $-10 + 35 = 25$ |

Final Result: **42** (Path: `15 -> 20 -> 7`).
Time Complexity: $O(N)$ visiting each node once.
Space Complexity: $O(H)$ recursion stack where $H$ is tree height.

---

## 5. Curated Problem Set (16 Problems)

Track your daily solutions by checking off the markdown boxes.

### NeetCode / LeetCode (Core)
- [ ] [LeetCode 226: Invert Binary Tree](https://leetcode.com/problems/invert-binary-tree/) | Easy | Post-Order Subtree Swap
- [ ] [LeetCode 104: Maximum Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/) | Easy | Bottom-Up Recursion
- [ ] [LeetCode 543: Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree/) | Easy | Path Sum Combination
- [ ] [LeetCode 110: Balanced Binary Tree](https://leetcode.com/problems/balanced-binary-tree/) | Easy | Early Exit Sentinel (`-1`)
- [ ] [LeetCode 100: Same Tree](https://leetcode.com/problems/same-tree/) | Easy | Dual Tree Traversal Recursion
- [ ] [LeetCode 572: Subtree of Another Tree](https://leetcode.com/problems/subtree-of-another-tree/) | Easy | Recursive Tree Match / Merkle Hashing
- [ ] [LeetCode 235: Lowest Common Ancestor of a BST](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/) | Medium | BST Ordering Split Direction
- [ ] [LeetCode 102: Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/) | Medium | Queue Level-Size Snapshots
- [ ] [LeetCode 199: Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view/) | Medium | Level Order Last Element
- [ ] [LeetCode 98: Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/) | Medium | Open Range `(low, high)` Validation
- [ ] [LeetCode 230: Kth Smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/) | Medium | In-Order Traversal Invariant
- [ ] [LeetCode 124: Binary Tree Maximum Path Sum](https://leetcode.com/problems/binary-tree-maximum-path-sum/) | Hard | Global Max vs Parent Return Gain

### CSES Problem Set (Free Tier, No Login Wall)
- [ ] [CSES 1674: Subordinates](https://cses.fi/problemset/task/1674) | Tree Algorithms | Subtree Node Count Aggregation
- [ ] [CSES 1131: Tree Diameter](https://cses.fi/problemset/task/1131) | Tree Algorithms | Double BFS / DFS for Longest Path
- [ ] [CSES 1132: Tree Distances I](https://cses.fi/problemset/task/1132) | Tree Algorithms | Tree Rerooting / DP on Trees

### Codeforces & AtCoder (Rating 800–1400)
- [ ] [Codeforces 115A: Party](https://codeforces.com/problemset/problem/115/A) | 900 | Forest Max Depth Calculation
- [ ] [AtCoder ABC126_D: Even Relation](https://atcoder.jp/contests/abc126/tasks/abc126_d) | 1100 | Tree Bipartite Parity Coloring

---

## 6. Post-Session Active Recall Flashcards (15 Minutes)

<details>
<summary><strong>Flashcard 1: Why does an In-Order traversal of a Binary Search Tree always yield values in strictly ascending order?</strong></summary>

In-order traversal visits: `Left Subtree -> Root -> Right Subtree`.
By BST definition, all nodes in the left subtree are smaller than the root, and all nodes in the right subtree are larger than the root. Recursively applying this structure guarantees every node is visited after its smaller descendants and before its larger descendants, producing a sorted sequence.
</details>

<details>
<summary><strong>Flashcard 2: How can Lowest Common Ancestor (LCA) in a BST be solved in $O(H)$ time and $O(1)$ space without extra data structures?</strong></summary>

Start at `root`:
- If both nodes $p$ and $q$ have values strictly less than `root.val`, the LCA must be in the left subtree: `root = root.left`.
- If both $p$ and $q$ have values strictly greater than `root.val`, the LCA must be in the right subtree: `root = root.right`.
- If $p$ and $q$ split (one is smaller, one is larger) or one equals `root.val`, the current root is the split point and therefore the Lowest Common Ancestor.
</details>

<details>
<summary><strong>Flashcard 3: How does the Morris Traversal achieve $O(N)$ time and $O(1)$ space for tree traversals?</strong></summary>

It utilizes unused `null` pointers in leaf nodes (threaded binary trees). Before visiting the left child, it finds the in-order predecessor of the current node. If the predecessor's right child is null, it temporarily points it to the current node (creating a thread to return) and moves left. When it returns via the thread, it removes the link and visits the node, eliminating recursion stack memory.
</details>

---

[← Topic 06: Linked Lists](06-linked-lists.md) \| [Next Topic: Heaps & Priority Queues →](08-heaps.md)
