---
layout: default
title: "Topic 02: Two Pointers"
---

# Topic 02: Two Pointers

[← Topic 01: Arrays & Hashing](01-arrays-and-hashing.md) \| [Schedule](schedule.md) \| [Next: Sliding Window →](03-sliding-window.md)

---

## Daily Timeboxing (1.5 – 2.0 Hours)
- **00:00 – 00:15 (15m) | Active Recall Pre-Test**: Answer the conceptual questions below before reading further.
- **00:15 – 00:40 (25m) | YouTube Conceptual Anchor**: Watch the curated video segment on sorted search space monotonicity.
- **00:40 – 01:30 (50m) | Coding Deep Dive**: Pen-and-paper dry run of Container With Most Water & Trapping Rain Water.
- **01:30 – 01:45 (15m) | Post-Session Flashcards**: Complete the flashcard prompts from memory.

---

## 1. Active Recall Pre-Test (15 Minutes)
Attempt to answer these questions on paper before expanding the solutions.

<details>
<summary><strong>Prompt 1: What mathematical property of a sorted array enables the two-pointer technique to solve Two Sum in $O(N)$ time and $O(1)$ space?</strong></summary>

Monotonicity. In a sorted array, moving the `left` pointer to the right (`left += 1`) strictly non-decreases the sum ($A[\text{left}] + A[\text{right}]$), while moving the `right` pointer to the left (`right -= 1`) strictly non-increases the sum. 
Because the directional response to every comparison ($< \text{target}$ or $> \text{target}$) is monotonic and unambiguous, we eliminate an entire row or column of candidate pairs at each step without ever needing to evaluate them.
</details>

<details>
<summary><strong>Prompt 2: In Container With Most Water, why is it always optimal to move the pointer pointing to the shorter line?</strong></summary>

The volume (area) between two boundaries at indices $L$ and $R$ is given by:
$$\text{Area} = (R - L) \times \min(H[L], H[R])$$
As we move inward, the width $(R - L)$ strictly decreases by 1. 
If we were to move the pointer pointing to the *taller* line, the height of the new container would remain bottlenecked by the shorter line ($\min(\text{shorter}, H_{\text{new}}) \le \text{shorter}$). Thus, the area can only stay the same or decrease. 
The *only* mathematical possibility of obtaining a larger area is to move the pointer pointing to the shorter line in the hope of finding a significantly taller boundary that compensates for the decreased width.
</details>

<details>
<summary><strong>Prompt 3: How does the two-pointer approach for Trapping Rain Water eliminate the $O(N)$ auxiliary space required by prefix/suffix max arrays?</strong></summary>

The water trapped at index $i$ depends on $\min(\text{max\_left}[i], \text{max\_right}[i]) - H[i]$.
Using two pointers $L$ and $R$ starting at opposite ends:
- If `max_left < max_right`, we know with certainty that the water trapped at $L$ is bounded by `max_left`, regardless of whatever higher peaks might exist between $L$ and $R$. Thus, we can immediately calculate trapped water at $L$ and increment $L$.
- Conversely, if `max_right <= max_left`, water at $R$ is strictly bounded by `max_right`, so we calculate water at $R$ and decrement $R$.
</details>

---

## 2. YouTube Conceptual Anchor (25 Minutes)

