---
layout: default
title: "Home"
permalink: /
---

# C# DSA Mastery: Active Recall Roadmap

> **Philosophy**: Master Data Structures and Algorithms with a calm, unhurried **1.5 to 2 hours per day** study schedule. Strictly **one problem per day** to prioritize deep thinking, paper-and-pencil invariant tracing, and production-grade **C# (.NET 6+)** implementations over rote memorization.

---

## 🧠 The Active Recall & Paper-First Methodology

Most engineers fail technical interviews not because they lack intelligence, but because of **passive recognition illusion**—watching a video solution, nodding along, and assuming the brain has wired the neural pathways. 

This curriculum enforces strict cognitive friction through **Active Recall** and **Spaced Retrieval**:

```
┌────────────────────────────────────────────────────────────────────────┐
│                   DAILY 90–120 MINUTE TIMEBOX                          │
│                                                                        │
│  [10 Mins]  Closed-Book Warm-up: Answer yesterday's invariant check   │
│  [25 Mins]  Paper & Pencil Tracing: Dry-run invariants & edge cases   │
│  [50 Mins]  C# Implementation: Clean, idiomatic .NET code             │
│  [15 Mins]  Reflection Log: Record time/space trade-offs & edge cases │
└────────────────────────────────────────────────────────────────────────┘
```

### The Two Session Formats

#### 1. Theory Days (Day 1 of Each Topic, ~Once Every 10–12 Days)
- **40–50 mins | Concept Video**: Watch a deep-dive lecture from masters (*NeetCode*, *Abdul Bari*, *William Fiset*). Pause frequently to sketch state transitions.
- **15–20 mins | Invariant Formulation**: Formulate the invariant in natural language (e.g., *"At the start of iteration $i$, $window$ contains at most $K$ distinct elements"*).
- **45–50 mins | Problem 1 (Warm-Up)**: Implement the baseline canonical problem in C#.
- **10 mins | Closed-Book Self-Test**: Test yourself on the fundamental invariant before sleeping.

#### 2. Practice Days (Days 2 to 12, Strictly 1 Problem/Day)
- **10 mins | Spaced Retrieval**: Answer the active recall prompt inside the collapsible `<details>` tag without looking at your notes.
- **25 mins | Mandatory Thinking Window**: **No typing for 25 minutes**. Draw pointers, recursion trees, or monotonic stacks on physical paper. Trace arrays with small $N$ ($N=0, 1, 2$) and edge cases (duplicates, negative values).
- **50 mins | C# Implementation**: Write type-safe, idiomatic C# code. Verify boundaries and off-by-one errors.
- **15 mins | Invariant Logging**: Document where your initial hypothesis failed and log the exact edge case that triggered an off-by-one or runtime exception.

---

## 🛠️ C# (.NET 6+) DSA Cheatsheet & Mental Models

In competitive programming and technical interviews, C# is an exceptional language provided you understand the runtime costs of managed memory and garbage collection.

### 1. Collections & Big-O Reference

| Data Structure | C# Type | Common Methods | Complexity | Gotcha / Tip |
|:---|:---|:---|:---:|:---|
| **Dynamic Array** | `List<T>` | `Add()`, `RemoveAt(i)`, `Count` | $O(1)$ amortized append, $O(N)$ remove | Pre-size with `new List<T>(capacity)` to eliminate reallocation overhead. |
| **Hash Map** | `Dictionary<TKey, TValue>` | `ContainsKey()`, `TryGetValue()`, `[key]` | $O(1)$ avg, $O(N)$ worst | Prefer `TryGetValue()` over `ContainsKey()` followed by indexer to avoid double hashing. |
| **Hash Set** | `HashSet<T>` | `Add()`, `Contains()`, `Remove()` | $O(1)$ avg | Implement `IEqualityComparer<T>` for custom class keys or use records / ValueTuples. |
| **LIFO Stack** | `Stack<T>` | `Push()`, `Pop()`, `Peek()`, `Count` | $O(1)$ | Avoid LINQ `.ElementAt()`; iterate natively with `foreach` or array copy. |
| **FIFO Queue** | `Queue<T>` | `Enqueue()`, `Dequeue()`, `Peek()` | $O(1)$ | Circular array under the hood. |
| **Min/Max Heap** | `PriorityQueue<TElement, TPriority>` | `Enqueue()`, `Dequeue()`, `Peek()` | $O(\log N)$ | Available in **.NET 6+**. Min-heap by default. For Max-heap, pass a custom `Comparer<TPriority>.Create((a, b) => b.CompareTo(a))`. |
| **Doubly Linked List** | `LinkedList<T>` | `AddFirst()`, `AddLast()`, `Remove()` | $O(1)$ node insert/delete | Node allocations produce GC pressure; prefer array-backed structures when possible. |
| **Sorted Set / Map** | `SortedSet<T>`, `SortedDictionary<K, V>` | `Add()`, `Remove()`, `Min`, `Max` | $O(\log N)$ | Red-Black Tree backed. |

