---
layout: default
title: "Sliding Window"
---

# Sliding Window

> **Estimated Time**: 11 Days • **Pace**: 1.5–2 hours/day (Strictly 1 problem/day) • **Language**: C# (.NET 6+)

---

## 🛠️ C# Mental Model & Gotchas

### 1. Fixed vs Dynamic Window Mechanics
- **Fixed Window**: The window size $K$ is invariant. Expand until length equals $K$, then slide forward by adding element at `right` and subtracting element at `right - K`.
- **Dynamic Window**: The window grows greedily (`right++`) until an invariant is violated (e.g. more than $K$ distinct elements or character frequency mismatch). Then shrink from the left (`left++`) until validity is restored.

### 2. High-Performance Frequency Tracking in C#
- Avoid calling `Dictionary.Count` or scanning the 26-element array inside every loop iteration ($O(26)$ per step).
- Use an integer `matches` counter. When adding `s[right]`, if its frequency matches the target, increment `matches`. When shrinking `left`, update `matches` accordingly. Check `if (matches == required)` in instantaneous $O(1)$ time.
- For standard ASCII strings, use `int[] map = new int[128]` to map characters directly to their last seen positions or frequencies without hash map allocations.

---

## 📅 Day-by-Day Schedule

### Day 1: Theory Anchor & Baseline Pattern
- **Step 1 (45 mins) | Concept Video**: [Sliding Window Algorithm Tutorial](https://www.youtube.com/watch?v=wiGpQwVHdE0) by *NeetCode*
  - Key focus: What is the window invariant? Why does the right pointer never reset backwards ($O(N)$ amortized guarantee)?
- **Step 2 (20 mins) | Pen & Paper Invariant**:
  - Sketch prices array `[7, 1, 5, 3, 6, 4]`. Track `minPrice` and compute maximum profit at each day $i$. Formulate the invariant: at day $i$, what is the optimal previous buying day?
- **Step 3 (45 mins) | Problem 1 (Warm-Up)**:
  - [Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/) (NeetCode - Easy)
  - Objective: Single pass tracking running minimum and max profit.
- **Step 4 (10 mins) | Active Recall**:

<details markdown="1">
<summary><b>🧠 Self-Test: Why is Sliding Window guaranteed to be $O(N)$ when there is a nested <code>while</code> loop?</b></summary>

<b>Answer:</b> Because both the <code>left</code> and <code>right</code> pointers only increment forward from $0$ to $N - 1$. Each element enters the window once via <code>right</code> and leaves at most once via <code>left</code>. The amortized number of pointer movements is at most $2N$, which is strictly $O(N)$.

</details>

---

### Day 2: Dynamic Window with Last-Seen Index Map
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: When a duplicate character $c$ is encountered at <code>right</code>, why can we jump <code>left = Math.Max(left, lastSeen[c] + 1)</code> instead of incrementing <code>left</code> one step at a time?</b></summary>

<b>Answer:</b> Any window starting before or at <code>lastSeen[c]</code> is guaranteed to contain the duplicate character. Taking <code>Math.Max(left, lastSeen[c] + 1)</code> skips directly past the duplicate while ensuring <code>left</code> never moves backwards.

</details>

- **Deep Practice (75–85 mins) | Problem 2**:
  - [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/) (NeetCode - Medium)
  - Thinking prompts: Use `int[] lastSeen = new int[128]` initialized to `-1`. Trace string `"pwwkew"` on paper.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int LengthOfLongestSubstring(string s) {
        int[] lastPos = new int[128];
        Array.Fill(lastPos, -1);

        int maxLen = 0;
        int left = 0;

        for (int right = 0; right < s.Length; right++) {
            char c = s[right];
            if (lastPos[c] >= left) {
                left = lastPos[c] + 1;
            }
            lastPos[c] = right;
            maxLen = Math.Max(maxLen, right - left + 1);
        }
        return maxLen;
    }
}
```

<b>Complexity:</b> Time: $O(N)$ single pass, Space: $O(1)$ fixed 128-element table.

</details>

---

### Day 3: Window Feasibility Invariant (Max Frequency + K)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: In Character Replacement, what is the mathematical formula that proves a window $[L, R]$ can be converted into identical characters with at most $K$ operations?</b></summary>

<b>Answer:</b> <code>(windowLength - maxFrequency) &lt;= K</code>. The non-majority characters count is $(R - L + 1) - \text{maxFreq}$. If this is $\le K$, we can flip all of them.

</details>

- **Deep Practice (75–85 mins) | Problem 3**:
  - [Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/) (NeetCode - Medium)
  - Thinking prompts: Notice that `maxFreq` does not need to decrease when the window shrinks, because a smaller window will never beat our global max length record!
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int CharacterReplacement(string s, int k) {
        int[] count = new int[26];
        int maxFreq = 0;
        int maxLen = 0;
        int left = 0;

        for (int right = 0; right < s.Length; right++) {
            count[s[right] - 'A']++;
            maxFreq = Math.Max(maxFreq, count[s[right] - 'A']);

            // If remaining characters to flip exceed K, shrink window
            while ((right - left + 1) - maxFreq > k) {
                count[s[left] - 'A']--;
                left++;
            }

            maxLen = Math.Max(maxLen, right - left + 1);
        }
        return maxLen;
    }
}
```

