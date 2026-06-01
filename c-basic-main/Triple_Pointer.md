# Triple Pointer (`***`) in C (Embedded Systems)

## Table of Contents

1. [Introduction](#introduction)
2. [What is a Triple Pointer?](#what-is-a-triple-pointer)
3. [Pointer Levels Explained](#pointer-levels-explained)
4. [How Triple Pointer Works](#how-triple-pointer-works)
5. [Syntax](#syntax)
6. [Example](#example)
7. [Memory Representation](#memory-representation)
8. [Why we use Triple Pointer](#why-we-use-triple-pointer)
9. [Use cases in Embedded Systems](#use-cases-in-embedded-systems)
10. [Advantages and Disadvantages](#advantages-and-disadvantages)
11. [Interview Questions](#interview-questions)
12. [Conclusion](#conclusion)

---

# Introduction

Pointers are one of the most powerful concepts in C. When we go beyond simple pointers and double pointers, we reach **triple pointers**, which are rarely used but important in advanced embedded and system-level programming.

---

# What is a Triple Pointer?

### Definition

> A triple pointer is a pointer that stores the address of another pointer, which itself points to another pointer.

---

## Simple Meaning

```text id="tp1"
data → pointer → pointer → pointer
```

So:

```text id="tp2"
***p = value
```

---

# Pointer Levels Explained

| Level | Type       | Meaning                       |
| ----- | ---------- | ----------------------------- |
| 1     | int *p     | pointer to int                |
| 2     | int **pp   | pointer to pointer            |
| 3     | int ***ppp | pointer to pointer to pointer |

---

# How Triple Pointer Works

## Chain Concept

```text id="tp3"
value
  ↑
pointer
  ↑
pointer to pointer
  ↑
triple pointer
```

---

## Example Flow

```text id="tp4"
a = 10
p → &a
pp → &p
ppp → &pp
```

---

# Syntax

```c id="tp5"
int ***ptr;
```

---

# Example

```c id="tp6"
#include <stdio.h>

int main()
{
    int a = 10;

    int *p = &a;
    int **pp = &p;
    int ***ppp = &pp;

    printf("%d\n", ***ppp);

    return 0;
}
```

---

## Output

```text id="tp7"
10
```

---

# Memory Representation

```text id="tp8"
a   = 10
p   → address of a
pp  → address of p
ppp → address of pp
```

---

## Diagram

```text id="tp9"
ppp ───► pp ───► p ───► a
                          │
                          ▼
                         10
```

---

# Why we use Triple Pointer

Triple pointers are NOT commonly used like normal pointers.

They are used when:

✔ Multi-level dynamic memory is required
✔ Complex data structures are involved
✔ Passing 2D/3D dynamic arrays
✔ Modifying pointer inside functions

---

# Use cases in Embedded Systems

## 1. Dynamic 2D/3D Buffers

Used in:

* Image processing
* Sensor grids
* Radar systems

---

## 2. Function modifying pointer

```c id="tp10"
void allocate(int ***ptr)
```

Used to allocate memory inside function.

---

## 3. Multi-layer data structures

* Network stacks
* Protocol parsing
* Firmware configuration tables

---

## 4. Embedded Linux / Drivers

* Device tree structures
* Complex kernel objects

---

## Example (Memory Allocation)

```c id="tp11"
void allocate(int ***p)
{
    *p = malloc(sizeof(int *));
    **p = malloc(sizeof(int));
    ***p = 100;
}
```

---

# Advantages

✔ Enables multi-level dynamic memory handling
✔ Useful in complex systems
✔ Helps in modifying pointers in functions
✔ Useful in advanced embedded designs

---

# Disadvantages

✘ Very complex to understand
✘ Hard to debug
✘ High risk of segmentation faults
✘ Rarely needed in simple embedded projects
✘ Poor readability

---

# Interview Questions

## What is a triple pointer?

A triple pointer is a pointer that stores the address of another pointer, which itself points to another pointer.

---

## Why do we use triple pointers?

To handle multi-level dynamic memory and modify pointers inside functions.

---

## What is `***p`?

It is used to access the final value in a triple pointer chain.

---

## Is triple pointer commonly used?

No, it is rarely used in normal embedded systems but used in advanced memory handling.

---

## Difference between pointer, double pointer, and triple pointer?

| Type       | Meaning                       |
| ---------- | ----------------------------- |
| `int *p`   | pointer to int                |
| `int **p`  | pointer to pointer            |
| `int ***p` | pointer to pointer to pointer |

---

# Most Asked Interview Question

## Explain triple pointer with example.

A triple pointer is a pointer that stores the address of a double pointer. It is used to access or modify data through multiple levels of indirection. For example, `int ***ppp` allows access to an integer value using three dereference operations `***ppp`. It is mainly used in complex memory allocation and advanced embedded systems where multi-level data structures are required.

---

# Quick Revision

```text id="tp12"
Triple Pointer = Pointer to Pointer to Pointer

a → *p → **pp → ***ppp

Used in:
- Dynamic memory
- Complex structures
- Embedded Linux
```

---

# Conclusion

Triple pointers represent the highest level of pointer indirection in C. Although rarely used in simple embedded applications, they are important in advanced system programming, dynamic memory management, and complex data structures. Understanding them strengthens your grasp of memory and pointer concepts in C.
