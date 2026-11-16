---
layout: page
title: "Linked Lists & Fast/Slow Pointers"
permalink: /linked-lists/
---

# Linked Lists & Fast/Slow Pointers

> **Estimated Time**: 11 Days • **Pace**: 1.5–2 hours/day (Strictly 1 problem/day) • **Language**: C# (.NET 6+)

---

## 🛠️ C# Mental Model & Gotchas

### 1. Reference Types & Null Safety
In C#, `ListNode` is a reference type allocated on the managed heap:
```csharp
public class ListNode {
    public int val;
    public ListNode next;
    public ListNode(int val = 0, ListNode next = null) {
        this.val = val;
        this.next = next;
    }
}
```
- **NullReferenceException**: Always guard against dereferencing `curr.next` when `curr` could be `null`.
- **Reference Equality**: Two `ListNode` references `a == b` check for object memory address identity, not value equality.

### 2. The Sentinel (Dummy Head) Invariant
Almost every linked list manipulation problem is simplified by prepending a sentinel node:
```csharp
ListNode dummy = new ListNode(0, head);
```
The dummy node eliminates special casing when inserting, deleting, or reordering the very first node of the list. Always return `dummy.next`.

### 3. Pointer Rewiring Order
When rewiring pointers, always cache the upcoming node **before** breaking references:
```csharp
ListNode nextNode = curr.next; // 1. Save upcoming node
curr.next = prev;              // 2. Safely reverse direction
prev = curr;                   // 3. Advance prev
curr = nextNode;               // 4. Advance curr
```

---

## 📅 Day-by-Day Schedule

