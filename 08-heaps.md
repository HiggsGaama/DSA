---
layout: default
title: "Topic 08: Heaps & Priority Queues"
---

# Topic 08: Heaps & Priority Queues

[← Topic 07: Binary Trees](07-binary-trees.md) \| [Schedule](schedule.md) \| [Next: Backtracking →](09-backtracking.md)

---

## Daily Timeboxing (1.5 – 2.0 Hours)
- **00:00 – 00:15 (15m) | Active Recall Pre-Test**: Answer the conceptual questions below before reading further.
- **00:15 – 00:40 (25m) | YouTube Conceptual Anchor**: Watch the curated video segment on dual-heap median tracking and heapify complexity.
- **00:40 – 01:30 (50m) | Coding Deep Dive**: Pen-and-paper dry run of Find Median from Data Stream & Task Scheduler.
- **01:30 – 01:45 (15m) | Post-Session Flashcards**: Complete the flashcard prompts from memory.

---

## 1. Active Recall Pre-Test (15 Minutes)
Attempt to answer these questions on paper before expanding the solutions.

<details>
<summary><strong>Prompt 1: Why does building a heap from an unordered array of $N$ elements (`heapify`) take $O(N)$ time, while inserting $N$ elements one by one takes $O(N \log N)$?</strong></summary>

Inserting elements one by one performs up-heap (sift-up) operations. The number of nodes at depth $d$ is $2^d$, and their distance from the root is $d$. The upper layers (many nodes) perform the maximum work:
$$\sum_{d=0}^{\log N} d \cdot 2^d = \Theta(N \log N)$$
In bottom-up `heapify` (sift-down), work is proportional to the **height** of each node from the bottom. Half the nodes ($N/2$) are leaves with height 0 (zero swaps). A quarter of nodes ($N/4$) have height 1, an eighth have height 2, etc.:
$$\sum_{h=1}^{\log N} \frac{N}{2^{h+1}} \cdot h = N \sum_{h=1}^{\log N} \frac{h}{2^{h+1}} = O(N)$$
Because the vast majority of nodes are near the bottom and do almost no work, bottom-up heap construction runs in linear $O(N)$ time.
</details>

<details>
<summary><strong>Prompt 2: To find the $K$-th largest element in a stream of unbounded numbers, should you use a Min-Heap or a Max-Heap of size $K$?</strong></summary>

A **Min-Heap of size $K$**.
- The min-heap stores the $K$ largest elements encountered so far.
- The root of this min-heap holds the smallest of these $K$ elements—which is precisely the $K$-th largest element overall!
- When a new element arrives: if it is smaller than the root, it cannot be among the top $K$, so discard it. If it is larger, push it and pop the root.
- Space is strictly bounded to $O(K)$, and each insertion costs $O(\log K)$.
</details>

<details>
<summary><strong>Prompt 3: In the Two-Heap pattern for streaming median, what two invariants must be maintained after every insertion?</strong></summary>

1. **Order Invariant**: Every element in `small` (Max-Heap) must be less than or equal to every element in `large` (Min-Heap): $\max(\text{small}) \le \min(\text{large})$.
2. **Size / Balance Invariant**: The sizes must remain balanced: either `len(small) == len(large)` (even count), or `len(small) == len(large) + 1` (odd count).
</details>

---

## 2. YouTube Conceptual Anchor (25 Minutes)

