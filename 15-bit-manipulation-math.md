---
layout: default
title: "Topic 15: Bit Manipulation & Core Math"
---

# Topic 15: Bit Manipulation & Core Math

[← Topic 14: Greedy & Intervals](14-greedy-intervals.md) \| [Schedule](schedule.md) \| [Home →](index.md)

---

## Daily Timeboxing (1.5 – 2.0 Hours)
- **00:00 – 00:15 (15m) | Active Recall Pre-Test**: Answer the conceptual questions below before reading further.
- **00:15 – 00:40 (25m) | YouTube Conceptual Anchor**: Watch the curated video segment on bitwise invariants and binary exponentiation.
- **00:40 – 01:30 (50m) | Coding Deep Dive**: Pen-and-paper dry run of Single Number III & Binary Exponentiation.
- **01:30 – 01:45 (15m) | Post-Session Flashcards**: Complete the flashcard prompts from memory.

---

## 1. Active Recall Pre-Test (15 Minutes)
Attempt to answer these questions on paper before expanding the solutions.

<details>
<summary><strong>Prompt 1: What does the bitwise operation `n & (n - 1)` do, and why does it form the basis of Brian Kernighan’s algorithm?</strong></summary>

Subtracting 1 from a binary number $n$ flips all the trailing zeros up to and including the lowest set bit (the rightmost 1-bit). 
Performing a bitwise AND between $n$ and $n - 1$ leaves all higher bits unchanged while clearing that lowest set bit to 0:
$$\text{Example: } n = 12 \ (1100_2), \quad n - 1 = 11 \ (1011_2), \quad n \ \& \ (n - 1) = 8 \ (1000_2)$$
This guarantees that each iteration of `n = n & (n - 1)` eliminates exactly one set bit, allowing popcount to run in $O(\text{number of set bits})$ iterations rather than looping through all 32 or 64 bit positions.
</details>

<details>
<summary><strong>Prompt 2: How does the bitwise expression `n & (-n)` isolate the lowest set bit of $n$?</strong></summary>

In Two's Complement representation, the negative of a signed integer is formed by bitwise NOT followed by adding 1:
$$-n = \sim n + 1$$
Inverting $\sim n$ turns the lowest set bit of $n$ to 0 and all trailing zeros to 1s. Adding 1 carries through all those trailing 1s until it hits that lowest bit, turning it back to 1 and setting all bits below it to 0. 
Performing $n \ \& \ (-n)$ results in a single bit set to 1 at the position of the lowest set bit, with all other bits being 0:
$$\text{Example: } n = 12 \ (01100_2), \quad -n = -12 \ (10100_2), \quad n \ \& \ (-n) = 4 \ (00100_2)$$
</details>

<details>
<summary><strong>Prompt 3: How does Binary Exponentiation compute $X^N \pmod M$ in $O(\log N)$ time instead of $O(N)$?</strong></summary>

By exploiting the binary representation of exponent $N$:
$$X^N = \begin{cases} (X^{N/2})^2 & \text{if } N \text{ is even} \\ X \cdot (X^{(N-1)/2})^2 & \text{if } N \text{ is odd} \end{cases}$$
At each step, we square the base $X \leftarrow X^2 \pmod M$ and halve the exponent $N \leftarrow N // 2$. 
Because $N$ is halved on every iteration, the computation terminates in exactly $\lfloor \log_2 N \rfloor$ steps, avoiding $O(N)$ linear multiplication.
</details>

---

## 2. YouTube Conceptual Anchor (25 Minutes)

