---
layout: default
title: "Heaps & Priority Queues"
---

# Heaps & Priority Queues

> **Estimated Time**: 11 Days • **Pace**: 1.5–2 hours/day (Strictly 1 problem/day) • **Language**: C# (.NET 6+)

---

## 🛠️ C# Mental Model & Gotchas

### 1. Modern .NET 6+ `PriorityQueue<TElement, TPriority>`
Prior to .NET 6, C# lacked a native heap collection, forcing developers to implement custom binary heaps or use `SortedSet<T>`.
In modern C# (.NET 6+):
```csharp
// Min-heap (default): lowest priority value dequeues first
PriorityQueue<string, int> minHeap = new();

// Max-heap: highest priority value dequeues first
PriorityQueue<int, int> maxHeap = new(Comparer<int>.Create((a, b) => b.CompareTo(a)));
```
- **Internal Structure**: Backed by a 4-ary array-based heap (d-ary heap with $d=4$) providing superior CPU cache locality compared to traditional binary heaps.
- **Complexity**: `Enqueue()` and `Dequeue()` run in $O(\log_4 N)$ time; `Peek()` runs in $O(1)$ time.

### 2. The Fixed-Size $K$ Streaming Invariant
To find the **$K$-th largest** elements in a stream of size $N$:
- Maintain a **Min-Heap** of size $K$.
- For every incoming element, push it into the heap. If `heap.Count > K`, pop the minimum.
- The top of the heap is guaranteed to be the $K$-th largest element in $O(N \log K)$ time and $O(K)$ space.

---

## 📅 Day-by-Day Schedule

