---
layout: default
title: "Topic 09: Backtracking & Recursive Search"
---

# Topic 09: Backtracking & Recursive Search

[← Topic 08: Heaps](08-heaps.md) \| [Schedule](schedule.md) \| [Next: Graphs I (Traversals) →](10-graphs-traversal.md)

---

## Daily Timeboxing (1.5 – 2.0 Hours)
- **00:00 – 00:15 (15m) | Active Recall Pre-Test**: Answer the conceptual questions below before reading further.
- **00:15 – 00:40 (25m) | YouTube Conceptual Anchor**: Watch the curated video segment on state tree exploration and duplicate pruning.
- **00:40 – 01:30 (50m) | Coding Deep Dive**: Pen-and-paper dry run of Subsets II & N-Queens.
- **01:30 – 01:45 (15m) | Post-Session Flashcards**: Complete the flashcard prompts from memory.

---

## 1. Active Recall Pre-Test (15 Minutes)
Attempt to answer these questions on paper before expanding the solutions.

<details>
<summary><strong>Prompt 1: What is the formal definition of Backtracking, and how does it differ from exhaustive brute-force search?</strong></summary>

Backtracking is a systematic method for generating all (or optimal) solutions to a computational problem incrementally. 
Unlike exhaustive brute force—which blindly generates all $O(2^N)$ or $O(N!)$ candidate leaves before evaluating validity—backtracking constructs candidate states step by step. 
As soon as a partial state violates a constraint, backtracking **prunes** the entire subtree below that state immediately, abandoning the choice and backtracking to the previous decision point.
</details>

<details>
<summary><strong>Prompt 2: In problems with duplicate elements (like Subsets II and Combination Sum II), what condition prunes duplicate branches without missing valid combinations?</strong></summary>

1. Sort the array first: `nums.sort()`.
2. When iterating over candidates at the current recursive depth using loop index `i` from `start` to $N-1$:
   ```python
   if i > start and nums[i] == nums[i - 1]:
       continue  # Prune duplicate sibling branch
   ```
This invariant permits using identical values down the *same* vertical branch (`i == start`), but forbids picking the same value across *sibling* branches at the same recursion level.
</details>

<details>
<summary><strong>Prompt 3: In the N-Queens problem, how can column and diagonal conflicts be checked in $O(1)$ time without scanning the board?</strong></summary>

For a queen placed at coordinate `(row, col)` on an $N \times N$ chessboard:
- **Column Conflict**: Tracked via `cols = set()` storing `col`.
- **Positive Diagonal ($\nearrow$)**: All cells on the same positive diagonal share the constant property `row + col`. Tracked via `pos_diag = set()` storing `row + col`.
- **Negative Diagonal ($\searrow$)**: All cells on the same negative diagonal share the constant property `row - col`. Tracked via `neg_diag = set()` storing `row - col`.
Checking or inserting a queen into these three sets takes $O(1)$ time.
</details>

---

## 2. YouTube Conceptual Anchor (25 Minutes)

