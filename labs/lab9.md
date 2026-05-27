# CPTR 242 — Lab 11: Balanced Trees and B-Trees

**Estimated time:** 75–90 minutes
**Submission:** Upload your completed answers to all observation and reflection questions.

---

## Overview

This lab has five parts. You will observe a plain BST degenerating under sorted input, watch the four AVL rotation cases fire in isolation, trace balance factors through a growing AVL tree, step through 2-3-4 tree insertions and splits, and then measure the performance consequences of all these design choices on real timing data. No implementation is required — compile, run, observe, and reason.

---

## Setup

Save the starter code below as `lab11.cpp` and compile with:

```bash
g++ -std=c++17 -O2 -o lab11 lab11.cpp && ./lab11
```

**Starter code — `lab11.cpp`:**

```cpp

```

---

## Part A — Sorted Insertion: BST vs AVL

Run the program and observe the Part A output.

**[OBSERVE 1]**
Record the three printed values: BST height, AVL height, and ⌊log₂(7)⌋. What is the ratio of BST height to AVL height? What would the ratio be for 1,000 elements inserted in sorted order?

BST height: 6
AVL height: 2
log2(7) = 2
BST to AVL is O(n)/log2(n)
ratio at 1000 elements is about 100/1
---

## Part B — The Four Rotation Cases

Run the program and observe the Part B output.

**[OBSERVE 2]**
All four cases — LL, RR, LR, RL — produce `root=20, left=10, right=30`. Why does every three-node rotation case end up with the median value at the root?

It is the only way a balanced tree with three nodes can be sorted.
---

## Part C — Balance Factor Trace

Run the program and observe the Part C output.

**[OBSERVE 3]**
Copy the full output for Part C. For the first seven insertions (`50` through `80`), the root never changes and the root balance factor stays in {−1, 0}. Why? What property of the insertion sequence ensures no rotation is needed during these seven inserts?


```cpp
cout << "  left child of root: " << tr->left->val
     << "  bf=" << abf(tr->left) << "\n";
cout << "  left-left child:    " << tr->left->left->val
     << "  bf=" << abf(tr->left->left) << "\n";
```

Record what you see. Which node had balance factor ±2 before `afix` corrected it?

---

## Part D — 2-3-4 Tree Insertions

Run the program and observe the Part D output.

**[OBSERVE 5]**
The sequence `10, 20, 30` is inserted into a single root node. When `40` is inserted, the root was a 4-node (`[10|20|30]`) and had to be split before the insert could proceed.

Trace what happens during the split:
1. What value moves **up** to become the new root?
2. What two nodes remain as children?
3. Where does `40` finally land?

**[OBSERVE 6]**
The tree is built from a purely sorted sequence `10, 20, ..., 80` — exactly the input that destroys a plain BST. Yet the printed height after all 8 insertions is **1**.

Why doesn't sorted input degenerate a 2-3-4 tree the way it degenerates a plain BST? What structural property prevents it?

---

## Part E — Timing: BST vs AVL vs `std::map`

Run the program and observe the Part E output.

**[OBSERVE 7]**
The plain BST sorted insert takes roughly **1,000× longer** than the AVL insert for the same input. Both are O(h) per operation, but their heights differ by a factor of ~1,400 (height 19,999 vs 14). Yet the timing ratio is only ~1,000, not ~1,400.

What would explain the timing ratio being smaller than the height ratio? (Hint: think about what operations besides comparisons are happening, and how cache behavior changes with tree depth.)

**[OBSERVE 8]**
For **random** input, all three structures are within a small constant factor of each other in insert time, and the BST height (~32) is within 2× of the AVL height (~16).

What does this tell you about when the plain BST is an acceptable choice? What kind of guarantee does it still fail to provide even for random input?

**[OBSERVE 9]**
The sorted-built search for `std::map` is slower than AVL in this run, even though both are O(log n) with similar heights. One reason: `std::map` uses a Red-Black tree, which is slightly less balanced than AVL (height ≤ 2 log n vs ≤ 1.44 log n).

But there is another reason related to **memory layout**. A `std::map` node is heap-allocated separately and contains extra pointers (parent, color flag). An `ANode` in this lab also heap-allocates, but what does the `std::map` node contain that the lab's `ANode` does not, and how might that affect cache performance?

**[OBSERVE 10]**
You have now seen three balanced structures — AVL tree, Red-Black tree (`std::map`), and 2-3-4 tree — and timed two of them.

Suppose you are building a database index that:
- Is stored on disk (each node access is a disk read costing ~5ms)
- Has 10 million keys
- Needs to support search and range queries

Which of the three structures from this lab would you choose, and why? Be specific about height, disk reads per operation, and the cost of the alternative.
