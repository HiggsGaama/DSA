---
layout: default
title: "Stack & Monotonic Stack"
---

# Stack & Monotonic Stack

> **Estimated Time**: 11 Days • **Pace**: 1.5–2 hours/day (Strictly 1 problem/day) • **Language**: C# (.NET 6+)

---

## 🛠️ C# Mental Model & Gotchas

### 1. `Stack<T>` Mechanics in .NET
- Under the hood, `Stack<T>` is backed by a dynamic array $T[]$. Pushing and popping are amortized $O(1)$.
- Pre-size your stack when the upper bound is known: `new Stack<int>(s.Length)`.
- Always check `stack.Count > 0` before calling `stack.Peek()` or `stack.Pop()` to prevent `InvalidOperationException`.
- Store composite state using ValueTuples: `Stack<(int val, int minSoFar)>` or `Stack<(int height, int index)>`. This avoids allocating extra helper node objects on the heap.

### 2. The Monotonic Stack Invariant
A monotonic stack maintains elements in strictly increasing or decreasing order.
- When an incoming element violates the monotonic order, elements are popped from the stack until the invariant is restored.
- **The Core Rule**: The element being popped finds its **nearest greater or smaller element boundary** at the exact moment of being popped!
- Every element is pushed once and popped at most once, providing a strict $O(N)$ amortized time guarantee for range boundary queries.

---

## 📅 Day-by-Day Schedule

