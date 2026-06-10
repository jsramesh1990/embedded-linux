# Loader (Program Loader in Operating Systems)

## Table of Contents

1. [Introduction](#introduction)
2. [What is a Loader?](#what-is-a-loader)
3. [Why Do We Need a Loader?](#why-do-we-need-a-loader)
4. [Program Execution Flow](#program-execution-flow)
5. [How a Loader Works](#how-a-loader-works)
6. [Types of Loaders](#types-of-loaders)
7. [Absolute Loader](#absolute-loader)
8. [Relocating Loader](#relocating-loader)
9. [Dynamic Loader](#dynamic-loader)
10. [Bootstrap Loader](#bootstrap-loader)
11. [Linux Program Loader](#linux-program-loader)
12. [ELF Loader](#elf-loader)
13. [Static vs Dynamic Linking](#static-vs-dynamic-linking)
14. [Memory Layout After Loading](#memory-layout-after-loading)
15. [Loader in Embedded Systems](#loader-in-embedded-systems)
16. [Bootloader vs Loader](#bootloader-vs-loader)
17. [What Happens When Code Hits It?](#what-happens-when-code-hits-it)
18. [Real-Time Examples](#real-time-examples)
19. [Advantages and Disadvantages](#advantages-and-disadvantages)
20. [Interview Questions](#interview-questions)
21. [Quick Revision](#quick-revision)
22. [Conclusion](#conclusion)

---

# Introduction

A **Loader** is an Operating System component responsible for:

```text
Loading an Executable Program
Into Memory
And Starting Its Execution
```

Without a loader:

```text
Executable File Exists
BUT
CPU Cannot Execute It
```

The loader acts as a bridge between:

```text
Executable File
        ↓
RAM
        ↓
CPU Execution
```

---

# What is a Loader?

Definition:

> A Loader is a system program that loads an executable file into memory, prepares it for execution, and transfers control to the program.

---

## Example

Suppose:

```bash
./app
```

is executed.

The loader:

```text
1. Reads app file

2. Allocates Memory

3. Loads Code Sections

4. Loads Data Sections

5. Resolves Libraries

6. Sets Stack

7. Starts main()
```

---

# Why Do We Need a Loader?

Programs are stored on disk as:

```text
ELF Files
EXE Files
BIN Files
```

CPU cannot directly execute these files from storage.

Need to:

```text
Load Into RAM
Assign Memory Addresses
Resolve Symbols
Prepare Execution Environment
```

This is the loader's job.

---

# Program Execution Flow

```text
Source Code
     ↓
Compiler
     ↓
Object File (.o)
     ↓
Linker
     ↓
Executable (ELF)
     ↓
Loader
     ↓
RAM
     ↓
CPU Executes
```

---

# How a Loader Works

When user runs:

```bash
./hello
```

Loader performs:

```text
Step 1:
Read ELF Header

Step 2:
Create Process

Step 3:
Allocate Memory

Step 4:
Load Code Segment

Step 5:
Load Data Segment

Step 6:
Setup Stack

Step 7:
Resolve Libraries

Step 8:
Jump To Entry Point
```

---

# Types of Loaders

| Loader Type       | Purpose                |
| ----------------- | ---------------------- |
| Absolute Loader   | Loads at fixed address |
| Relocating Loader | Adjusts addresses      |
| Dynamic Loader    | Loads shared libraries |
| Bootstrap Loader  | Loads OS during boot   |

---

# Absolute Loader

Loads program into predefined address.

---

Example

```text
Program Must Load At

0x1000
```

Loader places code exactly there.

---

Advantages:

```text
Simple
Fast
```

Disadvantages:

```text
No Flexibility
```

---

# Relocating Loader

Can load program anywhere in memory.

---

Example

Program designed for:

```text
0x1000
```

Actually loaded at:

```text
0x8000
```

Loader updates all addresses.

---

Why Needed?

```text
Multiple Programs
Virtual Memory
Modern OS
```

---

# Dynamic Loader

Loads shared libraries at runtime.

---

Example

```cpp
printf();
```

comes from:

```text
libc.so
```

---

Loader:

```text
Loads libc.so
Resolves printf()
Links Program
```

during execution.

---

Advantages

```text
Smaller Executables
Shared Libraries
Lower Memory Usage
```

---

# Bootstrap Loader

Special loader used during system startup.

---

Flow:

```text
Power ON
    ↓
ROM Code
    ↓
Bootstrap Loader
    ↓
Operating System
```

---

Examples

```text
U-Boot
GRUB
BIOS Loader
```

---

# Linux Program Loader

Linux uses:

```text
execve()
```

system call.

---

When:

```bash
./app
```

is executed.

Kernel:

```text
Reads ELF

Creates Process

Loads Segments

Maps Libraries

Starts Program
```

---

# ELF Loader

Linux executable format:

```text
ELF
=
Executable and Linkable Format
```

---

Tools:

```bash
readelf app

objdump app
```

---

ELF Contains:

```text
Header
Text Section
Data Section
BSS
Symbols
Entry Point
```

---

# Static vs Dynamic Linking

## Static Linking

Libraries copied into executable.

```text
Executable
 ├── Application
 ├── libc
 ├── libm
```

---

Advantages

```text
No External Dependency
```

---

Disadvantages

```text
Large Binary
```

---

## Dynamic Linking

Libraries loaded by loader.

```text
Executable
      ↓
Loader
      ↓
libc.so
libm.so
```

---

Advantages

```text
Small Binary
Shared Libraries
```

---

# Memory Layout After Loading

Example:

```text
+----------------+
| Stack          |
+----------------+
| Heap           |
+----------------+
| BSS            |
+----------------+
| Data Segment   |
+----------------+
| Text Segment   |
+----------------+
```

---

Meaning:

### Text Segment

```text
Program Instructions
```

---

### Data Segment

```text
Initialized Variables
```

---

### BSS

```text
Uninitialized Variables
```

---

### Heap

```text
malloc()
new
```

---

### Stack

```text
Function Calls
Local Variables
```

---

# Loader in Embedded Systems

Commonly used in:

```text
Embedded Linux
Automotive ECUs
RTOS Systems
IoT Devices
```

---

Functions:

```text
Load Firmware
Verify Firmware
Relocate Sections
Initialize Memory
Start Application
```

---

# Bootloader vs Loader

| Bootloader      | Loader                |
| --------------- | --------------------- |
| Loads OS        | Loads Application     |
| Runs At Startup | Runs During Execution |
| U-Boot, GRUB    | ELF Loader            |
| Firmware Level  | OS Level              |

---

Example:

```text
Power ON
    ↓
Bootloader
    ↓
Linux Kernel
    ↓
Loader
    ↓
User Application
```

---

# What Happens When Code Hits It?

## Example

```bash
./hello
```

Execution:

```text
1. Shell Calls execve()

2. Kernel Invokes Loader

3. ELF Header Read

4. Memory Allocated

5. Sections Loaded

6. Libraries Mapped

7. Stack Created

8. Registers Initialized

9. Entry Point Executed

10. main() Starts
```

---

## Internal Flow

```text
Disk
 ↓
ELF File
 ↓
Loader
 ↓
Memory Mapping
 ↓
Program Counter Set
 ↓
_start()
 ↓
main()
```

---

# Real-Time Examples

## Example 1: Linux Application

```bash
./camera_app
```

Loader:

```text
Loads camera_app
Loads OpenCV Libraries
Starts Program
```

---

## Example 2: Embedded Linux

System Boot:

```text
U-Boot
 ↓
Linux Kernel
 ↓
Loader
 ↓
ADAS Application
```

---

## Example 3: Automotive ECU

```text
Firmware Image
      ↓
Loader
      ↓
RAM
      ↓
Application Starts
```

---

## Example 4: Shared Library Loading

```cpp
printf("Hello");
```

Loader:

```text
Find printf()

Map libc.so

Resolve Symbol
```

---

# Advantages and Disadvantages

## Advantages

✔ Automatic Program Loading

✔ Dynamic Library Support

✔ Memory Management

✔ Address Relocation

✔ Process Initialization

✔ Shared Library Optimization

---

## Disadvantages

✘ Startup Overhead

✘ Dynamic Linking Complexity

✘ Relocation Cost

✘ Dependency Issues

---

# Interview Questions

## What is a Loader?

A loader is an OS component that loads executable files into memory, prepares them for execution, and transfers control to the program.

---

## Why Do We Need a Loader?

Because executables stored on disk cannot be executed directly by the CPU. They must first be loaded into RAM and prepared for execution.

---

## What Happens When Code Hits It?

### Example

```bash
./app
```

### Answer

```text
1. execve() Called

2. ELF Read

3. Memory Allocated

4. Sections Loaded

5. Libraries Resolved

6. Entry Point Executed

7. main() Starts
```

---

## Difference Between Loader and Linker?

| Linker             | Loader            |
| ------------------ | ----------------- |
| Creates Executable | Runs Executable   |
| Build Time         | Runtime           |
| Resolves Symbols   | Loads Into Memory |

---

## What is Dynamic Loading?

Loading shared libraries during runtime.

Example:

```text
libc.so
libpthread.so
```

---

## What is Relocation?

Adjusting memory addresses when a program is loaded into a different memory location than originally expected.

---

## What is ELF?

```text
Executable and Linkable Format
```

Used by Linux executables.

---

## What is execve()?

Linux system call that creates a process and invokes the program loader.

---

# Most Asked Interview Question

## Explain the Complete Flow From Executing an Application to main().

When a user runs an executable such as `./app`, the shell invokes the `execve()` system call. The Linux kernel reads the ELF executable, creates a new process, allocates memory, loads code and data sections, sets up the stack and heap, maps required shared libraries, initializes CPU registers, and jumps to the executable's entry point (`_start`). The C runtime initializes the application environment and finally calls the `main()` function.

---

# Interview Answer (2-Minute Version)

A loader is an operating system component responsible for loading executable files into memory and preparing them for execution. When a program is launched, the loader reads the executable format (such as ELF in Linux), allocates memory, loads program sections, resolves shared library dependencies, sets up the stack and heap, and transfers execution to the program's entry point. Loaders support relocation, dynamic linking, and process initialization. In embedded Linux and automotive systems, loaders are used to load firmware, applications, and shared libraries efficiently.

---

# Quick Revision

```text
Loader

Purpose:
Load Executable Into Memory

Functions:
1. Allocate Memory
2. Load Sections
3. Resolve Libraries
4. Setup Stack
5. Start Execution

Types:
1. Absolute Loader
2. Relocating Loader
3. Dynamic Loader
4. Bootstrap Loader

Linux:
execve()
ELF Loader

Flow:
Executable
    ↓
Loader
    ↓
Memory
    ↓
_start()
    ↓
main()

Used In:
✔ Linux
✔ Embedded Linux
✔ Automotive
✔ RTOS
✔ Firmware Systems
```

---

# Conclusion

The Loader is a critical operating system component that bridges the gap between an executable file stored on disk and a running process in memory. It handles memory allocation, relocation, dynamic library loading, stack initialization, and execution startup. Understanding the loader is essential for Embedded Linux, Automotive, System Programming, Firmware Development, and OS-level interviews because it explains exactly how a program transitions from an executable file to a running application.
