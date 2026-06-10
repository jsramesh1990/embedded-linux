# Data Section (.data)

## Table of Contents

1. [Introduction](#introduction)
2. [What is the Data Section?](#what-is-the-data-section)
3. [Why Do We Need the Data Section?](#why-do-we-need-the-data-section)
4. [Program Memory Layout](#program-memory-layout)
5. [What is Stored in the Data Section?](#what-is-stored-in-the-data-section)
6. [Types of Variables Stored in .data](#types-of-variables-stored-in-data)
7. [How the Data Section Works](#how-the-data-section-works)
8. [Data Section in ELF Files](#data-section-in-elf-files)
9. [Data Section vs Other Memory Sections](#data-section-vs-other-memory-sections)
10. [Memory Protection](#memory-protection)
11. [What Happens When Code Hits It?](#what-happens-when-code-hits-it)
12. [Real-Time Examples](#real-time-examples)
13. [Data Section in Embedded Systems](#data-section-in-embedded-systems)
14. [Advantages and Disadvantages](#advantages-and-disadvantages)
15. [Interview Questions](#interview-questions)
16. [Quick Revision](#quick-revision)
17. [Conclusion](#conclusion)

---

# Introduction

A compiled C/C++ program is divided into multiple memory sections:

```text
.text
.data
.bss
.heap
.stack
.rodata
```

The **Data Section (.data)** stores:

> **Global and Static variables that are initialized before program execution.**

---

# What is the Data Section?

## Definition

The `.data` section is a memory region that stores:

* Global variables with initial values
* Static variables with initial values
* Variables that can be modified during runtime

---

Example:

```c
int speed = 100;
```

Stored in:

```text
.data
```

because it is initialized.

---

Another example:

```c
static int count = 10;
```

Also stored in:

```text
.data
```

---

# Why Do We Need the Data Section?

Programs often require variables whose values must exist before execution begins.

Example:

```c
int max_speed = 180;
```

The operating system or startup code must know:

```text
Variable Name
Initial Value
Memory Location
```

The `.data` section stores this information.

---

Without `.data`:

```text
Initialized global variables
would not exist after program startup.
```

---

# Program Memory Layout

Typical Process Memory Layout:

```text
+----------------------+
|        Stack         |
+----------------------+
|        Heap          |
+----------------------+
|        BSS           |
+----------------------+
|       Data           |
+----------------------+
|      Text            |
+----------------------+
```

---

## Memory Section Purpose

| Section | Purpose                               |
| ------- | ------------------------------------- |
| .text   | Program Instructions                  |
| .data   | Initialized Global/Static Variables   |
| .bss    | Uninitialized Global/Static Variables |
| .rodata | Read-Only Constants                   |
| Heap    | Dynamic Memory                        |
| Stack   | Local Variables                       |

---

# What is Stored in the Data Section?

---

## Global Initialized Variables

```c
int speed = 100;
```

Stored in:

```text
.data
```

---

## Static Initialized Variables

```c
static int counter = 1;
```

Stored in:

```text
.data
```

---

## Static Class Variables (C++)

```cpp
class Car
{
public:
    static int totalCars;
};

int Car::totalCars = 0;
```

Stored in:

```text
.data
```

---

# Types of Variables Stored in .data

---

## Example 1

```c
int a = 10;
```

Stored in:

```text
.data
```

---

## Example 2

```c
static int b = 20;
```

Stored in:

```text
.data
```

---

## Example 3

```c
char name[] = "ECU";
```

Stored in:

```text
.data
```

because the array is initialized.

---

# How the Data Section Works

Example:

```c
int speed = 100;

int main()
{
    speed = 120;
}
```

---

During Compilation:

```text
Variable:
speed

Initial Value:
100
```

stored in:

```text
.data
```

---

During Program Startup:

```text
Executable
      ↓
Loader
      ↓
Copy .data to RAM
      ↓
speed = 100
```

---

Runtime:

```c
speed = 120;
```

Value changes.

Therefore:

```text
.data is writable
```

---

# Data Section in ELF Files

Linux executables use:

```text
ELF
(Executable and Linkable Format)
```

---

View sections:

```bash
readelf -S app
```

Output:

```text
.text
.data
.bss
.rodata
```

---

View details:

```bash
objdump -h app
```

Displays:

```text
.data
```

section size and address.

---

# Data Section vs Other Memory Sections

| Section | Contains                |
| ------- | ----------------------- |
| .text   | Machine Code            |
| .data   | Initialized Variables   |
| .bss    | Uninitialized Variables |
| .rodata | Constants               |
| Heap    | malloc/new              |
| Stack   | Local Variables         |

---

Example:

```c
int a = 10;
```

Stored in:

```text
.data
```

---

Example:

```c
int b;
```

Stored in:

```text
.bss
```

---

Example:

```c
const char msg[] = "Hello";
```

Stored in:

```text
.rodata
```

---

Example:

```c
int local = 5;
```

Stored in:

```text
Stack
```

---

# Memory Protection

The Data Section is:

```text
Readable
Writable
```

Permission:

```text
RW-
```

---

Why?

Because variables must change during execution.

Example:

```c
count++;
```

requires write access.

---

# What Happens When Code Hits It?

Example:

```c
int speed = 100;

int main()
{
    speed = 120;
}
```

---

## Compilation

Compiler places:

```text
speed = 100
```

inside:

```text
.data
```

---

## Program Startup

Loader:

```text
1. Reads Executable

2. Finds .data Section

3. Allocates RAM

4. Copies Initial Values

5. speed = 100
```

---

## Runtime

Code executes:

```c
speed = 120;
```

CPU performs:

```text
Read Address of speed

Write New Value

speed = 120
```

---

### Interview Answer

**What happens when code accesses the Data Section?**

When the program starts, the loader copies initialized global and static variables from the executable's `.data` section into RAM. Whenever the code reads or modifies these variables, the CPU accesses the memory locations allocated for the `.data` section. Since the section is writable, variable values can change during program execution.

---

# Real-Time Examples

## Example 1: Automotive ECU

```c
int engine_rpm_limit = 6500;
```

Stored in:

```text
.data
```

---

Runtime:

```c
engine_rpm_limit = 7000;
```

Updated in RAM.

---

## Example 2: Sensor Driver

```c
static int adc_channel = 3;
```

Stored in:

```text
.data
```

---

## Example 3: Communication Stack

```c
int retry_count = 5;
```

Stored in:

```text
.data
```

---

## Example 4: Embedded Linux Application

```c
int camera_port = 8080;
```

Stored in:

```text
.data
```

---

# Data Section in Embedded Systems

Very Important Interview Topic.

---

Microcontroller Memory Layout:

```text
Flash Memory
 ├── .text
 ├── Initial .data Values

RAM
 ├── Runtime .data
 ├── .bss
 ├── Heap
 ├── Stack
```

---

Startup Sequence:

```text
Power ON
     ↓
Reset Handler
     ↓
Copy .data
Flash → RAM
     ↓
Initialize Variables
     ↓
main()
```

---

Example:

```c
int speed = 100;
```

Initially:

```text
Flash
```

At startup:

```text
Copied to RAM
```

Runtime changes occur in RAM.

---

# Advantages and Disadvantages

## Advantages

✔ Stores Initialized Variables

✔ Fast Access

✔ Supports Runtime Updates

✔ Persistent Initial Values

✔ Required for Global Data

---

## Disadvantages

✘ Consumes RAM

✘ Large Variables Increase Memory Usage

✘ Startup Copy Time Required

✘ Not Suitable for Huge Buffers

---

# Interview Questions

## What is the Data Section?

The `.data` section stores initialized global and static variables that can be modified during program execution.

---

## What Variables are Stored in .data?

```text
Global Initialized Variables
Static Initialized Variables
Static Class Variables
```

---

## Is the Data Section Writable?

Yes.

Permissions:

```text
RW-
```

---

## Difference Between .data and .bss?

| .data                        | .bss                        |
| ---------------------------- | --------------------------- |
| Initialized Variables        | Uninitialized Variables     |
| Occupies Space in Executable | Does Not Store Initial Data |
| Copied to RAM                | Zero Initialized            |

---

## Difference Between .data and Stack?

| .data                   | Stack                    |
| ----------------------- | ------------------------ |
| Global/Static Variables | Local Variables          |
| Exists Entire Program   | Exists Per Function Call |
| Fixed Memory Region     | Dynamic Growth           |

---

## Why is .data Stored in RAM?

Because initialized variables must be writable during execution.

---

## What Happens When Code Hits It?

The CPU reads or writes initialized global and static variables stored in RAM. These variables were copied from the executable's `.data` section during program startup.

---

# Most Asked Interview Question

## Explain the Data Section in an Embedded System.

In embedded systems, the `.data` section stores initialized global and static variables. Their initial values are stored in Flash memory as part of the firmware image. During startup, the reset handler copies these values from Flash to RAM because variables in the `.data` section must be writable. After initialization, the CPU accesses and modifies these variables from RAM during program execution.

---

# Interview Answer (2-Minute Version)

The `.data` section is a memory segment that stores initialized global and static variables. The compiler places variables with predefined values into this section. In embedded systems, the initial values are stored in Flash and copied to RAM during startup. Since these variables can change during execution, the `.data` section is readable and writable. It differs from the `.bss` section, which stores uninitialized variables, and from the `.text` section, which stores executable code.

---

# Quick Revision

```text
Data Section (.data)

Purpose:
Stores Initialized Variables

Contains:
✔ Global Variables
✔ Static Variables
✔ Static Class Variables

Permissions:
RW-

Startup:
Flash
 ↓
Copy to RAM
 ↓
main()

Examples:
int speed = 100;
static int count = 1;

Not Stored:
✘ Local Variables
✘ Dynamic Memory
✘ Program Code

Most Important:
.data = Initialized Global/Static Variables
Stored in RAM During Runtime
```

---

# Conclusion

The `.data` section is a fundamental memory segment responsible for storing initialized global and static variables. During program startup, the loader or embedded startup code copies the initial values from non-volatile storage into RAM, allowing the variables to be modified during execution. Understanding the `.data` section is essential for Embedded Systems, Firmware Development, Linux Internals, Automotive Software, Memory Management, and technical interviews.
