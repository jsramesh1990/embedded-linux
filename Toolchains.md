# Embedded Linux Toolchains

## Overview

A **toolchain** is a collection of software development tools used to build programs for a target system.

In embedded Linux, toolchains are used to:
- compile source code
- assemble machine instructions
- link binaries
- debug applications
- generate firmware images

---

# 1. Toolchain Architecture

```

Source Code (.c/.cpp)
↓
Preprocessor
↓
Compiler
↓
Assembler
↓
Object Files (.o)
↓
Linker
↓
ELF Executable
↓
Binary Image (.bin/.img)
↓
Embedded Target

```id="flow1"

---

# 2. Why Toolchains are Important

Embedded systems usually:
- use different CPU architectures
- have limited resources
- require cross compilation

Toolchains solve this problem.

---

# 3. Main Components of a Toolchain

| Component | Purpose |
|----------|--------|
| gcc | Compiler |
| cpp | Preprocessor |
| as | Assembler |
| ld | Linker |
| gdb | Debugger |
| libc | Standard C library |
| objcopy | Binary conversion |
| objdump | Binary inspection |

---

# 4. Compiler

## Role

Converts:
```

C/C++ → Assembly

````id="cmp1"

---

## Example

```bash
gcc -S main.c
``` id="cmp2"

---

# 5. Assembler

## Role

Converts:
````

Assembly → Machine Code

````id="asm1"

---

## Example

```bash
as file.s -o file.o
``` id="asm2"

---

# 6. Linker

## Role

Combines object files into executable.

````

main.o + driver.o + libc → app

````id="ld1"

---

## Example

```bash
ld main.o utils.o -o app
``` id="ld2"

---

# 7. C Library

Provides standard functions:

- printf()
- malloc()
- fopen()

---

## Common Libraries

| Library | Usage |
|--------|------|
| glibc | Full Linux systems |
| musl | Lightweight embedded |
| uClibc | Small embedded systems |

---

# 8. Cross Toolchain

## Definition

Toolchain that generates binaries for another architecture.

---

## Example

| Host | Target |
|------|--------|
| x86 PC | ARM board |

---

## Toolchain Prefix

```bash
arm-linux-gnueabihf-gcc
``` id="cross1"

---

# 9. Common Toolchain Prefixes

| Architecture | Prefix |
|-------------|--------|
| ARM32 | arm-linux-gnueabihf- |
| ARM64 | aarch64-linux-gnu- |
| RISC-V | riscv64-linux-gnu- |
| MIPS | mips-linux-gnu- |

---

# 10. Cross Compilation Flow

````

x86 Host PC
↓
Cross Toolchain
↓
ARM Binary
↓
Embedded Linux Board

```id="flow2"

---

# 11. Toolchain Directory Structure

```

toolchain/
├── bin/
├── include/
├── lib/
├── sysroot/
└── share/

```id="dir1"

---

# 12. bin Directory

Contains executable tools:

```

gcc
g++
ld
as
objdump
gdb

````id="bin1"

---

# 13. Sysroot

## Definition

Target filesystem used during compilation.

Contains:
- headers
- libraries
- runtime files

---

## Example

```bash
--sysroot=/opt/arm-sysroot
``` id="sys1"

---

# 14. Toolchain Types

## 14.1 Native Toolchain

````

x86 → x86

```id="nat1"

---

## 14.2 Cross Toolchain

```

x86 → ARM

````id="cross2"

---

## 14.3 Bare Metal Toolchain

No operating system.

Example:
```bash
arm-none-eabi-gcc
``` id="bare1"

---

# 15. Embedded Linux Toolchains

## Popular Toolchains

| Toolchain | Vendor |
|----------|--------|
| GNU GCC | Open source |
| Linaro | ARM optimized |
| Buildroot | Embedded Linux |
| Yocto SDK | Custom Linux builds |

---

# 16. Buildroot Toolchain

Buildroot can generate:
- cross compiler
- root filesystem
- kernel image

---

## Buildroot Flow

````

Buildroot
↓
Toolchain
↓
Kernel
↓
RootFS

````id="br1"

---

# 17. Yocto Toolchain

Yocto creates:
- SDK
- sysroot
- cross compiler
- package manager

---

## Example

```bash
source environment-setup-armv7a
``` id="yocto1"

---

# 18. Debugging Tools

## GDB

```bash
arm-linux-gnueabihf-gdb app
``` id="gdb1"

---

## objdump

```bash
objdump -d app
``` id="obj1"

---

## readelf

```bash
readelf -a app
``` id="elf1"

---

# 19. ELF Binary

Toolchain generates:

````

ELF executable

````id="elf2"

Contains:
- machine code
- symbols
- debug information

---

# 20. Linker Scripts

Controls memory placement.

Example:

```ld
SECTIONS
{
   .text : { *(.text) }
   .data : { *(.data) }
   .bss  : { *(.bss) }
}
``` id="lds1"

---

# 21. Toolchain Setup Example

## Install ARM GCC

```bash
sudo apt install gcc-arm-linux-gnueabihf
``` id="inst1"

---

## Verify

```bash
arm-linux-gnueabihf-gcc --version
``` id="inst2"

---

# 22. Makefile Example

```makefile
CC = arm-linux-gnueabihf-gcc

all:
	$(CC) main.c -o app
``` id="mk1"

---

# 23. Common Problems

## Wrong Architecture

Error:
``` id="mybtbf"
Exec format error
````

---

## Missing Libraries

Error:

```id="0g4d1o"
libc.so not found
```

---

## ABI Mismatch

* hard-float vs soft-float
* ARMv7 vs ARMv8

---

# 24. Embedded Linux Build Flow

````
Application
      ↓
Cross Compiler
      ↓
ARM ELF Binary
      ↓
Root Filesystem
      ↓
Bootloader
      ↓
Embedded Linux System
``` id="flow3"

---
