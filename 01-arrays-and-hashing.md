---
layout: default
title: "Topic 01: Arrays, Hashing & Frequency Counting"
---

# Topic 01: Arrays, Hashing & Frequency Counting

[← Home](index.md) \| [Schedule](schedule.md) \| [Next: Two Pointers →](02-two-pointers.md)

---

## Daily Timeboxing (1.5 – 2.0 Hours)
- **00:00 – 00:15 (15m) | Active Recall Pre-Test**: Answer the conceptual questions below before reading further.
- **00:15 – 00:40 (25m) | YouTube Conceptual Anchor**: Watch the recommended video segment on hash collisions and prefix states.
- **00:40 – 01:30 (50m) | Coding Deep Dive**: Dry run on paper first, then implement Two Sum & Subarray Sum Equals K.
- **01:30 – 01:45 (15m) | Post-Session Flashcards**: Self-test using the retrieval cards at the bottom.

---

## 1. Active Recall Pre-Test (15 Minutes)
Attempt to answer these questions on paper before expanding the solutions.

<details>
<summary><strong>Prompt 1: What is the underlying data structure of a Hash Map, and how are collisions resolved?</strong></summary>

A hash map is backed by a contiguous fixed-size array (buckets). A hash function maps an arbitrary key to an integer bucket index: `index = hash(key) % capacity`.
Collisions (two keys mapping to the same bucket) are typically resolved via:
1. **Separate Chaining**: Each bucket stores a linked list (or balanced red-black tree when chain length exceeds a threshold, as in Java 8+ HashMap). Worst-case lookup degrades to $O(N)$ or $O(\log N)$, but amortized lookup is $O(1)$ under uniform hashing and a load factor $\alpha = N / \text{capacity} \le 0.75$.
2. **Open Addressing** (Linear/Quadratic Probing, Double Hashing): All entries live in the array itself. When a collision occurs, subsequent slots are probed until an empty cell is found. Requires careful tombstone marking on deletions.
</details>

<details>
<summary><strong>Prompt 2: In the classic Two Sum problem, why does checking `target - num in seen` work in $O(N)$ time instead of $O(N^2)$?</strong></summary>

In the naive nested loop approach, for every element $x$, we scan the rest of the array ($N-1$ elements) to find $y$ such that $x + y = \text{target}$. 
By maintaining a hash map of elements visited so far, checking if the complementary value $y = \text{target} - x$ exists requires only an $O(1)$ hash table lookup. Because each element is inserted and checked exactly once, the entire array is processed in a single pass of $O(N)$ time and $O(N)$ auxiliary space.
</details>

<details>
<summary><strong>Prompt 3: What invariant makes Prefix Sums ideal for answering range sum queries in $O(1)$ time?</strong></summary>

Given an array $A$ of length $N$, the prefix sum array $P$ is defined as:
$$P[i] = \sum_{k=0}^{i-1} A[k], \quad P[0] = 0$$
The sum of any contiguous slice $A[L \dots R]$ (0-indexed, inclusive) can be computed directly as:
$$\text{Sum}(L, R) = P[R + 1] - P[L]$$
This transforms any range sum query from an $O(N)$ linear accumulation into an $O(1)$ arithmetic subtraction.
</details>

---

## 2. YouTube Conceptual Anchor (25 Minutes)

