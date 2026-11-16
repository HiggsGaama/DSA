---
layout: default
title: "Topic 05: Binary Search & Search Space Reduction"
---

# Topic 05: Binary Search & Search Space Reduction

[← Topic 04: Stack](04-stack.md) \| [Schedule](schedule.md) \| [Next: Linked Lists →](06-linked-lists.md)

---

## Daily Timeboxing (1.5 – 2.0 Hours)
- **00:00 – 00:15 (15m) | Active Recall Pre-Test**: Answer the conceptual questions below before reading further.
- **00:15 – 00:40 (25m) | YouTube Conceptual Anchor**: Watch the curated video segment on binary search invariants and monotonic predicates.
- **00:40 – 01:30 (50m) | Coding Deep Dive**: Pen-and-paper dry run of Koko Eating Bananas & Search in Rotated Sorted Array.
- **01:30 – 01:45 (15m) | Post-Session Flashcards**: Complete the flashcard prompts from memory.

---

## 1. Active Recall Pre-Test (15 Minutes)
Attempt to answer these questions on paper before expanding the solutions.

<details>
<summary><strong>Prompt 1: What is the exact mathematical condition required to perform binary search, and why is "the array must be sorted" an incomplete answer?</strong></summary>

The general requirement for binary search is **monotonicity of a boolean predicate function $P(x)$ over a domain $D$**.
The domain does not have to be a sorted array of numbers; it only requires that evaluating $P(x)$ yields a monotonic sequence of boolean values, typically:
$$\underbrace{[\text{False}, \text{False}, \dots, \text{False}]}_{\text{Invalid prefix}}, \quad \underbrace{[\text{True}, \text{True}, \dots, \text{True}]}_{\text{Valid suffix}}$$
Binary search finds the exact transition boundary (the first `True` or last `False`) in $O(\log |D|)$ evaluations of $P(x)$. This applies directly to search spaces such as speed, capacity, or time (Binary Search on Answer).
</details>

<details>
<summary><strong>Prompt 2: In a rotated sorted array with unique elements, how do you determine which half is normally ordered?</strong></summary>

Compare `nums[mid]` against `nums[left]`:
- If `nums[mid] >= nums[left]`: The left half `[left ... mid]` is guaranteed to be strictly sorted.
- If `nums[mid] < nums[left]`: The rotation pivot lies within the left half, which means the right half `[mid ... right]` is guaranteed to be strictly sorted.
Once you identify the normally ordered half, check if the target falls within its range (`nums[left] <= target <= nums[mid]` or `nums[mid] <= target <= nums[right]`). If it does, eliminate the other half; if not, discard the sorted half.
</details>

<details>
<summary><strong>Prompt 3: Why does `mid = left + (right - left) // 2` prevent integer overflow in languages like C++ and Java, and why is `(left + right) // 2` dangerous?</strong></summary>

If `left` and `right` are large positive 32-bit signed integers (e.g., $1.5 \times 10^9$), their sum `left + right` equals $3.0 \times 10^9$, exceeding the maximum 32-bit signed integer value ($2^{31} - 1 \approx 2.14 \times 10^9$) and overflowing into a negative number.
The identity:
$$\text{mid} = \text{left} + \frac{\text{right} - \text{left}}{2}$$
ensures that the subtraction $\text{right} - \text{left}$ stays $\ge 0$ and never exceeds the domain range, completely eliminating arithmetic overflow.
</details>

---

## 2. YouTube Conceptual Anchor (25 Minutes)

