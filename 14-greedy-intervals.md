---
layout: default
title: "Greedy Algorithms & Intervals"
---

# Greedy Algorithms & Intervals

> **Estimated Time**: 11 Days • **Pace**: 1.5–2 hours/day (Strictly 1 problem/day) • **Language**: C# (.NET 6+)

---

## 🛠️ C# Mental Model & Gotchas

### 1. Sorting Invariants: Start Time vs End Time
Interval problems live or die by the choice of sorting dimension:
- **Interval Merging (`Merge Intervals`, `Insert Interval`)**: Sort by **START TIME** (`a[0].CompareTo(b[0])`). If the next interval starts before or at the current interval's end (`next[0] <= curr[1]`), merge them: `curr[1] = Math.Max(curr[1], next[1])`.
- **Interval Scheduling (`Non-overlapping Intervals`, `Movie Festival`)**: Sort by **END TIME** (`a[1].CompareTo(b[1])`). Greedily picking the task that finishes earliest leaves maximum room for all subsequent tasks.

### 2. High-Performance Sweep-Line in C#
To find the maximum concurrent overlapping intervals (e.g. `Meeting Rooms II`, `Restaurant Customers`):
- Instead of using a priority queue, separate starts and ends into two flat arrays:
```csharp
int[] starts = new int[n];
int[] ends = new int[n];
// populate and sort independently
Array.Sort(starts);
Array.Sort(ends);
```
Traverse both with two pointers: whenever `starts[s] < ends[e]`, a room is occupied (`count++`, `s++`); otherwise, a room is vacated (`count--`, `e++`). This avoids boxing and heap allocations entirely!

---

## 📅 Day-by-Day Schedule

