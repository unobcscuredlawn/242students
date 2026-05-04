# Lab 5: Observing Hash Tables

**CPTR 242 — Sequential and Parallel Data Structures and Algorithms**
Walla Walla University

---

## Overview

In this lab you will **run complete programs and observe their behavior**. There is no coding required. Your job is to read the output carefully, record what you see, and reason about what it means.

By the end you should be able to:

- Trace a hash table through a sequence of insertions and look up the expected slot for any key
- Explain how chaining handles collisions using a linked list per bucket
- Describe how linear probing finds an open slot and why deletion requires tombstones
- Compare hash table performance against a sorted array and reason about when O(1) average breaks down
- Recognize the load-factor threshold that triggers resizing and understand why rehashing is necessary

---

## Background: From Search to O(1) Lookup

Every data structure you have studied so far — arrays, linked lists, trees — locates a value by *comparing* keys. Binary search on a sorted array takes O(log n); a balanced BST takes O(log n). A hash table sidesteps comparison entirely: it computes an array index from the key and accesses that slot directly. When the hash function distributes keys uniformly and the table is not too full, every lookup, insertion, and deletion takes O(1) average time.

The central question: **where does that O(1) come from, and when does it break?**

---

## Setup

All programs are self-contained. Each generates its own test data and prints results directly to the terminal.

**Compiler command for all programs:**
```bash
g++ -O0 -o program program.cpp && ./program
```

> `-O0` disables compiler optimizations so you observe true algorithm behavior.

---

## Model 1: Hash Function Behavior and Slot Distribution

Before examining a full hash table, you need to understand what a hash function actually does to a set of keys. This program hashes a fixed list of strings using a polynomial rolling hash and prints the raw hash value, the slot index (`hash % TABLE_SIZE`), and the final slot assignment for each key.

### Program 1 — `hash_distribution.cpp`

```cpp

```

### Observation Table 1a — Hash Slot Assignments

Fill in the slot each key maps to:

| Key | Slot (0–10) |
|---|---|
| alice | 10 |
| bob | 4 |
| carol | 6 |
| dave | 3 |
| eve | 5 |
| frank | 5 |
| grace | 4 |
| heidi | 4 |
| ivan | 10 |
| judy | 3 |

### Observation Table 1b — Slot Occupancy

| Slot | Number of keys assigned |
|---|---|
| 0 | 0 |
| 1 | 0 |
| 2 | 0 |
| 3 | 2 |
| 4 | 3 |
| 5 | 2 |
| 6 | 1 |
| 7 | 0 |
| 8 | 0 |
| 9 | 0 |
| 10 | 2 |

---

### Critical Thinking Questions — Model 1

**Q1.** How many slots are empty after inserting 10 keys into an 11-slot table? How many collisions occurred? Does the distribution look roughly uniform, or are keys clumped heavily in a few slots?

> Your answer:There are 6 slots empty. The distribution is clumped around 4.

**Q2.** The hash function multiplies by 31 before adding each character. What would happen if you simply summed the character values without multiplying? Give an example of two different strings that would collide under the summing approach but likely not under the polynomial approach.

> Your answer: When summing without multiplying, the hash is the same no matter the order. An example could be "ab" and "ba" or "rats" and "star".

**Q3.** TABLE_SIZE is chosen to be prime (11). Suppose you changed it to 10 (not prime). Would you expect more or fewer collisions? What property of prime-sized tables makes them distribute keys more evenly?

> Your answer: 10 has small factors so patterns in hash values will cause clustering, and more collisions. A prime number avoids this issue

**Q4.** The load factor after 10 insertions is approximately 0.91. This is high. What are the consequences of a high load factor for (a) a chaining table and (b) an open-addressing table?

> Your answer: a high load will cause chaining to have longer chains and more time spent searching. for open adderssing, there is more clustering and longer probes.

---

## Model 2: Chaining — Insertion, Lookup, and Collision Handling

Chaining resolves collisions by storing a linked list at each bucket. This program builds a chaining hash table from a sequence of key–value pairs, prints the internal state of every bucket after each insertion, and then performs several lookups showing the probe path through the chain.

### Program 2 — `chaining.cpp`

```cpp

```

### Observation Table 2a — Table State After Each Insertion

