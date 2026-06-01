# Header Files in C (Embedded Systems)

## Table of Contents

1. [Introduction](#introduction)
2. [What is a Header File?](#what-is-a-header-file)
3. [File Extension of Header Files](#file-extension-of-header-files)
4. [Why We Use Header Files](#why-we-use-header-files)
5. [How Header Files Work](#how-header-files-work)
6. [How Header Files Link with Code](#how-header-files-link-with-code)
7. [Types of Header Files](#types-of-header-files)
8. [Preprocessor Directives](#preprocessor-directives)
9. [Compilation Flow](#compilation-flow)
10. [Real-Time Embedded Example](#real-time-embedded-example)
11. [Advantages and Disadvantages](#advantages-and-disadvantages)
12. [Interview Questions](#interview-questions)
13. [Conclusion](#conclusion)

---

# Introduction

In C programming (especially embedded systems), header files are used to **share declarations across multiple source files**.

They are a key part of modular programming.

---

# What is a Header File?

### Definition

> A header file is a file that contains function declarations, macro definitions, constants, and data type definitions that can be shared across multiple C source files.

---

## Example

```c
// my_header.h
void led_on();
void led_off();
```

---

# File Extension of Header Files

### Standard Extension

```text
.h  → Header file
```

---

## Examples

* `stdio.h`
* `stdlib.h`
* `string.h`
* `math.h`
* `gpio.h` (embedded custom driver header)
* `uart_driver.h`

---

# Why We Use Header Files

Without header files:

```text id="h3xq7p"
Each file repeats code → duplication
```

With header files:

```text id="z9p1ld"
Single declaration → reusable across files
```

---

## Key Reasons

✔ Code reusability
✔ Modular programming
✔ Easier maintenance
✔ Separation of interface and implementation
✔ Faster compilation management

---

# How Header Files Work

Header files are processed by the **C Preprocessor** before compilation.

---

## Step-by-Step Flow

### 1. Include Directive

```c
#include "my_header.h"
```

---

### 2. Preprocessor Action

It copies the content of header file into the source file.

---

## Example

### main.c

```c
#include "my_header.h"
```

### After preprocessing

```c
void led_on();
void led_off();
```

---

# How Header Files Link with Code

## Important Concept

Header files do NOT contain actual code (usually).

They contain:

* Function declarations
* Macros
* Struct definitions

---

## Actual Code is in `.c` files

### led.c

```c
#include "my_header.h"

void led_on()
{
    // hardware register write
}

void led_off()
{
    // hardware register write
}
```

---

## Linking Process

### Step 1: Compilation

```text id="k9q1as"
main.c → main.o
led.c  → led.o
```

---

### Step 2: Linking

```text id="8p3twr"
main.o + led.o → final executable
```

---

## Final Flow

```text id="q7n0vm"
Header (.h)
   ↓ include
Source (.c)
   ↓ compile
Object (.o)
   ↓ link
Executable
```

---

# Types of Header Files

## 1. Standard Library Headers

Provided by compiler

Examples:

* `stdio.h`
* `string.h`
* `math.h`

---

## 2. User-Defined Headers

Created by developers

Examples:

* `gpio.h`
* `sensor.h`
* `uart.h`

---

## 3. System / Kernel Headers (Embedded Linux)

Examples:

* `linux/gpio.h`
* `sys/socket.h`

---

# Preprocessor Directives

Header files use preprocessor commands.

---

## 1. #include

Includes header file.

```c
#include <stdio.h>
```

---

## 2. Include Guards

Prevents multiple inclusion.

```c
#ifndef MY_HEADER_H
#define MY_HEADER_H

void led_on();

#endif
```

---

## Why Include Guards?

Without it:

```text id="p8d7kx"
Multiple inclusion → redefinition error
```

---

# Compilation Flow

## Full Process

```text id="g1n9qd"
Source Code (.c)
        ↓
Preprocessor (.i)
        ↓
Compiler (.s)
        ↓
Assembler (.o)
        ↓
Linker (Executable)
```

---

## Role of Header File

It is included at:

```text id="v3m8pn"
Preprocessing stage
```

---

# Real-Time Embedded Example

## LED Driver System

---

### led.h

```c
void led_on();
void led_off();
```

---

### led.c

```c
#include "led.h"

void led_on()
{
    GPIO_WRITE(1);
}

void led_off()
{
    GPIO_WRITE(0);
}
```

---

### main.c

```c
#include "led.h"

int main()
{
    led_on();
}
```

---

## What Happens Internally?

```text id="x2p7ka"
main.c uses led.h declarations
        ↓
Compiler checks function signature
        ↓
Linker connects led.o implementation
        ↓
Final firmware generated
```

---

# Advantages

✔ Code modularity

✔ Reusability

✔ Easy maintenance

✔ Separation of interface and implementation

✔ Faster development in large projects

---

# Disadvantages

✘ Too many includes can slow compilation

✘ Circular dependency issues

✘ Poor design leads to complexity

✘ Debugging header issues can be tricky

---

# Interview Questions

### What is a Header File?

A file containing function declarations, macros, and type definitions used across multiple source files.

---

### What is the extension of a header file?

```text
.h
```

---

### What is the purpose of header files?

To share declarations between multiple C source files and enable modular programming.

---

### How are header files linked to code?

They are included using `#include`, processed during preprocessing, and their declarations are resolved during compilation and linking.

---

### What is the difference between `.h` and `.c` files?

| .h (Header)  | .c (Source)    |
| ------------ | -------------- |
| Declarations | Definitions    |
| Interface    | Implementation |

---

### What is include guard?

A technique used to prevent multiple inclusion of the same header file.

---

### Why do we use #include?

To copy the content of a header file into the source file during preprocessing.

---

# Most Asked Interview Question

### Explain Header Files and their working.

Header files are files with `.h` extension that contain function declarations, macros, and data type definitions. They are included in C programs using the `#include` directive. During preprocessing, the contents of the header file are copied into the source file. The actual implementation is present in `.c` files, and during linking, object files are combined to create the final executable. Header files help in modular programming, code reuse, and better project structure.

---

# Quick Revision

```text id="4k2pqn"
Header File (.h)
   ↓ contains declarations
Included using #include
   ↓ preprocessing
Copied into source file
   ↓ compile & link
Final executable
```

---

# Conclusion

Header files are essential in C and embedded systems for modular programming. They define interfaces between modules and allow code reuse across multiple files. Understanding how header files are included, compiled, and linked is critical for firmware development, driver design, and large-scale embedded projects like automotive ECUs, IoT devices, and embedded Linux systems.
