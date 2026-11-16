---
layout: default
title: "Binary Search & Answer Space Reduction"
---

# Binary Search & Answer Space Reduction

> **Estimated Time**: 11 Days • **Pace**: 1.5–2 hours/day (Strictly 1 problem/day) • **Language**: C# (.NET 6+)

---

## 🛠️ C# Mental Model & Gotchas

### 1. The Integer Overflow Invariant
In C#, calculating the midpoint as `(left + right) / 2` can cause signed 32-bit integer overflow if `left + right > int.MaxValue` ($2 \times 10^9$).
Always use:
```csharp
int mid = left + (right - left) / 2;
```
For large search spaces (e.g. cumulative sums or factory machine outputs), use `long`:
```csharp
long mid = left + (right - left) / 2;
```

### 2. Predicate Monotonicity ($FF\dots TT$)
Binary Search is not limited to sorted arrays. Any search space that maps to a monotonic boolean predicate:
$$\text{False}, \text{False}, \dots, \text{False}, \text{True}, \text{True}, \dots, \text{True}$$
can be partitioned in $O(\log(\text{Range}))$ steps to locate the exact transition boundary.

### 3. Loop Invariant: $[left, right]$ vs $[left, right)$
- **Closed Interval** (`while (left <= right)`): Use when searching for a discrete value where `mid` can be returned immediately. When advancing, use `left = mid + 1` and `right = mid - 1`.
- **Half-Open Interval** (`while (left < right)`): Standard for binary searching an answer space. Shrink to `right = mid` when condition is met and `left = mid + 1` otherwise. The loop terminates when `left == right`.

---

## 📅 Day-by-Day Schedule