Recommended Video Anchors:
1. **Errichto**: [Binary Search Tutorial and Common Pitfalls](https://www.youtube.com/watch?v=GU7DpgHINWQ) (20 mins)
2. **NeetCode**: [Binary Search / Koko Eating Bananas Explained](https://www.youtube.com/watch?v=U2Qp5JMbehY) (12 mins)

### Core Invariants & Mental Model
1. **The Invariant Interval**: At every iteration, the search target (or optimal boundary) is guaranteed to lie within the closed interval $[L, R]$.
2. **Strict Search Space Reduction**: Every step must strictly shrink the interval ($R - L$ strictly decreases) to avoid infinite loops when $R - L = 1$.
3. **Binary Search on the Answer**: When asked to "minimize the maximum" or "maximize the minimum", define a feasibility function `can_achieve(x) -> bool`. If `can_achieve(x)` is monotonic, binary search over the numerical range $[x_{\min}, x_{\max}]$.

---

## 3. Canonical Code Templates

### Pattern A: Standard Exact Match ($O(\log N)$ Time, $O(1)$ Space)
```python
from typing import List

def binary_search(nums: List[int], target: int) -> int:
    left, right = 0, len(nums) - 1
    
    while left <= right:
        mid = left + (right - left) // 2
        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
            
    return -1
```

### Pattern B: Lower Bound / First `True` ($O(\log N)$ Time, $O(1)$ Space)
```python
from typing import List, Callable

def binary_search_first_true(low: int, high: int, predicate: Callable[[int], bool]) -> int:
    # Invariant: Answer always lies in [low, high]
    while low < high:
        mid = low + (high - low) // 2
        if predicate(mid):
            high = mid      # mid could be the first True; retain it
        else:
            low = mid + 1   # mid is False; discard it
            
    return low
```

### Pattern C: Search Space Reduction (Koko Eating Bananas — $O(N \log M)$ Time)
```python
import math
from typing import List

def min_eating_speed(piles: List[int], h: int) -> int:
    def can_eat_all(speed: int) -> bool:
        # Predicate: Can Koko eat all bananas at this speed within h hours?
        hours_needed = sum(math.ceil(p / speed) for p in piles)
        return hours_needed <= h

    low = 1
    high = max(piles)
    
    while low < high:
        mid = low + (high - low) // 2
        if can_eat_all(mid):
            high = mid
        else:
            low = mid + 1
            
    return low
```

---

## 4. Coding Deep Dive: Worked Walkthrough (50 Minutes)

### Problem: Koko Eating Bananas (LeetCode 875)
**Problem Statement**: Koko loves to eat bananas. There are `n` piles of bananas, the $i$-th pile has `piles[i]` bananas. The guards will return in `h` hours. Return the minimum integer `k` such that she can eat all the bananas within `h` hours.

#### Pen-and-Paper Dry Run Trace
Let `piles = [3, 6, 7, 11]`, `h = 8`.
Range of possible speeds: `low = 1`, `high = max(piles) = 11`.
Formula for hours at speed $k$: $\sum \lceil \text{pile} / k \rceil$.

| Iteration | `low` | `high` | `mid` (speed) | Hours Calculation: $\sum \lceil p / \text{mid} \rceil$ | $\le 8$? (Predicate) | Next Interval |
|:---:|:---:|:---:|:---:|:---|:---:|:---:|
| 1 | 1 | 11 | 6 | $\lceil 3/6 \rceil + \lceil 6/6 \rceil + \lceil 7/6 \rceil + \lceil 11/6 \rceil = 1 + 1 + 2 + 2 = 6$ | **True** ($6 \le 8$) | `high = 6` |
| 2 | 1 | 6 | 3 | $\lceil 3/3 \rceil + \lceil 6/3 \rceil + \lceil 7/3 \rceil + \lceil 11/3 \rceil = 1 + 2 + 3 + 4 = 10$ | **False** ($10 > 8$) | `low = 4` |
| 3 | 4 | 6 | 5 | $\lceil 3/5 \rceil + \lceil 6/5 \rceil + \lceil 7/5 \rceil + \lceil 11/5 \rceil = 1 + 2 + 2 + 3 = 8$ | **True** ($8 \le 8$) | `high = 5` |
| 4 | 4 | 5 | 4 | $\lceil 3/4 \rceil + \lceil 6/4 \rceil + \lceil 7/4 \rceil + \lceil 11/4 \rceil = 1 + 2 + 2 + 3 = 8$ | **True** ($8 \le 8$) | `high = 4` |
| End | 4 | 4 | - | `low == high == 4` | - | Return **4** |

Result: Minimum eating speed is **4**.
Time Complexity: $O(N \log(\max(\text{piles})))$ where $N = 4, \max = 11$.
Space Complexity: $O(1)$.

---

## 5. Curated Problem Set (16 Problems)

Track your daily solutions by checking off the markdown boxes.

### NeetCode / LeetCode (Core)
- [ ] [LeetCode 704: Binary Search](https://leetcode.com/problems/binary-search/) | Easy | Standard Exact Match Template
- [ ] [LeetCode 74: Search a 2D Matrix](https://leetcode.com/problems/search-a-2d-matrix/) | Medium | Matrix Coordinate Flattening `(mid // cols, mid % cols)`
- [ ] [LeetCode 153: Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/) | Medium | Rotation Inflection Point
- [ ] [LeetCode 33: Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/) | Medium | Sorted Half Identification & Range Check
- [ ] [LeetCode 34: Find First and Last Position of Element in Sorted Array](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/) | Medium | Lower Bound & Upper Bound Delineation
- [ ] [LeetCode 875: Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/) | Medium | Binary Search on Answer / Eating Rate
- [ ] [LeetCode 1011: Capacity To Ship Packages Within D Days](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/) | Medium | Ship Capacity Predicate Function
- [ ] [LeetCode 410: Split Array Largest Sum](https://leetcode.com/problems/split-array-largest-sum/) | Hard | Max Subarray Sum Binary Search
- [ ] [LeetCode 981: Time Based Key-Value Store](https://leetcode.com/problems/time-based-key-value-store/) | Medium | Timestamp Range Search with Binary Search
- [ ] [LeetCode 4: Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/) | Hard | Dual Array Partitioning Invariant

### CSES Problem Set (Free Tier, No Login Wall)
- [ ] [CSES 1074: Stick Lengths](https://cses.fi/problemset/task/1074) | Sorting & Searching | Median Optimization / Ternary Search
- [ ] [CSES 1620: Factory Machines](https://cses.fi/problemset/task/1620) | Sorting & Searching | Monotonic Production Rate Predicate
- [ ] [CSES 1085: Array Division](https://cses.fi/problemset/task/1085) | Sorting & Searching | Partitioning into K Subarrays with Minimized Max Sum

### Codeforces & AtCoder (Rating 800–1400)
- [ ] [Codeforces 706B: Interesting drink](https://codeforces.com/problemset/problem/706/B) | 1100 | Upper Bound Coin Budget Queries
- [ ] [Codeforces 778A: String Game](https://codeforces.com/problemset/problem/778/A) | 1400 | Binary Search on Deletion Operations
- [ ] [AtCoder ABC248_C: Dice Sum](https://atcoder.jp/contests/abc248/tasks/abc248_c) | 1000 | Monotonic Range Validation

---

## 6. Post-Session Active Recall Flashcards (15 Minutes)

<details>
<summary><strong>Flashcard 1: When using the `while low < high` template, how do you prevent an infinite loop when updating `low = mid` vs `high = mid`?</strong></summary>

If the update rule is `low = mid`, when `high - low == 1`, standard integer division calculates `mid = low`. Because `low` remains unchanged, the loop runs infinitely.
To fix this, bias `mid` toward the upper boundary:
`mid = low + (high - low + 1) // 2`.
Conversely, if the update rule is `high = mid`, standard division `mid = low + (high - low) // 2` is safe because `mid` strictly equals `low < high`, ensuring `high` decreases.
</details>

<details>
<summary><strong>Flashcard 2: How do you map a 1D binary search index `mid` to a 2D matrix of dimensions $M \times N$?</strong></summary>

Given total elements $K = M \times N$ and index $\text{mid} \in [0, K - 1]$:
$$\text{row} = \text{mid} // N$$
$$\text{col} = \text{mid} \% N$$
where $N$ is the number of columns.
</details>

<details>
<summary><strong>Flashcard 3: What is the optimal time complexity of finding the median of two sorted arrays of lengths $M$ and $N$?</strong></summary>

$O(\log(\min(M, N)))$. We perform binary search on the partition cut of the *smaller* array. Once the cut on the smaller array is chosen, the required cut on the larger array is determined arithmetically to ensure equal total elements on both sides.
</details>

---

[← Topic 04: Stack](04-stack.md) \| [Next Topic: Linked Lists →](06-linked-lists.md)
