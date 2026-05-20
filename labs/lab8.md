# CPTR 242 — Lab 8: Binary Search Trees & Tries

---

## Overview

This lab has four parts. You will observe a working BST — including all three removal cases — and measure a Trie and a string BST head-to-head on 50,000 words. No implementation is required; the goal is understanding through careful observation and analysis.

---

## Setup

Save the starter code below as `lab8.cpp` and compile with:

```bash
g++ -std=c++17 -O2 -o lab8 lab8.cpp && ./lab8
```

**Starter code — `lab8.cpp`:**

```cpp

---

## Part A — Insertion Order and BST Height

Run the program and observe the Part A output.

**[OBSERVE 1]**
Record both printed heights. The same 15 values are inserted in two different orders:

- **Sorted:** `1, 2, 3, 4, …, 15` height: 14
- **Shuffled:** `8, 4, 12, 2, 6, 10, 14, 1, 3, 5, 7, 9, 11, 13, 15` height: 3

What is the ratio of sorted height to shuffled height? What is log₂(15) rounded down?
the ratio is 14 : 3 or 4.667. the log is 3.907.

**[OBSERVE 2]**
Copy the in-order output from the shuffled BST. What do you notice? Is this output affected by which insertion order was used?

In-order output:  1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 
It appears to be the same output for both insertion orders.
---

## Part B — Trie Prefix Search

Run the program and observe the Part B output.

The dictionary contains: `apple`, `application`, `apply`, `apt`, `banana`, `band`, `bandana`, `bat`, `cat`, `card`, `care`, `car`, `carpet`.

**[OBSERVE 3]**
`"car"` is in the dictionary. `"carpet"` is also in the dictionary. Both queries give `hasPrefix=1`. But their `search` results differ. Explain exactly what the trie node reached after traversing `c → a → r` looks like — specifically its `isEnd` flag and its child pointers.

the trie node reached the end pointer on r* and sees the children d*, e*, and p.

**[OBSERVE 4]**
`"ap"` is **not** a word in the dictionary, but `hasPrefix` returns 1. Draw the trie path for `a → p` and identify where the traversal stops. Why does `trie_hasPrefix` return true while `trie_search` returns false?

**[OBSERVE 5]**
Look at the four words sharing the prefix `"app"`: `apple`, `application`, `apply`, `apt`. 

- How many trie nodes do all four share before their paths diverge?
- How many nodes would a string BST use to store the same four words?

**[OBSERVE 6]**
A BST storing strings compares full strings at each node: O(L) per comparison, O(log n) comparisons → O(L log n) per lookup. A trie does O(L) total regardless of n.

But there is a storage scenario where the BST uses *less* memory than the trie. Describe it precisely. (Hint: think about what the trie's sharing advantage requires.)

---

## Part C — BST Removal: All Three Cases

Run the program and observe the Part C output. The starting tree is:

```
        50
       /  \
     30    70
    /  \   / \
  20   40 60  80
        /
       35
```

**[OBSERVE 7]**
After every removal, is the BST property preserved? How does the in-order output confirm this without drawing the tree?

**[OBSERVE 8]**
`remove(30)` is a one-child case. After inserting `{50,30,70,20,40,60,80,35}` and removing 20, node 30 has only a right child (40), and 40 has a left child (35). 

Trace what `bst_remove` does step-by-step when called with key=30:
1. Which case fires?
2. What pointer is returned to replace node 30?
3. What happens to node 35?

**[OBSERVE 9]**
`remove(50)` is a two-children case. The code finds the **in-order successor** — the minimum of the right subtree.

After removing 20 and 30, what does the tree look like just before `remove(50)` is called? What is 50's in-order successor at that point?

**[OBSERVE 10]**
The code always uses the in-order **successor** (smallest in right subtree) for Case 3. Using the in-order **predecessor** (largest in left subtree) would also produce a valid BST.

Would choosing one over the other affect correctness? Would it affect the height or balance of the resulting tree over many insertions and deletions? Describe a sequence of operations where the choice would matter practically.

---

## Part D — Empirical Comparison: Trie vs. String BST

The program builds both structures from 50,000 distinct randomly generated words, then runs 500,000 lookups on each (10 passes × 50,000 words).

**[OBSERVE 11]**
Compute the speedup ratios: how many times faster is the Trie for exact search?

**[OBSERVE 12]**
The BST height is printed alongside log₂(50,000) ≈ 15. The actual BST height for random words should be around 35–40. Why is a randomly built BST not balanced, even with random data? What would need to be different about the insertion order to achieve height ≈ 15?



