# Pointer to Array in C (Embedded Systems)

## Table of Contents

1. [Introduction](#introduction)
2. [What is a Pointer to Array?](#what-is-a-pointer-to-array)
3. [Why do we use it?](#why-do-we-use-it)
4. [Syntax](#syntax)
5. [Pointer to Array vs Array of Pointers](#pointer-to-array-vs-array-of-pointers)
6. [Examples](#examples)
7. [Memory Representation](#memory-representation)
8. [Pointer Arithmetic with Arrays](#pointer-arithmetic-with-arrays)
9. [Use cases in Embedded Systems](#use-cases-in-embedded-systems)
10. [Advantages and Disadvantages](#advantages-and-disadvantages)
11. [Interview Questions](#interview-questions)
12. [Conclusion](#conclusion)

---

# Introduction

In C programming, arrays and pointers are closely related. A **pointer to an array** is used when we want to point to the *entire array block*, not just the first element.

This concept is very useful in embedded systems where we handle buffers, sensor data blocks, and memory-mapped data.

---

# What is a Pointer to Array?

### Definition

> A pointer to an array is a pointer that points to the whole array instead of pointing to a single element.

---

## Simple Meaning

👉 It points to a complete array memory block.

---

# Why do we use it?

✔ Handle multi-dimensional arrays
✔ Pass arrays efficiently to functions
✔ Work with buffers in drivers
✔ Useful in embedded communication stacks
✔ Better control over memory layout

---

# Syntax

```c id="pa1"
data_type (*pointer_name)[size];
```

---

## Example

```c id="pa2"
int arr[5] = {1, 2, 3, 4, 5};
int (*ptr)[5] = &arr;
```

---

# Pointer to Array vs Array of Pointers

| Concept           | Meaning                         |
| ----------------- | ------------------------------- |
| Pointer to Array  | Points to whole array           |
| Array of Pointers | Array storing multiple pointers |

---

## Array of pointers

```c id="pa3"
int *ptr[5];
```

---

## Pointer to array

```c id="pa4"
int (*ptr)[5];
```

---

# Examples

## 1. Basic Example

```c id="pa5"
#include <stdio.h>

int main()
{
    int arr[3] = {10, 20, 30};
    int (*ptr)[3] = &arr;

    printf("%d\n", (*ptr)[1]);

    return 0;
}
```

---

## Output

```text id="pa6"
20
```

---

## Explanation

```text id="pa7"
*ptr → whole array
(*ptr)[1] → second element
```

---

# Memory Representation

```text id="pa8"
arr → [10][20][30]
ptr → address of entire array
```

---

## Diagram

```text id="pa9"
ptr ─────► [ 10 | 20 | 30 ]
```

---

# Pointer Arithmetic with Arrays

## Important Rule

Pointer to array moves in **blocks**, not single elements.

---

## Example

```c id="pa10"
int arr[3][4];
int (*ptr)[4] = arr;
```

---

## ptr++

Moves to next row, not next element.

---

# Use cases in Embedded Systems

## 1. 2D Buffers (very important)

* Image processing
* Sensor grids
* Display buffers

---

## 2. DMA Transfers

* Block memory transfers
* Peripheral buffers

---

## 3. Communication Protocols

* CAN frames
* Ethernet packets

---

## 4. Register Arrays

* Hardware register blocks

---

## 5. RTOS Data Structures

* Task parameter blocks

---

# Advantages

✔ Better handling of multi-dimensional arrays
✔ Efficient memory access
✔ Useful in low-level programming
✔ Supports block-based operations
✔ Improves code clarity in drivers

---

# Disadvantages

✘ Confusing syntax
✘ Easy to misinterpret
✘ Risk of segmentation faults
✘ Hard for beginners
✘ Requires careful memory understanding

---

# Interview Questions

## What is a pointer to array?

A pointer to an array is a pointer that points to the entire array instead of a single element.

---

## Difference between pointer to array and array of pointers?

| Pointer to Array      | Array of Pointers        |
| --------------------- | ------------------------ |
| Points to whole array | Stores multiple pointers |
| `int (*p)[5]`         | `int *p[5]`              |

---

## Why do we use pointer to array?

To efficiently handle multi-dimensional arrays and memory blocks in embedded systems.

---

## What happens when pointer to array is incremented?

It moves to the next array block, not a single element.

---

## Is pointer to array same as pointer?

No, it points to entire array structure, not individual elements.

---

# Most Asked Interview Question

## Explain pointer to array with example.

A pointer to array is a pointer that stores the address of an entire array instead of a single element. It is defined using syntax like `int (*ptr)[5]`. It is commonly used in embedded systems for handling multi-dimensional arrays, buffers, and hardware data blocks. Unlike normal pointers, incrementing a pointer to array moves it by the size of the entire array.

---

# Quick Revision

```text id="pa11"
Pointer to Array = Points to whole array

Syntax:
int (*ptr)[size];

Use:
- 2D arrays
- Buffers
- Embedded systems
```

---

# Conclusion

Pointer to array is an advanced pointer concept in C used to handle entire arrays as a single unit. It is widely used in embedded systems for efficient memory handling, especially in drivers, communication protocols, and multi-dimensional data processing. Understanding this concept is important for system-level programming and interviews.