<b>Complexity:</b> Time: $O(N)$, Space: $O(1)$ fixed 26-element array.

</details>

---

### Day 4: Fixed Window with Match Counting
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: In Permutation in String, what is the fixed size of the window?</b></summary>

<b>Answer:</b> Exactly <code>s1.Length</code>. We maintain character count equality between <code>s1</code> and the sliding window over <code>s2</code> of length <code>s1.Length</code>.

</details>

- **Deep Practice (75–85 mins) | Problem 4**:
  - [Permutation in String](https://leetcode.com/problems/permutation-in-string/) (NeetCode - Medium)
  - Thinking prompts: Initialize frequencies of `s1`. Slide window across `s2`. Instead of comparing entire 26-element arrays every step, track a single variable `matches` (0 to 26).
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public bool CheckInclusion(string s1, string s2) {
        if (s1.Length > s2.Length) return false;

        int[] count1 = new int[26];
        int[] count2 = new int[26];
        for (int i = 0; i < s1.Length; i++) {
            count1[s1[i] - 'a']++;
            count2[s2[i] - 'a']++;
        }

        int matches = 0;
        for (int i = 0; i < 26; i++) {
            if (count1[i] == count2[i]) matches++;
        }

        int left = 0;
        for (int right = s1.Length; right < s2.Length; right++) {
            if (matches == 26) return true;

            // Add right character
            int rIdx = s2[right] - 'a';
            count2[rIdx]++;
            if (count2[rIdx] == count1[rIdx]) {
                matches++;
            } else if (count2[rIdx] == count1[rIdx] + 1) {
                matches--;
            }

            // Remove left character
            int lIdx = s2[left] - 'a';
            count2[lIdx]--;
            if (count2[lIdx] == count1[lIdx]) {
                matches++;
            } else if (count2[lIdx] == count1[lIdx] - 1) {
                matches--;
            }
            left++;
        }
        return matches == 26;
    }
}
```

<b>Complexity:</b> Time: $O(N)$, Space: $O(1)$.

</details>

---

### Day 5: Minimum Window Expansion & Contraction
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: In Minimum Window Substring, how do we know the current window $[L, R]$ contains all required characters of string $T$?</b></summary>

<b>Answer:</b> Maintain <code>have</code> and <code>need</code> counts. <code>need</code> is the count of unique characters in $T$. When a character's window count reaches its required count in $T$, increment <code>have</code>. The window is valid when <code>have == need</code>.

</details>

- **Deep Practice (75–85 mins) | Problem 5**:
  - [Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/) (NeetCode - Hard)
  - Thinking prompts: Expand `right` until valid (`have == need`). Once valid, shrink `left` as much as possible while maintaining validity to find the minimum substring.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public string MinWindow(string s, string t) {
        if (s.Length == 0 || t.Length == 0) return "";

        int[] targetCount = new int[128];
        int requiredChars = 0;
        foreach (char c in t) {
            if (targetCount[c] == 0) requiredChars++;
            targetCount[c]++;
        }

        int[] windowCount = new int[128];
        int have = 0;
        int minLen = int.MaxValue;
        int startIdx = 0;
        int left = 0;

        for (int right = 0; right < s.Length; right++) {
            char c = s[right];
            windowCount[c]++;
            if (targetCount[c] > 0 && windowCount[c] == targetCount[c]) {
                have++;
            }

            while (have == requiredChars) {
                if (right - left + 1 < minLen) {
                    minLen = right - left + 1;
                    startIdx = left;
                }

                char leftChar = s[left];
                windowCount[leftChar]--;
                if (targetCount[leftChar] > 0 && windowCount[leftChar] < targetCount[leftChar]) {
                    have--;
                }
                left++;
            }
        }
        return minLen == int.MaxValue ? "" : s.Substring(startIdx, minLen);
    }
}
```

