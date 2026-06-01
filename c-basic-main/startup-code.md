# startup-code

---

#  Introduction

In embedded systems, **startup code** is the first code that runs after a microcontroller reset (power-on or reset button).

It is responsible for preparing the system before `main()` starts executing.

Without startup code, the program cannot run correctly because RAM, stack, and memory sections are not initialized.

---

#  Definition

Startup code is a **low-level initialization routine** that prepares memory, sets up stack pointer, initializes data sections, and finally calls `main()`.

---

#  Startup Execution Flow

```text id="flow1"
Power ON / Reset
        |
        v
Reset Handler (Startup Code)
        |
        v
Initialize Stack Pointer
        |
        v
Copy .data from Flash → RAM
        |
        v
Clear .bss section
        |
        v
System Initialization (clock, peripherals)
        |
        v
Call main()
```

---

#  Why Startup Code is Required

* RAM is not automatically initialized
* .data must be copied from Flash
* .bss must be cleared
* Stack pointer must be set
* CPU must be configured properly
* Clock system must be initialized

---

#  Embedded Memory Initialization

```text id="mem1"
FLASH (ROM)
+----------------------+
| .text (code)         |
| .rodata              |
| initial .data        |
+----------------------+

RAM
+----------------------+
| .data (runtime)      |
| .bss (zeroed)        |
| heap                 |
| stack                |
+----------------------+
```

---

#  Startup Code Flow Diagram

```text id="flow2"
RESET
  |
  v
Set Stack Pointer
  |
  v
Copy .data (Flash → RAM)
  |
  v
Zero .bss section
  |
  v
System Clock Setup
  |
  v
Peripheral Init
  |
  v
Call main()
```

---

#  Typical Startup Code (Simplified C)

```c id="ex1"
extern int main();

extern unsigned int _sidata;
extern unsigned int _sdata;
extern unsigned int _edata;

extern unsigned int _sbss;
extern unsigned int _ebss;

void Reset_Handler(void) {

    unsigned int *src = &_sidata;
    unsigned int *dst = &_sdata;

    // Copy .data section
    while (dst < &_edata) {
        *dst++ = *src++;
    }

    // Clear .bss section
    dst = &_sbss;
    while (dst < &_ebss) {
        *dst++ = 0;
    }

    main();  // jump to application
}
```

---

#  Startup Flow Explanation

```text id="flow3"
Flash contains initial values
        |
        v
Startup copies values into RAM
        |
        v
.bss is cleared to zero
        |
        v
System becomes ready
        |
        v
main() executes
```

---

#  Stack Initialization

Stack pointer is usually set in assembly:

```asm id="asm1"
LDR SP, =_estack
```

---

#  Stack Setup Diagram

```text id="stack1"
High Memory Address
+----------------------+
| Stack Pointer (SP)  |
| Stack grows ↓       |
+----------------------+
Low Memory Address
```

---

#  Vector Table Role in Startup

Startup code is pointed by the **vector table**.

```text id="flow4"
Reset Vector → Reset_Handler → Startup Code → main()
```

---

#  Vector Table Example

```c id="ex2"
__attribute__((section(".isr_vector")))
void (*vector_table[])(void) = {
    (void*) &_estack,
    Reset_Handler,
};
```

---

#  Embedded System Responsibilities

Startup code handles:

* memory initialization
* stack setup
* clock configuration
* peripheral reset
* interrupt vector setup
* call to main

---

# ⚠ Common Issues

---

## 1. .data not copied correctly

```text id="err1"
→ global variables get garbage values
```

---

## 2. .bss not cleared

```text id="err2"
→ uninitialized variables contain random data
```

---

## 3. Stack pointer not set

```text id="err3"
→ system crash immediately
```

---

## 4. Wrong linker script integration

```text id="err4"
→ memory mapping failure
```

---

#  Startup Sequence Summary

```text id="flow5"
RESET
  |
  v
Hardware setup
  |
  v
Memory initialization
  |
  v
Clock configuration
  |
  v
Peripheral setup
  |
  v
main()
```

---

#  Real-Time Embedded Example

```c id="ex3"
int globalVar = 10;  // placed in .data

int main() {
    while(1) {
        // system runs
    }
}
```

Startup ensures:

* globalVar is copied correctly
* system clock is stable
* stack is ready

---

#  Advantages

| Advantage             | Explanation                    |
| --------------------- | ------------------------------ |
| System initialization | Prepares MCU for execution     |
| Memory setup          | Ensures correct RAM state      |
| Hardware setup        | Configures clock & peripherals |
| Predictable execution | Ensures safe startup           |

---

#  Disadvantages

| Issue                  | Explanation                           |
| ---------------------- | ------------------------------------- |
| Complex low-level code | Requires deep MCU knowledge           |
| Hardware dependent     | Different MCUs need different startup |
| Debug difficulty       | Early boot errors are hard to trace   |

---


