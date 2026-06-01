# `union` in C (Embedded Systems)

## Table of Contents

1. [Introduction](#introduction)
2. [What is union?](#what-is-union)
3. [Why do we use union?](#why-do-we-use-union)
4. [How union works in memory](#how-union-works-in-memory)
5. [Syntax](#syntax)
6. [Example](#example)
7. [Memory representation](#memory-representation)
8. [Difference between union and struct](#difference-between-union-and-struct)
9. [union use cases in embedded systems](#union-use-cases-in-embedded-systems)
10. [Advantages and disadvantages](#advantages-and-disadvantages)
11. [Interview questions](#interview-questions)
12. [Conclusion](#conclusion)

---

# Introduction

In embedded systems, memory is very limited.

To optimize memory usage, C provides a special data type called **union**.

---

# What is union?

### Definition

> A union is a user-defined data type in C where all members share the same memory location.

---

## Simple Meaning

Instead of separate memory for each variable, **all variables use the same memory space**.

---

## Example Idea

```text id="u1"
Only one value can be stored at a time
```

---

# Why do we use union?

Without union:

```text id="u2"
int (4 bytes)
float (4 bytes)
char (1 byte)
→ Total memory = 9 bytes
```

With union:

```text id="u3"
All share same memory
→ Memory = size of largest member
```

---

## Key Benefits

✔ Memory optimization
✔ Efficient for embedded systems
✔ Useful for hardware registers
✔ Useful in communication protocols

---

# How union works in memory

All members share the **same memory address**.

---

## Example

```c id="u4"
union Data {
    int i;
    float f;
    char c;
};
```

---

## Memory layout

```text id="u5"
Memory Block (shared)
----------------------
|   i / f / c        |
----------------------
```

---

## Important Rule

```text id="u6"
Only one member is valid at a time
```

---

# Syntax

```c id="u7"
union union_name {
    data_type member1;
    data_type member2;
};
```

---

# Example

```c id="u8"
union Data {
    int i;
    float f;
    char c;
};

union Data d;
```

---

## Usage

```c id="u9"
d.i = 10;
d.f = 3.14;   // overwrites i
```

---

# Memory representation

## Structure vs Union

### Struct

```text id="u10"
| int | float | char |
Separate memory
```

---

### Union

```text id="u11"
| shared memory |
Only one value at a time
```

---

## Size comparison

| Type   | Memory             |
| ------ | ------------------ |
| struct | sum of all members |
| union  | largest member     |

---

# Difference between union and struct

| Feature       | struct             | union               |
| ------------- | ------------------ | ------------------- |
| Memory        | Separate           | Shared              |
| Size          | Sum of all members | Largest member      |
| Value storage | All valid          | One at a time       |
| Use case      | Data grouping      | Memory optimization |

---

# union use cases in embedded systems

## 1. Hardware Register Mapping

```c id="u12"
union Register {
    unsigned int value;
    struct {
        unsigned int bit0 : 1;
        unsigned int bit1 : 1;
    } bits;
};
```

---

## 2. Communication Protocols

Used in:

* CAN
* UART
* TCP/IP packets

---

## 3. Sensor Data Interpretation

Same data in different formats:

```c id="u13"
union SensorData {
    int raw;
    float voltage;
};
```

---

## 4. Memory-Constrained Systems

* Microcontrollers
* IoT devices

---

## 5. Type Punning (Advanced use)

Access same memory in different formats.

---

# Advantages

✔ Memory efficient
✔ Useful in embedded systems
✔ Good for low-level programming
✔ Helps in hardware register access
✔ Reduces RAM usage

---

# Disadvantages

✘ Only one value valid at a time
✘ Easy to overwrite data accidentally
✘ Debugging can be confusing
✘ Not type-safe if misused

---

# Interview Questions

## What is union?

A union is a data type where all members share the same memory location.

---

## Difference between struct and union?

| struct                  | union               |
| ----------------------- | ------------------- |
| Multiple values at once | One value at a time |
| More memory             | Less memory         |

---

## Why do we use union in embedded systems?

To save memory and efficiently handle hardware registers and communication data.

---

## What happens when we write to one member?

It overwrites the previously stored value because all members share memory.

---

## Which is bigger: struct or union?

Struct is usually larger; union size is equal to the largest member.

---

# Most Asked Interview Question

## Explain union with example.

A union is a user-defined data type in C where all members share the same memory location. Only one member can store a value at a time. For example, `union Data { int i; float f; };` allows storing either an integer or float, but not both simultaneously. It is widely used in embedded systems for memory optimization, hardware register access, and communication protocols.

---

# Quick Revision

```text id="u14"
union = Shared Memory

Key Points:
- One memory location
- One value at a time
- Saves memory

Used in:
- Embedded systems
- Registers
- Communication protocols
```

---

# Conclusion

Union is a powerful feature in C used for memory optimization in embedded systems. It allows multiple variables to share the same memory location, making it highly efficient for low-level programming, hardware interaction, and communication systems. However, it must be used carefully to avoid data overwriting issues.