For each insertion, record which slot the key lands in and whether a collision occurred (another key was already in that slot):

| Key inserted | Slot | Collision? (Y/N) | Chain length after insertion |
|---|---|---|---|
| alice | 6 | N | 1 |
| bob | 4 | N | 1 |
| carol | 2 | N | 1 |
| dave | 3 | N | 1 |
| eve | 6 | Y | 2 |
| frank | 1 | N | 1 |
| grace | 1 | Y | 2 |
| heidi | 6 | Y | 3 |

### Observation Table 2b — Lookup Probe Counts

| Key searched | Slot | Probes needed | Found? |
|---|---|---|---|
| alice | 6 | 3 | Found |
| carol | 2 | 1 | Found |
| grace | 1 | 1 | Found |
| zara | 0 | 0 | Not found |

---

### Critical Thinking Questions — Model 2

**Q5.** When two keys land in the same slot, the new key is **prepended** to the chain (inserted at the front), not appended to the back. Does this affect correctness? Does it affect performance? Would you ever prefer to append instead?

> Your answer: It does not affect correctness but it does affect performance. appending is situational and is better for older elements.

**Q6.** The `search("zara")` call walks an entire chain and finds nothing. How many nodes did it examine? In terms of load factor λ, what is the expected number of probes for an unsuccessful search in a chaining table?

> Your answer: it examines 3 nodes. The expected number of probes is 0.73

**Q7.** After all 8 insertions into a 7-slot table the load factor exceeds 1.0. Chaining still works. Open addressing would not — why not? What invariant does open addressing require that chaining does not?

> Your answer: open addressing does not work because there is no empty slot. chaining will grow the chain in the same situation.

**Q8.** Look at the final table printout. Some slots have chains of length 2 or more; others are empty. Even with a good hash function, perfect uniformity is not guaranteed. What is the theoretical expected maximum chain length for n keys in a table of size n, under uniform hashing?

> Your answer: under uniform hashing, a table size of n would have a theoretical maximum of log n


---

## Model 3: Linear Probing — Insertion, Clustering, and Tombstones

Linear probing stores everything in the array itself — no linked lists, no extra allocation. On collision it steps forward by 1 until an empty slot is found. This program inserts a sequence of keys, prints the full array after each insertion (showing which slots are occupied and where keys "landed" after probing), then demonstrates the tombstone deletion problem.

### Program 3 — `linear_probing.cpp`

```cpp

```

### Observation Table 3a — Slot Placement After Each Insertion

Record where each key ended up (its final slot index) and how many extra probes were needed beyond the home slot:

| Key | Home slot (`hash % 11`) | Final slot | Extra probes | Caused by cluster? |
|---|---|---|---|---|
| alice | 10 | 10 | 0 | N |
| bob | 4 | 4 | 0 | N |
| carol | 6 | 6 | 0 | N |
| dave | 3 | 3 | 0 | N |
| eve | 5 | 5 | 0 | N |
| frank | 5 | 7 | 2 | Y |
| grace | 4 | 8 | 4 | Y |

### Observation Table 3b — Search Probe Paths After Tombstone Insertion

After `bob` is deleted (tombstone), record the slots visited for each search:

| Key searched | Slots visited (in order) | Probes | Result |
|---|---|---|---|
| carol (before delete) | 6 | 1 | carol |
| carol (after delete) | 6 | 1 | carol |
| alice | 10 | 1 | alice |
| grace | 4 5 6 7 8 | 5 | grace |
| zara | 7 8 9 | 2 | Empty not found |

---

### Critical Thinking Questions — Model 3

**Q9.** Look at the extra-probe column in Table 3a. Several keys required probing past their home slot. This is **primary clustering** — runs of occupied slots that grow longer over time. Describe in your own words why a new key hashing *anywhere into* an existing cluster makes the cluster grow longer.

> Your answer: a new key hashing into a cluster will first try to go to its home slot, then it will probe the next slot forward until it reaches an empty slot which will be at the end of the cluster, making it longer.

**Q10.** After `bob` is deleted, the search for `carol` must cross the tombstone. What would happen if the tombstone were replaced with an EMPTY marker instead — would the search for `carol` succeed or fail? Explain why.

