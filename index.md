---
layout: default
title: "Home | DSA Mastery Curriculum"
---

# Data Structures & Algorithms: The 24-Week Mastery Curriculum

> A complete, publish-ready, active-recall-driven DSA roadmap optimized for **1.5 to 2.0 hours per day** (1–2 problems daily). Built with zero build steps, zero CSS, and zero external dependencies—hosted directly via Jekyll on GitHub Pages.

---

## 1. Pedagogical Architecture & Philosophy

Most DSA learners fail not because they lack intelligence, but because they succumb to the **Illusion of Competence**: watching an algorithm tutorial, feeling an intuitive sense of understanding, and finding themselves completely unable to write code on a blank editor 48 hours later.

This curriculum is engineered strictly upon cognitive science principles:
1. **Active Retrieval Practice (Pre-Test Effect)**: Attempting to recall core invariants *before* viewing solutions triggers neural plasticity and primes your brain to encode edge cases.
2. **Cognitive Load Optimization**: Each session targets exactly **one core invariant** and **1–2 canonical problems** within a strict 90–120 minute window, preventing mental fatigue.
3. **Spaced Deliberate Practice**: Concepts from Day 1 re-emerge in Week 6, Week 12, and Week 20 in more complex graph and dynamic programming scenarios.

---

## 2. The Strict Daily 90–120 Minute Protocol

Every day follows this non-negotiable 4-block timebox. Set a physical timer before beginning.

```text
+----------------------------------------------------------------------------------+
|                     DAILY TIME BUDGET: 90 - 120 MINUTES                          |
+----------------------+--------------------+--------------------+-----------------+
| 00:00 - 00:15 (15m)  | 00:15 - 00:40 (25m)| 00:40 - 01:30 (50m)| 01:30 - 01:45   |
| Active Recall        | YouTube Conceptual | Coding Deep Dive   | Post-Session    |
| Pre-Test             | Anchor             | (1-2 Problems)     | Flashcards      |
+----------------------+--------------------+--------------------+-----------------+
```

### Block 1: 00:00 – 00:15 (15 min) | Active Recall Pre-Test
* **Objective**: Closed-book mental activation.
* **Protocol**: Open today's module. Do **not** look at the code templates. Read the pre-test prompts and write your answers down on paper or in a scratchpad.
* **Why**: Forces memory retrieval under friction, exposing exact gaps in understanding.

### Block 2: 00:15 – 00:40 (25 min) | YouTube Conceptual Anchor
* **Objective**: Mental model construction & invariant extraction.
* **Protocol**: Watch the curated 15–25 minute video anchor at 1.25x–1.5x speed. Focus exclusively on:
  1. What is the fundamental invariant maintained across iterations?
  2. Why does the naive $O(N^2)$ brute force fail?
  3. What state is tracked in auxiliary memory?

### Block 3: 00:40 – 01:30 (50 min) | Coding Deep Dive (1–2 Problems)
* **Objective**: Implementation without crutches.
* **Protocol**:
  1. **Minutes 00–15**: Pen-and-paper dry run on a small, hand-crafted test case (including an edge case: empty input, duplicates, or single-element array).
  2. **Minutes 15–40**: Write the code on your platform (LeetCode / CSES / Codeforces). No copy-pasting. Run against sample tests.
  3. **Minutes 40–50**: Analyze time & space complexity, annotate runtime invariants, and commit solution.
* **Rule**: If stuck after 25 minutes of dry running, look at the **first hint only**. Never jump to the full code solution.

### Block 4: 01:30 – 01:45 (15 min) | Post-Session Active Recall Flashcards
* **Objective**: Memory consolidation before sleep.
* **Protocol**: Close all tabs. Answer the 4–5 flashcard review prompts located at the bottom of the module using `<details><summary>` dropdowns.

---

## 3. Curriculum Syllabus (15 Core Topics)

Each topic module contains conceptual foundations, invariant templates, worked walkthroughs, self-testing flashcards, and a curated set of **15+ problems** drawn from NeetCode/LeetCode, CSES, and Codeforces/AtCoder.

