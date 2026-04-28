# Lab 4: Observing Stacks and Queues

**CPTR 242 — Sequential and Parallel Data Structures and Algorithms**
Walla Walla University

---

## Overview

In this lab you will **run complete programs and observe their behavior**. There is no coding required. Your job is to read the output carefully, record what you see, and reason about what it means.

By the end you should be able to:

- Trace the state of a stack and queue through a sequence of push, pop, enqueue, and dequeue operations
- Explain how a linked-list stack and an array-based stack produce identical external behavior through different internal mechanisms
- Describe what goes wrong when a circular array queue wraps around and why the modulo trick fixes it
- Identify a real problem — bracket matching — that is naturally solved by a stack

---

## Background: One Interface, Many Implementations

A stack and a queue are both defined entirely by their ordering rules — LIFO and FIFO respectively. The ADT says nothing about internals. Today you will observe two implementations of each: one backed by a linked list and one backed by an array. The external behavior must be identical. The internal mechanics are completely different.

The central question: **do the two implementations always agree — and where do their differences become visible?**

---

## Setup

All programs are self-contained. Each generates its own test data and prints results directly to the terminal.

**Compiler command for all programs:**
```bash
g++ -O0 -o program program.cpp && ./program
```

> `-O0` disables compiler optimizations so you observe true algorithm behavior.

---

## Model 1: Stack State Tracing

Before measuring anything, you need to be able to predict and verify what a stack contains after a sequence of operations. This program executes a fixed sequence of push and pop operations on both a linked-list stack and an array-based stack, printing the full state after every step.

### Program 1 — `stack_trace.cpp`



### Observation Table 1 — Stack State After Each Operation

Fill in both columns after running the program. Record the contents from top to bottom.

| Operation | Linked stack (top → bottom) | Array stack (top → bottom) |
|---|---|---|
| Initial | *(empty)* | *(empty)* |
| push(10) | [top→] 10 (size=1) | [top→] 10 (size=1)|
| push(20) | [top→] 20 10 (size=2) | [top→] 20 10 (size=2) |
| push(30) | [top→] 30 20 10 (size=2) | [top→] 30 20 10 (size=2) |
| peek | [top→] 30 20 10 (size=3) | [top→] 30 20 10 (size=3) |
| pop | [top→] 20 10 (size=2) | [top→] 20 10 (size=2) |
| push(40) | [top→] 40 20 10 (size=3) | [top→] 40 20 10 (size=3) |
| push(50) | [top→] 50 40 20 10 (size=4) | [top→] 50 40 20 10 (size=4) |
| pop | [top→] 40 20 10 (size=3) | [top→] 40 20 10 (size=3) |
| pop | [top→] 20 10 (size=2) | [top→] 20 10 (size=2) |
| pop | [top→] 10 (size=1) | [top→] 10 (size=1) |

---

### Critical Thinking Questions — Model 1

**Q1.** After every operation, the linked stack and array stack print the same contents in the same order. They are implemented completely differently internally. What does this tell you about the relationship between an ADT and its implementation?

> Your answer:For basic implementation, the ADT will stay the same. This is because the same operations can be used.

**Q2.** The linked stack prints from `head` forward. The array stack prints from `top` downward (index `top` to 0). Both show elements in top-to-bottom order. What does the array `top` index physically represent — and how does it change on push versus pop?

> Your answer: The array top is an integer that marks a position. with push, top increments and with pop, top decrements.  

**Q3.** The program does `push(10), push(20), push(30), pop, push(40), push(50), pop, pop, pop`. Before running the program, predict the return value of each `pop` in order. Check your prediction against the output.

> Your prediction (three values, in order): 30, 50, 40, 20.

**Q4.** The array stack has a compile-time capacity of 16. The linked stack has no capacity limit. If you pushed 17 items onto the array stack what would happen? What would happen to the linked stack? What is the practical consequence of choosing each implementation for an application where maximum stack depth is unknown?

> Your answer: Pushing 17 items would cause a stack overflow in an array but not a linked stack. Array stacks would be worse for large amounts of items. Linked stacks might run out of memory causing problems.

---

## Model 2: A Stack Solving a Real Problem

