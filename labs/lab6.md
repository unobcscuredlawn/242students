# Lab 6: Observing Heaps and Heapsort

**CPTR 242 — Sequential and Parallel Data Structures and Algorithms**
Walla Walla University

---

## Overview

In this lab you will **run complete programs and observe their behavior**. There is no coding required. Your job is to read the output carefully, record what you see, and reason about what it means.

By the end you should be able to:

- Trace a max-heap through a sequence of insertions and deletions using array-index arithmetic
- Explain why a complete binary tree maps perfectly onto an array with no pointers
- Compare the memory footprint and runtime overhead of an array heap versus a linked-node heap
- Describe the two phases of heapsort and explain why the heapify phase runs in O(n), not O(n log n)
- Compare heapsort's time and space usage against mergesort and std::sort, and reason about when each is preferable

---

## Background: Priority and Shape

A **heap** is a complete binary tree satisfying the **heap property**: in a max-heap, every node's key is greater than or equal to its children's keys. Because the tree is always *complete* — every level full except possibly the last, filled left-to-right — it maps perfectly onto a flat array with no wasted space and no pointers.

For a node at 1-based index `i`:

| Relationship | Index formula |
|---|---|
| Left child   | `2i`     |
| Right child  | `2i + 1` |
| Parent       | `i / 2`  |

This arithmetic is the entire "structure" of the tree. Inserting a key appends it to the array and **sifts up** (swaps with its parent while the heap property is violated). Removing the maximum swaps the root with the last element, shrinks the array by one, and **sifts down** (swaps with the larger child while the heap property is violated). Both operations touch at most O(log n) nodes — the height of the tree.

The central question: **does representing the same logical structure as a linked tree instead of an array change correctness, and what does it cost?**

---

## Setup

All programs are self-contained. Each generates its own test data and prints results directly to the terminal.

**Compiler command for all programs:**
```bash
g++ -O0 -o program program.cpp && ./program
```

> `-O0` disables compiler optimizations so you observe true algorithm behavior.

---

## Model 1: The Array Heap — Sift-Up on Insertion

Before measuring anything, you need to see exactly what happens inside the array on every insertion. This program builds a max-heap by inserting keys one at a time and prints the full array and a swap trace after each insertion. It then performs several extractions with sift-down traces.

### Program 1 — `array_heap.cpp`

```cpp

```

### Observation Table 1a — Array State After Each Insertion

Run the program and fill in the array contents (h[1] through h[8]) after each insertion.

| After inserting | h[1] | h[2] | h[3] | h[4] | h[5] | h[6] | h[7] | h[8] |
|---|---|---|---|---|---|---|---|---|
| 10 | 10 | | | | | | | |
| 4  | 10 | 4 | | | | | | |
| 15 | 15 | 4 | 10 | | | | | |
| 7  | 15 | 7 | 10 | 4 | | | | |
| 20 | 20 | 15 | 10 | 4 | 7 | | | |
| 3  | 20 | 15 | 10 | 4 | 7 | 3 | | |
| 18 | 20 | 15 | 18 | 4 | 7 | 3 | 10 | |
| 9  | 20 | 15 | 18 | 9 | 7 | 3 | 10 | 4 |

### Observation Table 1b — Extraction Sequence

| Extraction # | Value returned | Number of sift-down swaps |
|---|---|---|
| 1st | 20 | 2 |
| 2nd | 18 | 2 |
| 3rd | 15 | 1 |

---

### Critical Thinking Questions — Model 1

**Q1.** What value always sits at `h[1]` after every insertion, regardless of the order keys arrive? Why does the heap property guarantee this?

> Your answer: The largest value sits at h[1]. This is because every child is less than or equal to their parent so the one at the top is guaranteed to be the highest value, no matter the order

**Q2.** When 20 is inserted, it sifts all the way to the root. Trace the swaps by hand using the formula `parent = i / 2` starting from the index where 20 lands. Do the printed swaps match your calculation?