Recommended Video Anchors:
1. **NeetCode**: [Two Pointers / 3Sum Intuition & Trapping Rain Water](https://www.youtube.com/watch?v=jzZsG8n2R9A) (15 mins)
2. **Striver**: [Two Pointer & Greedy Paradigm Overview](https://www.youtube.com/watch?v=1pkOGXXUtxk) (16 mins)

### Core Invariants & Mental Model
1. **Opposing Pointers Invariant ($L \to \dots \leftarrow R$)**: Used when a global metric changes monotonically as boundaries contract from the extremities.
2. **Parallel / Fast-Slow Invariant ($S \to F \to$)**: Used for in-place array compaction (e.g., removing duplicates) or cycle detection. The slow pointer $S$ demarcates the boundary of already-processed valid elements, while $F$ explores unprocessed values.
3. **Fixed Pivot Reduction ($O(N^2)$ for 3Sum)**: Fix the first element at index $i$ with a single loop. Then use two opposing pointers on the remaining subarray $i+1 \dots N-1$ to find the two-element complement $-(A[i])$. Always skip duplicate values for both $i$, $L$, and $R$ to maintain distinct triplets.

---

## 3. Canonical Code Templates

### Pattern A: Opposing Two Pointers (Two Sum II — $O(N)$ Time, $O(1)$ Space)
```python
from typing import List

def two_sum_sorted(numbers: List[int], target: int) -> List[int]:
    left = 0
    right = len(numbers) - 1
    
    while left < right:
        current_sum = numbers[left] + numbers[right]
        if current_sum == target:
            return [left + 1, right + 1]  # 1-indexed return
        elif current_sum < target:
            left += 1
        else:
            right -= 1
            
    return []
```

### Pattern B: Fast-Slow In-Place Compaction ($O(N)$ Time, $O(1)$ Space)
```python
from typing import List

def remove_duplicates(nums: List[int]) -> int:
    if not nums:
        return 0
        
    slow = 0  # Invariant: nums[0...slow] contains unique elements in sorted order
    
    for fast in range(1, len(nums)):
        if nums[fast] != nums[slow]:
            slow += 1
            nums[slow] = nums[fast]
            
    return slow + 1
```

### Pattern C: Trapping Rain Water Two-Pointer Invariant ($O(N)$ Time, $O(1)$ Space)
```python
from typing import List

def trap_rain_water(height: List[int]) -> int:
    if not height:
        return 0
        
    left, right = 0, len(height) - 1
    max_left, max_right = height[left], height[right]
    trapped_water = 0
    
    while left < right:
        if max_left < max_right:
            left += 1
            max_left = max(max_left, height[left])
            trapped_water += max_left - height[left]
        else:
            right -= 1
            max_right = max(max_right, height[right])
            trapped_water += max_right - height[right]
            
    return trapped_water
```

---

## 4. Coding Deep Dive: Worked Walkthrough (50 Minutes)

### Problem: 3Sum (LeetCode 15)
**Problem Statement**: Given an integer array `nums`, return all unique triplets `[nums[i], nums[j], nums[k]]` such that `i != j != k` and `nums[i] + nums[j] + nums[k] == 0`. The solution set must not contain duplicate triplets.

#### Pen-and-Paper Dry Run Trace
Let `nums = [-1, 0, 1, 2, -1, -4]`.
1. Sort array: `nums = [-4, -1, -1, 0, 1, 2]`. Length $N = 6$.
2. Outer loop `i` from 0 to $N-3$:

| Step | `i` | `nums[i]` | `left` | `right` | `nums[left] + nums[right]` | Target (`-nums[i]`) | Action Taken | Triplets Recorded |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---|:---|
| 1 | 0 | -4 | 1 (-1) | 5 (2) | 1 | 4 | $1 < 4 \implies \text{left}++$ | `[]` |
| 2 | 0 | -4 | 2 (-1) | 5 (2) | 1 | 4 | $1 < 4 \implies \text{left}++$ | `[]` |
| 3 | 0 | -4 | 4 (1) | 5 (2) | 3 | 4 | $3 < 4 \implies \text{left}++$ (stops) | `[]` |
| 4 | 1 | -1 | 2 (-1) | 5 (2) | 1 | 1 | **Match!** Record, advance both, skip duplicates | `[[-1, -1, 2]]` |
| 5 | 1 | -1 | 3 (0) | 4 (1) | 1 | 1 | **Match!** Record, advance both | `[[-1, -1, 2], [-1, 0, 1]]` |
| 6 | 2 | -1 | - | - | - | - | `nums[2] == nums[1]`, **Skip duplicate pivot** | - |
| 7 | 3 | 0 | 4 (1) | 5 (2) | 3 | 0 | $3 > 0 \implies \text{right}--$ (stops) | - |

Result: `[[-1, -1, 2], [-1, 0, 1]]`.
Time Complexity: $O(N \log N)$ sort + $O(N^2)$ two pointers = $O(N^2)$.
Space Complexity: $O(1)$ auxiliary (or $O(N)$ depending on sorting implementation).

---

## 5. Curated Problem Set (15 Problems)

Track your daily solutions by checking off the markdown boxes.

### NeetCode / LeetCode (Core)
- [ ] [LeetCode 125: Valid Palindrome](https://leetcode.com/problems/valid-palindrome/) | Easy | Alphanumeric Filtering & Converging Pointers
- [ ] [LeetCode 167: Two Sum II - Input Array Is Sorted](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/) | Medium | Monotonic Sum Boundary Movement
- [ ] [LeetCode 15: 3Sum](https://leetcode.com/problems/3sum/) | Medium | Sorted Pivot Fixation & Duplicate Pruning
- [ ] [LeetCode 11: Container With Most Water](https://leetcode.com/problems/container-with-most-water/) | Medium | Area Minimizing Height Elimination
- [ ] [LeetCode 42: Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/) | Hard | Min-Height Boundary Invariant
- [ ] [LeetCode 26: Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/) | Easy | Fast-Slow Pointer Overwriting
- [ ] [LeetCode 75: Sort Colors](https://leetcode.com/problems/sort-colors/) | Medium | Dutch National Flag (3-Way Partitioning)
- [ ] [LeetCode 16: 3Sum Closest](https://leetcode.com/problems/3sum-closest/) | Medium | Absolute Difference Tracking
- [ ] [LeetCode 18: 4Sum](https://leetcode.com/problems/4sum/) | Medium | Generalized K-Sum Recursion / Pointers
- [ ] [LeetCode 392: Is Subsequence](https://leetcode.com/problems/is-subsequence/) | Easy | Two Pointers Single-Pass Matching

### CSES Problem Set (Free Tier, No Login Wall)
- [ ] [CSES 1640: Sum of Two Values](https://cses.fi/problemset/task/1640) | Sorting & Searching | Two Pointers on Sorted Value-Index Pairs
- [ ] [CSES 1084: Apartments](https://cses.fi/problemset/task/1084) | Sorting & Searching | Greedy Two-Pointer Matching with Tolerance
- [ ] [CSES 1090: Ferris Wheel](https://cses.fi/problemset/task/1090) | Sorting & Searching | Pairing Heaviest with Lightest
- [ ] [CSES 1641: Sum of Three Values](https://cses.fi/problemset/task/1641) | Sorting & Searching | 3Sum with 1-based Index Output

### Codeforces & AtCoder (Rating 800–1400)
- [ ] [Codeforces 381A: Sereja and Dima](https://codeforces.com/problemset/problem/381/A) | 800 | Greedy Pick from Outer Extremities
- [ ] [Codeforces 279B: Books](https://codeforces.com/problemset/problem/279/B) | 1100 | Two Pointers / Sliding Window Sum Bound

---

## 6. Post-Session Active Recall Flashcards (15 Minutes)

<details>
<summary><strong>Flashcard 1: In the Dutch National Flag algorithm (Sort Colors), what are the three invariants maintained by pointers `low`, `mid`, and `high`?</strong></summary>

1. `nums[0 ... low - 1]` contains strictly `0`s.
2. `nums[low ... mid - 1]` contains strictly `1`s.
3. `nums[high + 1 ... N - 1]` contains strictly `2`s.
The unexamined subarray is `nums[mid ... high]`.
- If `nums[mid] == 0`: swap `nums[low]` and `nums[mid]`, then `low += 1, mid += 1`.
- If `nums[mid] == 1`: simply advance `mid += 1`.
- If `nums[mid] == 2`: swap `nums[mid]` and `nums[high]`, then `high -= 1` (do NOT advance `mid` because the swapped-in value has not yet been inspected).
</details>

<details>
<summary><strong>Flashcard 2: How do you guarantee zero duplicate triplets in 3Sum without allocating a hash set?</strong></summary>

1. Sort the array first.
2. For the outer loop `i`: if `i > 0 and nums[i] == nums[i - 1]`, `continue`.
3. Inside the two-pointer loop, after finding a match `nums[i] + nums[L] + nums[R] == 0`:
   - Increment `L` while `L < R and nums[L] == nums[L + 1]`.
   - Decrement `R` while `L < R and nums[R] == nums[R - 1]`.
   - Finally, advance `L += 1` and `R -= 1`.
</details>

<details>
<summary><strong>Flashcard 3: What is the primary indicator that a problem can be solved with two pointers rather than a hash table?</strong></summary>

The problem exhibits monotonic movement over an ordered (or sortable) linear structure, and requires $O(1)$ auxiliary space. Hash tables trade $O(N)$ memory to enable arbitrary non-ordered lookups; two pointers exploit the geometric structure of sorted order to search without extra memory.
</details>

---

[← Topic 01: Arrays & Hashing](01-arrays-and-hashing.md) \| [Next Topic: Sliding Window →](03-sliding-window.md)