> Your answer: a tombstone will have the search continue but a slot marked empty will stop the search.

**Q11.** Tombstones accumulate over time. A slot marked DELETED counts as "occupied" for searches (you must keep probing past it) but as "available" for insertions (you can place a new key there). What happens to average probe length as the fraction of tombstone slots grows? How does rehashing fix this?

> Your answer: as the fraction of tombstone slots grows, the avg probe length grows. rehashing makes a new table and reinserts only the active keys.

**Q12.** Linear probing requires λ < 1 (the table can never be completely full). Chaining from Model 2 allowed λ > 1. Why does open addressing impose this strict limit, while chaining does not?

> Your answer: open addressing must have all keys inside the table array itself and only one key per slot, it also probes until there is an empty slot, so all together, it means that there must be one empty slot. Chaining avoids this by having the slots be chains that can be added to.

---

## Model 4: Collision Strategy Comparison and Load Factor Effects

Linear probing degrades quickly as the table fills. Double hashing avoids clustering by using a second hash function to compute the step size. This program inserts the same keys into three tables — chaining, linear probing, and double hashing — and reports the average probe count for both successful and unsuccessful searches at several load factors.

### Program 4 — `collision_comparison.cpp`

```cpp

```

### Observation Table 4a — Measured Average Probe Counts

| Load factor (λ) | Chaining | Linear probing | Double hashing |
|---|---|---|---|
| ~0.10 | 1 | 1 | 1 |
| ~0.25 | 1 | 1 | 1 |
| ~0.50 | 1.140 | 3.520 | 1.280 |
| ~0.69 | 1.271 | 5.957 | 1.814 |
| ~0.89 | 1.356 | 8.756 | 2.322 |

### Observation Table 4b — Theoretical Linear vs. Double Hashing Predictions

| Load factor (λ) | Linear (theory) | Double (theory) |
|---|---|---|
| 0.10 | 1.056 | 1.054 |
| 0.25 | 1.167 | 1.151 |
| 0.50 | 1.5 | 1.386 |
| 0.69 | 2.113 | 1.697 |
| 0.89 | 5.045 | 2.480 |

---

### Critical Thinking Questions — Model 4

**Q13.** At λ = 0.50 (half full), linear probing already takes noticeably more probes than double hashing. At λ ≈ 0.89 the gap widens dramatically. In your own words, explain *why* double hashing avoids the primary clustering that hurts linear probing.

> Your answer: double hashing will take different paths through the table jumping over clusters and preventing merging.

**Q14.** Chaining's probe count grows slowly and almost linearly with λ — and it still works above λ = 1.0. Open-addressing schemes blow up as λ → 1. From the table, at what load factor does linear probing exceed 3 probes on average? What does that tell you about a safe upper bound for λ in a linearly-probed table?

> Your answer: avg probes exceed 3 at λ 0.7. meaning a safe upper bound for λ is 0.7.

**Q15.** Both theoretical formulas (linear and double hashing) are derived assuming **uniform random hashing** — every key equally likely in any slot. Your measured values are based on a specific deterministic hash function on synthetic keys. Are the measured values close to the theoretical predictions? What might cause discrepancies?

> Your answer: they are close for lower load factor. decrepencies happen because of primary clustering.

---

## Extra Credit Questions

**A1.** The polynomial rolling hash uses multiplier 31 (`h = h * 31 + c`). Java's `String.hashCode()` also uses 31. One reason is that `31 * x == (x << 5) - x`, which some compilers optimize to a shift and subtract instead of a multiply. Why would this matter for hash table performance, and under what conditions would it matter most?

**A2.** Linear probing has better **cache performance** than double hashing even though double hashing produces fewer probes. Explain why fewer probes does not automatically mean faster in practice, referencing what you observed about array versus linked-list performance in Lab 4.

**A3.** Cryptographic hash functions like SHA-256 are deliberately slow and designed to be one-way. Hash table hash functions are designed to be fast and do not need to be one-way. Describe one concrete security attack that becomes possible if you use a fast non-cryptographic hash (like the polynomial hash from this lab) to hash passwords in a login system, and explain how bcrypt or Argon2 defeats it.

---

*Next lab: trees — where we trade O(1) average for O(log n) guaranteed and gain the ability to iterate keys in sorted order.*