| # | Topic Module | Core Invariant / Pattern | Curated Problems | Suggested Weeks |
|---|---|---|---|---|
| 01 | [Arrays, Hashing & Frequency Counting](01-arrays-and-hashing.md) | Value-to-Index mapping, Prefix parity, $O(1)$ lookups | 16 Problems | Weeks 1 – 2 |
| 02 | [Two Pointers](02-two-pointers.md) | Sorted space monotonicity, Opposing & parallel scans | 15 Problems | Weeks 3 – 4 |
| 03 | [Sliding Window](03-sliding-window.md) | Contiguous subarray state maintenance, Expand/Contract | 15 Problems | Weeks 5 – 6 |
| 04 | [Stack & Monotonic Stack](04-stack.md) | LIFO processing, Next Greater/Smaller Element invariant | 15 Problems | Weeks 7 – 8 |
| 05 | [Binary Search & Search Space Reduction](05-binary-search.md) | Monotonic predicate $P(x)$, Boundary convergence | 16 Problems | Weeks 9 – 10 |
| 06 | [Linked Lists & Fast/Slow Pointers](06-linked-lists.md) | Pointer manipulation, Tortoise & Hare cycle invariant | 15 Problems | Week 11 |
| 07 | [Binary Trees & BSTs](07-binary-trees.md) | Recursive decomposition, Depth/Height state, BST ordering | 16 Problems | Weeks 12 – 13 |
| 08 | [Heaps & Priority Queues](08-heaps.md) | Dynamic extremum tracking, Top-K maintenance | 15 Problems | Week 14 |
| 09 | [Backtracking & Recursive Search](09-backtracking.md) | State tree exploration, Choice-Constraint-Undo cycle | 15 Problems | Weeks 15 – 16 |
| 10 | [Graphs I: BFS, DFS & Grid Traversals](10-graphs-traversal.md) | Unweighted shortest path, Connected components, Visited set | 16 Problems | Weeks 17 – 18 |
| 11 | [Graphs II: Topo Sort, Dijkstra & DSU](11-graphs-advanced.md) | DAG dependencies, Non-negative edge SP, Disjoint sets | 16 Problems | Weeks 19 – 20 |
| 12 | [1-D Dynamic Programming](12-1d-dp.md) | Optimal substructure, Overlapping subproblems, Memoization | 16 Problems | Weeks 21 – 22 |
| 13 | [2-D Dynamic Programming & Knapsack](13-2d-dp.md) | 2D Grid state, Subsequence alignment, Decision branches | 15 Problems | Week 23 |
| 14 | [Greedy Algorithms & Intervals](14-greedy-intervals.md) | Locally optimal choice, Interval sorting by endpoints | 15 Problems | Week 24 |
| 15 | [Bit Manipulation & Core Math](15-bit-manipulation-math.md) | Bitwise masks, XOR cancellation, Modular arithmetic | 15 Problems | Supplemental / Review |

**Total Curated Problems across the Course**: **230+ problems** (all free to access).

---

## 4. Master Pacing Calendar

Ready to map your 24-week journey day by day?

👉 **[Access the Full 24-Week Daily Schedule](schedule.md)**

The calendar assigns exactly 1–2 specific problems to each day of the week, with dedicated consolidation and mock contest days on weekends.

---

## 5. How to Track Your Progress

This curriculum uses pure GitHub Flavored Markdown task lists. To track your personal progress:

1. Fork this repository to your personal GitHub account.
2. Enable **GitHub Pages** in your repository settings (`Settings -> Pages -> Branch: main -> / (root)`).
3. As you solve each problem and complete flashcard reviews, check off the markdown boxes directly on GitHub or in your editor:
   ```markdown
   - [x] Two Sum (Solved in 18 min on 2026-11-16)
   - [ ] Valid Anagram
   ```
4. Commit your changes. GitHub Pages updates automatically with zero compilation steps.

---

[Start Topic 01: Arrays, Hashing & Frequency Counting →](01-arrays-and-hashing.md)