> Your answer: using [ 15 7 10 4 ] , swap h[5] with h[2] swap h[2] with h[1]

**Q3.** `extractMax` moves the *last* element in the array to the root before sifting down. Why the last element specifically? What structural property of a complete binary tree makes this the correct choice?

> Your answer: Removing the last element will keep the complete binary tree structure intact. The binary tree does not need the last node filled to be considered complete.

**Q4.** The heap has 8 nodes, so its height is ⌊log₂ 8⌋ = 3. From Table 1b, did any extraction require more than 3 sift-down swaps? Explain why this bound holds.

> Your answer: no more than 2 swaps were needed. 3 was the amount of levels that can be sifted through, so 2 is withing that boundry.

---

## Model 2: Array Heap vs. Linked-Node Heap — Memory and Speed

An array heap requires no pointers. A linked-node heap stores explicit `left`, `right`, and `parent` pointers in every node. This program builds both heaps from identical insertions and reports the memory consumed by each representation at several values of n, then times n insertions for both.

### Program 2 — `heap_comparison.cpp`

```cpp

```

### Observation Table 2a — Memory Usage

| n | Array heap (bytes) | Linked heap (bytes) | Ratio (linked / array) |
|---|---|---|---|
| 1,000 | 4096 | 32000 | 7.8x |
| 10,000 | 65536 | 320000 | 4.9x |
| 100,000 | 524288 | 3200000 | 6.1x |
| 500,000 | 2097152 | 16000000 | 7.6x |

### Observation Table 2b — Insertion Time

| n | Array heap (ms) | Linked heap (ms) |
|---|---|---|
| 1,000 | 2.966 | 0.239 |
| 10,000 | 0.786 | 2.620 |
| 100,000 | 7.661 | 31.353 |
| 500,000 | 38.629 | 197.562 |

---

### Critical Thinking Questions — Model 2

**Q5.** The program prints `sizeof(int)` and `sizeof(Node)` at startup. Using those two numbers, compute the *theoretical* ratio of linked-to-array memory and compare it to the ratios you measured in Table 2a. Does the ratio stay constant as n grows? What explains any discrepancy at small n?

> Your answer: the ratio is 8, meaning that the linked heap should use 8 times more memory than an array heap. as n grows the rate is consistent. there might be a discrepancy at small n because the capacity is larger than the num of elements

**Q6.** The linked heap's `nodeAt(idx)` navigates from the root to the target node by following pointers level by level. The array heap computes the parent in one step with `i / 2`. Both are O(log n) in theory, but with very different constants. Explain in terms of CPU cache behavior why pointer chasing is slower than index arithmetic, referencing what you observed about linked lists vs. arrays in Lab 4.

> Your answer: The CPU can't determine the next address until the current node is loaded causing frequent cache misses.

**Q7.** Given what Tables 2a and 2b show, propose at least one realistic scenario where you would choose the linked-node heap over the array heap despite its overhead.

> Your answer: it might be perfered to use a linked-node heap when the variables are large or unknown.

---

## Model 3: Heapsort — Heapify and Sort-Down

Heapsort sorts an array in two phases. **Phase 1 (heapify):** rearrange the whole array into a max-heap in O(n) time by applying sift-down from the bottom of the tree upward. **Phase 2 (sort-down):** repeatedly swap the root (the current maximum) to the end of the unsorted region and sift the new root down. This program prints the full array after every step of both phases so you can trace the sort exactly.

### Program 3 — `heapsort_trace.cpp`

```cpp

```

### Observation Table 3a — Array After Each Heapify Step

| siftDown from index | Array contents after step |
|---|---|
| (initial) | 5, 3, 8, 1, 9, 2, 7, 4, 6 |
| 3 | |
| 2 | |
| 1 | |
| 0 | |

### Observation Table 3b — Sort-Down Phase

Record the value moved into the sorted region and the remaining unsorted size after each step.