### Day 1: Theory Anchor & Baseline Pattern
- **Step 1 (45 mins) | Concept Video**: [Stack Data Structure Explained](https://www.youtube.com/watch?v=FNZ5o9bEBjo) by *Abdul Bari*
  - Key focus: What is LIFO? How does the call stack mirror explicit stack execution?
- **Step 2 (20 mins) | Pen & Paper Invariant**:
  - Trace `"{[()]}"` and `"{[(])}"`. For each character, write down the stack state. When does mismatch detection trigger?
- **Step 3 (45 mins) | Problem 1 (Warm-Up)**:
  - [Valid Parentheses](https://leetcode.com/problems/valid-parentheses/) (NeetCode - Easy)
  - Objective: Match closing brackets against top of `Stack<char>`.
- **Step 4 (10 mins) | Active Recall**:

<details markdown="1">
<summary><b>🧠 Self-Test: What happens if you call <code>stack.Pop()</code> on an empty C# stack?</b></summary>

<b>Answer:</b> C# throws an <code>InvalidOperationException: Stack empty</code>. Always verify <code>stack.Count &gt; 0</code> before popping.

</details>

---

### Day 2: Parallel State Tracking (Min Stack)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: Why cannot a single <code>int min</code> variable maintain the minimum of a stack when elements are popped?</b></summary>

<b>Answer:</b> If the current minimum is popped, there is no historical record of what the previous minimum was without scanning the entire stack ($O(N)$). We must maintain a parallel history of minimums.

</details>

- **Deep Practice (75–85 mins) | Problem 2**:
  - [Min Stack](https://leetcode.com/problems/min-stack/) (NeetCode - Medium)
  - Thinking prompts: Either maintain two stacks (`mainStack` and `minStack`) or push a single ValueTuple `(int val, int currentMin)` onto one stack.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class MinStack {
  private readonly Stack<(int val, int min)> _stack;

  public MinStack() {
      _stack = new Stack<(int val, int min)>();
  }

  public void Push(int val) {
      int currentMin = _stack.Count == 0 ? val : Math.Min(val, _stack.Peek().min);
      _stack.Push((val, currentMin));
  }

  public void Pop() {
      _stack.Pop();
  }

  public int Top() {
      return _stack.Peek().val;
  }

  public int GetMin() {
      return _stack.Peek().min;
  }
}
```

<b>Complexity:</b> Time: $O(1)$ for all operations, Space: $O(N)$ using tuple stack.

</details>

---

### Day 3: Postfix Evaluation (Reverse Polish Notation)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: In Reverse Polish Notation, when an operator is encountered, which operand is on top of the stack?</b></summary>

<b>Answer:</b> The right-hand (second) operand is popped first, followed by the left-hand (first) operand. For non-commutative operations like subtraction and division, order matters: <code>int b = stack.Pop(); int a = stack.Pop(); result = a - b;</code>.

</details>

- **Deep Practice (75–85 mins) | Problem 3**:
  - [Evaluate Reverse Polish Notation](https://leetcode.com/problems/evaluate-reverse-polish-notation/) (NeetCode - Medium)
  - Thinking prompts: Use `int.TryParse` or `switch` statements to apply operators `+`, `-`, `*`, `/`.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
  public int EvalRPN(string[] tokens) {
      Stack<int> stack = new();

      foreach (string token in tokens) {
          if (token == "+" || token == "-" || token == "*" || token == "/") {
              int b = stack.Pop();
              int a = stack.Pop();
              int res = token switch {
                  "+" => a + b,
                  "-" => a - b,
                  "*" => a * b,
                  "/" => a / b,
                  _ => 0
              };
              stack.Push(res);
          } else {
              stack.Push(int.Parse(token));
          }
      }
      return stack.Pop();
  }
}
```

<b>Complexity:</b> Time: $O(N)$, Space: $O(N)$.

</details>

---

### Day 4: Combinatorial Stack Invariant (Generate Parentheses)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: In recursive parenthesis generation for $N$ pairs, under what conditions can you add an open parenthesis vs a closed parenthesis?</b></summary>

<b>Answer:</b> You can add <code>'('</code> if <code>openCount &lt; n</code>. You can add <code>')'</code> if <code>closeCount &lt; openCount</code>.

</details>

- **Deep Practice (75–85 mins) | Problem 4**:
  - [Generate Parentheses](https://leetcode.com/problems/generate-parentheses/) (NeetCode - Medium)
  - Thinking prompts: Backtracking with a character buffer or recursion maintaining `open` and `close` counts.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
  public IList<string> GenerateParenthesis(int n) {
      List<string> result = new();
      char[] current = new char[2 * n];
      Backtrack(0, 0, 0, n, current, result);
      return result;
  }

  private void Backtrack(int idx, int open, int close, int n, char[] current, List<string> result) {
      if (idx == 2 * n) {
          result.Add(new string(current));
          return;
      }

      if (open < n) {
          current[idx] = '(';
          Backtrack(idx + 1, open + 1, close, n, current, result);
      }
      if (close < open) {
          current[idx] = ')';
          Backtrack(idx + 1, open, close + 1, n, current, result);
      }
  }
}
```

<b>Complexity:</b> Time: $O(\frac{4^n}{\sqrt{n}})$ (Catalan number), Space: $O(N)$ recursion depth.

</details>

---

### Day 5: Midpoint Shift — The Monotonic Stack Pattern
- **Step 1 (20 mins) | Focused Video**: [Monotonic Stack / Daily Temperatures](https://www.youtube.com/watch?v=cTBiBSnjO3c) by *NeetCode*
  - Key focus: Why does a decreasing monotonic stack reveal the nearest warmer day in $O(1)$ amortized steps?
- **Deep Practice (75 mins) | Problem 5**:
  - [Daily Temperatures](https://leetcode.com/problems/daily-temperatures/) (NeetCode - Medium)
  - Thinking prompts: Store indices in the stack. While incoming temperature is greater than `temperatures[stack.Peek()]`, pop index and calculate distance `i - prevIndex`.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
  public int[] DailyTemperatures(int[] temperatures) {
      int n = temperatures.Length;
      int[] ans = new int[n];
      Stack<int> stack = new(); // Stores indices of unresolved cooler days

      for (int i = 0; i < n; i++) {
          while (stack.Count > 0 && temperatures[i] > temperatures[stack.Peek()]) {
              int prevIdx = stack.Pop();
              ans[prevIdx] = i - prevIdx;
          }
          stack.Push(i);
      }
      return ans;
  }
}
```

<b>Complexity:</b> Time: $O(N)$ amortized, Space: $O(N)$.

</details>

---

### Day 6: Monotonic Stack on Rates of Change (Car Fleet)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: In Car Fleet, why do we sort cars by starting position in descending order?</b></summary>

<b>Answer:</b> A car behind can only catch up to a car ahead. Sorting from closest to destination backwards ensures we determine fleet leaders first. If a car behind arrives earlier or at the same time as the fleet ahead, it merges into that fleet.

</details>

- **Deep Practice (75–85 mins) | Problem 6**:
  - [Car Fleet](https://leetcode.com/problems/car-fleet/) (NeetCode - Medium)
  - Thinking prompts: Compute `time = (target - position) / (double)speed`. Maintain monotonic stack of arrival times.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
  public int CarFleet(int target, int[] position, int[] speed) {
      int n = position.Length;
      (int pos, int spd)[] cars = new (int, int)[n];
      for (int i = 0; i < n; i++) {
          cars[i] = (position[i], speed[i]);
      }

      // Sort descending by starting position
      Array.Sort(cars, (a, b) => b.pos.CompareTo(a.pos));

      int fleets = 0;
      double prevTime = 0.0;

      foreach (var car in cars) {
          double time = (double)(target - car.pos) / car.spd;
          if (time > prevTime) {
              fleets++;
              prevTime = time;
          }
      }
      return fleets;
  }
}
```

<b>Complexity:</b> Time: $O(N \log N)$ sorting, Space: $O(N)$.

</details>

---

### Day 7: The Master Benchmark (Largest Rectangle in Histogram)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: When a bar is popped in Largest Rectangle in Histogram, what are its left and right boundaries?</b></summary>

<b>Answer:</b> The right boundary is the current index $i$ that caused the pop (first smaller bar on right). The left boundary is the index of the bar directly beneath it in the stack (first smaller bar on left).

</details>

- **Deep Practice (75–85 mins) | Problem 7**:
  - [Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/) (NeetCode - Hard)
  - Thinking prompts: Draw histograms with decreasing heights. Trace how extending the width backwards using `start = poppedIndex` accounts for earlier taller bars.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
  public int LargestRectangleArea(int[] heights) {
      Stack<(int index, int height)> stack = new();
      int maxArea = 0;

      for (int i = 0; i < heights.Length; i++) {
          int start = i;
          while (stack.Count > 0 && stack.Peek().height > heights[i]) {
              var (poppedIdx, poppedHeight) = stack.Pop();
              maxArea = Math.Max(maxArea, poppedHeight * (i - poppedIdx));
              start = poppedIdx; // Extend backwards
          }
          stack.Push((start, heights[i]));
      }

      // Process remaining elements extending to end of array
      int n = heights.Length;
      while (stack.Count > 0) {
          var (poppedIdx, poppedHeight) = stack.Pop();
          maxArea = Math.Max(maxArea, poppedHeight * (n - poppedIdx));
      }
      return maxArea;
  }
}
```

<b>Complexity:</b> Time: $O(N)$, Space: $O(N)$.

</details>

---

### Day 8: Next Greater Element with Hash Mapping
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: How do you bridge monotonic stack results across two arrays in Next Greater Element I?</b></summary>

<b>Answer:</b> Run monotonic stack on <code>nums2</code> to find the next greater element for every number, storing results in a <code>Dictionary&lt;int, int&gt;</code>. Then map <code>nums1</code> queries directly via the dictionary.

</details>

- **Deep Practice (75–85 mins) | Problem 8**:
  - [Next Greater Element I](https://leetcode.com/problems/next-greater-element-i/) (LeetCode 496 - Easy)
  - Thinking prompts: Precompute mappings using a monotonic stack on `nums2`.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
  public int[] NextGreaterElement(int[] nums1, int[] nums2) {
      Dictionary<int, int> nextGreater = new();
      Stack<int> stack = new();

      foreach (int n in nums2) {
          while (stack.Count > 0 && stack.Peek() < n) {
              nextGreater[stack.Pop()] = n;
          }
          stack.Push(n);
      }

      int[] res = new int[nums1.Length];
      for (int i = 0; i < nums1.Length; i++) {
          res[i] = nextGreater.GetValueOrDefault(nums1[i], -1);
      }
      return res;
  }
}
```

<b>Complexity:</b> Time: $O(N + M)$, Space: $O(M)$.

</details>

---

### Day 9: Online Streaming Monotonic Aggregation (Stock Span)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: In Stock Spanner, why can we compress popped days into an accumulated span count?</b></summary>

<b>Answer:</b> If today's price is higher than previous days, today's price will also be higher than all days covered by those previous days. We can merge their spans directly: <code>span += stack.Pop().span</code>.

</details>

- **Deep Practice (75–85 mins) | Problem 9**:
  - [Online Stock Span](https://leetcode.com/problems/online-stock-span/) (LeetCode 901 - Medium)
  - Thinking prompts: Store `(int price, int span)` tuples in a decreasing monotonic stack.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class StockSpanner {
  private readonly Stack<(int price, int span)> _stack;

  public StockSpanner() {
      _stack = new Stack<(int, int)>();
  }

  public int Next(int price) {
      int span = 1;
      while (_stack.Count > 0 && _stack.Peek().price <= price) {
          span += _stack.Pop().span;
      }
      _stack.Push((price, span));
      return span;
  }
}
```

<b>Complexity:</b> Time: $O(1)$ amortized per call, Space: $O(N)$.

</details>

---

### Day 10: Stack Collision Simulation (Asteroid Collision)
- **Warm-Up (10 mins)**:

<details markdown="1">
<summary><b>🧠 Quick Recall: When do two asteroids collide?</b></summary>

<b>Answer:</b> Only when the asteroid on the left moves right (<code>top &gt; 0</code>) and the current asteroid moves left (<code>curr &lt; 0</code>). If both move the same direction or move away from each other (left moves left, right moves right), no collision occurs.

</details>

- **Deep Practice (75–85 mins) | Problem 10**:
  - [Asteroid Collision](https://leetcode.com/problems/asteroid-collision/) (LeetCode 735 - Medium)
  - Thinking prompts: Use a `bool alive` flag to track whether the incoming asteroid survives collisions with the stack.
- **Reference Solution**:

<details markdown="1">
<summary><b>View Optimal C# Solution & Complexity</b></summary>

```csharp
public class Solution {
  public int[] AsteroidCollision(int[] asteroids) {
      Stack<int> stack = new();

      foreach (int ast in asteroids) {
          bool destroyed = false;
          while (stack.Count > 0 && stack.Peek() > 0 && ast < 0) {
              int top = stack.Peek();
              if (top < -ast) {
                  stack.Pop(); // Top explodes, incoming continues
              } else if (top == -ast) {
                  stack.Pop(); // Both explode
                  destroyed = true;
                  break;
              } else {
                  destroyed = true; // Incoming explodes
                  break;
              }
          }
          if (!destroyed) {
              stack.Push(ast);
          }
      }

      int[] res = new int[stack.Count];
      for (int i = res.Length - 1; i >= 0; i--) {
          res[i] = stack.Pop();
      }
      return res;
  }
}
```

<b>Complexity:</b> Time: $O(N)$, Space: $O(N)$.

</details>

---

### Day 11: CSES & Competitive Benchmarks + Spaced Retrieval
- **Benchmark 1**: [CSES: Nearest Smaller Values](https://cses.fi/problemset/task/1645) (Standard 1-based monotonic stack)
- **Benchmark 2**: [CSES: Maximum Building I](https://cses.fi/problemset/task/1147) (Largest rectangle in 2D grid matrix)
- **Benchmark 3**: [CSES: Towers](https://cses.fi/problemset/task/1073) (Greedy stack piles)
- **Benchmark 4**: [Codeforces 520B - Two Buttons](https://codeforces.com/problemset/problem/520/B) (State search / reverse greedy)
- **Benchmark 5**: [Codeforces 115A - Party](https://codeforces.com/problemset/problem/115/A) (Tree depth / stack evaluation)

---

### Spaced Retrieval Flashcards

<details markdown="1">
<summary><b>Flashcard 1: What is the defining characteristic of problems solved by Monotonic Stacks?</b></summary>

Finding the "nearest element greater (or smaller) than $X$" to the left or right in an array in $O(N)$ total time.
</details>

<details markdown="1">
<summary><b>Flashcard 2: Why does an element popped from a monotonic stack never need to be inspected again?</b></summary>

Because the new incoming element is both "closer" and "better" (more extreme) for all subsequent future elements than the popped element could ever be.
</details>

<details markdown="1">
<summary><b>Flashcard 3: How can you iterate through a C# <code>Stack&lt;T&gt;</code> without destroying its contents?</b></summary>

Using a standard <code>foreach (var item in stack)</code> loop. In C#, <code>foreach</code> on a stack enumerates from top to bottom without popping elements.
</details>

---

## 🎯 Topic Checklist (15 Problems)

| Day | # | Problem | Platform | Difficulty | Status |
|:---:|:---:|:---|:---:|:---:|:---:|
| Day 1 | 1 | [Valid Parentheses](https://leetcode.com/problems/valid-parentheses/) | NeetCode | Easy | - [ ] |
| Day 2 | 2 | [Min Stack](https://leetcode.com/problems/min-stack/) | NeetCode | Medium | - [ ] |
| Day 3 | 3 | [Evaluate Reverse Polish Notation](https://leetcode.com/problems/evaluate-reverse-polish-notation/) | NeetCode | Medium | - [ ] |
| Day 4 | 4 | [Generate Parentheses](https://leetcode.com/problems/generate-parentheses/) | NeetCode | Medium | - [ ] |
| Day 5 | 5 | [Daily Temperatures](https://leetcode.com/problems/daily-temperatures/) | NeetCode | Medium | - [ ] |
| Day 6 | 6 | [Car Fleet](https://leetcode.com/problems/car-fleet/) | NeetCode | Medium | - [ ] |
| Day 7 | 7 | [Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/) | NeetCode | Hard | - [ ] |
| Day 8 | 8 | [Next Greater Element I](https://leetcode.com/problems/next-greater-element-i/) | LeetCode | Easy | - [ ] |
| Day 9 | 9 | [Online Stock Span](https://leetcode.com/problems/online-stock-span/) | LeetCode | Medium | - [ ] |
| Day 10 | 10 | [Asteroid Collision](https://leetcode.com/problems/asteroid-collision/) | LeetCode | Medium | - [ ] |
| Day 11 | 11 | [Nearest Smaller Values](https://cses.fi/problemset/task/1645) | CSES | Medium | - [ ] |
| Day 11 | 12 | [Maximum Building I](https://cses.fi/problemset/task/1147) | CSES | Hard | - [ ] |
| Day 11 | 13 | [Towers](https://cses.fi/problemset/task/1073) | CSES | Medium | - [ ] |
| Day 11 | 14 | [Two Buttons (520B)](https://codeforces.com/problemset/problem/520/B) | Codeforces | Rating 1000 | - [ ] |
| Day 11 | 15 | [Party (115A)](https://codeforces.com/problemset/problem/115/A) | Codeforces | Rating 900 | - [ ] |

---

[⬅️ Previous Topic: Sliding Window](03-sliding-window.html) | [Home](index.html) | [Next Topic: Binary Search & Answer Space ➡️](05-binary-search.html)