<b>Complexity:</b> Time: $O(N + M)$, Space: $O(1)$ fixed 128 arrays.

</details>

---

### Day 6: Monotonic Deque Window (Sliding Window Maximum)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: Why is a regular heap $O(N \log K)$ for Sliding Window Maximum, and how does a Monotonic Deque achieve $O(N)$?</b></summary>

<b>Answer:</b> A heap cannot remove elements leaving the window from the other side in $O(1)$. A monotonic double-ended queue (storing indices) maintains elements in strictly decreasing order: elements smaller than the incoming element are popped from the back because they can never be the maximum again.

</details>

- **Deep Practice (75–85 mins) | Problem 6**:
  - [Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/) (NeetCode - Hard)
  - Thinking prompts: Use `LinkedList<int>` as a double-ended deque storing indices. The front is always the maximum for the current window.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int[] MaxSlidingWindow(int[] nums, int k) {
        int n = nums.Length;
        int[] res = new int[n - k + 1];
        LinkedList<int> deque = new(); // Stores indices

        for (int i = 0; i < n; i++) {
            // Remove indices outside the current window
            if (deque.Count > 0 && deque.First.Value < i - k + 1) {
                deque.RemoveFirst();
            }

            // Maintain decreasing order: remove smaller elements from back
            while (deque.Count > 0 && nums[deque.Last.Value] < nums[i]) {
                deque.RemoveLast();
            }

            deque.AddLast(i);

            // Record answer once first window is formed
            if (i >= k - 1) {
                res[i - k + 1] = nums[deque.First.Value];
            }
        }
        return res;
    }
}
```

<b>Complexity:</b> Time: $O(N)$ (each index added and removed at most once), Space: $O(K)$.

</details>

---

### Day 7: Dynamic Sum Boundaries (Minimum Size Subarray Sum)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: Can Sliding Window be used for Minimum Size Subarray Sum if array elements can be negative?</b></summary>

<b>Answer:</b> No! Sliding Window requires non-negative elements so that moving <code>right</code> monotonically increases the sum and moving <code>left</code> monotonically decreases it. For negative numbers, prefix sums with a monotonic deque or hash map must be used.

</details>

- **Deep Practice (75–85 mins) | Problem 7**:
  - [Minimum Size Subarray Sum](https://leetcode.com/problems/minimum-size-subarray-sum/) (LeetCode 209 - Medium)
  - Thinking prompts: Accumulate `sum += nums[right]`. While `sum >= target`, record length `right - left + 1`, then subtract `nums[left++]`.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int MinSubArrayLen(int target, int[] nums) {
        int minLen = int.MaxValue;
        int sum = 0;
        int left = 0;

        for (int right = 0; right < nums.Length; right++) {
            sum += nums[right];
            while (sum >= target) {
                minLen = Math.Min(minLen, right - left + 1);
                sum -= nums[left++];
            }
        }
        return minLen == int.MaxValue ? 0 : minLen;
    }
}
```

<b>Complexity:</b> Time: $O(N)$, Space: $O(1)$.

</details>