Recommended Video Anchors:
1. **Errichto**: [Bitwise Operations in Competitive Programming](https://www.youtube.com/watch?v=xXKL9YBWgCY) (22 mins)
2. **NeetCode**: [Single Number & Counting Bits Tricks](https://www.youtube.com/watch?v=qMPX1AOa83k) (14 mins)

### Core Invariants & Mental Model
1. **XOR Self-Inverse Invariant**:
   $$A \oplus A = 0, \quad A \oplus 0 = A$$
   Because XOR is commutative and associative, XORing an entire array cancels out all elements that appear an even number of times, leaving only the unique odd-frequency element.
2. **Bitmask Set Representation**: An integer $M$ can represent a subset of up to 30 elements:
   - Check membership of $i$: `(M >> i) & 1`
   - Add $i$: `M |= (1 << i)`
   - Remove $i$: `M &= ~(1 << i)`
   - Toggle $i$: `M ^= (1 << i)`
3. **Sieve of Eratosthenes Complexity**: Finding all primes up to $N$ marks composite multiples of each prime $p$. The total operations equal $\sum_{p \le N} \frac{N}{p} = O(N \log \log N)$.

---

## 3. Canonical Code Templates

### Pattern A: Brian Kernighan's Bit Counting ($O(K)$ Time where $K \le 32$)
```python
def hamming_weight(n: int) -> int:
    count = 0
    while n:
        n &= (n - 1) # Drops the lowest set bit
        count += 1
    return count
```

### Pattern B: Single Number III (Two Unique Elements — $O(N)$ Time, $O(1)$ Space)
```python
from typing import List

def single_number_three(nums: List[int]) -> List[int]:
    # Step 1: XOR all numbers; result equals xor_sum = a ^ b
    xor_sum = 0
    for num in nums:
        xor_sum ^= num
        
    # Step 2: Find any bit where a and b differ (use lowest set bit)
    diff_bit = xor_sum & (-xor_sum)
    
    # Step 3: Partition elements into two groups based on diff_bit
    a = 0
    b = 0
    for num in nums:
        if num & diff_bit:
            a ^= num
        else:
            b ^= num
            
    return [a, b]
```

### Pattern C: Binary Modular Exponentiation ($O(\log N)$ Time, $O(1)$ Space)
```python
def modular_pow(base: int, exp: int, mod: int) -> int:
    result = 1
    base %= mod
    
    while exp > 0:
        # If exp is odd, multiply with current base
        if exp & 1:
            result = (result * base) % mod
        base = (base * base) % mod
        exp >>= 1 # Divide exponent by 2
        
    return result
```

---

## 4. Coding Deep Dive: Worked Walkthrough (50 Minutes)

### Problem: Single Number III (LeetCode 260)
**Problem Statement**: Given an integer array `nums` in which exactly two elements appear only once and all other elements appear exactly twice, return the two elements in any order. Algorithm must run in $O(N)$ time and $O(1)$ space.

#### Pen-and-Paper Dry Run Trace
Let `nums = [1, 2, 1, 3, 2, 5]`.
Two unique numbers to isolate: `3` ($011_2$) and `5` ($101_2$).

| Step | Operation | Binary Representation | Result |
|:---:|:---|:---:|:---:|
| 1 | XOR entire array: $1 \oplus 2 \oplus 1 \oplus 3 \oplus 2 \oplus 5$ | - | $3 \oplus 5 = 6$ ($110_2$) |
| 2 | Extract lowest set bit: `diff_bit = 6 & (-6)` | $110_2 \ \& \ 010_2$ | **`2` ($010_2$)** |
| 3 | **Group 1** (`num & 2 != 0`): `[2, 3, 2]` | $2 \oplus 3 \oplus 2$ | **`3`** |
| 4 | **Group 2** (`num & 2 == 0`): `[1, 1, 5]` | $1 \oplus 1 \oplus 5$ | **`5`** |

Result: `[3, 5]`.
Time Complexity: $O(N)$ two passes.
Space Complexity: $O(1)$ auxiliary memory.

---

## 5. Curated Problem Set (15 Problems)

Track your daily solutions by checking off the markdown boxes.

### NeetCode / LeetCode (Core)
- [ ] [LeetCode 136: Single Number](https://leetcode.com/problems/single-number/) | Easy | XOR Cancellation Invariant
- [ ] [LeetCode 191: Number of 1 Bits](https://leetcode.com/problems/number-of-1-bits/) | Easy | Brian Kernighan Lowest Bit Clearing
- [ ] [LeetCode 338: Counting Bits](https://leetcode.com/problems/counting-bits/) | Easy | Bitwise DP `dp[i] = dp[i >> 1] + (i & 1)`
- [ ] [LeetCode 268: Missing Number](https://leetcode.com/problems/missing-number/) | Easy | XOR Index-Value Pairing / Gauss Sum
- [ ] [LeetCode 190: Reverse Bits](https://leetcode.com/problems/reverse-bits/) | Easy | Bitwise Shift Reversal
- [ ] [LeetCode 371: Sum of Two Integers](https://leetcode.com/problems/sum-of-two-integers/) | Medium | Half Adder Bitwise XOR & AND Carry
- [ ] [LeetCode 137: Single Number II](https://leetcode.com/problems/single-number-ii/) | Medium | Modulo-3 Bit State Machine (`ones`, `twos`)
- [ ] [LeetCode 260: Single Number III](https://leetcode.com/problems/single-number-iii/) | Medium | Lowest Set Bit Group Partitioning
- [ ] [LeetCode 7: Reverse Integer](https://leetcode.com/problems/reverse-integer/) | Medium | 32-Bit Signed Overflow Boundaries
- [ ] [LeetCode 50: Pow(x, n)](https://leetcode.com/problems/powx-n/) | Medium | Fast Binary Exponentiation

### CSES Problem Set (Free Tier, No Login Wall)
- [ ] [CSES 1068: Weird Algorithm](https://cses.fi/problemset/task/1068) | Introductory Problems | Collatz Conjecture Simulation
- [ ] [CSES 1083: Missing Number](https://cses.fi/problemset/task/1083) | Introductory Problems | XOR Cancellation / Arithmetic Sum
- [ ] [CSES 1095: Exponentiation](https://cses.fi/problemset/task/1095) | Mathematics | Fast Modular Exponentiation Modulo $10^9+7$
- [ ] [CSES 1713: Counting Divisors](https://cses.fi/problemset/task/1713) | Mathematics | Sieve-Based Prime Factorization

### Codeforces & AtCoder (Rating 800–1400)
- [ ] [Codeforces 579A: Raising Bacteria](https://codeforces.com/problemset/problem/579/A) | 1000 | Set Bit Popcount Representation
- [ ] [AtCoder ABC147_C: HonestOrUnkind2](https://atcoder.jp/contests/abc147/tasks/abc147_c) | 1100 | Bitmask Subset Consistency Verification

---

## 6. Post-Session Active Recall Flashcards (15 Minutes)

<details>
<summary><strong>Flashcard 1: In Counting Bits (LeetCode 338), what $O(1)$ state transition generates the popcount for all numbers $0 \dots N$ in $O(N)$ total time?</strong></summary>

$$dp[i] = dp[i \gg 1] + (i \ \& \ 1)$$
Right-shifting $i$ by 1 bit ($i \gg 1$) removes the least significant bit, leaving a smaller integer whose bit count is already memoized in `dp`. Adding $i \ \& \ 1$ adds 1 if the removed bit was 1, and 0 if it was 0.
</details>

<details>
<summary><strong>Flashcard 2: How does the bitwise half-adder in Sum of Two Integers (LeetCode 371) simulate addition without arithmetic operators?</strong></summary>

$$\text{sum\_without\_carry} = a \oplus b$$
$$\text{carry} = (a \ \& \ b) \ll 1$$
Repeat until `carry == 0`. 
In Python, integers have arbitrary precision, so bitmasks like `0xFFFFFFFF` are required to simulate 32-bit signed integer overflow.
</details>

<details>
<summary><strong>Flashcard 3: How does the Sieve of Eratosthenes avoid redundant work by starting inner loops at $p^2$?</strong></summary>

Any composite number $k < p^2$ that has $p$ as a divisor must also have another prime factor $q < p$. 
That composite number was already marked when processing that smaller prime $q$. 
Therefore, the first composite multiple of $p$ that has not yet been marked by any smaller prime is $p \times p = p^2$.
</details>

---

[← Topic 14: Greedy & Intervals](14-greedy-intervals.md) \| [Schedule](schedule.md) \| [Back to Home →](index.md)
