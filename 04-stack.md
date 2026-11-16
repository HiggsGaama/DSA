---
layout: default
title: "Topic 04: Stack & Monotonic Stack"
---

# Topic 04: Stack & Monotonic Stack

[← Topic 03: Sliding Window](03-sliding-window.md) \| [Schedule](schedule.md) \| [Next: Binary Search →](05-binary-search.md)

---

## Daily Timeboxing (1.5 – 2.0 Hours)
- **00:00 – 00:15 (15m) | Active Recall Pre-Test**: Answer the conceptual questions below before reading further.
- **00:15 – 00:40 (25m) | YouTube Conceptual Anchor**: Watch the curated video segment on monotonic stack invariants.
- **00:40 – 01:30 (50m) | Coding Deep Dive**: Pen-and-paper dry run of Largest Rectangle in Histogram & Daily Temperatures.
- **01:30 – 01:45 (15m) | Post-Session Flashcards**: Complete the flashcard prompts from memory.

---

## 1. Active Recall Pre-Test (15 Minutes)
Attempt to answer these questions on paper before expanding the solutions.

<details>
<summary><strong>Prompt 1: What is a Monotonic Stack, and what specific problem archetype does it solve in $O(N)$ time instead of $O(N^2)$?</strong></summary>

A Monotonic Stack is a stack whose elements are strictly monotonically increasing or monotonically decreasing from bottom to top.
It is the optimal data structure for the **Next/Previous Greater/Smaller Element** archetype. Whenever we need to find the first element to the left or right that is larger or smaller than the current element, a monotonic stack eliminates redundant comparisons by popping elements that can no longer serve as an answer for any future query.
</details>

<details>
<summary><strong>Prompt 2: In Largest Rectangle in Histogram, when an element is popped from the stack, how are the left and right boundaries of its maximum rectangle determined?</strong></summary>

Let the stack maintain indices of strictly increasing heights:
- **Right Boundary**: When an incoming bar at index $i$ has height strictly less than the bar at stack top, index $i$ is the *first smaller bar to the right*. It acts as the right bounding fence for the popped bar.
- **Left Boundary**: After popping index $p$, the *new* element at the top of the stack is the *first smaller bar to the left*. If the stack becomes empty, no bar to the left was smaller, so the left boundary extends all the way to index $-1$.
- **Width**: $\text{width} = i - \text{stack\_top} - 1$.
</details>

<details>
<summary><strong>Prompt 3: How does Min Stack achieve $O(1)$ retrieval of the minimum element without scanning the stack?</strong></summary>

By maintaining either:
1. An auxiliary `min_stack` where each entry is $\min(\text{current\_val}, \text{min\_stack[-1]})$.
2. Storing pairs `(val, current_min)` on a single stack.
Because LIFO order guarantees that an element popped was pushed after all elements below it, the minimum of the stack at any given depth never changes until that layer is popped.
</details>

---

## 2. YouTube Conceptual Anchor (25 Minutes)

