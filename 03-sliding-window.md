---
layout: default
title: "Topic 03: Sliding Window"
---

# Topic 03: Sliding Window

[← Topic 02: Two Pointers](02-two-pointers.md) \| [Schedule](schedule.md) \| [Next: Stack & Monotonic Stack →](04-stack.md)

---

## Daily Timeboxing (1.5 – 2.0 Hours)
- **00:00 – 00:15 (15m) | Active Recall Pre-Test**: Answer the conceptual questions below before reading further.
- **00:15 – 00:40 (25m) | YouTube Conceptual Anchor**: Watch the curated video segment on dynamic window expansion and monotonic queues.
- **00:40 – 01:30 (50m) | Coding Deep Dive**: Pen-and-paper dry run of Minimum Window Substring & Longest Repeating Character Replacement.
- **01:30 – 01:45 (15m) | Post-Session Flashcards**: Complete the flashcard prompts from memory.

---

## 1. Active Recall Pre-Test (15 Minutes)
Attempt to answer these questions on paper before expanding the solutions.

<details>
<summary><strong>Prompt 1: What is the fundamental difference between the Two Pointers paradigm and the Sliding Window paradigm?</strong></summary>

While both maintain two pointers (`left` and `right`), **Two Pointers** typically moves inward from opposite ends of a sorted array to converge on a single pair or boundary, or modifies array elements in place.
**Sliding Window** always moves pointers in the same direction ($L \le R$) to track the continuous aggregate state of a contiguous subarray / substring $[L \dots R]$. The right pointer *expands* to capture elements, while the left pointer *contracts* to restore a violated window invariant.
</details>

<details>
<summary><strong>Prompt 2: In Longest Repeating Character Replacement, why does the window validity condition `(window_length - max_frequency) <= k` remain correct even if `max_frequency` is not decremented when shrinking the window?</strong></summary>

We only care about finding the **maximum** window length. 
A new maximum window can only be formed if we discover a state with an even larger `max_frequency`. 
If shrinking the window from the left causes the true maximum frequency inside the window to decrease, the window will simply not grow. The recorded `max_frequency` acts as a historical high-water mark. Because the answer is monotonic, stale values of `max_frequency` never produce false positives, preserving $O(N)$ time instead of scanning a 26-character frequency table on every contraction.
</details>

<details>
<summary><strong>Prompt 3: How does the "Exact K = At Most(K) - At Most(K - 1)" reduction simplify difficult subarray counting problems?</strong></summary>

Directly counting subarrays that contain *exactly* $K$ distinct elements with a sliding window is awkward because shrinking $L$ when the window has $K$ elements could either preserve or break the condition.
However, finding the number of subarrays with *at most* $K$ distinct elements is straightforward: whenever $[L \dots R]$ has $\le K$ distinct elements, every subarray ending at $R$ and starting anywhere from $L$ to $R$ is valid, adding exactly $(R - L + 1)$ to the count.
Using the algebraic identity:
$$\text{Count}(\text{Exact } K) = \text{Count}(\text{At Most } K) - \text{Count}(\text{At Most } K - 1)$$
we solve the problem with two clean, monotonic sliding window passes.
</details>

---

## 2. YouTube Conceptual Anchor (25 Minutes)

