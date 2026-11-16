---
layout: default
title: "Arrays, Hashing & Frequency Counting"
---

# Arrays, Hashing & Frequency Counting

> **Estimated Time**: 11 Days • **Pace**: 1.5–2 hours/day (Strictly 1 problem/day) • **Language**: C# (.NET 6+)

---

## 🛠️ C# Mental Model & Gotchas

### 1. Hash Lookup Internals & Performance
In .NET, `Dictionary<TKey, TValue>` and `HashSet<T>` use open addressing with collision chaining via prime-sized bucket arrays.
- **Double Lookups**: Avoid calling `map.ContainsKey(key)` followed immediately by `map[key]`. In tight loops, this doubles the hashing overhead. Use `map.TryGetValue(key, out var val)`.
- **Custom Keys**: If using custom structs or classes as keys, always implement `IEquatable<T>` and override `GetHashCode()`. For multi-field composite keys, use C# `ValueTuple` (e.g. `(int row, int col)`), which generates high-quality value-based hash codes automatically.
- **Pre-sizing**: Avoid frequent table resizing and rehashing when the element count $N$ is known upfront. Always pass capacity: `new Dictionary<int, int>(nums.Length)`.

### 2. Frequency Tables: Fixed ASCII Arrays vs Dictionaries
When counting character frequencies in strings consisting only of lowercase English letters (`'a'`–`'z'`), **do not use `Dictionary<char, int>`**. A fixed-size array `int[] count = new int[26]` is cache-friendly, requires zero heap allocations per entry, and offers instantaneous $O(1)$ direct indexing: `count[c - 'a']++`.

### 3. Avoiding LINQ in Inner Loops
LINQ methods like `.Select()`, `.GroupBy()`, and `.Sum()` allocate enumerator objects on the managed heap. For LeetCode / competitive programming hot loops, write standard `for` or `foreach` loops.

---

## 📅 Day-by-Day Schedule

