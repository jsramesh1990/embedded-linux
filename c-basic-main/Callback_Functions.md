# Callback Functions in C (Embedded Systems)

## Table of Contents

1. [Introduction](#introduction)
2. [What is a Callback Function?](#what-is-a-callback-function)
3. [Why do we use callbacks?](#why-do-we-use-callbacks)
4. [How callback functions work](#how-callback-functions-work)
5. [Syntax](#syntax)
6. [Simple Example](#simple-example)
7. [Function Pointer Concept](#function-pointer-concept)
8. [Callback Flow Diagram](#callback-flow-diagram)
9. [Types of Callback Usage](#types-of-callback-usage)
10. [Use cases in Embedded Systems](#use-cases-in-embedded-systems)
11. [Advantages and Disadvantages](#advantages-and-disadvantages)
12. [Interview Questions](#interview-questions)
13. [Conclusion](#conclusion)

---

# Introduction

In embedded systems and firmware development, we often need **event-based execution** rather than sequential execution.

Callback functions help achieve this behavior.

---

# What is a Callback Function?

### Definition

> A callback function is a function that is passed as an argument to another function and is executed later when a specific event occurs.

---

## Simple Meaning

👉 “I will call your function when I need it.”

---

## Example Idea

```text id="cb1"
Driver → Event occurs → Calls your function
```

---

# Why do we use callbacks?

✔ Event-driven programming
✔ Hardware interrupt handling
✔ Driver abstraction
✔ Flexible API design
✔ RTOS task notifications

---

# How callback functions work

Callback works using **function pointers**.

---

## Flow

```text id="cb2"
Main code
   ↓
Pass function to driver
   ↓
Driver stores function pointer
   ↓
Event occurs
   ↓
Driver calls function (callback)
```

---

# Syntax

## Function pointer based callback

```c id="cb3"
void callback_function()
{
    printf("Callback executed");
}

void register_callback(void (*cb)())
{
    cb();
}
```

---

# Simple Example

```c id="cb4"
#include <stdio.h>

void my_callback()
{
    printf("Callback executed!\n");
}

void process(void (*cb)())
{
    printf("Processing...\n");
    cb();
}

int main()
{
    process(my_callback);
    return 0;
}
```

---

## Output

```text id="cb5"
Processing...
Callback executed!
```

---

# Function Pointer Concept

Callback = Function pointer in action.

---

## Example

```c id="cb6"
void (*ptr)();
ptr = my_callback;
ptr();
```

---

# Callback Flow Diagram

```text id="cb7"
Application
    ↓
Register callback
    ↓
Driver / Middleware
    ↓
Event Occurs (Interrupt / Data Ready)
    ↓
Callback Function Executes
```

---

# Types of Callback Usage

## 1. Synchronous Callback

Called immediately.

```text id="cb8"
Function → callback → return
```

---

## 2. Asynchronous Callback

Called later (event-driven).

```text id="cb9"
Interrupt → callback executes later
```

---

# Use cases in Embedded Systems

## 1. Interrupt Handling

```c id="cb10"
void ISR_callback()
{
    // handle interrupt
}
```

---

## 2. UART Driver

* Data received callback
* Transmission complete callback

---

## 3. RTOS Systems

* Task notifications
* Event flags
* Timers

---

## 4. Sensor Systems

* Data ready callback
* Threshold alert callback

---

## 5. Middleware / HAL

* Generic driver APIs
* Plug-and-play modules

---

# Real-Time Embedded Example

## UART Receive Callback

```c id="cb11"
void uart_rx_callback()
{
    printf("Data received\n");
}

void uart_register(void (*cb)())
{
    // store callback
    cb();
}
```

---

## Flow

```text id="cb12"
UART Interrupt → Data received → Callback executed
```

---

# Advantages

✔ Highly flexible design
✔ Supports event-driven programming
✔ Improves modularity
✔ Used in drivers and RTOS
✔ Reduces tight coupling

---

# Disadvantages

✘ Hard to debug
✘ Complex flow control
✘ Risk of incorrect callback usage
✘ Memory/function pointer overhead
✘ Can reduce code readability

---

# Interview Questions

## What is a callback function?

A callback function is a function passed as an argument to another function and executed later when a specific event occurs.

---

## Why do we use callback functions?

To implement event-driven programming and improve modularity in embedded systems and drivers.

---

## What is relation between function pointer and callback?

Callbacks are implemented using function pointers in C.

---

## Where are callbacks used in embedded systems?

* Interrupt handling
* UART/SPI drivers
* RTOS event handling
* Sensor systems

---

## Difference between normal function and callback?

| Normal Function | Callback          |
| --------------- | ----------------- |
| Called directly | Called indirectly |
| Fixed flow      | Event-driven      |

---

# Most Asked Interview Question

## Explain callback function with example.

A callback function is a function passed to another function as an argument and executed later when a specific event occurs. It is implemented using function pointers in C. For example, in embedded systems, a UART driver can call a user-defined callback when data is received. This helps in event-driven programming, improves modularity, and reduces tight coupling between modules.

---

# Quick Revision

```text id="cb13"
Callback = Function passed as argument

Used for:
- Interrupts
- Drivers
- RTOS
- Event handling

Implemented using:
Function pointers
```

---

# Conclusion

Callback functions are a key concept in embedded systems for building flexible, event-driven architectures. They are widely used in drivers, RTOS, and interrupt handling. Understanding callbacks is essential for firmware development and system-level programming interviews.