Recommended Video Anchors:
1. **Abdul Bari**: [Heap Sort, Heapify & Priority Queues](https://www.youtube.com/watch?v=HqPJF2L5h9U) (25 mins)
2. **NeetCode**: [Find Median from Data Stream (Two Heaps)](https://www.youtube.com/watch?v=itmhHWaHupI) (15 mins)

### Core Invariants & Mental Model
1. **Array Index Arithmetic**: For a 0-indexed complete binary tree stored in an array:
   - Parent: `parent(i) = (i - 1) // 2`
   - Left Child: `left(i) = 2 * i + 1`
   - Right Child: `right(i) = 2 * i + 2`
2. **Top-K Retention Invariant**: To find the Top-K *largest*, maintain a *min-heap* of size $K$. To find the Top-K *smallest*, maintain a *max-heap* of size $K$.
3. **Median Partition Invariant**: The median splits a sorted sequence into a lower half (stored in a max-heap) and an upper half (stored in a min-heap).

---

## 3. Canonical Code Templates

### Pattern A: Top-K Maintenance with Min-Heap ($O(N \log K)$ Time, $O(K)$ Space)
```python
import heapq
from typing import List

def find_kth_largest(nums: List[int], k: int) -> int:
    min_heap = []
    
    for num in nums:
        heapq.heappush(min_heap, num)
        if len(min_heap) > k:
            heapq.heappop(min_heap) # Evict smallest among top candidates
            
    return min_heap[0]
```

### Pattern B: Dual-Heap Streaming Median ($O(\log N)$ Add, $O(1)$ Find)
```python
import heapq

class MedianFinder:
    def __init__(self):
        # small is a Max-Heap (stores lower half; invert values for Python's min-heap)
        self.small = []
        # large is a Min-Heap (stores upper half)
        self.large = []

    def add_num(self, num: int) -> None:
        # Step 1: Push to small (inverted for max-heap behavior)
        heapq.heappush(self.small, -num)
        
        # Step 2: Ensure small's max <= large's min
        if self.small and self.large and (-self.small[0] > self.large[0]):
            val = -heapq.heappop(self.small)
            heapq.heappush(self.large, val)
            
        # Step 3: Ensure size balance (len(small) == len(large) or len(large) + 1)
        if len(self.small) > len(self.large) + 1:
            val = -heapq.heappop(self.small)
            heapq.heappush(self.large, val)
        elif len(self.large) > len(self.small):
            val = heapq.heappop(self.large)
            heapq.heappush(self.small, -val)

    def find_median(self) -> float:
        if len(self.small) > len(self.large):
            return float(-self.small[0])
        return (-self.small[0] + self.large[0]) / 2.0
```

---

## 4. Coding Deep Dive: Worked Walkthrough (50 Minutes)

### Problem: Find Median from Data Stream (LeetCode 295)
**Problem Statement**: Design a data structure that supports adding integer numbers from a data stream and finding the median of all elements so far.

#### Pen-and-Paper Dry Run Trace
Insert stream: `[5, 2, 10, 4, 1]`.

| Step | Added `num` | Action & Rebalance | `small` (Max-Heap) | `large` (Min-Heap) | Current Median |
|:---:|:---:|:---|:---:|:---:|:---:|
| 1 | 5 | Add to `small` | `[5]` | `[]` | **5.0** |
| 2 | 2 | Add to `small` (`[5, 2]`), rebalance to `large` | `[2]` | `[5]` | $(2 + 5)/2 =$ **3.5** |
| 3 | 10 | Add to `small`, move to `large` $\implies$ `large=[5, 10]`, rebalance size | `[5, 2]` | `[10]` | **5.0** |
| 4 | 4 | Add to `small` $\implies$ `small=[5, 4, 2]`, rebalance to `large` | `[4, 2]` | `[5, 10]` | $(4 + 5)/2 =$ **4.5** |
| 5 | 1 | Add to `small` $\implies$ `small=[4, 2, 1]` | `[4, 2, 1]` | `[5, 10]` | **4.0** |

Time Complexity: `add_num`: $O(\log N)$, `find_median`: $O(1)$.
Space Complexity: $O(N)$ total elements stored.

---

## 5. Curated Problem Set (15 Problems)

Track your daily solutions by checking off the markdown boxes.

### NeetCode / LeetCode (Core)
- [ ] [LeetCode 703: Kth Largest Element in a Stream](https://leetcode.com/problems/kth-largest-element-in-a-stream/) | Easy | Bounded Min-Heap of Size K
- [ ] [LeetCode 1046: Last Stone Weight](https://leetcode.com/problems/last-stone-weight/) | Easy | Max-Heap Simulation
- [ ] [LeetCode 973: K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/) | Medium | Max-Heap of Size K / Euclidean Metric
- [ ] [LeetCode 215: Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/) | Medium | Min-Heap vs QuickSelect ($O(N)$ Average)
- [ ] [LeetCode 621: Task Scheduler](https://leetcode.com/problems/task-scheduler/) | Medium | Max-Heap Frequency + Cooling Deque
- [ ] [LeetCode 355: Design Twitter](https://leetcode.com/problems/design-twitter/) | Medium | K-Way Merge with Priority Queue
- [ ] [LeetCode 295: Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/) | Hard | Dual Balanced Heaps (Min-Max)
- [ ] [LeetCode 1882: Process Tasks Using Servers](https://leetcode.com/problems/process-tasks-using-servers/) | Medium | Free vs Busy Servers Two-Heap Event Simulation
- [ ] [LeetCode 2542: Maximum Subsequence Score](https://leetcode.com/problems/maximum-subsequence-score/) | Medium | Sort by Multiplier + Min-Heap of Size K

### CSES Problem Set (Free Tier, No Login Wall)
- [ ] [CSES 1091: Concert Tickets](https://cses.fi/problemset/task/1091) | Sorting & Searching | Multiset / Balanced BST Lower Bound
- [ ] [CSES 1163: Traffic Lights](https://cses.fi/problemset/task/1163) | Sorting & Searching | Max Heap + Hash Map Lazy Deletion
- [ ] [CSES 1629: Movie Festival](https://cses.fi/problemset/task/1629) | Sorting & Searching | Greedy Earliest End Time Priority

### Codeforces & AtCoder (Rating 800–1400)
- [ ] [Codeforces 1353D: Constructing the Array](https://codeforces.com/problemset/problem/1353/D) | 1400 | Max-Heap Segment Length Priority Simulation
- [ ] [Codeforces 802J: Send the Fool Further](https://codeforces.com/problemset/problem/802/J) | 1000 | Priority Queue / DFS Traversal
- [ ] [AtCoder ABC137_D: Summer Vacation](https://atcoder.jp/contests/abc137/tasks/abc137_d) | 1200 | Greedy Scheduling with Max-Heap

---

## 6. Post-Session Active Recall Flashcards (15 Minutes)

<details>
<summary><strong>Flashcard 1: How does Python's `heapq` module implement max-heaps since it only provides a min-heap by default?</strong></summary>

Multiply all numeric values by $-1$ before pushing them onto the heap: `heapq.heappush(heap, -val)`. 
When popping, multiply by $-1$ again to restore original polarity: `val = -heapq.heappop(heap)`. 
Alternatively, for custom objects or tuples, provide an inverted key `(-priority, item)`.
</details>

<details>
<summary><strong>Flashcard 2: What is "Lazy Deletion" in heaps, and why is it used in problems like CSES Traffic Lights?</strong></summary>

Standard binary heaps do not support arbitrary element deletion in $O(\log N)$ without maintaining internal node pointers. 
With **Lazy Deletion**, we leave stale elements inside the heap and track their deletion count in a hash map (`deleted[val] += 1`). Whenever we inspect `heap[0]`, we pop elements while `deleted[heap[0]] > 0` and decrement the map count. This amortizes deletion to $O(\log N)$ without complex node-tracking data structures.
</details>

<details>
<summary><strong>Flashcard 3: What is the worst-case time complexity of QuickSelect for Kth Largest Element, and how can it be made deterministic?</strong></summary>

Worst-case time complexity of standard QuickSelect is $O(N^2)$ (occurring when pivot choices are consistently the extreme values of the partition). 
Average time is $O(N)$. 
It can be made deterministically $O(N)$ worst-case using the **Median-of-Medians** pivot selection algorithm.
</details>

---

[← Topic 07: Binary Trees](07-binary-trees.md) \| [Next Topic: Backtracking →](09-backtracking.md)