Recommended Video Anchors:
1. **Abdul Bari**: [N-Queens Problem using Backtracking](https://www.youtube.com/watch?v=xFv_Hl4B83A) (20 mins)
2. **NeetCode**: [Backtracking / Combination Sum & Subsets II](https://www.youtube.com/watch?v=GBKI9VSKdGg) (15 mins)

### Core Invariants & Mental Model
1. **The 3-Step Recursive Contract**:
   - **Choose**: Add candidate element to current path; update constraint sets.
   - **Explore**: Recurse to next index or depth level.
   - **Unchoose (Backtrack)**: Remove candidate element from path; restore constraint sets to their exact prior state.
2. **Path vs Output Copying**: In Python, `path` is passed by reference. When recording a valid solution, you must make a deep copy: `result.append(path[:])` or `list(path)`. Appending `path` directly records references that will be empty upon function exit.
3. **Complexity Ceiling**:
   - Subsets: $O(N \cdot 2^N)$
   - Permutations: $O(N \cdot N!)$
   - Combinations ($K$ elements from $N$): $O(K \cdot \binom{N}{K})$

---

## 3. Canonical Code Templates

### Pattern A: Subsets & Combinations with Sibling Pruning ($O(N \cdot 2^N)$ Time)
```python
from typing import List

def subsets_with_dup(nums: List[int]) -> List[List[int]]:
    nums.sort() # Mandatory for duplicate grouping
    result = []
    
    def backtrack(start: int, path: List[int]):
        result.append(path[:]) # Capture state at every tree node
        
        for i in range(start, len(nums)):
            # Sibling pruning invariant
            if i > start and nums[i] == nums[i - 1]:
                continue
                
            path.append(nums[i])
            backtrack(i + 1, path) # Move to next index
            path.pop()             # Revert choice
            
    backtrack(0, [])
    return result
```

### Pattern B: N-Queens Constraint Sets ($O(N!)$ Time, $O(N)$ Space)
```python
from typing import List

def solve_n_queens(n: int) -> List[List[str]]:
    col_set = set()
    pos_diag = set() # row + col
    neg_diag = set() # row - col
    
    result = []
    board = [["."] * n for _ in range(n)]
    
    def backtrack(row: int):
        if row == n:
            result.append(["".join(r) for r in board])
            return
            
        for col in range(n):
            if col in col_set or (row + col) in pos_diag or (row - col) in neg_diag:
                continue
                
            # Choose
            col_set.add(col)
            pos_diag.add(row + col)
            neg_diag.add(row - col)
            board[row][col] = "Q"
            
            # Explore
            backtrack(row + 1)
            
            # Unchoose
            col_set.remove(col)
            pos_diag.remove(row + col)
            neg_diag.remove(row - col)
            board[row][col] = "."
            
    backtrack(0)
    return result
```

---

## 4. Coding Deep Dive: Worked Walkthrough (50 Minutes)

### Problem: Combination Sum II (LeetCode 40)
**Problem Statement**: Given candidate numbers `candidates` (which may contain duplicates) and a target `target`, find all unique combinations where candidate numbers sum to `target`. Each number may be used only once per combination.

#### Pen-and-Paper Dry Run Trace
Let `candidates = [10, 1, 2, 7, 6, 1, 5]`, `target = 8`.
1. Sort: `candidates = [1, 1, 2, 5, 6, 7, 10]`.
2. Trace recursive decision tree from `start = 0`:

```text
                              Root (target=8)
                /          |           \              \
            Pick 1(idx 0)  Skip 1(idx 1) Pick 2(idx 2) Pick 5(idx 3)
           (target=7)       [PRUNED]     (target=6)    (target=3)
          /         \                     /         \
     Pick 1(idx 1) Pick 2(idx 2)      Pick 5(idx 3) Pick 6(idx 4)
     (target=6)    (target=5)         (target=1)    (target=0) -> [2, 6] VALID!
     /         \         \                 |
Pick 2(idx 2) Pick 6   Pick 5(idx 3)     [PRUNED]
(target=4)    [1,1,6]  [1,2,5] VALID!
```

Key Pruning Events:
- At `start = 0`, index `0` chooses value `1`. Index `1` has value `1`, but `i > start` ($1 > 0$) and `candidates[1] == candidates[0]`. Branch is **pruned** to prevent duplicate `[1, ...]` subsets.
- Valid combinations produced: `[[1, 1, 6], [1, 2, 5], [1, 7], [2, 6]]`.

---

## 5. Curated Problem Set (15 Problems)

Track your daily solutions by checking off the markdown boxes.

### NeetCode / LeetCode (Core)
- [ ] [LeetCode 78: Subsets](https://leetcode.com/problems/subsets/) | Medium | Power Set Recursive Decision Tree
- [ ] [LeetCode 90: Subsets II](https://leetcode.com/problems/subsets-ii/) | Medium | Sibling Pruning with Sorting
- [ ] [LeetCode 39: Combination Sum](https://leetcode.com/problems/combination-sum/) | Medium | Unbounded Element Reuse
- [ ] [LeetCode 40: Combination Sum II](https://leetcode.com/problems/combination-sum-ii/) | Medium | Single-Use Candidate Pruning
- [ ] [LeetCode 46: Permutations](https://leetcode.com/problems/permutations/) | Medium | Visited Flag / In-Place Swapping
- [ ] [LeetCode 47: Permutations II](https://leetcode.com/problems/permutations-ii/) | Medium | Permutations with Duplicates
- [ ] [LeetCode 17: Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number/) | Medium | Digit Mapping Product Tree
- [ ] [LeetCode 79: Word Search](https://leetcode.com/problems/word-search/) | Medium | 2D Grid DFS with Grid Marking
- [ ] [LeetCode 131: Palindrome Partitioning](https://leetcode.com/problems/palindrome-partitioning/) | Medium | String Slicing with Palindrome Verification
- [ ] [LeetCode 51: N-Queens](https://leetcode.com/problems/n-queens/) | Hard | Diagonal Constraint Sets
- [ ] [LeetCode 37: Sudoku Solver](https://leetcode.com/problems/sudoku-solver/) | Hard | 9x9 Constraint Propagation Backtracking

### CSES Problem Set (Free Tier, No Login Wall)
- [ ] [CSES 1623: Apple Division](https://cses.fi/problemset/task/1623) | Introductory Problems | Subset Weight Difference Minimization
- [ ] [CSES 1624: Chessboard and Queens](https://cses.fi/problemset/task/1624) | Introductory Problems | N-Queens with Reserved Cells
- [ ] [CSES 1622: Creating Strings](https://cses.fi/problemset/task/1622) | Introductory Problems | Lexicographical Unique Permutations

### Codeforces & AtCoder (Rating 800–1400)
- [ ] [Codeforces 550B: Preparing Olympiad](https://codeforces.com/problemset/problem/550/B) | 1400 | Subset Generation with Min/Max Constraints
- [ ] [AtCoder ABC197_C: ORXOR](https://atcoder.jp/contests/abc197/tasks/abc197_c) | 1000 | Bitwise Partition Backtracking

---

## 6. Post-Session Active Recall Flashcards (15 Minutes)

<details>
<summary><strong>Flashcard 1: When should you use `start` index vs a `visited` boolean array in backtracking?</strong></summary>

- **Use `start` index** for **Subsets and Combinations** where order does not matter ($[1, 2]$ is identical to $[2, 1]$). Advancing `start = i + 1` enforces strictly ascending index picks and prevents permutations.
- **Use `visited` array (or bitmask)** for **Permutations** where order matters ($[1, 2] \ne [2, 1]$) and every unused element must be considered at every position.
</details>

<details>
<summary><strong>Flashcard 2: In Word Search (LeetCode 79), how can you avoid allocating an $O(M \times N)$ visited matrix?</strong></summary>

Temporarily mutate the board in place:
1. `temp = board[r][c]`
2. `board[r][c] = '#'` (marks cell as visited during exploration)
3. Recurse to adjacent neighbors.
4. `board[r][c] = temp` (backtracks and restores the character upon returning).
This reduces auxiliary space to $O(L)$ where $L$ is word length (recursion stack).
</details>

<details>
<summary><strong>Flashcard 3: How does the bitmask optimization accelerate N-Queens?</strong></summary>

Instead of hash sets, use three integers (`cols`, `pos_diag`, `neg_diag`) as bitmasks. 
Available slots in row $r$ are found in a single CPU instruction:
$$\text{available} = \sim(\text{cols} \mid \text{pos\_diag} \mid \text{neg\_diag}) \ \& \ ((1 \ll N) - 1)$$
Each queen position is extracted via `p = available & -available` (lowest set bit), reducing execution time by over $10\times$.
</details>

---

[← Topic 08: Heaps](08-heaps.md) \| [Next Topic: Graphs I (Traversals) →](10-graphs-traversal.md)