### Day 1: Theory Anchor & Baseline Pattern
- **Step 1 (45 mins) | Concept Video**: [Linked List Data Structure & Inversions](https://www.youtube.com/watch?v=G0_I-ZF0S38) by *NeetCode*
  - Key focus: What is the 3-pointer invariant (`prev`, `curr`, `next`) during iterative list reversal?
- **Step 2 (20 mins) | Pen & Paper Invariant**:
  - Draw 4 nodes `1 -> 2 -> 3 -> 4 -> null`. Walk through the pointer inversion step-by-step. Verify where `prev` and `curr` end up when `curr == null`.
- **Step 3 (45 mins) | Problem 1 (Warm-Up)**:
  - [Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/) (NeetCode - Easy)
  - Objective: Implement iterative $O(N)$ reversal with $O(1)$ extra space.
- **Step 4 (10 mins) | Active Recall**:
  <details>
  <summary><b>🧠 Self-Test: When reversing a linked list iteratively, what does <code>prev</code> point to when the loop finishes?</b></summary>
  <br>
  <b>Answer:</b> <code>prev</code> points to the new head of the reversed list (the original tail). <code>curr</code> is <code>null</code>.
  </details>

---

### Day 2: Sentinel Node Splicing (Merge Two Sorted Lists)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: Why does a dummy node eliminate edge cases in list merging?</b></summary>
  <br>
  <b>Answer:</b> It provides a permanent anchor head so that the merged list's initial node does not require conditional checks. We simply return <code>dummy.next</code>.
  </details>
- **Deep Practice (75–85 mins) | Problem 2**:
  - [Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/) (NeetCode - Easy)
  - Thinking prompts: Compare `list1.val` and `list2.val`. Splicing existing nodes in-place requires $O(1)$ extra memory.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public ListNode MergeTwoLists(ListNode list1, ListNode list2) {
          ListNode dummy = new(0);
          ListNode tail = dummy;
          
          while (list1 != null && list2 != null) {
              if (list1.val <= list2.val) {
                  tail.next = list1;
                  list1 = list1.next;
              } else {
                  tail.next = list2;
                  list2 = list2.next;
              }
              tail = tail.next;
          }
          
          tail.next = list1 ?? list2;
          return dummy.next;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N + M)$, Space: $O(1)$.
  </details>

---

### Day 3: Fast & Slow Pointers for List Halving & Interleaving
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: In fast & slow pointer traversal (<code>fast = fast.next.next</code>, <code>slow = slow.next</code>), where is <code>slow</code> when <code>fast</code> hits the end?</b></summary>
  <br>
  <b>Answer:</b> <code>slow</code> is exactly at the midpoint of the linked list.
  </details>
- **Deep Practice (75–85 mins) | Problem 3**:
  - [Reorder List](https://leetcode.com/problems/reorder-list/) (NeetCode - Medium)
  - Thinking prompts: 1. Find midpoint; 2. Reverse second half; 3. Merge alternating nodes from both halves.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public void ReorderList(ListNode head) {
          if (head?.next == null) return;
          
          // Step 1: Find middle
          ListNode slow = head, fast = head;
          while (fast.next != null && fast.next.next != null) {
              slow = slow.next;
              fast = fast.next.next;
          }
          
          // Step 2: Reverse second half
          ListNode prev = null, curr = slow.next;
          slow.next = null; // Split the list into two halves
          while (curr != null) {
              ListNode nextNode = curr.next;
              curr.next = prev;
              prev = curr;
              curr = nextNode;
          }
          
          // Step 3: Interleave first half (head) and reversed second half (prev)
          ListNode p1 = head, p2 = prev;
          while (p2 != null) {
              ListNode next1 = p1.next;
              ListNode next2 = p2.next;
              
              p1.next = p2;
              p2.next = next1;
              
              p1 = next1;
              p2 = next2;
          }
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N)$, Space: $O(1)$.
  </details>

---

### Day 4: Fixed-Offset Two Pointers (Remove Nth From End)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: How do you position two pointers so the left pointer lands directly before the N-th node from the end?</b></summary>
  <br>
  <b>Answer:</b> Advance the <code>fast</code> pointer $N + 1$ steps ahead from a <code>dummy</code> node. Then advance both <code>slow</code> and <code>fast</code> together until <code>fast == null</code>. <code>slow</code> is then situated immediately before the target node.
  </details>
- **Deep Practice (75–85 mins) | Problem 4**:
  - [Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/) (NeetCode - Medium)
  - Thinking prompts: Use dummy head to gracefully handle removing the original head of the list.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public ListNode RemoveNthFromEnd(ListNode head, int n) {
          ListNode dummy = new(0, head);
          ListNode slow = dummy;
          ListNode fast = dummy;
          
          // Advance fast by n + 1 steps
          for (int i = 0; i <= n; i++) {
              fast = fast.next;
          }
          
          while (fast != null) {
              slow = slow.next;
              fast = fast.next;
          }
          
          slow.next = slow.next.next;
          return dummy.next;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N)$ single pass, Space: $O(1)$.
  </details>

---

### Day 5: Deep Copy with Dictionary vs In-Place Interweaving
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: How can a linked list with random pointers be cloned in $O(1)$ space without a hash map?</b></summary>
  <br>
  <b>Answer:</b> By interleaving cloned nodes directly next to originals (<code>A -&gt; A' -&gt; B -&gt; B'</code>). The clone's random pointer is simply <code>curr.next.random = curr.random.next</code>. Then decouple the lists.
  </details>
- **Deep Practice (75–85 mins) | Problem 5**:
  - [Copy List with Random Pointer](https://leetcode.com/problems/copy-list-with-random-pointer/) (NeetCode - Medium)
  - Thinking prompts: Compare `Dictionary<Node, Node>` mapping old to new nodes vs the 3-pass in-place interleave technique.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public Node CopyRandomList(Node head) {
          if (head == null) return null;
          
          Dictionary<Node, Node> cloneMap = new();
          
          // Pass 1: Clone all nodes without pointers
          Node curr = head;
          while (curr != null) {
              cloneMap[curr] = new Node(curr.val);
              curr = curr.next;
          }
          
          // Pass 2: Connect next and random pointers
          curr = head;
          while (curr != null) {
              if (curr.next != null) cloneMap[curr].next = cloneMap[curr.next];
              if (curr.random != null) cloneMap[curr].random = cloneMap[curr.random];
              curr = curr.next;
          }
          
          return cloneMap[head];
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N)$, Space: $O(N)$.
  </details>

---

### Day 6: Elementary Math & Carry Propagation
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: In Add Two Numbers, when does the loop terminate?</b></summary>
  <br>
  <b>Answer:</b> When <code>l1 == null &amp;&amp; l2 == null &amp;&amp; carry == 0</code>. Always verify that a trailing carry of 1 is not dropped!
  </details>
- **Deep Practice (75–85 mins) | Problem 6**:
  - [Add Two Numbers](https://leetcode.com/problems/add-two-numbers/) (NeetCode - Medium)
  - Thinking prompts: Digits are stored in reverse order. Traverse both lists simultaneously, computing `sum = v1 + v2 + carry`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public ListNode AddTwoNumbers(ListNode l1, ListNode l2) {
          ListNode dummy = new(0);
          ListNode tail = dummy;
          int carry = 0;
          
          while (l1 != null || l2 != null || carry != 0) {
              int v1 = l1?.val ?? 0;
              int v2 = l2?.val ?? 0;
              int sum = v1 + v2 + carry;
              
              carry = sum / 10;
              tail.next = new ListNode(sum % 10);
              tail = tail.next;
              
              l1 = l1?.next;
              l2 = l2?.next;
          }
          return dummy.next;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(\max(N, M))$, Space: $O(\max(N, M))$.
  </details>

---

### Day 7: Floyd's Cycle Detection (Tortoise and Hare)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: Why does Floyd's cycle algorithm guarantee that <code>fast</code> and <code>slow</code> will meet if a cycle exists?</b></summary>
  <br>
  <b>Answer:</b> Inside a cycle of length $C$, the relative distance between <code>fast</code> and <code>slow</code> decreases by exactly 1 on every iteration ($2 - 1 = 1$). Therefore, they must collide within at most $C$ iterations.
  </details>
- **Deep Practice (75–85 mins) | Problem 7**:
  - [Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/) (NeetCode - Easy)
  - Thinking prompts: Check `fast != null && fast.next != null`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public bool HasCycle(ListNode head) {
          ListNode slow = head;
          ListNode fast = head;
          
          while (fast != null && fast.next != null) {
              slow = slow.next;
              fast = fast.next.next;
              if (slow == fast) return true;
          }
          return false;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N)$, Space: $O(1)$.
  </details>

---

### Day 8: Floyd's Cycle on Array Values (Find the Duplicate Number)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: How can an array of integers in range $[1 \dots n]$ be viewed as a linked list?</b></summary>
  <br>
  <b>Answer:</b> Index $i$ points to node $nums[i]$. Because an element is duplicated, two different indices point to the same value, creating a cycle.
  </details>
- **Deep Practice (75–85 mins) | Problem 8**:
  - [Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/) (NeetCode - Medium)
  - Thinking prompts: Find collision point. Reset `slow = 0`. Advance both at 1 step per turn; the point where they meet is the cycle entrance (the duplicate number!).
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public int FindDuplicate(int[] nums) {
          int slow = nums[0];
          int fast = nums[0];
          
          // Phase 1: Find collision
          do {
              slow = nums[slow];
              fast = nums[nums[fast]];
          } while (slow != fast);
          
          // Phase 2: Find cycle entrance
          slow = nums[0];
          while (slow != fast) {
              slow = nums[slow];
              fast = nums[fast];
          }
          return slow;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N)$, Space: $O(1)$ without modifying input array.
  </details>

---

### Day 9: The Systems Pattern (LRU Cache Design)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: Why is a doubly linked list combined with a hash map used for LRU Cache instead of a singly linked list?</b></summary>
  <br>
  <b>Answer:</b> Removing a node from the middle of a doubly linked list takes $O(1)$ given its pointer because we can rewire <code>node.prev.next = node.next</code> directly. A singly linked list would require $O(N)$ traversal to locate the preceding node.
  </details>
- **Deep Practice (75–85 mins) | Problem 9**:
  - [LRU Cache](https://leetcode.com/problems/lru-cache/) (NeetCode - Medium)
  - Thinking prompts: Maintain pseudo `head` and `tail` sentinels in a custom Doubly Linked List, and map `key -> Node` in a `Dictionary`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class LRUCache {
      private class DNode {
          public int key, val;
          public DNode prev, next;
          public DNode(int k, int v) { key = k; val = v; }
      }

      private readonly int _capacity;
      private readonly Dictionary<int, DNode> _map = new();
      private readonly DNode _head = new(0, 0);
      private readonly DNode _tail = new(0, 0);

      public LRUCache(int capacity) {
          _capacity = capacity;
          _head.next = _tail;
          _tail.prev = _head;
      }
      
      public int Get(int key) {
          if (!_map.TryGetValue(key, out var node)) return -1;
          Remove(node);
          InsertAtHead(node);
          return node.val;
      }
      
      public void Put(int key, int value) {
          if (_map.TryGetValue(key, out var node)) {
              node.val = value;
              Remove(node);
              InsertAtHead(node);
          } else {
              if (_map.Count == _capacity) {
                  var lru = _tail.prev;
                  Remove(lru);
                  _map.Remove(lru.key);
              }
              var newNode = new DNode(key, value);
              _map[key] = newNode;
              InsertAtHead(newNode);
          }
      }
      
      private void Remove(DNode node) {
          node.prev.next = node.next;
          node.next.prev = node.prev;
      }
      
      private void InsertAtHead(DNode node) {
          node.next = _head.next;
          node.prev = _head;
          _head.next.prev = node;
          _head.next = node;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(1)$ for both `Get` and `Put`, Space: $O(\text{capacity})$.
  </details>

---

### Day 10: Multi-Way Divide & Conquer (Merge K Sorted Lists)
- **Warm-Up (10 mins)**:
  <details>
  <summary><b>🧠 Quick Recall: What are the two optimal ways to merge $K$ sorted lists of total length $N$?</b></summary>
  <br>
  <b>Answer:</b> 1. Min-Heap of size $K$ (inserting head of each list, popping minimum and pushing next); 2. Divide and conquer pairwise merging ($O(N \log K)$ time and $O(1)$ auxiliary space).
  </details>
- **Deep Practice (75–85 mins) | Problem 10**:
  - [Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/) (NeetCode - Hard)
  - Thinking prompts: Use modern .NET 6+ `PriorityQueue<ListNode, int>`.
- **Reference Solution**:
  <details>
  <summary><b>View Optimal C# Solution & Complexity</b></summary>

  ```csharp
  public class Solution {
      public ListNode MergeKLists(ListNode[] lists) {
          if (lists == null || lists.Length == 0) return null;
          
          PriorityQueue<ListNode, int> pq = new();
          foreach (var head in lists) {
              if (head != null) {
                  pq.Enqueue(head, head.val);
              }
          }
          
          ListNode dummy = new(0);
          ListNode tail = dummy;
          
          while (pq.Count > 0) {
              ListNode smallest = pq.Dequeue();
              tail.next = smallest;
              tail = tail.next;
              
              if (smallest.next != null) {
                  pq.Enqueue(smallest.next, smallest.next.val);
              }
          }
          return dummy.next;
      }
  }
  ```
  <b>Complexity:</b> Time: $O(N \log K)$, Space: $O(K)$ heap.
  </details>

---

### Day 11: CSES & Competitive Benchmarks + Spaced Retrieval
- **Benchmark 1**: [Reverse Nodes in k-Group](https://leetcode.com/problems/reverse-nodes-in-k-group/) (Hard boundary pointer manipulation)
- **Benchmark 2**: [CSES: Josephus Problem I](https://cses.fi/problemset/task/2162) (Circular list elimination simulation)
- **Benchmark 3**: [CSES: Josephus Problem II](https://cses.fi/problemset/task/2163) (Large-scale circular removal)
- **Benchmark 4**: [CSES: Playlist](https://cses.fi/problemset/task/1141) (Unique sequence verification)
- **Benchmark 5**: [Codeforces 158A - Next Round](https://codeforces.com/problemset/problem/158/A) (Sequential boundary conditions)

---

### Spaced Retrieval Flashcards

<details>
<summary><b>Flashcard 1: Why is a sentinel dummy node particularly useful when modifying a linked list?</b></summary>
<br>
It prevents special-case code when deleting or modifying the head node of the list, ensuring that every valid node in the list always has a non-null predecessor node.
</details>

<details>
<summary><b>Flashcard 2: In Floyd's Cycle Detection, why does starting one pointer at head and one pointer at the meeting point find the cycle entrance?</b></summary>
<br>
Let $L$ be distance from head to cycle entrance, and $X$ be distance from entrance to meeting point. The total distance traversed satisfies $2(L + X) = L + X + nC \implies L = nC - X$. Thus, walking $L$ steps from both locations lands precisely at the cycle entrance!
</details>

<details>
<summary><b>Flashcard 3: What is the benefit of .NET 6+ <code>PriorityQueue&lt;TElement, TPriority&gt;</code> over older C# heap alternatives?</b></summary>
<br>
It provides a built-in, 4-ary min-heap implementation where priority is decoupled from element type, avoiding the need to write custom heap classes or implement <code>IComparable</code> on problem entities.
</details>

---

## 🎯 Topic Checklist (15 Problems)

| Day | # | Problem | Platform | Difficulty | Status |
|:---:|:---:|:---|:---:|:---:|:---:|
| Day 1 | 1 | [Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/) | NeetCode | Easy | - [ ] |
| Day 2 | 2 | [Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/) | NeetCode | Easy | - [ ] |
| Day 3 | 3 | [Reorder List](https://leetcode.com/problems/reorder-list/) | NeetCode | Medium | - [ ] |
| Day 4 | 4 | [Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/) | NeetCode | Medium | - [ ] |
| Day 5 | 5 | [Copy List with Random Pointer](https://leetcode.com/problems/copy-list-with-random-pointer/) | NeetCode | Medium | - [ ] |
| Day 6 | 6 | [Add Two Numbers](https://leetcode.com/problems/add-two-numbers/) | NeetCode | Medium | - [ ] |
| Day 7 | 7 | [Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/) | NeetCode | Easy | - [ ] |
| Day 8 | 8 | [Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/) | NeetCode | Medium | - [ ] |
| Day 9 | 9 | [LRU Cache](https://leetcode.com/problems/lru-cache/) | NeetCode | Medium | - [ ] |
| Day 10 | 10 | [Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/) | NeetCode | Hard | - [ ] |
| Day 11 | 11 | [Reverse Nodes in k-Group](https://leetcode.com/problems/reverse-nodes-in-k-group/) | LeetCode | Hard | - [ ] |
| Day 11 | 12 | [Josephus Problem I](https://cses.fi/problemset/task/2162) | CSES | Easy | - [ ] |
| Day 11 | 13 | [Josephus Problem II](https://cses.fi/problemset/task/2163) | CSES | Hard | - [ ] |
| Day 11 | 14 | [Playlist](https://cses.fi/problemset/task/1141) | CSES | Medium | - [ ] |
| Day 11 | 15 | [Next Round (158A)](https://codeforces.com/problemset/problem/158/A) | Codeforces | Rating 800 | - [ ] |

---

[⬅️ Previous Topic: Binary Search](05-binary-search.md) | [Home](index.md) | [Next Topic: Binary Trees & BSTs ➡️](07-binary-trees.md)

