# compiler

---

#  Introduction

A **compiler** is a software tool that converts **high-level programming language code (C/C++/Python)** into **machine-readable code (binary / executable)** that a CPU or microcontroller can execute.

In embedded systems, compilers are critical because they determine:

* performance
* memory usage
* binary size
* hardware compatibility

---

#  Definition

A compiler is a program that **translates source code into machine code through multiple stages such as preprocessing, compilation, assembly, and linking**.

---

#  Why Compiler is Important in Embedded Systems

* converts code into MCU instructions
* optimizes memory and speed
* generates firmware (.hex / .bin files)
* detects syntax and semantic errors
* enables cross-compilation (PC → ARM MCU)

---

#  Languages vs Compilation

---

##  C / C++

* Compiled language
* Needs full compilation before execution
* Used in embedded systems (microcontrollers)

---

##  Python

* Interpreted + partially compiled
* Uses bytecode + interpreter (CPython)
* Runs slower but flexible
* Used for scripting, testing, automation

---

#  Compilation Flow (C / C++)

```text id="flow1"
Source Code (.c/.cpp)
        ↓
Preprocessor
        ↓
Compiler
        ↓
Assembler
        ↓
Object File (.o)
        ↓
Linker
        ↓
Executable / Firmware (.elf/.hex/.bin)
```

---

#  Graphical Representation (C/C++)

```text id="diag1"
+----------------------+
|  Source Code         |
|  main.c / main.cpp   |
+----------+-----------+
           ↓
+----------------------+
| Preprocessor         |
| #include, #define    |
+----------+-----------+
           ↓
+----------------------+
| Compiler             |
| Converts to assembly |
+----------+-----------+
           ↓
+----------------------+
| Assembler           |
| Assembly → Machine   |
+----------+-----------+
           ↓
+----------------------+
| Linker              |
| Combines libraries  |
+----------+-----------+
           ↓
+----------------------+
| Binary (.hex/.bin)  |
| Runs on MCU/CPU     |
+----------------------+
```

---

#  Compilation Flow (Python)

```text id="flow2"
Source Code (.py)
        ↓
Python Interpreter (CPython)
        ↓
Bytecode (.pyc)
        ↓
Python Virtual Machine (PVM)
        ↓
Execution
```

---

#  Graphical Representation (Python)

```text id="diag2"
+----------------------+
| Python Code (.py)    |
+----------+-----------+
           ↓
+----------------------+
| Interpreter (CPython)|
+----------+-----------+
           ↓
+----------------------+
| Bytecode (.pyc)      |
+----------+-----------+
           ↓
+----------------------+
| Python VM (PVM)      |
+----------------------+
```

---

#  Syntax (C / C++ / Python)

---

##  C Syntax

```c id="c1"
#include <stdio.h>

int main() {
    printf("Hello Embedded\n");
    return 0;
}
```

---

##  C++ Syntax

```cpp id="cpp1"
#include <iostream>

int main() {
    std::cout << "Hello Embedded C++";
    return 0;
}
```

---

##  Python Syntax

```python id="py1"
print("Hello Embedded Python")
```

---

#  Compiler Installation (Tools)

---

##  For C / C++

### GCC (Linux / Windows / Embedded)

### Install (Ubuntu)

```bash id="install1"
sudo apt update
sudo apt install gcc g++
```

---

### Verify

```bash id="verify1"
gcc --version
g++ --version
```

---

##  Embedded Compiler (ARM)

* ARM GCC Toolchain
* Used for STM32, NXP, etc.

```bash id="arm1"
sudo apt install gcc-arm-none-eabi
```

---

##  Python Interpreter

```bash id="pyinstall"
sudo apt install python3
```

---

### Verify

```bash id="pyverify"
python3 --version
```

---

#  Compiler Working Steps (Internal)

```text id="flow3"
1. Lexical Analysis → tokens
2. Syntax Analysis → parse tree
3. Semantic Analysis → meaning check
4. Optimization → improve performance
5. Code Generation → machine code
```

---

#  Internal Compiler Architecture

```text id="diag3"
Source Code
     ↓
Lexical Analyzer
     ↓
Parser (Syntax Tree)
     ↓
Semantic Analyzer
     ↓
Optimizer
     ↓
Code Generator
     ↓
Machine Code
```

---

#  Embedded System Example Flow

```text id="flow4"
C Code (Firmware)
     ↓
ARM GCC Compiler
     ↓
.elf file
     ↓
.hex file
     ↓
Flashed to Microcontroller
     ↓
Hardware Execution
```

---

#  Real Embedded Example

```c id="ex1"
#define LED 1

int main() {
    while(1) {
        LED = 1;
    }
}
```

---

#  Compiler Errors

---

## 1. Syntax Error

```text id="err1"
missing ;
```

---

## 2. Semantic Error

```text id="err2"
type mismatch
```

---

## 3. Linker Error

```text id="err3"
undefined reference
```

---

#  Compiler vs Interpreter

| Feature     | Compiler      | Interpreter  |
| ----------- | ------------- | ------------ |
| Execution   | Fast          | Slow         |
| Translation | Whole program | Line by line |
| Output      | Binary file   | No binary    |
| Example     | C, C++        | Python       |

---

#  Execution Comparison

```text id="cmp1"
C/C++ → Compile → Run (fast)
Python → Interpret → Run (slower)
```

---

#  Advantages of Compiler

* fast execution
* optimized machine code
* error detection before execution
* suitable for embedded systems

---

#  Disadvantages

* longer build time
* complex toolchain
* platform dependent binaries

---

