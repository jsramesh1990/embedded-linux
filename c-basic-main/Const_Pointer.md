# `const` Pointer in C (Embedded Systems)

## Table of Contents

1. [Introduction](#introduction)
2. [What is a const pointer?](#what-is-a-const-pointer)
3. [Types of const pointers](#types-of-const-pointers)
4. [Const pointer vs Pointer to const](#const-pointer-vs-pointer-to-const)
5. [Syntax](#syntax)
6. [Examples](#examples)
7. [Memory behavior](#memory-behavior)
8. [Use cases in Embedded Systems](#use-cases-in-embedded-systems)
9. [Advantages and Disadvantages](#advantages-and-disadvantages)
10. [Interview Questions](#interview-questions)
11. [Conclusion](#conclusion)

---

# Introduction

In embedded systems, we often need **controlled access to memory** to avoid accidental modification of critical data like registers, configuration tables, or calibration constants.

That is where `const` pointers are used.

---

# What is a const pointer?

### Definition

> A const pointer is a pointer whose behavior is restricted using the `const` keyword either on the pointer itself, the data it points to, or both.

---

## Important Idea

`const` can be applied in **two ways**:

* Make pointer constant
* Make data constant
* Or both

---

# Types of const pointers

```text id="cp1"
1. Pointer to constant data
2. Constant pointer
3. Constant pointer to constant data
```

---

# 1. Pointer to Const Data

## Syntax

```c id="cp2"
const int *ptr;
```

OR

```c id="cp3"
int const *ptr;
```

---

## Meaning

👉 You cannot change the value using pointer
👉 Pointer can point to different addresses

---

## Example

```c id="cp4"
int a = 10, b = 20;
const int *ptr = &a;

ptr = &b;   // Allowed
//*ptr = 30; // Not allowed
```

---

# 2. Const Pointer

## Syntax

```c id="cp5"
int *const ptr;
```

---

## Meaning

👉 Pointer address cannot change
👉 Value can be modified

---

## Example

```c id="cp6"
int a = 10, b = 20;
int *const ptr = &a;

*ptr = 15;  // Allowed
//ptr = &b; // Not allowed
```

---

# 3. Const Pointer to Const Data

## Syntax

```c id="cp7"
const int *const ptr;
```

---

## Meaning

👉 Pointer cannot change address
👉 Value cannot be modified

---

## Example

```c id="cp8"
int a = 10;
const int *const ptr = &a;

// *ptr = 20; // Not allowed
// ptr = &b;  // Not allowed
```

---

# Const pointer vs Pointer to const

| Type                   | Pointer change | Value change |
| ---------------------- | -------------- | ------------ |
| `const int *ptr`       | Yes            | No           |
| `int *const ptr`       | No             | Yes          |
| `const int *const ptr` | No             | No           |

---

# Memory Behavior

## Example

```text id="cp9"
a = 10
ptr → address of a
```

---

## Behavior rules

```text id="cp10"
const data → protected value
const pointer → fixed address
```

---

# Use cases in Embedded Systems

## 1. Hardware Registers

```c id="cp11"
const int *GPIO_REG = (int *)0x40021000;
```

Prevent accidental modification rules.

---

## 2. Calibration Data

```c id="cp12"
const int calibration_value = 100;
```

Stored in flash memory.

---

## 3. Driver APIs

```c id="cp13"
void read_data(const int *buffer);
```

Ensures input buffer is not modified.

---

## 4. RTOS Systems

* Task parameters protection
* Shared resource safety

---

## 5. Communication Buffers

* UART RX buffers
* CAN message frames

---

# Advantages

✔ Prevent accidental data modification
✔ Improves code safety
✔ Better debugging clarity
✔ Essential in embedded drivers
✔ Helps in hardware protection

---

# Disadvantages

✘ Can be confusing initially
✘ Improper use may restrict flexibility
✘ Compiler errors if misused
✘ Requires careful design understanding

---

# Interview Questions

## What is a const pointer?

A const pointer is a pointer where either the pointer, the data, or both are made constant using the `const` keyword.

---

## Difference between pointer to const and const pointer?

| Type           | Pointer        | Value       |
| -------------- | -------------- | ----------- |
| `const int *p` | Change allowed | Not allowed |
| `int *const p` | Not allowed    | Allowed     |

---

## Why do we use const pointers in embedded systems?

To protect critical memory like hardware registers, configuration data, and buffers from accidental modification.

---

## Can we change a const pointer address?

No, if the pointer itself is const.

---

## Can we change value through const pointer?

Only if data is not const.

---

# Most Asked Interview Question

## Explain const pointer with example.

A const pointer in C is a pointer where either the pointer itself, the data it points to, or both are made constant using the `const` keyword. For example, `const int *ptr` prevents modifying the value, while `int *const ptr` prevents changing the address. It is widely used in embedded systems for protecting hardware registers, calibration data, and ensuring safe driver APIs.

---

# Quick Revision

```text id="cp14"
const pointer types:

1. const int *p   → value fixed
2. int *const p   → address fixed
3. const int *const p → both fixed

Used in:
- Embedded systems
- Drivers
- RTOS
```

---

# Conclusion

Const pointers are essential in C programming for safety, reliability, and controlled memory access. In embedded systems, they are widely used in drivers, hardware registers, and communication buffers to prevent accidental modifications and ensure system stability.
