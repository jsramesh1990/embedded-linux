# Recursion (C Programming / Embedded Systems)

## Table of Contents

1. [Introduction](#introduction)
2. [What is Recursion?](#what-is-recursion)
3. [How Recursion Works](#how-recursion-works)
4. [Base Case and Recursive Case](#base-case-and-recursive-case)
5. [Call Stack Behavior](#call-stack-behavior)
6. [Types of Recursion](#types-of-recursion)
7. [Example Programs](#example-programs)
8. [Memory Representation](#memory-representation)
9. [Advantages](#advantages)
10. [Disadvantages](#disadvantages)
11. [Where Recursion is Used in Embedded Systems](#where-recursion-is-used-in-embedded-systems)
12. [Interview Questions](#interview-questions)
13. [Conclusion](#conclusion)

---

# Introduction

Recursion is an important concept in C programming and embedded systems, often asked in interviews.

It is widely used in:

* Tree structures
* File systems
* Algorithm design
* Embedded protocol parsing

---

# What is Recursion?

### Definition

> Recursion is a programming technique where a function calls itself to solve a smaller instance of the same problem.

---

## Simple Meaning

A function solving a problem by calling itself again and again until a condition is met.

---

## Example

```c id="rec1"
void fun()
{
    fun(); // function calling itself
}
```

---

# How Recursion Works

## Basic Flow

```text id="rec2"
Function Call
     ↓
Calls Itself
     ↓
Calls Itself Again
     ↓
Stops at Base Case
     ↓
Returns Back
```

---

## Example (Concept)

```text id="rec3"
main()
 ↓
func(3)
 ↓
func(2)
 ↓
func(1)
 ↓
STOP (base case)
```

---

# Base Case and Recursive Case

## 1. Base Case (VERY IMPORTANT)

Stops recursion.

Without base case → infinite loop → crash.

---

## Example

```c id="rec4"
if(n == 0)
    return;
```

---

## 2. Recursive Case

Function calls itself.

```c id="rec5"
func(n-1);
```

---

## Full Example

```c id="rec6"
void func(int n)
{
    if(n == 0)
        return;

    func(n-1);
}
```

---

# Call Stack Behavior

Recursion uses the **stack memory**.

---

## Example: func(3)

```text id="rec7"
func(3)
func(2)
func(1)
func(0) → stop
```

---

## Stack Representation

```text id="rec8"
TOP → func(0)
       func(1)
       func(2)
BOTTOM func(3)
```

Then:

```text id="rec9"
Unwinding starts
```

---

# Types of Recursion

```text id="rec10"
Recursion
│
├── Direct Recursion
├── Indirect Recursion
├── Tail Recursion
└── Non-Tail Recursion
```

---

## 1. Direct Recursion

Function calls itself.

```c id="rec11"
void f()
{
    f();
}
```

---

## 2. Indirect Recursion

Function A calls B, B calls A.

```c id="rec12"
void A() { B(); }
void B() { A(); }
```

---

## 3. Tail Recursion

Recursive call is last statement.

```c id="rec13"
void f(int n)
{
    if(n == 0) return;
    f(n-1);  // last call
}
```

---

## 4. Non-Tail Recursion

More work after recursive call.

```c id="rec14"
void f(int n)
{
    if(n == 0) return;
    f(n-1);
    printf("%d", n);
}
```

---

# Example Programs

## 1. Factorial

```c id="rec15"
int fact(int n)
{
    if(n == 0)
        return 1;

    return n * fact(n-1);
}
```

---

## Flow

```text id="rec16"
5 → 4 → 3 → 2 → 1 → 0
```

---

## 2. Fibonacci Series

```c id="rec17"
int fib(int n)
{
    if(n <= 1)
        return n;

    return fib(n-1) + fib(n-2);
}
```

---

## 3. Sum of Numbers

```c id="rec18"
int sum(int n)
{
    if(n == 0)
        return 0;

    return n + sum(n-1);
}
```

---

# Memory Representation

## Recursion uses Stack Memory

```text id="rec19"
Stack Frame per call:
- Parameters
- Local variables
- Return address
```

---

## Example

```text id="rec20"
sum(3)
 → sum(2)
   → sum(1)
     → sum(0)
```

---

## Stack Growth

```text id="rec21"
Bottom → sum(3)
         sum(2)
         sum(1)
Top    → sum(0)
```

---

# Advantages

✔ Simplifies complex problems
✔ Cleaner code
✔ Useful for tree/graph problems
✔ Matches mathematical definitions
✔ Easy problem decomposition

---

# Disadvantages

✘ High memory usage (stack overflow risk)
✘ Slower than loops
✘ Hard to debug
✘ Infinite recursion risk
✘ Not suitable for real-time constraints sometimes

---

# Where Recursion is Used in Embedded Systems

Even though recursion is limited in embedded systems due to memory constraints, it is still used in:

---

## 1. Tree Data Structures

* Device file systems
* Device tree parsing (Linux)

---

## 2. Protocol Parsing

* Nested packet parsing (TCP/IP stacks)

---

## 3. Firmware Bootloaders

* Directory traversal
* Configuration parsing

---

## 4. Embedded Linux

* Kernel tree structures
* Driver model traversal

---

## 5. Algorithms

* Searching
* Sorting (QuickSort, MergeSort)

---

## Example (Embedded Linux)

```text id="rec22"
Device Tree
    ├── CPU Node
    ├── Memory Node
    └── Peripheral Nodes
```

Traversal is often recursive.

---

# Interview Questions

## What is Recursion?

Recursion is a technique where a function calls itself to solve smaller instances of the same problem.

---

## What is a Base Case?

A condition that stops recursion.

---

## What happens if base case is missing?

Infinite recursion → stack overflow → program crash.

---

## Difference between recursion and iteration?

| Recursion    | Iteration        |
| ------------ | ---------------- |
| Uses stack   | Uses loops       |
| Slower       | Faster           |
| Memory heavy | Memory efficient |

---

## What is stack overflow?

When recursive calls exceed stack memory limit.

---

## What is tail recursion?

When recursive call is the last operation in the function.

---

## Why recursion is less preferred in embedded systems?

Due to limited stack memory and real-time constraints.

---

# Most Asked Interview Question

## Explain Recursion with Example.

Recursion is a programming technique where a function calls itself until a base condition is met. It breaks a problem into smaller sub-problems. For example, factorial calculation uses recursion where fact(n) = n * fact(n-1), and the process continues until n becomes 0. Each function call is stored in stack memory, and once the base case is reached, results are returned back step by step. Recursion is useful in tree traversal, parsing, and algorithm design but must be used carefully in embedded systems due to memory limitations.

---

# Quick Revision

```text
Recursion = Function calling itself

Needs:
- Base Case (stop condition)
- Recursive Case (self call)

Risk:
- Stack Overflow

Used in:
- Trees
- Algorithms
- Embedded Linux structures
```

---

# Conclusion

Recursion is a powerful programming technique used to solve complex problems by breaking them into smaller sub-problems. While it simplifies code and is widely used in algorithms and system-level programming, it must be used carefully in embedded systems due to memory and performance constraints. Understanding recursion is essential for interviews and advanced embedded software development.
