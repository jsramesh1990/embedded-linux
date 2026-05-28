# C Compilation Stages

## Overview

The compilation process converts **source code into an executable binary** through multiple stages.

Each stage transforms the code closer to machine-level instructions.

---

# 1. Complete Compilation Pipeline

```

Source Code (.c)
↓
Preprocessing
↓
Compilation
↓
Assembly
↓
Linking
↓
Executable (ELF)
↓
Binary / Flash Image

```id="cmp01"

---

# 2. Stage 1: Preprocessing

## Input
```

file.c

```id="p1"

## Output
```

Expanded C code

```id="p2"

## What happens

- Handles `#include`
- Expands `#define` macros
- Processes conditional compilation (`#ifdef`)

## Command

```

gcc -E file.c -o file.i

```

---

# 3. Stage 2: Compilation

## Input
```

Preprocessed file (.i)

```id="c1"

## Output
```

Assembly code (.s)

```id="c2"

## What happens

- Syntax analysis
- Semantic analysis
- Optimization
- Converts C → Assembly

## Command

```

gcc -S file.c -o file.s

```

---

# 4. Stage 3: Assembly

## Input
```

Assembly file (.s)

```id="a1"

## Output
```

Object file (.o)

```id="a2"

## What happens

- Converts assembly into machine code
- Creates relocatable object file

## Command

```

gcc -c file.c -o file.o

```

---

# 5. Stage 4: Linking

## Input
```

Object files (.o)

```id="l1"

## Output
```

Executable (ELF binary)

```id="l2"

## What happens

- Combines multiple object files
- Resolves function calls
- Links libraries (libc, math, etc.)

## Command

```

gcc file.o -o app

```

---

# 6. Full Build Flow Example

```

main.c + utils.c
↓
Preprocessor
↓
main.i + utils.i
↓
Compiler
↓
main.s + utils.s
↓
Assembler
↓
main.o + utils.o
↓
Linker
↓
app (ELF binary)

```id="flow1"

---

# 7. Embedded Cross Compilation Flow

```

x86 Host Machine
↓
Cross Compiler (ARM GCC)
↓
ARM Object Files
↓
ARM ELF Binary
↓
Flashed to Target Board

```id="cross1"

---

# 8. Output File Types

| Extension | Meaning |
|----------|--------|
| .c | Source code |
| .i | Preprocessed file |
| .s | Assembly code |
| .o | Object file |
| .elf | Executable file |
| .bin | Raw binary |

---

# 9. ELF File Structure

```

ELF Header
Program Headers
Section Headers
├── .text   (Code)
├── .data   (Initialized data)
├── .bss    (Uninitialized data)

```id="elf1"

---

# 10. Internal Compiler Phases

## Frontend
- Lexical analysis
- Syntax parsing
- Semantic analysis

## Middle-end
- Optimization

## Backend
- Instruction generation
- Architecture-specific code

---

# 11. Linking Types

## 11.1 Static Linking

- All libraries included in binary
- Larger executable
- No runtime dependency

## 11.2 Dynamic Linking

- Uses shared libraries (.so)
- Smaller binary
- Runtime dependency required

---

# 12. Memory Mapping After Linking

```

Text Segment   → Code
Data Segment   → Initialized variables
BSS Segment    → Zero-initialized variables
Heap           → Dynamic memory (malloc)
Stack          → Function calls

```id="mem1"

---

# 13. Debugging Compilation

## Generate assembly
```

gcc -S file.c

```

## Generate object file
```

gcc -c file.c

```

## View symbols
```

nm file.o

```

## View ELF details
```

readelf -a app

```

---

# 14. Common Errors

## 14.1 Undefined reference
- Missing function implementation

## 14.2 Multiple definition
- Same symbol defined in multiple files

## 14.3 Header missing
- Compilation failure in preprocessing stage

---

# 15. Embedded Linux Relevance

- Used in kernel compilation
- Used in bootloader builds (U-Boot)
- Used in driver compilation
- Cross-compilation essential for ARM targets

---

# 16. Complete Summary

- Preprocessing → macro expansion
- Compilation → C to assembly
- Assembly → machine code
- Linking → final executable
- Output → ELF / binary for embedded system

---

