# Array of Pointers in C (Embedded Systems)

## Table of Contents

1. [Introduction](#introduction)
2. [What is Array of Pointers?](#what-is-array-of-pointers)
3. [Why do we use it?](#why-do-we-use-it)
4. [Syntax](#syntax)
5. [How it works in memory](#how-it-works-in-memory)
6. [Examples](#examples)
7. [Array of Pointers vs Pointer to Array](#array-of-pointers-vs-pointer-to-array)
8. [Use cases in Embedded Systems](#use-cases-in-embedded-systems)
9. [Advantages and Disadvantages](#advantages-and-disadvantages)
10. [Interview Questions](#interview-questions)
11. [Conclusion](#conclusion)

---

# Introduction

In C programming, pointers can be organized in different ways. One important concept is the **array of pointers**, widely used in embedded systems for strings, buffers, and dynamic data handling.

---

# What is Array of Pointers?

### Definition

> An array of pointers is an array where each element stores the address of another variable.

---

## Simple Meaning

👉 Instead of storing values, it stores **addresses**.

---

# Why do we use it?

✔ Efficient memory usage
✔ Used for string handling
✔ Useful for multiple buffers
✔ Dynamic data handling
✔ Common in embedded drivers and OS

---

# Syntax

```c id="ap1"
data_type *array_name[size];
```

---

## Example

```c id="ap2"
int *ptr[5];
```

---

# How it works in memory

Each element of the array stores an address.

---

## Example

```c id="ap3"
int a = 10, b = 20, c = 30;
int *ptr[3] = {&a, &b, &c};
```

---

## Memory Representation

```text id="ap4"
ptr[0] → address of a → 10
ptr[1] → address of b → 20
ptr[2] → address of c → 30
```

---

# Examples

## 1. Simple Example

```c id="ap5"
#include <stdio.h>

int main()
{
    int a = 10, b = 20, c = 30;

    int *ptr[3] = {&a, &b, &c};

    printf("%d\n", *ptr[0]);
    printf("%d\n", *ptr[1]);
    printf("%d\n", *ptr[2]);

    return 0;
}
```

---

## Output

```text id="ap6"
10
20
30
```

---

## 2. String Example (Very Important)

```c id="ap7"
#include <stdio.h>

int main()
{
    char *names[] = {"ARM", "STM32", "ESP32"};

    printf("%s\n", names[0]);
    printf("%s\n", names[1]);
    printf("%s\n", names[2]);

    return 0;
}
```

---

# Array of Pointers vs Pointer to Array

| Feature  | Array of Pointers       | Pointer to Array       |
| -------- | ----------------------- | ---------------------- |
| Meaning  | Array storing addresses | Pointer to whole array |
| Syntax   | `int *p[5]`             | `int (*p)[5]`          |
| Memory   | Multiple pointers       | Single block           |
| Use case | Strings, dynamic data   | 2D arrays, buffers     |

---

# How it works in memory

## Array of pointers

```text id="ap8"
ptr[0] → addr1
ptr[1] → addr2
ptr[2] → addr3
```

Each pointer can point anywhere in memory.

---

# Use cases in Embedded Systems

## 1. String Tables

```c id="ap9"
char *cmds[] = {"START", "STOP", "RESET"};
```

Used in command interpreters.

---

## 2. Driver Buffers

* UART buffers
* SPI packets
* CAN frames

---

## 3. RTOS Systems

* Task parameter lists
* Message queues

---

## 4. Configuration Tables

* Sensor configuration
* Peripheral settings

---

## 5. Firmware Menus

* CLI systems
* Bootloader menus

---

# Advantages

✔ Flexible memory usage
✔ Efficient string handling
✔ Easy to manage multiple data sets
✔ Useful in embedded firmware
✔ Supports dynamic data

---

# Disadvantages

✘ Pointer complexity
✘ Risk of invalid memory access
✘ Harder debugging
✘ Requires careful memory management
✘ Not cache-friendly sometimes

---

# Interview Questions

## What is an array of pointers?

An array of pointers is an array where each element stores the address of another variable.

---

## Difference between array of pointers and pointer to array?

| Array of Pointers | Pointer to Array      |
| ----------------- | --------------------- |
| Stores addresses  | Points to whole array |
| `int *p[5]`       | `int (*p)[5]`         |

---

## Why do we use array of pointers?

To handle multiple independent memory locations efficiently, especially in strings and embedded systems.

---

## Where is it used in embedded systems?

* String handling
* Command processing
* Buffers
* RTOS communication

---

## Can array of pointers store different memory locations?

Yes, each pointer can point to different memory addresses.

---

# Most Asked Interview Question

## Explain array of pointers with example.

An array of pointers is an array where each element stores the address of another variable. For example, `int *ptr[3]` can store addresses of multiple integers. It is widely used in embedded systems for handling strings, buffers, and configuration tables. It provides flexibility because each pointer can point to different memory locations.

---

# Quick Revision

```text id="ap10"
Array of Pointers = Array storing addresses

Example:
int *p[3];

Used in:
- Strings
- Buffers
- Embedded systems
```

---

# Conclusion

Array of pointers is a powerful concept in C used for flexible memory handling. It is widely used in embedded systems for string processing, buffers, and configuration management. Understanding this concept is essential for firmware development and system-level programming interviews.
