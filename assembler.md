#  assembler.md

---

#  Introduction

An **assembler** is a system software tool that converts **assembly language code** into **machine code (binary instructions)** that the CPU or microcontroller can execute directly.

In embedded systems, assemblers are very important because:

* startup code is often written in assembly
* low-level hardware control needs assembly
* interrupt handling uses assembly instructions
* performance-critical code may use assembly optimization

---

#  Definition

An assembler is a program that **translates assembly language instructions into machine-level binary code**.

---

#  Why Assembler is Important in Embedded Systems

* converts assembly to machine instructions
* enables direct hardware access
* used in boot/startup code
* helps optimize speed and memory
* required for interrupt/vector table setup

---

#  Assembly Language Overview

Assembly language is a **low-level programming language** close to CPU instructions.

Example:

```text id="asm1"
MOV R0, #1
ADD R1, R0, #5
```

Each instruction maps almost directly to machine code.

---

#  Build Process Position

```text id="flow1"
C Source Code
      ↓
Compiler
      ↓
Assembly Code (.s)
      ↓
Assembler
      ↓
Object File (.o)
      ↓
Linker
      ↓
Executable (.elf/.hex)
```

---

#  Graphical Representation

```text id="diag1"
+----------------------+
| Assembly Code (.s)   |
| MOV, ADD, SUB        |
+----------+-----------+
           ↓
+----------------------+
|      ASSEMBLER       |
| Converts to binary   |
+----------+-----------+
           ↓
+----------------------+
| Object File (.o)     |
| Machine Instructions |
+----------------------+
```

---

#  Assembly Language Syntax

---

## ARM Assembly Example

```asm id="arm1"
MOV R0, #10
MOV R1, #20
ADD R2, R0, R1
```

---

## x86 Assembly Example

```asm id="x861"
MOV AX, 5
MOV BX, 10
ADD AX, BX
```

---

#  How Assembler Works Internally

```text id="flow2"
1. Reads assembly instructions
2. Converts mnemonics into opcodes
3. Resolves labels/symbols
4. Generates machine code
5. Produces object file (.o)
```

---

#  Internal Assembler Flow

```text id="diag2"
Assembly Source
      ↓
Lexical Analysis
      ↓
Opcode Translation
      ↓
Address Resolution
      ↓
Machine Code Generation
      ↓
Object File (.o)
```

---

#  Assembly Instruction Components

```text id="cmp1"
LABEL:   OPCODE   OPERAND
START:   MOV      R0, #1
```

---

#  Common Assembly Instructions

| Instruction | Purpose     |
| ----------- | ----------- |
| MOV         | Move data   |
| ADD         | Addition    |
| SUB         | Subtraction |
| CMP         | Compare     |
| B / JMP     | Branch/Jump |
| LDR         | Load data   |
| STR         | Store data  |

---

# ⚙ Example: C to Assembly

---

## C Code

```c id="c1"
int a = 5;
int b = 10;
int c = a + b;
```

---

## Generated Assembly (Simplified)

```asm id="asm2"
MOV R0, #5
MOV R1, #10
ADD R2, R0, R1
```

---

#  C → Assembly → Machine Code

```text id="flow3"
High-level C code
        ↓
Compiler generates assembly
        ↓
Assembler generates binary opcodes
        ↓
CPU executes instructions
```

---

#  Machine Code Example

```text id="machine1"
Assembly:
MOV R0, #1

Machine Code:
11100011101000000000000000000001
```

---

#  Real Embedded Example

## Startup Code

Embedded MCUs start execution from assembly startup file:

```asm id="startup1"
Reset_Handler:
    LDR SP, =_stack_top
    BL main
```

Purpose:

* initialize stack pointer
* jump to main()
* setup memory sections

---

#  Role in Embedded Firmware

```text id="flow4"
Power ON
    ↓
CPU executes startup assembly
    ↓
Initialize hardware
    ↓
Jump to C main()
```

---

#  Embedded Startup Flow Diagram

```text id="diag3"
+----------------------+
| Power ON Reset       |
+----------+-----------+
           ↓
+----------------------+
| Startup Assembly     |
| Stack Init           |
| Vector Table Setup   |
+----------+-----------+
           ↓
+----------------------+
| main() in C          |
+----------------------+
```

---

#  Inline Assembly in C

Used for direct CPU instructions.

```c id="inline1"
__asm__("NOP");
```

---

## Example

```c id="inline2"
__asm__(
    "MOV R0, #1\n"
    "ADD R1, R0, #2"
);
```

---

#  Advantages of Assembly Language

* very fast execution
* direct hardware control
* memory efficient
* precise timing control
* useful in embedded systems

---

#  Disadvantages

* difficult to write/debug
* hardware dependent
* less portable
* development time higher

---

#  Compiler vs Assembler

| Feature     | Compiler        | Assembler     |
| ----------- | --------------- | ------------- |
| Input       | C/C++ code      | Assembly code |
| Output      | Assembly/Object | Machine code  |
| Abstraction | High-level      | Low-level     |
| Ease        | Easier          | Harder        |

---

#  One-pass vs Two-pass Assembler

---

## One-pass Assembler

* faster
* limited symbol handling

---

## Two-pass Assembler

### Pass 1:

* collects labels/symbols

### Pass 2:

* generates machine code

---

#  Embedded Use Cases

* startup code
* interrupt service routines
* bootloader
* device drivers
* context switching in RTOS

---

#  Common Mistakes

---

## 1. Wrong register usage

---

## 2. Stack corruption

---

## 3. Infinite loops

```asm id="m1"
B .
```

---

## 4. Architecture mismatch

```text id="m2"
ARM assembly cannot run on x86
```

---