---

### Day 8: Counting Subarrays by Window Length
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: In Subarray Product Less Than K, how many new valid subarrays ending at index <code>right</code> are added when a window $[L, R]$ is valid?</b></summary>

<b>Answer:</b> Exactly <code>right - left + 1</code> subarrays. Every contiguous slice ending at <code>right</code> starting from any index between $L$ and $R$ has product strictly less than $K$.

</details>

- **Deep Practice (75–85 mins) | Problem 8**:
  - [Subarray Product Less Than K](https://leetcode.com/problems/subarray-product-less-than-k/) (LeetCode 713 - Medium)
  - Thinking prompts: Edge case: if $k \le 1$, return 0 immediately because array elements are positive integers $\ge 1$.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int NumSubarrayProductLessThanK(int[] nums, int k) {
        if (k <= 1) return 0;

        int count = 0;
        long product = 1;
        int left = 0;

        for (int right = 0; right < nums.Length; right++) {
            product *= nums[right];
            while (product >= k) {
                product /= nums[left++];
            }
            count += (right - left + 1);
        }
        return count;
    }
}
```

<b>Complexity:</b> Time: $O(N)$, Space: $O(1)$.

</details>

---

### Day 9: Binary Substring Inversion (Max Consecutive Ones III)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: How is Max Consecutive Ones III equivalent to Longest Substring with at most $K$ zeros?</b></summary>

<b>Answer:</b> Flipping at most $K$ zeros into ones means finding the longest subarray containing at most $K$ zero values.

</details>

- **Deep Practice (75–85 mins) | Problem 9**:
  - [Max Consecutive Ones III](https://leetcode.com/problems/max-consecutive-ones-iii/) (LeetCode 1004 - Medium)
  - Thinking prompts: Expand window `right`. If `nums[right] == 0`, increment `zeroCount`. If `zeroCount > k`, shrink `left` until `zeroCount <= k`.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int MaxConsecutiveOnes(int[] nums, int k) {
        int left = 0;
        int zeroCount = 0;
        int maxLen = 0;

        for (int right = 0; right < nums.Length; right++) {
            if (nums[right] == 0) zeroCount++;

            while (zeroCount > k) {
                if (nums[left] == 0) zeroCount--;
                left++;
            }
            maxLen = Math.Max(maxLen, right - left + 1);
        }
        return maxLen;
    }
}
```

<b>Complexity:</b> Time: $O(N)$, Space: $O(1)$.

</details>

---

### Day 10: Multi-State Sliding Window (Fruit Into Baskets)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: What is the core invariant of Fruit Into Baskets?</b></summary>

<b>Answer:</b> Find the length of the longest contiguous subarray that contains at most 2 distinct integer types.

</details>