Recommended Video Anchors:
1. **NeetCode**: [Sliding Window / Minimum Window Substring Deep Dive](https://www.youtube.com/watch?v=jSto0O4AJbM) (15 mins)
2. **Errichto**: [Sliding Window Monotonic Queue / Deque Explanation](https://www.youtube.com/watch?v=0hYl0Zl_1c4) (18 mins)

### Core Invariants & Mental Model
1. **The Dynamic Window Invariant**: The window $[L \dots R]$ maintains a valid property (e.g., no duplicates, sum $\le K$, or character budget). $R$ expands to absorb new elements until the property is violated; $L$ contracts until validity is restored.
2. **Amortized $O(N)$ Invariant**: Although there is an inner `while` loop contracting $L$, both $L$ and $R$ increment at most $N$ times. Total pointer movements $\le 2N$, guaranteeing $O(N)$ overall time.
3. **Monotonic Deque Invariant (Sliding Window Maximum)**: The double-ended queue stores *indices* in strictly decreasing order of their corresponding values:
$$A[\text{deque}[0]] > A[\text{deque}[1]] > A[\text{deque}[2]] > \dots$$
The front of the deque always holds the maximum of the current window.

---

## 3. Canonical Code Templates

### Pattern A: Dynamic Window with Contraction ($O(N)$ Time, $O(K)$ Space)
```python
from typing import List

def min_sub_array_len(target: int, nums: List[int]) -> int:
    left = 0
    running_sum = 0
    min_length = float('inf')
    
    for right in range(len(nums)):
        running_sum += nums[right]
        
        # Contract left until invariant (running_sum < target) is restored
        while running_sum >= target:
            min_length = min(min_length, right - left + 1)
            running_sum -= nums[left]
            left += 1
            
    return min_length if min_length != float('inf') else 0
```

### Pattern B: Frequency Matching / Minimum Window ($O(N)$ Time, $O(1)$ Alphabet Space)
```python
from collections import Counter

def min_window(s: str, t: str) -> str:
    if not t or not s:
        return ""
        
    target_counts = Counter(t)
    window_counts = Counter()
    
    have, need = 0, len(target_counts)
    best_len = float('inf')
    best_bounds = (-1, -1)
    
    left = 0
    for right, char in enumerate(s):
        window_counts[char] += 1
        if char in target_counts and window_counts[char] == target_counts[char]:
            have += 1
            
        # Contract from left while window satisfies all requirements
        while have == need:
            window_len = right - left + 1
            if window_len < best_len:
                best_len = window_len
                best_bounds = (left, right)
                
            left_char = s[left]
            window_counts[left_char] -= 1
            if left_char in target_counts and window_counts[left_char] < target_counts[left_char]:
                have -= 1
            left += 1
            
    start, end = best_bounds
    return s[start:end + 1] if best_len != float('inf') else ""
```

### Pattern C: Monotonic Deque for Sliding Window Maximum ($O(N)$ Time, $O(K)$ Space)
```python
from collections import deque
from typing import List

def max_sliding_window(nums: List[int], k: int) -> List[int]:
    dq = deque()  # Stores indices; values are monotonically decreasing
    result = []
    
    for i, num in enumerate(nums):
        # 1. Evict elements outside the current window [i - k + 1, i]
        if dq and dq[0] < i - k + 1:
            dq.popleft()
            
        # 2. Maintain monotonic decreasing order by popping smaller elements
        while dq and nums[dq[-1]] <= num:
            dq.pop()
            
        dq.append(i)
        
        # 3. Record maximum once the first full window of size k is formed
        if i >= k - 1:
            result.append(nums[dq[0]])
            
    return result
```

---

## 4. Coding Deep Dive: Worked Walkthrough (50 Minutes)

### Problem: Minimum Window Substring (LeetCode 76)
**Problem Statement**: Given two strings `s` and `t`, return the minimum window substring of `s` such that every character in `t` (including duplicates) is included in the window.

#### Pen-and-Paper Dry Run Trace
Let `s = "ADOBECODEBANC"`, `t = "ABC"`.
Target requirements: `need = 3` (distinct keys: `'A': 1, 'B': 1, 'C': 1`).
Initialize `have = 0`, `left = 0`, `best_len = inf`.

| Step | `right` | `s[right]` | `have` / `need` | Action / `left` Contraction | Window Candidate | Current Best |
|:---:|:---:|:---:|:---:|:---|:---:|:---:|
| 1 | 0 | 'A' | 1 / 3 | Incomplete | - | - |
| 2 | 3 | 'B' | 2 / 3 | Incomplete | - | - |
| 3 | 5 | 'C' | 3 / 3 | **Valid!** Contract $L$: $L=0$ ('A' removed $\implies have=2$). Advance $L \to 1$. | `"ADOBEC"` (len 6) | `"ADOBEC"` (len 6) |
| 4 | 9 | 'B' | 2 / 3 | Incomplete | - | `"ADOBEC"` (len 6) |
| 5 | 10 | 'A' | 3 / 3 | **Valid!** Contract $L$: $L=1$ ('D'), $L=2$ ('O'), $L=3$ ('B' $\implies have=2$). | `"BECODEBA"` (len 8) | `"ADOBEC"` (len 6) |
| 6 | 12 | 'C' | 3 / 3 | **Valid!** Contract $L$: $L=4 \dots 9$: reaches $L=9$ ('B'), $L=10$ ('A'), $L=11$ ('N'). | `"BANC"` (len 4) | **`"BANC"` (len 4)** |

Optimal Window: `"BANC"` (length 4).
Time Complexity: $O(|S| + |T|)$. Each character in $S$ is visited at most twice ($L$ and $R$).
Space Complexity: $O(|\Sigma|)$ where $\Sigma$ is the character set.

---

## 5. Curated Problem Set (15 Problems)

Track your daily solutions by checking off the markdown boxes.

### NeetCode / LeetCode (Core)
- [ ] [LeetCode 121: Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/) | Easy | Min-Price Tracking Window
- [ ] [LeetCode 3: Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/) | Medium | Last Seen Index Window Jump
- [ ] [LeetCode 424: Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/) | Medium | Max Frequency Invariant
- [ ] [LeetCode 567: Permutation in String](https://leetcode.com/problems/permutation-in-string/) | Medium | Fixed Window Frequency Matching
- [ ] [LeetCode 76: Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/) | Hard | Deficit Counter Contracting Window
- [ ] [LeetCode 239: Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/) | Hard | Monotonic Deque
- [ ] [LeetCode 209: Minimum Size Subarray Sum](https://leetcode.com/problems/minimum-size-subarray-sum/) | Medium | Positive Sum Dynamic Contraction
- [ ] [LeetCode 713: Subarray Product Less Than K](https://leetcode.com/problems/subarray-product-less-than-k/) | Medium | Product Boundary Counting ($R - L + 1$)
- [ ] [LeetCode 1004: Max Consecutive Ones III](https://leetcode.com/problems/max-consecutive-ones-iii/) | Medium | Zero-Budget Window Expansion
- [ ] [LeetCode 992: Subarrays with K Different Integers](https://leetcode.com/problems/subarrays-with-k-different-integers/) | Hard | AtMost(K) - AtMost(K-1) Reduction

### CSES Problem Set (Free Tier, No Login Wall)
- [ ] [CSES 1076: Sliding Window Median](https://cses.fi/problemset/task/1076) | Sorting & Searching | Dual Multiset / Two Heaps with Lazy Deletion
- [ ] [CSES 1077: Sliding Window Cost](https://cses.fi/problemset/task/1077) | Sorting & Searching | Median Absolute Deviations
- [ ] [CSES 1660: Subarray Sums I](https://cses.fi/problemset/task/1660) | Prefix Sums | Monotonic Two-Pointer Window for Positive Values

### Codeforces & AtCoder (Rating 800–1400)
- [ ] [Codeforces 600B: Queries about less or equal elements](https://codeforces.com/problemset/problem/600/B) | 1200 | Two Pointers / Binary Search Range
- [ ] [AtCoder ABC130_D: Enough Array](https://atcoder.jp/contests/abc130/tasks/abc130_d) | 1000 | Dynamic Window Subarray Counting

---

## 6. Post-Session Active Recall Flashcards (15 Minutes)

<details>
<summary><strong>Flashcard 1: When computing subarrays with a sum or property bounded by a window, why does adding `(right - left + 1)` correctly count all valid subarrays?</strong></summary>

Any contiguous subarray ending at index `right` must start at some index $k \in [\text{left}, \text{right}]$. 
If the entire window from `left` to `right` satisfies the monotonic property (e.g., all values are positive and sum $\le K$), then every smaller sub-segment ending at `right` with start $k \ge \text{left}$ must also satisfy the property. There are exactly $(\text{right} - \text{left} + 1)$ such start positions.
</details>

<details>
<summary><strong>Flashcard 2: In the Monotonic Deque implementation of Sliding Window Maximum, why do we store indices rather than the array values themselves?</strong></summary>

Storing indices allows $O(1)$ expiration verification: an element has fallen out of the window if and only if `dq[0] < i - k + 1`. If we only stored values, we would not know whether a duplicate value at the front belongs to the current window or a previous one.
</details>

<details>
<summary><strong>Flashcard 3: What happens to the time complexity if we try to solve Longest Substring Without Repeating Characters using a set vs storing the last seen index?</strong></summary>

Both achieve $O(N)$ amortized time. 
With a hash set, `left` increments one index at a time ($O(2N)$ pointer movements). 
With a hash map storing the `last_seen_index`, `left` can jump directly to `last_seen_index[char] + 1`, reducing the number of pointer increments to at most $N$.
</details>

---

[← Topic 02: Two Pointers](02-two-pointers.md) \| [Next Topic: Stack & Monotonic Stack →](04-stack.md)