Bracket matching is a classic stack application — and a good test of whether you understand LIFO behavior at a deeper level than just "last in, first out." This program runs a bracket checker on several strings and prints its internal state after each character is processed.

### Program 2 — `bracket_matching.cpp`

### Observation Table 2 — Bracket Matching Results

Record the final RESULT printed for each input string:

| Input | Result | Reason (in your words) |
|---|---|---|
| `({[]})` | Valid | The algorithm runs into no errors |
| `({[}])` | Invalid | The algorithm tries to pop `}` but it doesn't match so it throws an exception |
| `((())` | Invalid | The algorithm is searching for `)` but does not find one and throws an exception |
| `hello(world[!])` | Valid | The algorithm checks for `({[]})` and ignores all other char |
| *(empty string)* | Valid | The algorithm checks to see if the string is empty and says it is valid |

---

### Critical Thinking Questions — Model 2

**Q5.** For input `({[]})`, trace the stack contents at each step in the table below before checking your answer against the program output:

| Char | Action | Stack after (top → bottom) |
|---|---|---|
| `(` | push | `(`|
| `{` | push | `( {`|
| `[` | push | `( { [`|
| `]` | pop/match | `( {`|
| `}` | pop/match | `(`|
| `)` | pop/match | ---|

**Q6.** For input `({[}])` the program reports INVALID. At which character does it fail, and what exactly is wrong? Why is a queue not a useful data structure for bracket matching — what ordering property of the stack makes the matching work correctly?

> Your answer: it fails at `}` because it doesn't match with `[`. The program needs to access the top of the stack easily.

**Q7.** The input `hello(world[!])` contains letters, `!`, and brackets mixed together. The program still reports VALID. What does the program do with non-bracket characters? Why is it correct to ignore them?

> Your answer: The program ignores or skips non bracket characters. It is good to ignore them because it is efficient.

**Q8.** The empty string input reports VALID. Is this the right answer? Justify why an empty string is considered balanced.

> Your answer: The program automaticly assigns empty strings with valid. This is balanced because there a matching bracket, in order, for every bracket in the string, which is none.

---

## Model 3: Queue State Tracing and the Circular Array

A queue has two active ends — front and back — which makes its state harder to visualize than a stack. This program traces both a linked-list queue and a circular-array queue through the same sequence of operations, printing front, back, and contents after each step.

### Program 3 — `queue_trace.cpp`



### Observation Table 3a — Queue Contents After Each Operation

Record front-to-back contents and dequeue return values:

| Operation | Queue contents (front → back) | Dequeue returned |
|---|---|---|
| Initial | *(empty)* | — |
| enqueue(10) | 10 | — |
| enqueue(20) | 10 20 | — |
| enqueue(30) | 10 20 30 | — |
| dequeue | 20 30 | 10 |
| enqueue(40) | 20 30 40 | — |
| dequeue | 30 40 | 20 |
| dequeue | 40 | 30 |
| enqueue(50) | 40 50 | — |
| enqueue(60) | 40 50 60 | — |
| enqueue(70) | 40 50 60 70 | — |
| dequeue | 50 60 70 | 40 |
| dequeue | 60 70 | 50 |
| dequeue | 70 | 60 |
| dequeue | empty | 70 |

### Observation Table 3b — Circular Array Internal State

Record the `front` index, `back` index, and `cnt` printed for the array queue after each operation:

| Operation | `front` index | `back` index | `cnt` |
|---|---|---|---|
| Initial | 0 | 0 | 0 |
| enqueue(10) | 0 | 1 | 1 |
| enqueue(20) | 0 | 2 | 2 |
| enqueue(30) | 0 | 3 | 3 |
| dequeue | 1 | 3 | 2 |
| enqueue(40) | 1 | 4 | 3 |
| dequeue | 2 | 4 | 2 |
| dequeue | 3 | 4 | 1 |
| enqueue(50) | 3 | 5 | 2 |
| enqueue(60) | 3 | 6 | 3 |
| enqueue(70) | 3 | 7 | 4 |
| dequeue | 4 | 7 | 3 |
| dequeue | 5 | 7 | 2 |
| dequeue | 6 | 7 | 1 |
| dequeue | 7 | 7 | 0 |

