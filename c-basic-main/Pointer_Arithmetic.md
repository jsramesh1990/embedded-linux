# Pointer Arithmetic in C (Embedded Systems)

## Table of Contents

1. [Introduction](#introduction)
2. [What is Pointer Arithmetic?](#what-is-pointer-arithmetic)
3. [Why do we use it?](#why-do-we-use-it)
4. [How Pointer Arithmetic works](#how-pointer-arithmetic-works)
5. [Types of Pointer Operations](#types-of-pointer-operations)
6. [Examples](#examples)
7. [Memory behavior](#memory-behavior)
8. [Pointer arithmetic with arrays](#pointer-arithmetic-with-arrays)
9. [Pointer arithmetic with different data types](#pointer-arithmetic-with-different-data-types)
10. [Pointer arithmetic rules](#pointer-arithmetic-rules)
11. [Use cases in Embedded Systems](#use-cases-in-embedded-systems)
12. [Advantages and Disadvantages](#advantages-and-disadvantages)
13. [Interview Questions](#interview-questions)
14. [Conclusion](#conclusion)

---

# Introduction

Pointer arithmetic is one of the most important concepts in C, especially in **embedded systems programming**, where direct memory access is required.

It allows us to navigate memory efficiently using pointers.

---

# What is Pointer Arithmetic?

### Definition

> Pointer arithmetic is the set of operations performed on pointers to move between memory locations.

---

## Simple Meaning

Instead of working with raw memory addresses, we move pointers forward or backward.

---

# Why do we use it?

✔ Efficient memory access
✔ Faster array traversal
✔ Hardware register access
✔ Buffer handling in drivers
✔ Used in embedded firmware

---

# How Pointer Arithmetic works

Pointers do NOT move byte-by-byte always.

They move based on **data type size**.

---

## Example

```c id="p1"
int *ptr;
ptr++;
```

If `int` = 4 bytes:

```text id="p2"
Address increases by 4 bytes
```

---

# Types of Pointer Operations

```text id="p3"
Pointer Arithmetic
│
├── Increment (++)
├── Decrement (--)
├── Addition (+ n)
├── Subtraction (- n)
├── Pointer difference
```

---

# Examples

## 1. Increment

```c id="p4"
int a[3] = {10, 20, 30};
int *p = a;

p++;
```

Moves to next element.

---

## Memory

```text id="p5"
p → a[0]
p++ → a[1]
```

---

## 2. Decrement

```c id="p6"
p--;
```

Moves backward.

---

## 3. Addition

```c id="p7"
p = p + 2;
```

Skips 2 elements.

---

## 4. Subtraction

```c id="p8"
p = p - 1;
```

Moves one element back.

---

## 5. Pointer Difference

```c id="p9"
int diff = p2 - p1;
```

Gives number of elements between pointers.

---

# Memory Behavior

## Example

```c id="p10"
int arr[] = {10, 20, 30, 40};
int *p = arr;
```

---

## Memory Layout

```text id="p11"
Address   Value
1000      10
1004      20
1008      30
1012      40
```

---

## Pointer movement

```text id="p12"
p → 1000
p++ → 1004
p++ → 1008
```

---

# Pointer arithmetic with arrays

Arrays and pointers are closely related.

---

## Example

```c id="p13"
arr[i] == *(arr + i)
```

---

## Access using pointer

```c id="p14"
*(p + 2)
```

Means:

```text id="p15"
3rd element of array
```

---

# Pointer arithmetic with different data types

Pointer movement depends on data type size.

---

## Example

### int pointer

```c id="p16"
int *p;
p++;
```

Moves 4 bytes (typical).

---

### char pointer

```c id="p17"
char *c;
c++;
```

Moves 1 byte.

---

### float pointer

```c id="p18"
float *f;
f++;
```

Moves 4 bytes.

---

## Key Rule

```text id="p19"
Pointer + 1 = sizeof(data type)
```

---

# Pointer arithmetic rules

✔ Only addition/subtraction allowed
✔ Multiplication/division NOT allowed
✔ Only valid within same array
✔ Cannot go outside allocated memory

---

## Invalid operations

```c id="p20"
// NOT allowed
p * 2;
p / 2;
```

---

# Use cases in Embedded Systems

## 1. Buffer Handling

```c id="p21"
uint8_t *buffer;
buffer++;
```

Used in UART, SPI, CAN drivers.

---

## 2. Memory-Mapped Registers

```c id="p22"
#define REG (*(volatile int*)0x40021000)
REG++;
```

---

## 3. Sensor Data Processing

```c id="p23"
ptr = sensor_buffer;
ptr++;
```

---

## 4. DMA Operations

Pointer increments through memory blocks.

---

## 5. Protocol Stacks

Used in TCP/IP packet parsing.

---

# Advantages

✔ Fast memory access
✔ Efficient array handling
✔ Low-level hardware control
✔ Essential in embedded systems
✔ Reduces CPU overhead

---

# Disadvantages

✘ Risk of memory corruption
✘ Hard to debug
✘ Unsafe if misused
✘ Can lead to buffer overflow
✘ Requires deep understanding

---

# Interview Questions

## What is pointer arithmetic?

Pointer arithmetic is performing operations like increment and decrement on pointers to move through memory locations.

---

## Why do we use pointer arithmetic?

To efficiently access arrays, buffers, and hardware memory in embedded systems.

---

## What happens when we increment a pointer?

It moves to the next memory location based on the size of the data type.

---

## Can we multiply or divide pointers?

No, pointer arithmetic only supports addition and subtraction.

---

## Difference between array indexing and pointer arithmetic?

| Array  | Pointer       |
| ------ | ------------- |
| arr[i] | *(p + i)      |
| Easier | More flexible |

---

## Why is pointer arithmetic important in embedded systems?

Because embedded systems require direct and efficient memory and hardware register access.

---

# Most Asked Interview Question

## Explain pointer arithmetic with example.

Pointer arithmetic is the method of performing operations on pointers to navigate memory locations. When a pointer is incremented, it moves to the next memory location based on the size of its data type. For example, if an integer pointer points to address 1000 and is incremented, it moves to 1004 (assuming 4-byte integer). It is widely used in embedded systems for buffer handling, hardware registers, and communication protocols.

---

# Quick Revision

```text id="p24"
Pointer Arithmetic = Memory Navigation

Operations:
++  → next element
--  → previous element
+ n → skip elements

Rule:
Move depends on data type size
```

---

# Conclusion

Pointer arithmetic is a powerful concept in C that allows efficient memory manipulation. It is heavily used in embedded systems for handling arrays, buffers, hardware registers, and communication protocols. However, it must be used carefully to avoid memory corruption and system crashes.
