# Pointer to Const in C (Embedded Systems)

## Table of Contents

1. [Introduction](#introduction)
2. [What is Pointer to Const?](#what-is-pointer-to-const)
3. [Why do we use it?](#why-do-we-use-it)
4. [Syntax](#syntax)
5. [How it works](#how-it-works)
6. [Examples](#examples)
7. [Memory behavior](#memory-behavior)
8. [Difference from const pointer](#difference-from-const-pointer)
9. [Use cases in Embedded Systems](#use-cases-in-embedded-systems)
10. [Advantages and Disadvantages](#advantages-and-disadvantages)
11. [Interview Questions](#interview-questions)
12. [Conclusion](#conclusion)

---

# Introduction

In embedded systems, data safety is very important. We often want to **allow reading data but prevent modification**.

This is achieved using a **pointer to const**.

---

# What is Pointer to Const?

### Definition

> A pointer to const is a pointer that points to constant data, meaning the value cannot be modified through that pointer.

---

## Simple Meaning

👉 You can change the pointer address
👉 But you cannot change the value it points to

---

# Why do we use it?

✔ Protect critical data
✔ Improve safety in drivers
✔ Prevent accidental writes
✔ Used in APIs and firmware design
✔ Ensures read-only access

---

# Syntax

```c id="pc1"
const int *ptr;
```

OR

```c id="pc2"
int const *ptr;
```

---

# How it works

## Rule

```text id="pc3"
Value is read-only through pointer
Pointer can move freely
```

---

## Example behavior

```c id="pc4"
int a = 10;
int b = 20;

const int *ptr = &a;

ptr = &b;   // Allowed
//*ptr = 30; // Not allowed
```

---

# Examples

## 1. Basic Example

```c id="pc5"
#include <stdio.h>

int main()
{
    int a = 10;
    const int *ptr = &a;

    printf("%d\n", *ptr);

    return 0;
}
```

---

## 2. Changing pointer (allowed)

```c id="pc6"
int a = 10, b = 20;

const int *ptr = &a;
ptr = &b;   // OK
```

---

## 3. Changing value (NOT allowed)

```c id="pc7"
*ptr = 30;  // Error
```

---

# Memory Behavior

```text id="pc8"
a = 10
b = 20

ptr → can move between addresses

BUT values are protected
```

---

# Difference from const pointer

| Feature        | Pointer to Const | Const Pointer  |
| -------------- | ---------------- | -------------- |
| Syntax         | `const int *p`   | `int *const p` |
| Value change   | Not allowed      | Allowed        |
| Pointer change | Allowed          | Not allowed    |

---

## Easy trick

```text id="pc9"
const before * → value fixed
const after * → pointer fixed
```

---

# Use cases in Embedded Systems

## 1. Read-only Hardware Registers

```c id="pc10"
const int *GPIO_STATUS = (int *)0x40021010;
```

---

## 2. Driver APIs

```c id="pc11"
void read_sensor(const int *data);
```

Prevents modification of input data.

---

## 3. Calibration Data

```c id="pc12"
const int *calib_data;
```

Stored in flash memory.

---

## 4. Communication Buffers

* UART RX buffers
* CAN frames
* SPI receive data

---

## 5. RTOS Systems

* Task input parameters
* Shared read-only resources

---

# Advantages

✔ Protects data integrity
✔ Prevents accidental modification
✔ Improves code safety
✔ Useful in drivers and APIs
✔ Better system reliability

---

# Disadvantages

✘ Cannot modify data through pointer
✘ Requires careful API design
✘ Can cause confusion for beginners
✘ Compiler restrictions may appear

---

# Interview Questions

## What is pointer to const?

A pointer to const is a pointer that points to constant data, meaning the value cannot be modified through the pointer.

---

## Can we change pointer in pointer to const?

Yes, pointer can point to different addresses.

---

## Can we change value using pointer to const?

No, value is read-only through that pointer.

---

## Difference between pointer to const and const pointer?

| Pointer to Const   | Const Pointer         |
| ------------------ | --------------------- |
| Value fixed        | Pointer fixed         |
| Address can change | Address cannot change |

---

## Why use pointer to const in embedded systems?

To protect critical data like hardware registers, buffers, and calibration values from accidental modification.

---

# Most Asked Interview Question

## Explain pointer to const with example.

A pointer to const in C is a pointer that points to constant data. This means the value pointed to by the pointer cannot be modified, but the pointer itself can point to different memory locations. For example, `const int *ptr` allows changing the pointer address but prevents modifying the value. It is widely used in embedded systems for safe data handling in drivers, hardware registers, and communication buffers.

---

# Quick Revision

```text id="pc13"
Pointer to Const = Read-only data via pointer

Syntax:
const int *ptr;

Rules:
- Value cannot change
- Pointer can change

Used in:
- Embedded systems
- Drivers
- APIs
```

---

# Conclusion

Pointer to const is an important safety feature in C programming that ensures data integrity by preventing modification through pointers. It is widely used in embedded systems, especially in hardware interfaces, drivers, and communication protocols where read-only access is required.