### Day 1: Theory Anchor & Baseline Pattern
- **Step 1 (45 mins) | Concept Video**: [Hash Tables and Hash Functions](https://www.youtube.com/watch?v=shs0KM3wKv8) by *Abdul Bari*
  - Key focus: What is a hash collision? How do load factors, open addressing, and chaining impact $O(1)$ average time complexity?
- **Step 2 (20 mins) | Pen & Paper Invariant**:
  - Sketch a hash table with bucket array of size 7. Insert keys `12, 19, 26` with $h(k) = k \pmod 7$. Draw the collision chain.
- **Step 3 (45 mins) | Problem 1 (Warm-Up)**:
  - [Contains Duplicate](https://leetcode.com/problems/contains-duplicate/) (NeetCode - Easy)
  - Objective: Implement in C# using `HashSet<int>`. Verify early exit condition.
- **Step 4 (10 mins) | Active Recall**:

<details markdown="1">
<summary><b>🧠 Self-Test: What is the worst-case time complexity of a hash set insertion, and why does it occur?</b></summary>

<b>Answer:</b> $O(N)$ worst-case. This occurs when all inserted elements produce identical hash codes (hash collision catastrophe) or the hash table exceeds its load factor and must allocate a new prime-sized backing array and rehash every element.

</details>

---

### Day 2: Core Pattern Consolidation (Two-Sum Complement Technique)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: Why is checking <code>map.ContainsKey(k)</code> followed by <code>map[k]</code> suboptimal in C#?</b></summary>

<b>Answer:</b> It evaluates the hash function and traverses the bucket chain twice. Using <code>map.TryGetValue(k, out var val)</code> achieves the same lookup in a single hash computation and bucket lookup.

</details>

- **Deep Practice (75–85 mins) | Problem 2**:
  - [Two Sum](https://leetcode.com/problems/two-sum/) (NeetCode - Easy)
  - Thinking prompts: Instead of comparing every pair $O(N^2)$, maintain an invariant: at index $i$, the dictionary holds all values seen prior to $i$. Query target complement $T - nums[i]$.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int[] TwoSum(int[] nums, int target) {
        // Map value -> index seen so far
        Dictionary<int, int> seen = new(nums.Length);

        for (int i = 0; i < nums.Length; i++) {
            int complement = target - nums[i];
            if (seen.TryGetValue(complement, out int complementIndex)) {
                return new int[] { complementIndex, i };
            }
            // Store current number with its index
            seen[nums[i]] = i;
        }

        return Array.Empty<int>();
    }
}
```

<b>Complexity:</b> Time: $O(N)$ single pass, Space: $O(N)$ auxiliary hash map.

</details>

---

### Day 3: Character Frequency Arrays & Canonical Keys
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: When an input string is guaranteed to be lowercase English letters, what data structure should you use instead of <code>Dictionary&lt;char, int&gt;</code>?</b></summary>

<b>Answer:</b> A fixed-size array <code>int[26]</code>. It avoids heap object allocation, provides cache locality, and uses direct indexing <code>c - 'a'</code>.

</details>

- **Deep Practice (75–85 mins) | Problem 3**:
  - [Valid Anagram](https://leetcode.com/problems/valid-anagram/) (NeetCode - Easy)
  - Thinking prompts: An anagram is an invariant of character counts. Can you solve it with a single array of size 26 by incrementing for string $S$ and decrementing for string $T$?
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public bool IsAnagram(string s, string t) {
        if (s.Length != t.Length) return false;

        int[] freq = new int[26];
        for (int i = 0; i < s.Length; i++) {
            freq[s[i] - 'a']++;
            freq[t[i] - 'a']--;
        }

        for (int i = 0; i < 26; i++) {
            if (freq[i] != 0) return false;
        }
        return true;
    }
}
```

<b>Complexity:</b> Time: $O(N)$, Space: $O(1)$ fixed 26-element array.

</details>

---

### Day 4: Equivalence Class Hashing
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: How can two anagrams produce an identical key in a dictionary?</b></summary>

<b>Answer:</b> Either by sorting the characters ($O(K \log K)$ where $K$ is string length) or by serializing their 26-element frequency signature into a string (e.g. <code>#1#0#2...</code> in $O(K)$ time).

</details>

- **Deep Practice (75–85 mins) | Problem 4**:
  - [Group Anagrams](https://leetcode.com/problems/group-anagrams/) (NeetCode - Medium)
  - Thinking prompts: What represents the canonical equivalence class? Compare string sorting vs character frequency serialization.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public IList<IList<string>> GroupAnagrams(string[] strs) {
        Dictionary<string, List<string>> groups = new();

        foreach (string s in strs) {
            char[] chars = s.ToCharArray();
            Array.Sort(chars);
            string key = new string(chars);

            if (!groups.TryGetValue(key, out var list)) {
                list = new List<string>();
                groups[key] = list;
            }
            list.Add(s);
        }

        return groups.Values.Cast<IList<string>>().ToList();
    }
}
```

<b>Complexity:</b> Time: $O(N \cdot K \log K)$ where $N$ is word count and $K$ is maximum word length. Space: $O(N \cdot K)$.

</details>

---

### Day 5: Bucket Sort vs Heap for Top-K Elements
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: Why is Bucket Sort able to find Top-K frequent elements in $O(N)$ time whereas a heap takes $O(N \log K)$?</b></summary>

<b>Answer:</b> The maximum possible frequency of any element in an array of size $N$ is strictly bounded by $N$. Thus, we can create an array of buckets where index $f$ stores all numbers that appear with frequency $f$.

</details>

- **Deep Practice (75–85 mins) | Problem 5**:
  - [Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/) (NeetCode - Medium)
  - Thinking prompts: First count frequencies into a `Dictionary<int, int>`. Then build `List<int>[] buckets = new List<int>[nums.Length + 1]`. Scan from the highest bucket downwards.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int[] TopKFrequent(int[] nums, int k) {
        Dictionary<int, int> count = new();
        foreach (int n in nums) {
            count[n] = count.GetValueOrDefault(n, 0) + 1;
        }

        List<int>[] buckets = new List<int>[nums.Length + 1];
        foreach (var kvp in count) {
            int freq = kvp.Value;
            buckets[freq] ??= new List<int>();
            buckets[freq].Add(kvp.Key);
        }

        int[] result = new int[k];
        int idx = 0;
        for (int f = buckets.Length - 1; f >= 0 && idx < k; f--) {
            if (buckets[f] != null) {
                foreach (int val in buckets[f]) {
                    result[idx++] = val;
                    if (idx == k) break;
                }
            }
        }
        return result;
    }
}
```

<b>Complexity:</b> Time: $O(N)$, Space: $O(N)$.

</details>

---

### Day 6: Prefix & Suffix Products Without Division
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: If division was allowed, <code>Product of Array Except Self</code> would be trivial. Why is division problematic when handling zeros?</b></summary>

<b>Answer:</b> A single zero causes division by zero exceptions; multiple zeros mean every product except self is zero.

</details>

- **Deep Practice (75–85 mins) | Problem 6**:
  - [Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/) (NeetCode - Medium)
  - Thinking prompts: Any product of array except index $i$ is equal to $(\text{prefix product before } i) \times (\text{suffix product after } i)$. Compute prefix in the output array, then accumulate suffix product with a single running integer variable.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int[] ProductExceptSelf(int[] nums) {
        int n = nums.Length;
        int[] res = new int[n];

        // Pass 1: compute prefix products
        res[0] = 1;
        for (int i = 1; i < n; i++) {
            res[i] = res[i - 1] * nums[i - 1];
        }

        // Pass 2: accumulate suffix products
        int suffix = 1;
        for (int i = n - 1; i >= 0; i--) {
            res[i] *= suffix;
            suffix *= nums[i];
        }

        return res;
    }
}
```

<b>Complexity:</b> Time: $O(N)$, Space: $O(1)$ auxiliary (excluding the output array).

</details>

---

### Day 7: Grid Invariants & Bitmasks / Sets
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: How can a $9 \times 9$ Sudoku sub-box index $(0 \dots 8)$ be calculated from row $r$ and column $c$?</b></summary>

<b>Answer:</b> <code>(r / 3) * 3 + (c / 3)</code>.

</details>

- **Deep Practice (75–85 mins) | Problem 7**:
  - [Valid Sudoku](https://leetcode.com/problems/valid-sudoku/) (NeetCode - Medium)
  - Thinking prompts: Can you track seen digits using integer bitmasks (`int[9] rows`, `int[9] cols`, `int[9] boxes`) where the $d$-th bit indicates whether digit $d$ has been encountered?
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public bool IsValidSudoku(char[][] board) {
        int[] rows = new int[9];
        int[] cols = new int[9];
        int[] boxes = new int[9];

        for (int r = 0; r < 9; r++) {
            for (int c = 0; c < 9; c++) {
                char ch = board[r][c];
                if (ch == '.') continue;

                int digit = ch - '1'; // 0 to 8
                int mask = 1 << digit;
                int boxIdx = (r / 3) * 3 + (c / 3);

                if ((rows[r] & mask) != 0 || (cols[c] & mask) != 0 || (boxes[boxIdx] & mask) != 0) {
                    return false;
                }

                rows[r] |= mask;
                cols[c] |= mask;
                boxes[boxIdx] |= mask;
            }
        }
        return true;
    }
}
```

<b>Complexity:</b> Time: $O(1)$ (bounded $9 \times 9 = 81$ cells), Space: $O(1)$ using integer bitmasks.

</details>

---

### Day 8: String Framing Protocols (Length Prefixes)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: Why is using a single delimiter like <code>,</code> or <code>#</code> flawed when serializing arbitrary strings?</b></summary>

<b>Answer:</b> If the string itself contains the delimiter, decoding becomes ambiguous. A robust framing protocol encodes the length followed by a separator (e.g. <code>4#lint4#code</code>).

</details>

- **Deep Practice (75–85 mins) | Problem 8**:
  - [Encode and Decode Strings](https://leetcode.com/problems/encode-and-decode-strings/) (NeetCode - Medium)
  - Thinking prompts: Encode strings as `[length]#[string]`. During decoding, read digits until `#`, parse length $L$, slice the next $L$ characters, and advance pointer.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Codec {
    public string encode(IList<string> strs) {
        var sb = new System.Text.StringBuilder();
        foreach (var s in strs) {
            sb.Append(s.Length).Append('#').Append(s);
        }
        return sb.ToString();
    }

    public IList<string> decode(string s) {
        List<string> result = new();
        int i = 0;
        while (i < s.Length) {
            int delim = s.IndexOf('#', i);
            int len = int.Parse(s.Substring(i, delim - i));
            i = delim + 1;
            result.Add(s.Substring(i, len));
            i += len;
        }
        return result;
    }
}
```

<b>Complexity:</b> Time: $O(N)$ for both encode and decode. Space: $O(1)$ auxiliary.

</details>

---

### Day 9: Sequence Invariants in Unsorted Arrays
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: In <code>Longest Consecutive Sequence</code>, how do we guarantee each streak is traversed only once in $O(N)$ time instead of $O(N^2)$?</b></summary>

<b>Answer:</b> Only start counting a sequence from $x$ if $x - 1$ is NOT in the hash set. This ensures $x$ is the strictly designated beginning of a streak.

</details>

- **Deep Practice (75–85 mins) | Problem 9**:
  - [Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence/) (NeetCode - Medium)
  - Thinking prompts: Insert all numbers into `HashSet<int>`. Iterate through the set. If `!set.Contains(num - 1)`, initiate an upward streak probe `num + 1, num + 2, ...`.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int LongestConsecutive(int[] nums) {
        HashSet<int> numSet = new(nums);
        int longest = 0;

        foreach (int n in numSet) {
            // Check if n is the start of a streak
            if (!numSet.Contains(n - 1)) {
                int currentNum = n;
                int currentStreak = 1;

                while (numSet.Contains(currentNum + 1)) {
                    currentNum++;
                    currentStreak++;
                }
                longest = Math.Max(longest, currentStreak);
            }
        }
        return longest;
    }
}
```

<b>Complexity:</b> Time: $O(N)$ (each element visited at most twice), Space: $O(N)$.

</details>

---

### Day 10: Prefix Sum Frequencies (Subarray Sums Equals K)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: How does prefix sum transform a range sum query $nums[i \dots j] = K$ into a two-sum complement lookup?</b></summary>

<b>Answer:</b> Since $\text{sum}(i \dots j) = \text{prefix}[j] - \text{prefix}[i - 1] = K$, we can rearrange this to $\text{prefix}[i - 1] = \text{prefix}[j] - K$. We store prefix sum frequencies in a hash map as we iterate.

</details>

- **Deep Practice (75–85 mins) | Problem 10**:
  - [Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/) (LeetCode 560 - Medium)
  - Thinking prompts: Remember base case `map[0] = 1` before starting the loop (representing an empty prefix before index 0).
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
    public int SubarraySum(int[] nums, int k) {
        Dictionary<int, int> prefixFreq = new();
        prefixFreq[0] = 1; // Base case for subarrays starting at index 0

        int currentSum = 0;
        int count = 0;

        foreach (int n in nums) {
            currentSum += n;
            if (prefixFreq.TryGetValue(currentSum - k, out int freq)) {
                count += freq;
            }
            prefixFreq[currentSum] = prefixFreq.GetValueOrDefault(currentSum, 0) + 1;
        }
        return count;
    }
}
```

<b>Complexity:</b> Time: $O(N)$, Space: $O(N)$.

</details>

---

### Day 11: CSES & Competitive Benchmarks + Spaced Retrieval
- **Benchmark 1**: [CSES: Distinct Numbers](https://cses.fi/problemset/task/1621) (Testing raw sorting vs hash set throughput)
- **Benchmark 2**: [CSES: Maximum Subarray Sum](https://cses.fi/problemset/task/1643) (Kadane's algorithm / prefix minimums)
- **Benchmark 3**: [CSES: Subarray Sums I](https://cses.fi/problemset/task/1660) (Prefix sums on non-negative array)
- **Benchmark 4**: [Codeforces 4A - Watermelon](https://codeforces.com/problemset/problem/4/A) (Parity check)
- **Benchmark 5**: [Codeforces 1335C - Two Teams Composing](https://codeforces.com/problemset/problem/1335/C) (Frequency table vs distinct counts)

---

### Spaced Retrieval Flashcards

<details markdown="1">
<summary><b>Flashcard 1: When should you use a frequency array <code>int[26]</code> over <code>Dictionary&lt;char, int&gt;</code> in C#?</b></summary>

When the domain of characters is strictly bounded (e.g. lowercase English letters <code>'a'..'z'</code>). A direct array eliminates heap allocations, reduces garbage collection pressure, and leverages CPU cache line locality.
</details>

<details markdown="1">
<summary><b>Flashcard 2: Why must <code>prefixFreq[0] = 1</code> be initialized when counting subarrays with sum equal to $K$?</b></summary>

Because if the running prefix sum itself equals $K$, the expression $\text{currentSum} - K$ evaluates to $0$. An initial count of 1 for prefix sum 0 accounts for valid subarrays starting exactly from index 0.
</details>

<details markdown="1">
<summary><b>Flashcard 3: How does C# handle equality comparison for composite keys in a <code>Dictionary</code>?</b></summary>

If using a custom class without overriding <code>Equals()</code> and <code>GetHashCode()</code>, reference equality is used, causing lookups with new instances to fail. Using <code>ValueTuple</code> (e.g. <code>(int, int)</code>) or C# 9+ <code>record struct</code> provides automatic value-based equality and hashing.
</details>

---

## 🎯 Topic Checklist (15 Problems)

| Day | # | Problem | Platform | Difficulty | Status |
|:---:|:---:|:---|:---:|:---:|:---:|
| Day 1 | 1 | [Contains Duplicate](https://leetcode.com/problems/contains-duplicate/) | NeetCode | Easy | - [ ] |
| Day 2 | 2 | [Two Sum](https://leetcode.com/problems/two-sum/) | NeetCode | Easy | - [ ] |
| Day 3 | 3 | [Valid Anagram](https://leetcode.com/problems/valid-anagram/) | NeetCode | Easy | - [ ] |
| Day 4 | 4 | [Group Anagrams](https://leetcode.com/problems/group-anagrams/) | NeetCode | Medium | - [ ] |
| Day 5 | 5 | [Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/) | NeetCode | Medium | - [ ] |
| Day 6 | 6 | [Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/) | NeetCode | Medium | - [ ] |
| Day 7 | 7 | [Valid Sudoku](https://leetcode.com/problems/valid-sudoku/) | NeetCode | Medium | - [ ] |
| Day 8 | 8 | [Encode and Decode Strings](https://leetcode.com/problems/encode-and-decode-strings/) | NeetCode | Medium | - [ ] |
| Day 9 | 9 | [Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence/) | NeetCode | Medium | - [ ] |
| Day 10 | 10 | [Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/) | LeetCode | Medium | - [ ] |
| Day 11 | 11 | [Distinct Numbers](https://cses.fi/problemset/task/1621) | CSES | Easy | - [ ] |
| Day 11 | 12 | [Maximum Subarray Sum](https://cses.fi/problemset/task/1643) | CSES | Medium | - [ ] |
| Day 11 | 13 | [Subarray Sums I](https://cses.fi/problemset/task/1660) | CSES | Medium | - [ ] |
| Day 11 | 14 | [Watermelon (4A)](https://codeforces.com/problemset/problem/4/A) | Codeforces | Rating 800 | - [ ] |
| Day 11 | 15 | [Two Teams Composing (1335C)](https://codeforces.com/problemset/problem/1335/C) | Codeforces | Rating 1000 | - [ ] |

---

[Home](index.html) | [Next Topic: Two Pointers ➡️](02-two-pointers.html)