Recommended Video Anchors:
1. **NeetCode**: [Arrays & Hashing Playlist / Two Sum Deep Dive](https://www.youtube.com/watch?v=KLlXCFG5TnA) (10 mins)
2. **Abdul Bari**: [Hashing Techniques & Collision Resolution](https://www.youtube.com/watch?v=mFY0J5W8Udk) (18 mins)

### Core Invariants & Mental Model
1. **Complement Lookback Invariant**: As we iterate index $i$ from $0 \dots N-1$, our hash map contains exactly the state of elements from indices $0 \dots i-1$. We inspect the past before recording the present.
2. **Prefix Remainder / Running State Invariant**: For problems like *Subarray Sum Equals K*, we maintain a running sum $S_i$. A subarray ending at $i$ with sum $K$ exists if and only if there was a previous prefix sum $S_j = S_i - K$ with $j < i$.
3. **Bucket Sort Invariant ($O(N)$ frequency sorting)**: When finding the top $K$ frequent elements among $N$ items, the maximum frequency cannot exceed $N$. Instead of sorting with a min-heap ($O(N \log K)$) or full sort ($O(N \log N)$), we use array indices $0 \dots N$ as frequency buckets for true linear time.

---

## 3. Canonical Code Templates

### Pattern A: Complement Hash Map Lookup ($O(N)$ Time, $O(N)$ Space)
```python
from typing import List, Dict

def two_sum(nums: List[int], target: int) -> List[int]:
    # Invariant: seen_indices maps value -> index for all visited elements
    seen_indices: Dict[int, int] = {}
    
    for current_idx, num in enumerate(nums):
        complement = target - num
        if complement in seen_indices:
            return [seen_indices[complement], current_idx]
        seen_indices[num] = current_idx
        
    return []
```

### Pattern B: Prefix Sum with Frequency Hash Map ($O(N)$ Time, $O(N)$ Space)
```python
from typing import List
from collections import defaultdict

def subarray_sum(nums: List[int], k: int) -> int:
    # Invariant: prefix_counts stores the frequency of running prefix sums
    # Base case: A prefix sum of 0 has occurred once before processing any elements
    prefix_counts = defaultdict(int)
    prefix_counts[0] = 1
    
    running_sum = 0
    total_subarrays = 0
    
    for num in nums:
        running_sum += num
        # If (running_sum - k) was seen before, each instance forms a valid subarray ending here
        target_prefix = running_sum - k
        if target_prefix in prefix_counts:
            total_subarrays += prefix_counts[target_prefix]
            
        prefix_counts[running_sum] += 1
        
    return total_subarrays
```

---

## 4. Coding Deep Dive: Worked Walkthrough (50 Minutes)

### Problem: Subarray Sum Equals K (LeetCode 560)
**Problem Statement**: Given an integer array `nums` and an integer `k`, return the total number of subarrays whose sum equals `k`.

#### Pen-and-Paper Dry Run Trace
Let `nums = [1, -1, 1, 1, 1, -1]`, `k = 2`.
Initial state: `running_sum = 0`, `prefix_counts = {0: 1}`, `total = 0`.

| Step | `num` | `running_sum` | `target_prefix = running_sum - 2` | In `prefix_counts`? | Updated `total` | `prefix_counts` state |
|:---:|:---:|:---:|:---:|:---:|:---:|:---|
| Init | - | 0 | - | - | 0 | `{0: 1}` |
| 1 | 1 | 1 | -1 | No | 0 | `{0: 1, 1: 1}` |
| 2 | -1 | 0 | -2 | No | 0 | `{0: 2, 1: 1}` |
| 3 | 1 | 1 | -1 | No | 0 | `{0: 2, 1: 2}` |
| 4 | 1 | 2 | 0 | **Yes (count = 2)** | **0 + 2 = 2** | `{0: 2, 1: 2, 2: 1}` |
| 5 | 1 | 3 | 1 | **Yes (count = 2)** | **2 + 2 = 4** | `{0: 2, 1: 2, 2: 1, 3: 1}` |
| 6 | -1 | 2 | 0 | **Yes (count = 2)** | **4 + 2 = 6** | `{0: 2, 1: 2, 2: 2, 3: 1}` |

Total valid subarrays = **6**.
Time Complexity: $O(N)$ single pass.
Space Complexity: $O(N)$ for prefix hash table.

---

## 5. Curated Problem Set (16 Problems)

Track your daily solutions by checking off the markdown boxes.

### NeetCode / LeetCode (Core)
- [ ] [LeetCode 217: Contains Duplicate](https://leetcode.com/problems/contains-duplicate/) | Easy | Set Membership & Uniqueness
- [ ] [LeetCode 242: Valid Anagram](https://leetcode.com/problems/valid-anagram/) | Easy | Frequency Array / Counter
- [ ] [LeetCode 1: Two Sum](https://leetcode.com/problems/two-sum/) | Easy | Complement Hash Map
- [ ] [LeetCode 49: Group Anagrams](https://leetcode.com/problems/group-anagrams/) | Medium | Key Canonicalization (Tuple / Sort)
- [ ] [LeetCode 347: Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/) | Medium | Bucket Sort / Min-Heap
- [ ] [LeetCode 238: Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/) | Medium | Prefix & Suffix Products
- [ ] [LeetCode 36: Valid Sudoku](https://leetcode.com/problems/valid-sudoku/) | Medium | Bitmask / Subgrid Hash Set
- [ ] [LeetCode 128: Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence/) | Medium | Set Sequence Start Detection
- [ ] [LeetCode 560: Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/) | Medium | Prefix Sum + Frequency Map
- [ ] [LeetCode 271: Encode and Decode Strings](https://leetcode.com/problems/encode-and-decode-strings/) | Medium | Length Delimiter Protocol

### CSES Problem Set (Free Tier, No Login Wall)
- [ ] [CSES 1621: Distinct Numbers](https://cses.fi/problemset/task/1621) | Sorting & Searching | Unique Value Counting
- [ ] [CSES 1660: Subarray Sums I](https://cses.fi/problemset/task/1660) | Prefix Sums | Positive Array Subarray Sum
- [ ] [CSES 1661: Subarray Sums II](https://cses.fi/problemset/task/1661) | Prefix Sums | Subarray Sums with Negatives
- [ ] [CSES 1662: Subarray Divisibility](https://cses.fi/problemset/task/1662) | Math & Hashing | Modular Prefix Invariant

### Codeforces & AtCoder (Rating 800–1400)
- [ ] [Codeforces 4C: Registration System](https://codeforces.com/problemset/problem/4/C) | 1000 | Dynamic Name Collision Resolution
- [ ] [Codeforces 1360B: Honest Coach](https://codeforces.com/problemset/problem/1360/B) | 800 | Sorting & Adjacent Difference

---

## 6. Post-Session Active Recall Flashcards (15 Minutes)

<details>
<summary><strong>Flashcard 1: Why does `prefix_counts[0] = 1` need to be initialized before the loop in Subarray Sum Equals K?</strong></summary>

It accounts for subarrays that start precisely at index 0. If a prefix sum equals $K$ at index $i$, then `running_sum - k = 0`. Without initializing `prefix_counts[0] = 1`, that subarray would be omitted because no explicit prefix subtraction before index 0 was recorded.
</details>

<details>
<summary><strong>Flashcard 2: How can we achieve $O(N)$ time for finding the Longest Consecutive Sequence without sorting?</strong></summary>

Convert the array into a hash set. For each number $x$, check if $x - 1$ is in the set. If $x - 1$ exists, skip $x$ because it cannot be the beginning of a sequence. If $x - 1$ does not exist, $x$ is the start of a streak: iterate $x + 1, x + 2, \dots$ while present in the set. Each element is visited at most twice (once in the outer scan, once during a forward streak), resulting in $O(N)$ total time.
</details>

<details>
<summary><strong>Flashcard 3: In Python, why is a tuple `(0, 1, 0, ...)` commonly used as a dict key for Group Anagrams instead of a list?</strong></summary>

Python dictionary keys must be hashable and immutable. Lists are mutable and do not implement `__hash__`, raising a `TypeError`. Tuples are immutable and hashable, allowing a 26-element character count tuple to serve directly as a unique canonical key.
</details>

<details>
<summary><strong>Flashcard 4: What is the space-time tradeoff between bucket sorting and a min-heap for Top K Frequent Elements?</strong></summary>

- **Bucket Sort**: $O(N)$ time, $O(N)$ space. Uses frequencies as indices in an array of lists of length $N+1$.
- **Min-Heap of size K**: $O(N \log K)$ time, $O(N + K)$ space. Useful when $N$ is massive and streaming, and maintaining an entire frequency table in memory is constrained.
</details>

---

[← Back to Home](index.md) \| [Next Topic: Two Pointers →](02-two-pointers.md)
