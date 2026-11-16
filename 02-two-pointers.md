---
layout: default
title: "Two Pointers"
---

# Two Pointers

> **Estimated Time**: 11 Days • **Pace**: 1.5–2 hours/day (Strictly 1 problem/day) • **Language**: C# (.NET 6+)

---

## 🛠️ C# Mental Model & Gotchas

### 1. Pointer Mechanics on Arrays & Spans
In C#, two-pointer algorithms typically involve two integer indices (`int left = 0, right = n - 1`). 
- **Array Bounds Checking**: C# performs automatic JIT array bounds checking on index accesses `arr[left]`. When doing pointer arithmetic, ensure `left <= right` or `left < right` loop invariants strictly guard against `IndexOutOfRangeException`.
- **String Inspection Without Garbage**: For string parsing (e.g. palindrome checks), avoid calling `.Replace()`, `.ToLower()`, or `.ToCharArray()`, which allocate new strings on the heap. Instead, use static utility methods directly: `char.IsLetterOrDigit(s[left])` and `char.ToLowerInvariant(s[left])`.
- **In-Place Mutation**: In C#, arrays are reference types passed by value. Mutating `nums[left]` directly updates the caller's array in-place without returning a new array.

### 2. Core Invariants of Two Pointers
- **Opposite-Direction (Inward Squeeze)**: Used when the array is sorted or bounded by extremes. At each step, comparing elements at `left` and `right` allows you to irrevocably discard one side because monotonic order guarantees no better solution exists on that side.
- **Same-Direction (Fast / Slow Runner)**: Used for in-place compaction, partition, or cycle detection. The slow pointer maintains the boundary of processed valid elements, while the fast pointer explores unseen candidates.

---

## 📅 Day-by-Day Schedule