### Day 1: Theory Anchor & Baseline Pattern
- **Step 1 (45 mins) | Concept Video**: [Heap - Heap Sort - Heapify](https://www.youtube.com/watch?v=HqPJF2L5h9U) by *Abdul Bari*
  - Key focus: What is the heap property? How does array representation map parent index $i$ to children $2i + 1$ and $2i + 2$?
- **Step 2 (20 mins) | Pen & Paper Invariant**:
  - Insert numbers `10, 4, 15, 20, 0, 30` into a min-heap. Trace the bubble-up operations and show the final array layout.
- **Step 3 (45 mins) | Problem 1 (Warm-Up)**:
  - [Kth Largest Element in a Stream](https://leetcode.com/problems/kth-largest-element-in-a-stream/) (NeetCode - Easy)
  - Objective: Maintain a `PriorityQueue<int, int>` capped at size $K$.
- **Step 4 (10 mins) | Active Recall**:
  <details>
  <summary><b>🧠 Self-Test: Why do we use a MIN-heap (instead of a max-heap) to track the $K$ LARGEST elements?</b></summary>
  <br>
  <b>Answer:</b> In a min-heap of size $K$, the smallest among the top $K$ largest elements sits at the root. Any incoming element smaller than the root is discarded; any larger element replaces the root, preserving the $K$ largest elements seen so far.
  </details>

---

### Day 2: Max-Heap Simulation (Last Stone Weight)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: How do you configure <code>PriorityQueue&lt;int, int&gt;</code> as a Max-Heap in C#?</b></summary>
  <br>
  <b>Answer:</b> By supplying a reverse custom comparer: <code>new PriorityQueue&lt;int, int&gt;(Comparer&lt;int&gt;.Create((a, b) =&gt; b.CompareTo(a)))</code> or by enqueuing with negated priority <code>-val</code>.
  </details>
- **Deep Practice (75–85 mins) | Problem 2**:
  - [Last Stone Weight](https://leetcode.com/problems/last-stone-weight/) (NeetCode - Easy)
  - Thinking prompts: Repeatedly extract the two largest stones. If unequal, re-enqueue the difference.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public int LastStoneWeight(int[] stones) {
          PriorityQueue<int, int> maxHeap = new(Comparer<int>.Create((a, b) => b.CompareTo(a)));
          foreach (int s in stones) {
              maxHeap.Enqueue(s, s);
          }
          
          while (maxHeap.Count > 1) {
              int y = maxHeap.Dequeue();
              int x = maxHeap.Dequeue();
              if (y != x) {
                  int rem = y - x;
                  maxHeap.Enqueue(rem, rem);
              }
          }
          return maxHeap.Count == 0 ? 0 : maxHeap.Dequeue();
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N \log N)$, Space: $O(N)$.
  </details>

---

### Day 3: Custom Geometric Distance Priors (K Closest Points)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: When calculating Euclidean distance $(x^2 + y^2)$ for comparing point closeness, why can we omit <code>Math.Sqrt()</code>?</b></summary>
  <br>
  <b>Answer:</b> The square root function is strictly monotonically increasing. If $x_1^2 + y_1^2 &lt; x_2^2 + y_2^2$, then $\sqrt{x_1^2 + y_1^2} &lt; \sqrt{x_2^2 + y_2^2}$. Omitting <code>Math.Sqrt()</code> eliminates floating-point arithmetic.
  </details>
- **Deep Practice (75–85 mins) | Problem 3**:
  - [K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/) (NeetCode - Medium)
  - Thinking prompts: Priority is squared Euclidean distance $x^2 + y^2$.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public int[][] KClosest(int[][] points, int k) {
          // Max-heap of size K keeping smallest distances
          PriorityQueue<int[], int> maxHeap = new(Comparer<int>.Create((a, b) => b.CompareTo(a)));
          
          foreach (var pt in points) {
              int distSq = pt[0] * pt[0] + pt[1] * pt[1];
              maxHeap.Enqueue(pt, distSq);
              if (maxHeap.Count > k) {
                  maxHeap.Dequeue();
              }
          }
          
          int[][] result = new int[k][];
          for (int i = 0; i < k; i++) {
              result[i] = maxHeap.Dequeue();
          }
          return result;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N \log K)$, Space: $O(K)$.
  </details>

---

### Day 4: QuickSelect vs Min-Heap (Kth Largest Element in an Array)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: What is the average time complexity of QuickSelect compared to a Heap for Top-K?</b></summary>
  <br>
  <b>Answer:</b> QuickSelect achieves $O(N)$ average time (worst-case $O(N^2)$), while a heap provides a deterministic $O(N \log K)$ guarantee.
  </details>
- **Deep Practice (75–85 mins) | Problem 4**:
  - [Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/) (NeetCode - Medium)
  - Thinking prompts: Compare Min-Heap of size $K$ versus Hoare's QuickSelect in-place partitioning.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public int FindKthLargest(int[] nums, int k) {
          PriorityQueue<int, int> minHeap = new();
          foreach (int n in nums) {
              minHeap.Enqueue(n, n);
              if (minHeap.Count > k) {
                  minHeap.Dequeue();
              }
          }
          return minHeap.Peek();
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N \log K)$, Space: $O(K)$.
  </details>

---

### Day 5: Multi-Queue Cooldown Simulation (Task Scheduler)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: In Task Scheduler with cooldown $N$, why do we greedily schedule the most frequent remaining task first?</b></summary>
  <br>
  <b>Answer:</b> High-frequency tasks are the primary bottleneck causing idle slots. Decrementing the highest counts first minimizes the trailing idle time.
  </details>
- **Deep Practice (75–85 mins) | Problem 5**:
  - [Task Scheduler](https://leetcode.com/problems/task-scheduler/) (NeetCode - Medium)
  - Thinking prompts: Use a Max-Heap for available task counts and a regular FIFO `Queue<(int count, int availableTime)>` for tasks on cooldown.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public int LeastInterval(char[] tasks, int n) {
          int[] count = new int[26];
          foreach (char c in tasks) count[c - 'A']++;
          
          PriorityQueue<int, int> maxHeap = new(Comparer<int>.Create((a, b) => b.CompareTo(a)));
          for (int i = 0; i < 26; i++) {
              if (count[i] > 0) maxHeap.Enqueue(count[i], count[i]);
          }
          
          Queue<(int remainingCount, int readyTime)> cooldown = new();
          int time = 0;
          
          while (maxHeap.Count > 0 || cooldown.Count > 0) {
              time++;
              
              if (maxHeap.Count > 0) {
                  int remaining = maxHeap.Dequeue() - 1;
                  if (remaining > 0) {
                      cooldown.Enqueue((remaining, time + n));
                  }
              }
              
              if (cooldown.Count > 0 && cooldown.Peek().readyTime == time) {
                  int taskCount = cooldown.Dequeue().remainingCount;
                  maxHeap.Enqueue(taskCount, taskCount);
              }
          }
          return time;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(\text{Total Intervals})$, Space: $O(1)$ fixed 26 tasks.
  </details>

---

### Day 6: Multi-Stream K-Way Merge (Design Twitter)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: How can a user's feed of the 10 most recent tweets be assembled from multiple followees without concatenating and sorting all tweets?</b></summary>
  <br>
  <b>Answer:</b> Treat each followee's tweet list as a reverse-chronologically sorted stream. Use a max-heap of size equal to the number of followees to run a $K$-way merge, taking only the top 10 tweets.
  </details>
- **Deep Practice (75–85 mins) | Problem 6**:
  - [Design Twitter](https://leetcode.com/problems/design-twitter/) (NeetCode - Medium)
  - Thinking prompts: Maintain global timestamp counter, user follow sets, and a linked list / list of tweets per user.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Twitter {
      private static int _time = 0;
      private readonly Dictionary<int, HashSet<int>> _following = new();
      private readonly Dictionary<int, List<(int time, int tweetId)>> _tweets = new();

      public void PostTweet(int userId, int tweetId) {
          _following.TryAdd(userId, new HashSet<int>());
          if (!_tweets.TryGetValue(userId, out var list)) {
              list = new List<(int, int)>();
              _tweets[userId] = list;
          }
          list.Add((++_time, tweetId));
      }
      
      public IList<int> GetNewsFeed(int userId) {
          List<int> feed = new();
          PriorityQueue<int, int> maxHeap = new(Comparer<int>.Create((a, b) => b.CompareTo(a)));
          
          HashSet<int> sources = new();
          if (_following.TryGetValue(userId, out var followees)) {
              sources.UnionWith(followees);
          }
          sources.Add(userId); // Include self
          
          foreach (int src in sources) {
              if (_tweets.TryGetValue(src, out var list)) {
                  int count = list.Count;
                  for (int i = count - 1; i >= Math.Max(0, count - 10); i--) {
                      maxHeap.Enqueue(list[i].tweetId, list[i].time);
                  }
              }
          }
          
          while (maxHeap.Count > 0 && feed.Count < 10) {
              feed.Add(maxHeap.Dequeue());
          }
          return feed;
      }
      
      public void Follow(int followerId, int followeeId) {
          if (!_following.TryGetValue(followerId, out var set)) {
              set = new HashSet<int>();
              _following[followerId] = set;
          }
          set.Add(followeeId);
      }
      
      public void Unfollow(int followerId, int followeeId) {
          if (_following.TryGetValue(followerId, out var set)) {
              set.Remove(followeeId);
          }
      }
  }
  ```
  <b>Complexity:</b> `PostTweet`: $O(1)$, `GetNewsFeed`: $O(F \log(10 \cdot F))$ where $F$ is followee count. Space: $O(U + T)$.
  </details>

---

### Day 7: Dual-Heap Dynamic Partitioning (Find Median from Data Stream)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: In the two-heap median algorithm, what are the two invariants regarding heap size and element order?</b></summary>
  <br>
  <b>Answer:</b> 1. All elements in <code>smallHeap</code> (Max-Heap) are $\le$ all elements in <code>largeHeap</code> (Min-Heap); 2. The sizes differ by at most 1: <code>smallHeap.Count == largeHeap.Count</code> or <code>smallHeap.Count == largeHeap.Count + 1</code>.
  </details>
- **Deep Practice (75–85 mins) | Problem 7**:
  - [Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/) (NeetCode - Hard)
  - Thinking prompts: Max-heap stores lower half; Min-heap stores upper half.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class MedianFinder {
      private readonly PriorityQueue<int, int> _small; // Max-heap: lower half
      private readonly PriorityQueue<int, int> _large; // Min-heap: upper half

      public MedianFinder() {
          _small = new PriorityQueue<int, int>(Comparer<int>.Create((a, b) => b.CompareTo(a)));
          _large = new PriorityQueue<int, int>();
      }
      
      public void AddNum(int num) {
          _small.Enqueue(num, num);
          
          // Invariant 1: all small <= all large
          if (_small.Count > 0 && _large.Count > 0 && _small.Peek() > _large.Peek()) {
              int val = _small.Dequeue();
              _large.Enqueue(val, val);
          }
          
          // Invariant 2: balance sizes
          if (_small.Count > _large.Count + 1) {
              int val = _small.Dequeue();
              _large.Enqueue(val, val);
          } else if (_large.Count > _small.Count) {
              int val = _large.Dequeue();
              _small.Enqueue(val, val);
          }
      }
      
      public double FindMedian() {
          if (_small.Count > _large.Count) {
              return _small.Peek();
          }
          return (_small.Peek() + _large.Peek()) / 2.0;
      }
  }
  ```
  <b>Complexity:</b> `AddNum`: $O(\log N)$, `FindMedian`: $O(1)$, Space: $O(N)$.
  </details>

---

### Day 8: Greedy Heap with Adjacent Holdback (Reorganize String)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: In Reorganize String, what frequency makes it mathematically impossible to avoid adjacent identical characters?</b></summary>
  <br>
  <b>Answer:</b> If any character appears with frequency $&gt; (N + 1) / 2$.
  </details>
- **Deep Practice (75–85 mins) | Problem 8**:
  - [Reorganize String](https://leetcode.com/problems/reorganize-string/) (LeetCode 767 - Medium)
  - Thinking prompts: Pop most frequent character, append it, hold it in a temporary variable, and re-enqueue it only after another distinct character is appended.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public string ReorganizeString(string s) {
          int[] count = new int[26];
          foreach (char c in s) count[c - 'a']++;
          
          PriorityQueue<char, int> maxHeap = new(Comparer<int>.Create((a, b) => b.CompareTo(a)));
          for (int i = 0; i < 26; i++) {
              if (count[i] > (s.Length + 1) / 2) return "";
              if (count[i] > 0) maxHeap.Enqueue((char)('a' + i), count[i]);
          }
          
          var sb = new System.Text.StringBuilder();
          (char ch, int freq)? prev = null;
          
          while (maxHeap.Count > 0) {
              char curr = maxHeap.Dequeue();
              sb.Append(curr);
              int remFreq = --count[curr - 'a'];
              
              if (prev.HasValue) {
                  maxHeap.Enqueue(prev.Value.ch, prev.Value.freq);
                  prev = null;
              }
              if (remFreq > 0) {
                  prev = (curr, remFreq);
              }
          }
          return sb.ToString();
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N \log 26) = O(N)$, Space: $O(1)$.
  </details>

---

### Day 9: Greedy Opportunity Replacement (Furthest Building You Can Reach)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: In Furthest Building You Can Reach, should ladders be allocated to the largest climbs or smallest climbs?</b></summary>
  <br>
  <b>Answer:</b> Ladders should cover the largest climbs! Maintain a min-heap of size equal to ladder count. When climbs exceed ladder count, pop the smallest climb from the min-heap and pay for it with bricks.
  </details>
- **Deep Practice (75–85 mins) | Problem 9**:
  - [Furthest Building You Can Reach](https://leetcode.com/problems/furthest-building-you-can-reach/) (LeetCode 1642 - Medium)
  - Thinking prompts: Use Min-Heap of climbs capped at size `ladders`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public int FurthestBuilding(int[] heights, int bricks, int ladders) {
          PriorityQueue<int, int> minHeap = new(); // Min-heap of climb heights assigned to ladders
          
          for (int i = 0; i < heights.Length - 1; i++) {
              int climb = heights[i + 1] - heights[i];
              if (climb <= 0) continue;
              
              minHeap.Enqueue(climb, climb);
              if (minHeap.Count > ladders) {
                  // Pay for the smallest climb with bricks
                  int smallestClimb = minHeap.Dequeue();
                  bricks -= smallestClimb;
                  if (bricks < 0) return i;
              }
          }
          return heights.Length - 1;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N \log(\text{ladders}))$, Space: $O(\text{ladders})$.
  </details>

---

### Day 10: Paired Sorting + Min-Heap Fixed Window (Maximum Subsequence Score)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: In Maximum Subsequence Score $(sum(nums1) \times \min(nums2))$, how do we fix the multiplier?</b></summary>
  <br>
  <b>Answer:</b> Sort pairs $(nums1[i], nums2[i])$ descending by $nums2[i]$. As we iterate, the current $nums2[i]$ is guaranteed to be the minimum of all candidates seen so far. We maintain a min-heap of size $K$ of $nums1$ values.
  </details>
- **Deep Practice (75–85 mins) | Problem 10**:
  - [Maximum Subsequence Score](https://leetcode.com/problems/maximum-subsequence-score/) (LeetCode 2542 - Medium)
  - Thinking prompts: Sort by `nums2` descending. Maintain running sum of top $K$ items from `nums1` using min-heap of size $K$.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public long MaxScore(int[] nums1, int[] nums2, int k) {
          int n = nums1.Length;
          (int n1, int n2)[] pairs = new (int, int)[n];
          for (int i = 0; i < n; i++) pairs[i] = (nums1[i], nums2[i]);
          
          // Sort descending by nums2
          Array.Sort(pairs, (a, b) => b.n2.CompareTo(a.n2));
          
          PriorityQueue<int, int> minHeap = new();
          long sum1 = 0;
          long maxScore = 0;
          
          for (int i = 0; i < n; i++) {
              minHeap.Enqueue(pairs[i].n1, pairs[i].n1);
              sum1 += pairs[i].n1;
              
              if (minHeap.Count > k) {
                  sum1 -= minHeap.Dequeue();
              }
              
              if (minHeap.Count == k) {
                  maxScore = Math.Max(maxScore, sum1 * pairs[i].n2);
              }
          }
          return maxScore;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N \log N + N \log K)$, Space: $O(N)$.
  </details>

---

### Day 11: CSES & Competitive Benchmarks + Spaced Retrieval
- **Benchmark 1**: [CSES: Concert Tickets](https://cses.fi/problemset/task/1091) (Binary search / multiset floor query)
- **Benchmark 2**: [CSES: Room Allocation](https://cses.fi/problemset/task/1164) (Min-heap tracking departure times of occupied rooms)
- **Benchmark 3**: [CSES: Movie Festival II](https://cses.fi/problemset/task/1632) (Greedy interval scheduling with multiset / heap)
- **Benchmark 4**: [Codeforces 1352C - K-th Not Divisible by n](https://codeforces.com/problemset/problem/1352/C) (Math invariant)
- **Benchmark 5**: [Codeforces 977D - Divide by three, multiply by two](https://codeforces.com/problemset/problem/977/D) (Topological sort / prime factor ordering)

---

### Spaced Retrieval Flashcards

<details>
<summary><b>Flashcard 1: When should you use a Heap over sorting the entire array?</b></summary>
<br>
When you only need the top $K$ extreme elements from a stream or collection of size $N$, where $K \ll N$. A heap achieves $O(N \log K)$ time and $O(K)$ space, whereas full sorting requires $O(N \log N)$ time and $O(N)$ space.
</details>

<details>
<summary><b>Flashcard 2: Why does .NET 6+ <code>PriorityQueue</code> use a 4-ary heap instead of a binary (2-ary) heap?</b></summary>
<br>
A 4-ary heap has shallower tree depth ($\log_4 N$ vs $\log_2 N$), reducing memory pointer hops and maximizing CPU L1/L2 cache line hits during siftdown operations.
</details>

<details>
<summary><b>Flashcard 3: How does the two-heap pattern find the median in $O(1)$ query time?</b></summary>
<br>
By keeping the lower half of numbers in a max-heap and the upper half in a min-heap. The median is either the top of the larger heap (for odd total counts) or the average of the two heap tops (for even total counts).
</details>

---

## 🎯 Topic Checklist (15 Problems)

| Day | # | Problem | Platform | Difficulty | Status |
|:---:|:---:|:---|:---:|:---:|:---:|
| Day 1 | 1 | [Kth Largest Element in a Stream](https://leetcode.com/problems/kth-largest-element-in-a-stream/) | NeetCode | Easy | - [ ] |
| Day 2 | 2 | [Last Stone Weight](https://leetcode.com/problems/last-stone-weight/) | NeetCode | Easy | - [ ] |
| Day 3 | 3 | [K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/) | NeetCode | Medium | - [ ] |
| Day 4 | 4 | [Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/) | NeetCode | Medium | - [ ] |
| Day 5 | 5 | [Task Scheduler](https://leetcode.com/problems/task-scheduler/) | NeetCode | Medium | - [ ] |
| Day 6 | 6 | [Design Twitter](https://leetcode.com/problems/design-twitter/) | NeetCode | Medium | - [ ] |
| Day 7 | 7 | [Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/) | NeetCode | Hard | - [ ] |
| Day 8 | 8 | [Reorganize String](https://leetcode.com/problems/reorganize-string/) | LeetCode | Medium | - [ ] |
| Day 9 | 9 | [Furthest Building You Can Reach](https://leetcode.com/problems/furthest-building-you-can-reach/) | LeetCode | Medium | - [ ] |
| Day 10 | 10 | [Maximum Subsequence Score](https://leetcode.com/problems/maximum-subsequence-score/) | LeetCode | Medium | - [ ] |
| Day 11 | 11 | [Concert Tickets](https://cses.fi/problemset/task/1091) | CSES | Medium | - [ ] |
| Day 11 | 12 | [Room Allocation](https://cses.fi/problemset/task/1164) | CSES | Medium | - [ ] |
| Day 11 | 13 | [Movie Festival II](https://cses.fi/problemset/task/1632) | CSES | Hard | - [ ] |
| Day 11 | 14 | [K-th Not Divisible by n (1352C)](https://codeforces.com/problemset/problem/1352/C) | Codeforces | Rating 1200 | - [ ] |
| Day 11 | 15 | [Divide by three, multiply by two (977D)](https://codeforces.com/problemset/problem/977/D) | Codeforces | Rating 1000 | - [ ] |

---

[⬅️ Previous Topic: Binary Trees](07-binary-trees.html) | [Home](index.html) | [Next Topic: Backtracking & Recursion ➡️](09-backtracking.html)

