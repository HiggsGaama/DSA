---
layout: default
title: "Topic 06: Linked Lists & Fast/Slow Pointers"
---

# Topic 06: Linked Lists & Fast/Slow Pointers

[← Topic 05: Binary Search](05-binary-search.md) \| [Schedule](schedule.md) \| [Next: Binary Trees & BSTs →](07-binary-trees.md)

---

## Daily Timeboxing (1.5 – 2.0 Hours)
- **00:00 – 00:15 (15m) | Active Recall Pre-Test**: Answer the conceptual questions below before reading further.
- **00:15 – 00:40 (25m) | YouTube Conceptual Anchor**: Watch the curated video segment on Floyd's Cycle-Finding algorithm.
- **00:40 – 01:30 (50m) | Coding Deep Dive**: Pen-and-paper dry run of Reorder List & Reverse Nodes in k-Group.
- **01:30 – 01:45 (15m) | Post-Session Flashcards**: Complete the flashcard prompts from memory.

---

## 1. Active Recall Pre-Test (15 Minutes)
Attempt to answer these questions on paper before expanding the solutions.

<details>
<summary><strong>Prompt 1: Why does Floyd's Tortoise and Hare algorithm guarantee cycle detection in $O(N)$ time without extra space?</strong></summary>

Let the fast pointer move at 2 steps/iteration and the slow pointer at 1 step/iteration.
Once both enter a cycle of length $C$, their relative distance decreases by exactly $2 - 1 = 1$ node on every step. 
Because the distance strictly closes at a rate of 1 node per iteration, fast is guaranteed to catch up to slow within at most $C$ iterations after entering the loop. 
This requires $O(1)$ auxiliary space and $O(N)$ time, avoiding the $O(N)$ memory required by a hash set of visited nodes.
</details>

<details>
<summary><strong>Prompt 2: What is the algebraic proof that resetting one pointer to `head` after cycle collision finds the exact start of the cycle?</strong></summary>

Let:
- $L$ = distance from list head to the cycle entrance.
- $C$ = length of the cycle.
- $d$ = distance from cycle entrance to the collision point.

When slow reaches the collision point:
$$\text{Distance}(\text{slow}) = L + d$$
$$\text{Distance}(\text{fast}) = L + d + kC \quad (\text{for some integer } k \ge 1)$$
Because fast moves at twice the speed of slow:
$$2(L + d) = L + d + kC \implies L + d = kC \implies L = kC - d$$
The distance from the head to the entrance is $L$. 
The remaining distance from the collision point forward to the entrance along the cycle is $C - d$, which differs from $L$ by an integer multiple of $C$. 
Therefore, if pointer $P_1$ starts at `head` and pointer $P_2$ starts at the collision point, moving both at identical speed of 1 step/iteration will cause them to collide exactly at the cycle entrance after $L$ steps.
</details>

<details>
<summary><strong>Prompt 3: Why are Dummy Sentinel Nodes considered the most important defensive programming pattern in linked list algorithms?</strong></summary>

Sentinel nodes (e.g., `dummy = ListNode(0, head)`) eliminate null-pointer checks for edge cases where the head of the list itself must be modified, removed, or prepended (e.g., deleting the first element, merging two lists, or partitioning). 
The operation code treats all nodes identically, and the resulting list is cleanly retrieved via `dummy.next`.
</details>

---

## 2. YouTube Conceptual Anchor (25 Minutes)

