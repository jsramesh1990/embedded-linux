# Little Endian

## Table of Contents

1. [Introduction](#introduction)
2. [What is Little Endian?](#what-is-little-endian)
3. [Why Do We Need Endianness?](#why-do-we-need-endianness)
4. [What is Endianness?](#what-is-endianness)
5. [Little Endian Memory Layout](#little-endian-memory-layout)
6. [How Little Endian Works](#how-little-endian-works)
7. [Byte Ordering Example](#byte-ordering-example)
8. [Little Endian vs Big Endian](#little-endian-vs-big-endian)
9. [How CPU Reads Data](#how-cpu-reads-data)
10. [What Happens When Code Hits It?](#what-happens-when-code-hits-it)
11. [Checking System Endianness](#checking-system-endianness)
12. [Little Endian in Embedded Systems](#little-endian-in-embedded-systems)
13. [Little Endian in Automotive Systems](#little-endian-in-automotive-systems)
14. [Network Byte Order](#network-byte-order)
15. [Advantages and Disadvantages](#advantages-and-disadvantages)
16. [Interview Questions](#interview-questions)
17. [Quick Revision](#quick-revision)
18. [Conclusion](#conclusion)

---

# Introduction

Computers store data in memory as bytes.

Example:

```c
int num = 0x12345678;
```

This number occupies:

```text
4 Bytes
```

Question:

```text
Which Byte Is Stored First?
```

The answer depends on:

```text
Endianness
```

---

# What is Little Endian?

## Definition

Little Endian is a memory storage format where:

> **The Least Significant Byte (LSB) is stored at the lowest memory address.**

---

Example:

```c
int num = 0x12345678;
```

Memory:

```text
Address      Value

1000         78
1001         56
1002         34
1003         12
```

---

Lowest Address:

```text
1000
```

contains:

```text
78
```

which is the Least Significant Byte.

Therefore:

```text
Little Endian
```

---

# Why Do We Need Endianness?

Processors store:

```text
16-bit
32-bit
64-bit
128-bit
```

values as multiple bytes.

Example:

```c
0x12345678
```

needs:

```text
12
34
56
78
```

The CPU must know:

```text
Which Byte Goes First?
```

Endianness defines this rule.

---

# What is Endianness?

Endianness determines:

> The order in which bytes are stored in memory.

---

Two major types:

| Type          | Lowest Address Contains |
| ------------- | ----------------------- |
| Little Endian | Least Significant Byte  |
| Big Endian    | Most Significant Byte   |

---

# Little Endian Memory Layout

Example:

```c
int value = 0x12345678;
```

Hex bytes:

```text
12 34 56 78
```

---

Memory:

```text
Address      Byte

1000         78
1001         56
1002         34
1003         12
```

---

Visualization:

```text
Value

0x12345678

MSB ---------------- LSB
12   34   56   78

Memory

1000 1001 1002 1003
 78   56   34   12
```

---

# How Little Endian Works

Example:

```c
int num = 0x12345678;
```

CPU stores:

```text
78 → Lowest Address

56 → Next Address

34 → Next Address

12 → Highest Address
```

---

Rule:

```text
LSB Stored First
```

---

# Byte Ordering Example

## 16-bit Number

```c
short num = 0x1234;
```

Memory:

```text
Address      Value

1000         34
1001         12
```

---

## 32-bit Number

```c
int num = 0x12345678;
```

Memory:

```text
1000         78
1001         56
1002         34
1003         12
```

---

## 64-bit Number

```c
long long num =
0x1122334455667788;
```

Memory:

```text
1000    88
1001    77
1002    66
1003    55
1004    44
1005    33
1006    22
1007    11
```

---

# Little Endian vs Big Endian

Example:

```c
0x12345678
```

| Address | Little Endian | Big Endian |
| ------- | ------------- | ---------- |
| 1000    | 78            | 12         |
| 1001    | 56            | 34         |
| 1002    | 34            | 56         |
| 1003    | 12            | 78         |

---

Visualization

### Little Endian

```text
1000 → 78
1001 → 56
1002 → 34
1003 → 12
```

---

### Big Endian

```text
1000 → 12
1001 → 34
1002 → 56
1003 → 78
```

---

# How CPU Reads Data

Example:

```c
int value = 0x12345678;
```

Memory:

```text
1000 78
1001 56
1002 34
1003 12
```

---

CPU fetches:

```text
78
56
34
12
```

and reconstructs:

```text
0x12345678
```

---

Program always sees:

```text
0x12345678
```

Endianness only affects memory storage.

---

# What Happens When Code Hits It?

Example:

```c
int num = 0x12345678;

char *ptr =
(char *)&num;
```

---

Memory:

```text
Address      Value

1000         78
1001         56
1002         34
1003         12
```

---

Execution:

```c
printf("%x", *ptr);
```

CPU reads:

```text
Address 1000
```

Value:

```text
78
```

Output:

```text
78
```

---

Execution:

```c
printf("%x", *(ptr+1));
```

Reads:

```text
56
```

---

### Interview Answer

**What happens when code accesses a multi-byte variable on a Little Endian system?**

The CPU stores the least significant byte at the lowest memory address. When code accesses the variable through a byte pointer, the first byte read is the least significant byte. Internally, the CPU reconstructs all bytes correctly, so the program still sees the original value.

---

# Checking System Endianness

## Method 1

```c
#include <stdio.h>

int main()
{
    int num = 1;

    char *ptr =
    (char *)&num;

    if(*ptr == 1)
        printf("Little Endian");
    else
        printf("Big Endian");
}
```

---

### Output

```text
Little Endian
```

if first byte contains:

```text
01
```

---

## Method 2

```c
union
{
    int num;
    char byte;
} test;

test.num = 1;

printf("%d",
       test.byte);
```

Output:

```text
1
```

means Little Endian.

---

# Little Endian in Embedded Systems

Most modern MCUs use Little Endian.

Examples:

| MCU          | Endianness    |
| ------------ | ------------- |
| ARM Cortex-M | Little Endian |
| STM32        | Little Endian |
| NXP LPC      | Little Endian |
| ESP32        | Little Endian |
| AVR          | Little Endian |

---

Why?

```text
Simpler Arithmetic Operations

Efficient Address Calculations

Industry Standard
```

---

# Little Endian in Automotive Systems

Common ECUs:

```text
Engine ECU
ABS ECU
ADAS ECU
Body Control Module
```

typically run on:

```text
ARM Processors
```

which are usually:

```text
Little Endian
```

---

Example:

CAN Signal:

```text
RPM = 0x1234
```

may need endian conversion before decoding.

---

# Network Byte Order

Important Interview Topic.

---

Network Protocols use:

```text
Big Endian
```

called:

```text
Network Byte Order
```

---

Example:

```text
TCP/IP
UDP
IPv4
IPv6
```

use Big Endian.

---

Conversion APIs:

```c
htons()
htonl()

ntohs()
ntohl()
```

---

Example:

```c
uint32_t ip =
htonl(0x12345678);
```

converts host format to network format.

---

# Advantages and Disadvantages

## Advantages

✔ Efficient Arithmetic Operations

✔ Common CPU Architecture Support

✔ Easy Access to Lower Bytes

✔ Widely Used in Embedded Systems

✔ Native on ARM and x86

---

## Disadvantages

✘ Network Protocol Conversion Needed

✘ Data Sharing Issues with Big Endian Systems

✘ Debugging Memory Dumps Can Be Confusing

---

# Interview Questions

## What is Little Endian?

Little Endian is a byte-ordering format in which the least significant byte is stored at the lowest memory address.

---

## What is Stored First in Little Endian?

```text
Least Significant Byte (LSB)
```

---

## What is the Opposite of Little Endian?

```text
Big Endian
```

---

## Is ARM Little Endian?

Most ARM processors operate in Little Endian mode.

---

## Is x86 Little Endian?

Yes.

Intel x86 and x64 are Little Endian.

---

## What Happens When Code Hits It?

The CPU stores multi-byte data with the least significant byte at the lowest address. When accessed byte-by-byte, the first byte read is the least significant byte.

---

## Why Do Network Protocols Use Big Endian?

To maintain a common standard called:

```text
Network Byte Order
```

across all systems.

---

## How Can You Check Endianness?

Using:

```c
char *ptr = (char *)&num;
```

and checking the first byte.

---

# Most Asked Interview Question

## Write a Program to Check Endianness.

```c
#include <stdio.h>

int main()
{
    int num = 1;

    char *ptr =
    (char *)&num;

    if(*ptr == 1)
        printf("Little Endian");
    else
        printf("Big Endian");

    return 0;
}
```

### Explanation

If the first memory byte contains `1`, the least significant byte is stored first, indicating a Little Endian system.

---

# Interview Answer (2-Minute Version)

Little Endian is a memory storage format where the least significant byte of a multi-byte value is stored at the lowest memory address. For example, the 32-bit value `0x12345678` is stored as `78 56 34 12` in memory. Most modern processors such as x86, x64, ARM Cortex-M, STM32, and ESP32 use Little Endian format. Endianness affects only the memory representation of data, not the logical value seen by the program. Network protocols use Big Endian format, so conversion functions such as `htonl()` and `ntohl()` are often required.

---

# Quick Revision

```text
Little Endian

Definition:
LSB Stored First

Example:
0x12345678

Memory:
78 56 34 12

Lowest Address:
Contains LSB

Used By:
✔ ARM
✔ STM32
✔ ESP32
✔ AVR
✔ x86

Network:
Uses Big Endian

Check:
char *ptr = (char *)&num;

*ptr == 1
→ Little Endian

Most Important:
Little Endian = Least Significant Byte
Stored At Lowest Memory Address
```

---

# Conclusion

Little Endian is the dominant byte-ordering scheme used in modern processors and embedded systems. It determines how multi-byte data is stored in memory, placing the least significant byte at the lowest address. Understanding Little Endian is essential for Embedded Systems, Automotive Software, Device Drivers, Networking, Firmware Development, Memory Debugging, and technical interviews, where questions about byte ordering and memory representation are very common.
