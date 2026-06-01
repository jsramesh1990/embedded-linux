# linker-script

---

#  Introduction

In embedded systems, a **linker script** is a configuration file used by the linker to decide how different parts of a program are arranged in memory (Flash and RAM).

It defines:

* where code is placed
* where variables are stored
* memory regions of microcontroller
* stack and heap placement
* startup memory layout

Without a linker script, the program cannot correctly map into MCU memory.

---

#  Definition

A linker script is a file that tells the linker **how to map compiled object code into physical memory regions**.

---

#  Why Linker Script is Important

* MCU has fixed memory (Flash + RAM)
* We must control memory placement
* Required for bare-metal embedded systems
* Ensures correct startup and execution
* Defines stack/heap boundaries

---

#  Memory Mapping Concept

```text id="flow1"
Source Code (.c/.cpp)
        |
        v
Compiler → Object Files (.o)
        |
        v
Linker + Linker Script
        |
        v
Final Binary (.elf/.hex)
        |
        v
Flashed into Microcontroller
```

---

#  Embedded Memory Map

```text id="mem1"
+----------------------+
| Flash (ROM)        |
|----------------------|
| .text (code)       |
| .rodata (const)    |
+----------------------+
| RAM                |
|----------------------|
| .data              |
| .bss               |
| heap               |
| stack              |
+----------------------+
```

---

#  Basic Linker Script Structure

```ld id="ld1"
MEMORY
{
    FLASH (rx) : ORIGIN = 0x08000000, LENGTH = 512K
    RAM   (rwx) : ORIGIN = 0x20000000, LENGTH = 128K
}
```

---

#  Section Mapping

```ld id="ld2"
SECTIONS
{
    .text :
    {
        *(.text)
        *(.rodata)
    } > FLASH

    .data :
    {
        *(.data)
    } > RAM AT > FLASH

    .bss :
    {
        *(.bss)
    } > RAM
}
```

---

#  Working Flow of Linker Script

```text id="flow2"
Compiled object files
        |
        v
Linker reads linker script
        |
        v
Assigns memory regions
        |
        v
Maps .text → Flash
Maps .data → RAM
Maps .bss  → RAM
        |
        v
Final executable generated
```

---

#  Memory Placement Diagram

```text id="mem2"
FLASH (0x08000000)
+----------------------+
| .text (code)       |
| .rodata            |
+----------------------+

RAM (0x20000000)
+----------------------+
| .data              |
| .bss               |
| heap               |
| stack              |
+----------------------+
```

---

#  Real Embedded Example

## C Code

```c id="ex1"
int globalVar = 10;   // .data
int uninitVar;        // .bss

const int max = 100;  // .rodata

int main() {
    return 0;
}
```

---

## Linker Behavior

```text id="flow3"
globalVar → RAM (.data)
uninitVar → RAM (.bss)
max       → Flash (.rodata)
main()    → Flash (.text)
```

---

#  Stack & Heap Placement (Optional)

```ld id="ld3"
_estack = ORIGIN(RAM) + LENGTH(RAM);

.heap (NOLOAD) :
{
    . = ALIGN(8);
    _heap_start = .;
    . = . + 0x1000;
    _heap_end = .;
} > RAM

.stack (NOLOAD) :
{
    . = ALIGN(8);
    _stack_start = .;
    . = . + 0x1000;
    _stack_end = .;
} > RAM
```

---

#  Startup Flow Using Linker Script

```text id="flow4"
Power ON
   |
   v
Reset Handler
   |
   v
Copy .data Flash → RAM
   |
   v
Zero .bss section
   |
   v
Initialize stack pointer
   |
   v
Call main()
```

---

#  Sections in Detail

---

# 1. .text Section

* Stores program instructions
* Located in Flash
* Read-only

---

# 2. .data Section

* Initialized global variables
* Copied from Flash to RAM

---

# 3. .bss Section

* Uninitialized globals
* Automatically set to zero

---

# 4. Heap

* Dynamic memory
* Managed using malloc/free

---

# 5. Stack

* Function calls
* Local variables
* Interrupt handling

---

#  Common Mistakes

---

## 1. Wrong memory address

```ld id="m1"
RAM start incorrect → system crash
```

---

## 2. Heap and stack overlap

```text id="m2"
Heap grows up
Stack grows down
→ collision → crash
```

---

## 3. Missing .data copy

```text id="m3"
Global variables not initialized correctly
```

---