### Day 1: Theory Anchor & Baseline Pattern
- **Step 1 (45 mins) | Concept Video**: [Two Pointer Technique Explained](https://www.youtube.com/watch?v=-gjxg6Pln50) by *NeetCode*
  - Key focus: What mathematical property of a sorted array guarantees that moving the left pointer strictly increases the sum and moving the right pointer strictly decreases it?
- **Step 2 (20 mins) | Pen & Paper Invariant**:
  - Write down the string `"A man, a plan, a canal: Panama"`. Walk two pointers inward, manually skipping non-alphanumeric characters. Note the pointer boundary condition when they meet.
- **Step 3 (45 mins) | Problem 1 (Warm-Up)**:
  - [Valid Palindrome](https://leetcode.com/problems/valid-palindrome/) (NeetCode - Easy)
  - Objective: Implement in C# with $O(1)$ extra space using `char.IsLetterOrDigit` and `char.ToLowerInvariant`.
- **Step 4 (10 mins) | Active Recall**:

<details markdown="1">
<summary><b>🧠 Self-Test: Why should you avoid <code>s.ToLower()</code> and regex replace when checking for palindromes?</b></summary>

<b>Answer:</b> <code>s.ToLower()</code> and <code>Regex.Replace()</code> allocate brand new strings of size $O(N)$ on the managed heap, triggering unnecessary GC allocations. Inspecting characters in-place using two integer pointers runs in $O(N)$ time with $O(1)$ auxiliary memory.

</details>

---

### Day 2: Inward Squeeze on Sorted Space
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: In Two Sum II (1-indexed sorted array), if <code>nums[left] + nums[right] &gt; target</code>, which pointer MUST move, and why?</b></summary>

<b>Answer:</b> Decrement <code>right</code>. Since the array is sorted in ascending order, any pair involving <code>right</code> with an index greater than or equal to <code>left</code> will also exceed <code>target</code>. Decrementing <code>right</code> is the only move that can decrease the sum.

</details>

- **Deep Practice (75–85 mins) | Problem 2**:
  - [Two Sum II - Input Array Is Sorted](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/) (NeetCode - Medium)
  - Thinking prompts: Identify the monotonic property. Show why $O(N)$ two-pointer traversal eliminates the need for an $O(N)$ auxiliary hash map.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int[] TwoSum(int[] numbers, int target) {
        int left = 0;
        int right = numbers.Length - 1;

        while (left < right) {
            int currentSum = numbers[left] + numbers[right];
            if (currentSum == target) {
                return new int[] { left + 1, right + 1 }; // 1-indexed
            } else if (currentSum < target) {
                left++;
            } else {
                right--;
            }
        }
        return Array.Empty<int>();
    }
}
```

<b>Complexity:</b> Time: $O(N)$, Space: $O(1)$.

</details>

---

### Day 3: Deduplication & 3-Pointer Anchoring
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: How do you prevent duplicate triplets in 3Sum without using a <code>HashSet</code> of tuples?</b></summary>

<b>Answer:</b> Sort the array first. When fixing the outer element $i$, skip if <code>i &gt; 0 &amp;&amp; nums[i] == nums[i - 1]</code>. When moving the two inner pointers after finding a triplet, skip duplicate adjacent values: <code>while (left &lt; right &amp;&amp; nums[left] == nums[left + 1]) left++;</code>.

</details>

- **Deep Practice (75–85 mins) | Problem 3**:
  - [3Sum](https://leetcode.com/problems/3sum/) (NeetCode - Medium)
  - Thinking prompts: Sort the array ($O(N \log N)$). Anchor index $i$, then execute Two Sum II inward squeeze on the remaining range $[i + 1, N - 1]$.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public IList<IList<int>> ThreeSum(int[] nums) {
        Array.Sort(nums);
        List<IList<int>> result = new();
        int n = nums.Length;

        for (int i = 0; i < n - 2; i++) {
            if (nums[i] > 0) break; // Cannot sum to 0 if smallest number > 0
            if (i > 0 && nums[i] == nums[i - 1]) continue; // Skip duplicate outer

            int left = i + 1;
            int right = n - 1;

            while (left < right) {
                int sum = nums[i] + nums[left] + nums[right];
                if (sum == 0) {
                    result.Add(new List<int> { nums[i], nums[left], nums[right] });
                    while (left < right && nums[left] == nums[left + 1]) left++;
                    while (left < right && nums[right] == nums[right - 1]) right--;
                    left++;
                    right--;
                } else if (sum < 0) {
                    left++;
                } else {
                    right--;
                }
            }
        }
        return result;
    }
}
```

<b>Complexity:</b> Time: $O(N^2)$, Space: $O(1)$ auxiliary (or $O(\log N)$ for sorting stack).

</details>

---

### Day 4: Greedy Capacity Bounds (Container With Most Water)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: In Container With Most Water, why do we always advance the pointer pointing to the shorter vertical bar?</b></summary>

<b>Answer:</b> The water area is bounded by $\min(h[L], h[R]) \times (R - L)$. Moving the taller bar only shrinks the width while being strictly capped by the shorter bar. Thus, advancing the shorter bar is the only choice that could potentially yield a larger area.

</details>

- **Deep Practice (75–85 mins) | Problem 4**:
  - [Container With Most Water](https://leetcode.com/problems/container-with-most-water/) (NeetCode - Medium)
  - Thinking prompts: Prove that no optimal solution is missed by discarding the shorter wall at each iteration.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int MaxArea(int[] height) {
        int left = 0;
        int right = height.Length - 1;
        int maxWater = 0;

        while (left < right) {
            int hLeft = height[left];
            int hRight = height[right];
            int currentWater = Math.Min(hLeft, hRight) * (right - left);
            if (currentWater > maxWater) {
                maxWater = currentWater;
            }

            if (hLeft < hRight) {
                left++;
            } else {
                right--;
            }
        }
        return maxWater;
    }
}
```

<b>Complexity:</b> Time: $O(N)$, Space: $O(1)$.

</details>

---

### Day 5: Two-Pointer Prefix/Suffix Extrema (Trapping Rain Water)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: What is the formula for trapped water at index $i$?</b></summary>

<b>Answer:</b> $\max(0, \min(\text{max\_left}[i], \text{max\_right}[i]) - \text{height}[i])$.

</details>

- **Deep Practice (75–85 mins) | Problem 5**:
  - [Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/) (NeetCode - Hard)
  - Thinking prompts: Instead of allocating two $O(N)$ arrays for prefix and suffix maximums, maintain `leftMax` and `rightMax` variables. Move whichever pointer has the smaller current max boundary.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int Trap(int[] height) {
        if (height.Length == 0) return 0;

        int left = 0, right = height.Length - 1;
        int leftMax = 0, rightMax = 0;
        int totalWater = 0;

        while (left < right) {
            if (height[left] <= height[right]) {
                if (height[left] >= leftMax) {
                    leftMax = height[left];
                } else {
                    totalWater += leftMax - height[left];
                }
                left++;
            } else {
                if (height[right] >= rightMax) {
                    rightMax = height[right];
                } else {
                    totalWater += rightMax - height[right];
                }
                right--;
            }
        }
        return totalWater;
    }
}
```

<b>Complexity:</b> Time: $O(N)$, Space: $O(1)$ auxiliary.

</details>

---

### Day 6: Dutch National Flag 3-Way Partitioning
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: In the Dutch National Flag algorithm, what do the three pointers <code>low</code>, <code>mid</code>, and <code>high</code> represent?</b></summary>

<b>Answer:</b> Invariants: elements in $[0 \dots low - 1]$ are 0; elements in $[low \dots mid - 1]$ are 1; elements in $[mid \dots high]$ are unknown; elements in $[high + 1 \dots n - 1]$ are 2.

</details>

- **Deep Practice (75–85 mins) | Problem 6**:
  - [Sort Colors](https://leetcode.com/problems/sort-colors/) (NeetCode - Medium)
  - Thinking prompts: Why do we increment `mid` when swapping with `low`, but NOT when swapping with `high`? (Because the element coming from `high` is unexamined!).
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public void SortColors(int[] nums) {
        int low = 0, mid = 0, high = nums.Length - 1;

        while (mid <= high) {
            if (nums[mid] == 0) {
                (nums[low], nums[mid]) = (nums[mid], nums[low]);
                low++;
                mid++;
            } else if (nums[mid] == 1) {
                mid++;
            } else { // nums[mid] == 2
                (nums[mid], nums[high]) = (nums[high], nums[mid]);
                high--;
                // Do not increment mid here: swapped element from high must be inspected
            }
        }
    }
}
```

<b>Complexity:</b> Time: $O(N)$ single pass, Space: $O(1)$ in-place swaps.

</details>

---

### Day 7: Fast & Slow In-Place Array Compaction
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: In fast & slow pointer compaction, what is the invariant of the slow pointer?</b></summary>

<b>Answer:</b> <code>slow</code> points to the index where the next valid, non-duplicate element should be written. All elements in $[0 \dots slow - 1]$ satisfy the required uniqueness/ordering invariant.

</details>

- **Deep Practice (75–85 mins) | Problem 7**:
  - [Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/) (NeetCode - Easy)
  - Thinking prompts: Maintain `slow = 1`. Iterate `fast` from 1 to $N - 1$. If `nums[fast] != nums[fast - 1]`, write `nums[slow] = nums[fast]` and increment `slow`.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int RemoveDuplicates(int[] nums) {
        if (nums.Length == 0) return 0;

        int slow = 1;
        for (int fast = 1; fast < nums.Length; fast++) {
            if (nums[fast] != nums[fast - 1]) {
                nums[slow] = nums[fast];
                slow++;
            }
        }
        return slow;
    }
}
```

<b>Complexity:</b> Time: $O(N)$, Space: $O(1)$.

</details>

---

### Day 8: Stable Compaction & Zero Relocation
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: How can Move Zeroes maintain relative ordering without extra space?</b></summary>

<b>Answer:</b> Maintain <code>slow = 0</code>. Whenever <code>nums[fast] != 0</code>, swap <code>nums[slow]</code> and <code>nums[fast]</code>, then increment <code>slow</code>.

</details>

- **Deep Practice (75–85 mins) | Problem 8**:
  - [Move Zeroes](https://leetcode.com/problems/move-zeroes/) (NeetCode - Easy)
  - Thinking prompts: Compare two-pass (overwrite with non-zeros then fill rest with zeros) versus single-pass swap.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public void MoveZeroes(int[] nums) {
        int slow = 0;
        for (int fast = 0; fast < nums.Length; fast++) {
            if (nums[fast] != 0) {
                if (slow != fast) {
                    (nums[slow], nums[fast]) = (nums[fast], nums[slow]);
                }
                slow++;
            }
        }
    }
}
```

<b>Complexity:</b> Time: $O(N)$, Space: $O(1)$.

</details>

---

### Day 9: Multi-Pointer Recursive Generalization (4Sum)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: What arithmetic overflow must you safeguard against when summing 4 integers in C#?</b></summary>

<b>Answer:</b> Adding four 32-bit signed integers can exceed <code>int.MaxValue</code> ($2 \times 10^9$). Cast intermediate sums to <code>long</code>.

</details>

- **Deep Practice (75–85 mins) | Problem 9**:
  - [4Sum](https://leetcode.com/problems/4sum/) (NeetCode - Medium)
  - Thinking prompts: Generalize 3Sum by nesting two outer loops $(i, j)$ and reducing the inner search to Two Sum II. Ensure duplicates are skipped at both outer loop levels.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public IList<IList<int>> FourSum(int[] nums, int target) {
        Array.Sort(nums);
        List<IList<int>> result = new();
        int n = nums.Length;

        for (int i = 0; i < n - 3; i++) {
            if (i > 0 && nums[i] == nums[i - 1]) continue;

            for (int j = i + 1; j < n - 2; j++) {
                if (j > i + 1 && nums[j] == nums[j - 1]) continue;

                int left = j + 1;
                int right = n - 1;
                long reqTarget = (long)target - nums[i] - nums[j];

                while (left < right) {
                    long sum = (long)nums[left] + nums[right];
                    if (sum == reqTarget) {
                        result.Add(new List<int> { nums[i], nums[j], nums[left], nums[right] });
                        while (left < right && nums[left] == nums[left + 1]) left++;
                        while (left < right && nums[right] == nums[right - 1]) right--;
                        left++;
                        right--;
                    } else if (sum < reqTarget) {
                        left++;
                    } else {
                        right--;
                    }
                }
            }
        }
        return result;
    }
}
```

<b>Complexity:</b> Time: $O(N^3)$, Space: $O(1)$ auxiliary.

</details>

---

### Day 10: Greedy Pairing with Two Pointers
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: In Boats to Save People (capacity $\le 2$ people, weight $\le limit$), why pair the heaviest person with the lightest person?</b></summary>

<b>Answer:</b> The heaviest person is the hardest to accommodate. If even the lightest person cannot share a boat with them, no one can, so the heaviest person must ride alone.

</details>

- **Deep Practice (75–85 mins) | Problem 10**:
  - [Boats to Save People](https://leetcode.com/problems/boats-to-save-people/) (LeetCode 881 - Medium)
  - Thinking prompts: Sort array. If `people[left] + people[right] <= limit`, both board (`left++`, `right--`). Otherwise, only the heavy person boards (`right--`).
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int NumRescueBoats(int[] people, int limit) {
        Array.Sort(people);
        int left = 0;
        int right = people.Length - 1;
        int boats = 0;

        while (left <= right) {
            if (people[left] + people[right] <= limit) {
                left++;
            }
            right--;
            boats++;
        }
        return boats;
    }
}
```

<b>Complexity:</b> Time: $O(N \log N)$ due to sorting, Space: $O(1)$.

</details>

---

### Day 11: CSES & Competitive Benchmarks + Spaced Retrieval
- **Benchmark 1**: [CSES: Sum of Two Values](https://cses.fi/problemset/task/1640) (Coordinate compression / sorting + two pointers)
- **Benchmark 2**: [CSES: Sum of Three Values](https://cses.fi/problemset/task/1641) (3Sum on $1$-based indices)
- **Benchmark 3**: [CSES: Nearest Smaller Values](https://cses.fi/problemset/task/1645) (Two pointer vs monotonic preview)
- **Benchmark 4**: [Codeforces 279B - Books](https://codeforces.com/problemset/problem/279/B) (Two pointers with prefix sum limit)
- **Benchmark 5**: [Codeforces 1619C - Wrong Addition](https://codeforces.com/problemset/problem/1619/C) (Right-to-left two pointer digit matching)

---

### Spaced Retrieval Flashcards

<details markdown="1">
<summary><b>Flashcard 1: When does the Two Pointer inward squeeze fail on an array?</b></summary>

When the array is unsorted and lacks a monotonic trade-off property (e.g. elements can be arbitrary positive/negative values). Without monotonicity, moving one pointer cannot guarantee the elimination of all pairs on that side.
</details>

<details markdown="1">
<summary><b>Flashcard 2: In Dutch National Flag 3-way partitioning, why is <code>mid</code> NOT incremented after swapping with <code>high</code>?</b></summary>

The element swapped from index <code>high</code> came from an unexamined region. It could be a 0, 1, or 2, and therefore must be processed at index <code>mid</code> during the next loop iteration.
</details>

<details markdown="1">
<summary><b>Flashcard 3: How do you swap two variables in C# without a temporary variable?</b></summary>

Using C# ValueTuple deconstruction: <code>(nums[i], nums[j]) = (nums[j], nums[i]);</code>. The C# compiler emits efficient IL that loads both values onto the evaluation stack and stores them without allocating heap objects.
</details>

---

## 🎯 Topic Checklist (15 Problems)

| Day | # | Problem | Platform | Difficulty | Status |
|:---:|:---:|:---|:---:|:---:|:---:|
| Day 1 | 1 | [Valid Palindrome](https://leetcode.com/problems/valid-palindrome/) | NeetCode | Easy | - [ ] |
| Day 2 | 2 | [Two Sum II - Input Array Is Sorted](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/) | NeetCode | Medium | - [ ] |
| Day 3 | 3 | [3Sum](https://leetcode.com/problems/3sum/) | NeetCode | Medium | - [ ] |
| Day 4 | 4 | [Container With Most Water](https://leetcode.com/problems/container-with-most-water/) | NeetCode | Medium | - [ ] |
| Day 5 | 5 | [Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/) | NeetCode | Hard | - [ ] |
| Day 6 | 6 | [Sort Colors](https://leetcode.com/problems/sort-colors/) | NeetCode | Medium | - [ ] |
| Day 7 | 7 | [Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/) | NeetCode | Easy | - [ ] |
| Day 8 | 8 | [Move Zeroes](https://leetcode.com/problems/move-zeroes/) | NeetCode | Easy | - [ ] |
| Day 9 | 9 | [4Sum](https://leetcode.com/problems/4sum/) | NeetCode | Medium | - [ ] |
| Day 10 | 10 | [Boats to Save People](https://leetcode.com/problems/boats-to-save-people/) | LeetCode | Medium | - [ ] |
| Day 11 | 11 | [Sum of Two Values](https://cses.fi/problemset/task/1640) | CSES | Easy | - [ ] |
| Day 11 | 12 | [Sum of Three Values](https://cses.fi/problemset/task/1641) | CSES | Medium | - [ ] |
| Day 11 | 13 | [Nearest Smaller Values](https://cses.fi/problemset/task/1645) | CSES | Medium | - [ ] |
| Day 11 | 14 | [Books (279B)](https://codeforces.com/problemset/problem/279/B) | Codeforces | Rating 1100 | - [ ] |
| Day 11 | 15 | [Wrong Addition (1619C)](https://codeforces.com/problemset/problem/1619/C) | Codeforces | Rating 1200 | - [ ] |

---

[⬅️ Previous Topic: Arrays & Hashing](01-arrays-and-hashing.html) | [Home](index.html) | [Next Topic: Sliding Window ➡️](03-sliding-window.html)
