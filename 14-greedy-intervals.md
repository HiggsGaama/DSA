---
layout: default
title: "Topic 14: Greedy Algorithms & Intervals"
---

# Topic 14: Greedy Algorithms & Intervals

[← Topic 13: 2-D Dynamic Programming](13-2d-dp.md) \| [Schedule](schedule.md) \| [Next: Bit Manipulation & Core Math →](15-bit-manipulation-math.md)

---

## Daily Timeboxing (1.5 – 2.0 Hours)
- **00:00 – 00:15 (15m) | Active Recall Pre-Test**: Answer the conceptual questions below before reading further.
- **00:15 – 00:40 (25m) | YouTube Conceptual Anchor**: Watch the curated video segment on greedy choice invariants and interval scheduling.
- **00:40 – 01:30 (50m) | Coding Deep Dive**: Pen-and-paper dry run of Non-overlapping Intervals & Merge Intervals.
- **01:30 – 01:45 (15m) | Post-Session Flashcards**: Complete the flashcard prompts from memory.

---

## 1. Active Recall Pre-Test (15 Minutes)
Attempt to answer these questions on paper before expanding the solutions.

<details>
<summary><strong>Prompt 1: What is the formal difference between a Greedy Algorithm and Dynamic Programming?</strong></summary>

Both paradigms solve problems with optimal substructure. 
However, **Dynamic Programming** exhaustively considers multiple candidate decisions at each step and evaluates their downstream subproblem consequences before making a choice (caching overlapping states).
A **Greedy Algorithm** makes a locally optimal choice at every step based on a static rule *without* exploring alternative branches or back-tracking, committing immediately to that choice. A greedy strategy is correct if and only if the **Greedy Choice Property** holds (a globally optimal solution can always be reached via locally optimal decisions).
</details>

<details>
<summary><strong>Prompt 2: When solving the Interval Scheduling problem (maximizing non-overlapping intervals), why MUST intervals be sorted by their END times rather than START times?</strong></summary>

Sorting by start times fails: an interval that starts early could have a massive duration that overlaps with and blocks hundreds of subsequent short intervals (e.g., `[0, 100]` vs `[1, 2], [2, 3], [3, 4]`).
By sorting by **end time**, picking the interval that finishes earliest frees up the resource as soon as possible, leaving the maximum possible remaining time for all subsequent candidate intervals.
</details>

<details>
<summary><strong>Prompt 3: How does the Jump Game (LeetCode 55) maintain an $O(1)$ space reachability horizon?</strong></summary>

Maintain a scalar variable `max_reach` initialized to 0. 
Iterate index $i$ through the array. 
If $i > \text{max\_reach}$, index $i$ is unreachable, and we immediately return `False`. 
Otherwise, update $\text{max\_reach} = \max(\text{max\_reach}, i + \text{nums}[i])$. 
If `max_reach >= len(nums) - 1`, the end is reachable.
</details>

---

## 2. YouTube Conceptual Anchor (25 Minutes)

