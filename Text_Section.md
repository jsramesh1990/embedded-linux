# Text Section (.text)

## Table of Contents

1. [Introduction](#introduction)
2. [What is the Text Section?](#what-is-the-text-section)
3. [Why Do We Need the Text Section?](#why-do-we-need-the-text-section)
4. [Program Memory Layout](#program-memory-layout)
5. [Where Text Section is Stored](#where-text-section-is-stored)
6. [What Goes Into the Text Section?](#what-goes-into-the-text-section)
7. [How the Text Section Works](#how-the-text-section-works)
8. [Text Section in ELF Files](#text-section-in-elf-files)
9. [Text Section vs Other Sections](#text-section-vs-other-sections)
10. [Memory Protection](#memory-protection)
11. [What Happens When Code Hits It?](#what-happens-when-code-hits-it)
12. [Real-Time Examples](#real-time-examples)
13. [Text Section in Embedded Systems](#text-section-in-embedded-systems)
14. [Advantages and Disadvantages](#advantages-and-disadvantages)
15. [Interview Questions](#interview-questions)
16. [Quick Revision](#quick-revision)
17. [Conclusion](#conclusion)

---

# Introduction

When a C/C++ program is compiled, the executable is divided into multiple memory sections:

```text
.text
.data
.bss
.heap
.stack
```

Among them:

```text
.text
```

contains the actual executable machine instructions.

---

# What is the Text Section?

Definition:

> The Text Section (.text) is the memory area that stores the compiled machine code instructions of a program.

---

Example:

C Code

```cpp
int add(int a, int b)
{
    return a + b;
}
```

Compiler converts it into machine instructions.

Those instructions are stored in:

```text
.text section
```

---

# Why Do We Need the Text Section?

CPU executes instructions.

Instructions must be stored somewhere.

The Text Section provides:

```text
Storage for Machine Code
Executable Instructions
Read-Only Protection
Efficient Memory Sharing
```

Without it:

```text
CPU Has No Instructions To Execute
```

---

# Program Memory Layout

Typical Linux Process Layout:

```text
+----------------------+
|       Stack          |
+----------------------+
|       Heap           |
+----------------------+
|       BSS            |
+----------------------+
|       Data           |
+----------------------+
|       Text           |
+----------------------+
```

---

## Section Purposes

| Section | Contains                         |
| ------- | -------------------------------- |
| .text   | Machine Instructions             |
| .data   | Initialized Global Variables     |
| .bss    | Uninitialized Global Variables   |
| Heap    | Dynamic Memory                   |
| Stack   | Function Calls & Local Variables |

---

# Where Text Section is Stored

During Build:

```text
Source Code
     ↓
Compiler
     ↓
Object File
     ↓
Executable
```

The generated machine code is stored in:

```text
.text
```

inside the executable.

---

When the program runs:

```text
Disk
 ↓
Loader
 ↓
RAM
 ↓
.text Mapped Into Memory
```

---

# What Goes Into the Text Section?

---

## Functions

```cpp
void display()
{
}
```

Stored in:

```text
.text
```

---

## Class Member Functions

```cpp
class Car
{
public:
    void start()
    {
    }
};
```

Stored in:

```text
.text
```

---

## Static Functions

```cpp
static void helper()
{
}
```

Stored in:

```text
.text
```

---

## Library Functions

```cpp
printf();
memcpy();
malloc();
```

Machine code stored in library text sections.

---

# How the Text Section Works

Example:

```cpp
#include <iostream>

void hello()
{
    std::cout<<"Hello";
}

int main()
{
    hello();
}
```

---

Compiler generates:

```text
Machine Instructions
```

for:

```text
main()
hello()
```

---

Stored in:

```text
.text
```

---

Execution:

```text
CPU
 ↓
Fetch Instruction
 ↓
Decode
 ↓
Execute
```

from the text section.

---

# Text Section in ELF Files

Linux executables use:

```text
ELF
(Executable and Linkable Format)
```

---

View Sections:

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

Example:

```bash
objdump -h app
```

Shows:

```text
.text
```

size and address.

---

# Text Section vs Other Sections

| Section | Stores                  |
| ------- | ----------------------- |
| .text   | Instructions            |
| .data   | Initialized Variables   |
| .bss    | Uninitialized Variables |
| .rodata | Read-Only Constants     |
| Heap    | Dynamic Allocations     |
| Stack   | Local Variables         |

---

Example:

```cpp
int global_var = 10;
```

Stored in:

```text
.data
```

---

Example:

```cpp
int global_uninit;
```

Stored in:

```text
.bss
```

---

Example:

```cpp
const char* msg = "Hello";
```

String:

```text
.rodata
```

Code:

```text
.text
```

---

# Memory Protection

Most operating systems mark:

```text
.text
```

as:

```text
Read-Only
Executable
```

---

Permissions:

```text
R-X
```

Meaning:

```text
Read
Execute
```

No write access.

---

Why?

Security.

Prevents:

```text
Accidental Code Modification
Malware Injection
Buffer Overflow Exploits
```

---

# What Happens When Code Hits It?

Consider:

```cpp
int add(int a,int b)
{
    return a+b;
}

int main()
{
    add(10,20);
}
```

---

Compilation Stage

```text
add()
main()
```

converted into machine instructions.

Stored in:

```text
.text
```

---

Program Start

```text
Loader
 ↓
Loads .text into RAM
```

---

Execution

CPU Program Counter (PC):

```text
Points to main()
```

---

Step-by-Step

```text
1. Fetch instruction from .text

2. Decode instruction

3. Execute instruction

4. Next instruction

5. Call add()

6. Jump to add() code in .text

7. Execute add()

8. Return to main()
```

---

### Interview Answer

**What happens when the code hits the Text Section?**

When a program starts, the loader maps the `.text` section into memory. The CPU's Program Counter points to instructions inside the `.text` section. The CPU continuously fetches, decodes, and executes machine instructions from this section. Every function call, loop, conditional branch, and return statement ultimately executes code stored in the `.text` section.

---

# Real-Time Examples

## Example 1: Linux Application

```cpp
int main()
{
    printf("Hello");
}
```

Machine instructions for:

```text
main()
```

stored in:

```text
.text
```

---

## Example 2: Embedded Firmware

```cpp
void Motor_Control()
{
}
```

Firmware code stored in:

```text
.text
```

inside Flash memory.

---

## Example 3: Automotive ECU

Functions:

```text
Brake Control
ABS
Airbag Logic
```

stored in:

```text
.text
```

and executed by ECU processor.

---

## Example 4: Sensor Driver

```cpp
void Read_ADC()
{
}
```

Instructions stored in:

```text
.text
```

---

# Text Section in Embedded Systems

Very important topic.

---

Typical MCU Memory Layout:

```text
Flash Memory
 ├── .text
 ├── .rodata

RAM
 ├── .data
 ├── .bss
 ├── Heap
 ├── Stack
```

---

Example:

STM32

```text
Flash
  ↓
.text stored here

RAM
  ↓
Variables stored here
```

---

Why?

Flash is non-volatile.

Program code remains after power loss.

---

# Advantages and Disadvantages

## Advantages

✔ Stores Executable Instructions

✔ Read-Only Protection

✔ Shared Among Processes

✔ Efficient Memory Usage

✔ Supports Security Mechanisms

---

## Disadvantages

✘ Cannot Be Modified at Runtime

✘ Limited by Flash/ROM Size

✘ Large Code Increases Memory Usage

---

# Interview Questions

## What is the Text Section?

The `.text` section stores the executable machine code instructions of a program.

---

## What is Stored in the Text Section?

```text
Functions
Class Methods
Machine Instructions
Library Code
```

---

## Is Text Section Read Only?

Yes.

Typically:

```text
Read + Execute
```

Permission:

```text
R-X
```

---

## Where is Text Section Stored?

| Environment     | Location            |
| --------------- | ------------------- |
| Linux           | RAM (after loading) |
| Embedded MCU    | Flash Memory        |
| Executable File | ELF .text Section   |

---

## What Happens When Code Hits It?

The CPU fetches instructions from the `.text` section, decodes them, executes them, and updates the Program Counter to continue execution.

---

## Difference Between .text and .data?

| .text        | .data          |
| ------------ | -------------- |
| Instructions | Variables      |
| Read Only    | Read/Write     |
| Executable   | Non-Executable |

---

## Difference Between .text and .rodata?

| .text           | .rodata          |
| --------------- | ---------------- |
| Machine Code    | Read-Only Data   |
| Executed by CPU | Accessed as Data |

---

## Why is .text Read-Only?

To protect program instructions from accidental modification and security attacks.

---

# Most Asked Interview Question

## Explain the Text Section in an Embedded System.

In embedded systems, the `.text` section contains the executable firmware instructions and is usually stored in Flash memory. When the microcontroller starts, the CPU fetches instructions directly from Flash or from memory mapped regions. Functions such as sensor reading, motor control, communication stacks, and interrupt handlers all reside in the `.text` section. Since code should not change during execution, the section is typically read-only and executable.

---

# Interview Answer (2-Minute Version)

The `.text` section is the part of an executable that contains compiled machine instructions. It stores program code such as functions, class methods, and library routines. During program execution, the loader maps the `.text` section into memory, and the CPU fetches and executes instructions from it. The section is usually marked as read-only and executable to improve security and prevent accidental modification. In embedded systems, the `.text` section is commonly stored in Flash memory, while variables are stored in RAM.

---

# Quick Revision

```text
Text Section (.text)

Purpose:
Stores Machine Instructions

Contains:
✔ Functions
✔ Methods
✔ Executable Code

Permissions:
✔ Read
✔ Execute

Permission Flag:
R-X

Not Stored:
✘ Variables
✘ Heap Data
✘ Stack Data

Linux:
ELF → .text

Embedded:
Flash Memory

Execution:
CPU Fetches Instructions
Directly From .text

Most Important:
.text = Program Code
```

---

# Conclusion

The `.text` section is one of the most fundamental memory segments in software systems. It contains the executable instructions generated by the compiler and is the primary source of code execution for the CPU. Understanding the `.text` section is essential for Embedded Systems, Linux Internals, Automotive Software, Firmware Development, Memory Management, and Operating System interviews because it explains where program instructions reside and how they are executed.