### Day 1: Theory Anchor & Baseline Pattern (Kadane's Algorithm)
- **Step 1 (45 mins) | Concept Video**: [Kadane's Algorithm Explained](https://www.youtube.com/watch?v=5WZl3MMT0Eg) by *NeetCode*
  - Key focus: What is the greedy invariant? Why does discarding a negative prefix sum guarantee the optimal subarray ending at index $i$?
- **Step 2 (20 mins) | Pen & Paper Invariant**:
  - Trace `[-2, 1, -3, 4, -1, 2, 1, -5, 4]`. Track `curSum = Math.Max(n, curSum + n)` at every step.
- **Step 3 (45 mins) | Problem 1 (Warm-Up)**:
  - [Maximum Subarray](https://leetcode.com/problems/maximum-subarray/) (NeetCode - Medium)
  - Objective: Single pass $O(N)$ time with $O(1)$ memory.
- **Step 4 (10 mins) | Active Recall**:
  <details>
  <summary><b>🧠 Self-Test: When does Kadane's algorithm reset the running sum to 0?</b></summary>
  <br>
  <b>Answer:</b> Whenever the accumulated running sum drops below 0. A negative prefix can only diminish the sum of any subsequent contiguous subarray, so it is greedily abandoned.
  </details>

---

### Day 2: Farthest Reach Horizon (Jump Game)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: In Jump Game, how do you verify if index $i$ is reachable?</b></summary>
  <br>
  <b>Answer:</b> Index $i$ is reachable if and only if $i \le \text{maxReach}$, where $\text{maxReach}$ is the maximum index reachable by any previously visited index: $\max_{j < i}(j + nums[j])$.
  </details>
- **Deep Practice (75–85 mins) | Problem 2**:
  - [Jump Game](https://leetcode.com/problems/jump-game/) (NeetCode - Medium)
  - Thinking prompts: Update `maxReach = Math.Max(maxReach, i + nums[i])`. If `i > maxReach`, return `false`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public bool CanJump(int[] nums) {
          int maxReach = 0;
          for (int i = 0; i < nums.Length; i++) {
              if (i > maxReach) return false;
              maxReach = Math.Max(maxReach, i + nums[i]);
              if (maxReach >= nums.Length - 1) return true;
          }
          return true;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N)$, Space: $O(1)$.
  </details>

---

### Day 3: BFS-Window Leapfrogging (Jump Game II)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: How can Jump Game II be modeled as a BFS level traversal without a queue?</b></summary>
  <br>
  <b>Answer:</b> Maintain current jump window $[curEnd, farthest]$. When index $i$ reaches $curEnd$, increment jumps and update $curEnd = farthest$.
  </details>
- **Deep Practice (75–85 mins) | Problem 3**:
  - [Jump Game II](https://leetcode.com/problems/jump-game-ii/) (NeetCode - Medium)
  - Thinking prompts: Find minimum jumps to reach end.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public int Jump(int[] nums) {
          int jumps = 0, curEnd = 0, farthest = 0;
          
          for (int i = 0; i < nums.Length - 1; i++) {
              farthest = Math.Max(farthest, i + nums[i]);
              if (i == curEnd) {
                  jumps++;
                  curEnd = farthest;
                  if (curEnd >= nums.Length - 1) break;
              }
          }
          return jumps;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N)$, Space: $O(1)$.
  </details>

---

### Day 4: Circular Deficit Reset (Gas Station)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: In Gas Station, if starting at station $A$ fails to reach station $B$, why can no station between $A$ and $B$ be a valid start?</b></summary>
  <br>
  <b>Answer:</b> Any intermediate station $C$ between $A$ and $B$ had a non-negative fuel surplus upon arriving from $A$. If even with that surplus you couldn't reach $B$, starting empty at $C$ will fail even earlier!
  </details>
- **Deep Practice (75–85 mins) | Problem 4**:
  - [Gas Station](https://leetcode.com/problems/gas-station/) (NeetCode - Medium)
  - Thinking prompts: If `totalGas >= totalCost`, a unique solution is guaranteed to exist. Whenever `tank < 0`, reset `start = i + 1` and `tank = 0`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public int CanCompleteCircuit(int[] gas, int[] cost) {
          int totalTank = 0;
          int currTank = 0;
          int startStation = 0;
          
          for (int i = 0; i < gas.Length; i++) {
              int net = gas[i] - cost[i];
              totalTank += net;
              currTank += net;
              
              if (currTank < 0) {
                  startStation = i + 1;
                  currTank = 0;
              }
          }
          return totalTank >= 0 ? startStation : -1;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N)$, Space: $O(1)$.
  </details>

---

### Day 5: Multi-Set Greedy Straights (Hand of Straights)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: In Hand of Straights, why must we start forming groups from the smallest remaining card?</b></summary>
  <br>
  <b>Answer:</b> The smallest card $X$ cannot be in the middle or end of any valid consecutive group; it MUST be the start of a group $(X, X+1, \dots, X+G-1)$.
  </details>
- **Deep Practice (75–85 mins) | Problem 5**:
  - [Hand of Straights](https://leetcode.com/problems/hand-of-straights/) (NeetCode - Medium)
  - Thinking prompts: Use `PriorityQueue<int, int>` or sort unique cards with a frequency `Dictionary<int, int>`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public bool IsNStraightHand(int[] hand, int groupSize) {
          if (hand.Length % groupSize != 0) return false;
          
          Dictionary<int, int> count = new();
          foreach (int card in hand) {
              count[card] = count.GetValueOrDefault(card, 0) + 1;
          }
          
          Array.Sort(hand);
          foreach (int card in hand) {
              if (count[card] == 0) continue;
              
              for (int i = 0; i < groupSize; i++) {
                  int nextCard = card + i;
                  if (count.GetValueOrDefault(nextCard, 0) == 0) {
                      return false;
                  }
                  count[nextCard]--;
              }
          }
          return true;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N \log N)$, Space: $O(N)$.
  </details>

---

### Day 6: Contiguous Interval Compaction (Merge Intervals)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: In Merge Intervals, when does interval $[s_2, e_2]$ overlap with current interval $[s_1, e_1]$ after sorting by start time?</b></summary>
  <br>
  <b>Answer:</b> When $s_2 \le e_1$. The merged interval becomes $[s_1, \max(e_1, e_2)]$.
  </details>
- **Deep Practice (75–85 mins) | Problem 6**:
  - [Merge Intervals](https://leetcode.com/problems/merge-intervals/) (NeetCode - Medium)
  - Thinking prompts: Sort intervals by start time `Array.Sort(intervals, (a, b) => a[0].CompareTo(b[0]))`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public int[][] Merge(int[][] intervals) {
          Array.Sort(intervals, (a, b) => a[0].CompareTo(b[0]));
          List<int[]> merged = new();
          
          int[] current = intervals[0];
          merged.Add(current);
          
          for (int i = 1; i < intervals.Length; i++) {
              if (intervals[i][0] <= current[1]) {
                  current[1] = Math.Max(current[1], intervals[i][1]);
              } else {
                  current = intervals[i];
                  merged.Add(current);
              }
          }
          return merged.ToArray();
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N \log N)$ sorting, Space: $O(N)$ for output.
  </details>

---

### Day 7: Earliest Deadline Scheduling (Non-overlapping Intervals)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: Why sort by END TIME in Non-overlapping Intervals instead of start time?</b></summary>
  <br>
  <b>Answer:</b> The interval that finishes earliest frees up resources as soon as possible, leaving the maximum amount of remaining time for other intervals.
  </details>
- **Deep Practice (75–85 mins) | Problem 7**:
  - [Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/) (NeetCode - Medium)
  - Thinking prompts: Sort by end time. If `intervals[i][0] < prevEnd`, an overlap exists, so increment removal count. Otherwise update `prevEnd = intervals[i][1]`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public int EraseOverlapIntervals(int[][] intervals) {
          Array.Sort(intervals, (a, b) => a[1].CompareTo(b[1]));
          int removals = 0;
          int prevEnd = intervals[0][1];
          
          for (int i = 1; i < intervals.Length; i++) {
              if (intervals[i][0] < prevEnd) {
                  removals++; // Must discard overlapping interval
              } else {
                  prevEnd = intervals[i][1];
              }
          }
          return removals;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N \log N)$, Space: $O(1)$.
  </details>

---

### Day 8: Interval Conflict Checking (Meeting Rooms)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: Can a person attend all meetings if one meeting ends at 10 and another starts at 10?</b></summary>
  <br>
  <b>Answer:</b> Yes, contiguous boundaries ($end_1 == start_2$) do not conflict unless the problem explicitly states closed intervals.
  </details>
- **Deep Practice (75–85 mins) | Problem 8**:
  - [Meeting Rooms](https://leetcode.com/problems/meeting-rooms/) (NeetCode - Easy)
  - Thinking prompts: Sort by start time. Check if `intervals[i][0] < intervals[i - 1][1]`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public bool CanAttendMeetings(int[][] intervals) {
          Array.Sort(intervals, (a, b) => a[0].CompareTo(b[0]));
          for (int i = 1; i < intervals.Length; i++) {
              if (intervals[i][0] < intervals[i - 1][1]) {
                  return false;
              }
          }
          return true;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N \log N)$, Space: $O(1)$.
  </details>

---

### Day 9: Two-Pointer Sweep-Line (Meeting Rooms II)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: How does separating start and end times into two sorted arrays simulate room occupancy?</b></summary>
  <br>
  <b>Answer:</b> We don't care *which* specific meeting is using a room, only the global net count of in-progress meetings. Moving through sorted start times tracks new meetings, while moving through sorted end times tracks meeting completions.
  </details>
- **Deep Practice (75–85 mins) | Problem 9**:
  - [Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii/) (NeetCode - Medium)
  - Thinking prompts: Array sorting + two pointers technique.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public int MinMeetingRooms(int[][] intervals) {
          int n = intervals.Length;
          int[] starts = new int[n];
          int[] ends = new int[n];
          
          for (int i = 0; i < n; i++) {
              starts[i] = intervals[i][0];
              ends[i] = intervals[i][1];
          }
          
          Array.Sort(starts);
          Array.Sort(ends);
          
          int rooms = 0, maxRooms = 0;
          int s = 0, e = 0;
          
          while (s < n) {
              if (starts[s] < ends[e]) {
                  rooms++;
                  maxRooms = Math.Max(maxRooms, rooms);
                  s++;
              } else {
                  rooms--;
                  e++;
              }
          }
          return maxRooms;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N \log N)$, Space: $O(N)$.
  </details>

---

### Day 10: Greedy Character Range Partitioning (Partition Labels)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: In Partition Labels, what defines the minimum partition boundary containing character $C$?</b></summary>
  <br>
  <b>Answer:</b> The partition must extend at least to the **last occurrence index** of character $C$ in the string: <code>lastIndex[c - 'a']</code>.
  </details>
- **Deep Practice (75–85 mins) | Problem 10**:
  - [Partition Labels](https://leetcode.com/problems/partition-labels/) (NeetCode - Medium)
  - Thinking prompts: Pass 1: record last occurrence of each character. Pass 2: extend window boundary `end = Math.Max(end, last[s[i] - 'a'])`. When `i == end`, cut partition!
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public IList<int> PartitionLabels(string s) {
          int[] last = new int[26];
          for (int i = 0; i < s.Length; i++) {
              last[s[i] - 'a'] = i;
          }
          
          List<int> result = new();
          int start = 0, end = 0;
          
          for (int i = 0; i < s.Length; i++) {
              end = Math.Max(end, last[s[i] - 'a']);
              if (i == end) {
                  result.Add(end - start + 1);
                  start = i + 1;
              }
          }
          return result;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N)$, Space: $O(1)$ fixed 26 array.
  </details>

---

### Day 11: CSES & Competitive Benchmarks + Spaced Retrieval
- **Benchmark 1**: [CSES: Movie Festival](https://cses.fi/problemset/task/1629) (Optimal interval scheduling)
- **Benchmark 2**: [CSES: Restaurant Customers](https://cses.fi/problemset/task/1619) (Sweep-line maximum overlapping intervals)
- **Benchmark 3**: [CSES: Tasks and Deadlines](https://cses.fi/problemset/task/1630) (Greedy sorting by duration)
- **Benchmark 4**: [Codeforces 455A - Boredom](https://codeforces.com/problemset/problem/455/A) (Frequency array + House Robber greedy DP)
- **Benchmark 5**: [Codeforces 1352C - K-th Not Divisible by n](https://codeforces.com/problemset/problem/1352/C) (Math greedy offset)

---

### Spaced Retrieval Flashcards

<details>
<summary><b>Flashcard 1: When does a Greedy algorithm yield an optimal solution over Dynamic Programming?</b></summary>
<br>
When the problem exhibits the <b>Greedy Choice Property</b> (a globally optimal solution can be reached by making a locally optimal choice without needing to reconsider or explore alternate branches).
</details>

<details>
<summary><b>Flashcard 2: What is the primary difference in sorting strategy between Merging Intervals and Selecting Non-Overlapping Intervals?</b></summary>
<br>
When <b>merging</b> overlapping intervals, sort by <b>start time</b> to process contiguous sequential blocks. When <b>maximizing non-overlapping intervals</b>, sort by <b>end time</b> to greedily finish tasks as early as possible.
</details>

<details>
<summary><b>Flashcard 3: Why is the two-pointer sweep-line method faster than a Min-Heap for Meeting Rooms II?</b></summary>
<br>
Sorting two plain arrays of size $N$ takes $O(N \log N)$ operations with high CPU cache locality and zero heap allocations. A priority queue performs $N$ separate enqueue/dequeue heap operations with higher per-element branching overhead.
</details>

---

## 🎯 Topic Checklist (15 Problems)

| Day | # | Problem | Platform | Difficulty | Status |
|:---:|:---:|:---|:---:|:---:|:---:|
| Day 1 | 1 | [Maximum Subarray](https://leetcode.com/problems/maximum-subarray/) | NeetCode | Medium | - [ ] |
| Day 2 | 2 | [Jump Game](https://leetcode.com/problems/jump-game/) | NeetCode | Medium | - [ ] |
| Day 3 | 3 | [Jump Game II](https://leetcode.com/problems/jump-game-ii/) | NeetCode | Medium | - [ ] |
| Day 4 | 4 | [Gas Station](https://leetcode.com/problems/gas-station/) | NeetCode | Medium | - [ ] |
| Day 5 | 5 | [Hand of Straights](https://leetcode.com/problems/hand-of-straights/) | NeetCode | Medium | - [ ] |
| Day 6 | 6 | [Merge Intervals](https://leetcode.com/problems/merge-intervals/) | NeetCode | Medium | - [ ] |
| Day 7 | 7 | [Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/) | NeetCode | Medium | - [ ] |
| Day 8 | 8 | [Meeting Rooms](https://leetcode.com/problems/meeting-rooms/) | NeetCode | Easy | - [ ] |
| Day 9 | 9 | [Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii/) | NeetCode | Medium | - [ ] |
| Day 10 | 10 | [Partition Labels](https://leetcode.com/problems/partition-labels/) | NeetCode | Medium | - [ ] |
| Day 11 | 11 | [Movie Festival](https://cses.fi/problemset/task/1629) | CSES | Easy | - [ ] |
| Day 11 | 12 | [Restaurant Customers](https://cses.fi/problemset/task/1619) | CSES | Easy | - [ ] |
| Day 11 | 13 | [Tasks and Deadlines](https://cses.fi/problemset/task/1630) | CSES | Easy | - [ ] |
| Day 11 | 14 | [Boredom (455A)](https://codeforces.com/problemset/problem/455/A) | Codeforces | Rating 1500 | - [ ] |
| Day 11 | 15 | [K-th Not Divisible by n (1352C)](https://codeforces.com/problemset/problem/1352/C) | Codeforces | Rating 1200 | - [ ] |

---

[⬅️ Previous Topic: 2-D Dynamic Programming](13-2d-dp.html) | [Home](index.html) | [Next Topic: Bit Manipulation & Math Fundamentals ➡️](15-bit-manipulation-math.html)