### Day 1: Theory Anchor & Baseline Pattern
- **Step 1 (45 mins) | Concept Video**: [Binary Search Algorithm Tutorial](https://www.youtube.com/watch?v=C2apEw9pgtw) by *Abdul Bari*
  - Key focus: Trace search space halving, recurrence relation $T(N) = T(N/2) + O(1)$, and logarithmic convergence.
- **Step 2 (20 mins) | Pen & Paper Invariant**:
  - Draw an array of size 8. Trace pointers `left`, `right`, and `mid` for an element present at index 2, and for a missing target. Note termination condition `left > right`.
- **Step 3 (45 mins) | Problem 1 (Warm-Up)**:
  - [Binary Search](https://leetcode.com/problems/binary-search/) (NeetCode - Easy)
  - Objective: Clean C# implementation using safe midpoint calculation.
- **Step 4 (10 mins) | Active Recall**:
  <details>
  <summary><b>🧠 Self-Test: Why does <code>int mid = (left + right) / 2</code> risk crashing in C#?</b></summary>
  <br>
  <b>Answer:</b> If <code>left + right</code> exceeds $2,147,483,647$ (<code>int.MaxValue</code>), it overflows to a negative number, resulting in a negative index and an immediate <code>IndexOutOfRangeException</code>.
  </details>

---

### Day 2: Coordinate Transformation (2D Matrix Flattening)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: How do you map a 1D index $i$ in range $[0 \dots M \times N - 1]$ to a 2D matrix row and column?</b></summary>
  <br>
  <b>Answer:</b> <code>row = i / N</code> and <code>col = i % N</code> where $N$ is the number of columns.
  </details>
- **Deep Practice (75–85 mins) | Problem 2**:
  - [Search a 2D Matrix](https://leetcode.com/problems/search-a-2d-matrix/) (NeetCode - Medium)
  - Thinking prompts: Treat the $M \times N$ matrix as a single continuous 1D sorted array of length $M \times N$.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public bool SearchMatrix(int[][] matrix, int target) {
          int m = matrix.Length;
          int n = matrix[0].Length;
          int left = 0;
          int right = m * n - 1;
          
          while (left <= right) {
              int mid = left + (right - left) / 2;
              int val = matrix[mid / n][mid % n];
              
              if (val == target) return true;
              if (val < target) left = mid + 1;
              else right = mid - 1;
          }
          return false;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(\log(M \times N))$, Space: $O(1)$.
  </details>

---

### Day 3: Rotated Array Invariant & Pivot Finding
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: In a rotated sorted array without duplicates, how do you know if the left half $[left \dots mid]$ is sorted?</b></summary>
  <br>
  <b>Answer:</b> If <code>nums[left] &lt;= nums[mid]</code>, the left half is monotonically increasing and unrotated. Otherwise, the inflection point lies in the left half.
  </details>
- **Deep Practice (75–85 mins) | Problem 3**:
  - [Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/) (NeetCode - Medium)
  - Thinking prompts: Compare `nums[mid]` with `nums[right]`. If `nums[mid] > nums[right]`, the minimum must reside in `[mid + 1, right]`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public int FindMin(int[] nums) {
          int left = 0;
          int right = nums.Length - 1;
          
          while (left < right) {
              int mid = left + (right - left) / 2;
              if (nums[mid] > nums[right]) {
                  left = mid + 1; // Inflection point is strictly right of mid
              } else {
                  right = mid; // mid could be the minimum
              }
          }
          return nums[left];
      }
  }
  ```
  <b>Complexity:</b> Time: $O(\log N)$, Space: $O(1)$.
  </details>

---

### Day 4: Partitioned Search in Rotated Array
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: In Search in Rotated Sorted Array, what two conditions confirm that <code>target</code> is inside the sorted left half?</b></summary>
  <br>
  <b>Answer:</b> <code>nums[left] &lt;= target &amp;&amp; target &lt; nums[mid]</code> (given that <code>nums[left] &lt;= nums[mid]</code>).
  </details>
- **Deep Practice (75–85 mins) | Problem 4**:
  - [Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/) (NeetCode - Medium)
  - Thinking prompts: Check which half is sorted first, then test if the target lies within the boundaries of that sorted half.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public int Search(int[] nums, int target) {
          int left = 0, right = nums.Length - 1;
          
          while (left <= right) {
              int mid = left + (right - left) / 2;
              if (nums[mid] == target) return mid;
              
              // Left half is normally sorted
              if (nums[left] <= nums[mid]) {
                  if (nums[left] <= target && target < nums[mid]) {
                      right = mid - 1;
                  } else {
                      left = mid + 1;
                  }
              } else { // Right half is normally sorted
                  if (nums[mid] < target && target <= nums[right]) {
                      left = mid + 1;
                  } else {
                      right = mid - 1;
                  }
              }
          }
          return -1;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(\log N)$, Space: $O(1)$.
  </details>

---

### Day 5: Midpoint Shift — Binary Search on Answer Space
- **Step 1 (20 mins) | Focused Video**: [Koko Eating Bananas Explained](https://www.youtube.com/watch?v=UuiTKBwPgAo) by *NeetCode*
  - Key focus: What is the answer range for speed $K$? Why is the predicate `CanEatAll(speed)` monotonic?
- **Deep Practice (75 mins) | Problem 5**:
  - [Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/) (NeetCode - Medium)
  - Thinking prompts: Minimum speed is 1; maximum speed is $\max(piles)$. Define helper `CanFinish(speed, h)`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public int MinEatingSpeed(int[] piles, int h) {
          int left = 1;
          int right = 0;
          foreach (int p in piles) {
              if (p > right) right = p;
          }
          
          int result = right;
          while (left <= right) {
              int mid = left + (right - left) / 2;
              if (CanFinish(piles, h, mid)) {
                  result = mid;
                  right = mid - 1; // Try finding a smaller feasible speed
              } else {
                  left = mid + 1;
              }
          }
          return result;
      }
      
      private bool CanFinish(int[] piles, int h, int speed) {
          long totalHours = 0;
          foreach (int p in piles) {
              // Ceiling division: (p + speed - 1) / speed
              totalHours += (p + speed - 1L) / speed;
          }
          return totalHours <= h;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N \log(\max(P)))$, Space: $O(1)$.
  </details>

---

### Day 6: Temporal Range Binary Search (Time-Based Key-Value Store)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: When timestamps are strictly increasing in <code>Set()</code>, how do we find the largest timestamp $\le target$ in <code>Get()</code>?</b></summary>
  <br>
  <b>Answer:</b> Binary search on the list of entries for key. When <code>list[mid].timestamp &lt;= target</code>, save <code>mid</code> as candidate and search right (<code>left = mid + 1</code>).
  </details>
- **Deep Practice (75–85 mins) | Problem 6**:
  - [Time Based Key-Value Store](https://leetcode.com/problems/time-based-key-value-store/) (NeetCode - Medium)
  - Thinking prompts: Use `Dictionary<string, List<(int timestamp, string value)>>`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class TimeMap {
      private readonly Dictionary<string, List<(int time, string val)>> _map = new();

      public void Set(string key, string value, int timestamp) {
          if (!_map.TryGetValue(key, out var list)) {
              list = new List<(int, string)>();
              _map[key] = list;
          }
          list.Add((timestamp, value));
      }
      
      public string Get(string key, int timestamp) {
          if (!_map.TryGetValue(key, out var list)) return "";
          
          int left = 0, right = list.Count - 1;
          string res = "";
          
          while (left <= right) {
              int mid = left + (right - left) / 2;
              if (list[mid].time <= timestamp) {
                  res = list[mid].val;
                  left = mid + 1;
              } else {
                  right = mid - 1;
              }
          }
          return res;
      }
  }
  ```
  <b>Complexity:</b> `Set`: $O(1)$, `Get`: $O(\log K)$ where $K$ is timestamps count per key. Space: $O(N)$.
  </details>

---

### Day 7: Capacity Constraints on Answer Space
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: In Capacity To Ship Packages Within D Days, what are the lower and upper bounds of shipping capacity?</b></summary>
  <br>
  <b>Answer:</b> Lower bound is $\max(weights)$ (a ship must carry at least the heaviest single package). Upper bound is $\sum weights$ (carrying all packages in 1 day).
  </details>
- **Deep Practice (75–85 mins) | Problem 7**:
  - [Capacity To Ship Packages Within D Days](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/) (LeetCode 1011 - Medium)
  - Thinking prompts: Predicate function greedily fills ship until adding next package exceeds `capacity`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public int ShipWithinDays(int[] weights, int days) {
          int left = 0, right = 0;
          foreach (int w in weights) {
              left = Math.Max(left, w);
              right += w;
          }
          
          int ans = right;
          while (left <= right) {
              int mid = left + (right - left) / 2;
              if (CanShip(weights, days, mid)) {
                  ans = mid;
                  right = mid - 1;
              } else {
                  left = mid + 1;
              }
          }
          return ans;
      }
      
      private bool CanShip(int[] weights, int days, int capacity) {
          int currentDays = 1;
          int currentLoad = 0;
          foreach (int w in weights) {
              if (currentLoad + w > capacity) {
                  currentDays++;
                  currentLoad = 0;
              }
              currentLoad += w;
          }
          return currentDays <= days;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N \log(\sum W))$, Space: $O(1)$.
  </details>

---

### Day 8: Unsorted Gradient Slope (Find Peak Element)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: How can binary search work on an unsorted array in Find Peak Element?</b></summary>
  <br>
  <b>Answer:</b> Follow the gradient! If <code>nums[mid] &lt; nums[mid + 1]</code>, there must be a peak to the right of <code>mid</code> because the slope is rising. Otherwise, a peak exists at or to the left of <code>mid</code>.
  </details>
- **Deep Practice (75–85 mins) | Problem 8**:
  - [Find Peak Element](https://leetcode.com/problems/find-peak-element/) (LeetCode 162 - Medium)
  - Thinking prompts: Compare `nums[mid]` with `nums[mid + 1]`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public int FindPeakElement(int[] nums) {
          int left = 0;
          int right = nums.Length - 1;
          
          while (left < right) {
              int mid = left + (right - left) / 2;
              if (nums[mid] < nums[mid + 1]) {
                  left = mid + 1;
              } else {
                  right = mid;
              }
          }
          return left;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(\log N)$, Space: $O(1)$.
  </details>

---

### Day 9: Dual Partitioning (Median of Two Sorted Arrays)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: What is the goal partition in Median of Two Sorted Arrays?</b></summary>
  <br>
  <b>Answer:</b> Partition arrays $A$ and $B$ into left and right halves such that $\text{totalLeft} = \text{totalRight}$ and $\max(\text{left } A, \text{left } B) \le \min(\text{right } A, \text{right } B)$.
  </details>
- **Deep Practice (75–85 mins) | Problem 9**:
  - [Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/) (NeetCode - Hard)
  - Thinking prompts: Always run binary search on the smaller array to guarantee $O(\log(\min(M, N)))$. Handle boundary conditions with $\pm \infty$.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public double FindMedianSortedArrays(int[] nums1, int[] nums2) {
          if (nums1.Length > nums2.Length) {
              return FindMedianSortedArrays(nums2, nums1); // Ensure nums1 is smaller
          }
          
          int m = nums1.Length;
          int n = nums2.Length;
          int totalLeft = (m + n + 1) / 2;
          
          int left = 0, right = m;
          while (left <= right) {
              int i = left + (right - left) / 2;
              int j = totalLeft - i;
              
              int left1 = (i > 0) ? nums1[i - 1] : int.MinValue;
              int right1 = (i < m) ? nums1[i] : int.MaxValue;
              int left2 = (j > 0) ? nums2[j - 1] : int.MinValue;
              int right2 = (j < n) ? nums2[j] : int.MaxValue;
              
              if (left1 <= right2 && left2 <= right1) {
                  if ((m + n) % 2 == 1) {
                      return Math.Max(left1, left2);
                  }
                  return (Math.Max(left1, left2) + Math.Min(right1, right2)) / 2.0;
              } else if (left1 > right2) {
                  right = i - 1;
              } else {
                  left = i + 1;
              }
          }
          return 0.0;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(\log(\min(M, N)))$, Space: $O(1)$.
  </details>

---

### Day 10: Array Splitting Optimization (Split Array Largest Sum)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: Why is Split Array Largest Sum identical to Capacity to Ship Packages?</b></summary>
  <br>
  <b>Answer:</b> Both ask for the minimum capacity (max subarray sum) such that the items can be divided into at most $K$ contiguous subarrays.
  </details>
- **Deep Practice (75–85 mins) | Problem 10**:
  - [Split Array Largest Sum](https://leetcode.com/problems/split-array-largest-sum/) (LeetCode 410 - Hard)
  - Thinking prompts: Binary search on the answer space range $[\max(nums), \sum nums]$.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public int SplitArray(int[] nums, int k) {
          int left = 0;
          int right = 0;
          foreach (int n in nums) {
              left = Math.Max(left, n);
              right += n;
          }
          
          int ans = right;
          while (left <= right) {
              int mid = left + (right - left) / 2;
              if (CanSplit(nums, k, mid)) {
                  ans = mid;
                  right = mid - 1;
              } else {
                  left = mid + 1;
              }
          }
          return ans;
      }
      
      private bool CanSplit(int[] nums, int k, int maxSubarraySum) {
          int count = 1;
          int currentSum = 0;
          foreach (int n in nums) {
              if (currentSum + n > maxSubarraySum) {
                  count++;
                  currentSum = 0;
              }
              currentSum += n;
          }
          return count <= k;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N \log(\sum N))$, Space: $O(1)$.
  </details>

---

### Day 11: CSES & Competitive Benchmarks + Spaced Retrieval
- **Benchmark 1**: [CSES: Factory Machines](https://cses.fi/problemset/task/1620) (Binary search on time to manufacture $T$ products)
- **Benchmark 2**: [CSES: Array Division](https://cses.fi/problemset/task/1085) (Partition array into $K$ subarrays minimizing max sum)
- **Benchmark 3**: [CSES: Multiplication Table](https://cses.fi/problemset/task/2422) (Find median in $N \times N$ multiplication grid)
- **Benchmark 4**: [Codeforces 1201C - Maximum Median](https://codeforces.com/problemset/problem/1201/C) (Binary search on median with $K$ operations)
- **Benchmark 5**: [Codeforces 492B - Vanya and Lanterns](https://codeforces.com/problemset/problem/492/B) (Floating point binary search / max distance)

---

### Spaced Retrieval Flashcards

<details>
<summary><b>Flashcard 1: When can Binary Search be applied to a problem that does not mention sorted arrays?</b></summary>
<br>
Whenever the decision problem forms a monotonic predicate: if a candidate answer $X$ is feasible, then all values greater than $X$ (or smaller than $X$) are also guaranteed to be feasible.
</details>

<details>
<summary><b>Flashcard 2: In ceiling integer division, how do you compute $\lceil A / B \rceil$ without floating point casting?</b></summary>
<br>
Use <code>(A + B - 1) / B</code> for positive integers. If $A$ and $B$ are large, write <code>(A + B - 1L) / B</code> to prevent 32-bit overflow.
</details>

<details>
<summary><b>Flashcard 3: What is the termination difference between <code>while (left &lt;= right)</code> and <code>while (left &lt; right)</code>?</b></summary>
<br>
In <code>while (left &lt;= right)</code>, the loop executes when <code>left == right</code>, and terminates with <code>left &gt; right</code>. In <code>while (left &lt; right)</code>, the loop stops the instant <code>left == right</code>, converging on a single boundary index.
</details>

---

## 🎯 Topic Checklist (15 Problems)

| Day | # | Problem | Platform | Difficulty | Status |
|:---:|:---:|:---|:---:|:---:|:---:|
| Day 1 | 1 | [Binary Search](https://leetcode.com/problems/binary-search/) | NeetCode | Easy | - [ ] |
| Day 2 | 2 | [Search a 2D Matrix](https://leetcode.com/problems/search-a-2d-matrix/) | NeetCode | Medium | - [ ] |
| Day 3 | 3 | [Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/) | NeetCode | Medium | - [ ] |
| Day 4 | 4 | [Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/) | NeetCode | Medium | - [ ] |
| Day 5 | 5 | [Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/) | NeetCode | Medium | - [ ] |
| Day 6 | 6 | [Time Based Key-Value Store](https://leetcode.com/problems/time-based-key-value-store/) | NeetCode | Medium | - [ ] |
| Day 7 | 7 | [Capacity To Ship Packages Within D Days](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/) | LeetCode | Medium | - [ ] |
| Day 8 | 8 | [Find Peak Element](https://leetcode.com/problems/find-peak-element/) | LeetCode | Medium | - [ ] |
| Day 9 | 9 | [Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/) | NeetCode | Hard | - [ ] |
| Day 10 | 10 | [Split Array Largest Sum](https://leetcode.com/problems/split-array-largest-sum/) | LeetCode | Hard | - [ ] |
| Day 11 | 11 | [Factory Machines](https://cses.fi/problemset/task/1620) | CSES | Medium | - [ ] |
| Day 11 | 12 | [Array Division](https://cses.fi/problemset/task/1085) | CSES | Medium | - [ ] |
| Day 11 | 13 | [Multiplication Table](https://cses.fi/problemset/task/2422) | CSES | Hard | - [ ] |
| Day 11 | 14 | [Maximum Median (1201C)](https://codeforces.com/problemset/problem/1201/C) | Codeforces | Rating 1400 | - [ ] |
| Day 11 | 15 | [Vanya and Lanterns (492B)](https://codeforces.com/problemset/problem/492/B) | Codeforces | Rating 1200 | - [ ] |

---

[⬅️ Previous Topic: Stack & Monotonic Stack](04-stack.html) | [Home](index.html) | [Next Topic: Linked Lists & Fast/Slow Pointers ➡️](06-linked-lists.html)

