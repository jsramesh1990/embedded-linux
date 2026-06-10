# Reset Handler

## Table of Contents

1. [Introduction](#introduction)
2. [What is a Reset Handler?](#what-is-a-reset-handler)
3. [Why Do We Need a Reset Handler?](#why-do-we-need-a-reset-handler)
4. [Boot Sequence Overview](#boot-sequence-overview)
5. [Position of Reset Handler in Startup Flow](#position-of-reset-handler-in-startup-flow)
6. [How Reset Handler Works](#how-reset-handler-works)
7. [Responsibilities of Reset Handler](#responsibilities-of-reset-handler)
8. [Copying .data Section](#copying-data-section)
9. [Initializing .bss Section](#initializing-bss-section)
10. [Vector Table and Reset Handler](#vector-table-and-reset-handler)
11. [Reset Handler Example](#reset-handler-example)
12. [What Happens When Code Hits It?](#what-happens-when-code-hits-it)
13. [Real-Time Examples](#real-time-examples)
14. [Reset Handler in ARM Cortex-M](#reset-handler-in-arm-cortex-m)
15. [Advantages and Disadvantages](#advantages-and-disadvantages)
16. [Interview Questions](#interview-questions)
17. [Quick Revision](#quick-revision)
18. [Conclusion](#conclusion)

---

# Introduction

When a microcontroller powers ON:

```text
main() DOES NOT RUN FIRST
```

A lot of initialization must happen before:

```c
int main()
{
}
```

can execute.

The first software function executed is usually:

```text
Reset_Handler()
```

---

# What is a Reset Handler?

## Definition

The **Reset Handler** is the first firmware function executed after a processor reset.

Its job is to:

```text
Initialize Memory
Initialize Runtime Environment
Prepare CPU
Call main()
```

---

Without a Reset Handler:

```text
Global Variables Not Initialized
Stack Not Ready
Program Cannot Run Correctly
```

---

# Why Do We Need a Reset Handler?

Consider:

```c
int speed = 100;
int counter;
```

Before `main()` runs:

```text
speed must become 100
counter must become 0
```

Who does this?

```text
Reset_Handler
```

---

It prepares the environment required by C/C++ programs.

---

# Boot Sequence Overview

Power ON Sequence:

```text
Power ON
    ↓
CPU Reset
    ↓
Vector Table Read
    ↓
Reset_Handler()
    ↓
Initialize Memory
    ↓
Initialize Runtime
    ↓
main()
```

---

# Position of Reset Handler in Startup Flow

```text
Flash Memory
│
├── Vector Table
│
├── Reset_Handler
│
├── Startup Code
│
└── Application Code
```

---

Execution Order:

```text
1. Power ON

2. CPU Fetches Reset Vector

3. Calls Reset_Handler()

4. Memory Initialization

5. System Initialization

6. main()
```

---

# How Reset Handler Works

Typical Steps:

```text
Step 1:
Initialize Stack Pointer

Step 2:
Copy .data Section

Step 3:
Clear .bss Section

Step 4:
Initialize Hardware

Step 5:
Call main()
```

---

# Responsibilities of Reset Handler

## 1. Initialize Stack Pointer

CPU requires a valid stack.

Stack pointer loaded from:

```text
Vector Table
```

---

## 2. Copy Initialized Variables

Example:

```c
int speed = 100;
```

Stored initially in:

```text
Flash
```

Must be copied to:

```text
RAM
```

---

## 3. Clear BSS

Example:

```c
int counter;
```

Must become:

```text
0
```

before `main()`.

---

## 4. Hardware Initialization

May configure:

```text
Clock
PLL
FPU
Memory
Caches
```

---

## 5. Call main()

Finally:

```c
main();
```

---

# Copying .data Section

Example:

```c
int rpm = 5000;
```

---

Stored in Flash:

```text
Flash
rpm = 5000
```

---

Runtime Location:

```text
RAM
rpm = 5000
```

---

Reset Handler performs:

```c
memcpy(
    &_sdata,
    &_sidata,
    &_edata - &_sdata
);
```

---

Visualization:

```text
Flash
 ↓
Copy
 ↓
RAM
```

---

# Initializing .bss Section

Example:

```c
int counter;
```

Stored in:

```text
.bss
```

---

Must become:

```text
counter = 0
```

---

Reset Handler performs:

```c
memset(
    &_sbss,
    0,
    &_ebss - &_sbss
);
```

---

Visualization:

```text
Allocate RAM
      ↓
Fill with Zero
      ↓
.bss Initialized
```

---

# Vector Table and Reset Handler

ARM Cortex-M Vector Table:

```c
__Vectors[] =
{
    Initial_Stack_Pointer,
    Reset_Handler,
    NMI_Handler,
    HardFault_Handler
};
```

---

Entry 0:

```text
Initial Stack Pointer
```

Entry 1:

```text
Reset_Handler Address
```

---

When reset occurs:

CPU automatically executes:

```text
Reset_Handler()
```

---

# Reset Handler Example

```c
extern int main();

void Reset_Handler(void)
{
    Copy_Data_Section();

    Clear_BSS_Section();

    main();

    while(1);
}
```

---

Real Startup Code:

```c
void Reset_Handler(void)
{
    SystemInit();

    __libc_init_array();

    main();

    while(1);
}
```

---

# What Happens When Code Hits It?

This is one of the most important interview questions.

---

Example Firmware:

```c
int speed = 100;

int counter;

int main()
{
    while(1)
    {
    }
}
```

---

Power ON

```text
CPU Reset
```

---

CPU Reads Vector Table

```text
Load Stack Pointer

Get Reset_Handler Address
```

---

CPU Jumps To

```text
Reset_Handler()
```

---

Reset Handler Executes

```text
1. Setup Stack

2. Copy .data
   speed = 100

3. Clear .bss
   counter = 0

4. Initialize Clock

5. Initialize Runtime

6. Call main()
```

---

Now:

```text
main()
```

starts execution.

---

### Interview Answer

**What happens when code hits the Reset Handler?**

Immediately after a reset, the CPU loads the initial stack pointer and jumps to the Reset Handler using the address stored in the vector table. The Reset Handler initializes the runtime environment by copying the `.data` section from Flash to RAM, clearing the `.bss` section, configuring system clocks and hardware, and then calling `main()`. Every embedded application starts through the Reset Handler before executing user code.

---

# Real-Time Examples

## Example 1: STM32 MCU

Power ON:

```text
Reset
 ↓
Reset_Handler
 ↓
SystemInit()
 ↓
main()
```

---

## Example 2: Automotive ECU

Ignition ON:

```text
ECU Reset
 ↓
Reset_Handler
 ↓
Clock Setup
 ↓
CAN Init
 ↓
Application Start
```

---

## Example 3: Sensor Controller

Power Applied:

```text
Reset_Handler
 ↓
ADC Initialization
 ↓
Sensor Task Start
```

---

## Example 4: Embedded Linux Bootloader

Although Linux uses a more complex startup sequence, bootloaders such as:

* U-Boot

perform similar initialization before handing control to the kernel.

---

# Reset Handler in ARM Cortex-M

Most interview questions target ARM.

---

Boot Sequence:

```text
Flash Address
0x00000000
      ↓
Vector Table
      ↓
Initial SP
      ↓
Reset_Handler
```

---

CPU Automatically:

```text
Loads MSP

Loads Reset Handler Address

Jumps To Reset_Handler
```

---

No software intervention required.

---

# Advantages and Disadvantages

## Advantages

✔ Initializes Runtime Environment

✔ Prepares RAM

✔ Initializes Global Variables

✔ Configures Hardware

✔ Ensures Correct Program Startup

---

## Disadvantages

✘ Startup Time Overhead

✘ Incorrect Implementation Causes Boot Failure

✘ Memory Initialization Can Be Time Consuming

---

# Interview Questions

## What is a Reset Handler?

The Reset Handler is the first firmware function executed after a processor reset. It initializes memory, hardware, and the runtime environment before calling `main()`.

---

## Does main() Execute First?

No.

Execution starts from:

```text
Reset_Handler()
```

---

## What Does Reset Handler Initialize?

```text
Stack Pointer
.data Section
.bss Section
System Clock
Runtime Libraries
```

---

## Where is Reset Handler Address Stored?

Inside the:

```text
Vector Table
```

---

## Why is .bss Cleared?

The C standard requires uninitialized global and static variables to start at zero.

---

## Why is .data Copied?

Initialized variables are stored in Flash and must be copied to RAM because they are writable.

---

## What Happens When Code Hits It?

The CPU jumps to the Reset Handler, which initializes memory and hardware, then transfers execution to `main()`.

---

# Most Asked Interview Question

## Explain the Complete Boot Sequence of an ARM Cortex-M MCU.

```text
Power ON
    ↓
CPU Reset
    ↓
Read Vector Table
    ↓
Load Initial Stack Pointer
    ↓
Load Reset_Handler Address
    ↓
Execute Reset_Handler
    ↓
Copy .data Section
    ↓
Clear .bss Section
    ↓
SystemInit()
    ↓
Initialize C Runtime
    ↓
main()
```

---

# Interview Answer (2-Minute Version)

The Reset Handler is the first function executed after a microcontroller reset. Its address is stored in the vector table, and the CPU jumps to it automatically after loading the initial stack pointer. The Reset Handler prepares the execution environment by copying initialized variables from Flash to RAM, clearing the `.bss` section, configuring clocks and hardware, initializing runtime libraries, and finally calling `main()`. Without the Reset Handler, global variables would not be initialized correctly and the application could not run reliably.

---

# Quick Revision

```text
Reset Handler

Purpose:
First Function After Reset

Responsibilities:
✔ Setup Stack
✔ Copy .data
✔ Clear .bss
✔ Initialize Hardware
✔ Call main()

Boot Flow:

Power ON
   ↓
CPU Reset
   ↓
Vector Table
   ↓
Reset_Handler
   ↓
SystemInit()
   ↓
main()

Important:
main() NEVER runs first

Most Important:
Reset_Handler prepares the complete
runtime environment before user code starts.
```

---

# Conclusion

The Reset Handler is the foundation of every embedded application startup. It is responsible for preparing memory, initializing hardware, configuring the runtime environment, and launching the application through `main()`. A strong understanding of the Reset Handler, vector table, `.data` initialization, and `.bss` clearing is essential for Embedded Systems, Firmware Development, Automotive ECUs, RTOS development, ARM Cortex-M programming, and technical interviews.