---

### Critical Thinking Questions — Model 3

**Q9.** Look at Table 3a. After every operation, do the linked queue and array queue always contain the same elements in the same order? What does this confirm about the two implementations?

> Your answer: The elements are the same and in the same order. For the two implementations, they do not change in the elements or the order.

**Q10.** Look at Table 3b. After the three initial enqueues and one dequeue, the `front` index is 1 (not 0). The element at array index 0 is logically gone — but the dequeue operation never moved any data. What did it do instead? What does this tell you about how "removal" works in a circular array queue?

> Your answer: Instead of deleting front and moving all the data, dequeue moves the front marker to the next index then marks the deleted index as empty.

**Q11.** The circular array queue has a `cnt` field tracking the number of elements. An alternative design uses only `front` and `back` indices and considers the queue empty when `front == back`. What ambiguity arises in that design? Why is the `cnt` field (or an `isFull` flag) needed?

> Your answer: front and back are equal in two situations, one with the list being full and one with the the array being first initialized. To avoid ambiaguity, a cnt feild is used.

---

## Model 4: Performance — Stack and Queue Operations at Scale

All four operations — push, pop, enqueue, dequeue — are O(1). But the *constant* hidden inside O(1) differs between a linked implementation and an array implementation. This program measures how long it takes to push/enqueue and pop/dequeue a large number of elements on both implementations.

### Program 4 — `stack_queue_perf.cpp`



### Observation Table 4a — Stack Performance (μs)

| n | Linked push+pop | Vector push+pop | Ratio |
|---|---|---|---|
| 100,000 | 5413 | 6639 | 0.82x |
| 500,000 | 25806 | 32971 | 0.78x |
| 1,000,000 | 52446 | 74730 | 0.70x |
| 5,000,000 | 262014 | 295076 | 0.89x |

### Observation Table 4b — Queue Performance (μs)

| n | Linked enq+deq | Array enq+deq | Ratio |
|---|---|---|---|
| 100,000 | 3375 | 16147 | 0.21x |
| 500,000 | 17540 | 23065 | 0.76x |
| 1,000,000 | 35385 | 32281 | 1.10x |
| 5,000,000 | 180349 | 106546 | 1.69x |

---

### Critical Thinking Questions — Model 4

**Q12.** Both linked and array implementations are O(1) per operation, so doubling n should double total time for both. When n grows from 100,000 to 5,000,000 (50×), by approximately what factor does each implementation's time grow? Is this consistent with O(n) total work?

> Your answer: When n grows by 50x the implementation grows by 10x.

**Q13.** The array/vector implementation is faster than the linked implementation by a consistent ratio. Both do the same logical work. What accounts for the difference? Name at least one concrete reason.

> Your answer: arrays are more cache friendly and the operations are more relient on cache.

**Q14.** The vector stack calls `reserve(n)` before pushing. Remove the `reserve` call mentally and predict whether the timing would change significantly. What does `reserve` prevent, and what cost does it avoid?

> Your answer: reserve prevents repeated heap allocation and it avoids cache disruption.
 
**Q15.** Based on your observations across all four models, state a concrete rule for when you would prefer a linked-list implementation of a stack or queue over an array-based one. Your rule should reference at least one specific situation where the linked implementation's properties are genuinely advantageous.

> Your answer: linked lists are good for garunteed O(1) removals or insertions without resizing or knowing the size of the input.

---


## Extra Credit Questions


**A1.** The call stack in your computer is literally a stack. Every function call pushes a frame; every return pops one. What happens when a recursive function has no base case? Connect what you observed about the array stack's overflow behavior to the term "stack overflow."

**A2.** The linked queue's `dequeue` function has a single special-case line: `if (!head) tail = nullptr`. This line is easy to forget. Given what you traced in Model 3, describe exactly what breaks in a subsequent `enqueue` if this line is missing.

**A3.** `std::stack` and `std::queue` in C++ wrap `std::deque` internally rather than a raw array or linked list. Based on what you observed in Model 4, why might `std::deque` be a better default backing container than either of your two implementations?

---

*Next lab: hash tables — where we abandon sequential structure entirely in favor of O(1) lookup by key.*
