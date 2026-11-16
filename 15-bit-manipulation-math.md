---
layout: default
title: "Bit Manipulation & Math Fundamentals"
---

# Bit Manipulation & Math Fundamentals

> **Estimated Time**: 11 Days • **Pace**: 1.5–2 hours/day (Strictly 1 problem/day) • **Language**: C# (.NET 6+)

---

## 🛠️ C# Mental Model & Gotchas

### 1. The Core Bitwise Identities
In C#, bitwise operations operate directly at the CPU register level:
- **XOR Cancellation**: $x \oplus x = 0$ and $x \oplus 0 = x$.
- **Clear Lowest Set Bit**: `n = n & (n - 1)`. Eliminates the least significant 1-bit in a single instruction (Brian Kernighan's Algorithm).
- **Extract Lowest Set Bit**: `lowestBit = n & (-n)`. Leverages Two's Complement representation.
- **Set Bit $i$**: `n |= (1 << i)`.
- **Clear Bit $i$**: `n &= ~(1 << i)`.
- **Toggle Bit $i$**: `n ^= (1 << i)`.
- **Test Bit $i$**: `bool isSet = (n & (1 << i)) != 0`.

### 2. Modern .NET `BitOperations` Intrinsics
Starting in .NET 6+, C# provides hardware-accelerated bit manipulation intrinsics via `System.Numerics.BitOperations`:
```csharp
int popCount = System.Numerics.BitOperations.PopCount((uint)n); // Hardware popcnt instruction
int leadingZeros = System.Numerics.BitOperations.LeadingZeroCount((uint)n);
int trailingZeros = System.Numerics.BitOperations.TrailingZeroCount((uint)n);
```

### 3. Signed vs Unsigned Shift Mechanics
In C#, the right shift operator `>>` on a signed integer (`int`) performs an **arithmetic shift** (replicates the sign bit). To perform a **logical shift** (filling with zeros), cast to unsigned `uint` first:
```csharp
uint unsignedVal = (uint)signedVal;
unsignedVal >>= 1;
```

---

## 📅 Day-by-Day Schedule

### Day 1: Theory Anchor & Baseline Pattern (Single Number)
- **Step 1 (45 mins) | Concept Video**: [Bit Manipulation & XOR Properties](https://www.youtube.com/watch?v=5Km3utixwZs) by *NeetCode*
  - Key focus: What is XOR? Why does XOR across all array elements cancel duplicate pairs and preserve the single unique element?
- **Step 2 (20 mins) | Pen & Paper Invariant**:
  - Trace `[4, 1, 2, 1, 2]`. Write binary representations for each number. Trace cumulative XOR cancellation.
- **Step 3 (45 mins) | Problem 1 (Warm-Up)**:
  - [Single Number](https://leetcode.com/problems/single-number/) (NeetCode - Easy)
  - Objective: Single pass $O(N)$ time with $O(1)$ memory using XOR.
- **Step 4 (10 mins) | Active Recall**:
  <details>
  <summary><b>🧠 Self-Test: What are the three algebraic properties of XOR that allow it to cancel duplicates in any order?</b></summary>
  <br>
  <b>Answer:</b> 1. Commutative: $A \oplus B = B \oplus A$; 2. Associative: $(A \oplus B) \oplus C = A \oplus (B \oplus C)$; 3. Self-inverse: $A \oplus A = 0$ and $A \oplus 0 = A$.
  </details>

---

### Day 2: Brian Kernighan's Invariant (Number of 1 Bits)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: Why does <code>n &amp; (n - 1)</code> clear the lowest set bit?</b></summary>
  <br>
  <b>Answer:</b> Subtracting 1 flips the rightmost set bit to 0 and all trailing zeros to 1. Performing bitwise AND with the original number cancels all trailing bits and turns off the rightmost set bit.
  </details>
- **Deep Practice (75–85 mins) | Problem 2**:
  - [Number of 1 Bits](https://leetcode.com/problems/number-of-1-bits/) (NeetCode - Easy)
  - Thinking prompts: Compare standard 32-iteration shift loop vs Brian Kernighan's loop running in $O(\text{set bits})$ steps.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public int HammingWeight(uint n) {
          int count = 0;
          while (n != 0) {
              n &= (n - 1); // Clears the lowest set bit
              count++;
          }
          return count;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(K)$ where $K$ is count of 1-bits ($K \le 32$), Space: $O(1)$.
  </details>

---

### Day 3: Bitwise DP Recurrence (Counting Bits)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: What is the relationship between the 1-bits of number $i$ and number $i / 2$?</b></summary>
  <br>
  <b>Answer:</b> Shifting $i$ right by 1 bit (<code>i &gt;&gt; 1</code>) drops the least significant bit. Therefore: <code>dp[i] = dp[i &gt;&gt; 1] + (i &amp; 1)</code>.
  </details>
- **Deep Practice (75–85 mins) | Problem 3**:
  - [Counting Bits](https://leetcode.com/problems/counting-bits/) (NeetCode - Easy)
  - Thinking prompts: Populate `ans[i]` for all $i \in [0 \dots n]$ in $O(N)$ linear time.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public int[] CountBits(int n) {
          int[] ans = new int[n + 1];
          for (int i = 1; i <= n; i++) {
              ans[i] = ans[i >> 1] + (i & 1);
          }
          return ans;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N)$, Space: $O(1)$ auxiliary.
  </details>

---

### Day 4: 32-Bit Register Mirroring (Reverse Bits)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: How do you transfer the lowest bit of $N$ into the highest empty position of a result register?</b></summary>
  <br>
  <b>Answer:</b> Shift the result register left by 1, bitwise OR with <code>(n &amp; 1)</code>, and shift $n$ right by 1.
  </details>
- **Deep Practice (75–85 mins) | Problem 4**:
  - [Reverse Bits](https://leetcode.com/problems/reverse-bits/) (NeetCode - Easy)
  - Thinking prompts: Exactly 32 iterations reversing all bits of a 32-bit unsigned integer.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public uint reverseBits(uint n) {
          uint res = 0;
          for (int i = 0; i < 32; i++) {
              res = (res << 1) | (n & 1);
              n >>= 1;
          }
          return res;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(1)$ (exactly 32 iterations), Space: $O(1)$.
  </details>

---

### Day 5: Gauss Summation vs XOR Offset (Missing Number)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: In an array of size $N$ containing distinct numbers from $[0 \dots N]$, what two mathematical methods find the missing value in $O(N)$ time and $O(1)$ space?</b></summary>
  <br>
  <b>Answer:</b> 1. Arithmetic sum: $\frac{N(N+1)}{2} - \sum nums$; 2. XOR cancellation: XOR all numbers from $0$ to $N$ with all elements in $nums$.
  </details>
- **Deep Practice (75–85 mins) | Problem 5**:
  - [Missing Number](https://leetcode.com/problems/missing-number/) (NeetCode - Easy)
  - Thinking prompts: XOR eliminates potential integer overflow that can occur with Gauss summation on huge arrays.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public int MissingNumber(int[] nums) {
          int xor = nums.Length;
          for (int i = 0; i < nums.Length; i++) {
              xor ^= i ^ nums[i];
          }
          return xor;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N)$, Space: $O(1)$.
  </details>

---

### Day 6: Half-Adder Logic Gates (Sum of Two Integers)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: In digital circuit design, what logical gate represents bitwise addition without carry, and what gate represents carry?</b></summary>
  <br>
  <b>Answer:</b> XOR (<code>a ^ b</code>) produces the sum without carry. AND shifted left by 1 (<code>(a &amp; b) &lt;&lt; 1</code>) produces the carry.
  </details>
- **Deep Practice (75–85 mins) | Problem 6**:
  - [Sum of Two Integers](https://leetcode.com/problems/sum-of-two-integers/) (NeetCode - Medium)
  - Thinking prompts: Add two integers without `+` or `-`. Loop until `carry == 0`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public int GetSum(int a, int b) {
          while (b != 0) {
              int carry = (a & b) << 1;
              a = a ^ b;
              b = carry;
          }
          return a;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(1)$ (at most 32 carry propagations), Space: $O(1)$.
  </details>

---

### Day 7: 32-Bit Arithmetic Overflow Bounds (Reverse Integer)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: Before multiplying <code>res * 10 + digit</code>, how do you verify it will not exceed <code>int.MaxValue</code> ($2,147,483,647$)?</b></summary>
  <br>
  <b>Answer:</b> Check if <code>res &gt; int.MaxValue / 10 || (res == int.MaxValue / 10 &amp;&amp; digit &gt; 7)</code>.
  </details>
- **Deep Practice (75–85 mins) | Problem 7**:
  - [Reverse Integer](https://leetcode.com/problems/reverse-integer/) (NeetCode - Medium)
  - Thinking prompts: Return 0 on 32-bit signed integer overflow.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public int Reverse(int x) {
          int res = 0;
          while (x != 0) {
              int pop = x % 10;
              x /= 10;
              
              if (res > int.MaxValue / 10 || (res == int.MaxValue / 10 && pop > 7)) return 0;
              if (res < int.MinValue / 10 || (res == int.MinValue / 10 && pop < -8)) return 0;
              
              res = res * 10 + pop;
          }
          return res;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(\log_{10} X)$, Space: $O(1)$.
  </details>

---

### Day 8: Binary Exponentiation (Pow(x, n))
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: How does Binary Exponentiation calculate $x^N$ in $O(\log N)$ steps instead of $O(N)$?</b></summary>
  <br>
  <b>Answer:</b> If $N$ is even, $x^N = (x^2)^{N/2}$. If $N$ is odd, $x^N = x \cdot (x^2)^{(N-1)/2}$. Each step squares the base and halves the exponent.
  </details>
- **Deep Practice (75–85 mins) | Problem 8**:
  - [Pow(x, n)](https://leetcode.com/problems/powx-n/) (NeetCode - Medium)
  - Thinking prompts: Edge case: $n = \text{int.MinValue}$ (negating it overflows signed 32-bit int!). Cast $n$ to `long`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public double MyPow(double x, int n) {
          long N = n;
          if (N < 0) {
              x = 1.0 / x;
              N = -N;
          }
          
          double res = 1.0;
          double currentProduct = x;
          
          for (long i = N; i > 0; i /= 2) {
              if (i % 2 == 1) {
                  res *= currentProduct;
              }
              currentProduct *= currentProduct;
          }
          return res;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(\log N)$, Space: $O(1)$.
  </details>

---

### Day 9: Elementary Grade-School Multiplication (Multiply Strings)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: When multiplying digits at indices $i$ and $j$, where do the product digits land in the output array?</b></summary>
  <br>
  <b>Answer:</b> The multiplication $num_1[i] \times num_2[j]$ spans indices $i + j$ and $i + j + 1$ in the result array of length $M + N$.
  </details>
- **Deep Practice (75–85 mins) | Problem 9**:
  - [Multiply Strings](https://leetcode.com/problems/multiply-strings/) (NeetCode - Medium)
  - Thinking prompts: Avoid `BigInteger`. Implement manual digit multiplication on `int[m + n]`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public string Multiply(string num1, string num2) {
          if (num1 == "0" || num2 == "0") return "0";
          
          int m = num1.Length, n = num2.Length;
          int[] res = new int[m + n];
          
          for (int i = m - 1; i >= 0; i--) {
              for (int j = n - 1; j >= 0; j--) {
                  int mul = (num1[i] - '0') * (num2[j] - '0');
                  int p1 = i + j, p2 = i + j + 1;
                  int sum = mul + res[p2];
                  
                  res[p2] = sum % 10;
                  res[p1] += sum / 10;
              }
          }
          
          var sb = new System.Text.StringBuilder();
          foreach (int digit in res) {
              if (sb.Length == 0 && digit == 0) continue; // Skip leading zeros
              sb.Append(digit);
          }
          return sb.ToString();
      }
  }
  ```
  <b>Complexity:</b> Time: $O(M \cdot N)$, Space: $O(M + N)$.
  </details>

---

### Day 10: Sieve of Eratosthenes (Count Primes)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: In the Sieve of Eratosthenes, at what multiple can we begin crossing off composites for prime $P$?</b></summary>
  <br>
  <b>Answer:</b> At $P^2$ (all smaller multiples like $2P, 3P$ have already been marked by smaller prime factors $2, 3$).
  </details>
- **Deep Practice (75–85 mins) | Problem 10**:
  - [Count Primes](https://leetcode.com/problems/count-primes/) (LeetCode 204 - Medium)
  - Thinking prompts: Sieve up to $N$. Outer loop condition: `p * p < n`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public int CountPrimes(int n) {
          if (n <= 2) return 0;
          
          bool[] isComposite = new bool[n];
          int count = 0;
          
          for (int p = 2; p < n; p++) {
              if (!isComposite[p]) {
                  count++;
                  if ((long)p * p < n) {
                      for (int mult = p * p; mult < n; mult += p) {
                          isComposite[mult] = true;
                      }
                  }
              }
          }
          return count;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N \log \log N)$, Space: $O(N)$.
  </details>

---

### Day 11: CSES & Competitive Benchmarks + Spaced Retrieval
- **Benchmark 1**: [CSES: Bit Strings](https://cses.fi/problemset/task/1617) (Calculate $2^N \pmod{10^9+7}$)
- **Benchmark 2**: [CSES: Exponentiation](https://cses.fi/problemset/task/1095) (Fast modular exponentiation $A^B \pmod{10^9+7}$)
- **Benchmark 3**: [CSES: Counting Divisors](https://cses.fi/problemset/task/1713) (Precomputed sieve of divisor counts)
- **Benchmark 4**: [CSES: Common Divisors](https://cses.fi/problemset/task/1081) (Maximum GCD of pairs in an array)
- **Benchmark 5**: [Codeforces 1097B - Petr and a Combination Lock](https://codeforces.com/problemset/problem/1097/B) (Bitmask subset rotation sum modulo 360)

---

### Spaced Retrieval Flashcards

<details>
<summary><b>Flashcard 1: Why is $n \ \& \ (-n)$ guaranteed to isolate the lowest set bit?</b></summary>
<br>
In Two's Complement representation, $-n = \sim n + 1$. Inverting $n$ flips all bits, and adding 1 turns all trailing 1s to 0 and flips the lowest set bit back to 1. Performing bitwise AND with the original number masks out all other bits except that solitary lowest bit.
</details>

<details>
<summary><b>Flashcard 2: How do you iterate through all $2^N$ subsets of an array using an integer bitmask in C#?</b></summary>
<br>
Run an outer loop <code>for (int mask = 0; mask &lt; (1 &lt;&lt; n); mask++)</code> and an inner loop <code>for (int i = 0; i &lt; n; i++)</code> testing <code>if ((mask &amp; (1 &lt;&lt; i)) != 0)</code>.
</details>

<details>
<summary><b>Flashcard 3: How does Modular Exponentiation avoid 64-bit arithmetic overflow during multiplication?</b></summary>
<br>
By reducing modulo $M$ at every single intermediate multiplication step: <code>res = (res * baseVal) % M</code>. Cast factors to <code>long</code> before multiplying: <code>(long)a * b % m</code>.
</details>

---

## 🎯 Topic Checklist (15 Problems)

| Day | # | Problem | Platform | Difficulty | Status |
|:---:|:---:|:---|:---:|:---:|:---:|
| Day 1 | 1 | [Single Number](https://leetcode.com/problems/single-number/) | NeetCode | Easy | - [ ] |
| Day 2 | 2 | [Number of 1 Bits](https://leetcode.com/problems/number-of-1-bits/) | NeetCode | Easy | - [ ] |
| Day 3 | 3 | [Counting Bits](https://leetcode.com/problems/counting-bits/) | NeetCode | Easy | - [ ] |
| Day 4 | 4 | [Reverse Bits](https://leetcode.com/problems/reverse-bits/) | NeetCode | Easy | - [ ] |
| Day 5 | 5 | [Missing Number](https://leetcode.com/problems/missing-number/) | NeetCode | Easy | - [ ] |
| Day 6 | 6 | [Sum of Two Integers](https://leetcode.com/problems/sum-of-two-integers/) | NeetCode | Medium | - [ ] |
| Day 7 | 7 | [Reverse Integer](https://leetcode.com/problems/reverse-integer/) | NeetCode | Medium | - [ ] |
| Day 8 | 8 | [Pow(x, n)](https://leetcode.com/problems/powx-n/) | NeetCode | Medium | - [ ] |
| Day 9 | 9 | [Multiply Strings](https://leetcode.com/problems/multiply-strings/) | NeetCode | Medium | - [ ] |
| Day 10 | 10 | [Count Primes](https://leetcode.com/problems/count-primes/) | LeetCode | Medium | - [ ] |
| Day 11 | 11 | [Bit Strings](https://cses.fi/problemset/task/1617) | CSES | Easy | - [ ] |
| Day 11 | 12 | [Exponentiation](https://cses.fi/problemset/task/1095) | CSES | Easy | - [ ] |
| Day 11 | 13 | [Counting Divisors](https://cses.fi/problemset/task/1713) | CSES | Easy | - [ ] |
| Day 11 | 14 | [Common Divisors](https://cses.fi/problemset/task/1081) | CSES | Hard | - [ ] |
| Day 11 | 15 | [Petr and a Combination Lock (1097B)](https://codeforces.com/problemset/problem/1097/B) | Codeforces | Rating 1000 | - [ ] |

---

[⬅️ Previous Topic: Greedy Algorithms & Intervals](14-greedy-intervals.html) | [Home](index.html) | **🎉 Course Completed!**