Recommended Video Anchors:
1. **NeetCode**: [Largest Rectangle in Histogram Detailed Walkthrough](https://www.youtube.com/watch?v=zx5SwR13044) (15 mins)
2. **Striver**: [Monotonic Stack / Next Greater Element Pattern](https://www.youtube.com/watch?v=Du881K7Jtk8) (18 mins)

### Core Invariants & Mental Model
1. **The Elimination Invariant**: If element $B$ comes after element $A$ and is taller/better than $A$, element $A$ will never be the "next greater" answer for any subsequent elements. $A$ is permanently dominated and can be discarded.
2. **Sentinel Invariant in Histograms**: Adding a dummy height of $0$ at the end of the heights array forces all remaining elements on the stack to be resolved and popped without requiring separate cleanup code.
3. **Amortized Analysis**: Every element is pushed onto the stack exactly once and popped at most once. Total push/pop operations $\le 2N$, proving $O(N)$ runtime.

---

## 3. Canonical Code Templates

### Pattern A: Next Greater Element (Monotonic Decreasing Stack — $O(N)$ Time)
```python
from typing import List

def daily_temperatures(temperatures: List[int]) -> List[int]:
    n = len(temperatures)
    answer = [0] * n
    stack = []  # Stores indices of temperatures in decreasing order
    
    for current_day, temp in enumerate(temperatures):
        # Resolve any cooler days waiting on the stack
        while stack and temperatures[stack[-1]] < temp:
            prev_day = stack.pop()
            answer[prev_day] = current_day - prev_day
            
        stack.append(current_day)
        
    return answer
```

### Pattern B: Largest Rectangle in Histogram ($O(N)$ Time, $O(N)$ Space)
```python
from typing import List

def largest_rectangle_area(heights: List[int]) -> int:
    stack = []  # Stores indices of strictly increasing heights
    max_area = 0
    # Append 0 sentinel to flush all remaining elements at the end
    augmented_heights = heights + [0]
    
    for i, h in enumerate(augmented_heights):
        while stack and augmented_heights[stack[-1]] > h:
            popped_idx = stack.pop()
            height = augmented_heights[popped_idx]
            # If stack is empty, width extends from 0 to i
            width = i if not stack else (i - stack[-1] - 1)
            max_area = max(max_area, height * width)
            
        stack.append(i)
        
    return max_area
```

---

## 4. Coding Deep Dive: Worked Walkthrough (50 Minutes)

### Problem: Largest Rectangle in Histogram (LeetCode 84)
**Problem Statement**: Given an array of integers `heights` representing the histogram's bar height where the width of each bar is 1, return the area of the largest rectangle in the histogram.

#### Pen-and-Paper Dry Run Trace
Let `heights = [2, 1, 5, 6, 2, 3]`.
With sentinel: `[2, 1, 5, 6, 2, 3, 0]`.

| Step `i` | Height `h` | Stack State (Indices) | Action / Popped Index | Height $\times$ Width Calculation | Max Area |
|:---:|:---:|:---:|:---|:---|:---:|
| 0 | 2 | `[0]` | Push 0 | - | 0 |
| 1 | 1 | `[]` $\to$ `[1]` | Pop 0 (H=2) | $\text{width} = 1 - (-1) - 1 = 1 \implies 2 \times 1 = 2$ | 2 |
| 2 | 5 | `[1, 2]` | Push 2 | - | 2 |
| 3 | 6 | `[1, 2, 3]` | Push 3 | - | 2 |
| 4 | 2 | `[1, 2]` | Pop 3 (H=6) | $\text{width} = 4 - 2 - 1 = 1 \implies 6 \times 1 = 6$ | 6 |
| - | 2 | `[1]` | Pop 2 (H=5) | $\text{width} = 4 - 1 - 1 = 2 \implies 5 \times 2 = 10$ | **10** |
| - | 2 | `[1, 4]` | Push 4 | - | 10 |
| 5 | 3 | `[1, 4, 5]` | Push 5 | - | 10 |
| 6 | 0 | `[1, 4]` | Pop 5 (H=3) | $\text{width} = 6 - 4 - 1 = 1 \implies 3 \times 1 = 3$ | 10 |
| - | 0 | `[1]` | Pop 4 (H=2) | $\text{width} = 6 - 1 - 1 = 4 \implies 2 \times 4 = 8$ | 10 |
| - | 0 | `[]` | Pop 1 (H=1) | $\text{width} = 6 \implies 1 \times 6 = 6$ | 10 |

Maximum Area Recorded: **10** (formed by heights 5 and 6 at indices 2 and 3).
Time Complexity: $O(N)$ since each index is pushed and popped at most once.
Space Complexity: $O(N)$ for the monotonic stack.

---

## 5. Curated Problem Set (15 Problems)

Track your daily solutions by checking off the markdown boxes.

### NeetCode / LeetCode (Core)
- [ ] [LeetCode 20: Valid Parentheses](https://leetcode.com/problems/valid-parentheses/) | Easy | Bracket Matching LIFO Invariant
- [ ] [LeetCode 155: Min Stack](https://leetcode.com/problems/min-stack/) | Medium | Dual Stack / Tuple State Tracking
- [ ] [LeetCode 150: Evaluate Reverse Polish Notation](https://leetcode.com/problems/evaluate-reverse-polish-notation/) | Medium | Postfix Operator Stack Evaluation
- [ ] [LeetCode 739: Daily Temperatures](https://leetcode.com/problems/daily-temperatures/) | Medium | Monotonic Decreasing Stack Distance
- [ ] [LeetCode 853: Car Fleet](https://leetcode.com/problems/car-fleet/) | Medium | Arrival Time Monotonic Stack
- [ ] [LeetCode 84: Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/) | Hard | Monotonic Increasing Stack Bounding
- [ ] [LeetCode 85: Maximal Rectangle](https://leetcode.com/problems/maximal-rectangle/) | Hard | 2D Grid Histogram Reduction
- [ ] [LeetCode 503: Next Greater Element II](https://leetcode.com/problems/next-greater-element-ii/) | Medium | Circular Array Double-Pass Stack
- [ ] [LeetCode 402: Remove K Digits](https://leetcode.com/problems/remove-k-digits/) | Medium | Greedy Monotonic Stack String Construction
- [ ] [LeetCode 901: Online Stock Span](https://leetcode.com/problems/online-stock-span/) | Medium | Monotonic Stack Pair Aggregation

### CSES Problem Set (Free Tier, No Login Wall)
- [ ] [CSES 1645: Nearest Smaller Values](https://cses.fi/problemset/task/1645) | Sorting & Searching | Monotonic Stack for Previous Smaller Element
- [ ] [CSES 1145: Increasing Subsequence](https://cses.fi/problemset/task/1145) | Dynamic Programming | Monotonic Tails / Patience Sorting
- [ ] [CSES 1644: Maximum Subarray Sum II](https://cses.fi/problemset/task/1644) | Sorting & Searching | Monotonic Deque over Bounded Prefix Sums

### Codeforces & AtCoder (Rating 800–1400)
- [ ] [Codeforces 158C: Cd and pwd commands](https://codeforces.com/problemset/problem/158/C) | 1200 | Filesystem Path Simulation with Stack
- [ ] [Codeforces 5C: Longest Regular Bracket Sequence](https://codeforces.com/problemset/problem/5/C) | 1900 | Index-Based Stack for Longest Balanced Substring

---

## 6. Post-Session Active Recall Flashcards (15 Minutes)

<details>
<summary><strong>Flashcard 1: Why does a circular array for Next Greater Element II only require iterating up to $2N - 1$?</strong></summary>

Any element in a circular array can look at most $N-1$ elements ahead before wrapping around to itself. By simulating an array of length $2N$ using `i % N`, we give every element a full $N-1$ window to locate its next greater element without copying the physical array.
</details>

<details>
<summary><strong>Flashcard 2: In Remove K Digits, what condition causes a digit to be popped from the stack?</strong></summary>

A digit at the top of the stack is popped if the current digit is strictly smaller (`stack[-1] > digit`) and we still have removals available (`k > 0`). This greedily minimizes the most significant digits first.
</details>

<details>
<summary><strong>Flashcard 3: How does Maximal Rectangle (LeetCode 85) reduce a 2D binary matrix into multiple histogram problems?</strong></summary>

Iterate row by row. Maintain an array `heights` of size $M$ (number of columns). For each row:
- If `matrix[row][col] == '1'`, `heights[col] += 1`.
- If `matrix[row][col] == '0'`, `heights[col] = 0` (reset because continuity is broken).
Run the $O(M)$ `largest_rectangle_area` on `heights`. Repeating this for all $N$ rows yields an $O(N \times M)$ solution.
</details>

---

[← Topic 03: Sliding Window](03-sliding-window.md) \| [Next Topic: Binary Search →](05-binary-search.md)