### 2. High-Performance C# Idioms

#### Avoiding Double Hashing with `CollectionsMarshal` or `TryGetValue`
```csharp
// ❌ Suboptimal: 2 hash lookups
if (counts.ContainsKey(item)) {
    counts[item]++;
} else {
    counts[item] = 1;
}

// ✅ Idiomatic C#: 1 hash lookup
if (counts.TryGetValue(item, out int existingCount)) {
    counts[item] = existingCount + 1;
} else {
    counts[item] = 1;
}

// 🚀 Advanced .NET 6+ (Zero allocations):
ref int count = ref System.Runtime.InteropServices.CollectionsMarshal.GetValueRefOrAddDefault(counts, item, out _);
count++;
```

#### Lightweight ValueTuples vs Custom Classes
```csharp
// Use ValueTuples for BFS coordinates, Dijkstra graph edges, and memoization keys
Queue<(int row, int col, int dist)> queue = new();
queue.Enqueue((0, 0, 0));

// Dictionary with tuple composite keys (natively implements value equality)
Dictionary<(int index, int remainingSum), int> memo = new();
```

#### Array Slicing Without Allocations (`Span<T>`)
```csharp
int[] array = new int[] { 10, 20, 30, 40, 50 };
ReadOnlySpan<int> slice = array.AsSpan(1, 3); // Points directly to [20, 30, 40] without memory allocation
```

#### Beware of LINQ in Hot Loops
```csharp
// ❌ Avoid in O(N^2) or inner loops: creates delegate allocations and enumerator structs
int maxVal = nums.Max(); 

// ✅ Use direct iteration or Math.Max inside the loop:
int maxVal = int.MinValue;
for (int i = 0; i < nums.Length; i++) {
    if (nums[i] > maxVal) maxVal = nums[i];
}
```

---

## 🗺️ Topic Curriculum Overview

Each module spans **10 to 12 days** and contains a minimum of **15 curated problems** from NeetCode, CSES Problem Set, and Codeforces.

| # | Topic | Key Mental Model | Duration | Curated Problems |
|:---:|:---|:---|:---:|:---:|
| 01 | [Arrays, Hashing & Frequency Counting](01-arrays-and-hashing.md) | Value-to-index mapping, prefix state, bucket sort | 11 Days | 15 Problems |
| 02 | [Two Pointers](02-two-pointers.md) | Inward squeeze, fast & slow scan, partition invariants | 11 Days | 15 Problems |
| 03 | [Sliding Window](03-sliding-window.md) | Monotonic window expansion & dynamic contraction | 11 Days | 15 Problems |
| 04 | [Stack & Monotonic Stack](04-stack.md) | Immediate next greater/smaller element, nearest boundary | 11 Days | 15 Problems |
| 05 | [Binary Search & Answer Space](05-binary-search.md) | Predicate monotonicity $FF\dots TT$, boundary convergence | 11 Days | 15 Problems |
| 06 | [Linked Lists & Pointer Logic](06-linked-lists.md) | Sentinel dummy head, pointer rewiring, Floyd's cycle | 11 Days | 15 Problems |
| 07 | [Binary Trees & BSTs](07-binary-trees.md) | Subtree induction, DFS state return, BFS level tracking | 12 Days | 16 Problems |
| 08 | [Heaps & Priority Queues](08-heaps.md) | Dynamic order statistics, $K$-way merge, two-heap median | 11 Days | 15 Problems |
| 09 | [Backtracking & Recursion](09-backtracking.md) | Decision tree traversal, state exploration & restoration | 11 Days | 15 Problems |
| 10 | [Graphs I: BFS, DFS & Grid Traversals](10-graphs-traversal.md) | Connected components, shortest path in unweighted graph | 11 Days | 15 Problems |
| 11 | [Graphs II: Topo Sort, Dijkstra & DSU](11-graphs-advanced.md) | Dependency resolution, non-negative weighted paths, disjoint sets | 12 Days | 16 Problems |
| 12 | [1-D Dynamic Programming](12-1d-dp.md) | Subproblem recurrence, optimal substructure, memory rolling | 11 Days | 15 Problems |
| 13 | [2-D Dynamic Programming & Knapsack](13-2d-dp.md) | Grid paths, string alignment, bounded/unbounded choice | 12 Days | 16 Problems |
| 14 | [Greedy Algorithms & Intervals](14-greedy-intervals.md) | Local optimum, sorting invariants, endpoint scheduling | 11 Days | 15 Problems |
| 15 | [Bit Manipulation & Math Fundamentals](15-bit-manipulation-math.md) | Bitwise masks, XOR cancellation, modular arithmetic | 11 Days | 15 Problems |

---

## 📅 High-Level Roadmap

View the complete week-by-week timeline and study calendar: **[Course Schedule](schedule.md)**.

Ready to begin? Start with **[Topic 1: Arrays, Hashing & Frequency Counting](01-arrays-and-hashing.md)**.