Recommended Video Anchors:
1. **NeetCode**: [Linked List Cycle II & Floyd's Algorithm Mathematical Proof](https://www.youtube.com/watch?v=wjYnzkAhcNk) (14 mins)
2. **Abdul Bari**: [Singly and Doubly Linked List Operations](https://www.youtube.com/watch?v=B31LgI4Y4DQ) (20 mins)

### Core Invariants & Mental Model
1. **The 3-Pointer Reversal Invariant**: When reversing links iteratively, maintain `prev`, `curr`, and `nxt`. Always save `nxt = curr.next` before overwriting `curr.next = prev`, then shift `prev = curr` and `curr = nxt`.
2. **Tortoise and Hare Midpoint Invariant**: 
   - Initializing `fast = head, slow = head` lands `slow` at the second middle node for even-length lists.
   - Initializing `fast = head.next, slow = head` lands `slow` at the first middle node (ideal for splitting a list into two equal halves).
3. **Doubly Linked List + Hash Map (LRU Cache)**:
   - Doubly linked list allows $O(1)$ removal and appending of arbitrary nodes.
   - Hash map provides $O(1)$ key-to-node lookups.

---

## 3. Canonical Code Templates

### Pattern A: In-Place Iterative Reversal ($O(N)$ Time, $O(1)$ Space)
```python
from typing import Optional

class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

def reverse_list(head: Optional[ListNode]) -> Optional[ListNode]:
    prev = None
    curr = head
    
    while curr:
        nxt = curr.next  # 1. Preserve forward link
        curr.next = prev # 2. Reverse link
        prev = curr      # 3. Advance prev
        curr = nxt       # 4. Advance curr
        
    return prev
```

### Pattern B: Fast & Slow Pointer Cycle Entrance ($O(N)$ Time, $O(1)$ Space)
```python
def detect_cycle_entry(head: Optional[ListNode]) -> Optional[ListNode]:
    slow = fast = head
    
    # 1. Detect collision inside cycle
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            break
    else:
        return None  # No cycle found
        
    # 2. Reset one pointer to head to find entrance
    ptr1 = head
    ptr2 = slow
    while ptr1 != ptr2:
        ptr1 = ptr1.next
        ptr2 = ptr2.next
        
    return ptr1
```

### Pattern C: Merge Two Sorted Lists with Dummy Node ($O(N)$ Time, $O(1)$ Space)
```python
def merge_two_lists(l1: Optional[ListNode], l2: Optional[ListNode]) -> Optional[ListNode]:
    dummy = ListNode(0)
    tail = dummy
    
    while l1 and l2:
        if l1.val <= l2.val:
            tail.next = l1
            l1 = l1.next
        else:
            tail.next = l2
            l2 = l2.next
        tail = tail.next
        
    tail.next = l1 if l1 else l2
    return dummy.next
```

---

## 4. Coding Deep Dive: Worked Walkthrough (50 Minutes)

### Problem: Reorder List (LeetCode 143)
**Problem Statement**: Given a list $L_0 \to L_1 \to \dots \to L_{n-1} \to L_n$, reorder it to:
$$L_0 \to L_n \to L_1 \to L_{n-1} \to L_2 \to L_{n-2} \to \dots$$
Do this in-place without altering node values.

#### Pen-and-Paper Dry Run Trace
Let list be: `1 -> 2 -> 3 -> 4 -> 5`.
1. **Find Middle**:
   - `slow = 1`, `fast = 1`
   - Step 1: `slow = 2`, `fast = 3`
   - Step 2: `slow = 3`, `fast = 5`
   - Middle is node `3`. Split into `1 -> 2 -> 3` and `4 -> 5`.
2. **Reverse Second Half**:
   - Reverse `4 -> 5` to get `5 -> 4`.
   - Disconnect first half: `3.next = None`.
3. **Interleave Two Halves**:
   - First half ($H_1$): `1 -> 2 -> 3`
   - Second half ($H_2$): `5 -> 4`

| Step | Current Node $H_1$ | Current Node $H_2$ | Interleaved Pointer Actions | Resulting Chain |
|:---:|:---:|:---:|:---|:---|
| 1 | 1 | 5 | $1 \to 5$, $5 \to 2$ | `1 -> 5 -> 2` |
| 2 | 2 | 4 | $2 \to 4$, $4 \to 3$ | `1 -> 5 -> 2 -> 4 -> 3` |
| End | 3 | None | $H_2$ exhausted, terminate | `1 -> 5 -> 2 -> 4 -> 3` |

Time Complexity: $O(N)$ across all three linear passes.
Space Complexity: $O(1)$ auxiliary space.

---

## 5. Curated Problem Set (15 Problems)

Track your daily solutions by checking off the markdown boxes.

### NeetCode / LeetCode (Core)
- [ ] [LeetCode 206: Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/) | Easy | 3-Pointer Iterative Invariant
- [ ] [LeetCode 21: Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/) | Easy | Dummy Node Splicing
- [ ] [LeetCode 141: Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/) | Easy | Floyd's Cycle Detection
- [ ] [LeetCode 142: Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/) | Medium | Cycle Entrance Algebraic Reset
- [ ] [LeetCode 143: Reorder List](https://leetcode.com/problems/reorder-list/) | Medium | Fast/Slow Split + Reverse + Interleave
- [ ] [LeetCode 19: Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/) | Medium | Two Pointers Separated by N Steps
- [ ] [LeetCode 146: LRU Cache](https://leetcode.com/problems/lru-cache/) | Medium | Doubly Linked List + Hash Map
- [ ] [LeetCode 23: Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/) | Hard | Divide & Conquer / Min-Heap
- [ ] [LeetCode 287: Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/) | Medium | Array Value as Linked List Pointer Cycle
- [ ] [LeetCode 25: Reverse Nodes in k-Group](https://leetcode.com/problems/reverse-nodes-in-k-group/) | Hard | Subsegment Length Check & Invariant Reversal

### CSES Problem Set (Free Tier, No Login Wall)
- [ ] [CSES 2162: Josephus Problem I](https://cses.fi/problemset/task/2162) | Introductory Problems | Circular Linked List Simulation
- [ ] [CSES 2163: Josephus Problem II](https://cses.fi/problemset/task/2163) | Additional Problems | Fenwick Tree / Balanced Order-Statistic Tree
- [ ] [CSES 1754: Coin Piles](https://cses.fi/problemset/task/1754) | Introductory Problems | State Invariant Parity Elimination

### Codeforces & AtCoder (Rating 800–1400)
- [ ] [Codeforces 116A: Tram](https://codeforces.com/problemset/problem/116/A) | 800 | Sequential State Capacity Tracking
- [ ] [AtCoder ABC158_D: String Formation](https://atcoder.jp/contests/abc158/tasks/abc158_d) | 1000 | Deque / Doubly Linked List End Insertion

---

## 6. Post-Session Active Recall Flashcards (15 Minutes)

<details>
<summary><strong>Flashcard 1: How does LeetCode 287 (Find the Duplicate Number) map an integer array of length $N+1$ with values in $[1, N]$ into a linked list cycle problem?</strong></summary>

Consider indices as node addresses and values as pointers: $i \to \text{nums}[i]$.
Because all values are in $[1, N]$, index 0 has an outgoing edge but no incoming edge (values never equal 0). 
By Pigeonhole Principle, at least one value must appear twice, meaning two distinct indices point to the same next node. This creates a cycle whose entrance is precisely the duplicate value. Floyd's cycle detection solves it in $O(N)$ time and $O(1)$ space without mutating the array.
</details>

<details>
<summary><strong>Flashcard 2: In LRU Cache, why is a Doubly Linked List required instead of a Singly Linked List?</strong></summary>

When an existing key is accessed or updated, it must be removed from its current position and moved to the front (most recently used). 
Deleting a node from a Singly Linked List requires access to its *predecessor*, which takes $O(N)$ traversal time. A Doubly Linked List stores `prev` pointers, allowing $O(1)$ unlinking: `node.prev.next = node.next; node.next.prev = node.prev`.
</details>

<details>
<summary><strong>Flashcard 3: How does Divide & Conquer merge $K$ sorted lists of total length $N$ in $O(N \log K)$ time?</strong></summary>

Instead of merging lists one by one (which takes $O(K \cdot N)$ time), merge pairs of lists iteratively:
- Round 1: Merge $K$ lists into $K/2$ lists.
- Round 2: Merge $K/2$ lists into $K/4$ lists.
- Total rounds: $\log_2 K$.
Each round processes every node in the $K$ lists once ($O(N)$ work). Overall complexity: $O(N \log K)$.
</details>

---

[← Topic 05: Binary Search](05-binary-search.md) \| [Next Topic: Binary Trees & BSTs →](07-binary-trees.md)