- **Deep Practice (75–85 mins) | Problem 10**:
  - [Fruit Into Baskets](https://leetcode.com/problems/fruit-into-baskets/) (LeetCode 904 - Medium)
  - Thinking prompts: Maintain `Dictionary<int, int>` of element frequencies. While `count.Count > 2`, decrement `fruits[left]` and remove when frequency hits 0.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int TotalFruit(int[] fruits) {
        Dictionary<int, int> basket = new();
        int left = 0;
        int maxFruits = 0;

        for (int right = 0; right < fruits.Length; right++) {
            basket[fruits[right]] = basket.GetValueOrDefault(fruits[right], 0) + 1;

            while (basket.Count > 2) {
                int fruit = fruits[left];
                basket[fruit]--;
                if (basket[fruit] == 0) {
                    basket.Remove(fruit);
                }
                left++;
            }
            maxFruits = Math.Max(maxFruits, right - left + 1);
        }
        return maxFruits;
    }
}
```

<b>Complexity:</b> Time: $O(N)$, Space: $O(1)$ (at most 3 keys in hash map).

</details>

---

### Day 11: CSES & Competitive Benchmarks + Spaced Retrieval
- **Benchmark 1**: [CSES: Playlist](https://cses.fi/problemset/task/1141) (Longest contiguous subarray with unique songs)
- **Benchmark 2**: [CSES: Subarray Sums II](https://cses.fi/problemset/task/1661) (Subarray sums with negative values: hash map vs window)
- **Benchmark 3**: [CSES: Towers](https://cses.fi/problemset/task/1073) (Greedy stack / multiset)
- **Benchmark 4**: [Codeforces 1352C - K-th Not Divisible by n](https://codeforces.com/problemset/problem/1352/C) (Math / index space)
- **Benchmark 5**: [Codeforces 1360E - Polygon](https://codeforces.com/problemset/problem/1360/E) (Grid boundary invariants)

---

### Spaced Retrieval Flashcards

<details markdown="1">
<summary><b>Flashcard 1: Why does Sliding Window fail to find a subarray with sum $K$ when negative numbers are present?</b></summary>

Monotonicity is destroyed: incrementing <code>right</code> might decrease the sum, and incrementing <code>left</code> might increase the sum. We cannot make greedy contraction decisions. In this case, prefix sum hashing ($O(N)$ time and space) must be used.
</details>

<details markdown="1">
<summary><b>Flashcard 2: How does tracking an integer <code>matches</code> variable speed up string permutation matching?</b></summary>

Instead of comparing two 26-element arrays on every step ($26 \times N$ operations), we only update <code>matches</code> when a character frequency transitions to or from its target count. Checking validity becomes a single $O(1)$ equality check: <code>matches == 26</code>.
</details>

<details markdown="1">
<summary><b>Flashcard 3: What is the optimal C# collection for a sliding window maximum query in $O(1)$ amortized time?</b></summary>

A double-ended queue (deque) implemented via <code>LinkedList&lt;int&gt;</code> storing array indices, maintained in monotonically decreasing order of values.
</details>

---

## 🎯 Topic Checklist (15 Problems)

| Day | # | Problem | Platform | Difficulty | Status |
|:---:|:---:|:---|:---:|:---:|:---:|
| Day 1 | 1 | [Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/) | NeetCode | Easy | - [ ] |
| Day 2 | 2 | [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/) | NeetCode | Medium | - [ ] |
| Day 3 | 3 | [Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/) | NeetCode | Medium | - [ ] |
| Day 4 | 4 | [Permutation in String](https://leetcode.com/problems/permutation-in-string/) | NeetCode | Medium | - [ ] |
| Day 5 | 5 | [Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/) | NeetCode | Hard | - [ ] |
| Day 6 | 6 | [Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/) | NeetCode | Hard | - [ ] |
| Day 7 | 7 | [Minimum Size Subarray Sum](https://leetcode.com/problems/minimum-size-subarray-sum/) | LeetCode | Medium | - [ ] |
| Day 8 | 8 | [Subarray Product Less Than K](https://leetcode.com/problems/subarray-product-less-than-k/) | LeetCode | Medium | - [ ] |
| Day 9 | 9 | [Max Consecutive Ones III](https://leetcode.com/problems/max-consecutive-ones-iii/) | LeetCode | Medium | - [ ] |
| Day 10 | 10 | [Fruit Into Baskets](https://leetcode.com/problems/fruit-into-baskets/) | LeetCode | Medium | - [ ] |
| Day 11 | 11 | [Playlist](https://cses.fi/problemset/task/1141) | CSES | Medium | - [ ] |
| Day 11 | 12 | [Subarray Sums II](https://cses.fi/problemset/task/1661) | CSES | Medium | - [ ] |
| Day 11 | 13 | [Towers](https://cses.fi/problemset/task/1073) | CSES | Medium | - [ ] |
| Day 11 | 14 | [K-th Not Divisible by n (1352C)](https://codeforces.com/problemset/problem/1352/C) | Codeforces | Rating 1200 | - [ ] |
| Day 11 | 15 | [Polygon (1360E)](https://codeforces.com/problemset/problem/1360/E) | Codeforces | Rating 1100 | - [ ] |

---

[⬅️ Previous Topic: Two Pointers](02-two-pointers.html) | [Home](index.html) | [Next Topic: Stack & Monotonic Stack ➡️](04-stack.html)