| Step | Value placed in sorted region | Unsorted size remaining |
|---|---|---|
| 1 | swap a[0]=9 with a[8]=1 | 8 |
| 2 | swap a[0]=8 with a[7]=4 | 7 |
| 3 | swap a[0]=7 with a[6]=1 | 6 |
| 4 | swap a[0]=6 with a[5]=2 | 5 |
| 5 | swap a[0]=5 with a[4]=2 | 4 |
| 6 | swap a[0]=4 with a[3]=1 | 3 |
| 7 | swap a[0]=3 with a[2]=2 | 2 |
| 8 | swap a[0]=2 with a[1]=1 | 0 |

---

### Critical Thinking Questions — Model 3

**Q8.** The heapify phase starts at index `n/2 - 1` and works *downward* to 0. Why not start at `n - 1`? What kind of nodes occupy indices `n/2` through `n - 1`, and why do they need no sifting?

> Your answer: because they are leaf nodes and sifting would not change anything.

**Q9.** After Phase 1, the maximum is at index 0. Phase 2 swaps it to the last position, temporarily placing a small value at the root and breaking the heap property. Why is it still correct to call `siftDown(a, 0, end)` with the sorted tail excluded? What guarantee does sift-down restore?

> Your answer: Its still correct because the element moved to the end is in the final sorted position. swaps can violate the heap property only at the root and still work because it restores max heap property for all but the last indices

---

## Model 4: Heapsort vs. Mergesort vs. std::sort — Time and Space

Heapsort's O(n log n) guarantee comes with a practical cost: non-sequential memory access causes frequent cache misses. Mergesort accesses memory sequentially but requires O(n) auxiliary space. `std::sort` uses a hybrid strategy that attempts to get the best of both worlds. This program benchmarks all three on random, already-sorted, and reverse-sorted inputs, and separately reports the auxiliary memory each algorithm requires.

### Program 4 — `sort_comparison.cpp`

```cpp

```

### Observation Table 4a — Sorting Time (ms)

**n = 10,000**

| Input order | heapsort (ms) | mergesort (ms) | std::sort (ms) |
|---|---|---|---|
| random  | 11.10 | 19.36 | 7.63 |
| sorted  | 10.11 | 13.64 | 4.75 |
| reverse | 9.36 | 13.74 | 3.85 |

**n = 100,000**

| Input order | heapsort (ms) | mergesort (ms) | std::sort (ms) |
|---|---|---|---|
| random  | 143.10 | 201.18 | 75.72 |
| sorted  | 102.58 | 114.48 | 44.83 |
| reverse | 87.43 | 85.22 | 24.92 |

**n = 500,000**

| Input order | heapsort (ms) | mergesort (ms) | std::sort (ms) |
|---|---|---|---|
| random  | 449.09 | 559.41 | 242.04 |
| sorted  | 324.12 | 339.18 | 124.33 |
| reverse | 278.31 | 292.06 | 88.92 |

### Observation Table 4b — Auxiliary Memory at n = 500,000

| Algorithm | Space complexity | Auxiliary bytes |
|---|---|---|
| heapsort  | O(1)     | 4 bytes |
| mergesort | O(n)     | 40000 bytes (39 KB)|
| std::sort | O(log n) | *(stack, not measured)* |

---

### Critical Thinking Questions — Model 4

**Q10.** Across all three input orders and all three values of n, which algorithm is consistently fastest? Which is consistently slowest? Does input order affect runtime meaningfully? Explain why or why not this would be the case, referencing heapsort's algorithm nature.

> Your answer: std::sort is the fastest and merge sort is the slowest. input affects runtime but not much, because heapsort runs in O(n log n)

**Q11.** `std::sort` uses **introsort**: it starts with quicksort but falls back to heapsort when recursion depth exceeds 2·⌊log₂ n⌋, preventing O(n²) worst-case behavior. Based on Table 4a, does `std::sort` appear to behave more like heapsort or something significantly faster? What does that tell you about how often the heapsort fallback is actually triggered in practice?

> Your answer: it behaves like a fast sorting algorithm especially for reversed input order. fallback is triggered not very often.

---