Recommended Video Anchors:
1. **Abdul Bari**: [Greedy Strategy & Interval Scheduling Knapsack](https://www.youtube.com/watch?v=ARvQcqJ_-NY) (22 mins)
2. **NeetCode**: [Merge Intervals & Non-Overlapping Intervals](https://www.youtube.com/watch?v=44H3cEC2fFM) (15 mins)

### Core Invariants & Mental Model
1. **Start vs End Sorting Rule**:
   - **Merge Intervals**: Sort by **start time** (`key=lambda x: x[0]`). If current interval starts before previous interval ends ($s_i \le e_{i-1}$), merge them: $e_{i-1} = \max(e_{i-1}, e_i)$.
   - **Interval Scheduling (Remove min overlaps)**: Sort by **end time** (`key=lambda x: x[1]`). Greedily keep the interval with the earliest finish time.
2. **Horizon Invariant (Jump Game II)**: Track `current_jump_end` and `farthest_reach`. When the loop index reaches `current_jump_end`, increment the jump counter and advance `current_jump_end = farthest_reach`.
3. **Two-Pass Slope Invariant (Candy Problem)**: Ensure left-neighbor condition with a left-to-right pass; ensure right-neighbor condition with a right-to-left pass. Take $\max(\text{left}[i], \text{right}[i])$ at each position.

---

## 3. Canonical Code Templates

### Pattern A: Merge Intervals ($O(N \log N)$ Time, $O(N)$ Space)
```python
from typing import List

def merge_intervals(intervals: List[List[int]]) -> List[List[int]]:
    intervals.sort(key=lambda x: x[0]) # Sort by start time
    merged = []
    
    for start, end in intervals:
        # If merged is empty or no overlap, append new interval
        if not merged or merged[-1][1] < start:
            merged.append([start, end])
        else:
            # Overlap exists: extend the end of the previous interval
            merged[-1][1] = max(merged[-1][1], end)
            
    return merged
```

### Pattern B: Non-Overlapping Intervals (Earliest End Time — $O(N \log N)$ Time)
```python
from typing import List

def erase_overlap_intervals(intervals: List[List[int]]) -> int:
    # Sort by end time to maximize remaining available room
    intervals.sort(key=lambda x: x[1])
    
    removals = 0
    prev_end = float('-inf')
    
    for start, end in intervals:
        if start >= prev_end:
            prev_end = end # Non-overlapping: keep this interval
        else:
            removals += 1  # Overlapping: greedily drop this interval
            
    return removals
```

### Pattern C: Greedy Reachability Horizon (Jump Game — $O(N)$ Time, $O(1)$ Space)
```python
from typing import List

def can_jump(nums: List[int]) -> bool:
    max_reach = 0
    
    for i, jump in enumerate(nums):
        if i > max_reach:
            return False # Trapped behind an unreachable gap
            
        max_reach = max(max_reach, i + jump)
        if max_reach >= len(nums) - 1:
            return True
            
    return True
```

---

## 4. Coding Deep Dive: Worked Walkthrough (50 Minutes)

### Problem: Non-Overlapping Intervals (LeetCode 435)
**Problem Statement**: Given an array of intervals `intervals` where `intervals[i] = [start, end]`, return the minimum number of intervals you need to remove to make the rest of the intervals non-overlapping.

#### Pen-and-Paper Dry Run Trace
Let `intervals = [[1, 2], [2, 3], [3, 4], [1, 3]]`.
1. Sort by end time: `[[1, 2], [2, 3], [1, 3], [3, 4]]`.
Initialize: `prev_end = -inf`, `removals = 0`.

| Step | Current Interval `[start, end]` | `start >= prev_end`? | Decision & Action | New `prev_end` | Total Removals |
|:---:|:---:|:---:|:---|:---:|:---:|
| 1 | `[1, 2]` | $1 \ge -\infty$ (**Yes**) | Keep interval | 2 | 0 |
| 2 | `[2, 3]` | $2 \ge 2$ (**Yes**) | Keep interval | 3 | 0 |
| 3 | `[1, 3]` | $1 \ge 3$ (**No**) | **Conflict!** Overlaps with `[2, 3]`. Drop `[1, 3]`. | 3 | **1** |
| 4 | `[3, 4]` | $3 \ge 3$ (**Yes**) | Keep interval | 4 | 1 |

Total Removals Required: **1** (Interval `[1, 3]` removed).
Time Complexity: $O(N \log N)$ for sorting.
Space Complexity: $O(1)$ auxiliary.

---

## 5. Curated Problem Set (15 Problems)

Track your daily solutions by checking off the markdown boxes.

### NeetCode / LeetCode (Core)
- [ ] [LeetCode 55: Jump Game](https://leetcode.com/problems/jump-game/) | Medium | Max Reachable Horizon
- [ ] [LeetCode 45: Jump Game II](https://leetcode.com/problems/jump-game-ii/) | Medium | BFS Level Boundary Jumps
- [ ] [LeetCode 56: Merge Intervals](https://leetcode.com/problems/merge-intervals/) | Medium | Start-Time Sorting & Boundary Extension
- [ ] [LeetCode 57: Insert Interval](https://leetcode.com/problems/insert-interval/) | Medium | Linear Pass Splice & Merge
- [ ] [LeetCode 435: Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/) | Medium | Earliest Finish Time Greedy Removal
- [ ] [LeetCode 135: Candy](https://leetcode.com/problems/candy/) | Hard | Two-Pass Slope Value Assignment
- [ ] [LeetCode 846: Hand of Straights](https://leetcode.com/problems/hand-of-straights/) | Medium | Min-Heap / TreeMap Consecutive Sequence Verification
- [ ] [LeetCode 763: Partition Labels](https://leetcode.com/problems/partition-labels/) | Medium | Last Seen Index Boundary Absorption
- [ ] [LeetCode 678: Valid Parenthesis String](https://leetcode.com/problems/valid-parenthesis-string/) | Medium | Dual Range Min/Max Open Bracket Counters
- [ ] [LeetCode 1899: Merge Triplets to Form Target Triplet](https://leetcode.com/problems/merge-triplets-to-form-target-triplet/) | Medium | Invalid Triplet Elimination Filter

### CSES Problem Set (Free Tier, No Login Wall)
- [ ] [CSES 1629: Movie Festival](https://cses.fi/problemset/task/1629) | Sorting & Searching | Classic Activity Selection Earliest End Time
- [ ] [CSES 1630: Tasks and Deadlines](https://cses.fi/problemset/task/1630) | Sorting & Searching | Shortest Duration First Reward Maximization
- [ ] [CSES 1631: Reading Books](https://cses.fi/problemset/task/1631) | Sorting & Searching | Max Element Dominance Threshold Check

### Codeforces & AtCoder (Rating 800–1400)
- [ ] [Codeforces 1221C: Perfect Team](https://codeforces.com/problemset/problem/1221/C) | 1100 | Math & Greedy Student Pairing
- [ ] [AtCoder ABC103_D: Islands War](https://atcoder.jp/contests/abc103/tasks/abc103_d) | 1200 | Interval End Cut Bridge Removal

---

## 6. Post-Session Active Recall Flashcards (15 Minutes)

<details>
<summary><strong>Flashcard 1: In Partition Labels (LeetCode 763), how do you decide where to cut a string into substrings so no letter appears in more than one partition?</strong></summary>

1. First pass: Record the `last_index[char]` for every character in the string.
2. Second pass: Maintain `end = 0` and `start = 0`. For each character at index $i$:
   $$\text{end} = \max(\text{end}, \text{last\_index}[S[i]])$$
   When $i == \text{end}$, all characters seen in $S[\text{start} \dots i]$ have their last appearances within this window. Cut the partition here, append $i - \text{start} + 1$ to the answer, and update $\text{start} = i + 1$.
</details>

<details>
<summary><strong>Flashcard 2: How does the two-pass algorithm for Candy (LeetCode 135) ensure all constraints are satisfied?</strong></summary>

Initialize `candies = [1] * N`.
1. **Left-to-Right Pass**: If `ratings[i] > ratings[i - 1]`, set `candies[i] = candies[i - 1] + 1`. This satisfies the left neighbor condition.
2. **Right-to-Left Pass**: If `ratings[i] > ratings[i + 1]`, set `candies[i] = max(candies[i], candies[i + 1] + 1)`. 
Taking the maximum preserves the left neighbor constraint while strictly satisfying the right neighbor constraint. Total time: $O(N)$, space: $O(N)$.
</details>

<details>
<summary><strong>Flashcard 3: What is the Exchange Argument in greedy algorithm proofs?</strong></summary>

A mathematical proof technique showing that any hypothetical optimal solution $OPT$ can be transformed into the greedy solution $G$ without worsening the objective value. 
By considering the first position where $OPT$ differs from $G$ and swapping $OPT$'s choice with the greedy choice, we show the solution remains valid and at least as good. By induction, $G$ is globally optimal.
</details>

---

[← Topic 13: 2-D Dynamic Programming](13-2d-dp.md) \| [Next Topic: Bit Manipulation & Core Math →](15-bit-manipulation-math.md)
